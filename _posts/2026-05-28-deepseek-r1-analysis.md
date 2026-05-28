---
title: "DeepSeek-R1 全解析：纯 RL 如何让模型学会自我反思"
date: 2026-05-28
tags: [DeepSeek, Reasoning, GRPO, RL, Paper]
summary: "DeepSeek-R1 证明了推理能力可以通过纯强化学习涌现——不做任何 SFT，直接对基座模型做 GRPO + rule-based reward，模型自发学会了自我验证、反思和顿悟。这是 2025 年最重要的 AI 论文之一，它的方法正在被 Qwen3、MiniMax-M1、Kimi K2 广泛采用。"
---

## 概述

**论文**: DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning  
**arXiv**: [2501.12948](https://arxiv.org/abs/2501.12948)  
**基座**: DeepSeek-V3-Base (671B MoE, 37B activated)  
**发布时间**: 2025 年 1 月  

DeepSeek-R1 是 2025 年最重要的大模型论文之一。它的核心发现——**推理能力可以通过纯强化学习涌现，不需要任何人类推理数据的 SFT**——改变了大模型对齐的研究方向。

这篇分析将分为四个部分：R1-Zero 的惊人实验、R1 的完整训练流程、Aha Moment 的机制分析、以及这篇论文如何影响了整个行业。

---

## Part 1: R1-Zero — 纯 RL 的首次胜利

### 实验设计

直接从 DeepSeek-V3-Base（一个只做了预训练、从未见过指令数据的模型）出发，用 **GRPO** + **规则化奖励** 进行 RL 训练：

```python
# R1-Zero 的训练循环
for prompt in reasoning_tasks:
    # 对同一 prompt 采样 4 个回答 (G=4)
    responses = [model.generate(prompt) for _ in range(4)]
    
    # 规则化奖励（没有神经网络 RM！）
    rewards = []
    for r in responses:
        accuracy = 1.0 if check_answer(r, ground_truth) else 0.0
        format_ok = 1.0 if has_correct_format(r) else 0.0
        rewards.append(accuracy + format_ok)
    
    # GRPO advantage: 组内归一化
    mean_r = sum(rewards) / 4
    std_r = (sum((r - mean_r)**2 for r in rewards) / 4) ** 0.5
    advantages = [(r - mean_r) / std_r for r in rewards]
    
    # 更新模型
    model.update(responses, advantages)
```

**关键设计选择：**
- 没有 Reward Model → 完全避免了 reward hacking
- 没有 SFT 预热 → 模型从零学习推理格式
- G=4 的组大小 → 在训练效率和 advantage 质量间平衡
- 规则化奖励 → 只检查答案对错和格式，不评价推理过程

### 涌现的行为

纯 RL 训练中，R1-Zero 自发出现了三种**未经训练、完全涌现**的行为：

**1. Self-Verification（自我验证）**
```
模型输出:
  ...所以答案是 42。
  等等，让我检查一下...
  42 确实满足 x² - 84x + 1764 = 0...
  验证通过。最终答案是 42。
```
这不是 SFT 教出来的——模型自己发现"验证可以降低错误率 → 更高的平均 reward → 被 GRPO 强化"。

**2. Reflection（反思）**
```
模型输出:
  我之前的推理第 3 步有问题...
  重新来: 从 y = 2x 出发...
```
模型学会了识别自己的错误并修正。这在传统 SFT 中几乎不可能发生——SFT 只会复制 ground-truth 推理过程，不会复制"犯错然后修正"的过程。

**3. Alternative Approaches（替代方案探索）**
```
模型输出:
  也许有更简单的方法...
  直接用公式 x = ... 代入...
```
模型学会了在推理中探索不同的解题路径。

### 量化结果

R1-Zero 在 AIME 2024 上达到 **pass@1 = 71.0%**，接近 OpenAI o1-0912。对于一个**从未见过推理数据**的模型来说，这是革命性的。

---

## Part 2: R1 的完整训练流程

R1-Zero 发现了推理能力，但它有一个严重问题：**可读性极差**。输出中英混杂、格式混乱、推理过程对读者不友好。

R1 的完整流程解决了这个问题：

### Stage 1: Cold Start SFT

用几千条高质量的、人工编写的推理链数据做 SFT。这不是教模型"如何推理"——R1-Zero 已经会了。这是教模型"如何把推理过程写得好看"。

### Stage 2: Reasoning RL (GRPO)

在 Cold Start SFT 之后，继续 GRPO 训练。这次有了好的格式基础，RL 可以专注于提升推理质量而不是从零学习格式。

### Stage 3: Rejection Sampling + SFT

从 Stage 2 的模型采样 ~600K 个推理链，只保留正确的（rejection sampling）。然后：
- 加入非推理任务数据（写作、翻译、对话）
- 做 SFT 训练
- 恢复通用能力（纯推理训练可能导致 forgetting）

### Stage 4: 全领域 RL

混合奖励的 GRPO 训练：
- 推理任务：规则化奖励（答案正确 + 格式）
- 通用任务：神经 RM（helpfulness + harmlessness）

最终得到可读性好、推理能力强、通用能力不丢失的 R1。

### AIME 2024 最终成绩

| 模型 | pass@1 |
|------|--------|
| DeepSeek-R1 (671B) | **79.8%** |
| OpenAI o1-0912 | ≈80% |
| R1-Distill-Qwen-32B | 72.6% |
| R1-Distill-Llama-70B | 70.0% |
| R1-Distill-Qwen-14B | 69.7% |

---

## Part 3: Aha Moment 的机制分析

R1 论文中引用了一个训练过程中的「Aha Moment」：

```
User: 如果 a>1，那么 √(a+√(a-2√(a-1))) 的平方根是多少？

模型输出:
  wait, wait. Wait. 等等...
  我意识到这里可以用配方方法...
  (开始重新推理)
  √(a+√(a-2√(a-1))) = √(√(a-1)² + ...)
  等等，这样不对。让我重新来...
  (最终给出正确答案)
```

### 为什么会发生？

从 RL 的角度来看，Aha Moment 的机制是这样的：

1. 模型最初会给一个错误的快速答案 → 得到 0 reward
2. 在某些采样中，模型意外地在中间插入了自我检查 → 检查后给出了正确答案 → 得到 1 reward
3. GRPO 的 group-relative advantage 让"有自我检查的轨迹"获得正 advantage
4. 模型学会：自我检查 → 更高的正确率 → 更高的 expected reward
5. 反复强化后，自我检查成为一种稳定的行为模式

**关键洞察**：规则化奖励只检查**最终答案**的正确性。它不检查推理过程。但模型自己发现——更好的推理过程 → 更高的答案正确率。这个元认知不是被奖励函数设计的，是 RL 搜索空间中的 emergent property。

### 为什么 SFT 做不到这一点？

SFT 是模仿学习：模型学习复制人类标注的推理链。但人类标注的推理链通常是"干净"的——它们不包含错误、修正和反思。所以 SFT 训练的模型只会生成干净的推理链，永远不会学到"发现自己的错误然后修正"。

RL 不同：模型在 trial-and-error 中学习。错误的推理链 → 低 reward → 负 advantage → 被抑制。包含自我修正的推理链 → 高 reward → 正 advantage → 被强化。这就是为什么 R1-Zero 能从纯 RL 中涌现出反思行为。

### 论文原话

> "This moment is not just an 'aha moment' for the model, but also for the researchers observing its behavior."

这确实是整篇论文最动人的一句话——研究者自己也被模型的涌现行为震惊了。

---

## Part 4: 蒸馏的惊人效果

R1 还发布了蒸馏版本——用 R1 的推理链做 SFT，训练小模型。结果令人震惊：

| 模型 | 参数量 | AIME 2024 |
|------|--------|-----------|
| DeepSeek-R1 | 671B | 79.8% |
| R1-Distill-Qwen-32B | 32B | **72.6%** |
| R1-Distill-Qwen-14B | 14B | **69.7%** |
| R1-Distill-Qwen-7B | 7B | 55.5% |
| GPT-4o | — | ~50% (estimated) |

32B 蒸馏模型以 **1/20 的参数**达到 R1 的 91% 推理能力。这证明了：
- 推理能力可以被蒸馏——用大模型生成数据训练小模型是高效的
- SFT 蒸馏比 KL 蒸馏更有效——直接复制推理链比匹配概率分布更好
- 推理链的质量比数量更重要

---

## Part 5: 对行业的影响

### 1. GRPO 成为推理训练的标准

R1 之后，GRPO 被广泛采用：
- **Qwen3**: 使用 GRPO + DPO 的混合 RL
- **MiniMax-M1**: 用 CISPO（GRPO 的变体）训练 456B 模型
- **Kimi K2**: 联合 RL 阶段使用了类似的规则化奖励

### 2. 规则化奖励的复兴

R1 证明了：对于有明确 ground truth 的任务（数学、代码），规则化奖励**优于**神经 Reward Model。这改变了 RLHF 的方向——未来的对齐训练可能会区分「可验证任务」（用规则奖励）和「偏好任务」（用神经 RM）。

### 3. 蒸馏成为开源模型的标配

R1 的蒸馏结果表明：小模型可以通过大模型的推理数据获得远超其参数量的推理能力。这催生了 QwQ-32B、Kimi K2.5 蒸馏版等模型——厂商开始把「可蒸馏性」作为模型设计的目标之一。

### 4. 「纯 RL 预训练」的研究方向

R1-Zero 成功后，出现了一批探索「不做 SFT 直接 RL」的论文。QwQ 的 3B 实验（arXiv 2503.09512）直接用 RL 训练一个小模型玩 Countdown Game，也观察到了 emergent reasoning——但同时也发现 Aha Moment 不保证正确。

---

## 局限与思考

### 论文没有解决的问题

1. **可读性问题**: R1-Zero 的输出质量不可接受，需要 Cold Start SFT + 多阶段训练来补救。纯 RL 能否生成可读的推理？还是必须结合 SFT？

2. **通用任务的对齐**: GRPO + 规则化奖励在数学和代码上效果惊人，但写作、翻译、对话等开放任务仍然需要神经 RM。两种奖励如何融合？

3. **Aha Moment 的可控性**: 涌现的反思行为不能被显式控制。模型可能学会「假装反思」（生成看起来很牛的中间步骤但不改变最终答案）——如何验证反思是真实的？

4. **推理预算的分配**: R1 总是进行完整的推理链。是否可以根据任务难度动态分配推理深度？Qwen3 的 Thinking Budget 是对这个问题的回应。

### 我的思考

R1 这篇论文最重要的贡献不是具体的数字——这些数字很快会被超越。它的贡献是**改变了我们对推理能力来源的理解**：

- 传统观点：推理能力需要通过 SFT 从人类数据中学习
- R1 的观点：推理能力是 RL 搜索空间中 emergent 的——只要奖励函数定义清楚，模型会自己找到最优的推理策略

这个观点的影响远超 DeepSeek 本身。它意味着——在足够强的基座模型上，**任何**可以精确定义 reward 的任务，都可以通过 RL 来优化。RL 不再是 SFT 之后的「微调」，而是**能力和行为的第一来源**。

---

## 相关论文

- DeepSeekMath (arXiv 2402.03300): GRPO 的起源论文
- DeepSeek-V3 (arXiv 2412.19437): R1 的基座模型
- MiniMax-M1 (arXiv 2506.13585): CISPO — GRPO 的效率改进
- QwQ-32B (arXiv 2503.09512): 小模型的纯 RL 推理实验
- Qwen3 (arXiv 2505.09388): Hybrid Thinking — 将推理模式整合进单个模型
