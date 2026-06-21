# Distributed Systems Toolkit

Original reference notes — load when the conversation touches circuit breakers, rate limiting, service discovery, SLA/SLO/SLI, disaster recovery, authentication protocols, or geospatial indexing.

---

## Circuit Breakers

The problem: when a service makes calls to a dependency that's failing or hanging, those calls tie up threads and connections waiting for timeouts, and the failure cascades backward into the calling service and beyond. A circuit breaker wraps the call and tracks failures, and once they cross a threshold, it stops attempting the call entirely for a cooldown period — failing fast instead of failing slow.

**Three states**: *Closed* — everything's normal, requests pass through. *Open* — the failure threshold was crossed, requests are rejected immediately without even attempting the call, for a configured cooldown. *Half-open* — after the cooldown, a limited number of test requests are allowed through; if they succeed, the breaker closes again, and if they fail, it reopens.

The point of a circuit breaker isn't to prevent the underlying failure — it's to prevent that failure from consuming your own resources and propagating into a system-wide cascade while the dependency recovers.

---

## Rate Limiting

Rate limiting caps how often an operation can happen in a given window, protecting shared resources from being overwhelmed — whether by malicious abuse, a buggy client retrying too aggressively, or simply more legitimate traffic than the system can handle.

**Common algorithms**:
- **Token bucket** — a bucket holds tokens that refill at a fixed rate; each request consumes one token, and requests are rejected when the bucket is empty. Naturally allows brief bursts up to the bucket's capacity while still enforcing a steady average rate.
- **Leaky bucket** — incoming requests join a queue and are processed at a fixed rate; once the queue is full, new requests are dropped. Smooths bursts into a steady output rate rather than allowing them through.
- **Fixed window** — count requests in discrete time windows (e.g., per minute) and reject once a threshold is hit. Simple, but has a boundary flaw: a burst right at the edge of two windows can let through nearly double the intended rate.
- **Sliding window** — blends fixed-window's cheap counting with smoother boundary behavior by weighting the previous window's count based on how far into the current window you are. The practical default when fixed window's boundary issue actually matters for your use case.

**Rate limiting gets genuinely hard in distributed systems** for two reasons. First, if each node tracks its own counter independently, a client hitting multiple nodes can blow past a supposedly global limit — the fix is usually a shared counter in a fast centralized store like Redis, accepting the added latency and coordination cost. Second, a naive read-increment-write cycle on that shared counter creates a race condition where concurrent requests can all read the same stale count before any of them writes back — solved with atomic increment operations rather than separate read-then-write steps.

---

## Service Discovery

In a system where service instances come and go dynamically (autoscaling, container scheduling, failures), hardcoding addresses doesn't work. Service discovery is the mechanism by which a service finds the current network location of another service it depends on.

**Client-side discovery** — the calling service queries a registry directly to get a list of healthy instances, then picks one itself (often combined with client-side load-balancing logic).

**Server-side discovery** — the calling service just sends its request to a known load balancer, which is the one querying the registry and routing the request onward. Simpler for the calling service, at the cost of routing every request through that extra hop.

**Registration** can be self-managed (each instance registers and sends heartbeats to prove it's alive) or third-party (an external watcher polls the deployment environment and updates the registry on its own). Self-registration is simpler to reason about; third-party registration decouples services from needing any awareness of the registry at all.

A **service registry** itself needs to be highly available — it's a hard dependency for nearly every request path in the system once you're relying on dynamic discovery, so treat it with the same seriousness as your primary datastore.

---

## SLA, SLO, SLI

These three terms describe the same underlying promise at different levels of formality, and people often use them interchangeably even though they mean distinct things:

- **SLA (Service Level Agreement)** — the external, often contractual, promise made to users or customers about a service's behavior (commonly availability), frequently with financial or legal consequences for missing it.
- **SLO (Service Level Objective)** — the specific internal target for a metric (e.g. "99.95% of requests succeed") that the team commits to in order to satisfy the SLA. The SLO is the operational goal; the SLA is the external commitment built on top of it.
- **SLI (Service Level Indicator)** — the actual measured value of that metric in production. The SLI is what you observe; the SLO is the target you're comparing it against; the SLA is what happens externally if the gap between them gets bad enough.

A good SLO always sits with some margin inside the SLA — committing the SLO and SLA to the exact same number leaves zero room for normal operational variance before you're in formal breach.

---

## Disaster Recovery

Disaster recovery planning revolves around two numbers you should be able to state for any serious system:

- **RTO (Recovery Time Objective)** — the maximum acceptable time the system can be down before service is restored.
- **RPO (Recovery Point Objective)** — the maximum acceptable amount of data loss, measured as time since the last good backup or replication point.

**Strategy tiers, roughly in increasing cost and decreasing RTO/RPO**: a simple off-site backup (cheapest, slowest to restore, highest potential data loss); a cold site with infrastructure provisioned but not running data continuously (faster than backups alone, still requires real setup time to bring online); a hot site with continuously synchronized data ready to take over immediately (most expensive, lowest RTO and RPO, effectively the basis for true high availability across regions).

The right tier depends entirely on what RTO and RPO the business actually requires — there's no universal correct answer, only the cost-vs-risk trade the requirements justify.

---

## Authentication and Authorization Building Blocks

**OAuth 2.0** is fundamentally an *authorization* protocol — it lets a user grant a third-party application limited access to their data on another service, without ever handing over their actual credentials. It defines a flow between a resource owner (the user), a client (the app requesting access), an authorization server (which issues tokens), and a resource server (which holds the protected data and validates tokens).

**OpenID Connect (OIDC)** is a thin identity layer built on top of OAuth 2.0 — OAuth alone says nothing about *who* the user is, only that access was granted. OIDC adds that missing authentication piece, returning identity information about the logged-in user (commonly as a JSON Web Token) alongside the access grant. The shorthand that's actually accurate: OAuth answers "is this app allowed to do X," OIDC additionally answers "who is this user."

**Single Sign-On (SSO)** lets a user authenticate once with a central Identity Provider (IdP) and gain access to multiple separate applications without logging into each one individually. SAML is the older, XML-based protocol commonly used for SSO in enterprise contexts; OIDC has become the more common modern choice, particularly for web and mobile, because it's JSON-based and noticeably simpler to implement.

**TLS and mTLS** — TLS encrypts traffic between a client and server and lets the client verify the server's identity via its certificate. **Mutual TLS (mTLS)** goes further by having *both* sides present certificates, so the server also verifies the client's identity cryptographically rather than only via something like a password. This is the backbone of zero-trust service-to-service communication, where you don't want to assume a request is legitimate just because it originated inside your own network perimeter.

---

## Geospatial Indexing: Geohashing and Quadtrees

Both solve the same core problem: naive latitude/longitude range queries ("find everything within R of this point") don't scale, because they require scanning or comparing against far more data than necessary.

**Geohashing** encodes a 2D coordinate into a single alphanumeric string by recursively subdividing the world into a grid, where longer strings mean smaller, more precise grid cells. The genuinely useful property: two locations that are physically close will very often share a long common string prefix, which turns "find nearby points" into a comparatively cheap string-prefix comparison instead of true geometric distance math. The catch to watch for: points can be close together but happen to land on opposite sides of a grid cell boundary, in which case they won't share a long prefix despite the short physical distance — so a robust implementation typically checks several neighboring grid cells, not just an exact prefix match.

**Quadtrees** instead build an actual tree structure, recursively splitting 2D space into four quadrants and only subdividing further in regions that are densely populated with points. This adapts naturally to uneven data distribution — dense urban areas get many small, precise cells, while sparse rural ones stay as large, coarse cells — and supports efficient range and nearest-neighbor queries directly against the tree structure.

**When to reach for which**: geohashing is simpler to store and shard, since it's just a string that fits naturally as a key in nearly any database or cache. Quadtrees give more precise, adaptive control over query performance in unevenly distributed data, at the cost of needing an actual tree structure (and the engineering that comes with maintaining one) rather than a plain indexable string.
