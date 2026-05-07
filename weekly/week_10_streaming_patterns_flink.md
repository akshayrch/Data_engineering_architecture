# Week 10 — Streaming Patterns: Structured Streaming + Flink, Watermarks, State

**Dates:** Jun 22 – Jun 28, 2026
**Phase:** 4 (Streaming Deep Mastery — closeout)
**Weekly budget:** ~15 hrs (Jun 26–29 partial-limited — compress Sat/Sun)
**Theme:** _Streaming theory (watermarks, windows, state, exactly-once) applied in two engines, plus Project P1 ship._

> **⚠️ Limited-access alert:** Jun 26–29 has reduced availability. Front-load this week Mon–Thu. Ship P1 by Friday evening; Saturday + Sunday are light closeout only.

---

## Why this week matters

Streaming theory is where most engineers stall. This week is where you get past it.

You'll finally deeply understand:
- Why event time matters more than processing time
- Why watermarks are a compromise, not a truth
- Why Flink is winning state-heavy workloads and Spark Streaming is winning simpler ones
- How exactly-once actually works with two-phase commit

And you'll ship Project P1 (clickstream analytics platform) end-to-end.

---

## Learning objectives

- Event time vs processing time vs ingestion time — know the distinction cold
- Watermarks: what they are, how Flink + Spark compute them, how to tune lateness
- Windows: tumbling, sliding, session. Alignment, offsets, trigger interval vs window length
- State backends: in-memory vs RocksDB; state TTL; state size impact on throughput
- Exactly-once in streaming: 2PC sinks, Flink's checkpointing, Spark's micro-batch + idempotent sinks
- Spark Structured Streaming vs Flink — choose for any given workload
- Backpressure + rate limiting
- Late-arriving data strategies

---

## Daily breakdown

> **Pace check:** Academic papers = 8–12 pages/hr for dense VLDB-style writing. The Dataflow paper is 15 pages — budget 90 min for a proper first read. Learning Spark Ch 8 is ~30 pages = a full 2-hr session at normal pace; the 60-min slot below is a focused second pass (you previewed it in Week 8). Flink docs sections = 30–45 min each.

### Mon (2 hrs) — The Dataflow Model
- 90 min read: Google "Dataflow Model" VLDB 2015 paper (~15 pages). **Approach:** skim abstract + intro + conclusions first (10 min to get the shape), then deep-read sections 2–4 on the What/Where/When/How model (80 min). Sections 5–6 (implementation) are reference — mark them, don't block on them during the first read.
- 30 min write: Draw the 4-quadrant "what / where / when / how" framework from the paper

### Tue (2 hrs) — Spark Structured Streaming depth
- 60 min read: _Learning Spark 2e_ Ch 8 (revisit) + Databricks blog "Arbitrary Stateful Processing in Structured Streaming"
- 45 min lab: Build a stateful aggregation with `mapGroupsWithState` / `flatMapGroupsWithState`. Understand checkpointing.
- 15 min read: Structured Streaming Programming Guide — "Managing Late Data and Watermarking"

### Wed (2 hrs) — Flink fundamentals
- 60 min read: Flink docs — "Stateful Stream Processing" + "Time and Watermarks" sections
- 45 min lab: Docker-compose Flink cluster, run the WordCount example, then adapt to windowed counts over Kafka input
- 30 min read: Flink "Exactly-Once with Two-Phase Commit" docs — understand `TwoPhaseCommitSinkFunction`

### Thu (1.5 hrs) — State backends + checkpointing
- 45 min read: Flink docs on state backends (memory / fs / rocksdb)
- 30 min read: "Checkpointing" docs + alignment vs unaligned checkpoints
- 15 min write: When do I pick RocksDB state backend? When is memory state fine?

### Fri (1.5 hrs) — Comparison + deciding
- 45 min read: _Streaming Systems_ book Ch 3–4 (if have the book; otherwise Tyler Akidau's free blog series "Streaming 101" + "Streaming 102")
- 30 min write: 6-dim comparison matrix: SSS vs Flink — latency, state model, exactly-once, watermarking, backpressure, operational complexity
- 15 min flashcards

### Sat (4.5 hrs) — Ship Project P1

Project P1 must ship today (or latest Sunday). Deliverables for the project:

- Kafka producer generating synthetic clickstream at 10k events/sec
- Spark Structured Streaming consumer with:
  - Event-time windows (5-min tumbling) for per-device session aggregation
  - Watermark handling (10 min late allowance)
  - Deduplication with `dropDuplicates`
  - Sink: Iceberg table (partitioned by date + bucket(user_id))
- **Second consumer in Flink** doing the same thing — prove you can do both
- Comparison README: latency, throughput, code complexity, operational feel
- End-to-end exactly-once verified: kill the producer mid-stream, restart, confirm no dupes/missed

### Sun (4.5 hrs) — P1 polish + Phase 3 Checkpoint
- 2 hrs: Ship P1 README — this is a portfolio piece. Include:
  - Architecture diagram
  - Design decisions with alternatives rejected
  - Throughput/latency measurements
  - What you'd change for 10x scale
  - Operational runbook (how to deploy, how to monitor, how to recover)
- 2 hrs: **Phase 3 Checkpoint drill** — the 4 checkpoint items from master plan. Record yourself on 2 of them.
- 30 min: Flashcards consolidation for Phase 3 (~60 cards total across Weeks 7–8)

---

## Must-read this week
- **[CORE]** Google Dataflow Model paper (VLDB 2015)
- **[CORE]** Tyler Akidau "Streaming 101" + "Streaming 102" free O'Reilly series
- **[CORE]** Flink docs: Stateful Stream Processing + Time & Watermarks + Exactly-Once
- **[CORE]** Databricks "Arbitrary Stateful Processing" blog
- **[DEEP]** _Streaming Systems_ book Ch 1–4 if you own it
- **[DEEP]** DataExpert.io — "Structured Streaming Kafka to Delta Live Table" (Spring 2025) + "Real-time pipelines with Flink and Kafka" (Community Edition)

---

## Deliverable
- **Project P1 shipped** with full README + comparison doc
- `notes/week_08/`:
  - `dataflow_model_notes.md`
  - `sss_vs_flink_matrix.md`
  - `state_backends.md`
  - `watermarks_explained.md`
  - `flashcards.md` (30 cards)

---

## Self-check (Sunday — Phase 3 Checkpoint)

See master plan § Checkpoint 3.

Additionally:
1. In 90 seconds: what is a watermark, how is it computed, what can go wrong?
2. When do I need RocksDB state backend, not memory?
3. How does Flink achieve exactly-once end-to-end with a Kafka sink? (2PC)
4. How does Structured Streaming achieve exactly-once with an idempotent Delta sink?
5. Sessionization with 30-min gap on 100M events/day — SSS or Flink, defend
6. My watermark is never advancing — what's wrong? (partition without data, skew, late source, etc.)
7. Throughput-vs-latency knobs in SSS (trigger interval, micro-batch size)

Pass = Phase 3 checkpoint green + 6/7 cold.

---

## Notes / Discoveries
