# Redis Installation and Configs:

## **Install on Ubuntu/Debian**

Add the repository to the APT index, update it, and install Redis:

```bash
sudo apt-get install lsb-release curl gpg

curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```

Redis will start automatically, and it should restart at boot time. If Redis doesn't start across reboots, you may need to manually enable it:

```bash
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

## Check Redis bind configuration

On your Redis server (192.168.XX.XX):

```bash
sudo nano /etc/redis/redis.conf
```

Find the line:

```bash
bind 127.0.0.1 ::1
```

Change it to:

```bash
bind 0.0.0.0
```

Also ensure:

```bash
protected-mode no
```

## Simple Docker direct method:

Run this command:

```bash
docker run -d \
  --name redis_exporter \
  -p 9121:9121 \
  --restart always \
  oliver006/redis_exporter \
  --redis.addr=redis://192.168.7.11:6379
```

## Docker Systemd Service (Redis Exporter)

```
# /etc/systemd/system/redis-exporter.service
[Unit]
Description=Redis Exporter (Single/Specific Redis)
After=docker.service
Requires=docker.service

[Service]
# Redis password (use secrets manager in production)
Environment="REDIS_PASSWORD=${REDIS_PASSWORD_PLACEHOLDER}"

Restart=always
ExecStartPre=-/usr/bin/docker rm -f redis-exporter
ExecStart=/usr/bin/docker run \
  --name redis-exporter \
  -p ${EXPORTER_PORT}:9121 \
  --restart always \
  oliver006/redis_exporter \
  --web.listen-address=:9121 \
  --web.telemetry-path=/metrics \
  --redis.addr=redis://${REDIS_HOST}:${REDIS_PORT} \
  --redis.password=${REDIS_PASSWORD}
ExecStop=/usr/bin/docker stop -t 10 redis-exporter

[Install]
WantedBy=multi-user.target

```

### **Placeholders used**

| Placeholder | Meaning |
| --- | --- |
| `${REDIS_PASSWORD_PLACEHOLDER}` | Redis password in environment variable |
| `${EXPORTER_PORT}` | Port Prometheus will scrape (exposed on host) |
| `${REDIS_HOST}` | Redis server IP/hostname |
| `${REDIS_PORT}` | Redis port (default: 6379) |

---

# Universal Prometheus Redis Scrape Job

```
  - job_name: 'redis_exporter_generic'
    static_configs:
      - targets:
        - redis://${REDIS_NODE_1}:${REDIS_PORT}
        - redis://${REDIS_NODE_2}:${REDIS_PORT}
        - redis://${REDIS_NODE_3}:${REDIS_PORT}
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: ${EXPORTER_PUBLIC_HOST}:${EXPORTER_PORT}

```

### **Universal placeholders used**

| Placeholder | Meaning |
| --- | --- |
| `${REDIS_NODE_1}` | First Redis node |
| `${REDIS_NODE_2}` | Second Redis node |
| `${REDIS_NODE_3}` | Third Redis node |
| `${REDIS_PORT}` | Redis port |
| `${EXPORTER_PUBLIC_HOST}` | The host/IP where Redis Exporter runs |
| `${EXPORTER_PORT}` | Port Redis Exporter exposes metrics |
