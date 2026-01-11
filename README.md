
# 🏎️ Formula 1 End-to-End Data Engineering Project

## Project Overview
This project is an enterprise-grade data engineering solution that processes over 70 years of Formula 1 racing data (1950-2025). It implements a **Lakehouse Architecture** on Azure to transform raw API/CSV data into actionable analytics for identifying dominant drivers and constructors.

The pipeline handles data ingestion, schema enforcement, transformation, and incremental loading using **Azure Databricks (PySpark)**, orchestrated by **Azure Data Factory (ADF)**, with final reporting in **Power BI**.

---

## Architecture & Design

The solution follows the **Medallion Architecture** (Bronze, Silver, Gold layers) to ensure data quality and scalability.

**![Architecture](https://github.com/user-attachments/assets/7271d368-20bf-4d5b-911c-fbb7a9664ae0)**

### **Data Flow:**
1.  **Ingestion (Bronze Layer):**
    * **Historical Backfill:** Bulk ingestion of legacy CSV files (1950–today) from Kaggle.
    * **Incremental Updates:** Automated weekly API calls to Jolpica/Ergast for the latest race results.
    * *Storage:* Azure Data Lake Gen2 (ADLS) `raw` container.
2.  **Transformation (Silver Layer):**
    * Data is cleaned, joined, and deduplicated.
    * **Schema Enforcement:** Strong typing applied to all columns to prevent data drift.
    * **Data Quality:** Null checks, date formatting, and standardizing column names.
    * *Format:* Stored as **Delta Tables** (Parquet + Transaction Log) for ACID compliance.
3.  **Aggregation (Gold Layer):**
    * Business logic applied to calculate specific performance metrics.
    * **Tables created:** `driver_standings`, `constructor_standings`, and `calculated_race_results`.
4.  **Reporting:**
    * **Power BI** connects directly to the Gold tables via Databricks Partner Connect for interactive visualization.

---

## Tech Stack

| Category | Technology | Usage |
| :--- | :--- | :--- |
| **Cloud Provider** | **Microsoft Azure** | Core Infrastructure |
| **Storage** | **ADLS Gen2** | Data Lake storage (Raw/Processed/Presentation) |
| **Compute** | **Azure Databricks** | Spark Clusters for heavy ETL processing |
| **Language** | **Python (PySpark)** | Transformation logic and SQL analysis |
| **Orchestration** | **Azure Data Factory** | Scheduling and pipeline dependency management |
| **Format** | **Delta Lake** | Open-source storage layer for reliability |
| **Security** | **Managed Identity** | Secure, keyless authentication between services |
| **BI** | **Power BI** | Interactive Dashboards |

---

## 📂 Repository Structure

This repository is organized as a **Monorepo** containing both infrastructure (ADF) and code (Databricks).

```text
formula1-data-pipeline/
├── adf/                          # Azure Data Factory Code (JSON)
│   ├── pipeline/                 # Pipelines (Ingest, Process, Transform)
│   ├── dataset/                  # Dataset definitions (ADLS & Databricks)
│   ├── linkedService/            # Connection strings (Key Vault, Storage)
│   └── trigger/                  # Schedule triggers (Weekly)
│
├── databricks/                   # PySpark Notebooks
│   ├── ingestion/                # (Bronze) Notebooks to ingest raw files/API
│   │   ├── 1.ingest_circuits.ipynb
│   │   ├── 2.ingest_races.ipynb
│   │   └── ...
│   ├── trans/                    # (Silver) Transformations & Logic
│   │   ├── 1.race_results.ipynb
│   │   ├── 2.driver_standings.ipynb
│   │   └── ...
│   ├── analysis/                 # (Gold) Ad-hoc analysis & aggregations
│   │   ├── 1.find_dominant_drivers.ipynb
│   │   └── ...
│   ├── includes/                 # Common functions & configuration paths
│   ├── raw/                      # Setup scripts for database creation
│   └── utils/                    # Utility scripts (Incremental load prep)
```

## Dashboards & Analytics

The final presentation layer is visualized in Power BI, connected directly to the **Gold Layer** (Databricks `f1_presentation` database).

### 1. Driver Dominance Analysis
**![Drivers Analysis](https://github.com/user-attachments/assets/2f3f0d8f-9fc3-4266-b38e-faa07f990344)**
* **Purpose:** Identifies the most successful drivers in F1 history based on total wins and win efficiency.
* **Key Metrics:**
    * **Total Wins:** The primary metric for success.
    * **Win %:** (Total Wins / Total Races) to highlight efficiency rather than just longevity.
    * **Avg Points:** Tracking performance consistency over career spans.
* **Insights:**
    * Visualizes the "Hamilton vs. Schumacher" debate.
    * Highlights high-efficiency drivers from early eras (e.g., Fangio) who had fewer races but higher win rates.

### 2. Constructor Standings & Team Dominance
**![Visualization](https://github.com/user-attachments/assets/adcd52aa-c604-453e-8f60-ccd0fd0afc18)**
* **Purpose:** Tracks the rise and fall of F1 teams (Constructors) over the decades.
* **Key Metrics:** Total Constructor Points and Championship Titles.
* **Visualization:**
    * **Heatmaps:** Show periods of total team dominance (e.g., Ferrari in early 2000s, Mercedes in 2014-2020).
    * **Trend Lines:** Visualizing the points trajectory for top teams like Red Bull, McLaren, and Ferrari.
