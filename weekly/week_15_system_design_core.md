# Week 15 — Data Engineering System Design: Core Patterns

**Dates:** Jul 27 – Aug 2, 2026
**Phase:** 7 (System Design + Interview Prep)
**Weekly budget:** ~18 hrs
**Theme:** _Whiteboard muscle. Four canonical DE system-design patterns and the rhetoric to defend them._

---

## Why this week matters

You've built the chops. Now you learn the interview game. System design is 40–60% of a staff DE interview and it has its own structure, pacing, and rhetorical devices. This week = the drill book.

---

## Learning objectives

- Master the 6-step DE system-design framework (Clarify → Estimate → Architect → Deep-Dive → Trade-Offs → Productionize)
- Back-of-envelope fluency — throughput, storage, cost, scaling numbers
- 4 canonical patterns cold: real-time analytics, recommendation/feed, CDC→warehouse, fraud/anomaly detection
- Vocabulary for trade-offs: consistency vs availability, cost vs latency, batch vs stream vs hybrid, column vs row store, push vs pull
- Speed: deliver each pattern in ~45 min without blanking

---

## The 6-step framework (memorize verbatim)

1. **Clarify (3–5 min)**: ask 5 questions — data source? volume? latency requirement? query shape? SLAs? budget? PII?
2. **Estimate (3–5 min)**: back-of-envelope numbers — events/sec, GB/day, TB/year, reads/sec. Tie to cluster size.
3. **Architect (8–12 min)**: draw the 5-layer diagram (ingest, store, transform, serve, observe). Name tools with 2-sentence reasons.
4. **Deep-dive (10–15 min)**: interviewer picks a layer. You go deep. This is where the weak candidates crack.
5. **Trade-offs (5–8 min)**: articulate 3 alternatives rejected + 1 thing you'd change at 10x scale.
6. **Productionize (5 min)**: failure modes, monitoring, DQ, on-call, cost ceiling.

---

## Daily breakdown

> **Pace check:** System design week shifts from reading to *doing*. The reading slots (Alex Xu, Netflix/Uber blogs) are 30–45 min of genuine reading. The drill slots are where the time actually goes — 45–75 min each. If the drill is running long, that's the right thing to run over on. Don't cut the drill to finish reading.

### Mon (2 hrs) — Framework + back-of-envelope
- 45 min read: Alex Xu _System Design Interview Vol 2_ — "Metrics Monitoring and Alerting" chapter
- 30 min read: Jordan Has No Life (YouTube) — 3 DE system design breakdowns, 10 min each
- 45 min drill: **Back-of-envelope drill.** Given "1B users × 5 events/day × 500 bytes/event," how big is daily data? Monthly? 7-yr retention with 3x replication compressed 5:1? How many partitions?

### Tue (2 hrs) — Pattern 1: Real-time analytics platform (LinkedIn R2 Q1)
- 45 min: Read or watch a reference design (Netflix Keystone or Uber M3 blogs)
- 75 min: **Drill pattern #1 end-to-end, 45-min target time.** Scenario: "Design a real-time viewing events pipeline for a video streaming platform. 500M users, 10B events/day, sub-second analytics dashboards."
  - Write out all 6 steps of framework in your notes
  - Record yourself reading it out loud — time it
  - Refine once

### Wed (2 hrs) — Pattern 2: Recommendation / feed (LinkedIn R2 Q6)
- 45 min: Read Alex Xu Vol 2 "News Feed System" + Airbnb's "Druid at Airbnb" blog
- 75 min: **Drill pattern #2.** Scenario: "Design user watch-history + recommendation feed for a video platform." This is Project P4 — written spec becomes the deliverable.
  - Full 6-step
  - Deep-dive on storage (wide-column for history vs keyed KV for recs)
  - Serving: precomputed vs online scoring

### Thu (1.5 hrs) — Pattern 3: CDC → warehouse
- 30 min: Skim Airbnb, LinkedIn, Shopify CDC architecture blog posts
- 60 min: **Drill pattern #3.** Scenario: "Design an end-to-end CDC pipeline from an operational MySQL cluster to analytics warehouse, 1TB DB, 100M events/day of changes, <5min latency for analytics."

### Fri (1.5 hrs) — Pattern 4: Fraud / anomaly detection
- 30 min: Read Stripe Radar architecture blog + Lyft fraud detection blog
- 60 min: **Drill pattern #4.** Scenario: "Design a fraud detection pipeline for a payments platform. 50k TPS peak, <500ms decision latency, features include user history + device fingerprint + merchant signals, ML model updated daily."

### Sat (4.5 hrs) — Drill day — repeat all 4 patterns
- For each pattern, 60 min: whiteboard cold (no notes), time yourself, record on phone, listen back
- Identify 3 things that tripped you up. Write them down.
- Do 1 more fresh pattern as bonus: "Design a clickstream analytics platform for an e-commerce site."

### Sun (4.5 hrs) — Weak-area remediation + Project P4 spec
- 2 hrs: Fix the 3 things that tripped you up — usually one of: estimate math, tool choice defense, or deep-dive depth
- 2 hrs: **Ship Project P4 written spec** for "user watch-history + recommendation feed." This is the only project delivered as a written architecture spec (no code). It should be a polished 8–12 page doc you could hand to an interviewer.
  - Sections: Requirements, Back-of-envelope, Architecture diagram, Ingestion, Storage, Transform, Serving, Failure modes, Monitoring, Cost estimate, 10x scale considerations, 3 rejected alternatives
- 30 min: Flashcards + Week 13 self-check

---

## Must-read this week
- **[CORE]** Alex Xu _System Design Interview Vol 2_ — Metrics Monitoring, News Feed, Real-time Analytics chapters
- **[CORE]** Jordan Has No Life YouTube — DE system design breakdowns
- **[CORE]** Netflix TechBlog — Keystone pipeline
- **[CORE]** Uber Eng — Aggregate streaming analytics (M3)
- **[DEEP]** Airbnb Engineering — Druid + CDC infrastructure posts
- **[DEEP]** Stripe / Lyft fraud detection architecture blogs

---

## Deliverable
- **Project P4 shipped** as written spec (8–12 pages)
- `notes/week_13/`:
  - `design_framework.md` (the 6 steps memorized)
  - `drills/` with 5 patterns (4 assigned + 1 bonus), each with full 6-step writeup + audio recording
  - `back_of_envelope_cheatsheet.md`
  - `flashcards.md` (20 cards on design heuristics)

---

## Self-check (Sunday)

1. Recite the 6-step framework in 60 seconds
2. Pick a pattern, run through it in 5 min (compressed) cold
3. Estimate storage for 10B events/day × 1KB × 365 days × 3 replicas compressed 5:1 → answer instantly
4. For the real-time viewing pipeline, give 3 tools for each of 5 layers with 1-line defenses each
5. Can you do the rec-feed deep-dive on storage choice (Cassandra vs HBase vs DynamoDB vs BigTable) in 2 min cold?

Pass = all 5 cold + P4 spec is interview-presentable.

---

## Notes / Discoveries
