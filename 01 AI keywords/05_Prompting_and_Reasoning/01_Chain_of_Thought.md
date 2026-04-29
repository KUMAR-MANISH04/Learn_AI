# 🧠 Chain-of-Thought (CoT) Prompting

> **A technique where you force the AI to "think step-by-step" before giving the final answer. This drastically improves an AI's ability to solve math, logic, and multi-step reasoning problems.**

---

## Phase 1: Core Foundations & Pre-requisites

### Prerequisites
- **Prompting Basics** — How to write instructions for LLMs
- **Tokenization** — How models process text
- **System Prompts** — (see [02_System_Prompt.md](02_System_Prompt.md))

### Definition
**Chain-of-Thought (CoT) prompting** is a technique where the model is instructed to generate intermediate reasoning steps before producing the final answer. Instead of jumping directly to the answer, the model "shows its work."

```
Without CoT: "What is 17 × 24?" → "408" ✅ (but often wrong for harder problems)

With CoT:    "What is 17 × 24? Think step-by-step."
             → "17 × 24 = 17 × 20 + 17 × 4 = 340 + 68 = 408" ✅
```

### The Problem It Solves

| Without CoT | With CoT |
|-------------|----------|
| Model guesses the answer in one step | Model reasons through the problem |
| Fails on multi-step math (~30% accuracy) | Succeeds on multi-step math (~80%+ accuracy) |
| Can't explain its reasoning | Shows transparent reasoning chain |
| Logic errors are hidden | Logic errors are visible and debuggable |

**The breakthrough:** Wei et al. (Google, 2022) showed that simply adding "Let's think step by step" to a prompt improved GPT-3's accuracy on math benchmarks from **17.7% to 78.7%** — with zero additional training.

### CoT Variants

| Variant | How It Works | Best For |
|---------|-------------|----------|
| **Zero-shot CoT** | Add "Let's think step by step" to any prompt | Quick reasoning boost |
| **Few-shot CoT** | Provide examples WITH reasoning chains | Consistent reasoning format |
| **Self-Consistency** | Generate multiple CoT paths, take majority vote | Higher accuracy (costly) |
| **Tree of Thought** | Explore branching reasoning paths | Complex problem-solving |
| **Auto-CoT** | LLM generates its own CoT examples | Automated prompt design |

### Trade-off Table

| Dimension | Direct Prompting | Zero-shot CoT | Few-shot CoT | Self-Consistency |
|-----------|-----------------|---------------|--------------|-----------------|
| **Accuracy** | ⚠️ Low on reasoning | ✅ Much better | ✅ Best | ✅ Highest |
| **Cost** | 💰 Low (short output) | 💰💰 Medium (longer) | 💰💰 Medium | 💰💰💰 High (N samples) |
| **Latency** | ✅ Fast | 🟡 Slower | 🟡 Slower | 🔴 N× slower |
| **Setup** | 🟢 None | 🟢 One sentence | 🟡 Craft examples | 🟡 Sampling logic |

### 🧩 Mini-Quiz

> **Q1:** Why does "think step by step" improve model accuracy?
> <details><summary>Answer</summary>LLMs generate tokens sequentially. When forced to generate intermediate steps, each step provides additional context for the next token prediction. The model essentially uses its output as a "scratchpad" for reasoning, breaking complex problems into simpler sub-problems it can solve reliably.</details>

---

## Phase 2: Anatomy & Internal Mechanisms

### CoT Reasoning Flow

![Chain-of-Thought Flow](images/01_cot_flow.png)

### Zero-Shot CoT (Simplest)

Just append the magic phrase to your prompt:

```
Prompt: "If a store has 15 apples, sells 7, then receives a shipment of 12, 
         how many apples does it have? Let's think step by step."

Response:
"Step 1: Start with 15 apples
 Step 2: Sell 7: 15 - 7 = 8 apples
 Step 3: Receive 12: 8 + 12 = 20 apples
 
 The store has 20 apples."
```

### Few-Shot CoT (More Reliable)

Provide examples WITH reasoning chains:

```
Example 1:
Q: A baker has 3 trays with 12 cookies each. She gives away 8 cookies. How many left?
A: Let me work through this step by step.
   - Total cookies: 3 × 12 = 36
   - After giving away 8: 36 - 8 = 28
   - Answer: 28 cookies

Example 2:
Q: [Another example with reasoning chain]

Your question:
Q: [User's actual question]
A: Let me work through this step by step.
```

### When CoT Helps vs. Doesn't

| ✅ Helps With | ❌ Doesn't Help With |
|--------------|---------------------|
| Math word problems | Simple factual lookups |
| Logic puzzles | Translation |
| Multi-step reasoning | Summarization |
| Code debugging | Creative writing |
| Planning and strategy | Sentiment analysis |

**Rule of thumb:** CoT helps when the task requires **multiple reasoning steps**. It's unnecessary (and wasteful) for single-step tasks.

### 🃏 Flashcard

> **Front:** What is "self-consistency" in CoT?
> <details><summary>Flip</summary>Generate <b>multiple</b> CoT reasoning paths (e.g., 5-10) for the same question with temperature > 0. Take a <b>majority vote</b> on the final answers. Different paths may make different errors, but the correct answer tends to appear most often. Improves accuracy by 5-15% over single CoT at the cost of N× more tokens.</details>

---

## Phase 3: Advanced / Enterprise Patterns & Pitfalls

### Advanced Patterns

| Pattern | Description |
|---------|-------------|
| **Tree of Thought (ToT)** | Explore multiple reasoning branches; evaluate and prune; backtrack if needed |
| **ReAct** | Combine reasoning (CoT) with actions (tool use) in an interleaved loop |
| **Scratchpad** | Give the model a designated "thinking" space separate from the answer |
| **Structured CoT** | Force reasoning into a specific format (premises → inference → conclusion) |

### OpenAI o1/o3 — Built-in CoT

OpenAI's **o1** and **o3** models have chain-of-thought reasoning **built into the model**:
- They "think" internally for seconds to minutes before answering
- The reasoning tokens are hidden from the user (but you pay for them)
- Dramatically better on math, coding, and science benchmarks
- Trade-off: 10-100x more expensive and slower than GPT-4o

### Anti-Patterns

- ❌ **CoT for simple factual questions** → "What year was Python created? Think step by step." → Waste of tokens
- ❌ **Not validating the chain** → Model can produce plausible-sounding but wrong reasoning → Verify steps
- ❌ **Too-rigid format** → "You MUST use exactly 5 steps" → Let reasoning flow naturally

---

## Phase 4: Practical Implementation

```python
from openai import OpenAI
client = OpenAI()

# ── Zero-shot CoT ────────────────────────────────────
def zero_shot_cot(question: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{
            "role": "user",
            "content": f"{question}\n\nLet's think through this step by step."
        }]
    )
    return response.choices[0].message.content

# ── Self-Consistency CoT ─────────────────────────────
from collections import Counter

def self_consistency_cot(question: str, n_samples: int = 5) -> str:
    """Generate multiple reasoning paths, take majority vote."""
    answers = []
    for _ in range(n_samples):
        response = client.chat.completions.create(
            model="gpt-4o",
            temperature=0.7,  # Need diversity for self-consistency
            messages=[
                {"role": "system", "content": "Reason step by step, then give your final answer on the last line as: ANSWER: <value>"},
                {"role": "user", "content": question}
            ]
        )
        text = response.choices[0].message.content
        # Extract final answer
        if "ANSWER:" in text:
            answer = text.split("ANSWER:")[-1].strip()
            answers.append(answer)
    
    # Majority vote
    most_common = Counter(answers).most_common(1)[0]
    print(f"Votes: {Counter(answers)} → Winner: {most_common[0]} ({most_common[1]}/{n_samples})")
    return most_common[0]

# Usage
print(zero_shot_cot("If I have 3 boxes with 4 red and 2 blue balls each, and I remove all blue balls, how many balls remain?"))
print(self_consistency_cot("What is 127 × 43?", n_samples=5))
```

---

## Phase 5: Interview Preparation

### Q1: "When and why would you use Chain-of-Thought prompting?"
<details><summary><b>Answer</b></summary>

**When:** Multi-step reasoning — math, logic, code debugging, planning, analysis.
**Why:** LLMs generate tokens sequentially. Intermediate reasoning steps provide context for subsequent predictions, breaking complex problems into manageable sub-problems. Google (2022) showed this improves math accuracy from 17.7% to 78.7% with zero additional training.
**When NOT:** Simple lookups, translation, summarization — CoT adds cost and latency without benefit.
</details>

---

## Phase 6: Summary Cheatsheet & Action Plan

### 📋 TL;DR

| Concept | Key Point |
|---------|-----------|
| **CoT** | "Think step by step" → model shows reasoning → better accuracy |
| **Zero-shot** | Just add "Let's think step by step" — free, instant boost |
| **Few-shot** | Provide example reasoning chains — most reliable |
| **Self-consistency** | Multiple CoT paths + majority vote — highest accuracy |
| **When to use** | Multi-step reasoning tasks only; skip for simple queries |

### 📖 Industry Reads
1. **Paper:** [Chain-of-Thought Prompting Elicits Reasoning in LLMs](https://arxiv.org/abs/2201.11903) — Wei et al. (Google, 2022)
2. **Paper:** [Tree of Thoughts](https://arxiv.org/abs/2305.10601) — Yao et al. (2023)

### 🧭 Next Topic
> What hidden instructions shape the AI's entire behavior? → [02_System_Prompt.md](02_System_Prompt.md)
