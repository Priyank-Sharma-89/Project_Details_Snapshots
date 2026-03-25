# AWS Glue Medallion Architecture ETL Pipeline


## Source Data Understanding:
- S3 bucket is categorized in 3 layers, 3 respective sub-folders are created inside 1 parent bucket:

![01_s3_bucket_created](images/01_s3_bucket_created.jpg)

| Layer  | Purpose                         |
| ------ | ------------------------------- |
| Bronze | Raw data                        |
| Silver | Clean structured data           |
| Gold   | Business Analytics ready aggregated data |

![02_medallian_setup](images/02_medallian_setup.png)

## Now, I will represent the ETL & steps of Transformations in visual screenshots:
