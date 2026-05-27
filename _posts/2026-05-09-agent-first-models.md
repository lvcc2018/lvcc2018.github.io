---
title: "The Rise of Agent-First Models: Kimi K2, GLM-5, and the Post-Chatbot Era"
date: 2026-05-09
tags: [Agent, Kimi, MoE, Architecture, Industry]
summary: "2025 saw a fundamental shift: the best open-source models are no longer optimized for chat — they're optimized for tool use, code execution, and autonomous problem-solving. Kimi K2's SWE-bench score of 65.8% (vs V3's 38.8%) tells the story."
---

## Two Models, Same Architecture, 27 Points Apart

Kimi K2 and DeepSeek-V3 share essentially the same architecture:

| Component | DeepSeek-V3 | Kimi K2 |
|-----------|-------------|---------|
| Architecture | MoE + MLA | MoE + MLA |
| Total params | 671B | 1T |
| Activated params | 37B | 32B |
| Attention | MLA | MLA |
| Activation | SwiGLU | SwiGLU |
| Context | 128K | 128K |

Yet on SWE-bench Verified (Agentic mode), K2 scores **65.8%** while V3 scores **38.8%**.

That's a 27-point gap from essentially the same architecture. Where does it come from?

## The Data Difference

The answer is in the post-training. K2's technical report emphasizes:

> "A large-scale agentic data synthesis pipeline and a joint reinforcement learning stage, where the model improves its capabilities through interactions with real and synthetic environments."

In plain English: K2 was trained to *do things*, not just *say things*.

The post-training pipeline for K2 includes:
1. **Agentic data synthesis**: Generating diverse tool-use scenarios at scale
2. **Real environment RL**: Interacting with actual code execution environments during RL training
3. **Joint RL stage**: Optimizing for both helpfulness and task completion simultaneously

V3's post-training, by contrast, focused on general instruction following and reasoning — not specifically on tool use in real environments.

## The Agent-First Thesis

This reflects a broader trend in 2025-2026:

| Model | Release | Agent Benchmark | Philosophy |
|-------|---------|----------------|------------|
| DeepSeek-V3 | Dec 2024 | SWE-bench 38.8% | General-purpose reasoning |
| Kimi K2 | Jul 2025 | SWE-bench 65.8% | **Agent-first** |
| MiniMax M2.7 | Mar 2026 | Coding Plan leader | **Agent-specialized** |
| GLM-5.1 | 2026 | Coding Plan leader | **Coding + DSA** |

The industry is splitting into two camps:
- **Generalists** (DeepSeek-V4, Qwen3.6): Broad capability, thinking modes, long context
- **Agent-specialists** (Kimi K2, MiniMax M2, GLM-5): Tool use, coding, task completion

## What "Agent-First" Means in Practice

Training an agent-first model means different design choices at every stage:

### Pre-training
- Higher proportion of code and structured data
- Multi-turn interaction data in the pre-training corpus
- Tool-call-formatted examples mixed into the general corpus

### Post-training
- SFT data with tool calls, not just text responses
- RL environments with sandboxed code execution
- Reward signals based on task completion, not just human preference

### Evaluation
- SWE-bench, Tau2-Bench, TerminalBench instead of just MMLU
- Multi-step task completion rate, not just single-turn accuracy
- Hallucination rate in tool-calling scenarios

## The Implication for Practitioners

If you're building an Agent system today, the choice of base model matters more than ever. A model optimized for SWE-bench will dramatically outperform a general-purpose model of the same architecture — not because of architecture differences, but because of what it was trained to do.

This also means that **post-training strategy is becoming more important than pre-training architecture**. Kimi K2 and DeepSeek-V3 prove that you can have the same MLA + MoE architecture and get wildly different Agent performance based on what you do after pre-training.

## What's Next?

The next frontier is **agent-specific architectures** — not just agent-specific training. We're already seeing hints:

- **Claw group collaboration** (Kimi K2.6): Multi-agent collaboration built into the model's context management
- **DSA for tool retrieval** (GLM-5.1): Sparse attention patterns optimized for tool schema lookup
- **Thinking budgets** (Qwen3, MiniMax-M1): Dynamic allocation of compute for planning vs execution
