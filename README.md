# DE Architecture Mastery — Folder Index

**What this folder is:** Your 18-week (Apr 20 – Aug 20, 2026) study plan for Lead/Staff Data Engineering interviews at FAANG, fintech/trading, and analytics-engineering shops.

---

## Where do I start?

1. **[`00_master_plan.md`](00_master_plan.md)** — Read this first. The master navigator: 8 phases, 18 weeks, 6 projects, checkpoints, and the "what to prioritize" section.
2. **This week's file in `weekly/`** — Open the file for the current week. Each file is self-contained: daily slots, readings, labs, deliverables, self-check.
3. **The active project brief in `projects/`** — Each weekend, you work on the project assigned to that week. Open the brief for scope.

Daily flow: open `00_master_plan.md` → jump to today's week file → execute today's slot → Saturday = project work.

---

## What's in each folder

### Top level
| File | Purpose |
|---|---|
| [`README.md`](README.md) | This file — folder index |
| [`00_master_plan.md`](00_master_plan.md) | **Start here.** 18-week plan overview, phases, checkpoints, project list, week-by-week table |
| [`resources.md`](resources.md) | Curated reading/watching list organized by topic (books, docs, blogs, YouTube) |

### `weekly/` — one file per week (18 total)
Each file has: _Why this week matters · Learning objectives · Daily Mon–Sun breakdown · Must-read list · Deliverables · Self-check · Notes/Discoveries_.

| Week | Dates | Topic | File |
|---|---|---|---|
| 1 | Apr 20 – Apr 26 | Architecture foundations, pipeline types | [`week_01_architecture_foundations.md`](weekly/week_01_architecture_foundations.md) |
| 2 | Apr 27 – May 3 | Dimensional modeling + SCD + incremental loads | [`week_02_dimensional_modeling.md`](weekly/week_02_dimensional_modeling.md) |
| 3 | May 4 – May 10 | Lakehouse modeling (Iceberg/Delta/Hudi) | [`week_03_lakehouse_modeling.md`](weekly/week_03_lakehouse_modeling.md) |
| 4 | May 11 – May 17 | Ingestion patterns (API, FTP, webhooks, SSE) | [`week_04_ingestion_patterns.md`](weekly/week_04_ingestion_patterns.md) |
| 5 | May 18 – May 24 | AWS-native pipelines (Glue, Kinesis, MSK, DMS) | [`week_05_aws_native_pipelines.md`](weekly/week_05_aws_native_pipelines.md) |
| 6 | May 25 – May 31 | PySpark internals (Catalyst, Tungsten) | [`week_06_pyspark_internals.md`](weekly/week_06_pyspark_internals.md) |
| 7 | Jun 1 – Jun 7 | Spark tuning, skew, shuffle, memory | [`week_07_spark_tuning_optimization.md`](weekly/week_07_spark_tuning_optimization.md) |
| 8 | Jun 8 – Jun 14 | Databricks, Delta, Structured Streaming intro | [`week_08_databricks_delta_streaming_intro.md`](weekly/week_08_databricks_delta_streaming_intro.md) |
| 9 | Jun 15 – Jun 21 | Kafka internals (ISR, rebalance, exactly-once) | [`week_09_kafka_deep_dive.md`](weekly/week_09_kafka_deep_dive.md) |
| 10 | Jun 22 – Jun 28 | Streaming patterns + Flink ⚠️ partial limited | [`week_10_streaming_patterns_flink.md`](weekly/week_10_streaming_patterns_flink.md) |
| 11 | Jun 29 – Jul 5 | Advanced Snowflake ⚠️ LIGHT | [`week_11_snowflake_advanced.md`](weekly/week_11_snowflake_advanced.md) |
| 12 | Jul 6 – Jul 12 | Advanced dbt + Trino ⚠️ **READING-ONLY** | [`week_12_dbt_advanced_iceberg_trino.md`](weekly/week_12_dbt_advanced_iceberg_trino.md) |
| 13 | Jul 13 – Jul 19 | Airflow advanced, MWAA ⚠️ LIGHT START | [`week_13_orchestration_airflow.md`](weekly/week_13_orchestration_airflow.md) |
| 14 | Jul 20 – Jul 26 | DQ, observability, CDC, debugging playbook | [`week_14_data_quality_observability_cdc.md`](weekly/week_14_data_quality_observability_cdc.md) |
| 15 | Jul 27 – Aug 2 | System design: core patterns | [`week_15_system_design_core.md`](weekly/week_15_system_design_core.md) |
| 16 | Aug 3 – Aug 9 | System design: scale, multi-tenancy, cost | [`week_16_system_design_advanced.md`](weekly/week_16_system_design_advanced.md) |
| 17 | Aug 10 – Aug 16 | STAR, SQL, coding, mock interviews | [`week_17_behavioral_sql_mock.md`](weekly/week_17_behavioral_sql_mock.md) |
| 18 | Aug 17 – Aug 20 | Final sprint: polish, resume, LinkedIn | [`week_18_final_sprint.md`](weekly/week_18_final_sprint.md) |

⚠️ = limited access window (vacation / holidays / no dev env). Plan is already shaped around them.

### `projects/` — one brief per hands-on project (6 total)
Each brief has: _Goal · Scope · Tech stack · Milestones · Deliverable · STAR angle for interviews_.

| # | File | Ship by | What it is |
|---|---|---|---|
| P1 | [`P1_realtime_clickstream.md`](projects/P1_realtime_clickstream.md) | Week 10 | Kafka → Spark SS → Iceberg → Snowflake real-time analytics |
| P2 | [`P2_batch_skew_mitigation.md`](projects/P2_batch_skew_mitigation.md) | Week 8 | PySpark batch job demonstrating 3 skew-mitigation strategies |
| P3 | [`P3_cdc_lakehouse.md`](projects/P3_cdc_lakehouse.md) | Week 14 | Debezium CDC → Kafka → Spark → Iceberg → dbt marts |
| P4 | [`P4_watch_history_recfeed.md`](projects/P4_watch_history_recfeed.md) | Week 15 | **Written system-design spec only** (no code) — 8–12 pg doc |
| P5 | [`P5_dq_observability.md`](projects/P5_dq_observability.md) | Week 14 | GE + Soda + OpenLineage framework on top of P3 |
| P6 | [`P6_multitenant_feature_store.md`](projects/P6_multitenant_feature_store.md) | Week 16 (stretch) | Iceberg + Trino + Redis multi-tenant feature store |

---

## Mental model: three file types, three jobs

- **The master plan** = your _strategy_. Read once a week to stay oriented.
- **Weekly files** = your _daily instructions_. One tab open each day.
- **Project briefs** = your _weekend work_. One active at a time, Saturdays.
- **`resources.md`** = your _bookshelf_. Don't read it cover-to-cover; look things up when a week file points to it.

If in doubt: open [`00_master_plan.md`](00_master_plan.md) and follow the links.

---

_Index last updated: 2026-04-20_
