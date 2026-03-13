# Azure Data Factory Medallion Architecture ETL Pipeline

Although, this looks and seems very beginner level basic ETL project, however this project was capable enough to familiarize me so well with lot of core concepts and Datafactory as tool.

- I learnt the navigational aspects of not only Azure cloud account, but also services like Datafactory and Azure Storage.

- With this project, I made sure that I understand core behavior and functionalities of ETL on cloud, core concepts like:
- Resource groups
- Integration runtime
- Datasets
- Linked Service
- Blob containers
- Data Flow

- Progressing one step further, I also tried advnaced concepts like Medallion archirecture, transformations & aggregations in Datafactory, output file partitions.

## Pipeline Architecture

Step 1 — Source Data
A CSV file containing sample retail sales data is stored in the Raw container of Azure Blob Storage.

Step 2 — Azure Data Factory Pipeline
A pipeline is created in ADF with a Copy Activity to move data from the raw container to the processed container.

Step 3 — Linked Services
Two linked services were configured:
- Source Blob Storage
- Destination Blob Storage

Step 4 — Dataset Configuration
Datasets were created for:
- Source CSV file
- Destination CSV file

Step 5 — Data Movement
ADF Copy Activity performs the following:
- Reads CSV file
- Preserves schema
- Copies file to processed storage container

Messy CSV
   │
   ▼
Blob Storage (Bronze Layer)
   │
   ▼
Azure Data Factory
   │
   ▼
Data Cleaning Logic
   │
   ▼
Blob Storage (Silver Layer)
   │
   ▼
Aggregation Pipeline
   │
   ▼
Blob Storage (Gold Layer)

## Source Data Understanding:
- A very small & simple Sales data
- Problems intentionally included

| Problem | Example |
| --- | --- |
| Duplicate Order | order_id 1006 |
| Missing customer | order_id 1005 |

## Now, I will represent major steps of my ETL in visual screenshots:
