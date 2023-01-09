## Logstash

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
        bootstrap_servers => "redpanda:9093"
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
        password => "7oRPxyP0GtQiMA93CWOE"
        ssl_certificate_verification => false
        index => "%{[@metadata][target_index]}"
      }
    }
EndOfMessage

helm upgrade --install logstash elastic/logstash \
  --version 8.5.1 \
  --namespace logging-system \
  --set terminationGracePeriod=0 \
  -f logstash.yaml
```

## Metricbeat

```bash
cat > metricbeat.yaml << "EndOfMessage"
daemonset:
  metricbeatConfig:
    metricbeat.yml: |
      metricbeat.modules:
      - module: kubernetes
        metricsets:
          - container
          - node
          - pod
          - system
          - volume
        period: 10s
        host: "${NODE_NAME}"
        hosts: ["https://${NODE_NAME}:10250"]
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        ssl.verification_mode: "none"
        processors:
        - add_kubernetes_metadata: ~
      - module: kubernetes
        enabled: true
        metricsets:
          - event
      - module: system
        period: 10s
        metricsets:
          - cpu
          - load
          - memory
          - network
          - process
          - process_summary
        processes: ['.*']
        process.include_top_n:
          by_cpu: 5
          by_memory: 5
      - module: system
        period: 1m
        metricsets:
          - filesystem
          - fsstat
        processors:
        - drop_event.when.regexp:
            system.filesystem.mount_point: '^/(sys|cgroup|proc|dev|etc|host|lib)($|/)'
      output.kafka:
        hosts: ["redpanda-0.redpanda:9093", "redpanda-1.redpanda:9093", "redpanda-2.redpanda:9093"]
        topic: metrics
        required_acks: 1
        compression: gzip
EndOfMessage

helm upgrade --install metricbeat elastic/metricbeat \
  --version 7.17.3 \
  --namespace logging-system \
  --set daemonset.enabled=true \
  --set deployment.enabled=false \
  -f metricbeat.yaml
```
