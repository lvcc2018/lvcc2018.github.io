---
title: "MuonClip: The Optimizer That Tamed Trillion-Parameter Training"
date: 2026-05-09
tags: [Training, Optimizer, Kimi, Scaling, Muon]
summary: "Kimi K2 is the first model to successfully apply the Muon optimizer at trillion-parameter scale, training 15.5T tokens with zero loss spikes. Their MuonClip technique — adding QK-norm clipping to the forward pass — solved the instability that had limited Muon to small-scale experiments for years."
---

## Why Not AdamW?

AdamW has been the default optimizer for LLM training since GPT-3. It is stable, well-characterized, and has predictable hyperparameters. But it is not the most token-efficient optimizer.

**Token efficiency** measures how many training tokens you need to reach a given loss. A 20% more efficient optimizer means you reach the same loss with 20% fewer tokens — saving millions of dollars at scale.

### The Muon Promise

The Muon optimizer, originally proposed by Keller Jordan et al. (2024), applies Newton-Schulz iterations to orthogonalize gradient matrices before applying updates. The core operation:

```python
def muon_update(grad, lr, momentum=0.95, nesterov=True, ns_steps=5):
    # 1. Apply momentum
    velocity = momentum * velocity + grad
    
    # 2. Reshape gradient into a matrix
    G = velocity.reshape(-1, d)  # or use the natural matrix shape
    
    # 3. Newton-Schulz orthogonalization
    # Goal: find the closest orthogonal matrix to G
    # Iteration: X_{k+1} = 1.5*X_k - 0.5*X_k @ X_k^T @ X_k
    X = G / torch.norm(G)  # normalize
    for _ in range(ns_steps):
        X = 1.5 * X - 0.5 * X @ X.T @ X
    
    # 4. Update with orthogonalized gradient
    update = X.reshape_as(grad)
    param -= lr * update
```

The intuition: orthogonal gradients decorrelate parameter updates across dimensions. When gradient components are correlated, optimization "wastes" steps moving along correlated directions. Orthogonalization ensures each dimension is updated independently, leading to faster convergence.

Muon had shown 15-30% better token efficiency than AdamW in small-scale experiments (<1B parameters). But every attempt to scale beyond ~1B hit the same wall: **training instability**.

## The Instability Problem

When Kimi's team tried Muon at scale, they observed a specific failure mode: **attention logit explosion**.

As training progressed:
1. The norms of Q and K projection weights grew
2. This caused attention logits (Q·K^T) to grow
3. Extreme logits saturated the softmax, producing near-one-hot attention patterns
4. The model stopped learning and produced a loss spike

This didn't happen with AdamW because AdamW's adaptive per-parameter learning rates implicitly bounded weight growth. Muon's orthogonalization step, while beneficial for convergence speed, didn't provide the same implicit regularization.

## The MuonClip Solution

Instead of modifying the optimizer, the Kimi team modified the forward pass:

```python
def qk_clip(Q, K, threshold):
    """
    Clip Q and K norms to prevent attention logit explosion.
    
    The scale is sqrt(q_norm * k_norm) because:
    attention_logit = Q @ K^T
    E[|attention_logit|] ∝ ||Q|| * ||K||
    
    We clip when ||Q|| * ||K|| > threshold.
    """
    q_norm = torch.norm(Q, dim=-1, keepdim=True)  # per-head norm
    k_norm = torch.norm(K, dim=-1, keepdim=True)
    
    scale = q_norm * k_norm
    
    # Identify positions where scale exceeds threshold
    mask = scale > threshold
    
    if mask.any():
        # Scale down Q and K proportionally
        factor = torch.sqrt(threshold / scale)
        Q = torch.where(mask, Q * factor, Q)
        K = torch.where(mask, K * factor, K)
    
    return Q, K
```

Applied in every attention layer:

```python
class AttentionWithQKClip(nn.Module):
    def forward(self, x):
        Q = self.W_Q(x)
        K = self.W_K(x)
        V = self.W_V(x)
        
        # QK-clip before attention computation
        Q, K = qk_clip(Q, K, threshold=self.qk_threshold)
        
        # Standard attention
        attn = softmax(Q @ K.T / sqrt(d_k))
        return attn @ V
```

### Why This Works

The insight is subtle: **you don't need to prevent weight growth — you only need to prevent its consequences.**

QK-clip doesn't modify the weights or gradients. It doesn't add a loss term. It simply caps the scale of the attention computation, preventing extreme logits regardless of what the weights do. The gradients flow through the clipping operation naturally — if the weights grow too large, the clipping reduces the effective gradient, creating an implicit regularization.

This is fundamentally different from gradient clipping (which clips the gradient norm) or weight decay (which penalizes large weights). QK-clip operates on activations, not parameters or gradients.

## Training Results

Kimi K2's pre-training with MuonClip:
- **15.5 trillion tokens** trained
- **Zero loss spikes** — not a single training interruption
- **No rollbacks** — never needed to restore from checkpoint
- 1T total parameters, 32B activated

This is unprecedented stability for a trillion-parameter model with a non-AdamW optimizer.

## MuonClip vs AdamW: Tradeoffs

| Dimension | AdamW | MuonClip |
|-----------|-------|----------|
| Token efficiency | Baseline | **~20-30% better** |
| Training stability | Excellent | Excellent (with QK-clip) |
| Implementation complexity | Low | Medium |
| Hyperparameter count | ~5 (lr, β₁, β₂, ε, λ) | ~7 (+ threshold, ns_steps) |
| Long-term track record | 5+ years | New (2025) |
| Interaction with FP8 | Well-tested | Unknown |
| Interaction with MoE | Well-tested | Works (K2 validation) |

## When Should You Use MuonClip?

**Consider MuonClip when:**
- Training a model >10B parameters where token efficiency translates to significant cost savings
- You have the engineering resources to tune the additional hyperparameters
- Training stability is a known concern (e.g., for MoE models)

**Stick with AdamW when:**
- Training a small model where the efficiency gain doesn't justify the engineering cost
- You're running a well-established training pipeline that already works
- You need to integrate with existing infrastructure that assumes AdamW

## The Future

MuonClip's success opens several research directions:

1. **MuonClip + FP8**: Does QK-clip interact with FP8 quantization? Can they work together?
2. **Adaptive thresholds**: Kimi K2 used a fixed threshold. Could an adaptive threshold that changes during training be more effective?
3. **Other optimizers with QK-clip**: Would adding QK-clip to AdamW improve its stability further?
4. **Architecture co-design**: If we know QK-clip will be applied, can we design attention mechanisms that are inherently more stable?

The broader lesson: when an optimizer fails at scale, the fix doesn't have to be in the optimizer. MuonClip succeeded by modifying the architecture (adding QK-clip to attention) rather than modifying the optimization algorithm. This pattern — architectural guardrails enabling better optimizers — may be the future of large-scale training.
