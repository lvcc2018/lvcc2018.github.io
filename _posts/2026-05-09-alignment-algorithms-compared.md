---
title: "CISPO vs GRPO vs PPO vs DPO: The Complete Guide to LLM Alignment Algorithms"
date: 2026-05-09
tags: [Alignment, RLHF, GRPO, DPO, Training, CISPO]
summary: "A comprehensive technical comparison of the four dominant alignment algorithms. We trace the evolution from PPO's four-model architecture through GRPO's group-relative advantage estimation to CISPO's importance-sampling clipping, with detailed mathematical derivations, memory analysis, training recipes, and guidance on when to use each approach."
---

## The Alignment Algorithm Landscape

Alignment algorithms for LLMs exist on a spectrum defined by three axes:

1. **Online vs Offline**: Does the algorithm require generating new data during training?
2. **Model Count**: How many copies of the model need to be in GPU memory?
3. **Reward Source**: Rule-based, neural reward model, or implicit (learned from preferences)?

```
PPO (2022)    — Online, 4 models, neural RM     ← Original RLHF
  │
  ├─ DPO (2023)   — Offline, 2 models, implicit RM  ← Elimination of RL
  │
  ├─ GRPO (2024)  — Online, 3 models, rule-based    ← Elimination of Critic
  │
  └─ CISPO (2025) — Online, 3 models, rule-based    ← Improved clipping
```

---

## Part 1: PPO — The Original RLHF

### The Objective

PPO for RLHF optimizes:

$$\max_\theta \mathbb{E}_{x \sim D, y \sim \pi_\theta(y|x)} \left[ r_\phi(x,y) - \beta \cdot D_{KL}(\pi_\theta(y|x) \| \pi_{ref}(y|x)) \right]$$

Where:
- $\pi_\theta$: Policy model (being optimized)
- $\pi_{ref}$: Reference model (SFT checkpoint, frozen)
- $r_\phi$: Reward model (trained on human preferences)
- $\beta$: KL penalty coefficient

### The PPO Clipped Objective

PPO's key innovation is **trust-region optimization through clipping**:

$$L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \hat{A}_t \right) \right]$$

Where:
- $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$: probability ratio
- $\hat{A}_t$: advantage estimate from Critic (Value Model)
- $\epsilon$: clipping range (typically 0.2)

The clipping prevents the policy from changing too much in a single update — if the probability ratio goes outside $[1-\epsilon, 1+\epsilon]$, the gradient is clipped.

### The Four-Model Problem

PPO RLHF requires four models simultaneously in GPU memory:

| Model | Role | Size | Required? |
|-------|------|------|-----------|
| Policy $\pi_\theta$ | Being optimized | ~Model size | Always |
| Reference $\pi_{ref}$ | KL divergence anchor | ~Model size | Always |
| Reward $r_\phi$ | Scoring outputs | Smaller | Always |
| **Critic $V_\psi$** | **Advantage estimation** | **~Model size** | **Only PPO** |

**Critic Model:** The Critic (Value Model) estimates the expected future reward from a given state:

$$V(s_t) \approx \mathbb{E}[ \sum_{k=0}^{\infty} \gamma^k r_{t+k} | s_t ]$$

The advantage is then:

$$\hat{A}_t = r_t + \gamma V(s_{t+1}) - V(s_t)$$

The Critic must be roughly the same size as the Policy to provide accurate value estimates. At 671B scale, this is ~1.3 TB of additional VRAM.

### Training Instability

PPO RLHF is notoriously unstable due to:
1. **Reward hacking**: Policy finds ways to get high reward without actually being helpful
2. **KL collapse**: Policy drifts too far from reference despite KL penalty
3. **Value estimation error**: Critic's estimates are noisy, leading to poor advantage signals
4. **Mode collapse**: Policy converges to a narrow set of high-reward responses

---

## Part 2: DPO — Eliminating RL Entirely

### The Key Insight

DPO (arXiv 2305.18290) proved that the optimal policy for the RLHF objective has a closed form:

$$\pi^*(y|x) = \frac{1}{Z(x)} \pi_{ref}(y|x) \cdot \exp\left( \frac{1}{\beta} r(x,y) \right)$$

From this, we can solve for the reward:

$$r(x,y) = \beta \log \frac{\pi^*(y|x)}{\pi_{ref}(y|x)} + \beta \log Z(x)$$

Substituting into the Bradley-Terry preference model gives:

$$p^*(y_w > y_l | x) = \sigma\left( \beta \log \frac{\pi^*(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi^*(y_l|x)}{\pi_{ref}(y_l|x)} \right)$$

### The DPO Loss

The final loss function implicitly encodes the reward model in the policy:

$$\mathcal{L}_{DPO} = -\mathbb{E}_{(x,y_w,y_l) \sim D} \left[ \log \sigma\left( \beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)} \right) \right]$$

### DPO Advantages and Limitations

**Advantages:**
- Only 2 models (policy + reference)
- No online generation needed
- Stable training (simple classification loss)
- No reward model training required

**Limitations:**
- Purely offline — can't learn from the model's own distribution
- The "implicit reward" is a function of the policy, which creates a feedback loop
- $\beta$ is critical: too small → overfitting, too large → no learning
- Tends to produce longer responses (length bias)

### DPO Variants

**IPO (Identity Preference Optimization)** replaces the sigmoid loss with a squared loss to prevent overfitting when $\beta \to 0$:

$$\mathcal{L}_{IPO} = \left( \log \frac{\pi_\theta(y_w)}{\pi_{ref}(y_w)} - \log \frac{\pi_\theta(y_l)}{\pi_{ref}(y_l)} - \frac{1}{2\beta} \right)^2$$

**SimPO (Simple Preference Optimization)** eliminates the reference model entirely by using response length as an implicit regularizer:

$$\mathcal{L}_{SimPO} = -\log \sigma\left( \frac{\beta}{|y_w|} \log \pi_\theta(y_w) - \frac{\beta}{|y_l|} \log \pi_\theta(y_l) - \gamma \right)$$

**KTO (Kahneman-Tversky Optimization)** removes the need for paired preferences, working with individual accept/reject labels inspired by prospect theory.

---

## Part 3: GRPO — Eliminating the Critic

### Origin: DeepSeekMath (arXiv 2402.03300)

GRPO was introduced in the context of mathematical reasoning, where rewards are deterministic (answer correct or not) and online generation is practical.

### The Core Idea

Instead of using a Critic to estimate advantage, GRPO estimates it from a **group of samples from the same prompt**:

**PPO advantage:**
$$\hat{A}^{PPO}_t = r_t + \gamma V(s_{t+1}) - V(s_t)$$

**GRPO advantage:**
$$\hat{A}^{GRPO}_i = \frac{r_i - \text{mean}(\{r_1, ..., r_G\})}{\text{std}(\{r_1, ..., r_G\})}$$

Where $G$ is the group size — GRPO samples $G$ responses for each prompt and normalizes their rewards.

### Why Group-Normalized Advantage Works

The intuition: for the same prompt, if one response gets a high reward and another gets a low reward, the high-reward response is "above average" and should be encouraged. The group provides a natural baseline — no Critic needed.

Mathematically, group normalization:
1. Removes the mean (centering): eliminates the need to estimate absolute value
2. Divides by standard deviation (scaling): makes the advantage scale-invariant
3. Is unbiased if rewards for the same prompt are i.i.d. samples from the policy

### GRPO Training Loop

```python
for batch in prompts:
    # 1. Sample G responses for each prompt
    responses = []
    for _ in range(G):
        resp = policy.generate(prompt)  # Online generation
        responses.append(resp)
    
    # 2. Compute rewards
    rewards = [reward_fn(prompt, resp) for resp in responses]
    
    # 3. Group-normalize to get advantages
    mean_r = np.mean(rewards)
    std_r = np.std(rewards) + 1e-8
    advantages = [(r - mean_r) / std_r for r in rewards]
    
    # 4. PPO-style update with group advantages
    for resp, adv in zip(responses, advantages):
        # Standard PPO clipped objective, but adv comes from group
        ratio = policy.log_prob(resp) / old_policy.log_prob(resp)
        loss = -min(ratio * adv, clip(ratio, 0.8, 1.2) * adv)
        optimizer.step(loss)
```

### GRPO's Limitations

1. **Group size tradeoff**: Larger G → better advantage estimates but G× more inference during training
2. **All-correct problem**: If all G responses are correct, all advantages are zero → no learning signal
3. **Reward function dependency**: Only works well with deterministic, verifiable rewards (math, code)
4. **Neural RM compatibility**: Group normalization with noisy neural reward scores is less stable

### DeepSeek-R1: GRPO at Scale

R1 used GRPO with:
- G = 4 (group size)
- Rule-based rewards: accuracy (math), test pass (code), format compliance
- 671B parameter model
- Emerged self-reflection and verification behaviors

The "Aha Moment" — where the model spontaneously learned to double-check its work — was a direct result of GRPO's group-relative reward: the model discovered that self-correction increased the probability of being above the group mean.

---

## Part 4: CISPO — Clipping for Efficiency

### Origin: MiniMax-M1 (arXiv 2506.13585)

CISPO (Clipped Importance Sampling Policy Optimization) was developed to make large-scale RL training affordable. MiniMax-M1 trained a 456B MoE on 512 H800 GPUs in 3 weeks for $534,700.

### The Key Difference

**PPO clips the probability ratio:**
$$L^{PPO} = -\min\left( r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \hat{A}_t \right)$$

**CISPO clips the importance sampling weight:**
$$L^{CISPO} = -\text{clip}\left( \frac{\pi_\theta(a|s)}{\pi_{old}(a|s)}, 1-\epsilon, 1+\epsilon \right) \cdot \hat{A}_t$$

The difference is subtle but consequential. PPO takes the minimum of clipped and unclipped objectives — this means sometimes the gradient comes from the unclipped path. CISPO always uses the clipped weight, which is more conservative but also more stable.

### Why CISPO Converges Faster

The MiniMax team found that CISPO converges in fewer steps than PPO because:
1. The clipped weight is always bounded → no extreme gradient updates
2. No "min" operation → smoother loss surface
3. Combined with Lightning Attention's O(n) complexity → much higher throughput per training step

### CISPO + Hybrid Attention

CISPO's efficiency advantage compounds with MiniMax's Hybrid Attention:

```
Standard PPO (Softmax attention):
  Forward: O(n²)
  Backward: O(n²)  
  Training step throughput: baseline

CISPO + Lightning Attention:
  Forward: O(n)
  Backward: O(n)
  Training step throughput: ~10× baseline for long sequences
```

This combination made 3-week RL training of a 456B model feasible.

---

## Part 5: When to Use What — Decision Framework

### Decision Matrix

| Scenario | Algorithm | Rationale |
|----------|-----------|-----------|
| Reasoning model, verifiable rewards | **GRPO** | Group-relative advantage is ideal for deterministic rewards |
| General chat, offline preference data | **DPO** | Simple, stable, no online generation needed |
| Budget-constrained RL at scale | **CISPO** | 3-week training vs months |
| Online RL with neural RM | **PPO** (or GRPO) | Online sampling matters for neural rewards |
| Iterative deployment (continuous) | **Iterative DPO** or **OPD** | Close the feedback loop |
| Very small models (<1B) | **PPO** | Most tested, stable at small scale |
| Quick experiments | **DPO** | Easiest to set up and debug |

### Memory Budget Comparison

For a 70B model in BF16:

| Algorithm | Models in GPU | VRAM (approx) | Feasibility |
|-----------|--------------|---------------|-------------|
| PPO | 4 | ~560 GB + optimizer | 8×A100 (80GB) minimum |
| GRPO | 3 | ~420 GB + optimizer | 4-8×A100 |
| CISPO | 3 | ~420 GB + optimizer | 4-8×A100 |
| DPO | 2 | ~280 GB + optimizer | 2-4×A100 |

### Quality Considerations

| Algorithm | Final Quality | Training Stability | Hyperparameter Sensitivity |
|-----------|--------------|-------------------|---------------------------|
| PPO | Good | Poor | High |
| DPO | Good (offline data) | Excellent | Medium ($\beta$) |
| GRPO | Best (reasoning) | Good | Medium (G, $\beta$) |
| CISPO | Very good | Very good | Low |

---

## Part 6: The Future — Algorithm Convergence

Several trends are converging:

1. **Critic-free RL** (GRPO, CISPO) is becoming the default for online training
2. **DPO derivatives** (SimPO, KTO) are making offline alignment accessible
3. **Hybrid approaches**: Qwen3 trains with both DPO and GRPO — different stages use different algorithms
4. **Cost reduction**: CISPO proved that RL training at scale can be affordable, opening the door for more teams

The next frontier: **Online Preference Distillation (OPD)** — continuously improving deployed models by distilling preference signals from production interactions. This combines the best of online learning (like GRPO) with the efficiency of distillation (like DPO).
