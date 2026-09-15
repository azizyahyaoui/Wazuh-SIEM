# Installation via Docker Compose (Single-Node)

Deploying Wazuh via Docker offers a clean, isolated, and replicable way to get the stack running.

## System Requirements
- **Docker**: 20.10+
- **Docker Compose**: 1.29+
- **RAM**: Minimum 4GB
- **Storage**: Minimum 20GB free space

## 1. Install Docker & Docker Compose

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io docker-compose

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
docker-compose --version

# Add user to docker group (optional, to run without sudo)
sudo usermod -aG docker $USER
```

## 2. Deployment Options

### Option A: Using Official Repository (Recommended)

```bash
# Clone the official Wazuh Docker repository
git clone https://github.com/wazuh/wazuh-docker.git -b v4.11.2
cd wazuh-docker/single-node

# Generate SSL Certificates
docker-compose -f generate-indexer-certs.yml run --rm generator

# Start services
docker-compose up -d
```

### Option B: Using Standalone Docker Compose File

```bash
# Create working directory
mkdir -p ~/wazuh-docker
cd ~/wazuh-docker

# Download official docker-compose file
curl -s https://raw.githubusercontent.com/wazuh/wazuh-docker/4.x/docker-compose.yml -o docker-compose.yml

# Create .env file for configuration
cat > .env << EOF
INDEXER_USERNAME=admin
INDEXER_PASSWORD=SecurePassword123
FILEBEAT_SSL_VERIFICATION_MODE=full
SSL_CERTIFICATE_AUTHORITIES=/etc/ssl/certs/ca.crt
SSL_CERTIFICATE=/etc/ssl/certs/node.crt
SSL_KEY=/etc/ssl/private/node.key
EOF

# Create volumes directory
mkdir -p certs

# Start services
docker-compose up -d

# Check running containers
docker-compose ps

# View logs
docker-compose logs -f wazuh.manager
```

### Option C: Using Docker Run Commands

```bash
# Pull official Wazuh image
docker pull wazuh/wazuh:4.7.0

# Create network
docker network create wazuh-net

# Run Wazuh Manager container
docker run -d \
  --name wazuh-manager \
  --network wazuh-net \
  -p 1514:1514 \
  -p 1515:1515 \
  -p 514:514/udp \
  -p 55000:55000 \
  -e INDEXER_URL=https://wazuh.indexer:9200 \
  -e INDEXER_USERNAME=admin \
  -e INDEXER_PASSWORD=SecurePassword123 \
  -e FILEBEAT_SSL_VERIFICATION_MODE=full \
  -v wazuh-mgr-conf:/var/ossec/etc \
  -v wazuh-mgr-logs:/var/ossec/logs \
  wazuh/wazuh:4.7.0

# Verify container
docker ps | grep wazuh
```

## 3. Official Docker Image Tags

```text
wazuh/wazuh:4.7.0            # Latest stable release
wazuh/wazuh:4.7.0-alpine     # Lightweight Alpine Linux version
wazuh/wazuh:latest           # Bleeding edge
```

## 4. Verification & Management

```bash
# Check container status
docker ps -a

# Check container logs
docker logs wazuh-manager

# Access Wazuh Manager shell
docker exec -it wazuh-manager bash

# Check Wazuh status inside container
docker exec -it wazuh-manager /var/ossec/bin/wazuh-control status
```

## 5. Ports Reference

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| Wazuh Manager | 1514 | TCP | Agent communication |
| Wazuh Manager | 1515 | TCP | Agent enrollment |
| Wazuh Manager | 514 | UDP | Syslog input |
| Wazuh API | 55000 | TCP | REST API |
| Elasticsearch / Indexer | 9200 | TCP | HTTP API |
| Kibana / Dashboard | 443 / 5601 | TCP | Web UI |

## 6. Accessing Dashboard & Initial Setup

1. Open your browser and navigate to `https://127.0.0.1:443` or `http://<your-server-ip>:5601`.
2. Default Credentials:
   - **Username:** `admin`
   - **Password:** `SecretPassword` (or `admin`)

```bash
# Change admin password via API:
curl -u admin:admin -X PUT "https://your-server-ip:55000/security/users/admin/password" \
  -H 'Content-Type: application/json' \
  -d '{"new_password": "YourNewPassword123"}'
```

## 7. Troubleshooting & Security Best Practices

### Troubleshooting Commands

```bash
# Container won't start
docker logs wazuh-manager

# Permission issues
sudo chown -R 1000:1000 /path/to/volume

# Reset and restart
docker-compose down -v
docker-compose up -d

# Check resource usage
docker stats
```

### Security Best Practices

- **Change default passwords immediately**
- **Use SSL/TLS certificates** for all communications
- **Restrict network access** to Wazuh ports
- **Configure Firewall (UFW)**:
  ```bash
  sudo ufw allow 443/tcp   # Dashboard
  sudo ufw allow 55000/tcp # Wazuh API
  sudo ufw allow 1514/tcp  # Agent communication
  ```

---

## Change config inside docker


To modify your Wazuh manager configuration in a single container, you'll want to update `/var/ossec/etc/ossec.conf`. Depending on your workflow, you can do this through the Wazuh Dashboard UI, directly in the container, or by mounting a local config file.

**Method 1: Through the Wazuh Dashboard (Easiest)**

1. Open the Wazuh Web UI.
2. Navigate to **Wazuh** > **Server Management** > **Settings** > **Edit configuration**.
3. Edit the XML directly in the browser editor.
4. Click **Save** and accept the prompt to restart the manager service.

**Method 2: Edit Live Inside the Container (`docker exec`)**

```bash
# Open a shell inside your Wazuh container
docker exec -it wazuh.manager bash

# Edit the config file
nano /var/ossec/etc/ossec.conf

# Restart the Wazuh control service to apply changes
/var/ossec/bin/wazuh-control restart

```

**Method 3: Copy to Host, Edit, and Copy Back (`docker cp`)**

```bash
# 1. Copy config from container to current host folder
docker cp wazuh.manager:/var/ossec/etc/ossec.conf ./ossec.conf

# 2. Edit locally on your host
nano ./ossec.conf

# 3. Copy back to container
docker cp ./ossec.conf wazuh.manager:/var/ossec/etc/ossec.conf

# 4. Restart the manager
docker exec -it wazuh.manager /var/ossec/bin/wazuh-control restart

```

**Method 4: Persistent Volume Mount (Best Practice for Docker)**
To prevent losing your configuration changes whenever the container is recreated, map a host `ossec.conf` directly into your `docker-compose.yml` or `docker run` command:

```yaml
services:
  wazuh.manager:
    image: wazuh/wazuh-manager:latest
    container_name: wazuh.manager
    volumes:
      - ./config/ossec.conf:/var/ossec/etc/ossec.conf:rw

```

### E.g activate File Integrity Monitoring

File Integrity Monitoring (FIM) in Wazuh is managed by the **Syscheck** module inside `/var/ossec/etc/ossec.conf`.

**1. Enable Syscheck & Add Target Directories**
Locate the `<syscheck>` XML block in `ossec.conf`, make sure it is enabled (`<disabled>no</disabled>`), and add the paths you want to monitor:

```xml
<syscheck>
  <disabled>no</disabled>

  <!-- Frequency of scheduled scans in seconds (e.g., 12 hours) -->
  <frequency>43200</frequency>

  <!-- Real-time monitoring for critical system directories -->
  <directories check_all="yes" realtime="yes">/etc,/usr/bin,/usr/sbin</directories>

  <!-- Track exact file content diffs (useful for web roots or script folders) -->
  <directories check_all="yes" report_changes="yes">/var/www/html</directories>

  <!-- Ignore frequently changing dynamic files -->
  <ignore>/etc/mtab</ignore>
  <ignore>/etc/hosts.deny</ignore>
</syscheck>

```

**Key Attribute Breakdown:**

* `check_all="yes"`: Validates file hashes (MD5, SHA1, SHA256), permissions, ownership, and file size.
* `realtime="yes"`: Uses `inotify` (Linux) or Directory Change Notifications (Windows) for instant alerts rather than waiting for the periodic scan.
* `report_changes="yes"`: Sends a diff of the text file modifications inside the Wazuh alert payload so you can see exactly what line was modified.

**2. Apply and Test the Changes**
Restart the Wazuh manager to load the updated configuration:

```bash
docker exec -it wazuh.manager /var/ossec/bin/wazuh-control restart

```

To test it, create or modify a file inside one of your monitored paths (e.g., `touch /etc/test-fim.txt`) and check your Wazuh Dashboard under **Security Events** or search for Rule ID `550` (file added) / `554` (file modified).