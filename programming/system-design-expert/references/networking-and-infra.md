# Networking and Infrastructure Fundamentals

Original reference notes — load when the conversation touches DNS, load balancing, caching strategy, CDNs, proxies, availability math, or basic scaling decisions.

---

## DNS, in one paragraph

DNS turns human-readable names into the IP addresses machines actually route on. A lookup typically hops from a recursive resolver to a root server, to a TLD server, to the authoritative server for the domain, with caching at every layer (each record carries a TTL that governs how long it can be reused before a fresh lookup is required). The practical system-design takeaway: DNS is itself a caching layer, and you can use it for crude load distribution by returning multiple IPs for one name — but DNS has no concept of server health, so it will happily hand out the address of a server that's on fire. Don't rely on it as your only load-balancing mechanism.

---

## Load Balancing

A load balancer's job is to spread incoming traffic across a pool of backends so no single one is overwhelmed, and to stop sending traffic to backends that have failed health checks.

**Layer 4 vs Layer 7**
A Layer 4 (transport-level) balancer routes based on IP and port only — it's fast and protocol-agnostic but can't make decisions based on the content of a request. A Layer 7 (application-level) balancer reads the actual HTTP request and can route by path, header, or cookie — more powerful, slightly more overhead.

**Routing algorithms worth knowing**
- *Round robin* — simplest, cycles through backends in order. Works fine when backends are roughly equal in capacity and requests are roughly equal in cost.
- *Least connections* — sends new traffic to whichever backend currently has the fewest open connections. Better than round robin when request duration varies a lot.
- *Weighted variants* of either — useful when your fleet is heterogeneous (some bigger boxes than others).
- *Hash-based* — routes based on a hash of some request property (client IP, session ID), which is what you want for sticky sessions.

**Don't forget the load balancer itself can fail.** A single load balancer is a single point of failure; production setups almost always run at least one passive standby that can take over.

---

## Caching

Caching trades capacity for speed by keeping a small, fast-access subset of data closer to where it's needed, banking on the fact that recently accessed data is disproportionately likely to be accessed again soon.

**The three write strategies**
- *Write-through* — every write goes to the cache and the database at the same time. Strong consistency between the two, at the cost of slower writes.
- *Write-around* — writes bypass the cache and go straight to the database; the cache only fills on a subsequent read. Fast writes, but the next read after a write is guaranteed to be a cache miss.
- *Write-back (write-behind)* — writes land in the cache first and are confirmed immediately; the cache asynchronously flushes to the database later. Fastest writes and highest throughput, but a cache crash before the flush completes means real data loss — mitigate by replicating the cache.

**Eviction policies**
When the cache is full and something new needs to come in, you need a rule for what gets thrown out. Least Recently Used (LRU) is the default that fits most workloads — discard whatever hasn't been touched in the longest time. Least Frequently Used (LFU) is better when some items are reliably "hot" regardless of recency. First In First Out (FIFO) is the simplest and the worst fit for almost everything, since it ignores access patterns entirely.

**When caching doesn't help**
- When fetching from the cache costs about as much as fetching from the source — there's no speed gain to bank.
- When access patterns are close to random — caching profits from repetition, and random access has none to exploit.
- When the underlying data changes faster than it's read — the cache spends more time being invalidated than being useful.

A cache should never be the only copy of data that matters. It's a volatile, transient layer by design.

---

## Content Delivery Networks (CDN)

A CDN is a network of geographically distributed servers (edge locations) that cache static content (images, video, JS/CSS bundles) physically closer to the people requesting it, so a user in Mumbai doesn't have to round-trip to a server in Virginia for every asset.

**Push vs pull**
A *push* CDN is one you proactively upload content to whenever it changes — efficient when content changes rarely and traffic is modest. A *pull* CDN fetches and caches content lazily on first request from each region — better for high-traffic sites where you don't want to manage uploads manually, since the CDN figures out what's actually being requested.

**The real cost of a CDN isn't always money** — it's that if most of your users are in a region the CDN doesn't have a strong presence in, requests can actually travel *further* than they would without one. Check your CDN provider's edge map against your actual user geography before assuming it helps.

---

## Proxies

A **forward proxy** sits in front of a group of clients and mediates their outbound requests — used for anonymizing client identity, enforcing content policy, or bypassing geo-restrictions. A **reverse proxy** sits in front of one or more servers and mediates inbound requests — used for caching, SSL termination, and as a security layer that keeps clients from ever talking to your origin servers directly.

The distinction that trips people up: a reverse proxy *can* act as a load balancer, but a load balancer's job — distributing across multiple backends — is conceptually different from a reverse proxy's job, which is useful even with a single backend server.

---

## Availability Math

Availability is usually expressed as a percentage of uptime over a period, commonly described by how many "nines" it has. The jump from 99% to 99.9% sounds small but the downtime budget collapses dramatically:

| Availability | Downtime per year |
|---|---|
| 99% (two nines) | ~3.65 days |
| 99.9% (three nines) | ~8.77 hours |
| 99.99% (four nines) | ~52.6 minutes |
| 99.999% (five nines) | ~5.25 minutes |

**Components in sequence multiply availability down.** If two components each run at 99.9% and a request depends on both succeeding in sequence, the combined availability is roughly 99.8% — worse than either alone. **Components in parallel (redundant paths) multiply availability up** — if either of two 99.9% components can serve the request, combined availability climbs toward 99.9999%. This is the entire mathematical case for redundancy: failure probabilities multiply, and parallel paths turn that multiplication in your favor instead of against you.

**Availability is not the same as reliability.** A system can have minimal downtime (highly available) while still failing requests intermittently in ways that don't trigger a full outage (not fully reliable). Fault tolerance — zero interruption even during a failure — is a stronger and far more expensive guarantee than high availability, which tolerates brief gaps.

---

## Scaling: Vertical vs Horizontal

**Vertical scaling** (bigger machine) is simple, requires no architectural change, and keeps data consistent because there's only one copy of anything — but it has a hard ceiling, real downtime risk during the upgrade, and creates a single point of failure that gets more expensive to mitigate the bigger the machine gets.

**Horizontal scaling** (more machines) has no real ceiling, improves fault tolerance because no single node loss is fatal, and is generally cheaper per unit of capacity at scale — but it introduces real complexity: data has to be partitioned or replicated across nodes, coordination between nodes adds overhead, and consistency becomes a design decision rather than a given.

The practical default: scale vertically first because it's nearly free in complexity, and only move to horizontal scaling once you've hit a ceiling that vertical scaling genuinely can't solve — not because horizontal scaling is fashionable.
