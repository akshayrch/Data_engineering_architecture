# Lakehouse Architecture — Decision Log

> Observations from implementing Lakehouse against the e-commerce clickstream dataset. What separates Lakehouse from the others in practice, not just in theory.

---

## Decision 1: Apache Iceberg vs. Delta Lake

**Context:** Both Iceberg and Delta Lake provide the core Lakehouse capabilities (ACID, time travel, schema evolution, efficient query planning). The choice has long-term consequences because table formats are not trivially migrated.

**Options considered:**
1. Apache Iceberg — open standard, multi-engine (Spark + Flink + Trino + Athena native), hidden partitioning
2. Delta Lake — Databricks-origin, excellent Spark integration, simpler operational model, strong Python API
3. Apache Hudi — best for CDC / upsert-heavy workloads, incremental pull queries, smaller community

**Decision:** Apache Iceberg.

**Rationale:** The decisive factor is multi-engine neutrality. Iceberg tables can be read by Spark, Trino, Athena (without Glue conversion), Flink, and increasingly DuckDB. This means analysts using Athena, engineers using Spark, and streaming consumers using Flink all read the same table — no format conversion, no data movement. Delta Lake is catching up (Delta Universal Format / UniForm), but Iceberg's multi-engine story is more mature.

Hidden partitioning is the second reason: Iceberg partitions data behind the scenes (by transformed columns — `days(event_ts)`, `bucket(user_id, 16)`) without exposing the partition scheme to query writers. Analysts query `WHERE event_date = '2025-01-15'` and Iceberg handles the partition pruning. With Delta Lake, analysts must know the partition column name and filter on it explicitly.

**Consequences:** Iceberg's operational tooling is slightly less polished than Delta Lake's. `OPTIMIZE` (compaction) and `VACUUM` (file cleanup) have direct equivalents in Iceberg (`rewriteDataFiles` and `expireSnapshots`/`removeOrphanFiles`) but require more explicit configuration. Databricks environments will be more naturally Delta-centric; EMR and non-Databricks Spark environments favor Iceberg.

---

## Decision 2: Catalog — AWS Glue Data Catalog vs. Hive Metastore vs. REST Catalog

**Context:** Iceberg requires a catalog to store and manage table metadata (current snapshot pointer, schema history, partition spec). The catalog is a critical infrastructure dependency.

**Options considered:**
1. AWS Glue Data Catalog — managed, integrates with Athena and EMR natively, low operational overhead
2. Hive Metastore (HMS) — battle-tested, Spark-native, requires managing a database (PostgreSQL/MySQL)
3. Iceberg REST Catalog — open standard, cloud-agnostic, requires a running server
4. Nessie — git-like branching for data, interesting for development workflows but operationally complex

**Decision:** AWS Glue Data Catalog for this implementation.

**Rationale:** For an AWS-centric stack (S3, EMR, Athena), Glue is the path of least resistance. Athena natively queries Glue-registered Iceberg tables. EMR Spark supports Glue as a Hive catalog. The managed aspect eliminates a database to operate. In a multi-cloud or on-premises environment, the REST Catalog would be the better choice for portability.

**Consequences:** Glue has a cost per API call at high table modification frequency. For workloads doing thousands of Iceberg commits per hour, Glue costs can add up. For this dataset (hourly batch + streaming micro-batch), the cost is negligible. Glue also has limitations on table property keys — some Iceberg configurations require workarounds.

---

## Decision 3: MERGE INTO patterns for deduplication

**Context:** The e-commerce clickstream has duplicate events (mobile apps retry failed sends, Kafka at-least-once delivery, upstream systems that send the same event twice on failure). Deduplication at ingest time is critical for accurate metrics.

**Options considered:**
1. Deduplicate in Silver Spark job (read Bronze, drop duplicates, write Silver) — simple but reads all Bronze data
2. Streaming deduplication with stateful Spark (maintain seen event_ids in state store) — real-time but state store grows unboundedly
3. Iceberg MERGE INTO (upsert: if event_id exists, skip; if not, insert) — SQL-native, efficient
4. Iceberg row-level deletes (insert all, then delete duplicates in a separate pass) — two-pass, more S3 writes

**Decision:** Option 3 — Iceberg MERGE INTO at Silver write time.

**Rationale:** `MERGE INTO silver_events USING staging AS s ON silver.event_id = s.event_id WHEN NOT MATCHED THEN INSERT *` is exactly-once by construction. The same job can run multiple times (idempotent) — re-running after a failure won't introduce duplicates. This is cleaner than stateful streaming deduplication (which requires managing growing state stores) and more efficient than two-pass delete.

**Consequences:** MERGE INTO on large tables does a full scan of the target table to find matching rows. Iceberg's equality delete files mitigate this for row-level deletes, but MERGE requires partition-level or full-scan matching. For large Silver tables, this means partitioning by event date and always including `event_date` in the MERGE predicate to enable partition pruning. Without this, MERGE scans the entire table on every run.

---

## Observations (updated as implementation progresses)

**Observation 1: The ACID transaction model changes how you think about pipeline failures.**
In a traditional Parquet-on-S3 pipeline, a failed write leaves partial data in S3 that must be manually cleaned up. With Iceberg, a failed write simply means the transaction didn't commit — the table is in exactly the same state as before the write started. This changes incident response: instead of "stop everything, find partial files, manually clean up, re-run," it's "check the Iceberg snapshot log, confirm the failed write didn't commit, re-run." This operational simplicity compounds over time.

**Observation 2: Compaction is the hidden maintenance job that matters most.**
Streaming ingestion to Iceberg produces many small files (one per micro-batch, per partition). Without compaction, query performance degrades because Spark/Trino must open thousands of files to answer a simple aggregation. Iceberg's `rewriteDataFiles` action merges small files into larger ones (~128MB target). This needs to run on a schedule — daily for batch-heavy tables, hourly for streaming-heavy tables. It's operationally simple but must be monitored: a failed compaction job silently allows file count to grow.

**Observation 3: Time travel is more useful than expected, but requires snapshot management.**
Iceberg keeps every committed snapshot by default. After weeks of streaming ingestion, a table can have thousands of snapshots, each referencing files that may or may not still be needed. `expireSnapshots` cleans up old snapshots and marks files for deletion; `removeOrphanFiles` actually deletes them. Forgetting to run these means S3 storage costs grow indefinitely and the metadata table becomes slow to navigate. Add both to the maintenance DAG from day one.

**Observation 4: Lakehouse + Medallion is the natural combination.**
Medallion applied on Iceberg tables (Bronze Iceberg → Silver Iceberg → Gold Iceberg) is significantly better than Medallion on plain Parquet. Each layer gets ACID guarantees (no partial Silver writes from failed jobs), schema evolution without breaking downstream readers, and time travel for debugging. The storage overhead of Medallion (3 copies of data) is partially offset by Iceberg's efficient storage layout, hidden partitioning, and compaction. This is the combination I would choose for any new production system.

---

*Log updated as implementation progresses. Last updated: — (implementation pending)*
