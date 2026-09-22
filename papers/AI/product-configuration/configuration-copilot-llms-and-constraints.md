# Configuration Copilot: Towards Integrating Large Language Models and Constraints

> Source: Philipp Kogler, Wei Chen, Andreas Falkner, Alois Haselböck, Stefan Wallner (Siemens AG Österreich), ConfWS'24. https://ceur-ws.org/Vol-3812/paper14.pdf

## Abstract

**What the paper presents:**
- A **configuration copilot** — a chat-based interface where a user configures a product by describing their needs in everyday language, step by step, instead of clicking through a technical GUI.

**The problem it targets:**
- A **product configurator** is software that lets a customer customize a product while making sure all the rules (which parts fit together, what's allowed) stay satisfied.
- Today users usually interact with these configurators through a **graphical user interface (GUI)** — buttons, menus, forms.
- But a complex product is built from components and parameters whose meaning is not obvious to a non-expert. That confusion stops people from configuring the product themselves, because they first have to learn what everything means.

**The core idea — combining two very different tools:**
- The authors combine the natural-language skills of a pre-trained **Large Language Model (LLM)** with the guaranteed correctness of a **constraint-based product configurator**.
- An **LLM** is a model trained on huge amounts of text to understand and generate human language.
- A **constraint solver** is a program that searches for a combination of choices that satisfies a set of rules, and can *prove* the answer is valid.
- The LLM handles the messy "what did the user mean?" part. The solver handles the "is this actually a valid configuration?" part.

**The technical architecture they introduce, in four stages:**
1. **Formalize** — turn the user's natural-language requirements into precise, machine-readable constraints.
2. **Solve** — use a constraint solver to find a valid product configuration given the product line and those constraints.
3. **Explain** — turn the resulting configuration back into plain language for the user.
4. The user can then refine their request and repeat.

**How they tested it — two use-cases:**
- Configuring the **GoPhone feature model** (a phone made of yes/no "features").
- Configuring a **metro wagon** (a train car with richer parameters — numbers, colors, arrays).

## Keywords

Product Configuration · Constraints · Feature Models · Large Language Models · Copilot

## 1. Introduction

**What product configuration is:**
- Product configuration means building a customized product from a fixed set of predefined components, while satisfying **constraints** — rules that limit which parameters are allowed and which combinations are possible.
- A **product configurator** is the software tool that lets users do this, usually through a GUI and often in a web browser.

**Why the interface matters (and is usually neglected):**
- How the interface is designed and how the user interacts with it plays a big role in whether a configurator works well — but it is often overlooked.
- This matters especially when complex products are configured by **non-expert users**. The meaning of configurable components and parameters may not be obvious, so users need explanations and face a learning curve before they can do anything.

**The proposed alternative:**
- Instead of a GUI, the authors propose a **configuration copilot** — a text-based chat interface.
- Uninformed users should be able to describe their requirements in natural language, *without* knowing the exact parameter names or components to pick.
- The copilot then configures the product and replies with a valid configuration that matches those requirements.
- The user can then interactively refine the configuration.

**Why they bring in a constraint solver:**
- They use a pre-trained LLM to process the natural language.
- Recent advances let LLMs understand and generate not just language but also code.
- But LLMs have well-known limits: a lack of reliability, no guaranteed correctness, missing domain-specific knowledge in general-purpose models, and limited reasoning ability.
- The authors address these shortcomings by combining the LLM with a constraint solver:
  - The **LLM's strength** is used only for processing the natural-language requirement descriptions.
  - The **reasoning** needed to find valid configurations is done by the **constraint solver** — which is reliable and correct by design.

**Roadmap of the paper:**
- Section 2 explains LLMs and constraint-based product configuration.
- Section 3 covers related work.
- Section 4 details the technical architecture of the configuration copilot.
- Section 5 presents the evaluation on the two use-cases (GoPhone feature model and metro wagon).
- Section 6 concludes with a summary, a statement of limitations, and future work.

## 2. Background

### 2.1. Large Language Models

**How LLMs came to be powerful:**
- A central idea behind LLMs is pre-training on the *general* aspects of natural-language processing tasks (things common to many tasks), before specializing.
- This is made possible at scale by the **Transformer** architecture, which parallelizes well.
- Transformers capture complex patterns and **long-range dependencies** in text (relationships between words far apart in a sentence) through the **multi-head self-attention mechanism**.
- Compared to older state-of-the-art models such as **recurrent neural networks (RNNs)** and **long short-term memory networks (LSTMs)**, Transformers show a clear performance improvement across many NLP tasks.

**Decoder-only models and "auto-regressive" generation:**
- **Decoder-only** models are a subclass of Transformer-based architectures, used mainly for sequence-to-sequence tasks like translation.
- **Auto-regressive** models predict the *next single token* (a token is a sub-word chunk) by maximizing the likelihood of that token given all previous words and the model parameters.
- In other words: they build text one piece at a time, each new piece depending on everything generated so far.

**Why training data matters:**
- The size and quality of the pre-training corpus strongly affect performance.
- LLMs are trained on publicly available data, so they excel at general language tasks.
- But highly specialized tasks need expert knowledge that often isn't in the training data — so the LLM may not generate accurate output for them.
- Task-specific knowledge can be added to a general-purpose LLM through **domain customization**, using techniques like:
  - **Prompting** — adding instructions and examples to the input.
  - **Fine-tuning** — further training the model on specialized data.

### 2.2. Constraint-based Product Configuration

**What product configuration involves:**
- Selecting and assembling various components and options to meet customer requirements *and* constraints.
- Its complexity comes from two things: the enormous number of possible combinations, and the need to satisfy all technical restrictions and customer preferences at once.

**Why constraint-based systems are the right tool:**
- Powerful technologies have been developed over the last decades to handle this complexity.
- Constraint-based systems represent the product line — with all its technical restrictions and requirements — in a clean, logical way.
- This guarantees that **only valid configurations** are ever generated.
- The heart of these systems is their ability to handle complex, combinatorial search spaces efficiently, using advanced solving algorithms such as:
  - **Backtracking** — undoing a choice and trying another when a dead end is hit.
  - **Forward checking** — looking ahead to catch conflicts early.
  - **Constraint propagation** — spreading the effect of one choice through related rules to prune the search space.
- Together these find feasible solutions efficiently while pruning invalid combinations.

**Feature models:**
- An important subdomain of configuration problems is **feature models**, used to represent product lines.
- Constraint techniques are especially well-suited to feature models because the language is simple and the variables are mostly **Boolean** (true/false — a feature is either in the product or not).

**MiniZinc:**
- **MiniZinc** is a constraint language that can represent configuration problems.
- Several efficient solvers can process this language, so it can be used as the backend of a configurator.

**Why configurators usually need a GUI — and why that's a burden:**
- A product configurator is almost always an *interactive* system.
- A GUI lets the user enter requirements, which are passed as input to the constraint solver.
- The solver's results are shown on the GUI, and the user can vary or refine their input, triggering another solver call.
- But designing and implementing a configurator GUI is challenging because the possible interactions are diverse: collecting requirements, reporting invalid combinations, representing a solution, showing a performance/quality value of a solution, and so on.
- Worse, every modification of the product line requires reviewing and possibly adjusting the GUI.
- In the sections that follow, the authors show how to eliminate the need for a product-specific GUI entirely — by using an LLM to talk to the user instead.

## 3. Related Work

**Approaches to make LLMs more reliable, more domain-aware, and better at reasoning:**

- **Few-shot prompting** — adds a small set of example interactions (input plus expected output) to the prompt. This effectively introduces domain-specific knowledge and improves task performance with far less training data than fine-tuning.
- **Chain-of-thought prompting** — provides exemplary intermediate reasoning steps. This improves reasoning, especially on complex tasks.
- **Grammar prompting** — used when a specific output format is expected. Wang et al. describe a "grammar specialization" process that selects a minimal specialized grammar (a subset of the full grammar) using an LLM, then minimizes it by parsing the output and keeping only the rules actually used. **Constrained decoding** then validates the output's syntax.
- **Synchromesh (Poesia et al.)** — a framework that uses few-shot prompting but picks *semantically similar* examples from a larger pool via a similarity metric called **Target Similarity Tuning**. Constraints are enforced through **Constrained Semantic Decoding** to verify syntax, scoping, or type checks. As the LLM builds its output token-by-token, a **Completion Engine** supplies only the valid tokens that can extend the partial program toward a correct full program.
- **Neuro-symbolic approaches** — combine the strengths of neural networks and symbolic reasoners. **Logic-LM (Pan et al.)** improves logical-reasoning datasets by 18% over chain-of-thought prompting. It translates natural-language input into symbolic formulations and hands the answer to a **symbolic reasoner**.

**How this paper relates to the authors' own prior work:**
- It builds on the authors' earlier work on reliably generating formal specifications with LLMs using **algorithmic post-processing**.
- This paper extends that approach to product configuration by applying post-processing to reliably integrate a constraint solver.
- Beyond guaranteeing syntactically valid output (as before), this extension enables arbitrary *semantic* constraints.

## 4. Configuration Copilot

This section presents the technical details of the configuration copilot that combines LLMs with constraint-based configuration.

### 4.1. Architecture

**Figure 1** shows an overview of the architecture. Here is the flow, step by step:

1. A user configures a product by providing a natural-language description of their requirements.
2. The **Formalizer** (see 4.2) is a specialized LLM-based component that translates those requirements into constraints.
3. The **Configuration Engine** is a constraint solver. It tries to find a configuration that satisfies the *general* constraints of the product line *combined with* the user constraints from the Formalizer.
4. The **Interpreter** (see 4.4) translates the found configuration back into natural language.
5. The copilot responds with a natural-language description of the configured product, *accompanied by* the full technical specification (the actual configuration values determined by the Configuration Engine).
6. The user can then further refine the configuration interactively.

> **Figure 1 (description):** "Architecture of the Configuration Copilot" — a diagram showing the loop from the user, through the Formalizer → Configuration Engine → Interpreter, and back to the user as a natural-language answer plus the technical configuration.

### 4.2. Formalizer

**The input and the task:**
- The input is a natural-language description of *arbitrary* product requirements from a non-expert.
- Using the NLP abilities of LLMs, formalization is treated as a **sequence-to-sequence translation task**: from natural language to a formal specification.
- The LLM is tasked with *understanding* the natural language and *identifying* the corresponding parameters or components of the product line — but it is **specifically NOT tasked with reasoning** (for example, constraint satisfaction). That job is left to the solver.

**How the product line knowledge gets in:**
- Pre-trained LLMs perform well on general tasks, but they don't know the specific product to configure, because that data isn't in their training corpus.
- Also, the probabilistic, token-by-token nature of LLM output gives **no guarantee** of correctly generating valid constraints.
- Knowledge of the product line is incorporated through a **system prompt** describing the product line with its parameters and components.
- A small set of examples is appended, each pairing a natural-language input with its expected output, to give the LLM context and guide it toward the expected behavior.
- (Few-shot prompting is known to extend an LLM's capabilities with domain knowledge while needing far less training data than fine-tuning.)

**Why they use an intermediary JSON language:**
- Rather than making the LLM generate output directly in a specific constraint language, they use an **intermediary JSON-based language** that can be easily transpiled (mechanically converted).
- A **transpiler** parses the JSON constraint representation and maps its elements to the corresponding constructs of the target constraint language, following predefined rules.
- Why JSON? Because JSON is widely used, pre-trained LLMs have encountered JSON far more often than less common constraint languages. So generating JSON is closer to what the LLM is already good at.
- Additionally, an intermediary language gives more control over the output, because the available language constructs can be constrained and tailored to the task.
- It also **decouples** the Formalizer from the Configuration Engine: the concrete constraint language can be swapped without changing the Formalizer.

**Which models they benchmark:**
- To generate valid JSON, several state-of-the-art LLMs are evaluated and benchmarked.
- They believe specialized **code LLMs** — pretrained on translating natural language to code in many programming languages — are more suitable for generating structured JSON output.
- They selected four open-access LLMs:
  - Two code LLMs: **CodeLlama** and **Codestral**.
  - Two general-purpose instruction-tuned LLMs: **Meta Llama 3** and **Mistral**.

**How correctness is guaranteed — algorithmic post-processing (Figure 2):**
- Algorithmic post-processing guarantees the correct generation of the JSON-based intermediary language.
- Because the auto-regressive Transformer generates output step-by-step as tokens, the post-processor gets involved at **every generation step**:
  1. The LLM generates a list of candidates for the next token, based on the prompt and the output so far.
  2. These candidates are sorted by priority (as evaluated by the LLM).
  3. The post-processor determines whether each token candidate is a **valid continuation** of the partial output (the partial intermediary JSON).
  4. The highest-priority *valid* token candidate is selected, handed back to the LLM, and added to the partial JSON — extending it one more step.
  5. A **completeness checker** runs after every step to decide whether the JSON is complete.
- The intermediary JSON language is formally defined by a **JSON schema** specification, so the post-processor is essentially a specialized JSON validator that can strictly validate *any partial JSON* against the schema.
- This implementation is based on **deterministic finite automata (DFA)** — a formal way to track, at every point in the output, which next steps are allowed and which would violate the schema.

> **Figure 2 (description):** "Detail view of the Formalizer with post-processing" — a diagram zooming into the LLM's token-by-token generation loop, showing how each candidate token is checked against the JSON schema before being accepted, with the completeness check deciding when the output is finished.

### 4.3. Configuration Engine

**What it does:**
- Given the user constraints combined with the complete product line definition, the Configuration Engine evaluates whether the constraints are **satisfiable** (whether any valid configuration exists) and returns a configuration.
- Both the product line and the user constraints are modeled in the **MiniZinc** constraint language.
- The solver returns the full product configuration as a list of **variable assignments**, which serves as input to the Interpreter.
- The authors treat the constraint solver as a **given, off-the-shelf technology** — it is neither further described nor evaluated in this paper.

### 4.4. Interpreter

**What it does:**
- The Interpreter is an LLM module that explains the product configuration found by the Configuration Engine.
- The goal is to give the user a less technical summary that a non-expert can understand.

**Why a simple approach works here:**
- Structured **few-shot prompting** is sufficient for this use case, because LLMs generally perform well at translating a formal specification into a natural-language summary — all the facts are already present in the prompt.

**What the LLM is given (three things):**
1. The product line definition.
2. Instructions.
3. Examples.

**How the summary stays relevant to the user:**
- The LLM is prompted to judge which properties and components are most important to include in the summary.
- This is done by adding **importance hints** to the product line definition, and by appending the original user input.
- Properties and components that the user mentioned directly are given more importance, so they are more likely to appear in the summary.
- The result is a more natural, context-aware explanation of the most relevant aspects of the configuration.

## 5. Evaluation

The configuration copilot is evaluated on two use-cases: the conceptually simpler task of configuring a **feature model**, and the configuration of a **metro wagon**.

### 5.1. Feature Model (GoPhone)

**The setting:**
- The first use case supports an uninformed user in configuring the **GoPhone** feature model from the SPLOT project.
- The GoPhone is a feature model with **77 features**. Some features are **mandatory**, some **optional**, some **depend on** other features, and some are **mutually exclusive**.
- Example: the feature `call` is mandatory for the GoPhone; `accept_incoming_call` is mandatory if `call` exists; but `show_missed_calls` and `show_received_calls` are optional.
- Each feature assignment is **Boolean**: either the feature is included (`true`) or not included (`false`).
- The product line definition is a MiniZinc program derived directly from the feature model. Each feature becomes a Boolean variable, and constraints limit which combinations of features are possible.

**An example user input:**
> "I need a basic phone to call people and browse the web but I don't play games. I also want to keep track of my appointments."

**Formalizing to JSON:**
- This description is formalized to the intermediary JSON language:

```json
{
  "features": [
    { "name": "make_call", "value": true },
    { "name": "browsing", "value": true },
    { "name": "game", "value": false },
    { "name": "calendar_entry", "value": true }
  ]
}
```

**Transpiling to MiniZinc constraints:**
- The solver-independent constraints are then transpiled to MiniZinc:

```
constraint make_call = true;
constraint browsing = true;
constraint game = false;
constraint calendar_entry = true;
```

**Solving:**
- Together with the MiniZinc product-line definition, the Configuration Engine evaluates the constraints and returns a *full* product configuration of the GoPhone — a list of Boolean feature assignments for every feature (not just the ones the user mentioned).
- The **Gecode** solver was used here, without further configuration or optimization.

**Interpreting back to natural language:**
- The Interpreter converts the configuration back to natural language. Example output (technical specification shortened):

> "Your GoPhone can manage ringing tones, messages, and browse the web. It can also manage calls, read multimedia, and display photos. It has a calendar entry feature and an address book processing system. However, it does not play games, organize tasks, or have currency conversion features."

- Followed by the full technical configuration, e.g.:

```
GoPhone = true; manage_ringing_tones = true; ... browse = true; ... game = false; play_games = false; install_games = false; ...
```

**Why only the Formalizer is formally evaluated:**
- The crucial, potentially failing component is the **Formalizer**: the probabilistic nature of LLMs gives no strict guarantees, especially when translating user requirements into feature assignments.
- A formal evaluation of the **Interpreter** is *not* done, for three reasons:
  1. The correctness requirements for a configuration summary are less strict.
  2. LLMs are known to perform well on simple summarization when the facts are given directly.
  3. It's hard to define a single "correct" reference summary, since many different summaries (explaining various feature assignments or not) could all be considered correct. Ultimately, only the user can decide whether a summary was helpful.

**The Formalizer dataset:**
- Evaluated on a custom dataset of **30 test cases**.
- **15 test cases** create a new configuration from scratch.
- **15 test cases** evaluate **re-configuration**, where an existing configuration is modified.
- Each test case consists of:
  - A natural-language input mentioning **2 to 6 feature requirements** (and, for modification cases, **up to 30 given feature assignments**).
  - The **expected feature assignments** in JSON.
- The Formalizer generates feature assignments in JSON, which are compared to the expected output.

**Handling the ambiguity of natural language:**
- The comparison is conceptually challenging because natural language is inherently ambiguous — often multiple feature assignments could reasonably be considered "correct".
- To reduce this, the dataset was hand-crafted to be less ambiguous.
- But GoPhone features are sometimes hard to distinguish anyway: for example, `browsing` is an optional sub-feature of the more general parent feature `browse`.
- This ambiguity was handled by encoding very similar features to the same representation, so all defined synonymous features count as a correct assignment for a requirement.
- Importantly, the assignment was **not limited to leaf features** — because that would add reasoning requirements to the Formalizer. Consider the leaf features `play_games` and `install_games`, under the parent `game`. If a user only mentions "games", the more abstract `game` feature should be assigned. Otherwise the LLM would have to reason about a proper assignment of leaf features, straying from the most direct translation from natural language.

**The similarity metrics (formulas):**

Let `T` be the set of features that should be `true` in the expected output, and `T̂` the set of features that are `true` in the actual output. Similarly, let `F` be the expected `false` features and `F̂` the actual `false` features.

The **true-set similarity** S_T is the Jaccard similarity (intersection over union) of `T` and `T̂`:

```
S_T = |T ∩ T̂| / |T ∪ T̂|
```

The **false-set similarity** S_F is the same for the `false` sets:

```
S_F = |F ∩ F̂| / |F ∪ F̂|
```

The **overall similarity** S is the weighted average of S_T and S_F, where each is weighted by the size of its union:

```
S = (S_T · |T ∪ T̂| + S_F · |F ∪ F̂|) / (|T ∪ T̂| + |F ∪ F̂|)
```

The result is a number between **0 and 1**: 0 means no similarity, 1 means a perfect match. This metric considers both *which* features were identified in the natural language *and* their Boolean assignment.

The **precision** P, **recall** R, and **F1 score** were also calculated:

```
P = (|T ∩ T̂| + |F ∩ F̂|) / (|T̂| + |F̂|)

R = (|T ∩ T̂| + |F ∩ F̂|) / (|T| + |F|)

F1 = 2 · P · R / (P + R)
```

**Table 1 — GoPhone Formalization results (S = similarity score, F1 = F1 score):**

| Model [Size/Quantization] | S | F1 |
|---|---|---|
| CodeLlama 34B/Q4 | 0.65 | 0.74 |
| **Codestral 22B/Q4** | **0.79** | **0.86** |
| Meta Llama 3 8B/Q8 | 0.46 | 0.58 |
| Mistral 7B/Q8 | 0.69 | 0.79 |

**Discussion of the GoPhone results:**
- Four open-access LLMs from HuggingFace were evaluated: two code models and two general-purpose models.
- **Codestral 22B/Q4**, a state-of-the-art code model, performed best.
- Surprising result: **Mistral 7B/Q8 outperformed the larger code model CodeLlama 34B/Q4**, against the authors' expectations.
- This shows that LLM performance is **use-case specific** and must actually be evaluated, not assumed.
- Performance degrades as instances become more complex. Remedies: use larger models, tune the technical approach, or wait for future model improvements.
- Overall, given the remaining ambiguity of natural language, the results indicate *reasonable* performance — the majority of feature requirements were formalized correctly.

### 5.2. Metro Wagon

**Why this second use-case:**
- It uses not only Boolean but also **numeric variables and arrays**.
- It has components that can occur **multiple times** (similar to "generative constraint satisfaction" or "cardinality-based feature models") — meaning the number of a component is itself a variable to be decided.

**The metro wagon's configurable parts:**
- **Size** — length in millimeters, `length_mm` in the range `10000..20000`.
- **Expected load** — number of passengers, `nr_passengers` in `50..200`, realized as seats or standing room.
- **Seats** — at most 4 per meter of length, so `nr_seats` in `0..200`; each seat has a **type** and a **color**.
- **Handrails** — at most one per wagon (mandatory if there is standing room); type "standard" or "premium".

**The constraints (rules):**
- `nr_seats + standing_room = nr_passengers` — seats plus standing room must equal the total passenger load.
- `nr_seats + standing_room / 3 ≤ 4 · length_mm / 1000` — a seat consumes standing room for 3 people, and total space is bounded by length.
- `nr_seats = count(Seat)` — the number of seats equals the number of `Seat` components.
- `standing_room > 0 → count(Handrail) = 1` — if there is standing room, exactly one handrail is required.
- `all-equal-type()` — all sub-parts must have the same type, except "special".
- `all-equal-color()` — all seats must have the same color, except "special" seats (which must be red).
- Optimization goal: **maximize** `nr_passengers / length_mm` (pack as many passengers per unit length as possible).
- Seat type is `{standard, premium, special}`; handrail type is `{standard, premium}`; color is `{blue, red, white}`.
- `type = special → color = red` — special seats must be red.
- "standard" and "premium" types cannot be mixed (for both seats and handrails).

> **Figure 3 (description):** "Class diagram of the Wagon example" — a UML class diagram showing the `Wagon` class with attributes `length_mm`, `nr_passengers`, `nr_seats`, `standing_room`, plus the `Seat` (type, color) and `Handrail` (type) components, the default values (underlined), and the pseudo-code constraints `all-equal-type()` and `all-equal-color()`.

**An example user input:**
> "The wagon should accommodate more than 120 people with room for 40 to sit. Seats should be red."

**Formalizing to JSON:**
- The description is formalized to intermediary JSON, expressing constraints like: `nr_passengers` greater than 120, `nr_seats` equal to 40, and `seat_color` equal to red for all seats.

**Transpiling to MiniZinc:**

```
constraint nr_passengers > 120;
constraint nr_seats = 40;
constraint forall (i in 1..nr_seats) (seat_color[i] = red);
```

**Solving:**
- Together with the MiniZinc product-line definition, the Configuration Engine evaluates the constraints and returns a full configuration as a list of value assignments to all configurable parameters.

**Interpreting back to natural language:**
- The Interpreter produces, for example:

> "Your metro Wagon is 20 meters long, has space for 160 passengers with 40 red standard seats and a standard handrail. There is also standing room for an additional 120 people."

- Followed by the full technical configuration:

```
length_mm = 20000; nr_passengers = 160; nr_seats = 40; standing_room = 120;
nr_handrails = 1; handrail_type = standard;
seat_color = [red, red, red, ...]; seat_type = [standard, standard, standard, ...];
```

**Extending the similarity metric for richer values:**
- Like the GoPhone case, the Formalizer was evaluated on **30 test cases** (15 new, 15 modifying a given configuration).
- The Jaccard-based similarity metric from before was extended to handle more general value types (strings, numbers, arrays).
- While Jaccard distance remained the basis, a **type-specific value metric** was applied to each configuration parameter present in *both* the expected and actual output:
  - For **numeric values**: consider the operator (`=`, `>`, `<`, etc.) and the value distance relative to the parameter's domain (value range).
  - For **array values**: consider the length and the positional item equality (are the same items in the same positions?).
  - For **string-enumerated values**: an exact value match.
- The total similarity is adjusted according to these value similarities, not just whether the parameters are present.

**The value-adjusted similarity formula:**

Let `C` be the set of configuration parameter names in the expected output, and `Ĉ` the set in the actual output. The **Jaccard similarity** S_J of the parameter name sets is:

```
S_J = |C ∩ Ĉ| / |C ∪ Ĉ|
```

Let `c` be a matching parameter present in *both* outputs, and let `S_v(c)` be the type-specific **value similarity** (between 0 and 1) of `c` between expected and actual output. The **value-adjusted Jaccard similarity** S is then the sum of value similarities over the shared parameters, divided by the size of the union:

```
S = Σ_{c ∈ C∩Ĉ} S_v(c) / |C ∪ Ĉ|
```

- The **F1 score** evaluation was omitted because it adds no value — it appears to correlate strongly with the already rather strict similarity score S.

**Table 2 — Metro Wagon Formalization results (S = similarity score):**

| Model [Size/Quantization] | S |
|---|---|
| CodeLlama 34B/Q4 | 0.77 |
| **Codestral 22B/Q4** | **0.78** |
| Meta Llama 3 8B/Q8 | 0.68 |
| Mistral 7B/Q8 | 0.72 |

**Discussion of the Metro Wagon results:**
- **Codestral 22B/Q4** performed best again, with a similar score to before.
- The other three models all *improved* compared to the GoPhone use-case.
- Even though the metro use-case is more complex, its **domain size** (number of named parameters) is *lower* — which may explain the higher performance.
- Overall, results again indicate reasonable performance for the metro wagon use-case.

## 6. Conclusion

**What the paper achieved:**
- A configuration copilot that lets non-expert users configure a product in natural language.
- The cooperative **neuro-symbolic** approach combines an LLM with a constraint solver to reliably support product configuration.
- An early evaluation on two use-cases (GoPhone feature model and metro wagon) indicates **practical feasibility**.
- The authors believe a configuration copilot is a valuable extension to GUI-based product configurators.
- For a productive (real-world) implementation, the limitations and future work in Sections 6.1 and 6.2 should be addressed.

### 6.1. Limitations

- **Size of the use-cases** — compared to real-world scenarios, the GoPhone feature model and metro wagon are smaller and less complex.
- **Dataset size** — the evaluation used a limited, manually created dataset with 30 instances per use-case.
- **Incomplete evaluation** — while the most critical part (the Formalizer) was evaluated, a formal evaluation of the **Interpreter** and of the **full configuration pipeline** was omitted, because that would require a user study.
- **Real-world value not studied** — the paper shows that creating a productive configuration copilot is *feasible*, but does not study how much value it actually provides to real users in a real-world scenario.

### 6.2. Future Work

- **More complex real use-cases** — evaluate the copilot on more complex use-cases from practice, via a user study.
- **Handling unsatisfiable requests** — when a configuration the user asked for is unsatisfiable, the copilot should suggest alternatives instead of just reverting to the last satisfiable configuration.
- **Soft constraints** — introduce "soft constraints" of the form *"If possible, I would like to …"*, so users can express preferences that are not hard requirements.

## References

The paper cites 25 references. The most notable foundational works:

1. Zhang (2014) — a review of product configuration research (state-of-the-art and future research).
2. Yi, Huang, Yu (2022) — the impact of configurator interaction design on consumer customization experience.
3. Nijkamp et al. (2023) — CodeGen, an open LLM for code with multi-turn program synthesis.
4. Vaswani et al. (2017) — "Attention Is All You Need", the original Transformer paper.
5. Min et al. (2023) — a survey of recent advances in NLP via large pre-trained language models.
6. Ling et al. (2023) — a survey of domain specialization for LLMs.
7. Benavides et al. (2013) — automated analysis in feature modelling and product configuration.
8. Nethercote et al. (2007) — MiniZinc, toward a standard CP modelling language.
9. Falkner et al. (2019) — constraint solver requirements for interactive configuration.
10. Brown et al. (2020) — "Language Models are Few-Shot Learners" (GPT-3).
11. Wei et al. (2024) — chain-of-thought prompting.
12. Wang et al. (2023) — grammar prompting for domain-specific language generation.
13. Poesia et al. (2022) — Synchromesh, reliable code generation from pre-trained models.
14. Pan et al. (2023) — Logic-LM, empowering LLMs with symbolic solvers.
15. Kogler, Falkner, Sperl (2024) — the authors' prior work on reliable generation of formal specifications with LLMs.
16. Rozière et al. (2023) — Code Llama.
17. MistralAI (2024) — Codestral introduction.
18. AI@Meta (2024) — Llama 3 model card.
19. Jiang et al. (2023) — Mistral 7B.
20. Mendonca et al. (2009) — S.P.L.O.T., software product lines online tools.
21. Gecode Team (2006) — Gecode, generic constraint development environment.
22. Levandowsky, Winter (1971) — distance between sets (the basis of Jaccard distance).
23. Falkner et al. (2020) — solver requirements for interactive configuration.
24. Fleischanderl et al. (1998) — configuring large systems using generative constraint satisfaction.
25. Czarnecki, Helsen, Eisenecker (2005) — formalizing cardinality-based feature models and their specialization.
