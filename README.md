# Fabric Medallion Analytics

# Project Overview

This project demonstrates an end-to-end Microsoft Fabric Data Engineering solution built using the Medallion Architecture approach.

The solution ingests data from multiple source systems including:

* CSV files
* JSON files
* GitHub raw HTTP sources

and processes them through Bronze, Silver and Gold layers inside a Microsoft Fabric Lakehouse.

The architecture is designed to simulate a scalable enterprise analytics platform using modern Lakehouse engineering principles in Microsoft Fabric.

---

# Key Features Implemented

* Metadata-driven ingestion pipeline
* Dynamic pipeline orchestration
* Incremental loading using Watermark table
* Delta Lakehouse architecture
* Spark SQL transformations
* Semantic modeling
* SQL analytics endpoint integration
* Direct Lake ready architecture
* Enterprise-style monitoring and notification activities

---

# Architecture

<p align="center">
  <img src="screenshots/architecture_diagram.gif" alt="Fabric Medallion Architecture" width="1000"/>
</p>

### Data Sources

* CRM CSV / JSON files
* ERP CSV files
* GitHub raw files using HTTP connection

### Bronze Layer

* Raw ingestion layer
* Delta table creation
* Metadata-driven ingestion

### Silver Layer

* Data cleansing
* Standardization
* Business rule implementation
* Duplicate handling

### Gold Layer

* Star schema model
* Fact and Dimension tables
* Business-ready analytical layer

### Analytics & Reporting

* Semantic Model
* SQL Analytics Endpoint
* Direct Lake ready architecture
* Power BI reporting capability

---

# Repository Structure

```text
fabric-medallion-analytics/
│
├── data_source/
├── notebooks/
├── pipelines/
├── screenshots/
└── README.md
```

---

# Project Setup & Execution

## 1. Create Lakehouse

Create a Lakehouse with the following name:

```text
sales_lakehouse
```

---

## 2. Upload Source Files

Upload the CSV and JSON source files into:

```text
Lakehouse → Files
```

### Steps

1. Open:

```text
sales_lakehouse
```

2. Navigate to:

```text
Files
```

3. Create the following folders if they do not exist:

```text
crm
erp
```

4. Upload the corresponding CSV and JSON files into the appropriate folders.

### Expected Folder Structure

```text
Files/
│
├── crm/
│   ├── sales_details.csv
│   └── prd_info.json
│
└── erp/
    ├── CUST_AZ12.csv
    ├── LOC_A101.csv
    └── PX_CAT_G1V2.csv
```

### Important

Ensure:

* File names match the values inside `config_ingestion`(table we are about to create)
* Folder structure matches configured source paths
* CSV and JSON files are uploaded to the correct folders
* The current implementation ingests `cust_info.csv` dynamically from a GitHub raw HTTP source

These files are dynamically ingested by the metadata-driven pipeline to create Bronze Delta tables.

---

## GitHub Raw URL Configuration

For GitHub ingestion:

* Open the file in GitHub
* Click:

```text
Raw
```

* Copy the generated URL

For the HTTP connection, use:

```text
https://raw.githubusercontent.com/
```

The remaining dynamic path is maintained inside the `config_ingestion` table.

This allows the pipeline to dynamically ingest GitHub files.

---

## 3. Create Config Table

Run notebook:

```text
00_NB_CONFIG_INGESTION
```

This notebook creates and populates the metadata-driven configuration table used by the ingestion pipeline.

---

# Notebook Configuration

Before running notebooks:

Select:

```text
Spark SQL
```

as the notebook language.

Although:

```text
%%sql
```

is already used inside notebooks, manually selecting Spark SQL helps avoid execution inconsistencies.

---

# Default Lakehouse Configuration

After importing notebooks, you may encounter errors like:
"Default Lakehouse is not accessible" or "Couldn't load artifact"

1. Remove the old Lakehouse reference
2. Click **Add Data Items**
3. From OneLake catalog
4. Select the correct Lakehouse
5. Click **... → Set as Default Lakehouse**
6. Restart the Spark session

### Why this is needed

The notebook retains a reference to an old Lakehouse ID, even if a Lakehouse with the same name exists. Fabric notebooks bind to Lakehouse ID, not name.

Setting the correct default Lakehouse ensures:

* Delta tables are accessible
* SQL objects resolve correctly
* Notebook execution uses the correct Lakehouse context

---

## 4. Create Watermark Table

Run notebook:

```text
01_NB_WATERMARK_TABLE_SETUP
```

This creates the watermark table used for incremental loading.

---

# Bronze Layer Setup

## 5. Import Pipeline

### Important

Before importing the pipeline, create a **Workspace Identity** for the workspace.

This identity is used by Microsoft Fabric to authenticate and securely connect pipeline activities such as Notebook execution, Semantic Model refresh, and other connected resources.

### Steps to create Workspace Identity

Navigate to:

```text
Workspace Settings → Workspace Identity
```

Click:

```text
Create Workspace Identity
```

### Why this is required

If Workspace Identity is not created before importing the pipeline, notebook activities may fail with errors similar to:

```text
AuthKind WorkspaceIdentity did not have accessToken specified
```

---

## Import Pipeline

Create a new pipeline and import the following pipeline `.zip` file.

### Important

Do NOT extract the `.zip` file before importing. Rename the pipeline if required.

```text
PL_INGEST_TRANSFORM_BRONZE_TO_GOLD
```

---

## Configure Pipeline Connections

During pipeline import, configure the following connections:

#### 1. GitHub HTTP Connection

* Connection Type: `HTTP`
* Base URL:

```text
https://raw.githubusercontent.com/
```

#### 2. Lakehouse Connection

Select the following Lakehouse:

```text
sales_lakehouse
```

#### 3. Notebook Connection

Configure the notebook connection using:

* Connection Type: `Notebook`
* Authentication Kind: `Workspace Identity`
* Privacy Level: `None`

#### 4. Semantic Model Connection

Configure using:

* Connection Type: `Power BI Semantic Model`
* Authentication Kind: `Organizational Account`
* Sign in if prompted
* Privacy Level: `None`

#### 5. Office 365 Email Connection

Configure using:

* Connection Type: `Office 365 Outlook`
* Authentication Kind: `Organizational Account`
* Sign in if prompted
* Privacy Level: `None`


---

## 6. Disable Incremental Activities Initially

Before the first pipeline execution:

Make sure the following activities are activated. If not, Activate.

* Lookup
* Foreach 

Deactivate the following activities temporarily:

* Incremental notebook activities
* Semantic Model Refresh activity
* Office 365 Email activity

### How to deactivate

* Right-click the activity
* Select:

```text
Deactivate
```

This ensures only Bronze ingestion is executed during the initial setup.

---

## 7. Run Pipeline

Validate the pipeline before running it. Rename the pipeline if required.
Run the pipeline manually.

Once completed successfully:

* 6 Bronze Delta tables will be created

### Pipeline Orchestration Design

A `ForEach` activity was used to dynamically iterate through all source configurations stored in the `config_ingestion` metadata table. This approach enables scalable, metadata-driven ingestion without creating separate pipelines for each source file.

`If` activities were used inside the `ForEach` loop to route different source types and file formats (CSV, JSON, GitHub HTTP sources) to the appropriate ingestion logic dynamically during execution.

For the current project scope, `If` conditions were intentionally chosen over alternatives such as `Switch` activities to keep the orchestration logic simple, readable and easier to debug while handling a limited number of ingestion scenarios.

---

## 8. Validate Bronze Layer

Run notebook:

```text
02_NB_BRONZE_DATA_QUALITY
```

Validation checks include:

* Row counts
* Null checks
* Duplicate checks
* Bronze data quality validation

---

# Silver Layer Setup

## 9. Run Silver Full Load

Run notebook:

```text
03_NB_SILVER_FULL_LOAD
```

Once completed successfully:

* 6 Silver tables will be created

---

## 10. Validate Silver Layer

Run notebook:

```text
04_NB_SILVER_DATA_QUALITY
```

Validation checks include:

* Data cleansing validation
* Standardization validation
* Business rule validation
* Silver layer quality checks

---

# Gold Layer Setup

## 11. Run Gold Full Load

Run notebook:

```text
06_NB_GOLD_FULL_LOAD
```

Once completed successfully:

* 2 Dimension tables will be created
* 1 Fact table will be created

---

## 12. Validate Gold Layer

Run notebook:

```text
07_NB_GOLD_DATA_QUALITY
```

Validation checks include:

* Fact table validation
* Dimension relationship validation
* Surrogate key mapping validation
* Gold model validation

---

# 13. Create Semantic Model

Inside:

```text
sales_lakehouse
```

click:

```text
New Semantic Model
```

Select the following tables:

* `gold_fact_sales`
* `gold_dim_customers`
* `gold_dim_products`

Create the relationships to form a proper Star Schema model.

This ensures:

* Direct Lake connectivity
* Semantic layer creation
* BI consumption readiness
* SQL analytics compatibility

---

# Incremental Load Testing

## 14. Simulate New Source Data

To test incremental processing, replace the original source file with the incremental test file.

Example:

```text
sales_details_incremental.csv
```

Rename the file to:

```text
sales_details.csv
```

and upload it to the same Lakehouse Files location, replacing the existing file.

The incremental test file contains all existing records along with an additional record:

```text
999999,P001,12345,20260425,20250105,20250110,1000,2,500
```

This simulates newly arrived source data before executing the pipeline again.

> **Note:** Use a future `order_dt` value when creating incremental test records to ensure they are detected by the watermark-based incremental logic.
---

## 15. Validate Existing Row Counts

Run notebook:

```text
09_NB_INCREMENTAL_LOAD_VALIDATION
```

This captures row counts before incremental execution.

---

## 16. Enable Incremental Activities

Before running the pipeline for incremental loading:

Make sure all activities are activated again, including:

* Incremental notebook activities

Configure the notebooks:
  * NB_Incremental_Load_Bronze_To_Silver -> select the right workspace -> select the notebook `05_NB_SILVER_INCREMENTAL_LOAD`
  * NB_Incremental_Load_Silver_To_Gold -> select the right workspace -> select the notebook `08_NB_GOLD_INCREMENTAL_LOAD`

The pipeline also includes:

* Semantic Model Refresh activity
* Office 365 Email notification activity

### Semantic Model Refresh

Configure the refresh activity using the Semantic Model created from:

* `gold_fact_sales`
* `gold_dim_customers`
* `gold_dim_products`

### Office 365 Email Activity

Configure:

* Recipient email addresses
* Failure dependency conditions

This simulates enterprise-style orchestration and monitoring

---

## 17. Verify Copy Activity Table Action

Inside Copy Activity:

Ensure table action is set to:

```text
Overwrite
```

instead of:

```text
Append
```

### Why Overwrite is used

The source files (CSV, JSON, and GitHub files) are treated as full snapshots and are reprocessed during each pipeline execution.

Using:

```text
Overwrite
```

ensures that the Bronze layer always contains the latest raw version of the source data and prevents duplicate records from accumulating across multiple pipeline runs.

Incremental processing is implemented in the Silver layer using watermark-based logic, where only newly arrived records are propagated to downstream layers.

Using:

```text
Append
```

would reload the entire source file during every execution and create duplicate records in the Bronze tables.

---

## 18. Run Pipeline Again

Run the pipeline manually once again.

This time:

* Newly added records should load incrementally
* Watermark logic should prevent duplicate processing
* Silver and Gold layers should process only new records
* Semantic Model Refresh activity should refresh the semantic model automatically
* Email Activity sends out an email to the recipient if the pipeline fails 

---

## 19. Validate Incremental Load

Run notebook:

```text
10_NB_INCREMENTAL_LOAD_VALIDATION
```

Validation checks include:

* Incremental row movement
* Watermark updates
* Bronze → Silver → Gold propagation
* Successful incremental processing

---

# Challenges Solved

* Dynamic ingestion from multiple source types
* GitHub HTTP ingestion using raw URLs
* Metadata-driven ETL implementation
* Incremental processing using watermark tables
* CSV and JSON ingestion handling
* Delta table processing in Fabric
* Dynamic pipeline orchestration
* Medallion architecture implementation

---

# Screenshots

The repository includes screenshots for:

* Pipeline execution
* Incremental loading
* Notebook execution
* Bronze, Silver, and Gold tables
* Semantic Model configuration
* Architecture diagram

---

# About This Project

This project was built to demonstrate practical Microsoft Fabric Data Engineering implementation using modern Lakehouse architecture principles.

The implementation focuses on:

* Dynamic orchestration
* Metadata-driven ingestion
* Incremental processing
* Scalable Lakehouse architecture
* Enterprise-style pipeline design

---

# Author

**Senthamilarasu V M**

BI Lead | Power BI Developer | Microsoft Fabric Developer
