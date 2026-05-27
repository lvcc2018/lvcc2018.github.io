---
title: "MuonClip: How Kimi K2 Trained 1 Trillion Parameters Without a Single Loss Spike"
date: 2026-05-09
tags: [Training, Optimizer, Kimi, Scaling]
summary: "Kimi K2 is the first model to successfully apply the Muon optimizer at trillion-parameter scale. Their novel QK-clip technique solved the training instability that had previously limited Muon to small-scale experiments."
---

## Why Not AdamW?

AdamW has been the default optimizer for LLM training since GPT-3. It's stable, well-understood, and has predictable hyperparameters. But it's not the most token-efficient optimizer.

**Muon** (originally proposed by Keller Jordan et al.) promises **20-30% better token efficiency** than AdamW by applying Newton-Schulz iterations to orthogonalize the gradient matrix. The intuition: orthogonal gradients decorrelate parameter updates across dimensions, leading to faster convergence.

The problem? Muon was unstable at scale. Previous attempts to use Muon beyond ~1B parameters consistently hit training instabilities — loss spikes that would kill a training run that costs millions of dollars.

## The QK-Clip Solution

The Kimi K2 team identified the root cause: attention logit explosion. As training progressed with Muon, the norms of Q and K projections would grow, leading to increasingly extreme attention logits. This didn't happen with AdamW because AdamW's adaptive learning rates implicitly bounded the growth.

Their solution is elegant:

```python
def qk_clip(Q, K, threshold):
    q_norm = norm(Q)
    k_norm = norm(K)
    scale = q_norm * k_norm
    
    if scale > threshold:
        Q = Q * sqrt(threshold / scale)
        K = K * sqrt(threshold / scale)
    
    return Q, K
```

Instead of modifying the optimizer, they modified the forward pass. When Q and K norms grow too large, they're scaled down before computing attention — preventing extreme logits while preserving the direction of the attention pattern.

## The Result

Kimi K2 was pre-trained on **15.5 trillion tokens** with:
- **Zero loss spikes**
- **No training rollbacks**
- 1T total parameters, 32B activated

This is remarkable. Even DeepSeek-V3, which was praised for training stability, had minor recoverable instabilities. K2 with MuonClip achieved perfect stability at larger scale.

## Why MuonClip Over AdamW?

Token efficiency is the practical answer. At 15.5T tokens, a 20% efficiency gain means you reach the same loss with ~12.4T tokens — saving millions in compute.

But there's a subtler advantage: **optimizer choice interacts with data mixture**. If Muon learns differently from AdamW, it may extract more value from certain data distributions. The K2 paper doesn't explore this deeply, but it's a promising research direction.

## Practical Considerations

If you're considering MuonClip for your own training:

1. **Start small**: Verify QK-clip on a 100M-1B scale model first
2. **Tune the threshold**: Too low = over-regularization; too high = instability. K2 used a fixed threshold throughout training
3. **Monitor QK norms**: Track Q and K norm statistics during training — they're now a critical signal
4. **Interaction with RoPE**: QK-clip needs to account for RoPE's effect on attention logit magnitudes

## The Big Picture

MuonClip represents a pattern we're seeing across LLM training: **fixing optimizer limitations by modifying the architecture, not the optimizer**. DeepSeek's FP8 training used fine-grained quantization strategies rather than a fundamentally different optimizer. MiniMax's CISPO modified the RL objective rather than the optimizer. The less you change the optimizer, the easier it is to reason about training dynamics.

The next frontier: can MuonClip + FP8 training work together? DeepSeek-V4's Engram architecture with MuonClip-style optimization would be a fascinating combination.
