# Project P5 — Data Quality & Observability Framework

**Phase:** 5 (Orchestration, Quality, Observability)
**Kickoff week:** 12 · **Ship week:** 12
**Effort:** ~6 hrs
**Primary tools:** Great Expectations + Soda + dbt tests + OpenLineage + Marquez + Airflow
**Leverages existing:** Your governance work at Republic Services
**Deliverable type:** Reusable framework layered on top of Project P3

---

## Why this project

LinkedIn Round 2 Q3 (DQ + monitoring streaming + batch) and Round 3 Q4 (reliability + observability at PB scale) — this project owns both. Plus the Republic Services governance line item on your resume becomes a modern STAR story with Great Expectations + OpenLineage.

---

## Success criteria

You're done when:

- Great Expectations suite runs on 3 critical tables from Project P3 (bronze, silver, gold)
- Soda-core checks run on 1 streaming table (row count SLA, freshness, uniqueness)
- dbt tests exist across every marts model: `not_null`, `unique`, `relationships`, `accepted_values`
- OpenLineage captures lineage across Airflow + dbt + Spark → visualized in Marquez
- Alerting wired: a DQ failure → Slack webhook (can be a local mock)
- 1-page "DQ incident runbook" documented
- README explains: what breaks if each layer is missing; why framework is better than ad-hoc tests

---

## Architecture (DQ overlay on P3)

```
                    ┌─────────────────────────────┐
                    │       Project P3 pipeline    │
                    │  (bronze → silver → gold)    │
                    └──────────┬──────────────────┘
                               │ emits events
                               ↓
         ┌──────────────────────────────────────────────┐
         │  Great Expectations + Soda + dbt tests       │  checkpoints per layer
         └──────────────────────┬───────────────────────┘
                                │ emits lineage/test-results
                                ↓
         ┌──────────────────────────────────────────────┐
         │            OpenLineage → Marquez              │
         └──────────────────────┬───────────────────────┘
                                │ on failure
                                ↓
                    ┌────────────────────────────┐
                    │   Slack webhook / email    │  alerts
                    └────────────────────────────┘
```

---

## Layers of DQ coverage

| Layer | What | Tool | When |
|---|---|---|---|
| **Ingestion (schema)** | Schema compatibility check on producers | Schema Registry compatibility mode | Pre-commit / CI |
| **Bronze (structural)** | Row counts, hourly freshness, no duplicate change events | Soda-core (streaming) | Every micro-batch |
| **Silver (semantic)** | PK uniqueness, RI across tables, SCD2 current-flag invariants | Great Expectations or dbt tests | End of each hourly run |
| **Gold (business)** | Totals tie to facts; metric-level sanity (e.g., no negative order amount) | dbt tests + custom SQL | End of each dbt run |
| **Consumption (SLA)** | Query latency p99, dashboard staleness, cost ceiling | Prometheus + Grafana | Continuous |

---

## DQ checks to implement (sample)

### Bronze layer — Soda-core YAML
```yaml
checks for bronze.cdc_customers:
  - row_count > 0 in last 1 hour
  - freshness using op_ts < 5m
  - duplicate_count(lsn) = 0

checks for bronze.cdc_orders:
  - row_count > 1000 in last 1 hour
  - missing_count(order_id) = 0
```

### Silver layer — Great Expectations
```python
from great_expectations.core.expectation_suite import ExpectationSuite
suite = ExpectationSuite("silver_customers")
suite.add_expectation("expect_column_values_to_be_unique", column="customer_id")
suite.add_expectation("expect_column_values_to_not_be_null", column="customer_id")
suite.add_expectation("expect_column_values_to_match_regex", column="email", regex=r"^[^@]+@[^@]+\.[^@]+$")
suite.add_expectation("expect_table_row_count_to_be_between", min_value=10000)
```

### Gold layer — dbt tests
```yaml
models:
  - name: fct_orders
    tests:
      - dbt_utils.expression_is_true:
          expression: "total_amount >= 0"
      - dbt_utils.equal_rowcount:
          compare_model: ref('stg_orders')
    columns:
      - name: order_id
        tests:
          - not_null
          - unique
      - name: customer_id
        tests:
          - relationships:
              to: ref('dim_customer_current')
              field: customer_id
```

---

## OpenLineage integration

Minimal setup:

- Airflow: `openlineage-airflow` provider, emit lineage events per task
- dbt: `dbt-openlineage` plugin on `dbt run`
- Spark: `openlineage-spark` jar on the Structured Streaming job

Marquez runs in docker-compose. After running P3 end-to-end once, Marquez UI shows lineage graph from Postgres → Kafka → Iceberg → Snowflake end-to-end.

---

## Alerting

Minimum viable: Soda/GE/dbt test failures → Airflow task fails → `on_failure_callback` posts to Slack webhook.

Include in alert:
- What check failed (precise check ID)
- What table + row sample (for quick debugging)
- Lineage link (Marquez URL)
- Last 3 successful runs (context)
- Runbook link

---

## Incident runbook (write this)

1. **Acknowledge** in Slack within 5 min
2. **Isolate**: which layer? bronze (upstream), silver (transform), gold (business)
3. **Decide**: page upstream team OR patch-in-place OR rollback
4. **Communicate**: update `#data-incidents` every 15 min
5. **Fix**: either with a manual backfill, a dbt run, or a producer-side fix
6. **Retro**: within 48 hrs, write: what happened, root cause, detection time, resolution time, preventive action
7. **Close** the loop: add a new test to prevent regression

---

## Deliverable

`projects/P5/` folder:
- `great_expectations/` — suites + checkpoints
- `soda/` — YAML checks + scheduler
- `dbt_tests/` — added to P3's dbt project, symlinked or copied here
- `openlineage_setup/` — docker-compose for Marquez + integration notes
- `alerting/` — Slack webhook mock + Airflow callbacks
- `runbook.md` — the 7-step runbook
- `README.md` — the framework overview + "why this layering matters" essay

---

## Design decisions to defend

- **Why GE + Soda + dbt tests all three?** Each wins at something: dbt for warehouse semantic; GE for richer assertions + data docs; Soda for streaming + SLA freshness
- **Why OpenLineage, not just dbt lineage?** Cross-tool lineage — dbt lineage ends at the warehouse boundary; OpenLineage captures Airflow orchestration + Spark transforms too
- **Why layered checks (bronze/silver/gold) vs one big suite at the end?** Catch failures closest to where they were introduced — root-causing is 10x easier
- **Why runbook?** Alerts without a runbook are noise; on-call rotation needs decision support

---

## STAR story derived

Extends your Republic Services governance work:

> _"At Republic Services I built quality monitoring pipelines that detected schema changes and column-level discrepancies. That system was 60% better at catching issues — but it was bespoke. Today I'd replace the whole thing with Great Expectations + dbt tests + Soda for streaming + OpenLineage for cross-tool lineage, because..."_

You'll have actual screenshots + metrics from P5 to back this up.

---

## The interview payoff

When asked Round 2 Q3 ("DQ + monitoring in streaming + batch architecture") — answer with the layered framework. Show the layering chart above. Give the "each tool at what layer" matrix. That's a staff-level answer.
