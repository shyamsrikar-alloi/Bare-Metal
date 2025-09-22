# Bare-Metal
# Task-1 Provision AWS EC2 instance for monitoring server
```
Bare-metal instance requirements:
Component	Minimum Specs
OS	Ubuntu 22.04 LTS (or CentOS Stream 9)
instance_type a1.metal
CPU	4 cores (Intel/AMD x64)
RAM	8GB (16GB preferred)
Storage	50GB SSD (with /opt/monitoring partition)
Network	1Gbps with Internet access
```
# Task-2 Perform base OS setup and package installation
- Update System Packages
```
sudo apt update && sudo apt upgrade -y
```
- Install Core System Packages
```
sudo apt install -y \
  curl wget git htop nano vim unzip ca-certificates gnupg \
  lsb-release net-tools sysstat iotop jq
```
-  Install Monitoring & Hardware Tools
```
sudo apt install -y \
  ipmitool smartmontools lm-sensors nvme-cli
```
# Task-3 Create monitoring user and directory structure
- Create Dedicated Monitoring User & Directories
```
sudo useradd -r -s /bin/false -d /opt/monitoring monitoring
```
```
sudo mkdir -p /opt/monitoring/{data,config,logs,scripts}
sudo chown -R monitoring:monitoring /opt/monitoring
sudo chmod 755 /opt/monitoring
```
- Configure Automatic Security Updates
```
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```
# Task-4 Implement server security hardening
- (Optional) Firewall & SSH Hardening
```
# Install UFW
sudo apt install -y ufw

# Default deny incoming, allow outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (adjust port if not default 22)
sudo ufw allow ssh

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose
SSH hardening:

Edit /etc/ssh/sshd_config:

PermitRootLogin no
PasswordAuthentication no


Restart SSH:

sudo systemctl restart ssh
```
# Task-5 Install and configure monitoring agent
- Download Node Exporter:
```
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
```
-  Download Node Exporter for ARM64
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-arm64.tar.gz

# Extract the tarball
tar xvf node_exporter-1.7.0.linux-arm64.tar.gz

# Move the binary to /usr/local/bin so it’s in your PATH
sudo mv node_exporter-1.7.0.linux-arm64/node_exporter /usr/local/bin/

# Optional: check installation
node_exporter --version
```
- Configure Node Exporter as a systemd service:
```
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=monitoring
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```
- Verify Node Exporter is running:
```
curl http://localhost:9100/metrics
```
- Download Prometheus AMD (lightweight agent is fine):
```
wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz
tar xvf prometheus-2.50.0.linux-amd64.tar.gz
sudo mv prometheus-2.50.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.50.0.linux-amd64/promtool /usr/local/bin/
```
- for ARM
```
# Download Prometheus for ARM64
wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-arm64.tar.gz

# Extract the tarball
tar xvf prometheus-2.50.0.linux-arm64.tar.gz

# Move binaries to /usr/local/bin
sudo mv prometheus-2.50.0.linux-arm64/prometheus /usr/local/bin/
sudo mv prometheus-2.50.0.linux-arm64/promtool /usr/local/bin/

# Optional: verify installation
prometheus --version
promtool --version
```
- Create a Prometheus config file
```
nano /opt/monitoring/config/prometheus.yml:
# paste this
# Global configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Scrape Prometheus itself
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

# Scrape Node Exporter metrics
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

# Optional: Textfile collector metrics for hardware (SMART/NVMe/IPMI)
  - job_name: 'textfile_metrics'
    static_configs:
      - targets: ['localhost:9100']   # Node Exporter textfile collector path

# Remote write to Grafana Cloud (optional)
# Uncomment and replace with your Grafana Cloud instance ID and API key
#remote_write:
#  - url: "https://prometheus-blocks-prod-us-central1.grafana.net/api/prom/push"
#    basic_auth:
#      username: "<Grafana Cloud instance ID>"
#      password: "<API key>"
```
```
prometheus --config.file=/opt/monitoring/config/prometheus.yml
```


```
sudo nano /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=ubuntu
Group=ubuntu
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/opt/monitoring/config/prometheus.yml \
  --storage.tsdb.path=/opt/monitoring/data

Restart=always

[Install]
WantedBy=multi-user.target
```
```
- sudo nano /etc/systemd/system/node_exporter.service
```
```
sudo systemctl daemon-reload
sudo systemctl restart prometheus
```
