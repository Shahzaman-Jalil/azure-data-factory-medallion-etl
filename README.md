# Azure Data Factory Medallion ETL Pipeline

![Azure Data Factory](https://img.shields.io/badge/Azure-Data%20Factory-0078D4?logo=microsoftazure&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-Medallion%20(Bronze--Silver--Gold)-4B8BBE)
![Integration Runtime](https://img.shields.io/badge/Runtime-Self--Hosted-6A1B9A)
![Status](https://img.shields.io/badge/Project-ADF%20Git%20Integrated-success)

## Overview

This project is an **Azure Data Factory (ADF) Git-integrated ETL solution** (resource JSON artifacts by folder) implementing a **Medallion Architecture** for airline/retail-style booking analytics.  
It orchestrates ingestion from API, on-prem, and SQL sources into a lake-first layered model, then applies transformation and serving logic using ADF pipelines and Mapping Data Flows.

---

## Architecture (Medallion Pattern)

The solution follows a layered pipeline design:

```text
[Source Systems]
   |-- API
   |-- On-Prem Files (via Self-Hosted IR)
   |-- Azure SQL
        |
        v
[Bronze Layer - Raw Ingestion]
  API_Ingestion / onprem_ingestion / SQLToDatalake
        |
        v
[Silver Layer - Curated Transformations]
  SilverLayer pipeline -> DataTransformation data flow
        |
        v
[Gold Layer - Serving / Analytics]
  GoldLayer pipeline -> DataServing data flow
        |
        v
[Star Schema Outputs]
  Dimensions + Fact Booking for reporting/BI
```

---

## Core Components

### 1) Pipelines

| Pipeline | Purpose |
|---|---|
| `Parent pipeline` | Master orchestration pipeline coordinating end-to-end ETL flow across layers. |
| `API_Ingestion` | Ingests API source data into configured sink datasets (raw landing pattern). |
| `onprem_ingestion` | Ingests on-prem file data using **Self-Hosted Integration Runtime** and parameterized datasets. |
| `SQLToDatalake` | Moves relational SQL source data into Data Lake landing/bronze-style storage. |
| `SilverLayer` | Executes silver-stage transformation logic (curation/standardization). |
| `GoldLayer` | Executes gold-stage serving logic for analytical consumption. |

**Parent pipeline — Master Orchestrator**

![Parent Pipeline](./screenshots/parent-pipeline.png)

**API_Ingestion**

![API Ingestion](D:\screenshots\api_ingestion.PNG)

**onprem_ingestion**

![Onprem Ingestion](./screenshots/onprem-ingestion.png)

**SQLToDatalake**

![SQL To Datalake](./screenshots/sql-to-datalake.png)

**SilverLayer**

![Silver Layer](./screenshots/silver-layer.png)

**GoldLayer**

![Gold Layer](./screenshots/gold-layer.png)

### 2) Mapping Data Flows

| Data Flow | Purpose |
|---|---|
| `DataTransformation` | Main transformation flow for shaping curated entities and preparing analytical model inputs. |
| `DataServing` | Final serving flow for producing gold-level consumable datasets/tables. |

**DataTransformation — Data Flow Canvas**

![Data Transformation Flow](./screenshots/data-transformation-flow.png)

**DataServing — Data Flow Canvas**

![Data Serving Flow](./screenshots/data-serving-flow.png)

**DataServing — Window Transformation (Dense Rank) Settings**

![Dense Rank Window Settings](./screenshots/dense-rank-window-settings.png)

**DataServing — Data Preview (Ranked Output)**

![Data Preview Ranked Output](./screenshots/data-preview-ranked-output.png)

### 3) Trigger

| Trigger | Purpose |
|---|---|
| `Ingesttrigger` | Scheduled/event trigger to initiate ingestion/orchestration pipeline runs. |

**Ingesttrigger — Schedule Configuration**

![Ingest Trigger](./screenshots/ingest-trigger.png)

---

## Alerting (Email Notification via Logic App)

The Parent pipeline includes a **Web Activity** integrated with an **Azure Logic App**, which sends an automated email notification (via Gmail) once the pipeline run completes.

**How it works:**
- The Logic App is triggered via an **HTTP Request trigger** ("When an HTTP request is received")
- The Parent pipeline's Web Activity sends a **POST** request to the Logic App's HTTP endpoint, with a JSON body containing pipeline metadata (pipeline name, run ID, status, error message)
- The Logic App then executes a **Send Email (V2)** action, delivering the details directly to an inbox

> **Note:** This Web Activity is currently connected on the **On Success** path (for testing/demo purposes), so it sends a run-completion email rather than a failure-only alert. In a production setup, this would be moved to the **On Failure** path to alert specifically on pipeline failures.

**Logic App Flow — HTTP Trigger → Send Email**

![Logic App Flow](./screenshots/logic-app-flow.png)

---

## Star Schema

### Fully Implemented
- **`dim_airline`** — dimension built through gold-layer data flow and integrated with a business view (fact-dimension join, ranking, and top-N analysis on airline performance)
- **`fact_booking`** — fact table with airline-wise analytics (ticket cost, flight duration, check-in status, etc.)

### Source Datasets Available (Transformation Pending)
The following dimensions have their source datasets already configured and connected, with gold-layer transformation and business views planned as a future enhancement:
- `dim_airport` — source dataset: `ds_dimairport_src`
- `dim_flight` — source dataset: `ds_dimflight_src`
- `dim_passenger` — source dataset: `ds_dimpassenger_src`

This reflects an incremental build approach — sources and architecture are designed for the full star schema, with dimensions being completed and published iteratively.

---

## Tech Stack

- **Azure Data Factory (ADF)**
- **ADF Pipelines** (orchestration)
- **ADF Mapping Data Flows** (transform + serving)
- **Azure Data Lake / Storage-linked datasets**
- **Azure SQL (linked service + SQL datasets)**
- **Self-Hosted Integration Runtime** (`Zaman-SelfHosted`) for on-prem connectivity
- **Azure Logic Apps** (automated email notification via Web Activity)
- **GitHub-integrated ADF artifacts** (ADF Git-mode JSON resource structure: pipeline/, dataset/, dataflow/, linkedService/, integrationRuntime/, trigger/, factory/)

---

## Linked Services

The project includes the following linked services:

- `ls_azuresql` – Azure SQL connectivity
- `ls_datalake` – Data Lake/Blob-style storage connectivity
- `ls_github` – GitHub connectivity
- `ls_onprem_file` – On-prem file source connectivity (via self-hosted IR)

---

## Datasets

Representative datasets in this project include:

- API datasets: `ds_api_source`, `ds_api_sink`
- SQL datasets: `ds_sql`, `ds_sqlsource`
- On-prem datasets: `ds_onprem_src_param`, `ds_onpremsink_csv`
- Star schema source datasets:  
  `ds_dimairline_src`, `ds_dimairport_src`, `ds_dimflight_src`, `ds_dimpassenger_src`, `ds_factbooking_src`
- Utility/lookup datasets: `ds_lookup`, `ds_emptyjson`

---

## Setup & Prerequisites

To deploy and run this project in your environment:

1. **Azure Subscription**
   - Active subscription with permissions to create/manage ADF, storage, IR, and SQL resources.

2. **Azure Data Factory Instance**
   - Import or connect this Git-based repository to ADF Studio.

3. **Storage Account / Data Lake**
   - Configure target containers/folders for bronze/silver/gold paths.

4. **Azure SQL Database**
   - Provision SQL source/target as required and update linked service credentials.

5. **Self-Hosted Integration Runtime**
   - Install and register IR on a network that can access on-prem files.
   - Ensure `Zaman-SelfHosted` equivalent runtime is available in your ADF instance.

6. **Linked Service Parameterization / Secrets**
   - Update connection strings, server names, and credentials before execution. (No Azure Key Vault reference artifacts are currently present in this repository.)

7. **Publish & Trigger**
   - Validate pipelines/data flows, publish changes, then enable `Ingesttrigger`.

---

## Project Structure

```text
.
├── dataflow/              # Mapping Data Flows (DataTransformation, DataServing)
├── dataset/               # Dataset definitions for API/SQL/on-prem/star-schema entities
├── factory/               # ADF factory-level metadata
├── integrationRuntime/    # Self-hosted IR definition (Zaman-SelfHosted)
├── linkedService/         # External system connections (SQL, Data Lake, GitHub, on-prem)
├── pipeline/              # Orchestration and ETL pipelines (Bronze/Silver/Gold + parent)
├── trigger/               # Trigger definitions (Ingesttrigger)
├── screenshots/           # ADF Studio canvas screenshots (pipelines, data flows, Logic App)
├── publish_config.json    # Publish configuration metadata
└── README.md              # Project documentation
```

---

## Future Enhancements

- Complete gold-layer transformations and business views for **`dim_flight`**, **`dim_passenger`**, and **`dim_airport`** (source datasets already in place)
- Switch the Logic App Web Activity from **On Success** to **On Failure** path for true failure-only alerting
- Add **CI/CD release pipeline** (ARM/Bicep + environment promotion strategy)
- Introduce **parameterized environment configs** (dev/test/prod)
- Add **data quality checks** and quarantine flow for bad records
- Implement **incremental load / watermark strategy** for large fact tables
- Add **monitoring dashboards and alerting** (pipeline failure SLA visibility)
- Extend gold layer with additional business marts and KPI aggregates

---

## Author

**Shahzaman Jalil**  
GitHub: [https://github.com/Shahzaman-Jalil](https://github.com/Shahzaman-Jalil)

---

## Notes

This documentation is based on the current repository artifacts, including folders:
`dataflow/`, `dataset/`, `factory/`, `integrationRuntime/`, `linkedService/`, `pipeline/`, and `trigger/`.
