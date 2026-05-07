# Week 16 — Advanced System Design: Scale, Multi-Tenant, Cost, Edge Cases

**Dates:** Aug 3 – Aug 9, 2026
**Phase:** 7 (System Design + Interview Prep)
**Weekly budget:** ~18 hrs
**Theme:** _The second-order questions interviewers ask after you nailed the first diagram. Scale this 10x. Make this multi-tenant. Cut the cost in half. What if a region fails?_

---

## Why this week matters

Week 13 gave you the patterns. Week 14 gives you the rare moves — multi-tenancy, disaster recovery, hot/cold tiering, egress cost, data residency (GDPR), cross-region replication. This is where staff/principal candidates separate from senior.

Also: bonus project P6 if you're on track.

---

## Learning objectives

- Multi-tenancy patterns: silo, pool, bridge — storage, compute, governance
- Cost architecture: egress, tiering (hot/warm/cold), data lifecycle, request pricing (S3, DynamoDB)
- Disaster recovery: RPO vs RTO, cross-region replication, point-in-time recovery
- Data residency + GDPR + right-to-be-forgotten implications on architecture
- Scaling extremes: 100x growth, cold start, burst traffic
- Edge cases: late data, duplicate data, out-of-order data, schema drift, partial failures

---

## Daily breakdown

> **Pace check:** This week's reading is whitepapers and engineering blogs — more scannable than dense books. AWS whitepapers have good structure; read the sections that matter, skim the rest. Drills on Saturday are the core value — each 90-min drill is actually 90 min, not 45. If something slips, let it be a reading slot, not a drill.

### Mon (2 hrs) — Multi-tenancy
- 45 min read: Databricks + Snowflake multi-tenancy patterns + Salesforce's "Pillar" paper (yes, the one ICE uses too)
- 45 min read: AWS "SaaS Tenant Isolation Strategies" whitepaper
- 30 min write: 3-pattern comparison (silo/pool/bridge) with pros/cons for each layer (compute, storage, control plane)

### Tue (2 hrs) — Cost architecture
- 45 min read: AWS "Cost Optimization Pillar" whitepaper (DE sections)
- 45 min read: Snowflake cost optimization docs (credit math, resource monitors)
- 30 min write: 10-point checklist "how to cut a data platform bill 30%"
- 15 min drill: Back-of-envelope for S3 egress — 1 TB/day cross-region costs how much/month?

### Wed (2 hrs) — DR + cross-region
- 45 min read: "Building Resilient Systems with Amazon S3" + Snowflake cross-region replication docs
- 45 min read: Delta / Iceberg cross-region replication approaches
- 30 min write: Design doc template — "DR plan for a data platform" — with RPO/RTO targets, failover procedure, runbook

### Thu (1.5 hrs) — GDPR + data residency
- 45 min read: AWS "Data Residency" whitepaper + Snowflake data residency docs
- 30 min write: How do you architect for right-to-be-forgotten in a lake + warehouse? (tombstone tables, partitioned-for-delete, time-limited retention)
- 15 min flashcards

### Fri (1.5 hrs) — Scaling extremes + edge cases
- 45 min read: Uber "Scaling Uber's data platform to 100 PB" + Netflix "Handling 10B events/day" blogs
- 30 min drill: Take P1 design and 10x it. What breaks? What scales? What needs a rewrite?
- 15 min flashcards

### Sat (4.5 hrs) — 3 advanced-twist drills + start P6 (optional)

3 drills today — each adds a twist to a Week 13 pattern:

1. **(90 min) Multi-tenant real-time analytics.** Take Week 13 Pattern 1, now it must support 500 customer tenants, each with isolated queries but shared compute. Design the isolation model.
2. **(90 min) Cost-constrained rec-feed.** Take Pattern 2, now with a hard $50k/month infra ceiling. What changes? (pre-aggregation, tiering, cache hierarchy)
3. **(90 min) Global CDC with data residency.** Pattern 3, but US + EU + APAC regions, GDPR required, analytics global. What stays local, what replicates?

After drills: **(60 min) Start Project P6 (feature store)** if on track. Otherwise use this time to polish STAR stories.

### Sun (4.5 hrs) — Phase 6 Checkpoint part 1 + mock
- 2 hrs: **First full system-design mock.** Pramp, interviewing.io, or a peer. Real 60-minute mock. Get feedback.
- 90 min: Fix gaps from the mock
- 60 min: Week 14 self-check + flashcards
- 30 min: Review all 8 STAR stories — they should all be drafted by now (#1 billing, #2 MWAA, #3 GBO Kafka, #4 RCA, #5 Iceberg migration, #6 Cortex agents, #7 dbt migration, #8 GenAI failure detector)

---

## Must-read this week
- **[CORE]** AWS SaaS Tenant Isolation + Cost Optimization whitepapers
- **[CORE]** Snowflake cost + replication docs
- **[CORE]** Uber + Netflix scale blogs
- **[DEEP]** CAP/PACELC literature applied to real DR
- **[DEEP]** "Site Reliability Engineering" book Ch 22–25 (load balancing, chaos, cascading failures)

---

## Deliverable
- 3 advanced drill writeups in `notes/week_14/drills/`
- First mock interview feedback documented
- All 8 STAR stories drafted
- (If time) Project P6 scaffolded
- `notes/week_14/`:
  - `multitenancy.md`
  - `cost_checklist.md`
  - `dr_template.md`
  - `gdpr_patterns.md`
  - `mock1_feedback.md`
  - `star_stories_status.md`
  - `flashcards.md` (25 cards)

---

## Self-check (Sunday)

1. Silo vs pool vs bridge multi-tenancy — pick-each scenarios
2. S3 egress pricing — 1 TB cross-region, cross-continent, same-region — ballpark numbers
3. RPO vs RTO — define, and give realistic values for: a trading pipeline, a daily sales dashboard, a marketing attribution pipeline
4. Right-to-be-forgotten in a data lake — 3-step plan
5. At 100x scale, which layer of a batch pipeline breaks first: ingestion, storage, compute, serving? Why?
6. You got the mock feedback. What 3 things will you fix this week?

Pass = 5/6 cold + mock feedback incorporated.

---

## Notes / Discoveries
