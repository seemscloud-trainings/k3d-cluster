```bash
helm upgrade \
  --version 0.22.0 \
  --namespace vault-system \
  --install vault hashicorp/vault \
  --set "ui.enabled=true" \
  --set "ui.serviceType=LoadBalancer" \
  --set "ui.annotations.networking\.\gke\.io\/load-balancer-type=Internal" \
  --set "server.ha.enabled=true" \
  --set "server.ha.raft.enabled=true" \
  --set "server.ha.raft.setNodeId=true" \
  --set "server.replicas=3" \
  --set "server.dataStorage.enabled=false" \
  --set "server.volumes[0].name=emptydir" \
  --set "server.volumeMounts[0].mountPath=/vault/data" \
  --set "server.volumeMounts[0].name=emptydir"

kubectl exec -ti vault-0 -- vault operator init -key-shares="1" -key-threshold="1" -format="json"
kubectl exec -ti vault-0 -- vault operator unseal

kubectl exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200

kubectl exec -ti vault-1 -- vault operator unseal
kubectl exec -ti vault-2 -- vault operator unseal

kubectl exec -ti vault-0 -- vault status
kubectl exec -ti vault-1 -- vault status
kubectl exec -ti vault-2 -- vault status
```

```bash
helm upgrade --install nginx-ingess nginx-stable/nginx-ingress \
  --version 0.15.2 \
  --namespace nginx-ingress-system \
  --set controller.name=nginx-ingress \
  --set controller.nginxDebug=true \
  --set-string controller.config.entries.http2=true \
  --set controller.kind=daemonset

helm upgrade --install keda kedacore/keda \
  --version 2.8.2 \
  --namespace keda-system
```

```bash
helm upgrade --install minio minio/minio \
  --version 5.0.1 \
  --namespace minio-system \
  --set replicas=2 \
  --set consoleService.type=LoadBalancer \
  --set consoleService.port=8080 \
  --set persistence.enabled=false \
  --set service.type=LoadBalancer \
  --set service.port=9000 \
  --set extraVolumes[0].name=emptydir \
  --set extraVolumeMounts[0].name=emptydir \
  --set extraVolumeMounts[0].mountPath=/export
```
