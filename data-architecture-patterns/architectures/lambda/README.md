# Lambda Architecture

> Batch for correctness. Stream for speed. Serve from both.

---

## What it is

Lambda architecture splits data processing into two independent layers that run in parallel:

- **Batch layer** — processes the complete historical dataset on a schedule (hourly, daily). Always correct because it reprocesses everything from source. Slow.
- **Speed layer** — processes the real-time stream as events arrive. Fast but approximate — it only covers the recent window since the last batch run.
- **Serving layer** — merges results from both layers at query time. Returns batch results for historical data, speed layer results for the recent window.

The name comes from Nathan Marz, who formalized the pattern in 2011 to solve a specific problem: how do you serve both low-latency real-time queries and high-accuracy historical queries from the same system?

---

## The core insight

Batch and stream processing have fundamentally different SLAs:

| Concern | Batch Layer | Speed Layer |
|---|---|---|
| Latency | Minutes to hours | Seconds |
| Accuracy | Exact (full recompute) | Approximate (recent window) |
| Failure recovery | Re-run the job | Complex — replay from offset |
| Late data handling | Natural — included in next run | Hard — window already closed |
| Complexity | Low per layer | High per layer |

Lambda's answer: don't try to make one system do both. Build two systems, serve from both.

---

## Applied to our dataset

For the e-commerce clickstream:

**Batch layer** computes:
- Daily/weekly/monthly revenue by product, category, region
- User cohort analysis (30-day retention, LTV)
- Funnel conversion rates (view → cart → purchase)
- Product recommendation models (trained on full history)

**Speed layer** computes:
- Live cart abandonment signals (user added to cart, no purchase in 30 min)
- Real-time revenue dashboard (last 15 minutes)
- Active session counts
- Fraud signals (unusual purchase velocity)

**Serving layer** merges:
- Historical funnel metrics (from batch) + today's real-time funnel (from speed)
- Yesterday's product rankings (batch) + today's trending (speed)

---

## Architecture Diagram

See [diagrams/architecture.mmd](./diagrams/architecture.mmd)

```
E-Commerce Events
       │
       ├──────────────────────────────────────────────┐
       │                                              │
       ▼                                              ▼
 [ Batch Layer ]                            [ Speed Layer ]
 S3 Raw Storage                             Kafka Topic
 PySpark Jobs (daily)                       Spark Structured Streaming
 Full recompute                             Micro-batch (30s windows)
 Parquet output                             In-memory / Redis state
       │                                              │
       └──────────────────┬───────────────────────────┘
                          │
                          ▼
                  [ Serving Layer ]
                  Query merges batch + speed results
                  Hive / Athena / Presto
                  Batch results for t-1 and older
                  Speed results for last N minutes
```

---

## When to use Lambda

Lambda is the right choice when:

1. **You have genuinely different SLA requirements** — some queries need sub-second latency, others need guaranteed correctness over full history
2. **Your batch jobs already exist** and you need to add real-time on top without rewriting them
3. **Late-arriving data is a real problem** — e.g., mobile apps that batch events, IoT sensors with intermittent connectivity
4. **Reprocessing history is non-negotiable** — compliance, auditing, ML model retraining
5. **Your team is comfortable operating two separate systems**

---

## When NOT to use Lambda

Lambda is the wrong choice when:

1. **Your business logic is complex** — maintaining two implementations of the same logic (batch + stream) doubles your bug surface area. A change in business logic must be applied twice.
2. **You have a small team** — two pipelines = two things to monitor, debug, and on-call for
3. **Your latency tolerance is tight enough for Kappa** — if you can tolerate 30-60 second latency, Kappa gives you one codebase
4. **Your data volume doesn't justify the complexity** — Lambda's overhead is real; don't use it for low-volume workloads
5. **You're starting fresh** — modern alternatives (Kappa, Lakehouse) are better defaults for new systems

---

## Key tradeoffs

**The dual-maintenance problem is the real cost.** Lambda's biggest weakness is not operational complexity — it's that you have to implement your business logic twice. Every transformation, aggregation, and join must be built in both the batch framework (PySpark) and the streaming framework (Spark Structured Streaming, Flink). When the logic changes, you change it in two places. When there's a bug, you debug in two places.

**The serving layer merge is subtle.** Deciding where the "batch horizon" ends and the "speed window" begins is non-trivial. If your batch job runs at 2AM and completes at 4AM, queries between midnight and 4AM are served from a stale batch + speed layer. You need to think carefully about consistency guarantees at that boundary.

**Reprocessability is a genuine advantage.** When you discover a bug in your batch logic, you fix it and re-run. All your historical data gets corrected. In Kappa, reprocessing from a Kafka log that's 90 days old requires that log to exist, be accessible, and be replayed at scale — which is its own operational challenge.

---

## Implementation plan (coming soon)

- `implementation/batch-layer/` — PySpark jobs for daily aggregations
- `implementation/speed-layer/` — Spark Structured Streaming for real-time metrics
- `implementation/serving-layer/` — Query layer with batch/speed merge logic
- `implementation/infrastructure/` — Terraform for S3, EMR, Kafka, Redis

---

## Further reading

- Nathan Marz, "How to beat the CAP theorem" (2011) — original formulation
- *Big Data* by Nathan Marz & James Warren — the definitive book on Lambda
- Jay Kreps, "Questioning the Lambda Architecture" (2014) — the Kappa counterargument
