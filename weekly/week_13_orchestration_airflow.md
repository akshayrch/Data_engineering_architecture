# Week 13 — Orchestration: Airflow Advanced, MWAA, Dagster/Prefect

**Dates:** Jul 13 – Jul 19, 2026
**Phase:** 6 (Orchestration, Quality, Debugging)
**Weekly budget:** ~14 hrs (LIGHT START — Jul 13–15 limited; resume full from Jul 16)
**Theme:** _You've already built MWAA + 400 pipelines. Now own every internal and know when Airflow is the wrong answer._

> **⚠️ Limited-access alert:** Jul 13–15 (Mon–Wed) limited. Use Mon–Wed for passive reading; resume hands-on work Thu–Sun.

---

## Why this week matters

Your resume has "spearheaded adoption of Airflow, AWS MWAA, 400 pipelines, 60% dependency improvement." A staff interviewer will drill: scheduler internals, executor types, sensor vs trigger, datasets vs XCom, dynamic task mapping, how MWAA differs from self-hosted, why not Dagster.

---

## Learning objectives

- Airflow architecture: scheduler, webserver, worker, triggerer, metadata DB
- Executors: Sequential, Local, Celery, Kubernetes — pick-each scenarios
- DAG scheduling internals: DagRun, TaskInstance, scheduler loop, pool slots
- Sensors: poke mode vs reschedule vs deferrable (triggerer-based)
- XCom vs Datasets (data-aware scheduling) vs TaskFlow API
- Dynamic task mapping (Airflow 2.3+)
- MWAA specifics: what's managed, what's not, limitations
- Dagster + Prefect — POV on when each wins over Airflow

---

## Daily breakdown

> **Pace check:** Airflow docs pages = 15–25 min each. Astronomer guides = 20–30 min each. The heavy item is Mon–Wed where Mon–Wed are limited-access light reading. The labs (executor config, sensor conversion) are 30–45 min each. Don't rush the MWAA STAR story on Saturday — it's the most valuable thing this week.

### Mon (2 hrs) — Architecture + scheduler
- 60 min read: Airflow docs — "Architecture Overview" + "Scheduler"
- 45 min read: "How does the scheduler loop work" blog posts (marclamberti.com has good ones)
- 15 min lab: Run `airflow db init` locally, inspect the metadata DB schema — understand DagRun, TaskInstance tables

### Tue (2 hrs) — Executors
- 60 min read: Airflow docs on each executor type
- 45 min read: Astronomer guides on Celery vs Kubernetes executor
- 15 min write: Decision matrix — which executor for which team/scale/platform?

### Wed (2 hrs) — Sensors + deferrable + triggerer
- 45 min read: Airflow docs "Sensors" + "Deferrable Operators"
- 45 min read: Astronomer blog "Deferrable Operators: The Quiet Revolution"
- 30 min lab: Convert a poke sensor to deferrable. Observe worker slot usage.

### Thu (1.5 hrs) — Datasets + TaskFlow + dynamic mapping
- 45 min read: Airflow docs on Datasets (data-aware scheduling, 2.4+)
- 30 min read: Dynamic task mapping docs + examples
- 15 min write: Datasets vs ExternalTaskSensor vs cross-DAG deps — when to pick what

### Fri (1.5 hrs) — MWAA + Dagster/Prefect POV
- 30 min read: AWS MWAA docs — especially limitations (no triggerer until recently, requirements.txt quirks, KMS, networking)
- 45 min read: Dagster concepts page + a "Dagster vs Airflow" blog post
- 15 min read: Prefect 2 "orchestration as code" intro
- 15 min flashcards

### Sat (4.5 hrs) — MWAA adoption story + multi-orchestrator drill
- 2 hrs: **Rebuild your MWAA adoption story for interview.** Resume line: "set up AWS MWAA, built Snowflake/AWS/DBT Core modules, replaced Tidal, 400 pipelines, 60% dependency mgmt improvement." Defense:
  - Why Airflow over Dagster over Prefect over Step Functions?
  - Why MWAA over self-hosted EKS?
  - What did "module" mean in your abstraction — custom operators? Hooks?
  - Migration strategy off Tidal — blue-green? Phased?
  - 60% dependency improvement — what was the metric? (mean dep-resolution time? fewer cross-team blockers?)
  - MWAA pain points you hit and how you worked around them
  Save as `mwaa_story.md` → **STAR story #2**
- 2 hrs: **Multi-orchestrator design drill.** Design the orchestration layer for:
  1. A team with 20 batch daily pipelines — Airflow vs cron vs Step Functions
  2. A data platform team serving 5 product teams — Airflow vs Dagster
  3. A real-time ML training pipeline — Flyte vs Airflow vs Prefect
  4. A reverse-ETL operator — Hightouch scheduling vs Airflow
- 30 min: Flashcards

### Sun (4.5 hrs) — P3 continuation + orchestration-heavy drill
- 2.5 hrs: **Project P3 progress** — wire up Airflow DAG that orchestrates your Debezium → Spark → Iceberg → dbt pipeline with full retry, SLA, alerting
- 90 min: **Design drill** — orchestrate a 200-model dbt project with upstream dependencies on 3 Spark pipelines, 2 Kafka sinks, and a partner SFTP ingestion. Draw it. Name every Airflow construct used.
- 30 min: Week 11 self-check + flashcards

---

## Must-read this week
- **[CORE]** Airflow docs — Architecture, Scheduler, Executors, Sensors, Deferrable Operators, Datasets
- **[CORE]** Astronomer guides — deferrable operators, TaskFlow, dynamic mapping
- **[CORE]** AWS MWAA user guide
- **[DEEP]** Dagster concepts page
- **[DEEP]** DataExpert.io — "Airflow" + "Pipeline Spec Building + Airflow Fundamentals"

---

## Deliverable
- **Project P3 orchestration wired up** (Airflow DAG running the full pipeline)
- `notes/week_11/`:
  - `airflow_architecture.md`
  - `executor_matrix.md`
  - `deferrable_lab/` (poke → deferrable conversion)
  - `datasets_vs_xcom.md`
  - `mwaa_story.md` (STAR #2)
  - `orchestrator_comparison.md`
  - `flashcards.md` (30 cards)

---

## Self-check (Sunday)

1. Walk the Airflow scheduler loop in 2 min — what files it parses, what queries it runs, what the triggerer does
2. When is deferrable operator a must? When is it unnecessary?
3. Datasets vs ExternalTaskSensor vs TriggerDagRunOperator — pick each for a scenario
4. MWAA vs self-hosted EKS Airflow — pros/cons cold
5. Dagster software-defined assets vs Airflow datasets — what's the difference?
6. Your scheduler is slow. 5 things to check.
7. Dynamic task mapping — when is it better than a for-loop in DAG file?

Pass = 6/7 cold.

---

## Notes / Discoveries
