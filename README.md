# Spotify ETL Project using Python and AWS

## Overview
This project demonstrates how to create an ETL (Extract, Transform, Load) pipeline using Python and various AWS services. The pipeline extracts data from the Spotify API, transforms it, and then loads it into Amazon S3. The project uses AWS Lambda, S3, Glue, and Athena to manage the data workflow. This guide outlines the steps involved in setting up and executing the pipeline.

## Objective
The main goal of this project is to develop an ETL pipeline that extracts artist, album, and song information from the "Top 50" playlist on Spotify. The pipeline will then transform this data and load it into Amazon S3 for further analysis using Amazon Athena.

## Key Learnings
- How to extract data from an API.
- Building automated triggers to run data pipelines using EventBridge.
- Writing extraction and transformation jobs using AWS Lambda.
- Proper bucket structure for data storage in S3.
- Automating transformation jobs with S3 triggers.
- Building an analytics layer for SQL queries and business intelligence using Athena.

## Architecture Diagram
![ETL Model Diagram]()

## Tools Used
- **Python**
- **AWS Services**: Lambda, S3, Glue, Athena, CloudWatch, EventBridge

## Pre-Requisites
1. **Spotify API Credentials**: Obtain client ID and client secret from the Spotify Developer Dashboard.
2. **AWS Account**: Set up necessary permissions and roles.
3. **AWS S3 Bucket**: Create a bucket with a structured folder setup:
   - `/discover_weekly`: Main folder
   - `/raw_data`: Stores raw data
     - `/to_process`: Data extracted from the API
     - `/processed`: Data after initial processing
   - `/transformed_data`: Stores transformed data
     - `/album_data`
     - `/artist_data`
     - `/song_data`

## AWS Services Overview
- **S3 (Simple Storage Service)**: Easily store and retrieve large amounts of data. Each file is called an object and data is stored in buckets.
- **Lambda**: Serverless compute service to run code without managing servers. We will use Lambda to deploy the Python code to perform data extraction and transformation.
- **CloudWatch**: Monitor and collect metrics from AWS resources. Can be used to monitor log files and set alarms.
- **EventBridge**: Create and manage events, schedule them based on a defined pattern or cron expression.
- **Crawler**: Component of AWS Glue that automatically scans and analyzes data sources to infer their schema and create metadata tables.
- **Glue Data Catalog**: Fully managed metadata repository provided by AWS Glue. It acts as a central repository for storing and organizing metadata information about various data sources, including tables, schemas, and partitions. You can use the Glue Data Catalog without the Crawler if you already have the metadata information or prefer to define and manage the metadata manually and can directly create and populate tables in the Glue Data Catalog.
- **Athena**: Interactive query service to analyze data stored in various sources using standard SQL queries. You can query data from the Glue Data Catalog, S3, and other supported data sources.

## ETL Process Overview
These are the overall steps in the process:

### 1. Extract
- Extract data from Spotify API using the Spotipy library.
- Deploy the data extraction code using the Lambda function.
- Run trigger using EventBridge to automate data extraction every Tuesday at 4 PM UTC.
- Data extract is saved in the `discover_weekly/raw_data/to_process` folder in the S3 bucket.

### 2. Transform
- Run S3 trigger when any new data is added into the `discover_weekly/raw_data/to_process` folder in the S3 bucket. This will run the data transformation code on Lambda.
- The transformation code will clean and transform the data to prepare 3 files for the album, artist, and songs. The data will be stored in the 3 subfolders in `transformed_data`. Lastly, files in the `to_process` folder will be copied to the `processed` folder and files in `to_process` will be deleted. We are just moving data from one folder to another.

### 3. Load
- Glue crawler will infer schema when new data arrives in the 3 folders in the `transformed_data` folder.
- Data catalog manages metadata repository.
- Query S3 data using Athena.

## Data Model Diagram
![ETL Model Diagram](https://github.com/SuhailAhmedd/Spotify-Data-Pipeline-using-AWS/blob/main/Data%20Model.jpg)

## Detailed Steps
### Step 1: Create a Lambda Layer
- Package and upload the Spotipy library as a layer in AWS Lambda.

### Step 2: Create `extract_data` Lambda Function
- **Environment Variables**: Add `client_id` and `client_secret`.
- **Timeout**: Set to 1 minute 30 seconds.
- **Permissions**: Ensure the role has `AmazonS3FullAccess` and `AWSLambdaRole` permissions.
- **Code**: Implement the data extraction code using Spotipy.
- **Deployment**: Deploy and test the function.

### Step 3: Add EventBridge Trigger
- Create a new rule with a cron expression to trigger the `extract_data` function every Tuesday at 4 PM UTC.

### Step 4: Create `transform_spotify_data` Lambda Function
- **Timeout**: Set to 1 minute 30 seconds.
- **Permissions**: Use the same role as `extract_data`.
- **Code**: Implement the data transformation code using Pandas.
- **Deployment**: Deploy and test the function.

### Step 5: Add S3 Trigger
- Create a trigger for the `transform_spotify_data_function` function to run when new JSON files are added to `/to_process`.

### Step 6: Create Glue Crawlers
- Repeat for each data type (album, artist, song).
- **Crawler Properties**: Set name and data source.
- **Security**: Create and attach a new IAM role.
- **Output and Scheduling**: Configure database and schedule.
- **Run Crawlers**: Execute the crawlers to populate the Glue Data Catalog.

### Step 7: Querying with Athena
- Configure query result location.
- Use Athena to query the data stored in S3.


## Conclusion
This project provides a hands-on introduction to ETL pipelines using Python and AWS services, offering experience with the Spotify API and AWS components like Lambda, S3, Glue, and Athena. Future projects can expand on this foundation to explore more complex data engineering challenges.
