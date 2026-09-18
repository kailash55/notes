# ReacTOD: Bounded Neuro-Symbolic Agentic NLU for Zero-Shot Dialogue State Tracking

> Source: Proceedings of the 6th Workshop on Trustworthy NLP (TrustNLP 2026), pp. 342–352 (Amazon)

## Abstract

**What this paper builds:**
- ReacTOD is a new system that helps AI assistants (like booking chatbots) understand what a customer wants in a conversation — and be *reliable* about it

**The core problem it solves:**
- Chatbots that handle bookings and reservations must be predictable, but smaller AI models often make mistakes (wrong date, made-up info) that snowball into wrong actions

**The trick:**
- Instead of asking the model to do everything in one shot, ReacTOD splits the work into small tool calls inside a self-checking loop, with a rule-based "validator" that catches mistakes before they stick

**The headline results:**
- Better accuracy — up to 9.3 points higher than doing everything in one pass
- The validator catches and fixes 93.1% of the errors it spots
- New best score (state-of-the-art) on a standard test (MultiWOZ 2.1) — even a small 8-billion-parameter model beats the old best that used a much bigger model

## Introduction

**Why reliable understanding matters so much:**
- These systems handle real transactions (hotels, restaurants, transport), so a single wrong detail — like inferring a check-in date from the wrong sentence — causes silent failures or wrong bookings

**How the old way worked (and its limits):**
- Past systems used fixed models (like BERT) that classify intent and pull out details one step at a time — fast and predictable, but they need huge amounts of labeled data and must be retrained for every new scenario, so they can't handle brand-new situations (zero-shot)

**What newer AI-prompting approaches try, and where they fall down:**
- Newer methods let a large language model (LLM) answer from instructions alone (in-context learning), but a single pass is flaky — the model confidently "hallucinates" (invents) details to fill in gaps, which is dangerous when that value drives a real API call
- Real conversations are messy (referring back to earlier turns, "yes please" confirmations, switching topics), which needs multi-step thinking and dynamic memory

**Why unbounded "agent" systems aren't the answer either:**
- Fully open-ended agent loops with the biggest, most expensive models add too much delay and cost for production use

**The paper's core argument:**
- Making these systems reliable doesn't need *bigger* models — it needs *tighter control* over how the model reasons
- The key insight: most errors are small and local (a bad time format, a wrong slot name), not a fundamental misunderstanding — so they can be fixed locally instead of re-doing the whole conversation

**What ReacTOD does about it:**
- It breaks understanding into separate tool calls inside a bounded loop, and a rule-based validator checks every result before it's saved, giving the model clear error feedback to self-correct
- Smaller models (like Qwen3-8B) can then do the job without giant, expensive ones

**How it was tested:**
- Zero-shot (no training data, no examples) across five models from 8B params to frontier scale, on two benchmarks (MultiWOZ 2.1 and SGD)

**The three contributions:**
- A *bounded* agent loop that self-corrects (up to 9.3 points better than single-pass)
- A *deterministic validator* that gates every change and enforces three rules (valid actions, matching schema, consistent references)
- *Parameter-efficient* tracking that proves small models can beat bigger ones — needing only a machine-readable description of the domain

## Related Work

**Old pipelined NLU → generative tracking:**
- Early systems treated intent and slot-filling as fixed classification tasks; joint models unified them but were locked to predefined labels and couldn't generalize to new inputs
- Generative models (TRADE, SimpleTOD, SOLOIST) relaxed this, but still needed domain-specific retraining, leaving zero-shot open

**LLM prompting and knowledge distillation (and their gaps):**
- Instruction-tuned LLMs enabled zero-shot via in-context learning (D3ST, SERI-DST, FnCTOD), but single-pass generation stays flaky and prone to hallucination
- "Distillation" trains a smaller model on LLM output to cut cost, but it bakes the schema into the model's weights, killing zero-shot flexibility

**Tool-augmented agents and neuro-symbolic integration:**
- ReAct showed LLMs can interleave reasoning and actions, but open-ended agents often underperform structured methods in real task success despite sounding fluent
- Self-improvement alone tends toward "confirmation bias" without external grounding — which is exactly why this paper adds deterministic validation

## Methodology

### 3.1 — Problem Setup and High-Level Design

**How the system sees each turn:**
- Each turn is made of the user's words, the system's last action, the running "belief state" (what's known so far), and the current intent

**The core reformulation:**
- Instead of predicting everything at once, the model is limited to choosing *actions from a fixed tool library* — turning tracking from "generate a big answer" into "pick the right tool and let a checker verify it"
- Only validated results are allowed to update the belief state

**Two ideas that keep it cheap and focused:**
- **Incremental state:** the model only predicts *changes* (new or edited slots), not the whole state every time
- **Dynamic context:** instead of dumping the full schema and history each turn, it injects only what's needed — slot descriptions only for the active intent, and history only fetched on-demand when references need it; shorter prompts make small models follow instructions better

### 3.2 — Splitting the Work (Neuro-Symbolic Subdivision)

**Why split intent from slot-filling:**
- Prior work showed that breaking NLU into separate calls beats predicting everything together; it shrinks what the model has to reason about each step

**Intent classification (IC) — with a shortcut:**
- Maps the user's words to an intent, using the domain schema in-context so it works zero-shot
- It always includes a "fallback" (non-transactional) class, and if the intent is non-transactional, the system short-circuits and skips slot-filling entirely — so a simple "thanks!" doesn't trigger expensive detail extraction

**Slot resolution (SR):**
- For a transactional intent, it pulls out the relevant details; each slot gets both the *raw* text and a *normalized* value (e.g. "tmrw" → a proper date)
- It only injects slots for the active intent, and includes the system's last turn so it can handle "Yes" meaning "yes, the Hilton" (implicit acceptance)

### 3.3 — The Bounded ReAct Loop

**How the loop is controlled:**
- The agent runs a ReAct loop capped at Kmax iterations, following an instructed order: classify intent first, then extract slots

**What keeps it honest:**
- The order is encouraged by prompting, but *enforced* by the validator — slot extraction can't finish the loop unless the slots pass validation for the predicted intent
- If slots don't match the intent, it re-runs IC; if validation complains, it re-runs SR; if references are unclear, it fetches history — and if it hits the cap without success, it returns a safe fallback instead of guessing

### 3.4 — The Deterministic Validator (and Self-Correction)

**The safety rule:**
- The model proposes actions but never writes to state directly — a rule-based "validator" is the only gate, deciding Safe (pass) or Violated (with error feedback)

**Why rules, not another LLM-as-judge:**
- Symbolic checks avoid the infinite loop of "who checks the checker" and run in cheap O(1) time

**What the validator checks (three rule families):**
- **Action compliance** — rejects unknown tools, enforces ordering (IC before SR), blocks duplicate calls
- **Schema conformance** — checks intent/slot names against the real ontology, and checks value formats (regex for dates/times/numbers, membership in allowed lists)
- **Coreference consistency** — flags vague references like "restaurant" that mean an unresolved entity needs history retrieval

**How a failure drives self-correction:**
- On a violation, it injects a specific error message (e.g. "invalid format for slot taxi-arriveby: expected HH:MM") so the model re-examines and fixes just that piece
- A hard cap prevents infinite loops; hitting it ends the turn safely

**Why state is updated "deferred" and "upsert-only":**
- The belief state is never touched during reasoning — updates only land after validation passes, so rejected attempts can't poison the record
- Updates only add or overwrite (never delete — removals use an explicit null), so info is preserved even when the conversation hops between domains

## Experiment

**The two benchmarks:**
- **MultiWOZ 2.1** — 1,000 test dialogues across five domains with cross-domain references
- **Schema-Guided Dialogue (SGD)** — 4,201 test dialogues spanning 26 services and 16 domains, mirroring real API-driven systems

**How success is measured:**
- **Joint Goal Accuracy (JGA)** — whether every active slot matches exactly at once; reported overall and per-domain (or per-service for SGD), with fuzzy matching for non-categorical values

**Which models were tested:**
- Mainly open-source models under 32B params: Qwen3-8B, Qwen3-32B, gpt-oss-20B, Gemma3-12B, plus Claude-Opus-4.6 as a ceiling reference
- All run at temperature 0.0 (deterministic) with a max of 6 loop iterations

**How the schema was prepared:**
- MultiWOZ schema taken from v2.2, merging intents per domain and tagging each slot's type (categorical, time, number, or free text) for the validator
- SGD schema derived programmatically from official definitions, with required/filter roles and informational slots dropped to reduce hallucination

**What it's compared against:**
- SERI-DST and FnCTOD (the old zero-shot best), plus a re-run of FnCTOD on the same models for fairness, and DistDST (needs training, reference only); on SGD it compares to SRP (self-refined prompts)

## Results

### 5.1 — Zero-Shot Performance

**ReacTOD wins across the board:**
- Even the smallest model (Qwen3-8B) at 47.34% overall JGA beats FnCTOD with GPT-4 (38.71%), despite being far smaller
- Re-running FnCTOD on the same Qwen3-32B only got 40.36%, while ReacTOD's *smaller* Qwen3-8B beat it — proving the gains come from architecture, not model size

**Best numbers:**
- MultiWOZ: gpt-oss-20B at 52.71% overall / 71.77% domain average (beats even the fine-tuned DistDST baseline)
- SGD: Claude-Opus-4.6 at 80.68% average service JGA, beating the reproduced SRP baseline (45.20%) under *harder* conditions (predicting domains end-to-end vs. using gold labels)

**What the cross-model comparisons reveal:**
- Reasoning capability matters more than raw parameter count — a 20B model with native thinking matches a 32B model, and a smaller Qwen3-8B overtook the larger Gemma3-12B on both benchmarks

### 5.2 — Does the Loop Actually Help? (Ablation)

**How they isolated the loop's value:**
- They compared the full ReacTOD against a "decomposed" version that runs IC and SR as two independent calls with no iterative loop

**The loop is a big deal:**
- The ReAct loop adds up to 9.32 points (gpt-oss-20B on MultiWOZ) and 11.82 points (Qwen3-8B on SGD), helping every model
- Biggest gains go to models with native reasoning (gpt-oss-20B) and to smaller models where errors were more common to begin with

### 5.3 — Is It Efficient, and Does the Validator Do Its Job?

**Cost is acceptable:**
- Median turn uses exactly 2 LLM calls (the required IC + SR), with P99 of 3–6 calls — the loop stays bounded even in the worst tail
- Token use tracks reasoning style: compact text-based models use ~150 tokens/turn, while gpt-oss-20B uses ~3× more because it "thinks out loud" before each tool call

**The validator is genuinely active:**
- On Qwen3-8B, 9.3% of turns triggered at least one correction, mostly action-compliance (calling SR before IC), then schema errors (bad values, hallucinated slot names), then coreference issues
- 93.1% of these self-corrected after feedback (only 47 turns hit the loop ceiling)

**Proof the validator itself matters:**
- Turning the validator *off* while keeping the loop made Qwen3-8B drop from 47.34% to 43.00% (−4.34 pp) — so the gains come from structured feedback, not just "more tries"

## Limitations

**It still costs extra LLM calls:**
- The loop adds multiple inference requests per turn, which can strain cost-sensitive or high-throughput systems

**It still needs a schema:**
- Zero-shot about *training data*, but it still requires a configured domain schema (intents, slots, types); for MultiWOZ this meant manual slot-type annotation — a real setup cost, and performance can drop if the schema is incomplete or noisy

## Conclusion

**What ReacTOD proves overall:**
- A new zero-shot best: gpt-oss-20B at 52.71% on MultiWOZ (beating FnCTOD with GPT-4 by 14 points), Qwen3-8B at 47.34%, and 80.68% on SGD
- The agentic loop is the key — up to 9.3 pp (MultiWOZ) and 11.82 pp (SGD) over single-pass, yet stays cheap (median 2 LLM calls/turn, 150–207 tokens)

**The validator's value:**
- Catches and fixes common errors before they reach state, with 93.1% self-correction and a fully inspectable trace every turn

**The takeaway:**
- Reliable zero-shot understanding doesn't need frontier-scale models — *structured reasoning + deterministic safeguards* beat raw parameter count

## Future Work

**More ablation experiments:**
- Compare lazy vs. full schema injection and always-on vs. on-demand history retrieval across model sizes, especially on SGD's heavy 26-service schema

**A fuller dialogue manager:**
- Add tools beyond IC and SR — e.g. "wait/clarification" for when the system needs more info, and "repeat/confirm" to re-ask or verify ambiguous values — moving ReacTOD toward a complete agentic dialogue manager