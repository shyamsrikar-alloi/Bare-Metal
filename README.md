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
# Task-2 
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
- Create Dedicated Monitoring User & Directories
```
sudo useradd -r -s /bin/false -d /opt/monitoring monitoring
```
```
sudo mkdir -p /opt/monitoring/{data,config,logs,scripts}
sudo chown -R monitoring:monitoring /opt/monitoring
sudo chmod 755 /opt/monitoring
```
