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

### Memory

Memory is what allows an agent to remember things — not just within a single reply, but across a long conversation or a complex multi-step task. Without memory, an agent forgets what happened earlier and cannot make well-informed decisions.

Agents use two types of memory:
- **Short-term memory** — for keeping track of what is happening right now in the current task
- **Long-term memory** — for storing knowledge and past experiences that can be useful later

Memory is different from tool use. Tool use connects the agent to outside resources. Memory is about the agent retaining context from its own past interactions.

**Research on extending context with memory:**

A big challenge for LLMs is that they can only "see" a limited amount of text at once (context window). Memory mechanisms help work around this:

- **ReadAgent** — groups content into chunks, compresses them into memories, and retrieves relevant passages when needed. Tested on datasets like QUALITY, NarrativeQA, and QMSum
- **MemGPT** — manages a tiered memory system (like a computer's RAM vs. disk storage). Tested on NaturalQuestions and multi-session chat datasets
- **A-MEM** — a more advanced memory architecture, evaluated on the LoCoMo benchmark

**Episodic memory** (remembering specific events with context) has its own benchmark that uses synthetically generated book chapters and events, with LLM-based judges measuring accuracy and relevance.

**StreamBench** is a harder benchmark that tests whether agents can use memory of past interactions and external feedback to keep improving over time. It covers diverse tasks including text-to-SQL, ToolBench, and HotpotQA.

**Memory for better decision-making:**

Memory also helps agents make better real-time decisions and learn from past actions:

- **Reflexion** — tracks success rates on tasks like HotpotQA and ALFWorld, using memory of past failures to do better next time
- **RAISE** — adds a two-part memory system on top of the ReAct framework, evaluated via human judgment on quality and efficiency
- **KARMA** — tests memory in household tasks using metrics like success rate, retrieval accuracy, and memory hit rate
- **LTM-benchmark** — tests conversational agents across long, multi-task sessions with frequent topic switches. Key finding: LLMs do well in single tasks but struggle when tasks are interleaved. Interestingly, a smaller LLM with a good long-term memory system can match or even beat a larger LLM with a bigger context window

## Application-Specific Agents Evaluation

So far we looked at general agent capabilities. Now we look at agents built for specific real-world use cases — like browsing the web, writing code, doing science, or having conversations. These are called application-specific agents, and they each need their own tailored way of being tested.

The main categories covered here are: web agents, software engineering agents, scientific agents, and conversational agents.

**How agent benchmarks are structured:**

Every agent benchmark has three parts:

1. **Tasks** — a set of clearly defined things the agent needs to do, ranging from simple (navigate a website) to complex (solve a scientific problem)
2. **Environment** — the setting where the agent operates. This could be a simulation (static or dynamic), a real-world setup, or include things like user simulations, available tools, and rules the agent must follow
3. **Evaluation metrics** — how performance is measured. Common metrics are success rate, efficiency, and accuracy. These can be measured at a fine-grained level (did the agent take the right individual action?) or at a high level (did the agent complete the whole task end-to-end?)

### Web Agents

Web agents are AI systems that browse websites and complete tasks on them — like booking a flight, shopping online, or filling out a form. Evaluating them means checking how well they complete tasks, navigate websites, and follow safety and compliance rules.

Benchmarks for web agents have evolved in three generations:

**1. Early simulation environments**

The first benchmarks were simple simulated web environments:
- **MiniWob** and **MiniWoB++** — basic environments that tested navigation and task automation. They set the foundation and identified key challenges in web-based tasks

**2. Static datasets for offline testing**

Next came static datasets that allow reproducible, offline evaluation:
- **WebShop** — simulates online shopping, from product search all the way to checkout
- **Mind2Web** and **WebVoyager** — cover a wider range of web interactions, testing the agent's ability to navigate complex site structures and hit intermediate goals along the way

These made it easier to compare different approaches against each other consistently.

**3. Dynamic, real-world-like benchmarks**

More recent benchmarks try to simulate the actual messiness of the real web:
- **WebLinX** — the web interface keeps changing, testing how well the agent adapts on the fly
- **WebArena** and **Visual-WebArena** — include realistic UI elements and visual cues; the agent must interpret what it sees, not just follow a script
- **WorkArena** and **WorkArena++** — simulate complex office/enterprise tasks that require coordinating many actions to reach a long-term goal
- **MMInA** — multimodal and multi-hop evaluation (the agent needs to connect information across multiple steps and sources)
- **AssistantBench** — focuses on realistic, time-consuming tasks spread across multiple websites
- **WebCanvas** — measures completion of key navigational checkpoints, giving a more granular picture of where exactly an agent succeeds or fails
- **STWebAgentBench** — combines static and dynamic elements to test agents under varied conditions

**What is still missing:**

Most benchmarks still focus mainly on task completion and navigation speed. Important real-world concerns like policy compliance, risk mitigation, and safety protocols are largely untested. As web agents move toward actual deployment, these gaps will need to be addressed.

### Software Engineering Agents

Software engineering (SWE) agents are AI systems that can write code, fix bugs, and handle real programming tasks. Evaluating them has gone through a clear progression from simple coding tests to full real-world software development scenarios.

**Early benchmarks — simple coding tasks:**

- **HumanEval** and **MBPP** — the first benchmarks. They tested short, self-contained coding problems, mostly algorithm-style questions. Good starting point but too simple to reflect real software work
- **Open-domain coding benchmarks** — a step forward, covering application-specific scenarios and interactions with libraries and tools. Still limited to simpler tasks

**SWE-bench — the game changer:**

**SWE-bench** was built from real GitHub issues. It provides the full context a developer would have: a detailed issue description, the complete code repository, an execution environment (Docker), and automated tests to verify the fix. This is much closer to actual software engineering work.

Several variants were created to improve reliability:
- **SWE-bench Lite** — 300 focused bug-fixing issues, filtered to remove overly complex multi-file edits
- **SWE-bench LiteS** — further cleaned up by removing issues where the solution was too easy to guess or lacked enough description
- **SWE-bench Verified** — only includes issues with clear descriptions and strong test cases
- **SWE-bench+** — fixes flaws in the original like solution leakage (where hints made tasks too easy) and weak tests
- **SWE-bench (Java version)** — extends the benchmark to Java codebases
- **SWE-bench Multimodal** — tests agents on JavaScript apps that have visual elements (images, UI), highlighting how hard cross-language and visual problem-solving is — top systems still struggle here

**Other notable benchmarks:**

- **TDD-Bench Verified** and **SWT-Bench** — test whether an agent can write tests from a GitHub issue (test-driven development style)
- **ITBench** — focuses on real-world IT automation tasks
- **AgentBench** — evaluates how well SWE agents interact with dynamic environments in real time, not just whether their code is correct
- **SWELancer** — the newest direction: benchmarks based on real freelance coding jobs, where agent performance is tied to monetary value. This tests long-term reasoning and decision-making in genuinely complex, open-ended scenarios

### Scientific Agents

Scientific agents are AI systems that can assist with actual research — generating ideas, designing experiments, writing code to run them, and even reviewing papers. Evaluating these agents has progressed from basic science knowledge tests to full research workflow simulations.

**Early benchmarks — knowledge and reasoning:**

- **ARC**, **ScienceQA**, **ScienceWorld** — tested basic scientific knowledge and reasoning
- **QASPER**, **QASA**, **MS²** — tested whether agents could understand and summarize scientific literature
- **SciRiff** — broader coverage, testing whether agents can follow user instructions across different scientific domains

**Newer benchmarks — the full research process:**

Recent work has moved toward testing agents on actual stages of the scientific research pipeline:

1. **Scientific ideation** — can the agent come up with new, creative, feasible research ideas on its own, comparable to what a human expert would suggest?
2. **Experiment design** — can it plan experiments properly? This includes forming hypotheses, choosing the right methods, and laying out procedures rigorously. Tested by **AAAR-1.0**
3. **Code generation for experiments** — can it write accurate, runnable scientific code? Tested by **SciCode**, **ScienceAgentBench**, **SUPER**, and **COREBench**
4. **Peer review generation** — can it write a proper, substantive review of a research paper that matches the quality of a human reviewer?

**Unified frameworks — testing the whole research cycle:**

Some benchmarks now combine multiple research tasks into one platform:

- **AAAR-1.0** — covers equation inference, experiment design, identifying weaknesses in papers, and review critique — all requiring deep domain expertise
- **MLGym** — a gym-like environment with 13 AI research challenges covering the full workflow from hypothesis generation to experimentation and analysis
- **DiscoveryWorld** — a virtual text-based world with 120 diverse tasks simulating complete scientific discovery cycles, from forming a hypothesis to interpreting results
- **LAB-Bench** — focused specifically on biology research, testing agents on experiment design and interpreting texts, images, and tables

### Conversational Agents

Conversational agents are customer-facing AI systems — think of a customer service chatbot. They need to handle user requests across a multi-turn conversation, while following the company's specific policies and calling the right tools or APIs at the right time.

**Two main ways to evaluate them:**

1. **Prefix-based evaluation** — collect real conversations with all the right actions recorded. Then give the agent the first part of the conversation and ask it to predict the next step. Simple but rigid
2. **Simulation-based evaluation** — simulate both the environment and the user. The agent is judged on whether it reaches the correct end state and gives the right answer to the user. More flexible and realistic

**Key benchmarks:**

- **ABCD (Action-Based Conversations Dataset)** — over 10,000 customer-agent conversations with 55 different user intents, each requiring a specific sequence of actions defined by a policy. Collected via crowdsourcing
- **MultiWOZ** and **SMCalFlow** — other well-known crowdsourced task-oriented dialogue benchmarks
- **ALMITA** — built using a fully automated pipeline: an LLM generates intents, policies, tool APIs, and conversation graphs, then conversation paths are sampled and sliced into test cases. After manual filtering: 192 conversations, 14 intents, 1,420 tests
- **τ-Bench** — simulates dynamic conversations between a real agent and an LLM-simulated user in two customer service domains (airline and retail). Each domain has its own databases, APIs, and a policy document the agent must follow. Includes 115 retail tasks and 50 airline tasks
- **IntellAgent** — an open-source framework that takes a company's database schema and policy document as input, then automatically constructs test scenarios, simulates dialogues between the agent and a user, and has a separate critique agent analyze the conversation and give detailed feedback on policy compliance

## Generalist Agents Evaluation

So far we have looked at agents built for specific jobs — web browsing, coding, science, conversations. Now we look at generalist agents — agents that can do many different types of tasks, not just one. As LLMs have become more powerful, agents have moved from narrow, application-specific use to much broader, general-purpose use. This requires a different, wider style of evaluation.

**General reasoning and tool use:**

These benchmarks test the core skills a general agent needs — multi-step reasoning, problem-solving, and using tools flexibly:

- **GAIA** — 466 hand-crafted real-world questions testing reasoning, multimodal understanding (text + images), web navigation, and general tool use
- **Galileo's Agent Leaderboard** — focuses on function calls and API usage in real-world applications like database queries, calculators, and web services
- **AgentBench** — a suite of interactive environments including OS commands, SQL databases, digital games, and household tasks

**Computer operating environments:**

A step up from tool use — these benchmarks test whether agents can operate within a real computer system, running actual applications and handling complex tasks:

- **OSWorld** — tests navigation and task execution across real operating system environments
- **OmniACT** — similar focus on computer control and task coordination across multiple apps
- **AppWorld** — agents must write and modify code, handle complex control flows, and avoid breaking things unintentionally

**Professional / workplace environments:**

The most ambitious category — these put agents in simulated workplaces and see if they can function like a human employee:

- **TheAgentCompany** — simulates a small software company. The agent browses internal websites, writes code, runs programs, and communicates with simulated coworkers
- **CRMArena** — simulates a large customer relationship management (CRM) system with interconnected data (accounts, orders, cases, knowledge articles). Tests multi-step operations via both UI and API, policy adherence, and complex information integration

**Unified evaluation platforms:**

As the number of benchmarks grows, there is a need for one place to compare agents across all of them:

- **HAL (Holistic Agent Leaderboard)** — a standardized platform that aggregates multiple benchmarks covering coding, interactive applications, and safety assessments in one place

## Frameworks for Agent Evaluation

Benchmarks compare finished agents against a fixed set of tasks. Evaluation frameworks are different — they are tools that developers use throughout the building process to monitor, debug, and improve their agents continuously. You bring your own scenarios; the framework gives you the infrastructure to measure and analyze what is happening.

The key differences from benchmarks:
- Benchmarks use standardized test data; frameworks let you define your own scenarios
- Benchmarks compare final systems; frameworks support the entire development and deployment lifecycle
- Frameworks are general-purpose — they work across many agent types and use cases

**Why a new kind of framework was needed:**

Earlier evaluation tools (like OpenAI Evals) were built for simple single-call interactions — ask the model something, check the answer. But modern agentic workflows involve many steps, tool calls, and dynamic decision-making. That requires frameworks that can track multi-step reasoning, analyze full trajectories, and evaluate things like tool usage quality.

**Popular frameworks:**

- **LangSmith** and **AgentEvals** (by LangChain) — general-purpose agent evaluation and tracing
- **Langfuse** — open-source observability and evaluation, uses OpenTelemetry infrastructure
- **Google Vertex AI Evaluation Service** — Google's cloud-based evaluation service, also uses OpenTelemetry
- **Arize AI** and **Galileo Agentic Evaluation** — commercial platforms for monitoring and evaluating agent quality
- **Patronus AI** — evaluation focused on reliability and safety
- **Databricks Mosaic AI Agent Evaluation** — primarily designed for RAG (Retrieval-Augmented Generation) style tasks
- **Botpress Multi-Agent Evaluation System** and **AutoGen** — specifically support multi-agent systems

**What all frameworks track:**

All of these platforms monitor agent trajectories and measure common metrics like:
- Task completion rate
- Latency and execution speed
- In some cases: throughput and memory usage

Beyond basic monitoring, each framework adds its own methods for quality assessment. Evaluation can also happen at different levels of detail — from individual actions all the way up to full end-to-end task outcomes.

**Final Response Evaluation:**

Most frameworks use an LLM as a judge to evaluate the agent's final responses — basically, one AI grades another AI's output against a set of criteria. Some platforms (like Databricks Mosaic and PatronusAI) offer their own proprietary judge models built in. Most platforms also let you customize the evaluation metrics so you can measure quality and relevance in a way that fits your specific domain.

**Stepwise Evaluation:**

Instead of only checking the final answer, stepwise evaluation looks at each individual action the agent took along the way. This makes it much easier to find where exactly things went wrong.

This includes:
- Checking the text output at each step against predefined criteria
- Checking whether the agent picked the right tool for that step
- Verifying the tool was called with the right parameters and produced the correct output

**Galileo's action advancement metric** is a notable addition here — instead of just marking a step as pass or fail, it measures whether each step actually moved the agent closer to the user's goal. This is more nuanced than a simple success/failure check.

**The open challenge:** Automated judges used in stepwise evaluation tend to be task-specific — they work well for the task they were designed for but are hard to generalize. More general judges exist but don't come with strong quality guarantees. Finding judges that are both reliable and broadly applicable is still an unsolved problem.

**Trajectory-Based Assessment:**

While stepwise evaluation checks individual actions in isolation, trajectory-based assessment looks at the entire sequence of steps together — comparing the path the agent actually took against an expected optimal path. This gives a broader picture of how well the agent reasons and makes decisions, especially around which tools it used and in what order.

- **Google Vertex AI** and **LangSmith** both support this kind of trajectory-level analysis
- **AgentEvals** takes it further — it can evaluate trajectories with or without a reference path, using an LLM as a judge. It also supports graph-based evaluation for frameworks like LangGraph, where the agent is modeled as a graph of nodes and transitions — checking whether the agent followed the expected workflow and triggered the right nodes in the right order

**The open challenge:** Trajectory evaluation is tricky because agents are non-deterministic — the same task can be solved in multiple valid ways. Defining one "correct" trajectory as the reference and penalizing anything different is an oversimplification. This makes it hard to build reliable trajectory benchmarks.

**Datasets:**

Good evaluation requires good test data — and building that data is its own challenge. Most frameworks help with this in three ways:

1. **Annotation tools** — built-in tools to label and curate evaluation data
2. **Human-in-the-loop evaluation** — collecting human feedback from real production runs and using it to refine how the agent is configured
3. **Extracting datasets from production logs** — turning actual real-world interactions into evaluation datasets, which makes the test data more grounded in reality

Some platforms (like Patronus AI and Databricks Mosaic) also support **synthetic data generation** — using their own seed data to automatically create evaluation examples when real data is scarce.

**A/B Comparisons:**

Most frameworks let you run A/B comparisons — comparing two or more test runs side by side, looking at their inputs, outputs, and metrics. This is useful for figuring out whether a change to the agent actually improved things or made them worse.

Some platforms (like Patronus AI) go further and let you compare aggregated results across multiple runs from completely different experimental setups — not just two versions of the same agent.

All frameworks also let you drill down into individual trajectories to find specific failure points.

**The open challenge:** Getting large-scale insights at the trajectory or stepwise level is still hard. Fine-grained analysis works well for a handful of examples but does not scale easily to thousands of runs.

**Gym-like Environments:**

The frameworks discussed so far mostly watch and measure agents passively — observing what they do in real or simulated scenarios. But sometimes you need a more active, controlled setup where the agent can interact with a simulated environment and you can run repeatable experiments. This is where gym-like frameworks come in.

The concept comes from OpenAI Gym, which was originally built for training and testing Reinforcement Learning algorithms. The same idea has been adapted for LLM agents — giving them a dynamic, interactive environment to operate in, rather than just feeding them static inputs.

These environments allow standardized evaluation across different benchmarks and have been proposed for specific agent types:
- Web agents
- AI research agents
- Software engineering agents

## Discussion

### Current Trends

Looking across everything covered in this survey, two major forces are driving how agent benchmarks are being built today.

**1. More realistic and harder benchmarks:**

Early agent benchmarks were simple and artificial — clean environments, straightforward tasks. The field has clearly moved away from this:

- Web agents went from basic MiniWob simulations to dynamic real-world environments like WebArena and VisualWebArena
- Scientific agents went from narrow static benchmarks like LAB-Bench to full discovery simulations like DiscoveryWorld
- SWE agents moved from synthetic coding problems to real GitHub issues in SWE-bench
- Benchmarks like Natural Plan use simulated results from real tools like Google Calendar and Maps

At the same time, tasks are getting harder on purpose — to keep pace with improving agents and make sure benchmarks still challenge them. SWE-bench, SWELancer, COREBench, GAIA, and TheAgentCompany are all deliberately difficult. A telling sign: the best agents in these papers sometimes score as low as 2%. This difficulty is important — it reveals real limitations and pushes progress in long-horizon planning, reasoning, and tool use.

**2. Live, continuously updated benchmarks:**

Static benchmarks have a shelf life problem — as agents get better, the benchmarks get saturated and stop being useful for telling agents apart. The trend now is toward benchmarks that keep evolving:

- **BFCL** has gone through multiple versions, adding live datasets, organizational tools, and multi-turn evaluation
- The **SWE-bench family** keeps expanding — Lite, Verified, SWE-bench+
- **IntellAgent** was built as a more dynamic evolution of τ-Bench

Keeping benchmarks fresh and relevant is just as important as making them hard.

### Emergent Directions

These are trends that are starting to appear but are not fully established yet — the areas where future research needs to go.

**1. More fine-grained evaluation:**

Most benchmarks today only tell you whether the agent succeeded or failed at the end — a single pass/fail score. This is too coarse. It doesn't tell you where the agent went wrong, why it picked the wrong tool, or where its reasoning broke down.

The field needs standardized step-by-step evaluation metrics that track the agent's decisions throughout the task, not just the final outcome. WebCanvas, LangSmith, and Galileo are early examples of moving in this direction.

**2. Cost and efficiency metrics:**

Current evaluations almost always optimize for accuracy and ignore cost. But an agent that is accurate but uses 10x more tokens and API calls than needed is not practical to deploy. Future frameworks need to track:
- Token usage
- API costs
- Inference time
- Overall resource consumption

Without standardized cost metrics, we risk building very capable but completely impractical agents.

**3. Scaling and automating evaluation:**

Hand-annotated benchmarks are slow, expensive, and go stale quickly. The field needs evaluation that can scale:
- **Synthetic data generation** — automatically creating diverse, realistic test scenarios (IntellAgent and Databricks Mosaic are early examples)
- **Agent-as-a-Judge** — using LLM agents to evaluate other agents. This reduces the need for human annotation and can capture more nuanced aspects of performance that rule-based metrics miss

**4. Safety and compliance:**

This is the biggest gap. Most current benchmarks barely test whether an agent is safe, trustworthy, or policy-compliant. Early efforts like AgentHarm and ST-WebAgentBench have started, but coverage is still thin.

What is needed: comprehensive benchmarks that test agents against adversarial inputs, check for bias, and verify that they follow organizational and societal rules. This becomes especially important in multi-agent settings, where risks can emerge from agents interacting with each other in unexpected ways.

## Conclusion

The evaluation of LLM-based agents is a fast-moving field, because the agents themselves are getting more powerful and more autonomous every year. A lot of good work has been done — benchmarks have become more realistic, more dynamic, and much harder.

But the gaps are real. Safety testing, fine-grained step-by-step evaluation, and cost-efficiency metrics are all underdeveloped. Closing these gaps is not just an academic exercise — it is what will make it possible to deploy these agents responsibly in the real world.
