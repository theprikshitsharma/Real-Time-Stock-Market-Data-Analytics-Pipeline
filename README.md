# Stock Market Real Time Data Analytics Pipeline on AWS

### Overview of Project ☁️
This project builds a real-time stock market data analytics pipeline using AWS, leveraging event-driven architecture and serverless technologies. The architecture ingests, processes, stores, and analyzes stock market data in real-time while minimizing costs. .

### Key tasks include:
1. Streaming real-time stock data from sources like yfinance using Amazon Kinesis Data Streams.
2. Processing data and detecting anomalies with AWS Lambda.
3. Storing processed stock data in Amazon DynamoDB for low-latency querying.
4. Storing raw stock data in Amazon S3 for long-term analytics.
5. Querying historical data using Amazon Athena.
6. Sending real-time stock trend alerts using AWS Lambda & Amazon SNS (Email/SMS).

### What I Learned:
1. Designing event-driven systems using AWS managed services
2. Building real-time ingestion pipelines with Amazon Kinesis
3. Implementing reactive processing using DynamoDB Streams
4. Separating operational and analytical data workloads
5. Managing schemas manually using AWS Glue Data Catalog
6. Running serverless analytics using Amazon Athena
7. Applying IAM best practices for secure cloud systems

### [Project Walkthrough & Live Demo](https://www.youtube.com/watch?v=9U20mFqFRDA)

### Project Architecture:
<img width="1538" height="750" alt="image" src="https://github.com/user-attachments/assets/ec477d89-a2ee-4bab-8a5e-602d2cd81de4" />

## Steps: 
### Step 1: Launch an EC2 Instance and Run the Containerized Stock Producer

This step sets up the data ingestion layer of the system: a Dockerized Python application running on EC2 that streams live stock data into Amazon Kinesis.

1. Create an IAM Role for the EC2 Instance

Go to IAM → Roles → Create role

Select AWS service → EC2

Attach the policy:

```sh
AmazonKinesisFullAccess
```

Name the role: ```ec2-kinesis-producer-role```

2. Launch the EC2 Instance

Recommended configuration:

```sh
AMI: Amazon Linux 2023
Instance type: t3.micro
IAM role: ec2-kinesis-producer-role
Security group:
    Inbound: none required
    Outbound: allow all (required for stock API + AWS APIs)
```

Launch the instance and connect via SSH:

```ssh ec2-user@<EC2_PUBLIC_IP>```

3. Install Docker on EC2

```sh
sudo dnf install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```

Add the EC2 user to the Docker group:

```sh
sudo usermod -aG docker ec2-user
exit
```

Reconnect to apply group changes:
```ssh ec2-user@<EC2_PUBLIC_IP>```


Verify Docker:

```docker --version```

4. Prepare the Application on EC2

Clone the project containing the Dockerized producer.

```sh 
git clone https://github.com/theprikshitsharma/Real-Time-Stock-Market-Data-Analytics-Pipeline
cd kinesis-producer
```

The directory should contain:

```sh
stream_stock_data.py
Dockerfile
requirements.txt
```

5. Build the Docker Image

```docker build -t kinesis-producer .```

6. Run the Container

```sh
    docker run -d \
  --name kinesis-producer \
  --restart unless-stopped \
  kinesis-producer
  ```

### Step 2: Create the Amazon Kinesis Data Stream

This step sets up the real-time ingestion backbone that receives stock events from the EC2-hosted producer and makes them available to downstream consumers (Lambda).

Navigate to:
AWS Console → Amazon Kinesis → Data Streams → Create data stream

Basic settings:
```sh
Data stream name:
    stock-market-stream
Capacity mode:
    On-demand
Retention period:
    24 hours (default)
```

Click Create data stream. The stream will become ACTIVE in a few seconds.

### Step 3: Create a Lambda Consumer for the Kinesis Stream

This step sets up real-time processing of stock events flowing through Kinesis and persists structured data into DynamoDB.

1. Lambda needs permission to:
    Read from Kinesis
    Write logs
    Write to DynamoDB

Go to IAM → Roles → Create role
```sh 
Trusted entity: 
    AWS service
Use case:
    Lambda
```

Attach policies:
```sh
AmazonKinesisFullAccess
AmazonDynamoDBFullAccess
AWSLambdaBasicExecutionRole
AmazonS3FullAccess
AmazonSNSFullAccess
```

Name the role:

```StockMarketLambdaRole```

2. Create the Lambda Function

Navigate to:
AWS Console → Lambda → Create function

Basic settings:

```sh
Function name:
    kinesis-stock-processor
Runtime:
    Python 3.11 (or latest)
Execution role:
    StockMarketLambdaRole
```

Create the function.

3. Configure Kinesis as the Trigger

Inside the Lambda function:

Go to Triggers → Add trigger

Select Kinesis

Choose the stream:
```stock-market-stream```

Start position:
Latest

Batch size:
2 

Enable trigger

4. copy the content of python script from https://github.com/theprikshitsharma/Real-Time-Stock-Market-Data-Analytics-Pipeline/kinesis-stock-processor/lambda_function.py


### Step 4: Create DynamoDB Table (Operational Storage)

DynamoDB stores processed, structured stock data for low-latency access and downstream triggers.

1. Create the DynamoDB Table

Go to:
AWS Console → DynamoDB → Tables → Create table

Table configuration:
```sh
Table name
    stock-market-table
Partition key
    symbol (String)
Sort key
    timestamp (String)
```

2. Table Settings

```sh
Table class:
    Standard
Capacity mode:
    On-demand
Encryption:
    Default (AWS-managed key)
```

3. Enable DynamoDB Streams 

In the table settings:
Go to Exports and streams
Enable DynamoDB Streams

```sh
View type:
    New image
```
What DynamoDB Is Used For

At this point DynamoDB:

Holds the latest processed stock data

Supports trend analysis queries

Acts as the trigger source for alerts via Streams


### Step 5: Create an S3 Bucket (Data Lake Storage)

S3 stores raw, immutable stock events for long-term analytics and replay.

1. Create the S3 Bucket

Go to:
AWS Console → S3 → Create bucket

Bucket settings:

```sh
Bucket name:
    stock-market-data-bucket-<unique-id>
Region:
    Same as other services
Block public access:
    Enabled (default)
```

Create the bucket.

### Step 6: Creating Trend-Analysis Lambda

1. Create the DynamoDB Streams Lambda

Go to:
Lambda → Create function

Function name:
```sh
    dynamodb-stock-trend
Runtime:
    Python 3.11
Execution role:
    StockMarketLambdaRole
```

Create the function.

2. Add DynamoDB Stream as Trigger

Inside the Lambda function:

Add trigger: 
```sh
    DynamoDB
Select table:
    stock-market-table
Starting position:
    Latest
```
3. copy the content of python script from https://github.com/theprikshitsharma/Real-Time-Stock-Market-Data-Analytics-Pipeline/stock-market-table/lambda_function.py

### Step 7: Configure Amazon SNS for Email Alerts

This step enables real-time notifications when opportunities or anomalies are detected.

1. Create SNS Topic
Go to:
SNS → Topics → Create topic

```sh
Type:
    Standard
Name:
    stock-trend-alerts
```
Create the topic.

2. Add Email Subscription
Inside the topic:

Create subscription

```sh 
Protocol:
    Email
Endpoint:
    your email address
```

Confirm the subscription via email.

3. SNS Integration

When the Streams Lambda publishes a message:

SNS delivers an email alert

No SMTP setup required

Supports fan-out to multiple subscribers later

### Step 8: Create an S3 Bucket to store query results from Athena 

S3 Bucket for Athena Query Results

1. Create the S3 Bucket

Go to:
AWS Console → S3 → Create bucket

Bucket settings:

```sh
Bucket name:
    stock-market-data-bucket-<unique-id>
Region:
    Same as other services
Block public access:
    Enabled (default)
```

Create the bucket.



### Step 9: Create Glue Data Catalog

1. Create Glue Database

Go to:
AWS Glue → Data Catalog → Databases → Add database

```sh 
Database name:
    stock_market_catalog
```

Create the database.

2. Create Glue Table

Go to:
AWS Glue → Data Catalog → Tables → Add table → Add tables manually

Table configuration:
```sh 
Table name:
    stock_data_table
Database:
    stock_market_catalog
```

3. Define Data Store

```sh
Data source:
    S3
S3 location:
    s3://stock-market-data-bucket-<unique-id>/ 
Data format: 
    JSON
```

4. Define Columns (Schema)

copy the scema from https://github.com/theprikshitsharma/Real-Time-Stock-Market-Data-Analytics-Pipeline/schema/schema.json

This enables Athena to scan only relevant data, improving performance and reducing cost.

5. Finalize Table

Review and create the table.

Glue now stores:

```sh
Schema
S3 location
```
### Step 10: Query Historical Data Using Amazon Athena (Serverless SQL)

This step enables serverless SQL analytics on historical stock data stored in Amazon S3, without provisioning or managing any infrastructure.

1. Configure Amazon Athena

Go to:
AWS Console → Amazon Athena

Set query result location

Athena requires an S3 bucket to store query results.

Open Athena → Settings

```sh 
Set Query result location:
    s3://athena-query-results-<unique-id>/
```

Save settings

This bucket is used only for query outputs and metadata.

2. Select Glue Data Catalog

In the Athena query editor:

```sh 
Data source:
    AwsDataCatalog
Database:
    stock_market_catalog
```

You should now see the table you created in Glue.

3. Run Serverless SQL Queries

Athena runs SQL directly on data stored in S3.

Run this SQL query to test:
```sql
SELECT * FROM stock_data_table LIMIT 10;
```

Find Top 5 Stocks with the Highest Price Change
```sql
SELECT symbol, price, previous_close,
       (price - previous_close) AS price_change
FROM stock_data_table
ORDER BY price_change DESC
LIMIT 5;
```

Get Average Trading Volume Per Stock
```sql
SELECT symbol, AVG(volume) AS avg_volume
FROM stock_data_table
GROUP BY symbol;
```

```sql
SELECT symbol, price, previous_close,
       ROUND(((price - previous_close) / previous_close) * 100, 2) AS change_percent
FROM stock_data_table
WHERE ABS(((price - previous_close) / previous_close) * 100) > 5;
```

Athena can immediately query this table and would automatically write query results to:
    ```s3://athena-query-results-<unique-id>/```

