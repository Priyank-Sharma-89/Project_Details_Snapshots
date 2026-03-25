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


**Data Issues which we will fix in Silver layer**

**1. Mixed date formates**
| Order_Date  |
| ------ |
| 2024-01-05 |
| 05-01-2024 |
| 2024/01/05 |
| (blank) |

I will standardize them to one format: yyyy-mm-dd and also handle blank dates

**2. NULL Values**

Some rows have:
- product = NULL
- quantity = NULL

Task: remove or impute

**3. BLANK Values**

Some rows have:
- city = ""
- order_date = ""

Task: Treat blank as NULL, clean or filter

**4. Data Type Issues**
- quantity may be null → needs casting
- order_date string → convert to date

Task: Treat blank as NULL, clean or filter


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
![04_catalog_db_created](images/04_catalog_db_created.png)


**Creating crawler for bronze layer**
![05_creating_bronze_crawler](images/05_creating_bronze_crawler.png)


**Assigned IAM role to glue**
![06_assigned_IAMrole_for_glue](images/06_assigned_IAMrole_for_glue.png)


**Destination Table**
![07_destination_table_selection](images/07_destination_table_selection.png)


**Final config view of Crawler**
![08_final_view_of_config](images/08_final_view_of_config.png)


**Crawler Run**
![09_crawler_run_done](images/09_crawler_run_done.png)


**Table created**
![10_table_created](images/10_table_created.png)


**Table schema**
![11_table_schema](images/11_table_schema.png)


**Athena first launch view**
![12_athena_first_view](images/12_athena_first_view.png)


**Querying Bronze layer data**
![13_athena_query_output](images/13_athena_query_output.png)


**Started creating ETL**
![14_etl_flow_01](images/14_etl_flow_01.png)

![14_etl_flow_02](images/14_etl_flow_02.png)

![14_etl_flow_03](images/14_etl_flow_03.png)

![14_etl_flow_04](images/14_etl_flow_04.png)



**After creating this first, basic simple ETL job, when I ran it, I faced my first error in AWS**
![14_etl_flow_05](images/14_etl_flow_05.png)


**This error was due to incomplete/improper permissions attached to Glue IAM role. I fixed this issue, by giving full read/write permission to IAM role. While finding solutions to fix this issue, I got to know that in maximum possible production scenarios, it is not recommended to give full permissions to any service related IAM role. Here I give it to just continue with demo project.**
![15_fixing_iam_issue](images/15_fixing_iam_issue.png)

![15_fixing_iam_issue_02](images/15_fixing_iam_issue_02.png)



**After this fix, my ETL job ran successfully**
![16_etl_job_success](images/16_etl_job_success.png)


**Silver layer output after job run**
![17_silver_layer_output](images/17_silver_layer_output.png)

![18_silver_layer_crawler_table](images/18_silver_layer_crawler_table.png)


**Query error: When I tried to view the data in Athena (this is the very first time I was using Athena), I got this error**

**Initially I could not understand the exact reason of this error, however I pasted the error in chatgpt, and immediately got to know the reason**

**It was caused due to double quotes used in where clause. In Athena (Presto SQL engine), double quotes are treated as column name, not string literal, so Athena thinks this is a column name, not a string value.**

**That’s why error says: Column 'cust1073' cannot be resolved. Fixed the issue by using single quotes: WHERE customer_id = 'CUST1073'**
![19_query_error](images/19_query_error.png)

**After getting familiar with navigation, options & UI, I started recreating ETL, with proper actual required data cleaning transformations**

**In doing so, one more issue was waiting for me. When I tried to use 'Filter' transformation from Glue transformation tab options, I could not use exactly how I wanted to filter the data**
![20_filter_transformation_issue](images/20_filter_transformation_issue.png)

**I was trying to apply this condition: product IS NOT NULL AND product != '' AND quantity IS NOT NULL AND order_date IS NOT NULL AND order_date != '',but as you can see in screenshot, under 'Operator' filed, I was not getting option to put "!="**

**After doing google what can be done, I got fix that I can use SQL query option from transformation menu**
![21_etl_changes_01](images/21_etl_changes_01.png)


**This worked, and you can see, I verified the result by checking row count of both bronze and silver table. You can see the lesser row count in silver table, means our filter transformation worked**

Bronze Table count
![21_etl_changes_02](images/21_etl_changes_02.png)

Silver Table count
![21_etl_changes_03](images/21_etl_changes_03.png)


**Apart from row count, I verified further by analyzing one record, if filter worked properly**

CUST1003 data in bronze table
![](images/21_etl_changes_04.png)

CUST1003 data in silver table, notice 1 row with blank value in product column is filtered out
![](images/21_etl_changes_05.png)


**Now, rest of the transformation in ETL**
![](images/21_etl_changes_06.png)

![](images/21_etl_changes_07.png)

![](images/21_etl_changes_08.png)


**Now, here I faced one more error, when I run this new recreated ETL job**
![](images/21_etl_changes_09.png)


**On debug, I got to know this is due to one sql built-in function, which is not compatible with sql transformation editor in glue**
![](images/21_etl_changes_10.png)

Moving further
![](images/21_etl_changes_11.png)

Gold layer data aggregation job
![](images/21_etl_changes_12.png)


Gold layer aggregated Parquet output
![](images/21_etl_changes_13.png)


Gold layer table created in Database via Catalog
![](images/22_gold_layer_table.png)


So, as you can observer, this AWS project, gave me more learning apart from usual ETL pipeline creation. I was thinking that it will be as easy or smooth as Azure, or atleast will be a similar experience, but AWS taught me some things really well by throwing notorious error here and there. And honestly, I really felt happy, because I learnt how to tackle errors, how I can handle them, how to fix them etc.

**Moving one step further after creating ETL, I also tried to find answers to below 2 doubts (honestly speaking I aksed my doubts to chatgpt, to get answers)**

**Even though my data is so miniscule and small in kbs, still my ETL job took 1 min 24 secs to finish. Why so much time in such a small dataset?**
What I got to know, The execution time is dominated by cluster initialization overhead in AWS Glue, not the data processing itself. For large-scale data, this overhead becomes negligible compared to actual processing time.
**What's really happening**
In Glue, when we run a job,<br>
- Step 1 → Spin up Spark cluster (30–60 sec)
- Step 2 → Load libraries & environment
- Step 3 → Read data
- Step 4 → Process data (your SQL)
- Step 5 → Write output
- Step 6 → Tear down cluster<br>

For my tiny dataset;<br>
- Processing time = maybe 2–3 seconds
- Glue overhead = ~60–80 seconds
