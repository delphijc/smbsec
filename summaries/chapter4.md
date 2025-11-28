# Chapter 4 – Summary

**Key Takeaways**
- Incident‑response lifecycle: detection → triage → containment → eradication → recovery → lessons learned.
- RL playbooks (Stable‑Baselines3) learn optimal actions from alert metadata, IOC context, and telemetry.
- Wazuh’s rule engine and active‑response scripts enable automated containment actions.
- OpenCTI enriches alerts with actor/malware TTPs and can receive new IOCs from playbooks.
- A lightweight SOAR‑style orchestrator coordinates Elastic SIEM, Wazuh, and the RL service, logging all actions for audit.
- Continuous improvement loop: analyst feedback, model retraining, and metrics (MTTC, FP rate, analyst effort).

**Actionable Next Steps**
1. Spin up Elastic SIEM, Wazuh, and OpenCTI in a lab environment.
2. Train a simple RL policy on a small set of simulated alerts.
3. Deploy the policy as a REST endpoint and hook it into the orchestrator.
4. Configure Wazuh active‑response scripts to block IPs or quarantine hosts.
5. Set up a PostgreSQL audit log and monitor metrics.

**Next Chapter Preview**
- Chapter 5 will dive into threat hunting with LLM‑powered query generation and automated data collection.
