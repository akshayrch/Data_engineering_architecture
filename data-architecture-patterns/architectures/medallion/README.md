# Medallion Architecture

> Raw is sacred. Refined is useful. Curated is what the business actually needs.

---

## What it is

Medallion architecture organizes data into three progressive quality layers — Bronze, Silver, and Gold — each representing a higher level of refinement, trust, and business alignment. It was popularized by Databricks as the recommended pattern for building Lakehouse solutions but applies broadly to any multi-hop data pipeline.

The defining principle is **incremental quality improvement**: you never modify or delete raw data. You transform it forward. Each layer adds cleaning, conformance, enrichment, or aggregation — and if something goes wrong at any layer, you can reprocess forward from the clean layer behind it.

---

## The three layers

### Bronze — Raw Ingestion
Everything lands here exactly as it arrived. No cleaning, no transformation, no schema enforcement. Bronze is your audit trail.

- Append-only (never delete, never update)
- Schema-on-read — store as-is, validate later
- Includes metadata: ingestion timestamp, source system, file name, batch ID
- Partitioned by ingestion date, not event date
- Retention: forever (or as long as compliance requires)

**For our dataset:** Raw Kafka event JSON, exactly as published by the web/mobile clients, landed to S3 with ingestion timestamp appended. Schema errors, malformed events, and duplicates all land in Bronze.

### Silver — Cleaned and Conformed
Bronze data cleaned, deduplicated, validated, and conformed to a canonical schema. Silver is the "single source of truth" for analytical work.

- Deduplication applied (event ID based)
- Schema validation and enforcement
- Null handling and type coercion
- Business logic-free — Silver knows nothing about KPIs
- Joined with reference data (product catalog, user profiles) for enrichment
- Partitioned by event date (not ingestion date)
- Retention: typically 2–5 years

**For our dataset:** Validated events with canonical types, deduplicated by `event_id`, enriched with `product_name`, `category`, `user_segment` from dimension tables.

### Gold — Business Aggregates
Silver data aggregated, modeled, and shaped for specific business use cases. Gold tables are what analysts, dashboards, and ML models consume directly.

- Aggregated metrics: daily revenue by product, weekly cohort retention, funnel conversion rates
- Star schema or wide denormalized tables optimized for query performance
- Business logic lives here: revenue = quantity × price - discount - returns
- Multiple Gold tables for different domains (marketing, finance, ops, ML)
- Retention: typically indefinite (aggregated, low storage cost)

**For our dataset:** `gold_daily_revenue`, `gold_funnel_metrics`, `gold_user_cohorts`, `gold_product_performance`, `gold_cart_abandonment_rates`.

---

## Applied to our dataset

```
BRONZE                      SILVER                      GOLD
──────────────────────────────────────────────────────────────
raw_events/                 events_clean/               daily_revenue/
  page_view_raw               page_view                   by_product
  add_to_cart_raw             add_to_cart                 by_category
  purchase_raw                purchase                    by_region
  checkout_raw                checkout
                            user_sessions/              funnel_metrics/
                              session_attributed          daily_conversion
                              events                      step_drop_off

                            product_enriched/           user_cohorts/
                              events joined               30d_retention
                              to catalog                  ltv_estimate

                                                        cart_abandonment/
                                                          rate_by_segment
                                                          recovery_revenue
```

---

## Architecture Diagram

See [diagrams/architecture.mmd](./diagrams/architecture.mmd)

---

## When to use Medallion

Medallion is the right choice when:

1. **Multiple teams consume the same data** — Bronze/Silver/Gold creates clear ownership boundaries. The ingest team owns Bronze, the data engineering team owns Silver, domain teams own their Gold tables.
2. **Data quality is a first-class concern** — the layered approach makes quality issues visible and traceable. You can always answer "where did this bad data come from?"
3. **You need an audit trail** — Bronze is immutable and serves as the legal/compliance record
4. **Your pipelines have multiple consumers with different needs** — one Silver table, many Gold views shaped per team
5. **You're building a Lakehouse** — Medallion is the natural organizational pattern for Delta Lake or Iceberg-based architectures

---

## When NOT to use Medallion

Medallion is the wrong choice when:

1. **You need real-time results** — Medallion is inherently a batch/micro-batch pattern. It doesn't natively answer "what's the revenue right now?" without adding a streaming layer.
2. **Storage cost is critical** — you're storing essentially 3 copies of your data (different transformations, but same volume). At petabyte scale this is significant.
3. **Your pipeline is simple** — a two-hop ETL (raw → marts) doesn't benefit from the overhead of three layers and their associated orchestration
4. **Your organization is small** — Medallion's multi-team ownership model is most valuable in organizations with 5+ data engineers. Smaller teams pay the organizational overhead without the benefit.

---

## Key tradeoffs

**The audit trail is genuinely valuable.** When a Gold metric looks wrong, the investigation path is clear: Gold → Silver → Bronze → source. You can trace any number back to the raw event that produced it. This is architecturally elegant and practically very useful — especially in regulated industries.

**Storage costs are real.** Bronze + Silver + Gold for a high-volume dataset means you're storing the same event approximately three times (raw, cleaned, aggregated). Delta Lake's Z-ordering, compaction, and Iceberg's hidden partitioning help, but don't eliminate the overhead. Budget accordingly.

**Medallion doesn't solve streaming.** A common misconception: Medallion is an organizational pattern, not a streaming architecture. You can run Medallion on batch Spark jobs, on Delta Live Tables, or on streaming pipelines — but the architecture doesn't prescribe how the data flows between layers. Many teams run Bronze ingestion as streaming (Kafka → Bronze in near-real-time) but Silver and Gold as batch (hourly or daily). This is a valid hybrid but requires careful thought about consistency.

**Gold table proliferation is a real risk.** Once teams can create Gold tables, they will — often duplicating logic and creating competing definitions of "revenue" or "active user." Good Medallion governance requires a catalog (Hive, Glue, Unity Catalog) and clear ownership policies.

---

## Implementation plan (coming soon)

- `implementation/bronze/` — Kafka → S3 Bronze ingestion (Spark Structured Streaming)
- `implementation/silver/` — Bronze → Silver cleaning and enrichment (PySpark)
- `implementation/gold/` — Silver → Gold aggregations (PySpark / dbt)
- `implementation/orchestration/` — Airflow DAGs managing the three-layer pipeline
- `implementation/quality/` — Great Expectations checks at Silver and Gold boundaries

---

## Further reading

- Databricks, "The Medallion Architecture" — original formulation
- Delta Lake documentation on table properties and data layout
- *The Data Warehouse Toolkit* by Ralph Kimball — Gold layer modeling patterns
