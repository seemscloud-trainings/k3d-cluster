```bash
helm upgrade --install kiali-1-17-0 kiali/kiali-server \
  --version 1.66.0 \
  --namespace kiali-system \
  \
  --set fullnameOverride=kiali-1-17-0 \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer \
  --set kiali_feature_flags.istio_upgrade_action=true \
  --set external_services.istio.component_status.enabled=true \
  --set external_services.istio.component_status.components[0].app_label=istio-ingress-gateway-1-17-0 \
  --set external_services.istio.component_status.components[0].is_core=true \
  --set external_services.istio.component_status.components[0].is_proxy=true \
  --set external_services.istio.component_status.components[0].namespace=istio-gateway-system \
  --set external_services.istio.istiod_deployment_name=istiod-1-17-0 \
  --set external_services.istio.config_map_name=istio-1-17-0 \
  --set external_services.istio.istio_sidecar_injector_config_map_name=istio-sidecar-injector-1-17-0 \
  --set external_services.grafana.enabled=true \
  --set external_services.grafana.in_cluster_url=http://grafana.metrics-system:8080 \
  --set external_services.prometheus.enabled=true \
  --set external_services.prometheus.url=http://prometheus-server.metrics-system:8080 \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.in_cluster_url=http://jaeger-query.istio-tracing-system:16685
  
helm upgrade --install kiali-1-17-2 kiali/kiali-server \
  --version 1.66.0 \
  --namespace kiali-system \
  \
  --set fullnameOverride=kiali-1-17-2 \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer \
  --set kiali_feature_flags.istio_upgrade_action=true \
  --set external_services.istio.component_status.enabled=true \
  --set external_services.istio.component_status.components[0].app_label=istio-ingress-gateway-1-17-2 \
  --set external_services.istio.component_status.components[0].is_core=true \
  --set external_services.istio.component_status.components[0].is_proxy=true \
  --set external_services.istio.component_status.components[0].namespace=istio-gateway-system \
  --set external_services.istio.istiod_deployment_name=istiod-1-17-2 \
  --set external_services.istio.config_map_name=istio-1-17-2 \
  --set external_services.istio.istio_sidecar_injector_config_map_name=istio-sidecar-injector-1-17-2 \
  --set external_services.grafana.enabled=true \
  --set external_services.grafana.in_cluster_url=http://grafana.metrics-system:8080 \
  --set external_services.prometheus.enabled=true \
  --set external_services.prometheus.url=http://prometheus-server.metrics-system:8080 \
  --set external_services.tracing.enabled=true \
  --set external_services.tracing.in_cluster_url=http://jaeger-query.istio-tracing-system:16685
```
