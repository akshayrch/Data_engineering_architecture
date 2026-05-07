# Week 12 — Advanced dbt + Iceberg + Trino + Semantic Layer (READING-ONLY WEEK)

**Dates:** Jul 6 – Jul 12, 2026
**Phase:** 5 (Warehouse & Lakehouse — closeout)
**Weekly budget:** ~6–8 hrs (LIMITED ACCESS — Jul 6–12 fully limited, no hands-on work)
**Theme:** _A reading-only week. Absorb the theory. Keep the reps for Week 13._

> **⚠️ Limited-access alert:** Jul 6 – Jul 12 you will NOT have access to a dev environment. This week is structured entirely as **reading + writing + flashcards**. No labs, no projects, no Docker. Hands-on P3 work resumes Week 13. Use this week well — it's the quiet time to build conceptual mastery that the busy weeks never give you.

---

## Why this week matters

The analytics-engineering profile you're targeting demands this. Your resume has "DBT Core" and the Republic Services migration — this week makes it interview-grade.

Trino + Iceberg is the emerging open-source alternative to Snowflake — you need a POV.

Because you can only read this week, you'll lean into the hardest skill many engineers skip: **explaining the internals without running anything**. That's exactly what staff/lead interviews test.

---

## Learning objectives

- dbt internals: how it compiles, how `ref` / `source` / `macros` work, what `run-operation` does
- Incremental materializations deep: `append`, `merge`, `delete+insert`, `insert_overwrite` — know each
- Snapshots (SCD2 in dbt): the algorithm, the limitations, when NOT to use
- Tests: schema tests, singular tests, generic tests, `dbt-expectations`
- Macros: when to abstract, when to stop (rule of 3)
- Exposures, metrics, semantic layer (MetricFlow)
- Trino on Iceberg: connector architecture, cost-based optimizer, pushdown capabilities
- LookML / MetricFlow / Cube — compare semantic layer frameworks
- Republic Services dbt migration STAR story (written from memory — no lab required)

---

## Daily breakdown (all reading + writing)

> **Pace check:** This is a reading-only week with a 6–8 hr budget — the lightest week in the plan by design. dbt docs are well-written and scannable; the incremental models page is dense but short. Read slowly. Writing tasks here are the actual deliverable: the STAR story and the decision trees are the things that get used in interviews, not the reading.


### Mon (1 hr) — dbt compilation model (theory only)
- 45 min read: dbt docs — "How we structure our dbt projects" + "About dbt models"
- 15 min write: _"What happens when I type `dbt run`?"_ — explain scheduler → compile → run lifecycle in 1 page, from memory. Include: manifest.json, DAG construction, ref() resolution, execution order, run_results.json.

### Tue (1.5 hrs) — Incremental materializations (theory)
- 60 min read: dbt docs "Incremental models" in full — read it twice, slow
- 30 min write: **Decision tree** for incremental strategy:
  - `append` when: …
  - `merge` when: …
  - `delete+insert` when: …
  - `insert_overwrite` when: …
  Include edge cases (late data, duplicates, partition pruning, unique_key pitfalls).

### Wed (1 hr) — Snapshots + tests + macros
- 30 min read: dbt docs on snapshots — focus on the SQL the snapshot macro generates + known limitations (no hard deletes, no type-2-with-flag, performance at scale)
- 20 min read: dbt docs on tests + dbt-expectations README
- 10 min write: _"When would I NOT use a dbt snapshot?"_ — 3 scenarios with the alternative pattern

### Thu (1 hr) — Semantic layer
- 30 min read: dbt Semantic Layer (MetricFlow) docs
- 15 min read: Cube.js docs
- 15 min write: 3-column trade-off table: MetricFlow vs Cube vs LookML. Axes: ownership model, query-time vs build-time, consumer API, BI tool support, cost.

### Fri (1 hr) — Trino + Iceberg (theory only)
- 30 min read: Trino docs — Iceberg connector page + "How Trino works" overview
- 20 min read: Starburst blog on Trino vs Spark SQL performance on Iceberg + any Netflix blog on Iceberg + Trino
- 10 min write: **Decision matrix** — Trino vs Spark SQL on Iceberg. Axes: latency profile, concurrency, workload fit (interactive BI vs heavy batch), join-heavy vs scan-heavy, cost model, operational maturity.

### Sat (1.5 hrs) — Republic Services migration STAR
- 90 min write: **Rebuild Republic Services dbt migration story.** No lab possible this week — this is a pure writing exercise done from memory. Resume line: "replaced Informatica IICS with DBT Core, 70% reduction in pipeline execution time." Defense:
  - Why dbt over Matillion / Coalesce / raw SQL?
  - What patterns moved (incremental strategy, lineage, testing)?
  - What did you lose moving off Informatica (push-down? visual? connectors?)
  - 70% reduction — where did it come from? Fewer inter-query waits? Better materialization choices? Parallelism? Fewer full-scans?
  - What scale was this operating at? (Tables, rows, jobs, concurrency)
  - How did you handle the migration cut-over? (blue-green? canary? dual-run?)
  - What would you do differently today?
  Save as `dbt_migration_story.md` → **STAR story #7**

### Sun (1.5 hrs) — Flashcards + mental-model consolidation
- 60 min: Build 40 flashcards covering: dbt compile lifecycle, 4 incremental strategies, snapshot SQL, test types, semantic layer concepts, Trino vs Spark on Iceberg
- 30 min: **Mental-model reread.** Skim Weeks 9, 10, 11 notes. You have zero hands-on this week; compensate by drilling your memory of prior labs. Recall from memory: how Iceberg snapshots work, how Snowflake micro-partitions prune, how Kafka rebalance works. 5 min each, cold.

---

## Must-read this week (all reading, no labs)
- **[CORE]** dbt docs — Incremental Models, Snapshots, Tests, Macros, Semantic Layer
- **[CORE]** Trino docs — Iceberg connector + "How Trino works"
- **[DEEP]** Cube.js vs MetricFlow comparison articles
- **[DEEP]** Starburst / Netflix blogs on Trino on Iceberg at scale
- **[DEEP]** dbt-core GitHub — macros directory (browse on mobile / GitHub web — no clone needed)

---

## Deliverable (writing-only, no code)
- `notes/week_12/`:
  - `dbt_compile_lifecycle.md`
  - `incremental_decision_tree.md`
  - `snapshot_limitations.md`
  - `semantic_layer_comparison.md`
  - `trino_vs_spark_on_iceberg.md`
  - `dbt_migration_story.md` (STAR #7)
  - `flashcards.md` (40 cards)

No project deliverable this week. **Project P3 resumes Week 13** when hands-on access returns.

---

## Self-check (Sunday)

1. `append` vs `merge` vs `delete+insert` vs `insert_overwrite` — when each? (Cold recall.)
2. dbt snapshots — what they do, why they're limited, when to use raw SCD2 SQL instead
3. How does `ref()` resolve? What happens at compile vs run time?
4. Trino vs Spark SQL on Iceberg — pick-each-for scenarios
5. Semantic layer — what's the actual value prop vs views in the warehouse?
6. 70% execution-time reduction migrating off Informatica — what caused it?

Pass = 5/6 cold + STAR #7 drafted.

---

## Note on the limited window

Reading-only weeks are a gift. You rarely get structured time to just _think_ about architecture without a keyboard pulling you back to debugging. Use it. Write in prose, not bullet points. Explain out loud on a walk. When Week 13 begins and hands return, you'll execute faster because the theory is already internalized.

---

## Notes / Discoveries
