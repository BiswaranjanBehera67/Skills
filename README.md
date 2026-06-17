# Skills

A personal library of structured prompting skills for Claude.

Each skill encodes a **reasoning process**, not just instructions. The difference: a prompt tells Claude what to do. A skill tells Claude how to think before doing it — with a defined sequence of internal steps, explicit quality checks, and structured output formats that make the result consistently better than asking directly.

---

## Skill Index

| Skill | Category | What it does |
|-------|----------|-------------|
| [ridiculous-yet-possible-ideas](./innovation/ridiculous-yet-possible-ideas/) | innovation | Generates ideas that sound absurd but become plausible — crosses 3+ industries, inverts assumptions, applies 8 creativity modes |
| [learning-accelerator](./productivity/learning-accelerator/) | productivity | Feynman-technique learning loop — probes before explaining, tests with active recall, ends every session with spaced repetition questions |

**Planned:**

| Skill | Category | Status |
|-------|----------|--------|
| code-archaeology | programming | In design — systematic process for reading and mapping unfamiliar codebases |
| cognitive-bias-auditor | programming | In design — scans reasoning for named biases with specific evidence |
| decision-architect | productivity | Planned — pre-mortem + scenario planning + BATNA in one structured flow |
| customer-insight-synthesizer | research | Planned — extracts JTBD and pain hierarchies from raw interview notes |

---

## How to Use a Skill

There are three ways, depending on your setup.

### Option 1 — Paste into conversation (works everywhere)
1. Open the skill folder and copy the contents of `skill.md`
2. Paste it as your first message in a new Claude conversation
3. Follow with your actual request on the next line

### Option 2 — System prompt (cleanest experience)
1. Copy `skill.md` into Claude's system prompt field before starting
2. Your first message is just your request — the skill is already active

### Option 3 — Install as a `.skill` file (Claude.ai)
1. Download the `.skill` file from the skill folder (if packaged)
2. Drag it into Claude's Skills settings
3. Invoke it with `/skill-name` at the start of any message

For most skills, Option 1 is fine. Option 3 is the best experience if you use a skill repeatedly.

---

## What's Inside Each Skill Folder

Every skill follows the same four-file format:

```
skill-name/
├── skill.md       The prompt — copy this into Claude to activate the skill
├── README.md      Usage guide: input format, modes, tips
├── examples.md    Real outputs from actual runs, including failures
└── changelog.md   Version history
```

**`skill.md`** is the only file Claude sees. The others are for you — to understand what the skill does, see what outputs look like before running it, and track what changed between versions.

---

## What Makes These Different from Prompts

A prompt is a request. A skill is a process.

Most "prompt libraries" are collections of clever phrasing. The outputs vary wildly depending on how Claude interprets the request. Skills are different in three ways:

**1. Process before output.**
Every skill defines a sequence of internal reasoning steps that Claude runs before writing anything. The `ridiculous-idea-generator` inverts industry assumptions, combines three fields, and searches for second-order effects before generating a single idea. The `learning-accelerator` maps the learner's existing knowledge before writing a single sentence of explanation. This changes the output structurally, not just stylistically.

**2. Explicit rejection lists.**
Each skill defines what it will not produce. The idea generator rejects anything resembling a generic SaaS, AI chatbot, or marketplace — which forces every output past the obvious zone. The learning accelerator forbids "does that make sense?" and requires an active recall test instead of a comprehension check. Constraints make the outputs better, not worse.

**3. Verifiable output structure.**
Skills produce outputs in defined formats with named fields, not freeform prose. This means you can compare outputs across runs, know what to expect before running, and catch when the skill is performing below par. The `examples.md` in each folder shows what good output looks like — and what a weak run looks like — so you have a reference.

---

## Repo Structure

```
Skills/
├── README.md                               ← you are here
│
├── innovation/
│   └── ridiculous-yet-possible-ideas/      ✓ built
│       ├── skill.md
│       ├── README.md
│       ├── examples.md
│       └── changelog.md
│
├── programming/                            ← in progress
│
├── research/                               ← planned
│
└── productivity/
    └── learning-accelerator/               ✓ built
        ├── skill.md
        ├── README.md
        ├── examples.md
        └── changelog.md
```

---

## Adding a Skill

Pull requests are welcome. A skill earns a merge when:

- It has all four files (`skill.md`, `README.md`, `examples.md`, `changelog.md`)
- `skill.md` defines at least one internal reasoning process — not just output instructions
- `skill.md` has an explicit rejection list or quality filter
- `examples.md` includes at least one run where the skill caught an error or produced a non-obvious output
- The skill doesn't duplicate something that a good direct prompt already handles reliably

The last criterion matters most. If asking Claude "summarize this document" already works well, there's no skill to build — the process is trivial. Skills are worth building when the reasoning sequence is what makes the difference.

---

## Tested On

Claude Sonnet 4.6 · Claude Opus 4.6

