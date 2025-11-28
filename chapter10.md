# Chapter 10 – Practical Implementation Guide

## 10.1 Overview

This chapter transforms the concepts and architectures discussed in previous chapters into a concrete, executable reality. We provide a step-by-step blueprint for building, deploying, and maintaining the AI-powered security stack. This guide is designed for a small security team (5–15 analysts) with basic DevOps skills.

## 10.2 Prerequisites

Before we begin, ensure you have the following in place:

*   **Hardware:**
    *   **Elastic Node:** At least 4 vCPUs and 16 GB RAM.
    *   **GPU Node:** A machine with a dedicated GPU (e.g., NVIDIA T4 or A10G) and at least 8 GB VRAM for efficient model inference.
*   **Software:**
    *   **Docker & Docker Compose:** Essential for container orchestration.
    *   **Python 3.10+:** For scripting and model serving.
    *   **Node.js 18+:** For any custom dashboard extensions.
    *   **Terraform (Optional):** Recommended for Infrastructure-as-Code (IaC) management.
*   **Credentials:** Administrative access to your cloud provider (AWS, GCP, Azure) or on-premise virtualization platform.
*   **Team:** A small squad comprising 1–2 security engineers (for infrastructure), 1 data scientist (for model tuning), and 1–2 analysts (for testing and feedback).

## 10.3 Infrastructure Setup

We recommend using Terraform to provision your infrastructure to ensure reproducibility.

1.  **Provision VMs:**
    *   Create the `elastic-node` for the SIEM.
    *   Create the `gpu-node` for AI workloads. Ensure you select a machine image with NVIDIA drivers pre-installed (e.g., AWS Deep Learning AMI).
2.  **Networking:**
    *   Deploy both nodes into a **private subnet**.
    *   Use a **Bastion Host** or **VPN** for administrative access.
    *   Expose only necessary internal ports: `9200` (Elasticsearch), `5601` (Kibana), `5000` (FastAPI Model Service).
3.  **Security Groups:**
    *   Strictly limit inbound traffic. The `elastic-node` should only accept log traffic from your endpoints and API traffic from the `gpu-node`.

## 10.4 Elastic Stack Installation

On the `elastic-node`, we will install the core SIEM components.

```bash
# Import the Elastic PGP Key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Install Elasticsearch
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch

# Configure elasticsearch.yml (Enable Security!)
echo "xpack.security.enabled: true" | sudo tee -a /etc/elasticsearch/elasticsearch.yml
echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml

# Start Elasticsearch
sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch

# Generate Passwords
sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto

# Install Kibana
sudo apt-get install kibana
# Configure kibana.yml with the generated 'kibana_system' password
sudo systemctl enable kibana && sudo systemctl start kibana
```

*   **Post-Install:** Log into Kibana (port 5601), navigate to the **SIEM** app, and follow the guide to enable the detection engine.

## 10.5 Wazuh Deployment

Wazuh provides the endpoint visibility we need.

**Manager Installation (on `elastic-node` or separate VM):**
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

**Agent Deployment (on Endpoints):**
```bash
# Linux
sudo WAZUH_MANAGER="<MANAGER_IP>" apt-get install wazuh-agent
sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent

# Windows (PowerShell)
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.x.msi -OutFile wazuh-agent.msi; msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="<MANAGER_IP>"
```

*   **Configuration:** Edit `/var/ossec/etc/ossec.conf` on the Manager to enable **Active Response** and configure the connection to Elasticsearch.

## 10.6 OpenCTI Setup

We will deploy OpenCTI using Docker Compose on the `elastic-node`.

```yaml
version: '3'
services:
  opencti:
    image: opencti/platform:latest
    environment:
      - APP__ADMIN__EMAIL=admin@opencti.io
      - APP__ADMIN__PASSWORD=ChangeMe123!
      - APP__ADMIN__TOKEN=<UUID>
    ports:
      - "8080:8080"
    depends_on:
      - redis
      - elasticsearch
      - minio
      - rabbitmq
  # ... (Add dependency services: Redis, MinIO, RabbitMQ)
```

*   **Initial Data:** Once running, go to **Data > Connectors** and enable the **MISP** and **AlienVault** connectors to start pulling in threat feeds.

## 10.7 Model Training & Serving

On the `gpu-node`, we set up our AI services.

**1. Fine-Tune Llama-2:**
Use the Hugging Face `transformers` library to fine-tune the model on your parsed logs.

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer

model_id = "meta-llama/Llama-2-7b-chat-hf"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id, device_map="auto")

# ... Load your dataset and configure TrainingArguments ...

trainer = Trainer(model=model, args=training_args, train_dataset=dataset)
trainer.train()
model.save_pretrained("./fine-tuned-llama2")
```

**2. Serve with FastAPI:**
Expose the model as a microservice.

```python
from fastapi import FastAPI, HTTPException
from transformers import pipeline

app = FastAPI()
summarizer = pipeline("text-generation", model="./fine-tuned-llama2", device=0)

@app.post("/summarize")
async def summarize_log(log_entry: str):
    try:
        summary = summarizer(f"Summarize this security log: {log_entry}")[0]['generated_text']
        return {"summary": summary}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

**3. Run the Service:**
```bash
uvicorn main:app --host 0.0.0.0 --port 5000
```

## 10.8 Orchestration Layer

The orchestrator connects the dots.

*   **Flask App:** Create a simple Flask app that listens for HTTP POST requests from Elastic SIEM alerts.
*   **Logic:**
    1.  Parse the alert payload.
    2.  Send the log message to the `gpu-node` (`/summarize` endpoint).
    3.  Query OpenCTI for related IOCs.
    4.  (Optional) Query the RL Policy service for a recommended action.
    5.  Update the Elastic SIEM case with the AI summary and recommendation.

## 10.9 Automation & CI/CD

Treat your infrastructure and models as code.

*   **GitHub Actions:** Create workflows to automatically build your Docker images and deploy updates to your VMs whenever you push code to the `main` branch.
*   **Ansible:** Use Ansible playbooks to manage the configuration of your VMs, ensuring that security patches and dependencies are always up to date.
*   **Monitoring:** Deploy **Prometheus** node exporters on all VMs and visualize system health (CPU, Memory, GPU utilization) in **Grafana**.

## 10.10 Security Hardening

Do not neglect the security of your security tools.

*   **TLS Everywhere:** Use **Certbot** (Let's Encrypt) or an internal CA to issue certificates. Ensure all services (Elastic, Kibana, OpenCTI, FastAPI) communicate over HTTPS.
*   **Secrets Management:** Never hardcode passwords or API keys. Use **HashiCorp Vault** or Docker Secrets to inject credentials at runtime.
*   **Vulnerability Scanning:** Schedule daily **OpenVAS** scans against your own infrastructure to catch any misconfigurations or unpatched vulnerabilities.

## 10.11 Maintenance Checklist

| Task | Frequency | Owner |
| :--- | :--- | :--- |
| **Model Retraining** | Monthly | Data Scientist |
| **Log Retention Review** | Quarterly | Security Engineer |
| **Compliance Audit** | Annually | Compliance Officer |
| **Infrastructure Scaling** | As needed (based on monitoring) | DevOps Engineer |
| **Incident Post-Mortem** | After every critical incident | Lead Analyst |

## 10.12 Summary

By following this guide, you have built a system that rivals those of large enterprises. You have a centralized SIEM, deep endpoint visibility, automated threat intelligence, and a custom AI brain that understands your environment. This is the future of blue teaming: smart, automated, and accessible.
