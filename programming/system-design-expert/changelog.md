# Changelog

## v1.0.0 — Initial Release

**Added**
- Core `skill.md` with the 7-phase design methodology (Requirements → Estimation → Data Model → API Design → High-Level Design → Detailed Design → Bottleneck Resolution)
- Three operating modes: DESIGN MODE, CONCEPT MODE, CRITIQUE MODE
- Seven always-loaded quick decision trees (SQL vs NoSQL, queue vs pub/sub, monolith vs microservices, WebSocket vs SSE vs long polling, strong vs eventual consistency, cache write strategies, vertical vs horizontal scaling)
- Five reference files under `references/`, loaded selectively:
  - `networking-and-infra.md` — DNS, load balancing, caching, CDNs, proxies, availability math, scaling fundamentals
  - `data-layer.md` — SQL/NoSQL, ACID/BASE, CAP/PACELC, replication, sharding, consistent hashing, indexing, normalization
  - `architecture-patterns.md` — monolith/microservices, messaging patterns, event sourcing, CQRS, API gateways, REST/GraphQL/gRPC, real-time delivery
  - `distributed-systems-toolkit.md` — circuit breakers, rate limiting, service discovery, SLA/SLO/SLI, disaster recovery, auth protocols, geospatial indexing
  - `estimation-playbook.md` — estimation formulas plus one fully worked end-to-end example
- "Common Pitfalls to Actively Flag" section for self-correction during design work
- Mandatory "Trade-offs and what I'd reconsider" closing section for every DESIGN MODE response

**Design notes**
- This skill uses progressive disclosure (a `references/` folder) rather than a single monolithic file, given the breadth of system design as a topic — `skill.md` stays focused on methodology and decision-making, while domain depth lives in reference files loaded only when relevant.
- All reference content is an original synthesis of standard, industry-wide system design concepts, written independently rather than derived from any single course or text.

**Planned for future versions**
- A dedicated reference file for capacity planning under multi-region / geo-distributed constraints
- Expanded coverage of database internals (LSM trees vs B-trees, write-ahead logs) if requested
- A lightweight self-check rubric for grading a user's own draft design against the 7 phases
