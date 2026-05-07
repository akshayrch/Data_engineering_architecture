# Week 5 — AWS-Native Data Pipelines (Batch + Real-Time)

**Dates:** May 18 – May 24, 2026
**Phase:** 2 (Ingestion Patterns & Cloud Pipelines)
**Weekly budget:** ~18 hrs
**Theme:** _Every AWS data service your resume mentions — plus the ones you'll be asked about — mapped into pick-right-tool frameworks._

---

## Why this week matters

Your resume lists S3, Glue, EMR, Kinesis, Lambda, Step Functions, Firehose, MWAA — but you've never defended them as a coherent architecture. Interviewers will ask "why Kinesis over MSK?", "why Glue over EMR?", "why Step Functions over Airflow?", "when does Lambda stop being the right answer?" This week gives you the 8 decision frameworks you need.

For the fintech / trading interview in particular, AWS-native depth is non-negotiable because most banks are on AWS and need you to be fluent in their stack.

---

## Learning objectives

- AWS batch pipeline stack: S3, Glue (Crawlers, ETL, Catalog), EMR, Athena, Redshift, Step Functions, Lambda
- AWS streaming stack: Kinesis Data Streams, Kinesis Firehose, MSK, MSK Serverless, Lambda event sources
- AWS orchestration: Step Functions vs MWAA vs EventBridge pipelines
- AWS CDC: Database Migration Service (DMS) for MySQL/Postgres/SQL Server → S3 or Kafka
- Cost model: spot vs on-demand, S3 storage classes, Glue DPU pricing, Kinesis shard pricing
- IAM patterns: cross-account, assume-role, KMS per-data-product
- Decision frameworks for every service pair

---

## Daily breakdown

> **Pace check:** This week is primarily AWS docs, blog posts, and hands-on labs — not dense books. AWS docs pages = 15–30 min each depending on length. Blog posts = 5–10 min. Labs are the time sinks, not the reading. If a lab overruns, let it — the hands-on muscle matters more than finishing the doc.

### Mon (2 hrs) — AWS batch stack overview
- 60 min read: AWS "Analytics Lens - AWS Well-Architected Framework" — the Data Processing + Data Storage sections
- 45 min read: AWS Glue docs — Crawlers, Catalog, ETL jobs, Glue Studio
- 15 min lab: Point a Glue Crawler at your S3 bucket of synthetic parquet from Week 4. Catalog it. Query via Athena.

### Tue (2 hrs) — EMR vs Glue vs Databricks
- 45 min read: AWS EMR docs + "EMR Serverless" announcement blog
- 45 min read: "Glue vs EMR vs Databricks" comparison blog posts (2–3 recent ones, filter noise)
- 30 min write: **3-way decision matrix.** For each scenario, pick Glue / EMR / Databricks + defend:
  - Daily 500GB job, small team, no Spark expertise → Glue
  - Complex ML pipeline, large team with Spark depth, cost-sensitive → EMR on Spot
  - Enterprise platform for 50 analysts + ML team, on Databricks already → Databricks

### Wed (2 hrs) — Kinesis (Data Streams + Firehose) vs MSK
- 45 min read: AWS Kinesis Data Streams docs + "Kinesis vs MSK" official AWS comparison
- 30 min read: MSK Serverless announcement + when it becomes cheaper than provisioned MSK
- 45 min lab: Build a mini Kinesis pipeline:
  - Python producer → Kinesis Data Stream (3 shards) → Lambda consumer → DynamoDB
  - Second flow: Kinesis Data Stream → Firehose → S3 partitioned parquet → Athena
- Compare to a Kafka equivalent: what's harder? What's easier? Cost?

### Thu (1.5 hrs) — DMS + CDC into AWS
- 45 min read: AWS DMS docs — especially "Ongoing Replication" (CDC mode)
- 30 min read: "DMS vs Debezium" blog posts — the trade-offs
- 15 min write: When DMS beats Debezium: managed, Oracle/SQL Server support, enterprise. When Debezium wins: open-source, Kafka-native, granular control, cheaper at scale.

### Fri (1.5 hrs) — Orchestration choices: Step Functions vs Airflow vs EventBridge
- 30 min read: AWS Step Functions docs — "Standard vs Express" workflows
- 30 min read: AWS EventBridge Pipes + Rules docs
- 30 min write: **Decision tree for orchestration on AWS.** Long-running stateful workflow → Step Functions Standard. Event-driven cheap glue → EventBridge Pipes. Complex DAG with dependencies + backfill → MWAA. Hybrid → MWAA triggered by EventBridge.

### Sat (4.5 hrs) — Full AWS pipeline lab — Batch + Real-Time end-to-end
Build a mini but complete AWS data pipeline today:

**Batch leg:**
- S3 bucket `raw/` partition by date → EventBridge on object-created → Lambda that validates manifest → Step Functions triggers → Glue ETL job → S3 `processed/` → Glue Crawler → Athena table → QuickSight (or Snowflake external table)

**Real-time leg:**
- Python producer → Kinesis Data Stream → Kinesis Firehose → S3 `realtime/` (partitioned by 5-min buckets) → Glue Crawler → Athena
- In parallel: Kinesis Data Stream → Lambda → DynamoDB for per-entity current state

Infrastructure as code via Terraform or AWS CDK. **No clicking around in the console** — that doesn't prove skill.

### Sun (4.5 hrs) — Cost + IAM + patterns drill
- 90 min: AWS cost calculator — estimate monthly run cost for your Saturday pipeline at 3 volumes: 1GB/day, 100GB/day, 10TB/day. Identify where costs explode.
- 90 min: IAM setup — cross-account assume-role from the dev account into a prod data account; KMS key per data product; least-privilege policy for Glue job that writes only to its specific prefix. Get this right ONCE; it's a frequent interview question.
- 60 min: **Lead-DE drill — architecting from scratch on AWS.**
  Scenario 1: "Ingest 50 partner APIs + 10 SFTP drops + 5 real-time webhook firehoses + CDC from 3 operational DBs into a lakehouse on AWS." Draw the full picture. Pick every service with 1-line defense.
- 30 min: Flashcards + Week 5 self-check

---

## Must-read this week
- **[CORE]** AWS Well-Architected Analytics Lens
- **[CORE]** AWS Glue + EMR + Kinesis + DMS + Step Functions docs (all 5)
- **[CORE]** "AWS Data Pipeline Decision Tree" blog posts (search the AWS Big Data blog)
- **[DEEP]** Chris Riccomini's blog — cloud data platform patterns
- **[DEEP]** DataExpert.io — "Bonus - Azure" (May 2025) if also covering multi-cloud

---

## Deliverable
- `projects/P1.5_aws_pipeline_demo/` — Terraform/CDK + code for the Saturday batch + real-time pipeline
- `notes/week_05/`:
  - `aws_service_matrix.md` (Glue/EMR/Databricks, Kinesis/MSK, DMS/Debezium, Step Functions/MWAA/EventBridge decision trees)
  - `aws_costing.md` (cost calc notes)
  - `iam_cross_account.md` (least-privilege examples)
  - `end_to_end_lab/` (IAC + README)
  - `lead_de_scenarios.md` (architecture drill from Sunday)
  - `flashcards.md` (30 cards)

---

## Self-check (Sunday)

1. Kinesis Data Streams vs MSK vs MSK Serverless — one-sentence pick for each
2. Glue vs EMR — 3 scenarios, pick + defend
3. Step Functions Standard vs Express — when each?
4. DMS vs Debezium — 2 scenarios, pick + defend
5. 500GB daily job on Spark — cheapest AWS path?
6. Orchestrating a 20-step workflow with one step that might take 3 hours — Standard Step Functions or Express? (Standard — Express has 5-min limit)
7. Lambda or Firehose — when is each the right Kinesis consumer?
8. Your AWS bill is $200k/month. Top 5 things you'd audit for a data team?

Pass = 7/8 cold.

---

## Notes / Discoveries
