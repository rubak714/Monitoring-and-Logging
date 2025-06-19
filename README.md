# 🔗 Monitoring & Logging Project

---

This project is capable for:

1. Adding observability to app & infrastructure.

2. Prometheus + Grafana for metrics.

3. ELK Stack (Elasticsearch, Logstash, Kibana) or EFK (Fluentd) for logs.

4. Export metrics from Flask with prometheus_flask_exporter.

##### 🔗 Tools: Prometheus, Grafana, ELK, Fluent Bit

---

This guide can help anyone to build a  **separate DevOps project** for monitoring and logging a Python (Flask) app using:

* Prometheus + Grafana (metrics)
* EFK (Fluent Bit, Elasticsearch, Kibana) for logging

---

# 🔗 Monitoring & Logging Architecture – Explained with Diagram

This section explains how all the components work together to enable real-time monitoring and logging for user's Flask app.

---

## 🔗 What Happens Internally?

1. **Flask App**: The Python app exposes Prometheus metrics via `prometheus_flask_exporter` on port `5000/metrics`.
2. **Prometheus**: Periodically scrapes metrics from the Flask app endpoint.
3. **Grafana**: Connects to Prometheus and visualizes those metrics on dashboards.
4. **Fluent Bit**: Collects logs from the Flask container and sends them to Elasticsearch.
5. **Elasticsearch**: Stores structured log data for search and analysis.
6. **Kibana**: Reads logs from Elasticsearch and provides visualization/search UI.

---

## 🔗 Connections Summary

| Source        | Destination   | Purpose                      |
| ------------- | ------------- | ---------------------------- |
| Flask App     | Prometheus    | Exposes metrics (`/metrics`) |
| Prometheus    | Grafana       | Metric visualization         |
| Flask Logs    | Fluent Bit    | Container log collection     |
| Fluent Bit    | Elasticsearch | Store logs                   |
| Elasticsearch | Kibana        | Visualize/search logs        |

---

## 🔗 Component Roles

* **Flask App**: Generates logs and exposes performance metrics
* **Prometheus**: Pulls metrics from Flask and stores them
* **Grafana**: Queries Prometheus to show metrics in dashboards
* **Fluent Bit**: Acts as the log shipper
* **Elasticsearch**: Indexes and stores log entries
* **Kibana**: Helps anyone search and visualize logs from Elasticsearch

---

## 🔗 Block Diagram in Markdown

```
+----------------+       +----------------+       +-----------+
|                |       |                |       |           |
|   Flask App    +------>+   Prometheus   +------>+  Grafana  |
| (with metrics) |       | (scrapes /metrics)     | (dashboards)
|                |       |                |       |           |
+----------------+       +----------------+       +-----------+
        |
        | logs
        v
+----------------+
|   Fluent Bit   |
| (log forwarder)|
+--------+-------+
         |
         v
+-------------------+
|   Elasticsearch   |
|  (log storage)    |
+--------+----------+
         |
         v
+-------------------+
|      Kibana       |
|  (log viewer UI)  |
+-------------------+
```

---

<p align="center">
  <img src="monitoring_logging.png" alt="Monitoring and Logging Image" width="500"/>
</p>

<p align="center">
  <img src="monitoring_logging_2.png" alt="Monitoring and Logging Flowchart" width="500"/>
</p>

---

### Flow Breakdown

* **Flask App → Prometheus**: Prometheus scrapes `/metrics` from Flask
* **Prometheus → Grafana**: Grafana reads metrics from Prometheus to generate dashboards
* **Flask App → Fluent Bit**: Application logs are streamed
* **Fluent Bit → Elasticsearch**: Logs are stored and indexed
* **Elasticsearch → Kibana**: Kibana reads and visualizes logs


## 🔗 Prerequisites

* Ubuntu 20.04 or later
* Docker and Docker Compose installed
* Basic Python (Flask) app

---

## 🔗 Project Structure

```
monitoring-logging-project/
├── app/
│   ├── app.py
│   ├── requirements.txt
├── prometheus/
│   └── prometheus.yml
├── docker-compose.yml
└── README.md
```

---

## 🔗 app/app.py

```python
from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
metrics = PrometheusMetrics(app)

@app.route("/")
def home():
    return "Hello from Flask with Prometheus!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 🔗 app/requirements.txt

```
flask
prometheus_flask_exporter
```

---

## 🔗 prometheus/prometheus.yml

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'flask_app'
    static_configs:
      - targets: ['flask:5000']
```

---

## 🔗 docker-compose.yml

```yaml
version: '3.8'

services:
  flask:
    build: ./app
    container_name: flask
    ports:
      - "5000:5000"

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:7.10.2
    ports:
      - "5601:5601"

  fluent-bit:
    image: fluent/fluent-bit:latest
    volumes:
      - ./fluent-bit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
    depends_on:
      - elasticsearch
```

---

## 🔗 Run the Project

```bash
docker-compose up --build
```

Visit in browser:

* Flask App: [http://localhost:5000](http://localhost:5000)
* Prometheus: [http://localhost:9090](http://localhost:9090)
* Grafana: [http://localhost:3000](http://localhost:3000)
* Kibana: [http://localhost:5601](http://localhost:5601)

---

## 🔗 Export Metrics from Flask with prometheus\_flask\_exporter

To make the Flask application observable, we use the `prometheus_flask_exporter` Python library. This exporter provides a ready-made `/metrics` endpoint that Prometheus can scrape.

### 🔗 What It Does:

* Automatically tracks HTTP request metrics like latency, request count, and error rate.
* Exposes these metrics at `/metrics` on port 5000 (or any user defined port).
* Allows adding custom counters, histograms, and gauges.

---

### 🔗 File: `app/app.py`

```python
from flask import Flask
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)

# Default metrics for all endpoints
metrics = PrometheusMetrics(app)

# Optional: group endpoints by function name
metrics.info('app_info', 'Application info', version='1.0.0')

@app.route('/')
def home():
    return "Hello from Flask with Prometheus!"

@app.route('/api/data')
def data():
    return {"data": "Some useful data"}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

### 🔗 File: `prometheus/prometheus.yml`

Make sure Prometheus is configured to scrape the Flask app:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'flask_app'
    static_configs:
      - targets: ['flask:5000']
```

---

### 🔗 Access Metrics

Once the containers are up, user can access:

* The metrics endpoint at: [http://localhost:5000/metrics](http://localhost:5000/metrics)
* Prometheus UI at: [http://localhost:9090](http://localhost:9090)
* Use queries like `http_server_requests_total` in Prometheus
* Connect Grafana to Prometheus and build dashboards using these metrics

---

### Add Custom Metrics (Optional)

User can add custom counters or histograms like this:

```python
from prometheus_client import Counter

endpoint_hits = Counter('endpoint_hits', 'Number of hits to /api/data')

@app.route('/api/data')
def data():
    endpoint_hits.inc()
    return {"data": "Some useful data"}
```

This gives full control to monitor important custom logic in app.

---

So, here is full metric observability from Flask app using `prometheus_flask_exporter`!


## 🔗 Summary

Completed Activities:

* Metrics exposed from user's Flask app using Prometheus and Grafana
* Log collection and visualization using Fluent Bit, Elasticsearch and Kibana
