## **Prometheus**

---

### **Create Prometheus User**

```jsx
	sudo useradd --no-create-home --shell /bin/false prometheus
```

---

### **Create Required Directories**

```jsx
sudo mkdir -p /etc/prometheus /var/lib/prometheus
```

Then copy config files from your extracted folder:

```jsx
sudo cp /opt/prometheus-3.7.2.linux-amd64/prometheus.yml /etc/prometheus/
```

```jsx
sudo cp -r /opt/prometheus-3.7.2.linux-amd64/consoles /opt/prometheus-3.7.2.linux-amd64/console_libraries /etc/prometheus/
```

---

### **Fix Ownership**

```jsx
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

```jsx
sudo chown -R prometheus:prometheus /opt/prometheus-3.7.2.linux-amd64
```

---

### **Create Prometheus Systemd Unit**

Open:

```jsx
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```jsx
[Unit]

Description=Prometheus

Wants=network-online.target

After=network-online.target

[Service]

User=prometheus

Group=prometheus

Type=simple

ExecStart=/opt/prometheus-3.7.2.linux-amd64/prometheus \

--config.file=/etc/prometheus/prometheus.yml \

--storage.tsdb.path=/var/lib/prometheus \

--web.console.templates=/etc/prometheus/consoles \

--web.console.libraries=/etc/prometheus/console_libraries \

--web.listen-address=0.0.0.0:9090 \
--web.enable-remote-write-receiver \
--storage.tsdb.retention.size=25GB

[Install]

WantedBy=multi-user.target
```

Save and exit.

---

### **Start and Enable Prometheus**

```jsx
sudo systemctl daemon-reload
```

```jsx
sudo systemctl enable prometheus
```

```jsx
sudo systemctl start prometheus
```

```jsx
sudo systemctl status prometheus
```

If all is good, you’ll see Active: active (running)

### Edit Scrap Config: (/etc/prometheus/prometheus.yaml)

```jsx
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
```
