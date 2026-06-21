# The Data Layer

Original reference notes — load when the conversation touches database choice, replication, sharding, consistent hashing, CAP/PACELC, ACID/BASE, indexing, or normalization.

---

## SQL vs NoSQL — the real distinction

The marketing version of this debate is "structured vs flexible." The more useful version is about what you're optimizing for:

**SQL (relational)** databases enforce a schema up front, support complex multi-table joins, and most implementations give you full ACID transaction guarantees. The cost is that schema changes require migrations, and horizontal scaling is fundamentally harder because relationships and joins don't partition cleanly across machines.

**NoSQL** databases relax the schema (or drop it entirely), generally trade strict consistency for availability and partition tolerance, and are built from the ground up to scale horizontally. The cost is weaker query flexibility — you typically design your NoSQL schema around your *access patterns*, not your *data's natural shape*, which means changing how you query later can require re-architecting your data model.

**Practical heuristic**: if you can describe your most common queries today and they all look like "fetch by key" or "fetch by a known partition," NoSQL will probably scale better with less pain. If your queries involve joining across entities in ways you can't fully predict yet, SQL's flexibility is worth the scaling cost until proven otherwise.

### The NoSQL family isn't one thing
- **Document stores** (MongoDB-style) — general purpose, good when each record is a self-contained JSON-like blob.
- **Key-value stores** (Redis, DynamoDB-style) — the simplest and fastest, ideal for session data and lookup-heavy workloads, but offer almost no query flexibility beyond "give me the value for this key."
- **Graph databases** (Neo4j-style) — when relationships themselves are the primary thing you query (social graphs, fraud detection, recommendation traversal).
- **Wide-column stores** (Cassandra-style) — built for write-heavy, massive-scale workloads where you mostly query by a known row key.
- **Time-series databases** — optimized specifically for timestamped data with fast range queries over time windows (metrics, IoT).

---

## ACID vs BASE

**ACID** (Atomic, Consistent, Isolated, Durable) is the guarantee that a transaction either fully completes or fully rolls back, that the database is never left in a structurally invalid state, that concurrent transactions don't corrupt each other, and that a committed write survives a crash. This is the relational database promise, and it's expensive to provide at scale.

**BASE** (Basically Available, Soft state, Eventual consistency) is the looser alternative many distributed/NoSQL systems adopt: the system stays responsive most of the time, replicas don't have to agree instantly, and consistency arrives eventually rather than immediately. This is the trade you make deliberately in exchange for availability and horizontal scalability.

Neither is "better" — ACID is the right call when an inconsistent intermediate state causes real damage (money, inventory). BASE is the right call when responsiveness and scale matter more than every reader seeing the same answer at the same instant.

---

## CAP Theorem (and why PACELC matters more in practice)

CAP states that a distributed system can only fully guarantee two of three properties: **C**onsistency (every node sees the same data at the same time), **A**vailability (every request gets a response), and **P**artition tolerance (the system keeps working despite network failures between nodes).

In practice, partition tolerance isn't optional — networks fail, so you must design for **P**. That collapses the real decision down to a single trade-off: **C** vs **A** *when a partition actually occurs*.

- **CP systems** (e.g. many configurations of MongoDB, HBase) sacrifice availability during a partition — the inconsistent side simply refuses requests until it's resolved.
- **AP systems** (e.g. Cassandra, CouchDB) sacrifice consistency during a partition — all nodes keep answering, but some may serve stale data until the partition heals and they resync.

**PACELC** extends this with a sharper, more honest observation: even when there's *no* partition, you still have to choose between **L**atency and **C**onsistency in normal operation, because keeping replicas perfectly in sync costs time. CAP only describes behavior during a network failure; PACELC describes the trade-off you're making every single day the system is healthy. When evaluating a database, ask both questions: what does it do during a partition, and what does it cost in latency to keep its normal-case consistency guarantee?

---

## Replication

**Leader-follower (primary-replica) replication**: one node accepts writes and replicates them to one or more read-only followers. Reads can be spread across followers, which is a cheap way to scale read capacity, and a follower can be promoted if the leader fails. The cost: all writes still funnel through one node, and there's a real risk of replication lag — a follower might briefly serve stale data relative to the leader.

**Multi-leader (leader-leader) replication**: more than one node accepts writes, and they sync with each other. This removes the single-writer bottleneck and gives you a faster failover story, but you now have to handle write conflicts — two leaders accepting contradictory writes to the same record at nearly the same time — which adds real application-level complexity.

**Synchronous vs asynchronous replication**: synchronous replication confirms a write only after it's been copied to the replica, guaranteeing the copies never diverge but adding latency to every write. Asynchronous replication confirms the write immediately and copies the data afterward — faster, but there's a window where a crash on the primary could lose writes that hadn't replicated yet.

---

## Sharding (Horizontal Partitioning)

Sharding splits one logical dataset across multiple physical machines, each holding a subset of the rows. It's usually the answer once a single database instance can no longer hold the data or handle the write volume, regardless of how much you vertically scale it.

**Common partitioning strategies**:
- **Range-based** — partition by contiguous ranges of a key (e.g., users A–M on shard 1, N–Z on shard 2). Simple, but prone to hotspots if the data or traffic isn't evenly distributed across the range.
- **Hash-based** — hash the partition key and assign by the result. Distributes load far more evenly than range partitioning, at the cost of making range queries across the whole dataset expensive (you'd have to query every shard).
- **Directory-based** — a lookup service maps each key to its shard explicitly. Flexible and easy to rebalance, but that lookup service becomes a new dependency and potential bottleneck.

**The real cost of sharding** isn't the partitioning itself — it's that joins and transactions across shards become slow or impossible, and rebalancing data when you add or remove shards is genuinely hard to do without downtime or temporary hotspots.

---

## Consistent Hashing

Plain hash-based partitioning (`hash(key) mod N`) has one serious flaw: when `N` changes — a node is added or removed — almost every key's assignment changes too, forcing a massive, unnecessary data shuffle.

**Consistent hashing** solves this by mapping both nodes and keys onto positions on an abstract ring (the output range of a hash function), and routing each key to the nearest node clockwise on that ring. Adding or removing a single node now only affects the keys in its immediate neighborhood on the ring — roughly `K/N` keys need to move, not nearly all of them.

**Virtual nodes** are the standard refinement: instead of placing each physical node at one point on the ring, you place many virtual copies of it at different points. This smooths out load distribution (plain consistent hashing can still produce uneven "hot" segments of the ring) and makes rebalancing after a node change faster and more even.

This is the mechanism underneath distributed caches, distributed hash tables, and the partitioning layer of databases like Cassandra and DynamoDB.

---

## Indexing

An index is a separate data structure that lets the database find rows without scanning the whole table — the classic trade is faster reads in exchange for slower writes (every write now has to update the index too) and extra storage.

**Dense indexes** have an entry for every single row — fast lookups via something like binary search, but more memory and more write-time maintenance. **Sparse indexes** have entries for only some rows (often one per data block) — cheaper to maintain on writes, smaller, but slower to search since you typically still need a scan within a block after the initial lookup.

The practical rule: index the columns your application actually filters or joins on frequently. An index that's never used by a real query is pure write-time tax with no corresponding read benefit.

---

## Normalization and Denormalization

**Normalization** organizes data to eliminate redundancy — each fact is stored exactly once, referenced by key elsewhere. This prevents the three classic anomalies: you can't insert a partial record cleanly (insertion anomaly), updating one fact requires touching multiple rows to stay consistent (update anomaly), or deleting one thing forces deleting unrelated data along with it (deletion anomaly). The cost of normalization is more joins, which get more expensive as data and distribution grow.

**Denormalization** deliberately reintroduces redundancy — duplicating some data across tables or documents — specifically to avoid those expensive joins at read time. This is the standard move once a system is distributed (sharded or federated), because joining across network boundaries is far more expensive than joining within a single machine. The cost: writes become more expensive (you may need to update the same fact in multiple places) and there's more surface area for the copies to drift out of sync if writes aren't handled carefully.

The general pattern in growing systems: start normalized for correctness and simplicity, then denormalize selectively, exactly where read performance actually demands it — not preemptively everywhere.
