# Efficient Agent Evaluation via Diversity-Guided User Simulation

> Source: https://aclanthology.org/2026.acl-industry.112.pdf
>
> Authors: Itay Nakash, George Kour, Ateret Anaby-Tavor (IBM Research)
>
> Published: ACL 2026 Industry Track, pp. 1627–1648

## Abstract

**What this paper is about:**
- A new way to test customer-facing AI agents (AI that talks to users over many back-and-forth turns) called **DIVERT**
- DIVERT makes testing cheaper *and* better at finding bugs, compared to how agents are tested today

**The problem with how agents are tested today:**
- An **agent** is an AI that chats with a user over many turns while calling tools and APIs
- Testing it means running the whole conversation from the start, over and over, with slight random changes each time
- Two big flaws: it wastes money by redoing the same early turns every run, and it mostly sticks to "normal" friendly users — so it misses failures that only show up when a user acts unusual

**The core idea behind DIVERT:**
- Conversations aren't really linear — they look more like a **tree**: many talks share the same start, then split at a few important "decision points"
- Instead of restarting from the beginning each time, DIVERT **saves a snapshot** of the conversation at key moments and **branches off** from there
- From each branch point, it generates a *deliberately different* user response to steer the agent somewhere new

**The result:**
- DIVERT finds more failures while using *fewer* total tokens (computing budget) than the old method
- Tests become cheaper and catch more problems

## 1. Introduction

**Why evaluating agents is hard:**
- Normal AI testing is "one question in, one answer out". Agent testing is different — behavior *emerges* from long conversations, random decisions, and errors that pile up over many turns
- The same setup can give different results each run, because *both* the agent's choices *and* the user's responses vary

**Why tests need to run many times:**
- **User-side variety** — different user behaviors can send the agent down completely different paths, so you need to see how it handles them
- **Agent-side consistency** — an agent that only succeeds *sometimes* on the same task is unreliable, and you can only catch that by re-running

**Three flaws in the standard "start from scratch" approach:**
1. **Wasteful** — every run regenerates nearly identical early turns (logins, basic questions), burning tokens for nothing
2. **Blocks cache reuse** — because the repeated prefixes are only "similar" not "identical", the system can't reuse its saved work (KV-cache)
3. **Poor coverage** — standard user simulators are overly friendly and cooperative, so they rarely trigger the failures that come from *rare* user behavior

**The key observation:**
- Today's evaluations are linear, but agent conversations are actually **tree-shaped** — many share long beginnings and only split at a few critical points
- Restarting from the root throws away this structure, so you can never *systematically explore* a promising or fragile state once you reach it

**What DIVERT does instead:**
- Saves the full state (conversation, environment, tools) at critical "junction" points, then branches from there
- A "directed divergence" step creates user responses that are *different in meaning* but *still intent-consistent*, pushing the agent toward unexplored failure modes

## 2. Related Work

**Agent evaluation benchmarks:**
- General benchmarks like AgentBench, GAIA, and WebArena test reasoning, tool-use, and web interaction
- For customer-facing settings with rules to follow, **τ-bench** and **τ²-bench** model realistic domains (airline, retail, telecom) with tools, policies, and LLM-driven user simulators

**User simulation (the "fake user" that talks to the agent):**
- Early simulators used rigid rule-based scripts; newer ones use LLMs to create varied interactions
- A known weakness is **benevolence bias** — LLM simulators are too cooperative, unlike real users
- Recent work counters this with "non-collaborative" simulators modeling malicious intent, emotional manipulation, and insistence

**What makes DIVERT different:**
- DIVERT isn't another user simulator — it's a *branching evaluation structure* that works on top of *any* simulator (friendly, adversarial, or red-team)
- So it's compatible with whatever user behavior you want to test

**Evaluation efficiency:**
- Most benchmarks rely on restart-from-root rollouts, causing redundant early turns and missing deep failures
- "Cost-of-pass" studies show reliable statistics can demand huge token budgets because identical prefixes keep getting regenerated
- Tree-search methods (like Monte Carlo Tree Search) exist in agent *training*, but *evaluation* still lacks a branching mechanism — DIVERT fills that gap

## 3. Method

### 3.1 Overview of the DIVERT Pipeline

**The idea in one line:**
- Resume execution from saved mid-conversation states instead of restarting from the beginning every time

**The four stages of the pipeline:**
1. **Initial rollout + snapshot** — run a conversation once and save the full state at key points
2. **Junction selection** — pick the "pivotal" turn where a change is most likely to alter what happens next
3. **Diversity-guided user response generation** — create a deliberately different (but intent-consistent) user reply
4. **Snapshot-based resumption** — reload the saved state and continue from the branch point

**How branching scales:**
- Every branch point can itself be branched later, expanding coverage toward under-explored areas over multiple rounds
- The number of branches per trajectory is a setting (hyperparameter) you can tune
- In practice, trajectories are sampled **round-robin** to avoid unfairly favoring trajectories that already failed

### 3.2 Junction Selection

**What the "junction chooser" does:**
- An LLM reads the full conversation (user messages, agent replies, tool calls) and picks the single user turn most likely to change downstream behavior if altered
- It outputs (a) which turn to modify and (b) *why* changing it should cause the biggest behavior change while still keeping the user's intent intact

**Why this matters:**
- It branches at *meaningful decision points* rather than random or evenly-sampled turns
- Each branching attempt picks its junction independently, so different "what-if" pivots get explored across repetitions

### 3.3 Diversity-Guided Directed User Response Generation

**How alternative user replies are made:**
- At the chosen junction, generate a small set of candidate responses (fixed at **k = 3** in all experiments)
- Generation is told to keep the original task intent, but vary the wording/meaning meaningfully

**How the "most different" reply is picked:**
- Each reply is turned into an **embedding** (a numerical fingerprint of its meaning)
- The candidate with the *lowest cosine similarity* to the original reply is chosen — i.e., the most semantically different one

**Preventing the reply from drifting off-task:**
- Candidates are conditioned on the user's backstory and the evaluation purpose
- An offline check (LLM-as-judge) confirms intent is preserved — this runs *after* testing, so it costs nothing during evaluation
- On the Airline domain (700+ trajectories), branched replies actually had a *lower* "missed intent" rate than the original simulator (25.27% vs 28.12%) — the divergence slightly *helps* alignment rather than hurting it

### 3.4 Snapshot-Based Branching and Execution

**What gets snapshotted:**
- Taken before each user turn, the snapshot holds the full conversation history, agent state, tool/environment state, and the original random seed

**How branching resumes:**
- Once a junction and divergent reply are chosen, reload the saved state from just before that turn and continue from there
- Only the modified user input is changed; everything else re-runs identically

**Why reuse helps:**
- No re-generating identical early turns → saves tokens
- Exact prefixes are preserved → enables **KV-cache reuse** (reusing saved computation) on compatible servers

### 3.5 Controlling Coverage and Cost

**The branch-number knob:**
- More branches = more coverage (more alternative paths explored) but more tokens
- The overhead of picking junctions and generating replies is tiny — **0.2% to 0.08%** of total evaluation cost

**Net effect:**
- Even after paying this overhead, DIVERT ends up using *fewer* total tokens than a full linear rollout — saving on both agent tokens and evaluation-side tokens

## 4. Experimental Setup

### 4.1 Benchmarks

**Where they tested:**
- The **τ-bench** collection, on three domains: **Airline**, **Retail**, and **Telecom**
- Each task defines an initial user intent, available tools, and a way to automatically judge success

**Why they reuse tasks instead of writing new ones:**
- Defining a new task is expensive — it needs aligned tools, policies, gold success criteria, and consistent data
- So benchmarks re-run the *same* tasks many times with varied agent and user behavior (exactly the setting DIVERT targets)

**Models used:**
- Two instruction-tuned models: OpenAI **GPT-OSS-120B** and Google **Gemini-2.5-Flash** (different providers, architectures, deployment)
- GPT-OSS-120B serves as the user simulator in most setups (good cost/performance tradeoff)
- All comparisons use identical decoding settings within each model

**Two metrics reported:**
- **Errors per 100K tokens (efficiency)** — how many failed trajectories you find per fixed computing budget; higher is better, and token counts are used because they're reproducible and provider-agnostic
- **Task Failure Count (coverage)** — how many *distinct* tasks show at least one failure; captures how broadly the method explores

## 5. Results

### 5.1 Errors Discovery Rate (Efficiency)

**The headline:**
- Branch-based evaluation is *consistently* more token-efficient than full rollouts, across all three domains and both models
- Even a **single branch** (8 rollouts + 1 branch) improves failures found per token

**Why it works:**
- Reusing shared prefixes and branching from mid-trajectory states reallocates computation toward *high-leverage* paths instead of redundant re-generation
- Efficiency gains grow steadily as you add more branches (Table 1)

### 5.2 Task Failure Count (Coverage)

**The headline:**
- More branches → more *distinct* tasks show failures, across all domains

**The key contrast (saturation effect):**
- Increasing rollouts *alone* gives diminishing returns in coverage
- Increasing the *branch* budget keeps unlocking *new* failing tasks

**What this means:**
- Many failure modes can't be found just by restarting — they need *directed branching* from the original conversations
- DIVERT both finds failures faster *and* widens coverage to more tasks → a more informative evaluation

### 5.3 Diversity Selection Validation

**Does the "pick the most different reply" step actually matter? Two checks:**

1. **Candidate level (4,500 candidates):** the "most dissimilar" candidate is substantially less similar to the original than the 2nd and 3rd candidates — in every domain
2. **Trajectory level (1,200 continuations):** continuing from a *more dissimilar* candidate produces a *more different* downstream conversation

**Takeaway:**
- The diversity isn't just cosmetic — a more divergent reply genuinely leads to a more divergent (and thus more informative) conversation

### 5.4 Ablation Study

**What each component contributes (built up step by step):**

- **Junction chooser alone (JC):** cheaper (fewer tokens), so errors-per-token goes up — but coverage of *new* tasks drops, because exploration is limited to continuing previously-seen prefixes
- **Adding directed user generation (DG):** big improvement in *both* efficiency and coverage, by actively steering toward unexplored behavior
- **Adding diverse response selection (DC):** extra gains by filtering out replies that came out too similar to the original

**Bottom line:**
- JC alone mainly saves money; actually *finding* failures requires the targeted, diverse user perturbations on top

## 6. Conclusions

**The motivating insight:**
- Not all conversation turns matter equally — greetings, logins, and static context barely affect later decisions, yet get regenerated every run, wasting tokens

**What DIVERT contributes:**
- Reuses shared prefixes and reallocates computation to branch at the turns that *actually* change behavior
- Gets broader coverage *without* raising overall cost

**Future directions:**
- Branch on tool outputs and environment dynamics too (not just user turns), enabling richer "what-if" evaluation
- Replace the LLM junction-chooser and cosine-similarity selection with alternative signals (e.g., perplexity-based junction selection, or diversity metrics beyond cosine similarity)