```bash
export CLUSTER_NAME=seems
```

```bash
k3d cluster delete "${CLUSTER_NAME}"
```

```bash
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
