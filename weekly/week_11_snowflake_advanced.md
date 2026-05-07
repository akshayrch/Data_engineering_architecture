# Week 11 — Advanced Snowflake: Architecture, Micro-Partitions, Optimization, Cost

**Dates:** Jun 29 – Jul 5, 2026
**Phase:** 5 (Warehouse & Analytics Stack)
**Weekly budget:** ~12 hrs (LIGHT — Jun 29 + Jul 4 limited availability)
**Theme:** _You've built on Snowflake for years. Now defend every query and cost decision like you designed the FDN yourself._

> **⚠️ Limited-access alert:** Jun 29 (Mon) and Jul 4 (Sat) are limited. Structure the week with reading-heavy days Mon + Sat; hands-on days Tue/Wed/Thu. Compress Sat/Sun if needed.

---

## Why this week matters

Interviewers love Snowflake questions because it's ubiquitous and the internals are well-documented. Expect: micro-partitions + clustering depth, query profile reading, cost attribution, Streams/Tasks/Dynamic Tables, Snowpark vs dbt, zero-copy cloning trade-offs. Your billing automation + Cortex agents stories live here — make sure they're interview-tight.

---

## Learning objectives

- Snowflake FDN (Foundation) architecture: storage layer (micro-partitions), compute (virtual warehouses), cloud services
- Micro-partitions: size, metadata, pruning, clustering depth
- Clustering keys: when to use, when to skip, clustering depth metric, auto-cluster cost
- Query profile: read it top-to-bottom, identify pruning, spilling, remote disk reads
- Streams (CDC on Snowflake tables), Tasks, Dynamic Tables — trade-offs vs dbt
- Snowpark vs dbt vs raw SQL — where each wins
- Cost: credits, auto-suspend, resource monitors, query tagging, chargeback
- Cortex / Arctic / LLM features — enough for the fintech angle

---

## Daily breakdown

> **Pace check:** Snowflake docs pages vary from 5-min reads to 30-min reads. Read the page, don't skim it — Snowflake internals questions reward precision over breadth. Labs here are mostly SQL and query profiles; they're fast. This is a lighter reading week by design (12 hrs budget) — use the slack for the STAR story writing on Saturday, which is the highest-ROI work here.

### Mon (2 hrs) — Architecture + micro-partitions
- 60 min read: Snowflake docs "Key Concepts & Architecture" (full page)
- 45 min read: Snowflake engineering blog on micro-partitions + pruning
- 15 min lab: Create a table, load 100M rows, run `SYSTEM$CLUSTERING_INFORMATION`, read the output

### Tue (2 hrs) — Clustering depth
- 60 min read: Snowflake docs on clustering keys + auto-clustering
- 45 min lab: Same 100M-row table — cluster by date vs date+user. Measure query time for 3 common queries. Check credits consumed.
- 15 min write: When is a clustering key a waste? (tables < 1 TB, uniform scans, high cardinality keys)

### Wed (2 hrs) — Query profile
- 60 min lab: Take 5 of your real Snowflake queries (sanitized from work) and pull query profiles. Narrate each.
- 45 min read: Snowflake docs "Understanding the Query Profile" in full
- 15 min write: Query profile red-flag cheatsheet — partitions scanned > 1M, bytes spilled to remote, JoinOrder operator, etc.

### Thu (1.5 hrs) — Streams, Tasks, Dynamic Tables
- 45 min read: Snowflake docs on Streams + Tasks + Dynamic Tables
- 30 min lab: Build Stream on a table, Task that processes the stream, compare to Dynamic Table equivalent
- 15 min write: Dynamic Tables vs dbt incremental — which wins? (hint: depends on latency + orchestration + team ownership)

### Fri (1.5 hrs) — Cost + governance
- 45 min read: Snowflake docs on resource monitors + query tagging + credit usage views (ACCOUNT_USAGE schema)
- 30 min lab: Write 3 queries against ACCOUNT_USAGE — most expensive queries last 7 days, by warehouse; credits per team; idle warehouses
- 15 min flashcards

### Sat (4.5 hrs) — Billing automation + Cortex story rebuilds
- 2 hrs: **Rebuild your billing automation story for interview.** Your resume line: "multi-stakeholder approval workflow across markets and firms, Python dataframe + SQL pipelines, 80% reduction in manual calc." Now write the technical defense:
  - Data model (rows, grain, SCDs)
  - Pipeline pattern (micro-batch? batch? real-time?)
  - Python/SQL split — what belonged where, why?
  - Approval workflow — stateful, so where did state live?
  - What was the alternative you considered and rejected?
  - How would you rearchitect with Snowflake Dynamic Tables if starting today?
  Save as `billing_automation_story.md` → **STAR story #1**
- 2 hrs: **Cortex agents story.** Resume line: "deployed Snowflake Cortex agents, 90% reduction in ad-hoc requests, migrated Tableau to Streamlit." Build the interview defense:
  - What models did the agents use (Arctic? Claude? Mixtral on Cortex?)
  - How did you train/ground them on your data models?
  - Guardrails / evals?
  - What failure modes did you hit?
  - Why Streamlit instead of keeping Tableau?
  Save as `cortex_agents_story.md` → **STAR story #6**
- 30 min: Flashcards

### Sun (4.5 hrs) — Performance tuning drill + cost optimization drill
- 2 hrs: **Query tuning drill** — take 5 queries from StrataScratch or LeetCode Hard DB. For each:
  - Write naive version
  - Write optimized version with window function or CTE restructure
  - Predict + check query profile differences
- 90 min: **Cost optimization case study** — write a 2-page memo: "how to cut our Snowflake bill 30% in Q1 without touching business functionality." Checklist:
  - Warehouse auto-suspend tuning (was 600s → 60s saved us X%)
  - Resource monitors to cap dev/qa
  - Query tagging + chargeback visibility
  - Clustering review (add where useful, remove auto-cluster on stable tables)
  - Materialized views audit
  - Search optimization service audit
  - File format review (Iceberg vs native)
- 60 min: Week 9 self-check + flashcards

---

## Must-read this week
- **[CORE]** Snowflake docs: Key Concepts, Micro-partitions, Clustering, Query Profile
- **[CORE]** Snowflake docs: Streams, Tasks, Dynamic Tables
- **[CORE]** _Snowflake Definitive Guide_ Ch 2, 4, 9, 14
- **[DEEP]** DataExpert.io — "Advanced Snowflake" (multiple cohorts)
- **[REF]** Snowflake ACCOUNT_USAGE schema docs

---

## Deliverable
- `notes/week_09/`:
  - `snowflake_architecture.md`
  - `clustering_experiment/` (lab results)
  - `query_profile_cheatsheet.md`
  - `streams_tasks_dynamic_tables.md`
  - `cost_optimization_memo.md`
  - `billing_automation_story.md` (STAR #1)
  - `cortex_agents_story.md` (STAR #6)
  - `flashcards.md` (30 cards)

---

## Self-check (Sunday)

1. Explain micro-partitions in 90 seconds — size, content, pruning role
2. When does a clustering key help? When does it hurt?
3. Read this query profile [pick any from your experiments] and tell me the bottleneck
4. Dynamic Tables vs dbt incremental — when do I pick each?
5. My Snowflake bill doubled this month. Walk me through RCA.
6. Streams: append-only vs standard vs insert-only. When does each apply?
7. Snowpark Python — when is it better than dbt + SQL?

Pass = 6/7 cold.

---

## Notes / Discoveries
