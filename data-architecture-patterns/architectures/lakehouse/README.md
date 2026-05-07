# Lakehouse Architecture

> The reliability of a data warehouse. The flexibility and scale of a data lake. One system.

---

## What it is

Lakehouse architecture unifies data lakes and data warehouses into a single system built on open file formats stored in object storage (S3, GCS, ADLS). It emerged around 2020 from academic research at Databricks and Berkeley, motivated by a fundamental observation: organizations were maintaining both a data lake *and* a data warehouse, paying the cost of both, and constantly struggling to keep them in sync.

The key technical enabler is the **open table format** — Apache Iceberg, Delta Lake, or Apache Hudi. These formats add a metadata layer on top of Parquet files in object storage that provides ACID transactions, schema evolution, time travel, and efficient query planning. The result: you get warehouse-quality reliability and query performance while keeping your data in open formats on cheap object storage.

---

## The core insight

Traditional architecture forced a choice:

| Data Lake | Data Warehouse |
|---|---|
| Cheap storage (S3) | Expensive storage (proprietary) |
| Open formats (Parquet) | Proprietary formats |
| Schema-on-read (flexible) | Schema-on-write (rigid) |
| No ACID guarantees | Full ACID guarantees |
| Poor BI query performance | Optimized for BI queries |
| ETL pipelines to warehouse | Direct query |

Lakehouse collapses this into one system by solving the data lake's core weaknesses — ACID, performance, data quality — without giving up cheap storage or open formats.

---

## Applied to our dataset

The e-commerce clickstream in a Lakehouse is one coherent system:

**Ingestion:** Kafka → Iceberg tables directly via Spark Structured Streaming (no separate "landing zone" required, though Bronze-equivalent raw tables are still good practice)

**Storage:** All data in Iceberg tables on S3. One format, one storage layer, one set of access controls.

**ACID in practice:**
- `purchase` events update inventory counts atomically — no overselling
- Order deduplication via `MERGE INTO` (upsert) — idempotent exactly-once writes from Spark
- Failed pipeline writes don't leave partial partitions — transaction either commits or rolls back

**Time travel in practice:**
- Compliance: `SELECT * FROM purchases FOR SYSTEM_TIME AS OF '2025-01-01'`
- Debugging: compare yesterday's revenue calculation against today's to isolate a regression
- ML: reproducible training datasets — pin to a specific table snapshot

**Multi-engine in practice:**
- Spark writes the Iceberg tables
- Trino/Athena reads for ad-hoc analyst queries (no data movement)
- Flink reads for streaming consumers
- dbt transforms Silver → Gold layers

---

## Architecture Diagram

See [diagrams/architecture.mmd](./diagrams/architecture.mmd)

---

## How Lakehouse compares to the others

| Capability | Lambda | Kappa | Medallion | Lakehouse |
|---|---|---|---|---|
| Real-time ingestion | ✅ (speed layer) | ✅ (native) | ⚠️ (add streaming) | ✅ (streaming ingest) |
| Historical batch | ✅ (batch layer) | ✅ (replay) | ✅ (native) | ✅ (native) |
| ACID transactions | ❌ | ❌ | ⚠️ (with Delta/Iceberg) | ✅ (native) |
| Time travel | ❌ | ❌ | ✅ (with Delta/Iceberg) | ✅ (native) |
| Schema evolution | ❌ | ⚠️ | ✅ | ✅ |
| Multi-engine support | ⚠️ | ⚠️ | ✅ | ✅ |
| Single codebase | ❌ | ✅ | ✅ | ✅ |
| Operational complexity | High | Medium | Low-Medium | Medium |
| Storage efficiency | Medium | Medium | Low (3 copies) | High |

---

## When to use Lakehouse

Lakehouse is the right choice when:

1. **You're building a new system from scratch** — it's the modern default; avoid Lambda and pure data lakes for greenfield work
2. **You need ACID transactions** — inventory management, financial reconciliation, deduplication at scale
3. **You have multiple compute engines** — your Spark team, your Trino/Athena analysts, and your Flink streaming consumers can all read the same tables
4. **You want to eliminate the lake → warehouse ETL** — Lakehouse collapses a two-tier architecture into one, removing a class of pipeline entirely
5. **Schema evolution is frequent** — Iceberg's schema evolution support (add column, rename column, change type for compatible types) handles this cleanly without rewriting data
6. **Storage cost at scale matters** — Iceberg's hidden partitioning, compaction, and expiration of old snapshots keep storage efficient

---

## When NOT to use Lakehouse

Lakehouse is the wrong choice when:

1. **Your team is deeply invested in a managed data warehouse** — migrating from Snowflake or BigQuery to a Lakehouse requires rebuilding tooling, governance, and query patterns. The break-even is long.
2. **You need sub-second query latency for dashboards** — Iceberg on S3 is fast, but it's not Snowflake fast. For executive dashboards that need < 500ms query response, you still want a columnar warehouse or a pre-aggregated serving layer.
3. **Your data team is small** — Lakehouse introduces new concepts (table formats, metadata management, compaction jobs, snapshot expiration) that require engineering investment to operate well
4. **You're heavily dependent on SQL semantics your warehouse provides** — Snowflake's MATCH_RECOGNIZE, BigQuery's ARRAY/STRUCT types, and Redshift's distribution keys don't have direct equivalents in the Lakehouse world

---

## Key tradeoffs

**The open format advantage is real and compounds over time.** Proprietary warehouse formats create vendor lock-in that becomes increasingly expensive as your data grows. Iceberg on S3 means you can switch compute engines, query services, and even cloud providers without migrating data. At petabyte scale, this flexibility has real monetary value.

**ACID on S3 required a solved hard problem.** Object stores like S3 have no native transaction support. Iceberg solves this with optimistic concurrency control via a metadata tree — writers propose a new snapshot, and the commit either atomically replaces the metadata pointer or fails. This is elegant but requires understanding the failure modes: write-write conflicts, orphaned files from failed writes, and the need for periodic cleanup jobs (Iceberg's `expireSnapshots` and `removeOrphanFiles`).

**Table maintenance is a new operational concern.** Iceberg tables accumulate small files, old snapshots, and orphaned data files over time. Without compaction (merging small files into larger ones), query performance degrades. Without snapshot expiration, storage costs grow indefinitely. These maintenance operations need to be scheduled and monitored — they're the Lakehouse equivalent of a warehouse's `VACUUM` or statistics update job.

**The "no ETL" promise is partially true.** Lakehouse eliminates the lake → warehouse ETL hop. But you still need transformation pipelines (raw → clean → aggregated). The Medallion pattern applied on top of Lakehouse tables is a natural fit and gives you quality layering without the duplicate storage cost of traditional Medallion on plain Parquet.

---

## Implementation plan (coming soon)

- `implementation/ingestion/` — Kafka → Bronze Iceberg tables via Spark Structured Streaming
- `implementation/transforms/` — Bronze → Silver → Gold via PySpark + dbt
- `implementation/acid-patterns/` — MERGE INTO for upserts, deduplication patterns
- `implementation/time-travel/` — Snapshot management, point-in-time queries
- `implementation/maintenance/` — Compaction, snapshot expiration, orphan file cleanup
- `implementation/infrastructure/` — S3, Glue Catalog for Iceberg metadata, Athena query setup

---

## Further reading

- Armbrust et al., "Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics" (CIDR 2021) — the original paper
- Apache Iceberg documentation — especially table spec and partitioning guide
- *Delta Lake: Up and Running* by O'Reilly — practical Lakehouse patterns
