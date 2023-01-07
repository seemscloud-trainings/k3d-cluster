```bash
cat << "EndOFMessage" | /bin/bash
# Addresses
MINIO_IP=$(kubectl get svc -n minio-system minio -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
KIALI_IP=$(kubectl get svc -n kiali-system kiali -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
AROCD_IP=$(kubectl get svc -n argocd-system argocd-server -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
PROM_IP=$(kubectl get svc -n metrics-system prometheus-server -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
GRAFANA_IP=$(kubectl get svc -n metrics-system grafana -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
JAEGER_IP=$(kubectl get svc -n tracing-system jaeger-query -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
NGINX_INGRESS_IP=$(kubectl get svc -n nginx-ingress-system nginx-ingess-nginx-ingress -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
ISTIO_INGRESS_IP=$(kubectl get svc -n istio-gateway-system istio-ingress-gateway -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')

# Credentials
MINIO_CONSOLE_IP=$(kubectl get svc -n minio-system minio-console -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
MINIO_USER=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootUser}}' | base64 -d)
MINIO_PASSWORD=$(kubectl get secrets -n minio-system minio -o go-template='{{.data.rootPassword}}' | base64 -d)

clear

echo "addresses:"
echo -e " - kiali:\t\thttp://${KIALI_IP}:8080"
echo -e " - argocd:\t\thttp://${AROCD_IP}:8080"
echo -e " - jeager:\t\thttp://${JAEGER_IP}:8080"
echo -e " - grafana:\t\thttp://${GRAFANA_IP}:8080"
echo -e " - prometheus:\t\thttp://${PROM_IP}:8080"
echo -e " - minio:\t\thttp://${MINIO_IP}:9000"
echo -e " - minio (console):\thttp://${MINIO_CONSOLE_IP}:8080"
echo
echo -e " - nginx ingress:\t${NGINX_INGRESS_IP}"
echo -e " - istio ingress:\t${ISTIO_INGRESS_IP}"
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
