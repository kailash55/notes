# Chapter 1: Introduction to Building AI Applications with Foundation Models

#### The Big Idea: Scale

- Since 2020, the defining trend in AI is **scale** — models have grown massively in size and capability
- These large models (powering ChatGPT, Gemini, Midjourney, etc.) now consume a significant chunk of the world's electricity
- We're even running low on publicly available internet data to train them

#### Two Major Consequences of Scale

1. **AI is more powerful** — capable of more tasks, enabling more applications across industries
2. **Only a few orgs can build these models** — training requires huge data, compute, and talent

#### Model as a Service

- Because building models is so expensive, a few organizations build them and offer them to everyone else as a **service**
- This means anyone can now build AI-powered apps **without building a model from scratch**
- Result: demand for AI apps is up, barrier to entry is down → **AI engineering is booming**

#### AI Engineering vs Traditional ML Engineering

- AI in apps isn't new — recommendations, fraud detection, churn prediction all existed before LLMs
- But the new generation of large, readily available models brings:
  - **New possibilities** (things AI can now do)
  - **New challenges** (things that are harder or different now)

#### What This Chapter Covers

- Overview of **foundation models** — the key driver behind AI engineering's growth
- Successful **AI use cases** — what AI is good at and not yet good at
- The **new AI stack** — what changed, what stayed the same, and how the AI engineer role differs from a traditional ML engineer role

## The Rise of AI Engineering
### From Language Models to Large Language Models
#### Language Models

- A language model captures **statistical patterns in language** — essentially, how likely a word is to appear given some context
- This idea has been around for centuries (Sherlock Holmes used letter frequency to crack codes; Claude Shannon used it in WWII codebreaking)
- Today, a language model can work with **multiple languages**

**Tokens — the basic unit:**
- Text is broken into **tokens** (characters, words, or word-parts like `-tion`)
- This process is called **tokenization**
- ~100 tokens ≈ 75 words (for GPT-4)
- The full set of tokens a model knows is its **vocabulary** (GPT-4 has ~100,000 tokens)
- Tokens are used instead of words or characters because they:
  - Break words into meaningful parts (e.g., "cooking" → "cook" + "ing")
  - Keep vocabulary size small → more efficient
  - Handle unknown/made-up words gracefully

**Two types of language models:**

| Type | How it works | Used for |
|------|-------------|----------|
| **Masked** (e.g., BERT) | Predicts missing tokens using context from both sides | Sentiment analysis, classification, code debugging |
| **Autoregressive** (e.g., GPT-4) | Predicts the *next* token using only what came before | Text generation — far more popular today |

> In this book, "language model" always means autoregressive unless stated otherwise.

**Language models as completion machines:**
- Give a model a prompt → it completes it
- Example: `"To be or not to be"` → `", that is the question."`
- Outputs are **probabilistic** (predictions, not guaranteed facts) — this makes them powerful but also unreliable
- Completion is surprisingly versatile:
  - **Translation**: `"How are you in French is …"` → `"Comment ça va"`
  - **Classification**: `"Is this email spam? <email> Answer:"` → `"Likely spam"`
- Note: completion ≠ conversation — extra training ("post-training") is needed to make models respond the way users expect
