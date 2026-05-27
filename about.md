---
title: About
layout: default
---

# About

## Work

I'm a Tech Lead at **Tencent WXG**, building the WeChat AI Assistant Agent Harness — a multi-agent orchestration platform serving hundreds of millions of users. My focus spans post-training alignment (SFT / Iterative DPO / GRPO / Constitutional AI), Agent infrastructure (tool-calling, planning, memory, safety guardrails), and online preference optimization (OPD) for continuous post-deployment improvement.

Before Tencent, I spent 3 years at Shenyan Technology as an LLM Algorithm Engineer, leading data strategy for 70B-scale foundation models — scaling training data pipelines to 10T+ tokens, designing multi-stage filtering and deduplication systems, and running Scaling Law experiments to identify optimal pre-training recipes.

---

## Research

I work at the intersection of **post-training alignment** and **AI Agent systems**. My research interests include:

- **Online Preference Optimization** — can we continuously improve deployed models without costly offline retraining?
- **Agent Evaluation** — how do you measure an Agent's performance across tool accuracy, task completion, hallucination, and safety in a way that forms a closed data flywheel?
- **Long-Context Alignment** — how do we select training data that actually improves long-context performance rather than just measuring it?

### Selected Papers
{% assign papers = "GATEAU: Selecting Influential Samples for Long Context Alignment|EMNLP 2025,Document Segmentation Matters for RAG|ACL 2025,HyperLoRA: Constrained Low-Rank Adapters Generation|EMNLP 2024,C-Eval: Chinese LLM Benchmark|NeurIPS 2023" | split: "," %}
{% for p in papers %}{% assign parts = p | split: "|" %}- **{{ parts[0] }}** · *{{ parts[1] }}*
{% endfor %}

---

## This Site

Built with Jekyll, hosted on GitHub Pages. A place to document technical notes on LLM training, Agent architecture, and model alignment. No analytics, no tracking — just words and code.

---

## Contact

- **Email**: [thulvcc2017@gmail.com](mailto:thulvcc2017@gmail.com)
- **GitHub**: [github.com/lvcc2018](https://github.com/lvcc2018)
