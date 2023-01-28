```bash
helm upgrade --install logging-operator banzaicloud-stable/logging-operator \
  --version 3.17.10 \
  --namespace logging-system
```

### To Kafka

```bash
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

### To GELF

```bash
helm upgrade --install logging-operator-logging banzaicloud-stable/logging-operator-logging \
  --version 3.17.10 \
  --namespace logging-system \
  --set tls.enabled=false \
  --set clusterFlows[0].name=logs \
  --set clusterFlows[0].spec.globalOutputRefs[0]=gelf \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.mapped_time_key="@timestamp" \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[0]=event.eventTime \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[1]=event.lastTimestamp \
  --set clusterFlows[0].spec.filters[0].kube_events_timestamp.timestamp_fields[2]=event.firstTimestamp \
  --set clusterOutputs[0].name=gelf \
  --set clusterOutputs[0].spec.gelf.host=graylog-input-gelf.default.svc \
  --set clusterOutputs[0].spec.gelf.port=12201 \
  --set clusterOutputs[0].spec.gelf.protocol=tcp \
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
