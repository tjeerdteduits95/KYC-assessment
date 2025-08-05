# 🧾 KYC assessment – Documentation

## Overview

This project implements a simplified Know Your Customer (KYC) solution using **Azure Databricks**, simulating ingestion, enrichment, risk scoring, and model integration for client and transaction monitoring. The solution includes structured risk rules, an anomaly detection model, and optional LLM augmentation via **Azure OpenAI**.

---

## 🔧 Design Decisions and Trade-offs

### 1. Data Modeling
- **Normalized structure**: Separated core entities (`clients`, `transactions`, `aml_risk_score`, `risk_events`, `model_outputs`) for clarity and scalability.
- **AML Risk Score Table**: Modeled `aml_risk_score` separately to allow country-based enrichment and updates without modifying core client records.
- **ER Diagram**: Created using Draw.io for visualization; designed with future extensibility and scalability in mind. Any additional risk factors, transaction or client information can be added without the need of changing the relations. 

**Trade-off**: Normalization (organising the data into separate, related tables) increases join complexity but enables flexibility for future schema evolution.

---

### 2. Lakehouse Architecture
- Adopted a medallion architecture **Bronze → Silver → Gold** layer strategy:
  - **Bronze**: Raw ingested data (CSV format).
  - **Silver**: Cleaned and enriched data (enriched `clients` and `transactions` as well as `risk events` and `model outputs`).
  - **Gold**: Risk-scored outputs for consumption by BI tools or APIs.

**Trade-off**: While more complex than a single pipeline, the layered approach enables reproducibility, modularity, and better governance. 

---

### 3. Risk Scoring
- Business rules include:
  - Flagging transactions over a defined threshold (e.g., €10,000).
  - Country-based risk scores using `aml_risk_score`.
  - Flagging transactions with a 30 day rolling window sum of €50,000.
  - Flagging transactions with a 30 day rolling window count of 10.
- **Trigger reasons** are consolidated into a single column for easier filtering and BI reporting.

**Trade-off**: Rule-based logic is simple but may miss subtle fraud patterns; hence the addition of ML. For future improvements with regards to parametrize the thresholds and models here, see next section.

---

### 4. ML-Based Anomaly Detection
- Chose **Isolation Forest** for detecting anomalous transaction behavior.
- Model is trained using PySpark, integrated into the pipeline, and outputs:
  - `ml_risk_score`
  - `confidence_score`

**Trade-off**: Isolation Forest lacks interpretability; chosen for simplicity and performance over deep model insights.

---

### 5. Possible LLM Integration
- **Azure OpenAI** used via API to analyze unstructured fields (e.g., transaction descriptions or client comments).
- Data comes from the Silver tables in Azure Data Lake Or directly from Spark DataFrames in Databricks. (e.g., enriched transactions and client data).
- A Databricks Notebook step calls the Azure OpenAI REST API with requests.
- Authentication happens via API key stored in Azure Key Vault.
- LLM returns enriched annotations like risk indicators, summaries, entity tags.
- Outputs stored alongside risk events for contextual insights and can be consumed by Power BI dashboards or SQL warehouse.

**Describe how unstructured data would be passed to the LLM**
- Unstructured data refers to free text fields such as notes, transaction descriptions, chat logs, etc
- Ingested via Azure Data Lake storage, then Azure OpenAI API is called within Databricks to for example classify the risk in a certain transaction/client
- Stored in Silver/Gold tables for reporting, rule triggers, or ML models

**Describe how structured would be passed to the LLM**
- Structured data is queried from Silver tables in Databricks.
- Row-level context is converted to text and passed to the LLM.
- Results are appended as columns (e.g., llm_risk_flag, llm_reason) in Gold tables.


**Trade-off**: External API call adds latency and cost; mitigated by caching and batch inference.

---

### 6. Pipeline Orchestration
- Built modular, reusable notebooks (Ingest → Enrich → Score → Serve).
- Orchestrated with **Databricks Jobs** (or alternative: Airflow).

**Trade-off**: Modular notebooks require coordination but allow easier testing and versioning.

---

### 7. CI/CD
- CI/CD process described in drawio diagram.

**Trade-off**: CI/CD complexity increases early in project lifecycle but offers long-term maintainability and team scaling.

---

## 🚀 Ideas for Future Improvements

### 1. Parametrize rule scoring and ML scoring steps
- Use parameters for thresholds 
- Use parameters to vary in terms of ML models (and subsequently use model registry in MLflow)

### 2. Data Quality Layer
- Add automatic data validation (e.g., using Deequ or Great Expectations).
- Introduce data SLAs and alerting on schema drift or missing values.

### 3. Model Monitoring
- Track drift or degradation in the anomaly detection model.
- Implement continuous model retraining using MLflow and Delta Live Tables.

### 4. Real-time Ingestion
- Replace batch CSV load with streaming ingestion from event sources (Kafka, Event Hub).

### 5. Explainable AI
- Replace or augment Isolation Forest with models that provide feature attribution (e.g., SHAP values).
- Use GPT-based summarization to explain scores in natural language.

### 6. Role-Based Dashboards
- Use Power BI with row-level security to build dashboards for compliance officers, analysts, and auditors.

### 7. LLM Usage Expansion
- Expand transaction data with transaction descriptions and apply LLMs to detect anomalies or suspicious transactions
- Expand client data with onboarding comments or client notes and apply LLMs
- Perform name matching (e.g., PEPs/sanctions) using LLMs with vector embeddings.


---

## 📎 Files Delivered

| File / Notebook               | Purpose                                  |
|------------------------------|------------------------------------------|
| `notebooks/01 Ingest data.ipynb`    | Load and store raw CSV data              |
| `notebooks/02 Enrich data.ipynb`   | Join, enrich, apply rule-based scoring   |
| `notebooks/03 Rule scoring.ipynb`  | Generate risk event logs and reasons     |
| `notebooks/04 ML scoring.ipynb`        | Train and apply Isolation Forest model   |
| `CI CD.drawio`       | CI/CD deployment flow using Azure Pipelines |
| `Cloud architecture.drawio`| Cloud architecture in Azure + Databricks |
| `ER.drawio`          | ER diagram for entities and relationships  |
| `aml_risk.csv`          | Raw CSV input data - AML risk list from ECB  |
| `clients.csv`          | Raw CSV input data - clients  |
| `transactions.csv`          | Raw CSV input data - transactions  |