```bash
cat > metricbeat.yaml << "EndOfMessage"
daemonset:
  metricbeatConfig:
    metricbeat.yml: |
      metricbeat.modules:
      - module: kubernetes
        metricsets:
          - container
          - node
          - pod
          - system
          - volume
        period: 10s
        host: "${NODE_NAME}"
        hosts: ["https://${NODE_NAME}:10250"]
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        ssl.verification_mode: "none"
        processors:
        - add_kubernetes_metadata: ~
      - module: kubernetes
        enabled: true
        metricsets:
          - event
      - module: system
        period: 10s
        metricsets:
          - cpu
          - load
          - memory
          - network
          - process
          - process_summary
        processes: ['.*']
        process.include_top_n:
          by_cpu: 5
          by_memory: 5
      - module: system
        period: 1m
        metricsets:
          - filesystem
          - fsstat
        processors:
        - drop_event.when.regexp:
            system.filesystem.mount_point: '^/(sys|cgroup|proc|dev|etc|host|lib)($|/)'
      output.kafka:
        hosts: ["redpanda-0.redpanda:9093", "redpanda-1.redpanda:9093", "redpanda-2.redpanda:9093"]
        topic: metrics
        required_acks: 1
        compression: gzip
EndOfMessage

helm upgrade --install metricbeat elastic/metricbeat \
  --version 7.17.3 \
  --namespace logging-system \
  --set daemonset.enabled=true \
  --set deployment.enabled=false \
  -f metricbeat.yaml
```
