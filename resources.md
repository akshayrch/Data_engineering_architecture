# Resources — Curated Reading / Watching / Practice List

Priority key:
- **[CORE]** — do not skip. This is the 20% that gives 80%.
- **[DEEP]** — go here after CORE if you want depth on the topic.
- **[REF]** — reference / look up when needed, don't read cover-to-cover.

---

## Books

### Architecture + Foundations
- **[CORE]** _Designing Data-Intensive Applications_ — Martin Kleppmann. Chapters 2, 3, 5, 6, 7, 10, 11 are mandatory. The single most valuable DE book in existence.
- **[CORE]** _The Data Warehouse Toolkit_ — Kimball & Ross. Chapters 1–5 on dimensional modeling; Chapter 7 for SCD depth.
- **[DEEP]** _Building Evolutionary Architectures_ — Ford/Parsons/Kua. For architecture mindset, not DE-specific.
- **[DEEP]** _Fundamentals of Data Engineering_ — Reis & Housley. A solid modern survey; read chapters 1–3, 6–9.
- **[DEEP]** _The Data Vault Guru_ — Patrick Cuba. If Data Vault 2.0 comes up, this is the reference.

### Spark / Databricks
- **[CORE]** _Learning Spark, 2nd Edition_ — Damji et al. Ch 1–8; mandatory for PySpark + Structured Streaming foundation.
- **[CORE]** _Spark: The Definitive Guide_ — Chambers & Zaharia. Ch 14–20 for internals (Catalyst, Tungsten, execution).
- **[DEEP]** _High Performance Spark_ — Karau & Warren. The tuning bible. Ch 4, 5, 7 specifically.
- **[REF]** Databricks docs → official AQE, skew join, bucketing pages. Bookmark.

### Streaming
- **[CORE]** _Kafka: The Definitive Guide, 2nd Edition_ — Shapira, Palino, et al. Ch 2–6 (producer, consumer, internals). Ch 11 (stream processing).
- **[DEEP]** _Streaming Systems_ — Akidau, Chernyak, Lax. THE book on streaming theory — watermarks, windowing, exactly-once. Ch 1–4 mandatory for Flink/Beam understanding.
- **[DEEP]** _Kafka Streams in Action_ — Bejeck. Only if targeting Kafka Streams / ksqlDB roles.

### Warehouse / Lakehouse
- **[CORE]** _Snowflake: The Definitive Guide_ — Joyce Kay Avila. Ch 2 (architecture), Ch 4 (micro-partitions), Ch 9 (optimization), Ch 14 (cost).
- **[CORE]** _Apache Iceberg: The Definitive Guide_ — Bauer/Merced/Hughes. All of it — this format is taking over.
- **[DEEP]** _dbt Learn_ + _Analytics Engineering with SQL and dbt_ — Maraca & Teixeira.

### Interview-specific
- **[CORE]** _Ace the Data Science Interview_ — Huang & Singh. SQL + stats patterns.
- **[CORE]** _Designing Data-Intensive Applications_ (again — but now for interview review)
- **[DEEP]** _System Design Interview Volume 2_ — Alex Xu. Chapters on metrics monitoring, recommendation systems, newsfeed, real-time analytics.

---

## Papers (high signal)

- **[CORE]** Google — "MapReduce: Simplified Data Processing on Large Clusters" (2004). Historical foundation.
- **[CORE]** Google — "The Dataflow Model" (VLDB 2015) — watermark + trigger semantics. Required for Flink depth.
- **[CORE]** Databricks — "Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores" (VLDB 2020).
- **[CORE]** Netflix — "Iceberg at Netflix" talks (search YouTube) + Apache Iceberg spec.
- **[CORE]** Google — "Spanner: Google's Globally Distributed Database" (OSDI 2012). External consistency, TrueTime.
- **[DEEP]** "Kafka: a Distributed Messaging System for Log Processing" — Kreps et al. LinkedIn, 2011.
- **[DEEP]** "Apache Flink: Stream and Batch Processing in a Single Engine" — Carbone et al., 2015.
- **[DEEP]** "Photon: A Fast Query Engine for Lakehouse Systems" — Databricks, SIGMOD 2022.
- **[DEEP]** "Lakehouse: A New Generation of Open Platforms..." — Armbrust et al., CIDR 2021.

---

## Courses / Videos

### Spark (in order of depth)
- **[CORE]** Databricks Academy — "Apache Spark Programming with Databricks" (free)
- **[CORE]** Jaceklaskowski's _Mastering Apache Spark_ free online book (jaceklaskowski.gitbooks.io)
- **[DEEP]** Tathagata Das talks on Structured Streaming (Spark Summit)
- **[DEEP]** Holden Karau conference talks — search YouTube for "High Performance Spark"

### Kafka
- **[CORE]** Confluent Kafka 101 series (free on developer.confluent.io)
- **[CORE]** Jun Rao's "Kafka Internals" talks
- **[DEEP]** Tim Berglund's Kafka internals deep-dives on YouTube

### System Design for DE
- **[CORE]** ByteByteGo YouTube channel — Alex Xu's system design shorts
- **[CORE]** Jordan Has No Life (YouTube) — DE system design breakdowns; best free resource for DE-specific system design
- **[CORE]** Stanford CS 245 / CMU 15-445 lectures on database internals (free on YouTube)

### DataExpert.io (since you have access)
Prioritize in this order based on your gaps:
1. **Advanced Spark on Databricks** (Jan 2025 cohort) — Weeks 4–6 of this plan
2. **Real Time Data (Spark and Kafka Streaming)** (Jan 2025 cohort) — Weeks 7–8
3. **Dimensional + Fact Data Modeling** (Community Edition) — Weeks 2–3
4. **Apache Spark Fundamentals** — pre-read for Week 4
5. **Data Modeling on Iceberg** (Analytics Camp Apr 2025) — Week 3 + Week 10
6. **Change Data Capture (CDC) and Analytical Patterns** (Oct 2024) — Week 12
7. **Airflow + Trino** (Jan 2025) — Week 11
8. **Data Quality Patterns** (Community Edition) — Week 12
9. **Applying Analytical Patterns** — Week 15 SQL drill
10. **Interview Skills** (Data Engineer Interview Skills) — Week 15–16

---

## Documentation (bookmark these)

- Apache Spark docs — spark.apache.org/docs/latest/
- Apache Iceberg spec — iceberg.apache.org/spec/
- Apache Kafka docs — kafka.apache.org/documentation/
- Apache Flink docs — flink.apache.org/docs/stable/
- Snowflake docs — docs.snowflake.com (especially "Query Optimization" + "Micro-partitions")
- Databricks docs — docs.databricks.com (especially Photon + AQE + Delta)
- dbt docs — docs.getdbt.com (incremental materializations page is required reading)
- Airflow docs — airflow.apache.org/docs/apache-airflow/stable/

---

## GitHub repos (read the code)

- **[CORE]** `apache/spark` — read at minimum: `sql/catalyst/optimizer/Optimizer.scala`, `sql/adaptive/AdaptiveSparkPlanExec.scala` (to understand AQE)
- **[CORE]** `apache/iceberg` — the spec + Java table API
- **[CORE]** `dbt-labs/dbt-core` — the incremental materialization macros
- **[DEEP]** `apache/kafka` — consumer coordinator source (for rebalance internals)

---

## Practice platforms

- **[CORE]** StrataScratch — SQL practice, filter to "Data Engineer" tag
- **[CORE]** DataLemur — SQL practice with explanations
- **[CORE]** LeetCode — Database tag + Python medium for coding rounds
- **[DEEP]** Pramp — free mock interviews (DE/SWE)
- **[DEEP]** interviewing.io — paid but high quality

---

## Blogs worth subscribing to

- Netflix TechBlog — netflixtechblog.com (especially Data Engineering tag)
- Uber Engineering — eng.uber.com
- LinkedIn Engineering — engineering.linkedin.com
- Databricks Blog — databricks.com/blog
- Confluent Blog — confluent.io/blog
- dbt Labs Blog — getdbt.com/blog
- Airbnb Data Science — medium.com/airbnb-engineering
- Pinterest Engineering — medium.com/pinterest-engineering
- `eczachly` on Substack — Zach Wilson's DE content; heavy signal

---

## Flashcard decks to build

Build these yourself as you go — the act of making them is where learning sticks. One deck per phase:

1. `phase1_architecture.md` — 40 cards on lambda/kappa/medallion/modeling
2. `phase2_spark.md` — 60 cards on Spark internals + tuning
3. `phase3_streaming.md` — 40 cards on Kafka + streaming semantics
4. `phase4_warehouse.md` — 40 cards on Snowflake + dbt + Iceberg
5. `phase5_orchestration_dq.md` — 30 cards on Airflow + DQ + CDC
6. `phase6_system_design.md` — 30 cards on patterns + scale estimation numbers

Total ~240 cards. Do 20 reviews/day starting Week 4. By Week 16 you'll have seen each card 5+ times.

---

## Scale-estimation numbers to memorize

Any DE system design interview will require back-of-envelope math. Memorize these.

| Thing | Number to remember |
|---|---|
| 1 KB average row size | 1 GB = ~1M rows |
| 1 TB compressed Parquet (snappy) | ≈ 3–5 TB uncompressed; ≈ 1–3B rows of typical event |
| S3 PUT latency | 50–100ms typical, up to 200ms |
| S3 throughput | 3,500 PUT/s, 5,500 GET/s per prefix |
| Kafka producer throughput | 100k–1M msgs/s per broker (varies with batch size) |
| Kafka partition throughput | ~10MB/s write, 40MB/s read practical |
| Spark shuffle ideal partition size | 128–200 MB |
| Snowflake micro-partition size | 50–500 MB compressed (~16 MB target uncompressed) |
| DynamoDB single-partition throughput | 3k RCU / 1k WCU |
| Typical fact-table fact row | 100–500 bytes |
| 1 day of "big" — YouTube video views | ~10B |
| 1 day of "medium" — Netflix play events | ~200M |
| 1 day of "small" — trading exchange | ~1B orders, ~75B matches (you live here) |

---

_Keep this file open when planning each week's study. Update as you find new gems._
