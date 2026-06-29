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
