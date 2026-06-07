# Examples

Real outputs from actual runs of the Ridiculous Yet Possible Idea Generator skill.
These are unedited results — what Claude actually produced given the inputs shown.

---

## How to Read These

Each example shows:
- The **exact input** used
- **Selected ideas** from that run (not all 10 — just the most illustrative)
- A **post-run note** on what worked and what to try next time

---

## Run 01 — Tech Ideas, Bootstrap Budget

**Input:**
```
Category: Tech
Budget:   bootstrap (minimum money)
Timeline: implement right now
```

**Creativity Modes used:** Contrarian, Human Weirdness × 2, Cross-Industry, Impossible Until Recently

**Ideas from this run:**

---

### 💡 Assumption Auditor
*Mode: Contrarian*

You upload any business plan, pitch deck, or strategy doc. The AI extracts every hidden assumption the author didn't know they were making — then ranks them by how dangerous each one is if wrong.

**Why It Sounds Ridiculous**
"I know my own assumptions" — is exactly what every founder who failed said too.

**Why It Might Actually Work**
Every failed startup traces back to 2–3 unexamined assumptions. McKinsey charges $50K for facilitated assumption-mapping workshops. An adversarial prompt + PDF parser automates 80% of that. No tool exists that does assumption extraction specifically — everything else is generic "feedback."

**Technologies Required**
- PDF.js (free, client-side)
- Claude API (~$0.02 per doc)
- Vercel (free hosting)

**Difficulty** `2` · **Build Timeline** `1 weekend` · **Market Size** `Large`

**Why Nobody Is Doing It Yet**
Too "meta" — people think they need a collaboration tool, not a thinking tool. Hard to explain until you try it once and it surfaces an assumption that breaks your entire model.

**Hidden Advantage**
The AI cross-references your plan against patterns from thousands of failed companies. A human consultant literally cannot do that.

**Future Evolution**
Becomes the standard pre-launch checklist for startups. Integrates with YC and AngelList application flows. $50M ARR in the B2B strategy consulting displacement market.

---

### 💡 Ghosting Probability Meter
*Mode: Human Weirdness*

Paste an email thread or LinkedIn DM. Get a precise probability score that you're being ghosted, with specific behavioral signals cited: "Response latency increased 4×. Message length dropped 60%. Switched from questions to statements."

**Why It Sounds Ridiculous**
"I need a machine to tell me I'm being ignored?"

**Why It Might Actually Work**
The pain isn't the ghosting — it's the uncertainty. People burn 80% of their follow-up energy on dead leads. A "74% ghosting probability — redirect your energy" message is actionable in a way that gut feeling isn't. The behavioral signals are measurable and real.

**Technologies Required**
- Text paste box (HTML)
- Claude API with behavioral analysis prompt
- Vercel free tier

**Difficulty** `1` · **Build Timeline** `2 days` · **Market Size** `Large`

**Why Nobody Is Doing It Yet**
Feels like a novelty. Actually it's a productivity tool. The insight gap is that nobody has framed the emotional problem as a data problem.

**Hidden Advantage**
Virality is built-in — people share their "you're definitely ghosted" results. Every share is free acquisition.

**Future Evolution**
B2B: integrates with CRM (Salesforce, HubSpot). "Your pipeline has a ghosting risk score." $200/seat/month. Acquirable by any sales intelligence company.

---

### 💡 Meeting Eulogy Generator
*Mode: Human Weirdness × Cross-Industry (Memorial Writing + Productivity)*

Paste a calendar invite. Receive a mournful, beautifully written eulogy for the 45 minutes about to die in that meeting — including what could have been accomplished instead, and a "cause of death" diagnosis (no agenda, too many attendees, recurring with no purpose).

**Why It Sounds Ridiculous**
A grief service for meetings?

**Why It Might Actually Work**
Goes instantly viral on LinkedIn because it's funny and true. The free viral tool is the acquisition funnel. The real product is the meeting hygiene analytics dashboard that shows companies their weekly "meeting mortality rate," cost in salary-hours, and agenda quality score. That dashboard is a real product for ops and HR teams.

**Technologies Required**
- Simple web form (HTML/CSS)
- Claude API
- Vercel free tier

**Difficulty** `1` · **Build Timeline** `1 day` · **Market Size** `Large`

**Why Nobody Is Doing It Yet**
No one launches a joke as a B2B funnel intentionally. Most productivity tools try to be serious from day one and fail to acquire users organically.

**Hidden Advantage**
Zero customer acquisition cost — LinkedIn shares do the work. When the enterprise version launches, you already have 100K users to upsell.

**Future Evolution**
"Meeting Intelligence Platform." Real-time cost counter, agenda enforcement, recurring meeting audits. Acquirable by Zoom, Google Workspace, or Microsoft Teams.

---

**Post-run notes:**
- The 1-day builds (Ghosting Meter, Meeting Eulogy) are real MVPs — literally a textarea, an API call, and a deploy
- "Assumption Auditor" is the strongest long-term business of this batch — high willingness-to-pay, clear B2B path
- Next run: add `Country: India` to see how these ideas change for local infrastructure and market conditions

---

---

## Run 02 — SaaS Ideas, Bootstrap Budget

**Input:**
```
Category: SaaS
Budget:   minimum money (bootstrap)
```

*Note: The Novelty Blacklist was enforced — no generic AI wrappers, no subscription clones.*

**Creativity Modes used:** Contrarian × 3, Human Weirdness × 2, Impossible Until Recently, Cross-Industry × 2

**Ideas from this run:**

---

### 💡 Invisible Competitor Monitor
*Mode: Contrarian × Impossible Until Recently*

Instead of tracking what competitors launch, this tracks what they're quietly killing — abandoned features, removed integrations, deleted job postings, disappeared documentation. A competitor's retreat is a stronger intelligence signal than their launch.

**Why It Sounds Ridiculous**
"I want to track things that no longer exist?"

**Why It Might Actually Work**
When a competitor removes a feature, it signals failure, customer indifference, or a bigger rebuild. When they stop hiring for a role, a team is being cut. Existing tools (Crayon, Klue) track additions. Nobody tracks deletions. The Wayback Machine has all historical data — it just needs a systematic watcher and a "so what" layer.

**Technologies Required**
- Wayback Machine / Common Crawl API (free)
- Puppeteer for page diffing (free)
- Supabase for historical snapshots
- Claude API for change summarization

**Difficulty** `5` · **Build Timeline** `3 weeks` · **Market Size** `Large`

**Why Nobody Is Doing It Yet**
Competitive intelligence is mentally framed as "what are they doing?" not "what did they stop doing?" The inversion hasn't occurred to anyone building in the space. Technically harder too — tracking absence requires historical baselines.

**Hidden Advantage**
The historical archive compounds in value over time and is impossible to replicate retroactively. First-mover owns the data moat.

**Future Evolution**
"Competitive archaeology" as a named category. Add intent signals: job posting changes + pricing page edits + feature removals = acquisition incoming. $500/month for product teams. Acquirable by G2, Gartner, or any CI firm.

---

### 💡 Customer Rage Forensics
*Mode: Impossible Until Recently × Cross-Industry (Forensics + Customer Success)*

Connects to your support ticket corpus and identifies the exact words, UI flows, and moments that trigger disproportionate anger — before your support team even responds. Ranks rage triggers by frequency and churn correlation.

**Why It Sounds Ridiculous**
"A forensics lab for customer anger?"

**Why It Might Actually Work**
Support analytics tools (Zendesk Analytics, Intercom Reports) track volume and resolution time. None analyze the language patterns that predict churn. The word "finally" appearing in a support ticket ("I finally figured out...") is a documented signal of a customer who waited too long. That linguistic pattern is invisible without corpus analysis. Companies spend $200K/year on support and would pay $500/month to eliminate 40% of tickets with 5 product changes.

**Technologies Required**
- Zendesk / Intercom API (free read access as the customer)
- Claude API for semantic anger pattern extraction
- React dashboard + Recharts
- Supabase

**Difficulty** `4` · **Build Timeline** `2–3 weeks` · **Market Size** `Massive`

**Why Nobody Is Doing It Yet**
The content-analysis piece was too generic before LLMs. Existing sentiment tools return "negative" — not "your error message on step 3 of onboarding triggers 60% of your cancellations."

**Hidden Advantage**
"Finally" before a churn event is a real, proprietary signal. First company to map language → churn creates a model that no competitor can replicate without the same corpus.

**Future Evolution**
Predictive churn from ticket language. Real-time alert: "This open ticket has a 74% churn probability." $599/month per workspace. Acquirable by Zendesk, Intercom, or Salesforce.

---

### 💡 Cultural Technical Debt Scanner
*Mode: Cross-Industry (Software Engineering + Organizational Psychology)*

Applies the engineering concept of "technical debt" to company culture — identifying cultural shortcuts taken during rapid scaling that will become expensive to fix later. Generates a "cultural debt score" with a prioritized payoff roadmap.

**Why It Sounds Ridiculous**
"Technical debt... but for vibes?"

**Why It Might Actually Work**
The metaphor is exact. Companies that scale fast cut cultural corners ("we'll fix comp bands later," "we'll write values at 50 people") exactly like engineers cut code corners. Both accrue interest. Cultural debt pays out as attrition, toxic behavior normalization, and DEI failure — all with documented dollar costs ($50K+ per senior hire lost). Culture Amp and Lattice measure *current* health. Nobody quantifies *accumulated* cultural debt or provides a payoff roadmap.

**Technologies Required**
- Glassdoor scraper (public data, free)
- Exit interview upload (CSV/PDF)
- Employee survey integration (Typeform free tier)
- Claude API for pattern analysis and scoring
- React dashboard

**Difficulty** `5` · **Build Timeline** `3–4 weeks` · **Market Size** `Massive`

**Why Nobody Is Doing It Yet**
"Cultural debt" as a named, measurable concept doesn't exist in HR tech. Creating a new category requires evangelical selling — but it also means zero competition in the category you define.

**Hidden Advantage**
You name the category, you own it. "Cultural technical debt" is a concept that writes its own press coverage once credible companies adopt the language.

**Future Evolution**
"Org Health Engineering" as a practice. Culture managed with the same rigor as infrastructure. $500/month per company. Integrates with Rippling and Workday. Acquirable by Culture Amp ($1.5B) or Lattice.

---

**Post-run notes:**
- "Invisible Competitor Monitor" is the most defensible long-term because the data moat is the product — the longer you run it, the more historical data you have that no new entrant can replicate
- "Cultural Technical Debt Scanner" requires the most category-creation selling but has the highest ceiling — it's a new mental model, not just a new tool
- For both runs: the $0 fixed cost structure is real — Vercel free + Supabase free + pay-per-use API means you only pay when customers use the product, which means revenue covers cost from day one

---

---

## Patterns Observed Across Runs

After both runs, a few patterns in what makes the strongest ideas:

**1. Inversion of a named existing tool works better than pure novelty**
"Invisible Competitor Monitor" (tracks what competitors *stop* doing) is stronger than a generic "AI intelligence tool" because it has a clear point of differentiation from named competitors (Crayon, Klue).

**2. Viral entry + B2B upsell is the cleanest zero-budget business model**
Meeting Eulogy Generator (free viral tool → enterprise meeting analytics) is the template. The free tool acquires users at $0 CAC; the paid tool converts a fraction at high LTV.

**3. The best ideas sound like jokes until you explain one mechanism**
Every strong idea in these runs has a single specific mechanism that makes you say "oh, that's actually real": the "finally" signal in support tickets, the Wayback Machine as a deletion tracker, the Ebbinghaus forgetting curve applied to professional skills.

**4. Difficulty 1–2 ideas are real MVPs**
The lowest-difficulty ideas in these runs are genuinely just a textarea, an API call, and a Vercel deploy. If you've been stuck on "what should I build," starting with a Difficulty 2 idea and seeing if anyone pays is a better use of a weekend than planning.

---

*Examples last updated: June 2026 · Skill version: v1.0.0*
