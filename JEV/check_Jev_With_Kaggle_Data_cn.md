- https://www.youtube.com/watch?v=Ptwhkqut2Q0&t=635s
  
这个测试比前一个博主单纯介绍 Jev 更有价值，因为他真正拿了**有人工标注 Ground Truth 的公开数据集**来测，而且还加入了通用 LLM 作为对照。核心结论其实非常清楚：

> **Jev 并没有让分类变得更准确；它真正的优势是：在接近通用 LLM 准确率的情况下，更快、更便宜，而且它的 Confidence 很适合用来做“自动处理 / 升级处理”的门控。**

## 1. 他到底怎么测试的？

他主要想回答三个问题：

1. **Jev 到底准不准？**
2. **Jev 给出的 Probability / Confidence 能不能信？**
3. **如果换成便宜的通用 LLM，效果是不是也差不多？**

他最终选择了两个有人工 Ground Truth 的公开数据集，总共进行了 **662 个样本测试**，并用 **Claude Haiku 4.5** 对相同数据、相同问题做对照。

### 两个测试数据集

| Dataset | 任务 | 测试数量 | 目的 |
|---|---|---:|---|
| **SMS Spam Collection** | 判断短信是不是 Spam | 200 | 简单二分类 / Smoke Test |
| **Banking77** | 从 77 个银行业务 Intent 中选一个 | 462 | 测复杂多分类能力 |

Banking77 特别有意思，因为它不是简单的 4 选 1，而是：

> **77选1**

因此非常适合测试 Jev 的 Choice 能力。

---

# 2. 最重要的结果：准确率

### SMS Spam

Jev：

**193 / 200 = 96.5%**

另外：

- AUC = **0.993**
- 27 条真正 Spam 中命中 **24 条**
- False Positive = **4 条**

Claude Haiku 4.5 在这个测试中的总体准确率是：

**93.5%**

所以简单 Spam 分类中，**Jev 略好于 Haiku**。

不过经典专用模型在这个老数据集上可以达到 **97%+**，所以 Jev 并没有创造新的准确率纪录。

---

### Banking77：77选1

这个结果更有代表性：

| Model | Correct | Accuracy |
|---|---:|---:|
| **Jev** | 377 / 462 | **81.6%** |
| **Claude Haiku 4.5** | 370 / 462 | **80.1%** |
| Fine-tuned BERT（论文基准） | — | **~93.7%** |

所以非常有意思：

> **Jev ≈ 便宜的通用 LLM，但明显低于专门 Fine-tuned 的分类模型。**

这基本回答了他的第一个问题。

### 结论 ①

**Jev 并不是一个“更准确”的分类器。**

它与便宜的通用 LLM 大致处于同一档次，在他的实验中略好一点；但与针对具体任务 Fine-tune 的模型相比，还有大约十几个百分点的差距。

---

# 3. 更有意思的是：两个模型经常错在同一个地方

Jev 在 Banking77 上错了 **85 条**。

很多错误集中在非常相似的 Intent，例如：

- 如何获得实体卡 vs 卡什么时候到
- 为什么需要身份验证 vs 如何进行身份验证
- 不同但语义非常接近的银行转账 Intent

而且他发现一个很重要的现象：

> **Claude Haiku 的错误中，有很大一部分和 Jev 错的是同一批样本。**

这意味着一部分问题未必是模型“不聪明”，而是：

**Dataset 本身就存在 Ambiguity / Label Noise。**

他甚至发现一些 Jev 给出 **1.0 probability**、但按照 Ground Truth 算错的案例，人工重新看之后，会觉得 Ground Truth 本身也未必非常合理。

所以他得到另一个很实用的经验：

> **评估 Judgment Model 时，Ground Truth 的标注质量极其重要。**

---

# 4. Probability 到底能不能相信？

这是这次实验中我觉得**最值得关注的部分**。

他的定义很简单：

如果 Jev 说：

> “我有 80% probability。”

那么把所有类似“80%”的样本放在一起，理论上应该大约有 80% 真的是正确的。

这就是在看 **Probability Calibration**。

Banking77 大概得到：

| Jev 报告概率 | 实际正确率 |
|---:|---:|
| ~55% | ~55% |
| ~66% | ~52% |
| ~75% | ~65% |
| ~85% | ~74% |

因此：

### **50%–85% 区间，大致有参考价值，但总体略微过度自信。**

真正的问题出现在非常高的概率区间。

330 个样本最高 probability > **0.9**：

- 平均报告 probability：约 **0.99**
- 实际 accuracy：约 **91%**

更加有意思的是：

### Jev 给出 `1.00` 的样本

大约 **45% 的样本直接给了 1.00**。

但这些所谓“100%确定”的结果，实际上：

**Accuracy = 97.6%**

也就是说：

> **Jev 说 100% 并不是真的 100%。大约每 40 个“100%确定”的判断里，仍然可能错 1 个。**

因此：

### 结论 ②

**Jev 的 Probability 有参考价值，但高概率区域存在 Overconfidence。**

尤其程序里千万不能写成：

```text
probability == 1.0
→ absolutely correct
```

---

# 5. Confidence 反而比 Probability 更有价值

这个结果可能是整个实验最重要的发现。

Jev 除了每个 Choice 的 probability，还会提供一个 **Confidence**。

官方推荐的一种方式就是：

> **High Confidence → 自动处理**  
> **Low Confidence → 人工 / 更强模型处理**

博主于是直接测试这个策略。

他把 462 条 Banking77 按 Confidence 从高到低排列。

### 只接受 Confidence 最高的 50%

准确率竟然达到：

> **~97%**

也就是说：

```text
462 requests
      ↓
Jev
      ↓
High Confidence ──→ Auto Process
      │
Low Confidence  ──→ Human / Strong LLM
```

这就非常有实际意义了。

---

# 6. 用 Confidence Threshold 做 Gate

他进一步测试不同阈值。

### Confidence ≥ 0.95

可以自动处理大约 **60%+ 的请求**，这部分：

> **Accuracy ≈ 95.5%**

剩下约 40%：

→ Human Review  
或者  
→ Larger / Stronger LLM

### Confidence ≥ 0.8

覆盖大约：

**79% requests**

准确率下降到：

**89.9%**

### Confidence < 0.5

17 条里面：

**只有 3 条正确。**

这说明 Confidence 确实包含非常明显的信息：

> **Low Confidence = High Error Risk**

因此他的结论是：

### 结论 ③

**Confidence-based gating 不但能用，而且可能是 Jev 最重要的实际使用方式之一。**

这其实比简单地看 probability 更有工程价值。

---

# 7. Spam 测试也验证了 Probability 的价值

Spam 数据的概率分布非常“干净”。

超过一半的短信：

**Spam Probability < 0.05**

这些里面：

**0 条是真正 Spam。**

而：

**Spam Probability > 0.9**

的 17 条：

**全部都是 Spam。**

真正模糊的只是中间很小一部分。

而漏掉的三条 Spam，Jev 给出的概率分别大约：

**0.24 / 0.44 / 0.46**

也就是说，它虽然最终分类错了，但也没有非常自信地说：

> “这肯定不是 Spam。”

这说明对于这种边界比较清楚的任务，Probability 的实际使用价值相当不错。

---

# 8. 速度：Jev 大约快 3 倍

端到端海外网络测试：

| | Jev | Claude Haiku 4.5 |
|---|---:|---:|
| Median Latency | **432 ms** | **1.4 s** |
| P95 | **658 ms** | **2.1 s** |

因此在他的实际 API 测试环境中：

> **Jev 大约快 3 倍以上。**

这和前一个博主所强调的“快”方向是一致的。

---

# 9. 成本差距非常明显

Banking77 每次请求大约有 **1,900 input tokens**，主要是因为需要把 77 个候选 Intent 全部传进去。

462 次 Jev 调用：

> **总成本不到 $0.04**

而与 Claude Haiku 4.5 相比，他估算的成本差距接近：

> **~30×**

对于 SMS 这种输入非常短的场景，Jev 的成本更低：

> **约 $0.013 / 1,000 requests**

所以它特别适合：

**大量、高频、短输入的 Judgment。**

---

# 10. 一致性也不错，但不是完全 deterministic

他还拿 50 个样本连续跑了三次：

**49 / 50 每次选择相同。**

Probability 最大偏差大约：

**0.09**

所以 Jev：

> **不是完全 deterministic，但稳定性相当不错。**

---

# 最终结论

这个博主最后实际上得出了一个非常清晰的 Jev 定位：

### **① Accuracy：不错，但没有奇迹**

Jev ≈ 廉价通用 LLM。

简单任务可以达到很高准确率；复杂 77 分类大约 **81.6%**。

但专门 Fine-tuned 模型仍然可以达到 **~93%+**。

所以：

> **Don't use Jev because you expect better classification accuracy.**

---

### **② Probability：能用，但高概率要打折**

中间概率区间有一定 calibration。

但：

> **1.00 ≠ 100% correct**

在他的测试里：

**1.00 → 实际约 97.6%**

因此不能把 probability 当成绝对真值。

---

### **③ Confidence：非常有用**

这可能是最大的发现。

例如：

> **Confidence ≥ 0.95 → 自动处理 → ~95.5% accuracy**

其余：

> **Human / Strong LLM**

因此 Jev 最有价值的架构可能并不是：

```text
Request
   ↓
Jev
   ↓
Final Decision
```

而是：

```text
                 ┌─ High Confidence → Auto Process
Request → Jev ───┤
                 └─ Low Confidence  → Strong LLM / Human
```

---

## 我认为这次实验对我们最大的启发

它实际上很好地验证了我们前面讨论的 **Judgment Layer** 架构：

```text
                         ┌─ Simple Action
                         ├─ Tool / MCP
Input → State → Jev ─────┤
                         ├─ Strong LLM
                         └─ Human Review
                              ↑
                       based on Confidence
```

所以 Jev 真正的价值并不是：

> **“它比大模型判断得更准。”**

而是：

> **“它能够非常便宜、非常快地处理绝大多数容易判断的问题，并且通过 Confidence 把自己不确定的问题识别出来，再升级给昂贵模型或人工。”**

这其实比单纯的 **81.6% accuracy** 更重要。对于我们前面讨论的 **Tool/MCP routing、天气 State → Decision、以及大量实时判断**，这个结果尤其有意义。
