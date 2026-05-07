# Data Architecture Patterns

> Same dataset. Four architectures. One verdict.

This repository is a hands-on study of the four most important data architecture patterns a staff-level data engineer needs to internalize — not just know by name, but understand deeply enough to defend in a design review.

Each architecture is implemented against the **same e-commerce clickstream dataset** so the tradeoffs aren't theoretical. They show up in actual code, actual latency numbers, and actual operational complexity.

---

## Why this exists

Most engineers learn architecture patterns by reading a blog post and moving on. I'm doing something different: implementing each pattern against identical requirements, documenting every tradeoff I encounter, and writing up which one wins and why.

The goal isn't to collect technologies. It's to build the architectural judgment that lets you walk into a room and say *here's what we should build and here's why we shouldn't build the other thing.*

---

## The Four Architectures

| Architecture | Core Idea | Best For | Complexity |
|---|---|---|---|
| [Lambda](./architectures/lambda/) | Separate batch + speed layers, merge at serving | Mixed real-time + historical with different SLAs | High — two codebases |
| [Kappa](./architectures/kappa/) | Stream-only, reprocess from log when needed | Unified streaming with reprocessability | Medium — one codebase, harder ops |
| [Medallion](./architectures/medallion/) | Bronze → Silver → Gold quality layers | Incremental data quality, multi-team consumption | Low-Medium — intuitive layering |
| [Lakehouse](./architectures/lakehouse/) | ACID transactions + analytics on object storage | Everything — the modern default | Medium — depends on table format maturity |

---

## The Dataset: E-Commerce Clickstream

All four implementations use the same domain: an e-commerce platform generating user behavior events.

See [dataset/README.md](./dataset/README.md) for the full schema. At a glance:

- **`page_view`** — user, session, product, timestamp, referrer
- **`add_to_cart`** — user, product, quantity, price
- **`checkout_start`** / **`checkout_complete`** — order lifecycle events
- **`purchase`** — final order with line items and payment info
- **`product_impression`** — search and recommendation exposures

This dataset is ideal because it has:
- High-velocity streaming (thousands of events/sec at peak)
- Late-arriving data (mobile apps batching offline events)
- Slowly changing dimensions (product catalog, user profiles)
- Both real-time needs (cart abandonment alerts) and historical needs (cohort analysis, revenue reporting)
- ACID requirements (inventory counts, order deduplication)

---

## Repository Structure

```
data-architecture-patterns/
├── README.md                          ← You are here
├── dataset/
│   └── README.md                      ← Full event schema + sample data
├── architectures/
│   ├── lambda/
│   │   ├── README.md                  ← Pattern overview, when to use, tradeoffs
│   │   ├── decision-log.md            ← My observations from implementing this
│   │   ├── diagrams/
│   │   │   └── architecture.mmd       ← Mermaid diagram
│   │   └── implementation/            ← Code (coming soon)
│   ├── kappa/
│   ├── medallion/
│   └── lakehouse/
├── comparisons/
│   ├── lambda-vs-kappa.md             ← Head-to-head: streaming architectures
│   ├── medallion-vs-lakehouse.md      ← Head-to-head: layered architectures
│   └── when-to-use-what.md            ← Decision framework: all four
└── templates/
    └── adr-template.md                ← Architecture Decision Record template
```

---

## How to read this repo

**If you're comparing architectures:** Start with [comparisons/when-to-use-what.md](./comparisons/when-to-use-what.md) for the decision framework, then drill into the specific head-to-heads.

**If you're studying one architecture:** Go to its folder, read the `README.md` first, then the `decision-log.md`. The README has the textbook view; the decision log has what I actually learned.

**If you're evaluating for a project:** The `decision-log.md` files are the most useful. They document tradeoffs I encountered in the actual implementation, not tradeoffs I read about.

---

## Roadmap

- [x] Repository structure and documentation framework
- [ ] Lambda — batch layer (PySpark), speed layer (Spark Streaming), serving layer
- [ ] Kappa — unified streaming pipeline with Kafka + Flink/Spark Structured Streaming
- [ ] Medallion — Bronze/Silver/Gold with Delta Lake or Iceberg
- [ ] Lakehouse — full Iceberg implementation with time travel + ACID transactions
- [ ] Performance benchmarks across all four
- [ ] Final verdict document

**Next stack:** Spark · Kafka · Airflow · Apache Iceberg · Delta Lake

---

## Architecture Decision Records

Every non-obvious design decision in this repo is documented using the [ADR template](./templates/adr-template.md). ADRs capture the context, the options considered, the decision made, and the consequences — including the ones I didn't anticipate.

---

*Built by [Akshay Reddy Chada](https://linkedin.com/in/akshayrch) as part of deliberate preparation for staff/lead data engineering roles.*
