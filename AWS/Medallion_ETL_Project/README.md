# AWS Glue Medallion Architecture ETL Pipeline
Just like Azure Medallion project, this is doing exactly the same use-case.

## Source Data Understanding:
- S3 bucket is categorized in 3 layers, 3 respective sub-folders are created inside 1 parent bucket:

![01_s3_bucket_created](images/01_s3_bucket_created.jpg)

| Layer  | Purpose                         |
| ------ | ------------------------------- |
| Bronze | Raw data                        |
| Silver | Clean structured data           |
| Gold   | Business Analytics ready aggregated data |

![02_medallian_setup](images/02_medallian_setup.png)

## ETL Pipeline Flow:

Step 1: Raw uncleaned csv data file is uploaded in bronze layer of S3 storage.<br>
Step 2: Glue Database is created, to hold the schema and structure of this data.<br>

## Now, I will represent the ETL & steps of Transformations in visual screenshots:
