# Week 2 — Data Modeling + SCD Types + Incremental Load Patterns

**Dates:** Apr 27 – May 3, 2026
**Phase:** 1 (Foundations)
**Weekly budget:** ~18 hrs
**Theme:** _Kimball, Data Vault, OBT, Activity Schema + every SCD type + every incremental load pattern a lead DE needs._

---

## Why this week matters

LinkedIn Round 1 Q5 ("watch history at scale") and Round 2 Q4 ("schema evolution") can't be answered without modeling fundamentals. Your ICE billing automation was a modeling exercise in disguise — this week makes that explicit so you can defend it.

---

## Learning objectives

- Explain Kimball (star/snowflake) end-to-end with a real example
- Model the 7 SCD types and know the storage/query trade-offs for each
- Explain Data Vault 2.0 (Hub/Link/Satellite) and when it beats Kimball
- Know One Big Table (OBT) and Activity Schema as modern alternatives
- Design fact tables: transaction, periodic snapshot, accumulating snapshot, factless — know the difference cold
- Model grain → know the single most important sentence in dimensional modeling: _"Declare the grain of the fact table before anything else."_

---

## Daily breakdown

> **Pace check:** DW Toolkit is dense but more narrative than DDIA — target ~15–18 pages/hr with notes. Each chapter runs 25–40 pages. Plan for one chapter per session, not two. If a chapter runs long, bookmark and carry to pre-work next morning.

### Mon (2 hrs) — Kimball core
- 90 min read: _DW Toolkit_ Ch 1 (intro, ~15 pages) + the **first half of Ch 2** (retail case study through dimension table design, ~15 pages). Stop at the bookmark — don't rush into Ch 2's second half.
- 30 min write: Draft the retail dimension list from what you've read so far. Complete the star schema sketch after finishing Ch 2.

> **Tue pre-work (20–30 min):** Finish DW Toolkit Ch 2 second half before starting Tuesday's session.

### Tue (2 hrs) — SCDs 1–7
- 75 min read: _DW Toolkit_ Ch 7 (SCDs) — focus depth on Types 1, 2, and 6 (these are the interview core). Skim Types 0, 3, 4, 5 for awareness; you'll drill them Friday.
- 45 min build: Write DDL + `MERGE` SQL for SCD Type 1, 2, and 4. Run against a local Postgres or Snowflake trial

### Wed (2 hrs) — Fact tables
- 60 min read: _DW Toolkit_ Ch 3 (inventory, snapshot/accumulating facts, ~25 pages) — **Ch 3 only**. Ch 6 is reference; skim its intro (5 min) and use it Thursday if time allows.
- 45 min build: Model your **ICE billing automation** as a star schema. Be honest about what you actually modeled there. Grain statement required.
- 15 min write: Draft alternatives question for billing model (Data Vault + OBT). Full comparison carries to weekend.

### Thu (1.5 hrs) — Data Vault 2.0 + OBT + Activity Schema
- 45 min read: DV2 summary (Data Vault Guru sample + Hans Hultgren blog posts)
- 30 min write: Draw Hub/Link/Satellite for the billing domain. OBT + Activity Schema comparison.
- 15 min flashcards: 10 cards

### Fri (1.5 hrs) — **Incremental Load Patterns (the lead-DE backbone)**
This is the module that plugs into every project you'll ever touch. Memorize all 6 patterns.

- 45 min read + write: Document each pattern with a 1-page decision write-up:

  1. **Full refresh (truncate + reload)** — only when table is tiny, upstream is sometimes inconsistent, or target is easy to rebuild. Know when it's right.

  2. **Append-only (pure insert)** — for immutable event data. Partition by date, never update. Idempotency via dedup on event_id.

  3. **Watermark-based incremental (high-water mark pattern)** — source has a monotonic column (updated_at, id, lsn). Store last-processed value, filter `WHERE col > bookmark`, persist new watermark atomically after successful load. Covers 60% of real-world incrementals.

  4. **MERGE / UPSERT** — target has business key, source gives you current state. `MERGE INTO target USING staging ON key WHEN MATCHED UPDATE WHEN NOT MATCHED INSERT`. Snowflake, Iceberg, Delta, BigQuery all support. Handles late-arriving data.

  5. **Delete + Insert (idempotent re-process)** — partition-based reprocessing. Overwrite only the affected partitions. Pattern: `DELETE WHERE partition_date = X; INSERT ...`. Perfect for day-level backfills.

  6. **CDC-driven merge (log-based)** — source streams c/u/d events (Debezium / DMS). Downstream applies changes in lsn order. Know: handling out-of-order events, tombstones for deletes, compaction.

- 30 min: Decision tree — "given these 6 source characteristics, which pattern?" (volume, mutability, lateness, business key, target format, idempotency requirement)
- 15 min flashcards: 10 cards on patterns

### Fri-evening (30 min add-on) — **SCD Types deep dive (all 7)**
Memorize all 7 SCD types cold. Know when interviewers expect the "modern" answer:
- **Type 0** — retain original, never change (e.g., birth date)
- **Type 1** — overwrite, no history
- **Type 2** — add new row with effective/end dates + current flag (MOST COMMON)
- **Type 3** — add prior-value column (limited history)
- **Type 4** — history table + current table (lead DE pattern for hot dimensions)
- **Type 5** — Type 4 + mini-dimension outrigger
- **Type 6** — hybrid 1+2+3 (current col + effective dates + prior col)

For each: one-line trade-off + SQL sketch. Document in `scd_types.md`.

### Sat (4.5 hrs) — Modeling a user watch-history at scale (LinkedIn Round 1 Q5)
- Full deliverable: Design 3 different models for user watch-history at global scale — 500M users, 5B events/day
  1. **Kimball star** — dim_user, dim_video, fact_watch_event (grain = one watch session)
  2. **Data Vault** — Hub_User, Hub_Video, Link_UserVideo, Sat_UserProfile, Sat_WatchEvent
  3. **OBT / Activity Schema** — one wide events table partitioned by date, clustered by user_id
- For each: DDL, 3 sample queries (count by device, retention by week, top-N last 30 days), pros/cons table
- Write a 5-min script for each model — you'll use this in mocks Week 15

### Sun (4.5 hrs) — Consolidation + SCD Type 2 deep dive
- 60 min: Read Kimball Ch 7 again with new perspective
- 90 min build: Implement SCD Type 2 ingestion from a CDC feed to a Kimball dimension — end to end SQL (MERGE with hash comparison, current-flag handling, effective dates). Test idempotency.
- 60 min: Record yourself (phone) answering "design a watch-history data model for global scale" in 5 min. Listen back. Note filler words, hand-waves, gaps.
- 60 min: Week 2 self-check + flashcard review

---

## Must-read this week
- **[CORE]** _Data Warehouse Toolkit_ Ch 1, 2, 3, 6, 7
- **[CORE]** Kimball's 4-step dimensional modeling process (memorize)
- **[DEEP]** DataExpert.io — "Dimensional Data Modeling" + "Fact Data Modeling" (Community Edition cohort)
- **[REF]** Hans Hultgren on Data Vault — free blog archives

---

## Deliverable
- `notes/week_02/`:
  - `kimball_retail.md` (retail star redrawn from memory)
  - `billing_starschema.md` (your ICE billing in star + DV + OBT form)
  - `watch_history_3models.md` (the Saturday deliverable)
  - `scd_type2_impl.sql` (working SCD2 MERGE)
  - `scd_types.md` (all 7 types with sketches)
  - `incremental_load_patterns.md` (all 6 patterns + decision tree)
  - `flashcards.md`

---

## Self-check (Sunday)

1. In 60 seconds, run Kimball's 4-step dimensional modeling process (business process → grain → dimensions → facts)
2. Give the single-sentence definition of "grain" and why it matters
3. Explain SCD Type 2 vs Type 6, including storage overhead and query complexity
4. When is Data Vault worth the complexity? Give 2 concrete examples
5. Why do modern platforms push toward OBT? What does Kimball do that OBT doesn't?
6. Model a Netflix watch-history in 3 forms — pick one and defend
7. What's the difference between a _transaction fact_ and an _accumulating snapshot fact_? Give examples.
8. All 7 SCD types — name each with a 1-line use case cold
9. Walk through the 6 incremental load patterns — name each + one concrete scenario
10. A source table has an `updated_at` column, 10M rows, I need to sync to warehouse every hour — which pattern? Why? Write the SQL sketch.

Pass = answer 8/10 cold.

---

## Notes / Discoveries
