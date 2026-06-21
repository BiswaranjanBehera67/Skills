# System Design Expert

> *There's no universally correct architecture — only the right trade-off for the stated constraints.*

A structured prompting skill that turns Claude into a principal-level systems architect. It encodes a disciplined 7-phase design methodology, a library of decision trees for the trade-offs that come up in nearly every system, and a set of original reference files covering networking, databases, architecture patterns, and distributed systems resilience.

---

## Why This Skill Looks Different From the Others

This skill is bigger in scope than a single concept, so it's built differently from a typical single-file skill. Instead of one `skill.md` containing everything, it uses **progressive disclosure**:

```
system-design-expert/
├── skill.md                                  ← always loaded: methodology + decision trees
├── README.md                                 ← this file
├── examples.md
├── changelog.md
└── references/                               ← loaded selectively, only when relevant
    ├── networking-and-infra.md
    ├── data-layer.md
    ├── architecture-patterns.md
    ├── distributed-systems-toolkit.md
    └── estimation-playbook.md
```

`skill.md` always loads and contains the methodology, the quick decision trees, and a map telling Claude which reference file to consult for which topic. The reference files only get pulled in when the conversation actually touches that domain — asking about caching loads `networking-and-infra.md`; asking about sharding loads `data-layer.md`. This keeps every response focused instead of dumping unrelated material into a question about, say, rate limiting.

---

## The Three Modes

### DESIGN MODE
*"Design a [system]" / "How would you build [X] at scale?" / interview prep*

Runs the full 7-phase process: Requirements → Estimation → Data Model → API Design → High-Level Design → Detailed Design → Bottleneck Resolution. Every design closes with a **"Trade-offs and what I'd reconsider"** section — an explicit, honest accounting of what's fragile or simplified.

### CONCEPT MODE
*"What's consistent hashing?" / "SQL or NoSQL for this?" / "Kafka or SQS?"*

Skips the full scaffolding. Direct answer → trade-off framing → one concrete scenario for choosing one over the other. Built for fast, focused questions that don't need a full design treatment.

### CRITIQUE MODE
*"Review my architecture" / "What's wrong with this design?" / "Is this scalable?"*

Jumps straight to Phase 7 — bottleneck hunting — applied to the user's actual design rather than starting fresh. Names specific failure scenarios tied to the user's stated numbers, not generic scalability platitudes.

---

## The 7-Phase Methodology

| Phase | What it forces |
|---|---|
| 1. Requirements Clarification | A ranked, scoped list of what the system must do — and explicitly what it doesn't |
| 2. Back-of-Envelope Estimation | Real numbers for traffic, storage, and bandwidth before any architecture decision |
| 3. Data Model | Core entities, relationships, and — critically — the dominant access pattern |
| 4. API Design | One clean function signature per capability, surfacing requirement gaps |
| 5. High-Level Design | The actual component diagram, narrated as a request flow, not a static list of boxes |
| 6. Detailed Design | Deep dives on the 1–3 most load-bearing components — not everything |
| 7. Bottleneck Resolution | Active interrogation: single points of failure, 10× growth, partial outages, cost |

This is the standard system design interview framework used across the industry — the value here isn't a novel process, it's that the skill *enforces* every phase instead of letting Claude jump straight to drawing boxes (the most common shortcut that produces shallow designs).

---

## Quick Decision Trees (always loaded, in `skill.md`)

These cover the trade-offs that come up constantly enough to warrant being in the always-loaded file rather than a reference:

- SQL vs NoSQL
- Message queue vs pub/sub
- Monolith vs microservices
- WebSocket vs SSE vs long polling
- Strong vs eventual consistency
- Cache-aside vs write-through vs write-behind
- Vertical vs horizontal scaling

Each one states the decision rule **and** what you give up by choosing it — per the skill's core principle that no architecture choice is presented without naming its cost.

---

## Reference File Map

| File | Covers |
|---|---|
| `networking-and-infra.md` | DNS, load balancing, caching strategy, CDNs, proxies, availability math, scaling |
| `data-layer.md` | SQL/NoSQL, ACID/BASE, CAP/PACELC, replication, sharding, consistent hashing, indexing, normalization |
| `architecture-patterns.md` | Monolith/microservices, message queues, pub/sub, event sourcing, CQRS, API gateways, REST/GraphQL/gRPC, real-time delivery |
| `distributed-systems-toolkit.md` | Circuit breakers, rate limiting, service discovery, SLA/SLO/SLI, disaster recovery, auth (OAuth/OIDC/SSO/TLS), geospatial indexing |
| `estimation-playbook.md` | Every estimation formula, plus one fully worked example end-to-end |

---

## Tips

**Give real constraints, not just a system name**
"Design a notification system" gets a generic, assumption-heavy answer. "Design a notification system for 5 million users, mostly mobile push, must survive a 10-minute outage of any single dependency" gets a sharply tailored one. The more constraints in Phase 1, the better everything downstream.

**Use CONCEPT MODE for fast trade-off questions mid-design**
If you're deep in a design conversation and just need "wait, should this be a queue or pub/sub," you don't need to restart the whole methodology — that single question gets a direct CONCEPT MODE answer inline.

**Push back on the trade-offs section**
The "Trade-offs and what I'd reconsider" section at the end of every DESIGN MODE response is the most useful place to dig in further — ask "what would actually break first if we 10×'d this," and the skill will go deeper on exactly that bottleneck.

**Use CRITIQUE MODE on your own real designs**
Paste an actual architecture diagram or description and ask "what's wrong with this." The skill is calibrated to name specific failure scenarios tied to your stated numbers, not generic "consider adding more redundancy" feedback.

---

## Files in This Skill

| File | Description |
|------|-------------|
| [`skill.md`](./skill.md) | The methodology, decision trees, and reference map — always loaded |
| [`README.md`](./README.md) | This file |
| [`examples.md`](./examples.md) | Real outputs across all three modes |
| [`changelog.md`](./changelog.md) | Version history |
| [`references/`](./references/) | Domain-specific knowledge, loaded selectively |

---

## Version

Current: `v1.0.0` — see [`changelog.md`](./changelog.md)
