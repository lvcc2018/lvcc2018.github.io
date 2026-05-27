---
title: Ethan Lu
layout: default
---

# Ethan Lu

**大模型算法工程师 · AI Agent 方向 · 量化交易者**

---

## 👋 Hi

我是 Ethan，目前在腾讯 WXG 从事大模型 Agent 方向的研究与工程落地。日常深耕 LLM 对齐（RLHF/DPO/GRPO）、Agent 架构设计、模型推理优化。业余时间用 Python 做 A 股量化策略研究。

这个网站放我的**简历、项目和技术博客**——论文笔记、工程实践、量化心得。

---

## 🔧 技术栈

`LLM Training` `RLHF/DPO/GRPO` `Agent Architecture` `MoE` `vLLM/SGLang`
`PyTorch` `DeepSpeed` `FSDP` `FlashAttention` `Python` `Quantitative Finance`

---

## 📝 最新文章

{% for post in site.posts limit:5 %}
- **{{ post.date | date: "%Y-%m-%d" }}** [{{ post.title }}]({{ post.url }})
{% endfor %}
