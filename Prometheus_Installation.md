## 🚀 Prometheus Installation

### 1. Create Prometheus User and Download Binary

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```

### 2. Extract and Move Files

```bash
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
```

### 3. Set Ownership

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

### 4. Create Prometheus systemd Service

```bash
sudo vim /etc/systemd/system/prometheus.service
```

Paste the following:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

### 5. Enable and Start Prometheus

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

### 6. Check Status

```bash
sudo systemctl status prometheus
```

Access Prometheus at:
📍 `http://<your-server-ip>:9090`

---

## 🖥️ Node Exporter Installation

### 1. Create Node Exporter User and Download Binary

```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

### 2. Extract and Move Binary

```bash
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

### 3. Create Node Exporter systemd Service

```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Paste the following:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```

### 4. Enable and Start Node Exporter

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### 5. Check Status

```bash
sudo systemctl status node_exporter
```

### 6. Add to Prometheus Config

```yaml
- job_name: "node_exporter"
  static_configs:
    - targets: ["<IP>:9100"]
```

