---
title: Ethan Lu
layout: home
---

<div class="hero">
  <div class="hero-text">
    <h1>Ethan Lu</h1>
    <p class="subtitle">大模型算法工程师 · LLM Agent · 量化交易</p>
    <p class="location">Tencent WXG · Beijing</p>
    <p class="social-links">
      <a href="https://github.com/lvcc2018">GitHub</a>
      <span class="sep">·</span>
      <a href="mailto:thulvcc2017@gmail.com">Email</a>
      <span class="sep">·</span>
      <a href="/about">About</a>
      <span class="sep">·</span>
      <a href="/blog">Blog</a>
    </p>
  </div>
</div>

---

## 关于我

目前在**腾讯 WXG** 从事大模型 **Agent 方向** 的研究与工程落地，T10 职级。日常工作聚焦于：

- **Agent 架构**：多 Agent 协作、工具调用（Function Calling / MCP）、Planning 策略、三层记忆系统
- **模型对齐**：SFT / DPO / GRPO 训练管线，RLHF 全流程
- **推理优化**：vLLM 部署、PagedAttention、KV Cache 管理、Continuous Batching

业余时间维护 **QuantAce**——A 股多因子量化策略研究平台（111 只股票池，28 个因子，10 个策略）。

---

## 研究方向

<div class="research-grid">

<div class="research-card">
<h3>🤖 Agent 架构与对齐</h3>
<ul>
  <li>ReAct / Plan-Then-Execute / Multi-Agent 协作范式</li>
  <li>Tool Use 可靠性：结构化输出、错误恢复、Fallback 策略</li>
  <li>SFT → DPO → GRPO 全链路对齐管线</li>
  <li>生产环境 Agent 评估体系（任务完成率、工具调用准确率、Token 效率）</li>
</ul>
</div>

<div class="research-card">
<h3>⚡ 推理优化</h3>
<ul>
  <li>vLLM / SGLang 部署与调优</li>
  <li>PagedAttention KV Cache 管理</li>
  <li>Continuous Batching & Chunked Prefill</li>
  <li>投机解码、INT8/INT4 量化、前缀缓存</li>
</ul>
</div>

<div class="research-card">
<h3>🧠 训练 & 架构</h3>
<ul>
  <li>MoE 架构：DeepSeekMoE / Qwen-MoE / Mixtral</li>
  <li>分布式训练：FSDP / ZeRO-3 / TP + PP</li>
  <li>长上下文：RoPE 外推 / YaRN / MLA / Linear Attention</li>
  <li>FP8 混合精度训练</li>
</ul>
</div>

</div>

---

## 技术栈

<div class="skills">

`LLM` `Agent` `RLHF/DPO/GRPO` `SFT` `MoE`  
`PyTorch` `DeepSpeed` `FSDP` `FlashAttention`  
`vLLM` `SGLang` `PagedAttention` `KV Cache`  
`Python` `C++` `CUDA` `Git` `Linux`  
`AKShare` `Pandas` `Backtrader` `Quantitative Finance`

</div>

---

## 项目

### LLM Agent 平台 · *Tencent WXG*

企业级 Agent 推理平台，支持工具调用、RAG、多 Agent 编排。在生产环境中服务 **百万级用户**。

核心技术决策：
- 工具调用标准化（OpenAI Function Calling / Anthropic 兼容）
- 多 Agent 通信机制（共享上下文 vs 消息传递）
- RAG 管线：Hybrid Search（向量 + BM25）+ BGE-Reranker
- 推理服务化：vLLM + Prefix Caching + INT8 KV Cache

### QuantAce · 量化策略研究平台

基于 A 股市场的多因子量化策略研究系统。[[GitHub](https://github.com/lvcc2018/QuantAce)]

- **股票池**：111 只 A 股，覆盖主板 + 创业板 + 科创板
- **因子库**：28 个因子（估值、动量、质量、波动率等维度）
- **策略**：10 个策略，涨跌停限制模拟
- **技术栈**：AKShare 数据源 + Parquet 缓存 + Backtrader 回测 + LightGBM 预测

---

## 📝 最新文章

<div class="post-list-home">
{% for post in site.posts limit:5 %}
<a href="{{ post.url }}" class="post-item">
  <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  <span class="post-title">{{ post.title }}</span>
  <span class="post-tags">{% for tag in post.tags %}<code>{{ tag }}</code> {% endfor %}</span>
</a>
{% endfor %}
</div>

[📚 全部文章 →](/blog)
