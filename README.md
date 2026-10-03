# Automating Data Lake Creation with AWS Lake Formation Blueprints

A hands-on AWS lab where I used a Lake Formation blueprint to automatically build a full ETL workflow that ingests AWS CloudTrail logs into a queryable data lake — then built a smaller version of the same workflow by hand to understand what the blueprint actually generates under the hood.

## Scenario
Acting as a data engineer, the task was to populate a data lake with CloudTrail log data so an operational analytics team could query it efficiently in Athena — using a Lake Formation blueprint to automate what would otherwise be a lot of manual Glue configuration.

## What I did

### 1. Registered S3 storage and created a database
Registered the CloudTrail logs' S3 location with Lake Formation, granted the workflow's IAM role access to it, and created a `cloudtraillogs-db` database in the Glue Data Catalog to hold the resulting metadata.

### 2. Used a blueprint to auto-generate an AWS Glue workflow
Used Lake Formation's built-in **AWS CloudTrail** blueprint, pointing it at the existing CloudTrail trail, the new database, and a target S3 location, with Parquet as the output format and on-demand scheduling. This single blueprint generated a complete AWS Glue workflow — crawlers, ETL jobs, and triggers — wired together automatically as a DAG (directed acyclic graph).

### 3. Ran and monitored the generated workflow
Started the workflow and watched its DAG progress through stages in the Glue console: a pre-crawl job marks the workflow as DISCOVERING, a crawler infers the CloudTrail log schema, a post-crawl job marks it IMPORTING, an ETL job transforms the data to Parquet, and a final job marks it COMPLETED.

### 4. Built a smaller version of the same workflow manually
To understand what the blueprint had actually built, manually assembled a simplified version of the same DAG node by node — start node, pre-crawl job, pre-crawl trigger, crawler, post-crawl trigger, post-crawl job — then extended it further with ETL and post-ETL triggers and jobs to replicate the blueprint's full structure.

### 5. Validated the results in Athena
Once the workflow completed, found two resulting tables — the pre-transform CloudTrail table and the final Parquet-formatted `lab_cloudtrail` table (20 columns) — and queried the transformed table directly in Athena, including a query isolating every log entry that contained an error code.

## Key takeaways
- A blueprint isn't magic — it's a pre-built template that generates the exact same crawler/trigger/job DAG you'd otherwise have to wire up manually, which became obvious once I rebuilt a version of it by hand
- Workflows let Lake Formation track a multi-step ETL process (crawl → transform → load) as a single trackable entity instead of several disconnected Glue jobs
- Querying CloudTrail logs directly through a data lake, rather than scrolling through raw logs, makes finding specific patterns (like every entry with a non-empty error code) a one-line SQL query instead of manual searching

## Tools
AWS Lake Formation, AWS Glue (Workflows, Crawlers, ETL), AWS CloudTrail, Amazon Athena, Amazon S3

---
*Completed as an AWS hands-on lab, including a hands-on challenge building a custom Glue workflow.*
