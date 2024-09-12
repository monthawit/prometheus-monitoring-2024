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
cd /etc/node_exporter
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
## Create TLS Cert
### Create TLS Key
```bash
openssl genrsa -out tls.key 2048
```
### Create TLS CSR
```bash
openssl req -new -key tls.key -out tls.csr
```
### Create TLS Cert From CA  10 Years
```bash
openssl x509 -req -in tls.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out tls.crt -days 3650
```
# Create node_exporter config.yaml

### move to node_exporter dir 
```bash
cd /etc/node_exporter
```
### Create file
```bash
vi /etc/node_exporter/config.yml
```
add code 
```yaml
basic_auth_users:
  add-your-username: Password Gen From htpasswd
tls_server_config:
  cert_file: /etc/node_exporter/tls.crt
  key_file: /etc/node_exporter/tls.key
  #client_auth_type: RequireAndVerifyClientCert
  #client_ca_file: /etc/node_exporter/ca.crt
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


# Config Prometheus and ScrapeConfig on Kubernetes

### Create Basic Auth secret 
```bash
kubectl apply -f 01-node-exporter-basic-auth-secret.yaml
```

### Create tls secret
```bash
kubectl apply -f 02-node-exporter-tls-secret.yaml
```

### Create hostlist Configmap 
```bash
kubectl apply -f 03-node-exporter-hostlist-configmap.yaml
```
### Edit Prometheus Server
```bash
vi /home/user01/kube-prometheus/manifests/prometheus-prometheus.yaml
```
edit volume and volumemount for config configmap in deployment

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.53.0
  name: k8s
  namespace: monitoring
spec:
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: nfs-storageclass 
        resources:
          requests:
            storage: 100Gi
  alerting:
    alertmanagers:
    - apiVersion: v2
      name: alertmanager-main
      namespace: monitoring
      port: web
  enableFeatures: []
  externalLabels: {}
  image: quay.io/prometheus/prometheus:v2.53.0
  volumeMounts:
    - name: ols-ping-01
      mountPath: /etc/prometheus/targets/ols-ping-01
    - name: ols-snmp-01
      mountPath: /etc/prometheus/targets/ols-snmp-01    
    - name: ols-node-exporter-hostlist-01
      mountPath: /etc/prometheus/targets/ols-node-expoter-01    ##### node-exporter-hostlist-configmap
  volumes:
    - name: ols-node-exporter-hostlist-01    ##### node-exporter-hostlist-configmap
      configMap:
        name: node-exporter-ols-hostlist-01
    - name: ols-ping-01
      configMap:
        name: bbex-ols-ping-01-hostlist    
    - name: ols-snmp-01
      configMap:
        name: snmp-targets-config      
  nodeSelector:
    kubernetes.io/os: linux
  podMetadata:
    labels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
      app.kubernetes.io/version: 2.53.0
  podMonitorNamespaceSelector: {}
  podMonitorSelector: {}
  probeNamespaceSelector: {}
  probeSelector: {}
  replicas: 2
  resources:
    requests:
      memory: 400Mi
  ruleNamespaceSelector: {}
  ruleSelector: {}
  scrapeConfigNamespaceSelector: {}
  scrapeConfigSelector: {}
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus-k8s
  serviceMonitorNamespaceSelector: {}
  serviceMonitorSelector: {}
  version: 2.53.0
  remoteWrite:
  - url: http://mimir-ingress.undyingk8s.olsdemo.com/api/v1/push
    name: undyingk8s
    basicAuth:
      username:
        name: kubepromsecret
        key: username
      password:
        name: kubepromsecret
        key: password
    headers:
      "X-Scope-OrgID": undyingk8s
```

###
