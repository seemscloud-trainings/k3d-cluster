```bash
helm upgrade \
  --version 0.22.0 \
  --namespace vault-system \
  --install vault hashicorp/vault \
  --set "ui.enabled=true" \
  --set "ui.serviceType=LoadBalancer" \
  --set "ui.annotations.networking\.\gke\.io\/load-balancer-type=Internal" \
  --set "server.ha.enabled=true" \
  --set "server.ha.raft.enabled=true" \
  --set "server.ha.raft.setNodeId=true" \
  --set "server.replicas=3" \
  --set "server.dataStorage.enabled=false" \
  --set "server.volumes[0].name=emptydir" \
  --set "server.volumeMounts[0].mountPath=/vault/data" \
  --set "server.volumeMounts[0].name=emptydir"

kubectl exec -ti vault-0 -- vault operator init -key-shares="1" -key-threshold="1" -format="json"
kubectl exec -ti vault-0 -- vault operator unseal

kubectl exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200

kubectl exec -ti vault-1 -- vault operator unseal
kubectl exec -ti vault-2 -- vault operator unseal

kubectl exec -ti vault-0 -- vault status
kubectl exec -ti vault-1 -- vault status
kubectl exec -ti vault-2 -- vault status
```

```bash
helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443

helm upgrade --install nginx-ingess nginx-stable/nginx-ingress \
  --version 0.15.2 \
  --namespace nginx-ingress-system \
  --set controller.name=nginx-ingress \
  --set controller.nginxDebug=true \
  --set-string controller.config.entries.http2=true \
  --set controller.kind=daemonset

helm upgrade --install keda kedacore/keda \
  --version 2.8.2 \
  --namespace keda-system
```

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
```

```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --version 7.17.3 \
  --namespace tracing-system \
  --set masterService=elasticsearch-aio \
  --set nodeGroup=aio \
  --set replicas=3 \
  --set minimumMasterNodes=3 \
  --set maxUnavailable=3 \
  --set persistence.enabled=false \
  --set terminationGracePeriod=0 \
  --set service.type=LoadBalancer \
  --set-string extraEnvs[0].name=xpack.security.enabled \
  --set-string extraEnvs[0].value=false
  
helm upgrade --install kibana elastic/kibana \
  --version 7.17.3 \
  --namespace tracing-system \
  --set fullnameOverride=kibana \
  --set elasticsearchHosts="http://elasticsearch-aio:9200" \
  --set replicas=1 \
  --set service.type=LoadBalancer
  
helm upgrade --install jaeger jaegertracing/jaeger \
  --version 0.65.2 \
  --namespace tracing-system \
  --set query.service.type=LoadBalancer \
  --set query.service.port=8080 \
  --set storage.type=elasticsearch \
  --set provisionDataStore.cassandra=false \
  --set storage.elasticsearch.host=elasticsearch-aio \
  --set storage.elasticsearch.usePassword=false
```

```bash
helm upgrade --install istio-base istio/base \
  --version 1.17.0 \
  --namespace istio-system \
  --set defaultRevision=1-17-0

helm upgrade --install istio-istiod-1-17-0 istio/istiod \
  --set revision=1-17-0 \
  --set global.configValidation=true \
  --set pilot.autoscaleEnabled=true \
  --set pilot.autoscaleMin=3 \
  --set pilot.autoscaleMax=3 \
  --set pilot.replicaCount=3 \
  --version 1.17.0 \
  --namespace istio-system
  
helm upgrade --install istio-istiod-1-17-2 istio/istiod \
  --set revision=1-17-2 \
  --set global.configValidation=true \
  --set pilot.autoscaleEnabled=true \
  --set pilot.autoscaleMin=3 \
  --set pilot.autoscaleMax=3 \
  --set pilot.replicaCount=3 \
  --version 1.17.2 \
  --namespace istio-system

helm upgrade --install istio-ingress-gateway-1-17-0 istio/gateway \
  --version 1.17.0 \
  --namespace istio-gateway-system \
  --set revision=1-17-0 \
  --set service.type=LoadBalancer
  
helm upgrade --install istio-ingress-gateway-1-17-2 istio/gateway \
  --version 1.17.2 \
  --namespace istio-gateway-system \
  --set revision=1-17-2 \
  --set service.type=LoadBalancer
  
helm upgrade --install istio-egress-gateway-1-17-0 istio/gateway \
  --version 1.17.0 \
  --namespace istio-gateway-system \
  --set revision=1-17-0 \
  --set service.type=ClusterIP

helm upgrade --install istio-egress-gateway-1-17-2 istio/gateway \
  --version 1.17.2 \
  --namespace istio-gateway-system \
  --set revision=1-17-2 \
  --set service.type=ClusterIP

helm upgrade --install kiali-1-17-0 kiali/kiali-server \
  --version 1.66.0 \
  --namespace kiali-system \
  --set fullnameOverride=kiali-1-17-0 \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer \
  --set kiali_feature_flags.istio_upgrade_action=true \
  --set external_services.istio.istiod_deployment_name=istiod-1-17-0 \
  --set external_services.istio.config_map_name=istio-1-17-0 \
  --set external_services.istio.istio_sidecar_injector_config_map_name=istio-sidecar-injector-1-17-0 \
  --set external_services.grafana.enabled=true \
  --set external_services.grafana.in_cluster_url=http://grafana.metrics-system:8080 \
  --set external_services.prometheus.enabled=true \
  --set external_services.prometheus.url=http://prometheus-server.metrics-system:8080 \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.in_cluster_url=http://jaeger-query.tracing-system:16685
  
helm upgrade --install kiali-1-17-2 kiali/kiali-server \
  --version 1.66.0 \
  --namespace kiali-system \
  --set fullnameOverride=kiali-1-17-2 \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer \
  --set kiali_feature_flags.istio_upgrade_action=true \
  --set external_services.istio.istiod_deployment_name=istiod-1-17-2 \
  --set external_services.istio.config_map_name=istio-1-17-2 \
  --set external_services.istio.istio_sidecar_injector_config_map_name=istio-sidecar-injector-1-17-2 \
  --set external_services.grafana.enabled=true \
  --set external_services.grafana.in_cluster_url=http://grafana.metrics-system:8080 \
  --set external_services.prometheus.enabled=true \
  --set external_services.prometheus.url=http://prometheus-server.metrics-system:8080 \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.in_cluster_url=http://jaeger-query.tracing-system:16685
```

```bash
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
