# Week 6 — PySpark Internals: Catalyst, Tungsten, Execution Model

**Dates:** May 25 – May 31, 2026
**Phase:** 3 (Spark Deep Mastery)
**Weekly budget:** ~18 hrs
**Theme:** _Read any Spark plan and know exactly what Spark is doing, why, and where it will hurt._

---

## Why this week matters

You _use_ PySpark. Mastery means you can read `.explain(mode='formatted')` output and narrate what every line does — and predict which one is about to go OOM or shuffle 500GB. Staff interviewers will paste a plan in front of you and ask "what's wrong here?"

---

## Learning objectives

- Explain the Spark execution hierarchy: Application → Job → Stage → Task, with boundary triggers
- Catalyst optimizer end-to-end: unresolved logical → resolved logical → optimized logical → physical plans
- Read `.explain(mode='formatted')` and `.explain(mode='codegen')` fluently
- Tungsten: what off-heap memory / whole-stage codegen / UnsafeRow actually buy you
- RDD vs DataFrame vs Dataset — know when each is correct in 2026 (hint: Dataset rarely, RDD almost never)
- Executor memory model: storage, execution, overhead, user memory — know the fractions
- Narrow vs wide transformation — list 15 of each

---

## Daily breakdown

> **Pace check:** Spark TDG chapters run 25–35 pages at ~12 pages/hr = 2–3 hrs each. One chapter per session, not two. Databricks blog posts = 20–30 min each. The Jacek Laskowski internals book is free online and works well as a reference — skim sections, deep-read the parts that surprise you. Labs here are short and essential; don't skip them.

### Mon (2 hrs) — The execution hierarchy
- 75 min read: _Spark: The Definitive Guide_ **Ch 15 only** (Execution — Jobs, Stages, Tasks, ~20-25 pages). **Ch 16 carries to Tuesday pre-work.** Two Spark TDG chapters in 60 min is not achievable with real notes.
- 15 min read: Jaceklaskowski's "Execution" section (skim for terms you don't know from Ch 15)
- 30 min lab: Spin up local Spark shell, run a job with 2 wide transforms. Open Spark UI. Identify jobs, stages, tasks. Take screenshots.

> **Tue pre-work (20 min):** Skim Spark TDG Ch 16 (Spark Runtime). It's shorter and less critical — 20-min skim is sufficient here.

### Tue (2 hrs) — Catalyst optimizer
- 45 min read: Databricks blog "Deep Dive into Spark SQL's Catalyst Optimizer" (~20 min read — read it twice)
- 45 min read: _Spark Definitive Guide_ Ch 17 — **Catalyst section only** (skip to the optimizer sub-section, ~10 pages). This is the actionable part for `.explain()` reading.
- 30 min lab: `df.explain(mode='extended')` — see unresolved → resolved → optimized → physical. Do it for a join with a filter and narrate every transformation the optimizer made.

### Wed (2 hrs) — Tungsten + codegen
- 45 min read: Databricks blog "Project Tungsten: Bringing Apache Spark Closer to Bare Metal" (~20 min — read it twice, it's worth it)
- 30 min read: _High Performance Spark_ Ch 3 — **skim only**: read the section headers and DataFrames vs Datasets comparison (~10 pages of the chapter). This is context, not core.
- 45 min lab: Run `df.explain(mode='codegen')` on a filter+project+sum. Read the generated Java. Disable codegen with `spark.sql.codegen.wholeStage=false` and compare runtime on a 100M-row dataset.

### Thu (1.5 hrs) — Memory model
- 60 min read: Databricks "Tuning Spark" doc — memory management section, in full
- 30 min write: Draw the 4-region executor heap diagram (storage + execution unified, user, overhead). Know the defaults (spark.memory.fraction=0.6, storage.fraction=0.5).

### Fri (1.5 hrs) — Partitions, DAG, task scheduling
- 45 min read: Jaceklaskowski — "TaskScheduler" and "DAGScheduler" chapters (skim)
- 30 min lab: Take a 10GB Parquet, read with default partitioning, count partitions. Change `spark.sql.files.maxPartitionBytes`, re-read, observe. Measure task count.
- 15 min flashcards: 15 cards on today's content

### Sat (4.5 hrs) — Plan-reading gauntlet
Take 5 real queries (give yourself variety — join-heavy, aggregation-heavy, window-function-heavy, UDF-heavy, filter-only) and for each:

1. Write the query
2. Predict the physical plan BEFORE running
3. Run `explain(mode='formatted')`
4. Narrate every operator in writing: what it does, whether it shuffles, whether it broadcasts, whether there's pushdown
5. Where's the bottleneck? What would you change?

Example queries:
- 2 fact-fact join (both 1B rows)
- Fact-dim join where dim is small (100k rows)
- GROUP BY with window function
- UDF filtering a column (notice Python UDFs break codegen — know why)
- Read Parquet with complex predicate

Document all 5 as `plan_reading_drill.md`.

### Sun (4.5 hrs) — Consolidation + DAG deep dive
- 90 min: Read Saturday's drill aloud as if explaining to a peer
- 60 min read: "Spark Core Internals" sections on DAGScheduler + ShuffleManager
- 60 min build: Write a Spark job that forces 2 wide transforms in sequence. Examine `SparkContext.textFile.count()` vs DataFrame plans. Compare.
- 60 min: Week 4 self-check + flashcard consolidation (30 cards for the week)

---

## Must-read this week
- **[CORE]** _Spark: The Definitive Guide_ Ch 15, 16, 17
- **[CORE]** Databricks blog — Catalyst Optimizer deep dive
- **[CORE]** Databricks blog — Project Tungsten
- **[DEEP]** Jaceklaskowski _Internals of Apache Spark_ (free online)
- **[REF]** Spark docs — Configuration page

---

## Deliverable
- `notes/week_04/`:
  - `execution_hierarchy.md` (diagram + narrative)
  - `catalyst_walkthrough.md` (with `.explain` outputs narrated)
  - `tungsten_codegen_experiment/` (lab notebook + measurements)
  - `memory_model_diagram.md`
  - `plan_reading_drill.md` (5 queries, plans, analyses)
  - `flashcards.md` (30 cards)

---

## Self-check (Sunday)

1. Narrate the full Catalyst pipeline from SQL text to physical plan in 90 seconds
2. Draw the executor heap with percentages and explain what each region does
3. What's the difference between a job, stage, and task? What triggers each boundary?
4. Why does a Python UDF typically make a Spark job slower? How do you fix it?
5. What does whole-stage codegen do? When does it get disabled?
6. Which of these is a wide transformation: `map`, `flatMap`, `groupByKey`, `distinct`, `filter`, `join`, `coalesce`, `repartition`? Group correctly.
7. You see 200 stages in your job UI. Your code has 3 `.join()`s and 2 `.groupBy()`s. Something is off. Where do you look first?

Pass = 6/7 cold.

---

## Notes / Discoveries
