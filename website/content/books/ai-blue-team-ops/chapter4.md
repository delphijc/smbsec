+++
title = "Chapter 4 – Automated Incident Response"
date = "2025-11-28"
weight = 40
+++
# Chapter 4 – Automated Incident Response

## 4.1 The Incident‑Response Lifecycle
1. **Detection** – Alerts from the AI‑driven threat‑detection pipeline.
2. **Triage** – Automated scoring and enrichment to prioritize incidents.
3. **Containment** – Automated actions (e.g., blocking IPs, disabling accounts) via playbooks.
4. **Eradication** – Removal of malware, patching, or re‑image of compromised hosts.
5. **Recovery** – Restoring services and validating integrity.
6. **Lessons Learned** – Post‑mortem analysis and knowledge base updates.

## 4.2 Playbook Generation with Stable‑Baselines3
- **Reinforcement Learning (RL)** – Train an RL agent to decide the best action given an incident state.
- **State Representation** – Combine alert metadata, IOC context, and system telemetry into a vector.
- **Action Space** – Block IP, quarantine host, run script, or request analyst review.
- **Reward Function** – Minimize time to containment and false positives.
- **Deployment** – The trained policy is exposed via a REST endpoint; playbooks are generated on‑demand.

## 4.3 Integration with Wazuh
- **Rule Engine** – Wazuh’s rule engine can trigger custom scripts when a specific alert type is detected.
- **Active Response** – Wazuh can automatically execute commands (e.g., `iptables -A INPUT -s <ip> -j DROP`).
- **Correlation** – Wazuh alerts are forwarded to Elastic SIEM for correlation and enrichment.
- **Feedback Loop** – Wazuh’s event logs are fed back into the RL training pipeline to improve future playbooks.

## 4.4 OpenCTI for Threat Context
- **Actor & Malware Mapping** – OpenCTI provides detailed TTPs for known threat actors.
- **Automated Playbook Augmentation** – The RL agent can query OpenCTI to add context‑specific actions (e.g., block known malicious domains).
- **Data Sharing** – Playbooks can push new IOCs back into OpenCTI for community sharing.

## 4.5 Orchestration Layer
- **SOAR‑like Workflow** – A lightweight orchestrator (e.g., a Python Flask app) coordinates between Elastic SIEM, Wazuh, and the RL playbook service.
- **Webhook Endpoints** – Elastic SIEM posts alerts to the orchestrator; the orchestrator calls the RL service and then triggers Wazuh active responses.
- **Audit Trail** – All actions are logged in a PostgreSQL database for compliance and forensic analysis.

## 4.6 Continuous Improvement
- **Model Retraining** – After each incident, the RL agent’s experience replay buffer is updated with real outcomes.
- **Human‑in‑the‑Loop** – Analysts can override or approve playbooks; the decisions are recorded and used for supervised fine‑tuning.
- **Metrics** – Track mean time to containment (MTTC), false‑positive rate, and analyst effort reduction.

---

*This chapter shows how to turn detection into action by combining reinforcement learning, Wazuh’s active‑response capabilities, and OpenCTI’s threat context into an automated, continuously improving incident‑response workflow.*
