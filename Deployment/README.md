# ⚙️ Deploy — Setting Up n8n with Docker & Compose

This section documents how **n8n** was deployed on an **Ubuntu 22.04 Server VM** to automate enrichment and response actions in the *AlertFlow — Enrichment & Response* project.  
It uses **Docker + Docker Compose** for a modular, persistent, and easily reproducible setup.

---

## 🧩 Host Environment

| Parameter | Configuration |
|------------|---------------|
| **OS** | Ubuntu 22.04 LTS (Server) |
| **Memory** | 6.5 GB |
| **Processors** | 4 vCPUs |
| **Deployment Type** | Docker + Compose |
| **Webhook Endpoint** | `http://192.168.137.10:5678/webhook/enrichment-workflow` |
| **Persistent Volume** | `/var/lib/docker/volumes/n8n-docker_n8n_data/_data` |

---

## 🚀 Step-by-Step Deployment

### 1️⃣ Install Docker & Docker Compose

```bash
sudo apt update && sudo apt install docker.io docker-compose -y
sudo systemctl enable docker && sudo systemctl start docker
docker --version && docker-compose version
````

---

### 2️⃣ Create the `docker-compose.yml` File

Inside your chosen directory (e.g., `/opt/n8n`), create a new file named `docker-compose.yml`:

```yaml
version: '3.3'
services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    restart: always
volumes:
  n8n_data:
```

### 3️⃣ Launch the n8n Container

```bash
sudo docker-compose up -d
```

✅ **Verify that the container is running:**

```bash
docker ps
```
<img width="960" height="365" alt="Screenshot 2025-11-13 010113" src="https://github.com/user-attachments/assets/391e40ed-c815-414f-b183-59eb49b3b1de" />

---
### 4️⃣ Access the n8n Web Interface

Once running, open a browser and navigate to:

```
http://192.168.137.10:5678
```

This loads the **n8n Dashboard**, where workflows can be imported, credentials configured (VirusTotal, Gmail), and automation validated.

<img width="807" height="242" alt="Screenshot 2025-11-12 202341" src="https://github.com/user-attachments/assets/274d3971-8a10-462f-90a1-698b9bb600c4" />

---

## 💾 Data Persistence

n8n automatically stores workflows, credentials, and logs inside the persistent Docker volume:

```
/var/lib/docker/volumes/n8n-docker_n8n_data/_data
```

To back up your data safely:

```bash
sudo tar -czvf n8n_data_backup.tar.gz /var/lib/docker/volumes/n8n-docker_n8n_data/_data
```

This ensures that your workflows remain intact after system restarts or migrations.

---

## 🧠 Troubleshooting (Beginner Learnings)

| Issue                           | Symptom                          | Resolution                                                                  |
| ------------------------------- | -------------------------------- | --------------------------------------------------------------------------- |
| **Port Conflict**               | Port `5678` already in use       | Stop the conflicting service or change port mapping                         |
| **Permission Denied on Volume** | Container can’t write to volume  | `sudo chown -R 1000:1000 /var/lib/docker/volumes/n8n-docker_n8n_data/_data` |
| **Restart Loop**                | n8n container keeps restarting   | Checked logs via `docker logs n8n` to identify config issues                |
| **Workflow Import Error**       | JSON file not importing          | Verified export validity and corrected malformed JSON                       |
| **API Credential Error**        | API keys not found after restart | Reconfigured keys in **n8n → Credentials tab**                              |

💡 *As a beginner, I encountered and resolved multiple deployment challenges using AI tools such as **Claude**, **Grok**, and **ChatGPT**, which guided me in diagnosing Docker permission issues, container restarts, and port conflicts — turning setup roadblocks into structured learning experiences.*

---

## ✅ Verification Checklist

* [x] Docker & Compose installed successfully
* [x] Container launched and verified with `docker ps`
* [x] Web UI accessible via `http://192.168.137.10:5678`
* [x] Persistent volume mounted and writable
* [x] Credentials configured securely in n8n UI
* [x] Webhook endpoint ready for Wazuh integration:
  `http://192.168.137.10:5678/webhook/enrichment-workflow`

---

📌 *This deployment stage forms the automation backbone of the project — connecting Wazuh alerts to n8n’s enrichment and response engine through a containerized, low-code orchestration environment.*

---

