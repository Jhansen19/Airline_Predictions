# Big Data ELT Pipeline on Azure

**Author:** Jonathan Hansen  
**Date:** November 21, 2024  

---

## Table of Contents

1. [Project Overview](#project-overview)  
2. [Motivation](#motivation)  
3. [Architecture & Azure Services](#architecture--azure-services)  
4. [Data Pipeline Flow](#data-pipeline-flow)  
   - [Bronze Layer (Raw Ingestion)](#bronze-layer-raw-ingestion)  
   - [Silver Layer (Cleaning & Structuring)](#silver-layer-cleaning--structuring)  
   - [Gold Layer (Ready for ML)](#gold-layer-ready-for-ml)  
5. [Setup & Deployment](#setup--deployment)  
   - [Prerequisites](#prerequisites)  
   - [Resource Provisioning](#resource-provisioning)  
   - [Data Factory Configuration](#data-factory-configuration)  
6. [Security & Reliability](#security--reliability)  
7. [Results](#results)  
8. [Future Improvements](#future-improvements)  
9. [Where to find reports and files](#Where-to-find-reports-and-files) 

---

## Project Overview

This project demonstrates an end-to-end ELT (Extract–Load–Transform) pipeline built entirely in Microsoft Azure. We ingest raw CSV data from the Bureau of Transportation Statistics into Azure Data Lake Storage Gen2, clean and transform it using Azure Data Factory and Spark, and produce “silver” and “gold” datasets optimized for machine learning workloads. The implementation showcases scalability, cost-efficiency, and best practices for cloud-native big data processing.

---

## Motivation

Modern machine learning pipelines require handling massive datasets reliably and flexibly. On-premises solutions can struggle with capacity planning, maintenance overhead, and scaling costs. By leveraging Azure’s managed services, we achieve:

- **Elastic Scalability**: Automatically handle spikes in data volume  
- **Cost Efficiency**: Pay only for what you use, with geo-redundant backups  
- **High Availability**: Built-in redundancy and SLA-backed services  
- **Global Collaboration**: Team members can develop and trigger pipelines from anywhere  

---

## Architecture & Azure Services

- **Resource Group**: Logical container for all resources  
- **Azure Storage Account (Gen2)**:  
  - Hierarchical namespace enabled  
  - Geo-redundant storage (GRS) for disaster resilience  
- **Azure Data Factory (ADF)**:  
  - Orchestrates ELT workflows  
  - Hosts Integration Runtimes for on-prem and cloud connectivity  
- **Self-Hosted Integration Runtime**: Securely pulls data from local filesystems  
- **Azure Key Vault**: Centralized secrets management for connection strings and keys  
- **Azure Synapse/Spark Pool** (via ADF Data Flows): Performs distributed transformations  

---

## Data Pipeline Flow

### Bronze Layer (Raw Ingestion)
1. **Linked Service** → Connect Data Lake Gen2 (bronze container)  
2. **Dataset** → Point to raw CSV in bronze folder  
3. **Pipeline** → Copy activity ingests CSV from BTS into bronze

### Silver Layer (Cleaning & Structuring)
1. **Data Flow** in ADF:  
   - Drop irrelevant columns (119 → 66)  
   - Convert types (string → int/date)  
   - Filter out null rows where appropriate  
2. **Sink** → Write cleaned data to silver container

### Gold Layer (Ready for ML)
1. **Data Flow**:  
   - Further split and partition data for specific ML use cases  
   - Finalize schema (e.g., feature selection, normalization hooks)  
2. **Sink** → Persist feature-engineered tables in gold container

---

## Setup & Deployment

### Prerequisites

- Azure subscription with contributor permissions  
- Azure CLI installed (or access to Azure Portal)  
- Local folder or database credentials for source CSVs  

### Resource Provisioning

> **Using Azure CLI**  
> ```bash
> az login
> az group create \
>   --name BigDataRG \
>   --location eastus
> 
> az storage account create \
>   --name mydatalakergs \
>   --resource-group BigDataRG \
>   --sku Standard_GRS \
>   --kind StorageV2 \
>   --hierarchical-namespace true
> 
> az keyvault create \
>   --name BigDataKeyVault \
>   --resource-group BigDataRG \
>   --location eastus
> 
> az datafactory create \
>   --resource-group BigDataRG \
>   --name BigDataFactory \
>   --location eastus
> ```

### Data Factory Configuration

1. **Integration Runtime**  
   - In ADF “Manage” → “Integration Runtimes” → “New” → Self-Hosted  
   - Download runtime, install locally, register with Key-Vault-stored keys  

2. **Linked Services & Datasets**  
   - Bronze: Gen2 → CSV  
   - Silver & Gold: Gen2 → Parquet/Delta  

3. **Data Flows & Pipelines**  
   - Author data flows for bronze→silver and silver→gold  
   - Create pipelines that run data flows with manual or schedule triggers  

---

## Security & Reliability

- **Key Vault** stores all storage connection strings and runtime keys  
- **Geo-redundant Storage** ensures data persists across region failures  
- **Role-Based Access Control (RBAC)** on resource group level  
- **Audit Logging** via Azure Monitor for all pipeline runs and secret access  

---

## Results

- **Bronze Container**: Raw BTS CSV files (multiple GB)  
- **Silver Container**: Cleaned Parquet tables with 66 columns  
- **Gold Container**: Partitioned, ML-ready datasets  
- **Performance**:  
  - Typical bronze→silver run: ~5 minutes on 8-node Spark pool  
  - Silver→gold run: ~3 minutes with partition pruning  

---

## Future Improvements

- Fully automate pipeline triggers via Event Grid  
- Integrate Databricks for advanced transformations and notebooks  
- Implement incremental loads with watermarking  
- Add data quality checks using Azure Data Factory’s Validation activities  
- Expose gold datasets as Synapse external tables for BI consumption  

---

### Where to find reports and files
- Final project report here: https://github.com/Jhansen19/Airline_Predictions/blob/main/Final%20Project%20-%20Report.pdf

- Further Documenation Guide posted here: https://github.com/Jhansen19/Airline_Predictions/blob/main/Documentation_Guide.pdf

- Cleaned Data in 'PredictingFlightDelays' gold directory: https://github.com/Jhansen19/Airline_Predictions/blob/main/PredictingFlightDelays_part-00000-60a22a71-d029-42d7-bbfb-42544de6e9ba-c000.csv

- Cleaned data in 'RoutePerformanceAnalysis' gold directory:https://github.com/Jhansen19/Airline_Predictions/blob/main/RoutePerformanceAnalysis_part-00000-a6ab636d-a099-4f18-a0f9-d4a6eab381e1-c000%20(1).csv
