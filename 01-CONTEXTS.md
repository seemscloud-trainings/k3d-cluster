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
