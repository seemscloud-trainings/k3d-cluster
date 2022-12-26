# Setup

```bash
CLUSTER_NAME="seems"
```

```bash
k3d cluster delete "${CLUSTER_NAME}"

SERVERS=3
AGENTS=3

k3d cluster create \
  --servers "${SERVERS}" \
  --agents "${AGENTS}" \
  --k3s-arg "--cluster-cidr=10.100.0.0/16@server:*" \
  --k3s-arg "--service-cidr=10.200.0.0/16@server:*" \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --k3s-arg "--disable=metrics-server@server:*" \
  --no-lb \
  ${CLUSTER_NAME}

for i in `seq 0 $(("${SERVERS}"-1))` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done
# NoSchedule- untaint node
```

```bash
kubectl config set-context k3d-${CLUSTER_NAME}-default \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace default

kubectl config use-context k3d-${CLUSTER_NAME}-default

kubectl config delete-context k3d-${CLUSTER_NAME}
```

```bash
kubectl config set-context k3d-${CLUSTER_NAME}-kube-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace kube-system

kubectl config set-context k3d-${CLUSTER_NAME}-metallb-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace metallb-system

kubectl config set-context k3d-${CLUSTER_NAME}-cert-manager-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace cert-manager-system

kubectl config set-context k3d-${CLUSTER_NAME}-nginx-ingress-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace nginx-ingress-system

kubectl config set-context k3d-${CLUSTER_NAME}-argocd-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace argocd-system

kubectl config set-context k3d-${CLUSTER_NAME}-keda-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace keda-system

kubectl config set-context k3d-${CLUSTER_NAME}-kiali-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace kiali-system

kubectl config set-context k3d-${CLUSTER_NAME}-istio-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace istio-system

kubectl config set-context k3d-${CLUSTER_NAME}-istio-gateway-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace istio-gateway-system

kubectl config set-context k3d-${CLUSTER_NAME}-minio-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace minio-system

kubectl config set-context k3d-${CLUSTER_NAME}-metrics-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace metrics-system

kubectl config set-context k3d-${CLUSTER_NAME}-tracing-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace tracing-system

kubectl config set-context k3d-${CLUSTER_NAME}-logging-system \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace logging-system
```

```bash
kubectl create namespace metallb-system
kubectl create namespace cert-manager-system
kubectl create namespace nginx-ingress-system
kubectl create namespace argocd-system
kubectl create namespace keda-system
kubectl create namespace istio-system
kubectl create namespace kiali-system
kubectl create namespace istio-gateway-system
kubectl create namespace minio-system

kubectl create namespace metrics-system
kubectl create namespace tracing-system
kubectl create namespace logging-system
```

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add elastic https://helm.elastic.co
helm repo add banzaicloud-stable https://kubernetes-charts.banzaicloud.com
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add elastic https://helm.elastic.co
helm repo add metallb https://metallb.github.io/metallb
helm repo add jetstack https://charts.jetstack.io
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo add argo https://argoproj.github.io/argo-helm
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo add kedacore https://kedacore.github.io/charts
helm repo add kiali https://kiali.org/helm-charts
helm repo add minio https://charts.min.io
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts

helm repo update
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

helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443

helm upgrade --install keda kedacore/keda \
  --version 2.8.2 \
  --namespace keda-system

helm upgrade --install metallb metallb/metallb \
  --version 0.13.4 \
  --namespace metallb-system

helm upgrade --install cert-manager jetstack/cert-manager \
  --version v1.7.2 \
  --namespace cert-manager-system \
  --set installCRDs=true

helm upgrade --install nginx-ingess nginx-stable/nginx-ingress \
  --version 0.15.2 \
  --namespace nginx-ingress-system \
  --set controller.name=nginx-ingress \
  --set controller.nginxDebug=true \
  --set-string controller.config.entries.http2=true \
  --set controller.kind=daemonset

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

helm upgrade --install jaeger jaegertracing/jaeger \
  --version 0.65.2 \
  --namespace tracing-system \
  --set query.service.type=LoadBalancer \
  --set query.service.port=8080
```

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
  --set createCert=true \
  --set terminationGracePeriod=0 \
  --set secret.password=elastic

helm uninstall -n logging-system kibana

kubectl -n logging-system delete cm kibana-helm-scripts
kubectl -n logging-system delete secrets kibana-es-token
kubectl -n logging-system delete serviceaccounts pre-install-kibana
kubectl -n logging-system delete roles  pre-install-kibana
kubectl -n logging-system delete rolebindings pre-install-kibana
kubectl -n logging-system delete job pre-install-kibana
kubectl -n logging-system delete job post-delete-kibana

helm upgrade --install kibana elastic/kibana \
  --version 8.5.1 \
  --namespace logging-system \
  --set fullnameOverride=kibana \
  --set elasticsearchHosts="https://elasticsearch-aio:9200" \
  --set elasticsearchCertificateSecret=elasticsearch-aio-certs \
  --set elasticsearchCredentialSecret=elasticsearch-aio-credentials \
  --set replicas=1 \
  --set service.type=LoadBalancer
```
  
```bash
helm upgrade --install logging-operator banzaicloud-stable/logging-operator \
  --version 3.17.10 \
  --namespace logging-system

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
