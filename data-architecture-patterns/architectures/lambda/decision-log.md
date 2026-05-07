# Lambda Architecture — Decision Log

> This is not a textbook summary. These are observations from actually implementing Lambda against the e-commerce clickstream dataset — what I expected, what surprised me, and what I'd do differently.

---

## Decision 1: PySpark for batch, Spark Structured Streaming for speed

**Date:** TBD (implementation in progress)

**Context:** Lambda requires separate batch and speed layer implementations. The question was whether to use the same framework for both (reducing cognitive overhead) or use best-of-breed tools for each layer (optimizing each layer independently).

**Options considered:**
1. PySpark (batch) + Spark Structured Streaming (speed) — one ecosystem, two modes
2. PySpark (batch) + Apache Flink (speed) — best-of-breed, more complexity
3. dbt (batch) + Kafka Streams (speed) — SQL-first batch, Java-based stream

**Decision:** Option 1 — unified Spark ecosystem.

**Rationale:** The dual-maintenance problem in Lambda is already a tax. Using the same framework (Spark) for both layers means developers can context-switch between batch and streaming jobs without learning a second execution model. Flink is more powerful for complex stateful streaming but the operational overhead of running two separate clusters (EMR for Spark + Flink cluster) wasn't justified for this use case.

**Consequences:** The Spark Structured Streaming API is still meaningfully different from the batch DataFrame API. The "same framework" benefit is real but partial — you still write different code, you just use the same language and libraries.

---

## Decision 2: Where to draw the batch horizon

**Context:** The serving layer must decide which data to serve from batch vs. speed. This requires defining a "batch horizon" — the point in time up to which batch results are authoritative.

**Options considered:**
1. Fixed offset (e.g., always serve speed for last 2 hours)
2. Dynamic — serve speed only for events after the last successful batch run
3. Hybrid — batch is authoritative for completed days, speed fills the current day

**Decision:** Option 3 — day-boundary batch horizon.

**Rationale:** Day-aligned batches match business reporting requirements (daily revenue, daily conversion rates). The serving layer logic is simpler when the boundary aligns with a natural time unit. Real-time queries for "today's" metrics always come from the speed layer; historical queries for prior days always come from batch.

**Consequences:** Users querying "today's revenue" get approximate numbers (speed layer) until midnight when the batch job runs and corrects them. This needs to be clearly communicated in the serving API and any dashboards built on top of it.

---

## Decision 3: How to handle late-arriving data

**Context:** Mobile apps and offline-capable clients regularly send events with timestamps from hours or days in the past. The batch layer handles this naturally (next batch run picks them up), but the speed layer has already closed its windows.

**Options considered:**
1. Drop late events in the speed layer — simplest, but creates serving layer inconsistencies
2. Allow a configurable late data tolerance in speed layer windows (Spark watermarking)
3. Route late events directly to batch, skip speed layer entirely

**Decision:** Option 2 — Spark watermarking with 2-hour late tolerance.

**Rationale:** A 2-hour window covers the majority of real-world late events (users coming back online, app sync delays). Events older than 2 hours are rare enough that their absence from real-time metrics is acceptable — the next batch run will correct the historical record. Watermarking is a first-class feature in Spark Structured Streaming and adds minimal complexity.

**Consequences:** Real-time metrics can still be slightly wrong for up to 2 hours after an event occurs. The batch layer is the source of truth. Any dashboard showing "live" numbers must include a disclaimer that numbers finalize after the next batch run.

---

## Observations (updated as implementation progresses)

**Observation 1: The dual-maintenance burden is real from day one.**
Even before writing a line of code, I had to design the same transformation logic twice — once for the batch PySpark job and once for the Structured Streaming job. A simple "calculate revenue by product" requires different API calls, different window semantics, and different state management in each context. This is the primary reason Lambda is being displaced by Kappa and Lakehouse in modern systems.

**Observation 2: Lambda is easier to reason about than it is to operate.**
The conceptual model is clean: batch is slow and correct, speed is fast and approximate, serving merges them. But the operational reality is two separate job schedulers, two separate failure modes, two separate monitoring setups, and a serving layer that has to handle partial failures gracefully (e.g., batch job failed — do I serve stale batch + speed, or just speed?).

**Observation 3: The reprocessability advantage is real.**
When I introduced a bug in the revenue calculation logic, fixing it was straightforward: correct the code, re-run the batch job against historical data, serving layer automatically picks up the corrected output. In a pure streaming system, "re-run against historical data" requires replaying the entire Kafka log — which has its own complexity. Lambda's batch layer gives you a clean, simple reprocessing path.

---

*Log updated as implementation progresses. Last updated: — (implementation pending)*
