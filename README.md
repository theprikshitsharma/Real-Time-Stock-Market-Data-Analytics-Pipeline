# Real-Time Stock Market Data Analytics Pipeline on AWS

### Overview of Project ☁️
This project builds a real-time stock market data analytics pipeline using AWS, leveraging event-driven architecture and serverless technologies. The architecture ingests, processes, stores, and analyzes stock market data in real-time while minimizing costs. .

### Key tasks include:
1. Streaming real-time stock data from sources like yfinance using Amazon Kinesis Data Streams.
2. Processing data and detecting anomalies with AWS Lambda.
3. Storing processed stock data in Amazon DynamoDB for low-latency querying.
4. Storing raw stock data in Amazon S3 for long-term analytics.
5. Querying historical data using Amazon Athena.
6. Sending real-time stock trend alerts using AWS Lambda & Amazon SNS (Email/SMS).

### Project Architecture:
<img width="1538" height="750" alt="image" src="https://github.com/user-attachments/assets/ec477d89-a2ee-4bab-8a5e-602d2cd81de4" />

### Prerequisites: 
1. Install Python (if not already installed): Open a code editor like VSCode and ensure you have Python 3.8+ installed by running the below command in the terminal.
2. Install Required Python Libraries (pip3 install -r [requirements.txt](https://github.com/yeshwanthlm/Stock-Market-Real-Time-Data-Analytics-Pipeline-on-AWS/blob/main/Step1/requirements.txt))
3. Configure AWS Credentials (if not done already):
* Install AWS CLI (if not installed):
* Download from [AWS CLI](https://aws.amazon.com/cli/) and install it.
* Configure AWS CLI (if not done previously):
* Run the following command and enter your AWS credentials.
```sh
aws configure
```
* AWS Access Key ID → Enter your key
* AWS Secret Access Key → Enter your secret
* Default region name → Use the region where you created the Kinesis stream (e.g., `us-east-1`).
* Default output format → Leave empty (press Enter).

4. Create a Role for LambdaFunction ```StockMarketLambdaRole``` and Attach the following policies: 
```sh
AmazonKinesisFullAccess
AmazonDynamoDBFullAccess
AWSLambdaBasicExecutionRole
AmazonS3FullAccess
AmazonSNSFullAccess
```
5. Clone [this](https://github.com/yeshwanthlm/Stock-Market-Real-Time-Data-Analytics-Pipeline-on-AWS/tree/main) repository on your local machine:
```sh
git clone https://github.com/yeshwanthlm/Stock-Market-Real-Time-Data-Analytics-Pipeline-on-AWS.git
```

### Required SQL Queries: 
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
#### What does this query do?
This query calculates the price change for each stock by subtracting yesterday’s closing price from the current price, creating a new column called price_change. It then sorts all stocks by this change in descending order and returns the top 5 stocks with the biggest price increase.

Get Average Trading Volume Per Stock
```sql
SELECT symbol, AVG(volume) AS avg_volume
FROM stock_data_table
GROUP BY symbol;
```
#### What does this query do?
This query groups all records by stock symbol and calculates the average trading volume for each stock using AVG(volume). The result shows how actively each stock is traded on average.

Find Anomalous Stocks (Price Change > 5%)
```sql
SELECT symbol, price, previous_close,
       ROUND(((price - previous_close) / previous_close) * 100, 2) AS change_percent
FROM stock_data_table
WHERE ABS(((price - previous_close) / previous_close) * 100) > 5;
```
#### What does this query do?
This query calculates the percentage price change for each stock compared to its previous close and labels it as change_percent. It then filters and returns only those stocks whose price moved more than 5% up or down, identifying unusual or anomalous price movements.
