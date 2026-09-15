# Wazuh Docker Admin Password Reset (Cheat Sheet)

> Default password is bad habit

### 📌 Summary of Correct Docker Paths

| Target | Bare-Metal Path (Do NOT use) | Docker Container Path (USE THIS) |
| --- | --- | --- |
| **Certificates** | `/usr/share/wazuh-indexer/config/certs/` | `/usr/share/wazuh-indexer/certs/` |
| **Security Config** | `/usr/share/wazuh-indexer/config/opensearch-security/` | `/usr/share/wazuh-indexer/opensearch-security/` |

---

### Step 1: Generate a New Bcrypt Hash

Run a temporary container to hash your new password:

```bash
docker run --rm -ti wazuh/wazuh-indexer:latest bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/hash.sh

```

*(Copy the generated `$2y$12$...` hash output)*

---

### Step 2: Edit Host Files

1. **Update `config/wazuh_indexer/internal_users.yml**`:
Replace the `hash` under the `admin:` block:
Replace the old plain-text password in `INDEXER_PASSWORD` / manager environment variables.

> **Pro Tip:** If your password contains `$`, escape it as `$$` inside `docker-compose.yml`!

```yaml
admin:
  hash: "$2y$12$YOUR_NEW_HASH_HERE"
  reserved: true
  backend_roles:
  - "admin"
  description: "Demo admin user"
```


2. **Update `docker-compose.yml**`:

```yaml
services:
  wazuh.manager:
    image: wazuh/wazuh-manager:4.12.0
    hostname: wazuh.manager
    ...
    environment:
      - INDEXER_URL=https://wazuh.indexer:9200
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=YOUR_NEW_PASSWORD
...
...
...
 wazuh.dashboard:
    image: wazuh/wazuh-dashboard:4.12.0
    hostname: wazuh.dashboard
    restart: always
    ports:
      - 443:5601
    environment:
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=YOUR_NEW_PASSWORD
```

---

### Step 3: Restart Stack & Wait for Health check

```bash
docker compose down && docker compose up -d

```

⏱️ *Wait 2–3 minutes for the Indexer cluster state to hit GREEN before running Step 4.*

---

### Step 4: Apply Changes inside the Container (One-Liner Execution)

#### Method A: Interactive Shell

```bash
docker exec -it single-node-wazuh.indexer-1 bash

```

Inside the container, run:

```bash
export CACERT=/usr/share/wazuh-indexer/certs/root-ca.pem
export KEY=/usr/share/wazuh-indexer/certs/admin-key.pem
export CERT=/usr/share/wazuh-indexer/certs/admin.pem
export JAVA_HOME=/usr/share/wazuh-indexer/jdk

bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /usr/share/wazuh-indexer/opensearch-security/ \
  -nhnv \
  -cacert $CACERT \
  -cert $CERT \
  -key $KEY \
  -p 9200 \
  -icl \
  -h 127.0.0.1

```

#### Method B: One-Liner Execution from Host Machine 🔥

```bash
docker exec -it single-node-wazuh.indexer-1 bash -c "
export CACERT=/usr/share/wazuh-indexer/certs/root-ca.pem
export KEY=/usr/share/wazuh-indexer/certs/admin-key.pem
export CERT=/usr/share/wazuh-indexer/certs/admin.pem
export JAVA_HOME=/usr/share/wazuh-indexer/jdk

bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /usr/share/wazuh-indexer/opensearch-security/ \
  -nhnv \
  -cacert \$CACERT \
  -cert \$CERT \
  -key \$KEY \
  -p 9200 \
  -icl \
  -h 127.0.0.1
"

```

---

### 💡 Troubleshooting Pro-Tips

* **`ERR: Seems there is no OpenSearch running on localhost:9200`**
Always append `-h 127.0.0.1` to force loopback IP resolution inside the container.
* **`FileNotFoundException: root-ca.pem`**
Ensure path points to `/usr/share/wazuh-indexer/certs/`, **not** `/config/certs/`.
* **`FileNotFoundException: config.yml`**
Ensure `-cd` points to `/usr/share/wazuh-indexer/opensearch-security/`, **not** `/config/opensearch-security/`.