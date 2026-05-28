---
title: "DeepSeek 全史：从 DeepSeek LLM 到 V4 —— 一部关于极致效率的技术进化论"
date: 2026-05-28
tags: [DeepSeek, Survey, Architecture, MoE, MLA, GRPO, Training]
summary: "一篇覆盖 DeepSeek 全部技术论文的深度综述。从 2024 年初的第一个 LLM 到 2026 年 4 月的 V4，我们追溯 MLA、DeepSeekMoE、FP8 训练、GRPO、DSA、Engram 等每一项创新的演进脉络，解读为什么「效率」是理解 DeepSeek 技术路线唯一正确的关键词。"
---

## 引言：理解 DeepSeek 的唯一关键词

2024 年 1 月，一家名为「深度求索」的中国公司发布了它的第一个大语言模型 DeepSeek LLM。两年后，这家公司以不到 $6M 的训练成本做出了 671B 参数、性能比肩 GPT-4 的 V3，又用纯 RL 训练出了能够自我反思的 R1，最后在 2026 年 4 月发布了支持百万上下文、采用全新 Engram 架构的 V4。

如何理解这两年的技术演进？如果只能用一个词，那就是：**效率（Efficiency）**。

DeepSeek 的每一篇论文、每一项创新，都在回答同一个问题：**如何用更少的资源做到同样的事？** 这个「资源」在不同的论文中指向不同的东西——训练成本、推理显存、KV Cache、RL 训练时间、注意力计算——但核心逻辑从未改变：找到最大的浪费点，用架构创新（而非堆算力）消除它。

本文将按照时间线，逐一拆解 DeepSeek 的每一项关键技术，并展示它们如何构成一个自洽的、层层递进的效率优化体系。

---

## 第一章：奠基（2024.01 – 2024.05）

### DeepSeek LLM：从零开始

DeepSeek LLM（2024 年 1 月）是 DeepSeek 的第一个通用大语言模型。它的意义不在于性能——虽然它确实做到了当时开源模型的先进水平——而在于它确立了 DeepSeek 的技术路线：

- **从零预训练**：不基于任何开源模型进行微调
- **全栈自研**：训练框架、数据处理管线、评估体系全部自建
- **成本意识**：从一开始就关注训练效率，每 FLOP 的产出

这三点成为 DeepSeek 所有后续工作的基础。当其他公司选择基于 LLaMA 微调来快速起步时，DeepSeek 选择了更难但更自主的道路。

### DeepSeekMoE：稀疏激活的早期探索

arXiv 2401.02954 是 DeepSeek 对 MoE（Mixture of Experts）架构的探索性工作。这篇论文提出了两个对后续 V2/V3 至关重要的设计：

**1. 细粒度专家分割（Fine-Grained Expert Segmentation）**

传统的 MoE（如 GShard、Switch Transformer）使用 8-64 个大专家。DeepSeekMoE 提出：**把大专家切成更多小专家**。

```
传统 MoE: 8 个 expert，每个 7B → 总 56B
DeepSeekMoE: 64 个 expert，每个 ~0.9B → 总 ~56B

为什么更好？
- 更多组合可能：64 选 2 = 2016 种组合 vs 8 选 2 = 28 种
- 更精细的专业化：小专家更容易专注于窄领域
- 更灵活的路由：更多选择 → 更可能找到最优专家组合
```

**2. 共享专家隔离（Shared Expert Isolation）**

将所有 token 强制通过的「共享专家」与按 token 路由的「路由专家」分开：

```
每个 MoE 层:
  ├─ Shared Expert × 2 (所有 token 固定通过，处理通用知识)
  └─ Routed Experts × 64 (按 token 路由，处理专业知识)
       └─ Top-K 激活，K 通常为 2-8
```

共享专家学习语言基础、常识等通用知识；路由专家学习数学、代码、特定领域等专业知识。这种设计避免了「所有 token 都需要所有专家」的浪费——通用知识走共享路径，专业知识走路由路径。

---

## 第二章：MLA —— 改变了推理规则的创新（2024.05）

### DeepSeek-V2：236B 参数，21B 激活

DeepSeek-V2（arXiv 2405.04434）是 DeepSeek 技术路线的第一个真正突破。它引入了 **Multi-head Latent Attention（MLA）**——一项完全改变了 LLM 推理成本计算的创新。

### MLA 的核心思想

在标准 Multi-Head Attention 中，推理时需要缓存所有历史 token 的 Key 和 Value 矩阵（KV Cache）。KV Cache 的大小为：

```
KV Cache = 2 × batch × seq_len × num_heads × head_dim × 2 bytes
```

对于 128 个 head、128 维 head_dim 的模型，这是巨大的内存开销。MQA 和 GQA 通过减少 KV head 数量来压缩，但 MLA 走了一条完全不同的路：**将 KV 表示投影到低秩潜空间，只缓存压缩后的潜向量**。

```
标准 MHA:
  K = X @ W_K → 存储完整的 K (d_model → H×d_k)
  V = X @ W_V → 存储完整的 V (d_model → H×d_k)

MLA:
  C = X @ W_DKV → 只存储 C (d_model → d_c, d_c ≪ H×d_k)
  K = C @ W_UK → 需要时从 C 重建 K
  V = C @ W_UV → 需要时从 C 重建 V
```

在 DeepSeek-V2 中，d_c = 512，H×d_k = 128×128 = 16384。**压缩比 ≈ 32×**。

### 解耦 RoPE

RoPE 通过旋转变换编码位置信息，但旋转操作与低秩压缩不兼容——你不能先压缩再旋转，因为旋转和压缩不交换。DeepSeek 的解决方案是将 K 分成两部分：

```
K = [K_content; K_rope]

K_content = C @ W_UK  → 从潜向量重建（不经过 RoPE）
K_rope    = X @ W_KR  → 直接计算 + RoPE（维度很小，~64）
```

位置信息只需要很小的维度来编码。内容信息通过潜向量压缩。两者分离后，总 KV Cache ≈ d_c + d_rope ≈ 512 + 64 = 576，相比 16384 的实际压缩比 ≈ 28×。

### V2 的量化成果

| 指标 | vs DeepSeek 67B |
|------|----------------|
| KV Cache | **-93.3%** |
| 训练成本 | **-42.5%** |
| 推理吞吐 | **+5.76×** |
| 模型质量 | **更好**（MMLU 71.3→78.4） |

V2 同时做到了更便宜、更快、更好——这在 ML 中几乎不可能。

### DeepSeek-Coder-V2：MLA 的首次领域扩展

arXiv 2406.11931。从 V2 的中间 checkpoint 出发，额外训练 6T tokens 的代码和数学数据。支持 338 种编程语言，128K 上下文，性能持平 GPT-4 Turbo。

这验证了「从通用 MoE 出发做领域持续预训练」的可行性——后来被 Kimi K2.5（K2→多模态持续预训练）继承。

---

## 第三章：规模化（2024.02 – 2024.12）

### DeepSeekMath：GRPO 的诞生地

arXiv 2402.03300。这篇论文表面上是一个 7B 的数学推理模型，但它的真正遗产是引入了 **Group Relative Policy Optimization（GRPO）**。

**GRPO 的核心公式：**

```
PPO:
  Advantage = reward + γ × V(next_state) - V(current_state)
  需要 Critic Model (Value Network) → 额外的模型参数和显存

GRPO:
  Advantage_i = (reward_i - mean(rewards_group)) / std(rewards_group)
  不需要 Critic → 省掉一个与 Policy 等大的模型
```

PPO 需要 4 个模型（Policy + Reference + Reward + Critic），GRPO 只需要 3 个（去掉了 Critic）。对于 671B 模型，省掉 Critic 意味着省掉 ~130GB VRAM。

更重要的是：GRPO 证明了你不需要 Critic 来估计 advantage。给定同一 prompt 的 G 个采样，组内归一化就是一个足够好的优势估计——直觉上，「比平均好」或「比平均差」的逻辑比「绝对价值估计」更简单也更鲁棒。

DeepSeekMath 的另一个贡献是**数据选择管线**——从 Common Crawl 中提取 120B 数学相关 token。这个管线后来被 V3 的数据工程继承。

### DeepSeek-V3：671B，$5.6M

arXiv 2412.19437。V3 是 V2 的规模化版本，671B 总参数、37B 激活，训练成本仅 $5.6M（2.788M H800 GPU hours）。V3 带来了三项核心创新。

#### Auxiliary-Loss-Free Load Balancing

MoE 的经典问题：专家负载不均衡。某些专家被过度使用，其他专家「饿死」。传统方案是添加 auxiliary loss 强制均衡，但这会干扰主要的语言建模 loss。

V3 的解决方案：**动态 bias 调整**。

```python
for each training step:
    tokens_per_expert = count_tokens(router(tokens))
    
    for expert in experts:
        if tokens_per_expert[expert] > threshold:
            expert.bias -= delta  # 降低 bias → 被选概率下降
        else:
            expert.bias += delta  # 提高 bias → 被选概率上升
    
    # 路由时在 logits 上加 bias
    expert_scores = X @ W_g + biases
    # bias 只影响 top-K 选择，不参与最终加权
```

Bias 只影响「哪些专家被选中」，不影响「选中的专家怎么加权」。这样负载均衡不干扰模型学习——没有 gradient 从 bias 流向奖励信号。

#### Multi-Token Prediction (MTP)

传统语言模型只预测下一个 token。V3 同时预测下 2/3/4 个 token：

```
输入 [t1, t2, t3] → 主模型
    ├─ Output Head → 预测 t4 (主 loss)
    ├─ MTP Module 1 → 预测 t5
    ├─ MTP Module 2 → 预测 t6
    └─ MTP Module 3 → 预测 t7
```

每个 MTP Module 是一个轻量的 Transformer 层，接收主模型的 hidden state + 前一个预测 token 的 embedding。训练信号密度提升 4× → 同等 loss 所需的训练步数更少 → 成本更低。

推理时丢弃所有 MTP Modules——所以推理速度和 V2 一样。

#### FP8 Mixed Precision Training

V3 是首个大规模验证 FP8 训练的模型。相比于 BF16：
- **FP8 E4M3**（指数 4 位、尾数 3 位）：用于前向传播（精度更高）
- **FP8 E5M2**（指数 5 位、尾数 2 位）：用于反向传播（范围更大）

关键技巧：
1. **Block-wise quantization**：不全局 scale，而是 1×128 tile 为单位量化
2. **高精度保留**：Attention softmax、LayerNorm、Embedding 保持 BF16/FP32
3. **低精度存储 + 高精度累加**：FP8 tensor core 计算，FP32 累加器

训练吞吐量提升 ~2×，精度损失 <0.25%。

#### V3 训练稳定性

论文特别强调：**整个训练过程中，没有出现过不可恢复的 loss spike，没有做过任何 rollback**。对于 14.8T tokens 的 671B MoE 训练，这是卓越的工程成就。

---

## 第四章：推理革命（2025.01）

### DeepSeek-R1：纯 RL 让模型学会思考

arXiv 2501.12948。如果 MLA 是 DeepSeek 在效率上最伟大的贡献，R1 就是在方法论上最伟大的贡献。

#### R1-Zero：不做 SFT，直接 RL

**从 DeepSeek-V3-Base（一个从未见过指令数据的模型）出发，只做 GRPO + 规则化奖励。**

奖励函数极其简单：
- 数学题：答案正确 → +1
- 代码题：测试通过 → +1
- 格式：正确使用 think/ response 标签 → +1

没有人类标注的推理数据。没有 Reward Model。没有 Critic。只有这三个规则。

结果：AIME 2024 pass@1 = **71.0%**，接近 OpenAI o1-0912。

#### 涌现的三种行为

纯 RL 训练中，R1-Zero 自发出现了三种行为：

**Self-Verification（自我验证）**：
```
...所以答案是 42。等等，让我检查一下...验证通过。答案是 42。
```

**Reflection（反思）**：
```
我之前的推理第 3 步有误。重新来：从 y = 2x 出发...
```

**Alternative Exploration（替代探索）**：
```
也许有更简单的方法...直接用公式代入...
```

这些行为不是被 SFT 教出来的——SFT 只会复制人类标注的「干净」推理链，永远不会包含错误和修正。这些是 RL 搜索中的 emergent strategy——模型发现「自我检查 → 更高正确率 → 更高 reward → 被强化」。

#### R1 的完整流程

R1-Zero 推理能力极强，但输出不可读——中英混杂、格式混乱。R1 通过多阶段训练解决：

```
Stage 1: Cold Start SFT（几千条手工推理链）→ 教格式
Stage 2: GRPO（推理 RL）→ 强化能力
Stage 3: Rejection Sampling + 通用 SFT → 恢复通用能力
Stage 4: 全领域 RL → 对齐 helpfulness + harmlessness
```

最终 R1：AIME 2024 **79.8%**，可读性好，通用能力不丢失。

#### 蒸馏的力量

用 R1 的推理链做 SFT 训练小模型（注意：是 SFT，不是 KL 蒸馏）：

| 模型 | 参数 | AIME 2024 |
|------|------|-----------|
| R1 (teacher) | 671B | 79.8% |
| **R1-Distill-Qwen-32B** | **32B** | **72.6%** |
| R1-Distill-Qwen-14B | 14B | 69.7% |
| R1-Distill-Llama-70B | 70B | 70.0% |

32B 以 1/20 的参数达到 R1 的 91% 推理能力。这证明：好的推理数据 > 大的模型参数。

---

## 第五章：基础设施与持续进化（2025.02 – 2026.04）

### Infra 三件套（2025.02）

V3/R1 发布后的一个月，DeepSeek 陆续开源了三个基础设施项目：

**FlashMLA**：MLA 的专用高效 kernel。类似 FlashAttention 的 IO-aware 策略（tiling + SRAM），但专门适配 MLA 的潜向量上投影计算模式。~2× 加速。

**DeepGEMM**：清洁高效的 FP8 GEMM kernel。细粒度缩放（fine-grained scaling），专门优化了 FP8 tensor core 的使用方式。

**3FS**：高性能分布式文件系统。为 AI 训练/推理的 I/O 模式设计——高吞吐、低延迟、大规模随机读取。用于训练数据读取、checkpoint 存储、KV Cache offload。

这三个项目不是论文，但构成了 DeepSeek 技术栈的「工程底座」。它们说明了一个事实：DeepSeek 不只是在发表论文，他们在打造一个从底层 kernel 到顶层推理服务的**完整技术体系**。

### DeepSeek-V3.2-Exp：稀疏注意力的验证（2025.09）

在 V3 和 V4 之间，DeepSeek 发布了 V3.2-Exp——一个验证 DSA（DeepSeek Sparse Attention）/ NSA（Native Sparse Attention）的中间版本。

核心创新：**可训练的稀疏注意力**。

传统稀疏注意力（Sliding Window、Strided、BigBird）使用固定的、人工设计的 sparse pattern。DSA 让模型学习「哪些 token 之间需要相互 attend」：

```
Sliding Window: 固定窗口大小 w → O(n·w)
DSA: attention_mask = learnable_sparsity(Q, K) → 可学习的 pattern
```

DSA 保留了 attention 的表达能力（不像 Mamba 那样完全切换到 SSM），但通过稀疏化将计算降到 O(n log n) 级别。这是为 V4 的百万上下文铺路的中间步骤。

### DeepSeek-V4：百万上下文的 Engram 架构（2026.04）

2026 年 4 月 24 日，DeepSeek 发布了 V4 系列。两个变体：V4-Flash（快速模式）和 V4-Pro（专家模式/深度推理）。

#### 核心数据

- **1M token 上下文窗口**：V3 的 128K → V4 的 1M，8 倍
- **参数量**：V3 的 ~1.6 倍 → 约 1T 总参数
- **Engram 架构** + **mHC 框架**：全新的长上下文处理范式
- **原生多模态**：发布后一周上线识图模式
- **成本**：推理成本仅为海外模型的 5-20%

#### Engram 架构

"Engram"一词源于神经科学中的「记忆痕迹」。V4 的 Engram 架构将模型设计为**记忆感知的计算系统**——把上下文切分为可寻址的「记忆单元」，在架构层面实现类似操作系统的虚拟内存管理。

Engram 可以理解为 MLA 和 DSA 的统一框架：

```
MLA  → 将 KV 压缩为低秩潜向量（存储面 → 记忆痕迹的形成）
DSA  → 可训练稀疏注意力选择需要唤起的记忆（检索面 → 记忆的唤起）
mHC  → 多尺度层次化上下文（微观/中观/宏观 → 记忆的分层管理）
```

在 Engram 框架下，1M token 的上下文不再是一个需要线性增长的 KV Cache 问题，而是一个**可寻址的外部记忆系统**——模型只关注当前需要的部分，其余保持压缩状态。

#### 模式切换

V4 引入了 Fast Mode（V4-Flash）和 Expert Mode（V4-Pro）的双模式：

- **Fast Mode**: 快速响应，类似 GPT-4o-mini 的体验，适合对话和简单任务
- **Expert Mode**: 深度推理，支持 `reasoning_effort` 参数控制思考深度，类似 o1 但更灵活

这与 Qwen3 的 Hybrid Thinking 思路类似，但实现不同——V4 使用两个部署变体，Qwen3 使用单一模型。

---

## 第六章：技术演进的统一叙事

### 效率优化的四个维度

回顾 DeepSeek 的所有技术，它们恰好分布在四个维度上：

| 维度 | 创新 | 优化目标 | 成果 |
|------|------|---------|------|
| **存储效率** | MLA | KV Cache 显存 | -93.3% |
| **计算效率** | DSA, Engram | Attention 计算 | 稀疏化至 O(n log n) |
| **训练效率** | FP8, MTP, MoE | 训练时间/成本 | V3 仅 $5.6M |
| **学习效率** | GRPO, R1 | RL 训练效率 | 省去 Critic, 纯规则奖励 |

每个创新都在消除一个具体的效率瓶颈，而这些创新之间又有内在联系：

```
MLA 压缩 KV Cache → 使长上下文训练成为可能
    ↓
MTP 增加训练信号密度 → 使 FP8 训练的精度损失可接受
    ↓
FP8 降低单步成本 → 使更大规模的 MoE 训练成为可能
    ↓
MoE 增大模型容量 → 使 GRPO 的 emergent reasoning 有更强的基座
    ↓
GRPO 省去 Critic → 使大规模 RL 训练成为可能
    ↓
R1 纯 RL 推理 → 验证了推理能力来自 RL 而非 SFT
    ↓
DSA 稀疏注意力 → 为 V4 的百万上下文铺路
    ↓
Engram 统一框架 → 高效长上下文的最终形态
```

这不是一系列孤立的创新。这是一个**持续追求效率的工程体系**中自然演化出的技术闭环。

### 与行业的对比

理解 DeepSeek 最好的方式是与同行对比：

| 维度 | OpenAI | Anthropic | DeepSeek |
|------|--------|-----------|----------|
| 核心理念 | Scaling | Safety | **Efficiency** |
| 训练成本 | ~$78M (GPT-4 est.) | 未公开 | **$5.6M (V3)** |
| 推理成本 | 高 | 高 | **5-20% of competitors** |
| 架构创新 | 闭源 | 闭源 | **MLA, MoE, DSA, Engram (全开源)** |
| 开源策略 | 不开源 | 不开源 | **MIT/Apache 2.0** |
| 方法论贡献 | Scaling Laws | Constitutional AI | **GRPO, 纯 RL 推理** |

DeepSeek 的特殊之处在于：**它在极其有限的资源约束下，通过架构创新追上了拥有近乎无限资源的对手**。这不是预算的胜利，是工程哲学的胜利。

---

## 结语：DeepSeek 教给我们的事

1. **架构创新 > 算力堆砌**：MLA、GRPO、DSA 都是在减少资源需求而非增加。用架构解决问题永远比用算力硬扛更可持续。

2. **开源是最好的护城河**：MLA 被 Kimi K2 和 GLM-5.1 采用，但 DeepSeek 已经在做下一件事。开源推动行业前进，也让领先者保持领先。

3. **效率优先是可持续的策略**：如果 V3 的成本是 GPT-4 的 1/15，那么即使 GPT-4 的能力提升 20%，V3 仍然有巨大的性价比优势。

4. **RL 是新的 SFT**：R1 证明推理能力不来自模仿人类数据，而来自 RL 搜索。未来的模型能力提升可能越来越多地依赖 RL 而非更大规模的预训练。

5. **系统思维 > 单点优化**：MLA 到 DSA 到 Engram 不是三个独立的点子，而是一个解决「如何高效处理长上下文」问题的系统方案。每一代都建立在前一代之上。

---

## 相关论文

本文涉及的全部 DeepSeek 论文（按时间顺序）：

- DeepSeek LLM (arXiv 2401.06066): 第一个通用 LLM
- DeepSeekMoE (arXiv 2401.02954): 细粒度专家 + 共享专家
- DeepSeekMath (arXiv 2402.03300): GRPO 诞生
- DeepSeek-V2 (arXiv 2405.04434): MLA + 236B MoE
- DeepSeek-Coder-V2 (arXiv 2406.11931): MoE 代码模型
- DeepSeek-V3 (arXiv 2412.19437): FP8 + MTP + Aux-loss-free, 671B
- DeepSeek-R1 (arXiv 2501.12948): 纯 RL 推理 + GRPO
- DeepSeek-V3.2-Exp (2025.09): DSA 稀疏注意力
- DeepSeek-V4 (2026.04): Engram, 1M context, 多模态
