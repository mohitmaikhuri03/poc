# 🚀 Logs Monitoring | PoC using Grafana, Promtail, and Loki

| Author       | Created On  | Version | Last Updated By | Internal Reviewer | Reviewer L0    | Reviewer L1     | Reviewer L2     |
|--------------|-------------|---------|------------------|-------------------|----------------|------------------|------------------|
| Mohit Kumar  | 09-Apr-2025 | v1.0    | Mohit Kumar      | Harshit Singh     | Akshit Kapil   | Shashi Sharma   | Mahesh Kumar    |

## 📑 Table of Contents

- [Introduction](#-introduction)  
- [Tools Used](#-tools-used)  
- [How It Works](#️-how-it-works)  
- [Step-by-Step Installation Guide](#-step-by-step-installation-guide)  
- [Conclusion](#-conclusion)  
- [Contact Information](#-contact-information)  
- [References](#-references)

---

## 🧭 Introduction

This document provides a proof of concept (PoC) for logs monitoring using open-source tools: Grafana, Loki, and Promtail.

The goal is to demonstrate how logs from various applications or systems can be collected, aggregated, and visualized in real time with minimal setup.

This is intended purely as a PoC setup to show that it works — not optimized for production, but enough to help you get started quickly.

---

## 🛠️ Tools Used

| Tool      | Role                      | Description                                                       |
|-----------|---------------------------|-------------------------------------------------------------------|
| Grafana   | Dashboard & Visualization | Used to build dashboards and visualize log data collected from Loki. |
| Loki      | Logs Aggregator           | A log aggregation system optimized for quick querying of logs.   |
| Promtail  | Log Collector & Shipper   | Runs on servers, collects log files, and ships them to Loki.     |

---

## ⚙️ How It Works

- Promtail collects logs from system or application log files.
- It sends those logs to Loki, which stores and indexes them.
- Grafana is connected to Loki and visualizes logs through dashboards or search queries.

---

## 🏗️ Step-by-Step Installation Guide

> All installations are done on **Ubuntu Linux (20.04+)**. Adjust commands as necessary for other distributions.

### 🔹 Step 1: Install Grafana

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y gnupg2 curl software-properties-common

# Add Grafana GPG key and repo
curl -fsSL https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/grafana.gpg
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

# Install Grafana
sudo apt update && sudo apt install -y grafana

# Start and enable Grafana service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Allow Grafana port
sudo ufw allow 3000/tcp
```

📍 Access Grafana at: http://localhost:3000  
(Default login: `admin` / `admin`)

---

### 🔹 Step 2: Install Loki (Binary)

```bash
sudo apt install -y unzip

# Download latest Loki binary
curl -s https://api.github.com/repos/grafana/loki/releases/latest \
| grep browser_download_url \
| grep loki-linux-amd64.zip \
| cut -d '"' -f 4 \
| wget -i -

# Unzip and move binary
unzip loki-linux-amd64.zip
sudo mv loki-linux-amd64 /usr/local/bin/loki
chmod +x /usr/local/bin/loki

# Confirm version
loki --version
```

Create Loki configuration:

```bash
sudo mkdir -p /data/loki
sudo wget -O /etc/loki-local-config.yaml https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml
sudo sed -i 's/auth_enabled: true/auth_enabled: false/' /etc/loki-local-config.yaml
```

Create a systemd service:

```bash
sudo tee /etc/systemd/system/loki.service <<EOF
[Unit]
Description=Loki Log Aggregation
After=network.target

[Service]
ExecStart=/usr/local/bin/loki -config.file /etc/loki-local-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start loki
sudo systemctl enable loki
```

---

### 🔹 Step 3: Install Promtail

```bash
# Download latest Promtail binary
curl -s https://api.github.com/repos/grafana/loki/releases/latest \
| grep browser_download_url \
| grep promtail-linux-amd64.zip \
| cut -d '"' -f 4 \
| wget -i -

unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
chmod +x /usr/local/bin/promtail
```

Create config file `/etc/promtail-config.yaml`:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

Create a systemd service:

```bash
sudo tee /etc/systemd/system/promtail.service <<EOF
[Unit]
Description=Promtail Log Collector
After=network.target

[Service]
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start promtail
sudo systemctl enable promtail
```

---

### 🔹 Step 4: Connect Loki to Grafana

- Open Grafana in browser: http://localhost:3000  
- Login with `admin / admin`
- Go to **Settings → Data Sources → Add data source**
- Choose **Loki**
- URL: `http://localhost:3100`
- Click **Save & Test**

---

### 🔹 Step 5: Visualize Logs

- Go to the **Explore** tab in Grafana.
- Use the query: `{job="varlogs"}` to see logs.
- Create dashboards or set up alerts as needed.

---

## ✅ Conclusion

This PoC demonstrates how you can easily implement a log monitoring solution using Grafana, Loki, and Promtail.

It's a simple yet powerful stack that works well for small-to-medium setups or development environments.

> ⚠️ **Note:** This setup is not optimized for large-scale or high-availability deployments. For production use, consider:
> - Log rotation and compression
> - Centralized configuration management
> - Scalable storage backends
> - Horizontal scaling with Promtail and Loki

---

## 📬 Contact Information

| Name        | Email Address                  |
|-------------|-------------------------------|
| Mohit Kumar | mohit.kumar@mygurukulam.co    |

---

## 🔗 References

| Resource              | Description                            |
|-----------------------|----------------------------------------|
| Grafana Loki Docs     | Official documentation for Loki        |
| Promtail Setup        | Guide to setting up Promtail           |
| Grafana Dashboards    | Pre-built dashboards                   |
| Logs Monitoring Blog  | Overview of logs monitoring tools      |
