# Jev Accuracy, Confidence, Cost, and Latency  
## An Independent Benchmark Test

This test is particularly interesting because, instead of simply discussing Jev conceptually, the blogger tested it against **public datasets with human-labeled ground truth** and compared the results with a general-purpose LLM.

The overall conclusion is quite clear:

> **Jev does not necessarily make classification more accurate. Its real advantages are speed, cost, and the ability to use Confidence as a gate for deciding which requests can be handled automatically and which should be escalated.**

---

# 1. What Did He Test?

The experiment was designed to answer three questions:

1. **How accurate is Jev?**
2. **Can Jev's Probability and Confidence values be trusted?**
3. **Would a cheap general-purpose LLM perform about the same?**

He selected two public datasets with human-labeled ground truth and tested a total of **662 samples**.

The same data and questions were also sent to **Claude Haiku 4.5** as a comparison.

### Test Datasets

| Dataset | Task | Samples | Purpose |
|---|---|---:|---|
| **SMS Spam Collection** | Determine whether a message is spam | 200 | Simple binary classification / Smoke Test |
| **Banking77** | Select one banking intent from 77 categories | 462 | Complex multi-class classification |

Banking77 is especially interesting because it is not a simple four-choice problem.

It is:

> **1 choice out of 77 possible intents**

This makes it a useful stress test for Jev's Choice capability.

---

# 2. Accuracy Results

## SMS Spam

Jev correctly classified:

> **193 / 200 = 96.5%**

Additional results:

- AUC: **0.993**
- 27 actual spam messages
- **24 detected**
- **4 false positives**

Claude Haiku 4.5 achieved approximately:

> **93.5% accuracy**

So in this simple spam-classification task, **Jev slightly outperformed Haiku**.

However, traditional specialized models can achieve **97%+** on this relatively old and well-understood dataset.

Therefore, Jev did not establish a new accuracy benchmark.

---

## Banking77: 1-of-77 Classification

The more representative result came from Banking77:

| Model | Correct | Accuracy |
|---|---:|---:|
| **Jev** | 377 / 462 | **81.6%** |
| **Claude Haiku 4.5** | 370 / 462 | **80.1%** |
| Fine-tuned BERT — published benchmark | — | **~93.7%** |

This is an important result:

> **Jev performs roughly at the level of a low-cost general-purpose LLM, but remains significantly below a model specifically fine-tuned for the task.**

### Conclusion #1

**Jev is not a “more accurate” classifier.**

Its performance is roughly comparable to an inexpensive general-purpose LLM and was slightly better in this particular experiment.

However, compared with a specialized fine-tuned model, it still trails by more than ten percentage points.

---

# 3. Jev and the LLM Often Fail on the Same Cases

Jev made **85 errors** on Banking77.

Many of these errors occurred between very similar intents, for example:

- How to obtain a physical card vs. when the card will arrive
- Why identity verification is required vs. how to verify identity
- Similar but distinct bank-transfer intents

Another interesting observation was that many of Claude Haiku's errors occurred on the **same samples that Jev got wrong**.

This suggests that some of the difficulty may come from:

**Ambiguity and label noise in the dataset itself**, rather than simply insufficient model intelligence.

There were even cases where Jev assigned a probability of **1.0** to an answer that was considered incorrect according to the ground-truth label, but where human inspection suggested that the official label itself might be debatable.

This leads to another useful lesson:

> **The quality of ground-truth labeling is extremely important when evaluating a Judgment Model.**

---

# 4. Can Jev's Probability Be Trusted?

This is one of the most interesting parts of the experiment.

The basic idea of probability calibration is simple.

If Jev says:

> **“I am 80% confident in this choice.”**

then among a large number of predictions around 80%, approximately 80% should actually be correct.

For Banking77, the approximate results were:

| Reported Probability | Actual Accuracy |
|---:|---:|
| ~55% | ~55% |
| ~66% | ~52% |
| ~75% | ~65% |
| ~85% | ~74% |

The result suggests:

> **In the 50%–85% range, Jev's probabilities are reasonably informative, although somewhat optimistic overall.**

The larger problem appears at very high probabilities.

Among approximately **330 predictions with probability above 0.9**:

- Average reported probability: approximately **0.99**
- Actual accuracy: approximately **91%**

Even more interesting:

### When Jev Reports `1.00`

Approximately **45% of the samples received a probability of exactly 1.00**.

But their actual accuracy was:

> **97.6%**

In other words:

> **When Jev says “100%,” roughly 1 out of every 40 predictions may still be wrong.**

### Conclusion #2

**Jev's probability is useful, but it becomes overconfident at the high end.**

A system should never interpret:

```text
probability == 1.0
→ absolutely correct
```

as a guarantee.

---

# 5. Confidence May Be More Valuable Than Probability

This may be the most important finding in the entire experiment.

In addition to the probability distribution across choices, Jev returns a **Confidence** value.

One recommended use is:

> **High Confidence → Automatic Processing**  
> **Low Confidence → Human Review / Stronger Model**

The blogger directly tested this idea.

He sorted all 462 Banking77 samples from highest to lowest Confidence.

### Accept Only the Highest-Confidence 50%

If the system automatically accepts only the highest-confidence half of Jev's predictions:

> **Accuracy rises to approximately 97%.**

This immediately suggests a much more useful architecture:

```text
462 Requests
      ↓
     Jev
      ↓
High Confidence ──→ Auto Process
      │
Low Confidence  ──→ Human / Strong LLM
```

---

# 6. Using Confidence as a Gate

The experiment also tested specific Confidence thresholds.

### Confidence ≥ 0.95

The system can automatically process **more than 60% of requests**, with approximately:

> **95.5% accuracy**

The remaining requests can be escalated to:

**Human Review**

or:

**A Larger / Stronger LLM**

---

### Confidence ≥ 0.80

This covers approximately:

> **79% of requests**

with accuracy dropping to:

> **89.9%**

---

### Confidence < 0.50

There were 17 such samples.

Only:

> **3 out of 17 were correct**

This strongly suggests:

> **Low Confidence corresponds to high error risk.**

### Conclusion #3

**Confidence-based gating works and may be one of the most important practical ways to use Jev.**

From an engineering perspective, this may be much more useful than simply looking at the top probability.

---

# 7. The Spam Test Also Shows Useful Probability Separation

The probability distribution in the SMS Spam experiment was particularly clean.

More than half of the messages received:

> **Spam Probability < 0.05**

Among those messages:

> **None were actually spam.**

Meanwhile, all 17 messages with:

> **Spam Probability > 0.90**

were actually spam.

Only a relatively small number of messages fell into the uncertain middle region.

The three spam messages Jev missed had spam probabilities of approximately:

> **0.24 / 0.44 / 0.46**

So even when Jev misclassified them, it did not confidently classify them as non-spam.

For tasks with relatively clear decision boundaries, this suggests that Jev's probability output can be quite useful.

---

# 8. Latency: Jev Was More Than 3× Faster

In the blogger's end-to-end API tests over an overseas network:

| | Jev | Claude Haiku 4.5 |
|---|---:|---:|
| **Median Latency** | **432 ms** | **1.4 s** |
| **P95 Latency** | **658 ms** | **2.1 s** |

In this particular environment:

> **Jev was more than three times faster.**

This supports one of Jev's main value propositions: fast, lightweight decision-making.

---

# 9. Cost Difference Was Significant

Each Banking77 request contained approximately **1,900 input tokens**, largely because all 77 candidate intents and their descriptions had to be included.

The complete set of 462 Jev calls cost:

> **Less than $0.04**

Compared with Claude Haiku 4.5, the blogger estimated the cost difference at approximately:

> **~30×**

For very short inputs such as SMS classification, Jev was even cheaper:

> **Approximately $0.013 per 1,000 requests**

This makes Jev particularly attractive for:

> **High-volume, high-frequency, short-input judgment tasks.**

---

# 10. Consistency Was Also Good

The blogger also tested repeatability.

He selected 50 samples and ran each one three times.

Results:

> **49 out of 50 produced the same selected answer on every run.**

The maximum probability variation was approximately:

> **0.09**

Therefore, Jev is not completely deterministic, but it appears reasonably stable.

---

# Final Conclusions

The experiment provides a much clearer picture of where Jev fits.

## ① Accuracy: Good, but No Miracle

Jev performs approximately at the level of a low-cost general-purpose LLM.

On the more difficult Banking77 test:

> **Jev: 81.6%**  
> **Claude Haiku 4.5: 80.1%**

But a specialized fine-tuned model can reach approximately:

> **93%+**

Therefore:

> **Don't use Jev because you expect better classification accuracy.**

---

## ② Probability: Useful, but Discount Very High Probabilities

Probability is reasonably informative, particularly in the middle range.

However:

> **1.00 does not mean 100% correct.**

In this experiment:

> **Reported 1.00 → Actual accuracy ~97.6%**

Probability should therefore be treated as a statistical signal, not an absolute guarantee.

---

## ③ Confidence: Extremely Useful

This may be the most important result.

For example:

> **Confidence ≥ 0.95 → Auto Process → ~95.5% Accuracy**

Everything else can be escalated to:

> **Human Review / Stronger LLM**

Therefore, the most useful Jev architecture may not be:

```text
Request
   ↓
Jev
   ↓
Final Decision
```

Instead:

```text
                 ┌─ High Confidence → Auto Process
Request → Jev ───┤
                 └─ Low Confidence  → Strong LLM / Human
```

---

# The Most Important Takeaway

This experiment strongly supports the idea of using Jev as a **Judgment Layer** rather than as a replacement for a general-purpose LLM.

A practical architecture could look like:

```text
                         ┌─ Simple Action
                         ├─ Tool / MCP
Input → State → Jev ─────┤
                         ├─ Strong LLM
                         └─ Human Review
                              ↑
                       Based on Confidence
```

The real value of Jev is therefore not:

> **“It makes better decisions than large models.”**

It is:

> **“It can handle a large percentage of easy decisions extremely quickly and cheaply, identify many of the cases where it is uncertain, and escalate those difficult cases to a stronger model or a human.”**

For applications such as **tool/MCP routing, filtering, scoring, high-frequency decisions, and State → Decision systems**, this is potentially much more important than the headline **81.6% accuracy**.
