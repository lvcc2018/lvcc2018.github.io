---
title: "Qwen 全史：从 2023 到 2026 —— 一部关于广度和开放的技术进化论"
date: 2026-05-28
tags: [Qwen, Alibaba, Survey, MoE, Architecture, Agent, Reasoning]
summary: "一篇覆盖阿里通义 Qwen 系列全部模型与技术报告的深度综述。从 2023 年 9 月的第一个 Qwen 到 2026 年 4 月的 Qwen3.6，我们追溯 GQA、长上下文、MoE 双线、QwQ 推理探索、Hybrid Thinking、极致激活稀疏等每一项创新的演进脉络，展示为什么「广度」和「开放」是理解 Qwen 技术路线的两个关键词。"
---

## 引言：理解 Qwen 的两个关键词

2023 年 9 月，阿里通义实验室发布了第一个 Qwen 模型。三年后，Qwen 系列已经发展为一个涵盖 0.5B 到 235B 参数、Dense 和 MoE 双架构、文本、视觉、音频全模态、支持 119 种语言的**全栈开源大模型生态**。

如果 DeepSeek 的关键词是「效率」，Qwen 的关键词有两个：**广度（Breadth）** 和 **开放（Openness）**。

- **广度**：Qwen 不做单一模型的极致优化，而是覆盖全参数谱系、全架构类型、全模态、全语言。从 0.5B 的端侧模型到 235B 的旗舰 MoE，Qwen 都有。
- **开放**：Qwen 是唯一全系列使用 **Apache 2.0** 许可的头部模型——这是对商业使用最友好的开源协议。相比之下，DeepSeek 使用 MIT（允许但专利条款模糊），LLaMA 使用自定义限制性许可。

本文将按照时间线，逐一拆解 Qwen 系列的每一次演进，展示这个「全栈开源帝国」是如何建成的。

---

## 第一章：起航（2023.09 – 2024.02）

### Qwen 1.0：从零开始的基座

2023 年 9 月发布的 Qwen 1.0 是阿里通义实验室的第一个正式大模型。虽然它没有像 Transformer 或 GPT-3 那样引发轰动，但它确立了 Qwen 系列的三个基本方向：

1. **中英双语优先**：原生中英双语训练，而非在英文模型上做中文适配
2. **全参数谱系**：从对话模型到基座模型，从 1.8B 到 72B 的完整覆盖
3. **开源承诺**：首批模型即以开放许可发布

Qwen 1.0 的训练数据来自经过严格过滤的网络语料、书籍、学术论文和代码，总训练 token 量约 3T——以当时的标准已经相当大。但在性能上，它并未超越同期最强的开源模型（如 LLaMA 2）。

### Qwen1.5：质量的质变

2024 年 2 月发布的 Qwen1.5 是 Qwen 系列的第一次重要迭代。这次升级的重点不是规模，而是**数据质量**。

Qwen1.5 重新设计了数据处理管线：
- 更强的质量过滤：KenLM 困惑度 + fastText 分类器 + 启发式规则
- 更好的去重策略：MinHash + LSH 近似去重
- 更精细的数据分类：为不同领域分配不同的训练权重

结果是：同等参数规模下，Qwen1.5 在大多数 benchmark 上超越 LLaMA 2。这证明了 Qwen 的核心方法论——**数据质量 > 模型规模**——这条路是走得通的。

Qwen1.5 还首次引入了 **MoE 版本**（Qwen1.5-MoE-A2.7B，14.3B total / 2.7B active），为后来的 MoE 全面铺开做了一次早期验证。

---

## 第二章：规模化与多语言（2024.06 – 2024.09）

### Qwen2：GQA、长上下文、多语言

arXiv 2407.10671。Qwen2 是 Qwen 系列的第一个重大架构升级。它带来了三个关键变化：

**1. GQA 架构**

Qwen2 从 MHA（Multi-Head Attention）切换到 **GQA（Grouped-Query Attention）**——这是 LLaMA 2/3 也采用的策略。GQA 将 Q heads 分成 G 组，每组共享一个 KV head：

```
MHA: 每个 Q head 有独立的 KV → H 组 KV Cache
GQA: G 个 Q heads 共享一个 KV → G 组 KV Cache (G < H)
```

对于 72B 模型（H=64），GQA（G=8）将 KV Cache 从 64 组压缩到 8 组——压缩 8×。这使得 Qwen2 能够原生支持更长的上下文。

**2. 128K 原生上下文**

这是 Qwen2 相比同期开源模型最大的差异化优势之一。LLaMA 2 在 2024 年仍只支持 4K 原生上下文，需要 NTK/YaRN 等外推技术才能扩展到更长的长度。Qwen2 通过以下组合原生支持 128K：

- GQA（减少 KV Cache）
- 大 RoPE base（1,000,000，是 LLaMA 2 的 100 倍）
- 长文本数据的专项训练

**3. 30 种语言**

Qwen2 将语言支持从最初的中英双语扩展到约 30 种语言，包括西班牙语、法语、德语、阿拉伯语、俄语、韩语、日语、泰语、越南语等。这为 Qwen3 后来扩展到 119 种语言打下了基础。

**旗舰性能：Qwen2-72B**

| Benchmark | Base | Instruct |
|-----------|------|----------|
| MMLU | 84.2 | — |
| HumanEval | 64.6 | — |
| GSM8K | 89.5 | — |
| MT-Bench | — | 9.1 |
| Arena-Hard | — | 48.1 |

在 2024 年年中，Qwen2-72B 是开源模型中综合能力最强的之一，在多项 benchmark 上超越 LLaMA 2-70B。

### Qwen2.5：数据规模的质变

arXiv 2412.15115。Qwen2.5 表面上是 Qwen2 的迭代版本，但数据规模的变化是**数量级的**：

| 维度 | Qwen2 | Qwen2.5 |
|------|-------|---------|
| **预训练 token** | 7T | **18T** |
| **SFT 样本** | 数十万 | **>100 万** |
| RL 训练 | 基础 | **多阶段 RL** |

**预训练数据的质变**

Qwen2.5 将预训练数据从 7T 扩展到 18T——这不仅仅是量的增加，更是质的提升：
- 更好的数据筛选管线
- 更高的知识密度
- 更广的知识覆盖（STEM、推理、代码占比提升）

**后训练的质变**

Qwen2.5 的后训练做了三个关键改进：
1. **大规模 SFT**：超过 100 万高质量指令样本
2. **多阶段 RL**：不是单次 RLHF，而是多轮迭代
3. **专项能力提升**：结构化数据分析、长文本生成、指令遵循

**产品矩阵**

Qwen2.5 提供了比 Qwen2 更丰富的产品线：

| 类型 | 模型 | 许可 |
|------|------|------|
| 开源 Dense | 0.5B ~ 72B | Apache 2.0 |
| 开源量化 | GPTQ/AWQ/GGUF | Apache 2.0 |
| 商业 MoE | Qwen2.5-Turbo, Qwen2.5-Plus | — |

Qwen2.5 还发布了一系列专项模型：
- **Qwen2.5-Coder**：代码生成专项
- **Qwen2.5-Math**：数学推理专项
- **Qwen2.5-VL**：视觉-语言多模态
- **Qwen2.5-Omni**：全模态（文本+视觉+音频）

---

## 第三章：推理探索（2025.01 – 2025.03）

### QwQ：对推理模型的第一次探索

arXiv 2503.09512。在 DeepSeek-R1（2025 年 1 月）以纯 RL 推理震惊业界后，阿里迅速推出了自己的推理模型探索——QwQ。

QwQ 的实验规模远小于 R1（3B vs 671B），但它在方法论上的贡献是独特的。这篇论文的核心发现：

1. **RL-only 训练可以提升小模型的推理**：3B 模型通过纯 RL 训练在 5 个 benchmark 中 4 个超越 baseline
2. **Response length 与推理质量不相关**：更长的回答不一定更好——QwQ 实验中发现回答长度与正确率之间没有显著相关性
3. **Aha moment 涌现但不保证正确**：模型确实学会了自我反思和修正，但这不总是导致正确答案

**QwQ 论文原话**：

> "Inspired by the success of DeepSeek R1 in reasoning via reinforcement learning without human feedback, we train a 3B language model using the Countdown Game with pure reinforcement learning."

QwQ 的重要贡献在于：它证明了 **R1 的方法（纯 RL 推理）不是大模型独有的**——即使是 3B 的小模型，也可以通过 RL 涌现出推理行为。

**与 DeepSeek-R1 的方法对比**：

| 维度 | DeepSeek-R1 | QwQ-3B |
|------|-----------|--------|
| 模型规模 | 671B | 3B |
| 训练方法 | GRPO (G=4) | 纯 RL |
| 任务 | 数学 + 代码 + STEM | Countdown Game |
| 涌现 | Self-reflection, Aha moment | Aha moment（但不保证成功） |
| 核心贡献 | 大规模验证纯 RL 推理 | 小规模验证 RL 推理的普适性 |

QwQ 后来发展出了更大规模的版本（QwQ-32B），对开源生态产生了重要影响——特别是作为 R1 蒸馏的基座模型之一。

---

## 第四章：Hybrid Thinking —— 一个模型，两种思维（2025.04）

### Qwen3：标志性的架构创新

arXiv 2505.09388。Qwen3 是 Qwen 系列的第一个标志性创新——**Hybrid Thinking**。它解决了一个困扰行业的核心问题：**如何在一个模型中同时拥有快速响应的聊天能力和深度思考的推理能力？**

### 两模型问题的终结

在 Qwen3 之前，如果你想要两种能力，你需要两个模型：

| | Chat 模型 | Reasoning 模型 |
|---|---|---|
| 响应 | 快（< 500ms） | 慢（秒到分钟） |
| 适用 | 对话、翻译、简单问答 | 数学、代码、复杂推理 |
| 代表 | Qwen2.5-Instruct | QwQ-32B, DeepSeek-R1 |
| 部署成本 | 1× | 额外的 1× |

Qwen3 通过 Hybrid Thinking 彻底解决了这个问题：**一个模型，通过 chat template 控制两种模式。**

```
# Non-Thinking 模式
response = model.chat("法国的首都是什么？", enable_thinking=False)
→ "巴黎。"

# Thinking 模式
response = model.chat("证明根号2是无理数", enable_thinking=True)
→ <think>假设根号2 = p/q...矛盾</think>
  <response>因此，根号2是无理数。</response>
```

模式切换不是通过架构实现的——是训练数据教会模型根据 query 复杂度自主决定的。这种设计比 DeepSeek-V4 的双模型方案（V4-Flash + V4-Pro）更优雅——只有一个模型，省掉了一个部署实例。

### Thinking Budget：推理深度的可控旋钮

Qwen3 引入了 **thinking budget**——一个控制推理深度的显式参数：

```python
# 低 budget：快速、浅层推理
response = model.chat(question, thinking_budget=100)

# 高 budget：深度、全面推理
response = model.chat(question, thinking_budget=2000)
```

Thinking budget 控制模型在生成最终答案之前可以使用的 thinking token 数量。这与 temperature 或 top-p 不同——它控制的是推理深度，不是输出的随机性。

实践意义：
- 客服场景：budget=100，快速响应
- 代码审查：budget=2000，深入分析
- 安全审计：budget=5000，详尽检查

### 全参数谱系 + 119 种语言

| 架构 | 模型 | 参数量 |
|------|------|--------|
| Dense | Qwen3-0.6B ~ 32B | 0.6B ~ 32B |
| **MoE** | Qwen3-30B-A3B | 30B total / 3B active |
| **MoE** | **Qwen3-235B-A22B** | **235B total / 22B active** |

Qwen3 同时支持 Dense 和 MoE 两种架构，从 0.6B 到 235B。语言支持从 Qwen2 的 30 种扩展到 **119 种**——对于全球部署场景，这是实质性的竞争力。

### Qwen3-2507：快速迭代

2025 年 7 月发布的迭代版本 Qwen3-2507：
- Qwen3-Instruct-2507
- Qwen3-Thinking-2507
- 改进的指令遵循和推理能力

这是 Qwen 系列「月更」节奏的体现。

---

## 第五章：极致效率与 Agent 时代（2026.04）

### Qwen3.6：MoE 效率的极致

2026 年 4 月 16 日发布。Qwen3.6 系列将 MoE 的效率推到极致：

| 模型 | 总参数 | 激活参数 | 激活比 |
|------|--------|---------|--------|
| Qwen3.6-35B-A3B (Flash) | 35B | **3B** | 8.6% |
| Qwen3.6-Plus | 未公开 | — | — |

**Qwen3.6-35B-A3B 的特点**：

- 35B 的知识广度 + 3B 的推理速度
- 可以运行在消费级 GPU 上（3B active ≈ 6GB FP16）
- Agentic Coding 显著增强
- 空间定位与逻辑推理能力更精准

这是「小激活、大知识」理念的极致体现——与 Kimi K2 的 1T/32B（3.2% 激活比）形成对比：
- Kimi K2: 更大绝对知识容量（1T total），更低激活比（3.2%）
- Qwen3.6-Flash: 更极致的部署友好度（3B active），可单卡运行

---

## 第六章：技术统合与行业定位

### Qwen 的技术演进全景

```
Qwen 1.0 (2023.09)   — 基座 + 中英双语 + 全参数谱系
    ↓
Qwen1.5 (2024.02)    — 数据质量 > 规模 + 首次 MoE 尝试
    ↓
Qwen2 (2024.06)      — GQA + 128K 上下文 + 30 语言
    ↓
Qwen2.5 (2024.09)    — 18T tokens + 100 万 SFT + 全模态矩阵
    ↓
QwQ (2025.03)        — 推理探索：RL-only 的普适性
    ↓
Qwen3 (2025.04)      — Hybrid Thinking + Thinking Budget + 119 语言
    ↓
Qwen3-2507 (2025.07) — 快速迭代
    ↓
Qwen3.6 (2026.04)    — MoE 极致效率 (35B/3B) + Agentic Coding
```

### Qwen vs DeepSeek：两种哲学

| 维度 | Qwen (阿里) | DeepSeek |
|------|-----------|----------|
| **核心理念** | 广度 + 开放 | 效率 + 极致 |
| **架构路线** | Dense + MoE 双线 | MoE 单线（从 V2 开始） |
| **注意力机制** | GQA (保守) | **MLA (激进，自研)** |
| **推理创新** | Hybrid Thinking (模式切换) | GRPO + 纯 RL |
| **开源许可** | **Apache 2.0** (最宽松) | MIT |
| **模型覆盖** | 0.5B ~ 235B，全谱系 | 聚焦大模型（~670B+） |
| **多模态** | Qwen-VL, Qwen-Omni | V4 才开始识图 |
| **Agent 生态** | Qwen-Agent 框架 | 非重点（API 为主） |
| **训练成本** | 未公开（推测高于 V3） | **$5.6M (V3)** |
| **商业策略** | 阿里云百炼平台 | API 服务 |

**关键差异：**

DeepSeek 选择了「深度优先」——在 MoE 一条路上做到极致，MLA 就是这种思维的代表。Qwen 选择了「广度优先」——覆盖所有场景、所有参数规模、所有模态，Apache 2.0 许可最大化生态渗透。

DeepSeek 的论文更「性感」——MLA、GRPO、Aha Moment 都是能上头条的突破。Qwen 的贡献更「扎实」——Hybrid Thinking 是工程上的优雅设计，119 种语言、Apache 2.0 许可、全模态覆盖是实实在在的生态价值。

### Qwen 的真正贡献

1. **Apache 2.0 许可**：对商业使用没有任何限制。DeepSeek 的 MIT 有专利条款的模糊地带，LLaMA 有使用限制。Qwen 是唯一一个「零限制」的头部开源模型。

2. **全栈覆盖**：从端侧 0.5B 到旗舰 235B MoE，从纯文本到全模态（VL + Omni），从基座到专项（Coder + Math），从模型到框架（Qwen-Agent）。开发者可以在 Qwen 生态内找到几乎任何需要的组件。

3. **Hybrid Thinking**：虽然不是最「炫」的创新，但它是**最实用的**——一个模型替代两个部署实例，降低了 50% 的部署成本。在生产环境中，这比 benchmark 上的 2% 提升更有价值。

4. **多语言能力**：从 30 种到 119 种语言，Qwen 在多语言覆盖上领先所有开源模型。对于全球化的产品部署——比如跨境电商、多语言客服——这是不可替代的优势。

5. **Qwen-Agent 框架**：阿里将 Agent 开发工具化，提供开箱即用的框架而非仅仅 API。这降低了 Agent 开发的门槛——你不需要自己实现 function calling 和 planning，Qwen-Agent 已经做好了。

---

## 结语：Qwen 教给我们的事

1. **广度也是一种深度**：不是所有创新都需要是「行业首次」。Hybrid Thinking 不是第一个推理模式切换方案（DeepSeek-V4 的 Fast/Expert Mode 更早），但 Qwen3 的实现（单模型 vs 双模型）在工程上更优雅。

2. **开放是长期竞争力**：Apache 2.0 许可意味着 Qwen 可以被任何公司自由使用、修改和商用。这降低了 Qwen 的短期商业变现能力，但建立了长期的生态壁垒——当你的模型成为整个行业的基础设施时，竞争对手很难取代你。

3. **数据质量驱动而非规模驱动**：Qwen1.5 用了更好的数据而非更多的参数，Qwen2.5 用了 18T 经过精细筛选的 token 而非随便堆到 30T。数据的质量 > 数据的量——这是 Qwen 和 DeepSeek 共同验证的教训。

4. **全栈生态 > 单点突破**：Qwen 不只是发布模型——它同时提供 Qwen-Agent 框架、阿里云百炼平台、HuggingFace 完整模型库、量化版本。这种「全栈生态」让开发者可以在 Qwen 体系内完成从选模型到部署的全流程。

5. **主航道之外有蓝海**：当所有人都在追 DeepSeek 的 MLA 和 GRPO 时，Qwen 在 Hybrid Thinking、119 种语言、Apache 2.0 许可这些「不那么性感」的领域建立了独特的竞争力。这些竞争力在 benchmark 上看不出来，但在生产环境中非常真实。

---

## 相关论文

本文涉及的全部 Qwen 技术报告（按时间顺序）：

- Qwen Technical Report (2023.09): 首个 Qwen 模型
- Qwen2 Technical Report (arXiv 2407.10671): GQA, 128K context, 30 语言
- Qwen2.5 Technical Report (arXiv 2412.15115): 18T tokens, 100 万 SFT
- QwQ-32B (arXiv 2503.09512): RL-only 推理探索
- Qwen3 Technical Report (arXiv 2505.09388): Hybrid Thinking, 119 语言
