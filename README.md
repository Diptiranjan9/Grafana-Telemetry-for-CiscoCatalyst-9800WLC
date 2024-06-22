# Grafana Telemetry for CiscoCatalyst 9800WLC

## Abstract
This project provides a comprehensive Grafana dashboard for monitoring the Cisco C9800 Wireless LAN Controller (WLC) running IOS-XE. The monitoring solution leverages Cisco's Model-Driven Telemetry system using the gRPC Dial-Out method to collect real-time data. The Telemetry, InfluxDB, and Grafana (TIG) stack is utilized to process and visualize this data. Telegraf acts as the collector, InfluxDB as the time-series database, and Grafana as the front-end application for creating detailed, dynamic dashboards. This setup allows network administrators to gain deep insights into the performance and status of their Cisco C9800 WLC, ensuring optimal network management and troubleshooting capabilities.

## Requirements
- Linux Server: Distributions like Ubuntu, CentOS, etc.
- Docker Engine: Follow the installation instructions available [here](https://get.docker.com/).
- Telemetry Configuration: Ensure it is properly set up on the Cisco 9800 WLC. A sample configuration file is included in this [repository](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/blob/main/WLC-Config.txt).

## Server Configuration Guide

### 
