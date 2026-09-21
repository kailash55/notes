# Prompt Caching Doesn't Cache Prompts

> Source: https://karanbansal.in/blog/prompt-caching/

## TL;DR

**Summary:** Prompt caching stores the model's internal working state, not your text — and it only matches from the very start of the prompt, so the first change kills everything after it.

- Turn on prompt caching and repeated input costs drop up to 90%, and the model starts responding faster too.
- A 30-step agent workflow can go from $4.50 to $0.62 just by caching.
- Despite the name, it doesn't cache your prompt or your tokens — it caches the model's working state (the **KV cache**).
- The cache only matches from position 0; any divergence invalidates everything right of the mismatch.
- Rule of thumb: put stable content (tools, system prompt) at the front, dynamic content (timestamps, new messages) at the back.

## What Actually Happens When You Call an LLM

**Summary:** Every LLM call runs in phases — expensive setup first, then cheap generation — and the expensive setup is exactly what caching avoids.

- **Tokenization** chops your text into integer IDs (e.g. "Hello world" becomes `[9906, 1917]`). Same input always gives same output.
- **Prefill** is the expensive phase: the model reads every token and builds an internal working state — a representation of what each token means *in the context of every token before it*. This is where the GPU does most of the work.
- **Decode** is the cheap phase: with the working state ready, the model generates output tokens one at a time.
- The working state in the middle is the **KV cache** — the expensive thing prefill produces, and exactly what prompt caching stores and reloads.

**How you know it's working:** the API usage stats reveal it.

- `cache_read_input_tokens: 98432` means the provider loaded a stored KV cache instead of rerunning prefill — you skipped the expensive part.
- `cache_creation_input_tokens` shows when a fresh cache was written; `input_tokens` shows only the new, uncached part.

## Why the Cache Is the Whole Game

**Summary:** Without caching every call rebuilds the entire working state, even when 99% of the prompt is identical to the last call; with caching you only pay for prefill once.

- A 50K-token prompt with no caching = full prefill on every request, even if 49,999 tokens are identical to last call.
- With caching: the first request stores the KV cache; later requests load it instead of recomputing it, so long as the prompt *starts* with the same tokens in the same order.
- Agents resend the same system prompt, tools, and history every turn — so caching means you pay for prefill once, then just generate.
- The state is big: on Llama 3 70B each cached token is ~320 KB — a 4K-token system prompt is ~1.25 GB, a 128K context window ~40 GB. Recomputing that each call is what makes uncached agents painful.

## Why Prefix Matching Must Be Exact

**Summary:** The working state at any position depends on every token before it, so one changed token upstream invalidates everything downstream.

- The state at position `t` encodes the full context of all prior tokens, not token `t` alone — change token 5 and everything from position 5 onward was computed against a prefix that no longer exists.
- Small things break the match: a trailing space (which becomes an extra token), a reordered JSON key, a timestamp that ticked forward.
- Prompt order therefore directly controls cache hit rate: tools → system prompt → context/docs → history → new message (most stable → least stable).
- Everything left of the first mismatch is cached; everything right gets recomputed. Push stable content to the front.

## How Providers Implement This

**Summary:** All three major providers use the same prefix-matched KV cache idea, but with different API surfaces and pricing.

### Anthropic

- **Automatic caching** (Feb 2026) needs a single `cache_control` field at the top of the request; it advances its breakpoint forward as the conversation grows.
- **Explicit breakpoints** (max 4) let you layer caches for multi-tenant agents — tools cached for all requests, context per tenant, conversation per session.
- Minimum cacheable prompt keeps falling by model: 4,096 tokens (Opus 4.5/4.6), down to 512 (Fable 5 / Mythos 5).

### OpenAI

- Caches automatically once a prompt passes its minimum size — no code changes, no write surcharge.
- On GPT-5.5+ the cache lasts 24 hours by default (older 5–10 min `in_memory` cache isn't supported there).
- An optional `prompt_cache_key` routes same-prefix requests to the same machines, where the cache actually lives.

### Google Gemini

- **Implicit caching** is on by default for 2.5+ models, no setup needed.
- **Explicit caching** uses named cache objects with custom TTLs and a storage fee that scales with size/duration (lives on the older `generateContent` API; the newer Interactions API is implicit-only).

## What This Means for Agents

**Summary:** For tool-calling agents doing 20–50 round trips per task, cache behavior dominates cost and latency — these patterns keep the prefix intact.

- **Never change your tool set mid-conversation.** Tools sit at the front of the prefix; add/remove one and you invalidate the system prompt and all history. Keep all tools in every request; control behavior via system messages.
- **Put dynamic content in messages, not the system prompt.** Timestamps, mode flags, and session IDs in the system prompt invalidate everything after them. Keep the system prompt frozen.
- **Use deterministic serialization.** Unstable JSON key order produces different tokens and cache misses — use `sort_keys=True`.
- **Keep history append-only.** Editing or truncating an earlier message kills the cache from that point. Manage context by summarizing from the beginning, then cache fresh.
- **Warm the cache before going parallel.** A cache is only available after the first response begins — firing 10 concurrent requests means 9 miss. Send one, wait for the first token, then fan out.
- **Use the 1-hour TTL for slow agents.** Anthropic's default is 5 minutes; agents that pause (human approval, slow tools) expire that. The 1-hour TTL costs 2x to write but avoids full recomputation.

## Silent Cache Killers

**Summary:** Some things break the cache with no error or warning.

- Toggling image attachments on or off between requests.
- Changing `tool_choice` between turns.
- Enabling/disabling extended thinking.
- Non-deterministic key ordering in `tool_use` content blocks (common in Swift and Go).
- Switching workspaces — Anthropic isolates caches per workspace (as of Feb 2026).
- Upgrading model versions — a tokenizer change alters token sequences and silently breaks old caches; re-warm after a version bump.

## Monitoring

**Summary:** The same usage fields tell you whether your cache is actually working.

- Go by `cache_read_input_tokens` — it should be high after turn 1.
- `cache_creation_input_tokens` should be 0 after turn 1.
- If both read and creation stay at 0, something is breaking the prefix: check timestamps in the system prompt, non-deterministic serialization, tool-set changes, or gaps beyond the TTL.

## The Economics

**Summary:** Caching is the difference between an agent costing 5–10x more and being economically viable.

- Example: 30 round trips, 50K-token prefix, Claude Sonnet 4.6 at $3/MTok input and $0.30/MTok cache read.
- Un-cached: $0.15 per call → $4.50 for 30 trips. Cached: $0.015 per call → $0.62.
- That's an 86% blended saving (not the full 90%) because the first call pays the 1.25x write surcharge; every hit after is 90% off.
- The cache is the load-bearing wall for agents, not a tuning knob to visit later.

## Cache Optimization Checklist

- **Prompt structure:** stable content first — tools → system prompt → context → history → new message; no timestamps/session IDs/mode flags in the system prompt.
- **Serialization:** `sort_keys=True` on all tool outputs; stable key ordering in `tool_use` blocks.
- **Tool management:** same tool set every request; behavior changes via system messages, not tool-set swaps.
- **Conversation:** append-only history; compress context from the start, not the middle.
- **Cache hygiene:** warm cache before parallel; 1-hour TTL if steps exceed 5 minutes; no toggling images/extended thinking; consistent `tool_choice` and workspace; pin the model version.
- **Monitoring:** high `cache_read_input_tokens` and 0 `cache_creation_input_tokens` after turn 1.

> The *why* behind these rules (transformer layers, attention math, prefill vs decode internals) is promised in a follow-up post by the author.