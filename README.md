# 🧾 KYC assessment – Documentation

## Overview

This project implements a simplified Know Your Customer (KYC) solution using **Azure Databricks**, simulating ingestion, enrichment, risk scoring, and model integration for client and transaction monitoring. The solution includes structured risk rules, an anomaly detection model, and optional LLM augmentation via **Azure OpenAI**.

---

## 🔧 Design Decisions and Trade-offs

### 1. Data Modeling
- **Normalized structure**: Separated core entities (`clients`, `transactions`, `risk_events`, `model_outputs`) for clarity and scalability.
- **AML Risk Score Table**: Modeled separately (`aml_risk_score`) to allow country-based enrichment and updates without modifying core client records.
- **ERD**: Created using Draw.io for visualization; designed with future extensibility in mind.

**Trade-off**: Normalization increases join complexity but enables flexibility for future schema evolution.

---

### 2. Lakehouse Architecture
- Adopted a **Bronze → Silver → Gold** layer strategy:
  - **Bronze**: Raw ingested data (CSV format).
  - **Silver**: Cleaned and enriched data (joined, typed).
  - **Gold**: Risk-scored outputs for consumption by BI tools or APIs.

**Trade-off**: While more complex than a single pipeline, the layered approach enables reproducibility, modularity, and better governance.

---

### 3. Risk Scoring
- Business rules include:
  - Flagging transactions over a defined threshold (e.g., €10,000).
  - Country-based risk scores using `aml_risk_score`.
- **Trigger reasons** are consolidated into a single column for easier filtering and BI reporting.

**Trade-off**: Rule-based logic is simple but may miss subtle fraud patterns; hence the addition of ML.

---

### 4. ML-Based Anomaly Detection
- Chose **Isolation Forest** for detecting anomalous transaction behavior.
- Model is trained using PySpark, integrated into the pipeline, and outputs:
  - `ml_risk_score`
  - `confidence_score`

**Trade-off**: Isolation Forest lacks interpretability; chosen for simplicity and performance over deep model insights.

---

### 5. LLM Integration
- **Azure OpenAI** used via API to analyze unstructured fields (e.g., transaction descriptions or client comments).
- Integrated from Databricks using REST API.
- Outputs stored alongside risk events for contextual insights.

**Trade-off**: External API call adds latency and cost; mitigated by caching and batch inference.

---

### 6. Pipeline Orchestration
- Built modular, reusable notebooks (Ingest → Enrich → Score → Serve).
- Orchestrated with **Databricks Jobs** (or alternative: Airflow).

**Trade-off**: Modular notebooks require coordination but allow easier testing and versioning.

---

### 7. CI/CD
- CI/CD pipeline built with **Azure Pipelines**, deploying notebooks, jobs, and configurations to Databricks.
- Secured via **Azure Key Vault**.
- Includes PR validation, integration tests, and deployment to **Staging → Production**.

**Trade-off**: CI/CD complexity increases early in project lifecycle but offers long-term maintainability and team scaling.

---

## 🚀 Ideas for Future Improvements

### 1. Data Quality Layer
- Add automatic data validation (e.g., using Deequ or Great Expectations).
- Introduce data SLAs and alerting on schema drift or missing values.

### 2. Model Monitoring
- Track drift or degradation in the anomaly detection model.
- Implement continuous model retraining using MLflow and Delta Live Tables.

### 3. Real-time Ingestion
- Replace batch CSV load with streaming ingestion from event sources (Kafka, Event Hub).

### 4. Explainable AI
- Replace or augment Isolation Forest with models that provide feature attribution (e.g., SHAP values).
- Use GPT-based summarization to explain scores in natural language.

### 5. Role-Based Dashboards
- Use Power BI with row-level security to build dashboards for compliance officers, analysts, and auditors.

### 6. LLM Usage Expansion
- Expand transaction data with transaction descriptions and apply LLMs to detect anomalies or suspicious transactions
- Expand client data with onboarding comments or client notes and apply LLMs
- Perform name matching (e.g., PEPs/sanctions) using LLMs with vector embeddings.


---

## 📎 Files Delivered

| File / Notebook               | Purpose                                  |
|------------------------------|------------------------------------------|
| `notebook_1_ingestion.py`    | Load and store raw CSV data              |
| `notebook_2_processing.py`   | Join, enrich, apply rule-based scoring   |
| `notebook_3_risk_events.py`  | Generate risk event logs and reasons     |
| `notebook_4_model.py`        | Train and apply Isolation Forest model   |
| `ci_cd_diagram.drawio`       | CI/CD deployment flow using Azure Pipelines |
| `architecture_diagram.drawio`| Cloud architecture in Azure + Databricks |
| `data_model.drawio`          | ER diagram for tables and relationships  |