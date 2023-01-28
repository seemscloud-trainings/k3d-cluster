```bash
kubectl apply -f - << "EndOfMessage"
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: elasticsearch-ca
  namespace: logging-system
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: elasticsearch-ca
  namespace: logging-system
spec:
  secretName: elasticsearch-ca-tls
  isCA: true
  commonName: "Root CA"
  duration: 86400h
  renewBefore: 24h
  privateKey:
    algorithm: RSA
    encoding: PKCS8
    size: 2048
  issuerRef:
    name: elasticsearch-ca
    kind: Issuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: elasticsearch-ca-ends
  namespace: logging-system
spec:
  ca:
    secretName: elasticsearch-ca-tls
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: elasticsearch-ca-ends
  namespace: logging-system
spec:
  secretName: elasticsearch-ca-ends-tls
  commonName: elasticsearch-aio
  dnsNames:
    - elasticsearch-aio
    - elasticsearch-aio-headless
  duration: 8640h
  renewBefore: 1h
  privateKey:
    algorithm: RSA
    encoding: PKCS8
    size: 2048
  issuerRef:
    name: elasticsearch-ca-ends
    kind: Issuer
    group: cert-manager.io
EndOfMessage
```

```bash
cat > elasticsearch.yaml << "EndOfMessage"
esConfig:
  elasticsearch.yml: |
    cluster.name: "elastic"
    network.host: 0.0.0.0
    
    xpack.security.enabled: true
    
    xpack.security.http.ssl.enabled: true
    xpack.security.http.ssl.key: certs/tls.key
    xpack.security.http.ssl.certificate: certs/tls.crt
    xpack.security.http.ssl.certificate_authorities: certs/ca.crt
    xpack.security.http.ssl.verification_mode: certificate
    
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.key: certs/tls.key
    xpack.security.transport.ssl.certificate: certs/tls.crt
    xpack.security.transport.ssl.certificate_authorities: certs/ca.crt
    xpack.security.transport.ssl.verification_mode: certificate
EndOfMessage
```

```bash
cat > kibana.yaml << "EndOfMessage"
kibanaConfig:
  kibana.yml: |
    server.host: "0.0.0.0"
    server.shutdownTimeout: "5s"
    elasticsearch.ssl.verificationMode: "certificate"
EndOfMessage
```
