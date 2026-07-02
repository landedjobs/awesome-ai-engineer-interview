# xAI (Grok) — AI Engineer Interview (2026)

🏗️ *"Ship the platform."* A fast, high-throughput loop: a rapid-fire recruiter chat, a dense **4-question / 70-minute CodeSignal**, and three onsite cores centered on **serving Grok at scale**.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** Phone/recruiter → a **45–60 min DSA** screen → **3 onsite cores** (Coding, System Design, ML/AI). Roughly **4–5 interviews** over **1–3 weeks**. One candidate (Nov 2025) reported a 30-min recruiter call ("why xAI") and a **CodeSignal with 4 questions in 70 minutes**. The distinguishing round is a **serving/inference system-design** question for Grok, including distributed matrix multiplication.

## The loop

1. **Recruiter screen (~15–30 min)** — resume + "why xAI," rapid-fire.
2. **CodeSignal / DSA (70 min)** — 4 questions (two medium-hard, one graph).
3. **Onsite core: Coding** — live coding (~60 min).
4. **Onsite core: System Design** — inference/serving for Grok; distributed matmul.
5. **Onsite core: ML / interpretability** — SHAP/LIME, interpretability trade-offs.

Source: candidate reports [31], [32], [33], [34], [30], [96], [97].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Recruiter screen | Resume + "why xAI" (15 min, rapid-fire) | ✅ Reported — candidate report [30] |
| CodeSignal 70-min | "4 questions in 70 min (two medium-hard, one graph...)" | ✅ Reported — candidate report [33] |
| System design | Inference/serving for Grok; distributed matrix multiplication | ✅ Reported — candidate reports [32], [34] |
| ML / interpretability | SHAP/LIME and interpretability trade-offs | ✅ Reported — candidate report [32] |
| Live coding | "Ace the live coding" (candidate tip) | ✅ Reported — candidate report [96] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After "serving for Grok": *"Shard a 200B-parameter matmul across 8 GPUs — where's the communication bottleneck?"*
- After SHAP/LIME: *"When is a post-hoc explanation actively misleading, and what would you use instead?"*
- CodeSignal graph problem: expect a shortest-path or topological-sort variant under time pressure.

## What they're really testing

- **Speed.** The 70-minute / 4-question CodeSignal and the fast overall loop reward candidates who ship correct code quickly.
- **Serving intuition.** Distributed matmul and inference for Grok are the core signal — you should know where FLOPs and communication go.
- **Honest interpretability.** SHAP/LIME questions want you to name their limits, not oversell them.
- **Mission energy.** "Why xAI" is asked early and directly; conviction matters.

## How to prep

- **Distributed serving / matmul** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) (tensor/pipeline parallelism, batching, KV-cache).
- **CodeSignal speed** → [questions/](../questions/) coding tier; drill graph + two medium-hard problems under a 70-min cap.
- **Interpretability** → [content/03](../content/06-fine-tuning-and-inference.md) for SHAP/LIME trade-offs.
- **"Why xAI"** → prepare a 60-second mission answer you actually believe.

## Culture signals

**Speed-of-light progress, a post-AGI mission, and AI safety.** The loop is built to move fast; match that energy.

## Cross-links

- 🛠️ Worked design: [LLM inference at scale / serving Grok](../answers/llm-inference-at-scale.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with xAI. MIT-licensed.</sub>
