# Grafana Telemetry for Cisco Catalyst 9800WLC

## Abstract
This project provides a comprehensive Grafana dashboard for monitoring the Cisco C9800 Wireless LAN Controller (WLC) running IOS-XE. The monitoring solution leverages Cisco's Model-Driven Telemetry system using the **gRPC Dial-Out** method to collect real-time data. **Telegraf, InfluxDB, and Grafana (TIG)** stack are utilized to process and visualize this data. Telegraf acts as the collector, InfluxDB as the time-series database, and Grafana as the front-end application for creating detailed, dynamic dashboards. This setup allows network administrators to gain deep insights into the performance and status of their Cisco C9800 WLC, ensuring optimal network management and troubleshooting capabilities.

## Requirements
- Linux Server: Distributions like Ubuntu, CentOS, etc.
- Docker Engine: Follow the installation instructions available [here](https://get.docker.com/).
- Telemetry Configuration: Ensure it is properly set up on the ***Cisco 9800 WLC***. A sample configuration file is included in this [repository](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/blob/main/WLC-Config.txt).

## Server Configuration Guide

### Docker Instllation:
Gain root access to the Linux server, then clone the repository using the command below.

```
git clone https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git
```

The repository will be copied to your /root directory under the name Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git. Navigate to this folder and run the following command to download and start the Docker services in detached mode:

```
cd /root/Grafana-Telemetry-for-CiscoCatalyst-9800WLC.git
```

```
docker compose up -d
```

Please verify if the Docker containers are up. If you notice that the Telegraf container has restarted, do not worry; it may be due to the InfluxDB authentication not configured. Follow the steps below to verify:

```
docker container ls or docker ps
```

[<img width="1453" alt="Screenshot 2024-06-22 at 4 58 30 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/875c6797-7f92-4388-9627-b4fd7ea0c8a3">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

### Set up InfluxDB: 

Find your server's IP address, then open your browser and access the InfluxDB web page forwarded on port 8086. Ensure that the firewall on your server allows traffic on ports 8086, 3000, and 57000 for the necessary services to run.

```
ip addr show  # This command is to verify your server ip address
```

```
1. With InfluxDB running, visit `http://[server ip]:8086`
2. Click Get Started

Set up your initial user

3. Enter a Username for your initial user.
4. Enter a Password and Confirm Password for your user.
5. Enter your initial Organization Name.
6. Enter your initial Bucket Name.
7. Click Continue.

Copy the provided operator API token and store it for safe keeping.
```

[<img width="1400" alt="Screenshot 2024-06-22 at 5 29 49 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/14a6b9a9-eb05-4b88-aab5-97fb5a2b5f56">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

[<img width="1390" alt="Screenshot 2024-06-22 at 5 21 00 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/7907e95f-59b5-4e85-a2f1-673ce524df1d">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

### Setting up InfluxQL Connection to InfluxDB 2.x

This is a bit more complicated process which is explained in this [forum topic](https://community.influxdata.com/t/how-to-connect-grafana-to-influxdbv2-by-influxql-method/17377/2), but essentially it requires below steps:

- Gain CLI access to the InfluxDB container.
```
docker exec -it influxdb bash
```
- First, authenticate to gain access, then execute the DBRP command to map from the bucket to the database.
```
influx config create --config-name <config-name> --host-url http://localhost:8086 --org <your-org-name> --token <your-auth-token>

influx auth list

influx v1 dbrp create --db telegraf-influxsql --rp autogen --bucket-id <bucketid> --org-id <org-id>

```

- Note: You will find the authentication token, bucket ID, and organization ID from the HTTP access of the database.

[<img width="1512" alt="Screenshot 2024-06-22 at 6 16 13 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/c606ddfa-59ba-46cd-9628-af5b42f20820">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)
[<img width="1512" alt="Screenshot 2024-06-22 at 6 17 56 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/8ce36185-fab5-4127-ab3e-4745be97888b">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)
[<img width="1512" alt="Screenshot 2024-06-22 at 6 18 48 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/d98b1b21-f052-42b1-9928-38fda77e4df5">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

### Set up Telegraf:

***Configure Telegraf to receive Cisco Network Telemetry***

```
vim /var/lib/docker/volumes/grafana_telegraf_data/_data/telegraf.conf
```
We must add the necessary information to enable the “cisco_telemetry_mdt” Telegraf plugin. We need to add a few lines of code to the Service Input Plugins section of the file. First, lets navigate to that section of the file:

`/SERVICE INPUT`

- We will add the following Telegraf plugin lines just inside this section:
```
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"
```
[<img width="673" alt="Screenshot 2024-06-22 at 6 50 56 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/6ec172c5-8d7c-4622-8405-2a043e347ba5">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

- We also need to configure Telegraf to connect to the InfluxDB that we created earlier:

  `/OUTPUT PLUGINS`
  
```
[[outputs.influxdb_v2]]
 urls = ["http://influxdb:8086"]
 token = "xxxxxxxxxxxxxxxxxx"   #put admin token here
 organization = "Org Name"
 bucket = "telegraf"
```
[<img width="692" alt="Screenshot 2024-06-22 at 6 57 01 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/f366a762-85ab-45ea-aa34-3bf4ad4916a9">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

- After completing these configurations, restart the Telegraf container using the following steps:

```
docker restart telegraf
```

### Setting up InfluxQL connection in Grafana

You should verify that you can access the Grafana dashboard at http://[server ip]:3000



