# Evaluation Techniques

> Source: https://careersatdoordash.com/blog/building-ask-doordash-part-three-evaluation/

## About this file

This is a running index of ways to evaluate AI agents. Add each new technique here as you come across it. Each entry explains what the technique is, how it works, and when to use it — written so a beginner can follow along.

The techniques below all come from DoorDash's blog post about how they test their food-ordering agent ("Ask DoorDash").

---

## 1. Rubric-Based Evaluation

**What it is:** A checklist of rules that define what a "good" answer looks like, used to grade the agent.

- Agent tasks usually don't have one single correct answer, so you grade against *criteria* instead of expecting one exact result.
- Each criterion is a simple **pass / fail** check — not a number score.
- The pass/fail checks are added up into one final score for the whole conversation.
- The rules describe what a *successful* session looks like, rather than dictating one exact answer.

**Example criteria:**
- *Communication quality*: kept the reply short, didn't talk through its reasoning out loud.
- *Trust & honesty*: didn't state anything false or contradict what the user was shown.
- *Meeting constraints*: results respected limits like delivery time, budget, or dietary needs.

## 2. Eligibility Gating

**What it is:** Deciding whether each rule even applies before grading it.

- Before grading a criterion, the judge first asks: "does this rule apply to *this* conversation?"
- If it doesn't apply, the rule is skipped — the session isn't penalized for it.
- This prevents unfair markdowns. Example: a recipe request shouldn't be graded down for missing a "reorder groceries" step, because that step was never required.

## 3. Checklist Rubric (Deterministic Offline Checks)

**What it is:** A scenario-specific list of facts that must be true for a correct run.

- Unlike the broad criteria above, these are concrete and predictable (low-variance).
- Example: for a "vegetarian taco groceries for two, under $60" test, verify (a) every item is actually an ingredient for vegetarian tacos, and (b) the subtotal is $60 or less.
- These let you test a specific behavior directly, so even a small batch of test runs is useful and lets you iterate fast.

## 4. Transcript Builder (Criterion-Specific Views)

**What it is:** Turning the raw record of a session into a clean, focused view that a judge can read.

- Every session is recorded as a **trace** (a log of every step) via a tool called **OpenTelemetry**, and stored in **ClickHouse** — a fast analytical database.
- The trace captures: what the user typed, what the model output, what tools it called, and what it showed on screen.
- The raw trace is complete but noisy — some tool responses are huge, and lots of it (like schema scaffolding) carries no signal about quality.
- The builder cleans this up: reassembles scattered steps, removes noise, trims oversized payloads.
- Each criterion says *which piece of evidence it needs*, and the builder hands the judge *only that slice*. (A "did it tell the truth" check gets the claim + backing tool output; a "are results varied" check gets the recommendations + the original request.)

**Why it matters:** Focused evidence makes judging more consistent.

## 5. Conversation Simulator (Offline Rehearsal)

**What it is:** Generating fake test conversations with an AI playing the customer, so you can test a new version of the agent before real users see it.

- A **simulated user** (an LLM posing as the shopper) drives the conversation.
- A **scenario** defines the opening message, the user's goal, and how they should react to questions.
- **Fixtures** are pre-recorded tool responses used in place of live calls. They freeze outside state (past orders, cart, inventory) so every run of a scenario sees identical data — no drift from changing catalogs or store availability.

**Scale example:** 50 scenarios × 8 tries each = 400 conversations. Doing this by hand (chatting with the agent, ~1 min each) took over 6 hours; the simulator does it in ~20 minutes.

## 6. LLM-as-a-Judge (Calibrated)

**What it is:** Using an AI model to do the grading automatically, in place of a human.

- The judge reads the cleaned-up transcript and scores each criterion on its own, giving a verdict plus a short reason.
- It's only useful if it *agrees with human reviewers*, so it's **calibrated**: a method called **GEPA** repeatedly suggests edits to the judge's instructions and keeps the edits that improve agreement against human-labeled examples.
- Calibration is ongoing — when the rubrics change, new labels are collected and the judge is retuned.
- Humans still do the high-value work: writing the rules, labeling example sessions, checking the judge's behavior.

## 7. Failure Clustering into Themes

**What it is:** Grouping failures together so you fix the most common problems, not just the latest one.

- Each failed session already lists *which rule it broke*, so the rule itself becomes a natural group.
- Grouping by broken rule separates distinct problems: a "made stuff up" failure is different from a "missed a substitution" failure and should be investigated separately.
- This turns a long list of individual failures into a ranked list of themes, so you fix what matters most.

## 8. Agent Skills for Eval-Driven Development

**What it is:** Packaging "fix the failures" workflows as reusable skills that an AI coding agent can run.

- Starting from a failure cluster, a coding agent inspects the failing traces, relevant code, recent changes, and past investigations.
- If the fix is clear, it drafts a pull request directly. For prompt changes, it writes a diagnosis and a proposed fix for a human to review.
- Each workflow is saved as an **Agent Skill** (a reusable task). When a skill makes a wrong call, you fix the *skill* rather than patching one run — so the fix carries forward.

---

## Lessons Learned

**Judges need evidence shaped for the question:**
- Remove unrelated and no-signal content; keep only the evidence the rule actually needs.

**The environment must be controlled, or the score measures noise:**
- Fixtures freeze outside state; fixed scenarios keep the task stable — so score changes reflect the agent, not its surroundings.

**An offline result only matters if it carries to production:**
- Use the *same* rubric and calibrated judge in both places, or the scores can diverge in ways you can't reconcile.

**The eval system reports its own bugs too:**
- A real share of "failures" are faults in the test harness (judge false-positives, incomplete traces), not the agent. Fix these to keep the eval trustworthy.