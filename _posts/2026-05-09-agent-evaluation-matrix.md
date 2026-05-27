---
title: "Building an Agent Evaluation Matrix: Lessons from Production"
date: 2026-05-09
tags: [Agent, Evaluation, Production, Engineering]
summary: "How do you measure an Agent's real-world performance? Beyond accuracy — hallucination rate, tool-call correctness, safety boundary adherence, and long-horizon consistency. Here's the evaluation framework that works at scale."
---

## Beyond Accuracy

Evaluating a chatbot is straightforward: is the response helpful, harmless, and honest? You can use human preference labels or LLM-as-judge.

Evaluating an Agent is fundamentally harder. An Agent doesn't just produce text — it takes actions. And those actions have consequences that may only be visible several steps later. A correct tool call with incorrect parameters is a failure. A correct tool call that leads to a dead end three steps later is also a failure.

## The Evaluation Matrix

After building Agent systems at production scale, here's the evaluation taxonomy that actually works:

### 1. Single-Step Tool Accuracy

The most basic metric: when the Agent decides to call a tool, does it call the right one with the right parameters?

```
Tool Accuracy = correct_tool_calls / total_tool_calls

Breakdown:
  - Tool selection accuracy: Did I pick the right tool?
  - Parameter correctness: Are the arguments valid?
  - Parameter completeness: Are all required parameters present?
```

### 2. Multi-Step Task Completion Rate

This is where most Agent evaluations fall short. A single correct tool call doesn't mean the task was completed.

```
Task Completion = completed_tasks / total_tasks

A task is "completed" when:
  - The user's goal is achieved (not just "the last tool call succeeded")
  - The final response correctly reflects the tool results
  - No unnecessary tool calls were made
```

**Critical insight**: Task completion rate is always lower than tool accuracy. A 95% single-step accuracy can translate to 70% task completion if the errors compound across steps.

### 3. Hallucination Rate in Tool Context

Agents hallucinate differently from chatbots. The dangerous hallucinations are:

- **Tool hallucination**: Calling a tool that doesn't exist
- **Parameter hallucination**: Inventing parameter values (dates, IDs, names)
- **Result fabrication**: Claiming tool results that weren't actually returned

### 4. Safety Boundary Adherence

```
Safety metrics:
  - Refusal rate on dangerous requests: should be ~100%
  - Over-refusal rate on benign requests: should be <2%
  - Tool-call safety: blocking dangerous tool invocations before execution
```

### 5. Long-Horizon Consistency

Does the Agent maintain coherent behavior across 10+ tool-calling turns?

```
Consistency metrics:
  - Context retention: Does it remember earlier tool results?
  - Plan adherence: Does it stick to the plan or drift?
  - Error recovery: When a tool fails, does it retry with corrections?
```

## The Data Flywheel

The evaluation matrix isn't just for measurement — it's the engine of a data flywheel:

```
1. Deploy model → 2. Collect Agent traces
      ↑                        ↓
4. Retrain model ← 3. Identify failure patterns
```

Each failure pattern becomes a training data point. A hallucinated tool call becomes a negative example for DPO. A successful error recovery becomes a positive example for SFT.

## Tooling

The evaluation pipeline at scale:

```
Agent Traces → Structured Logging → Metric Computation → Dashboard
     │              │                      │
     │              ├─ Tool call schema    ├─ Per-model breakdown
     │              ├─ Tool call params    ├─ Per-task breakdown  
     │              ├─ Tool results        ├─ Trend over time
     │              └─ Final response      └─ Failure categorization
```

Key design decisions:
1. **Log everything**: Every tool call, parameter, result, and intermediate thought
2. **Categorize failures**: Not just "wrong" but "wrong tool", "wrong param", "missing param", "hallucinated result"
3. **Track trends**: Is Agent performance improving or degrading over time?

## Why Most Benchmarks Miss the Point

SWE-bench, Tau2-Bench, and TerminalBench are valuable research tools. But they don't capture:

- **Distribution shift**: Your users' tasks are different from the benchmark's
- **Latency constraints**: A correct answer in 30 seconds may be useless in production
- **Cost constraints**: A perfect Agent that costs $1/query won't work at scale
- **Safety tradeoffs**: Benchmarks don't measure over-refusal

The most useful evaluation is always the one you build yourself, on your own data, measuring what matters to your users.
