# Grafana Telemetry for Cisco Catalyst 9800WLC

## Abstract
This project provides a comprehensive Grafana dashboard for monitoring the Cisco C9800 Wireless LAN Controller (WLC) running IOS-XE. The monitoring solution leverages Cisco's Model-Driven Telemetry system using the **gRPC Dial-Out** method to collect real-time data. **Telemetry, InfluxDB, and Grafana (TIG)** stack are utilized to process and visualize this data. Telegraf acts as the collector, InfluxDB as the time-series database, and Grafana as the front-end application for creating detailed, dynamic dashboards. This setup allows network administrators to gain deep insights into the performance and status of their Cisco C9800 WLC, ensuring optimal network management and troubleshooting capabilities.

## Requirements
- Linux Server: Distributions like Ubuntu, CentOS, etc.
- Docker Engine: Follow the installation instructions available [here](https://get.docker.com/).
- Telemetry Configuration: Ensure it is properly set up on the ***Cisco 9800 WLC***. A sample configuration file is included in this [repository](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/blob/main/WLC-Config.txt).

## Server Configuration Guide

### Docker Instllation:
Gain root access to the Linux server, then clone the repository using the command below.

>`git clone https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git`

The repository will be copied to your /root directory under the name Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git. Navigate to this folder and run the following command to download and start the Docker services in detached mode:

>`cd /root/Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git`

>`docker compose up -d `

Please verify if the Docker containers are up. If you notice that the Telegraf container has restarted, do not worry; it may be due to the InfluxDB authentication not configured. Follow the steps below to verify

>`docker container ls` or `docker ps`

[<img width="1453" alt="Screenshot 2024-06-22 at 4 58 30 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/875c6797-7f92-4388-9627-b4fd7ea0c8a3">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

