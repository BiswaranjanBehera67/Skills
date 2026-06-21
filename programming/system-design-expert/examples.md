# Examples

Real session outputs from the System Design Expert skill across all three modes.

---

## Session 01 — DESIGN MODE: Collaborative Code Editor

**Input:**
```
Design a collaborative code editor — multiple people editing the same file
in real time, like a simplified Google Docs but for code.
Assume ~50,000 concurrent active editing sessions at peak.
```

---

**Phase 1 — Requirements Clarification**

*Functional*
- Multiple users can edit the same file simultaneously, with changes visible to all participants in near real time.
- Users can see where other participants' cursors are.
- Edit history should be recoverable (undo, and ideally a version timeline).

*Non-functional*
- Latency is the dominant constraint here — edits feel broken above roughly 150ms of perceived lag. Ranking: latency > consistency > durability-of-every-intermediate-keystroke.
- Availability matters, but a few seconds of degraded sync during a failover is more acceptable than visibly wrong text appearing in someone's editor.

*Out of scope*
- Authentication/authorization beyond "user has a valid session."
- Syntax highlighting, linting, or any language-specific tooling — purely the real-time sync problem.

---

**Phase 2 — Back-of-Envelope Estimation**

Assumptions: 50,000 concurrent sessions at peak, average 4 participants per active document, each participant sends roughly 3 edit events per second while actively typing.

$$
\text{Concurrent documents} \approx \frac{50{,}000}{4} = 12{,}500
$$
$$
\text{Edit events/sec at peak} \approx 50{,}000 \times 3 = 150{,}000/\text{s}
$$

This is a write-heavy, low-payload-size, latency-critical workload — the opposite profile from a typical CRUD app. That single observation rules out a request/response REST model for the live-editing path before any other architecture decision gets made.

---

**Phase 3 — Data Model**

- `Document` — id, title, current full content snapshot, version number
- `EditOp` — id, documentId, authorId, operation (insert/delete + position + content), timestamp, sequence number
- `Session` — userId, documentId, cursor position, connection id

Access pattern: nearly all reads and writes are scoped to "the current state of one document" — there's no cross-document querying need in the live-editing path. That single fact pushes toward a model optimized for fast per-document reads/writes over one optimized for complex relational queries.

---

**Phase 4 — API Design**

```
openDocument(userId, documentId): DocumentState
applyEdit(userId, documentId, operation): AckOrConflict
subscribeToDocument(userId, documentId): EditStream
getHistory(documentId, fromVersion, toVersion): EditOp[]
```

`applyEdit` deliberately returns `AckOrConflict` rather than a plain boolean — multiple users editing concurrently means conflicting operations are a first-class expected outcome, not an edge case, and the API needs to be honest about that from the start.

---

**Phase 5 — High-Level Design**

Flow: `client → WebSocket gateway → document session service → operation log → broadcast to other connected clients`

- **WebSocket gateway** — every active editor holds one persistent WebSocket connection. This is non-negotiable given the latency requirement from Phase 1; long polling's request-cycle overhead would be immediately noticeable in this use case.
- **Document session service** — one logical owner per active document, responsible for applying incoming operations in order and resolving conflicts using an operational-transform or CRDT-style merge algorithm, then broadcasting the merged result to all connected clients for that document.
- **Operation log** — every accepted edit is appended here before being broadcast, giving durability and a replayable history independent of the in-memory session state.
- **Snapshot store** — periodic full-content snapshots so reconnecting clients (or someone opening the document fresh) don't need to replay the entire operation history from the beginning.

For inter-service communication: the gateway-to-session-service hop is a natural fit for a lightweight pub/sub layer (Phase 5 of the methodology's architecture-pattern reasoning) — each document acts like its own topic, and every connected client for that document is a subscriber.

---

**Phase 6 — Detailed Design**

Going deep on the piece that's actually hard here: **conflict resolution under concurrent edits.**

Two participants editing near the same position at nearly the same time is the core technical problem this entire system exists to solve. A naive "last write wins" approach silently destroys one user's work, which is unacceptable for a collaborative editor. The standard answer is either Operational Transformation (transform each incoming operation against all operations that happened concurrently with it, so the final state converges no matter what order operations arrive in) or a CRDT (a data structure specifically designed so concurrent operations are mathematically guaranteed to merge into the same final state regardless of arrival order, without needing a central transform step).

The practical trade-off: OT requires a central authority per document doing the transform work (simpler to reason about, but that authority is now a bottleneck and a single point of failure for that document specifically). CRDTs push the merge logic into the data structure itself, which scales better to peer-to-peer or multi-region setups but is a meaningfully harder data structure to design and debug correctly.

Given this system already has a natural "document session service" owning each document, OT is the more proportionate choice here — the central authority already exists for other reasons, so it isn't new complexity, whereas adopting a CRDT would be solving a peer-to-peer problem this system doesn't actually have.

---

**Phase 7 — Identify and Resolve Bottlenecks**

- **Single point of failure**: the document session service instance is, by design, the one authority for a given document. If it crashes mid-edit, every connected client for that document loses their live session. Mitigation: the operation log (already durable) lets a freshly-spawned session service for that document replay recent operations and rebuild state quickly — the crash becomes a brief reconnect, not data loss.
- **10× growth**: 500,000 concurrent sessions instead of 50,000 means roughly 1.5M edit events/sec. A single WebSocket gateway tier won't hold that — this is the point where you'd shard gateway instances by document ID (so all clients on one document land on the same instance, simplifying broadcast) behind a consistent-hashing-based router.
- **Partial outage**: if the snapshot store is briefly unavailable, the system can still serve from the operation log directly (slower reconstruction, but not a hard failure) — that's a deliberate design choice worth stating explicitly rather than discovering during an actual incident.

---

**Trade-offs and what I'd reconsider**

This design assumes one document session service instance "owns" a document at a time, which is simple and matches how OT naturally wants to work — but it does mean that document's edit throughput is capped by what a single instance can process. If a small number of documents ever needed truly massive concurrent editor counts (thousands of simultaneous editors on one file, well beyond this system's stated assumptions), I'd revisit toward a CRDT-based model that doesn't require single-document ownership at all. I'd also want real production data on actual conflict frequency before committing to OT over a simpler "lock the line being edited" approach — if concurrent edits to the *exact* same position turn out to be rare in practice, a much simpler design might suffice.

---
---

## Session 02 — CONCEPT MODE: Queue vs Pub/Sub for Order Processing

**Input:**
```
Should I use a message queue or pub/sub for processing e-commerce orders
after checkout?
```

---

It depends on what happens after checkout, and the answer is probably **both**, applied to different parts of the flow.

**The trade-off**: a message queue guarantees that exactly one consumer handles a given unit of work — which is what you want for the actual order fulfillment step (charging the payment, decrementing inventory). You do not want two different workers independently "fulfilling" the same order because they both picked it up off a shared channel. Pub/sub, by contrast, is for fan-out: multiple independent systems that each need to react to the same event without coordinating with each other.

**Where each fits in your actual flow**: "order placed" is genuinely one thing that multiple unrelated systems care about — inventory needs to decrement, a confirmation email needs to send, an analytics event needs to fire, a fraud-check might need to run. That's a pub/sub shape: one event, several independent subscribers, none of which should block on the others. But "process this specific payment" is work that must happen exactly once, with a clear owner and clear retry semantics if it fails — that's a queue.

**Concrete scenario**: publish a single `OrderPlaced` event to a pub/sub topic the moment checkout completes. The payment-processing service, inventory service, and notification service all subscribe independently. But the payment-processing service itself, internally, pulls its actual charge-the-card work off a dedicated queue — so that if two instances of that service are running, only one of them ever actually attempts to charge a given order.

---
---

## Session 03 — CRITIQUE MODE: Notification System Review

**Input:**
```
Review this design: we have a single PostgreSQL database storing all
notification records. When an event happens, our API server writes a
notification row directly to Postgres and then makes a synchronous HTTP
call to Firebase Cloud Messaging to push it to the user's phone, in the
same request. We're at about 2 million notifications/day right now but
expect to hit 50 million/day within a year.
</br>
Is this going to hold up?
```

---

Walking through where this breaks, in the order it'll actually break:

**The synchronous push call is the most urgent problem, not the database.** Right now, every API request that triggers a notification is blocking on a network call to an external service (FCM) before it can respond. If FCM has a slow moment — and every third-party push provider does, regularly — every request that happens to trigger a notification gets slow too, even though the actual business logic for that request had nothing to do with push delivery. At 2M/day this is already a live latency risk; at 50M/day, you're guaranteeing it. **Mitigation**: decouple this immediately — write the notification record, then enqueue the actual push send as an async job a worker picks up. The API response should never wait on FCM.

**The single Postgres instance is the second problem, and it's a scaling-math one.** 2M/day is about 23 writes/second on average — comfortably within what a single well-tuned Postgres instance handles. 50M/day is about 580 writes/second average, which is still plausible on a single primary depending on row size and indexing, *but* you haven't mentioned what your peak-to-average ratio looks like, and notification traffic in particular tends to be extremely bursty (a single popular event can trigger millions of notifications in a tight window rather than spread evenly across the day). If peak bursts run anywhere near 5–10× average, you're looking at 3,000–6,000 writes/second at peak on a workload that's currently architected as a single point of write contention. **Mitigation**: this is very likely where you want a queue in front of the database write itself, not just in front of the FCM call — let the API enqueue "a notification needs to be created and sent," and have workers drain that queue at a rate the database can sustain, rather than the database having to absorb whatever the traffic spike throws at it directly.

**What I don't have enough information to fully assess**: you haven't said whether notification records ever get read back at any real volume (a user-facing "notification history" screen, for instance). If they do, that read pattern — and whether it's hitting the same primary that's absorbing all these writes — matters a lot and I'd want to know before recommending read replicas or a separate read-optimized store.

**What I'd actually change, in priority order**: (1) move the FCM call off the synchronous request path into an async worker immediately — this is cheap to fix and is actively costing you latency today, not just at future scale; (2) put a queue in front of the database write path before you're anywhere near 50M/day, so the database is draining at a sustainable rate instead of absorbing bursts directly; (3) only *after* those two changes, revisit whether Postgres itself needs sharding — with the queue absorbing bursts, a well-indexed single primary (or a primary plus a couple of read replicas if the history-read pattern turns out to matter) may genuinely be enough even at 50M/day, and I wouldn't want to add sharding complexity you don't yet have evidence you need.

---
---

## Patterns Across Sessions

**DESIGN MODE earns its 7 phases by catching things a jump-to-architecture answer would miss.** In Session 01, Phase 1's non-functional ranking (latency over consistency) is what justified WebSocket over long polling in Phase 5, and Phase 3's access-pattern observation is what set up the OT-vs-CRDT reasoning in Phase 6. Each phase's output becomes an input to a later one — skipping straight to "here's the architecture" would have produced a design with no stated justification for any of its choices.

**CONCEPT MODE works best when the user's framing is a false binary.** Session 02's actual answer was "both, for different parts of the flow" — the skill's job was recognizing that the question as asked ("queue or pub/sub") didn't map cleanly onto the actual system, which has multiple distinct flows hiding inside one checkout process.

**CRITIQUE MODE is only useful when it's specific to the numbers given.** Session 03's critique ties every claim back to the user's actual stated traffic (2M/day now, 50M/day in a year) rather than offering generic "you should probably add caching and sharding" advice that would apply to literally any system. The prioritized fix list at the end — cheapest and most urgent fix first — is what makes the critique actionable instead of just diagnostic.

---

*Examples last updated: June 2026 · Skill version: v1.0.0*
