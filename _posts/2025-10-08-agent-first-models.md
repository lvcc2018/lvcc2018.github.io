---
title: "Agent-First Models: The Post-Chatbot Era of Language Models"
date: 2025-10-08
tags: [Agent, Kimi, MoE, Architecture, Industry, Evaluation]
summary: "In 2025, a new category of language models emerged: Agent-First. These models are optimized not for conversation but for tool use, code execution, and autonomous task completion. Kimi K2's 27-point SWE-bench lead over DeepSeek-V3 — despite nearly identical architecture — reveals that post-training strategy now matters more than pre-training architecture."
---

## The Architecture Paradox

Consider two models:

| | DeepSeek-V3 | Kimi K2 |
|---|---|---|
| Architecture | MoE + MLA | MoE + MLA |
| Total params | 671B | 1T |
| Activated params | 37B | 32B |
| Attention heads | 128 | 64 |
| Hidden dim | 7168 | 7168 |
| MoE hidden dim | 2048 | 2048 |
| Shared experts | 1 | 1 |
| Activated experts | 8 | 8 |
| Activation fn | SwiGLU | SwiGLU |

These models share the same fundamental design. Yet on SWE-bench Verified (Agentic mode):

- DeepSeek-V3: **38.8%**
- Kimi K2: **65.8%**

A **27-point gap** from architecture twins. What explains this?

## The Data Hypothesis

Kimi K2's technical report (arXiv 2507.20534) states:

> "During post-training, K2 undergoes a multi-stage post-training process, highlighted by a **large-scale agentic data synthesis pipeline** and a **joint reinforcement learning stage**, where the model improves its capabilities through interactions with real and synthetic environments."

The emphasis is mine. The key words are "agentic data synthesis" and "real environments."

### What "Real Environment RL" Means

Unlike standard RLHF where the model generates text that a reward model scores, K2's RL training involves:

```python
# Agentic RL training loop
for task in agentic_tasks:
    # 1. Model generates actions
    actions = model.generate_actions(task_description)
    
    # 2. Actions are executed in a real sandbox
    results = sandbox.execute(actions)
    
    # 3. Reward is based on task completion, not text quality
    reward = task.is_completed(results)  # e.g., "does the code pass the tests?"
    
    # 4. Model learns from real environment feedback
    model.update(actions, reward)
```

This is fundamentally different from RLHF. The model isn't learning to produce text that "looks good" — it's learning to produce actions that *actually work* in a real environment. The reward signal comes from a compiler, a test runner, a file system — not from a neural network.

### The Agentic Data Synthesis Pipeline

K2's data pipeline generates diverse agentic scenarios:
1. **Task generation**: LLM generates realistic coding/agent tasks
2. **Solution generation**: Multiple solution attempts per task
3. **Environment validation**: Solutions are executed, only correct ones retained
4. **Preference construction**: Correct solutions become "chosen," incorrect ones become "rejected"

This creates a feedback loop: better models → better data generation → better training data → better models.

---

## The Agent-First Model Spectrum

By mid-2026, a clear spectrum has emerged:

### Tier 1: Pure Agent-First

**Kimi K2/K2.5/K2.6**: Designed from the ground up for tool use.
- SWE-bench Verified: 65.8% (agentic mode)
- SWE-bench Multilingual: 47.3%
- Tau2-Bench: 70.6% (retail)

**MiniMax M2.7**: Agent-specialized, coding-focused.
- Coding Plan optimized
- Claude API compatible
- ~90 TPS at launch

### Tier 2: Agent-Capable Generalists

**DeepSeek-V4**: 1M context enables new agentic patterns.
- Fast Mode (V4-Flash) vs Expert Mode (V4-Pro)
- Native multimodal for GUI agents

**GLM-5.1**: DSA for efficient tool schema retrieval.
- Prefix LM architecture adapted for tool use
- Introduced DSA from DeepSeek ecosystem

### Tier 3: General-Purpose with Agent Features

**Qwen3.6**: Hybrid Thinking enables reasoning-budgeted agent tasks.
- 35B-A3B: 3B active for cost-efficient agent deployment
- Agentic Coding enhanced in 3.6

---

## The Training Recipe That Creates Agent-First Models

### Pre-Training Differences

Agent-first models differ from general-purpose models even at pre-training:

| Component | General-Purpose | Agent-First |
|-----------|----------------|-------------|
| Code proportion | 5-10% | **15-25%** |
| Structured data | Minimal | **JSON, SQL, API schemas** |
| Multi-turn data | Rare | **Abundant** |
| Tool-call format | Absent | **Native in corpus** |

### Post-Training Differences

The post-training pipeline is where the gap emerges:

```python
# General-purpose post-training
for epoch in range(num_epochs):
    # SFT on instruction-following data
    model.sft(instruction_data)
    
    # DPO on preference data
    model.dpo(chosen, rejected)
    
    # RLHF with reward model
    model.rlhf(reward_model)

# Agent-first post-training  
for iteration in range(num_iterations):
    # 1. Generate agentic tasks
    tasks = synthesize_agentic_tasks(model)
    
    # 2. Execute in sandbox
    results = sandbox.execute(model, tasks)
    
    # 3. Construct preference data from execution results
    chosen, rejected = construct_preferences(results)
    
    # 4. Update model
    model.dpo(chosen, rejected)
    model.grpo(tasks, rule_based_reward)
    
    # 5. Evaluate on real benchmarks
    swb_score = evaluate_swebench(model)
    
    # 6. Use results to guide next iteration's task synthesis
    synthesis_prompt += f"Focus on: {failure_modes}"
```

The critical difference: **the environment provides ground-truth feedback.** You don't need a reward model to tell you if code compiles.

---

## Evaluation: Why SWE-bench Is Eating the World

The rise of agent-first models has shifted evaluation priorities:

### Old Paradigm (2022-2024)
- MMLU: "Does the model know things?"
- HumanEval: "Can the model write a function?"
- MT-Bench: "Is the model a good conversationalist?"

### New Paradigm (2025-2026)  
- **SWE-bench Verified**: "Can the model fix a real GitHub issue?"
- **SWE-bench Multilingual**: "Can it do this across languages?"
- **Tau2-Bench**: "Can the model use tools to complete a real-world task?"
- **TerminalBench**: "Can the model operate a terminal?"

These benchmarks measure capability, not just knowledge. A model with perfect MMLU but 0% SWE-bench is useless as an agent.

### The SWE-bench Leaderboard (Mid-2026)

| Model | Agentic Score | Notes |
|-------|-------------|-------|
| Claude Opus 4 | ~80% | Closed-source leader |
| Claude Sonnet 4 | ~73% | Closed-source |
| **Kimi K2.6** | ~72%* | Open-source leader, multi-agent |
| **Kimi K2** | 65.8% | Open-source, single-agent |
| GPT-4.1 | ~55% | Closed-source |
| **DeepSeek-V4** | ~50%* | Open-source, 1M context |
| **DeepSeek-V3** | 38.8% | Open-source |
| Qwen3-235B | 34.4% | Non-thinking mode |

(*estimated from partial reports)

The gap between the best agent-first models and the best general-purpose models is **20-30 points** on the most important agent benchmark. This is not noise.

---

## The Economics of Agent-First

### Inference Cost

Agent tasks consume more tokens per interaction than chat:

| Task Type | Tokens per interaction | Cost (at V3 pricing) |
|-----------|----------------------|---------------------|
| Simple chat | ~500 | $0.0002 |
| Code generation | ~2000 | $0.0008 |
| SWE-bench task | ~50,000 | $0.02 |
| Complex multi-step agent | ~200,000 | $0.08 |

At 50K tokens per task, a 1% improvement in SWE-bench score saves ~500 tokens per successful task — the model needs fewer retries. This creates a reinforcing cycle: better agents → lower cost per task → more usage → better training data.

### Model Size Economics

Agent-first models like K2 (1T total, 32B active) and Qwen3.6-35B-A3B (35B, 3B active) are exploring extreme activation sparsity:

| Model | Total params | Activated | Ratio |
|-------|-------------|-----------|-------|
| DeepSeek-V3 | 671B | 37B | 5.5% |
| Kimi K2 | 1T | 32B | 3.2% |
| Qwen3.6-Flash | 35B | 3B | 8.6% |

The trend: pack more knowledge into fewer activated parameters. For agent tasks, activation sparsity is particularly valuable — different tools and domains activate different experts, so the model can have broad knowledge without paying for it on every token.

---

## What This Means for the Industry

### For Model Developers

If you're training a model in 2026, you need to decide early: is this a chatbot or an agent? The training recipe diverges significantly:

- **Chatbot**: Optimize for helpfulness, harmlessness, honesty (HHH). DPO on human preference data.
- **Agent**: Optimize for task completion. GRPO/CISPO with environment feedback. SWE-bench as primary metric.

### For Practitioners

When choosing a base model for an Agent system:
1. **Don't look at MMLU**: It's nearly irrelevant for agent performance
2. **Prioritize SWE-bench and Tau2-Bench**: These measure what agents actually do
3. **Test tool-calling accuracy directly**: Even within the same model family, tool-calling quality varies dramatically
4. **Consider the training recipe**: A model trained with environment feedback will outperform one trained only on human preferences

### For the Research Community

The agent-first trend raises open questions:
- Can we design architectures specifically for tool use (not just train general architectures for it)?
- How do we handle the combinatorial explosion of possible tool combinations?
- What's the equivalent of Chinchilla scaling laws for agent capabilities?

---

## Case Study: The 27-Point Gap

Let's examine specific tasks where K2 dramatically outperforms V3:

### SWE-bench Example: Bug Fix in Django

**Task**: Fix a race condition in Django's cache framework.

**DeepSeek-V3 approach**:
```
→ Reads bug report
→ Generates a fix
→ Applies the fix  
→ Result: Tests fail — fix introduces a new bug
```

**Kimi K2 approach**:
```
→ Reads bug report
→ Reads relevant Django source code (multiple files)
→ Writes a test that reproduces the bug
→ Verifies the test fails (confirming reproduction)
→ Generates a fix
→ Runs the reproduction test — passes
→ Runs the full test suite — passes
→ Verifies no regressions
```

The difference is not in the model's "intelligence" — both models understand the bug and can generate plausible fixes. The difference is in the *learned workflow*: K2 has internalized a debugging methodology that includes reproduction, verification, and regression testing. This comes from training on real debugging workflows, not from architectural superiority.

---

## The Post-Chatbot Era

We are witnessing a phase transition in LLM development:

**2022-2024: The Chatbot Era**
- Models optimized for conversation
- Evaluation: human preference, MMLU
- Training: RLHF with neural reward models
- Products: ChatGPT, Claude, Gemini (chat interfaces)

**2025-2026: The Agent Era**  
- Models optimized for task completion
- Evaluation: SWE-bench, Tau2-Bench, TerminalBench
- Training: Environment feedback, rule-based rewards
- Products: Claude Code, Kimi K2.6 (agent clusters), MiniMax Agent

The models that will dominate the next phase are not necessarily the ones with the best architecture. They're the ones with the best *training environments* — the richest feedback loops between model actions and real-world outcomes.
