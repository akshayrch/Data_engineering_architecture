# Week 1 — Architecture Foundations & Types of Pipelines

**Dates:** Apr 22 – Apr 26, 2026 _(starts Wednesday — Mon/Tue slots already passed)_
**Phase:** 1 (Foundations)
**Weekly budget:** ~13 hrs _(5-day week reduced to Wed–Sun; content compressed accordingly)_
**Theme:** _Wire the architect's mental model. Every pipeline shape has a name, a reason, and a rejected alternative._

---

## Why this week matters

Before any tool — Spark, Kafka, dbt — you need the mental scaffold a staff-level architect uses: what shape does this pipeline take, why this shape over others, what breaks first at scale. Without this, Weeks 2–15 are just tool memorization. With this, everything slots in.

---

## Learning objectives

By Sunday night you can, out loud, for any scenario:

1. Name the pipeline type (batch, micro-batch, stream, lambda, kappa, medallion, CDC, reverse ETL) and defend the choice
2. List the 5 canonical layers (ingest, store, transform, serve, observe) and the interface between each
3. Place any tool from your stack on the layer it owns and explain what it replaces
4. Give at least 2 trade-offs for every architectural choice (latency ↔ cost, consistency ↔ availability, flexibility ↔ performance)

---

## Daily breakdown

### Reading-pace reality check

Per 2-hr weekday slot: **~15–20 pages of dense reading with notes** is a sustainable target. DDIA runs ~12 pages/hr when you're actually internalizing it. Blog posts and short docs = 5–10 min each. The rule from here on: **one chapter OR one paper per session, not both.** If a reading overruns, stop and bookmark — finish it in a 20–30 min pre-work slot the next morning. Cramming to "finish the chapter" by skimming kills retention.

> **Where things stand entering Wednesday:** You've started DDIA Ch 2 and read ~12 pages. Mon and Tue slots are behind us. The week below picks up from where you are and fits everything into Wed–Sun.

---

### Wed Apr 22 (2 hrs) — Continue DDIA Ch 2 + pipeline shapes foundation

- 90 min read: Pick up DDIA Ch 2 from where you left off (~page 12). Read the next ~15 pages (graph databases + column-family section). **Stop when the 90 min is up** — don't push to finish the chapter in one go. Remaining pages carry to Thu pre-work.
- 15 min read: Jay Kreps — "Questioning the Lambda Architecture" (short blog, ~10 min — read it twice)
- 15 min write: Draft the first 3 rows of the 5-column table — _shape · ingest · store · transform · serve_ — for batch, micro-batch, stream. You don't need all 6 rows today; the rest come after Thursday's reading.

> **Thu pre-work (20–30 min before your session):** Finish the remaining pages of DDIA Ch 2. At that point the chapter is done and you can write the full data-model mini-table.

---

### Thu Apr 23 (1.5 hrs) — Architectural shapes + Lakehouse paper

- 30 min read: DDIA Ch 10 intro (~8 pages) — how DDIA frames batch
- 15 min read: DDIA Ch 11 intro (~5 pages) — how DDIA frames stream
- 15 min read: Databricks "Medallion Architecture" docs (1 page)
- 15 min write: Complete the 5-col table — add lambda, kappa, medallion rows. Add 2 pipeline sketches from the wild (daily sales batch, CDC-to-warehouse).
- 15 min read: Lakehouse CIDR 2021 paper — sections 1–2 (~4 pages of dense academic; section 3 carries to Fri pre-work)

> **Fri pre-work (15 min):** Finish Lakehouse paper section 3.

---

### Fri Apr 24 (1.5 hrs) — Trade-offs + Lakehouse write-up + flashcards

- 45 min read: DDIA Ch 9 selections on CAP + linearizability (~15 pages — skim for conclusions first, then re-read the section that surprises you)
- 30 min write: Two things — (1) 2-paragraph Lakehouse answer: _"Why does the lakehouse argue we don't need a separate lake + warehouse? Where might that argument still be wrong?"_ (2) CAP vs PACELC memo with 2 examples from systems you've built
- 15 min flashcards: 10 cards covering today's content

---

### Sat Apr 25 (4.5 hrs) — Architecture whiteboard day

- **Drill #1 (90 min):** Design a real-time fraud detection pipeline for a payments company. Constraints: 50K events/sec peak, <500ms decision latency, $500K/yr infra budget. No tool allowed until you've drawn the shape.
- **Drill #2 (90 min):** Design a batch daily-sales warehouse for a retail company with 200 stores, 10M events/day, 5-year retention.
- **Drill #3 (90 min):** Design a Netflix-style watch-history store (this is your LinkedIn Round 1 Q5 rehearsal). Focus only on shape + layer choices — no deep dive into any tool yet (that's later weeks).
- After each drill: list 3 alternative architectures you did NOT pick and 1 reason each

---

### Sun Apr 26 (4.5 hrs) — Consolidation + tool map + project setup

- 90 min: Read through all 3 drills from Saturday, rewrite each as if explaining to an interviewer in 5 minutes
- 60 min read: _Fundamentals of Data Engineering_ Ch 3 (DE Lifecycle) — lighter than DDIA, ~35 pages; read at a skim pace for the mental model, not for memorization. This is context, not core.
- 45 min write: "Tool ownership" grid. For 10 tools from your resume: _layer owned · what it replaces · 2 alternatives · 1 "don't use when"_ line.
- 30 min: Read Project P1 brief + Project P2 brief in `projects/` — decide which to start Week 4
- 45 min: Week 1 self-check (below) + environment setup (Docker, Databricks Community Edition account, local Spark)

---

## Must-read this week
- **[CORE]** DDIA Ch 2, Ch 10 intro
- **[CORE]** Lakehouse CIDR 2021 paper sections 1–3
- **[CORE]** Jay Kreps — "Questioning the Lambda Architecture"
- **[REF]** DataExpert.io — "Bootcamp Kickoff" (Jan 2025) for architecture mindset video

---

## Deliverable
- `notes/week_01/` folder in your project repo with:
  - `pipeline_shapes.md` (the 5-column table)
  - `tool_map.md` (the layer-ownership grid)
  - `drills/` with 3 architecture drills (drawings + 5-min scripts)
  - `flashcards.md` (~25 cards)

---

## Self-check (do Sunday, pass = move on)

Answer out loud, no notes:

1. Describe lambda vs kappa vs medallion in 60 seconds each, with when-to-use-what
2. A payments company does 50K TPS — streaming or batch? Defend in 3 reasons
3. What's wrong with choosing Kafka + Flink for a pipeline that only runs once a day?
4. CAP vs PACELC in 60 seconds — and which do real engineers actually care about?
5. You have 10 tools in your resume; for 3 of them, give the single best "do NOT use this when…" statement

If you can't answer 4/5 cold, repeat Saturday's drills before Week 2.

---

## Notes / Discoveries
<!-- Keep a running log of "aha" moments this week -->
