# Kappa Architecture — Decision Log

> Observations from implementing Kappa against the e-commerce clickstream dataset. What the theory says vs. what the implementation reveals.

---

## Decision 1: Spark Structured Streaming vs. Apache Flink

**Context:** Kappa requires a streaming engine capable of handling stateful operations, event-time processing, and exactly-once delivery. Two serious contenders exist: Spark Structured Streaming and Apache Flink.

**Options considered:**
1. Spark Structured Streaming — familiar API, tight integration with the broader Spark/EMR ecosystem
2. Apache Flink — purpose-built for streaming, lower latency, more mature stateful processing
3. Kafka Streams — JVM-only, tight Kafka coupling, simpler for pure Kafka-to-Kafka pipelines

**Decision:** Spark Structured Streaming, with Flink noted as the better choice for sub-second latency requirements.

**Rationale:** For this study, consistency with the Lambda implementation (which also uses Spark) is valuable — it isolates the architectural difference from the framework difference. In a production context where latency < 1 second is required, Flink's lower overhead and more mature stateful APIs would be the better choice. Kafka Streams was ruled out because it ties the processing logic to JVM and Kafka-to-Kafka topology, which limits flexibility.

**Consequences:** Spark Structured Streaming micro-batches (minimum ~100ms trigger interval) means this isn't a true event-at-a-time system. True sub-second latency requires Flink or a Kafka Streams topology. For the e-commerce use case (30-second cart abandonment windows, minute-level revenue metrics), Spark's latency is acceptable.

---

## Decision 2: Log retention strategy — Kafka vs. S3 archive

**Context:** Kappa's reprocessability depends entirely on the availability of the historical event log. Kafka's native retention is expensive at scale. 6 months of e-commerce clickstream at 10,000 events/second exceeds 100TB — not practical to keep in Kafka brokers.

**Options considered:**
1. Kafka with extended retention (expensive, operationally complex)
2. Kafka + S3 archival via Kafka Connect S3 Sink (Kafka for recent data, S3 for history)
3. Confluent Tiered Storage / Amazon MSK with S3 (transparent tiering)
4. S3 as the primary log, Kafka for recent window only (hybrid approach)

**Decision:** Option 2 — Kafka for 7-day hot retention, S3 archival for historical replay.

**Rationale:** Option 3 (managed tiered storage) is the cleanest but requires Confluent Cloud or a specific MSK configuration. Option 2 is achievable with open-source tooling and makes the reprocessing path explicit: recent events replay from Kafka, historical events replay from S3 using a Spark batch job that writes to a temporary Kafka topic. This is more operationally transparent than tiered storage magic.

**Consequences:** Reprocessing events older than 7 days requires a two-phase approach: load from S3 → temporary Kafka topic → replay through streaming job. This adds operational steps but keeps the processing logic unified (the streaming job doesn't need to know whether it's reading from "real" Kafka or a replay topic).

**Key learning:** This is where Kappa quietly starts looking like Lambda again. Once you have a separate historical store (S3) feeding into your pipeline alongside the live stream, you've reintroduced a dual-path architecture. The difference is you're using the same processing code — but the operational topology is more complex than "just replay from Kafka."

---

## Decision 3: Exactly-once delivery — configuration and verification

**Context:** Kappa's reprocessing guarantee is only meaningful if the streaming job produces exactly-once output. Duplicate processing during replay would corrupt aggregations.

**Options considered:**
1. At-least-once + idempotent writes (deduplication at the sink)
2. Spark Structured Streaming with Kafka source + idempotent sink (checkpointing-based)
3. Transactional Kafka producers + exactly-once Spark configuration

**Decision:** Option 2 — Spark checkpointing with idempotent output sinks.

**Rationale:** Spark Structured Streaming's checkpoint mechanism provides end-to-end exactly-once when combined with idempotent output sinks. For Parquet output to S3, this means partition-aligned writes (a failed write doesn't leave partial partitions). For Redis state, it means conditional writes. Full Kafka transactions (Option 3) add complexity without meaningful benefit for this topology.

**Consequences:** Checkpointing adds recovery overhead. If the streaming job crashes, recovery replays from the last checkpoint — which can be 1-5 minutes of reprocessing on restart depending on checkpoint interval. Checkpoint storage (S3 or HDFS) becomes a critical dependency. Lost checkpoints = potential duplicates, requiring a full replay.

---

## Observations (updated as implementation progresses)

**Observation 1: Stateful streaming is where complexity actually lives.**
Writing a stateless streaming job (filter, project, aggregate over a fixed window) is straightforward. The hard part is stateful logic: user sessions that can span hours, funnel steps that must be joined across multiple events, and cart abandonment detection that requires timers. This logic is expressible in Structured Streaming but requires careful reasoning about state retention, watermarks, and memory bounds. Equivalent batch logic is far simpler to write and debug.

**Observation 2: Watermarks require a real understanding of your data's lateness distribution.**
Setting a watermark too tight drops legitimate late events. Setting it too loose holds state in memory indefinitely and delays output. Getting this right requires looking at actual lateness distributions in production data — not guessing. For the e-commerce dataset, mobile app events can arrive hours late (app opened after user was offline). A 2-hour watermark handles 95% of late events; the tail is accepted as lost in the speed view.

**Observation 3: The "one codebase" benefit requires discipline to maintain.**
It's easy for a Kappa implementation to drift into two separate jobs — one optimized for real-time with aggressive windowing, one for historical accuracy with longer windows and more state. Once that happens, you've recreated Lambda's dual-maintenance problem inside a single framework. Keeping one unified job requires thoughtful parameterization and the discipline to resist "just make a separate job for the batch case."

---

*Log updated as implementation progresses. Last updated: — (implementation pending)*
