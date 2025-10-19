# Prometheus
Open-source metrics and monitoring for systems and services

# **Prometheus Remote Write Setup**

## **1. Prerequisites**

- **Source**: Prometheus instance that will send metrics.
- **Destination**: Prometheus server or other remote-write-capable service (running on 4th server).
- Network access between them (ensure ports are open).

---

## **2. Enable Remote Write Receiver on the Destination Prometheus**

On your remote Prometheus server, ensure the service includes:

```bash
--web.enable-remote-write-receiver
```

**Example systemd unit** (`/etc/systemd/system/prometheus.service`):

```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.enable-remote-write-receiver

[Install]
WantedBy=multi-user.target
```

Reload & restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart prometheus
```

Verify:

```bash
curl http://<4th-server-ip>:9090/api/v1/status/config
```

You should see `"remote_write"` section if configured on sender side.

---

## **3. Configure Remote Write on the Source Prometheus**

If you’re using **kube-prometheus-stack** in K8s:

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack -n monitoring \
  --set prometheus.prometheusSpec.remoteWrite[0].url="http://<4th-server-ip>:9090/api/v1/write"
```

Example:

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack -n monitoring \
  --set prometheus.prometheusSpec.remoteWrite[0].url="http://192.168.7.84:9090/api/v1/write"
```

---

## **4. Validate Remote Write is Working**

**On the source (K8s Prometheus)**:

```bash
kubectl -n monitoring exec -it prometheus-monitoring-kube-prometheus-prometheus-0 -- \
  cat /etc/prometheus/config_out/prometheus.env.yaml | grep -A5 remote_write
```

You should see the configured URL.

**On the destination (Remote Prometheus server)**:

Open the UI at:

```
http://<4th-server-ip>:9090/targets
```

You should see `remote_write` endpoint receiving data under **Service Discovery** or by checking incoming time series via the "graph" tab.

## 5. Add Cluster Data Source

Create YAML file named `values.yaml` 

```bash
prometheus:
  prometheusSpec:
    externalLabels:
      cluster: cluster-a #rename this based on the needs
    remoteWrite:
      - url: "http://192.168.7.84:9090/api/v1/write"
```

Then upgrade/install Prometheus using this file:

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring -f <file_location>/values.yaml
```

Then check the config: (Takes some time to load the config)

```bash
kubectl -n monitoring exec -it prometheus-monitoring-kube-prometheus-prometheus-0 -- \
  cat /etc/prometheus/config_out/prometheus.env.yaml | grep -A 5 external_labels
```

---

## **6. Troubleshooting If Required**

**Firewall:** Make sure port `9090` on the remote server is open for inbound connections from the source.

**Logs:**

On destination:

```bash
sudo journalctl -u prometheus -f
```

On source:

```bash
kubectl logs -n monitoring prometheus-monitoring-kube-prometheus-prometheus-0
```

Network test:

```bash
curl -v http://<4th-server-ip>:9090/api/v1/write
```

Config file check:

```bash
kubectl -n monitoring exec -it prometheus-monitoring-kube-prometheus-prometheus-0 -- cat /etc/prometheus/config_out/prometheus.env.yaml
```
