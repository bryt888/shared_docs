
# Jev: Separating “Judgment” from Large Language Models

![Jev Overview and Examples](Jev-Summary_Examples.png)

## 1. What Is Jev?

The blogger’s core description of Jev is:

> **Jev is not a large language model designed to generate content. It is a model specialized in making judgments and decisions.**

Traditional LLMs usually handle multiple jobs at once: understanding information, reasoning, making decisions, and generating content.

Jev takes the **Judgment / Decision** part out of that pipeline and turns it into an extremely lightweight, fast, and inexpensive service.

It currently focuses on three main types of tasks:

| Type | What Jev Does |
|---|---|
| **Boolean** | Returns True / False with probabilities |
| **Choice** | Selects from up to 255 candidates and assigns probabilities to each |
| **Score** | Scores something on a specified scale and returns confidence |

Jev does not write articles or code, and it does not explain its reasoning. It currently focuses primarily on text-based judgment tasks.

The blogger describes it as a **System 1 / Fast Thinking** model.

Unlike large reasoning models that perform step-by-step reasoning, Jev attempts to make the judgment through a single forward pass.

---

# 2. Jev’s Biggest Advantage: Not Higher Accuracy, but Extreme Speed and Low Cost

This is one of the most important points repeatedly emphasized by the blogger:

**Jev should not be understood as “a model smarter than GPT.”**

According to the benchmark numbers cited in the video, Jev achieves approximately **67.8% accuracy**, while the other models mentioned range roughly from 66.8% to 74.1%.

Therefore, Jev’s accuracy is in roughly the same general range as mainstream LLMs, but it does not outperform the strongest models.

Its real advantages are elsewhere.

### ⚡ Speed

According to the blogger, Jev can usually complete judgments within **a few hundred milliseconds**.

More importantly, many judgment tasks can be processed in parallel.

He cites one experiment in which approximately:

> **1,500 questions were submitted at once, with results returned in about 610 ms.**

### 💰 Cost

The price cited in the video is approximately:

**$0.042 per 1 million input tokens**

Because Jev only outputs small structured results—probabilities, selections, or scores—it essentially avoids the expensive output-token generation associated with traditional LLMs.

The blogger summarizes the difference as approximately:

> **1/200 the cost of GPT-5.6 and around 400× faster.**

Therefore, the real idea is not:

**“Replace GPT with Jev.”**

Instead, it is:

**“Continue using GPT, Claude, or other powerful models for complex understanding and generation, while outsourcing large numbers of simple judgment tasks to Jev.”**

---

# 3. Practical Use Cases Discussed by the Blogger

This is probably the most valuable part of the video.

## Case 1: YouTube Comment Filtering

This is a workflow the blogger himself plans to use.

Every week, he needs to select interesting comments from a large number of YouTube comments.

Originally, he did this manually. Later, he used GPT-5.6 Sol.

The system needs to determine:

- Is the comment too long or too short?
- Does it contain useful information?
- Is it relevant to the video?
- Does it express strong emotion?
- Is it worth including in the program?

These are not really generation tasks.

They are a collection of **judgment tasks**.

His new workflow therefore becomes:

**Program collects comments → Jev performs batch judgments → Only selected comments continue to later stages**

These judgments can also be processed in parallel.

The blogger says he tested five real-world scenarios and the total cost was approximately **$0.0001 (0.01 cent)**.

At that cost level, even thousands of calls per day might cost only a few dollars per month.

---

## Case 2: An AI Agent “Firewall” for Tool Calling

This is one of the most important engineering applications discussed in the video.

Modern AI agents constantly need to decide:

> Does this request require Search?  
> Does it require a Browser?  
> Does it require webpage extraction?  
> Should we invoke a more expensive model?  
> Or do we not need a tool at all?

If an expensive LLM repeatedly makes all of these routing decisions itself, it can consume a large number of unnecessary tokens.

Instead, Jev can be inserted in front of the tools:

**User Request → Jev Decision → Select Tool / Model → Execute**

The video cites tests suggesting that this approach can block approximately **94% of incorrect tool calls**.

In this role, Jev essentially becomes a very inexpensive:

**AI Router / Semantic Firewall / Decision Gate**

---

## Case 3: Quality Verification in Large LLM Workflows

The blogger says that in some of his complex workflows, nearly **50% of token consumption can come from verification and validation**.

For example, after generating a report, the system may still need to check:

- Is the data factual?
- Is the source trustworthy?
- Should this claim cite a research paper?
- Should it cite a news source?
- Does the output satisfy a specific requirement?
- Does the final result comply with all instructions?

Traditionally, these checks may still be performed by the most expensive model.

His proposed architecture is:

**Expensive model → Generate**

**Jev → Verify / Judge**

This could significantly reduce the cost of complex Agent pipelines.

---

## Case 4: Gaming

People in the community have experimented with using Jev for games such as:

**Super Mario, Doom, Tetris, Pac-Man, and Gomoku.**

Jev itself does not interpret the game screen.

Visual understanding is still handled by another model.

Jev is responsible for decisions such as:

> Is there a hole ahead?  
> Should I jump now?  
> Can this jump succeed?  
> Which action should I choose next?

The architecture becomes:

**Vision Model → State → Jev → Action Decision**

rather than asking Jev itself to understand the image.

---

## Case 5: High-Level Drone Decisions

Similarly, Jev is not suitable for directly controlling a drone.

Real-time flight control must remain with traditional flight-control systems because a latency of several hundred milliseconds is far too slow for low-level control.

However, Jev can potentially handle higher-level decisions such as:

> Should I fly left or right?  
> Which route should I choose?  
> Which strategy should I use?

Therefore:

**Flight Controller → Real-time Control**

**Jev → Tactical / Strategic Decision**

---

## Case 6: GitHub Commit Classification

Some users have applied Jev to GitHub commits to determine whether a commit represents:

- Bug fix
- Security fix
- Feature
- Other category

Again, this is a classic example of:

**Large Amount of Text → Classification / Judgment**

rather than content generation.

---

## Case 7: The “Human Compiler”

This is one of the experiments the blogger found particularly interesting.

The idea is to feed corporate emails into Jev and classify statements such as:

- Which sentences create artificial urgency?
- Which statements are sarcastic?
- Which statements are aggressive?
- Which statements contain passive-aggressive language?

The system then produces a structured analysis.

Fundamentally, this is still:

**Natural Language → Classification / Scoring**

---

# 4. Where Does Jev Fit in an AI Architecture?

Combining all of the blogger’s examples, the architecture can be summarized like this:

```text
                 ┌── Search
                 ├── Browser
User → State → JEV ── Expensive LLM
                 ├── Tool / MCP
                 ├── Reject
                 └── Continue
                       ↓
                 Generate Result
                       ↓
                     JEV
                       ↓
               Verify / Score
```

In other words:

> **Jev is better understood as a low-cost decision layer inside an AI system rather than as a new primary AI brain.**

Large models continue to handle:

**Reasoning + Understanding + Generation**

Jev handles:

**Classification + Routing + Filtering + Scoring + Verification**

This is why the blogger believes it is particularly well suited for Agent workflows.

---

# 5. The Key to Using Jev: State Matters More Than Prompt Size

The blogger emphasizes another important point:

**Giving Jev more information is not necessarily better.**

A better architecture is:

```text
Raw Data
   ↓
Python / RAG / Small LLM
   ↓
Extract Useful State
   ↓
Jev
   ↓
Decision
```

If large amounts of irrelevant information are included in the input, accuracy may actually decrease.

Therefore, Jev is better understood as:

> **State + Rules / Question → Decision**

rather than:

> **Everything → Jev → Magic Answer**

---

# 6. How Could Jev Change the AI Industry?

The blogger connects the idea directly to the **Jevons Paradox**.

The name “Jev” itself comes from Jevons.

The basic idea is that when a resource becomes dramatically more efficient and cheaper to use, total consumption may actually increase because many new use cases become economically viable.

The blogger therefore does not believe the long-term result will simply be:

> “Jev reduces AI token consumption.”

Instead, he sees two stages.

### Short Term

A large amount of validation, routing, classification, and filtering no longer needs to be handled by expensive LLMs.

The cost of individual AI workflows could therefore fall significantly.

### Long Term

Once judgment becomes almost free, developers may begin inserting AI judgment into places where using AI previously made no economic sense.

As a result, the total number of AI calls could actually increase dramatically.

---

# 7. Jev Itself May Not Have a Deep Technological Moat

The video also raises an important question about defensibility.

At the moment, Jev has:

- No open-source release
- No published paper
- No disclosed parameter count
- No disclosed architecture
- No detailed public training methodology

The blogger cites external probing experiments suggesting that Jev may be a **sparse Mixture-of-Experts model with roughly 10B active parameters**.

More interestingly, someone on Hacker News reportedly summarized Jev as essentially:

> **“A zero-shot classifier.”**

According to the video, founder Almeida essentially agreed with that characterization.

The blogger therefore argues that the most important development may not be **Jev as a specific company or model**.

Instead, Jev may demonstrate that:

> **“Judgment Models” could become an independent category of AI models.**

If that proves correct, companies such as OpenAI, Google, Anthropic, and others could eventually introduce their own specialized judgment or classification models.

---

# 8. The Biggest Concern: Cheap Judgment Can Lead to Over-Reliance on Judgment

The final part of the video focuses heavily on risk.

Jev’s reported accuracy is only around **68%**, and its output generally consists of:

**Decision + Probability / Confidence**

It does not necessarily provide a detailed explanation of why it made the decision.

The real danger appears when judgment becomes so inexpensive that developers start thinking:

> **“Why not let AI judge everything?”**

If software reads only the final decision and ignores confidence levels, biases, first impressions, or stereotypes embedded in the model could be converted directly into automated decisions.

Therefore, the blogger’s final position is fairly clear:

**Jev is extremely useful, but it should not be treated as a smarter or inherently correct brain.**

---

# One-Sentence Summary

If I had to summarize the entire video for a colleague in one sentence:

> **Jev’s breakthrough is not that AI can make smarter judgments, but that judgment has become so fast and inexpensive that it can potentially be used everywhere. It separates judgment from expensive large language models and turns it into an independent infrastructure component for routing, filtering, scoring, and verification—while expensive LLMs are reserved for tasks that genuinely require reasoning and generation.**

This is also why the concept is particularly relevant to architectures involving **State → Decision**, deciding whether to invoke **MCP / Tools**, and other Agent-based workflows.

The most interesting thing to study may therefore not be Jev itself, but the broader architectural pattern:

> **Small / Fast Judgment Model + Large Reasoning Model + Tools**
