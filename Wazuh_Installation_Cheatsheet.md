# Wazuh Installation Cheat Sheet
## Ubuntu Server
---

## Part 1: Direct Installation on Ubuntu Server

### System Requirements
- **OS**: Ubuntu 18.04, 20.04, or 22.04 LTS (64-bit)
- **RAM**: Minimum 2GB (4GB+ recommended)
- **Storage**: Minimum 10GB free space
- **CPU**: 2+ cores recommended
- **Internet**: Required for package downloads

### Prerequisites
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl gnupg2 apt-transport-https software-properties-common

# Import GPG key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -

# Add Wazuh repository
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Update package list
sudo apt update
```

### Install Wazuh Manager
```bash
# Install Wazuh manager package
sudo apt install -y wazuh-manager

# Start and enable Wazuh service
sudo systemctl daemon-reload
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager

# Verify status
sudo systemctl status wazuh-manager
```

### Install Elasticsearch
```bash
# Import Elasticsearch GPG key
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -

# Add Elasticsearch repository
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

# Update and install
sudo apt update
sudo apt install -y elasticsearch=7.10.2

# Configure Elasticsearch
sudo nano /etc/elasticsearch/elasticsearch.yml
# Set: cluster.name: wazuh
#      node.name: node-1
#      network.host: 0.0.0.0
#      discovery.seed_hosts: ["127.0.0.1"]

# Start Elasticsearch
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

### Install Kibana
```bash
# Install Kibana
sudo apt install -y kibana=7.10.2

# Configure Kibana
sudo nano /etc/kibana/kibana.yml
# Set: server.host: "0.0.0.0"
#      elasticsearch.hosts: ["http://localhost:9200"]

# Start Kibana
sudo systemctl daemon-reload
sudo systemctl enable kibana
sudo systemctl start kibana

# Access Kibana at http://your-ip:5601
```

### Install Filebeat (for Wazuh logs)
```bash
# Install Filebeat
sudo apt install -y filebeat=7.10.2

# Copy Wazuh template
sudo curl -s https://raw.githubusercontent.com/wazuh/wazuh/4.2/extensions/filebeat/7.x/wazuh-template.json | sudo tee /etc/filebeat/wazuh-template.json

# Configure Filebeat
sudo nano /etc/filebeat/filebeat.yml
# Add Wazuh input configuration and Elasticsearch output

# Start Filebeat
sudo systemctl daemon-reload
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

### Useful Commands
```bash
# Check Wazuh Manager status
sudo systemctl status wazuh-manager

# View Wazuh logs
sudo tail -f /var/ossec/logs/ossec.log

# Restart all services
sudo systemctl restart wazuh-manager elasticsearch kibana filebeat

# Stop all services
sudo systemctl stop wazuh-manager elasticsearch kibana filebeat

# Check listening ports
sudo netstat -tlnp | grep LISTEN
```

---
