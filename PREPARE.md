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
helm repo add redpanda https://charts.redpanda.com
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
