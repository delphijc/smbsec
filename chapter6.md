# Chapter 6 – Human-AI Collaboration Models

## 6.1 The Analyst-AI Workflow

The goal of AI in security is not to replace the analyst, but to augment them—to create a "Centaur" model where human intuition is paired with machine speed.

1.  **Alert Reception:** The workflow begins when an analyst logs into **Elastic SIEM**. Instead of a raw list of logs, they are presented with a prioritized queue of incidents.
2.  **AI-Assisted Triage:** When an analyst opens a case, they don't see just raw data. They see a **Llama-2 generated summary**: "Suspected Brute Force attack on Server X from IP Y." The system also provides a calculated **Risk Score**.
3.  **Contextual Enrichment:** Alongside the summary, the interface displays context from **OpenCTI** (e.g., "IP Y is a known Tor exit node") and **FAISS** (e.g., "This looks 95% similar to Incident #123 from last month").
4.  **Decision Support:** The **RL Playbook Engine** suggests a course of action: "Recommended: Block IP and reset user password (Confidence: 88%)."
5.  **Human Override:** The analyst reviews the evidence. If they agree, they click "Approve." If they disagree—perhaps they know that Server X is running a scheduled vulnerability scan—they click "Reject" and provide a reason.
6.  **Feedback Loop:** This interaction is the most valuable data point in the system. The analyst's decision (Approve/Reject) and rationale are logged and used to fine-tune the models, making the AI smarter for the next time.

## 6.2 Designing the Collaboration Interface

The User Interface (UI) is the bridge between the human and the machine. A poor UI can render even the best AI useless.

*   **Unified Dashboard:** We avoid "swivel-chair" analysis. The dashboard combines the **Kibana** visualization, a custom **React** panel for AI insights, and a chat-style interface for querying the LLM, all in one view.
*   **Context Panel:** A dedicated side panel shows the "Why." It lists the top-k similar incidents retrieved by FAISS, displays relevant IOCs, and shows a graph of related entities from OpenCTI.
*   **Action Panel:** This is where the rubber meets the road. It lists the suggested playbook steps. Each step shows a confidence score and a "Execute" button, giving the analyst control.
*   **Annotation Tool:** Analysts can tag alerts (e.g., "False Positive," "Malware," "Misconfiguration"). These tags become the labels for future supervised learning.

## 6.3 LLM-Powered Summaries and Explanations

"Black box" AI is dangerous in security. We need transparency.

*   **Prompt Engineering:** We use carefully crafted prompts to ensure the LLM explains *why* it flagged an event. "Explain why this log is suspicious, citing specific fields."
*   **Explainability:** The model highlights the specific parts of the log—the "features"—that contributed most to the risk score. "Risk Score is high because of 'failed login' combined with 'root user'."
*   **Knowledge Base Retrieval:** The system uses **RAG (Retrieval-Augmented Generation)**. It retrieves relevant snippets from the organization's internal wiki or playbook library and presents them alongside the alert, giving the analyst immediate access to the "Standard Operating Procedure."

## 6.4 Reinforcement Learning for Adaptive Playbooks

Static playbooks gather dust. Adaptive playbooks learn.

*   **State Representation:** The RL agent sees the incident as a vector of data points: alert severity, asset criticality, time of day, and analyst annotations.
*   **Reward Shaping:** We design the reward function to align with business goals. The agent gets a positive reward for actions that reduce **Mean Time to Containment (MTTC)**. It gets a large negative reward for actions that are flagged as **False Positives** by analysts.
*   **Policy Deployment:** The trained policy is deployed as a microservice. When an alert arrives, the orchestrator queries this service for the optimal action.
*   **Human-in-the-Loop:** Analyst overrides are treated as "interventions." These are added to the agent's replay buffer, effectively teaching the agent "Don't do that, do this instead."

## 6.5 Trust and Transparency

Trust is earned in drops and lost in buckets. To build trust in the AI:

*   **Audit Trail:** Every AI suggestion and every human decision is logged immutably. Timestamps, user IDs, and model version IDs are recorded.
*   **Model Versioning:** We track exactly which version of the model generated a specific alert. If a bad model update causes a spike in false positives, we can roll back immediately.
*   **Explainable AI (XAI):** We use techniques like **SHAP (SHapley Additive exPlanations)** or **LIME** to generate visual explanations of model predictions, showing exactly which features pushed the score over the threshold.
*   **Consent & Privacy:** We ensure that analyst performance data is anonymized. The goal is to improve the system, not to surveil the employees.

## 6.6 Training the Collaboration Loop

This is a continuous cycle of improvement.

1.  **Data Collection:** We capture all analyst interactions—clicks, approvals, rejections, comments—over a set period (e.g., 3 months).
2.  **Supervised Fine-Tuning:** We use this "gold standard" labeled data to fine-tune the LLM. The model learns to write summaries that analysts find useful and to suggest actions that analysts typically approve.
3.  **RL Retraining:** We periodically retrain the playbook policy using the updated replay buffer, allowing the agent to adapt to new attack patterns and changing business rules.
4.  **Evaluation:** We measure the impact. Did MTTC go down? Did the false-positive rate decrease? Did analyst satisfaction scores improve?

## 6.7 Deployment Checklist

Ready to go live?

- [ ] **Elastic SIEM + Kibana:** Dashboards are configured and receiving data.
- [ ] **LLM Inference Service:** Llama-2 is running on the GPU node and responding to API requests.
- [ ] **FAISS Index:** Populated with a baseline of historical incidents.
- [ ] **RL Policy Endpoint:** The playbook service is up and running.
- [ ] **Orchestrator:** Configured to route alerts between SIEM, AI, and Response tools.
- [ ] **Audit Logging:** Verified that all actions are being recorded to the database.
- [ ] **Analyst Training:** The team has been trained on how to use the new interface and how to interpret AI insights.

---

*This chapter outlines a practical, human-centric approach to integrating AI into the security analyst workflow, ensuring that automation augments rather than replaces human judgment.*