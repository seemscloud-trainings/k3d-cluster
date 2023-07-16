```bash
helm upgrade --install argocd argo/argo-cd \
  --version 5.16.1 \
  --namespace argocd-system \
  --set fullnameOverride=argocd \
  \
  --set configs.params."server\.disable\.auth"=true \
  --set configs.params."server\.insecure"=true \
  --set server.service.type=LoadBalancer \
  --set server.service.servicePortHttp=8080 \
  --set server.service.servicePortHttps=8443 \
  --set repoServer.replicas=3 \
  --set applicationSet.replicas=3 \
  --set controller.replicas=3 \
  --set server.replicas=3 \
  --set redis.enabled=false \
  --set redis-ha.enabled=true
```

```bash
helm upgrade --install argo-events argo/argo-events \
  --version 2.2.0 \
  --namespace argo-system \
  --set fullnameOverride=argo-events \
  --set crds.install=true \
  --set crds.keep=false \
  --set controller.name=controller-manager \
  --set controller.replicas=1 \
  --set controller.metrics.enabled=true \
  --set controller.serviceAccount.create=true \
  --set webhook.enabled=false \
  --set webhook.name=events-webhook \
  --set webhook.replicas=1 \
  --set webhook.serviceAccoun.create=true

helm upgrade --install argo-workflows-minio minio/minio \
  --version 5.0.1 \
  --namespace argo-system \
  --set fullnameOverride=argo-workflows-minio \
  --set replicas=2 \
  --set consoleService.type=LoadBalancer \
  --set consoleService.port=8080 \
  --set service.type=LoadBalancer \
  --set service.port=9000 \
  --set persistence.enabled=false \
  --set extraVolumes[0].name=emptydir \
  --set extraVolumeMounts[0].name=emptydir \
  --set extraVolumeMounts[0].mountPath=/export \
  --set rootUser=root \
  --set rootPassword="Root123\!@#" \
  --set users[0].accessKey=user01 \
  --set users[0].secretKey="User123\!@#" \
  --set users[0].policy=user01 \
  --set policies[0].name=user01 \
  --set policies[0].statements[0].resources[0]="arn:aws:s3:::user01" \
  --set policies[0].statements[0].resources[1]="arn:aws:s3:::user01/*" \
  --set policies[0].statements[0].actions[0]="s3:*" \
  --set buckets[0].name=user01 \
  --set buckets[0].versioning=true \
  --set buckets[0].objectlocking=true

helm upgrade --install argo-workflows argo/argo-workflows \
  --version 0.25.1 \
  --namespace argo-system \
  --set fullnameOverride=argo-workflows \
  --set crds.install=true \
  --set crds.keep=false \
  --set controller.name=controller \
  --set controller.metricsConfig.enabled=true \
  --set controller.telemetryConfig.enabled=true \
  --set controller.serviceAccount.enabled=true \
  --set controller.logging.level=info \
  --set server.enabled=true \
  --set server.name=server \
  --set server.replicas=1 \
  --set server.secure=false \
  --set server.serviceAccount.create=true \
  --set server.logging.level=info \
  --set server.serviceType=LoadBalancer \
  --set server.servicePort=8080 \
  --set server.extraArgs[0]="--auth-mode=server" \
  --set extraObjects[0].apiVersion=v1 \
  --set extraObjects[0].kind=Secret \
  --set extraObjects[0].metadata.name=argo-workflows-minio-creds \
  --set extraObjects[0].data.accessKey=dXNlcjAx \
  --set extraObjects[0].data.secretKey=VXNlcjEyMyFAIw== \
  --set useDefaultArtifactRepo=true \
  --set artifactRepository.archiveLogs=true \
  --set artifactRepository.s3.endpoint=argo-workflows-minio:9000 \
  --set artifactRepository.s3.bucket=user01 \
  --set artifactRepository.s3.insecure=true \
  --set artifactRepository.s3.accessKeySecret.name=argo-workflows-minio-creds \
  --set artifactRepository.s3.accessKeySecret.key=accessKey \
  --set artifactRepository.s3.secretKeySecret.name=argo-workflows-minio-creds \
  --set artifactRepository.s3.secretKeySecret.key=secretKey
```
