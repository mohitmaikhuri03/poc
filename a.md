#  Logs Monitoring | PoC using Grafana, Promtail, and Loki

| Author       | Created On  | Version | Last Updated By | Internal Reviewer | Reviewer L0    | Reviewer L1     | Reviewer L2     |
|--------------|-------------|---------|------------------|-------------------|----------------|------------------|------------------|
| Mohit Kumar  | 09-Apr-2025 | v1.0    | Mohit Kumar      | Harshit Singh     | Akshit Kapil   | Shashi Sharma   | Mahesh Kumar    |

##  Table of Contents

- [Introduction](#introduction)  
- [Tools Used](#tools-used)
- [Architecture Diagram](#architecture-diagram)
- [How It Works](#how-it-works)
- [Step-by-Step Installation Guide](#step-by-step-installation-guide)  
- [Conclusion](#conclusion)  
- [Contact Information](#contact-information)  
- [References](-references)

---

##  Introduction

This document provides a proof of concept (PoC) for logs monitoring using open-source tools: Grafana, Loki, and Promtail.

The goal is to demonstrate how logs from various applications or systems can be collected, aggregated, and visualized in real time with minimal setup.


---

##  Tools Used

| Tool      | Role                      | Description                                                       |
|-----------|---------------------------|-------------------------------------------------------------------|
| Grafana   | Dashboard & Visualization | Used to build dashboards and visualize log data collected from Loki. |
| Loki      | Logs Aggregator           | A log aggregation system optimized for quick querying of logs.   |
| Promtail  | Log Collector & Shipper   | Runs on servers, collects log files, and ships them to Loki.     |

---
##  Architecture Diagram

![image](https://github.com/user-attachments/assets/270797f8-84ee-4f03-8b7e-22a87cf7121d)

## How It Works

- Promtail reads log files from your system (like /var/log/syslog, /var/log/auth.log, etc.). It keeps track of which parts of the log files it has already read to avoid duplicates.

- It then sends those logs to Loki, which stores and organizes them using labels (like job name or file path) so you can search logs easily later.

- Loki saves the log content and also keeps a small index of labels to make searching fast without using too much storage.

- Grafana is connected to Loki and lets you view and search the logs using simple queries. You can build dashboards, filter logs by time or content, and even create alerts.

---

## Step-by-Step Installation Guide

### 🔹 Step 1: Install Grafana

- To Setup Grafana on your system, please follow the link below for the Grafana Setup Guide. :-[ Grafana Setup  Guide](https://github.com/snaatak-Zero-Downtime-Crew/Documentation/blob/Nikita-SCRUM-104/Common/Software/Grafana/README.md)

📍 Access Grafana at: http://<server_IP>:3000  

   (Default login: `admin` / `admin`)

![image](https://github.com/user-attachments/assets/e352bcf3-2f82-4c63-9bd7-14db8151e3c4)


---

### 🔹 Step 2: Install Loki (Binary)

```bash

# Download latest Loki binary
curl -s https://api.github.com/repos/grafana/loki/releases/latest \
| grep browser_download_url \
| grep loki-linux-amd64.zip \
| cut -d '"' -f 4 \
| wget -i -

# Unzip the move binary file to /usr/local/bin
sudo apt install unzip
unzip loki-linux-amd64.zip
sudo mv loki-linux-amd64 /usr/local/bin/loki

# Confirm version
loki --version
```

---
- Create Loki configuration: 

```bash
# Navigate to the desired directory
cd /usr/local/bin

# Create necessary Loki data directory
sudo mkdir -p /data/loki

# Create the Loki configuration file
sudo nano /etc/loki-config.yaml
```

- Paste the following configuration into the /etc/loki-config.yaml file:
```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  instance_addr: 127.0.0.1
  path_prefix: /data/loki
  storage:
    filesystem:
      chunks_directory: /data/loki/chunks
      rules_directory: /data/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```
___
- Create a systemd service:

```bash
sudo tee /etc/systemd/system/loki.service <<EOF
[Unit]
Description=Loki Log Aggregation
After=network.target

[Service]
ExecStart=/usr/local/bin/loki -config.file /etc/loki-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

#  Reload enalbe and start the Promtail service
sudo systemctl daemon-reload
sudo systemctl enable loki
sudo systemctl restart loki
sudo systemctl status loki
```
![image](https://github.com/user-attachments/assets/2c439d25-abdc-4643-8d92-b5249c0097a8)

---

### 🔹 Step 3: Install Promtail

```bash
# Download latest Promtail binary
curl -s https://api.github.com/repos/grafana/loki/releases/latest \
| grep browser_download_url \
| grep promtail-linux-amd64.zip \
| cut -d '"' -f 4 \
| wget -i -

# Once the file is downloaded extract it to /usr/local/bin
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
chmod +x /usr/local/bin/promtail
```

 Create a YAML configuration file for Promtail in the /usr/local/bin directory:
 
 ```bash
cd /usr/local/bin
sudo nano /etc/promtail-config.yaml
```

 Add the following content to the file :- 
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

#  Reload enalbe and start the Promtail service
sudo systemctl daemon-reload
sudo systemctl start promtail
sudo systemctl enable promtail
sudo systemctl status promtail
```
![image](https://github.com/user-attachments/assets/f60f825e-6058-4bfc-b3e4-8ea55a86c073)

---

### 🔹 Step 4: Connect Loki to Grafana

- Open Grafana in browser: <http://public-ip:3000>

- Login with `admin / admin`
 
  ![image](https://github.com/user-attachments/assets/a8b2a2ac-2013-4214-8054-f7b83aa769a3)

  
- Go to **Settings → Data Sources → Add data source**
- Choose **Loki**
  ![image](https://github.com/user-attachments/assets/3e6bc1fd-0991-4642-9736-b6321a10f657)

- URL: `http://localhost:3100`

![image](https://github.com/user-attachments/assets/d6660cb5-c9a7-460b-8f28-a3285c3d1e55)

- Click **Save & Test**

![image](https://github.com/user-attachments/assets/472264c7-dd62-46b1-8e88-e13cbfddee96)

---

### 🔹 Step 5: Visualize Logs

- Go to the **Explore** tab in Grafana.
- Use the query: `{job="varlogs"}` to see logs.

![image](https://github.com/user-attachments/assets/cca9e8b5-f3e2-44e0-a482-fcb77901d217)

![image](https://github.com/user-attachments/assets/8ec37e0d-ab45-419c-a639-bbbcabde54be)


---

##  Conclusion

This PoC serves as a great starting point for exploring Grafana, Loki, and Promtail. It helps you understand the fundamentals of log monitoring, which can be extended and hardened for production environments as needed.



---

## Contact Information

| Name        | Email Address                  |
|-------------|-------------------------------|
| Mohit Kumar | mohit.kumar@mygurukulam.co    |

---

##  References

| Resource              | Description                            |
|-----------------------|----------------------------------------|
| Grafana Loki Docs     | Official documentation for Loki        |
| Promtail Setup        | Guide to setting up Promtail           |
| Grafana Dashboards    | Pre-built dashboards                   |
| Logs Monitoring Blog  | Overview of logs monitoring tools      |
