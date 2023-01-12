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
