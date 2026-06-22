---
title: Ethan Lv
layout: home
---

<div class="hero" id="hero">
  <div class="hero-text">
    <h1>Ethan Lv</h1>
    <p class="subtitle">LLM Specialist · Agent Infrastructure · Model Alignment</p>
    <p class="location">Beijing, China</p>
  </div>
</div>

<div class="terminal-wrapper" id="terminal">
  <div class="terminal-bar">
    <span class="terminal-dot dot-red"></span>
    <span class="terminal-dot dot-yellow"></span>
    <span class="terminal-dot dot-green"></span>
    <span class="terminal-title">ethan@agent ~ type <em>help</em> to explore</span>
  </div>
  <div class="terminal-body" id="terminal-body">
    <div class="terminal-line">
      <span class="terminal-prompt">ethan@agent:~$ </span>
      <span class="terminal-welcome">Welcome to my terminal. Type <code>help</code> to see available commands.</span>
    </div>
    <div class="terminal-line" id="terminal-input-line">
      <span class="terminal-prompt">ethan@agent:~$ </span><span class="terminal-input" id="terminal-input" contenteditable="true"></span><span class="terminal-cursor" id="cursor">▊</span>
    </div>
  </div>
</div>

<div class="thinking-section">
  <h2>💭 What I'm Thinking About</h2>
  <div class="thinking-grid" id="thinking-grid">
    <div class="thinking-card">
      <div class="thinking-icon">⚡</div>
      <h3>Online Preference Distillation</h3>
      <p>Can we continuously align deployed models without costly offline retraining? Exploring lightweight distillation pipelines that close the loop between production feedback and model improvement.</p>
    </div>
    <div class="thinking-card">
      <div class="thinking-icon">🤖</div>
      <h3>Agent Evaluation Beyond Accuracy</h3>
      <p>Current benchmarks measure task completion, but real Agent quality requires measuring hallucination rate, safety adherence, and long-horizon consistency — all of which need production data.</p>
    </div>
    <div class="thinking-card">
      <div class="thinking-icon">🧠</div>
      <h3>Architecture vs Training: What Matters More?</h3>
      <p>Kimi K2 and DeepSeek-V3 share the same MLA+MoE architecture yet differ by 27 points on SWE-bench. Post-training recipe &gt; architecture. But how far does this go?</p>
    </div>
    <div class="thinking-card">
      <div class="thinking-icon">🔬</div>
      <h3>Sparse Attention for 1M+ Context</h3>
      <p>DSA/NSA show that trainable sparsity beats static patterns. The next step: can sparse attention be dynamic — allocating more compute to harder parts of the input?</p>
    </div>
    <div class="thinking-card">
      <div class="thinking-icon">🏗️</div>
      <h3>Multi-Agent Architectures at Scale</h3>
      <p>When should agents collaborate vs compete? What communication protocol works best? How do you prevent agent hallucinations from cascading through a team?</p>
    </div>
    <div class="thinking-card">
      <div class="thinking-icon">📊</div>
      <h3>Training Data Curation with LLM Judges</h3>
      <p>Using strong LLMs as data quality filters creates a dependency loop. How do we break it? Weak-to-strong generalization might be part of the answer.</p>
    </div>
  </div>
</div>





<script>
(function() {
  const body = document.getElementById('terminal-body');
  const input = document.getElementById('terminal-input');
  const cursor = document.getElementById('cursor');
  const inputLine = document.getElementById('terminal-input-line');
  
  const data = {
    about: `<strong>Ethan Lv</strong> — LLM Specialist at Tencent WXG (Tech Lead, WeChat AI Assistant).
    <br>4 years of end-to-end experience spanning data strategy, pre-training,
    post-training alignment (RLHF / DPO / GRPO / OPD), and Agent infrastructure.
    <br>Published 6 papers at NeurIPS / EMNLP / ACL, including C-Eval (NeurIPS 2023).
    <br>Currently focused on Agent orchestration, online preference optimization, and
    continual model improvement at WeChat scale (100M+ DAU).`,
    
    experience: `<strong>Tencent (Beijing)</strong> — Tech Lead, WeChat AI Assistant — May 2025–Present
    <br>• Architected Agent harness from scratch: tool-calling, planning, memory, safety guardrails
    <br>• Led post-training: SFT, DPO, GRPO, Constitutional AI, Online Preference Distillation
    <br>• Built multi-dimensional Agent evaluation matrix (tool accuracy, task completion, hallucination)
    <br>• Integrated Agent with WeChat ecosystem: Mini Programs, Official Accounts, Pay, Search
    <br><br><strong>Shenyan Technology (Beijing)</strong> — LLM Algorithm Engineer — Mar 2022–Apr 2025
    <br>• Led data strategy for 70B-scale models: 10T+ tokens, multi-stage filtering pipelines
    <br>• Designed Scaling Law experiment matrix, validated data mixture and curriculum strategies
    <br>• Executed long-context extension: 8K→32K→128K via RoPE tuning and progressive training
    <br>• Applied DPO and GRPO training for hallucination reduction and citation accuracy`,
    
    papers: `<strong>6 papers at NeurIPS / EMNLP / ACL / WSDM:</strong>
    <br>1. GATEAU: Selecting Influential Samples for Long Context Alignment — EMNLP 2025
    <br>2. Document Segmentation Matters for RAG — ACL 2025 Findings
    <br>3. HyperLoRA: Constrained Low-Rank Adapters Generation — EMNLP 2024 Findings
    <br>4. C-Eval: Chinese LLM Evaluation Suite — NeurIPS 2023 (Datasets & Benchmarks)
    <br>5. Sememe Prediction for BabelNet Synsets — ACL 2022 Findings
    <br>6. Temporal Cross-Effects in Knowledge Tracing — WSDM 2021`,
    
    skills: `<strong>LLM Training & Alignment:</strong> Scaling Laws, SFT data strategy, Iterative DPO / GRPO / OPD
    <br><strong>AI Agent Systems:</strong> Agent framework, function calling, multi-step planning, safety guardrails
    <br><strong>Data Engineering:</strong> 10T+ corpus filtering, data synthesis, automated annotation
    <br><strong>Model Evaluation:</strong> Benchmark construction (C-Eval), capability diagnosis, unleaked eval data
    <br><strong>Tech Stack:</strong> Python, PyTorch, DeepSpeed, Transformers, FSDP, vLLM, vector databases`,
    
    education: `<strong>Tsinghua University</strong>
    <br>M.S. Computer Science (2021–2024) — NLP & Large Language Models
    <br>B.S. Computer Science (2017–2021)`,
    
    contact: `📧 <a href="mailto:thulvcc2017@gmail.com">thulvcc2017@gmail.com</a>
    <br>🐙 <a href="https://github.com/lvcc2018">github.com/lvcc2018</a>
    <br>📄 <a href="/about">Full about page</a>`,
    
    help: `<strong>Available commands:</strong>
    <br>  <code>about</code>       — Who I am
    <br>  <code>experience</code>  — Work history
    <br>  <code>papers</code>      — Publications
    <br>  <code>skills</code>      — Technical skills
    <br>  <code>education</code>   — Academic background
    <br>  <code>contact</code>     — Get in touch
    <br>  <code>clear</code>       — Clear terminal
    <br>  <code>help</code>        — Show this message`,
  };

  function createLine(text, isResponse) {
    const div = document.createElement('div');
    div.className = 'terminal-line' + (isResponse ? ' terminal-response' : '');
    div.innerHTML = text;
    body.insertBefore(div, inputLine);
  }

  function createPrompt() {
    const div = document.createElement('div');
    div.className = 'terminal-line terminal-response';
    div.innerHTML = '<span class="terminal-prompt">ethan@agent:~$ </span>' + 
      '<span style="color:var(--terminal-text)">' + (input.textContent || '') + '</span>';
    body.insertBefore(div, inputLine);
  }

  input.addEventListener('keydown', function(e) {
    if (e.key === 'Enter') {
      e.preventDefault();
      const cmd = input.textContent.trim().toLowerCase();
      
      if (cmd === 'clear') {
        while (body.firstChild !== inputLine) {
          body.removeChild(body.firstChild);
        }
      } else if (data[cmd]) {
        createPrompt();
        createLine(data[cmd], true);
      } else if (cmd) {
        createPrompt();
        createLine(`<span style="color:#f87171">command not found: ${cmd}</span>. Type <code>help</code> for available commands.`, true);
      }
      
      input.textContent = '';
      body.scrollTop = body.scrollHeight;
    }
  });

  // Focus on click anywhere in terminal
  document.getElementById('terminal').addEventListener('click', function() {
    input.focus();
  });

  // Blinking cursor
  setInterval(function() {
    cursor.style.visibility = cursor.style.visibility === 'hidden' ? 'visible' : 'hidden';
  }, 530);
})();
</script>
