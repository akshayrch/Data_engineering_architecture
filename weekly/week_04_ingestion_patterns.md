# Week 4 — Ingestion Patterns: APIs, FTP, Files, Webhooks

**Dates:** May 11 – May 17, 2026
**Phase:** 2 (Ingestion Patterns & Cloud Pipelines)
**Weekly budget:** ~18 hrs
**Theme:** _The 80% of lead-DE work that nobody teaches: getting data reliably from 15 different sources._

---

## Why this week matters

Your resume has Kafka consumers, RabbitMQ listeners, Kinesis/Firehose, and Snowflake loads. But it's silent on the everyday stuff a lead DE deals with: a partner's REST API with flaky pagination, a vendor's SFTP drop at 3am, a webhook firehose, an SSE stream from a SaaS product. These are the ingestion breadths that separate a lead from an IC.

LinkedIn interviews rarely ask "design a Kafka consumer from scratch" — but they will ask "we have 30 partner data sources of every kind, how do you build the ingestion layer?"

---

## Learning objectives

- REST API ingestion: pagination (cursor/offset/link-header), auth (OAuth2/JWT/API-key), rate limiting, retry + backoff, idempotency keys, state/bookmark management
- Streaming APIs: Server-Sent Events (SSE), WebSockets, webhook receivers, long-polling — know when each applies
- Webhook receivers at scale: replay, dedup, signature verification, back-pressure, dead-letter queues
- FTP/SFTP patterns: file arrival detection, manifest-based loads (you do this at ICE — formalize it), file integrity (checksum + size), archival
- S3/GCS file landing patterns: event-driven (S3 events → Lambda/SQS), poll-based (Airflow sensors), manifest-driven
- Connector frameworks: Airbyte, Meltano, Fivetran — when to buy vs build
- Kafka Connect + Debezium as ingestion glue (source connectors)

---

## Daily breakdown

> **Pace check:** This week is mostly hands-on and docs/blogs, not dense book chapters. Blog posts = 5–10 min each. RFC sections = 10–15 min per section. The reading is spread thin; the time pressure here is building, not reading.

### Mon (2 hrs) — REST API ingestion patterns
- 45 min read: "Robust API ingestion" blogs — Stripe's pagination docs, GitHub's rate-limit docs, Shopify's REST Admin API patterns
- 30 min read: RFC 6749 (OAuth2) — just the client-credentials + refresh-token flows
- 45 min build: Write a resilient REST puller in Python:
  - Exponential backoff with jitter on 5xx / 429
  - Cursor pagination with state persistence (last cursor) to a local file or S3
  - OAuth2 token refresh transparent to caller
  - Rate-limit awareness (respect `Retry-After` header)
  - Idempotent writes to target (de-dup by event_id)
- Target API for lab: use a public one — Stripe test mode, GitHub Events API, NYC Open Data, or Polygon.io (finance — matches your domain)

### Tue (2 hrs) — Streaming APIs + webhooks
- 30 min read: MDN — Server-Sent Events (SSE) vs WebSockets fundamentals
- 30 min read: Stripe webhooks documentation in full (signing, replay, idempotency) — it's the gold standard
- 45 min build: SSE client that consumes a test stream (e.g., Wikipedia EventStream at `stream.wikimedia.org`) and writes to local Kafka
- 15 min build: Minimal webhook receiver (FastAPI) with signature verification + 200-ms ACK, enqueues to SQS/Kafka for downstream

### Wed (2 hrs) — Webhook receiver at scale (the lead-DE question)
- 30 min read: "How we scaled webhook ingestion" — Shopify + Vercel + Stripe engineering blogs
- 45 min build: Extend yesterday's webhook receiver with:
  - Dead-letter queue for signature failures
  - Event dedup table (Redis or Postgres with TTL)
  - Replay endpoint that reads from DLQ back into primary flow
  - Back-pressure: reject with 503 when queue depth > threshold
- 45 min write: Design doc — "How to ingest webhooks at 100k/sec peak." Architecture, dedup strategy, replay strategy, sizing math.

### Thu (1.5 hrs) — FTP/SFTP + manifest-driven loads
- 30 min read: Your own ICE Pillar pipeline story — reread your resume line about "idempotent manifest-driven batch assignment"
- 30 min build: Write a Python SFTP watcher:
  - Connects, lists new files since last bookmark
  - Downloads each, verifies checksum
  - Moves original to archive/ on the remote
  - Writes a manifest-of-arrivals to S3 for downstream Spark
- 30 min write: Document your ICE manifest-driven pattern formally — it becomes an interview answer. "For file-arrival-based ingestion with no duplicate guarantees, we wrote a manifest per micro-batch that…" — defend it.

### Fri (1.5 hrs) — S3 event-driven landing + connector frameworks
- 30 min read: AWS docs — S3 Event Notifications → SQS → Lambda patterns
- 30 min read: Airbyte + Meltano + Fivetran comparison — when is managed better, when custom?
- 15 min build: Wire S3 event → SQS → Lambda that triggers an Airflow DAG via REST (or emits to a Kafka topic)
- 15 min flashcards

### Sat (4.5 hrs) — Unified ingestion framework mini-project
Spend today building a **"ingestion framework" abstraction** — the kind a lead DE would design to let the team add new sources in a day:

- `IngestionSource` base class with: `list_new()`, `fetch(token)`, `checkpoint(token)`
- 4 concrete implementations:
  1. `RestApiSource` (cursor pagination, OAuth2)
  2. `WebhookSource` (HTTP receiver → Kafka)
  3. `SftpSource` (file watcher + manifest)
  4. `S3EventSource` (event-driven)
- Common plumbing: state persistence, retries, metrics emission, structured logging
- Airflow DAG that runs all 4 with identical monitoring
- README: "Add a new source in 30 lines" + how it ships to prod

This becomes part of your **P1.5 ingestion project** — a portfolio artifact that shows architectural thinking, not just coding.

### Sun (4.5 hrs) — Connector decision framework + consolidation
- 90 min build: Finish the framework. Write tests.
- 90 min write: **"Connector build vs buy" decision tree** for a lead DE. Questions:
  - Source changes frequency?
  - Volume + cost?
  - Team familiarity?
  - Compliance/isolation requirements?
  - How custom is the transform?
  - SLA required?
  Map to: buy Fivetran, buy Airbyte cloud, run Airbyte self-hosted, custom Python, Kafka Connect, Debezium.
- 60 min: Week 4 self-check + flashcards (25 cards)

---

## Must-read this week
- **[CORE]** Stripe webhook docs — the reference
- **[CORE]** Shopify + Vercel engineering blogs on webhook scaling
- **[CORE]** Airbyte docs — connector SDK overview
- **[CORE]** AWS docs — S3 Event Notifications + SQS/Lambda patterns
- **[DEEP]** RFC 6749 (OAuth2) + RFC 7519 (JWT)
- **[REF]** Debezium connector configs as prior art

---

## Deliverable
- `projects/P1.5_ingestion_framework/` — the 4-source framework shipped
- `notes/week_04/`:
  - `rest_pull_patterns.md` (pagination, auth, retry, rate-limit)
  - `webhook_scale_design.md` (100k/s design doc)
  - `ftp_manifest_pattern.md` (your ICE pattern formalized)
  - `s3_event_driven.md`
  - `connector_build_vs_buy.md` (decision tree)
  - `flashcards.md` (25 cards)

---

## Self-check (Sunday)

1. REST API returns 429 with `Retry-After: 60`. What do I do? What if I'm at 1000 concurrent pulls?
2. Webhook producer sends same event 3 times in 2 seconds due to their retry. How do I dedup without a global lock?
3. FTP drop arrives every night at 3am but sometimes 3:05 sometimes 4:00. How do I detect + process without polling every 10 sec?
4. I get a new CSV in S3 every 10 min. Lambda or Airflow sensor or EventBridge?
5. When do I Airbyte vs custom Python vs Kafka Connect? 3 scenarios.
6. SSE vs WebSocket vs webhook vs long-polling. One-line pick for each.
7. "Design ingestion for 30 partner APIs, 5 SFTP sources, 3 webhook firehoses." Give 2-min framework answer.

Pass = 6/7 cold.

---

## Notes / Discoveries
