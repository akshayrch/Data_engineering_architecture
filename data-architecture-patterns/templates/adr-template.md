# ADR-XXXX: [Short title — imperative sentence describing the decision]

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX
**Deciders:** [Names or roles of people involved in the decision]
**Architecture:** Lambda | Kappa | Medallion | Lakehouse | Cross-cutting

---

## Context

*What is the situation that requires a decision? Describe the forces at play — technical, organizational, resource constraints. What problem are we solving?*

*Keep this factual and neutral. Don't editorialize — just describe the situation.*

---

## Decision Drivers

*What criteria matter most for this decision? List 3-5 things that the decision must optimize for.*

- Driver 1 (e.g., operational simplicity — small team, can't manage two systems)
- Driver 2 (e.g., correctness — late data must not be dropped)
- Driver 3 (e.g., cost — storage budget is constrained)
- Driver 4 (e.g., reusability — multiple teams will depend on this)

---

## Options Considered

### Option 1: [Name]
*Brief description of the option.*

**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

---

### Option 2: [Name]
*Brief description of the option.*

**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

---

### Option 3: [Name] *(if applicable)*
*Brief description.*

**Pros / Cons:** ...

---

## Decision

**We chose Option [N]: [Name].**

*One paragraph explaining why. Reference the decision drivers. Be specific — "we chose this because it best satisfies drivers 1 and 3" is better than "it was the simplest option."*

*If this was a close call, say so and explain what tipped it.*

---

## Consequences

### Positive
- Consequence 1
- Consequence 2

### Negative / Tradeoffs
- Consequence 1 (what we're giving up or accepting as a cost)
- Consequence 2

### Risks
- Risk 1 — and what we'll do if it materializes
- Risk 2

---

## Implementation Notes

*Any specific implementation details that are important to capture alongside the decision. Code patterns, configuration values, or operational steps that encode the decision.*

```python
# Example code snippet if relevant
```

---

## Follow-up Actions

- [ ] Action 1 — Owner — Due date
- [ ] Action 2 — Owner — Due date

---

## References

- Link to relevant documentation
- Link to prior art or related ADRs
- Link to benchmarks or data that informed the decision

---

*ADR format adapted from Michael Nygard's original proposal. The goal is to capture not just what was decided, but why — including the context that made this the right decision at this point in time, which may not be the right decision later.*
