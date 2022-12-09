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
                --namespace kube-default

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
```

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add metallb https://metallb.github.io/metallb
helm repo add jetstack https://charts.jetstack.io
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo add argo https://argoproj.github.io/argo-helm
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
  --set controller.name=nginx-ingress

helm upgrade --install istio-base istio/base \
  --version 1.16.0 \
  --namespace istio-system

helm upgrade --install istio-istiod istio/istiod \
  --version 1.16.0 \
  --namespace istio-system

helm upgrade --install istio-gateway istio/gateway \
  --version 1.16.0 \
  --namespace istio-gateway-system

helm upgrade --install kiali kiali/kiali-server \
  --version 1.60.0 \
  --namespace kiali-system \
  --set istio_namespace=istio-system \
  --set auth.strategy=anonymous \
  --set server.port=8080 \
  --set deployment.service_type=LoadBalancer

helm upgrade --install minio minio/minio \
  --version 5.0.1 \
  --namespace minio-system \
  --set replicas=3 \
  --set consoleService.type=LoadBalancer \
  --set consoleService.port=8080 \
  --set service.type=LoadBalancer \
  --set service.port=9000
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

# Credentials
MINIO_CONSOLE_IP=$(kubectl get svc -n minio-system minio-console -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
MINIO_USER=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootUser}}' | base64 -d)
MINIO_PASSWORD=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootPassword}}' | base64 -d)

echo "addresses:"
echo -e " - prometheus:\t\thttp://${PROM_IP}:8080"
echo -e " - grafana:\t\thttp://${GRAFANA_IP}:8080"
echo -e " - kiali:\t\thttp://${KIALI_IP}:8080"
echo -e " - argocd:\t\thttp://${AROCD_IP}:8080"
echo -e " - minio:\t\thttp://${MINIO_IP}:9000"
echo -e " - minio (console):\thttp://${MINIO_CONSOLE_IP}:8080"
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

```bash
cat <<ENDOFMESSAGE | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
ENDOFMESSAGE
```

```bash
cat <<ENDOFMESSAGE | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: dupa.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 8080
  ingressClassName: nginx
ENDOFMESSAGE
```
