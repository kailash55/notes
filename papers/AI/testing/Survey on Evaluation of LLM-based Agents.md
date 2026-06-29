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
