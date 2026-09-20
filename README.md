# azure-elt-pipeline-with-incremental-loading
An end-to-end data engineering project built with **Azure Data Factory, Azure SQL Database, Azure Data Lake Storage Gen2, Azure Databricks, PySpark, and Delta Lake.


Azure End-to-End Incremental Data Engineering Pipeline
Project Overview

This project implements an end-to-end cloud data engineering pipeline on Microsoft Azure for processing car-sales data.

The solution combines:

Azure Data Factory
Azure SQL Database
Azure Data Lake Storage Gen2
Azure Databricks
Apache Spark / PySpark
Spark SQL
Delta Lake
Databricks Workflows

The project demonstrates:

cloud data ingestion
incremental data loading
watermark-based processing
data lake architecture
PySpark transformations
bad-record handling
derived columns
medallion architecture
dimensional modeling
surrogate keys
Delta Lake
MERGE operations
workflow dependencies
parallel Databricks execution
pipeline monitoring

<img width="1774" height="887" alt="ChatGPT Image Sep 21, 2026, 02_05_51 AM" src="https://github.com/user-attachments/assets/c673f430-3a9d-4701-ab0f-172e3dc4ff70" />

End-to-End Pipeline Flow

The overall workflow can be divided into five major stages:

1. Source ingestion
        ↓
2. Incremental ingestion using Azure Data Factory
        ↓
3. Bronze → Silver transformation using Databricks
        ↓
4. Silver → Gold dimension processing
        ↓
5. Fact-table generation and workflow orchestration




## Technology Stack

| Technology | Purpose |
|---|---|
| Azure Data Factory | Data ingestion, incremental loading, and pipeline orchestration |
| Azure SQL Database | Staging/source layer and watermark management |
| Azure Data Lake Storage Gen2 | Stores Bronze and Silver datasets |
| Azure Databricks | Data transformation and workflow orchestration |
| Apache Spark / PySpark | Cleansing, transformation, and dimensional modelling |
| Unity Catalog | Governed access to storage and Databricks tables |
| Parquet | Storage format for Bronze/Silver datasets |
| Delta Lake | Gold dimension and fact tables with MERGE/upsert support | 


Data Pipeline
1. Source Ingestion

<img width="1068" height="692" alt="Screenshot 2026-09-18 at 5 22 25 PM" src="https://github.com/user-attachments/assets/6d5bb2df-5ac1-4ea9-895f-9919f0fe5cef" />


Azure Data Factory is used to ingest source data into Azure SQL Database.

ADF Copy Activity manages the movement of the data and provides monitoring information such as rows processed, data volume, duration, and throughput.

2. Incremental Loading with Watermark

Instead of reloading the entire dataset on every run, the pipeline uses a watermark-based incremental loading pattern.

The ADF pipeline contains:

last_load Lookup – retrieves the previous watermark
sql_data_move Lookup – retrieves the latest source value
sql_to_adlsgen2 – copies the required records from Azure SQL to ADLS Gen2
water_mark Stored Procedure – updates the watermark after a successful load

Example:

Previous Watermark: DT00000
Updated Watermark:  DT01247

<img width="898" height="594" alt="Screenshot 2026-09-18 at 5 12 18 PM" src="https://github.com/user-attachments/assets/6c067f39-c706-46b9-8813-552c0ba1cfc9" />

<img width="1298" height="660" alt="Screenshot 2026-09-21 at 1 28 54 AM" src="https://github.com/user-attachments/assets/d0e9c3a6-7728-4219-98fe-20ad79158dc2" />




This allows future pipeline runs to process only newly available data.

Medallion Architecture
Bronze Layer

The Bronze layer stores the ingested source data in ADLS Gen2 with minimal modification.

Databricks accesses the files through Unity Catalog volumes.

Azure SQL
    │
    ▼
ADF Incremental Copy
    │
    ▼
ADLS Gen2 - Bronze
Silver Layer

The Bronze data is processed in Databricks using PySpark.

Main transformations include:

Creating a valid Sale_Date
Removing records containing invalid dates
Creating Model_cat from Model_ID
Calculating rev_per_unit
Performing aggregation and validation checks

Example derived fields:

Sale_Date
Model_cat
rev_per_unit = Revenue / Units_Sold

The transformed dataset is stored in the Silver layer using Parquet.

Gold Layer – Dimensional Model

The Silver dataset is transformed into an analytics-ready dimensional model.

Dimension Tables
dim_branch
dim_date
dim_dealer
dim_model

Each dimension uses a surrogate key and Delta Lake MERGE operations to support inserts and updates.

Fact Table

The fact table combines the dimension keys with the main business measures:

dim_branch_key
dim_date_key
dim_dealer_key
dim_model_key
Revenue
Units_Sold

This creates a simple star-schema structure suitable for downstream analytics.

                 dim_date
                    │
                    │
dim_branch ──── fact_table ──── dim_dealer
                    │
                    │
                dim_model
Databricks Workflow Orchestration

The complete transformation process is automated using a Databricks Job workflow.

                 ┌── dim_branch ──┐
                 ├── dim_date ────┤
Silver Layer ────┼── dim_dealer ──┼──► Fact Table
                 └── dim_model ───┘

The workflow first processes the Silver layer.

The four dimension notebooks then run in parallel, and the Fact task starts only after all dimensions complete successfully.

Pipeline Result

The complete pipeline was successfully executed across Azure Data Factory and Databricks.


<img width="1308" height="731" alt="Screenshot 2026-09-21 at 1 03 56 AM" src="https://github.com/user-attachments/assets/403c29c9-a43a-476e-a975-9c25ebdb7aec" />


Results include:

Successful source-to-Azure SQL ingestion
Successful incremental Azure SQL → ADLS Gen2 load
Watermark updated from DT00000 to DT01247
Bronze → Silver transformation completed successfully
Four Gold dimension tables created
Fact table generated using dimension surrogate keys
Databricks dependency workflow completed successfully
Key Data Engineering Concepts Demonstrated
End-to-end Azure ETL pipeline
Azure Data Factory orchestration
Incremental loading



Project Goal

The goal of this project was to build a practical end-to-end Azure data engineering solution covering cloud ingestion, incremental loading, data lake storage, PySpark transformations, medallion architecture, dimensional modelling, Delta Lake, and automated workflow orchestration.
