```bash
cat > fileConfigs.yaml << "EndOfMessage"
fileConfigs:
  01_sources.conf: |-
    <source>
      @type kafka_group
      
      brokers redpanda:9093
      consumer_group logs
    
      topics  logs
      format json
    </source>
  02_filters.conf: |-
  03_dispatch.conf: |-
  04_outputs.conf: |-
    <match **>
      @type elasticsearch
      hosts "https://elasticsearch-aio:9200"
      user elastic
      password elastic
      ssl_verify false
      
      index_name logs
    </match>
EndOfMessage

helm upgrade --install fluentd fluent/fluentd \
  --version 0.3.9 \
  --namespace logging-system \
  --set kind=Deployment \
  --set replicaCount=3 \
  --set plugins[0]=fluent-plugin-kafka \
  --set plugins[1]=fluent-plugin-elasticsearch \
  -f fileConfigs.yaml
  
# Or

cat >logstash.yaml <<"EndOfMessage"
logstashConfig:
  logstash.yml: |-
    http.host: "0.0.0.0"
  pipelines.yml: |-
    - pipeline.id: logstash
      path.config: "/usr/share/logstash/pipeline/logstash.conf"

logstashPipeline:
  logstash.conf: |-
    input {
      kafka {
        consumer_threads => 1
        bootstrap_servers => "redpanda:9093"
        topics => ["logs"]
        codec => "json"
        group_id => "logstash"
      }
    }
    filter {
        ruby {
            code =>'
                sub_field = "[kubernetes][labels]"
                target_field = "app"

                source_field = "#{sub_field}[#{target_field}]"
    
                obj = event.get(source_field)
    
                if obj.instance_of?(Array)
                    event.set("#{sub_field}[#{target_field}_arr]", obj.to_s)
                elsif obj.instance_of?(Hash)
                    event.set("#{sub_field}[#{target_field}_obj]", obj)
                elsif obj.to_i.to_s == obj || obj.instance_of?(Integer)
                    event.set("#{sub_field}[#{target_field}_int]", obj.to_i)
                elsif obj.to_f.to_s == obj || obj.instance_of?(Float)
                    event.set("#{sub_field}[#{target_field}_float]", obj.to_f)
                else
                    event.set("#{sub_field}[#{target_field}_str]", obj)
                end
    
                event.remove(source_field)
            '
        }
    }
    output {
      elasticsearch { 
        hosts => "https://elasticsearch-aio:9200"
        user => "elastic"
        password => "elastic"
        index => "logs"
        ssl_certificate_verification => false
      }
    }
EndOfMessage

helm upgrade --install logstash elastic/logstash \
  --version 8.5.1 \
  --namespace logging-system \
  --set terminationGracePeriod=0 \
  -f logstash.yaml
```

```bash  
METALLB_CIDR=`docker network inspect k3d-${CLUSTER_NAME} | jq -r ".[0].IPAM.Config[0].Subnet" | awk -F'.' '{print $1"."$2"."$3"."128"/"25}'`

cat <<EndOfMessage | kubectl -n metallb-system apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: docker-host
spec:
  addresses:
    - "${METALLB_CIDR}"
  avoidBuggyIPs: true
EndOfMessage

cat <<EndOfMessage | kubectl -n metallb-system apply -f -
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: docker-host
spec:
  ipAddressPools:
  - docker-host
EndOfMessage
```

# Workspace

```bash
cat << "EndOFMessage" | /bin/bash
# Addresses
MINIO_IP=$(kubectl get svc -n minio-system minio -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
KIALI_IP=$(kubectl get svc -n kiali-system kiali -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
AROCD_IP=$(kubectl get svc -n argocd-system argocd-server -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
PROM_IP=$(kubectl get svc -n metrics-system prometheus-server -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
GRAFANA_IP=$(kubectl get svc -n metrics-system grafana -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
JAEGER_IP=$(kubectl get svc -n tracing-system jaeger-query -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
NGINX_INGRESS_IP=$(kubectl get svc -n nginx-ingress-system nginx-ingess-nginx-ingress -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
ISTIO_INGRESS_IP=$(kubectl get svc -n istio-gateway-system istio-ingress-gateway -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')

# Credentials
MINIO_CONSOLE_IP=$(kubectl get svc -n minio-system minio-console -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
MINIO_USER=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootUser}}' | base64 -d)
MINIO_PASSWORD=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootPassword}}' | base64 -d)

clear

echo "addresses:"
echo -e " - kiali:\t\thttp://${KIALI_IP}:8080"
echo -e " - argocd:\t\thttp://${AROCD_IP}:8080"
echo -e " - jeager:\t\thttp://${JAEGER_IP}:8080"
echo -e " - grafana:\t\thttp://${GRAFANA_IP}:8080"
echo -e " - prometheus:\t\thttp://${PROM_IP}:8080"
echo -e " - minio:\t\thttp://${MINIO_IP}:9000"
echo -e " - minio (console):\thttp://${MINIO_CONSOLE_IP}:8080"
echo
echo -e " - nginx ingress:\t${NGINX_INGRESS_IP}"
echo -e " - istio ingress:\t${ISTIO_INGRESS_IP}"
echo
echo "credentials:"
echo -e " - minio:\t${MINIO_USER} ${MINIO_PASSWORD}"

EndOFMessage
```

```bash
kubectl port-forward service/nginx-ingess-nginx-ingress \
  --namespace nginx-ingress-system \
  8080:80
```

# Examples

 - [Nginx Ingress](examples/nginx-ingress.yaml)
 - [Istio Ingress](examples/istio-ingress)
