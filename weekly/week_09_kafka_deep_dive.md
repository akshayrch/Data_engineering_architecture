# Week 9 — Kafka Deep Dive: Internals, ISR, Rebalance, Exactly-Once

**Dates:** Jun 15 – Jun 21, 2026
**Phase:** 4 (Streaming Deep Mastery)
**Weekly budget:** ~18 hrs
**Theme:** _Everything about Kafka that a consumer-builder like you should have known years ago but probably didn't have time to dig into._

---

## Why this week matters

You built Kafka consumers across 40 queues + 8 exchanges. Staff interviewers will ask: Explain the replication protocol. What happens when a broker dies during a producer send? What's the difference between `acks=all` + idempotence + exactly-once semantics? How do rebalances actually work, and why is StickyAssignor better than Range? When should I choose Kafka vs Pulsar vs Kinesis?

---

## Learning objectives

- Kafka architecture: broker, partition, replica, controller, ZK/KRaft
- Replication: ISR, leader election, unclean leader election, min.insync.replicas
- Producer: acks, retries, max.in.flight, idempotence, transactions
- Consumer: group coordinator, consumer offsets topic, rebalance protocol (eager vs cooperative sticky), static membership
- Exactly-once semantics — producer-side vs end-to-end
- Delivery semantics — at-most-once / at-least-once / exactly-once
- Schema registry, Avro/Protobuf, schema evolution compatibility modes
- Kafka vs Kinesis vs Pulsar — 5-dim comparison

---

## Daily breakdown

> **Pace check:** KDG (Kafka Definitive Guide) chapters run 25–35 pages at ~12 pages/hr = 2–3 hrs each to truly absorb. Structure below splits them cleanly: Mon = architecture, Tue = producer (Ch 3), Wed = consumer (Ch 4). Don't stack multiple chapters in the same session. Confluent blog posts = 20–30 min each.

### Mon (2 hrs) — Architecture + replication
- 75 min read: _Kafka: The Definitive Guide 2e_ **Ch 2 in full** (~15 pages — Ch 2 is shorter than the others, this is the right slot for it). Read the broker/replica architecture, ZK/KRaft, controller election. **Skip Ch 3 and Ch 4 today** — they each need their own day.
- 30 min read: Confluent docs on ISR + replication protocol + `min.insync.replicas`
- 15 min write: Draw the broker/partition/replica topology with ISR callouts

### Tue (2 hrs) — Producer internals
- 90 min read: _Definitive Guide_ **Ch 3 in full** (~25–30 pages). This is a proper chapter and needs the full slot. If it runs long, finish in pre-work tomorrow morning.
- 30 min read + write: Confluent blog "Exactly-Once Semantics Are Possible" (Neha Narkhede) + draw the `enable.idempotence=true` sequence diagram.

### Wed (2 hrs) — Consumer + rebalance
- 90 min read: _Definitive Guide_ **Ch 4 in full** (~25–30 pages). Same as Tuesday — Ch 4 is the full session.
- 20 min read: Confluent blog on "Cooperative Rebalance" (Sophie Blee-Goldman) + skim the static membership post
- 10 min write: Rebalance flow: joinGroup → syncGroup → fetch → heartbeat. Write from memory.

### Thu (1.5 hrs) — Exactly-once end-to-end
- 45 min read: _Definitive Guide_ Ch 8 (Exactly-Once Semantics)
- 30 min read: Confluent post on transactional consumer + read_committed isolation
- 15 min write: Why does EOS require cooperation from producer + broker + consumer? Give 1 failure mode that breaks if any piece is missing.

### Fri (1.5 hrs) — Schema registry + format evolution
- 45 min read: Confluent Schema Registry docs + "Compatibility Types" page
- 30 min lab: Spin up local Kafka + Schema Registry (docker-compose). Register Avro schema. Evolve it (BACKWARD, FORWARD, FULL). See what breaks.
- 15 min flashcards: 15 cards

### Sat (4.5 hrs) — Kafka vs alternatives + rebuild the GBO story
- 2 hrs: **Comparison matrix.** Kafka vs Kinesis vs Pulsar vs AWS MSK Serverless vs Redpanda. 8 dimensions: latency, throughput, ordering, durability, pricing model, multi-tenancy, replication, ecosystem.
- 2.5 hrs: **Rebuild your GBO Kafka story for interview.** Your resume says: "fault-tolerant Kafka consumer ETL, 4M daily GBO messages, 40 queues, 8 U.S. exchanges, checkpoint-based offset commits, automated crash recovery." Now write the 5-minute defense:
  - Delivery semantics you chose (at-least-once? why not EOS?)
  - Offset commit strategy (async vs sync; commit-after-process vs commit-before)
  - Rebalance behavior in production (did you tune session.timeout? use sticky?)
  - Crash recovery: what state did you checkpoint beyond offsets?
  - If I said "make this exactly-once now," what would change?
  - What failed mode did you actually hit and how did you debug?

Save as `gbo_kafka_story.md` — this is **STAR story #3**.

### Sun (4.5 hrs) — Sessionization drill + P1 kickoff
- 90 min: **LinkedIn Round 1 Q2 drill** — click-stream per-session duration by device_type. Write the solution in:
  1. Spark SQL (batch)
  2. Spark Structured Streaming (with session window)
  3. Flink (coming next week, but sketch the approach)
  Each with the "sessionization" logic: 30-min inactivity gap closes a session.
- 2 hrs: **Project P1 kickoff** — open `projects/P1_realtime_clickstream.md`. Spin up local Kafka + Zookeeper, produce first synthetic events, build first consumer that lands to Delta/Iceberg
- 60 min: Week 7 self-check + flashcards (35 cards)

---

## Must-read this week
- **[CORE]** _Kafka Definitive Guide 2e_ Ch 3, 4, 8
- **[CORE]** Confluent — Exactly-Once Semantics blog (Neha Narkhede)
- **[CORE]** Confluent — Cooperative Rebalance blog
- **[DEEP]** Jun Rao Kafka Internals YouTube talks
- **[DEEP]** DataExpert.io — "Real Time Data (Spark and Kafka Streaming)" (Jan 2025 cohort)

---

## Deliverable
- `notes/week_07/`:
  - `kafka_topology.md` (drawings)
  - `producer_idempotence.md`
  - `rebalance_flow.md`
  - `eos_end_to_end.md`
  - `kafka_vs_alternatives.md`
  - `gbo_kafka_story.md` (STAR #3)
  - `sessionization.md`
  - `flashcards.md` (35 cards)
- **Project P1 started** — first consumer lands events to Iceberg

---

## Self-check (Sunday)

1. Walk the ISR protocol in 2 min — what happens on leader failure with `min.insync.replicas=2`?
2. Producer: `acks=all` + `enable.idempotence=true` + transactions. What does each one prevent?
3. Consumer rebalance: old eager protocol vs new cooperative. What changed, why?
4. Write the exact code path a message takes for end-to-end exactly-once, naming every config
5. Schema compatibility — BACKWARD vs FORWARD vs FULL — give one example schema change each allows
6. Kafka vs Kinesis vs Pulsar — give one clear reason to pick each
7. Session-window SQL in Structured Streaming — write it
8. Your consumer is lagging. List 10 things to check

Pass = 7/8 cold.

---

## Notes / Discoveries
