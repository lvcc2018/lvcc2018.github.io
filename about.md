---
title: About · 关于我
layout: default
---

# 关于我

## 工作经历

### 腾讯 · WXG · 大模型算法工程师
*2024 — 至今 · T10*

负责大模型 **Agent 方向** 的算法研究、系统设计与工程落地。

**Agent 架构**
- 多 Agent 协作框架：Manager-Worker 编排、共享上下文通信
- Tool Use：OpenAI/Anthropic Function Calling 标准化、MCP 协议
- Planning：ReAct / Plan-Then-Execute，支持动态重规划
- 记忆系统：三层架构（Working / Short-term / Long-term Memory）

**模型对齐**
- SFT 数据管线：质量筛选、多样性控制、Evol-Instruct
- DPO 训练：离线偏好优化，$\beta$ 调参，Iterative DPO
- GRPO 探索：Reasoning RL，Group-wise advantage estimation

**推理优化**
- vLLM 集群部署：PagedAttention、Continuous Batching、Prefix Caching
- KV Cache 优化：INT8 量化、GQA 配置、Chunked Prefill
- 吞吐优化：投机解码、语义缓存、模型量化（AWQ/GPTQ）

### 早期经历
*2022 — 2024*

- NLP 算法工程师：预训练语言模型、文本生成、信息抽取
- 大规模数据处理管线：CommonCrawl 过滤、MinHash 去重、质量分类

---

## 教育背景

**硕士** · 计算机科学与技术 · 985 高校

---

## 开源项目

### QuantAce — A 股量化策略研究平台
[github.com/lvcc2018/QuantAce](https://github.com/lvcc2018/QuantAce)

```
111 只 A 股 · 28 个因子 · 10 个策略 · 涨跌停模拟
```

- 数据源：AKShare → Parquet 缓存
- 回测引擎：Backtrader + 自定义涨跌停限制
- 因子 IC 分析：因子 Ranker + IC 权重动态调整
- 机器学习：LightGBM 收益预测
- 定时任务：每交易日 15:30 自动同步数据，16:00 信号扫描

---

## 联系我

- **Email**: [thulvcc2017@gmail.com](mailto:thulvcc2017@gmail.com)
- **GitHub**: [github.com/lvcc2018](https://github.com/lvcc2018)
