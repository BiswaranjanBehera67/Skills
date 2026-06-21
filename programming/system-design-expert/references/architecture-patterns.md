# Architecture Patterns

Original reference notes — load when the conversation touches monolith vs microservices, message queues, pub/sub, event sourcing, CQRS, API gateways, REST/GraphQL/gRPC, or real-time delivery mechanisms.

---

## Monolith vs Modular Monolith vs Microservices

**Monolith** — one deployable unit containing all the logic. Fast to build initially, simple to debug (everything's in one process), and trivially supports transactional consistency since there's only one datastore. The cost grows with the codebase: as more people touch it, deploys get riskier, a bug anywhere can take down everything, and the whole thing is locked into one technology stack.

**Modular monolith** — still one deployable unit, but internally organized into well-bounded modules with deliberately limited dependencies between them. This is the underrated middle option: you get most of microservices' organizational clarity (modules can be owned by different teams, with clear boundaries) without paying the distributed-systems tax of network calls, partial failure, and operational overhead between every component.

**Microservices** — independently deployable services, each owning its own data and a single business capability. The benefit is real: teams can deploy independently, scale components independently, and choose different tech per service. The cost is also real and often underestimated: you've traded in-process function calls for network calls (which fail, time out, and add latency), you need service discovery and inter-service auth, debugging now spans multiple processes and logs, and maintaining transactional consistency across services requires patterns like sagas instead of a simple database transaction.

**The decision isn't really technical — it's organizational.** Microservices solve a *team coordination* problem (multiple teams blocked on each other's release cycles) more than a *technical scaling* problem. If your team is small enough to coordinate easily on a shared codebase, you almost certainly don't need them yet, no matter how large your user base becomes.

**The distributed monolith trap**: it's entirely possible to split a system into separately-deployed services that are still tightly coupled — sharing a database, requiring synchronized deploys, calling each other in ways that create hard dependencies. This configuration gets you all of microservices' operational complexity with none of the independence benefits. If your "microservices" can't actually be deployed independently of each other, you've built this trap.

---

## Message Queues vs Publish-Subscribe vs Event Streaming

**Message queues** are point-to-point: a producer puts a task on the queue, and exactly one consumer picks it up and processes it. This is the right shape when work needs to happen exactly once — payment processing, order fulfillment, any job where double-processing or no-processing is a real problem.

**Publish-subscribe (pub/sub)** is fan-out: a publisher emits an event to a topic, and every subscriber to that topic gets its own copy. This fits when multiple independent systems each need to react to the same event without knowing about each other — one user action triggering an email service, an analytics pipeline, and a cache invalidation, all simultaneously and independently.

**Event streaming platforms** (Kafka-style) are a specialized evolution of pub/sub built for very high throughput: they durably store an ordered log of events for a configurable retention window, allowing multiple consumers to replay or process at their own pace. The trade is that streaming platforms generally offer weaker delivery guarantees and routing flexibility than a dedicated message broker — they're built for throughput and replay, not for the fine-grained guarantees (exactly-once delivery, dead-letter handling, flexible routing) a broker provides.

**A practical filter**: if you find yourself asking "does this need to go to exactly one handler, or to every handler that cares," you've already answered queue-vs-pub/sub. If you find yourself asking "do I need to replay the last week of events for a new consumer," you're in event-streaming territory.

---

## Event Sourcing and CQRS

**Event sourcing** stores the full sequence of state-changing events as the system of record, rather than just the current state. The current state is derived (or "materialized") by replaying events. This gives you a complete audit trail for free, makes the system naturally resilient (you can always rebuild state from the log), and is extremely well suited to compliance-heavy domains. The cost is real complexity: different events can have very different shapes, you need a strategy for schema evolution as event formats change over time, and querying "current state" efficiently usually requires building and maintaining a separate materialized view.

**CQRS (Command Query Responsibility Segregation)** separates the write path (commands — instructions that change state and return only success/failure) from the read path (queries — requests for data with no side effects), often backing each with a different, independently optimized data model. This lets you scale and tune reads and writes completely independently, and avoids contorting one schema to serve both jobs well. The cost is added architectural complexity and, when paired with event sourcing (a very common combination), an eventual-consistency gap between when a write lands and when the read-side materialized view reflects it.

These two patterns are frequently used together but are conceptually separable — you can do CQRS with a conventional database on both sides, and you can do event sourcing without splitting reads and writes. Reach for either only when you've identified a concrete reason (audit requirements, very different read/write scaling needs) — both add real complexity that isn't worth paying for by default.

---

## API Gateway and Backend-for-Frontend (BFF)

An **API Gateway** is a single entry point in front of a collection of backend services, handling cross-cutting concerns once instead of in every service: authentication, rate limiting, request routing, response caching, and logging. This matters in a microservices architecture because clients shouldn't need to know your internal service topology, and you don't want to reimplement auth in every single service.

The risk: the gateway itself becomes a single point of failure and a potential bottleneck if not scaled and made redundant carefully, and it adds one more network hop's worth of latency to every request.

**Backend-for-Frontend (BFF)** takes this further by building a *separate* tailored backend layer for each client type (web, mobile, third-party API) rather than one generic gateway trying to serve all of them identically. This avoids the common failure mode of a single backend API accumulating awkward, frontend-specific special cases. The trade is more code to maintain — one BFF per client type instead of one shared layer.

---

## REST vs GraphQL vs gRPC

**REST** organizes an API around resources, accessed via standard HTTP verbs. It's simple, universally understood, and caches well at the HTTP layer by default. Its main weakness is over- or under-fetching — a client often gets more fields than it needs, or has to make several round trips to assemble what one screen actually requires.

**GraphQL** lets the client specify exactly which fields it needs in a single request, solving the over-fetching problem directly and making it easy to evolve a schema without versioning (deprecate fields without breaking old clients). The trade: caching becomes much harder because every query can be shaped differently, and complexity shifts to the server, which now has to resolve arbitrary client-specified query shapes efficiently.

**gRPC** uses a strongly typed, compact binary protocol (protocol buffers) for very low-latency, high-throughput communication, with code generation for type-safe clients in any language. It's the natural choice for service-to-service communication inside your own infrastructure. It's a poor fit for public-facing or browser-based APIs — limited browser support and binary payloads make it far less accessible than REST or GraphQL for that audience.

**Practical default**: REST for public-facing APIs (broad compatibility, good caching, easy to document), GraphQL when client-side flexibility in shaping responses is a recurring pain point (especially with varied client types like web + mobile pulling different field subsets), gRPC for internal service-to-service calls where performance matters more than human-readability.

---

## Real-Time Delivery: Long Polling, WebSocket, SSE

The underlying problem all three solve: HTTP was originally built around the client always initiating requests, with no native way for a server to push data unprompted.

**Long polling** — the client makes a request and the server holds it open until there's new data (or a timeout), then the client immediately re-requests. Simple and universally compatible, but it creates a new connection per cycle and adds latency waiting for the next request cycle to start.

**WebSocket** — a single persistent connection that's fully bidirectional; either side can send data at any time without re-establishing anything. The right choice whenever the client also needs to send data back over the same channel — chat, multiplayer games, collaborative editing.

**Server-Sent Events (SSE)** — a long-lived, unidirectional connection where the server can push events to the client at will, but the client can't send anything back over that same connection. Simpler to implement and operate than WebSocket, broadly supported by browsers, and the right fit when the data flow is genuinely one-directional — live scores, notifications, activity feeds.

**Decision shortcut**: need the client talking back over the same channel? WebSocket. Strictly server-to-client and want the simpler operational story? SSE. Stuck behind infrastructure that blocks persistent connections entirely? Long polling, as a fallback rather than a first choice.
