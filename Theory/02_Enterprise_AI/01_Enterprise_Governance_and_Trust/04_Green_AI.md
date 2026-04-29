# 🌲 Green AI

> **The push to develop AI models that are computationally efficient and environmentally sustainable, aiming to drastically reduce the massive carbon footprint and energy/water costs required to run giant data centers.**

---

## Phase 1: Core Foundations & Pre-requisites

### Prerequisites
- **LLMs vs SLMs** — The difference between massive and small models.
- **Inference vs. Training** — The two phases of AI compute.
- **PEFT / Quantization** — Efficiency techniques (see [Module 4](../../04_Training_and_Tweaking/02_PEFT.md)).

### Definition
**Green AI** is the research and engineering discipline focused on making AI environmentally sustainable and economically efficient. It stands in direct contrast to **"Red AI"**—the practice of chasing slightly higher accuracy by throwing exponentially more compute, electricity, and data at massive models.

As enterprise AI adoption scales, the physical limits of electricity generation and data center cooling (water consumption) are becoming hard blockers for businesses.

### The Problem It Solves

| Red AI (The Problem) | Green AI (The Solution) |
|----------------------|-------------------------|
| Training a single massive LLM emits hundreds of tons of CO2. | Training Small Language Models (SLMs) on highly curated data. |
| GPU data centers consume millions of gallons of drinking water for cooling. | Deploying Edge AI so compute happens locally, off-grid. |
| Inference costs $10M/month in electricity and cloud compute. | Quantization (INT4) reduces energy per token by 75%. |
| Discarding older GPUs every 2 years for the latest H100s. | Writing optimized architecture (e.g., FlashAttention) to run on older hardware. |

### 🧩 Mini-Quiz

> **Q1:** Does AI consume more energy during Training or Inference?
> <details><summary>Answer</summary>While <i>Training</i> requires a massive, continuous spike of energy (running 10,000 GPUs for 3 months), <b>Inference</b> (generating answers for millions of users daily over years) consumes vastly more total energy over the lifecycle of a successful model. Therefore, optimizing inference efficiency is the top priority for Green AI.</details>

---

## Phase 2: Anatomy & Internal Mechanisms

### The Carbon Footprint of AI

The environmental impact of AI is measured in two phases:
1. **Embodied Carbon:** The carbon emitted during the manufacturing of the physical hardware (mining silicon, building GPUs, constructing data centers).
2. **Operational Carbon:** The carbon emitted from generating the electricity to run and cool the servers.

### Green AI Engineering Techniques

| Technique | How it helps |
|-----------|--------------|
| **Quantization (INT4/INT8)** | Reduces the precision of math calculations. Uses far less VRAM and electricity per token generated. |
| **Distillation** | Training a small, efficient model (student) to mimic a massive, energy-hungry model (teacher). |
| **MoE (Mixture of Experts)** | Only activating 10% of a model's "brain" for any given prompt, saving 90% of the active compute energy. |
| **Edge AI** | Running models on phones/laptops instead of centralized cloud servers, distributing the energy load. |
| **Scheduled Batching** | Running heavy AI training jobs at night or during times when renewable energy (solar/wind) is abundant on the grid. |

### 🃏 Flashcard

> **Front:** What is "Knowledge Distillation" in Green AI?
> <details><summary>Flip</summary>A technique where a massive, energy-hungry model (like GPT-4) is used to generate high-quality training data, which is then used to train a much smaller, highly efficient model (like a 3B parameter model). The small model learns the specific task efficiently, saving massive amounts of inference electricity in production.</details>

---

## Phase 3: Advanced / Enterprise Patterns & Pitfalls

### Enterprise Incentives
Enterprises are not just adopting Green AI to save the planet; they are doing it because **Compute = Cash**.
- **ESG Compliance:** Public companies face strict regulatory reporting on their carbon emissions (ESG goals). Massive AI usage ruins these metrics.
- **Margin Optimization:** If an AI startup spends $0.05 per API call on cloud compute, they lose money. Switching to a distilled SLM drops the cost to $0.001, making the business viable.

### Anti-Patterns

- ❌ **Using a 70B parameter model for simple routing** → Asking GPT-4 to check if an email is "Spam" or "Not Spam" is an egregious waste of energy. Use a traditional ML classifier or an SLM.
- ❌ **Training from scratch unnecessarily** → Training an LLM from scratch is a massive carbon event. Always Fine-Tune or use RAG on existing Foundation Models instead.
- ❌ **Ignoring geographical compute location** → Running a data center in a region powered by coal vs. a region powered by hydro-electric makes a massive difference in the AI's carbon footprint.

---

## Phase 4: Practical Implementation

### Monitoring Carbon Emissions (Python via CodeCarbon)

The open-source package `codecarbon` allows developers to track the exact energy consumption and CO2 emissions of their Python scripts or AI training loops.

```python
# pip install codecarbon
from codecarbon import EmissionsTracker
import time

# Initialize the tracker
# It measures CPU/GPU energy consumption and checks local grid carbon intensity
tracker = EmissionsTracker(project_name="LLM_Inference_Test")

# Start tracking
tracker.start()

# --- Simulate an AI Task ---
print("Running AI Inference...")
# In reality, you would run model.generate() here
for i in range(10000000):
    pass
time.sleep(2)
# ---------------------------

# Stop tracking and print emissions
emissions = tracker.stop()
print(f"Energy Consumed: {tracker.final_emissions_data.energy_consumed} kWh")
print(f"CO2 Emissions: {emissions} kg")

# This data is saved to a CSV, allowing enterprises to report on their AI carbon footprint.
```

---

## Phase 5: Interview Preparation

### Q1: "How would you optimize our AI pipeline to reduce cloud compute costs and meet our corporate sustainability goals?"
<details><summary><b>STAR Answer</b></summary>

**Situation:** The company was using a massive 70B cloud-hosted LLM to summarize daily customer support transcripts, resulting in $50,000/month cloud bills and a massive operational carbon footprint.

**Task:** Reduce both cost and carbon footprint without losing summary quality.

**Action:**
1. **Model Distillation:** Used the 70B model to generate 10,000 perfect summaries, and used that data to fine-tune a small 3B parameter open-source model (SLM).
2. **Quantization:** Quantized the new SLM to 4-bit precision (INT4), drastically reducing memory and energy requirements.
3. **Batch Processing:** Moved the summarization job to run as an overnight batch process in an AWS region powered predominantly by renewable energy, rather than running it continuously all day.

**Result:** Cut compute costs by 95% (saving $47,500/month), reduced energy usage by an estimated 80%, and met the company's ESG sustainability targets.
</details>

---

## Phase 6: Summary Cheatsheet & Action Plan

### 📋 TL;DR

| Concept | Key Point |
|---------|-----------|
| **Green AI** | Focusing on efficiency, sustainability, and low compute costs. |
| **Red AI** | Chasing accuracy by wasting massive amounts of compute and energy. |
| **Top Techniques** | SLMs, Quantization, Distillation, Edge AI, Renewable Energy routing. |
| **Why enterprises care** | Less energy used = lower cloud bills and better ESG compliance. |

### 🚀 Do These Now
1. **Right-size your tasks:** Look at your current AI projects. Are you using a sledgehammer (GPT-4) to drive a nail (basic classification)? Switch the nail task to a cheaper, greener model like GPT-4o-mini or Claude Haiku.
2. **Install CodeCarbon:** Add the `codecarbon` library to one of your Python scripts to see exactly how much CO2 your compute is generating.
