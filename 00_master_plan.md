# Data Engineering Architecture Mastery — 18-Week Plan

**Owner:** Akshay Reddy Chada
**Start date:** 2026-04-22 (Wed)
**End date:** 2026-08-20 (Thu) — 18 weeks / ~123 days
**Target roles:** FAANG Big Tech Senior/Staff DE · Fintech/Trading Lead DE · Analytics Engineering Platform Lead
**Weekly budget:** ~18–20 focused hours (1.5–2 hrs weekday · 4–5 hrs weekend)
**Total:** ~300 hours (adjusted for 2 limited-access windows)

> **Limited-access windows:** Jun 26–29 (partial) and Jul 4–15 (largely no dev environment). Plan is shaped around them — light/reading-only weeks are baked in.

---

## 1. Why this plan looks the way it does

You are not a beginner. You own 75B records/day PySpark pipelines, Kafka consumers across 8 exchanges, a 250 TB Iceberg migration, and a 400-pipeline Airflow/MWAA rollout. The gap between "I built this" and "I can defend every architectural choice under pressure from a Staff-level interviewer" is where this plan lives.

The plan does five things:

1. **Retroactively formalizes what you've already built.** Every existing project becomes STAR stories with numbers, trade-offs articulated, and alternatives you rejected (and why). You don't just _know_ Kafka — you can defend why you didn't pick Pulsar or Kinesis.
2. **Covers the 9 lead-level topics you flagged as must-have:** API ingestion (REST + streaming), FTP loads, SCD + incremental patterns, real-time pipelines, AWS pipelines (batch + real-time), Iceberg + data modeling, Airflow orchestration, and DQ/monitoring/debugging.
3. **Keeps Spark + Kafka depth** despite the ask to "focus less on them." Every FAANG/fintech Lead-DE interview includes a Spark internals + a streaming deep-dive gauntlet. Skipping them would be malpractice. But they're compressed, and **breadth weeks come first**.
4. **Adds breadth for the three target profiles.** FAANG wants system design + distributed internals. Fintech wants exactly-once + cost + regulatory. Analytics Engineering wants dbt/Snowflake/Iceberg + semantic layers. All three, simultaneously.
5. **Ships 6 hands-on projects** so your answers are "I built this," not "I read about this." Two net-new; four formalized extensions of existing ICE work.

### What I recommend you prioritize (if you only had to pick)

You asked if you're missing anything. Here's the honest answer:

- **Highest-ROI for Lead/Staff interviews, ranked:**
  1. **System design fluency** (the 6-step framework, 4 canonical DE patterns, cold). _You cannot pass Round 2/3 without this._ — Weeks 15–16
  2. **STAR stories, 8 of them, rehearsed out loud** — Weeks throughout, Sat slots
  3. **Spark internals + tuning depth** — Weeks 6–7 (yes, still critical)
  4. **Kafka internals + streaming trade-offs** — Weeks 9–10
  5. **CDC + lakehouse (Iceberg)** — Weeks 3, 14
  6. **Airflow internals + orchestration patterns** — Week 13
  7. **DQ + observability + debugging playbook** — Week 14
  8. **Snowflake defense + cost** — Week 11
  9. **dbt + semantic layer** — Week 12 (reading-only window)
  10. **Ingestion patterns (API, FTP, SSE, webhooks)** — Week 4 (new)
  11. **AWS-native pipelines (Glue, Kinesis, MSK, DMS, Step Functions)** — Week 5 (new)

- **Things I intentionally de-prioritized** (you can skip if short on time):
  - Flink _hands-on_ (keep the theory, skip the lab — you won't ship Flink in most roles)
  - Trino _deployment_ (theory only — Trino is a decision-matrix interview topic, not a hands-on one)
  - Prefect/Dagster _coding_ (POV only — you'll be defending Airflow choices, not switching)
  - Snowflake Cortex _beyond_ your resume story (you already have it)

- **One thing worth adding if you have spare cycles:** a tiny private-LLM-on-DE-data side project (P6 stretch). Staff DE interviews are increasingly asking "what does AI change for DE?" Having a demo answers it.

---

## 2. The eight phases

| Phase | Weeks | Theme | Checkpoint |
|---|---|---|---|
| **1. Architecture Foundations** | 1–3 | Architecting mindset, data modeling (Kimball, Data Vault, Lakehouse), SCD + incremental patterns, pipeline types, trade-offs | Whiteboard any DE architecture from first principles |
| **2. Ingestion & AWS-Native** | 4–5 | REST API/SSE/WebSocket/FTP patterns, connectors, AWS pipelines (Glue, Kinesis, MSK, DMS, Step Functions, EventBridge) | Build one AWS batch + one AWS real-time pipeline end-to-end |
| **3. Spark Deep Mastery** | 6–8 | PySpark internals (Catalyst, Tungsten, AQE), tuning, skew, shuffle, Delta + Databricks, streaming intro | Read any Spark plan; tune skew 3 ways; ship Project P2 |
| **4. Streaming Deep Mastery** | 9–10 | Kafka internals, Structured Streaming + Flink, exactly-once, watermarking | Design a streaming system for any throughput/latency/cost combo; ship Project P1 |
| **5. Warehouse & Lakehouse** | 11–12 | Snowflake advanced, dbt advanced, Iceberg + Trino, semantic layer _(Week 12 = reading-only, limited window)_ | Defend Snowflake/dbt/Iceberg choices; STAR #1 + #6 + #7 written |
| **6. Orchestration, Quality, Debugging** | 13–14 | Airflow depth, MWAA, Dagster POV, Great Expectations/Soda, OpenLineage, CDC patterns, debugging playbook | Ship Projects P3 + P5; debugging playbook complete |
| **7. System Design + Interview Prep** | 15–17 | DE system design patterns (4 core + 3 advanced), behavioral STAR, SQL/coding, 3 mocks | Pass 3 system-design mocks + all 8 STARs drilled cold |
| **8. Final Sprint** | 18 | One clean pass, resume polish, gap-closing, LinkedIn posts | Interview-ready across all rounds |

---

## 3. Checkpoints — the "am I good?" tests

### Checkpoint 1 (end of Week 3) — Architecture
- [ ] Whiteboard 3 architectures cold: Netflix watch history, Uber real-time analytics, a fraud detection pipeline
- [ ] For each, articulate: ingestion, storage, compute, serving, tools chosen, 2 alternatives rejected, failure modes
- [ ] Explain Kimball vs Data Vault vs One Big Table vs Activity Schema with when-to-use-each
- [ ] Defend Lambda vs Kappa vs Lakehouse medallion with concrete examples
- [ ] Recall all 7 SCD types and 6 incremental-load patterns cold

### Checkpoint 2 (end of Week 5) — Ingestion & AWS
- [ ] Build a REST API ingestion with OAuth2, pagination, rate-limit, retries — working in notebook
- [ ] Wire an SFTP → S3 → Glue pipeline with event-driven trigger
- [ ] Compare Step Functions vs Airflow vs EventBridge across 3 scenarios
- [ ] Lab-ship: one AWS batch pipeline + one AWS real-time pipeline (Kinesis or MSK)

### Checkpoint 3 (end of Week 8) — Spark
- [ ] Read any Spark physical plan and identify bottleneck
- [ ] Tune a skewed join 3 ways (salting, AQE skew join, broadcast, bucketing) and explain trade-offs
- [ ] Explain executor memory model (overhead, storage, execution, user) and when each is the bottleneck
- [ ] Design a Spark job for a 500B-row join with 200 GB executor memory
- [ ] Ship **Project P2** (batch skew mitigation)

### Checkpoint 4 (end of Week 10) — Streaming
- [ ] Explain Kafka's ISR / leader election / rebalance protocol from memory
- [ ] Compare Spark Structured Streaming vs Flink across: latency, state, exactly-once, watermarking, back-pressure
- [ ] Design an exactly-once pipeline from Kafka → transform → Iceberg with failure modes mapped
- [ ] Ship **Project P1** (real-time clickstream)

### Checkpoint 5 (end of Week 12) — Warehouse & Lakehouse
- [ ] Explain Snowflake's architecture end-to-end: FDN, MPP, micro-partitions, cache tiers, automatic clustering
- [ ] Write a dbt incremental model with `merge`, `insert_overwrite`, and `append`; know when to use each
- [ ] Design a semantic layer and know MetricFlow/LookML trade-offs
- [ ] Compare Iceberg vs Delta vs Hudi across 6 dimensions
- [ ] STAR #1 (billing automation), #6 (Cortex agents), #7 (dbt migration) drafted

### Checkpoint 6 (end of Week 14) — Orchestration & Quality
- [ ] Walk the Airflow scheduler loop in 2 minutes; explain executor types cold
- [ ] Design a CDC ingestion with idempotency, watermarking, late-arrival handling, and monitoring
- [ ] Articulate data contracts vs schema registry vs DQ tests — when each fails
- [ ] Debugging playbook complete (8 scenarios, each 5-step RCA + prevention)
- [ ] Ship **Projects P3** (CDC lakehouse) + **P5** (DQ framework)

### Checkpoint 7 (end of Week 17) — Interview
- [ ] Pass 3 system-design mocks (1 batch-heavy, 1 streaming-heavy, 1 hybrid)
- [ ] All 8 STAR stories drilled cold (billing, MWAA, GBO Kafka, RCA, Iceberg, Cortex agents, dbt migration, GenAI failure detector)
- [ ] Pass a 45-min SQL round (window functions, CTEs, gaps-and-islands, sessionization)
- [ ] Pass 2 coding rounds (data-heavy Python: generators, context managers, functools, concurrency)

### Checkpoint 8 (end of Week 18) — Ready
- [ ] One clean pass through all flashcards / notes
- [ ] One full-length end-to-end mock (1 SQL + 1 coding + 1 system design + 1 behavioral)
- [ ] Resume updated with quantifiable impact from every project
- [ ] LinkedIn posts published for visibility

---

## 4. Daily cadence — what 1.5–2 hours actually looks like

Mon–Fri slots (choose one pattern per day):

- **Pattern A — Study-heavy (Mon/Wed):** 45 min reading/video · 30 min notes · 30 min small exercise
- **Pattern B — Build-heavy (Tue/Thu):** 15 min theory refresh · 90 min hands-on coding on the active project
- **Pattern C — Review-heavy (Fri):** 60 min spaced-repetition flashcards · 45 min mock-question answering out loud

Sat/Sun slots (4–5 hrs each):

- **Saturday — Project day:** 4–5 hr uninterrupted on active project brief (biggest lever for retention)
- **Sunday — Architecture & review:** 2 hr system-design whiteboard drill · 1 hr week review + next-week setup · 1 hr flashcards

**Non-negotiables:**
- Keep `notes/` per week with your own distilled explanations (Feynman — if you can't explain it, you don't know it)
- End every week with a 10–30 item flashcard dump
- Record yourself answering 2 questions out loud each week (painful, highest ROI)

**During limited-access windows** (Jun 26–29 partial; Jul 4–15 no dev env): switch to Pattern A all week, use Sat/Sun for writing (STAR stories, decision trees, architecture memos) rather than building.

---

## 5. The 6 projects

| # | Project | Phase shipped | Primary tools | Leverages existing ICE work? |
|---|---|---|---|---|
| **P1** | Real-time clickstream analytics platform | Week 10 | Kafka + Spark Structured Streaming + Iceberg + Snowflake + Airflow | Extends your Kafka/GBO work |
| **P2** | Large-scale batch pipeline with data-skew mitigation | Week 8 | PySpark on Databricks + Delta/Iceberg + AQE | Extends your 75B Pillar pipeline |
| **P3** | CDC-based analytics lakehouse | Week 14 | Debezium/Kafka + dbt + Snowflake + Airflow + Iceberg | Extends your RabbitMQ CDC work |
| **P4** | User watch-history + recommendation feed (system-design spec only) | Week 15 | Whiteboard + deep written spec | Net new — mirrors LinkedIn Round 2 Q6 |
| **P5** | Data quality + observability framework | Week 14 | Great Expectations + Airflow + Soda + OpenLineage | Extends your governance work at Republic |
| **P6** | Multi-tenant feature store (stretch) | Week 16 if time | Iceberg + Trino + Redis + Python | Stretch — for FAANG ML adjacency |

Each has its own brief in `projects/`.

---

## 6. Weekly files

| Week | Dates | Phase | Note | File |
|---|---|---|---|---|
| 1 | Apr 22 – Apr 26 | 1 | ⚠️ Starts Wed — 5-day week compressed to Wed–Sun | [week_01_architecture_foundations.md](weekly/week_01_architecture_foundations.md) |
| 2 | Apr 27 – May 3 | 1 | SCD + incremental loads | [week_02_dimensional_modeling.md](weekly/week_02_dimensional_modeling.md) |
| 3 | May 4 – May 10 | 1 | | [week_03_lakehouse_modeling.md](weekly/week_03_lakehouse_modeling.md) |
| 4 | May 11 – May 17 | 2 | **NEW — ingestion patterns** | [week_04_ingestion_patterns.md](weekly/week_04_ingestion_patterns.md) |
| 5 | May 18 – May 24 | 2 | **NEW — AWS-native pipelines** | [week_05_aws_native_pipelines.md](weekly/week_05_aws_native_pipelines.md) |
| 6 | May 25 – May 31 | 3 | | [week_06_pyspark_internals.md](weekly/week_06_pyspark_internals.md) |
| 7 | Jun 1 – Jun 7 | 3 | | [week_07_spark_tuning_optimization.md](weekly/week_07_spark_tuning_optimization.md) |
| 8 | Jun 8 – Jun 14 | 3 | | [week_08_databricks_delta_streaming_intro.md](weekly/week_08_databricks_delta_streaming_intro.md) |
| 9 | Jun 15 – Jun 21 | 4 | | [week_09_kafka_deep_dive.md](weekly/week_09_kafka_deep_dive.md) |
| 10 | Jun 22 – Jun 28 | 4 | ⚠️ Jun 26–29 partial-limited | [week_10_streaming_patterns_flink.md](weekly/week_10_streaming_patterns_flink.md) |
| 11 | Jun 29 – Jul 5 | 5 | ⚠️ Jun 29 + Jul 4 limited; LIGHT | [week_11_snowflake_advanced.md](weekly/week_11_snowflake_advanced.md) |
| 12 | Jul 6 – Jul 12 | 5 | ⚠️ **READING-ONLY WEEK** (Jul 6–12 fully limited) | [week_12_dbt_advanced_iceberg_trino.md](weekly/week_12_dbt_advanced_iceberg_trino.md) |
| 13 | Jul 13 – Jul 19 | 6 | ⚠️ Jul 13–15 limited; LIGHT START | [week_13_orchestration_airflow.md](weekly/week_13_orchestration_airflow.md) |
| 14 | Jul 20 – Jul 26 | 6 | Heavy — ship P3 + P5 | [week_14_data_quality_observability_cdc.md](weekly/week_14_data_quality_observability_cdc.md) |
| 15 | Jul 27 – Aug 2 | 7 | | [week_15_system_design_core.md](weekly/week_15_system_design_core.md) |
| 16 | Aug 3 – Aug 9 | 7 | | [week_16_system_design_advanced.md](weekly/week_16_system_design_advanced.md) |
| 17 | Aug 10 – Aug 16 | 7 | | [week_17_behavioral_sql_mock.md](weekly/week_17_behavioral_sql_mock.md) |
| 18 | Aug 17 – Aug 20 | 8 | 4-day close (Mon–Thu) | [week_18_final_sprint.md](weekly/week_18_final_sprint.md) |

### How the limited-access windows are handled

- **Jun 26–29 (partial):** Week 10 budget trimmed to ~15 hrs; Saturday (Jun 27) reading-only. Catch up Sun (Jun 28) and Tue (Jun 30).
- **Jul 4 (Sat, holiday):** Week 11 trimmed; Jul 4 rest day.
- **Jul 6–12 (fully limited):** Week 12 converted to reading + writing + flashcards only. No labs, no projects. Use the time for STAR #7 + decision trees + mental-model review. P3 deferred to Week 13–14.
- **Jul 13–15 (limited):** Week 13 light start (reading Mon–Wed), hands-on resumes Thu–Sun.

---

## 7. LinkedIn interview capture — mapping rounds to weeks

**Round 1 – Technical**

| Question | Mapped to |
|---|---|
| Q1. SQL top-10 users by watch hours | Week 17 (SQL drill — window functions + aggregation) |
| Q2. Click-stream per-session duration by device | Week 10 (sessionization) + Week 17 SQL drill |
| Q3. Batch vs stream on video platform | Week 1 + Week 10 + Project P1 |
| Q4. Optimize Spark with skew + shuffle | Week 7 + Project P2 |
| Q5. Watch-history data model at global scale | Week 2 + Week 3 + Project P4 |

**Round 2 – Technical (Advanced)**

| Question | Mapped to |
|---|---|
| Q1. Real-time viewing events pipeline | Week 9–10 + Project P1 |
| Q2. 45min → 3hr job regression — RCA | Week 7 + Week 14 (debugging playbook) |
| Q3. DQ + monitoring streaming + batch | Week 14 + Project P5 |
| Q4. Schema evolution across analytics teams | Week 12 + Week 14 |
| Q5. ETL performance/cost story | STAR — Iceberg migration + Pillar |
| Q6. End-to-end watch-history + recfeed architecture | Week 15–16 + Project P4 |

**Round 3 – Managerial / Behavioral**

| Question | Mapped to |
|---|---|
| Q1. Pipeline performance improvement story | Week 17 STAR drill (Iceberg migration) |
| Q2. Conflict: business speed vs engineering stability | Week 17 STAR drill |
| Q3. Tight-deadline story | Week 17 STAR drill (billing automation) |
| Q4. DQ + reliability + observability at PB scale | Week 14 + Week 17 STAR |

---

## 8. How to use this plan

1. **Sunday evening, 30 min:** Read upcoming week's file. Block calendar. Prep reading materials.
2. **Each day:** Open the week file, execute today's slot. Don't overthink — the plan has done the thinking.
3. **Friday evening:** Do the week's self-check. Failures carry into Saturday buffer.
4. **Saturday:** Project day. Phone off, 4–5 hrs deep work.
5. **Sunday:** 1 hr review + 1 hr system-design drill + next-week prep.
6. **End of each phase:** Take the checkpoint. Don't move on if you fail it — carve time from the next phase.

Built-in buffer: Weeks 10, 11, 12, 13 are intentionally light/reading-weighted around the limited-access windows. Week 14 is heavy to catch up.

---

## 9. What to do when you fall behind

- **Missed a weekday:** Skip it. Do not cram.
- **Missed a weekend:** Reclaim on next weekday mornings (30 min × 5 days).
- **Missed a project deadline:** Project moves into Week 14 buffer. Subsequent projects compress, don't skip.
- **Checkpoint fails:** 3 days of next-week buffer to remediate. If you fail twice, pair with a peer/mentor.

---

## 10. Resources

See [resources.md](resources.md) for the curated reading/watching list, organized by topic and priority.

---

_Last updated: 2026-04-22 · Plan v3 (start date shifted to Apr 22; reading pace calibrated across all weeks; Week 1 compressed to Wed–Sun)_
