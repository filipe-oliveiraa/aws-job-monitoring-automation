# Automated Job Monitoring – Cross-Account AWS Observability

## The Problem

Before this solution was implemented, support and analytics teams faced multiple challenges in monitoring scheduled AWS data jobs:

* ❌ **Manual Verification**: Teams had to manually check dozens of jobs daily across ERP, Fin-Data, and Analytics accounts, which was time-consuming and error-prone.
* ❌ **Delayed Detection**: Failures in critical jobs (e.g., Glue, Step Functions, AppFlow, DataSync) were often identified late, impacting data freshness and business decisions.
* ❌ **No Central Visibility**: Each AWS account operated in isolation with no unified monitoring framework, making it difficult to correlate failures or view job health trends.
* ❌ **Inconsistent Incident Handling**: Without automation, incident creation and follow-up were reactive and lacked standardization.
* ❌ **Limited Auditability**: No structured record of job execution history was available for compliance, troubleshooting, or business reporting.

---

## ✅ The Solution

To address these limitations, we designed and implemented a **serverless, cross-account job monitoring system** using native AWS services:

* **EventBridge** to capture and route job state changes from all accounts
* **Lambda Functions** to process events, trigger ServiceNow tickets, and send alert emails
* **DynamoDB** to temporarily store job states for traceability and reporting
* **SNS / Email Logic** to notify teams of critical failures or SLA breaches

---

## 📈 Key Benefits

* ✅ **Unified cross-account monitoring** for all critical job pipelines
* ✅ **Near real-time detection** of job failures across Glue, Step Functions, AppFlow, and DataSync
* ✅ **Saves 15+ hours/week** in operational time by eliminating manual checks
* ✅ **Improves auditability** and historical traceability of job runs
* ✅ **Automated alerting** and incident generation via ServiceNow and scheduled email reports
* ✅ **Better insights** into job health trends via BI dashboards

---

## 🧱 Architecture Overview

![Job Monitoring Architecture drawio (2)](https://github.com/user-attachments/assets/e7f51d0a-bde2-437e-9381-f3755f5d4f83)

---

🧠 Lambda Function Responsibilities

The monitoring system relies on **three purpose-specific AWS Lambda functions**, each handling a core part of the pipeline:

### 🔹 Lambda 1 – Job State Handler

* **Trigger**: EventBridge (cross-account)
* **Purpose**:

  * Receives job execution state change events (e.g., SUCCEEDED, FAILED) from AWS Glue, Step Functions, AppFlow, and DataSync.
  * Writes job run metadata (e.g., job name, status, timestamps, region, system, etc.) into a centralized **DynamoDB** table with a 7-day TTL.
  * Automatically creates **ServiceNow incidents** for job failures using defined business rules.

### 🔹 Lambda 2 – Email Reporter

* **Trigger**: Scheduled (3–4 times per day via EventBridge rule)
* **Purpose**:

  * Scans the most recent job states stored in DynamoDB.
  * Applies business filtering logic (e.g., critical job types or systems).
  * Sends consolidated **email alerts** to operations and analytics teams for any flagged jobs.

### 🔹 Lambda 3 – Weekly BI Exporter

* **Trigger**: Scheduled weekly (via cron rule)
* **Purpose**:

  * Retrieves **all job executions** from the last 7 days in DynamoDB.
  * Outputs structured data files for consumption by **Power BI** or other operational dashboards.
  * Enables historical job trend analysis and SLA tracking.

---

## About This Repository

This repository serves as a **portfolio showcase** for a real-world DevOps and cloud automation solution developed as part of my work. Source code is not publicly available due to client confidentiality, but this page summarizes the architecture, purpose, and value of the system.

---

## Contact

Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/filipe-amaro-oliveira/) for more details or to discuss similar infrastructure/DevOps solutions.
