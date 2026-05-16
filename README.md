# Real-Time Infrastructure Monitoring and Alerting using Prometheus & Grafana

## Project Overview

This project demonstrates a complete real-time infrastructure monitoring and alerting setup using **Prometheus**, **Grafana**, and **Node Exporter** on AWS EC2 instances.

The monitoring server collects metrics from multiple application servers and visualizes them through Grafana dashboards. Alert rules were also configured to detect high CPU usage and trigger alerts automatically.

This project helped me understand:
- Infrastructure monitoring
- Metrics collection
- Prometheus scraping
- Grafana dashboards
- Alerting workflows
- Linux service management
- Basic observability concepts

---

# Architecture

The architecture contains:

- 1 Monitoring Server
  - Prometheus
  - Grafana

- 2 Application Servers
  - Node Exporter

Prometheus scrapes metrics from both application servers and Grafana visualizes those metrics.

Alerts are triggered when CPU usage crosses the defined threshold.

## Architecture Diagram

![Architecture](architecture.png)

---

# AWS Infrastructure

## EC2 Instances Used

| Server | Purpose |
|---|---|
| monitoring-server | Prometheus + Grafana |
| app-server-1 | Node Exporter |
| app-server-2 | Node Exporter |

## Running Instances

![EC2 Instances](ec2-running.png)

---

# Technologies Used

- AWS EC2
- Linux
- Prometheus
- Grafana
- Node Exporter
- PromQL
- systemd

---

# Security Group Configuration

The monitoring security group allowed these ports:

| Port | Service |
|---|---|
| 22 | SSH |
| 3000 | Grafana |
| 9090 | Prometheus |
| 9100 | Node Exporter |

---

# Step 1 — Install Prometheus on Monitoring Server

## Download Prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.49.0/prometheus-2.49.0.linux-amd64.tar.gz
```

## Extract Files

```bash
tar -xvf prometheus-2.49.0.linux-amd64.tar.gz
```

## Start Prometheus

```bash
cd prometheus-2.49.0.linux-amd64
./prometheus --config.file=prometheus.yml
```

Prometheus runs on:

```text
http://<monitoring-server-ip>:9090
```

---

# Step 2 — Install Grafana on Monitoring Server

## Install Grafana

```bash
sudo yum install -y https://dl.grafana.com/enterprise/release/grafana-enterprise-10.4.1-1.x86_64.rpm
```

## Start Grafana

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Grafana runs on:

```text
http://<monitoring-server-ip>:3000
```

Default login:

```text
Username: admin
Password: admin
```

---

# Step 3 — Install Node Exporter on Application Servers

Performed on:
- app-server-1
- app-server-2

## Download Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
```

## Extract Files

```bash
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
```

## Create systemd Service

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

## Service File

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=ec2-user
ExecStart=/home/ec2-user/node_exporter-1.7.0.linux-amd64/node_exporter

[Install]
WantedBy=multi-user.target
```

## Start Node Exporter

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

Verify:

```bash
systemctl status node_exporter
```

Node Exporter runs on:

```text
http://<app-server-ip>:9100/metrics
```

---

# Step 4 — Configure Prometheus Targets

Edit:

```bash
vi prometheus.yml
```

Added scrape targets:

```yaml
scrape_configs:
  - job_name: "app-servers"
    static_configs:
      - targets:
          - "172.31.15.168:9100"
          - "172.31.13.217:9100"
```

Restart Prometheus after changes.

---

# Step 5 — Verify Targets in Prometheus

Opened:

```text
http://<monitoring-server-ip>:9090/targets
```

Both application servers were successfully scraped and showed UP status.

## Prometheus Targets

![Prometheus Targets](prometheus-targets.png)

---

# Step 6 — Configure Grafana Datasource

Inside Grafana:

1. Open Connections
2. Add Datasource
3. Select Prometheus
4. Add Prometheus URL:

```text
http://localhost:9090
```

5. Save & Test

---

# Step 7 — Import Node Exporter Dashboard

Imported Node Exporter dashboard inside Grafana to monitor:

- CPU Usage
- Memory Usage
- Disk Usage
- Network Usage
- System Load

## Grafana Dashboard

![Grafana Dashboard](grafana-dashboard.png)

---

# Step 8 — Create CPU Alert Rule

Created a Grafana-managed alert rule for high CPU usage detection.

## PromQL Query Used

```promql
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100)
```

## Alert Condition

```text
Trigger alert when CPU usage > 30%
```

## Evaluation Settings

| Setting | Value |
|---|---|
| Pending Period | 1 minute |
| Evaluation Interval | Every 1 minute |

---

# Alert Rule Screenshot

![Alert Rule](alert-rule.png)

---

# Step 9 — Test Alert Using Stress Command

Used stress tool on application server to increase CPU usage manually.

## Install Stress Tool

```bash
sudo yum install stress -y
```

## Generate CPU Load

```bash
stress --cpu 4
```

This increased CPU usage and triggered the Grafana alert.

---

# Step 10 — Verify FIRING Alert

After CPU usage crossed the threshold, Grafana changed alert state to:

```text
FIRING
```

## Firing Alert Screenshot

![Firing Alert](firing-alert.png)

---

# Monitoring Features Implemented

- Real-time infrastructure monitoring
- Centralized metrics collection
- CPU monitoring
- Memory monitoring
- Disk monitoring
- Network monitoring
- Grafana dashboards
- CPU alerting
- Multi-server monitoring

---

# Project Outcome

Successfully built a real-time monitoring and alerting setup using Prometheus and Grafana on AWS EC2 instances.

The setup continuously monitors application servers and generates alerts whenever CPU usage exceeds the configured threshold.

This project improved my understanding of:
- Monitoring systems
- Infrastructure observability
- Metrics-based alerting
- Linux services
- Prometheus queries
- Grafana alert workflows

---