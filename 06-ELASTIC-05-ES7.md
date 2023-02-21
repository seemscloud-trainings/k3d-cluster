```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --version 7.17.3 \
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
  --set extraEnvs[0].name=ELASTIC_PASSWORD \
  --set extraEnvs[0].value=elastic
```

```bash
# kubectl  exec -it elasticsearch-aio-0 -n logging-system -- ./bin/elasticsearch-setup-passwords auto
```

```bash
helm upgrade --install kibana elastic/kibana \
  --version 7.17.3 \
  --namespace logging-system \
  --set fullnameOverride=kibana \
  --set elasticsearchHosts="https://elasticsearch-aio:9200" \
  --set replicas=1 \
  --set service.type=LoadBalancer \
  -f kibana.yaml \
  --set extraVolumes[0].name=tls \
  --set extraVolumes[0].secret.secretName=elasticsearch-ca-ends-tls \
  --set extraVolumeMounts[0].name=tls \
  --set extraVolumeMounts[0].mountPath=/usr/share/kibana/config/certs/ca.crt \
  --set extraVolumeMounts[0].subPath=ca.crt \
  --set extraVolumeMounts[0].readOnly=true \
  --set extraEnvs[0].name=ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES \
  --set extraEnvs[0].value=/usr/share/kibana/config/certs/ca.crt \
  --set extraEnvs[1].name=ELASTICSEARCH_USERNAME \
  --set extraEnvs[1].value=elastic \
  --set extraEnvs[2].name=ELASTICSEARCH_PASSWORD \
  --set extraEnvs[2].value=elastic
```
