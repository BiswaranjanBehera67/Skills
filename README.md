# Skills

A personal library of structured prompting skills for Claude — engineered to produce outputs that default prompting can't.

Default prompts produce default outputs. Each skill here encodes a **deliberate reasoning process** that forces Claude past pattern-matching into genuine ideation, deep analysis, or structured synthesis. Think of them as reusable cognitive tools, not just system prompts.

---

## Structure

```
Skills/
├── README.md                          ← you are here
│
├── innovation/
│   └── ridiculous-idea-generator/     ← absurd-but-buildable idea generation
│
├── programming/                       ← coming soon
├── research/                          ← coming soon
└── productivity/                      ← coming soon
```

Each skill folder contains four files:

| File | Purpose |
|------|---------|
| `skill.md` | The core prompt — copy this into Claude |
| `README.md` | Usage guide, input/output format, tips |
| `examples.md` | Real outputs to set expectations |
| `changelog.md` | Version history and breaking changes |

---

## Categories

| Category | Skills | Description |
|----------|:------:|-------------|
| [innovation](./innovation/) | 1 | Idea generation, contrarian thinking, cross-industry innovation |
| [programming](./programming/) | — | Code generation, debugging, architecture review |
| [research](./research/) | — | Deep research, synthesis, structured fact extraction |
| [productivity](./productivity/) | — | Writing, summarization, decision-making frameworks |

---

## How to Use a Skill

**Option A — Direct paste (quickest)**
1. Open a skill's `skill.md`
2. Copy the full contents
3. Paste it as your first message to Claude, followed by your actual request

**Option B — System prompt (cleanest)**
1. Copy `skill.md` contents into Claude's system prompt field
2. Start the conversation with just your request

**Option C — Claude.ai Skills (if installed)**
If you've installed the `.skill` file, invoke it with `/skill-name` at the start of your message.

---

## Philosophy

Three rules govern every skill in this library:

**1. Process over prompt.** A skill doesn't just tell Claude *what to do* — it encodes *how to think* before doing it. Step-by-step internal reasoning before output.

**2. Constraints breed creativity.** Every skill has an explicit rejection list — things it will *not* produce. Boundaries force Claude away from the obvious.

**3. Format is functional.** Output structure isn't cosmetic. Each field in a skill's output format exists because it forces a specific kind of thinking (e.g., "Why Nobody Is Doing It Yet" forces honest market analysis, not optimistic hand-waving).

---

## Adding a Skill

1. Fork the repo
2. Create a folder under the appropriate category: `category/skill-name/`
3. Add all four required files (`skill.md`, `README.md`, `examples.md`, `changelog.md`)
4. Open a PR — skills missing any of the four files won't be merged

---

## Tested On

Claude Sonnet 4.6 · Claude Opus 4.6

---

*Skills are version-controlled. Check individual `changelog.md` files for breaking changes before updating.*
