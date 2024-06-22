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

[<img width="439" alt="Screenshot 2024-06-22 at 7 20 21 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/7015613f-8209-4ce1-975e-3274d295a7c9">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

- Login with User: admin, Pass:   # which you have mentioned in your docker compose file

- We need to configure InfluxDB as the data source. Click the “Add data source” button on the dashboard and then select “InfluxDB” as the database.
- Please follow below step:

[<img width="1512" alt="Screenshot 2024-06-22 at 6 30 34 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/0eae0a8c-c074-42c1-b4eb-9d08a12642e0">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

[<img width="1245" alt="Screenshot 2024-06-22 at 6 39 44 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/238964af-e4b2-435b-8d01-afba66f9e8d9">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)
[<img width="1243" alt="Screenshot 2024-06-22 at 6 39 56 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/d2bd7d16-1200-4a2d-95f5-d460d30fd994">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

- Note: Use authentication credentials for the InfluxDB login ID and password.

## Configuring the Catalyst 9800 to stream Network Telemetry to Telegraf

This portion is fairly straight forward, once you have the commands which I will included in this [repository](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/blob/main/WLC-Config.txt) to get the data you are looking for.

Cisco’s model-driven telemetry is provided by “subscriptions.” You subscribe to a particular metric category that you want to send to the receiver. Because I’m not an expert with yang models, I struggled to find some of the statistics that I was interested in seeing. I’ll show you the method I used to find the metrics.

### Finding the metrics

The statistics or metrics that you will want to visualize, are organized in a yang model. The yang model is basically a standard way of organizing information into a hierarchy, and you will need to reference the entire hierarchy/path when you subscribe to a group of metrics.

First, you will want to obtain the yang models for IOS-XE devices. The yang models can be found [here](https://github.com/YangModels/yang). It’s easiest to download the entire bundle, and then just navigate to the IOS-XE files. Note: These models are version specific, so you should make sure you are referencing the files for the version you are working with.

If you’ve never worked with Yang models before, you’re probably looking at this pile of files and wondering what the hell you’re supposed to do with them. **I should make it clear that you don’t actually need these files to make this work.** The files simply provide a way for us to figure out where the interesting metrics exist within the yang model, so that we can configure the 9800 to send them. We won’t be uploading these files to the server. We will just be referencing them on our workstation.

You can utilize the ***Cisco YANG*** Suite to explore all these metrics. Please click [here](https://www.youtube.com/watch?v=nnd4KqeeqIw) for details on the YANG Suite and instructions on how to explore all metrics using XPath.

[<img width="1511" alt="Screenshot 2024-06-22 at 8 11 49 PM" src="https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/assets/162305666/b522dc0e-28a9-47b6-ac10-05932b18bbd0">](https://github.com/Diptiranjan9/Grafana-Telemetry-for-CiscoCatalyst-9800WLC/issues/1#issue-2367761790)

### Setting up the telemetry subscriptions in the Catalyst 9800

Now that you have found the metrics you want to visualize, we need to configure the Catalyst 9800 to stream the data to Telegraf. This is done by creating “telemetry subscriptions” in the 9800. These subscriptions define the information you want, the format you want it in, where you want to send it, and how often you want to send it there. Here is an example of one telemetry subscription (we will have many, by the time we are done with this):

```
telemetry ietf subscription 21
 encoding encode-kvgpb
 filter xpath /wireless-client-oper:client-oper-data/traffic-stats
 source-address [wlc ip]
 stream yang-push
 update-policy periodic 1000
 receiver ip address [server ip] 57000 protocol grpc-tcp
```
You must also enable netconf on the controller for this to work:

```
netconf ssh
netconf-yang
```

Once you paste this configuration into the controller, you will want to see if the stream is active, and if the subscriptions are valid. You can do that with the following commands:

```
show telemetry internal connection

show telemetry ietf subscription [id]
```

Please find the xpaths which I have set up to find the metrics:

```
/wireless-client-oper:client-oper-data/traffic-stats
/wireless-client-oper:client-oper-data/common-oper-data
/wireless-client-oper:client-oper-data/dc-info
/wireless-client-oper:client-oper-data/sisf-db-mac
/wireless-client-oper:client-oper-data/dot11-oper-data
/if:interfaces-state/interface
/platform-sw-ios-xe-oper:cisco-platform-software/control-processes
/process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/one-minute
/cdp-ios-xe-oper:cdp-neighbor-details/cdp-neighbor-detail/device-name
/native/version
/interfaces-ios-xe-oper:interfaces/interface/ipv4
/cdp-ios-xe-oper:cdp-neighbor-details/cdp-neighbor-detail/local-intf-name
/platform-ios-xe-oper:components/component/state/serial-no
/cdp-ios-xe-oper:cdp-neighbor-details/cdp-neighbor-detail/port-id
/cdp-ios-xe-oper:cdp-neighbor-details/cdp-neighbor-detail/mgmt-address
/memory-ios-xe-oper:memory-statistics/memory-statistic
/platform-ios-xe-oper:components/component/state/description
/wireless-access-point-oper:access-point-oper-data/capwap-data
/wireless-access-point-oper:access-point-oper-data/ap-name-mac-map
/wireless-access-point-oper:access-point-oper-data/radio-oper-data
/wireless-rrm-oper:rrm-oper-data/rrm-measurement
/wireless-mobility-oper:mobility-oper-data/ap-peer-list
/wireless-mobility-oper:mobility-oper-data/wlan-client-limit
/wireless-access-point-oper:access-point-oper-data/ssid-counters
/ap-global-oper-data/ap-join-stats/ap-join-info
```

