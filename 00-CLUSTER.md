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
  --no-lb \
  ${CLUSTER_NAME}

#   --k3s-arg "--disable=metrics-server@server:*" \

for i in `seq 0 $(("${SERVERS}"-1))` ; do
  kubectl taint nodes "k3d-${CLUSTER_NAME}-server-$i" dedicated=control-plane:NoSchedule
done
# NoSchedule- untaint node
```
