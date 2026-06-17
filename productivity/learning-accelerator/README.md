# Learning Accelerator

> *The goal is not that you can recall the explanation. The goal is that you can generate it.*

A structured tutoring skill that turns Claude from a textbook into a tutor. It forces a deliberate sequence — calibrate, explain, test, find gaps, drill gaps, build retention — so that understanding actually sticks instead of feeling like it did until you closed the tab.

---

## The Problem This Solves

When you ask Claude to explain something, you get an explanation. You read it. You think "got it." You close the tab. Three days later, you can't explain it to anyone.

That's not learning. That's reading.

The difference between reading and learning is whether you've been forced to **generate** the understanding yourself — not just receive it. This skill encodes that process.

---

## What It Does Differently

| Default Claude | Learning Accelerator |
|----------------|---------------------|
| Explains the concept | Calibrates your level first |
| Comprehensive coverage | Right depth for your goal |
| Moves to next point | Stops and tests you after each concept |
| "Does that make sense?" | "Now explain it back to me" |
| Generic feedback | Names the exact gap in your explanation |
| Ends at explanation | Ends with spaced repetition questions + application challenge |

The core mechanism is the **Feynman Challenge**: after every concept, Claude stops explaining and asks you to explain it back. Your explanation is analyzed for specific failure modes. Gaps are drilled with Socratic questions, not re-explanation.

---

## When to Use It

- You want to *actually* understand something, not just have it explained
- You keep forgetting the same concept and need it to stick
- You know a topic well enough to use it but can't explain it to anyone
- You're preparing to teach or present something
- You're prepping for a technical interview or exam
- You're trying to build something and want to learn what you need along the way

**Don't use it for:**
- Quick factual lookups ("what's the capital of X")
- Simple reminders ("remind me of the syntax for X")
- Skimming a topic landscape (use Mode 2 — Rapid Map for that)

---

## Input Format

**Minimum:**
```
Topic: [what you want to learn]
```

**Full input for a better session:**
```
Topic:  [the concept]
Level:  complete beginner / know the basics / I use it but can't explain it / I can explain it but want depth
Goal:   understand it / apply it / explain it to others / pass an exam / build something
Time:   10 min / 30 min / no limit
Style:  examples-first / theory-first / analogies / proof-based
```

> If you skip the optional fields, Claude will ask one or two calibration questions at the start of the session.

---

## The 6 Learning Modes

### 1 — Feynman Loop *(default)*
Deep mastery of a single concept. The full 7-phase process: calibrate → explain → test → gap drill → deepen → misconceptions → retention.

Best for: anything you want to actually understand and remember.

### 2 — Rapid Map
A broad orientation of a topic — not deep on any one part, but confident about how the pieces fit. Exit with a "what to learn next" map.

Best for: exploring a new domain before specializing.
```
Topic: Machine Learning
Mode: Rapid Map
```

### 3 — Debug My Understanding
You already know something but sense gaps or inconsistencies. Start by explaining what you know. Claude finds the crack. You drill it.

Best for: "I know this but I keep getting it wrong" situations.
```
Topic: Recursion
Mode: Debug My Understanding
[I know it's when a function calls itself, and I can trace simple examples, but I lose track when the stack gets deep]
```

### 4 — Interview / Exam Prep
Learn to answer specific question formats, not just understand the concept. Claude simulates the question types an interviewer or examiner actually uses.

Best for: coding interviews, professional exams, university finals.
```
Topic: Binary search trees
Mode: Interview Prep
Exam: FAANG software engineering interview
```

### 5 — Build to Learn
Every concept is introduced only when a build requires it. Understanding follows necessity.

Best for: people who learn best by doing, or when you have a specific project in mind.
```
Topic: SQL joins
Mode: Build to Learn
Build: I'm building a simple blog database with users, posts, and comments
```

### 6 — Teach to Learn
You have to explain the concept to a specific audience. Claude coaches the *explanation*, not the concept — making you find the gaps in your understanding by trying to close them for someone else.

Best for: preparing presentations, writing documentation, teaching a colleague.
```
Topic: How HTTPS works
Mode: Teach to Learn
Audience: my non-technical manager
```

---

## The Retention Package

Every session closes with a structured retention package:

**1. Summary in your voice**
3–5 bullets that capture how *you* now understand the concept — using your vocabulary and examples from the session, not a textbook definition.

**2. Spaced Repetition Questions (5)**
Questions at escalating Bloom's Taxonomy levels:

| # | Type | Purpose |
|---|------|---------|
| Q1 | Recall | Can you retrieve the core idea? |
| Q2 | Understanding | Can you explain why it works? |
| Q3 | Application | Can you use it in a new situation? |
| Q4 | Analysis | Can you distinguish it from similar concepts? |
| Q5 | Synthesis | Can you connect it to other things you know? |

Review these tomorrow, in a week, and in a month for actual retention.

**3. Application Challenge**
One concrete thing to do in the next 24 hours that uses this knowledge. Achievable, specific, and slightly uncomfortable.

**4. Forward Connection**
What this concept unlocks and what to learn next.

---

## Tips

**Trust the Feynman Challenge**
When Claude asks you to explain it back, don't skip it or say "I got it." The explanation attempt is the learning. Struggling to explain it is the most productive moment of the session.

**Be honest about your level**
Saying "I know this" when you don't wastes the session. Claude starts too deep and builds on a foundation that isn't there. "I use it but can't explain it" is the most common and most useful honest answer.

**Use "Debug My Understanding" for sticky concepts**
If you've tried to learn something twice and it keeps not sticking, Mode 3 is more efficient than Mode 1. Start by explaining what you think you know — the gap will surface in 2 minutes.

**Time-box if you're busy**
`Time: 15 min` tells Claude to use Mode 2 (map) or trim Mode 1 to essentials only. A 15-minute calibrated session beats an hour of passive reading.

**Save the Retention Package**
Copy the 5 spaced repetition questions somewhere you'll actually see them — Anki, Notion, a sticky note. The questions are worthless if you never review them.

---

## Files in This Skill

| File | Description |
|------|-------------|
| [`skill.md`](./skill.md) | The full prompt — copy this into Claude |
| [`README.md`](./README.md) | This file |
| [`examples.md`](./examples.md) | Two complete session transcripts |
| [`changelog.md`](./changelog.md) | Version history |

---

## Version

Current: `v1.0.0` — see [`changelog.md`](./changelog.md)
