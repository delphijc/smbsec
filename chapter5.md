# Chapter 5 – Building a Low-Cost AI Security Stack

## 5.1 Architecture Overview

Designing a security stack for a small team requires a balance of capability, complexity, and cost. We propose a modular architecture that leverages best-in-class open-source tools.

```
+-------------------+          +-------------------+
|  Data Sources     |  --->    |  Ingestion Layer  |
+-------------------+          +-------------------+
          |                                 |
          v                                 v
+-------------------+          +-------------------+
|  Feature Store    |  --->    |  Model Serving    |
+-------------------+          +-------------------+
          |                                 |
          v                                 v
+-------------------+          +-------------------+
|  Alert Hub (SIEM) |  <---    |  Orchestration    |
+-------------------+          +-------------------+
```

*   **Ingestion Layer:** This is the entry point. Tools like **Beats**, **Syslog**, and custom Python collectors gather data from across the infrastructure and feed it into the system.
*   **Feature Store:** Data needs to be organized for AI. A **PostgreSQL** database holds structured telemetry and metadata, while a **FAISS** index stores the vector embeddings generated from logs.
*   **Model Serving:** The "brain" of the operation. **FastAPI** endpoints expose our AI models—Llama-2 for text analysis, Mistral for lightweight inference, and RL policies for decision making—allowing other components to query them via standard HTTP requests.
*   **Alert Hub (SIEM):** **Elastic SIEM** serves as the central command post. It aggregates alerts, provides visualization via Kibana, and acts as the primary interface for analysts.
*   **Orchestration:** A lightweight **Flask** application acts as the glue, coordinating actions between the SIEM, the AI models, and the response tools (Wazuh).

## 5.2 Hardware & Cost Considerations

One of the myths of AI is that it requires a supercomputer. For a small team, a modest investment can yield significant results.

| Component | Minimum Specs | Approx. Cost (USD) |
| :--- | :--- | :--- |
| **VM for Elastic Stack** | 4 vCPU, 16 GB RAM | ~$30/month (e.g., AWS t3.medium) |
| **GPU for LLM Inference** | 1 x NVIDIA T4 | ~$0.35/hr (e.g., AWS g4dn.xlarge) |
| **Storage** | 500 GB SSD | ~$10/month |
| **Backup** | 1 TB Object Storage | ~$5/month |
| **Network** | 1 Gbps | $0 (Internal traffic usually free) |

*   **Total Cost:** Roughly **$50 - $100/month** for a starter setup.
*   **Scaling:** The architecture is horizontally scalable. As data volume grows, you can add more Elastic nodes. If inference demand spikes, you can scale the GPU instances or use auto-scaling groups to manage costs (e.g., shutting down GPU nodes at night if 24/7 real-time inference isn't critical).

## 5.3 Open-Source Tool Stack

We have selected a suite of tools that are powerful, community-supported, and free to use.

| Tool | Purpose | Key Features |
| :--- | :--- | :--- |
| **Elastic Stack** | Log ingestion, search, and SIEM | Beats for shipping, Logstash for parsing, Kibana for visualization, and the dedicated SIEM app. |
| **Wazuh** | Host-based intrusion detection (HIDS) | Lightweight agent, powerful rule engine, and Active Response capabilities. |
| **OpenCTI** | Threat-intelligence management | STIX/TAXII support, graph-based data model, and rich connector ecosystem. |
| **Llama-2** | Log summarization, IOC extraction | Open weights, fine-tunable, and capable of running locally for privacy. |
| **Mistral** | Lightweight LLM inference | High performance-to-size ratio (7B params), ideal for lower-resource environments. |
| **Sentence-Transformers** | Semantic embeddings | Pre-trained models for converting text to vectors, optimized for GPU. |
| **FAISS** | Vector similarity search | Facebook's library for efficient similarity search, scaling to billions of vectors. |
| **Stable-Baselines3** | RL for playbook generation | Reliable implementations of PPO, DQN, and SAC algorithms for training response agents. |
| **Neo4j + GNNs** | Graph analytics | Graph database combined with Graph Neural Networks to find relationships in threat data. |
| **OpenVAS** | Vulnerability scanning | Comprehensive vulnerability scanner with a daily-updated feed of Network Vulnerability Tests (NVTs). |

## 5.4 Deployment Steps

Building this stack is a step-by-step process.

1.  **Provision Infrastructure:** Use **Terraform** or **CloudFormation** to define your infrastructure as code. Spin up your VMs, ensuring the GPU instance has the correct NVIDIA drivers installed.
2.  **Install Elastic Stack:** Follow the official Elastic guide to deploy Elasticsearch and Kibana. Enable the **SIEM** app and configure security (TLS, authentication).
3.  **Deploy Wazuh:** Install the Wazuh Manager on a central node and deploy Agents to your endpoints. Configure the `ossec.conf` to forward alerts to your Elastic instance.
4.  **Set Up OpenCTI:** Use Docker Compose to spin up the OpenCTI stack (Platform, Redis, RabbitMQ, Elasticsearch, MinIO). Import initial threat feeds like MISP or AlienVault.
5.  **Model Training & Serving:**
    *   Fine-tune **Llama-2** on a dataset of your internal logs to teach it your environment's "language."
    *   Train your **RL policy** using Stable-Baselines3 in a simulated environment.
    *   Wrap these models in a **FastAPI** application and deploy it to the GPU node.
6.  **Feature Store & FAISS Index:** Set up PostgreSQL. Write a script to batch-process historical logs, generate embeddings, and populate the FAISS index.
7.  **Orchestrator:** Deploy your Flask orchestration app. Configure **Webhooks** in Elastic SIEM to send alerts to this app.
8.  **Testing:** Don't wait for a real attack. Simulate alerts (e.g., using `logger` or a breach simulation tool) to verify that the pipeline works: Alert -> SIEM -> Orchestrator -> AI Model -> Response.
9.  **Monitoring & Logging:** Use **Prometheus** and **Grafana** to keep an eye on the health of your stack. Monitor CPU, RAM, disk usage, and model inference latency.
10. **Documentation:** Write a "Runbook" for your team. Document the architecture, API endpoints, and troubleshooting steps.

## 5.5 Maintenance & Scaling

A security stack is never "done."

*   **Model Retraining:** Set up automated jobs (using **Airflow** or cron) to retrain your models nightly or weekly. This prevents "model drift" as attack patterns change.
*   **Data Retention:** Configure **Index Lifecycle Management (ILM)** policies in Elastic. Keep "hot" data on SSDs for fast search, and move older data to cheaper "cold" storage or delete it after a set period.
*   **Security Hardening:** Practice what you preach. Enable **TLS** for all internal communication. Store API keys and secrets in a vault (like **HashiCorp Vault**), not in plain text config files.
*   **Cost Optimization:** Use **Spot Instances** for your GPU nodes to save up to 90% on compute costs. Implement auto-scaling to spin down resources when they aren't needed.

---

*This chapter provides a practical, low-cost blueprint for building a full AI-powered security stack that a small team can deploy, maintain, and scale.*