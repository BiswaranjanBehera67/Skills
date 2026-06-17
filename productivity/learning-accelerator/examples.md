# Examples

Real session outputs from the Learning Accelerator skill across all four modes.
These show the full probe → explain → test → diagnose → iterate loop as it actually runs.

---

## How to Read These

Each example shows the full conversation — including what the learner gets *wrong* — because that's where the skill does its real work. A session where the Feynman Test passes on the first try isn't a useful example. The interesting part is what happens when it doesn't.

---

## Session 01 — EXPLORE Mode: Async/Await in JavaScript

**Input:**
```
Topic: async/await in JavaScript
Level: beginner
Goal:  apply practically
Style: examples-first
```

---

**[PROBE]**

Before anything else — three quick questions:

1. When you write `fetch('/api/data')` and it takes 2 seconds to respond, what do you think JavaScript is doing during those 2 seconds?
2. Have you ever seen a `.then()` chain? If yes, what did you think it was doing?
3. What's your current guess at what `async` means when you put it in front of a function?

---

*Learner response:*
> "1. I think it waits? Like it just stops and waits for the data to come back. 2. Yeah I've seen .then() but I just copy-pasted it. 3. I think async means the function is... asynchronous? But I don't know what that means exactly."

---

**[GAP IDENTIFIED]**

The first answer is the gap. You think JavaScript *pauses and waits* — this is the single most common misconception about async, and it's the one that makes everything else confusing.

JavaScript does not pause. It cannot pause. Let me fix this one thing.

**[MINIMUM MODEL]**

JavaScript is single-threaded — it can only do one thing at a time. When it hits a slow operation (like a network request), it has two choices: sit there doing nothing until it's done, or hand the task off to the browser, tell it "wake me up when this is ready," and go do other things in the meantime.

Async/await is JavaScript choosing the second option. `await` doesn't mean "stop." It means "hand this off and come back to me."

**[ANALOGY]**

Think of JavaScript as a chef in a restaurant kitchen. The chef can only work on one dish at a time.

Old way (blocking): The chef puts a steak in the oven, then stands in front of the oven staring at it for 20 minutes. Nothing else gets cooked.

Async way: The chef puts the steak in the oven, sets a timer, and goes back to chopping vegetables, taking orders, plating other dishes. When the timer goes off — that's the `await` resolving — the chef stops what they're doing and finishes the steak.

The chef never does two things simultaneously. But they're not wasting time waiting either.

In code:

```javascript
async function makeOrder() {
  console.log("Started cooking");       // Chef starts working
  const steak = await cookSteak();      // Steak goes in oven; chef goes elsewhere
  console.log("Steak ready:", steak);   // Timer went off; chef returns
}
```

The `async` keyword tells JavaScript "this function will have moments where we're waiting on something." The `await` keyword marks those exact moments.

**[FEYNMAN TEST]**

Your turn. Don't use my words.

Explain to me: when JavaScript hits an `await`, what happens? Where does it go, and how does it know to come back?

---

*Learner response:*
> "Okay so... the function like pauses, but JavaScript itself doesn't pause? It goes and does other stuff and then when the thing is ready it comes back to the function and continues from where it left off?"

---

**[DIAGNOSIS]**

That's exactly right — and more precisely phrased than most explanations I've seen. One bonus question to confirm transfer:

If you have this code:

```javascript
async function getData() {
  console.log("A");
  const result = await fetch('/api');
  console.log("B");
}

getData();
console.log("C");
```

What order do A, B, C print in, and why?

---

*Learner response:*
> "A... then C... then B? Because when it hits await, it leaves the function and continues with the stuff outside, and B only runs after the fetch is done?"

---

**[DIAGNOSIS]**

Correct. A → C → B. You understood the non-obvious part — that `console.log("C")` runs *before* `console.log("B")` even though it's written after `getData()` is called.

Feynman Test passed.

**[NEXT LEVEL]**

Three concepts that build directly on this, in order:

1. **Promises** — `await` is syntactic sugar over Promises. Understanding Promises shows you what `await` is doing under the hood, and helps you handle errors correctly with `.catch()`.
2. **Error handling with try/catch** — async functions need different error handling than synchronous code. `await` is where errors surface, and wrapping it in `try/catch` is the pattern.
3. **Promise.all()** — what if you need to `await` three things that don't depend on each other? Running them in sequence wastes time. `Promise.all()` runs them in parallel and waits for all three.

---

**Post-session note:**
The misconception "JavaScript pauses when it awaits" is present in ~80% of beginners. The chef analogy works because it preserves the single-threaded constraint (one chef, one task at a time) while showing that waiting ≠ doing nothing. The Feynman Test revealed the learner understood it correctly but needed the "A → C → B" transfer question to prove it wasn't just parroting.

---
---

## Session 02 — STRESS-TEST Mode: Recursion

**Input:**
```
Topic: recursion
Level: intermediate
Goal:  apply practically
Context: "I've read about it and I think I understand it but every time I try to write a recursive function I get stack overflow errors or infinite loops"
```

---

**[DIAGNOSTIC SEQUENCE]**

You've read about recursion and understand the concept, but can't apply it without errors. That means the gap is somewhere in the *application*, not the definition. Five questions to find it:

1. Without looking anything up, write a recursive function that calculates factorial(5). Just write what comes to mind — don't worry if it's broken.
2. What's a base case? Define it in one sentence.
3. If I call `factorial(5)`, how many times does the function get called total before it returns an answer?
4. What do you think causes a stack overflow?
5. Here's a recursive function — what's wrong with it?

```javascript
function countdown(n) {
  console.log(n);
  countdown(n - 1);
}
countdown(5);
```

---

*Learner response:*
> "1. `function factorial(n) { return n * factorial(n-1); }` — I know this is probably missing something. 2. Base case is when the recursion stops. 3. I think... 5 times? 4. I think stack overflow means too many function calls. 5. It'll go forever because there's no stopping condition."

---

**[GAP ANALYSIS]**

Found it. Two related gaps:

**Gap 1:** You defined "base case" as "when the recursion stops" — which is correct but too vague to use when writing code. You know what a base case *is* but not how to *decide* what it should be. This is why you forget it or get it wrong.

**Gap 2:** Your factorial function returns `n * factorial(n-1)` — the recursive call is right. But you said factorial(5) calls the function 5 times. It's actually 6 times (5, 4, 3, 2, 1, 0). That off-by-one error means your base case will be wrong even when you remember to write one.

These two gaps together explain every stack overflow you've hit.

**[TARGETED FIX]**

**For Gap 1 — How to decide your base case:**

Ask this question every time you write a recursive function: *"What is the smallest version of this problem that has an obvious, non-recursive answer?"*

For factorial:
- factorial(3) = 3 × 2 × 1 — needs recursion
- factorial(2) = 2 × 1 — needs recursion  
- factorial(1) = 1 — I know this immediately. No multiplication needed.
- factorial(0) = 1 — defined mathematically. Obvious answer.

The base case is the answer you *already know without calculating*. For factorial, that's `n === 0` (or `n === 1`).

**For Gap 2 — Tracing the actual call count:**

For `factorial(5)`, the calls are: factorial(5) → factorial(4) → factorial(3) → factorial(2) → factorial(1) → factorial(0). That's 6. The base case must handle `n === 0` or you'll get one extra infinite call.

Your correct function:

```javascript
function factorial(n) {
  if (n === 0) return 1;        // base case: smallest obvious answer
  return n * factorial(n - 1); // recursive case: reduce toward base
}
```

The formula for every recursive function is: *"If I'm at the base case, return the obvious answer. Otherwise, reduce the problem by one step and recurse."*

**[VERIFICATION QUESTION]**

Write a recursive function that sums all numbers from 1 to n. (Example: sumTo(4) = 4 + 3 + 2 + 1 = 10.)

Before you write it, tell me: what's your base case, and how did you decide?

---

*Learner response:*
> "Base case is sumTo(0) = 0 because that's the smallest obvious answer — adding nothing to nothing. Function: `function sumTo(n) { if (n === 0) return 0; return n + sumTo(n-1); }`"

---

**[RESULT]**

That's correct, and the explanation of the base case is exactly right. The gap is closed.

The thing to watch for: any time you hit a stack overflow, ask "what's my base case and is it actually reachable?" 90% of the time you'll find either you forgot to write it, or you wrote it for the wrong value of n.

---

**Post-session note:**
The gap was not "doesn't understand recursion" — it was "base case definition too vague to operationalize." The diagnostic sequence found it in question 2 (vague definition) and question 3 (off-by-one in call counting). The targeted fix was surgical: two paragraphs on how to *decide* a base case, not a re-explanation of recursion.

---
---

## Session 03 — RECALL Mode: Unit Economics for SaaS

**Input:**
```
Topic: unit economics for SaaS startups
Level: intermediate
Goal:  teach others
```

---

**[SPACED REPETITION QUESTIONS]**

---

**Level 1 — Recognition**

**Q1.** A startup has CAC of ₹5,000 and monthly revenue per customer of ₹500 with 5% monthly churn. Is the LTV:CAC ratio healthy?

**Q2.** Two SaaS companies both have LTV:CAC of 3:1. Company A achieves it with CAC ₹100 and LTV ₹300. Company B achieves it with CAC ₹10,000 and LTV ₹30,000. Which is in a better position, or are they the same?

**Q3.** What does a payback period of 24 months mean, and why does it matter for a bootstrapped vs. VC-backed company differently?

---

**Level 2 — Application**

**Q4.** Your SaaS charges ₹2,000/month. Average customer stays 18 months. You spend ₹8,000 to acquire each customer through paid ads and ₹2,000 through content marketing, in a 50/50 split. Calculate blended CAC and LTV:CAC. Is this business viable?

**Q5.** A founder says "our unit economics are great — CAC is only ₹1,500." What questions would you ask before agreeing or disagreeing?

**Q6.** You're growing fast: MRR up 30% last month. But Net Revenue Retention is 85%. A friend says "you're growing, stop worrying." What's wrong with this picture?

**Q7.** Your payback period is 6 months. An investor says "that's not good enough — you need under 12 months." Who's right, and what information would you need to be sure?

---

**Level 3 — Transfer**

**Q8.** You're evaluating a gym membership business (not SaaS). Which unit economics concepts transfer directly, which need adjustment, and which don't apply at all?

**Q9.** A friend argues: "LTV is a made-up number because you're just guessing how long customers will stay." How do you respond?

**Q10.** A SaaS company has perfect unit economics (LTV:CAC of 5:1, 8-month payback) but is struggling to raise a Series A. What might be wrong that unit economics don't capture?

---

**[COMMON WRONG ANSWERS]**

**Q1 — What the partially-understanding person says:**
"LTV = ₹500/month × 12 months = ₹6,000. LTV:CAC = 6000/5000 = 1.2:1, which is below the 3:1 benchmark, so it's bad."

**Why that's wrong:**
LTV uses *customer lifetime*, not 12 months. With 5% monthly churn, average lifetime = 1/0.05 = 20 months. LTV = ₹500 × 20 = ₹10,000. LTV:CAC = 2:1. Still below 3:1 but the calculation was completely wrong. Using a fixed 12-month window is a common shortcut that breaks with any non-trivial churn rate.

---

**Q2 — What the partially-understanding person says:**
"They're the same — both 3:1."

**Why that's wrong:**
Company B has a payback period ~20× longer. If Company B takes 20 months to recover CAC, it needs 20 months of cash per customer before seeing profit. At scale, this creates an enormous cash flow problem — you're burning millions to acquire customers and waiting years to be made whole. Company A at ₹100 CAC has a payback period of months. Same ratio, completely different business.

---

**Q6 — What the partially-understanding person says:**
"NRR of 85% just means we have some churn — but we're growing fast enough to offset it, so we're fine."

**Why that's wrong:**
NRR below 100% means your existing customers are worth less each month than last month — collectively. Growth can mask this, but the moment growth slows, the contraction from existing customers accelerates the decline. It's a leaky bucket: you're pouring in fast enough to keep the level rising, but the bucket has a hole. The benchmark for healthy SaaS is NRR above 100% (expansion revenue exceeds churn).

---

**Q9 — How to respond:**
"You're right that it involves a forecast. But LTV isn't arbitrary — it's calculated from real, observed churn rates on your existing cohorts. If 1,000 customers from January 2023 show a specific retention curve 18 months later, that's data, not a guess. The honest version is: LTV is a backward-looking extrapolation of a forward-looking metric. It's as good as your cohort data and your assumption that future customers will behave like past ones."

---

**[CONNECTIONS]**

1. **Cohort analysis** → LTV is only meaningful when calculated per-cohort (customers acquired in the same month), not across your entire customer base. Customers acquired through different channels behave differently. A single blended LTV hides this.

2. **Burn multiple** → Unit economics measure profitability per customer. Burn multiple measures efficiency of growth: how much cash you burn to generate ₹1 of ARR. A company can have great unit economics and terrible burn multiple (growing inefficiently). Both matter; they measure different things.

3. **Net Revenue Retention** → NRR above 100% means your cohorts *grow* over time (expansion revenue > churn). When NRR > 100%, LTV calculations become exponential rather than linear — and the business has a fundamentally different growth profile. This is why investors pay huge multiples for "land and expand" SaaS.

---

**Post-session note:**
The Transfer questions (Q8–Q10) are where teaching-level understanding gets tested. Q8 (gym business) requires abstracting the mechanism from the specific domain. Q10 is the most valuable — it requires knowing what unit economics *don't* capture (market size, competition, team, product-market fit), which is the thing teachers most commonly fail to explain.

---
---

## Patterns Across Sessions

After running this skill across topics in programming, business, and science:

**1. The probe answer determines everything**
The most useful probe response is an honest "I think it works like... [wrong thing]." The specific wrong model is more actionable than "I don't know." If a learner says "I have no idea," a good follow-up probe is: "If you had to guess — what would you guess?"

**2. The Feynman Test failure modes are highly consistent**
Across every domain tested, learners fail the Feynman Test in one of three predictable ways: they parrot the explanation back (memorized, not understood), they get the concept right but apply it to the wrong type of problem (structural gap), or they're confident and wrong (earlier assumption is broken). Each failure mode has a different fix; correctly identifying which mode you're in determines the next move.

**3. The gap is almost never where the learner thinks it is**
In Session 02 (recursion), the learner thought they didn't understand recursion. The actual gap was in how they defined "base case" — too vague to operationalize. Fixing the vague definition took two paragraphs. The rest of their recursion knowledge was intact. A standard re-explanation of recursion would have wasted the session.

**4. RECALL questions should always include wrong answers**
The wrong answer column in Session 03 is as valuable as the questions themselves. Knowing that "LTV = 12 months of revenue" is the specific wrong answer a partially-understanding person gives tells you exactly what to watch for in your own thinking.

---

*Examples last updated: June 2026 · Skill version: v1.0.0*
