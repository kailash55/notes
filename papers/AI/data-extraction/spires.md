# SPIRES: Structured Prompt Interrogation and Recursive Extraction of Semantics

## What is this paper about?

Imagine you want to build a giant organized database of knowledge — like a detailed encyclopedia that links concepts together (called a **knowledge base** or **ontology**). Normally, human experts have to read lots of papers and manually fill in this database. That's slow and expensive.

This paper introduces **SPIRES**, a system that uses AI (specifically Large Language Models like GPT) to do this job automatically — without needing to train the AI on thousands of examples first.

---

## The Problem It Solves

1. **Building knowledge bases is painful.** Experts have to read papers and manually extract facts — who interacts with what, what causes what, etc.
2. **Existing AI tools need lots of labeled training data.** You have to show the AI hundreds of examples before it understands what to extract.
3. **Existing tools can't handle complex, nested structures.** Real-world knowledge (e.g., a multi-step drug mechanism) is layered — one thing leads to another leads to another. Most tools can't capture that.

---

## What SPIRES Does

SPIRES takes two inputs:
- A **schema** — a blueprint you define that says "I want to extract: drug → targets → gene → disease"
- A **piece of text** — like a scientific paper or article

Then it **repeatedly asks the LLM questions** (prompt interrogation) to fill in each part of the schema, going deeper and deeper (that's the "recursive" part). It also links extracted terms to real IDs from existing databases (like gene databases or disease ontologies), so the output isn't just words — it's structured, linked data.

---

## Key Strengths

- **Zero-shot learning** — no training data needed. You just describe what you want.
- **Handles complex nested schemas** — it can go multiple levels deep.
- **Easy to customize** — define a new schema, point it at text, and go.
- **Grounded outputs** — extracted entities get real database IDs, not just raw text.

---

## How Good Is It?

- Accuracy is in the **mid-range** of existing tools — not the best, but competitive.
- It **far outperforms** what an LLM can do on its own when it comes to linking entities to real identifiers (called "grounding").
- Big advantage: it works **out of the box on new tasks** with no extra training.

---

## Where Was It Tested?

The authors applied SPIRES to extract:
- Food recipes
- Cell signaling pathways (across multiple species)
- Disease treatments
- Multi-step drug mechanisms
- Chemical-to-disease relationships

---

## The Big Idea

Use LLMs not as a black box that gives free-text answers, but as a **structured extraction engine** — guided by a schema you define, validated against real-world databases. This makes LLMs actually useful for scientific knowledge curation.

---

## Code

Available as part of the open-source **OntoGPT** package:
https://github.com/monarch-initiative/ontogpt

---

*Source: SPIRES paper abstract — zero-shot knowledge extraction using LLMs with schema-guided recursive prompting.*

---

## Section 1: Introduction (Plain Language)

### What are Knowledge Bases and why do they matter?

A **Knowledge Base (KB)** is like a super-organized database where facts are stored in a way that computers can reason over and query precisely. Think of it less like a spreadsheet and more like a web of connected facts.

Two flavors:
- **General-purpose KBs** (e.g., Wikidata) — broad knowledge about anything: countries, ingredients, historical events. Used to power websites, search, and cross-domain analysis.
- **Domain-specific KBs** (e.g., Gene Ontology, Reactome) — deep knowledge for a specific field, like how genes interact inside cells. Scientists use these to interpret experiment results.

All KBs have one thing in common: **humans built them**, usually domain experts painstakingly reading papers and entering data.

---

### The gap: most knowledge is trapped in text

Scientific knowledge lives in journal articles and abstracts — written in natural language, which computers have historically struggled to read and understand.

Modern **Large Language Models (LLMs)** like GPT-3/4 are very good at reading technical text and answering questions about it. But they have two well-known problems:
- **Hallucinations** — they sometimes confidently make up facts that aren't true.
- **Insensitivity to negations** — they can miss or misread statements like "X does NOT cause Y."

So you can't just trust an LLM's raw output to fill a knowledge base — you'd be inserting wrong facts.

---

### The smarter approach: use LLMs to assist curation, not replace it

Instead of asking an LLM to directly answer and then trust it blindly, you can use LLMs as one step in a pipeline that still validates facts against real databases before anything gets stored.

NLP can help at multiple stages of KB construction:
| Stage | What it does |
|---|---|
| **Literature triage** | Picks which papers are even worth reading |
| **Named Entity Recognition (NER)** | Finds mentions of relevant things (genes, ingredients, diseases) in text |
| **Grounding** | Maps those text mentions to real IDs in databases (e.g., "insulin" → DB:P01308) |
| **Relation Extraction (RE)** | Connects entities with relationships (e.g., "Drug X *causes* Disease Y") |

The latest LLMs can do all of these in **zero-shot** or **few-shot** mode — meaning you don't have to train them with hundreds of labeled examples. You just describe the task in a prompt.

---

### The hard part: complex, nested schemas

Most real KBs aren't just flat lists of triples like "A causes B." They have **nested schemas** — structured blueprints that describe how data should look.

**Example — a recipe schema:**
- A Recipe has a list of Steps
- Each Step has: an Action, Utensils, Inputs, and Outputs
- Each Input/Output is a tuple of: a Food Type + a State (e.g., "potato, raw" or "potato, boiled")
- Each food type maps to an ID in a food ontology (FOODON)

**Example — a biological pathway schema:**
- A Pathway has Subprocesses
- Each Subprocess has Steps
- Each Step has: an Action, a Subcellular Location, Inputs/Outputs with activation states and stoichiometry

Filling out these layered structures automatically is hard. Existing NLP pipelines need a lot of custom engineering every time you want to handle a new schema.

---

### What SPIRES does differently

SPIRES is designed to fill in any such schema automatically, given just:
1. A **schema definition** (you describe what you want to extract)
2. An **input text** (the paper or article to extract from)

It works by **recursively prompting the LLM** — asking about each field in the schema, and if a field contains another complex object, going deeper and prompting about that too.

For entity grounding (mapping words to real database IDs), SPIRES uses **external ontology tools** — over 1,000 ontologies from the OntoPortal Alliance, plus tools like Gilda and OGER — rather than trusting the LLM to recall IDs from memory (which it's bad at). This produces far more reliable, consistent mappings.

---

### Why this matters

SPIRES is the first approach that can:
- Handle **arbitrarily nested schemas** without custom engineering per schema
- Do this **without any training data** (zero-shot)
- **Ground extracted entities** to real database IDs reliably, not just return raw text

It turns an LLM from a free-text answering machine into a structured, schema-aware extraction engine — the missing piece for automating knowledge base construction.

---

## Section 2: System and Methods (Plain Language)

### What is a Schema?

A **schema** is like a form or template that tells you what information to collect and how to organize it.

Think of it like a job application form:
- It has specific fields (name, experience, skills)
- Some fields accept free text, some only accept specific values (e.g., "Full-time" or "Part-time")
- Some fields can have multiple answers (multivalued), some only one

In SPIRES, a schema defines exactly what information should be extracted from text — the shape of the data.

**Figure 1 — Example Schema (Recipe)**

![Figure 1: Example recipe schema](../../assets/spires-figure1.png)

> The diagram shows a Recipe at the top. It breaks down into Steps, Ingredients, and Categories. Steps further break into Utensils, FoodItems (inputs/outputs), and Actions. Ingredients break into FoodItems and Quantities. Every leaf node (like FoodType or Unit) has an `id` (a real database identifier) and a `label` (a human-readable name). The "crows feet" symbols on arrows mean "multiple values allowed."

---

### Key Concepts in a Schema

Every schema is made of **classes** (types of things) and **attributes** (properties of those things).

| Concept | What it means | Example |
|---|---|---|
| **Class** | A category of thing | `Recipe`, `Ingredient`, `Step` |
| **Attribute** | A property of that class | `label`, `steps`, `ingredients` |
| **Range** | What type of value an attribute holds | string, number, another class, or a fixed list |
| **Multivalued** | Can it hold multiple values? | A recipe has many steps (yes); one name (no) |
| **Identifier** | Is this a persistent database ID? | `FOODON:03301704` for "onion" |
| **Prompt** | Custom instruction to the LLM for this field | "List only the cooking actions" |
| **Inlined** | Is the sub-object embedded or referenced? | Steps are embedded inside a Recipe |
| **ID Spaces** | What ID prefixes are allowed? | `FOODON`, `MESH`, `GO`, `WIKIDATA` |

---

### What does an extracted result look like?

**Figure 2 — Example text → structured YAML output**

![Figure 2: Recipe text to YAML extraction example](../../assets/spires-figure2.png)

> Given a few lines of recipe text ("On medium heat melt the butter and sauté the onion..."), SPIRES produces a structured YAML object. Each ingredient is mapped to a FOODON ontology ID (e.g., `FOODON:03301704` = onion, whole, raw). Each step lists its action, inputs, and outputs. Where no ontology term exists, a placeholder blank node is used (e.g., `_:ChoppedOnion`). Human-readable labels appear as comments in blue.

This output isn't just text — it's **machine-readable linked data** that can be loaded directly into a knowledge base.

---

### How does SPIRES work overall?

**Figure 3 — The SPIRES pipeline**

![Figure 3: Overview of the SPIRES approach](../../assets/spires-figure3.png)

> The pipeline: a **Schema** + **Text** go into **OntoGPT**. OntoGPT sends prompts to **GPT-3+** (via OpenAI API) and gets answers back. Those answers are then **grounded** against public databases and ontologies (over 1,000 of them). The final output is **Structured Data** — instances and relationships that match the schema exactly.

The key insight: the LLM handles language understanding; external databases handle reliable ID lookup. Neither does the other's job.

---

### Why use external ontologies for grounding?

When you ask an LLM "what's the database ID for onion?", it will often guess or hallucinate a wrong ID. Instead, SPIRES takes the LLM's text answer ("onion") and looks it up in real ontology databases (like FOODON, MeSH, Gene Ontology) using tools like **Gilda** and **OGER**. This gives far more accurate and consistent identifiers.

SPIRES can draw from over **1,000 ontologies** in the OntoPortal Alliance — covering biology, chemistry, food, medicine, and more.
