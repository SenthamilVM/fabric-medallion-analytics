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

Senthamil VM

BI Lead | Power BI Developer | Microsoft Fabric Enthusiast
