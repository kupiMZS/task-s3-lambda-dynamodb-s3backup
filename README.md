
# AWS Lambda, DynamoDB & Scheduled S3 Backup

**Date:** November 20, 2026  
**Region:** Asia Pacific (Mumbai) — `ap-south-1`

## 1. Project Overview

This assignment implements a serverless AWS pipeline that retrieves 10 users from the JSONPlaceholder API, stores them in DynamoDB, and creates JSON backups in Amazon S3 using AWS Lambda and EventBridge Scheduler.

### Architecture

```mermaid
flowchart TD
    A["JSONPlaceholder API<br/>10 Users"]
    B["Lambda #1<br/>Import Users"]
    C[("DynamoDB<br/>jsonplaceholder-users")]
    D["Lambda #2<br/>Export Users"]
    E[("Amazon S3<br/>JSON Backups")]
    F["EventBridge Scheduler<br/>Every 2 Minutes"]
    G["CloudWatch<br/>Logs and Metrics"]

    A -->|"HTTP GET"| B
    B -->|"PutItem"| C
    C -->|"Scan"| D
    D -->|"PutObject"| E
    F -->|"Invoke"| D
    B -.-> G
    D -.-> G

    classDef api fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef lambda fill:#ffedd5,stroke:#ea580c,color:#7c2d12
    classDef storage fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef other fill:#f3e8ff,stroke:#9333ea,color:#581c87

    class A api
    class B,D lambda
    class C,E storage
    class F,G other
```

## 2. Implementation and Results

### 2.1 DynamoDB Setup

Created the `jsonplaceholder-users` table using `id` as the Number partition key.

**DynamoDB Table:**

![DynamoDB Table](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-18-38.png>)

**Stored Records:**

![DynamoDB Users](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-14-26.png>)

The table contains 10 user records.

### 2.2 Lambda #1 — Import Users

**Function:** `fetch-jsonplaceholder-user`

The first Lambda retrieves users from the JSONPlaceholder API and stores them in DynamoDB using `PutItem`.

**Result:** HTTP 200, 10 users saved successfully.

![Lambda 1 Test](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-22-47.png>)

### 2.3 IAM Permissions

Configured separate IAM execution roles:

- Lambda #1: `dynamodb:PutItem`
- Lambda #2: `dynamodb:Scan`, `s3:PutObject`
- CloudWatch logging permissions for both functions

**Lambda #2 IAM Policy:**

![IAM Configuration](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 15-54-23.png>)

### 2.4 Lambda #2 — Export to S3

**Function:** `export-dynamodb-to-s3`

The second Lambda scans DynamoDB and exports the records as timestamped JSON files.

**Environment Variables:**

![Lambda 2 Environment Variables](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-19-46.png>)

**Test Result:** HTTP 200, 10 records exported successfully.

![Lambda 2 Test](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-22-24.png>)

**S3 Bucket:** `student-lambda-backups-kabbo2026`

Multiple JSON files were created during testing.

![S3 Backup Files](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-14-09.png>)

### 2.5 EventBridge Scheduler

Created a recurring EventBridge schedule named `dynamodb-backup-every-2min`.

**Configuration:**
- Schedule: `rate(2 minutes)`
- Target: `export-dynamodb-to-s3`
- Flexible time window: Off

![EventBridge Scheduler](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-19-14.png>)

The schedule was configured to invoke Lambda #2 automatically every two minutes.

### 2.6 CloudWatch Monitoring

CloudWatch log groups were created for both Lambda functions. CloudWatch provides execution logs and metrics to help monitor errors and function performance.

![CloudWatch Log Groups](<s3_lambda_dynamodb_eventbridge_cloudwatch_ss/Screenshot From 2026-11-20 16-24-15.png>)

## 3. Assignment Questions and Answers

**Q1. Difference between Lambda and Scheduler execution roles?**

Lambda's execution role allows access to DynamoDB and S3. Scheduler's execution role allows EventBridge to invoke Lambda.

**Q2. Why doesn't running Lambda #1 twice create 20 users?**

`PutItem` replaces records with matching partition keys, keeping 10 unique users.

**Q3. Why use unique timestamps and request IDs?**

To create unique backup filenames and prevent overwriting previous exports.

**Q4. Difference between logs and metrics?**

Logs contain execution details, while metrics measure invocations, errors, and duration.

**Q5. What if DynamoDB is deleted but S3 backups remain?**

The JSON records remain in S3, but the original table configuration and indexes cannot be automatically recovered from JSON alone.

**Q6. Why is a two-minute export unsuitable for large databases?**

Frequent scans consume resources, increase costs, and do not guarantee consistent backups. DynamoDB Point-in-Time Recovery (PITR) is better for production.

## 4. Conclusion

The AWS serverless pipeline was implemented and successfully tested. Lambda #1 saved 10 users in DynamoDB, and Lambda #2 exported their records to JSON files in S3.

EventBridge Scheduler was configured for recurring backups, while CloudWatch log groups were available for monitoring.

The screenshots demonstrate successful manual tests and the configured schedule. Separate evidence is needed to confirm two automated exports and the schedule's final disabled status.
