# Week 14 — Data Quality, Observability, Debugging + CDC Patterns

**Dates:** Jul 20 – Jul 26, 2026
**Phase:** 6 (Orchestration, Quality, Debugging — closeout)
**Weekly budget:** ~20 hrs (heavy week — first full week back from limited access; ship P3 + P5)
**Theme:** _If you ship data, you are responsible for its quality. Here's how to prove you are. Plus: how to debug what goes wrong._

---

## Why this week matters

LinkedIn Round 2 Q3 (DQ + monitoring streaming + batch), Round 2 Q4 (schema evolution), Round 3 Q4 (reliability + observability at PB scale). This week owns all three. Plus: Ship Projects P3 + P5 so you head into system-design weeks with 5/6 projects done.

---

## Learning objectives

- DQ frameworks: Great Expectations, Soda, dbt tests, Monte Carlo (observability), Datafold (diff)
- Data contracts: what they are, who owns them, how they fail in practice
- Schema registry + schema evolution compatibility modes — how this interacts with DQ
- CDC patterns: log-based (Debezium) vs query-based (JDBC connector) vs trigger-based — trade-offs
- CDC downstream patterns: staged delta tables, merge-into, SCD2 + timeline
- Observability: OpenLineage, Marquez, dbt artifacts, Airflow logs, DataDog/Grafana integration
- Incident response + postmortem templates

---

## Daily breakdown

> **Pace check:** This is the heaviest shipping week (P3 + P5 due). Reading slots are intentionally shorter this week — docs and blog posts, not dense books. Prioritize the lab/build time over squeezing in extra reading. If a Monday/Tuesday reading runs long, cut it short — you need the Saturday/Sunday for project delivery.

### Mon (2 hrs) — DQ landscape
- 45 min read: Great Expectations docs "Core Concepts"
- 30 min read: Soda core docs
- 30 min read: dbt tests + dbt-expectations + elementary-data (observability on dbt)
- 15 min write: DQ tool comparison — when GE, when Soda, when dbt-only, when you need Monte Carlo

### Tue (2 hrs) — Data contracts
- 45 min read: Chad Sanderson's blog posts on data contracts (full series on Substack)
- 45 min read: Google's "Data Mesh" sections on contracts
- 30 min write: Contract template — what schema, what SLA, what DQ checks, what owner, what consumer, what break-glass process

### Wed (2 hrs) — CDC deep dive
- 60 min read: Debezium docs + "How CDC Works" Confluent blog series
- 45 min read: Fivetran/Airbyte docs on incremental sync — contrast with log-based
- 15 min write: CDC pattern comparison (log vs query vs trigger) — trade-offs including: ordering guarantees, load on source, schema change handling, latency

### Thu (1.5 hrs) — Observability
- 45 min read: OpenLineage + Marquez docs
- 30 min read: dbt artifacts (`manifest.json` + `run_results.json`) — how elementary-data / dbt-cloud consume them
- 15 min flashcards

### Fri (1.5 hrs) — Schema evolution in practice
- 45 min read: Confluent Schema Registry compatibility modes (revisit from Week 7) + Buf schema registry posts
- 30 min write: Decision tree — BACKWARD/FORWARD/FULL/NONE — which to set per use case
- 15 min flashcards

### Sat (4.5 hrs) — Ship Project P3 + Start Project P5
- 2 hrs: **Ship Project P3 (CDC Lakehouse).** Final deliverable must include:
  - End-to-end pipeline running: Debezium → Kafka → Spark SS → Iceberg landing → dbt marts
  - Airflow orchestration with SLAs + retries
  - Monitoring: at least 3 DQ checks per table (not-null, row count delta, referential integrity)
  - README with architecture diagram + 3 STAR-able lessons learned
- 2 hrs: **Start Project P5 (DQ & Observability Framework).** Open `projects/P5_dq_observability.md`. Deliverable for today:
  - Great Expectations suite created for P3's marts tables
  - Soda checks YAML for one streaming table
  - OpenLineage integration wired into Airflow DAG
  - First dashboard sketch
- 30 min: Flashcards

### Sun (4.5 hrs) — Ship P5 + Phase 5 Checkpoint
- 2 hrs: Ship P5 — final deliverable:
  - DQ suite running in CI + nightly
  - Lineage graph visible (Marquez UI screenshot)
  - Alerting wired (Slack or email mock)
  - README + "what breaks if you skip this" one-pager
- 90 min: **Phase 5 Checkpoint** from master plan (4 items)
- 60 min: **45 min → 3 hr RCA story rebuild** — you wrote a playbook in Week 5. Now write the actual STAR story (even if hypothetical, grounded in a real incident you lived through). Save as `rca_star.md` → **STAR story #4**

---

## Must-read this week
- **[CORE]** Great Expectations docs
- **[CORE]** Debezium docs + Confluent CDC blog series
- **[CORE]** Chad Sanderson data contracts Substack (pin several posts)
- **[CORE]** OpenLineage + Marquez docs
- **[DEEP]** DataExpert.io — "Change Data Capture (CDC) and Analytical Patterns" (Oct 2024) + "Data Quality Patterns" (Community Edition)
- **[DEEP]** Elementary-data docs (dbt observability)

---

## Deliverable
- **Project P3 shipped** (CDC lakehouse end-to-end)
- **Project P5 shipped** (DQ + observability framework on top of P3)
- `notes/week_14/`:
  - `dq_tool_comparison.md`
  - `data_contract_template.md`
  - `cdc_pattern_comparison.md`
  - `schema_evolution_tree.md`
  - `debugging_playbook.md` (the lead-DE debug mental model — see below)
  - `rca_star.md` (STAR #4)
  - `flashcards.md` (30 cards)

## Bonus — Lead-DE debugging playbook (add this Thu)

Replace the Thu slot (1.5 hrs) with a dedicated **debugging playbook** writeup. This is the lead-level skill interviewers test with "walk me through how you debug X."

Write the playbook for each scenario:

1. **Pipeline suddenly slow** → 10-step RCA (from Week 7 — now formalize)
2. **Data quality check fails** → isolate layer (source/transform/target), diff, backfill, prevent
3. **Row count mismatch between source and target** → usual suspects (dedup, filter, join explosion, late arrivals, partition skew, joins on null, timezone, schema drift)
4. **Spark job OOM** → executor sizing, skew, memory fraction, GC, too-wide schema, bad UDF
5. **Streaming consumer lagging** → partition imbalance, slow downstream, GC, back-pressure, watermark stuck
6. **Warehouse query suddenly 10x slower** → plan change, stats staleness, partition pruning lost, spills, warehouse sized wrong
7. **Late data arriving** → source retry buffer, timezone bug, watermark too tight, deliberately late (batch reruns)
8. **Consumer outage caused data gap** → backfill strategy, replay from archive, idempotency test

Each scenario gets: _5 things I check in order_ + _one thing I'd put in place to prevent this next time_. This doc becomes your interview ammunition for Round 3 Q4.

---

## Self-check (Sunday — Phase 5 Checkpoint)

See master plan § Checkpoint 5.

Additionally:
1. DQ in streaming is harder than batch because… (answer: no rerun, no backfill comfort, must do inline or sample)
2. Data contract vs schema registry — who owns what?
3. Debezium + Kafka — how do you handle schema change on the source table?
4. OpenLineage — why care? What does it give you that dbt lineage alone doesn't?
5. Your observability setup catches 80% of issues. What's in the missed 20%? (semantic correctness, freshness tails, cross-table consistency)
6. Design a DQ system for a PB-scale production environment

Pass = checkpoint green + 5/6 cold + all 5 STAR stories drafted.

**Milestone:** By end of Week 12 you have 5 of 6 projects shipped + 5 STAR stories written. You're 75% through. Spend an hour Sunday reviewing progress against the master plan.

---

## Notes / Discoveries
