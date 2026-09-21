# Context-Driven Incremental Compression for Multi-Turn Dialogue Generation

> Source: https://arxiv.org/html/2606.12411v1

## Abstract

**Summary: Chatbots get slower and dumber as conversations get longer, because every new message forces the model to re-read the entire history. This paper proposes C-DIC, a way to keep a compact, editable memory of a conversation instead of re-reading everything each turn.**

- Chatbots today re-process the full conversation history at every turn, which wastes computation and gets worse the longer the chat goes.
- Naive shortcuts — chopping off old messages or summarizing them — lose important detail and don't track the conversation well.
- The authors introduce **C-DIC** (Context-Driven Incremental Compression), which treats a conversation as separate topic "threads" and stores a small, revisable memory per thread.
- At each turn, a lightweight *retrieve → revise → write-back* loop updates the memory and shares information between turns.
- They also adapt **truncated backpropagation-through-time (TBPTT)** — a way of training only on recent steps rather than the whole history — to their multi-turn setting.
- Results show C-DIC keeps stable speed and quality over hundreds of turns, beating truncation, summarization, and one-shot compression.

## 1. Introduction

### The problem with long conversations

**Summary: Multi-turn chats are powerful, but feeding the whole history every turn is both expensive (attention cost grows with length) and error-prone (the model "loses the thread").**

- ChatGPT and Gemini are popular because they handle back-and-forth conversation, where topics drift, earlier points get referenced, and ideas get refined.
- The current common approach is to paste the *entire* history into every turn, which creates two problems:
  1. **Inefficiency** — attention cost scales quadratically with input length, so long chats get very expensive.
  2. **Drift** — the model "loses the thread" and gives irrelevant answers, especially for details buried hundreds of turns back.

**Summary: Existing fixes are either too aggressive or too rigid — they lose long-range context or can't adapt to changes mid-conversation.**

- **Truncation** (keeping only recent turns) throws away important earlier details.
- **Summarization** produces losses, generic, query-blind summaries that can't be revised mid-conversation.
- **Static latent compression** (turning a document into a fixed set of learned vectors) works once, but collapses when applied turn after turn because it never shares or revises memory between turns.

**Summary: The authors' idea is to make compression topic-aware and progressive, so the model only pulls up the context that matters for the current topic.**

- The model should retrieve and reason over context that matches the current topic, wherever it sits in the history.
- Without topic-awareness, even compressed input can miss key context.

**Summary: C-DIC models a conversation as interleaved topic "threads" and keeps a compact memory of revisable per-thread compressed states. It solves three challenges: topic-aligned retrieval, incremental revision, and efficient memory management.**

- Three challenges it tackles:
  1. **Topic-aligned retrieval under drift** — find the right memory as the subject changes.
  2. **Incremental revision** — update memory without re-encoding the full history.
  3. **Efficient inference-time memory management** — keep it cheap.

**Summary: At each turn, C-DIC runs a light retrieve-revise-write-back loop with three parts: retrieve the relevant thread memory, compress the current turn incrementally, and update memory without gradients.**

- **Thread-aware memory retrieval** — pull up only the compressed history relevant to the active topic.
- **Incremental compression** — compress the new turn together with its thread states, so later turns can reuse it without re-encoding everything.
- **Gradient-free memory update** — memory is updated at inference time without backpropagation, so it stays cheap and light.

**Summary: Training mirrors the loop via a retrieval-aware truncated BPTT, avoiding full-history backpropagation and the error accumulation of one-shot compressors. The whole thing runs on a 7B backbone and shows stable latency and perplexity over hundreds of turns.**

### Contributions

**Summary: Three headline claims.**

- **C-DIC** is (to the authors' knowledge) the first framework for turn-level incremental compression inside a single compact dialogue memory, using a *retrieve → revise → write-back* scheme.
- It gets long-range coherence and reference fidelity, while cutting inference cost and input size, beating truncation, summarization, and static compression baselines.
- Empirically it stays stable (latency and perplexity flat) over hundreds of turns.

## 2. Preliminaries & Related Work

### 2.1 Multi-Turn Dialogue Generation

**Summary: Multi-turn dialogue is what lets agents track references, preferences, and assumptions over time. But formally, the full history must be supplied at every future turn, and the cost grows cubic-ally with conversation length.**

- Prior context is needed to resolve cross-references (who/what "it" refers to), track preferences, and revise assumptions.
- If each exchange averages some tokens, the prompt length at turn t grows linearly, and the cumulative attention cost across a t-turn dialogue scales **cubically** — this quickly dominates latency and memory.

### 2.2 Textual Context Management

**Summary: Common text-based shortcuts all fall short: truncation drops early info, summarization goes stale, and retrieval/prompt-compression don't learn a shared latent state.**

- **Truncation** keeps only recent utterances, risking loss of earlier needed info.
- **Summarization** compresses history into a natural-language summary, which may omit detail or become outdated.
- **Retrieval & long-context methods** selectively reuse relevant segments; **prompt compression** shortens the text before generation.
- These all manage text, rather than learning a single latent dialogue state that is retrieved, revised, and written back across turns.

### 2.3 Latent Context Compression

**Summary: Latent compression maps variable-length context to a fixed set of learned vectors, giving the generator a fixed-size interface. But existing compressors are one-shot and static — they don't revise across turns.**

- Common setup: append trainable "compression tokens" to input, run the model once, keep hidden state at those positions as a dense matrix.
- **AutoCompressor** recursively accumulates compression embeddings over chunks.
- **ICAE** (the method C-DIC builds on) maps context to a fixed-size latent matrix consumed by a frozen generator.
- In standard one-shot mode, this matrix is made once and stays static; reflecting new info requires recompressing everything, and fixed capacity risks forgetting as dialogue grows.

## 3. Methodology

**Summary: C-DIC treats a dialogue as interleaved topic threads and keeps a compact memory of revisable per-thread states. It freezes the generator and only trains the compressor plus learnable compression tokens.**

**At each turn it: (i) retrieves a small relevant subset, (ii) generates with a frozen decoder, (iii) compresses the new turn and writes it back into a memory slot — all gradient-free at inference.**

### 3.1 Compressor Initialization

**Summary: The compressor is initialized from a pretrained ICAE checkpoint (trained for one-shot document compression) rather than trained from scratch, so it starts with strong compression ability for free.**

### 3.2 Incremental Compression & Context-Aware Retrieval

**Summary (base case): Without retrieval, a turn's query + gold response is compressed into a small state written to memory; the generator stays frozen, so learning focuses purely on producing useful compressed context.**

**Summary (adding retrieval): C-DIC scores each memory slot by semantic similarity with mild recency decay, then conditions generation only on the retrieved relevant subset rather than all history. This keeps per-turn cost proportional to the retrieved set, not dialogue length.**

**Summary (write-back): Memory is updated with a deterministic, gradient-free rule — insert a new slot on topic shift, or revise the best-matching slot on continuation. This preserves "thread continuity" without backpropagating through selection.**

### Retrieval-aware truncated BPTT

**Summary: Instead of full backprop through every past turn (expensive) or fixed-window truncation (blind to what was actually used), C-DIC backpropagates credit only along the specific memory-update path that was actually selected.**

- Full **BPTT** backprops through all turns; costly.
- Fixed-window **TBPTT** truncates to a window, ignoring which turns were really consulted.
- C-DIC's retrieval-aware version assigns credit only along the selected memory-update chain (a one-hop reverse pass), so non-selected states act as stop-gradient context and off-topic credit never flows into mismatched memory.

## 4. Experiments

### 4.1 Datasets

**Summary: The paper evaluates on two multi-session chat corpora plus targeted diagnostics.**

- **MSC** — human-human chats across up to 5 sessions; ~53 utterances per episode training, ~66 at eval.
- **REALTALK** — real WhatsApp-style corpus, 10 conversations over 21 days, ~894 utterances each; evaluated zero-shot (never trained on it) in two modes: *all-sessions* (full cross-session history) and *per-session*.
- **MSC-QA** — QA pairs generated with GPT-4o whose answers depend on earlier mentions, as a targeted test of long-range context.
- **LongMemEval** — external long-context QA benchmark.

### 4.2 Baselines

**Summary: A broad comparison against raw prompting, textual retrieval, prompt-compression, and latent-compression methods, all using the same frozen Llama-2-Chat-7B generator for fairness.**

- **Raw prompting**: full history, plus a stronger-backbone variant (Llama3.1-8B) and truncation.
- **Textual**: recursive summarization, RAG@K, RAG-threshold, LLMLingua, InfLLM.
- **Latent**: AutoCompressor, and three ICAE variants (incremental, one-shot, append).

### 4.3 Evaluation

**Summary: Quality measured with standard metrics: perplexity (fluency), BLEU and ROUGE (match to human responses), plus GPT-4o-judged accuracy on MSC-QA.**

### 4.4 Results

**Summary: C-DIC wins across the board on MSC and REALTALK while using far less context — and the naive incremental ICAE baseline collapses catastrophically.**

** big picture from the results table:**

- **C-DIC (Ours)** achieves by the lowest perplexity (8.431 on MSC, 9.789 on REALTALK) and highest BLEU/ROUGE.
- **Naive incremental ICAE** blows up to PPL ~514 (MSC) / ~124 (REALTALK) — a structural mismatch between its one-shot training and repeated compression.
- C-DIC achieves the best PPL/BLEU/ROUGE while conditioning on a small retrieved latent set instead of the full history.

### 4.5 Long-Range Context Diagnostics

**Summary: On MSC-QA (questions that require remembering earlier turns), C-DIC beats all baselines on accuracy, confirming its gains come from genuinely using long-range context, not just lexical overlap.**

### 4.6 Latency Comparison

**Summary: C-DIC is the only method that handles 428 turns under the same hardware — full-prompting and ICAE variants run out of memory (OOM) far earlier — while keeping latency roughly flat (~33.5s) as history grows.**

### 4.7 Ablations

**Summary: Dropping each component hurts, but removing incremental compression (IC) hurts most, confirming every piece matters.**

**From the ablation table:**

- **Removing IC**: PPL jumps from 9.356 to 25.527, ROUGE-2 drops from 0.056 to 0.018 — the biggest degradation.
- **Removing retrieval-aware TBPTT**: PPL rises to 12.295 — weaker supervision.
- **Removing memory threading**: slightly better PPL (9.197) but notably worse BLEU/recall — threading is what drives long-range coherence.

## 5. Limitations

**Summary: Four honest limits.**

- Memory slots can grow if topics keep shifting (though generation uses only a small retrieved subset).
- Results rely on a pretrained compressor, so they may vary across initializations or backbones.
- Compressed latent memory can retain sensitive info and propagate stale/incorrect content — it is not a privacy or deletion mechanism.
- Evaluation is limited to a few benchmarks; broader domains remain future work.

## 6. Conclusion

**Summary: C-DIC replaces full-context prompting with a thread-aware memory updated via retrieve-revise-write-back, trained with retrieval-aware TBPTT. It stays stable where static compressors collapse and is the only method shown to reach 428 turns.**

## Why it matters (takeaways)

- The key idea is **persistent, revisable memory**: compressed states are updated in place instead of being discarded or recreated each turn.
- **Retrieval-aware training** matters — you only backprop through the memory you actually used.
- The problem is efficiency + coherence, solved together: a fixed small retrieved context keeps both latency flat and the model focused on the current topic.