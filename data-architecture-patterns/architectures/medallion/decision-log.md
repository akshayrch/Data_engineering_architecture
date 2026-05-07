# Medallion Architecture — Decision Log

> Observations from implementing Medallion against the e-commerce clickstream dataset.

---

## Decision 1: Delta Lake vs. Apache Iceberg for the table format

**Context:** Medallion architecture benefits enormously from a modern table format that supports ACID transactions, schema evolution, and time travel. Two formats dominate: Delta Lake (Databricks-origin) and Apache Iceberg (Netflix/Apple-origin).

**Options considered:**
1. Delta Lake — tighter Databricks integration, simpler API, strong community
2. Apache Iceberg — open standard, better multi-engine support (Spark + Trino + Flink), hidden partitioning
3. Apache Hudi — strong for CDC and upsert patterns, but smaller community and more complex

**Decision:** Apache Iceberg for this implementation.

**Rationale:** Since the goal is to compare this against Lakehouse architecture (which will also use Iceberg), using the same table format isolates the architectural pattern from the table format choice. In practice, Delta Lake is the more common choice in Databricks-centric shops; Iceberg is preferred in multi-engine environments (reading Iceberg tables from both Spark and Trino/Athena without conversion).

**Consequences:** Iceberg's hidden partitioning is genuinely better than Delta's partition-by-column approach for e-commerce data — querying by event date doesn't require knowing the physical partition layout. However, Delta Lake's OPTIMIZE and VACUUM commands have a slightly friendlier UX for teams that don't want to manage file sizes manually.

---

## Decision 2: How to draw the Bronze → Silver boundary

**Context:** The line between "raw" (Bronze) and "clean" (Silver) is often debated. If Bronze has too many transforms, it loses its audit value. If Silver has too few, it becomes unusable without further prep.

**Options considered:**
1. Bronze = raw bytes, Silver = fully clean and enriched
2. Bronze = raw with metadata appended, Silver = deduplicated + typed + enriched
3. Bronze = raw + type coercion only, Silver = business-logic-free clean, Gold = business logic

**Decision:** Option 2 with a strict rule: Silver has zero business logic.

**Rationale:** The Bronze → Silver boundary should be about technical quality (is this a valid event? is it a duplicate? are the types right?) not business meaning. "Is this a fraudulent order?" is a business question — it belongs in a Gold table. "Is this event a valid JSON with required fields?" is a technical question — it belongs at the Silver boundary. Keeping this strict makes Silver reusable across teams without encoding assumptions about how any one team defines their business rules.

**Consequences:** Some transformations feel awkward. Sessionization (grouping page views into sessions) is technically a join operation but carries business meaning. Decision: sessionization is Silver (it's event attribution, not business logic) but session-level metrics (bounce rate, pages per session) are Gold.

---

## Decision 3: Orchestration — Airflow DAG structure for three-layer dependencies

**Context:** Bronze → Silver → Gold is a dependency chain. The DAG must ensure Silver doesn't run against incomplete Bronze, and Gold doesn't run against incomplete Silver.

**Options considered:**
1. Three independent DAGs with manual triggering or time-based scheduling
2. Single DAG with task dependencies (Bronze tasks → Silver tasks → Gold tasks)
3. Delta Live Tables / Databricks Workflows (managed dependency resolution)
4. Single DAG per domain (e.g., revenue DAG: ingest events → clean events → compute revenue)

**Decision:** Option 2 — single Airflow DAG with layered task groups.

**Rationale:** Domain-based DAGs (Option 4) sound intuitive but create hidden coupling — two domain DAGs that both depend on the same Silver table will race, and you end up with implicit ordering. A single DAG with explicit layer-based task groups makes the dependency chain visible and debuggable. Option 3 is excellent but ties the implementation to Databricks.

**Consequences:** The single DAG becomes large as the number of Gold tables grows. Mitigation: use Airflow's TaskGroup to organize by layer, and use SubDAGs (or dynamic task mapping in Airflow 2.3+) for domain-level Gold generation. Pipeline-level retries need to be configured at the task group level, not the DAG level.

---

## Observations (updated as implementation progresses)

**Observation 1: Bronze immutability is harder to enforce than to declare.**
The rule "never modify Bronze" is easy to state but requires active enforcement. Developers under pressure to fix a data issue will reach for the Bronze table. The solution is access controls: Bronze tables are written by ingest pipelines and read by Silver jobs — no one else has write access. Without this enforcement, Bronze's audit value erodes quickly.

**Observation 2: The quality check layer (Silver boundary) is where most engineering time goes.**
Defining "what makes a valid event" sounds simple. In practice: what do you do with an event where `user_id` is null (anonymous user or tracking failure)? What about `price = 0` on a purchase event (free item, data error, or promo code that reduced to zero)? Each edge case requires a decision and documentation. Great Expectations is invaluable here — it forces you to codify quality rules as executable assertions rather than tribal knowledge.

**Observation 3: Multiple Gold tables for the same concept create silent inconsistencies.**
Two teams independently built Gold tables for "daily active users." They used slightly different definitions (one included mobile web, one excluded it). Both claimed to be authoritative. The fix required a governance conversation, not a technical one. Medallion architecture needs a data catalog with ownership and a review process for new Gold table definitions.

**Observation 4: Iceberg time travel makes Bronze less critical for debugging.**
One expected use of Bronze was debugging: "what did the raw data look like before Silver cleaning?" But Iceberg time travel on Silver and Gold tables provides an equivalent capability — you can query the table as it looked at any past snapshot. This doesn't replace Bronze's compliance/audit value, but it reduces how often you actually need to query Bronze during debugging.

---

*Log updated as implementation progresses. Last updated: — (implementation pending)*
