# Learning Accelerator

> *Find the gap. Fill the gap. Prove the gap is gone.*

A structured prompting skill that turns Claude into a Feynman-technique teacher. Instead of generating explanations at you, it finds exactly where your current understanding breaks down — then fixes only that, nothing more — and verifies the fix worked before moving on.

---

## Why This Exists

When you ask Claude "explain X to me," Claude gives you a good explanation. But good explanations don't guarantee understanding. You can read a perfect explanation of recursion, nod along, close the tab, and still be unable to write a recursive function 20 minutes later.

The problem: **explanations are calibrated to the concept, not to your current model.** They fill in what you don't know without first checking what you do know, and without verifying anything actually landed.

This skill enforces a different process:

```
What do you already know? → Where exactly does your model break down?
→ Minimum fix for that gap → Did the fix actually work? (Feynman test)
→ Repeat until you can explain it to someone else → Generate recall questions
```

The difference is not style — it's sequence. The probe comes before the explanation. The test comes before moving on. The recall questions come before ending the session.

---

## The Four Modes

### EXPLORE
*"Teach me X" / "I want to understand X" / "Explain X"*

For learning something new. The skill probes your starting point, builds a minimum working model using an analogy from something you already know, then tests whether you genuinely understood it or just recognized it.

**What you get:** A minimum viable mental model → one calibrated analogy → a Feynman test → diagnosis if you fail → the three concepts to learn next once you pass.

---

### STRESS-TEST
*"I know X but I'm confused about..." / "Why does X do Y?"*

For concepts you think you understand but that collapse when you try to use them. The skill runs a diagnostic sequence of 5 increasing-specificity questions to find exactly where your model breaks down — then fixes only that part.

**What you get:** The precise gap identified by name → a surgical fix → a verification question you can only answer if the gap is genuinely closed.

---

### PREPARE
*"I have an exam on X" / "I need to learn X by Friday"*

For structured coverage of a topic area. The skill maps the full learning landscape — which concepts depend on which, what order makes sense, how long each piece takes, and where most people go wrong before you get there.

**What you get:** A dependency-ordered map → prioritized study sequence → time estimates per topic → danger zones (common misconceptions) → the exact first question to start with.

---

### RECALL
*"Quiz me on X" / "I learned X last week, test me" / "Help me retain X"*

For concepts you've already learned. The skill generates 10 spaced repetition questions at three difficulty levels — recognition, application, and transfer — plus the wrong answers that partially-understanding learners give, so you know what to watch for.

**What you get:** 3 recognition + 4 application + 3 transfer questions → the specific wrong answer a partially-understanding person gives for each → 3 connections to related concepts.

---

## Input Format

**Minimum (works fine):**
```
Topic: [what you want to learn]
```

**Full format (sharper results):**
```
Topic:    [concept, skill, or subject area]
Level:    beginner / intermediate / advanced
Goal:     understand deeply / apply practically / pass exam / teach others
Time:     [how long you have — optional]
Style:    examples-first / theory-first / problem-first
Context:  [what you already know that's related — optional]
```

**Constraint effects:**

| Constraint | What it changes |
|---|---|
| `Goal: pass exam` | Depth stops at exam scope. No rabbit holes. |
| `Goal: teach others` | Pushes to edge cases and common student mistakes. |
| `Goal: apply practically` | Leads with working examples before theory. |
| `Style: examples-first` | Analogy and example before any formal definition. |
| `Context: I know Python` | Uses Python-domain analogies. Skips CS fundamentals you already have. |
| `Time: 20 minutes` | Cuts to minimum model only. Flags what's being skipped. |

---

## How the Feynman Loop Works

The skill never asks "does that make sense?" — because learners say yes ~90% of the time regardless of actual comprehension. Instead it uses one of four tests:

| Test | Example prompt |
|---|---|
| **Re-explain** | "Explain async/await back to me as if I've never heard of it." |
| **Novel example** | "Give me a new example of memoization that I haven't used." |
| **Application** | "If this API call fails, walk me through what your code does." |
| **Debug** | "Here's broken code — what's wrong and why?" |

Your response to the test tells the skill exactly what happened:

- **Explained it in your own words** → you got it. Go deeper.
- **Parroted the explanation back** → memorized, not understood. Novel problem incoming.
- **Correct structure, wrong detail** → specific misconception named and fixed.
- **Confident but wrong** → earlier assumption is broken. Skill backs up to find it.
- **Silence** → analogy didn't work. New analogy, simpler model.

This loop repeats until you can explain the concept in your own words **and** apply it to a scenario the skill hasn't shown you. That's the minimum bar for actual understanding.

---

## Tips

**Let it probe you before jumping in**
The first thing the skill does is ask you questions. Don't skip past them — your answers to the probe questions are what determine the quality of everything that follows.

**Give honest answers to the probe**
"I have no idea" is a better probe answer than a guess that sounds more confident than it is. The skill calibrates to your actual starting point.

**Don't say "yes" to "does that make sense?" if it doesn't**
The skill won't ask this (it's forbidden). But if you feel the urge to say "got it" before you could actually explain the concept — don't. Ask for another pass.

**Use STRESS-TEST for concepts you half-know**
"I kind of understand X" is the perfect STRESS-TEST input. The diagnostic sequence finds the exact boundary of your understanding with surgical precision — the experience is uncomfortable in a productive way.

**Run PREPARE before a big study session**
The dependency map from PREPARE mode changes how you study: instead of going through a syllabus in order, you front-load the load-bearing concepts that make everything else make sense.

**Always end with RECALL**
Understanding fades. The spaced repetition questions from RECALL mode are designed to be answered 1 day, 3 days, and 1 week later — don't just read them once and close the tab.

---

## Files in This Skill

| File | Description |
|------|-------------|
| [`skill.md`](./skill.md) | The full prompt — copy this into Claude to activate |
| [`README.md`](./README.md) | This file |
| [`examples.md`](./examples.md) | Real session outputs across all four modes |
| [`changelog.md`](./changelog.md) | Version history |

---

## Version

Current: `v1.0.0` — see [`changelog.md`](./changelog.md)
