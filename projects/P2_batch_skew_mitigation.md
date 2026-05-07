# Project P2 — Large-Scale Batch Pipeline with Data-Skew Mitigation

**Phase:** 2 (Spark Deep Mastery)
**Kickoff week:** 5 · **Ship week:** 6
**Effort:** ~10 hrs across weeks 5–6
**Primary tools:** PySpark on Databricks Community Edition + Delta/Iceberg + AQE
**Leverages existing:** Your 75B-row Pillar pipeline at ICE
**Deliverable type:** 4 implementations + comparison report

---

## Why this project

Your 75B-row Pillar pipeline is THE story. But the resume line ("80 parallel tasks, idempotent manifest") doesn't prove you understand why skew exists, how AQE changes the game post-Spark 3.0, or how to pick between 4 different fixes. This project forces you to build all 4 and measure them.

LinkedIn Round 1 Q4 — "optimize a Spark job with data skew and excessive shuffle" — becomes a story with actual numbers.

---

## Success criteria

You're done when:

- Synthetic skewed dataset (1B rows, 80/20 Pareto distribution on join key) generated
- Baseline join runs with NO mitigation — you've recorded shuffle bytes, runtime, task duration distribution, and where it hurts (probably spills to disk, maybe OOM)
- 4 mitigation implementations ship + benchmarked:
  1. **Salting** (random suffix on skewed keys in both sides)
  2. **AQE Skew Join** (`spark.sql.adaptive.skewJoin.enabled=true`)
  3. **Broadcast** (if other side is < 10GB, force broadcast)
  4. **Bucketing** (pre-sort + pre-bucket by join key to eliminate shuffle)
- Comparison report: runtime, shuffle bytes, cluster utilization, cost, code complexity
- A 2-page "Decision tree — which mitigation for which scenario" written from the results

---

## Data generation

Simulate skew that mirrors your production reality: a small number of heavy-hitter users/entities dominate.

```python
import random, uuid
from pyspark.sql import SparkSession
from pyspark.sql.functions import expr, lit, rand, when

spark = SparkSession.builder.getOrCreate()

# Left side: 1B rows, 80% of traffic concentrated on 0.1% of keys (~1000 "whale" keys)
df_left = (spark.range(1_000_000_000)
    .withColumn("join_key",
        when(rand() < 0.8,  (expr("cast(rand() * 1000 as int)")))   # whales
        .otherwise(expr("cast(rand() * 10000000 as int)"))            # the rest
    )
    .withColumn("amount", expr("rand() * 1000"))
    .withColumn("tx_id", expr("uuid()"))
)

# Right side: 10M rows, uniform distribution across 10M keys
df_right = (spark.range(10_000_000)
    .withColumn("join_key", expr("id"))
    .withColumn("user_name", expr("concat('user_', id)"))
)

df_left.write.parquet("/tmp/skewed_left")
df_right.write.parquet("/tmp/dim_right")
```

---

## Implementations

### 1. Baseline (no mitigation)
```python
res = (df_left.join(df_right, "join_key")
       .groupBy("join_key").sum("amount"))
res.write.parquet("/tmp/out_baseline")
```

Measure + record. Expected: 90%+ of work done in <5% of tasks. Spills. Slow.

### 2. Salting
```python
from pyspark.sql.functions import concat, lit, floor, rand, explode, array

N = 16  # salt factor
df_left_salted = df_left.withColumn("salt", floor(rand() * N))
df_right_exploded = df_right.withColumn("salt", explode(array([lit(i) for i in range(N)])))
res = (df_left_salted.join(df_right_exploded, ["join_key", "salt"])
       .groupBy("join_key").sum("amount"))
```
Trade-off: right-side becomes 16x bigger, but distributes evenly. Useful when skew is extreme and the "other" side is small enough to explode.

### 3. AQE Skew Join
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionFactor", "5")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256MB")
# Then run baseline join — AQE handles it
```
Modern approach. Works without code change. Know the config knobs cold.

### 4. Broadcast (when applicable)
```python
from pyspark.sql.functions import broadcast
res = df_left.join(broadcast(df_right), "join_key")
```
Only if right fits in executor memory. Know `spark.sql.autoBroadcastJoinThreshold` (default 10MB).

### 5. Bucketing
```python
# Pre-bucket both tables on join_key
df_left.write.bucketBy(128, "join_key").sortBy("join_key").saveAsTable("left_bucketed")
df_right.write.bucketBy(128, "join_key").sortBy("join_key").saveAsTable("right_bucketed")
# Now join — no shuffle needed
spark.table("left_bucketed").join(spark.table("right_bucketed"), "join_key")
```
Pays upfront cost on write, zero shuffle on read. Useful for tables joined repeatedly.

---

## Measurements to capture

For each of the 5 runs, record:

| Metric | How |
|---|---|
| Wall clock | `spark.sparkContext.statusTracker()` or Spark UI |
| Shuffle read/write bytes | Spark UI Stage page |
| Max/median/min task duration | Spark UI Stage task distribution |
| Peak memory per executor | Spark UI Executors page |
| Spill to disk bytes | Stage metrics |
| Cost (in Databricks DBUs if possible) | Databricks UI |
| Code complexity (LOC, config knobs) | Subjective but rate 1–5 |

---

## Deliverable format

`projects/P2/` folder:
- `gen/` — data generation scripts
- `runs/` — 5 Spark jobs (baseline + 4 mitigations)
- `measurements.csv` — metrics table
- `plans/` — screenshots / text of each physical plan
- `README.md` — the 2-page report with the decision tree
- `DECISION_TREE.md` — "when to use which mitigation"

---

## The decision tree (your output — fill in from results)

- Skew factor < 5x → default AQE usually enough. Set `adaptive.enabled=true`.
- Skew factor 5–50x → AQE Skew Join with tuned thresholds. First line of defense.
- Skew factor > 50x or AQE insufficient → Salting. Plan for 16–64 salt.
- Other side < 10GB → Broadcast. Fastest. Zero shuffle.
- Join performed > 10x on same pair of tables → Bucket them on ingestion. Permanent win.
- Mixed skew signals → Combine: bucket + AQE, or broadcast + salted fallback.

---

## STAR story derived

This becomes the technical backbone of **STAR story #10** (extension of Pillar pipeline story):

> _"My production pipeline had extreme key skew. A single whale account held 40% of the join keys. Default Spark defaulted to sort-merge join and spilled 2 TB to disk, running 45 min. I measured the distribution, applied AQE Skew Join as a first line — that alone dropped us to 12 min. For the top 5 whales, I layered salting with factor 32 — final runtime 3 min 20 sec. The rejected alternative was bucketing on ingestion, which I ruled out because..."_
