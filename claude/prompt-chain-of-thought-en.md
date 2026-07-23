---
title: "Chain of Thought Prompting: Solve Complex Problems Step by Step with Claude"
slug: prompt-chain-of-thought-en
category: claude
tags: [prompt-engineering, chain-of-thought, reasoning, advanced]
summary: "Master Chain of Thought prompting to guide Claude through step-by-step reasoning for math, logic, and complex multi-step problems."
status: published
---

## What Is Chain of Thought Prompting?

Chain of Thought (CoT) is a prompting technique that asks the AI to show its reasoning process. Instead of jumping straight to an answer, CoT makes Claude write out each step, significantly improving accuracy on complex problems.

Research shows that for tasks requiring multi-step reasoning — math, logic, code debugging — chain of thought prompting substantially reduces errors.

## The Simplest Approach

Just add one sentence to your prompt:

> Think through this step by step, then give your final answer.

**Example without CoT:**
> A pool has two pipes. Pipe A fills it alone in 6 hours, pipe B in 3 hours. How long to fill it with both open?

Claude might answer "2 hours" directly but occasionally makes mistakes.

**Example with CoT:**
> A pool has two pipes. Pipe A fills it alone in 6 hours, pipe B in 3 hours. How long to fill it with both open? Please reason step by step.

Claude will output:
1. Pipe A fills 1/6 of the pool per hour
2. Pipe B fills 1/3 of the pool per hour
3. Combined rate: 1/6 + 1/3 = 1/6 + 2/6 = 3/6 = 1/2 per hour
4. Time to fill: 1 / (1/2) = 2 hours

Showing the reasoning process lets Claude self-check and reduces calculation errors.

## Extended Thinking: Built-In Deep Reasoning

Claude offers Extended Thinking. When enabled, Claude performs internal deep reasoning before answering (users can see the thinking process). Via the API, you control depth with the `budget_tokens` parameter:

```json
{
  "thinking": {
    "type": "enabled",
    "budget_tokens": 5000
  }
}
```

Best suited for scenarios demanding high accuracy, such as mathematical proofs and complex code logic analysis.

## Few-Shot CoT: Teach Claude by Example

By providing examples that include reasoning steps, Claude mirrors the same reasoning pattern:

```
Question: Tom has 5 apples. He gives 2 to Jane and buys 3 more. How many does Tom have now?
Reasoning:
- Start: 5
- Gave away 2: 5 - 2 = 3
- Bought 3: 3 + 3 = 6
Answer: 6

Question: A store sold 45 items in the morning, 38 in the afternoon, and had 7 returns. What is the actual number of items sold?
Reasoning:
```

Claude will automatically continue using the reasoning format from the example.

## Task Decomposition Strategy

For particularly complex problems, proactively break the task into sub-questions:

```
Complete this task in three steps:
1. First, analyze the time complexity of this Python code
2. Then, identify performance bottlenecks
3. Finally, provide optimized solutions and explain why they are faster
```

This is far more effective than a single "optimize this code" because each sub-step forces Claude to reason independently.

## When to Use What

| Scenario | Recommended Technique |
|----------|----------------------|
| Math / logic problems | "Please reason step by step" |
| High-stakes decisions | Extended Thinking |
| Fixed reasoning format | Few-Shot CoT |
| Complex multi-step tasks | Proactive task decomposition |

## Frequently Asked Questions

### Does every problem need chain of thought?

No. For simple factual queries, translations, or copywriting, CoT adds unnecessary verbosity. Chain of thought works best for problems requiring multi-step reasoning — calculations, logical deductions, code debugging, and multi-criteria decisions.

### What is the difference between Extended Thinking and manual CoT?

Manual CoT is when you ask Claude to "think step by step" in your prompt. Extended Thinking is Claude's built-in capability with deeper reasoning, suited for truly challenging problems. You can combine both, but Extended Thinking alone is usually sufficient.

### Does chain of thought make responses too long?

It can. If you only need the final answer, add: "After your reasoning, provide the final answer on a separate line." This gives you the accuracy benefits of reasoning while making the conclusion easy to find.
