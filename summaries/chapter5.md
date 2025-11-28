# Chapter 5 – Summary

**Key Takeaways**
- Low‑cost AI security stack: ingestion → feature store (PostgreSQL + FAISS) → model serving (FastAPI with Llama‑2/Mistral) → SIEM (Elastic) + orchestrator (Flask).
- Hardware: ~4 vCPU/16 GB RAM VM for Elastic, 1 x NVIDIA T4 GPU for inference, ~500 GB SSD storage. Rough monthly cost ≈ $50.
- Open‑source stack: Elastic Stack, Wazuh, OpenCTI, Llama‑2, Mistral, Sentence‑Transformers, FAISS, Stable‑Baselines3, Neo4j+GNNs, OpenVAS.
- Deployment steps: provision infra, install Elastic/Wazuh/OpenCTI, train/fine‑tune models, set up feature store & FAISS, deploy orchestrator, test alerts, monitor, document.
- Maintenance: nightly retraining, ILM policies, TLS hardening, cost‑optimization via spot instances and auto‑scaling.

**Actionable Next Steps**
1. Spin up a small VM (t3.medium) and a GPU instance (g4dn.xlarge) on AWS or equivalent.
2. Install Elastic Stack and enable SIEM.
3. Deploy Wazuh manager and agents.
4. Run OpenCTI Docker Compose stack.
5. Fine‑tune Llama‑2 on a sample log set.
6. Train a simple RL policy with Stable‑Baselines3.
7. Expose models via FastAPI and hook into the Flask orchestrator.
8. Configure Elastic SIEM webhooks to trigger the orchestrator.
9. Verify that Wazuh active‑response scripts block IPs.
10. Set up Prometheus + Grafana for monitoring.

**Next Chapter Preview**
- Chapter 6 will cover advanced threat hunting using LLM‑powered query generation and automated data collection.
