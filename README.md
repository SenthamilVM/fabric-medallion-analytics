# Fabric Medallion Analytics

## Project Overview

This project demonstrates an end-to-end Medallion Architecture implementation in Microsoft Fabric using dynamic pipelines, Lakehouse architecture, incremental loading, and dimensional modeling.

The objective of this project was to simulate a real-world enterprise data engineering workflow where raw CRM and ERP data is ingested, transformed, validated, and modeled into business-ready analytical tables.

The project was designed to focus on:

* Microsoft Fabric Lakehouse implementation
* Metadata-driven ingestion pipelines
* Incremental loading using watermark tables
* Bronze → Silver → Gold transformation architecture
* Dynamic orchestration using Fabric Pipelines
* Multi-source ingestion (Local + GitHub)
* Delta table processing
* Business-ready dimensional modeling

---

# Architecture Used

The project follows the Medallion Architecture pattern:

```text
Source Systems
      ↓
Bronze Layer (Raw Data)
      ↓
Silver Layer (Cleaned & Standardized Data)
      ↓
Gold Layer (Business Model)
      ↓
Power BI Semantic Model
```

---

# Project Flow

## 1. Source Data Ingestion

Source files were collected from:

* Local Lakehouse Files
* GitHub Raw Files

Both CSV and JSON formats were used.

A metadata-driven configuration table controls:

* Source type
* File path
* File format
* Target table
* Load type

This allows new files to be added without redesigning the pipeline.

---

## 2. Bronze Layer

The Bronze layer stores raw ingested data as Delta tables.

### Objectives

* Preserve raw source data
* Standardize ingestion
* Support replay/reprocessing
* Enable downstream transformations

### Features Implemented

* Dynamic table loading
* Dynamic path handling
* GitHub HTTP ingestion
* CSV and JSON ingestion
* Config-driven orchestration
* Delta table creation

---

## 3. Silver Layer

The Silver layer applies data cleansing and transformation rules.

### Transformations Performed

* Data type standardization
* Invalid date handling
* NULL handling
* Sales amount correction
* Price validation
* Metadata column creation

### Incremental Loading

Incremental loading was implemented using:

* Watermark control table
* Last loaded date tracking
* Incremental filtering logic

Only newly arrived records are processed during each execution.

---

## 4. Gold Layer

The Gold layer contains business-ready analytical tables.

### Data Modeling

A dimensional model was implemented using:

* Fact table
* Dimension tables
* Surrogate keys
* Star schema concepts

### Gold Layer Features

* MERGE-based incremental updates
* Customer dimension
* Product dimension
* Sales fact table
* Business-ready schema

---

# Pipeline Design

A metadata-driven Fabric Pipeline was built to orchestrate ingestion.

## Pipeline Workflow

```text
Lookup Config
    ↓
ForEach Loop
    ↓
Conditional Routing
    ├── Local CSV
    ├── Local JSON
    └── GitHub CSV
    ↓
Bronze Loading
    ↓
Silver Incremental Processing
    ↓
Gold Incremental Processing
```

---

# Dynamic Ingestion Logic

The pipeline dynamically determines:

* Which source to read
* Which format to process
* Which target table to load

This was implemented using:

* Lookup Activity
* ForEach Loop
* IF Conditions
* Dynamic Expressions
* Copy Activities

---

# Incremental Load Strategy

The project implements incremental loading using watermark-based processing.

## Workflow

1. Read last loaded value
2. Identify new records
3. Load only incremental data
4. Update watermark table

This reduces unnecessary full refreshes and improves scalability.

---

# Validation & Monitoring

Validation notebooks were added to:

* Verify incremental rows
* Validate Bronze/Silver/Gold counts
* Check successful data movement
* Validate watermark updates

Pipeline monitoring was also used to:

* Validate activity execution
* Verify successful ingestion
* Monitor Bronze to Silver and Silver to Gold execution

---

# Repository Structure

```text
fabric-medallion-analytics/
│
├── notebooks/
├── pipelines/
├── data_source/
├── screenshots/
├── sql/
└── README.md
```

---

# Project Setup & Execution Steps

## 1. Create Lakehouse

Create a Lakehouse in Microsoft Fabric with the following name:

```text
sales_lakehouse
```

This project uses the Lakehouse name directly inside notebooks and SQL scripts.

---

## 2. Create Configuration Table

Run the notebook:

```text
00_NB_CONFIG_INGESTION
```

This notebook creates and populates the metadata-driven configuration table used by the ingestion pipeline.

### Important

Update the GitHub raw file path inside the notebook before running it.

---

## 3. Create Watermark Table

Run the notebook:

```text
01_NB_WATERMARK_TABLE_SETUP
```

This creates the watermark control table used for incremental loading.

---

## 4. Upload Source Files

Upload the source CSV and JSON files into the:

```text
Lakehouse → Files
```

section.

### Important

Make sure:

* File names match the values inside the config_ingestion table
* Folder structure matches the configured source paths

This is required because the pipeline dynamically reads files using metadata-driven paths.

---

# Bronze Layer Setup

## 5. Import Pipeline

Import the provided pipeline (.zip file) -> Create a Pipeline item and click 'Import' from the top menu and select the .zip file

```text
PL_INGEST_TRANSFORM_BRONZE_TO_GOLD.zip
```
This can be created manually based on the screenshots provided in the <screenshots folder>

---

## 6. Disable Incremental Notebooks Temporarily in Pipeline 

```text
PL_INGEST_TRANSFORM_BRONZE_TO_GOLD
```

Before the initial run:

* Deactivate incremental load notebook activities inside the pipeline (right click on notebook and select Deactivate).

This ensures only Bronze tables are created during the first execution.

---

## 7. Run Pipeline

Run the pipeline manually. The Pipeline is parametrized based on the config_ingestion file.

```text
PL_INGEST_TRANSFORM_BRONZE_TO_GOLD
```

Once completed successfully:

6 Bronze Delta tables will be created (The number of tables depends entirely on the number of files you are loading. Here, I'm uploading 6 data files)
* bronze_crm_customer_info
* bronze_crm_product_info
* bronze_crm_sales_details
* bronze_erp_customers
* bronze_erp_location
* bronze_erp_product_category 

---

# Important Notebook Configuration

Before running any notebook:

## Ensure Spark SQL is selected as the default language

Although:

```text
%%sql
```

is already used at the top of notebooks, manually selecting:

```text
Spark SQL
```

as the notebook language helps avoid execution inconsistencies.

---

## 8. Validate Bronze Layer

Run the notebook:

```text
02_NB_BRONZE_DATA_QUALITY
```

This validates:

* Row counts
* Null values
* Duplicate checks
* General Bronze layer quality

---

## Default Lakehouse Configuration

After importing notebooks, you may encounter errors like:
"Default Lakehouse is not accessible"

### Cause
The notebook retains a reference to an old Lakehouse ID, even if a Lakehouse with the same name exists.

### Fix
1. Click **Add Data**
2. Select the correct Lakehouse
3. Click **... → Set as Default Lakehouse**
4. Remove the old Lakehouse reference
5. Restart the Spark session

### Note
Fabric notebooks bind to Lakehouse ID, not name.

---

# Silver Layer Setup (Initial Full Load)

## 9. Run Silver Full Load Notebook

Run the notebook:

```text
03_NB_SILVER_FULL_LOAD
```

This notebook performs the initial Silver layer creation.

Once completed successfully:

6 Silver tables will be created
* silver_crm_customer_info
* silver_crm_product_info
* silver_crm_sales_details
* silver_erp_customers
* silver_erp_location
* silver_erp_product_category

---

## 10. Validate Silver Layer

Run the notebook:

```text
04_NB_SILVER_DATA_QUALITY
```

This validates:

* Cleansed data
* Transformation quality
* Data standardization
* Business rule implementation

---

# Gold Layer Setup (Initial Full Load)

## 11. Run Gold Full Load Notebook

Run the notebook:

```text
06_NB_GOLD_FULL_LOAD
```

This creates the Gold business model.

Once completed successfully:

* 2 Dimension tables will be created
* 1 Fact table will be created

---

## 12. Validate Gold Layer

Run the notebook:

```text
07_NB_GOLD_DATA_QUALITY
```

This validates:

* Fact table integrity
* Dimension relationships
* Surrogate key mappings
* Gold layer business model quality

---

# Incremental Load Testing

Once all tables are created successfully, incremental loading can be tested.

---

## 13. Insert New Test Records

Run the notebook:

```text
09_NB_INCREMENTAL_TEST_DATA_LOAD
```

This notebook manually inserts additional records into the source tables to simulate newly arrived data.

---

## 14. Enable Incremental Notebooks

Re-enable the incremental notebook activities inside the pipeline.

---

## 15. Update Copy Activity Table Action

For incremental testing:

Set:

```text
Append
```

inside Copy Activity table actions.

### Why Append is required ?

Incremental loading should preserve existing Bronze records and add only newly arrived records.

Using:

```text
Overwrite
```

would replace the existing Bronze data during every execution.

### Recommended Approach

* Use Overwrite only during initial testing/setup
* Use Append for incremental processing

---

## 16. Run Pipeline Again

Run the pipeline once again.

```text
PL_INGEST_TRANSFORM_BRONZE_TO_GOLD
```

This time:

* Only newly added records should flow into Silver and Gold layers
* Watermark logic should prevent duplicate processing

---

## 17. Validate Incremental Load

Run the notebook:

```text
10_NB_INCREMENTAL_LOAD_VALIDATION
```

This validates:

* Incremental row movement
* Watermark updates
* Bronze → Silver → Gold propagation
* Successful incremental processing

---

# Key Features Demonstrated

* Microsoft Fabric Lakehouse
* Medallion Architecture
* Metadata-driven pipelines
* Dynamic ingestion
* Incremental loading
* Delta Lake processing
* MERGE operations
* GitHub HTTP ingestion
* JSON ingestion
* Fabric orchestration
* Dimensional modeling
* Data quality transformations

---

# Challenges Solved

During development, several real-world engineering challenges were handled:

* Dynamic file routing
* GitHub raw file ingestion
* JSON and CSV handling
* Incremental watermark logic
* Delta MERGE conflicts
* Pipeline conditional execution
* Dynamic path generation
* Notebook orchestration
* Lakehouse connectivity issues

---

# Screenshots

## Pipeline Orchestration

(Add screenshots here)

## Bronze / Silver / Gold Architecture

(Add screenshots here)

## Incremental Loading

(Add screenshots here)

## Validation Outputs

(Add screenshots here)

---

# About This Project

This project was built as part of a Microsoft Fabric Data Engineering portfolio to demonstrate practical implementation of enterprise-style data engineering concepts using Fabric Lakehouse, Spark SQL, and dynamic pipelines.

---

# Author

Senthamilarasu V M

BI Lead | Power BI Developer | Microsoft Fabric Enthusiast
