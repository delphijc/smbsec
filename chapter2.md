# Chapter 2 – Machine Learning Basics for Security

## 2.1 Core Machine-Learning Concepts

To effectively leverage AI in cybersecurity, one must first understand the underlying mechanics of Machine Learning (ML). At its core, ML is about teaching computers to learn from data without being explicitly programmed for every specific rule.

*   **Supervised vs. Unsupervised Learning:**
    *   **Supervised Learning** involves training a model on a labeled dataset, where the "correct answer" is known. In security, this might look like training a classifier on a dataset of known malicious and benign files. The model learns to map inputs (file features) to outputs (malicious/benign).
    *   **Unsupervised Learning** deals with unlabeled data. The model tries to find structure within the data itself. This is crucial for **anomaly detection**, where the goal is to identify patterns that deviate significantly from the norm—such as a user logging in from an unusual location at 3 AM—without having prior examples of that specific attack.

*   **Feature Engineering:** Raw data—logs, network packets, code—must be transformed into a format that algorithms can understand, typically numeric vectors. This process is called feature engineering. For example, a log message might be converted into a vector using techniques like TF-IDF or modern embeddings, while a network flow might be represented by packet size, duration, and port numbers.

*   **Model Evaluation:** How do we know if a model is working? We use metrics like **Accuracy** (overall correctness), **Precision** (how many flagged threats were actually threats), and **Recall** (how many actual threats were caught). In security, missing a threat (false negative) is often worse than a false alarm (false positive), so recall is critical. However, too many false positives lead to alert fatigue, so a balance is struck, often measured by the **ROC-AUC** score.

*   **Overfitting & Regularization:** A common pitfall is overfitting, where a model learns the training data *too* well, including its noise, and fails to generalize to new, unseen data. Techniques like **Regularization** (L1/L2 penalties) and **Dropout** (randomly ignoring neurons during training) help prevent this, ensuring the model remains robust.

## 2.2 Data Pipelines for Security

An AI model is only as good as the data that feeds it. A robust data pipeline is the circulatory system of an AI-enabled SOC.

1.  **Ingestion:** Data must be collected from diverse sources—firewalls, endpoints, cloud services. Tools like **Beats** or **Logstash** act as shippers, forwarding telemetry to a central repository.
2.  **Pre-processing:** Raw data is messy. It needs to be cleaned, normalized (e.g., converting all timestamps to UTC), and tokenized. For text data, this is where we might use **Sentence-Transformers** to generate semantic embeddings.
3.  **Storage:** The processed data needs a home. **Relational databases** (like PostgreSQL) are great for structured metadata. **Vector databases** (like FAISS or Milvus) are essential for storing embeddings, enabling fast similarity searches. **Elasticsearch** serves as a powerful engine for full-text search and log aggregation.
4.  **Serving:** Finally, the model needs to be accessible. This is often achieved through **REST** or **GraphQL APIs** (using frameworks like FastAPI) that allow other security tools to query the model for predictions or summaries in real-time.

## 2.3 Model Types & Use-Cases

Different security problems require different tools. Here is a breakdown of common model types and their applications:

| Model | Typical Security Use-Case | Example Tool |
| :--- | :--- | :--- |
| **Logistic Regression** | Simple binary classification (e.g., Spam vs. Ham, Malicious vs. Benign). | scikit-learn |
| **Random Forest** | robust anomaly detection and classification using ensembles of decision trees. | scikit-learn |
| **Autoencoder** | Unsupervised anomaly detection; learning the "normal" state to flag deviations. | PyTorch / TensorFlow |
| **LLM (e.g., Llama-2)** | Natural language tasks: summarizing logs, explaining alerts, extracting IOCs. | Hugging Face Transformers |
| **Graph Neural Network** | Analyzing relationships in data, such as tracking lateral movement or supply chain risks. | PyTorch Geometric |

## 2.4 Open-Source AI Tools for Security

The open-source community has produced a powerful arsenal of tools that rival commercial offerings.

*   **Llama-2:** A state-of-the-art Large Language Model (LLM) from Meta. It can be fine-tuned for security tasks like parsing complex logs or generating incident reports, running locally to preserve data privacy.
*   **Mistral:** A highly efficient, lightweight LLM that offers impressive performance for its size, making it ideal for on-premise inference where hardware resources are limited.
*   **Sentence-Transformers:** A Python framework for state-of-the-art sentence, text, and image embeddings. It allows us to mathematically compare the meaning of log messages to find semantically similar threats.
*   **FAISS (Facebook AI Similarity Search):** A library for efficient similarity search and clustering of dense vectors. It enables us to search through millions of log embeddings in milliseconds to find "nearest neighbors" (similar past incidents).
*   **Stable-Baselines3:** A set of reliable implementations of reinforcement learning algorithms in PyTorch. It can be used to train agents that learn optimal response strategies (playbooks) through simulation.

---

*This chapter equips readers with the foundational ML knowledge needed to build and evaluate security-specific models, and introduces the open-source tools that will be explored in later chapters.*