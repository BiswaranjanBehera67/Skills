# Ridiculous Yet Possible — Idea Generator

> *Generate ideas that sound absurd in 2026 but are obvious in 2040.*

A structured prompting skill that turns Claude into a contrarian innovation strategist. It forces a 5-step internal reasoning process — inverting assumptions, fusing industries, hunting second-order effects — before a single idea is written.

---

## What Makes This Different

Most "idea generation" prompts produce repackaged versions of what already exists. This skill is built around **three hard constraints**:

1. Every idea must combine at least 3 unrelated fields
2. Every idea must invert at least one industry assumption
3. Ideas that resemble anything on the [Novelty Blacklist](#novelty-blacklist) are discarded and regenerated

The result: ideas that feel strange, then obvious — not ideas that feel safe from the start.

---

## When to Use It

- You want startup ideas in a specific domain and keep getting generic results
- You're trying to find cross-industry angles nobody in your field is looking at
- You need to break out of category thinking before a brainstorm session
- You want ideas that are actually buildable — not just thought experiments

---

## Input Format

**Minimum required:**
```
Category: [your domain]
```

**Full format (for better results):**
```
Category:    Tech / Healthcare / Finance / Agriculture / Space / etc.
Budget:      bootstrap / <$10K / VC-backed / no constraint
Timeline:    implement this week / 1-year horizon / 20-year horizon
Country:     India / USA / global / [specific region]
Industry:    [narrow the domain further if needed]
Target:      solo devs / enterprises / governments / consumers
Stack:       [e.g., only free APIs / Python only / no hardware]
```

> **Tip:** The more constraints you add, the more specific and actionable the ideas. `Category: Tech` gives broad ideas. `Category: Tech, Budget: bootstrap, Country: India, Target: solo developers` gives ideas calibrated to what you can actually build next week.

---

## Output Format

Each idea is structured across 10 sections:

| Section | What It Forces |
|---------|---------------|
| **Idea Name** | Memorable, searchable title |
| **One-line description** | Plain English — no jargon |
| **Why It Sounds Ridiculous** | The instinctive rejection most people would have |
| **Why It Might Actually Work** | Scientific / economic / behavioral reasoning — must be specific |
| **Technologies Required** | Concrete list — no vague hand-waving |
| **Difficulty** | 1–10 scale |
| **Build Timeline** | Realistic, not optimistic |
| **Potential Market Size** | Small / Medium / Large / Massive |
| **Why Nobody Is Doing It Yet** | Real barriers — psychological, regulatory, economic, or technical |
| **Hidden Advantage** | What a competitor would miss even if they copied the idea |
| **Future Evolution** | How this becomes a billion-dollar category |

---

## Output Quantity

| Request phrasing | Ideas generated |
|-----------------|:---------------:|
| Default (no quantity specified) | 10 |
| "deep exploration" or "detailed" | 25 |
| "brainstorming mode" or "maximum" | 50 |

Start with 10. Ask for 25+ when you want the weirder outputs — ideas 15–25 are typically less obvious than the first batch.

---

## The 5-Step Generation Process

Claude runs this internally before writing any idea:

```
Step 1 — IDENTIFY ASSUMPTIONS
  What does everyone in this industry believe is fixed and non-negotiable?
  Example (SaaS): "Products need many features to compete."

Step 2 — INVERT
  Flip each assumption to its opposite.
  Example: "One-feature SaaS wins a niche."

Step 3 — COMBINE 3+ FIELDS
  Force-fuse the category with at least 2 unrelated industries.
  Example: SaaS + Behavioral Economics + Forensics = Customer Rage Forensics

Step 4 — SECOND-ORDER EFFECTS
  "If this became common, what new problem would appear?"
  Use that problem as an additional opportunity.

Step 5 — APPLY A CREATIVITY MODE
  Pick one of the 8 lenses below and look through it.
```

---

## The 8 Creativity Modes

| # | Mode | The Lens |
|---|------|----------|
| 1 | **Contrarian** | Assume the industry experts are wrong about a core belief |
| 2 | **Sci-Fi Becoming Real** | Find science fiction concepts now technically feasible |
| 3 | **Human Weirdness** | Exploit irrational, paradoxical, or unusual human behavior |
| 4 | **Extreme Future** | Assume 2040 technologies already exist today |
| 5 | **Cross-Industry Mutation** | Combine two industries that have never spoken to each other |
| 6 | **Government Scale** | Think at national or planetary scale |
| 7 | **Nature Inspired** | Copy, invert, or scale a biological system |
| 8 | **Impossible Until Recently** | Use a scientific or technological breakthrough from the last 5 years |

Modes are varied across ideas — the same mode won't appear back-to-back.

---

## Quality Filter

Every idea passes this check before appearing in output. Failing ideas are discarded and regenerated:

- ✓ Physically possible under known or near-future science
- ✓ Not already mainstream or widely commercialized
- ✓ Technically feasible within a 1–20 year window
- ✓ Novel enough to surprise a domain expert
- ✓ Could generate real value if executed correctly

---

## Novelty Blacklist

The following are immediately rejected and replaced:

- Generic AI chatbot or LLM wrapper
- AI SaaS with no novel application
- Food delivery or restaurant tech
- E-commerce store or marketplace clone
- Social media app or community platform
- "Uber for X" or generic gig economy concept
- Standard automation startup
- Generic subscription business
- Anything a 2020 pitch deck could have described

---

## Tips

**Add a budget constraint for buildable ideas**
`Budget: bootstrap` produces ideas you can start this weekend. Without it, you'll get ideas that require VC funding.

**Add your country for localized ideas**
India-specific constraints produce very different ideas from US-specific ones — infrastructure gaps, regulatory environments, and market behaviors differ enormously.

**Combine categories to force cross-domain ideas**
`Category: Healthcare + Finance` is more generative than either alone. The intersection is where nobody is looking.

**Ask for 25+ ideas to get the stranger ones**
The first 10 ideas are the most accessible. Ideas 15–25 are typically where the genuinely weird-but-brilliant ones appear.

**Use the output as a starting point, not an endpoint**
Pick the most interesting idea and run it through a second prompt: "Now go 10x deeper on [idea name]. What's the exact MVP? What's the first customer? What's the biggest technical risk?"

---

## Files in This Skill

| File | Description |
|------|-------------|
| [`skill.md`](./skill.md) | The full prompt — copy this into Claude to activate the skill |
| [`README.md`](./README.md) | This file |
| [`examples.md`](./examples.md) | Real outputs from actual runs, with full context |
| [`changelog.md`](./changelog.md) | Version history and what changed |

---

## Version

Current: `v1.0.0` — see [`changelog.md`](./changelog.md)
