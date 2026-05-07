# Week 3 — Lakehouse Modeling: Iceberg, Delta, Hudi + Partitioning Strategy

**Dates:** May 4 – May 10, 2026
**Phase:** 1 (Foundations)
**Weekly budget:** ~18 hrs
**Theme:** _The modeling choices that matter on object storage — partitioning, clustering, file layout, format wars._

---

## Why this week matters

Your resume says "250 TB S3 → Iceberg migration, 40% cost reduction." A staff interviewer will grill you on: why Iceberg over Delta, what partitioning scheme you chose, how you handle small files, hidden partitioning, schema evolution, time travel costs. This week turns that line item into a 30-minute defensible story.

---

## Learning objectives

- Compare Iceberg vs Delta Lake vs Hudi on 8 dimensions (catalog, file format, partitioning, schema evolution, time travel, concurrency, compaction, ecosystem)
- Design partitioning strategy for write-heavy, read-heavy, and mixed workloads
- Explain Iceberg hidden partitioning and why it's game-changing
- Understand metadata trees: manifest list → manifest → data files
- Plan for small-file problem: target file size, compaction strategies, Z-ordering vs hash clustering
- Model a fact table 3 ways on Iceberg with different partitioning and compare query plans

---

## Daily breakdown

> **Pace check:** Iceberg spec text is specification-dense — treat it like code, not narrative. 5–6 pages/hr is normal. Academic papers (~12 pages) are 60–90 min of real work. The Iceberg Definitive Guide is lighter — ~18 pages/hr. Don't stack two heavy sources in the same session.

### Mon (2 hrs) — Iceberg architecture (spec foundation)
- 90 min read: Iceberg spec (iceberg.apache.org/spec) **sections 1–2 only** — this covers the data files, manifests, and snapshot model (the core mental picture). Sections 3–4 (partitioning + inheritance) carry to Tuesday pre-work. The spec is dense; sections 1–2 alone will fill 90 min.
- 30 min: Sketch manifest list → manifest → data file tree in notes from what you've read.

> **Tue pre-work (20 min):** Skim Iceberg spec sections 3–4 (partitioning spec) before starting Tuesday's session.

### Tue (2 hrs) — Iceberg book + Delta Lake
- 45 min read: _Apache Iceberg: The Definitive Guide_ Ch 1–2 (~25 pages, lighter than the spec — this is the accessible companion)
- 45 min read: Delta Lake VLDB 2020 paper (~12 pages — short but dense. 45 min is the right amount.)
- 30 min write: Start 8-dimension comparison matrix (Iceberg vs Delta for now). Add Hudi column after Wednesday.

### Wed (2 hrs) — Partitioning + clustering theory
- 45 min read: _Iceberg Definitive Guide_ Ch 3 (partitioning) + Ch 5 (optimization)
- 45 min build: In local Spark (or Databricks Community), create the same fact table 3 ways:
  - Partition by date (day)
  - Partition by date (month) + Z-order by user_id
  - Hidden partition by `bucket(user_id, 64)` (Iceberg)
- 30 min: Run same 3 queries against each, compare physical plans + file reads

### Thu (1.5 hrs) — Schema evolution + time travel
- 45 min read: Iceberg + Delta schema evolution docs
- 30 min build: Evolve schema (add column, rename, reorder, drop) on Iceberg table; show it works without rewriting
- 15 min write: 2-paragraph note on what exactly fails in a Hive-style table during schema evolution (answer: rename is impossible, reorder breaks readers)

### Fri (1.5 hrs) — Small files + compaction
- 45 min read: Iceberg `rewrite_data_files` procedure docs + Netflix "Iceberg at Netflix" talks on YouTube (30 min talk is enough)
- 30 min build: Create 1000-partition table, deliberately generate small files, then compact with target-file-size=128MB. Measure files before/after.
- 15 min flashcards

### Sat (4.5 hrs) — The Iceberg-migration defense
This Saturday is a single long exercise: **rewrite your 250 TB S3 → Iceberg migration story as if presenting to a staff architect panel.**

Structure:
1. **Context (5 min):** why we were on S3 flat Parquet, what hurt (query scan cost, schema pain, no ACID, no time travel, Hive metastore hell)
2. **Decision matrix (10 min):** Iceberg vs Delta vs Hudi — what were the 3 reasons Iceberg won in your specific context (Snowflake external tables? Trino ecosystem? Vendor neutrality?)
3. **Partitioning design (10 min):** trade date partition + bucket(trade_id, 128). Defend the bucket count math.
4. **Migration mechanics (10 min):** rewrite tables on the fly with `migrate` procedure vs a full rewrite? How did you validate parity?
5. **The 40% cost reduction (15 min):** where did it actually come from? Storage tiering? Fewer scans? Compaction? Be specific — interviewers call this out if numbers don't add up.
6. **What you'd do differently (5 min):** honest retrospective. Maybe: Z-order vs hidden partitioning choice; compaction schedule tuning; catalog choice (Glue vs Nessie vs Polaris).

Write all 6 sections. This becomes your **STAR story #5** for Week 15.

### Sun (4.5 hrs) — Consolidation + Phase 1 Checkpoint
- 90 min: Read Saturday's writeup aloud. Record yourself. Time it (should hit 30–45 min as a full story or 5 min for the compressed version).
- 60 min: Week 3 flashcards (30 cards on Iceberg/Delta/Hudi)
- **2 hrs: Phase 1 Checkpoint from master plan.** Run the 4 checkpoint questions, honestly self-grade. Any fail = budget 1 hr Mon AM to fix before starting Week 4.

---

## Must-read this week
- **[CORE]** _Apache Iceberg: The Definitive Guide_ Ch 1–5
- **[CORE]** Iceberg spec sections 1–4
- **[CORE]** Delta Lake VLDB 2020 paper
- **[DEEP]** DataExpert.io — "Data Modeling on Iceberg" (Analytics Camp Apr 2025)
- **[DEEP]** Netflix "Iceberg at Netflix" YouTube talks (Ryan Blue + Daniel Weeks)

---

## Deliverable
- `notes/week_03/`:
  - `format_comparison.md` (8-dim matrix)
  - `partitioning_experiments/` — 3 partitioned Iceberg tables + query plans + notes
  - `iceberg_migration_story.md` (the Saturday defense — this goes into STAR stories)
  - `schema_evolution_demo/` — reproducible notebook
  - `flashcards.md`

---

## Self-check (Sunday — Phase 1 Checkpoint)

See master plan § Checkpoint 1. All 4 questions must be answerable cold. If any fail, next week has a remediation slot.

Additionally:
1. Explain Iceberg's hidden partitioning in 90 seconds with a concrete example
2. Why is schema-rename impossible on a Hive table but trivial on Iceberg?
3. You have 10 PB Parquet on S3 you want to migrate. Walk the migration plan in 5 min.
4. You see a 10x query slowdown on an Iceberg table — what are the top 5 things to check?

---

## Notes / Discoveries
