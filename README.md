# workflows
Reusable workflows and actions


## How to create a self-hosted runner on OrbStack

```
orb create -a arm64 ubuntu:noble self-hosted-runner
orb shell -m self-hosted-runner
```

TODO https://github.com/actions/partner-runner-images/blob/main/images/arm-ubuntu-24-image.md

```
sudo apt-get install -y git curl wget net-tools jq build-essential zlib1g-dev

sudo groupadd runner
sudo useradd -m -g runner -s /bin/bash runner-categolj

curl -sSL https://get.docker.com/ | sudo sh
sudo usermod -aG docker runner-categolj
sudo usermod -aG docker $(id -un)
```

```
sudo su - runner-categolj

mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-arm64-2.321.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-arm64-2.321.0.tar.gz
tar xzf ./actions-runner-linux-arm64-2.321.0.tar.gz

./config.sh --url https://github.com/categolj --name orbstack --labels orbstack --runnergroup Default --work _work --token AAA********
exit


sudo su -

cd /home/runner-categolj/actions-runner 
./svc.sh install runner-categolj
./svc.sh start
./svc.sh status
```


```
sudo apt-get install prometheus-node-exporter -y
```

```
wget https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.115.1/otelcol-contrib_0.115.1_linux_arm64.deb
sudo dpkg -i otelcol-contrib_0.115.1_linux_arm64.deb
```

```
sudo systemctl stop otelcol-contrib
```

```yaml
cat <<'EOF' | sudo tee /etc/otelcol-contrib/config.yaml
---
extensions:
  health_check:
  pprof:
    endpoint: 0.0.0.0:1777
  zpages:
    endpoint: 0.0.0.0:55679

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

  prometheus:
    config:
      scrape_configs:
      - job_name: node-exporter
        scrape_interval: 60s
        static_configs:
        - targets: ['0.0.0.0:9100']

processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 75
    spike_limit_percentage: 15
  batch:
    send_batch_size: 10000
    timeout: 10s
  attributes:
    actions:
    - key: cluster
      value: orbstack
      action: upsert
  resource:
    attributes:
    - action: upsert
      key: k8s.cluster.name
      value: orbstack

exporters:
  debug:
    verbosity: detailed
  prometheusremotewrite:
    endpoint: ${PROMETHEUS_ENDPOINT}
    headers:
      authorization: "Basic ${PROMETHEUS_BASIC_AUTH}"
    tls:
      insecure_skip_verify: true
    compression: gzip
service:

  pipelines:

    traces:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [debug]

    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, attributes, batch]
      exporters: [prometheusremotewrite]

    logs:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [debug]

  extensions: [health_check, pprof, zpages]
---
EOF
```

```bash
PROMETHEUS_ENDPOINT=https://.../api/v1/write
PROMETHEUS_BASIC_AUTH=...

cat <<EOF | sudo tee -a /etc/otelcol-contrib/otelcol-contrib.conf
PROMETHEUS_ENDPOINT=${PROMETHEUS_ENDPOINT}
PROMETHEUS_BASIC_AUTH=${PROMETHEUS_BASIC_AUTH}
EOF
```