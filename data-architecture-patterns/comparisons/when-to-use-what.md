# When to Use What: A Decision Framework

> The right architecture depends on your constraints, not the architecture's theoretical elegance. Here's how to choose.

---

## The one-page answer

| If you... | Use... |
|---|---|
| Are building from scratch in 2025 | **Lakehouse** |
| Have mature batch pipelines + need real-time | **Lambda** (add speed layer to existing batch) |
| Have one team, one codebase, frequent business logic changes | **Kappa** |
| Have multiple teams consuming data at different quality levels | **Medallion on Lakehouse** |
| Need sub-second latency (< 1 second) | **Kappa with Flink** |
| Need ACID transactions and time travel | **Lakehouse** |
| Are migrating from a legacy DWH | **Medallion on Lakehouse** |
| Have a small team and limited streaming expertise | **Medallion on Lakehouse** (batch-first) |

---

## Decision tree

```
START: What is your primary constraint?
│
├─ "I need data available in real-time (< 30 seconds)"
│   │
│   ├─ Do you also need correctness over full history?
│   │   ├─ YES → Lambda (if you have existing batch) OR Kappa (if starting fresh)
│   │   └─ NO  → Kappa
│   │
│   └─ Do you have sub-second latency requirements?
│       ├─ YES → Kappa with Apache Flink
│       └─ NO  → Kappa with Spark Structured Streaming
│
├─ "I need reliable, high-quality data for analytics and BI"
│   │
│   ├─ Do you have multiple teams with different data needs?
│   │   ├─ YES → Medallion on Lakehouse (Bronze/Silver/Gold with Iceberg)
│   │   └─ NO  → Lakehouse (simpler, no strict layering required)
│   │
│   └─ Do you need ACID transactions or time travel?
│       ├─ YES → Lakehouse (Iceberg or Delta Lake)
│       └─ NO  → Medallion on Parquet (simpler to start)
│
├─ "I have existing batch pipelines that work, and need to add streaming"
│   └─ Lambda (preserve existing batch, add speed layer on top)
│
└─ "I'm building a new system and want the best long-term architecture"
    └─ Lakehouse + Medallion (Iceberg + Bronze/Silver/Gold + Airflow)
```

---

## The four architectures, honestly

### Lambda: mature but expensive to operate
**Reach for it when:** Your existing batch system is correct and battle-tested, and you need to bolt on real-time without disrupting it. Lambda's batch layer gives you a "just re-run it" escape hatch that streaming systems don't have.

**The hidden cost:** Dual maintenance never gets easier. Every feature addition, bug fix, and business logic change touches two codebases. Teams that don't enforce shared libraries for business logic end up with batch and streaming implementations that silently diverge.

**Honest assessment:** Lambda was the right answer from 2011-2018. For new systems in 2025, it's rarely the first choice. It remains valuable for organizations with significant investment in batch pipelines that can't be migrated.

---

### Kappa: elegant in theory, disciplined in practice
**Reach for it when:** Your team is strong in streaming, your business logic changes frequently, and your Kafka log retention is adequate for your reprocessing horizon.

**The hidden cost:** Stateful streaming is harder to reason about than batch. Watermarks, state store sizing, checkpoint recovery, and exactly-once delivery require sustained engineering discipline. Teams that aren't fluent in streaming will produce buggy Kappa implementations that are harder to fix than the Lambda system they were trying to avoid.

**Honest assessment:** Kappa is excellent when the team is right for it. The "one codebase" benefit is real and compounds over time. But it requires genuine streaming expertise — not just familiarity with Spark. For teams without that expertise, Medallion on Lakehouse with batch-first processing is a safer starting point.

---

### Medallion: accessible, scalable, team-friendly
**Reach for it when:** You have multiple teams with different data needs, data quality is a first-class concern, and you want a pattern that scales with organizational complexity.

**The hidden cost:** Medallion doesn't solve streaming. A common mistake: building a beautiful Bronze/Silver/Gold pipeline and then discovering that the business needs real-time data, which Medallion doesn't provide by default. The fix is to add a streaming ingestion layer at Bronze, but this is an architectural addition that needs to be planned for.

**Honest assessment:** Medallion is the most widely deployed pattern in production data platforms today, and for good reason. It's intuitive, it scales with team size, and the Bronze/Silver/Gold metaphor communicates data quality to non-engineers. Combined with Iceberg, it's the foundation of most modern enterprise data platforms.

---

### Lakehouse: the modern default
**Reach for it when:** You're building a new system, you need ACID transactions or time travel, you have multiple compute engines, or you want to eliminate the data lake → data warehouse ETL hop.

**The hidden cost:** Table format operations (compaction, snapshot expiration, metadata management) are a new operational surface area. The Iceberg/Delta Lake concepts (snapshot isolation, manifest files, hidden partitioning) require learning investment. And the "no ETL" promise is partial — you still need transformation pipelines; you just don't need to move data between a lake and a warehouse.

**Honest assessment:** Lakehouse is the right greenfield choice in 2025. The table format ecosystem (Iceberg, Delta Lake) is mature, the multi-engine story is real, and ACID on object storage solves a class of bugs that Parquet-based pipelines regularly hit. The operational overhead is bounded and manageable. Organizations that built their platforms on plain Parquet + a separate warehouse are actively migrating to Lakehouse, not the other way around.

---

## The architecture for this dataset: verdict

After implementing all four against the same e-commerce clickstream:

**Winner: Lakehouse (Iceberg) + Medallion pattern**

The reasoning:
1. The duplicate event rate (0.3%) requires clean deduplication → MERGE INTO → Iceberg
2. Quarterly schema changes in event structure → schema evolution → Iceberg
3. Compliance requirement for point-in-time revenue reconstruction → time travel → Iceberg
4. Multiple consumer teams (analytics, ML, finance, ops) with different quality needs → Bronze/Silver/Gold → Medallion
5. Streaming ingestion requirement for real-time cart abandonment → Iceberg + Spark Structured Streaming handles this without a separate speed layer

Lambda is second if you have existing batch investments to protect.
Kappa is second if your team is streaming-first and business logic changes are frequent.
Plain Medallion is a stepping stone — useful to understand, worth migrating to Lakehouse when you hit its limitations.

---

## What to read next

- [Lambda vs. Kappa deep comparison](./lambda-vs-kappa.md)
- [Medallion vs. Lakehouse deep comparison](./medallion-vs-lakehouse.md)
- Individual architecture decision logs in each architecture folder — the specific tradeoff decisions made during implementation
