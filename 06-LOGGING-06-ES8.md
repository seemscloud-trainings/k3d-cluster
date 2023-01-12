```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --version 8.5.1 \
  --namespace logging-system \
  --set masterService=elasticsearch-aio \
  --set nodeGroup=aio \
  --set replicas=3 \
  --set minimumMasterNodes=3 \
  --set maxUnavailable=3 \
  --set persistence.enabled=false \
  --set terminationGracePeriod=0 \
  --set service.type=LoadBalancer \
  --set service.transportPortName=https \
  --set protocol=https \
  --set extraVolumes[0].name=tls \
  --set extraVolumes[0].secret.secretName=elasticsearch-ca-ends-tls \
  --set extraVolumeMounts[0].name=tls \
  --set extraVolumeMounts[0].mountPath=/usr/share/elasticsearch/config/certs \
  --set extraVolumeMounts[0].readOnly=true \
  -f elasticsearch.yaml \
  --set createCert=false \
  --set secret.password=elastic
```

```bash
helm upgrade --install kibana elastic/kibana \
  --version 8.5.1 \
  --namespace logging-system \
  --set fullnameOverride=kibana \
  --set elasticsearchHosts="https://elasticsearch-aio:9200" \
  --set replicas=1 \
  --set service.type=LoadBalancer \
  -f kibana.yaml \
  --set elasticsearchCertificateSecret=elasticsearch-ca-ends-tls \
  --set elasticsearchCredentialSecret=elasticsearch-aio-credentials
```

## Cleanup Kibana

```bash
helm -n logging-system uninstall kibana

kubectl -n logging-system delete cm kibana-helm-scripts
kubectl -n logging-system delete secrets kibana-es-token
kubectl -n logging-system delete serviceaccounts pre-install-kibana
kubectl -n logging-system delete roles pre-install-kibana
kubectl -n logging-system delete rolebindings pre-install-kibana
kubectl -n logging-system delete job pre-install-kibana
kubectl -n logging-system delete job post-delete-kibana
```
