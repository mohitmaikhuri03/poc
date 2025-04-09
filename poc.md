## Proof-of-Concept (POC) of logs Monitoring
![image](https://github.com/user-attachments/assets/b81b38c2-bacf-4e6c-8fa4-78a096457906)


| Created     |    Version   | Author | Comment | reviewer |
|:------------------:|:-------------:|:-------------:|:-------------:|:------------------:|
| 14-01-2025   | V1 | Pritam Kondapratiwar | L0| 
|    | V2 | Pritam Kondapratiwar | L1  |  |
|    | V3 |  Pritam Kondapratiwar | L2 |     |          |  


## Table of Contents

* [Introduction](#introduction)
* [Components](#components)
* [Prerequisites](#prerequisites)
* [Setting Up Grafana,Loki & Promatail on Ubunutu](#setting-up-grafanaloki--promatail-on-ubunutu)
* [Conclusion](#conclusion)
* [Contact Information](#contact-information)
* [References](#references)


## Introduction

This Proof of Concept (PoC) demonstrates how to set up a log monitoring solution using the Loki log aggregation system, Promtail for log collection, and Grafana for visualizing the logs. This stack is useful for monitoring logs across your infrastructure in a scalable and efficient manner.

## Components
| Component | Description | Purpose |
|-----------|-------------|---------|
| **Loki**  | It is designed to efficiently store, query, and manage logs. | Collect and store logs efficiently. |
| **Promtail** | An agent that ships logs to Loki. | Send logs from various sources to Loki. |
| **Grafana** | A powerful tool for data visualization. | Visualize logs and metrics collected by Loki. |

## Prerequisites

- A basic understanding of Docker and Grafana.
- [Docker installed](#https://docs.docker.com/engine/install/ubuntu/)


## Setting Up Grafana,Loki & Promatail on Ubunutu 

### Install with Docker on Linux

#### 1. Create a directory called loki. Make loki your current working directory:

```sh
mkdir loki
cd loki
```

#### 2. Run following commands into your command line to download loki-local-config.yaml and promtail-docker-config.yaml to your loki directory.

```sh
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```

#### 3. Run following commands into your command line to start the Docker containers using the configuration files you downloaded in the previous step.

```sh
docker run --name loki -d -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:3.2.1 -config.file=/mnt/config/loki-config.yaml
docker run --name promtail -d -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:3.2.1 -config.file=/mnt/config/promtail-config.yaml
```
#### 4. Verify that your containers are running:

```sh
docker ps -a
```
You should see something similar to the following:

![Screenshot from 2025-01-14 16-44-20](https://github.com/user-attachments/assets/77d26fdb-6ce8-49c9-ba58-8180ec7cdccf)

#### 5. Install the prerequisite packages for Grafana: 

```sh
sudo apt-get install -y apt-transport-https software-properties-common wget
```
#### 6. Import the GPG key:

```sh
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```
#### 7. To add a repository for stable releases, run the following command:

```sh
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

#### 8. To install Grafana on ubuntu , run the following command:
```
sudo apt-get update
sudo apt-get install grafana
```

#### 9. configure loki-config.yaml
```sh
 auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  instance_addr: 127.0.0.1
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
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


```

#### 10. configure promtail-config.yaml

```sh
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
```


#### 11. Access Grafana dashboard.

![image](https://github.com/user-attachments/assets/e2e3a3c4-b54c-4d39-b218-65fa75fdebb2)


#### 12. Add data source i.e, loki from the Grafana dashboard.
![image](https://github.com/user-attachments/assets/1f67b89c-1892-461e-a5e9-78f9d5ef4b5c)


#### 13. Create dashboard for log visualization. in this POC i am going to import dashboard from grafana official website .
![image](https://github.com/user-attachments/assets/bbc22e0a-ae68-4ab0-99f0-67d3a7011931)

- Select the dashboard that you want and copy the id of the dashboard
![image](https://github.com/user-attachments/assets/748f7bbd-19c2-49e9-8a51-6cc31fa566ec)

- Create a dashboard and  click on import dashboard then paste the id 
![image](https://github.com/user-attachments/assets/a5254a3c-519a-43e3-abb2-d9f604c7cdc9)

#### 14. You can see all the  logs on dashboard.
![image](https://github.com/user-attachments/assets/edbe6518-7b6e-43d1-8797-54ec225f071a)

## Conclusion
Now you have  set up a basic log monitoring system using the Grafana. This system enables you to collect, process, and visualize logs from your applications, providing insights that can help you maintain system health and quickly address any issues. 

## Contact Information

|    NAME           |   Email Address                       |
|:-----------------:|:-------------------------------------:|
|Pritam Kondapratiwar| 	pritam.pratiwar.snaatak@mygurukulam.co

## References

| Tools |  Links
|--------|------------|
| Loki  | https://grafana.com/oss/loki/ | 
| Promtail | https://grafana.com/docs/loki/latest/send-data/promtail/
| visualize logs on grafana dashboard |https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/logs/ | 
