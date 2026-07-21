# From GPT-3 to GPT-5: Mapping their capabilities, scope, limitations, and consequences

## Abstract

**What this paper is about:**
- Tracks how OpenAI's GPT models changed from GPT-3 all the way to GPT-5
- It's not just a history lesson — it compares them and asks "how did they actually change?"

**What they looked at:**
- Official docs, release notes, API docs, product announcements, and research papers

**The big idea:**
- GPT models didn't just get "bigger and smarter" — they became something fundamentally different
- GPT-3 was basically a text predictor; GPT-5 is more like a full system that can use tools, understand images, follow safety rules, and fit into real workflows

**5 things they tracked across generations:**
- Technical changes (how it's built)
- What it can do (capabilities)
- How it's deployed (cloud, APIs, products)
- What still doesn't work
- How it affected the real world

**What STILL doesn't work, even in GPT-5:**
- Makes stuff up (hallucinations)
- Sensitive to how you phrase prompts
- Behaves unevenly for different people/topics
- OpenAI still doesn't share full details about how models are built

**Bottom line:**
- Going from GPT-3 to GPT-5 isn't just "model got better"
- It's a rethinking of what AI even *is* — how we test it, how we use it, and who's responsible when it goes wrong

## Introduction

**Why GPT models matter — 3 big reasons:**
- They defined how AI models are sold and accessed today (APIs, tiers, safety reports, paid services)
- They changed how people *talk* to AI — from "complete my text" to full conversations with images and tools
- They've rippled into many industries: software, education, media, content creation, and policy debates

**How each generation moved the needle:**
- GPT-3 — proved that big models can learn from just a few examples
- GPT-3.5 / ChatGPT — reset expectations; people now expected to *chat* with AI
- GPT-4 — raised the bar on general intelligence and performance
- GPT-4o — added real-time voice and image understanding
- GPT-4.1 — focused on developers: long documents, coding, tool use
- GPT-5 — introduced smarter routing and workflow integration

**Why reviewing this family is tricky:**
- GPT-3 and GPT-4 have proper research papers; newer ones just have release notes and blog posts
- Newer models are bundled with tools, products, and routing logic — so "the model" and "the product" are hard to separate
- OpenAI doesn't always share full details, and the authors treat those gaps as intentional governance choices

**What this paper does (and doesn't do):**
- Treats GPT models as an evolving *system*, not a list of isolated releases
- Looks at: technical profile, what each can do, how users interact, known limits, and real-world consequences
- Does NOT try to reverse-engineer undisclosed training details from rumors

**What the paper covers (section by section):**
- Methodology → Terminology → History → Technical profiles → Capability comparison
- Interaction changes → Failure modes → Safety & governance → Economic impact
- Societal consequences → Cross-generation changes → Future challenges → Conclusion

## 2 Scope, Sources, and Methodology

**Where they got their information (5 source types):**
- Official research papers (e.g. GPT-3 and GPT-4 papers)
- System cards (safety-focused docs OpenAI publishes with newer models)
- API and model documentation (context limits, token limits, use cases)
- Product announcements and release notes (blog posts, launch pages)
- Academic research papers that study or cite GPT models

**The documentation problem:**
- Older models (GPT-3, GPT-4) have proper research papers
- Newer models (GPT-4o, GPT-5) mostly have safety cards and model pages — less technical depth
- GPT-3.5 Turbo, GPT-4 Turbo, GPT-4.1 only have model pages with basic specs

**How they used research papers:**
- Some papers directly test GPT on specific tasks (e.g. eye care, science experiments)
- Others are used to explain big-picture concepts like alignment or foundation models

**User reviews / comparisons:**
- They looked at these but didn't rely on them heavily
- Used mainly to cross-check other sources, not as standalone evidence

**What this paper is NOT:**
- Not trying to crown one model as the "best"
- Not giving a single score or ranking

**What they DO compare across each model (7 dimensions):**
1. How you interact with it (chat, tools, voice, etc.)
2. What formats it handles (text, images, audio)
3. How much it can read and output at once (context/token limits)
4. Whether it can use tools or reason step-by-step
5. What it's reported to be good at
6. What it's reported to struggle with
7. Its impact on society

## 3 Terminology and Taxonomy

**Why this section exists:**
- GPT models span research papers, API products, and consumer apps — so terms need to be pinned down clearly

**Key terms explained simply:**

- **GPT model family** — all OpenAI models from GPT-3 onward built on the same "generative pre-trained transformer" foundation, including chat, image, and routed variants

- **LLM (Large Language Model)** — a big model trained on text to generate or predict text; still applies to GPT even after it gained image/audio abilities, because text generation is still central

- **Foundation model** — a general-purpose model trained on massive data that can be adapted to many tasks; GPT models fit this

- **Frontier model** — just means the most powerful/capable model publicly available right now; OpenAI calls GPT-5.4 a frontier model built for demanding professional tasks

- **Multimodal model** — can handle more than one type of input/output (text, images, audio, video); GPT-4 introduced image input, GPT-4o handles text + images + audio all at once

- **Chat model** — designed for back-and-forth conversation and following instructions, not just completing a sentence; GPT-3.5 Turbo made this mainstream

- **Reasoning model** — a model that can control how much "thinking effort" it puts in before answering; GPT-5.4 lets developers tune this via the API; the paper uses this term for the feature, not to claim AI truly thinks

- **Agentic workflow / tool-using system** — an AI that can call external tools, handle long tasks, and work through multi-step problems with minimal human guidance; GPT-4.1 and GPT-5.4 are positioned for this

## 4 Historical Evolution of the GPT Family

### 4.1 GPT-3: Scaling and Few-Shot Learning

- GPT-3 proved a key idea: just making a model *really big* made it good at many tasks — without needing task-specific training
- It was introduced by Brown et al. (2020) as a **175 billion parameter** autoregressive language model
- It was trained on a massive amount of text data (pre-training), and then used at deployment time via **in-context prompting** — no extra training needed
- It performed competently at translation, question answering, and many other language tasks
- Its context window was **2,048 tokens** — meaning it could only "read" that much text at once
- GPT-3 became the symbol of a research philosophy: **one general model → many tasks**, with very few or no gradient updates (no fine-tuning) at deployment
- It was **not designed to be a chatbot** — it was a text-completion engine, not a conversational assistant
- The user had to do all the heavy lifting: write the right prompt, provide examples, and format things carefully to get good outputs
- **Capability and usability were separate** — it could do things, but making it do them reliably required significant effort from the user
- GPT-3 made **in-context learning** (learning from examples in the prompt) seem practical and scalable
- It also made **prompt engineering** a real and necessary skill — how you asked mattered as much as what you asked
- In hindsight, GPT-3 should be seen not as a finished assistant, but as a **scalable text engine** — its few-shot capabilities reshaped the direction of AI research for every generation that followed

### 4.2 GPT-3.5: Chat Optimization and Instruction Following

- GPT-3.5 is the moment where public-facing AI shifted from **text completion engines** to **chat assistants**
- This was made possible by alignment research — specifically **InstructGPT**, which fine-tuned models using **human feedback (RLHF)**
- InstructGPT showed that models trained this way were better at:
  - Understanding what the user actually wants
  - Generating more factual responses
  - Reducing made-up or untruthful text — compared to raw GPT-3-style prompting
- **GPT-3.5 Turbo** brought this to the public via an API focused on chat, while also remaining strong at coding and non-chat tasks
- The most important change from GPT-3 wasn't just technical improvement — it was a **shift in how users interact with the model**:
  - GPT-3: "show the model a pattern and it continues it"
  - GPT-3.5: "just tell the assistant what you want"
- This was **socially significant** — it opened up AI to people who had no idea how to write prompts or understand model internals
- However, GPT-3.5 is **less transparent** to researchers than GPT-3:
  - No single flagship technical paper exists for it
  - Documentation is scattered across alignment research papers, product blog posts, and model pages

### 4.3 GPT-4: Stronger Reasoning and Multimodal Input

- GPT-4 set a new public benchmark — both in raw performance and in how capabilities were reported
- Announced as a **large multimodal model**: it could take **text and images as input** and produce text output
- The technical report highlighted:
  - **Predictable scaling** — performance improved in a foreseeable way as the model grew
  - **Widespread benchmark improvements** across many tasks
  - **Better instruction-following** after its post-training (fine-tuning) phase
- GPT-4 **outperformed GPT-3.5 on many professional and academic tests**, including scoring in the **top 10% on a simulated bar exam**
- It set a new standard for **how AI capabilities are publicly reported**:
  - Benchmark comparisons were shared openly
  - Safety disclosures were included
  - Limitations were stated outright alongside strengths
- Known weaknesses openly acknowledged in the report:
  - Still **hallucinated** (made things up)
  - Had a **short context window** (couldn't handle very long documents)
  - Could **not learn or update from experience** after deployment
- This combination of honest strengths-and-weaknesses reporting influenced how later GPT releases were communicated — and shaped transparency norms across the wider AI industry

### 4.4 GPT-4 Turbo: Efficiency and a Larger Practical Deployment Window

- GPT-4 Turbo was OpenAI's first major effort to make GPT-4-level smarts cheaper and easier to actually use
- OpenAI described it simply as a **cheaper and better GPT-4**
- It could handle **text and image input**, and produce text output — same as GPT-4
- Its context window — how much text it can read at once — jumped to **128,000 tokens**, a massive increase
  - This made it practical for workflows involving very long documents, heavy retrieval, or extended back-and-forth sessions
- GPT-4 Turbo did **not** change the fundamental way people interact with AI — no new paradigm, no new interaction model
- What it changed instead was the **deployment envelope** — meaning: who could use GPT-4-level AI, at what cost, and at what scale
  - Lower cost → more use cases became affordable via the API
  - Bigger context → longer documents and more complex sessions became practical
- Its legacy is likely **not a conceptual breakthrough**, but rather an important efficiency step along the GPT-4 family's trajectory

### 4.5 GPT-4o: Omni and Real-Time Multimodal Interaction

- GPT-4o (the "o" stands for **omni** — meaning all types at once) was a clearer conceptual leap than GPT-4 Turbo
- OpenAI described it as a model that can take **any combination** of text, audio, image, and video as input
- It can output **text, audio, and images** — not just text like previous versions
- It responds to voice inputs with speed close to **real-time human conversation** — very low delay
- The key shift is not just "more capable" — it's a **fundamentally different way** of interacting with AI
  - Previous models: text model with some extras bolted on
  - GPT-4o: built from the ground up to handle voice and multiple types of input naturally
- **Speech became a first-class channel** — not an add-on, but central to how the model works
- **Multimodal understanding** — making sense of different types of input together — became a core feature
- With more power came more risk — the **risk profile expanded** alongside the utility
- New risks flagged in the system card (a document OpenAI publishes about safety):
  - Generating content that violates **copyright**
  - Identifying who a speaker is (**speaker identification** — privacy concern)
  - **Voice cloning** — faking someone's voice for harmful purposes
  - Using the model to **persuade** people in manipulative ways
  - These risks are **unique to the added voice and image modalities** — not issues in earlier text-only GPT versions
- Independent research studies confirmed GPT-4o goes beyond text-only tasks:
  - They tested it on **language, vision, speech, and multimodal tasks** combined
  - It still showed **variability** — results weren't consistent across all modalities
  - It had **modality-specific weaknesses** — things it was worse at in certain input/output types

### 4.6 GPT-4.1: Long Context, Stronger Tool Use, and Developer Orientation

- GPT-4.1 is notable mainly because it was built **for developers**, not everyday consumers
- OpenAI released three versions: **GPT-4.1**, **GPT-4.1 mini**, and **GPT-4.1 nano** — all API-only models (no consumer chatbot)
- The three improvements it focused on:
  - Better at **coding**
  - Better at **following instructions** precisely
  - Much larger **context window** — how much text it can read at once
- Context window: **1,047,576 tokens** — roughly one million tokens, enough for very long documents or entire codebases
- Maximum output: **32,768 tokens** per response
- OpenAI described it as the smartest model **without a reasoning step** — meaning it doesn't "think out loud" before answering, so it's faster
- It also has **low latency** — quick response times — and strong **tool-calling** ability (can trigger external functions or APIs)
- This release marked a shift in who GPT is for:
  - Earlier ChatGPT versions → marketed to **consumers** as a chatbot to chat with
  - GPT-4.1 → marketed to **developers** as **backend infrastructure** (a component you build things on top of)
- Specific developer use cases OpenAI highlighted:
  - Building **code editors**
  - Working inside **code repositories** (large collections of code)
  - **Analyzing long documents**
  - Building **agent-like systems** — AI that can take multi-step actions automatically
- Independent research backed this up: GPT-4.1 showed **strong Python code generation** even when working with libraries it hadn't seen before
  - But researchers also noted it still needs **careful prompt design** — how you ask matters
  - And results should still be **verified** — don't blindly trust the output

### 4.7 GPT-5.x: Frontier Positioning, Reasoning Control, and Professional Workflows

- GPT-5 is not a single model — it's a **family of different models working together as a system**
- The system has three parts:
  - A **fast model** — quick responses for simple tasks
  - A **deeper reasoning model** — slower but more thorough for hard tasks
  - A **router** — a piece of software that decides which model handles each request
- The router picks the right model based on:
  - What type of conversation it is
  - How complex the task seems
  - What tools are needed
  - What the user explicitly says they want
- **GPT-5.4** is the top-tier version, built for complex professional tasks
- GPT-5.4 key specs:
  - Adjustable **reasoning effort** — users can dial up or down how hard the model "thinks" before answering
  - Context window: ~**1 million tokens** — can read very large documents
  - Max output: **128,000 tokens** per response
  - Supports tools: **functions** (custom code), **internet search**, **file search**, and **computer use** (controlling a computer)
- The way OpenAI presents GPT-5 deliberately blurs what counts as "the model":
  - It's no longer just about what the model was trained on
  - The **routing logic**, **safety training**, and **reasoning controls** are all presented as part of the model itself
- OpenAI also documents **tiers of functionality** — different levels of access or capability depending on the plan or use case
- **Safe-completions** — a safety training method built into GPT-5 to reduce harmful outputs
- GPT-5 is designed around **specialized professional workflows** in three areas:
  - **Writing**
  - **Coding**
  - **Healthcare**
- The big-picture shift: GPT-5 is the first point where **deployment architecture** (how it's set up), **safety mechanisms**, and **reasoning options** are all bundled into what OpenAI publicly calls "the model" — the line between model and system has been erased

## 5 Technical Profile of the GPT Family

- A comparison table across all GPT generations reveals **four clear patterns**

### Pattern 1: Growing Scope of Interaction
- GPT models evolved through four stages of what they're used for:
  - **Completion engine** (GPT-3) — just finish the text I started
  - **Chatbot** (GPT-3.5) — have a back-and-forth conversation
  - **Multimodal** (GPT-4, GPT-4o) — handle text, images, audio, video together
  - **Workflow agent** (GPT-4.1, GPT-5) — complete multi-step tasks using tools automatically

### Pattern 2: Massive Expansion of Context
- **Context window** — how much text the model can read at once — grew enormously:
  - GPT-3: **2,048 tokens** (a few pages)
  - Modern long-context models (GPT-4.1, GPT-5): ~**1 million tokens** (entire books or codebases)
- This means newer models can work with far more information in a single session

### Pattern 3: Tools Became Central
- In GPT-3 and GPT-4, tools (like web search or running code) were **not a significant part** of how people used or understood these models
- In GPT-4.1 and GPT-5.4, tools are **core** — both to what the models can do and to what they are designed for
- Tools are no longer an add-on; they define the use case

### Pattern 4: More Use, Less Transparency
- Newer models are used in **far more real-world applications**
- But at the same time, **less detailed technical information** is shared publicly about how they work
- This creates a gap: the models are more impactful, but researchers have less to study
