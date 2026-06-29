# Survey on Evaluation of LLM-based Agents

## Abstract

These days, AI has become very advanced. Now we have something called LLM-based agents — these are AI systems that can think on their own, make plans, use different tools, remember things, and work in changing environments without needing humans to guide them every step.

This paper is the first full survey on how we test and evaluate these agents — basically, how do we check if they are doing a good job or not.

The survey covers 4 main areas:

1. **Basic agent skills** — like planning, using tools, checking their own work (self-reflection), and remembering things (memory)
2. **Application-specific tests** — separate tests for web agents, software engineering agents, science agents, and conversation agents
3. **General-purpose agent tests** — tests for agents that can do many different types of tasks
4. **Evaluation frameworks** — the methods and tools used to measure how good an agent is

The paper also points out that testing is becoming more realistic and harder over time, with benchmarks that keep getting updated. It also highlights some important gaps — like we still don't have good ways to test how cost-effective, safe, or robust these agents are. These are areas where more work is needed in the future.

## Introduction

LLMs have become very powerful in recent years — they can handle all kinds of difficult tasks. But there is one limitation: a normal LLM only works in a single back-and-forth interaction, just text in and text out, nothing more.

LLM-based agents go beyond this. They connect LLMs into a multi-step process where the AI can remember context across many steps, use external tools, access outside knowledge, and actually interact with its environment. These agents can independently come up with a plan, carry it out, and adjust when things change — all on their own. This opens up a huge range of real-world use cases that were not possible before.

Because these agents are being used in real-world situations, it is very important to properly test and evaluate them — to make sure they actually work well. But testing agents is harder than testing a regular LLM because agents are more complex: they work in multiple steps, depend on specific LLM abilities, operate in dynamic environments, and handle diverse and complicated tasks.

This survey is the first comprehensive look at how LLM-based agents are evaluated. It is useful for four types of people:

1. **Agent developers** — who want to check how capable their system is
2. **Practitioners** — who are deploying agents in specific industries or domains
3. **Benchmark developers** — who are building new ways to test agents
4. **AI researchers** — who want to understand the current abilities, risks, and limits of agents

The survey is organized into these main parts:

- **Fundamental agent skills** — planning, tool use, self-reflection, and memory
- **Application-specific benchmarks** — for web agents, software engineering agents, science agents, and conversation agents
- **General-purpose agent benchmarks** — for agents that can handle many different types of tasks
- **Evaluation frameworks** — tools that help developers test their agents throughout the whole development process
- **Discussion** — current trends and future directions in agent evaluation

One important clarification: this survey is specifically about evaluating LLM-based agents. It does not focus on standard LLM benchmarks like MMLU or GSM8K, and it does not go into agent architectures or design choices. It also does not deeply cover multi-agent systems, game agents, or embodied agents — though they are mentioned briefly. The sole focus is on how we evaluate these agents.

## Agent Capabilities Evaluation

LLM-based agents depend on a set of core abilities to do their job well. To understand what they can and cannot do, we need to evaluate these abilities carefully. This section focuses on four foundational capabilities of LLM-based agents.

### Planning and Multi-Step Reasoning

Planning and multi-step reasoning is basically the agent's ability to look at a big, complex problem and break it into smaller steps — and then figure out the right order to tackle those steps to reach a solution.

Normal LLMs can answer questions in one shot, but many real tasks require 3 to 10 intermediate steps before you get to the final answer. This is called multi-step reasoning. Agents need to be good at this to work in the real world.

To test this, researchers have created many different benchmarks — essentially test sets that measure how well an agent reasons across different types of problems:

- **Math reasoning** — GSM8K, MATH, AQUA-RAT
- **Multi-hop Q&A** (where you need to connect multiple pieces of info) — HotpotQA, StrategyQA, MultiRC
- **Science reasoning** — ARC
- **Logical reasoning** — FOLIO, P-FOLIO
- **Puzzles** — Game of 24
- **Common sense** — MUSR
- **Hard reasoning tasks** — BBH

Some of these benchmarks have been specifically adapted for agent-based testing, like HotpotQA and Game of 24, where the agent has to plan and use tools at the same time.

Beyond these general benchmarks, researchers have also built more specialized frameworks just for testing agent planning:

- **ToolEmu** — simulates tool-using scenarios and found that agents need to track state well and recover from mistakes
- **MINT** — tests planning in interactive environments; even advanced LLMs struggle with long multi-step tasks
- **PlanBench** — tests planning across many domains; found that LLMs are okay at short-term planning but poor at long-horizon strategic planning
- **AutoPlanBench** — tests everyday planning scenarios; found that even the best LLM agents are worse than older classical symbolic planners
- **FlowBench** — tests workflow planning in expert-level tasks
- **ACPBench** — tests core reasoning skills
- **Natural Plan** — tests real-world planning expressed in natural language; current best LLMs perform poorly, especially as tasks get more complex

All these benchmarks point to five key skills an agent needs for good planning:

1. Breaking down big problems into smaller sub-tasks
2. Keeping track of what has happened so far (state tracking)
3. Catching and fixing its own mistakes (self-correction)
4. Understanding cause and effect (causal reasoning)
5. Knowing when to change its own plan (meta-planning)

### Function Calling & Tool Use

For an LLM agent to be truly useful, it needs to be able to use external tools — like calling an API, running a search, doing a calculation, etc. This is called function calling or tool use, and it is one of the most important abilities for building real-world agents.

Early agents used simple tools like retrieval systems (fetching relevant documents). Later, more general-purpose tool use came in — examples include ToolFormer, Chameleon, and MRKL.

**How function calling works — step by step:**

1. **Intent recognition** — figuring out that a tool is needed based on what the user asked
2. **Function selection** — picking the right tool for the job
3. **Parameter mapping** — pulling out the right values from the conversation and feeding them into the tool
4. **Function execution** — actually running the tool
5. **Response generation** — taking the tool's output and turning it into a useful reply for the user

**How it has been evaluated over time:**

Early benchmarks like ToolAlpaca, APIBench, ToolBench, and BFCL v1 tested simple, one-step tool calls with clearly stated parameters. They used rule-based matching to check if the agent called the right function with the right inputs. These were a good starting point but did not reflect the messiness of real conversations.

As tool use got more complex, benchmarks improved too:

- **BFCL v2 & v3** — added multi-turn conversations and multi-step tool use, getting closer to real-world scenarios
- **ToolSandbox** — introduced stateful tools (where the tool's state changes between calls), implicit dependencies, and dynamic evaluation
- **Seal-Tools** — tested nested tool calls, where one tool's output feeds into another
- **API-Bank** — used realistic dialogue-based evaluations with large training datasets
- **NexusRaven** — focused on diverse, general tool-use scenarios
- **API-Blend** — built a large dataset covering API detection, slot filling, and sequencing of tool calls — useful for both training and testing
- **RestBench** — tested using multiple APIs together to handle complex user requests
- **APIGen** — automated pipeline to generate high-quality function-calling test data
- **StableToolBench** — solved the problem of real APIs going down by using a virtual API server with caching

More recent benchmarks push into even harder territory:

- **ComplexFuncBench** — tests scenarios where parameters are not explicitly stated, the user has specific constraints, and long context needs to be managed
- **NESTFUL** — tests nested API call sequences where the output of one call becomes the input to the next

### Self-Reflection

Self-reflection is the agent's ability to look at its own work, recognize mistakes, and improve — all on its own, based on feedback. This is important because in long multi-step tasks, errors will happen, and a good agent should be able to catch and fix them without a human stepping in.

Early research on this did not have dedicated benchmarks. Instead, researchers took existing reasoning and planning tests (like AGIEval, MedMCQA, ALFWorld, MiniWoB++) and turned them into multi-turn feedback loops — basically, give the agent feedback and see if it can fix its answer. The problem with this approach was that it only measured whether the final answer got corrected, which is too rough a measure. Also, improvements often depended on clever prompting tricks rather than genuine self-reflection ability.

Over time, dedicated benchmarks were built:

- **LLF-Bench** — a proper standardized benchmark for self-reflection across diverse decision-making tasks. It treats task instructions as part of the environment (not given directly to the agent), and allows randomizing the wording of instructions and feedback to prevent the agent from just memorizing patterns
- **LLM-Evolve** — evaluates self-reflection on standard benchmarks like MMLU, where the agent learns from its past queries and feedback and uses them as examples to do better next time
- **Coding-specific reflection** — researchers extended coding benchmarks like APPS and LiveCodeBench into interactive settings, specifically to test different types of feedback and how agents respond to them
- **ReflectionBench** — takes a cognitive science angle, breaking self-reflection into detailed components:
  - Noticing new information
  - Using memory
  - Updating beliefs when something unexpected happens
  - Adjusting decisions accordingly
  - Counterfactual reasoning (thinking "what if I had done X instead?")
  - Meta-reflection (reflecting on the reflection process itself)
