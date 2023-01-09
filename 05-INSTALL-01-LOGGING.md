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
