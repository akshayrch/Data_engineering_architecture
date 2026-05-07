# Week 8 — Databricks, Delta Lake, Structured Streaming Intro

**Dates:** Jun 8 – Jun 14, 2026
**Phase:** 3 (Spark Deep Mastery — closeout)
**Weekly budget:** ~18 hrs (lighter mid-week — buffer for P2 slippage)
**Theme:** _Where Spark actually runs. Delta internals. First touch of Structured Streaming before Phase 3._

---

## Why this week matters

Databricks runs Spark in most shops now — knowing the platform (clusters, Photon, workflows, DLT) is expected. Delta internals are critical if the role uses Databricks. Structured Streaming intro here sets up Phase 3 so Kafka week doesn't start cold.

Buffer week: if P2 slipped in Week 5, Monday/Tuesday of this week absorb it.

---

## Learning objectives

- Databricks cluster types (all-purpose, job, DLT, SQL warehouse) — know cost/use-case for each
- Photon — what it is, when it accelerates, when it doesn't (hint: UDFs)
- Delta transaction log, DELETE/UPDATE/MERGE mechanics, deletion vectors
- DLT (Delta Live Tables) — when it's a win, when it's a trap
- Structured Streaming basics: sources, sinks, triggers, checkpointing, output modes
- Finish Project P2 deliverable + close Phase 2 checkpoint

---

## Daily breakdown

> **Pace check:** This week's reading is a mix of papers (~12 pages each) and docs/blogs. Papers = 45–60 min per paper. Databricks blog posts = 20–30 min. Learning Spark 2e Ch 8 is moderately dense (~30 pages = ~2 hrs with notes); the streaming primer slot below allocates 45 min — skim it for concepts, deep-read it fully over the weekend.

### Mon (2 hrs) — Buffer / P2 closeout
- If P2 not shipped: finish it today. Comparison report + 1-pager lessons-learned.
- If P2 shipped: start Mon content early

### Tue (2 hrs) — Databricks platform
- 60 min read: Databricks architecture docs — Control plane vs Data plane
- 45 min read: Photon announcement blog + Photon VLDB 2022 paper sections 1–3
- 15 min: Spin up a Databricks Community Edition cluster, load a 100M-row dataset, toggle Photon on/off, measure

### Wed (2 hrs) — Delta Lake internals
- 60 min read: Delta Lake VLDB 2020 paper (re-read with the transaction log as focus)
- 45 min read: Databricks docs on Delta transaction log + deletion vectors
- 15 min lab: Create a Delta table, do 10 updates, inspect `_delta_log/` folder, read the JSON files, understand how current snapshot is constructed

### Thu (1.5 hrs) — MERGE mechanics + DLT
- 45 min read: Databricks MERGE INTO docs + Delta docs on deletion vectors
- 30 min read: DLT docs — when to use, when to skip
- 15 min lab: Write a MERGE with 3 WHEN clauses (insert, update, delete) for an SCD Type 2

### Fri (1.5 hrs) — Structured Streaming primer
- 45 min read: _Learning Spark 2e_ Ch 8 (Structured Streaming)
- 30 min read: Spark docs on Structured Streaming programming guide — sections 1–4
- 15 min flashcards: 10 cards on streaming semantics

### Sat (4.5 hrs) — P2 polish + First Structured Streaming app
- 90 min: Polish Project P2 deliverable — write the 2-page README as if an interviewer would read it. Include: problem, approach, 4 variants, numbers, which won and why, and a "what I'd do differently" section.
- 2 hrs: **Build your first Structured Streaming app**: rate source → stateful aggregation → console sink. Then: Kafka source (embedded kafka) → event-time windowed aggregation → Delta sink. Understand watermarking, output modes (append vs update vs complete), checkpoint location.
- 60 min: Read Project P1 brief thoroughly — you start it Week 7

### Sun (4.5 hrs) — Phase 2 Checkpoint
- 2 hrs: **Phase 2 Checkpoint drill** (see master plan). Specifically:
  - Read 3 unknown Spark physical plans and narrate (use internet for examples)
  - Fix a skewed join 3 ways cold
  - Design Spark job for 500B-row join with 200GB cluster — whiteboard
  - Explain executor memory model — record yourself
- 90 min: If any fail, remediate. Otherwise, begin Kafka prep reading (head-start into Week 7)
- 60 min: Week 6 flashcards + consolidate Weeks 4–6 flashcards

---

## Must-read this week
- **[CORE]** _Learning Spark 2e_ Ch 8
- **[CORE]** Delta Lake VLDB paper (re-read)
- **[CORE]** Databricks Photon paper (sections 1–3)
- **[CORE]** Spark Structured Streaming programming guide
- **[DEEP]** Databricks DLT documentation
- **[DEEP]** DataExpert.io — "Databricks Basics" + "Data Lakes with Delta Table" (Jan/May 2025 cohorts)

---

## Deliverable
- **Project P2 shipped** — with 2-page README defending every decision
- First Structured Streaming app in `projects/scratch/streaming_warmup/`
- `notes/week_06/`:
  - `databricks_platform_map.md`
  - `delta_log_walkthrough.md`
  - `merge_scd2.sql`
  - `flashcards.md` (25 cards)

---

## Self-check (Sunday — Phase 2 Checkpoint)

See master plan § Checkpoint 2. All 5 items must pass.

Additionally:
1. Explain the Delta transaction log in 2 min
2. What's deletion vectors and why do they exist?
3. Photon doesn't help with X. What's X and why?
4. Structured Streaming watermark — what it does, what it doesn't do
5. Output modes — append vs update vs complete. When do I use each?

Pass = checkpoint green + 4/5 cold.

---

## Notes / Discoveries
