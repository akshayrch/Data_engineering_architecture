# Project P4 — User Watch-History + Recommendation Feed (Architecture Spec)

**Phase:** 6 (System Design)
**Kickoff week:** 13 · **Ship week:** 13
**Effort:** ~6 hrs
**Deliverable type:** Written architecture spec (8–12 pages) — **no code**, this is a pure design exercise
**Leverages existing:** None — this is greenfield by design, mirrors LinkedIn Round 2 Q6

---

## Why this project

LinkedIn Round 2 Q6 is literally: _"Design an end-to-end architecture for user-watch-history and recommendation feed, including ingestion, storage, and reporting layers."_

This is also the most common FAANG DE system design question. The difference between a strong answer and a staff-level answer is depth in 3 places:
1. Storage choice for history (OLTP vs column vs wide-column vs key-value)
2. Serving-layer topology (precompute vs online, KV cache, CDN)
3. Model-data hand-off (offline training → online serving — feature store)

The project is a polished architecture spec you can hand to any interviewer — and iteratively improve across Weeks 13–15 as you mock-interview it.

---

## Success criteria

You're done when the spec exists as a 8–12 page markdown doc with:

- [ ] Requirements section with 5 explicit clarifications made + assumptions stated
- [ ] Back-of-envelope calculations (users, events, storage, bandwidth, QPS)
- [ ] Architecture diagram (5 layers: ingest, store, transform, serve, observe)
- [ ] Tool choices with 2-line defense each + 1 rejected alternative
- [ ] Deep-dive on each of the 5 layers
- [ ] "Top-10 videos" query flow end-to-end (LinkedIn Round 1 Q1)
- [ ] Failure modes mapped (producer failure, Kafka down, cache miss storm, model serving failure)
- [ ] 10x scale writeup (what breaks, what scales)
- [ ] Cost estimate at baseline + at 10x
- [ ] Data residency / GDPR considerations

---

## Required clarifications (ask in interview before designing)

1. **Scale.** Users? Events/day? Concurrent viewers? Peak QPS on feed?
2. **Latency.** Feed refresh SLA? History write-visible latency? Analytical query latency?
3. **Consistency.** Is "I watched this 5 min ago" OK to miss? (usually yes) What about "did I finish the episode"? (usually no)
4. **History use cases.** Continue-watching? Analytics? Recsys training data? ALL of them have different storage needs.
5. **Rec feed characteristics.** Personalized? Trending? Category-based? Cold-start for new users?
6. **Budget.** Is this Netflix-scale or Disney-scale or a startup?
7. **Team.** How big? How sophisticated? Who owns model training?

**Rule:** spend 3–5 min on clarifications in interview. Candidates who skip this miss points.

---

## Baseline assumptions (for your spec)

- 500M users
- Avg 5 watch sessions/day per active user (≈ 40% DAU)
- → 1B sessions/day
- → ~10B events/day (including play/pause/seek)
- Each event ~500 bytes → 5 TB/day raw
- 1-yr retention → ~1.8 PB
- Feed QPS: 200M DAU × 3 refreshes/session × 5 sessions = 3B views/day → peak ~100k QPS
- History writes: 10B/day → avg 115k writes/sec, peak ~500k

---

## Architectural decisions to make + defend

### 1. Ingestion layer
- **Choose**: Kafka (most likely) vs Kinesis vs Pulsar
- **Defense**: throughput, multi-DC, ecosystem, operator talent
- **Trade-off**: Kafka has steep ops learning curve; Kinesis has fewer knobs but AWS lock-in; Pulsar geo-rep is best but ecosystem thinner

### 2. Storage — "watch history" (hot writes, hot reads per user)
Candidate choices: Cassandra, ScyllaDB, DynamoDB, BigTable, HBase, Aerospike
- **Recommend Cassandra or DynamoDB.** Defend on:
  - Multi-region writes (DyDB global tables vs Cassandra multi-DC)
  - Cost at 10B writes/day
  - Query pattern: partition by user_id, sort by watch_time desc. Perfect for wide-column.
  - Opposed: not Postgres (doesn't scale to this write rate without sharding complexity)

### 3. Storage — analytics copy of watch history
- **Choose**: Iceberg on S3 (for ML + analytical queries) partitioned by day + bucket by user_id
- **Defense**: schema evolution, time travel, OSS, queryable from both Spark and Trino
- **Why duplicate?** Because the operational store is optimized for per-user reads; analytical needs full-scan-friendly

### 4. Transform
- **Batch**: Spark on Iceberg daily → feature tables
- **Streaming**: Flink for near-real-time aggregations (top-N trending, session metrics) → Redis/Cassandra for serving
- **Defense lambda here** — rare, but justified because offline training batch jobs and online trending both need the same raw events

### 5. Serving
- **Feed composition**:
  - Cold-path pre-compute: batch job nightly → Cassandra/DynamoDB with pre-computed top-200 per user
  - Warm path: online re-ranking with recent history + context (device, time of day)
- **Cache**: Redis cluster in front of Cassandra for P99 tail latency
- **API**: GraphQL or thrift, 20ms P99 SLA

### 6. Recsys model plane
- **Offline training**: Spark ML or PyTorch, features from Iceberg
- **Feature store**: Feast or in-house — critical for offline/online parity
- **Online inference**: TF Serving or Triton, embedded in the serving API
- **Deferred**: don't go deep on the model itself — this is a DE interview, not ML

### 7. Observability
- **Pipeline metrics**: Prometheus + Grafana
- **Data quality**: Great Expectations on Iceberg tables + OpenLineage
- **Model quality**: drift detection + A/B test framework

---

## The "top 10 videos last month" flow (LinkedIn R1 Q1 embedded)

To answer the SQL question baked into this domain, trace the flow:

1. All watch events land in Kafka → SS consumer → Iceberg bronze partitioned by day
2. Daily Airflow DAG: Spark job aggregates events by video_id → video-level fact
3. dbt model: `fct_video_watch_hours` with columns `(video_id, date, total_seconds)`
4. Materialized query in Snowflake or Trino:
```sql
SELECT video_id, SUM(total_seconds) / 3600.0 AS hours
FROM fct_video_watch_hours
WHERE date >= DATEADD(day, -30, CURRENT_DATE)
GROUP BY video_id
ORDER BY hours DESC
LIMIT 10
```

That's the answer to LinkedIn R1 Q1. Interview twist: ask "what if I want this in real-time?" — then you pivot to Flink session aggregation → Redis/Cassandra → served by API, materialized hourly top-10.

---

## Failure modes to cover

- **Kafka partition down**: replication factor 3, min.insync.replicas 2 — writes continue
- **Hot partition in Cassandra** (celebrity user problem): randomize partition key with user_id + day bucket
- **Cache miss storm on cold event**: jittered warm-up + negative cache
- **Serving latency spike under load**: circuit breaker + bulkhead + regional failover
- **Model serving down**: fallback to "popular in your country" generic feed
- **Data pipeline stops for a day**: continue-watching served from Cassandra (not pipeline-dependent); analytics backfills when pipeline recovers

---

## 10x scale writeup

At 100B events/day (10x baseline):

- Kafka: 10 clusters with MirrorMaker or multi-DC? Probably. Partition count 10x.
- Cassandra: more nodes; check if token range splits hold up. Evaluate migrating to ScyllaDB.
- Iceberg: partitioning strategy needs bucket(user_id, 1024) up from 128.
- Serving: regional reads; more Redis cache. Possibly edge cache via CDN for non-personalized items.
- Cost: linear + overhead. Expect 12–15x cost for 10x load if not careful.
- Pipeline: streaming batch window may need to shrink from 5 min to 1 min to keep window sizes reasonable.

---

## Cost estimate (rough order-of-magnitude)

Per month:

- Kafka (MSK or equivalent): $20k-40k
- Cassandra/DynamoDB for history: $80k-150k
- Iceberg S3 + compute: $30k-50k
- Streaming compute (Flink): $15k-25k
- Serving layer + cache: $20k-40k
- Observability + misc: $5k-10k
- **Total: ~$170k-315k/mo** for 500M users / 10B events/day

---

## Deliverable

`projects/P4/ARCHITECTURE_SPEC.md` — polished markdown, 8–12 pages, with all of the above.

Iterate on it across Weeks 13–15 as you mock-interview variants.

---

## How to practice with this spec

- Week 13: write it once cold, time yourself (target: 60 min)
- Week 14: re-do it with a "multi-tenant" twist (500 customer orgs sharing the platform)
- Week 14: re-do with a "cost-constrained" twist ($50k/mo budget)
- Week 15: verbal-walkthrough in a mock interview, get feedback, revise

---

## Tips for the interview version

- Do not start drawing tools. Start with 5 questions + assumptions + back-of-envelope. 70% of candidates skip this and lose.
- When asked to deep-dive, commit — go 15 min on one layer rather than 5 min on three.
- State rejected alternatives proactively. "I chose Cassandra, not Postgres, because…"
- End with "at 10x I would change X, Y, Z" — this signals staff-level thinking.
