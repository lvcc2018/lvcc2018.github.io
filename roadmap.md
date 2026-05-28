---
title: LLM Roadmap
layout: raw
---

<style>
  :root { --bg: #fafafa; --text: #1a1a1f; --muted: #666; --card: #fff; --border: #e5e5e5; }
  body.dark { --bg: #111118; --text: #e4e4e7; --muted: #a1a1aa; --card: #1a1a24; --border: #27272a; }

  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
    margin: 0; padding: 0; background: var(--bg); color: var(--text);
  }

  .roadmap-header {
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
    color: #fff; padding: 80px 40px 60px; text-align: center;
  }
  .roadmap-header h1 { font-size: 2.8em; margin: 0; letter-spacing: -0.03em; font-weight: 700; }
  .roadmap-header p { font-size: 1.15em; opacity: 0.8; margin: 12px 0 0; max-width: 600px; margin-left: auto; margin-right: auto; }
  .roadmap-header a { color: #93c5fd; text-decoration: none; }
  .roadmap-header a:hover { text-decoration: underline; }

  .controls {
    max-width: 1400px; margin: 0 auto; padding: 20px 32px; display: flex; flex-wrap: wrap;
    gap: 8px; align-items: center;
  }
  .controls label { font-size: 0.85em; font-weight: 600; color: var(--muted); margin-right: 4px; }
  .controls .filter-btn {
    padding: 6px 14px; border: 1px solid var(--border); border-radius: 20px;
    background: var(--card); color: var(--muted); cursor: pointer; font-size: 0.82em;
    transition: all 0.2s;
  }
  .controls .filter-btn:hover { border-color: #aaa; }
  .controls .filter-btn.active { background: #2563eb; color: #fff; border-color: #2563eb; }

  .chart-container {
    max-width: 1400px; margin: 0 auto; padding: 20px 24px;
    background: var(--card); border-radius: 16px; border: 1px solid var(--border);
    margin-bottom: 40px; position: relative;
  }
  .chart-container h2 { font-size: 1.1em; margin: 0 0 16px; color: var(--muted); }

  .timeline-container {
    max-width: 1400px; margin: 0 auto; padding: 0 24px 60px;
  }
  .timeline-container h2 { font-size: 1.3em; margin: 0 0 24px; }
  .timeline-filters { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 24px; }

  .tl-group { margin-bottom: 40px; }
  .tl-group h3 { font-size: 1em; color: var(--muted); margin: 0 0 12px; padding-bottom: 8px; border-bottom: 1px solid var(--border); }
  .tl-cards { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 12px; }

  .tl-card {
    background: var(--card); border: 1px solid var(--border); border-radius: 10px;
    padding: 16px 18px; font-size: 0.88em; position: relative; overflow: hidden;
  }
  .tl-card::before {
    content: ''; position: absolute; left: 0; top: 0; bottom: 0; width: 3px;
    background: var(--card-accent, #6b7280);
  }
  .tl-card .tl-name { font-weight: 600; font-size: 1.05em; margin-bottom: 4px; }
  .tl-card .tl-org { color: var(--muted); font-size: 0.85em; }
  .tl-card .tl-params { margin-top: 6px; font-size: 0.9em; }
  .tl-card .tl-note { margin-top: 4px; color: var(--muted); font-size: 0.8em; line-height: 1.5; }

  footer { text-align: center; padding: 40px; color: var(--muted); font-size: 0.82em; border-top: 1px solid var(--border); }
  footer a { color: var(--muted); }

  @media (max-width: 600px) {
    .roadmap-header { padding: 50px 20px 40px; }
    .roadmap-header h1 { font-size: 1.8em; }
    .chart-container { padding: 12px; }
    .tl-cards { grid-template-columns: 1fr; }
  }
</style>

<div class="roadmap-header">
  <h1>LLM Roadmap</h1>
  <p>A comprehensive visualization of major language models from 2018 to 2026. Bubble size ≈ parameter count. <a href="/">← Back to home</a></p>
</div>

<div class="controls" id="controls">
  <label>Filter:</label>
  <button class="filter-btn active" data-org="all">All</button>
  <button class="filter-btn" data-org="OpenAI"      style="border-left: 3px solid #10a37f;">OpenAI</button>
  <button class="filter-btn" data-org="Google"       style="border-left: 3px solid #4285f4;">Google</button>
  <button class="filter-btn" data-org="Anthropic"    style="border-left: 3px solid #d97706;">Anthropic</button>
  <button class="filter-btn" data-org="Meta"         style="border-left: 3px solid #0668e1;">Meta</button>
  <button class="filter-btn" data-org="DeepSeek"     style="border-left: 3px solid #4f46e5;">DeepSeek</button>
  <button class="filter-btn" data-org="Alibaba"      style="border-left: 3px solid #ff6a00;">Alibaba</button>
  <button class="filter-btn" data-org="Moonshot"     style="border-left: 3px solid #ef4444;">Moonshot</button>
  <button class="filter-btn" data-org="MiniMax"      style="border-left: 3px solid #8b5cf6;">MiniMax</button>
  <button class="filter-btn" data-org="Zhipu"        style="border-left: 3px solid #0891b2;">Zhipu</button>
  <button class="filter-btn" data-org="Other"        style="border-left: 3px solid #6b7280;">Other</button>
</div>

<div class="chart-container">
  <h2>📊 Parameter Scale vs Time (log scale) — click legend to toggle, hover for details</h2>
  <canvas id="scatterChart" height="520"></canvas>
</div>

<div class="timeline-container" id="timeline">
  <h2>📋 Timeline</h2>
  <div class="timeline-filters" id="tl-filters"></div>
  <div id="tl-content"></div>
</div>

<footer>
  <p>Data compiled from public sources. Parameter counts are approximate. Built with Chart.js. <a href="/">lvcc2018.github.io</a></p>
</footer>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<script>
const MODELS = [
  // OpenAI
  {name:"GPT-1",       org:"OpenAI", params:0.117,  date:"2018-06-01", note:"First GPT. 117M params, 12-layer decoder-only."},
  {name:"GPT-2",       org:"OpenAI", params:1.5,    date:"2019-02-01", note:"1.5B params. Initially \"too dangerous to release\"."},
  {name:"GPT-3",       org:"OpenAI", params:175,    date:"2020-06-01", note:"175B params. Few-shot learning emerges at scale."},
  {name:"InstructGPT", org:"OpenAI", params:175,    date:"2022-03-01", note:"RLHF alignment. The precursor to ChatGPT."},
  {name:"GPT-4",       org:"OpenAI", params:1800,   date:"2023-03-01", note:"~1.8T MoE (rumored). Multimodal. SOTA for 1.5 years."},
  {name:"GPT-4o",      org:"OpenAI", params:1800,   date:"2024-05-01", note:"Omni-modal. Real-time voice. Native multimodality."},
  {name:"o1",          org:"OpenAI", params:null,   date:"2024-09-01", note:"Reasoning model. Chain-of-thought at inference time."},
  // Google
  {name:"BERT-Large",  org:"Google",  params:0.34,   date:"2018-10-01", note:"340M params. Bidirectional. Revolutionized NLP."},
  {name:"T5-11B",      org:"Google",  params:11,     date:"2019-10-01", note:"Text-to-text framework. Encoder-decoder."},
  {name:"GLaM",        org:"Google",  params:1200,   date:"2021-12-01", note:"1.2T MoE. First trillion-param model."},
  {name:"PaLM",        org:"Google",  params:540,    date:"2022-04-01", note:"540B. Pathways system. Chain-of-thought prompting."},
  {name:"PaLM 2",      org:"Google",  params:340,    date:"2023-05-01", note:"Smaller but better. Multilingual. ~340B."},
  {name:"Gemini 1.0",  org:"Google",  params:null,   date:"2023-12-01", note:"Natively multimodal. Ultra/Pro/Nano tiers."},
  {name:"Gemini 1.5",  org:"Google",  params:null,   date:"2024-02-01", note:"1M token context window. Mixture-of-experts."},
  {name:"Gemini 2.0",  org:"Google",  params:null,   date:"2024-12-01", note:"Flash thinking. Agentic capabilities."},
  // Anthropic
  {name:"Claude 1",    org:"Anthropic", params:null, date:"2023-03-01", note:"Constitutional AI. Safety-first design."},
  {name:"Claude 2",    org:"Anthropic", params:null, date:"2023-07-01", note:"100K context. Improved reasoning."},
  {name:"Claude 3",    org:"Anthropic", params:null, date:"2024-03-01", note:"Opus/Sonnet/Haiku tiers. Opus beats GPT-4 on some tasks."},
  {name:"Claude 3.5",  org:"Anthropic", params:null, date:"2024-06-01", note:"Sonnet 3.5. Coding + vision. Artifact system."},
  {name:"Claude 4",    org:"Anthropic", params:null, date:"2025-05-01", note:"Opus 4. Extended thinking. Computer use."},
  // Meta
  {name:"LLaMA",       org:"Meta",     params:65,     date:"2023-02-01", note:"7B-65B. Research-only. Over-trained on 1.4T tokens."},
  {name:"LLaMA 2",     org:"Meta",     params:70,     date:"2023-07-01", note:"2T tokens. Commercial license. Chat version."},
  {name:"LLaMA 3",     org:"Meta",     params:70,     date:"2024-04-01", note:"15T tokens. GQA. 8K context. 400B in training."},
  {name:"LLaMA 3.1",   org:"Meta",     params:405,    date:"2024-07-01", note:"405B dense. 128K context. Multilingual."},
  // DeepSeek
  {name:"DeepSeek-V2", org:"DeepSeek", params:236,    date:"2024-05-01", note:"MLA! 21B active. KV Cache -93%. MoE 160 experts."},
  {name:"DeepSeek-V3", org:"DeepSeek", params:671,    date:"2024-12-01", note:"37B active. FP8 training. $5.6M. MTP. Aux-loss-free MoE."},
  {name:"DeepSeek-R1", org:"DeepSeek", params:671,    date:"2025-01-01", note:"Pure RL reasoning. GRPO. Aha Moment. AIME 79.8%."},
  {name:"DeepSeek-V4", org:"DeepSeek", params:1074,   date:"2026-04-01", note:"~1T. 1M context. Engram architecture. Multimodal."},
  // Alibaba
  {name:"Qwen-72B",    org:"Alibaba",  params:72,     date:"2023-09-01", note:"First Qwen. Bilingual CN/EN."},
  {name:"Qwen2-72B",   org:"Alibaba",  params:72,     date:"2024-06-01", note:"GQA. 128K context. 30 languages. Apache 2.0."},
  {name:"Qwen2.5-72B", org:"Alibaba",  params:72,     date:"2024-09-01", note:"18T tokens. 1M+ SFT samples. Multi-stage RL."},
  {name:"Qwen3-235B",  org:"Alibaba",  params:235,    date:"2025-04-01", note:"Hybrid Thinking. 22B active. 119 languages."},
  {name:"Qwen3.6-35B", org:"Alibaba",  params:35,     date:"2026-04-01", note:"3B active! Extreme MoE efficiency. Agentic Coding."},
  // Moonshot
  {name:"Kimi K2",     org:"Moonshot", params:1000,   date:"2025-07-01", note:"1T MoE. 32B active. MuonClip. SWE-bench 65.8%."},
  {name:"Kimi K2.6",   org:"Moonshot", params:1000,   date:"2026-04-01", note:"Agent clusters. Claw multi-agent. Native multimodal."},
  // MiniMax
  {name:"MiniMax-01",  org:"MiniMax",  params:456,    date:"2025-01-01", note:"Lightning Attention. 45.9B active. 1M context."},
  {name:"MiniMax-M1",  org:"MiniMax",  params:456,    date:"2025-06-01", note:"Hybrid Attention. CISPO. 3 weeks $534K RL."},
  // Zhipu
  {name:"GLM-4",       org:"Zhipu",    params:130,    date:"2024-01-01", note:"128K context. All-Tools. Prefix LM architecture."},
  {name:"GLM-5.1",     org:"Zhipu",    params:null,   date:"2026-01-01", note:"DSA sparse attention. Top-tier coding model."},
  // Other
  {name:"Chinchilla",  org:"Other",    params:70,     date:"2022-03-01", note:"Compute-optimal. Token:param ≈ 20:1. Changed training forever."},
  {name:"BLOOM",       org:"Other",    params:176,    date:"2022-07-01", note:"BigScience. 46 languages. 1000+ researchers."},
  {name:"OPT-175B",    org:"Other",    params:175,    date:"2022-05-01", note:"Meta. Open replication of GPT-3. Research-only."},
  {name:"Mistral-7B",  org:"Other",    params:7,      date:"2023-09-01", note:"Sliding window attention. Apache 2.0. Punches above weight."},
  {name:"Mixtral-8x7B",org:"Other",    params:46.7,   date:"2023-12-01", note:"MoE. 12.9B active. Outperforms LLaMA 2 70B."},
  {name:"Falcon-180B", org:"Other",    params:180,    date:"2023-09-01", note:"TII. Trained on RefinedWeb. MQA attention."},
  {name:"Yi-34B",      org:"Other",    params:34,     date:"2023-11-01", note:"01.AI. Strong bilingual performance."},
  {name:"Gemma-7B",    org:"Other",    params:7,      date:"2024-02-01", note:"Google. Open weights. From Gemini research."},
];

const COLORS = {
  OpenAI:    '#10a37f',
  Google:    '#4285f4',
  Anthropic: '#d97706',
  Meta:      '#0668e1',
  DeepSeek:  '#4f46e5',
  Alibaba:   '#ff6a00',
  Moonshot:  '#ef4444',
  MiniMax:   '#8b5cf6',
  Zhipu:     '#0891b2',
  Other:     '#6b7280',
};

// ===== Scatter Chart =====
const ctx = document.getElementById('scatterChart');
let activeOrg = 'all';

function buildChart(filter) {
  const models = MODELS.filter(m => m.params != null);
  const filtered = filter === 'all' ? models : models.filter(m => m.org === filter);

  const datasets = [];
  const orgs = filter === 'all' ? Object.keys(COLORS) : [filter];

  orgs.forEach(org => {
    const orgModels = filtered.filter(m => m.org === org);
    if (orgModels.length === 0) return;
    datasets.push({
      label: org,
      data: orgModels.map(m => ({
        x: new Date(m.date).getTime(),
        y: m.params,
        r: Math.max(4, Math.sqrt(m.params) * 0.4 + 3),
        model: m,
      })),
      backgroundColor: COLORS[org] + '99',
      borderColor: COLORS[org],
      borderWidth: 1,
    });
  });

  if (window._chart) window._chart.destroy();

  window._chart = new Chart(ctx, {
    type: 'bubble',
    data: { datasets },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: { mode: 'nearest', intersect: false },
      plugins: {
        tooltip: {
          callbacks: {
            label: function(ctx) {
              const m = ctx.raw.model;
              let s = m.name + ' (' + m.org + ')';
              if (m.params) s += ' — ' + (m.params >= 1000 ? (m.params/1000).toFixed(1)+'T' : m.params+'B') + ' params';
              if (m.note) s += '\n' + m.note;
              return s;
            }
          }
        },
        legend: { position: 'bottom', labels: { usePointStyle: true, padding: 20, font: {size: 12} } }
      },
      scales: {
        x: {
          type: 'time',
          time: { unit: 'year', displayFormats: { year: 'yyyy' } },
          title: { display: true, text: 'Release Date', font: {size: 13} },
          min: new Date('2017-06-01').getTime(),
          max: new Date('2027-01-01').getTime(),
          grid: { color: '#e5e5e5' }
        },
        y: {
          type: 'logarithmic',
          title: { display: true, text: 'Parameters (Billions, log scale)', font: {size: 13} },
          ticks: {
            callback: function(v) { return v >= 1000 ? (v/1000).toFixed(1)+'T' : v+'B'; }
          },
          grid: { color: '#e5e5e5' }
        }
      }
    }
  });
}

// Filter buttons
document.getElementById('controls').addEventListener('click', function(e) {
  if (e.target.classList.contains('filter-btn')) {
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    e.target.classList.add('active');
    activeOrg = e.target.dataset.org;
    buildChart(activeOrg);
    buildTimeline(activeOrg);
  }
});

// ===== Timeline =====
function buildTimeline(filter) {
  const models = MODELS.filter(m => filter === 'all' || m.org === filter);
  // Group by year
  const grouped = {};
  models.forEach(m => {
    const yr = new Date(m.date).getFullYear();
    if (!grouped[yr]) grouped[yr] = [];
    grouped[yr].push(m);
  });

  const years = Object.keys(grouped).sort().reverse();
  let html = '';
  years.forEach(yr => {
    html += '<div class="tl-group"><h3>' + yr + '</h3><div class="tl-cards">';
    grouped[yr].sort((a,b) => new Date(b.date) - new Date(a.date)).forEach(m => {
      const pStr = m.params ? (m.params >= 1000 ? (m.params/1000).toFixed(1)+'T' : m.params+'B') : '—';
      html += '<div class="tl-card" style="--card-accent:' + (COLORS[m.org] || '#999') + '">';
      html += '<div class="tl-name">' + m.name + '</div>';
      html += '<div class="tl-org">' + m.org + ' · ' + m.date.split('-')[0] + '-' + m.date.split('-')[1] + '</div>';
      html += '<div class="tl-params">' + pStr + ' params</div>';
      if (m.note) html += '<div class="tl-note">' + m.note + '</div>';
      html += '</div>';
    });
    html += '</div></div>';
  });

  document.getElementById('tl-content').innerHTML = html;

  // Card accent
  document.querySelectorAll('.tl-card').forEach(card => {
    const accent = card.style.getPropertyValue('--card-accent');
    const before = card.querySelector('::before');
    card.style.setProperty('--accent', accent);
  });
}

// Init
document.addEventListener('DOMContentLoaded', function() {
  buildChart('all');
  buildTimeline('all');
});
</script>
