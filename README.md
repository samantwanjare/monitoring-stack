# monitoring-stack# 📊 monitoring-stack

> Full observability stack — Prometheus + Grafana + ELK (Elasticsearch, Logstash, Kibana) deployed via Docker Compose and Kubernetes.

---

## 📁 Folder Structure

```
monitoring-stack/
├── docker-compose/
│   ├── docker-compose.yml          # Full stack with one command
│   ├── prometheus/
│   │   └── prometheus.yml
│   ├── grafana/
│   │   └── provisioning/
│   │       ├── datasources/prometheus.yml
│   │       └── dashboards/node-exporter.json
│   └── alertmanager/
│       └── alertmanager.yml
├── kubernetes/
│   ├── namespace.yaml
│   ├── prometheus/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── grafana/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── elk/
│       ├── elasticsearch.yaml
│       ├── logstash.yaml
│       └── kibana.yaml
└── README.md
```

---

## 🚀 Quick Start (Docker Compose)

```bash
# Clone and start
git clone https://github.com/samantwanjare/monitoring-stack
cd monitoring-stack/docker-compose

# Start full stack
docker-compose up -d

# Access dashboards
# Grafana   → http://localhost:3000  (admin/admin)
# Prometheus → http://localhost:9090
# Kibana    → http://localhost:5601
```

---

## 📄 docker-compose.yml

```yaml
version: '3.8'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}
  grafana_data: {}
  elasticsearch_data: {}

services:
  prometheus:
    image: prom/prometheus:v2.45.0
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:10.0.0
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    ports:
      - "3000:3000"
    networks:
      - monitoring
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: unless-stopped

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.0
    container_name: logstash
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
    networks:
      - monitoring

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    networks:
      - monitoring
```

---

## 📄 Prometheus Config (prometheus.yml)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

---

## 🏷️ Tech Stack

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-005571?style=flat-square&logo=elasticsearch)
![Kibana](https://img.shields.io/badge/Kibana-005571?style=flat-square&logo=kibana)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker)

---

> 💡 **Author:** Samant Wanjare | [GitHub](https://github.com/samantwanjare) | [LinkedIn](https://linkedin.com/in/samantwanjare)
