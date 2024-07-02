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
