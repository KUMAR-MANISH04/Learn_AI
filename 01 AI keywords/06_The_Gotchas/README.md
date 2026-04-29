# ⚠️ The "Gotchas" — The "Safety" Layer

> **The failure modes every AI engineer must understand — when AI lies convincingly and when users break through the guardrails.**

This module covers the two critical failure modes that every production AI system must defend against: hallucination (AI making things up) and jailbreaking (users bypassing safety rules).

---

## 📚 Topics Covered

| # | Topic | File | Core Idea |
|---|-------|------|-----------|
| 1 | [Hallucinations](01_Hallucinations.md) | `01_Hallucinations.md` | When AI confidently invents facts |
| 2 | [Jailbreaking](02_Jailbreaking.md) | `02_Jailbreaking.md` | Bypassing AI safety with clever prompts |

---

## 🗺️ How These Topics Connect

![How Gotcha Topics Connect](images/00_topics_connect.png)

---

## 🎯 Learning Path

1. **Start** with [Hallucinations](01_Hallucinations.md) — the most common failure mode
2. **Then** [Jailbreaking](02_Jailbreaking.md) — the adversarial attack vector

---

## 🧠 Prerequisites

- **LLMs & Prompting** — How models generate text (see [Module 5](../05_Prompting_and_Reasoning/README.md))
- **Grounding** — How we anchor AI to facts (see [Module 2](../02_Data_and_Context_The_Knowing_Layer/05_Grounding.md))
- **RLHF / DPO** — How models are aligned (see [Module 4](../04_Training_and_Tweaking/README.md))

---

> **💡 Key Insight:** These aren't bugs — they're fundamental properties of how language models work. A senior engineer's job is not to "fix" them (they can't be fully eliminated) but to **build systems that mitigate and contain them**.

---

*Each topic file follows the [Educator Skill](../.github/Educator_skill.md) 6-phase teaching methodology.*
