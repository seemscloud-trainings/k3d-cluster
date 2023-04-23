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
