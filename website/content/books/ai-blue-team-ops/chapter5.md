+++
title = "Chapter 5 – Building a Low‑Cost AI Security Stack"
date = "2025-11-28"
weight = 50
+++
# Chapter 5 – Building a Low‑Cost AI Security Stack

## 5.1 Architecture Overview
```
+-------------------+          +-------------------+
|  Data Sources    |  --->   |  Ingestion Layer  |
+-------------------+          +-------------------+
          |                               |
          v                               v
+-------------------+          +-------------------+
|  Feature Store   |  --->   |  Model Serving    |
+-------------------+          +-------------------+
          |                               |
          v                               v
+-------------------+          +-------------------+
|  Alert Hub (SIEM)|  <---   |  Orchestration    |
+-------------------+          +-------------------+
```
- **Ingestion**: Beats, syslog, and custom collectors feed into a central store.
- **Feature Store**: A PostgreSQL database holds structured telemetry; FAISS indexes store embeddings.
- **Model Serving**: FastAPI endpoints expose LLMs (Llama‑2, Mistral) and RL policies.
- **Alert Hub**: Elastic SIEM aggregates alerts and provides dashboards.
- **Orchestration**: A lightweight Flask app coordinates playbooks and active responses.

## 5.2 Hardware & Cost Considerations
| Component | Minimum Specs | Approx. Cost (USD) |
|-----------|---------------|--------------------|
| VM for Elastic Stack | 4 vCPU, 16 GB RAM | $30/month (AWS t3.medium) |
| GPU for LLM inference | 1 x NVIDIA T4 | $0.35/hr (AWS g4dn.xlarge) |
| Storage | 500 GB SSD | $10/month |
| Backup | 1 TB | $5/month |
| Network | 1 Gbps | $0 |

- **Total**: Roughly $50/month for a small team.
- **Scaling**: Add nodes or upgrade GPU as data volume grows.

## 5.3 Open‑Source Tool Stack
| Tool | Purpose | Key Features |
|------|---------|--------------|
| **Elastic Stack** | Log ingestion, search, and SIEM | Beats, Logstash, Kibana, SIEM app |
| **Wazuh** | Host‑based intrusion detection | Agent, rule engine, active response |
| **OpenCTI** | Threat‑intel management | STIX/TAXII API, connectors |
| **Llama‑2** | Log summarization, IOC extraction | Hugging Face Transformers, fine‑tune |
| **Mistral** | Lightweight LLM for on‑prem inference | 7B parameters, fast inference |
| **Sentence‑Transformers** | Dense embeddings for similarity | Pre‑trained models, GPU support |
| **FAISS** | Approximate nearest‑neighbor search | Scales to billions of vectors |
| **Stable‑Baselines3** | RL for playbook generation | PPO, DQN, SAC implementations |
| **Neo4j + GNNs** | Graph analytics on threat intel | Cypher queries, PyTorch Geometric |
| **OpenVAS** | Vulnerability scanning | Open‑source scanner, CVE database |

## 5.4 Deployment Steps
1. **Provision Infrastructure** – Use Terraform or CloudFormation to spin up VMs, GPU instances, and storage.
2. **Install Elastic Stack** – Follow the official Elastic quickstart; enable the SIEM app.
3. **Deploy Wazuh** – Install the manager and agents; configure rules for common threats.
4. **Set Up OpenCTI** – Run the Docker Compose stack; import initial threat feeds.
5. **Model Training & Serving**
   - Fine‑tune Llama‑2 on internal logs.
   - Train an RL policy with Stable‑Baselines3.
   - Expose models via FastAPI.
6. **Feature Store & FAISS Index** – Load embeddings into FAISS; store metadata in PostgreSQL.
7. **Orchestrator** – Deploy the Flask app; configure webhooks in Elastic SIEM.
8. **Testing** – Simulate alerts; verify that playbooks trigger and Wazuh responds.
9. **Monitoring & Logging** – Use Prometheus + Grafana to monitor resource usage.
10. **Documentation** – Write a run‑book for onboarding new analysts.

## 5.5 Maintenance & Scaling
- **Model Retraining** – Schedule nightly retraining jobs; use Airflow for orchestration.
- **Data Retention** – Configure ILM policies in Elastic to tier older data to cheaper storage.
- **Security Hardening** – Enable TLS for all services; rotate secrets via Vault.
- **Cost Optimization** – Spot instances for GPU inference; use auto‑scaling groups.

---

*This chapter provides a practical, low‑cost blueprint for building a full AI‑powered security stack that a small team can deploy, maintain, and scale.*
