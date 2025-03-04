# AWS Athena to Analyze AWS Logs Stored in S3

## Overview
AWS Athena enables the analysis of unstructured, semi-structured, and structured data stored in Amazon S3 using simple SQL queries. In this guide, we will focus on analyzing Application Load Balancer (ALB) access logs stored in S3.

## Prerequisites
- An ALB running with access logs enabled.
- ALB access logs are being stored in an S3 bucket.

## Step 1: Set Up Query Output Location
1. Open the Amazon Athena console.
2. Under **Administration**, click on **Workgroups**.
3. Click on **Create a new Workgroup**.
4. Provide a **Workgroup name**.
5. In **Query result configuration**, specify the S3 path where you want to save the query output.
6. Click **Create** to finalize the workgroup.

## Step 2: Select the Workgroup
1. In the **Query Editor**, click on the dropdown for **Workgroup**.
2. Select the Workgroup you just created.

## Step 3: Create a Database (Optional)
If you want to create a logical grouping of tables, you can create a database in AWS Athena.

### Command to Create Database
```sql
CREATE DATABASE elb_logs;
```

## Step 4: Create Table for ALB Logs
Since ALB access logs have a known structure, we can create a table with partitioning to optimize query performance.

### Command to Create External Table
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS alb_access_logs (
    type string,
    time string,
    elb string,
    client_ip string,
    client_port int,
    target_ip string,
    target_port int,
    request_processing_time double,
    target_processing_time double,
    response_processing_time double,
    elb_status_code int,
    target_status_code string,
    received_bytes bigint,
    sent_bytes bigint,
    request_verb string,
    request_url string,
    request_proto string,
    user_agent string,
    ssl_cipher string,
    ssl_protocol string,
    target_group_arn string,
    trace_id string,
    domain_name string,
    chosen_cert_arn string,
    matched_rule_priority string,
    request_creation_time string,
    actions_executed string,
    redirect_url string,
    lambda_error_reason string,
    target_port_list string,
    target_status_code_list string,
    classification string,
    classification_reason string,
    conn_trace_id string
)
PARTITIONED BY (day STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
    'serialization.format' = '1',
    'input.regex' = '([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*)[:-]([0-9]*) ([-.0-9]*) ([-.0-9]*) ([-.0-9]*) (|[-0-9]*) (-|[-0-9]*) ([-0-9]*) ([-0-9]*) \"([^ ]*) (.*) (- |[^ ]*)\" \"([^\"]*)\" ([A-Z0-9-_]+) ([A-Za-z0-9.-]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^\"]*)\" ([-.0-9]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^ ]*)\" \"([^\\s]+?)\" \"([^\\s]+)\" \"([^ ]*)\" \"([^ ]*)\" ?([^ ]*)?'
)
LOCATION 's3://amzn-s3-demo-bucket/AWSLogs/<ACCOUNT-NUMBER>/elasticloadbalancing/<REGION>/'
TBLPROPERTIES (
    "projection.enabled" = "true",
    "projection.day.type" = "date",
    "projection.day.range" = "2022/01/01,NOW",
    "projection.day.format" = "yyyy/MM/dd",
    "projection.day.interval" = "1",
    "projection.day.interval.unit" = "DAYS",
    "storage.location.template" = "s3://amzn-s3-demo-bucket/AWSLogs/<ACCOUNT-NUMBER>/elasticloadbalancing/<REGION>/${day}"
);
```
[SKIP] Simple Example:

```sql
CREATE EXTERNAL TABLE alb_access_logs (
  timestamp string,
  version string,
  messages string,
  logger_name string,
  thread_name string,
  level string,
  level_value string,
  UserAgent string,
  X_Forwarded_For string,
  X_Amzn_Trace_Id string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  'separatorChar' = ',',
  'quoteChar' = '"',
  'escapeChar' = '\\'
)
LOCATION 's3://cloudwatch-loggroups-archive-prod/alb_access_logs/prod/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

### Note
- If the input log is in JSON format, use `org.openx.data.jsonserde.JsonSerDe` for `ROW FORMAT SERDE`.
- If you modify the schema, drop the existing table and recreate it with the updated schema using:
```sql
DROP TABLE alb_access_logs;
```

## Step 5: Run Queries Against the Table
Once the table is created, you can run various SQL queries to analyze the logs.

### Query 1: View All Attributes
```sql
SELECT * FROM "elb_logs"."alb_access_logs" LIMIT 10;
```

### Query 2: View Selected Attributes
```sql
SELECT type, time, client_ip, request_verb, request_url, elb_status_code 
FROM "elb_logs"."alb_access_logs" 
LIMIT 100;
```

### Query 3: View All Unique Client IPs
```sql
SELECT DISTINCT client_ip 
FROM "elb_logs"."alb_access_logs";
```

### Query 4: Order Client IPs Based on Status Code
```sql
SELECT DISTINCT(client_ip), elb_status_code 
FROM "elb_logs"."alb_access_logs" 
ORDER BY client_ip;
```

### Query 5: Count Requests from Each IP
```sql
SELECT client_ip, COUNT(client_ip) AS number_of_requests
FROM "elb_logs"."alb_access_logs"
GROUP BY client_ip
ORDER BY number_of_requests DESC;
```

### Query 6: Count Requests from Each User-Agent
```sql
SELECT COUNT(user_agent) AS request_count_from_agent, user_agent
FROM "elb_logs"."alb_access_logs"
GROUP BY user_agent
ORDER BY request_count_from_agent DESC;
```

### Query 7: Filter Client IPs Based on Status Code
```sql
SELECT time, client_ip, request_verb, request_url, elb_status_code
FROM "elb_logs"."alb_access_logs"
WHERE elb_status_code >= 400;
```

### Query 8: Filter Client IPs Based on Status Code (Within a Range)
```sql
SELECT day, client_ip, request_verb, request_url, elb_status_code
FROM "elb_logs"."alb_access_logs"
WHERE day BETWEEN '2024/08/16' AND '2024/08/17'
ORDER BY day ASC;
```

### Query 9: Filter Based on Request Verbs
```sql
SELECT request_verb, request_url
FROM "elb_logs"."alb_access_logs"
WHERE request_verb IN ('GET', 'POST');
```

### Query 10: Filter Based on a Pattern (Using the LIKE Operator)
#### Example: Fetch Request URLs Containing the Keyword "admin"
```sql
SELECT request_verb, request_url
FROM "elb_logs"."alb_access_logs"
WHERE request_url LIKE '%/admin/%';
```

#### Exclude Specific Patterns
```sql
SELECT request_verb, request_url
FROM "elb_logs"."alb_access_logs"
WHERE request_url NOT LIKE '%admin%';
```
