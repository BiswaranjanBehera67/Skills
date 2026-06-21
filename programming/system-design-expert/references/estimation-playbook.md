# Estimation Playbook

Original reference notes — load whenever back-of-envelope capacity estimation is needed. The goal of this kind of math is never precision; it's showing you know which numbers actually drive architecture decisions, and roughly where the order of magnitude lands.

---

## The Core Formulas

**Requests per second (RPS), from daily volume**

$$
RPS = \frac{\text{requests per day}}{86{,}400 \text{ seconds}}
$$

86,400 is just 24 × 3600 — worth memorizing so you're not doing that multiplication live. As a rough shortcut, dividing daily volume by 100,000 gets you within about 15% of the true average RPS, which is usually precise enough for a design conversation.

**Peak vs average**
Average RPS assumes perfectly even traffic across 24 hours, which essentially never happens. A common, defensible assumption is that peak traffic runs at roughly 2–5× the average, concentrated in a few hours. Always state peak alongside average — the system has to survive the peak, not the average.

**Storage**

$$
\text{Storage} = \text{records per day} \times \text{size per record} \times \text{retention period}
$$

Round size-per-record generously (most people underestimate metadata, indexes, and replication overhead — a reasonable rule of thumb is to add 20–30% on top of the raw data size estimate to account for this).

**Bandwidth**

$$
\text{Bandwidth} = \text{requests per second} \times \text{average payload size}
$$

Calculate ingress (incoming writes/uploads) and egress (outgoing reads/downloads) separately — they're frequently very different orders of magnitude, especially in read-heavy systems, and conflating them hides the real bottleneck.

**Cache sizing (Pareto-style approximation)**
A common, reasonable starting assumption is that roughly 80% of read traffic targets roughly 20% of the data (the Pareto principle applied to access patterns). Under that assumption, caching the hottest 20% of data by volume captures a large majority of reads, so cache capacity can often be estimated as:

$$
\text{Cache size} \approx 0.2 \times \text{daily unique data accessed} \times \text{average record size}
$$

This is a starting assumption to state explicitly, not a law of nature — real access patterns vary, and you should say so when using it.

---

## A Fully Worked Example

To show the pattern end-to-end, here's a complete estimation for a hypothetical photo-sharing service — deliberately a different scenario from any specific named product, so the arithmetic pattern is what transfers, not the specific numbers.

**Stated assumptions** (always say these out loud first):
- 20 million daily active users (DAU)
- Each user views 30 photos and uploads 1 photo per day on average
- Read:write ratio is therefore roughly 30:1
- Average photo file size: 2 MB
- Average metadata record size: 300 bytes
- Retention: photos kept indefinitely; planning horizon of 5 years for storage estimation

**Daily volume**

$$
\text{Uploads/day} = 20\text{M} \times 1 = 20\text{M}
$$
$$
\text{Views/day} = 20\text{M} \times 30 = 600\text{M}
$$

**Requests per second (average)**

$$
\text{Write RPS} = \frac{20\text{M}}{86{,}400} \approx 230/\text{s}
$$
$$
\text{Read RPS} = \frac{600\text{M}}{86{,}400} \approx 7{,}000/\text{s}
$$

Assuming peak traffic runs at 3× average: peak writes ≈ 700/s, peak reads ≈ 21,000/s. This single jump — from "average" to "peak" — is usually the moment a design conversation gets serious, because provisioning for 7,000/s and provisioning for 21,000/s are different architectural conversations.

**Storage (5-year horizon)**

$$
\text{Photo storage/day} = 20\text{M} \times 2\text{MB} = 40\text{TB/day}
$$
$$
\text{Metadata storage/day} = 20\text{M} \times 300\text{B} \approx 6\text{GB/day}
$$
$$
\text{5-year photo storage} \approx 40\text{TB} \times 365 \times 5 \approx 73\text{PB}
$$

That single number — tens of petabytes — immediately rules out a simple single-machine file store and points straight toward object storage (the kind built to scale horizontally and cheaply for exactly this access pattern) as a near-mandatory choice rather than an optional optimization.

**Bandwidth**

$$
\text{Ingress} = \frac{20\text{M} \times 2\text{MB}}{86{,}400} \approx 460\text{MB/s}
$$
$$
\text{Egress} = \frac{600\text{M} \times 2\text{MB}}{86{,}400} \approx 14\text{GB/s}
$$

The roughly 30× gap between ingress and egress is the read/write ratio showing up again, and it's the single strongest signal in this whole estimate: this system lives or dies on serving reads efficiently. That one observation should drive the architecture conversation toward CDN placement and aggressive read caching before almost anything else gets discussed.

**Cache sizing**
Assuming the 80/20 approximation holds and a typical user revisits roughly the same 20% of recently-uploaded photos on a given day: caching that 20% of daily uploads' worth of data, plus a reasonable multi-day rolling window of recent uploads (since recent content tends to be disproportionately viewed), gives a defensible starting cache budget in the hundreds of GB to low TB range — small enough to fit a managed in-memory cache cluster comfortably, while still absorbing the large majority of read traffic before it ever reaches the origin storage layer.

---

## What to Do With These Numbers

The numbers themselves are rarely the point — what they should drive is a small set of concrete architecture decisions:

- A read:write ratio above roughly 10:1 → prioritize caching, read replicas, and CDN placement before anything else.
- Storage in the petabyte range → object storage, not a conventional filesystem or a single relational database's blob column.
- Peak RPS that's a large multiple of average → the system needs to be provisioned (and ideally auto-scaled) for the peak, with the gap between peak and average representing pure unused capacity most of the time — which is itself a cost conversation worth having explicitly.
- A bandwidth number in the GB/s range → CDN and edge caching aren't optional optimizations at that point, they're load-bearing infrastructure.

If an estimate doesn't change any decision you'd otherwise make, it probably wasn't worth doing in that level of detail — the exercise exists to surface decisions, not to produce an impressively precise-looking number.
