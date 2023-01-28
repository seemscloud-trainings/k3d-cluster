```bash
cat > elasticsearch.yaml << "EndOfMessage"
esConfig:
  elasticsearch.yml: |
    cluster.name: "elastic"
    network.host: 0.0.0.0

    xpack.security.enabled: false
    xpack.security.http.ssl.enabled: false
    xpack.security.transport.ssl.enabled: false
EndOfMessage
cat > kibana.yaml << "EndOfMessage"
kibanaConfig:
  kibana.yml: |
    server.host: "0.0.0.0"
    server.shutdownTimeout: "5s"
    elasticsearch.ssl.verificationMode: "certificate"
EndOfMessage
```

```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --version 7.17.3 \
  --set masterService=elasticsearch-aio \
  --set nodeGroup=aio \
  --set replicas=3 \
  --set minimumMasterNodes=3 \
  --set maxUnavailable=3 \
  --set persistence.enabled=false \
  --set terminationGracePeriod=0 \
  --set service.type=LoadBalancer \
  --set service.transportPortName=http-transport \
  --set protocol=http \
  -f elasticsearch.yaml \
  --set extraEnvs[0].name=ELASTIC_PASSWORD \
  --set extraEnvs[0].value=elastic

helm upgrade --install mongodb bitnami/mongodb \
  --version 13.6.6 \
  --set useStatefulSet=true \
  --set architecture=replicaset \
  --set replicaCount=2 \
  --set auth.usernames[0]=graylog \
  --set auth.passwords[0]=graylog \
  --set auth.databases[0]=graylog \
  --set persistence.enabled=false
```

```bash
helm upgrade --install graylog kongz/graylog \
  --version 2.2.0 \
  --set tags.install-elasticsearch=false \
  --set tags.install-mongodb=false \
  --set graylog.replicas=2 \
  --set graylog.terminationGracePeriodSeconds=1 \
  --set graylog.persistence.enabled=false \
  --set graylog.elasticsearch.version=7 \
  --set graylog.elasticsearch.hosts=http://elasticsearch-aio:9200 \
  --set graylog.mongodb.uri="mongodb://graylog:graylog@mongodb-0.mongodb-headless:27017/graylog?replicaSet=rs0,mongodb://graylog:graylog@mongodb-1.mongodb-headless:27017/graylog?replicaSet=rs1" \
  --set graylog.service.type=LoadBalancer \
  --set graylog.input.tcp.service.name=graylog-input-gelf \
  --set graylog.input.tcp.ports[0].port=12201
````
