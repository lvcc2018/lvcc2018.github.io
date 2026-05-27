---
title: "The Data Flywheel: How Agent Feedback Improves Post-Training"
date: 2026-05-09
tags: [Agent, Training, Data, Production]
summary: "The most underrated component of Agent infrastructure isn't the model or the framework — it's the data flywheel. How do you turn production Agent interactions into training data that continuously improves the model? Here's the architecture."
---

## The Flywheel Pattern

A data flywheel is a closed loop:

```
Production Data → Failure Analysis → Training Data → Model Update → Better Performance → More Usage → More Production Data
```

It sounds simple. Implementing it at scale is not.

## Stage 1: Data Collection

Every Agent interaction produces a trace:

```json
{
  "session_id": "...",
  "user_query": "Book me a flight to Shanghai next Tuesday",
  "agent_actions": [
    {"tool": "search_flights", "params": {...}, "result": {...}},
    {"tool": "check_calendar", "params": {...}, "result": {...}},
    {"tool": "book_flight", "params": {...}, "result": "error: seat unavailable"}
  ],
  "final_response": "The 9am flight is full. Would you like the 11am instead?",
  "user_feedback": null,
  "task_completed": false
}
```

The key design decisions:
- **What to log**: Everything. Storage is cheap; missed insights are expensive.
- **Privacy boundary**: Strip PII before storage; log in aggregate, not per-user
- **Latency overhead**: Async logging; never block the user on telemetry

## Stage 2: Failure Analysis

Not every unsuccessful trace is useful for training. The failure needs to be **actionable**:

| Failure Pattern | Trainable? | Training Signal |
|----------------|-----------|----------------|
| Wrong tool selected | ✅ | Correct tool as positive example |
| Correct tool, wrong params | ✅ | Correct params as positive example |
| Tool result misinterpreted | ✅ | Correct interpretation |
| User changed their mind | ❌ | Not a model failure |
| External API down | ❌ | Not a model failure |
| Ambiguous user query | ✅ | Clarification as positive example |

**Key insight**: Only ~30-40% of production failures are actually trainable. The rest are either external issues or inherent task ambiguity. Filtering these out is critical — training on noise degrades performance.

## Stage 3: Training Data Construction

For each trainable failure, construct a preference pair:

```python
def construct_preference_pair(failed_trace, corrected_action):
    return {
        "prompt": failed_trace["user_query"],
        "chosen": corrected_action,      # What the model should have done
        "rejected": failed_trace["agent_actions"],  # What it actually did
        "source": "production_flywheel",
        "failure_category": "wrong_tool_params"
    }
```

This data can be used for:
- **DPO**: Direct preference optimization on (chosen, rejected) pairs
- **SFT**: Supervised fine-tuning on chosen-only examples
- **GRPO**: Reward signal for reinforcement learning

The choice depends on failure volume:
- <1000 pairs: Use SFT (safer, less prone to overfitting)
- 1000-10000 pairs: Use DPO (better utilization of negative examples)
- >10000 pairs: Consider GRPO with automated reward

## Stage 4: Model Update

This is the bottleneck. Full model retraining is expensive. Practical alternatives:

| Method | Update Cost | Quality | Best For |
|--------|------------|---------|----------|
| LoRA fine-tuning | Hours, 1 GPU | Good | Quick fixes |
| Full SFT | Days, cluster | Better | Quarterly updates |
| Iterative DPO | Days, cluster | Best | Continuous improvement |
| Online Preference Distillation (OPD) | Continuous | Best | Post-deployment alignment |

**OPD** is particularly promising: instead of periodic retraining, it continuously distills preference signals into the deployed model. Think of it as "online learning for alignment" — the model improves between full training cycles.

## Stage 5: Closing the Loop

The final step is measuring whether the update actually improved things:

```
Pre-update metrics → Deploy update → Post-update metrics → Compare
                                                              │
                                          Rollback if degraded ←─
```

Critical: always run an A/B test. Model updates that improve offline metrics can degrade online performance due to distribution shift.

## The Flywheel in Practice

At WeChat scale (100M+ DAU), the flywheel produces:
- Millions of Agent traces per day
- Thousands of trainable failures per day
- Weekly model updates via LoRA
- Monthly full retraining cycles

The flywheel doesn't just improve performance — it changes how you think about model development. You stop asking "how do we make the model better?" and start asking "what data do we need to collect to make the model better?"

The best Agent systems aren't the ones with the best initial models. They're the ones with the best flywheels.
