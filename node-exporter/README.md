# Install node_exporter

Ref : https://medium.com/@abdullah.eid.2604/node-exporter-installation-on-linux-ubuntu-8203d033f69c 

Download Link & Check Version : https://github.com/prometheus/node_exporter/releases 

### Step 1: We need a user. Create a system user for the installation.
```bash
sudo useradd -rs /bin/false node_exporter
```
### Step 2: Log into node_exporter user and download the latest node_exporter package.

Version ล่าสุด ณ วันที่ 11/09/2024

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```
### Step 3: Extract & move the node_exporter package and rename the directory to ‘node_exporter’ for your convenience.
```bash
sudo tar -xzvf node_exporter-1.8.2.linux-amd64.tar.gz

sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin
```
### Step 4: Enable 9100 for your Prometheus server. 

Allow port 9100 if you use firewall on OS

### Step 5: Create a service file 
```bash
vi /etc/systemd/system/node_exporter.service 
```
add code

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/local/bin/node_exporter --web.config.file="/etc/node_exporter/config.yml"
Restart=always

[Install]
WantedBy=multi-user.target
```

