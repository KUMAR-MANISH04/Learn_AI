# 📝 Few-Shot Prompting

> **Giving the AI a few examples of exactly what you want inside the prompt to guide its output style, format, and reasoning.**

---

## Phase 1: Core Foundations & Pre-requisites

### Prerequisites
- **System Prompts** — Role-setting and instructions (see [02_System_Prompt.md](02_System_Prompt.md))
- **API Message Format** — system/user/assistant roles

### Definition
**Few-shot prompting** is a technique where you include **N examples** (typically 2-5) of the desired input→output behavior directly in the prompt. The model learns the pattern from these examples and applies it to the new query — without any training or fine-tuning.

```
Zero-shot:  "Classify the sentiment: 'I love this product!'"
            → Model guesses format and labels

Few-shot:   "Examples:
             'Great quality!' → Positive
             'Terrible service' → Negative
             'It's okay' → Neutral
             
             Now classify: 'I love this product!'"
            → Model follows exact format: "Positive"
```

### The Spectrum

| Shots | Name | Description | Reliability |
|-------|------|-------------|-------------|
| 0 | **Zero-shot** | No examples; rely on instructions only | ⚠️ Variable |
| 1 | **One-shot** | Single example | 🟡 Better |
| 2-5 | **Few-shot** | Multiple examples showing the pattern | ✅ Reliable |
| 10+ | **Many-shot** | Large number of examples (with long context) | ✅ Most reliable |

### The Problem It Solves

| Zero-shot (no examples) | Few-shot (with examples) |
|------------------------|-------------------------|
| Model interprets format freely | Model matches exact format |
| Inconsistent output structure | Consistent, predictable output |
| May use wrong labels/categories | Uses exactly the labels you provided |
| Generic style | Matches the example style |

### Real-World Example

**Task:** Extract structured data from support emails.

```
Examples:

Email: "Hi, my order #12345 arrived damaged. I need a replacement ASAP."
Output: {"order_id": "12345", "issue": "damaged", "urgency": "high", "action": "replacement"}

Email: "Just wondering when my order #67890 will ship?"
Output: {"order_id": "67890", "issue": "shipping_inquiry", "urgency": "low", "action": "status_update"}

Now extract:
Email: "Order #11111 has the wrong item. Please send the correct one."
Output:
```

The model will produce: `{"order_id": "11111", "issue": "wrong_item", "urgency": "medium", "action": "replacement"}` — matching the exact JSON format.

### Trade-off Table

| Dimension | Zero-Shot | Few-Shot | Fine-Tuning |
|-----------|-----------|----------|-------------|
| **Setup time** | 🟢 Instant | 🟢 Minutes | 🔴 Hours-days |
| **Consistency** | ⚠️ Variable | ✅ High | ✅ Highest |
| **Cost per query** | 💰 Low | 💰💰 Medium (more tokens) | 💰 Low (shorter prompts) |
| **Format reliability** | ❌ Often wrong | ✅ Usually correct | ✅ Always correct |
| **When to use** | Quick tests | Production with <100 formats | High-volume, exact format |

### 🧩 Mini-Quiz

> **Q1:** When should you use few-shot prompting vs. fine-tuning?
> <details><summary>Answer</summary>Few-shot: when you need quick, consistent formatting with under 1M daily requests. The examples consume tokens every call, so at scale it's expensive. Fine-tuning: when you need the same behavior at high volume — the format is baked in, so prompts are shorter and cheaper. Rule of thumb: < 10K requests/day → few-shot. > 100K → consider fine-tuning.</details>

---

## Phase 2: Anatomy & Internal Mechanisms

### How Few-Shot Works Inside the Model

The model processes few-shot examples as **in-context learning**:

```
[System Prompt] [Example 1] [Example 2] [Example 3] [New Query]
                 ↑            ↑            ↑           ↑
                 Pattern recognition via self-attention
                 Model attends to examples to infer the pattern
```

The model doesn't "learn" in the training sense — it **infers the pattern** from examples via self-attention. This is why example quality matters enormously.

### Designing Effective Examples

| Principle | Why | Example |
|-----------|-----|---------|
| **Diverse** | Cover different cases | Positive, negative, AND neutral sentiment |
| **Representative** | Match real-world distribution | If 60% of queries are about orders, 60% of examples should be |
| **Edge cases** | Show how to handle tricky inputs | Empty input, ambiguous text, multiple issues |
| **Consistent format** | Model copies format exactly | Same JSON keys, same ordering every time |
| **Correct** | Wrong examples = wrong outputs | Double-check every example label |

### Few-Shot for Different Tasks

| Task | # Examples Needed | Key in Examples |
|------|------------------|-----------------|
| **Classification** | 1-2 per class | Show all possible labels |
| **Extraction** | 3-5 | Show field mapping clearly |
| **Format conversion** | 2-3 | Show input→output transformation |
| **Style matching** | 3-5 | Show tone and vocabulary |
| **Reasoning** | 2-3 (with CoT) | Show reasoning chains |

### 🃏 Flashcard

> **Front:** What is "in-context learning" and how does it relate to few-shot?
> <details><summary>Flip</summary>In-context learning is the model's ability to <b>infer a task pattern from examples</b> provided in the prompt, without any weight updates. Few-shot prompting exploits this: the model uses self-attention to compare the new query against the examples and produces output that follows the same pattern. This is an emergent capability — it wasn't explicitly trained, but appears in large-scale models.</details>

---

## Phase 3: Advanced / Enterprise Patterns & Pitfalls

### Advanced Techniques

| Technique | Description |
|-----------|-------------|
| **Dynamic few-shot** | Retrieve the most relevant examples per query from an example bank |
| **Chain-of-Thought few-shot** | Include reasoning chains in examples for better reasoning |
| **Negative examples** | Show what NOT to output: "Input: X → Output: NOT Y, because..." |
| **Many-shot (10-100)** | Use long context models to include many more examples |

### Anti-Patterns

- ❌ **Wrong examples** → Model will copy mistakes faithfully → Triple-check accuracy
- ❌ **All same category** → 3 positive examples → model defaults to positive → Include all classes
- ❌ **Too many examples** → 20+ examples = expensive tokens → 3-5 is usually enough
- ❌ **Examples don't match real data** → Clean examples but messy real data → Include realistic examples

---

## Phase 4: Practical Implementation

```python
from openai import OpenAI
client = OpenAI()

# ── Few-Shot Classification ─────────────────────────
def classify_ticket(ticket_text: str) -> str:
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Classify support tickets. Respond with only the category."},
            # Few-shot examples
            {"role": "user", "content": "My payment failed and I was charged twice"},
            {"role": "assistant", "content": "billing"},
            {"role": "user", "content": "I can't log into my account"},
            {"role": "assistant", "content": "authentication"},
            {"role": "user", "content": "The app crashes when I open settings"},
            {"role": "assistant", "content": "bug"},
            {"role": "user", "content": "How do I change my subscription plan?"},
            {"role": "assistant", "content": "account_management"},
            # Actual query
            {"role": "user", "content": ticket_text},
        ],
        temperature=0
    )
    return response.choices[0].message.content

# ── Dynamic Few-Shot (retrieve relevant examples) ───
from chromadb import PersistentClient

example_db = PersistentClient(path="./examples_db")
collection = example_db.get_or_create_collection("few_shot_examples")

def dynamic_few_shot(query: str, n_examples: int = 3) -> str:
    """Retrieve the most relevant examples for this specific query."""
    results = collection.query(query_texts=[query], n_results=n_examples)
    
    messages = [{"role": "system", "content": "Extract structured data. Follow the examples exactly."}]
    
    for doc, meta in zip(results["documents"][0], results["metadatas"][0]):
        messages.append({"role": "user", "content": doc})
        messages.append({"role": "assistant", "content": meta["output"]})
    
    messages.append({"role": "user", "content": query})
    
    response = client.chat.completions.create(model="gpt-4o-mini", messages=messages, temperature=0)
    return response.choices[0].message.content
```

---

## Phase 5: Interview Preparation

### Q1: "Few-shot vs. fine-tuning for a classification task with 50K daily requests?"
<details><summary><b>Answer</b></summary>

**Few-shot cost:** 5 examples × ~50 tokens each = 250 extra tokens/request. At 50K requests/day: 12.5M tokens/day = ~$1.88/day (GPT-4o-mini).

**Fine-tuned cost:** No examples needed → shorter prompts → ~$0.50/day.

**Recommendation:** At 50K daily, the cost difference is small (~$1.38/day). Start with few-shot for faster iteration, then fine-tune once the classification labels and examples are stable. Fine-tuning also gives slightly higher consistency (99%+ vs. 95% format compliance).
</details>

---

## Phase 6: Summary Cheatsheet & Action Plan

### 📋 TL;DR

| Concept | Key Point |
|---------|-----------|
| **Few-shot** | Include 2-5 examples in the prompt → model copies the pattern |
| **In-context learning** | Model infers the task from examples via attention — no training |
| **Example quality** | Diverse, representative, correct examples are critical |
| **Dynamic few-shot** | Retrieve relevant examples per query from a vector DB |
| **Cost trade-off** | More examples = more tokens/request. At scale → consider fine-tuning |

### 📖 Industry Reads
1. **Paper:** [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165) — Brown et al. (GPT-3 paper, 2020)

### 🧭 Continue Learning
> You've completed the "Prompting & Reasoning" module! → [README](README.md)
