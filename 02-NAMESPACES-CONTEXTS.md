```bash
kubectl config set-context k3d-${CLUSTER_NAME}-default \
                --cluster k3d-${CLUSTER_NAME} \
                --user admin@k3d-${CLUSTER_NAME} \
                --namespace default

kubectl config use-context k3d-${CLUSTER_NAME}-default

kubectl config delete-context k3d-${CLUSTER_NAME}
```

```bash
for NAMESPACE in `cat "k3d-${CLUSTER_NAME}-namespaces.txt"` ; do
    kubectl config set-context "k3d-${CLUSTER_NAME}-${NAMESPACE}" \
        --cluster "k3d-${CLUSTER_NAME}" \
        --user "admin@k3d-${CLUSTER_NAME}" \
        --namespace "${NAMESPACE}"
done
```

```bash
for NAMESPACE in `cat "k3d-${CLUSTER_NAME}-namespaces.txt"` ; do
    kubectl create namespace "${NAMESPACE}"
done
```
