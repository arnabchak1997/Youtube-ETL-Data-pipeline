# Youtube-ETL-Data-pipeline
ETL pipeline on YouTube data using Athena, Glue and Lambda

Dataset Description:
The project uses Kaggle dataset containing statistics (CSV files) on daily popular YouTube videos over
the course of many months. There are up to 200 trending videos published every day
for many locations. The data for each region is in its own file. The video title, channel
title, publication time, tags, views, likes and dislikes, description, and comment count
are among the items included in the data. A category_id field, which differs by area, is
also included in the JSON file linked to the region.

Tech Stack:
Languages: SQL, Python (Pandas, PySpark)
Services: AWS S3, AWS Glue, QuickSight, AWS Lambda, AWS Athena, AWS IAM

Architecture:

![image](https://github.com/user-attachments/assets/b9ea32e6-8113-4f7e-b758-8194855c01e1)

Execution Steps:

**Creation of S3 buckets:**
For this project, we have created four S3 buckets to store our raw, clenased, analytics data and log files from AWS services used. Below are the four buckets which have been created as part of this project:

![image](https://github.com/user-attachments/assets/0e799074-7748-4a8f-9f90-e7c049287eb4)

bigdata-on-youtube-raw-apsouth1-144025787116-dev stores the raw data from Kaggle
bigdata-on-youtube-cleansed-apsouth1-144025787116-dev stores the transaformed data which will be used for analytics
bigdata-on-youtube-analytics-apsouth1-144025787116-dev stores the materialized data which are used for frequent queries from business analyst end
bigdata-on-youtube-assets-apsouth1-144025787116-dev stores the Spark job logs and temporary files created during execution

Setting up AWS CLI in local machine and transferring the raw files to S3 bucket:

Using AWS CLI, the raw csv files have been copied to bucket bigdata-on-youtube-raw-apsouth1-144025787116-dev. The csv files have been stored in Hive style partitioned way in the bucket under youtube/raw_statistics/. A seperate folder youtube/raw_statistics_reference_data/ has been created where the json files will land. 

![image](https://github.com/user-attachments/assets/5498ba94-ef50-4341-bb22-b676f6192dcc)

**Creation of Lambda job to convert json data to parquet format and copying them to cleansed bucket and Glue data catalog:**

A lambda job has been created which reads the json files from bigdata-on-youtube-raw-apsouth1-144025787116-dev/youtube/raw_statistics_reference_data/ and converts it to parquet format and writes the files to bigdata-on-youtube-cleansed-apsouth1-144025787116-dev/youtube/cleansed_statistics_reference_data/ and to database "db_youtube_cleansed" in Glue data catalog. 

The following environment variables have been passed :
glue_catalog_db_name == db_youtube_cleansed
glue_catalog_table_name == cleansed_statistics_reference_data
s3_cleansed_layer == s3://bigdata-on-youtube-cleansed-apsouth1-144025787116-dev/youtube/cleansed_statistics_reference_data/
write_data_operation == append

A trigger has been created for the lambda function which checks if any new files lands in bigdata-on-youtube-raw-apsouth1-144025787116-dev/youtube/raw_statistics_reference_data/, it will trigger the function to process the new data. 

![image](https://github.com/user-attachments/assets/9e32787b-0ee0-4f96-a475-a99cb0d27101)

**Creation of Glue crawler and Spark transformation job for csv data in raw bucket:**

A glue crawler Bigadata-on-youtube-raw-raw_statistics-glue-crawler is created which reads the csv files in bigdata-on-youtube-raw-apsouth1-144025787116-dev/youtube/raw_statistics/ and creates a table raw_statistics under database db_youtube_raw in glue catalog. Then , a spark job bigdata-on-youtube-spark-csv-to-parquet has been created which is reading the table raw_statistics and applying transformation on the data and writing them to bucket bigdata-on-youtube-cleansed-apsouth1-144025787116-dev/youtube/raw_statistics/ in parquet format. 

Another glue crawler Bigadata-on-youtube-cleansed-cleansed_statistics-glue-crawler is created which reads bucket bigdata-on-youtube-cleansed-apsouth1-144025787116-dev/youtube/raw_statistics/ and creates a table raw_statistics under database db_youtube_cleansed. 

**Joining refernce data and statistics data:**

In glue catalog, we have the refernce data under table cleansed_statistics_reference_data (created by the lambda function) and statistics data under raw_statistics (created by the glue crawler igadata-on-youtube-cleansed-cleansed_statistics-glue-crawler).

In Glue studio, we create a spark job bigdata-on-youtube-spark-materialized using Visual ETL option to join the data in tables cleansed_statistics_reference_data and raw_statistics based on condition where category_id in raw_statistics = id in cleansed_statistics_reference_data and write the output to reporting layer bucket bigdata-on-youtube-analytics-apsouth1-144025787116-dev/youtube/rpt_youtube_statistics_categories/. Also, it creates a table rpt_youtube_statistics_categories under database db_youtube_analytics and stores the output. The data in both the S3 bucket and catalog is partitioned using region and id fields

![image](https://github.com/user-attachments/assets/f7b6aefa-1ae5-48be-9740-06531ac277be)

Using Quicksight to visualize the data in 








