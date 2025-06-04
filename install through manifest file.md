This setup includes:

* Node Exporter as a DaemonSet
* Prometheus as a Deployment with a Service
* Grafana with Prometheus as a data source

---

## ✅ Step 1: Create a Namespace (Optional but recommended)
### `namespace.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

Apply:

```bash
kubectl apply -f namespace.yaml
```

---

## ✅ Step 2: Deploy Node Exporter (as a DaemonSet)
### `node-exporter.yaml`
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostPID: true
      containers:
        - name: node-exporter
          image: prom/node-exporter:v1.7.0
          ports:
            - containerPort: 9100
              name: metrics
          volumeMounts:
            - name: proc
              mountPath: /host/proc
              readOnly: true
            - name: sys
              mountPath: /host/sys
              readOnly: true
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: sys
          hostPath:
            path: /sys
---
apiVersion: v1
kind: Service
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    app: node-exporter
  ports:
    - port: 9100
      targetPort: 9100
```

---

## ✅ Step 3: Deploy Prometheus

### `prometheus-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']

      - job_name: 'node-exporter'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: node-exporter
```

### `prometheus-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:v2.52.0
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus
      volumes:
        - name: prometheus-config-volume
          configMap:
            name: prometheus-config
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
  type: NodePort
```

---

## ✅ Step 4: Deploy Grafana
### `grafana.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:10.0.0
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
    - port: 3000
      targetPort: 3000
  type: NodePort
```

---

## 📦 Apply All:

Save each YAML to a separate file and run:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f node-exporter.yaml
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f grafana.yaml
```

---
## ✅ Step 5: Access the Dashboards

* **Grafana UI**: `http://<node-ip>:<NodePort>`
* Default credentials:

  * **User**: `admin`
  * **Password**: `admin`

> In Grafana, add **Prometheus** as a data source:

* URL: `http://prometheus.monitoring.svc.cluster.local:9090`

Then import dashboards like Node Exporter ID: `1860`.

---

## **ConfigMap for Kubernetes that supports both:**

1. ✅ **Static targets** (for known IPs of worker nodes running Node Exporter)
2. ✅ **Kubernetes service discovery** (for Node Exporter DaemonSet with label `app: node-exporter`)

---

### 📄 `prometheus-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    scrape_configs:

      # Prometheus itself
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']

      # Node Exporter - Static Targets
      - job_name: 'node-exporter-static'
        static_configs:
          - targets: ['192.168.1.101:9100', '192.168.1.102:9100']  # Replace with your worker node IPs

      # Node Exporter - Kubernetes SD (via Service with label app=node-exporter)
      - job_name: 'node-exporter-kubernetes'
        kubernetes_sd_configs:
          - role: endpoints
        relabel_configs:
          - source_labels: [__meta_kubernetes_service_label_app]
            action: keep
            regex: node-exporter
          - source_labels: [__meta_kubernetes_endpoint_port_name]
            action: keep
            regex: metrics
```

---

### ✅ Steps to Use:

1. **Save this file** as `prometheus-config.yaml`.

2. **Apply it to your cluster**:

   ```bash
   kubectl apply -f prometheus-config.yaml
   ```

3. **Restart Prometheus pod** (if not using a config reloader):

   ```bash
   kubectl delete pod -n monitoring -l app=prometheus
   ```

---

### 🛠 Notes:

* **`192.168.1.101:9100`** — Replace with actual IPs of your worker nodes.
* **Kubernetes SD section** assumes:

  * A `Service` exists selecting pods with label `app=node-exporter`.
  * The port name in the Service is `metrics`.

