```bash
export CLUSTER_NAME=seems
```

```bash
k3d cluster delete "${CLUSTER_NAME}"
```

```bash
export K3D_FIX_MOUNTS=1

k3d cluster create \
  --servers 3 \
  --agents 6 \
  --k3s-arg "--cluster-cidr=10.100.0.0/16@server:*" \
  --k3s-arg "--service-cidr=10.200.0.0/16@server:*" \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --no-lb ${CLUSTER_NAME}
```

```bash
for i in `seq 0 2` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done

for i in `seq 0 5` ; do
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/generic=true
  kubectl label node k3d-${CLUSTER_NAME}-agent-${i} node-role.kubernetes.io/worker=true
done
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
