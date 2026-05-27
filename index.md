---
title: Ethan Lv
layout: home
---

<div class="hero">
  <div class="hero-text">
    <h1>Ethan Lv</h1>
    <p class="subtitle">LLM Specialist · Agent Infrastructure · Model Alignment</p>
    <p class="location">Beijing, China</p>
    <p class="social-links">
      <a href="https://github.com/lvcc2018">GitHub</a>
      <span class="sep">·</span>
      <a href="mailto:thulvcc2017@gmail.com">thulvcc2017@gmail.com</a>
      <span class="sep">·</span>
      <a href="/blog">Blog</a>
    </p>
  </div>
</div>

---

## Summary

LLM specialist with 4 years of end-to-end experience spanning data strategy, pre-training, post-training alignment (RLHF / DPO / GRPO / OPD), and AI Agent infrastructure. Delivered 70B-scale model training with 10T+ token data pipelines; published 6 papers at **NeurIPS / EMNLP / ACL**, including **C-Eval** (NeurIPS 2023), a widely-adopted Chinese LLM benchmark. Currently focused on Agent infrastructure, online preference optimization, and continual model improvement at **WeChat scale (100M+ DAU)**.

---

## Experience

### Tencent Technology (Beijing)
**Tech Lead, WeChat AI Assistant** · May 2025 — Present

- Architected the WeChat AI Assistant Agent Harness from scratch — designing and implementing tool-calling orchestration, multi-step planning and reasoning, context memory management, and multi-layer safety guardrails for a system serving **hundreds of millions of daily active users**.
- Led Agent-specific post-training, developing systematic training recipes across instruction following, tool-use alignment, multi-turn consistency, and safety refusal boundaries; introduced **Online Preference Distillation (OPD)** to enable continuous preference alignment post-deployment, leveraging Iterative DPO, GRPO, and Constitutional AI.
- Built a multi-dimensional Agent evaluation matrix covering single-step tool accuracy, multi-step task completion rate, hallucination rate, safety boundary adherence, and long-horizon consistency, establishing a **data flywheel** for joint model–framework optimization.
- Drove deep integration of Agent capabilities with the WeChat ecosystem, designing a standardized tool-access protocol that unified tool schemas across Mini Programs, Official Accounts, WeChat Pay, and Search, significantly improving end-to-end task success rates.

### Shenyan Technology (Beijing)
**LLM Algorithm Engineer** · March 2022 — April 2025

- Led data strategy and training pipeline construction for general-purpose foundation models: designed multi-stage filtering pipelines for pre-training and instruction-tuning data, combining heuristic rules, perplexity distribution analysis, and model-based quality annotation for fine-grained classification and deduplication, ultimately producing **10T high-quality pre-training tokens** and millions of premium SFT samples.
- Built a comprehensive evaluation framework aligned with training data taxonomy, integrating mainstream benchmarks with proprietary test suites (including temporally relevant and rigorously unleaked evaluation data), establishing a closed-loop of data classification → evaluation alignment → capability diagnosis.
- Designed a **Scaling Law experiment matrix** grounded in fine-grained data labeling, systematically validating the impact of data mixture ratios, hyperparameters, and curriculum strategies on loss convergence and downstream performance across small-scale models (100M–several B), identifying optimal pre-training recipes that generalized successfully to the full 70B model series.
- Led multi-dimensional capability improvement initiatives: executed phased **long-context extension (8K → 32K → 128K)** via RoPE base-frequency tuning, data-mixture optimization, and progressive training; simultaneously drove specialized enhancements in STEM reasoning and code generation, with results approaching state-of-the-art models of comparable scale.
- For the financial vertical domain, constructed **100B high-quality CPT data** from 40T+ raw multi-source corpora; designed and implemented a document-level curriculum learning scheme that organized training data by difficulty and inter-document semantic similarity, combined with dynamic hyperparameter scheduling to effectively mitigate catastrophic forgetting.
- Delivered long-text summarization and intelligent Q&A product features atop internal LLMs by decomposing tasks into evaluable sub-modules (extraction–compression–rewriting–verification), performing independent data synthesis, quality filtering, and targeted training per module, forming a reusable data flywheel.
- Applied **DPO training** on auto-constructed preference pairs to substantially reduce hallucination and repetitive generation; implemented **GRPO training** with citation recall as the reward signal, achieving a step-change improvement in generated citation accuracy.

---

## Education

### Tsinghua University
**M.S. in Computer Science and Technology** · 2021 — 2024  
Research focus: Natural Language Processing and Large Language Models

### Tsinghua University
**B.S. in Computer Science and Technology** · 2017 — 2021

---

## Publications

1. **GATEAU: Selecting Influential Samples for Long Context Alignment**  
   Si et al. · *EMNLP 2025*

2. **Document Segmentation Matters for Retrieval-Augmented Generation**  
   Wang et al. · *ACL 2025 Findings*

3. **HyperLoRA: Efficient Cross-task Generalization via Constrained Low-Rank Adapters Generation**  
   Lv et al. · *EMNLP 2024 Findings*

4. **C-Eval: A Multi-Level Multi-Discipline Chinese Evaluation Suite for Foundation Models**  
   Huang et al. · *NeurIPS 2023 (Datasets & Benchmarks)*

5. **Sememe Prediction for BabelNet Synsets using Multilingual and Multimodal Information**  
   Qi, Lv et al. · *ACL 2022 Findings*

6. **Temporal Cross-Effects in Knowledge Tracing**  
   Wang et al. · *WSDM 2021*

---

## Skills

<div class="skills-grid">

<div class="skill-block">
<h3>LLM Training & Alignment</h3>
<p>Scaling Law experiments & pre-training strategy, SFT data strategy, Iterative DPO / GRPO / OPD / Constitutional AI, CPT with anti-forgetting techniques</p>
</div>

<div class="skill-block">
<h3>AI Agent Systems</h3>
<p>Agent framework architecture, function calling & tool orchestration, multi-step planning & reasoning chains, memory & context management, safety guardrails</p>
</div>

<div class="skill-block">
<h3>Data Engineering</h3>
<p>Large-scale corpus filtering & deduplication (10T+), data synthesis & augmentation, automated annotation & classification, multi-dimensional data evaluation</p>
</div>

<div class="skill-block">
<h3>Model Evaluation</h3>
<p>Benchmark construction (C-Eval), multi-dimensional capability diagnosis, fair evaluation design, unleaked evaluation data management</p>
</div>

<div class="skill-block">
<h3>Specialized Capabilities</h3>
<p>Long-context extension (128K), RAG system design & optimization, STEM / code / reasoning enhancement, catastrophic forgetting mitigation</p>
</div>

<div class="skill-block">
<h3>Tech Stack</h3>
<p>Python, PyTorch, DeepSpeed, Transformers, distributed training (FSDP / Megatron), vector databases & retrieval systems</p>
</div>

</div>

---

## 📝 Blog

<div class="post-list-home">
{% for post in site.posts limit:5 %}
<a href="{{ post.url }}" class="post-item">
  <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  <span class="post-title">{{ post.title }}</span>
  <span class="post-tags">{% for tag in post.tags %}<code>{{ tag }}</code> {% endfor %}</span>
</a>
{% endfor %}
</div>

[📚 All posts →](/blog)
