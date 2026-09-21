# Scroll: "Context as an Environment" — Programmatic Context Management for Long-Horizon Agents

> Source: Alibaba Group / Columbia University technical report (arXiv:2608.21690, Aug 2026)
> Local file: /Users/kd/.hermes/cache/documents/doc_dd3fbc90b13d_2608.21690.pdf

## Abstract

**What this paper is about:**
- Long-running AI agents (coding assistants, deep-research bots) build up a history far bigger than any single model context window
- Existing solutions compress or summarize the past before the agent knows what it will actually need later
- This paper presents **Scroll**, a "context manager" that stores history *outside* the prompt and lets the model reach into it with code

**The core idea:**
- Treat each agent session as a running **"Session Environment"** — a live, queryable thing, not a wall of text to stuff into the prompt
- The model writes short Python programs to search, pull up, and calculate over its own history, and only *prints* what it needs in the next step

**The main building blocks:**
- An append-only **Event Log** (the lossless, ground-truth record)
- A **sandboxed Python kernel** that survives across model calls, holding typed variables
- An **eviction index** so removed history is still findable by exact address

**The headline results (backbone: Qwen3.8-Max):**
- **94.8%** on LongMemEvalS; **73.1%** on BEAM10M (beats the best published memory system by 5.1 points); **86.7%** on LOCA256K (beats the best published long-horizon agent by 37.4 points)

**Bottom line:**
- Context management becomes a *coding* task — and models are already good at coding — while the raw history stays lossless and recoverable underneath

---

## 1. Why This Matters: The Context Problem

**The setup:**
- Agents work on long tasks ("trajectories" of model calls, tool runs, observations, failures, revisions)
- Every model call has a **bounded context window** — how much text it can "read" at once
- Worse: the amount a model can *reliably use* is far smaller than its claimed window, because reasoning degrades as input grows

**The two tactics everyone uses today:**
- **Compression** — truncate stale turns, throw away tool outputs, replace old parts with shorter summaries (used by Claude Code, Codex CLI, Cursor, etc.)
- **External memory** — pull out selected "facts"/"episodes" into a separate store and retrieve them later

**Why both are lossy in the same way:**
- The model only ever sees history *through* the summary or the memory store
- If the summary or store missed a detail, that detail is gone — even if the raw log still sits on disk
- Long tasks can suddenly need *exact* evidence or *computation over past events* (e.g. comparing two tool outputs made far apart). No summary can guarantee it kept what a future question will need, because the need isn't known until later.

**Scroll's answer:**
- Keep history **outside** the context, fully intact, and decide what to bring into the view *at query time* — via a program the model writes.

---

## 2. The Formal Idea

**Session state as three parts:**
- **`L` (event log)** — the sequence of events with metadata
- **`P` (payloads)** — the raw content behind events (e.g. a full tool result)
- **`V` (derived state)** — in Scroll, a variable namespace (in other systems, a "memory store" or summary buffer)

**The context-management problem, in one line:**
- At every step, choose the next *working view* `c_{t+1}` (≤ context budget) from the full state. Existing systems fix this map *before* needs are known (a summary operator that decides what survives as each segment is compacted). Scroll defers the choice to **query time** as a program the model writes.

**What this buys:**
- Nothing is thrown away, so nothing is *unrecoverable*
- The model decides what to recall, compute, and expose; the harness just makes those decisions *safe* (durable storage, stable addresses, sandboxed execution)

---

## 3. The Three Building Blocks

### 3.1 The Append-Only Event Log (holds `L`)
- A single, durable log spanning **all** sessions (not just the current one)
- Everything appends a typed event with role, session/agent IDs, timestamps, tool state
- Each event gets an **immutable, increasing `seq` (sequence) number** — its stable "address"
- Stored in **SQLite**; default search is **BM25** (keyword), *not* embeddings — because it's deterministic and needs no model call at index time

### 3.2 Durable Storage (holds `P`)
- A **payload** is the raw content an interaction produced (a full tool result, a generated artifact)
- Small payloads stay inline in SQLite; **large ones move to JSON/artifact files** on disk, leaving just a short preview + a recovery pointer in the log
- Accessed later through **lazy handles** (`ToolResultRef`, `ArtifactRef`)

### 3.3 Persistent Kernel & Resident Namespace (holds `V`)
- A **sandboxed Python kernel persists across model calls**, keeping a namespace of typed variables (values + lazy handles), each tagged with type, size, and where it came from (provenance)
- Tool calls issued programmatically return Python objects that later programs can *use* — not dead text
- Every model call gets a short **namespace digest** (name, type, shape of each variable; small scalars inline)
- **Fail-closed sandbox:** the Event Log is read-only from the kernel; DB/filesystem/network/tool access is limited to whatever the harness explicitly grants

---

## 4. How the Model Drives It: Four Operations

The interface is **"CodeAct"-style** — a controlled object `ms` with four moves (factorizing context construction):

| Operation | Call | What it does |
|---|---|---|
| **LOCATE** | `ms.search(query, k)` | Find candidate Event Log records via BM25 + filters; each hit carries its `seq` address |
| **MATERIALIZE** | `ms.expand(seq)` / `ms.expand(lo, hi)` | Pull back exact turns/ranges, load externalized payloads |
| **COMPUTE** | ordinary Python + permitted DB/filesystem/tools | Filter, join, aggregate, resolve updates, build derived state |
| **EXPOSE** | `print(value)` | Only what's printed becomes the next observation; everything else stays in the kernel |

**Key rule — `exec` vs `print`:**
- `exec` (running code) = how the environment is accessed/transformed
- `print` = which projection actually enters the model's working view
- So tool outputs can be huge, but only a few printed rows ever get *seen*

**Why this inherits coding skill for free:**
- Context construction becomes writing programs — no change to the harness needed — so it automatically improves as the underlying model's coding ability improves

---

## 5. Eviction: Keeping the View Bounded Without Losing History

**Triggers:** when the working view passes a budget (`ρC`), the procedure:
1. Persists live turns to the Event Log
2. Protects the active turn, recent tail, and newest tool results
3. Folds **completed tool payloads first** (a single `seq` pointer is enough to get them back)
4. Evicts the oldest completed spans if still over budget — *into an index*, not into oblivion

**The invariant:** everything removed stays *verbatim* in the Event Log, addressable by `seq`.

**Headlines as navigation anchors:**
- With each response, the model writes a short **headline** (task, verified state, next action, status), bound to the `seq` of that event
- Evicted spans leave their headlines in a **tiered index** ("what has left the view")
- Instead of growing linearly, it **rolls up** into tiers: each tier holds at most `k` blocks; when full, the newest block keeps detail and older ones collapse to one line each and move up
- Result: **fine anchors** for recent history, **coarse ranges** for distant history — size `O(k log_k n)`

**Why an index is needed on top of search:** keyword search only recovers what the agent *broadly recalls to ask for*. The index keeps the agent *aware of history it can no longer see*, so it can navigate directly to exactly the evicted region.

---

## 6. Evaluation

### 6.1 The three benchmarks
- **LongMemEval** — ask questions over a history of past user–assistant chats. Three increasingly-hard settings: **Oracle** (no distractor), **S** (~50 sessions / ~115K tokens), **M** (~500 sessions / ~1.5M tokens)
- **BEAM** — like LongMemEval, but longer, coherent histories (128K → 10M tokens), needing non-adjacent evidence, time tracking, deduplication, aggregation. **BEAM10M** can't fit into any model's context directly.
- **LOCA** — an agent that reasons, calls tools, and edits an environment as state grows; scales environment description from 8K to 256K tokens (they test the two largest: 128K and 256K)

### 6.2 Results vs. existing systems

**Long-term memory (Table 2):**

| System | LongMemEvalS | BEAM10M |
|---|---|---|
| Zep | 90.2 | – |
| Mem0 | 94.4 | 48.6 |
| Hindsight | 94.6 | 64.1 |
| Exabase M-1 | **96.4** | 68.0 |
| **Scroll** | 94.8 | **73.1** |

- On LongMemEvalS, Scroll is competitive with the strongest; on BEAM10M it **beats the best by 5.1 points**
- Existing systems follow a 3-stage pipeline: **ingest → retrieve → read** (a separate "reader" model).
  - Scroll instead ingests *raw history as-is*, writes retrieval code per question, and **needs no separate reader** — the writing agent also answers.

**Context-management strategies on LOCA (Table 3):**

| Agent loop | 128K | 256K | drop |
|---|---|---|---|
| Summarization (ReAct) | 86.7 | 65.3 | -21.4 |
| Retrieval (ReAct) | 88.0 | 66.7 | -21.3 |
| CodeAct | 89.3 | 85.3 | -4.0 |
| **Scroll** | **89.3** | **86.7** | **-2.6** |

- CodeAct and Scroll (which both bind intermediate results to environment objects, not raw text in context) hold up best, with the smallest drop as context grows
- vs. the best *published* LOCA results (different backbones): GPT-5.2-Medium + ReAct gets 38.7/21.3, MiniMax M3 + ReAct 49.3 at 256K — Scroll clears them by a huge margin

### 6.3 Can any backbone use Scroll? (Table 4)
- Re-run across six models (changing only the model): Qwen3.8-Max, Qwen3.7-Max, DeepSeek-v4-pro, GLM-5.2, Kimi-K2.7, and a small open-weight **Qwen3.6-35B-A3B**
- **Every model can use it; stronger models benefit more.**
- On LongMemEvalS the spread is tight — even the 35B hits 88.8 (vs 94.8 best)
- On LOCA 256K, where tasks need long trajectories + complex code, the gap widens to **64 points** (86.7 vs 22.7)
- Failure isn't interface-level: weaker models make more execution *errors* or quit early on aggregation-heavy tasks — suggesting room to distill frontier-model traces into smaller models

### 6.4 Ablations (BEAM10M)
- **Remove the original records (lossy summarize at ingestion)** → collapses to **19.9 overall**, near-zero wherever the answer must preserve *exact* values (info extraction, temporal reasoning, knowledge update). The most damaging
- **Remove REPL (expose `ms` as plain tool calls, no kernel)** → **-7.3 points**; hurts composing evidence from many records (knowledge update 92.5→82.5); single-lookup tasks unaffected
- **Remove the eviction index (keyword search only)** → **-1.8 points**; hurts detecting scattered evidence (preference following 89.1→74.9, summarization, event ordering)

### 6.5 Cost & efficiency
- Ingestion needs **no extra LLM calls**; records filtered in-kernel, only printed output enters context
- Median input on BEAM10M ≈ **105K tokens, ~1% of the corpus**; output tokens are an order of magnitude smaller than input. Reported in tokens (not $ or latency, which depend on serving).

---

## 7. Where Scroll Winds vs Loses (per-category, BEAM10M)
- **Strongest** where the answer hinges on a few exact records that must be ordered/reconciled: knowledge update (92.5), contradiction resolution (88.1), information extraction (75.0) — *punishes write-time compression*, because resolving a change needs *both sides* of a value's timeline in order
- **Weakest** where the graded artifact is itself a *condensed view* over many records: summarization (70.5 vs 91.9 Exabase), preference following (89.1 vs 97.5 Hindsight), temporal reasoning — because ingestion-heavy systems build those digests up front, while Scroll must rebuild per query
- **Multi-session reasoning is weak for everyone** (9.6–26.1); Scroll's misses come from *over-precise filters undercounting* evidence, not from unreachable records

---

## 8. Related Work (very short)
- Context compression vs. external memory (both choose what survives too early) — the prior art Scroll contrasts with
- "Code as the agent–environment interface" — CodeAct's lineage; idea of keeping tool results in sandbox variables and exposing only print projections
- Lossless session history / event-sourced logs — Scroll combines a queryable append-only log + external payload refs + an executable kernel, and adds a within-session eviction*index* as a navigation layer on top

---

## 9. Conclusion & Next Steps

**What they built:**
- Context management becomes an explicit model *policy* over a persistent Session Environment: `exec` to retrieve/compute, `print` to decide what enters context, the harness providing deterministic storage + execution + recovery

**What they plan next:**
- Distill the policy into smaller models — supervise two decisions via frontier-model traces: (1) when/how to write *retrieval* code, (2) which computed results to *print* back into the window (injection). The underlying context mechanisms stayed fixed.

**One-line takeaway for your own agent work:**
- Instead of summarizing history into the prompt (and permanently losing details), keep it in a queryable, addressable store and let the model pull exactly what it needs with code — the context view is a *projection*, never the *record*.