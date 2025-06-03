## 📦 1. What is **Prometheus**?

Prometheus is a **powerful open-source monitoring and alerting toolkit** originally developed by SoundCloud. It is designed for **reliability, scalability**, and works especially well with **cloud-native environments like Kubernetes**.

### 🔧 Key Features:

* Pull-based metrics scraping (not push)
* Time-series data storage
* PromQL (Prometheus Query Language)
* Service discovery support (Kubernetes, EC2, etc.)
* Alerting using **Alertmanager**

---

## 🖥️ 2. What is **Node Exporter**?

Node Exporter is a **lightweight agent** that runs on a Linux server to **expose system-level metrics** in a format Prometheus can scrape.

### 📊 Metrics Collected:

* CPU usage
* Memory usage
* Disk I/O
* Filesystem usage
* Network statistics
* System load averages

### 🧩 How It Works:

* Runs on each node (bare metal or VM)
* Exposes metrics on `http://<node-ip>:9100/metrics`
* Prometheus pulls data from this endpoint

---

## 📈 3. What is **Grafana**?

Grafana is an **open-source visualization tool** used to create dashboards and charts from time-series data sources like Prometheus, Loki, InfluxDB, etc.

### 🎨 Features:

* Customizable dashboards
* Supports multiple data sources
* Query editor for PromQL
* Built-in alerting and notification support

---

## 🔄 How Prometheus, Node Exporter & Grafana Work Together

```plaintext
+-------------------+       +----------------------+       +-------------------+
|  Linux Node / VM  | ----> |  Node Exporter (9100) | <---- |   Prometheus      |
|  (CPU, Mem, Disk) |       +----------------------+       | (Data Collector)  |
|                   |                                    -> | Pulls /metrics    |
+-------------------+                                        +-------------------+
                                                                   |
                                                                   v
                                                           +------------------+
                                                           |     Grafana      |
                                                           | Visualizes data  |
                                                           | Uses Prometheus  |
                                                           | as data source   |
                                                           +------------------+
```

### 💡 Flow:

1. **Node Exporter** runs on all target machines and exposes hardware & OS metrics at `/metrics`.
2. **Prometheus** periodically **scrapes** this data from all Node Exporters (defined in `prometheus.yml`).
3. **Grafana** connects to Prometheus as a **data source**.
4. Users build **dashboards** in Grafana using **PromQL** queries to visualize metrics like CPU usage, memory pressure, etc.

---

## 🧪 Real Use Case Example

### Use Case: Monitoring CPU usage on Kubernetes Nodes

* **Node Exporter** is deployed as a **DaemonSet** in the Kubernetes cluster.
* **Prometheus** is configured to auto-discover nodes and scrape metrics from Node Exporter.
* **Grafana** has a dashboard that shows per-node CPU, memory, disk, and network utilization.

---

## 🔧 Installation Overview (Kubernetes)

1. **Install Node Exporter:**

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/prometheus/node_exporter/main/examples/kubernetes-node.yaml
   ```

2. **Configure Prometheus (`prometheus.yml`):**

   ```yaml
   scrape_configs:
     - job_name: 'node-exporter'
       static_configs:
         - targets: ['<node-ip>:9100']
   ```

3. **Set up Grafana:**

   * Add Prometheus as a data source
   * Import dashboard ID `1860` (popular Node Exporter dashboard)

---

## ✅ Benefits of Using This Stack

| Component         | Role                       | Benefit                              |
| ----------------- | -------------------------- | ------------------------------------ |
| **Prometheus**    | Scrapes and stores metrics | Fast, scalable time-series DB        |
| **Node Exporter** | Exposes host-level metrics | Lightweight, no dependencies         |
| **Grafana**       | Visualizes metrics         | Interactive, customizable dashboards |

---

