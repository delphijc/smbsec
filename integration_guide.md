# Integration Guide for Open‑Source Security Tools

## 1. Overview
This guide walks you through connecting the following open‑source components into a cohesive security stack:

| Tool | Purpose |
|------|---------|
| **Elastic SIEM** | Central log ingestion, search, and alerting |
| **Wazuh** | Host‑based intrusion detection and log collection |
| **OpenCTI** | Threat‑intel platform and taxonomy management |
| **Llama‑2** | Natural‑language summarization of alerts |
| **FAISS** | Vector similarity search for IOC enrichment |
| **Stable‑Baselines3** | Reinforcement‑learning playbook execution |
| **Neo4j + GNNs** | Graph analytics for supply‑chain and lateral‑movement detection |

The stack is built on Docker Compose for simplicity, but the same steps apply to bare‑metal or cloud VMs.

## 2. Prerequisites
1. Docker & Docker‑Compose installed.
2. A machine with at least 8 GB RAM (12 GB recommended for GPU workloads).
3. NVIDIA GPU with Docker‑NVIDIA‑Runtime if you plan to run Llama‑2 locally.
4. Git to clone the repo.

## 3. Repository Layout
```
├── docker-compose.yml
├── elastic
│   ├── Dockerfile
│   └── config
├── wazuh
│   ├── Dockerfile
│   └── config
├── opencti
│   ├── Dockerfile
│   └── config
├── llama2
│   ├── Dockerfile
│   └── config
├── faiss
│   ├── Dockerfile
│   └── config
├── stable-baselines
│   ├── Dockerfile
│   └── config
├── neo4j
│   ├── Dockerfile
│   └── config
└── integration
    └── scripts
```
Each sub‑directory contains a minimal Dockerfile and configuration files.

## 4. Step‑by‑Step Integration
### 4.1 Spin Up the Core Stack
```bash
# Clone the repo
git clone https://github.com/delphijc/security-stack.git
cd security-stack

# Build images (GPU runtime for llama2)
docker compose build

# Start services
docker compose up -d
```
Verify all containers are healthy:
```bash
docker compose ps
```
You should see `elasticsearch`, `kibana`, `wazuh-manager`, `opencti`, `llama2`, `faiss`, `stable-baselines`, `neo4j`.

### 4.2 Configure Elastic SIEM
1. Open Kibana at `https://localhost:5601`.
2. Enable the **SIEM** app via **Stack Management → Security → SIEM**.
3. Create an **Elasticsearch index pattern** for Wazuh logs: `wazuh-*`.
4. Set up **Wazuh** to ship logs to Elastic:
   - In `wazuh/config/ossec.conf`, set `output` to `elasticsearch` with host `elasticsearch:9200`.
   - Restart Wazuh: `docker compose restart wazuh-manager`.

### 4.3 Connect Wazuh to OpenCTI
1. In OpenCTI, create a **custom connector**:
   - `Connector Type: Wazuh`.
   - `Endpoint: http://wazuh-manager:55000`.
   - `API Key: <generated>`.
2. Map Wazuh alerts to OpenCTI **Observables** (e.g., file hashes, IPs).
3. Enable **OpenCTI enrichment** in Elastic SIEM via the **OpenCTI plugin**.

### 4.4 Llama‑2 Summarization Service
1. Expose the Llama‑2 inference endpoint at `http://llama2:8000/summarize`.
2. In Elastic SIEM, create a **scripted field** that calls the Llama‑2 API:
   ```json
   "script": {
     "source": "import requests; requests.post('http://llama2:8000/summarize', json={'log': doc['message'].value}).json()['summary']"
   }
   ```
3. Use the scripted field in **alerting rules** to generate concise summaries.

### 4.5 FAISS Vector Search for IOC Enrichment
1. Load IOC vectors into FAISS via the `faiss/config/ingest.py` script.
2. In OpenCTI, configure the **FAISS connector**:
   - `Endpoint: http://faiss:5000/search`.
   - Map similarity scores to **confidence** in alerts.
3. In Elastic SIEM, add a **watcher** that queries FAISS when an IOC is detected.

### 4.6 Stable‑Baselines3 Playbook Execution
1. Deploy the RL policy at `http://stable-baselines:8000/act`.
2. In Elastic SIEM, create a **watcher** that sends alert context to the RL endpoint.
3. The RL service returns a **recommended action** (e.g., block IP, isolate host).
4. Use a **scripted action** in Elastic to trigger the corresponding Wazuh or firewall rule.

### 4.7 Neo4j + GNNs for Graph Analytics
1. Load host and network data into Neo4j via `neo4j/config/loader.py`.
2. Run the GNN model (`gnn/model.py`) to compute anomaly scores.
3. Expose the GNN inference endpoint at `http://neo4j:8000/analyze`.
4. In Elastic SIEM, add a **watcher** that queries Neo4j when a new host joins the network.
5. Use the anomaly score to adjust alert severity.

## 5. Orchestration & Automation
- **Celery** workers (in `integration/scripts/celery_worker.py`) coordinate cross‑service calls.
- **Prometheus** scrapes metrics from all services; **Grafana** dashboards visualize health and performance.
- **GitHub Actions** CI pipeline builds images, runs unit tests, and deploys to a staging environment.

## 6. Security & Compliance
- All services run behind a **reverse proxy** (NGINX) with TLS termination.
- Secrets are stored in **HashiCorp Vault** and injected via Docker secrets.
- Regular **OpenVAS** scans are scheduled via a cron job in the `openvas` container.

## 7. Troubleshooting
| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Elastic not reachable | Port conflict | Ensure `9200` and `5601` are free |
| Wazuh logs missing | Incorrect output config | Verify `output` section in `ossec.conf` |
| Llama‑2 API timeout | GPU not allocated | Add `--gpus all` to Docker run or use CPU fallback |
| FAISS search slow | Index not built | Run `faiss/config/ingest.py` again |
| RL policy returns no action | Empty replay buffer | Replay buffer must be populated with past alerts |

## 8. Next Steps
1. **Validate** each integration by generating a test alert and verifying the full pipeline.
2. **Automate** the deployment with Terraform for cloud environments.
3. **Document** the process in the book’s implementation guide chapter.

---

*This guide is a living document; feel free to adapt paths and configurations to your environment.*
