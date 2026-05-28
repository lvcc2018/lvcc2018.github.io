---
title: LLM Roadmap
layout: default
---

<style>
.roadmap-grid { display: grid; grid-template-columns: 80px 1fr; gap: 0; margin: 32px 0 48px; position: relative; }
.roadmap-grid::before { content: ''; position: absolute; left: 35px; top: 0; bottom: 0; width: 2px; background: var(--border); }
.roadmap-year { grid-column: 1 / -1; padding: 32px 0 16px 44px; font-size: 1.3em; font-weight: 700; color: var(--text); position: relative; }
.roadmap-year::before { content: ''; position: absolute; left: 29px; top: 38px; width: 14px; height: 14px; border-radius: 50%; background: var(--accent); border: 3px solid var(--bg); z-index: 2; }
.roadmap-date { grid-column: 1; text-align: right; padding: 10px 20px 0 0; font-family: "SF Mono", Menlo, monospace; font-size: 0.8em; color: var(--text-light); }
.roadmap-node { grid-column: 2; padding: 10px 0 24px 20px; position: relative; }
.roadmap-node::before { content: ''; position: absolute; left: -6px; top: 16px; width: 10px; height: 10px; border-radius: 50%; z-index: 2; }
.roadmap-card { background: var(--card-bg); border: 1px solid var(--border); border-radius: 10px; padding: 16px 20px; box-shadow: var(--card-shadow); }
.roadmap-card h3 { margin: 0 0 4px; font-size: 1em; }
.roadmap-card .rc-org { font-size: 0.82em; color: var(--text-muted); margin-bottom: 6px; }
.roadmap-card .rc-params { font-size: 0.9em; font-weight: 600; }
.roadmap-card .rc-note { font-size: 0.82em; color: var(--text-muted); margin-top: 6px; line-height: 1.6; }
.rc-bar { display: inline-block; height: 4px; border-radius: 2px; margin-right: 6px; vertical-align: middle; }

.color-openai { background: #10a37f; } .border-openai { border: 2px solid #10a37f; }
.color-google { background: #4285f4; } .border-google { border: 2px solid #4285f4; }
.color-anthropic { background: #d97706; } .border-anthropic { border: 2px solid #d97706; }
.color-meta { background: #0668e1; } .border-meta { border: 2px solid #0668e1; }
.color-deepseek { background: #4f46e5; } .border-deepseek { border: 2px solid #4f46e5; }
.color-alibaba { background: #ff6a00; } .border-alibaba { border: 2px solid #ff6a00; }
.color-moonshot { background: #ef4444; } .border-moonshot { border: 2px solid #ef4444; }
.color-minimax { background: #8b5cf6; } .border-minimax { border: 2px solid #8b5cf6; }
.color-zhipu { background: #0891b2; } .border-zhipu { border: 2px solid #0891b2; }
.color-other { background: #6b7280; } .border-other { border: 2px solid #6b7280; }

/* Parameter scale bar */
.scale-vis { margin: 0 0 48px; }
.scale-vis h2 { margin-bottom: 16px; }
.scale-row { display: flex; align-items: center; margin-bottom: 6px; gap: 12px; }
.scale-label { width: 110px; text-align: right; font-size: 0.82em; color: var(--text-muted); flex-shrink: 0; }
.scale-bar-wrap { flex: 1; background: var(--code-bg); border-radius: 4px; height: 22px; position: relative; overflow: hidden; }
.scale-bar { height: 100%; border-radius: 4px; transition: width 0.5s; display: flex; align-items: center; justify-content: flex-end; padding-right: 8px; }
.scale-bar span { font-size: 0.72em; color: rgba(255,255,255,0.9); font-weight: 600; white-space: nowrap; }

@media (max-width: 600px) {
  .roadmap-grid { grid-template-columns: 60px 1fr; }
  .scale-label { width: 70px; font-size: 0.75em; }
}
</style>

<h1>LLM Roadmap</h1>
<p style="color: var(--text-muted); margin-bottom: 40px;">A comprehensive timeline of major language models from 2018 to 2026. Models are grouped by year, colored by organization. The bar chart below shows relative parameter scale.</p>

<h2>📊 Parameter Scale Comparison</h2>
<p style="color: var(--text-muted); font-size: 0.88em; margin-bottom: 16px;">Horizontal bars show relative parameter counts. Click to jump to the model's entry in the timeline below.</p>

<div class="scale-vis">

<h3 style="margin-top: 28px;">🧠 Trillion-Parameter Club (>500B)</h3>
<div class="scale-row">
  <div class="scale-label">GPT-4</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-openai" style="width:100%"><span>~1.8T</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">GLaM</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-google" style="width:67%"><span>1.2T</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">DeepSeek-V4</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-deepseek" style="width:60%"><span>~1T</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Kimi K2</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-moonshot" style="width:56%"><span>1T</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">DeepSeek-V3</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-deepseek" style="width:37%"><span>671B</span></div></div>
</div>

<h3 style="margin-top: 32px;">🔥 Mid-Scale (70B–500B)</h3>
<div class="scale-row">
  <div class="scale-label">PaLM</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-google" style="width:30%"><span>540B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">MiniMax-01/M1</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-minimax" style="width:25%"><span>456B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">LLaMA 3.1</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-meta" style="width:23%"><span>405B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">PaLM 2</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-google" style="width:19%"><span>~340B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">DeepSeek-V2</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-deepseek" style="width:13%"><span>236B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Qwen3-235B</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-alibaba" style="width:13%"><span>235B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Falcon</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-other" style="width:10%"><span>180B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">GPT-3 / BLOOM / OPT</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-other" style="width:10%"><span>175B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">GLM-4</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-zhipu" style="width:7%"><span>130B</span></div></div>
</div>

<h3 style="margin-top: 32px;">💪 Small Giants (<70B, over-performing)</h3>
<div class="scale-row">
  <div class="scale-label">LLaMA 1/2/3</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-meta" style="width:4%"><span>65–70B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Qwen / Qwen2</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-alibaba" style="width:4%"><span>72B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Chinchilla</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-other" style="width:4%"><span>70B</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Mixtral 8×7B</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-other" style="width:3%"><span>47B (13B active)</span></div></div>
</div>
<div class="scale-row">
  <div class="scale-label">Qwen3.6-Flash</div>
  <div class="scale-bar-wrap"><div class="scale-bar color-alibaba" style="width:2%"><span>35B (3B active!)</span></div></div>
</div>

</div>

---

<h2>📋 Timeline</h2>

<div class="roadmap-grid">

<div class="roadmap-year">2026</div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-deepseek" style="width:12px;"></span>DeepSeek-V4</h3>
  <div class="rc-org">DeepSeek</div>
  <div class="rc-params">~1T params · 1M context</div>
  <div class="rc-note">Engram architecture + mHC framework. V4-Flash and V4-Pro dual variants. Native multimodal (image understanding). 1.6× parameters of V3.</div>
</div></div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-moonshot" style="width:12px;"></span>Kimi K2.6</h3>
  <div class="rc-org">Moonshot AI</div>
  <div class="rc-params">~1T params · Agent clusters</div>
  <div class="rc-note">Claw group multi-agent collaboration. Full website generation. Office documents → Agent skills. Native multimodal.</div>
</div></div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-alibaba" style="width:12px;"></span>Qwen3.6-35B-A3B</h3>
  <div class="rc-org">Alibaba (Qwen)</div>
  <div class="rc-params">35B total / 3B active</div>
  <div class="rc-note">Extreme MoE efficiency — 8.6% activation ratio. Agentic Coding significantly enhanced. Spatial reasoning improved.</div>
</div></div>

<div class="roadmap-year">2025</div>

<div class="roadmap-date">Jul</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-moonshot" style="width:12px;"></span>Kimi K2</h3>
  <div class="rc-org">Moonshot AI</div>
  <div class="rc-params">1T total / 32B active · 384 experts</div>
  <div class="rc-note">MuonClip optimizer: zero loss spikes over 15.5T tokens. SWE-bench Verified 65.8% (agentic). Agent-first design philosophy. MLA attention. Modified MIT license.</div>
</div></div>

<div class="roadmap-date">Jun</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-minimax" style="width:12px;"></span>MiniMax-M1</h3>
  <div class="rc-org">MiniMax</div>
  <div class="rc-params">456B total / 45.9B active · 1M context</div>
  <div class="rc-note">World's first open-weight hybrid-attention reasoning model. Hybrid Attention (Lightning + Softmax). CISPO algorithm. 512 H800 GPUs, 3 weeks, $534,700 total RL training cost.</div>
</div></div>

<div class="roadmap-date">May</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-anthropic" style="width:12px;"></span>Claude Opus 4</h3>
  <div class="rc-org">Anthropic</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">Extended thinking mode. Computer use capabilities. SWE-bench leader at launch.</div>
</div></div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-alibaba" style="width:12px;"></span>Qwen3-235B-A22B</h3>
  <div class="rc-org">Alibaba (Qwen)</div>
  <div class="rc-params">235B total / 22B active</div>
  <div class="rc-note">⭐ Hybrid Thinking: one model, two modes (thinking + non-thinking). Thinking Budget mechanism for controllable reasoning depth. 119 languages. Apache 2.0. Dense + MoE dual-line.</div>
</div></div>

<div class="roadmap-date">Jan</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-deepseek" style="width:12px;"></span>DeepSeek-R1</h3>
  <div class="rc-org">DeepSeek</div>
  <div class="rc-params">671B total / 37B active</div>
  <div class="rc-note">⭐ Pure RL reasoning without SFT. GRPO + rule-based reward. Emergent self-verification, reflection, Aha Moment. AIME 2024 pass@1 = 79.8%. Distilled versions down to 1.5B.</div>
</div></div>

<div class="roadmap-date">Jan</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-minimax" style="width:12px;"></span>MiniMax-Text-01</h3>
  <div class="rc-org">MiniMax</div>
  <div class="rc-params">456B total / 45.9B active</div>
  <div class="rc-note">First large-scale Lightning Attention (Linear, O(n)). 1M training context, 4M inference context via extrapolation. 32 experts. Validated O(n) attention at scale.</div>
</div></div>

<div class="roadmap-year">2024</div>

<div class="roadmap-date">Dec</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-deepseek" style="width:12px;"></span>DeepSeek-V3</h3>
  <div class="rc-org">DeepSeek</div>
  <div class="rc-params">671B total / 37B active · 14.8T tokens</div>
  <div class="rc-note">⭐ Aux-loss-free load balancing. Multi-Token Prediction (MTP). FP8 mixed precision training — first large-scale validation. 2.788M H800 GPU hours (~$5.6M). Zero training rollbacks.</div>
</div></div>

<div class="roadmap-date">Dec</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>Gemini 2.0 Flash</h3>
  <div class="rc-org">Google DeepMind</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">Flash thinking mode. Agentic capabilities. Multimodal native.</div>
</div></div>

<div class="roadmap-date">Sep</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-alibaba" style="width:12px;"></span>Qwen2.5-72B</h3>
  <div class="rc-org">Alibaba (Qwen)</div>
  <div class="rc-params">72B · 18T tokens pre-training</div>
  <div class="rc-note">Massive data quality leap: 7T→18T tokens. >1M SFT samples. Multi-stage RL. Coder, Math, VL, Omni specializations.</div>
</div></div>

<div class="roadmap-date">Sep</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>OpenAI o1</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">First major reasoning model. Chain-of-thought at inference time. Changed the industry's understanding of inference compute scaling.</div>
</div></div>

<div class="roadmap-date">Jul</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-meta" style="width:12px;"></span>LLaMA 3.1-405B</h3>
  <div class="rc-org">Meta</div>
  <div class="rc-params">405B dense · 128K context</div>
  <div class="rc-note">Largest dense open-weight model. Multilingual. GQA attention.</div>
</div></div>

<div class="roadmap-date">Jun</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-alibaba" style="width:12px;"></span>Qwen2-72B</h3>
  <div class="rc-org">Alibaba (Qwen)</div>
  <div class="rc-params">72B · GQA · 128K native context</div>
  <div class="rc-note">Architecture upgrade to GQA. Native 128K context via large RoPE base (1M). 30 languages. Apache 2.0.</div>
</div></div>

<div class="roadmap-date">Jun</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-anthropic" style="width:12px;"></span>Claude 3.5 Sonnet</h3>
  <div class="rc-org">Anthropic</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">Coding + vision leader. Artifact system. Became developer favorite for code generation.</div>
</div></div>

<div class="roadmap-date">May</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-deepseek" style="width:12px;"></span>DeepSeek-V2</h3>
  <div class="rc-org">DeepSeek</div>
  <div class="rc-params">236B total / 21B active</div>
  <div class="rc-note">⭐ MLA (Multi-head Latent Attention)! KV Cache -93.3%. Training cost -42.5%. Throughput +5.76×. DeepSeekMoE: 160 experts + 2 shared.</div>
</div></div>

<div class="roadmap-date">May</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>GPT-4o</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">~1.8T (MoE, rumored)</div>
  <div class="rc-note">Omni-modal: text + vision + audio in a single model. Real-time voice interaction. Dramatic cost reduction vs GPT-4.</div>
</div></div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-meta" style="width:12px;"></span>LLaMA 3-70B</h3>
  <div class="rc-org">Meta</div>
  <div class="rc-params">70B · 15T tokens</div>
  <div class="rc-note">GQA (8 KV heads). 8K context. RoPE base 500K. Dramatic improvement over LLaMA 2.</div>
</div></div>

<div class="roadmap-date">Mar</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-anthropic" style="width:12px;"></span>Claude 3 (Opus/Sonnet/Haiku)</h3>
  <div class="rc-org">Anthropic</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">Opus beat GPT-4 on several benchmarks at launch. First multi-tier model family from Anthropic.</div>
</div></div>

<div class="roadmap-date">Feb</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>Gemini 1.5 Pro</h3>
  <div class="rc-org">Google DeepMind</div>
  <div class="rc-params">Undisclosed · MoE</div>
  <div class="rc-note">1M token context window — first model to reach this milestone. Mixture-of-Experts architecture.</div>
</div></div>

<div class="roadmap-date">Jan</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-zhipu" style="width:12px;"></span>GLM-4</h3>
  <div class="rc-org">Zhipu AI (Tsinghua)</div>
  <div class="rc-params">~130B</div>
  <div class="rc-note">128K context. All-Tools integration (search, code interpreter, image gen). Prefix LM architecture.</div>
</div></div>

<div class="roadmap-year">2023</div>

<div class="roadmap-date">Dec</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>Gemini 1.0 Ultra</h3>
  <div class="rc-org">Google DeepMind</div>
  <div class="rc-params">Undisclosed</div>
  <div class="rc-note">First natively multimodal Google model. Ultra/Pro/Nano tiers. Claimed to beat GPT-4 on MMLU.</div>
</div></div>

<div class="roadmap-date">Dec</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-other" style="width:12px;"></span>Mixtral 8×7B</h3>
  <div class="rc-org">Mistral AI</div>
  <div class="rc-params">46.7B total / 12.9B active</div>
  <div class="rc-note">MoE architecture. Outperforms LLaMA 2 70B at lower cost. Apache 2.0. Established Mistral as a serious player.</div>
</div></div>

<div class="roadmap-date">Nov</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-other" style="width:12px;"></span>Yi-34B</h3>
  <div class="rc-org">01.AI (Kaifu Lee)</div>
  <div class="rc-params">34B</div>
  <div class="rc-note">Strong bilingual (CN/EN) performance. First model from 01.AI. Surpassed LLaMA 2-70B on several benchmarks.</div>
</div></div>

<div class="roadmap-date">Sep</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-alibaba" style="width:12px;"></span>Qwen-72B</h3>
  <div class="rc-org">Alibaba (Qwen)</div>
  <div class="rc-params">72B</div>
  <div class="rc-note">First Qwen model. Native bilingual Chinese/English. Full parameter spectrum 1.8B–72B.</div>
</div></div>

<div class="roadmap-date">Jul</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-meta" style="width:12px;"></span>LLaMA 2-70B</h3>
  <div class="rc-org">Meta</div>
  <div class="rc-params">70B · 2T tokens</div>
  <div class="rc-note">Commercial-friendly license. Chat-optimized variant. Doubled context to 4K. GQA attention.</div>
</div></div>

<div class="roadmap-date">Mar</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>GPT-4</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">~1.8T (8×220B MoE, rumored)</div>
  <div class="rc-note">Multimodal (text + image input). SOTA on nearly every benchmark for 1.5 years. Sparked global AI race.</div>
</div></div>

<div class="roadmap-date">Feb</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-meta" style="width:12px;"></span>LLaMA-65B</h3>
  <div class="rc-org">Meta</div>
  <div class="rc-params">65B · 1.4T tokens</div>
  <div class="rc-note">"Over-trained" — token:param ratio far above Chinchilla. Proved data quality + over-training beats scaling. Sparked open-source LLM movement.</div>
</div></div>

<div class="roadmap-year">2022</div>

<div class="roadmap-date">Jul</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-other" style="width:12px;"></span>BLOOM-176B</h3>
  <div class="rc-org">BigScience</div>
  <div class="rc-params">176B</div>
  <div class="rc-note">Largest open collaboration in ML history. 1000+ researchers, 46 languages, 1.6TB training data. Democratized LLM access.</div>
</div></div>

<div class="roadmap-date">Apr</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>PaLM-540B</h3>
  <div class="rc-org">Google</div>
  <div class="rc-params">540B</div>
  <div class="rc-note">Pathways system: 6144 TPUv4. Chain-of-thought prompting breakthrough. Largest dense model at release.</div>
</div></div>

<div class="roadmap-date">Mar</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-other" style="width:12px;"></span>Chinchilla-70B</h3>
  <div class="rc-org">DeepMind</div>
  <div class="rc-params">70B · 1.4T tokens</div>
  <div class="rc-note">⭐ Compute-optimal scaling laws: token:param ≈ 20:1. Proved most models are under-trained. Changed how everyone trains LLMs.</div>
</div></div>

<div class="roadmap-date">Mar</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>InstructGPT</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">175B (fine-tuned GPT-3)</div>
  <div class="rc-note">⭐ RLHF (Reinforcement Learning from Human Feedback) at scale. The technical foundation of ChatGPT. Introduced the alignment tax concept.</div>
</div></div>

<div class="roadmap-year">2021</div>

<div class="roadmap-date">Dec</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>GLaM</h3>
  <div class="rc-org">Google</div>
  <div class="rc-params">1.2T total / 96.6B active</div>
  <div class="rc-note">First trillion-parameter model. Mixture-of-Experts. Proved MoE viability at extreme scale.</div>
</div></div>

<div class="roadmap-year">2020</div>

<div class="roadmap-date">Jun</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>GPT-3-175B</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">175B · 300B tokens</div>
  <div class="rc-note">⭐ "Language Models are Few-Shot Learners". In-context learning emerges. Proved scaling works. Sparked the modern LLM era. API business model.</div>
</div></div>

<div class="roadmap-year">2019</div>

<div class="roadmap-date">Oct</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>T5-11B</h3>
  <div class="rc-org">Google</div>
  <div class="rc-params">11B</div>
  <div class="rc-note">"Text-to-Text Transfer Transformer". Unified all NLP tasks into text-to-text format. Encoder-decoder architecture.</div>
</div></div>

<div class="roadmap-date">Feb</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>GPT-2-1.5B</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">1.5B</div>
  <div class="rc-note">"Too dangerous to release" (initially). Zero-shot task transfer without fine-tuning. Staged release sparked AI safety debate.</div>
</div></div>

<div class="roadmap-year">2018</div>

<div class="roadmap-date">Oct</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-google" style="width:12px;"></span>BERT-Large</h3>
  <div class="rc-org">Google</div>
  <div class="rc-params">340M</div>
  <div class="rc-note">⭐ Bidirectional pre-training. Revolutionized NLP — 11 tasks to SOTA. Masked Language Modeling paradigm. Foundation for RAG and search.</div>
</div></div>

<div class="roadmap-date">Jun</div>
<div class="roadmap-node"><div class="roadmap-card">
  <h3><span class="rc-bar color-openai" style="width:12px;"></span>GPT-1</h3>
  <div class="rc-org">OpenAI</div>
  <div class="rc-params">117M</div>
  <div class="rc-note">"Improving Language Understanding by Generative Pre-Training". The origin of the GPT series. 12-layer decoder-only transformer.</div>
</div></div>

</div>
