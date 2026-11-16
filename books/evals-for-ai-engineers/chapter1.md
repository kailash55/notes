# Chapter 1: Introduction

**Main idea:** People are using AI chatbots and agents (LLM apps) faster than we can test them properly. Shipping one without testing can cause real harm, so evaluation is needed.

**Key points:**
- More and more companies use LLM apps, but there aren't enough good ways to test them yet.
- If you launch an AI agent without testing it, the results can be serious.
  - Example: In late 2022, a man named Jake Moffatt used Air Canada's website to book a last-minute flight after his grandmother died.
  - He asked the airline's chatbot about a bereavement fare, which is a cheaper ticket for people travelling because a family member died.
  - The chatbot got it wrong. It told him to buy a normal ticket and claim the discount as a refund within 90 days.
  - Air Canada's real rule was that you had to ask for the discount *before* the flight, not after.
  - The airline refused his refund, so he took the case to the British Columbia Civil Resolution Tribunal.
  - In February 2024 the tribunal ruled for him. It rejected Air Canada's claim that the chatbot was a "separate legal entity" and said the airline was responsible for what its chatbot made up.
  - Air Canada had to pay damages and soon took the chatbot off its website.
- Nobody knows exactly why the chatbot gave the wrong answer.
  - It may have rewritten the real policy text incorrectly.
  - It may have mixed in rules from other airlines that do allow refunds after the flight.
- What we do know is that nobody caught the mistake before a customer trusted it.
- Testing the chatbot's answers to common policy questions could have found this problem before launch.
- Wrong answers from AI in real use can lead to legal trouble and damage a company's reputation.
- Evaluation keeps going after launch:
  - **Prelaunch evaluation** tests the system before users see it, so problems are caught early.
  - **Production monitoring** keeps checking the system once it's live. It catches problems that only show up in real use, when users ask unexpected things and the kind of input changes over time. (Chapter 9 covers this.)
- LLM apps are harder to test than normal software:
  - They don't always give the same answer to the same question.
  - There's no exact written rulebook for what they should and shouldn't do.
  - They can be wrong in ways that are hard to predict.
  - Sometimes the mistake is obvious, like in the airline story.
  - Sometimes an answer is correct but feels wrong or out of place (the "vibes are off").
  - Sometimes an answer sounds very confident but is completely wrong.
- Because of this, testing LLM apps is different from normal software testing, and different from testing traditional machine learning (ML) too.
- The main challenge has two parts: how do we know if an AI agent is working well enough, and how do we find where it's going wrong?
- The book gives practical methods you can use on your own projects.
- This chapter covers:
  - What evaluation means for LLM apps
  - The **Three Gulfs** framework, which explains where apps fail
  - The **Analyze-Measure-Improve** cycle, which the rest of the book follows

**In short:** AI apps can give wrong answers with real legal and money costs, like the Air Canada chatbot. They need careful testing before launch and ongoing checks after launch, and that testing works differently from normal software testing.

## 1. What is Evaluation ?

**Main idea:** Evaluation means checking and measuring how good an LLM app is. It's done in different ways and used for different jobs inside the app.

**Key points:**
- Evaluation is a planned, step-by-step way to understand and measure the quality of an LLM app.
- It has two parts:
  - **Qualitative:** looking at the kinds of mistakes the app makes.
  - **Quantitative:** counting how often those mistakes happen.
- A good evaluation gives results that are easy to understand and that you can act on.
- Sometimes evaluation gives you a number.
  - Example: For a travel planning assistant, you could measure how often it suggests real flights and hotels that fit the budget, or whether its trip plans follow what the user asked for.
- Sometimes evaluation is based on human judgment instead.
  - Example: Experts, or the product team itself, read the app's answers and rate them on clarity, tone, or usefulness.
- Both methods can be done loosely or carefully:
  - Quick, informal checks help early on, while you're still building.
  - Clear scoring guides (rubrics) and organized sampling of answers are needed for apps used in production.
- One app usually needs several evaluations, each checking a different part of how well it works.
- Evaluations can be built into the app in four main ways:
  - **Monitoring:** checks that keep running in the background to spot when the app slowly gets worse over time.
  - **Guardrails:** checks placed directly in the app's main flow. If an answer fails set rules, the system can block it, try again, or switch to a backup.
  - **Improvement mechanisms:** using evaluation results to make the app better, for example by:
    - creating labelled examples to train the model further
    - putting hand-picked sample questions and answers into the prompt (called "few-shot examples")
    - finding failure cases that show the app's design needs to change
  - **Model migration:** comparing different model versions to decide when it's worth switching to a newer model.
- We use evaluations to:
  - find errors
  - keep users' trust
  - enforce safety rules
  - keep watching how the app behaves
  - guide steady, planned improvement

**In short:** Evaluation means studying an LLM app's mistakes and counting how often they happen. It can be done with numbers or with human judgment, and it's used for monitoring, guardrails, improvement and choosing when to switch models.

## 2. Evaluation Throughout the Lifecycle of an LLM

### 2.1. PreTraining

**Main idea:** Pretraining is the first step in making an AI model. The model learns language by guessing the next piece of text, and it's tested on how well it makes that guess.

**Key points:**
- Pretraining is the first stage, where a foundation model (a large base AI model) is built from scratch.
- A big neural network is shown huge amounts of text, and sometimes code.
- It has only one goal: guess the next token in a sentence.
- A **token** is the smallest piece of text the model reads or writes at one time. It's usually a short group of letters, about a word or part of a word.
  - Example: "The chatbot answered the question" might be split into: The, chat, bot, answered, the, question.
- Models read and write one token at a time.
- The cost of using an AI through an API is usually based on how many tokens go in and come out.
- By guessing the next token over billions of examples, the model learns:
  - a broad understanding of language
  - some ability to reason
  - some knowledge about the world
- Pretraining is done by big research labs and companies that make AI models.
- It needs a huge amount of computing power and often costs hundreds of millions of dollars.
- Testing at this stage is called **intrinsic** evaluation. It checks whether the model guesses the next token correctly on text it has never seen before.
- One common score is **perplexity**, which measures how surprised the model is by new text.
  - Lower perplexity is better: the model is less surprised by text it hasn't seen.
- Testing also includes **scaling-law analysis**, which tracks how the model's errors go down as you give it more computing power and more training data.
- These scores show whether the model has learned the patterns of language.
- But they **don't** show whether the model is actually useful for the real tasks people build with LLMs.

**In short:** Pretraining teaches a model language by having it guess the next token over billions of examples, at a cost of hundreds of millions of dollars. Its test scores show how well it learned language, not whether it's useful for real-world apps.

### 2.2. Post-training

**Main idea:** Post-training takes a pretrained model and makes it useful for real people. Testing at this stage moves to benchmarks that look like real tasks.

**Key points:**
- Post-training takes a model that has already been pretrained and adjusts it for practical use.
- Models trained only to guess the next token can already handle many new tasks they were never directly trained on.
- Post-training builds on this by pushing the model toward behaviour users want.
  - Example: following instructions, staying on topic, and not giving unsafe answers.
- Post-training usually has two parts:
  - **Step 1:** The company that makes the model trains it further on hand-picked examples of questions paired with good answers. This teaches it to answer in the format people want.
  - **Step 2:** The company uses **preference data**: pairs of answers where a person chose the better one. This nudges the model toward the style people prefer.
- Newer post-training methods change how the model thinks inside:
  - **Direct preference optimization (DPO)**
  - **Process reward models**
  - **Reinforcement learning with verifiable rewards (RLVR):** trains the model to think through long, step-by-step reasoning. (Source: DeepSeek-AI 2025)
- The post-training method affects what kinds of mistakes the model makes in apps.
- So it's useful to know which method was used, even if you never train the model yourself.
- The book doesn't explain these methods in detail, because most engineers only use the model after these steps are already done.
- Testing at this stage changes:
  - Before, it measured how well the model guessed the next token.
  - Now, the model is run on **benchmarks** (standard tests) that are similar to real user tasks.
  - Example: answering questions, reasoning puzzles, safety and bias tests, and coding.
- The model makers design and run these tests themselves.
- The results are often shown as headline benchmark scores.

**In short:** Post-training shapes a pretrained model to follow instructions and stay safe, using good examples and human preferences. The training method affects what mistakes the model makes, and model makers test it on benchmarks that resemble real tasks.

### 2.3 Application

**Main idea:** The application stage is where teams build a ready-made AI model into a real product. That team has to design its own tests for its specific use.

**Key points:**
- Most engineers and product managers work with LLMs at the application stage.
- By now, the company that made the model has already trained it and adjusted it to behave well.
- The job here is to fit the model into a real, specific workflow.
  - Example: sorting emails, helping with customer support, or analysing documents.
- Testing at this stage has to match the goals and data of that specific app.
- Sometimes the model makers' existing tests already cover the task.
  - Example: A coding assistant can use well-known programming benchmarks.
- But most of the time, the app's task isn't covered by public leaderboards (public rankings of models).
  - Example: Apps that review contracts, summarise medical notes, or check compliance with rules each need their own way of testing.
- So the app's own development team is responsible for:
  - deciding what "success" means
  - designing the right ways to measure it
  - setting up monitoring to make sure the app works reliably for what it's meant to do

**In short:** At the application stage, public benchmarks usually don't cover your specific task. Your own team has to define success, choose how to measure it, and keep checking that the app works.

## 3. Evaluation in Applications

**Main idea:** At the application stage, building around the AI model brings three common problems. The book groups them into the "Three Gulfs" framework.

**Key points:**
- At this stage the model is already trained. The work is building things around it.
- This work has three steps:
  - decide what the product should achieve
  - turn that goal into prompts and agent logic (the instructions and steps the AI follows)
  - check that the model's answers are good enough
- Each step brings its own problems:
  - **Goals are unclear at first.**
    - Example: "Summarise emails" sounds simple. But once you look at real outputs, questions come up: how detailed should it be, in what format, and what should be left out?
  - **Instructions can have gaps.**
    - Prompts and agent logic written in everyday language rarely capture everything you mean, so the AI behaves inconsistently or leaves things out.
  - **Behaviour changes with new data.**
    - Something that works on a few test cases may fail when it meets the wide variety of real inputs.
- These problems come up again and again when building LLM apps.
- To think about them in an organised way, the book uses the **Three Gulfs** framework.
- It names three places where development usually breaks down:
  - **Comprehension:** understanding what's going on
  - **Specification:** clearly saying what you want
  - **Generalisation:** working well on new, unseen inputs
- Each gulf is a different place where things go wrong, and each needs a different fix. The book comes back to this idea throughout.

**In short:** Building an LLM app runs into unclear goals, incomplete instructions, and failures on new data. The Three Gulfs (comprehension, specification and generalisation) give these problems names so you can deal with each one separately.

## 4. The Three Gulfs of LLM Application Development

### 4.1. The Gulf of Comprehension (Between the Developer and Data)

**Main idea:** The Gulf of Comprehension is the gap between developers and their data. If you don't understand what your real inputs look like and how the AI actually responds to them, you'll build for an imaginary version of the task.

**Key points:**
- The first gulf is between the developer and the data.
- To build good prompts and agents, you need to know what your real inputs look like.
- If you don't, you'll design for an imagined version of the task, not the real one.
  - Example (emails): If you only look at a few clean, well-written emails, your prompt may work fine on those.
  - But real emails also include:
    - forwarded messages
    - replies with long quoted threads
    - signatures in several languages
    - messy text copied from PDFs
  - A prompt that ignores these might make the AI:
    - summarise the whole old thread instead of just the new message
    - think a signature block is the main request
- This gulf is also about **outputs**, not just inputs. You need to understand how the AI behaves on those inputs.
- You can't read every answer the AI produces, but you still need to know the common ways it succeeds and fails.
- If you don't, a few good examples can fool you, and you'll miss problems that only show up across many cases.
  - Example (email agent): The AI usually finds the sender and writes a decent summary. But looking closer shows repeated mistakes:
    - picking the wrong name when several names are mentioned
    - leaving out key details in long emails
    - copying standard text, like confidentiality notices, into the summary
  - These mistakes may happen only now and then, but when they do, users will feel the system can't be trusted.
- Bridging this gulf means finding ways to understand your inputs and outputs across lots of data, so you know what the data really looks like and how the system really behaves, without checking everything by hand.

**In short:** Learn what your real data looks like and how the AI really behaves on it, across many examples, so you're not fooled by a few clean test cases.

### 4.2. The Gulf of Specification (Between the Developer and the LLM Agent)

**Main idea:** The Gulf of Specification is the gap between what you want and what you actually tell the AI. Vague instructions make the AI guess, and its guesses won't always match what you expected.

**Key points:**
- The second gulf is between what we want and what we actually tell the LLM to do.
- The prompts and agent logic we write often capture what we mean only loosely.
- Everyday language isn't precise, so small gaps in wording can cause big differences in how the AI behaves.
  - Example: The instruction "Extract the sender's name and summarize the key requests in this email" looks clear, but it leaves out important details:
    - Should the summary be a paragraph or a bullet list?
    - Should the sender be the display name, the full email address, or both?
    - Should it include requests that are only hinted at, or only ones stated clearly?
    - How short should the summary be?
- The AI can't make these choices for you.
- If you don't spell them out, the AI will guess, and its guesses won't always match what you expect.
- That's why two runs of the same agent can give:
  - answers in different formats
  - different amounts of detail
  - different handling of unusual cases
- The Gulf of Specification is this gap between what we intend and what our instructions say.
- To close it, state your expectations clearly and match them closely to the real data the agent will see.

**In short:** Instructions that seem clear often leave out key details, so the AI fills the gaps by guessing. Spell out exactly what you want, based on the real data it will handle.

### 4.3. The Gulf of Generalization (Between the Data and the LLM Agent)

**Main idea:** The Gulf of Generalization means that even when you understand your data and write clear instructions, the AI will still sometimes make mistakes on real-world inputs, and you can't avoid this completely.

**Key points:**
- The third gulf is between the data your system sees and how well the LLM handles that data.
- Even if you understand your inputs well and wrote a clear, focused prompt, the AI can still make mistakes.
- LLMs aren't perfect, and they will get things wrong.
  - Example: You clearly told the system to take the sender's name from the email header.
  - Most of the time, the AI does this correctly.
  - But sometimes, when the email body mentions another person (like a famous public figure), the AI wrongly returns that name instead.
  - Your understanding of the data was fine, and so was your prompt. The AI just didn't follow the instruction every time.
  - One possible reason: the AI may lean too much toward names it saw a lot in its training data.
- This is the Gulf of Generalization: AI can behave unpredictably even when comprehension (understanding the data) and specification (clear instructions) are both done well.
- These mistakes usually show up only when the system meets the full variety of real-world data.
- They can't be avoided. However advanced models get, there will always be some cases where they give the wrong answer.

**In short:** Even with good data understanding and clear instructions, AI will sometimes fail on real inputs. These mistakes are unavoidable, so you have to plan for them.

## 5. Why LLM Agent Evaluation Is Challenging

**Main idea:** Every LLM app has to deal with all Three Gulfs in its own way. How much effort that takes depends on whether users can catch mistakes themselves and how costly mistakes are.

**Key points:**
- Testing LLM apps is hard because every app has to bridge all Three Gulfs (Comprehension, Specification, Generalization) again, in its own situation.
- Evaluation isn't a one-time check. It's a repeated process, and you only learn your real requirements after looking at real outputs.
- The Three Gulfs give teams shared words for talking about failures:
  - **Specification failure:** the prompt didn't explain the requirements clearly enough.
  - **Generalization failure:** the AI isn't following instructions consistently.
  - This shared language helps teams find problems faster and pick the right fix, instead of arguing vaguely that the model "isn't working".
- When you start looking at outputs (closing the Gulf of Comprehension), you'll always find mistakes.
- The next step is to work out which kind of mistake it is:
  - If the prompt didn't clearly state the requirement → fix it with better instructions or better agent design.
  - If the prompt was already clear and the AI still fails → it's a generalization problem. Automated evaluations can help measure how often it happens.
- Apps differ in how much users can bridge the gulfs themselves.
  - Example (coding assistant): Programmers are usually skilled and in control. They can:
    - explain exactly what they want (specification)
    - read and fix the code the AI writes (comprehension)
    - run tests that show when the AI fails, like building a working React app sometimes but not always (generalization)
  - Example (AI personal stylist): A user asking for an outfit may not know how to explain clearly what they want.
    - The app may need to ask follow-up questions, such as about the weather or how formal the event is, to fill the gaps.
    - Users can't easily spot generalization failures. If the AI gives good casual winter outfits, they may assume it's also good for winter weddings, and that may not be true.
- Good evaluation means thinking about how much is at stake in each gulf for your app.
- Where the stakes are higher, because users can't easily catch mistakes or because mistakes are costly, you need stricter processes to close the gaps.

**In short:** Each app has to handle all three gulfs in its own way. Telling specification failures apart from generalization failures shows you which fix to use, and apps where users can't catch errors need more careful testing.

## 6. The LLM Evaluation Lifecycle: Bridging the Gulfs with Evaluation

### 6.1 Analyze

**Main idea:** "Analyze" is the first step of the evaluation cycle. You look at real inputs and outputs to find the ways your system fails, before you try to count them.

**Key points:**
- The cycle starts with **Analyze**.
- In this step, you look at real inputs and outputs to see how the system actually behaves.
- The goal is to **find** the types of failures (called failure modes), not to count them yet.
- Analyze is most closely linked to the **Gulf of Comprehension** (understanding your data and how your system behaves).
- But it also uncovers specification gaps and generalization problems. You often find all three kinds of problems in the same session.
- In practice, Analyze means:
  - reading **traces**, which are the full record of what the agent did for each request
  - talking to users or subject experts about what a "correct" answer looks like
  - building a list of the ways the system fails
- Chapter 3 explains this in detail.
  - Example: You check a batch of email summaries and see two repeated problems:
    - the AI sometimes adds useless text from email signatures
    - the AI sometimes leaves out the most important request
  - These findings show where things go wrong, and give hints about whether the cause is unclear instructions (specification) or the AI being inconsistent (generalization).

**In short:** Analyze means reading real traces and talking to experts to list how the system fails, without counting the failures yet.

### 6.2. Measure

**Main idea:** "Measure" is the step where you count how often each generalization failure happens, using automated checks, so you know whether the system is good enough to use.

**Key points:**
- After Analyze, you know what kinds of mistakes the system makes.
- Some are clearly **specification problems**: the prompt didn't say enough, so the fix is obvious.
- Others are **generalization errors**: the prompt was clear, but the AI still gets it wrong.
- The **Measure** step is about putting numbers on those generalization problems.
- The main questions are:
  - How often does each type of failure actually happen?
  - Is that rate low enough for the system to be usable?
- Without measuring, you're going by your impression from a few traces, which can mislead you either way:
  - a few bad examples can make a good system look broken
  - a few good examples can make a broken system look fine
- To measure across lots of data, you build **evaluators**: automated checks that score traces without a person reviewing each one.
- Evaluators can be:
  - rules written in code
  - another LLM acting as a judge (covered in Chapter 5)
  - Example: Even after you made the prompt clearer, the AI still sometimes adds email signature text to summaries. You write a rule-based check that looks for standard signature phrases across thousands of outputs, and find it happens in **8%** of cases.
- Measuring tells you whether a failure is a rare annoyance or a regular, built-in problem.

**In short:** Measure uses automated checks (code rules or an LLM judge) to count how often each failure happens. That way you rely on real numbers instead of impressions from a few examples.

### 6.3 Improve

**Main idea:** "Improve" is the last step of the cycle. You fix the system based on what you found, and the right fix depends on whether the problem comes from unclear instructions or from the AI being inconsistent.

**Key points:**
- The final step is **Improve**: deciding how to change the system based on what Analyze and Measure showed.
- The fix depends on where the errors come from:
  - **Specification gaps** (unclear instructions): usually fixed by making the prompt clearer or adjusting the agent's logic.
  - **Generalization failures** (the AI is inconsistent even with clear instructions): may need stronger fixes, such as:
    - adding more examples
    - improving retrieval (how the system finds and pulls in relevant information)
    - fine-tuning (training the model further) on labelled data
- Chapter 12 explains these fixes in detail, ordered from least effort (quick prompt edits) to most effort (changing the model itself).
  - Example (specification fix): Measurements show key requests are being missed because the prompt never said how detailed to be. You change the instruction to: "Summarize the sender's requests in three concise bullet points."
  - Example (generalization fix): The AI still mixes up names even with a clear prompt. Fixing this may mean:
    - adding labelled training data, or
    - redesigning the app to take the sender from the email's metadata (the technical details, like the From field) instead of asking the AI to find it in the email body
- Each time you go through the cycle, you understand your gulfs better and can choose how strict your testing needs to be for your app.

**In short:** Improve means fixing what you found. Unclear instructions get a better prompt, while inconsistent AI behaviour needs stronger fixes like examples, training data or a redesign.

## 7. Putting It Together

**Main idea:** When an LLM app fails, first work out which of the three gulfs the problem belongs to. Then use the Analyze-Measure-Improve cycle to fix it, again and again.

**Key points:**
- When an LLM app isn't working, ask which gulf the failure belongs to:
  - **Comprehension:** You don't yet understand what your inputs look like or how the system behaves on them.
    - Fix: look at more traces before changing anything.
  - **Specification:** Your requirements aren't clear enough for the AI to follow.
    - Fix: rewrite the prompt or the agent's logic.
  - **Generalization:** The requirements are clear, but the AI still fails on some inputs.
    - Fix: measure how often it happens, then try stronger fixes like more examples, better retrieval, or fine-tuning.
- The **Analyze-Measure-Improve** cycle turns this into a process you can repeat:
  - **Analyze:** read traces to find the ways the system fails.
  - **Measure:** count how often each failure happens.
  - **Improve:** apply a fix, then feed the results back into the next round of Analyze.
- The next chapter covers the first practical step: building a prototype (a first rough version) of the app and writing the first prompts.
- You need something real in place, even a rough version, before you can study how it behaves.

**In short:** Work out which gulf a failure belongs to, then keep repeating Analyze-Measure-Improve. Start by building a rough first version you can study.
