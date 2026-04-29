# 📏 Context Window

> **The AI's short-term memory. It's the maximum amount of text (measured in tokens) an AI can hold in its "head" at one time to process a request.**

---

## Phase 1: Core Foundations & Pre-requisites

### Prerequisites
- **Tokenization** — How text is split into tokens (words, sub-words)
- **LLM Basics** — How language models process input and generate output
- **Transformer Architecture** — Self-attention and its O(n²) complexity

### Definition
The **Context Window** (or context length) is the maximum number of **tokens** an LLM can process in a single forward pass. This includes both the **input** (system prompt + user message + any injected context) and the **output** (the model's response).

```
┌──────────────── Context Window (e.g., 128K tokens) ────────────────┐
│                                                                      │
│  [System Prompt] + [User Message] + [Retrieved Docs] + [Response]   │
│       500 tokens     200 tokens      50,000 tokens     5,000 tokens │
│                                                                      │
│  Total: 55,700 tokens ← Must be < 128,000                          │
└──────────────────────────────────────────────────────────────────────┘
```

**Key insight:** A token ≈ 0.75 words (English). So 128K tokens ≈ 96,000 words ≈ a 300-page book.

### The Problem It Solves

The context window determines:
- How much of your document the AI can read at once
- How long a conversation history the AI can remember
- How much retrieved context (RAG) you can inject

| Small Context (4K tokens) | Large Context (1M+ tokens) |
|--------------------------|---------------------------|
| Can read ~3 pages | Can read ~750 pages |
| Forgets conversation after a few turns | Remembers entire conversation |
| RAG retrieves only 2-3 chunks | Can inject hundreds of chunks |
| Cheap, fast | Expensive, slower |

**Legacy Issue:** GPT-3 (2020) had only 4K tokens (~3 pages). Complex tasks requiring large documents were simply impossible. Users had to manually summarize documents before feeding them to the AI.

### The Solution
Model providers have dramatically expanded context windows:

| Model | Context Window | Approx. Pages | Year |
|-------|---------------|---------------|------|
| GPT-3 | 4K tokens | 3 pages | 2020 |
| GPT-3.5 | 16K tokens | 12 pages | 2023 |
| GPT-4o | 128K tokens | 96 pages | 2024 |
| Claude 3.5 Sonnet | 200K tokens | 150 pages | 2024 |
| Gemini 1.5 Pro | 1M tokens | 750 pages | 2024 |
| Gemini 2.0 | 1M tokens | 750 pages | 2025 |

### Real-World Example — Legal Document Analysis
**Task:** "Analyze this 200-page contract for risky clauses."

- **4K context (2020):** Impossible in one pass. Must manually chunk and summarize.
- **128K context (2024):** Feed the entire contract. AI reads it all and identifies risks.
- **1M context (2025):** Feed the contract + 10 reference contracts + case law. AI compares and analyzes holistically.

### Trade-off Table

| Dimension | Small Context (4K-32K) | Medium (128K) | Large (1M+) |
|-----------|----------------------|---------------|-------------|
| **Cost** | 💰 Cheap | 💰💰 Medium | 💰💰💰 Expensive |
| **Latency** | ✅ Fast | 🟡 Medium | 🔴 Slower (TTFT) |
| **Accuracy on long input** | ❌ Can't process | ⚠️ "Lost in the middle" effect | ⚠️ Still has attention degradation |
| **Use case** | Chat, simple Q&A | Document analysis, RAG | Multi-document analysis, code repos |

### 🧩 Mini-Quiz

> **Q1:** If GPT-4o has a 128K context window, can you always use all 128K for your input?
> <details><summary>Answer</summary>No! The 128K includes BOTH input AND output tokens. If you fill 120K with input, only 8K is left for the response. You must budget for both. GPT-4o allows max 16K output tokens.</details>

> **Q2:** 1 token ≈ how many English words?
> <details><summary>Answer</summary>~0.75 words (or ~4 characters). So 1000 tokens ≈ 750 words. A useful rule of thumb: 1 page ≈ 500 words ≈ 670 tokens.</details>

---

## Phase 2: Anatomy & Internal Mechanisms

### Tokenization — How Text Becomes Tokens

```
Input:     "The quick brown fox"
Tokens:    ["The", " quick", " brown", " fox"]  → 4 tokens

Input:     "unbelievable"
Tokens:    ["un", "believ", "able"]              → 3 tokens (sub-word)

Input:     "GPT-4o"
Tokens:    ["G", "PT", "-", "4", "o"]            → 5 tokens
```

| Tokenizer | Used By | Avg Token/Word Ratio |
|-----------|---------|---------------------|
| **cl100k_base** | GPT-4, GPT-4o | ~1.3 tokens/word |
| **o200k_base** | GPT-4o (newer) | ~1.1 tokens/word |
| **SentencePiece** | Llama, Gemini | ~1.2 tokens/word |
| **Claude tokenizer** | Claude 3.x | ~1.3 tokens/word |

### Why Context Windows Have Limits

The core constraint is the **self-attention mechanism** in Transformers:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

- Self-attention compares every token to every other token → **O(n²) complexity**
- Doubling the context window → **4x the compute and memory**
- This is why 1M context models are expensive and slow

### The "Lost in the Middle" Problem

Research (Liu et al., 2023) showed that LLMs perform worse when key information is in the **middle** of a long context:

```
┌──────────────────── Performance across context position ────────────────────┐
│                                                                              │
│  HIGH ████                                                          ████    │
│       ████                                                          ████    │
│  MED  ████  ████                                          ████  ████████    │
│       ████  ████  ████                          ████  ████████  ████████    │
│  LOW  ████  ████  ████  ████████████████████  ████████  ████████████████    │
│                                                                              │
│       Beginning ──────────── Middle ──────────── End                         │
│                                                                              │
│  ✅ Best recall at the beginning and end of context                          │
│  ❌ Worst recall in the middle                                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Mitigation:** Place the most important information at the **beginning** and **end** of the prompt. Structure RAG results accordingly.

### Context Window vs. Working Memory

| Concept | What It Is | Limitation |
|---------|-----------|------------|
| **Context Window** | Max tokens per API call | Hard limit; exceeding = error |
| **Effective Attention** | What the model actually "focuses on" | Degrades in the middle |
| **Working Memory** | Information carried across turns (chat) | Resets every API call (stateless) |

**Key insight:** A 128K context window doesn't mean the model "remembers" 128K tokens equally well. Effective attention is much smaller.

### 🃏 Flashcard

> **Front:** What is the "Lost in the Middle" problem?
> <details><summary>Flip</summary>LLMs have lower accuracy retrieving information placed in the <b>middle</b> of a long context, compared to the <b>beginning</b> or <b>end</b>. This means when injecting RAG results, put the most relevant chunks first and last — not in the middle. From Liu et al. (2023).</details>

---

## Phase 3: Advanced / Enterprise Patterns & Pitfalls

### At Scale
- **Google NotebookLM** — Uploads entire books/papers into Gemini's 1M context
- **Cursor** — Loads relevant code files into context for AI-assisted coding
- **ChatGPT Canvas** — Maintains document + conversation in shared context
- **Claude Projects** — Persistent context across conversations (200K)

### Strategies for Context Window Management

| Strategy | How It Works | Best For |
|----------|-------------|----------|
| **RAG** | Retrieve only relevant chunks, not full documents | Large knowledge bases |
| **Summarization** | Compress long documents into summaries before injecting | Single long documents |
| **Sliding Window** | Keep most recent N messages, drop oldest | Long conversations |
| **Map-Reduce** | Process document in chunks, then combine chunk-level answers | Analysis of very long docs |
| **Hierarchical** | Summary of summaries → drill into detail on request | Multi-document analysis |

### Innovations Expanding Context

| Innovation | How | Impact |
|-----------|-----|--------|
| **RoPE (Rotary Position Embedding)** | Better position encoding that generalizes to longer sequences | Extends context without retraining |
| **Ring Attention** | Distribute attention across multiple GPUs | Enables 1M+ context |
| **Flash Attention** | Memory-efficient attention computation | 2-4x memory savings |
| **Sparse Attention** | Only attend to a subset of tokens | O(n√n) instead of O(n²) |
| **Mamba / SSM** | State-space models (not Transformers) | O(n) linear complexity |

### Edge Cases & Mitigations

| Issue | Mitigation |
|-------|------------|
| **Context overflow** | Count tokens before sending; truncate or summarize |
| **Lost in the middle** | Put key info at start/end; reorder RAG chunks |
| **Output truncation** | Reserve tokens for output (e.g., 16K out of 128K) |
| **Cost at scale** | Longer context = more tokens = higher cost → Use RAG to minimize input |
| **Latency** | First-token latency increases with context length → Cache where possible |

### Anti-Patterns

- ❌ **Stuffing entire codebase** into context — Exceeds limits, wastes tokens → Use RAG or targeted file inclusion
- ❌ **Ignoring token counting** — Request fails silently or truncates → Always count tokens before sending
- ❌ **Treating context window as memory** — It resets every API call → Use external storage for persistence
- ❌ **Using max context when you don't need it** — 128K costs 10x more than 4K → Use only what's needed

---

## Phase 4: Practical Implementation

### Token Counting (Python + tiktoken)

```python
import tiktoken

# Get the tokenizer for GPT-4o
enc = tiktoken.encoding_for_model("gpt-4o")

def count_tokens(text: str) -> int:
    """Count tokens for a given text."""
    return len(enc.encode(text))

def check_fits_context(messages: list, max_context: int = 128000, reserve_output: int = 4096):
    """Check if messages fit in context window with room for output."""
    total = sum(count_tokens(m["content"]) for m in messages)
    available = max_context - reserve_output
    fits = total <= available
    print(f"Tokens: {total:,} / {available:,} ({'✅ Fits' if fits else '❌ Too long'})")
    return fits

# Example
text = open("big_document.md").read()
print(f"Document: {count_tokens(text):,} tokens")
# Document: 45,231 tokens

check_fits_context([
    {"content": "You are a helpful assistant.", "role": "system"},
    {"content": f"Analyze this document:\n{text}", "role": "user"},
], max_context=128000, reserve_output=8000)
# Tokens: 45,252 / 120,000 (✅ Fits)
```

### Sliding Window for Long Conversations

```python
def manage_conversation_context(
    messages: list,
    max_tokens: int = 120000,
    system_prompt_tokens: int = 500
):
    """
    Keep conversation within context limit by dropping oldest messages.
    Always preserves the system prompt (first message).
    """
    enc = tiktoken.encoding_for_model("gpt-4o")
    
    # Always keep system prompt
    system_msg = messages[0]
    conversation = messages[1:]
    
    # Calculate total tokens
    total = system_prompt_tokens
    kept = []
    
    # Walk backwards (keep most recent)
    for msg in reversed(conversation):
        msg_tokens = len(enc.encode(msg["content"]))
        if total + msg_tokens > max_tokens:
            break  # Would exceed limit
        total += msg_tokens
        kept.insert(0, msg)
    
    print(f"Kept {len(kept)}/{len(conversation)} messages ({total:,} tokens)")
    return [system_msg] + kept
```

### Context Window Sizes Cheatsheet (2025)

| Model | Context | Max Output | Input Cost (per 1M tokens) |
|-------|---------|-----------|---------------------------|
| GPT-4o | 128K | 16K | $2.50 |
| GPT-4o-mini | 128K | 16K | $0.15 |
| Claude 3.5 Sonnet | 200K | 8K | $3.00 |
| Claude 3 Opus | 200K | 4K | $15.00 |
| Gemini 1.5 Pro | 1M | 8K | $1.25 |
| Gemini 2.0 Flash | 1M | 8K | $0.10 |
| Llama 3.1 405B | 128K | 4K | Self-hosted |

---

## Phase 5: Interview Preparation

### Q1: "You have a 500-page document but only 128K context. How do you analyze it?"
<details><summary><b>Answer</b></summary>

500 pages ≈ 335K tokens → exceeds 128K.

**Strategies (pick based on task):**
1. **Map-Reduce:** Split into 30 chunks → analyze each chunk independently → combine chunk summaries → final synthesis. Best for summarization.
2. **RAG:** Embed all chunks → retrieve only relevant ones per question. Best for Q&A.
3. **Hierarchical Summarization:** Summarize each section → summarize the summaries. Best for overview.
4. **Use a 1M context model:** Gemini 1.5 Pro can process the entire document. More expensive but simplest.

**Hybrid (production):** RAG for specific questions + map-reduce for global analysis.
</details>

### Q2: "Why doesn't a bigger context window solve everything?"
<details><summary><b>Answer</b></summary>

Three fundamental issues remain even with 1M context:
1. **Lost in the middle** — Accuracy degrades for info placed mid-context
2. **Cost** — 1M tokens of input = $1-15 per request (vs. $0.01 for 4K)
3. **Latency** — First-token time increases significantly with context length
4. **Attention degradation** — Model's effective "focus" is much smaller than the window

**RAG remains valuable** even with huge context windows because it's cheaper, faster, and focuses the model's attention on truly relevant content.
</details>

---

## Phase 6: Summary Cheatsheet & Action Plan

### 📋 TL;DR

| Concept | Key Point |
|---------|-----------|
| **Context Window** | Max tokens (input + output) per API call |
| **Token** | ≈ 0.75 English words, ≈ 4 characters |
| **Lost in the Middle** | Key info at start/end of context → better accuracy |
| **Why limited** | Self-attention is O(n²) — doubling context = 4x compute |
| **Management** | RAG, summarization, sliding window, map-reduce |
| **Key rule** | Budget tokens: input + desired output < context window |

### 📖 Industry Reads
1. **Paper:** [Lost in the Middle](https://arxiv.org/abs/2307.03172) — Liu et al. (2023). Essential reading.
2. **Blog:** [Google: Long context with Gemini](https://blog.google/technology/ai/long-context-window-ai-models/) — Practical 1M context applications.

### 🚀 Do These Now
1. **Count tokens (15 min):** `pip install tiktoken` → count tokens for 5 of your own documents
2. **Test Lost in the Middle (30 min):** Put a key fact at the beginning, middle, and end of a long prompt — see where the model misses it
3. **Build a sliding window (30 min):** Implement the conversation manager above for a chatbot

### 🧭 Next Topic
> How do you enforce that the AI's answers are based on facts and not made up? → [05_Grounding.md](05_Grounding.md)
