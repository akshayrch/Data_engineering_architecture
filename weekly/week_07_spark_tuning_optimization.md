# Week 7 — Spark Tuning, Skew, Shuffle, Joins, Memory

**Dates:** Jun 1 – Jun 7, 2026
**Phase:** 3 (Spark Deep Mastery)
**Weekly budget:** ~18 hrs
**Theme:** _Every knob that matters. Every failure mode. Every fix._

---

## Why this week matters

LinkedIn Round 1 Q4 (skew + shuffle) and Round 2 Q2 (45min → 3hr job RCA) live here. Your Pillar 75B pipeline story — you can't just say "80 parallel tasks, idempotent manifest." You need to answer "why 80, not 40 or 200? what's your shuffle partition strategy? what's the memory fraction? have you tuned executor overhead? how did you handle skew?"

---

## Learning objectives

- Tune shuffle: `spark.sql.shuffle.partitions`, partition size target, AQE dynamic coalesce
- AQE deep: dynamic partition pruning, dynamic join switching, dynamic skew join
- Skew handling — 4 approaches cold (salting, AQE skew join, broadcast, isolation-map-join)
- Join strategies: broadcast hash, shuffle hash, sort-merge, shuffle-replicated NL, broadcast NL. Know when each is chosen
- Bucketing vs partitioning vs Z-order. Know when each wins
- Executor sizing math: cores/executor, memory/executor, overhead, spark.memory.fraction
- Spill, GC tuning, kryo, file format choices

---

## Daily breakdown

> **Pace check:** HPS (High Performance Spark) chapters run 30–40 pages at ~12 pages/hr. One chapter per session. Databricks blog posts are the right density for a 30-min read. Labs here are the real work — don't shortchange them for reading.

### Mon (2 hrs) — Shuffle theory
- 60 min read: _High Performance Spark_ **Ch 4 (Joins) only** (~30 pages — this will fill your reading slot). Ch 7 (Tuning) is spread across Thu/Fri; don't try to stack it here.
- 30 min read: Databricks doc on Adaptive Query Execution (AQE) (~15 min read — read it twice)
- 30 min lab: Run a join with default `spark.sql.shuffle.partitions=200` vs 16 vs 2000 on a 10GB dataset. Measure runtime + task count.

### Tue (2 hrs) — Skew deep dive
- 45 min read: Databricks blog "Adaptive Query Execution: Speeding Up Spark SQL at Runtime" + Chengzhi Zhao post on 5 ways to handle skew (~40 min total for both blogs)
- 75 min lab: Generate a skewed dataset (80% of keys go to 1 value). Run a join 4 ways:
  1. Default (fails or crawls)
  2. AQE skew join on (`spark.sql.adaptive.skewJoin.enabled=true`)
  3. Salting (add random suffix to skewed keys, then drop)
  4. Broadcast (if the other side is small)
- Compare runtime + plan

### Wed (2 hrs) — Join strategies
- 60 min read: _Learning Spark 2e_ Ch 7 on joins (~25 pages — this is a different angle on joins than HPS Ch 4, not redundant). **Skip the HPS Ch 4 re-read** — you did it Monday.
- 30 min lab: Force each join type with hints: `broadcast`, `shuffle_hash`, `merge`. Compare plans + runtime
- 30 min write: Decision tree for which join strategy Spark picks (size thresholds, whether data is sorted, whether bucketed)

### Thu (1.5 hrs) — Bucketing + partitioning + Z-order
- 45 min read: Databricks doc "Best practices for bucketing" + Delta Z-order page
- 30 min lab: Create bucketed table (by join key), re-run the join without shuffle. Confirm zero-shuffle in plan.
- 15 min flashcards: 10 cards

### Fri (1.5 hrs) — Executor sizing + memory tuning
- 45 min read: Cloudera blog "How-to: Tune Your Apache Spark Jobs (Part 1 and 2)" — old but still gold
- 30 min build: Calculate sizing for a hypothetical 500GB dataset on a 200GB cluster. Write the `spark-submit` line. Defend each number.
- 15 min flashcards: 15 cards

### Sat (4.5 hrs) — Project P2 Kickoff: Batch Skew Mitigation Project

Open `projects/P2_batch_skew_mitigation.md`. Start executing.

Outcome by end of Saturday:
- Synthetic skewed dataset generated (1B rows, 80/20 skew)
- Baseline join runs (measure time, plan, shuffle bytes)
- Salting implementation committed
- AQE skew-join implementation committed
- Measurements logged in project README

### Sun (4.5 hrs) — P2 continues + 45min→3hr RCA drill
- 2 hrs: Continue P2 — add broadcast + bucketing variants, build comparison report
- 90 min: **LinkedIn Round 2 Q2 drill** — a nightly job that used to finish in 45 min now takes 3 hrs. Write out your RCA playbook:
  1. Check the Spark UI first — which stage got slower?
  2. Input volume grown? (check file listing + input size metric)
  3. Skew appeared? (check stage duration distribution — task skew = max / median)
  4. Shuffle partition sizing still OK?
  5. Cluster resources — other jobs stealing? Spot interruptions?
  6. Upstream schema change causing CBO to pick a worse plan?
  7. Data locality — are we reading cross-region?
  8. Small files exploded on upstream?
  9. GC time per task increased?
  10. Any code change landed?
- 60 min: Self-check + flashcards

---

## Must-read this week
- **[CORE]** _High Performance Spark_ Ch 4, 7
- **[CORE]** Databricks AQE blog posts (both)
- **[CORE]** _Learning Spark 2e_ Ch 7
- **[CORE]** Cloudera tuning guide parts 1 + 2
- **[DEEP]** DataExpert.io — "Advanced Spark on Databricks" (Jan 2025 cohort)
- **[DEEP]** Databricks bucketing docs

---

## Deliverable
- `projects/P2` baseline + 4 skew-handling variants + comparison report
- `notes/week_05/`:
  - `join_decision_tree.md`
  - `memory_sizing_calc.md`
  - `rca_playbook.md` (the 10-step drill)
  - `flashcards.md` (40 cards)

---

## Self-check (Sunday)

1. Explain AQE's 3 main tricks in 2 min
2. You have a skewed join. Walk me through 4 ways to fix it, with trade-offs
3. When does Spark pick sort-merge vs shuffle-hash vs broadcast-hash?
4. Your executor has 16 cores + 64 GB. What do you set `--executor-cores`, `--executor-memory`, `--executor-memory-overhead`? Defend.
5. Nightly job went from 45 min to 3 hrs. Walk me through RCA in 3 min
6. What's the difference between bucketing and partitioning?
7. When does Z-order beat bucketing?
8. What's `spark.memory.fraction`? What fraction stays under user control?

Pass = 7/8 cold.

---

## Notes / Discoveries
