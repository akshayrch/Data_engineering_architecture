# Lambda vs. Kappa: A Direct Comparison

> Both architectures solve the same problem: how to serve real-time and historical analytics from a streaming data source. They just disagree on whether you need two pipelines or one.

---

## The fundamental disagreement

**Lambda's position:** Batch and stream processing are fundamentally different. They have different performance characteristics, different failure modes, and different operational models. Trying to unify them into a single system creates a system that does both things badly. Accept the dual-layer complexity — it gives you the best of both worlds.

**Kappa's position:** The batch layer only exists because early streaming systems weren't powerful enough. Modern streaming systems (Kafka + Flink/Spark) can handle everything batch can. Maintaining two implementations of the same business logic is an unacceptable cost. Collapse it into one.

Both positions are defensible. The right answer depends on your context.

---

## Head-to-head: same e-commerce dataset

| Dimension | Lambda | Kappa |
|---|---|---|
| **Codebase** | Two codebases: PySpark (batch) + Spark Streaming | One codebase: Spark Structured Streaming |
| **Latency** | Speed layer: ~30s. Batch layer: hourly/daily | Single pipeline: ~30s (Spark) or ~1s (Flink) |
| **Correctness** | Batch layer is always correct (full recompute) | Streaming job is correct if watermarks and state are right |
| **Late data** | Batch picks it up naturally next run | Requires watermark configuration; data past watermark is dropped |
| **Reprocessing** | Re-run batch job against S3 history | Replay Kafka log (requires long retention or S3 archive) |
| **Bug fix workflow** | Fix code → re-run batch → batch output is corrected | Fix code → new consumer group → replay from offset |
| **Failure recovery** | Batch: re-run. Speed: replay from Kafka offset | Restore from checkpoint or replay from Kafka offset |
| **Operational load** | High: two schedulers, two monitoring setups, two failure modes | Medium: one pipeline, but stateful streaming is harder to debug |
| **Team skill requirement** | Batch SQL/PySpark + basic streaming | Deep streaming: exactly-once, watermarks, stateful ops |
| **Storage** | S3 for batch output + speed layer state (Redis/etc.) | S3 for output + Kafka log (expensive for long retention) |

---

## Where Lambda clearly wins

**Guaranteed correctness for historical data.** Lambda's batch layer processes complete data from an immutable source (S3). It cannot miss late events because it reprocesses everything. If a bug is found, fixing it and re-running the batch job is a one-step operation. The history is corrected automatically.

**Simpler business logic per layer.** PySpark batch code is easier to write, test, and reason about than equivalent stateful streaming code. Complex aggregations (cohort analysis, LTV calculation, multi-step funnel attribution) are straightforward in batch and require significant care to implement correctly in streaming. Lambda lets you write the hard logic in the batch layer where mistakes are recoverable.

**Your batch jobs already exist.** The most common Lambda adoption path: you have mature batch pipelines and need to add real-time on top. Lambda is the natural fit. Migrating those batch jobs to streaming is risky and expensive; Lambda lets you add speed without disrupting what works.

---

## Where Kappa clearly wins

**Single business logic implementation.** A revenue calculation that runs in both batch and streaming must be maintained in two places. When the business changes how it defines "gross revenue," you change it in two codebases. This is Lambda's fundamental weakness and Kappa's fundamental strength. One codebase, one definition, one test suite.

**Modern streaming handles the workload.** For the e-commerce dataset, Spark Structured Streaming handles daily aggregates (long tumbling windows), cohort analysis (session windows), and real-time metrics (short windows) in a single job. The argument that streaming can't do what batch can is less true every year.

**Operational simplicity in the long run.** One pipeline means one monitoring setup, one on-call runbook, one set of SLOs to track. Lambda's operational overhead is constant across the life of the system. Kappa's streaming complexity is a one-time investment.

---

## The nuanced reality

**Kappa works best when your Kafka log IS your archive.** If you have 90-day Kafka retention (or managed tiered storage), reprocessing is genuinely simple: set the offset back, replay. If you don't have long retention, you end up building a separate archival layer (S3 as a historical log), and now your "simple" Kappa system has two storage layers — you've quietly recreated Lambda.

**Lambda's dual-maintenance problem is manageable with discipline.** Teams that run Lambda successfully use a "shared library" approach: business logic is implemented once as pure functions in Python, then called from both the batch job and the streaming job. The framework-specific code (Spark batch vs. Spark Streaming) is thin wrappers. This doesn't eliminate the problem but reduces it significantly.

**For new systems in 2025+, Kappa (or Lakehouse) is the default.** Lambda made sense when streaming systems couldn't handle the full workload. That constraint is largely gone. Starting a new system today with Lambda means accepting dual-maintenance complexity upfront. Start with Kappa or Lakehouse and add batch layers only if streaming genuinely can't meet your requirements.

---

## Decision framework

Choose Lambda when:
- You have mature, correct batch pipelines you don't want to rewrite
- Your late data tail is extreme (> 24 hours) and you cannot drop those events from real-time views
- Your team is stronger in batch than streaming
- Correctness of historical data is non-negotiable and you don't trust your streaming reprocessing path

Choose Kappa when:
- You're building from scratch
- Your team is comfortable with streaming and stateful operations
- Business logic changes frequently (one codebase to update)
- Your Kafka log retention covers your reprocessing horizon
- Sub-second latency requirements rule out batch

---

## Verdict for the e-commerce dataset

**Kappa wins** for this use case, with one caveat.

The e-commerce clickstream has bounded late data (mobile apps batch for at most 6 hours), manageable complexity (aggregations and sessions, not complex event processing), and frequent business logic changes (promotional mechanics, attribution models). Kappa's single codebase means a promotion rule change is one commit, one deploy — not two.

The caveat: Kappa requires a serious Kafka retention strategy. A 7-day hot Kafka retention backed by S3 archive is the minimum viable setup. Without that, "just replay the log" fails for anything older than a week.
