# Chapter 10 – Practical Implementation Guide

## 10.1 Overview
This chapter provides a step‑by‑step blueprint for building, deploying, and maintaining the AI‑powered security stack described in the book. It assumes a small security team (5–15 analysts) with basic DevOps skills.

## 10.2 Prerequisites
- **Hardware** – At least one GPU‑enabled VM (e.g., NVIDIA T4) and one general‑purpose VM.
- **Software** – Docker, Docker‑Compose, Python 3.10+, Node 18+, and Terraform (optional).
- **Credentials** – Access to cloud provider (AWS, GCP, Azure) or on‑prem infrastructure.
- **Team** – 1–2 data scientists, 1–2 security engineers, 1–2 analysts.

## 10.3 Infrastructure Setup
1. **Provision VMs** – Use Terraform to spin up:
   - `elastic-node` (4 vCPU, 16 GB RAM)
   - `gpu-node` (1 x NVIDIA T4, 8 GB VRAM)
2. **Networking** – Create a private subnet; expose only necessary ports (5601 for Kibana, 9200 for Elasticsearch, 5000 for FastAPI).
3. **Security Groups** – Restrict inbound traffic to internal IPs; enable TLS termination.

## 10.4 Elastic Stack Installation
```bash
# On elastic-node
sudo apt-get update && sudo apt-get install -y apt-transport-https openjdk-11-jre
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.0-amd64.deb
sudo dpkg -i elasticsearch-7.17.0-amd64.deb
sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch

# Install Kibana
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.0-amd64.deb
sudo dpkg -i kibana-7.17.0-amd64.deb
sudo systemctl enable kibana && sudo systemctl start kibana
```
- Enable the SIEM app via Kibana UI.
- Configure Beats to ship logs to `elastic-node`.

## 10.5 Wazuh Deployment
```bash
# On elastic-node
sudo apt-get install -y wazuh-manager
sudo systemctl enable wazuh-manager && sudo systemctl start wazuh-manager
# On each endpoint
sudo apt-get install -y wazuh-agent
sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent
```
- Configure Wazuh to forward alerts to Elastic SIEM.

## 10.6 OpenCTI Setup
```bash
# On elastic-node
docker run -d --name opencti -p 8080:8080 opencti/platform:latest
# Create admin user via UI
```
- Import initial threat feeds (MISP, AlienVault).
- Expose the REST/TAXII API.

## 10.7 Model Training & Serving
1. **Data Preparation** – Collect 3 months of logs, label a subset.
2. **Fine‑Tune Llama‑2** – Use Hugging Face `transformers`.
   ```python
   from transformers import AutoModelForCausalLM, AutoTokenizer
   model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-chat-hf")
   tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-2-7b-chat-hf")
   # Fine‑tune on log summaries
   ```
3. **Serve with FastAPI** – Deploy on `gpu-node`.
   ```python
   from fastapi import FastAPI
   app = FastAPI()
   @app.post("/summarize")
   async def summarize(log: str):
       # Tokenize, generate, return summary
   ```
4. **RL Playbook** – Train Stable‑Baselines3 PPO policy.
   ```python
   from stable_baselines3 import PPO
   model = PPO("MlpPolicy", env, verbose=1)
   model.save("playbook_policy.zip")
   ```
5. **Serve RL Policy** – Expose via FastAPI.

## 10.8 Orchestration Layer
- Deploy a Flask app that receives alerts from Elastic SIEM, calls the LLM summarizer, queries OpenCTI, and invokes the RL policy.
- Use Celery for background tasks.
- Store audit logs in PostgreSQL.

## 10.9 Automation & CI/CD
- **GitHub Actions** – Build Docker images, run unit tests, and deploy to VMs.
- **Ansible** – Provision and configure services.
- **Prometheus + Grafana** – Monitor CPU, GPU, memory, and model latency.

## 10.10 Security Hardening
- Enable TLS for all services.
- Use HashiCorp Vault for secrets.
- Rotate API keys weekly.
- Run regular vulnerability scans with OpenVAS.

## 10.11 Maintenance Checklist
| Task | Frequency | Owner |
|------|-----------|-------|
| Model Retraining | Monthly | Data Scientist |
| Log Retention Policy | Quarterly | Security Engineer |
| Compliance Audit | Annually | Compliance Officer |
| Infrastructure Scaling | As needed | DevOps |
| Incident Review | After each incident | Analyst |

## 10.12 Summary
By following this guide, a small security team can deploy a full AI‑powered security stack that delivers real‑time detection, automated response, and continuous improvement—all while staying within budget constraints.
