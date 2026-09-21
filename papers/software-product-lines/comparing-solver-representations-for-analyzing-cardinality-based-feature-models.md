# Comparing Solver Representations for Analyzing Cardinality-Based Feature Models

> Source: Fabian Eger, Lukas Güthing, Kevin Feichtinger, Ina Schaefer (Karlsruhe Institute of Technology). GPCE ’26, Brussels. https://doi.org/10.1145/3814885.3816410

## Abstract

**One-line summary of the paragraph:**
- This paper compares three different ways of encoding Cardinality-Based Feature Models so that standard solver tools can catch problems in them.

**Key points:**
- **Feature Models (FMs)** describe which combinations of options ("features") a product can have. They are usually checked by translating them into Boolean logic and running a **SAT solver** (a program that finds whether a set of yes/no rules can all be true at once).
- **Cardinality-Based Feature Models (CFMs)** are a step beyond that: a feature can be turned *on more than once* (e.g. "2 to 8 storage drives"), so plain SAT tools no longer work.
- The standard FM checks — is there any valid configuration, are there "dead" features that can never be selected, "false optional" features that always have to be selected, and is a given configuration valid — need to be reworked for CFMs.
- The authors define what those checks mean in the cardinality world, present **three different mathematical encodings** of a CFM, and implement each for three solver families.
- The three solver families are **ILP** (integer linear optimization), **SMT** (satisfiability modulo theories), and **CSP** (constraint satisfaction).
- The finding: the **CSP encoding is the best** — it supports all the common checks and runs fastest among the compared encodings and solvers.

## Introduction

**One-line summary of the paragraph:**
- Product Line Engineering is about reusing shared parts instead of copying-and-pasting products, and Feature Models are the standard way to show what varies between products.

**Key points:**
- **Product Line Engineering (PLE)** promotes reuse to avoid the mess of maintaining many near-identical product copies.
- A **Feature Model (FM)** is a tree diagram that shows the *hierarchy* between configuration options (features) and the *relationships* between them.
- Features can be selected or deselected, and constraints rule out invalid combinations.
- FMs also expose **anomalies** — mistakes in the model, like:
  - a model that can't be satisfied at all (over-constrained),
  - a **dead feature** that can never be selected in any valid configuration.
- FM analysis usually turns the model into a Boolean formula and uses a **SAT solver**.

**One-line summary of the next paragraph:**
- For modern systems (cloud, distributed, cyber-physical), plain "on/off" features aren't expressive enough — you often need to say *how many* of a feature.

**Key points:**
- Real systems need to configure the *number of instances* of a feature (e.g. how many storage drives or CPU cores), not just whether it exists.
- **Cardinality-Based Feature Models (CFMs)** add **cardinalities**: a feature can be instantiated multiple times.
- Each feature instance also gets its *own* configurable subtree, so the instances can differ from each other.

**One-line summary of the final paragraph:**
- The goal of the paper is to work out which solver + encoding combination is fastest and most complete for analyzing CFMs.

**Key points:**
- Not every encoding can capture the full configuration space without losing information, so the authors test *feasibility* (can it even be done?) as well as speed.
- They build translation rules from CFM concepts into **ILP**, **SMT**, and **CSP** solver representations and implement them.
- They use one state-of-the-art solver per category and compare **analysis time**.
- The aim: figure out which encoding does best for which analysis, and which CFM properties affect performance.

## Foundations

### Cardinality-Based FMs

**One-line summary of the paragraph:**
- A CFM is a tree of features where each feature says how many times it can appear under its parent, plus group constraints for siblings.

**Key points:**
- **Feature Instance Cardinality** is written as an interval ⟨lower, upper⟩ above a feature — e.g. a Smartphone can have "2 to 8" Hardware components, or a Computing Hardware can have "0 to 8" RAM sticks.
- Sibling features are constrained by two group cardinalities:
  - **Group Instance Cardinality** (⟨lower, upper⟩ on the group's edge) — the total number of child instances across all siblings under a parent. E.g. a Smartphone needs "3 to 9" instances of Software *or* Hardware combined.
  - **Group Type Cardinality** ([lower, upper]) — *how many different kinds* of siblings can appear. E.g. a Smartphone must have exactly 2 types (both Software *and* Hardware); Communication Hardware can have exactly 1 type (either WiFi *or* BT, not both — same as the classic "alternative" group).
- **Cross-Tree Constraints (CTCs)** are simpler in CFMs than in Boolean FMs — only two kinds exist:
  - **Require** (single-headed arrow): with source/target cardinality intervals. E.g. if there is exactly 1 IOS instance, there must be 3 to 8 RAM instances.
  - **Exclude** (double-headed arrow): two features can't both be at given counts at once. E.g. exactly 1 WiFi and exactly 1 CPU can't coexist.
- **Compound Intervals** are intervals with *gaps* — some counts in between are disallowed.
- The **Big-M value** is used as a complexity measure: it is the biggest product of the maximum upper bounds along each branch, i.e. a rough ceiling on the total possible feature instances.

### Solver Categories

**One-line summary of the ILP paragraph:**
- ILP solvers optimize a linear objective over integer variables with only linear (non-multiplied) constraints.

**Key points:**
- **ILP (Integer Linear Programming)**: integer variables, constraints must be linear equalities/inequalities, plus a linear **objective function** to maximize or minimize.
- Non-linear constraints can't be expressed directly.
- The authors use Google **Or-Tools** `linear_solver` with the built-in free **MIP** backend (not a licensed solver), to keep results generalizable.

**One-line summary of the SMT paragraph:**
- SMT solvers are SAT solvers extended with richer variable types (booleans, integers, reals, strings).

**Key points:**
- **SMT (Satisfiability Modulo Theories)** supports the richest variable set, with a theory solver handling linear arithmetic and algebraic data types.
- Constraints stay within quantifier-free first-order logic over theory atoms.
- The authors use the **Z3** solver via the **SMT-Lib** standard interface (widely accepted and exchangeable).

**One-line summary of the CSP paragraph:**
- CSP solvers support the most abstract constraints — both linear and non-linear, over finite (and even infinite) discrete domains.

**Key points:**
- **CSP (Constraint Satisfaction)** allows linear *and* non-linear constraints over many variables on finite/infinite integer domains — the most flexible of the three.
- The authors use Or-Tools' **CP-SAT** solver (`ortools.sat.python.cp_model`), which has long outperformed alternatives like Choco in the MiniZinc challenge.

## Solver-Oriented Representations of CFMs

**One-line summary of the intro paragraph:**
- The paper presents three abstract ways to turn a CFM into equations, which are then adapted to each solver.

**Key points:**
- The three approaches are **Basic Cloning**, **Cloning improved with Integer Leaves**, and **Multi-Set**.
- Notation: subscript ₓ for a feature, subscript cl for a clone of a feature, and p for parent.
- The full encodings and implementation live in the paper's replication package.

### Representation using Basic Cloning

**One-line summary of the paragraph:**
- Basic cloning creates one Boolean variable per *clone* (instance) of each feature, establishing a one-to-one clone ↔ variable mapping.

**Key points:**
- **Key idea:** create a constant for every clone of a feature (e.g. 8 constants for 8 possible Hardware clones).
- Each clone is a Boolean flag meaning "is this specific instance active or not".
- This gives a clean one-to-one clone ↔ constant relationship and a *local* interpretation of cardinalities.
- **Downside:** large trees with big intervals generate *many* constants and constraints.

**One-line summary of the cardinality equations (Equations 1–4):**
- The four cardinality types are each encoded as a sum-of-clone inequalities, multiplied by the parent clone's on/off flag.

**Key points:**
- **Feature Instance Cardinality (Eq. 1):** the sum of a feature's clones under one parent clone must fall within [lower, upper] × (parent clone's flag). If the parent is off (0), the sum is forced to 0; if on (1), the sum is bounded normally.
- **Group Instance Cardinality (Eq. 2):** same idea, but summing all *direct children* clones under the parent, compared to the group's [lower, upper] × parent flag.
- **Group Type Cardinality (Eqs. 3–4):** count how many child *types* have at least one active clone. A helper predicate returns 1 if a child has any active clone, else 0; the sum must lie in [lower, upper] × parent flag.

**One-line summary of the cross-tree constraint equations (Eqs. 5–6):**
- Requires and excludes are written as conditions on the *global total* of clones of each feature.

**Key points:**
- **Require (Eq. 5):** if the total (global) active clones of feature A is within a given interval, then the total of feature B must be in its interval. E.g. "if IOS count is in [1,1], then CPU count is in [3,8]".
- **Exclude (Eq. 6):** forbid both features from being simultaneously within their specified intervals — a negated conjunction.

### Modified Cloning Representation using Integer Leaves

**One-line summary of the paragraph:**
- Replacing leaf features' many Boolean clones with a single integer counter saves a huge number of variables, because leaves carry the most clones.

**Key points:**
- **Key insight:** leaf features (bottom of the tree) generate the most clone constants.
- So each *leaf* feature is encoded as a single **integer** variable counting its active instances, instead of many Booleans.
- Example: basic cloning needs 22 Boolean constants for the running example; integer leaves needs only 18.
- The trade-off: this way you lose the info about *which* parent clone a child belongs to — but that's fine for leaves (the full Multi-Set approach, next section, accepts this loss everywhere).

**One-line summary of Equation 7:**
- The group-type helper predicate has to change because a leaf is now one integer, not several booleans.

**Key points:**
- For a leaf, the "is-active" predicate returns 1 if its integer count ≥ 1, else 0.
- For a non-leaf, it sums all clones under the parent and returns 1 if that sum ≥ 1, else 0.
- Requires/excludes, feature-instance, and group-instance cardinalities are unchanged structurally — only the sum-of-booleans-vs-integers encoding differs.

### Representation using Multi-sets

**One-line summary of the intro paragraph:**
- Multi-set encoding collapses all clones of a feature into a single "total count" variable, drastically cutting constants and constraints.

**Key points:**
- Introduced by Weckesser et al.; it collapses every clone of a feature into **one integer constant** representing the total instances in a valid configuration.
- It uses a *global* interpretation of cardinalities; the authors adapt the constraints for a *local* interpretation.

**One-line summary of Equations 8–13 (multi-set cardinalities):**
- Each cardinality type becomes a comparison of a per-feature count variable against bounds multiplied by the parent's count.

**Key points:**
- **Feature Instance Cardinality (Eq. 8):** compare the feature's count to [lower, upper] × parent's count (e.g. IOS: 0·Software ≤ IOS ≤ 1·Software).
- **Group Instance Cardinality (Eq. 9):** sum of child counts vs [lower, upper] × parent count.
- **Group Type Cardinality (Eqs. 10–13):** two constraints. The first (Eq. 10) counts how many child types have count ≥ 1, checked against bounds × parent's activity flag. A second constraint (using a stricter "count ≥ parent count" predicate, Eq. 13) is added for *upper-bound* analysis to correctly maximize features with respect to all parents.
- Requires/excludes work like in cloning, but on the collapsed count variables.

**One-line summary of the closing paragraph:**
- Multi-set is worth keeping for two reasons: its bound analysis is much faster, and it could speed up the cloning approaches via preprocessing.

## Analysis Operations

**One-line summary of the paragraph:**
- Each anomaly check is a specific solver query, and the three main operations are "check valid config", "find gaps", and "find bounds".

**Key points:**
- A **void CFM** is one where the solver returns UNSAT (no valid configuration exists at all).
- **Gap analysis:** to test whether a feature can have exactly *i* instances under one parent, add the constraint "sum of clones under that parent = i" and ask SAT/UNSAT. UNSAT means a **gap** at that cardinality.
- Gaps generalize known FM anomalies:
  - a feature with gaps at *every* possible cardinality is **dead**;
  - if 0 is in the interval but a gap exists at 0, it's a **false optional** feature.
- Gap analysis requires a *cloning* representation (multi-set collapses away the per-parent info).
- **Bound analysis:** minimize/maximize a feature's count; this is an optimization query (works with multi-set via ILP, and with cloning too).
- **Valid configuration check:** map a candidate configuration's values onto the solver constants; SAT means the configuration is valid.

## Evaluation

**One-line summary of the intro paragraph:**
- The authors tested on 34 hand-made CFMs and measured feasibility (RQ1) and performance (RQ2).

**Key points:**
- **34 synthetic CFMs**: 13 contain anomalies, 21 are anomaly-free.
- Average characteristics: tree depth 3.6, 38 features, 1.6 constraints, Big-M value ~60,981.
- CFMs were generated like BeTTy (Weckesser/Schnabel style) but with the authors' own generator, since the originals weren't available.
- Two research questions:
  - **RQ1** — how *feasible* is each encoding (can it create valid configs, find anomalies, and be generated with reasonable effort)?
  - **RQ2** — how *well* does each solver perform on each analysis operation?
- Hardware: Mac Mini 2023 (Apple M2, 16 GB RAM), macOS Sonoma 14.5.

### RQ1: Feasibility of the Different Representations

**One-line summary of the paragraph:**
- All representations could be generated quickly, but only the cloning representations could do every analysis.

**Key points:**
- ILP is optimization-only, so only the **multi-set** representation is created for it — used only for **bound analysis**.
- CSP and SMT get both cloning representations. All representations generated in under a minute.
- **Cloning representations found all anomalies** via bound + gap analysis.
- **Multi-set could NOT find gaps** — it only sees total instance counts, not per-parent breakdown, so it can't detect interior gaps.
- The subject CFMs had few "gap" anomalies (most anomalies were at interval bounds), so most representations found most anomalies anyway.
- RQ1 verdict: all reps/solvers can detect false bounds and void CFMs; only cloning + CSP/SMT can do gap analysis and config validation (they need non-optimizing solvers and local, tree-preserving representations).

### RQ2: Anomaly Analysis Performance

**Bound Analysis:**

**One-line summary of the paragraph:**
- For bound analysis, the CSP and ILP multi-set encodings finished most CFMs in about one second, while SMT was much slower.

**Key points:**
- Bound analysis = one maximize and one minimize solver call per feature.
- CSP and ILP did most subject CFMs in **under ~1 second**; SMT took significantly longer.
- Big CFMs (e.g. 62–78 features, Big-M ~70–112k): CSP ~1.3–1.8s, ILP ~1.6–2.7s, SMT ~8.8–10.7s.
- Small CFMs (5–15 features): all solvers under 1 second.
- Big-M alone didn't matter much — two CFMs with the same feature count but ~2× different Big-M had near-identical times.
- **Cloning bound analysis was disastrous:** a 51-feature CFM took 0.99s with CSP multi-set but ~40 hours (144,115s) with the best cloning variant.
- RQ2 Bound verdict: **CSP multi-set** is best, closely followed by ILP multi-set; cloning and SMT multi-set lag far behind.

**Gap Analysis:**

**One-line summary of the paragraph:**
- Only the two cloning encodings (CSP + SMT) can do gap analysis, and CSP "integer leaves" was the only one that finished every model within the 24-hour limit.

**Key points:**
- Gap analysis is restricted to the two cloning representations on CSP and SMT; each model was capped at 24 hours.
- Only **CSP cloning with integer leaves** completed gap analysis for *all* models in time — by far the most performant.
- **Integer leaves gave a huge speed-up**: on a 78-feature model, CSP basis took 68,364s vs integer leaves 4,568s.
- Both SMT cloning variants exceeded the time limit on large models.
- Number of features mattered more than Big-M; smaller models were fastest.
- A **preprocessing** step (run multi-set bound analysis first, update detected false bounds, then gap analysis) barely helped — e.g. 8,164s → 8,148s on one model.
- RQ2 Gap verdict: **CSP cloning with integer leaves** is best; preprocessing gives negligible benefit.

### Threats to Validity

**One-line summary of the paragraph:**
- Results may depend on the synthetic models and single-solver-per-category choices.

**Key points:**
- Generalizability is limited by **synthetic** CFMs (mitigated by having non-authors generate them, and matching related-work models).
- Synthetics make it easy to control size, feature count, and Big-M for scalability testing — but industrial CFMs would strengthen the comparison.
- Only one solver per category was used; the authors expect similar results from other solvers in each category.
- The encodings are **not formally proven correct** — validated only on a limited set of CFMs and by cross-checking encodings/solvers/approaches against each other.

## Related Work

**One-line summary of the paragraph:**
- Prior work analyzed CFMs with a *global* interpretation (using ILP for bounds, SMT for gaps); this paper uses a *local* interpretation instead.

**Key points:**
- Earlier work (Schnabel, Weckesser) used ILP for bound analysis and SMT for gap analysis — but under a **global** interpretation of cardinalities.
- This paper's multi-set approach is based on that global method, but adapted to **local** interpretation.
- Bontemps et al. added relative cardinalities to FODA FMs (not matching all of this paper's extensions), using the Choco CSP solver on a basis-cloning approach.
- Related techniques also appear in **Clafer** and **UML** model analysis, which use multi-set-like semantics and CSP reasoning (finite satisfiability, generalization sets).

## Conclusion

**One-line summary of the paragraph:**
- CSP is the best solver category overall, and the ideal representation depends on the analysis: multi-set for bounds, integer-leaves cloning for gaps.

**Key points:**
- Compared three representations (multi-set, cloning basis, cloning integer leaves) across three solver families (CSP, ILP, SMT).
- **Feasibility:** cloning (CSP + SMT) finds all anomalies; multi-set + ILP can only analyze cardinality bounds.
- **Performance:** CSP outperformed the other two solver categories across *all* representations.
- **Best representation by task:** multi-set for bound analysis; cloning-with-integer-leaves for gap analysis.
- Big-M and feature count had **no direct, consistent impact** on analysis time (comparing systems close on one property and differing on the other).
- Future work: broader industrial models, unbounded CFMs, and more analysis operations (core features, core cardinalities, and explaining anomalies).

## Key Takeaways

- **CFMs generalize FMs** with cardinalities (multi-instantiation) and are too complex for classic SAT tools — you need ILP, SMT, or CSP solvers.
- **Three encodings** trade variable count against expressiveness: Basic Cloning (most explicit, slowest), Integer Leaves (fast middle ground), Multi-Set (fewest variables, but loses per-parent detail so can't find gaps).
- **CSP (Or-Tools CP-SAT) wins on both feasibility and speed** across the board.
- **Match the encoding to the analysis:** multi-set for bound analysis, cloning-with-integer-leaves for gap analysis.
- **Practical lesson:** no single encoding is right for every check — a CFM tool likely wants multi-set for fast bounds plus integer-leaves cloning for the harder gap/dead/optional checks.

## Citation

> Fabian Eger, Lukas Güthing, Kevin Feichtinger, and Ina Schaefer. 2026. Comparing Solver Representations for Analyzing Cardinality-Based Feature Models. In *Proceedings of the 25th ACM SIGPLAN International Conference on Generative Programming: Concepts and Experiences (GPCE ’26)*. ACM. https://doi.org/10.1145/3814885.3816410