# Create Basic Auth Password 
### install htpasswd
```bash
sudo apt-get update && sudo apt install apache2-utils -y
```

### Gen Password 
```bash
htpasswd -nBC 12 "" | tr -d ':\n'
```
add your password and save encoded for create node_exporter config.yaml

# Create TLS Cert For Secure Exporter

*****  ในกรณีที่ไม่ใช่การ Implement ใหม่ หรือมี Cert สำหรับ Node exporter อยู่แล้ว ให้เอามาใช้เลย และข้ามขั้นตอนนี้ไป *****

* Create Directory for keep Cert and node exporter config.yaml
```bash
mkdir /etc/node_exporter
```
Move into node_exporter dir
```bash
cd node_exporter
```
to next step create ca and tls cert
## Create CA
### Create CA Key
```bash
openssl genrsa -out ca.key 2048
```
### Create CA Certificate-Signing-Request (CSR)
```bash
openssl req -new -key ca.key -out ca.csr
```
### Create CA Certificate from CA CSR, CA KEY  10 Years
```bash
openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt
```

# Install node_exporter On VM Linux

Ref : https://medium.com/@abdullah.eid.2604/node-exporter-installation-on-linux-ubuntu-8203d033f69c 

Download Link & Check Version : https://github.com/prometheus/node_exporter/releases 

### Step 1: We need a user. Create a system user for the installation.
```bash
sudo useradd -rs /bin/false node_exporter
```
### Step 2: Log into node_exporter user and download the latest node_exporter package.

คำสั่งด้านล่างนี้เป็น Version ล่าสุด ณ วันที่ 11/09/2024

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
### Step 6: Reload and Restart service node_exporter

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl restart node_exporter
```
### Step 7 : Test Access Node Exporter

use   IP:9100  on Web Browser and Check
