# Project P1 — Real-Time Clickstream Analytics Platform

**Phase:** 3 (Streaming)
**Kickoff week:** 7 · **Ship week:** 8
**Effort:** ~15 hrs across weeks 7–8
**Primary tools:** Kafka + Spark Structured Streaming + Flink + Iceberg + Snowflake + Airflow
**Leverages existing:** Your Kafka/GBO pipeline work at ICE
**Deliverable type:** Working system + comparison report + architecture writeup

---

## Why this project

LinkedIn Round 2 Q1 is literally "Design a real-time viewing events pipeline." The only thing better than being able to describe such a pipeline is having built one end-to-end. You'll also prove you can use Structured Streaming _and_ Flink on the same problem — a rare skill.

---

## Success criteria

You're done when:

- Kafka producer pumps 10k events/sec of synthetic clickstream continuously
- Two independent consumers exist:
  1. Spark Structured Streaming → Iceberg (partitioned by date + bucket(user_id))
  2. Flink → Iceberg (same target table, different sink)
- Event-time session windowing works (30-min inactivity gap)
- Watermark configured to tolerate 10 min lateness
- Exactly-once verified: kill producer and each consumer mid-stream; restart; confirm no dupes, no missed events
- dbt model computes "top-10 videos by watch hours yesterday" off the Iceberg table, runs daily via Airflow
- README contains architecture diagram, latency/throughput measurements, and a 3-dim comparison (SSS vs Flink)

---

## Architecture (target)

```
                    ┌─────────────────────┐
                    │ Synthetic Producer  │ 10k ev/sec
                    │ (Python, Faker)     │
                    └──────────┬──────────┘
                               ↓
                    ┌─────────────────────┐
                    │   Kafka (3 broker,  │ topic: clickstream
                    │   6 partitions)     │ retention: 7d
                    └──────────┬──────────┘
                               ├─────────────────┐
                               ↓                 ↓
         ┌─────────────────────────────┐  ┌──────────────────────────┐
         │ Spark Structured Streaming  │  │       Apache Flink       │
         │ - window (5m tumbling)      │  │ - window (5m tumbling)   │
         │ - watermark (10m)           │  │ - watermark (10m)        │
         │ - dropDuplicates by event_id│  │ - exactly-once w/ 2PC    │
         └─────────────┬───────────────┘  └────────────┬─────────────┘
                       ↓                               ↓
         ┌──────────────────────────────────────────────────────────┐
         │             Apache Iceberg table                          │
         │   partition: day(event_time), bucket(user_id, 32)         │
         │   catalog: REST / Nessie                                  │
         └──────────────────────────┬───────────────────────────────┘
                                    ↓
                    ┌──────────────────────┐
                    │    dbt / Snowflake   │ daily marts
                    │     (via Airflow)    │ top-N queries
                    └──────────────────────┘
```

---

## Day-by-day plan (Weeks 7–8)

### Week 7 — Stand up the basics
- **W7 Sat**: Kafka + ZK docker-compose. Producer generates 1k events/sec. Single consumer writes raw JSON to S3/local.
- **W7 Sun**: Upgrade producer to realistic click events (user_id, video_id, event_type, device, event_time, event_id UUID). Introduce 5% duplicates + 10% out-of-order intentionally.

### Week 8 — Build + ship
- **Mon–Thu (1.5 hr/day = 6 hr)**: Spark Structured Streaming consumer. Session window. Watermark. Deduplication. Iceberg sink with REST catalog.
- **Fri (1.5 hr)**: Airflow DAG runs daily dbt model.
- **Sat (4.5 hr)**: Flink equivalent consumer — full exactly-once with 2PC Iceberg sink.
- **Sun (4.5 hr)**: Polish README + comparison measurements.

---

## Synthetic schema

```json
{
  "event_id": "uuid",
  "user_id": "int (0..500000)",
  "video_id": "int (0..50000)",
  "event_type": "play | pause | seek | stop | buffer",
  "event_time": "ISO 8601 with ms",
  "device": "ios | android | web | tv",
  "session_id": "uuid (optional; derive if null)",
  "position_seconds": "int"
}
```

Average payload ~250 bytes. At 10k/sec → 2.5 MB/sec per consumer.

---

## Design decisions to defend (interview mode)

Fill these in as you build. Each becomes a 60-second answer in interviews.

- **Why Iceberg, not Delta?** (catalog choice, vendor neutral, Snowflake external table story)
- **Partition by bucket(user_id, 32) — why 32?** (math: 10k/sec × 86400 = 864M/day ÷ 32 buckets / 128MB target files ≈ right size)
- **Watermark 10 min — why not 5, why not 60?** (trade: latency of final answer vs late-event loss)
- **SSS 1-sec vs 5-sec vs 1-min micro-batch trigger?** (throughput vs latency, checkpoint overhead)
- **Why dedupe by event_id at consumer vs trust at-least-once + idempotent MERGE downstream?**
- **SSS vs Flink on this workload — who wins?** (your honest take after running both)

---

## Measurements to capture (the interview ammunition)

- End-to-end p50/p99 latency (producer commit → visible in Iceberg)
- Sustained throughput per consumer
- Checkpoint sizes + durations
- Recovery time after forced failure
- Storage efficiency: bytes ingested vs bytes in Iceberg (compression ratio)
- Small-file count per day (you want < 500)

---

## What "done" looks like

- `projects/P1/` folder contains:
  - `docker-compose.yaml`
  - `producer/` (Python)
  - `sss_consumer/` (PySpark job)
  - `flink_consumer/` (Scala or PyFlink)
  - `dbt/` (single daily mart model)
  - `airflow/` (single DAG)
  - `README.md` (architecture + measurements + comparison)
  - `DESIGN_DECISIONS.md` (the 6 defenses above, written out)

---

## Stretch goals (only if ahead of schedule)

- Schema registry + Avro (skip JSON)
- Backfill story: how would you re-process last 30 days from Kafka? (hint: retention vs dedicated cold archive)
- Cost estimate: monthly run cost at 10k/sec

---

## STAR story derived from this

After shipping, this becomes **STAR story #9** (bonus, beyond the 8 from resume):

> _"I built a real-time clickstream platform from scratch — Kafka producer, both Spark Structured Streaming and Flink consumers against the same Iceberg target, to prove out which engine wins on which axis. The most interesting finding was [your finding]. I'd use [X] for [Y] because [Z]."_
