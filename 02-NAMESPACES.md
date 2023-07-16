```bash
for NAMESPACE in `cat "k3d-${CLUSTER_NAME}-namespaces.txt"` ; do
    kubectl create namespace "${NAMESPACE}"
done
```
