# AWS Glue Medallion Architecture ETL Pipeline
Just like Azure Medallion project, this is doing exactly the same use-case.

## Source Data Understanding:
- S3 bucket is categorized in 3 layers, 3 respective sub-folders are created inside 1 parent bucket:

**Parent Bucket**
![01_s3_bucket_created](images/01_s3_bucket_created.jpg)

| Layer  | Purpose                         |
| ------ | ------------------------------- |
| Bronze | Raw data                        |
| Silver | Clean structured data           |
| Gold   | Business Analytics ready aggregated data |

**Medallion Layer Setup**
![02_medallian_setup](images/02_medallian_setup.png)

**Raw Source file uploaded**
![03_raw_file_uploaded](images/03_raw_file_uploaded.png)

## Data Flow Architecture:

Step 1: Raw uncleaned csv data file is uploaded in bronze layer of S3 storage.<br>
Step 2: Glue Database is created, to represent the bronze, silver, gold layer data in the form of respective tables.<br>
Step 3: Crawler is created, which will read this bronze layer data, will infer the schema, and will auto create the table in Catalog Database.<br>
Step 4: IAM role is created for Glue service, to interact with S3.<br>
Step 5: Glue ETL job created, with necessary cleaning transformations, to move data from bronze to silver layer.<br>
Step 6: Again, step 3 & 5 repeated for moving data from silver to gold layer.

## Now, I will represent the ETL & steps of Transformations in visual screenshots:

**Catalog Database created**
![04_catalog_db_created](images/04_catalog_db_created.png)<br><br><br>
**Creating crawler for bronze layer**
![05_creating_bronze_crawler](images/05_creating_bronze_crawler.png)<br><br><br>
**Assigned IAM role to glue**
![06_assigned_IAMrole_for_glue](images/06_assigned_IAMrole_for_glue.png)<br><br><br>
**Destination Table**
![07_destination_table_selection](images/07_destination_table_selection.png)<br><br><br>
**Final config view of Crawler**
![08_final_view_of_config](images/08_final_view_of_config.png)<br><br><br>
**Crawler Run**
![09_crawler_run_done](images/09_crawler_run_done.png)<br><br><br>
**Table created**
![10_table_created](images/10_table_created.png)<br><br><br>
**Table schema**
![11_table_schema](images/11_table_schema.png)<br><br><br>
**Athena first launch view**
![12_athena_first_view](images/12_athena_first_view.png)<br><br><br>
**Querying Bronze layer data**
![13_athena_query_output](images/13_athena_query_output.png)<br><br><br>
**Started creating ETL**
![14_etl_flow_01](images/14_etl_flow_01.png)

![14_etl_flow_02](images/14_etl_flow_02.png)

![14_etl_flow_03](images/14_etl_flow_03.png)

![14_etl_flow_04](images/14_etl_flow_04.png)<br><br><br>

**After creating this first, basic simple ETL job, when I ran it, I faced my first error in AWS**
![14_etl_flow_05](images/14_etl_flow_05.png)

**This error was due to incomplete/improper permissions attached to Glue IAM role. I fixed this issue, by giving full read/write permission to IAM role. While finding solutions to fix this issue, I got to know that in maximum possible production scenarios, it is not recommended to give full permissions to any service related IAM role. Here I give it to just continue with demo project.**
![15_fixing_iam_issue](images/15_fixing_iam_issue.png)

![15_fixing_iam_issue_02](images/15_fixing_iam_issue_02.png)<br><br><br>
**After this fix, my ETL job ran successfully**
![16_etl_job_success](images/16_etl_job_success.png)<br><br><br>
**Silver layer output after job run**
![17_silver_layer_output](images/17_silver_layer_output.png)<br><br><br>
![18_silver_layer_crawler_table](images/18_silver_layer_crawler_table.png)<br><br>
**Query error: When I tried to view the data in Athena (this is the very first time I was using Athena), I got this error**<br>
**Initially I could not understand the exact reason of this error, however I pasted the error in chatgpt, and immediately got to know the reason**<br>
**It was caused due to double quotes used in where clause. In Athena (Presto SQL engine), double quotes are treated as column name, not string literal, so Athena thinks this is a column name, not a string value.**
**That’s why error says: Column 'cust1073' cannot be resolved**
![17_silver_layer_output](images/17_silver_layer_output.png)<br><br><br>
