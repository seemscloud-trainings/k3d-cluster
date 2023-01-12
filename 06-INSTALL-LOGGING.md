```bash
helm upgrade --install redpanda redpanda/redpanda \
  --version 2.4.0 \
  --namespace logging-system \
  --set fullnameOverride=redpanda \
  --set serviceAccount.create=true \
  --set statefulset.replicas=3 \
  --set storage.persistentVolume.enabled=false \
  --set external.enabled=false \
  --set resources.cpu.cores=1 \
  --set resources.memory.container.max=2.5Gi

helm upgrade --install redpanda-console redpanda/console \
  --version 0.3.3 \
  --namespace logging-system \
  --set serviceAccount.create=true \
  --set service.type=LoadBalancer \
  --set console.config.kafka.brokers[0]=redpanda-0.redpanda:9093 \
  --set console.config.kafka.brokers[1]=redpanda-1.redpanda:9093 \
  --set console.config.kafka.brokers[2]=redpanda-2.redpanda:9093
```

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
  --set fullnameOverride=logstash \
  -f logstash.yaml
```

```bash
helm upgrade --install logging-operator banzaicloud-stable/logging-operator \
  --version 3.17.10 \
  --namespace logging-system
  
helm upgrade --install logging-operator-logging banzaicloud-stable/logging-operator-logging \
  --version 3.17.10 \
  --namespace logging-system \
  --set tls.enabled=false \
  --set clusterFlows[0].name=logs \
  --set clusterFlows[0].spec.globalOutputRefs[0]=kafka \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.mapped_time_key="@timestamp" \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[0]=event.eventTime \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[1]=event.lastTimestamp \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[2]=event.firstTimestamp \
  --set clusterOutputs[0].name=kafka \
  --set clusterOutputs[0].spec.kafka.brokers="redpanda:9093" \
  --set clusterOutputs[0].spec.kafka.format.type=json \
  --set clusterOutputs[0].spec.kafka.default_topic=logs \
  --set clusterOutputs[0].spec.kafka.buffer.flush_mode=interval \
  --set clusterOutputs[0].spec.kafka.buffer.flush_interval=5s \
  --set clusterOutputs[0].spec.kafka.buffer.flush_thread_count=4 \
  --set clusterOutputs[0].spec.kafka.buffer.overflow_action=block \
  --set clusterOutputs[0].spec.kafka.buffer.retry_forever=true \
  --set clusterOutputs[0].spec.kafka.buffer.retry_type=periodic \
  --set clusterOutputs[0].spec.kafka.buffer.retry_wait=5s \
  --set fluentbit.security.securityContext.privileged=true \
  --set fluentd.resources.limits.cpu=1 \
  --set fluentd.resources.limits.memory=3Gi \
  --set fluentd.resources.requests.cpu=10m \
  --set fluentd.resources.requests.memory=100Mi \
  --set fluentd.scaling.replicas=1 \
  --set fluentd.disablePvc=true \
  --set fluentd.bufferStorageVolume.emptyDir.sizeLimit=10Gi \
  --set fluentbit.tolerations[0].effect=NoSchedule \
  --set fluentbit.tolerations[0].key=dedicated \
  --set fluentbit.tolerations[0].value=control-plane
```

# Beats

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
