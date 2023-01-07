```bash
cat > fileConfigs.yaml << "EndOfMessage"
fileConfigs:
  01_sources.conf: |-
    <source>
      @type kafka_group
      
      brokers redpanda:9093
      consumer_group logs
    
      topics  logs
      format json
    </source>
  02_filters.conf: |-
  03_dispatch.conf: |-
  04_outputs.conf: |-
    <match **>
      @type elasticsearch
      hosts "https://elasticsearch-aio:9200"
      user elastic
      password elastic
      ssl_verify false
      
      index_name logs
    </match>
EndOfMessage

helm upgrade --install fluentd fluent/fluentd \
  --version 0.3.9 \
  --namespace logging-system \
  --set kind=Deployment \
  --set replicaCount=3 \
  --set plugins[0]=fluent-plugin-kafka \
  --set plugins[1]=fluent-plugin-elasticsearch \
  -f fileConfigs.yaml
  
# Or

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
        topics => ["logs"]
        codec => "json"
        group_id => "logstash"
      }
    }
    filter {
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
    }
    output {
      elasticsearch { 
        hosts => "https://elasticsearch-aio:9200"
        user => "elastic"
        password => "elastic"
        index => "logs"
        ssl_certificate_verification => false
      }
    }
EndOfMessage

helm upgrade --install logstash elastic/logstash \
  --version 8.5.1 \
  --namespace logging-system \
  --set terminationGracePeriod=0 \
  -f logstash.yaml
```

# Examples

 - [Nginx Ingress](examples/nginx-ingress.yaml)
 - [Istio Ingress](examples/istio-ingress)
