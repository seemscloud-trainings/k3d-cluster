## CD / CD

```bash
helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443
```

## Accessibility

```bash
helm upgrade --install metallb metallb/metallb \
  --version 0.13.4 \
  --namespace metallb-system

helm upgrade --install nginx-ingess nginx-stable/nginx-ingress \
  --version 0.15.2 \
  --namespace nginx-ingress-system \
  --set controller.name=nginx-ingress \
  --set controller.nginxDebug=true \
  --set-string controller.config.entries.http2=true \
  --set controller.kind=daemonset
```

## Tools

```bash
helm upgrade --install keda kedacore/keda \
  --version 2.8.2 \
  --namespace keda-system
  
helm upgrade --install cert-manager jetstack/cert-manager \
  --version v1.7.2 \
  --namespace cert-manager-system \
  --set installCRDs=true

helm upgrade --install minio minio/minio \
  --version 5.0.1 \
  --namespace minio-system \
  --set replicas=2 \
  --set consoleService.type=LoadBalancer \
  --set consoleService.port=8080 \
  --set persistence.enabled=false \
  --set service.type=LoadBalancer \
  --set service.port=9000 \
  --set persistence.enabled=false \
  --set extraVolumes[0].name=emptydir \
  --set extraVolumeMounts[0].name=emptydir \
  --set extraVolumeMounts[0].mountPath=/export
```

## Metrics / Tracing

```bash
helm upgrade --install prometheus prometheus-community/prometheus \
  --version 19.0.1 \
  --namespace metrics-system \
  --set alertmanager.enabled=false \
  --set prometheus-pushgateway.enabled=false \
  --set kube-state-metrics.enabled=false \
  --set server.persistentVolume.enabled=false \
  --set server.service.servicePort=8080 \
  --set server.service.type=LoadBalancer

helm upgrade --install grafana grafana/grafana \
  --version 6.46.1 \
  --namespace metrics-system \
  --set service.type=LoadBalancer \
  --set service.port=8080 \
  --set 'grafana\.ini'.'auth\.anonymous'.enabled=true \
  --set 'grafana\.ini'.'auth\.anonymous'.org_role=Admin \
  --set 'grafana\.ini'.auth.disable_login_form=true \
  --set datasources.'datasources\.yaml'.apiVersion=1 \
  --set datasources.'datasources\.yaml'.datasources[0].name=Prometheus \
  --set datasources.'datasources\.yaml'.datasources[0].type=prometheus \
  --set datasources.'datasources\.yaml'.datasources[0].url=http://prometheus-server:8080 \
  --set datasources.'datasources\.yaml'.datasources[0].isDefault=true

helm upgrade --install jaeger jaegertracing/jaeger \
  --version 0.65.2 \
  --namespace tracing-system \
  --set query.service.type=LoadBalancer \
  --set query.service.port=8080
```

### Istio

```bash
helm upgrade --install istio-base istio/base \
  --version 1.16.0 \
  --namespace istio-system

helm upgrade --install istio-cni istio/cni \
  --version 1.16.0 \
  --namespace kube-system

helm upgrade --install istio-istiod istio/istiod \
  --version 1.16.0 \
  --namespace istio-system

helm upgrade --install istio-ingress-gateway istio/gateway \
  --version 1.16.0 \
  --namespace istio-gateway-system \
  --set service.type=LoadBalancer
  
helm upgrade --install istio-egress-gateway istio/gateway \
  --version 1.16.0 \
  --namespace istio-gateway-system \
  --set service.type=ClusterIP

helm upgrade --install kiali kiali/kiali-server \
  --version 1.60.0 \
  --namespace kiali-system \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer \
  --set external_services.grafana.enabled=true \
  --set external_services.grafana.in_cluster_url=http://grafana.metrics-system:8080 \
  --set external_services.prometheus.enabled=true \
  --set external_services.prometheus.url=http://prometheus-server.metrics-system:8080 \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.in_cluster_url=http://jaeger-query.tracing-system:16685
```

## Logging

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
