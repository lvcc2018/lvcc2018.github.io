---
title: "MLA Deep Dive: How DeepSeek Compresses KV Cache by 93%"
date: 2026-05-09
tags: [DeepSeek, Architecture, Inference, Attention]
summary: "Multi-head Latent Attention (MLA) is the key innovation from DeepSeek-V2 that reduces KV Cache by 93.3% while boosting throughput 5.76x. Let's break down exactly how it works."
---

## The Problem

In transformer inference, the KV Cache grows linearly with both sequence length and batch size. For a model like LLaMA 70B with 64 Q heads and 8 KV heads (GQA), a single request with 128K context requires:

```
KV Cache = 2 × 1 × 128K × 8 × 128 × 2 bytes = 512 MB
```

That's just one request. At batch 32, it's **16 GB** for KV Cache alone — before the model weights.

Standard approaches to this problem:
- **MQA** (Multi-Query Attention): Share one KV head across all Q heads → 1/H compression, but quality degrades
- **GQA** (Grouped-Query Attention): Group Q heads to share KV heads → G/H compression, a compromise

MLA takes a fundamentally different approach.

## What MLA Does

Instead of sharing KV heads across Q heads, MLA compresses the KV representation into a **low-rank latent space** before storing it. During inference, only the compressed latent vector is cached — not the full K and V matrices.

```
Traditional MHA:
  K = X @ W_K      → store full K  (d_model → d_kv)
  V = X @ W_V      → store full V  (d_model → d_kv)

MLA:
  C = X @ W_DKV    → store only C  (d_model → d_c, where d_c ≪ d_kv)
  K = C @ W_UK     → reconstruct K from C when needed
  V = C @ W_UV     → reconstruct V from C when needed
```

In DeepSeek-V2:
- `d_c = 512` (latent dimension)
- `d_kv = 16384` (128 heads × 128 dim)

That's a **32× compression** from the latent representation alone.

## The RoPE Problem

Rotary Position Embedding (RoPE) applies a rotation to K based on position. But MLA's K is reconstructed from C — you can't apply RoPE to C and get the same result as applying RoPE to K.

DeepSeek's solution: **Decoupled RoPE**.

```
K_total = K_content + K_rope

K_content  = C @ W_UK     ← compressed path (most dimensions)
K_rope     = X @ W_KR     ← direct path, RoPE applied (small fraction of dimensions)
```

The RoPE portion (`K_rope`) is kept small — just enough to encode position information — while the bulk of K goes through the compressed path.

## The Numbers

From DeepSeek-V2 paper (arXiv 2405.04434):

| Metric | Improvement |
|--------|------------|
| KV Cache reduction | **93.3%** |
| Training cost reduction | **42.5%** (vs DeepSeek 67B) |
| Max generation throughput | **5.76×** |

## Why This Matters for 1M Context

MLA's compression ratio becomes more valuable as context length grows. At 1M tokens:

- **Without MLA** (GQA, H=64, G=8): KV Cache = 2 × 1 × 1M × 8 × 128 × 2 = **4 GB** per request
- **With MLA** (approximate): KV Cache = 2 × 1 × 1M × 512 × 2 ≈ **2 GB** per request (compressed) + RoPE overhead

But MLA alone isn't enough for 1M — you also need sparse attention (DSA) and efficient memory management (Engram in V4). MLA handles the storage bottleneck; DSA handles the compute bottleneck.

## MLA in the Wild

MLA has been adopted beyond DeepSeek:

- **Kimi K2**: 1T MoE with MLA (arXiv 2507.20534)
- **GLM-5.1**: MLA + Indexer for sparse attention
- **FlashMLA**: Dedicated kernel-level optimization for MLA inference (open-sourced by DeepSeek)

## Key Interview Question

> "Why can't you just store a smaller K and V?"

Because reducing the dimension directly reduces expressiveness — every head's attention pattern depends on the full `d_kv`-dimensional representation. MLA solves this by learning a compression that preserves the information needed for reconstruction, rather than just truncating dimensions. It's the difference between JPEG compression and simple downscaling.
