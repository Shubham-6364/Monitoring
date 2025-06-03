## 📈 Grafana Installation (Ubuntu 22.04)

### 1. Install Dependencies

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```

### 2. Add GPG Key and Repo

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### 3. Install Grafana

```bash
sudo apt-get update
sudo apt-get -y install grafana
```

### 4. Start and Enable Grafana

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### 5. Check Status

```bash
sudo systemctl status grafana-server
```

### 6. Access Grafana

Visit:
📍 `http://<your-server-ip>:3000`
Default credentials: `admin / admin`

### 7. Add Prometheus as a Data Source

* Go to ⚙️ **Configuration** → **Data Sources** → **Add data source**
* Choose **Prometheus**
* Set URL: `http://localhost:9090`
* Click **Save & Test**

### 8. Import Dashboards

* Click ➕ **Create** → **Import**
* Use dashboard ID (e.g., `1860`)
* Select **Prometheus** as data source
* Click **Import**

---
---



## 📊 Recommended Grafana Dashboards

* **Node Exporter**: Use dashboard ID `1860` or search "Node Exporter Full".
* **Blackbox Exporter**: Use dashboard ID `7587` or create custom dashboards.
* **Prometheus**: Use default dashboards or community-imported ones.

---
