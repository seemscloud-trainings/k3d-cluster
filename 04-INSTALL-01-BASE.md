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