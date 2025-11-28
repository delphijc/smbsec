# Chapter 3 – AI-Driven Threat Detection

## 3.1 The Threat-Detection Pipeline

Building an effective AI-driven threat detection system requires a structured pipeline that transforms raw data into actionable intelligence. This pipeline is the backbone of the modern SOC.

1.  **Data Collection:** The process begins with ingestion. Logs from servers, firewalls, and applications, along with network flow data and endpoint telemetry, are collected. Simultaneously, external threat intelligence feeds are pulled in. This diverse dataset is funneled into a central storage layer, typically a combination of **Elasticsearch** for text search and a vector database for semantic analysis.
2.  **Pre-processing & Normalization:** Raw data is rarely ready for analysis. It must be parsed to extract key fields (IPs, timestamps, user agents) and normalized to a common schema. Categorical fields are encoded, and timestamps are standardized to UTC to ensure consistency across time zones.
3.  **Feature Extraction:** This is where the magic happens. Textual data, such as error messages or command-line arguments, is passed through **Sentence-Transformers** to create dense vector embeddings. Numeric fields are scaled to ensure they don't skew the models.
4.  **Model Scoring:** The processed features are fed into trained models. An autoencoder might flag a login attempt as anomalous, while a fine-tuned LLM might classify a script as malicious. Each event receives a risk score.
5.  **Enrichment:** A high risk score alone isn't enough. The event is cross-referenced against known Indicators of Compromise (IOCs) stored in **OpenCTI**. Is this IP associated with a known botnet? Has this file hash been seen in recent ransomware campaigns?
6.  **Alert Generation:** If the aggregated risk score exceeds a defined threshold, an alert is generated. This alert is sent to **Elastic SIEM**, creating a case for analysts to review.
7.  **Feedback Loop:** The pipeline doesn't end with the alert. Analyst decisions—whether they confirm or reject the finding—are captured and used to retrain the models, creating a system that gets smarter over time.

## 3.2 Log Analysis with LLMs

Large Language Models (LLMs) like Llama-2 have revolutionized log analysis. Traditional regex-based parsers are brittle; LLMs are flexible and semantic.

*   **Log Summarization:** Security logs can be verbose and cryptic. Llama-2 can read a complex, multi-line error log and generate a concise, human-readable summary, such as "Failed SSH login attempt for root user from external IP."
*   **Pattern Extraction:** LLMs excel at pattern recognition. They can scan through gigabytes of logs to identify recurring sequences that indicate an attack, such as the specific chain of commands used in a credential dumping attack, and even suggest new regex signatures to catch them in the future.
*   **Zero-Shot Classification:** With the right prompt, an LLM can classify logs into categories like "Malicious," "Suspicious," or "Benign" without needing a massive labeled training set. This "zero-shot" capability allows teams to deploy detection logic for new threats almost instantly.

## 3.3 IOC Enrichment via OpenCTI

Context is king in cybersecurity. **OpenCTI** serves as the central nervous system for threat intelligence, providing the "who, what, and why" behind an alert.

*   **STIX/TAXII Integration:** OpenCTI supports the industry-standard STIX/TAXII protocols, allowing it to ingest data from a wide array of public and private feeds. It exposes this data via a robust API.
*   **Automated Matching:** A lightweight integration script can periodically pull the latest IOCs from OpenCTI—such as malicious IPs, domains, and file hashes—and load them into a local **FAISS** index. Incoming logs are then checked against this index in real-time.
*   **Contextualization:** When a match is found, the alert is enriched with deep context: the threat actor group (e.g., APT29), their typical Tactics, Techniques, and Procedures (TTPs), and links to historical campaigns. This empowers analysts to understand the broader scope of an attack immediately.

## 3.4 Anomaly Detection with FAISS & Sentence-Transformers

Traditional signature-based detection fails against zero-day attacks. Anomaly detection fills this gap by identifying "unknown unknowns."

*   **Embedding Generation:** We use **Sentence-Transformers** to convert log lines into high-dimensional vectors. Semantically similar logs (e.g., "Login failed" and "Authentication error") will have vectors that are close together in this mathematical space.
*   **Indexing:** **FAISS** allows us to index these vectors efficiently. We can build an index of "normal" system behavior based on historical logs.
*   **Distance Thresholding:** For every new log entry, we query the FAISS index. If the new log's vector is far away from any known "normal" vectors (i.e., the distance exceeds a threshold), it is flagged as an anomaly.
*   **Explainability:** One of the benefits of this approach is explainability. We can show the analyst the "nearest neighbors"—the most similar past logs. If a log is anomalous because it looks like nothing we've ever seen, that's a strong signal.

## 3.5 Elastic SIEM as the Alert Hub

**Elastic SIEM** acts as the "single pane of glass" for the SOC, aggregating alerts from all detection engines.

*   **Alert Correlation:** Elastic SIEM doesn't just list alerts; it correlates them. Using its built-in correlation engine, it can link a "Phishing Email Detected" alert with a subsequent "Suspicious Process Started" alert on the same host, presenting them as a single incident.
*   **Dashboards:** Kibana dashboards provide real-time situational awareness. Analysts can monitor key metrics like alert volume, false-positive rates, and detection latency at a glance.
*   **Playbook Integration:** Elastic SIEM is actionable. Alerts can trigger automated workflows—such as sending a notification to Slack, creating a ticket in Jira, or launching a **Wazuh** active response to block an IP.

## 3.6 Continuous Improvement

An AI system is a living thing. It requires care and feeding to maintain its performance.

*   **Model Retraining:** Threat landscapes shift. Models must be periodically retrained on new data—including the latest labeled alerts—to adapt to evolving attack techniques.
*   **Feedback Channels:** The most valuable data comes from the analysts. A simple feedback mechanism—"Was this alert useful? Yes/No"—provides the ground truth needed to fine-tune the models.
*   **Metrics:** We must rigorously track performance. Metrics like **Precision** (are we crying wolf?), **Recall** (are we missing attacks?), and **Mean Time to Detect (MTTD)** are the ultimate yardsticks of our success.

---

*This chapter demonstrates how to combine open-source AI tools—OpenCTI, Elastic SIEM, FAISS, Sentence-Transformers, and Llama-2—to build a robust, automated threat-detection system that scales with a small security team.*