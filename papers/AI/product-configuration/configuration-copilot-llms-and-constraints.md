# Configuration Copilot: Towards Integrating Large Language Models and Constraints

> Source: Kogler, Chen, Falkner, Haselböck, Wallner (Siemens AG Österreich), ConfWS'24. /Users/kd/.hermes/cache/documents/doc_d6d7ac8524b6_paper14.pdf

## Abstract

**What this paper builds:**
- A "configuration copilot" — a chat window where a non-expert describes what they want, and the system figures out a valid product configuration for them
- Instead of clicking through a complex GUI full of technical parameters, the user just types their needs in plain words

**The central problem it solves:**
- A **product configurator** lets a customer customize a product while keeping all the rules (what works with what) satisfied
- Complex products are made of components and parameters that only experts understand — so non-experts get stuck and can't configure the product themselves

**The core idea (a "neuro-symbolic" split):**
- Use an **LLM** (language model) to handle the messy translation between natural language and formal rules
- Use a **constraint solver** (a math engine that finds valid combinations) to do the actual reasoning and guarantee the result is valid
- The LLM talks to the human; the solver does the guaranteed-correct thinking

**How it works in three steps:**
1. The user types requirements in natural language
2. A "Formalizer" (an LLM) turns those words into formal constraints
3. A solver finds a valid configuration, and an "Interpreter" (another LLM) explains it back in plain words

**How it was tested:**
- Two use-cases: configuring the "GoPhone" phone feature model (yes/no feature choices), and configuring a metro train wagon (numbers, arrays, colors — richer parameters)

## 1. Introduction

**The usual way configuring works:**
- A product configurator is software — usually a GUI on a website — that lets users build a product from predefined components while honoring rules about what's allowed
- The interaction design (interface) is a big deal but often neglected

**Why non-experts struggle:**
- The meaning of configurable components and parameters isn't obvious
- Users need explanations and face a learning curve just to understand what to click

**The paper's alternative:**
- A text-based chat interface where users describe requirements without knowing the exact parameter names or components
- The copilot returns a valid configuration, and the user can keep refining it interactively

**Why they split the work (LLM + solver):**
- LLMs are great at understanding and generating language (and code), but they have known limits: no guaranteed correctness, missing domain knowledge, weak reasoning
- So the LLM only handles natural language; the **constraint solver** does the reasoning to find valid configurations

**Paper roadmap:**
- Section 2 background, Section 3 related work, Section 4 the architecture, Section 5 evaluation, Section 6 conclusion + limitations + future work

## 2. Background

### 2.1. Large Language Models

**How LLMs got powerful:**
- Modern LLMs are built on the **Transformer** architecture, which lets them train on huge data in parallel and capture long-range patterns in text via a "self-attention" mechanism
- They beat older approaches (RNNs, LSTMs) across many language tasks

**Two relevant model flavors:**
- **Decoder-only** models generate text one piece (**token** = a sub-word chunk) at a time, predicting the next token from everything that came before — this is "auto-regressive" generation
- This is the kind of model the copilot uses

**The key limitation:**
- LLMs know general language well, but specialized tasks (like a specific company's product line) aren't in their training data, so they can't reliably generate correct output for them
- Domain knowledge gets added via **prompting** (giving examples/instructions in the input) or **fine-tuning** (further training the model on specific data)

### 2.2. Constraint-based Product Configuration

**What product configuration is:**
- Selecting and combining components and options to satisfy both customer requirements *and* technical restrictions
- It's hard because the number of possible combinations is huge, and every technical rule must hold

**Why constraint-based systems are the tool of choice:**
- They let you write the product line — its rules and restrictions — in a clean, logical form
- They guarantee that only valid configurations come out
- They use search algorithms (backtracking, constraint propagation) to efficiently explore huge combination spaces and prune invalid ones

**Feature models:**
- A common way to represent a product line, where each "feature" is mostly a yes/no (Boolean) choice
- Constraint techniques fit feature models well because the language is simple and variables are mostly Boolean

**MiniZinc:**
- A constraint language for writing configuration problems
- Several efficient solvers understand it, so it can serve as a configurator's backend

**Why configurators almost always need a GUI (and that's a pain):**
- A GUI collects user requirements, passes them to the solver, shows results, and lets the user refine inputs
- But building and maintaining a GUI is hard — there are many interactions to support (collecting requirements, reporting invalid combos, showing a solution and its quality)
- Worse, every change to the product line means reviewing and possibly reworking the GUI

**The paper's answer:**
- Drop the product-specific GUI entirely and use an LLM to talk to the user instead

## 3. Related Work

**Techniques for making LLMs more reliable / domain-aware:**

- **Few-shot prompting** — adding a few worked examples to the prompt to teach domain knowledge with far less data than fine-tuning
- **Chain-of-thought prompting** — showing step-by-step reasoning in examples to improve performance on complex tasks
- **Grammar prompting** — when a specific output format is needed, restrict the model to a small specialized grammar and validate its output syntax (Wang et al.)
- **Synchromesh (Poesia et al.)** — picks similar examples via "Target Similarity Tuning" and enforces constraints through "Constrained Semantic Decoding"; a "Completion Engine" offers only valid next tokens while building the output
- **Neuro-symbolic approaches** — combining neural networks' strengths with symbolic reasoners. **Logic-LM (Pan et al.)** translates natural language into symbolic form and uses a symbolic reasoner, gaining 18% over chain-of-thought on logic tasks

**How this paper differs / builds on prior work:**
- Builds on the authors' earlier work on reliably generating formal specs with LLMs plus "algorithmic post-processing"
- Extends it to product configuration: post-processing now reliably hooks in a constraint solver, enabling arbitrary semantic constraints (not just syntactically valid output)

## 4. Configuration Copilot

### 4.1. Architecture

**The pipeline (see Figure 1):**
1. User gives a natural-language description of requirements
2. **Formalizer** (LLM component) → translates requirements into constraints
3. **Configuration Engine** (constraint solver) → finds a configuration satisfying the product line's rules + the user's constraints
4. **Interpreter** (LLM) → translates the configuration back into natural language
5. The copilot replies with a plain-language description *plus* the full technical specification (the actual configuration values)
6. The user can refine interactively

### 4.2. Formalizer

**What the Formalizer does:**
- Takes a non-expert's free-form requirement description and turns natural language into a formal specification (a "sequence-to-sequence" translation task)
- It understands language and identifies the right parameters/components — but it is *deliberately not* asked to reason about constraint satisfaction (that's the solver's job)

**How product-line knowledge is injected:**
- A **system prompt** describes the product line (parameters and components)
- A small set of example pairs (natural language input → expected output) is appended for few-shot guidance

**Key design choice — an intermediary JSON language:**
- The LLM does NOT output a specific constraint language directly
- Instead it outputs a JSON representation that is then "transpiled" (mechanically converted) by rules into the target constraint language
- Why: LLMs have seen JSON far more than niche constraint languages, so JSON generation is closer to their strengths — and it decouples the Formalizer from the solver, making the constraint language swappable

**Which models they tested:**
- Four open-access LLMs: two code models (CodeLlama, Codestral) and two general instruction-tuned models (Llama 3, Mistral)
- Reasoning: code models, pretrained on natural-language→code translation, should be better at structured JSON output

**How correctness is guaranteed — algorithmic post-processing (Figure 2):**
- The model generates its output token-by-token
- At every step the post-processor checks whether the next-token candidate is a valid continuation of the partial JSON; it picks the highest-priority *valid* token
- A completeness checker runs after each step to decide when the JSON is done
- The JSON language is defined by a **JSON schema**, so the post-processor is essentially a strict validator of any *partial* JSON against the schema (implemented with automata)

### 4.3. Configuration Engine

**What it does:**
- Takes the user constraints + the full product line definition and checks whether they're satisfiable, returning a concrete configuration
- Both the product line and user constraints are modeled in **MiniZinc**
- The solver returns a full configuration as a list of variable assignments, which feeds the Interpreter
- The paper treats the solver as a given, off-the-shelf technology (used as-is, not evaluated)

### 4.4. Interpreter

**What the Interpreter does:**
- Takes the solver's technical configuration and explains it in a non-technical summary the user can understand

**Why a simple approach suffices here:**
- Turning a formal spec into a natural-language summary is a task LLMs do well, because all the facts are already in the prompt
- Structured few-shot prompting is enough

**How it makes the summary relevant:**
- It gives the LLM three things: the product line definition, instructions, and examples
- It adds "importance hints" to the product line definition and re-includes the original user input
- Properties/components the user explicitly mentioned get more weight, so the summary emphasizes what the user actually cared about

## 5. Evaluation

### 5.1. Feature Model (GoPhone)

**Setup:**
- GoPhone— a feature model from the SPLOT project, with 77 features
- Features can be mandatory, optional, dependent on others, or mutually exclusive
- Example: "call" is mandatory; "accept_incoming_call" is mandatory once "call" exists; "show_missed_calls" is optional
- Each feature is a Boolean variable ("included" = true / false)

**Example user input:**
> "I need a basic phone to call people and browse the web but I don't play games. I also want to keep track of my appointments."

**The flow:**
- Input → formalized to intermediary JSON (e.g. make_call=true, browsing=true, game=false, calendar_entry=true)
- Transpiled to MiniZinc constraints
- Solved with the **Gecode** solver → returns full Boolean feature assignments
- Interpreted back to a plain summary plus the full technical spec

**Where it can fail:**
- The Formalizer is the risky part — the LLM's probabilistic nature offers no strict guarantees, especially when translating requirements to feature assignments
- The Interpreter was intentionally *not* formally evaluated, because a summary's correctness bar is looser, LLMs summarize well with facts in the prompt, and there's no single "correct" reference summary (many valid summaries exist; the user decides if it's helpful)

**How the Formalizer was measured:**
- A custom dataset of 30 test cases: 15 build from scratch, 15 modify an existing configuration
- Each case has natural-language input (2–6 feature requirements) + expected JSON feature assignments

**Handling natural-language ambiguity:**
- Natural language is inherently ambiguous, so they hand-crafted the dataset to be less ambiguous
- Some GoPhone features are nearly indistinguishable (e.g. "browsing" is a sub-feature of "browse"); they treated synonymous features as equally correct
- They deliberately did NOT require leaf-level precision (e.g. if a user says "games", the abstract parent "game" feature counts — not the leaf features "play_games"/"install_games"), to avoid adding reasoning burden to the Formalizer

**Metrics (Jaccard-based):**
- They measure similarity between expected (T, F) and actual (T̂, F̂) true/false feature sets
- True-set similarity S_T and false-set similarity S_F, then combine into an overall similarity S (0 = none, 1 = perfect)
- Also compute precision, recall, and F1

**Results (GoPhone):**
- Codestral 22B/Q4: best — similarity 0.79, F1 0.86
- Mistral 7B/Q8: 0.69 / 0.79
- CodeLlama 34B/Q4: 0.65 / 0.74
- Meta Llama 3 8B/Q8: 0.46 / 0.58

**Observations:**
- Surprising: the smaller Mistral 7B beat the larger CodeLlama 34B code model — LLM performance is use-case-specific and must be measured, not assumed
- Performance drops as instances get more complex; remedies are bigger models, better technique, or future model improvements
- Overall, "reasonable" performance — most feature requirements were formalized correctly despite remaining ambiguity

### 5.2. Metro Wagon

**Why a second, harder use-case:**
- Goes beyond Boolean features to numeric variables and arrays
- A wagon has components that can occur multiple times (like "generative constraint satisfaction" or "cardinality-based feature models")

**The wagon's configurable parts:**
- **Size** — length in mm (10,000–20,000)
- **Expected load** — number of passengers (50–200), realized as seats or standing room
- **Seats** — max 4 per meter of length; attributes: type (standard/premium/special) and color (blue/red/white)
- **Handrails** — at most one; mandatory if there's standing room; type standard/premium

**The rules (constraints):**
- One seat takes standing room for 3 people
- Type cannot mix standard and premium (for seats and handrails)
- All seats must be the same color, except "special" seats must be red

**Example user input:**
> "The wagon should accommodate more than 120 people with room for 40 to sit. Seats should be red."

**The flow:**
- Formalized to JSON: nr_passengers > 120, nr_seats = 40, all seats red
- Transpiled to MiniZinc constraints
- Solved → full variable assignment (length=20000, passengers=160, seats=40, standing_room=120, 1 handrail = standard, etc.)
- Interpreted back: "Your metro Wagon is 20 meters long, has space for 160 passengers with 40 red standard seats and a standard handrail…"

**How similarity is measured for richer values:**
- Same 30 test cases (15 new, 15 modify)
- Extended the Jaccard metric for non-Boolean values with a type-specific value metric:
  - numeric: operator match + distance relative to the value range
  - arrays: length + positional item equality
  - strings/enums: exact match
- Combines name-set Jaccard similarity with value similarities (value-adjusted Jaccard)

**Results (Metro Wagon):**
- Codestral 22B/Q4: best again — 0.78
- CodeLlama 34B/Q4: 0.77
- Mistral 7B/Q8: 0.72
- Meta Llama 3 8B/Q8: 0.68

**Observations:**
- The other three models improved vs. GoPhone
- The wagon is more complex, but has *fewer named parameters* — likely why scores went up
- Again "reasonable" performance

## 6. Conclusion / Limitations / Future Work

**What the paper concludes:**
- A configuration copilot is feasible: combine an LLM (neuro) with a constraint solver (symbolic) so non-experts configure products in natural language
- The early evaluation on two use-cases shows practical feasibility
- The copilot is a valuable extension to GUI-based configurators

**6.1 Limitations:**
- Small use-cases (GoPhone + metro wagon are smaller/less complex than real products)
- Small manually-created dataset (30 instances per use-case)
- Only the Formalizer was formally evaluated — the Interpreter and full pipeline were not (that needs a user study)
- Doesn't study how much real value it provides to real users in the real world

**6.2 Future work:**
- Evaluate on more complex real use-cases via a user study
- When a requested configuration is **unsatisfiable**, suggest alternatives instead of just reverting to the last valid one
- Add **soft constraints** ("If possible, I would like to…"), i.e. preferences rather than hard rules