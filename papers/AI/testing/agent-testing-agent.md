# Agent-Testing Agent: A Meta-Agent for Automated Testing and Evaluation of Conversational AI Agents

> Source: https://aclanthology.org/2026.eacl-long.339.pdf

## Abstract

**What this paper is about:**
- Presents the **Agent-Testing Agent (ATA)**, a "meta-agent" — an AI agent whose job is to test *other* AI agents
- ATA builds tests automatically by combining four sources of information, then uses judge feedback to make each next test harder or easier

**What ATA uses to generate tests:**
- **Static code analysis** — reading the target agent's actual code
- **Developer interrogation** — asking the person who built the agent about requirements and hidden assumptions
- **Literature mining** — searching papers and bug reports for known failure modes
- **Persona-driven adversarial testing** — inventing tricky fake users whose difficulty adapts based on judge feedback

**How it scores results:**
- Each test conversation is scored with an **LLM-as-a-Judge (LAAJ)** rubric (an AI that grades the dialogue against a checklist)
- Those scores steer the next test toward the target agent's weakest abilities

**The headline result:**
- Tested on a **travel planner** and a **Wikipedia writer**, ATA found *more* diverse and *more severe* failures than expert human annotators — while matching them on severity
- ATA finished in **20–30 minutes**, versus ten-annotator rounds that took **days**
- Removing code analysis and web search made ATA less reliable (more variance, worse calibration) — showing those grounded sources matter

**What developers get out of it:**
- Quantitative metrics (scores) plus qualitative bug reports
- The full implementation is **open source**

## 1 Introduction

**The problem ATA solves:**
- Modern AI agents chain model calls with tools and memory to accomplish user goals, but they often fail under distributional shift (new situations), noisy tool responses, and subtle interactions between constraints
- Yet evaluation still relies on **static, manually curated benchmarks** and **small human studies**

**Why static benchmarks fall short:**
- They don't cover the huge "combinatorial" space of possible inputs
- They age quickly as agent architectures evolve
- They're costly to maintain

**The two existing approaches ATA sits between:**
- Static benchmarks (like TRAVELPLANNER for itinerary design) — fixed test lists that don't adapt
- LLM-as-a-judge systems (using AI to grade outputs) — powerful but usually assume a human already wrote the test list

**What ATA adds:**
- It *automatically generates* the tests, instead of assuming a human-authored test list already exists

## 2 The Four Sources of Information

**ATA's test generation draws on four ingredients:**
- **(i) Codebase analysis** — reads the target agent's source code to build a picture of its structure
- **(ii) Developer interrogation** — asks the agent's creator to surface requirements and implicit assumptions that aren't written down anywhere
- **(iii) Literature/dataset retrieval** — mines academic papers and datasets for likely failure modes
- **(iv) Persona synthesis** — writes persona-driven dialogues whose difficulty is adjusted by a "posterior" (a running estimate of how hard to make things) updated after every judge score

**The three contributions:**
1. A **weakness-planning algorithm** that keeps an explicit difficulty estimate and uses it to adapt test generation on the fly
2. A **fully open-source evaluator** built on standard APIs, needing no domain-specific annotation, with both a CLI and a web interface for rapid developer use
3. **Evidence** that ATA covers more diverse and severe failure modes than expert annotators, at a fraction of the time and cost

## 3 Method

### 3.1 Weakness Planning

**Code & User Grounding:**
- First, select the **agent under test (AUT)** and initialize a shared "global state"
- Run a **static scan** of the AUT's codebase to build its structure
- Reason through plausible weaknesses using a **chain-of-thought prompt** (step-by-step reasoning), producing a validated, prioritized list of failure types to probe

**Developer interrogation:**
- ATA asks the developer questions, but stops early based on **information gain** — it stops when it has what it needs or when the user seems done answering
- This keeps the burden light on the human while maximizing what ATA learns
- Answers go into the shared state and personalize later steps

**Web search:**
- ATA enters a literature-search loop: over `n` iterations it retrieves `m` academic papers, public datasets, or bug reports about the target domain
- Each is summarized to extract lessons (common failure modes, recommended evaluation styles)
- It then reformulates its queries using those insights — a "bootstrap" literature review tailored to the agent's goals

**Weakness generation:**
- Combining code analysis, user input, and web-search context, a chain-of-thought prompt generates a list of potential weaknesses
- These are shown to the user for validation and refinement

### 3.2 ATA Thread

**Parallel execution:**
- Once weaknesses are identified, ATA launches a **dedicated thread per weakness**
- Threads run in **parallel**; each is responsible for generating adaptive test cases, simulating multi-turn conversations, and updating its own difficulty model based on performance

**Building each test prompt:**
- Combines (a) the user goal, (b) the linguistic tone (e.g. vague, impatient), (c) a **turn limit** (8–10 turns for medium tests, scaled by difficulty), and (d) the evaluation criteria

**Dialogue execution:**
- ATA talks to the AUT *from the generated persona's perspective* until the turn limit is reached or the goal is met
- The AUT doesn't know it's talking to an agent — it behaves as if helping a real person
- The full transcript is logged

**Evaluation with LAAJ:**
- After each dialogue, ATA grades it using the **LLM-as-a-Judge** framework, backed by the same deep reasoning agent that generated the dialogue
- The judge gets the scenario's purpose as context (mirroring how human evaluators know the test they created before scoring it)
- Beyond a numeric score, the judge writes **textual observations**: detailed analysis reports with specific dialogue examples, identified strengths/weaknesses, and insights about the agent's decision-making

**Difficulty adaptation (the formula):**
- Scores drive difficulty updates through a **logistic function** (an S-curve that maps score changes into the range [−η, η], with η = 3)
- This caps how much a single test can move difficulty, while still letting good scores have a real impact
- The loop runs **three rounds** (or until difficulty converges), "homing in" on the agent's failure boundary: harder tests after success, easier ones after failure
- Tests that score far from the midpoint (5.5) are weighted less, since they indicate the agent either wasn't challenged enough or was overwhelmed — either way, a less informative test

## 4 Experiments

### 4.1 Agents Under Test

**Travel-Planning Agent:**
- Plans complete trips through natural conversation, combining real-time web search with budget-aware optimization
- Supports multi-destination itineraries (flights, hotels, activities), follows up to elicit missing constraints, and explains trade-offs when a plan is infeasible
- Rubric evaluates **constraint handling** and **communication**

**Wikipedia Writer:**
- Rubric evaluates **citations**, **completeness**, and **style/organization**

### 4.2 Human Baseline

- Ten professional annotators per agent, each completing **three persona tests**
- Each annotator: define a persona (attitude + goal), run the dialogue from that persona's view, then score it with the agent-specific rubric plus free-text notes
- Each annotator worked 8 hours (4 hours per agent)

### 4.3 Results and Discussion

**Travel planner — what humans flagged (frequency = # of annotators who mentioned it):**
- Context retention & consistency issues (7)
- Tone & interpersonal issues (5)
- Constraint handling / partial constraint correction (5)
- Structure & formatting (2)
- Performance & speed (1)

**Travel planner — weaknesses ATA found that humans missed entirely:**
- Ambiguous user references
- Contradictory or unsatisfiable constraints
- Malformed or nonsensical input handling
- Unsafe/illegal activity requests
- Topic transition & digression recovery
- Hallucinated availability & pricing

**Wikipedia writer — what humans flagged:**
- Citation & sourcing issues (8)
- Structure & formatting (6)
- Context retention & consistency (4)
- Factual accuracy & misinformation (3)
- Tone & interpersonal issues (3)

**Wikipedia writer — weaknesses ATA found that humans missed:**
- Contradictory constraint handling
- Safety & harmful content moderation
- Overconfidence under uncertainty
- Topic drift & focus maintenance
- Persona adaptation issues

**Four key findings from the comparison:**
- **Long-context reasoning** is a bigger bottleneck for the Wikipedia writer (scored 4.3/10) than for the travel planner — multi-page article writing stresses long-context more than conversational planning, even though humans notice context gaps in both
- **Tone & interpersonal quality** is flagged frequently by humans but *undetected by ATA* — a known complementarity: humans better capture pragmatic/interpersonal expectations that are hard to encode as weaknesses up front
- **Structure & formatting** matters far more to humans for Wikipedia (6 mentions) than travel (2), yet ATA rates Wikipedia's structure highly (7.6–7.8) — humans apply stricter stylistic standards (e.g. Manual of Style adherence) than ATA's rubric encodes
- **Performance & speed** is mentioned infrequently by humans

**Holistic comparison:**
- Both routes surface overlapping *functional* weaknesses (constraint gaps in travel; citation/structure gaps in Wikipedia)
- Humans emphasize language, tone, and phrasing — dimensions ATA treats mechanically (it's an LLM too)
- ATA's threaded, adaptive design uncovers *deeper capability-level* problems humans don't systematically exercise, because each human explores different angles while ATA holds one weakness constant per thread and iterates across three personas

**The practical takeaway (a workflow):**
- Use ATA for **fast, depth-first probes** and aggregate scoring
- Then deploy **targeted human review** for tone, interpersonal quality, and style

**Cost/time:**
- ATA completes a full run in **20–30 minutes** on an Apple M3 Pro
- The ten-annotator round cost roughly **$1,600** (10 people × 8 hours × $20/hour), plus coordination overhead
- ATA costs roughly **$2.50–$3.50 per run**

## 5 Ablation Study

**What was removed:**
- The ablated ATA drops both **static code analysis** and **web search**, generating weaknesses only from user-provided goals and shallow persona prompting

**What happened:**
- The ablated version is **notably miscalibrated** on categories where humans and full ATA agree
- It's **too lenient** on context retention for Wikipedia (8.5/10 vs. 4.3/10 full ATA)
- Yet **too harsh** on travel constraint handling (4.1/10 vs. 6.3/10 full ATA)
- Sometimes it lands closer to human ratings in aggregate utility, but this reflects **variance**, not reliability

**The lesson:**
- Removing code analysis and literature search makes ATA **lose sensitivity** to criteria that hinge on factual grounding or structural reasoning — evidence-grounded test generation genuinely matters

## 6 Limitations

- **Circularity risk:** ATA uses LLMs both to generate adversarial personas *and* to judge outcomes — the same class of model being tested also provides the evaluation scaffolding, which may bias judgments toward failure modes salient to LLMs rather than to humans
- **Text-only scope:** extensions to multi-agent coordination, multimodal interfaces, or high-latency/stochastic toolchains would need reworked test generation and judge criteria (flagged as future work, out of scope here)
- **No direct quantitative comparison** to related frameworks (ALMITA for customer support, FACT-AUDIT for fact-checking, GOAT) — they target specific domains or static benchmarks, and ATA's grounding paradigm is different enough that head-to-head comparison needs domain-specific reimplementation

## Appendix: Cost & Model Details

**Model usage:**
- Most phases use **GPT-4.1 mini**: code analysis, parameter gathering, web search, dialogue generation, judging, and report writing
- The **o3** model is used selectively for (i) global weakness-planning and (ii) the "thought loop" at each test-case generation (18 calls total) — more expensive, but a small fraction of overall tokens

**Cost breakdown (April 2025 pricing):**
- GPT-4.1 mini: $0.15/1k input, $0.60/1k output
- o3: $1.10/1k input, $4.40/1k output
- Blended cost per run: roughly **$2.50–$3.50**, scaling linearly with weaknesses × test cases × dialogue turns, and depending on how verbose the target agent is

**Time breakdown (Apple M3 Pro, ~20–30 min total):**
- 2–3 min code analysis
- 5–7 min retrieval
- 12–15 min dialogue execution + judging
- 3–4 min report synthesis
