---
title: "CISPO vs GRPO vs PPO: A Practical Guide to LLM Alignment Algorithms"
date: 2026-05-09
tags: [Alignment, RLHF, GRPO, DPO, Training]
summary: "PPO, DPO, GRPO, CISPO — four alignment algorithms, four different philosophies. When should you use each one? Let's break down the tradeoffs with concrete numbers from DeepSeek-R1 and MiniMax-M1."
---

## The Alignment Algorithm Family Tree

```
                         RLHF (InstructGPT, 2022)
                              │
                    ┌─────────┼─────────┐
                    │         │         │
                   PPO       DPO     GRPO
              (4 models)  (2 models) (3 models)
                    │                   │
                    │                 CISPO
                    │           (3 models, faster)
```

Each algorithm represents a different tradeoff: model count vs training stability vs data requirements vs final performance.

## The Memory Problem

The fundamental constraint in alignment training is GPU memory. Each model loaded into GPU memory consumes VRAM proportional to the parameter count:

| Algorithm | Models in GPU | Memory (671B scale) | Relative |
|-----------|--------------|---------------------|----------|
| PPO | Policy + Ref + Reward + Critic | 4× | 100% |
| GRPO | Policy + Ref + Reward | 3× | 75% |
| CISPO | Policy + Ref + Reward | 3× | 75% |
| DPO | Policy + Ref | 2× | 50% |

At the 671B scale, removing the Critic saves ~25% of VRAM. That's the difference between fitting on a cluster and not fitting.

## GRPO: The Reasoning Specialist

**Origin**: DeepSeekMath (arXiv 2402.03300)  
**Key insight**: Group-relative advantage estimation — no Critic needed.

```python
# PPO advantage estimation
A_i = reward_i + gamma * V(s_next) - V(s_current)
# V requires a Critic model of equal size to the policy

# GRPO advantage estimation  
A_i = (reward_i - mean(rewards_group)) / std(rewards_group)
# Normalize within a group of G samples from the same prompt
```

**When GRPO excels**:
- Reasoning tasks with verifiable rewards (math, code)
- Rule-based reward signals (answer correct, tests pass)
- When you can afford G× more inference during training (G typically 4)

**GRPO's limitations**:
- Group size G determines advantage quality — too small = noisy
- If all samples in a group are correct, advantage = 0 everywhere → no learning signal
- Works best with rule-based rewards; less tested with neural reward models

**Real-world result**: DeepSeek-R1, trained with GRPO on V3-Base, achieves AIME 2024 pass@1 = 79.8%.

## CISPO: The Efficiency King

**Origin**: MiniMax-M1 (arXiv 2506.13585)  
**Key insight**: Clip importance sampling weights, not token updates.

```python
# PPO
loss = -min(ratio * advantage, clip(ratio, 1-eps, 1+eps) * advantage)

# CISPO
importance_weight = pi_new / pi_old
clipped_weight = clip(importance_weight, 1-eps, 1+eps)
loss = -clipped_weight * advantage
```

**When CISPO excels**:
- Large-scale RL training with budget constraints
- When training throughput (not sample efficiency) is the bottleneck
- Hybrid attention models that already have efficient forward passes

**Real-world result**: MiniMax-M1 trained a 456B MoE on 512 H800 GPUs in **3 weeks for $534,700**.

## DPO: The Pragmatist

**Origin**: DPO paper (arXiv 2305.18290)  
**Key insight**: You don't need RL at all — preference learning can be a classification problem.

```python
# DPO loss
loss = -log_sigmoid(
    beta * (log_pi(chosen) - log_ref(chosen)) -
    beta * (log_pi(rejected) - log_ref(rejected))
)
```

**When DPO excels**:
- Offline preference data available (no online generation needed)
- Simple setup (2 models, no RL instability)
- Quick iteration cycles

**DPO's limitations**:
- Offline only — can't learn from the model's own distribution shift
- Sensitive to beta hyperparameter
- Tends to prefer longer responses without length normalization

## When to Use What

| Scenario | Algorithm | Why |
|----------|-----------|-----|
| Reasoning model, math/code rewards | **GRPO** | Group-relative advantage + rule-based rewards |
| Budget-constrained RL at scale | **CISPO** | 3 weeks instead of months |
| General chat alignment, offline data | **DPO** | Simple, stable, fast iteration |
| Online RL with neural RM | **PPO** (or GRPO) | Online sampling matters for neural rewards |
| Iterative deployment improvement | **Iterative DPO** or **OPD** | Close the feedback loop |

## The Trend

The industry is converging on **GRPO for reasoning** and **DPO for chat**, with **CISPO** emerging as a compelling alternative for budget-constrained teams. The dream of a single algorithm that handles both reasoning and chat alignment efficiently is still open — Qwen3's Hybrid Thinking (one model, two modes) is a step in that direction.
