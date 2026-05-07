# Project P6 — Multi-Tenant Feature Store (Bonus / Stretch)

**Phase:** 6 (System Design)
**Kickoff week:** 14 or 15 · **Ship week:** 15
**Effort:** ~6 hrs
**Primary tools:** Iceberg + Trino + Redis + Feast (or custom Python) + Airflow
**Deliverable type:** Mini-system + 4-page architecture writeup
**Prereq:** Projects P1, P2, P3, P5 are shipped and Week 14 is on track. **Skip this project if behind.**

---

## Why this project (and why it's optional)

Feature stores are ML-engineering adjacent, not core DE. But:
- FAANG staff DE interviews often touch ML data platforms — this signals breadth
- Multi-tenancy is the rarest-tested architecture skill and Week 14 covers it
- The combination of "online + offline parity" is a hard design problem that showcases you can handle it

**Skip if P1-P5 aren't shipped.** The baseline 5 projects are sufficient.

---

## Success criteria

You're done when:

- 3 feature groups defined: `user_session_features` (last 30 day aggregates), `item_features` (video metadata + popularity), `real_time_features` (current session state)
- Offline store: Iceberg tables, partitioned by date + feature group
- Online store: Redis with 100ms P99 read latency target
- Ingestion: batch job for offline (daily) + streaming for real-time (Flink/Spark SS)
- Tenant isolation: 3 tenants with namespace-separated feature stores, shared compute
- Offline/online parity test: same feature vector materialized by both paths, assert equality ± 1% tolerance
- 4-page architecture writeup in `README.md`

---

## Architecture

```
                      ┌──────────────────────────┐
                      │   Event streams (Kafka)  │  multi-tenant topics
                      └────────────┬─────────────┘
                                   │
                   ┌───────────────┴────────────────┐
                   ↓                                ↓
       ┌────────────────────┐          ┌───────────────────────┐
       │   Batch (Spark)    │          │   Streaming (Flink)   │
       │   daily features   │          │  real-time features   │
       └──────────┬─────────┘          └───────────┬───────────┘
                  ↓                                ↓
       ┌────────────────────────────────────────────────────────┐
       │                 Feast registry (metadata)              │
       └─────────────────────┬──────────────────────────────────┘
                             ↓
            ┌────────────────────────────────────┐
            │  Offline store: Iceberg            │  (training + backfill)
            │  Online store:  Redis Cluster      │  (inference)
            └───────────────────┬────────────────┘
                                ↓
            ┌────────────────────────────────────┐
            │  Feature retrieval SDK (Python)    │
            │  → used by training and serving    │
            └────────────────────────────────────┘
```

---

## Multi-tenancy model

**Choose: Bridge pattern** — shared compute, isolated storage namespaces.

Details:
- Each tenant: `tenant_id` is a first-class column + a namespace prefix on all tables/topics
- Iceberg: `tenant_a.user_features`, `tenant_b.user_features`, etc.
- Redis: key prefix `{tenant_id}:{feature_group}:{entity_id}`
- Kafka: topic-per-tenant or key-by-tenant depending on scale
- Compute: one Spark + Flink cluster shared across tenants, isolated via Spark application-level quotas
- RBAC: per-tenant IAM roles + row-level security on shared Trino
- Cost attribution: query tags + Prometheus labels on compute

Why bridge not silo? Silo (per-tenant cluster) is the clean architecture but wildly expensive at small tenant counts. Pool (everything shared, partition by tenant_id) is cheap but audit/isolation is complex. Bridge wins the middle ground.

---

## Critical design decisions

### 1. Offline/online parity (the #1 feature store problem)
Feast (or any store) has 2 paths — nightly batch job writes to Iceberg, streaming job writes to Redis. They MUST compute the same features the same way. How:
- Shared feature-definition library (Python functions registered once, used by both paths)
- Parity test job that runs daily: picks 1000 entities, recomputes features via batch + reads current Redis, diffs.

### 2. Point-in-time correctness
When building training data, you must NOT leak future features. This is brutal.
- Every feature has `event_timestamp` column in Iceberg
- Training assembly: asof-join (each label joined with feature values where `feature.ts <= label.ts`)
- Use Iceberg's snapshot-at-timestamp for deterministic training set generation

### 3. Feature freshness SLA
- Batch features: stale by up to 24h, acceptable
- Streaming features: stale by up to 5 min (watermark + checkpoint + sink latency)
- Monitoring: `feature_last_updated_ts` per feature group, alert if stale beyond SLA

### 4. Online store sizing
- Entities: 500M users × 50 features × 100 bytes ≈ 2.5 TB — fits in Redis Enterprise cluster
- Writes: ~100k/sec burst — Redis handles this with cluster + pipelining
- Reads: ~500k/sec at peak — need Redis Enterprise or DAX-like pattern

### 5. Redis vs DynamoDB vs ScyllaDB online store
Trade-offs:
- Redis: sub-10ms P99, but ops overhead, memory cost
- DynamoDB: managed, auto-scaling, but 10-20ms P99
- ScyllaDB: best write throughput, but adds a new runtime to your stack
- **Choose Redis** for latency-critical, fallback to DynamoDB for cost-optimized

---

## The interview payoff

If you built this, these questions become trivial:

- "How do you handle offline/online parity in a feature store?"
- "Design a multi-tenant data platform with shared compute"
- "What's point-in-time correctness and why does it matter?"
- "Redis vs DynamoDB vs HBase for an online KV store at scale"

These land you in Staff+ territory because most candidates wave at feature stores; you built one.

---

## Deliverable

`projects/P6/` folder:
- `feast_setup/` — feature store config + 3 feature groups defined
- `batch_compute/` — Spark job computing features → Iceberg offline store
- `streaming_compute/` — Flink job computing features → Redis online store
- `parity_test/` — daily job that diffs batch vs streaming
- `retrieval_sdk/` — Python package for features_for_training() and features_for_inference()
- `multitenancy/` — IAM + RBAC setup, per-tenant namespace isolation
- `README.md` (4-page architecture writeup)

---

## Skip criteria

Skip this project if on Week 14 any of the following are true:
- P3 not shipped
- P5 not shipped
- Any checkpoint from Phases 2–5 not green
- You're behind on STAR story writing

The 5 baseline projects + solid STAR stories > 6 projects + fuzzy stories. Be honest with yourself.
