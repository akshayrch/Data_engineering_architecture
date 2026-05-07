# Medallion vs. Lakehouse: A Direct Comparison

> These two aren't really opposites — Medallion is an organizational pattern, Lakehouse is a storage architecture. They're most powerful when combined. But they're worth comparing to understand what each one actually solves.

---

## Clarifying the comparison

This is a slightly unusual comparison because Medallion and Lakehouse operate at different levels of abstraction:

- **Medallion** is an **organizational pattern** — it tells you how to layer your data (Bronze → Silver → Gold) and what each layer is responsible for
- **Lakehouse** is a **storage and compute architecture** — it tells you how to store and access data (ACID table formats on object storage, multi-engine access)

They're complementary, not competing. The most common modern pattern is **Lakehouse + Medallion**: Iceberg/Delta Lake tables organized into Bronze/Silver/Gold layers. This combination is described in the [Lakehouse architecture docs](../lakehouse/README.md).

The comparison here is: **Medallion on plain Parquet/S3** (no table format) vs. **Lakehouse (Iceberg/Delta)** with Medallion layering.

---

## Medallion on plain Parquet

What most teams implement when they say "Medallion architecture" without a modern table format:

- Bronze: raw JSON/CSV/Parquet files landed to S3
- Silver: Spark jobs read Bronze, write cleaned Parquet to Silver prefix in S3
- Gold: Spark jobs read Silver, write aggregated Parquet to Gold prefix in S3
- Orchestration: Airflow DAGs managing the three hops
- Query access: Athena/Presto with Glue catalog pointing at Parquet files

This works. It's the pattern most data teams built between 2017 and 2022. But it has real limitations.

---

## Head-to-head

| Dimension | Medallion (Parquet) | Lakehouse (Iceberg + Medallion) |
|---|---|---|
| **ACID transactions** | ❌ No — partial writes leave corrupt data | ✅ Yes — failed write = no-op, table unchanged |
| **Concurrent writers** | ❌ Race conditions on shared S3 prefixes | ✅ Optimistic concurrency control in Iceberg |
| **Deduplication** | Manual: read all data, drop duplicates, rewrite | ✅ MERGE INTO: atomic upsert, idempotent |
| **Schema evolution** | Painful: rewrite entire partition on type change | ✅ ADD COLUMN, RENAME, type-compatible changes |
| **Time travel** | ❌ Not supported | ✅ AS OF TIMESTAMP or snapshot ID |
| **Query performance** | Medium: Parquet is good, but no metadata pruning | High: Iceberg statistics, hidden partitioning |
| **Storage efficiency** | Medium: no compaction, many small files accumulate | High: scheduled compaction, Z-ordering |
| **Multi-engine reads** | ✅ Any engine can read Parquet | ✅ Any Iceberg-compatible engine |
| **Incremental processing** | Hard: must track watermarks externally | ✅ Iceberg incremental read (new files since snapshot N) |
| **Operational complexity** | Low: just files and folders | Medium: need compaction/expiration jobs |
| **Learning curve** | Low: everyone understands files | Medium: table format concepts, snapshot model |

---

## Where plain Medallion on Parquet still works

**Small-to-medium scale with stable schemas.** If your data volume is manageable, your schemas don't change often, and you don't have concurrent writers, Parquet-based Medallion is simpler to operate and reason about. You don't need a table format to add value.

**Teams without streaming ingestion.** ACID transactions matter most when multiple writers are committing simultaneously. If your Bronze ingestion is a single Spark job running every hour, you're unlikely to hit the race conditions that Iceberg solves.

**Quick-start environments.** Setting up Iceberg with a Glue catalog takes effort. A well-designed Parquet-based Medallion pipeline can be stood up faster and iterated on — with a planned migration to Lakehouse when scale demands it.

---

## Where Lakehouse clearly wins

**Any workload with concurrent writes.** Two Spark jobs writing to the same Silver partition simultaneously on Parquet will corrupt data. This isn't hypothetical — it happens whenever you have overlapping microbatch windows or parallel partition backfills. Iceberg eliminates this class of bug entirely.

**Pipelines that need MERGE / upsert semantics.** Deduplication, CDC (change data capture), and slowly changing dimensions all require "update if exists, insert if not." On plain Parquet, this requires reading a full partition, modifying it in memory, and rewriting it — expensive and non-atomic. Iceberg's MERGE INTO does this efficiently and safely.

**Debugging and compliance.** When a Gold metric is wrong, the investigation on Parquet is: find the Silver files, find the Bronze files, re-read them all locally. With Iceberg time travel, the investigation is: query Silver as of 3 hours ago and compare. One SQL statement vs. a multi-hour archaeological dig.

**Long-lived systems with schema changes.** A table format that handles ADD COLUMN gracefully (no partition rewrite, old files still readable) is worth its weight in gold for tables that live for years. Plain Parquet schema changes require careful choreography or full table rewrites.

---

## The combination: Medallion on Lakehouse

This is the pattern to use for production systems:

```
Bronze Iceberg Tables  →  Silver Iceberg Tables  →  Gold Iceberg Tables
   (append-only)           (MERGE INTO dedup)        (aggregated, certified)
   ACID ingest             Schema evolution           Time travel for audit
   Time travel             Quality assertions         Multi-engine reads
```

What you get beyond plain Medallion: ACID at every layer boundary means no corrupt partial writes. MERGE INTO for Silver deduplication means idempotent pipelines by default. Time travel at every layer means debugging and compliance are one SQL statement. Schema evolution without rewrites means you can add fields to Bronze without cascading changes to Silver and Gold.

---

## Verdict for the e-commerce dataset

**Lakehouse (Iceberg) + Medallion wins** over plain Medallion on Parquet.

The deciding factors for this dataset: the 0.3% duplicate event rate requires deduplication, which is a MERGE INTO operation — clean on Iceberg, painful on plain Parquet. Quarterly schema drift in the event schema (new fields, type changes) makes Iceberg's schema evolution worth the setup cost. And the compliance requirement for point-in-time revenue reconstruction makes time travel non-optional.

The additional operational cost (compaction jobs, snapshot expiration) is real but bounded. Both can be handled by a short Airflow maintenance DAG that runs daily.
