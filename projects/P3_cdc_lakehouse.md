# Project P3 — CDC-Based Analytics Lakehouse

**Phase:** 4–5 (Warehouse + Orchestration)
**Kickoff week:** 10 · **Ship week:** 12
**Effort:** ~15 hrs across weeks 10–12
**Primary tools:** Debezium + Kafka + Spark Structured Streaming + Iceberg + dbt + Airflow + Snowflake
**Leverages existing:** Your RabbitMQ CDC work at Republic Services
**Deliverable type:** End-to-end working system + orchestrated with DQ + STAR writeup

---

## Why this project

CDC is the #1 underrated DE skill. Every company has operational DBs and wants analytics on them. LinkedIn Round 2 Q4 (schema evolution across teams) and Round 2 Q3 (DQ in streaming + batch) both live here.

Your RabbitMQ story shows you did CDC. This project proves you can design the full modern stack version.

---

## Success criteria

You're done when:

- Source Postgres with 3 tables (customers, orders, order_items) populated with synthetic data
- Debezium connector in Kafka Connect replicating to Kafka (Avro + Schema Registry)
- Spark Structured Streaming consumer lands raw change events into Iceberg "bronze" layer
- dbt transforms bronze → silver (deduplicated, current state) → gold (star schema marts)
- Airflow DAG orchestrates the whole pipeline with sensors, retries, and SLAs
- Schema-evolution scenario tested: add a column to source Postgres, verify no downstream failure
- At least 4 DQ checks (Great Expectations or dbt tests) running per layer
- OpenLineage wired into Airflow for lineage visibility
- README includes architecture, schema-evolution writeup, and the "what I'd rearchitect" section

---

## Architecture

```
┌──────────────────┐
│   Postgres (OLTP)│  customers, orders, order_items
└────────┬─────────┘
         │ WAL
         ↓
┌──────────────────┐
│     Debezium     │  → produces change events (c/u/d) to Kafka topics
└────────┬─────────┘
         ↓
┌──────────────────┐
│ Kafka + Schema   │  Avro, BACKWARD compatibility
│    Registry      │
└────────┬─────────┘
         ↓
┌──────────────────────────┐
│ Spark Structured Streaming│  dedupe + land raw into Iceberg
│                           │  table: bronze.cdc_<tablename>
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│         Iceberg          │
│  bronze/silver/gold      │  silver = latest state; gold = star schema
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│           dbt            │  macros for SCD2, SCD1, merge-on-read
│     (Iceberg adapter)    │
└────────────┬─────────────┘
             ↓
┌──────────────────────────┐
│         Snowflake        │  external table over Iceberg, plus materialized views
│  (BI consumption layer)  │  exposed via Cortex/Streamlit for bonus
└──────────────────────────┘

Orchestrated by: Airflow (MWAA-style, local container for dev)
Monitored by:    OpenLineage + Great Expectations
```

---

## Week-by-week plan

### Week 10 (kickoff, ~6 hrs across Sat+Sun)
- Stand up Postgres + Debezium + Kafka + Schema Registry locally (docker-compose)
- Generate synthetic data: 10k customers, 100k orders, 300k order_items
- First Debezium connector created, change events flowing to Kafka
- Spark SS consumer lands bronze Iceberg tables
- dbt project scaffolded: `stg_`, `int_`, `fct_`/`dim_`

### Week 11 (~3 hrs mid-week + weekend overlap)
- Finalize 3 dbt models: `fct_orders` (one row per order), `dim_customer` (SCD Type 2), `fct_order_items`
- Airflow DAG wires everything: sensor on Kafka → Spark SS job → dbt run → DQ check → publish event
- Retries + SLA set on each task

### Week 12 (ship by Saturday)
- Add 4 DQ checks minimum — row counts, not-null, uniqueness, referential integrity
- OpenLineage integration
- Schema-evolution test: add a column to source `customers.email_verified`. Verify:
  - Debezium picks up the new column
  - Schema registry registers new version (BACKWARD compatibility)
  - Bronze Iceberg table gets the column via schema evolution
  - dbt silver/gold models keep running without failure
- README + STAR writeup

---

## Design decisions to defend

- **Why Debezium over query-based CDC (e.g., JDBC source connector)?**
  → log-based picks up deletes, has less source load, preserves ordering, minimal latency
- **Why Avro + Schema Registry over JSON?**
  → BACKWARD compatibility enforcement, schema-as-contract, smaller payloads, evolution safety
- **Why Spark Structured Streaming as the consumer, not Kafka Connect sink?**
  → dedup logic, time-based partitioning, compaction scheduling — all easier in SS
- **Why Iceberg bronze layer, not direct-to-warehouse?**
  → backfill-ability, format neutrality, cheap time travel for debugging
- **SCD Type 2 on dim_customer — why dbt snapshot vs custom MERGE?**
  → dbt snapshot is simpler if you can live with its limitations (only one updated_at); custom MERGE if you need multi-column change detection
- **Schema evolution — why BACKWARD compatibility, not FULL?**
  → FULL locks producers from adding new fields freely; BACKWARD lets producers add optional fields, consumers always work
- **Why OpenLineage, not just dbt docs?**
  → cross-tool lineage — dbt lineage alone stops at the warehouse boundary

---

## DQ checks to implement (at minimum)

- **Bronze layer**: row count > 0 per hour; no duplicate (table, op, key, lsn) combos
- **Silver layer**: no nulls in primary keys; referential integrity across order → customer
- **Gold layer**: business checks — total revenue ties to fact_orders sum; SCD Type 2 has at most one current record per natural key
- **Schema drift**: compatibility check on schema registry blocks bad producers at the source

---

## What "done" looks like

`projects/P3/` folder:
- `docker-compose.yaml` — Postgres + Kafka + Schema Registry + Debezium + Spark + Airflow
- `source_data/` — synthetic data gen scripts
- `debezium/` — connector configs
- `spark_ss/` — PySpark consumer
- `dbt/` — models, macros, tests, snapshots
- `airflow/` — DAG
- `dq/` — Great Expectations suite or dbt tests directory
- `schema_evolution_test/` — the test scenario + before/after evidence
- `README.md` — architecture + measurements + 3 lessons learned
- `DESIGN_DECISIONS.md` — all 7 defenses above written out
- `STAR_STORY.md` — your rewritten Republic Services CDC story reframed with today's stack

---

## STAR story derived

Extends your Republic Services CDC work — **STAR story #7** (dbt migration already) gets a companion streaming CDC story:

> _"At Republic Services we had 24-hour batch lag on CRM dimensions. I replaced it with a RabbitMQ + microbatch consumer so dashboards always saw current state. If I were doing it today I'd use Debezium → Kafka → Structured Streaming → Iceberg + dbt snapshots, because [3 reasons]. The trade-off was [X] and what I'd rearchitect now is [Y]."_

Having built this in P3, you can speak to every layer confidently.
