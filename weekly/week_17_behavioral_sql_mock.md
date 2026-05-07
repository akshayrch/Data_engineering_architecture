# Week 17 — Behavioral STAR, SQL Drills, Coding, Mock Week

**Dates:** Aug 10 – Aug 16, 2026
**Phase:** 7 (Interview Prep — closeout)
**Weekly budget:** ~18 hrs
**Theme:** _The parts everyone forgets to prepare: STAR fluency, SQL speed, Python quirks. And 3 mocks._

---

## Why this week matters

LinkedIn Round 3 is entirely behavioral. LinkedIn Round 1 Q1 + Q2 are SQL. Your SQL is strong but interview-SQL is a specific sport (time pressure, no docs, edge-case aware). Coding rounds at big tech require data-heavy Python (generators, context managers, decorators, collections, concurrency primitives).

---

## Learning objectives

- All 8 STAR stories memorized cold, with variants (2 min / 5 min / 10 min versions)
- SQL interview patterns: window functions (all variants), gaps-and-islands, sessionization, top-N per group, running/lagging/moving windows, pivoting, CTE recursion
- Python interview patterns: generators, itertools, collections (defaultdict/Counter/deque/OrderedDict), functools, context managers, concurrent.futures, asyncio basics, dataclasses, typing
- 3 mock interviews minimum

---

## Daily breakdown

> **Pace check:** No dense books this week. SQL drills are timed exercises, not reading. Python patterns = 30–45 min of coding practice. The STAR recording on Monday is uncomfortable and valuable — don't skip it. Mock #2 and #3 on Sat/Sun are the highest-ROI things in this entire 18-week plan.

### Mon (2 hrs) — STAR story drill day
- 30 min: Revisit the 8 STAR stories from Weeks 3, 5, 7, 9, 10, 11, 12 + GenAI failure detector (new, #8)
- 90 min: Record yourself telling each story. 2-min version. 5-min version. Listen back. Note: filler words, unclear impact numbers, missed "what I'd change."
- For each story, ensure these 6 elements are crisp:
  1. **Situation/context** in 2 sentences
  2. **Task/ownership** explicitly (what _you_ owned, not the team)
  3. **Actions** — 3 specific technical decisions, each with a defended trade-off
  4. **Result** — 2+ quantified outcomes
  5. **Learning** — 1 thing you'd change
  6. **Follow-up hooks** — 3 tangents the interviewer might dig into, with pre-rehearsed answers

### Tue (2 hrs) — SQL: window functions + top-N
- 30 min review: window function taxonomy (rank, dense_rank, row_number, lag, lead, first_value, last_value, nth_value, percent_rank, ntile)
- 90 min drill: 10 problems on StrataScratch/DataLemur — tagged "hard" + "window function." Time yourself: 10 min max per problem.

### Wed (2 hrs) — SQL: gaps-and-islands + sessionization + LinkedIn R1 Q1+Q2
- 30 min: The gaps-and-islands pattern — the 2 standard solutions (running difference, conditional sum)
- 90 min drill:
  - **LinkedIn R1 Q1** — top 10 users by watch hours last month. Write in 3 min.
  - **LinkedIn R1 Q2** — click-stream per-session duration by device_type. Sessions = 30-min inactivity gap. Write in 6 min.
  - 5 more gaps-and-islands or sessionization variants from StrataScratch

### Thu (1.5 hrs) — Python interview patterns
- 45 min drill: Implement from scratch — a streaming top-K with heapq; a LRU cache with OrderedDict; a generator that batches a file; a context manager for resource cleanup
- 30 min read: `functools` (lru_cache, partial, reduce, wraps); `itertools` (chain, groupby, tee, accumulate); `collections` (deque, Counter, defaultdict)
- 15 min drill: 5 LeetCode medium Python problems, timed

### Fri (1.5 hrs) — Coding patterns round 2
- 45 min: concurrency — concurrent.futures ThreadPool vs ProcessPool, when each; asyncio basics; the GIL
- 30 min drill: write a parallel ingestion loop with concurrent.futures — read 50 files, compute hash, write results
- 15 min flashcards

### Sat (4.5 hrs) — Mock #2 (full-length) + debrief
- **4 hrs: Full-length mock.** 1 SQL round (45 min) + 1 coding round (45 min) + 1 system design (60 min) + 1 behavioral (45 min). Use Pramp or a peer or interviewing.io.
- **30 min: Debrief.** What went well. What didn't. 3 follow-ups.

### Sun (4.5 hrs) — Mock #3 + final gaps
- 2 hrs: **Mock #3** — focus on your weakest round from Saturday
- 90 min: Close gaps from mocks #1, #2, #3
- 60 min: Final STAR polish — by end of Sunday, all 8 stories should be effortless in 2-min and 5-min variants. Put them in a single doc for quick review.

---

## Must-read this week
- **[CORE]** _Ace the Data Science Interview_ — SQL sections
- **[CORE]** StrataScratch + DataLemur — all DE-tagged "hard" problems
- **[CORE]** LeetCode top-100 SQL problems
- **[DEEP]** Fluent Python (Ramalho) — Ch 14 (generators), 17 (concurrency), 21 (async)
- **[REF]** `functools`, `itertools`, `collections` Python docs

---

## Deliverable
- **All 8 STAR stories polished** as `star_stories.md` — this is your interview bible. One doc, 8 stories, 2-min + 5-min variants of each.
- 3 mock interviews completed + debriefed
- `notes/week_15/`:
  - `sql_drill_log.md` (all problems solved, time taken, patterns used)
  - `python_patterns.md`
  - `mocks_feedback.md`
  - `star_stories.md` (the bible)
  - `flashcards.md`

---

## Self-check (Sunday — Phase 6 Checkpoint)

See master plan § Checkpoint 6.

Additionally:
1. Deliver each of your 8 STAR stories in exactly 2 minutes — no notes. Can you?
2. Solve a "sessionization with 30-min gap" SQL problem cold in 5 min
3. Pass 1 system-design mock with no "I don't know"s in deep-dive
4. 45-min SQL round with 3 problems, all solved, all optimal

Pass = checkpoint green + mocks show consistent B+ performance across all 4 rounds.

---

## Notes / Discoveries
