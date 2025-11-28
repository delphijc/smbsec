# Chapter 2 – Machine Learning Basics for Security

## 2.1 Core Machine‑Learning Concepts
- **Supervised vs. Unsupervised**: Labelled data for anomaly detection vs. clustering of unknown patterns.
- **Feature Engineering**: Transform raw logs, network flows, or threat‑intel into numeric vectors.
- **Model Evaluation**: Accuracy, precision/recall, ROC‑AUC, and the importance of a realistic validation set.
- **Overfitting & Regularization**: Techniques such as dropout, L1/L2 penalties, and cross‑validation.

## 2.2 Data Pipelines for Security
- **Ingestion**: Beats, Logstash, or custom collectors feeding into a central store (Elasticsearch, PostgreSQL, or a vector database).
- **Pre‑processing**: Normalization, tokenization, and embedding generation (e.g., Sentence‑Transformers for log text).
- **Storage**: Vector databases (FAISS, Milvus) for similarity search; relational DBs for structured telemetry.
- **Serving**: REST/GraphQL endpoints or batch jobs that score new data against the trained model.

## 2.3 Model Types & Use‑Cases
| Model | Typical Security Use‑Case | Example Tool |
|-------|--------------------------|--------------|
| Logistic Regression | Binary threat vs. benign classification | scikit‑learn |
| Random Forest | Feature‑rich anomaly detection | scikit‑learn |
| Autoencoder | Unsupervised anomaly detection | PyTorch / TensorFlow |
| LLM (e.g., Llama‑2) | Log summarization, IOC extraction | Hugging Face Transformers |
| Graph Neural Network | Threat‑intel graph analysis | PyTorch Geometric |

## 2.4 Open‑Source AI Tools for Security
- **Llama‑2**: Large‑language model for natural‑language log analysis and policy generation.
- **Mistral**: Lightweight LLM for on‑prem inference.
- **Sentence‑Transformers**: Generate dense embeddings for semantic similarity.
- **FAISS**: Efficient similarity search for large embedding collections.
- **Stable‑Baselines3**: Reinforcement‑learning framework for automated play‑book generation.

---

*This chapter equips readers with the foundational ML knowledge needed to build and evaluate security‑specific models, and introduces the open‑source tools that will be explored in later chapters.*