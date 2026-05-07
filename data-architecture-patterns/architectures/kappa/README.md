# Kappa Architecture

> One codebase. One pipeline. Reprocess from the log when you need to fix history.

---

## What it is

Kappa architecture is Lambda's answer to its own biggest problem. It was proposed by Jay Kreps (co-creator of Kafka) in 2014 as a direct response to Lambda's dual-maintenance burden.

The premise is simple: **if your streaming system is powerful enough, you don't need a separate batch layer.** You run one stream processing pipeline. When you need to reprocess historical data — because of a bug, a schema change, or a new business requirement — you replay the event log from the beginning.

The event log is the source of truth. Everything else is a derived view.

---

## The core insight

Lambda's batch layer exists because early streaming systems (Storm, early Spark) couldn't handle the scale and complexity that batch systems could. By 2014, that gap had closed significantly. Kreps' argument:

> "Why not just have a single processing infrastructure that handles both the real-time and reprocessing case? If we do this, we just need one system — a streaming system."

The key enabler is **log retention**. If Kafka retains your event log indefinitely (or long enough), you can replay from any point in history through your current streaming logic and rebuild any derived view. Reprocessing becomes a first-class operation, not an afterthought.

---

## Applied to our dataset

For the e-commerce clickstream, the entire system is one Spark Structured Streaming pipeline reading from Kafka:

**Single pipeline computes everything:**
- Real-time revenue metrics (same job, short windows)
- Daily/weekly aggregates (same job, longer windows with watermarking)
- User session analysis (stateful streaming with session windows)
- Cart abandonment detection (event time processing with timers)
- Funnel conversion metrics (multi-event stateful joins)

**When we need to reprocess:**
- Bug in revenue calculation? Fix the code, reset consumer group offset to beginning, replay.
- New feature: add product category to revenue breakdown? Same — fix the code, replay.
- Schema change in upstream events? Update parser, replay affected time range.

**What "reprocessing" looks like:**
1. Deploy new version of the streaming job
2. Set Kafka consumer group offset to the desired start time
3. Run the new job in parallel (new consumer group) until it catches up to real-time
4. Swap the serving layer to point at the new output
5. Decommission the old output

---

## Architecture Diagram

See [diagrams/architecture.mmd](./diagrams/architecture.mmd)

```
E-Commerce Events
       │
       ▼
 [ Kafka ]
 Immutable event log
 Long retention (weeks/months)
 Compacted topics for CDC
       │
       ├─── Current offset (real-time)
       │
       └─── Historical offset (reprocessing)
       │
       ▼
 [ Spark Structured Streaming ]
 Single unified job
 Handles both real-time AND reprocessing
 Stateful: sessions, funnels, aggregates
       │
       ▼
 [ Output Sinks ]
 S3 (analytics / historical)
 Redis (real-time dashboards)
 PostgreSQL (operational queries)
       │
       ▼
 [ Serving Layer ]
 One consistent view — no merge needed
```

---

## When to use Kappa

Kappa is the right choice when:

1. **Your streaming system can handle your entire workload** — both real-time and historical processing
2. **Business logic is complex enough that dual maintenance is genuinely painful** — one codebase is the main win
3. **You can afford the log retention** — replaying 2 years of events requires 2 years of Kafka retention (or an S3-backed log like Confluent Tiered Storage / MSK with S3)
4. **Your team is strong on streaming** — Kappa's operational model (stateful streaming, watermarks, exactly-once semantics) is harder to reason about than batch
5. **Reprocessing is infrequent** — reprocessing is cheap conceptually but expensive computationally; if you're doing it daily, reconsider

---

## When NOT to use Kappa

Kappa is the wrong choice when:

1. **Your log retention is limited** — if Kafka only retains 7 days, you can't reprocess 2 years of history without a separate archive. You've recreated Lambda.
2. **Your batch jobs are deeply SQL-centric** — Kappa's streaming SQL (Flink SQL, Spark Structured Streaming) is powerful but not as mature as batch SQL for complex analytical workloads
3. **You have large historical datasets with no streaming origin** — migrating a data warehouse that predates your event stream requires a separate historical load anyway
4. **Operational simplicity is critical** — stateful streaming with exactly-once semantics is harder to debug and operate than batch jobs on a scheduler
5. **Your team is more comfortable with batch** — technology debt from a poorly operated Kappa system is worse than a well-operated Lambda system

---

## Key tradeoffs

**The single codebase advantage is real — but smaller than advertised.** You do get one processing framework, one deployment pipeline, and one set of transformations to maintain. But streaming code is inherently more complex than equivalent batch code. Stateful joins, event-time semantics, watermarks, and exactly-once guarantees add cognitive overhead. The dual-maintenance tax in Lambda is replaced by the complexity tax in Kappa.

**Reprocessing is conceptually simple, operationally expensive.** "Just replay the log" sounds easy. In practice: you need to size a cluster large enough to process months of historical data at a reasonable speed, manage the dual-running period (old and new job in parallel), ensure output idempotency so partial replays don't corrupt state, and handle schema evolution in the log. None of these are insurmountable, but none are free.

**Kafka as the source of truth changes your operational model.** In Lambda, S3 is the durable record of truth. In Kappa, Kafka (or an equivalent log) is. This means your Kafka cluster is now a critical system that cannot be allowed to lose data. Extended Kafka retention is expensive. Many teams solve this with S3-backed log compaction or dedicated archival topics — which adds complexity back.

**Exactly-once semantics are not optional in Kappa.** If your streaming job can produce duplicates, reprocessing will double-count. Getting exactly-once right across Kafka → Spark → output sinks requires careful configuration and testing. It's achievable but requires discipline.

---

## Implementation plan (coming soon)

- `implementation/streaming-job/` — Unified Spark Structured Streaming pipeline
- `implementation/state-store/` — Stateful operations: sessions, funnels, aggregates
- `implementation/reprocessing/` — Scripts and runbook for log replay
- `implementation/infrastructure/` — Kafka cluster, MSK config, consumer group management

---

## Further reading

- Jay Kreps, "Questioning the Lambda Architecture" (2014) — the founding document
- Martin Kleppmann, *Designing Data-Intensive Applications* — Ch. 11 on stream processing
- Confluent documentation on Kafka log retention and tiered storage
