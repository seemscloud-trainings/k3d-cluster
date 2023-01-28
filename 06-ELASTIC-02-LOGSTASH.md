```bash
cat >logstash.yaml <<"EndOfMessage"
logstashConfig:
  logstash.yml: |-
    http.host: "0.0.0.0"
  pipelines.yml: |-
    - pipeline.id: logstash
      path.config: "/usr/share/logstash/pipeline/logstash.conf"

logstashPipeline:
  logstash.conf: |-
    input {
      kafka {
        consumer_threads => 1
        bootstrap_servers => "redpanda.logging-system.svc:9093,redpanda.logging-system.svc:9093,redpanda.logging-system.svc:9093"
        topics => ["logs", "metrics"]
        codec => "json"
        group_id => "logstash"
        decorate_events => "extended"
      }
    }
    filter {
        json {
            source => "message"
            target => "@message"
            skip_on_invalid_json => true
            tag_on_failure => ["fail/json"]
        }
        if [@metadata][kafka][topic] == "logs" {
          ruby {
              code =>'
                  sub_field = "[kubernetes][labels]"
                  target_field = "app"

                  source_field = "#{sub_field}[#{target_field}]"
      
                  obj = event.get(source_field)
      
                  if obj.instance_of?(Array)
                      event.set("#{sub_field}[#{target_field}_arr]", obj.to_s)
                  elsif obj.instance_of?(Hash)
                      event.set("#{sub_field}[#{target_field}_obj]", obj)
                  elsif obj.to_i.to_s == obj || obj.instance_of?(Integer)
                      event.set("#{sub_field}[#{target_field}_int]", obj.to_i)
                  elsif obj.to_f.to_s == obj || obj.instance_of?(Float)
                      event.set("#{sub_field}[#{target_field}_float]", obj.to_f)
                  else
                      event.set("#{sub_field}[#{target_field}_str]", obj)
                  end
      
                  event.remove(source_field)
              '
          }
          ruby {
              code =>'
                  sub_field = "[kubernetes][labels]"
                  target_field = "istio"

                  source_field = "#{sub_field}[#{target_field}]"
      
                  obj = event.get(source_field)
      
                  if obj.instance_of?(Array)
                      event.set("#{sub_field}[#{target_field}_arr]", obj.to_s)
                  elsif obj.instance_of?(Hash)
                      event.set("#{sub_field}[#{target_field}_obj]", obj)
                  elsif obj.to_i.to_s == obj || obj.instance_of?(Integer)
                      event.set("#{sub_field}[#{target_field}_int]", obj.to_i)
                  elsif obj.to_f.to_s == obj || obj.instance_of?(Float)
                      event.set("#{sub_field}[#{target_field}_float]", obj.to_f)
                  else
                      event.set("#{sub_field}[#{target_field}_str]", obj)
                  end
      
                  event.remove(source_field)
              '
          }
        
          mutate { add_field => { "[@metadata][target_index]" => "logs" } }
        } else if [@metadata][kafka][topic] == "metrics" {
          mutate { add_field => { "[@metadata][target_index]" => "metrics" } }
        }
    }
    output {
      elasticsearch { 
        hosts => "https://elasticsearch-aio:9200"
        user => "elastic"
        password => "elastic"
        ssl_certificate_verification => false
        index => "%{[@metadata][target_index]}"
      }
    }
EndOfMessage

helm upgrade --install logstash elastic/logstash \
  --version 8.5.1 \
  --namespace logging-system \
  --set replicas=1 \
  --set terminationGracePeriod=0 \
  --set fullnameOverride=logstash \
  -f logstash.yaml
```
