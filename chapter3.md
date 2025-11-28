# Chapter 3 – AI‑Driven Threat Detection

## 3.1 The Threat‑Detection Pipeline
1. **Data Collection** – Logs, network flows, endpoint telemetry, and threat‑intel feeds are ingested into a central store (Elasticsearch, PostgreSQL, or a vector database).
2. **Pre‑processing & Normalization** – Structured logs are parsed, timestamps aligned, and categorical fields encoded.
3. **Feature Extraction** – Textual fields (e.g., error messages, command lines) are embedded with **Sentence‑Transformers**; numeric fields are scaled.
4. **Model Scoring** – A trained model (e.g., an autoencoder or a fine‑tuned LLM) assigns a risk score to each event.
5. **Enrichment** – Scores are cross‑checked against known indicators of compromise (IOCs) from **OpenCTI** and enriched with threat‑intel feeds.
6. **Alert Generation** – High‑score events trigger alerts in **Elastic SIEM** or a custom dashboard.
7. **Feedback Loop** – Analysts label alerts; the labels feed back into the model for continual improvement.

## 3.2 Log Analysis with LLMs
- **Log Summarization** – Llama‑2 can condense long log entries into concise, human‑readable summaries.
- **Pattern Extraction** – The model can identify recurring malicious patterns (e.g., credential dumping, lateral movement) and suggest new IOC patterns.
- **Zero‑Shot Classification** – Using prompt engineering, the LLM can classify logs into categories (malicious, benign, suspicious) without explicit training data.

## 3.3 IOC Enrichment via OpenCTI
- **STIX/TAXII Integration** – OpenCTI exposes a REST/TAXII API that can be queried for the latest threat actors, malware, and vulnerability data.
- **Automated Matching** – A lightweight Python script pulls new IOCs from OpenCTI, stores them in a vector index (FAISS), and performs similarity search against incoming logs.
- **Contextualization** – Each alert can be annotated with the actor’s TTPs, known malware families, and historical campaign data.

## 3.4 Anomaly Detection with FAISS & Sentence‑Transformers
- **Embedding Generation** – Logs are converted to dense vectors using Sentence‑Transformers.
- **Indexing** – FAISS builds an approximate nearest‑neighbor index for millions of vectors.
- **Distance Thresholding** – Events that fall outside a defined similarity radius are flagged as anomalies.
- **Explainability** – The nearest neighbors are displayed to analysts, providing context for why an event is anomalous.

## 3.5 Elastic SIEM as the Alert Hub
- **Alert Correlation** – Elastic SIEM aggregates alerts from multiple sources and correlates them using machine‑learning‑based correlation rules.
- **Dashboards** – Kibana dashboards provide real‑time visibility into threat‑detection metrics (alert volume, false‑positive rate, detection latency).
- **Playbook Integration** – Alerts can trigger automated playbooks (e.g., Wazuh rules, SOAR actions) for rapid response.

## 3.6 Continuous Improvement
- **Model Retraining** – Periodically retrain models on newly labeled data to adapt to evolving attack patterns.
- **Feedback Channels** – Analysts can provide feedback via a simple web form; the data is fed back into the training pipeline.
- **Metrics** – Track precision, recall, and mean time to detect (MTTD) to measure the effectiveness of the AI‑driven pipeline.

---

*This chapter demonstrates how to combine open‑source AI tools—OpenCTI, Elastic SIEM, FAISS, Sentence‑Transformers, and Llama‑2—to build a robust, automated threat‑detection system that scales with a small security team.*