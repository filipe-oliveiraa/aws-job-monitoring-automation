# Automated Job Monitoring – Cross-Account AWS Observability

## About This Repository
**This solution was solely developed by me (Filipe Amaro Oliveira) in a proactive manner when faced with the challenges described.**

This repository serves as a to share a real-world DevOps and cloud automation solution engineered by myself. Source code is not publicly available due to client confidentiality, but this page summarizes the architecture, purpose, and value of the system.

---

## The Problem

Before this solution was implemented, support and analytics teams faced multiple challenges in monitoring scheduled AWS data jobs:

* ❌ **Manual Verification**: Teams had to manually check dozens of jobs daily across ERP, Fin-Data, and Analytics accounts, which was time-consuming and error-prone.
* ❌ **Delayed Detection**: Failures in critical jobs (e.g., Glue, Step Functions, AppFlow, DataSync) were often identified late, impacting data freshness and business decisions.
* ❌ **No Central Visibility**: Each AWS account operated in isolation with no unified monitoring framework, making it difficult to correlate failures or view job health trends.
* ❌ **Inconsistent Incident Handling**: Without automation, incident creation and follow-up were reactive and lacked standardization.
* ❌ **Limited Auditability**: No structured record of job execution history was available for compliance, troubleshooting, or business reporting.

---

## The Solution

To address these limitations, we designed and implemented a **serverless, cross-account job monitoring system** using native AWS services:

* **EventBridge** to capture and route job state changes from all accounts
* **Lambda Functions** to process events, trigger ServiceNow tickets, and send alert emails
* **DynamoDB** to temporarily store job states for traceability and reporting
* **SNS / Email Logic** to notify teams of critical failures or SLA breaches

---

## Key Benefits

* ✅ **Unified cross-account monitoring** for all critical job pipelines
* ✅ **Near real-time detection** of job failures across Glue, Step Functions, AppFlow, and DataSync
* ✅ **Saves 15+ hours/week** in operational time by eliminating manual checks
* ✅ **Improves auditability** and historical traceability of job runs
* ✅ **Automated alerting** and incident generation via ServiceNow and scheduled email reports
* ✅ **Better insights** into job health trends via BI dashboards

---

## Architecture Overview

![Job Monitoring Architecture drawio (2)](https://github.com/user-attachments/assets/44197012-40d9-40c9-b50b-3bb52706a3b6)

---

## Volumetrics – Job Monitoring Architecture

| **Service**                       | **Key Metrics & Configuration Summary**                                                                                                                                     | **Measured Unit**                 | **Expected Monthly Usage**                     | **Avg Growth (Monthly)**                     | **Max Growth (50%)**                          | **Max Growth (2 Years)**          |
|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------|-----------------------------------------------|----------------------------------------------|-----------------------------------------------|----------------------------------|
| **DynamoDB**                     | Stores job state changes for monitoring. TTL set to 7 days.                                                                                                                 | Storage (GB), requests (r/w)     | ~5 MB storage, ~110,000 requests               | 5% growth: 5.3 MB, 115,500 req                | 7.5 MB, 165,000 req                            | 16 MB, 290,000 req                |
| **Lambda 1: Job Handler**        | Triggered per job event (~45K/month). 128 MB memory, ~100–150 ms exec. Includes DynamoDB write and ServiceNow API call on failure.                                         | Invocations, compute time        | 45,000 invocations, ~844 GB-s                 | 5% growth: 47,250 invocations, ~886 GB-s      | 67,500 invocations, ~1,266 GB-s              | 145,350 invocations              |
| **Lambda 2: Email Reporter**     | Scheduled function querying latest job states (~507 jobs) and sends alerts. Runs 3–4×/day. 256 MB memory, ~1.5 s exec.                                                     | Invocations, compute time        | 100 invocations, ~38 GB-s                     | 5% growth: 105 invocations, ~40 GB-s          | 150 invocations, ~57 GB-s                    | 260 invocations                  |
| **Lambda 3: Weekly Report**      | Weekly function scanning full 7-day job state (~10,600 items). 512 MB memory, ~4 s execution.                                                                               | Invocations, compute time        | 4 invocations, ~8.2 GB-s                       | 5% growth: 5 invocations, ~10.2 GB-s          | 6 invocations, ~12.3 GB-s                    | 10 invocations                   |
| **S3 (Weekly BI Export)**        | Sends 3–4 emails/day to 50 recipients using SNS. Each publish = 1 message × 50 deliveries.                                                                                   | Requests (email deliveries)      | ~5,400–7,200 emails/month                      | 5% growth: ~7,560 emails/month                | 10,800 emails/month                          | 15,000 emails/month              |

---

## Cost Optimization & Estimation (AWS)
This system was designed to be highly cost-effective, leveraging event-driven serverless architecture and fine-tuned resource usage. Below is a breakdown of the monthly costs per component assuming production-level workloads and no free tier.

### Assumptions:
Component	Assumptions
Scheduled jobs monitored	~500 daily jobs (ERP, Analytics, FinData)
Email alerts sent	~3–4 per day to ~50 users
Job state storage (DynamoDB)	7-day TTL retention, ~15–25KB per record
BI Exports	One compressed 1.5MB file per week
Athena Queries	Weekly queries, each scanning ~500KB per table

### Estimated Monthly Costs (No Free Tier)  
| **Service**              | **Description**                                                                          | **Estimated Monthly Cost** |
| ------------------------ | ---------------------------------------------------------------------------------------- | -------------------------- |
| **DynamoDB**             | Stores job status events (\~110,000 r/w requests/month, \~5–7.5 MB data w/ TTL)          | \~\$0.04                   |
| **Lambda (3 functions)** | - Event-driven invocations (\~45k/month)<br>- Email batch 3–4x/day<br>- Weekly BI export | \~\$0.024                   |
| **SNS**                  | 4 messages/day × 50 recipients × 30 days = 6,000 notifications → 0.06 × \$2/100k         | \~\$0.135                   |
| **S3 (BI Reports)**      | Stores 1 compressed file/week (\~1.5 MB), retained permanently                           | \~\$0.00018                  |
| **Athena**               | Weekly BI query, scans 3 tables × 500KB = \~6MB/month @ \$5 per TB scanned               | \~\$0.03                   |
| **CloudWatch Logs**      | Execution logs for Lambdas (\~3MB/month)                                                 | <\$0.01                    |


### Cost Optimization Techniques Used:
Optimization Area	Technique Applied
Lambda	Batched reports (3–4/day) instead of per-job; efficient parsing & short execution time
DynamoDB	TTL used to auto-expire data after 7 days; avoids long-term storage costs
SNS	Aggregated alerts; avoids per-job messages to users
S3	Gzip compression; only 1 file per week stored
Athena	Small, selective partitions to limit scan volume; infrequent queries
No EC2 / Containers	100% serverless = no idle compute charges


➡️ Total monthly cost (all-in, no free tier): **~_$0.25 – $0.39 USD_** 

---

##  IAM Permissions & Access Relationships 
| **Actor / Component**                     | **Needs Access To**                             | **Type of Permission**                                    | **Example Permissions / Policy Requirements**                                                                                                                                                                               |
| ----------------------------------------- | ----------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ERP / Analytics / FinData Jobs**        | Local EventBridge (default bus) in same account | **No explicit permissions required**                      | Native service integration; services like Glue, Step Functions emit events to the default EventBridge bus automatically.                                                                                                    |
| **Local EventBridge (Secondary Account)** | Central EventBridge Bus (Analytics Account)     | **Cross-Account Target – Resource Policy on Central Bus** | Central EventBridge bus must include a **resource policy** allowing `events:PutEvents` from the **source account's root**.<br><br> |
| **EventBridge Rule (Secondary)**          | Central EventBridge Bus                         | **Configured as Event Rule Target**                       | Rule is created in the source account. Target ARN must point to the central event bus. IAM permission not required; EventBridge invokes itself.                                                                             |
| **EventBridge (Central)**                 | Lambda – Job State Handler                      | **Event Rule Target (Internal)**                          | Not applicable. Lambda is invoked as target by EventBridge rule in the central account.                                                                                                                                     |
| **Lambda – Job State Handler**            | DynamoDB Table                                  | **IAM Role**                                              | `dynamodb:PutItem`, `dynamodb:UpdateItem`, `dynamodb:DeleteItem`                                                                                                                                                            |
|                                           | CloudWatch Logs                                 | **IAM Role**                                              | `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`                                                                                                                                                          |
|                                           | (Optional) ServiceNow API                       | **IAM Role**                                              | `execute-api:Invoke` or `servicenow:*` (for incident reporting, if applicable)                                                                                                                                              |
| **Lambda – Email Reporter**               | DynamoDB Table                                  | **IAM Role**                                              | `dynamodb:Scan`, `dynamodb:Query`                                                                                                                                                                                           |
|                                           | SNS Topic                                       | **IAM Role + Resource Policy on Topic**                   | IAM role must allow `sns:Publish`.<br>The SNS topic must include a resource policy allowing publish from the Lambda role.                                                                                                   |
|                                           | CloudWatch Logs                                 | **IAM Role**                                              | `logs:CreateLogStream`, `logs:PutLogEvents`                                                                                                                                                                                 |
| **SNS Topic**                             | -                                               | **Resource Policy**                                       | Allow `sns:Publish` from the `email-reporter-lambda-role`.                                                                                                                                                                  |
| **Lambda – Weekly Exporter**              | DynamoDB Table                                  | **IAM Role**                                              | `dynamodb:Scan`, `dynamodb:Query`                                                                                                                                                                                           |
|                                           | S3 Export Bucket                                | **IAM Role + Bucket Policy**                              | IAM role must allow `s3:PutObject`, `s3:GetObject`, `s3:ListBucket`.<br>Bucket policy must allow this role access.                                                                                                          |
|                                           | CloudWatch Logs                                 | **IAM Role**                                              | `logs:CreateLogStream`, `logs:PutLogEvents`                                                                                                                                                                                 |
| **S3 Bucket (Exports)**                   | Export Lambda, Athena                           | **Bucket Policy**                                         | Allow `s3:PutObject` from export Lambda, and `s3:GetObject` from Athena and Power BI roles.                                                                                                                                 |
| **Athena (Query Engine)**                 | S3 Export Bucket                                | **IAM Role + Bucket Policy**                              | IAM role must allow `s3:GetObject`.<br>Bucket policy must allow access from this role.                                                                                                                                      |
|                                           | Glue Catalog (if used)                          | **IAM Role**                                              | `glue:GetTable`, `glue:GetDatabase`                                                                                                                                                                                         |
| **Power BI / Analysts**                   | Athena + S3 Export Bucket                       | **Federated IAM Role + Bucket Policy**                    | IAM role must allow `athena:StartQueryExecution`, `athena:GetQueryResults`.<br>Bucket must allow `s3:GetObject`.                                                                                                            |


---

## Lambda Function Responsibilities

The monitoring system relies on **three purpose-specific AWS Lambda functions**, each handling a core part of the pipeline:

### 🔹 Lambda 1 – Job State Handler

* **Trigger**: EventBridge (cross-account)
* **Purpose**:

  * Receives job execution state change events (e.g., SUCCEEDED, FAILED) from AWS Glue, Step Functions, AppFlow, and DataSync.
  * Writes job run metadata (e.g., job name, status, timestamps, region, system, etc.) into a centralized **DynamoDB** table with a 7-day TTL.
  * Automatically creates **ServiceNow incidents** for job failures using defined business rules.

### Lambda 2 – Email Reporter

* **Trigger**: Scheduled (3–4 times per day via EventBridge rule)
* **Purpose**:

  * Scans the most recent job states stored in DynamoDB.
  * Applies business filtering logic (e.g., critical job types or systems).
  * Sends consolidated **email alerts** to operations and analytics teams for any flagged jobs.

### Lambda 3 – Weekly BI Exporter

* **Trigger**: Scheduled weekly (via cron rule)
* **Purpose**:

  * Retrieves **all job executions** from the last 7 days in DynamoDB.
  * Outputs structured data files for consumption by **Power BI** or other operational dashboards.
  * Enables historical job trend analysis and SLA tracking.

---


## Contact

Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/filipe-amaro-oliveira/) for more details or to discuss similar infrastructure/DevOps solutions.
