## 🌐 Blackbox Exporter Setup

### 1. Download and Setup

```bash
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.19.0/blackbox_exporter-0.19.0.linux-amd64.tar.gz
tar -xvf blackbox_exporter-0.19.0.linux-amd64.tar.gz
sudo mv blackbox_exporter-0.19.0.linux-amd64/blackbox_exporter /usr/local/bin/
```

### 2. Create User and Directories

```bash
sudo useradd --no-create-home --shell /bin/false blackbox_exporter
sudo mkdir /etc/blackbox_exporter
sudo chown blackbox_exporter:blackbox_exporter /usr/local/bin/blackbox_exporter /etc/blackbox_exporter
```

### 3. Configure blackbox.yml

```bash
sudo vim /etc/blackbox_exporter/blackbox.yml
```

Add the following:

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []
      method: GET
```

```bash
sudo chown blackbox_exporter:blackbox_exporter /etc/blackbox_exporter/blackbox.yml
```

### 4. Create systemd Service

```bash
sudo vim /etc/systemd/system/blackbox_exporter.service
```

Paste:

```ini
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file /etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target
```

### 5. Enable and Start Blackbox Exporter

```bash
sudo systemctl daemon-reload
sudo systemctl enable blackbox_exporter
sudo systemctl start blackbox_exporter
sudo systemctl status blackbox_exporter
```

### 6. Update Prometheus Config

```yaml
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - http://prometheus.io
        - https://prometheus.io
        - http://example.com:8080
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9115
```

```bash
sudo systemctl restart prometheus
```

---
