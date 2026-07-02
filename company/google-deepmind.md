# Google DeepMind — AI Engineer Interview (2026)

🧠 *"Write the model."* Research depth is the currency here — you present or critique real work, then defend it at Gemini-scale infrastructure.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** Recruiter/HM → coding screen → an **ML-fundamentals deep dive** → a **research interview** (present your past work *or* critique a DeepMind paper) → **system design for Gemini-scale infra**. Typically **3–5 rounds** across **6–8 weeks**, and the process is **decentralized** (teams drive their own bars). The distinguishing rounds are the **research presentation/paper critique** and a **newly added calibration code-review round**.

## The loop

1. **Recruiter / hiring-manager call.**
2. **Coding screen** — LeetCode medium/hard, often ML-flavored.
3. **ML-fundamentals deep dive.**
4. **Research interview** — present past work or critique a DeepMind paper.
5. **System design** — Gemini-scale infrastructure.
6. **Code-review round** (newly added) — calibration on unfamiliar code, PR-style.

Source: candidate reports [105], [103], [4]; decentralized process [4].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Coding screen | LeetCode medium/hard, ML-flavored | ✅ Reported — candidate report [105] |
| ML fundamentals | "Implement a transformer block" | ✅ Reported — candidate report [105] |
| ML fundamentals | "Calibrated evaluation of multimodal models" | ✅ Reported — candidate report [105] |
| ML fundamentals | "Design a vector DB for Gemini-scale infrastructure" | ✅ Reported — candidate report [105] |
| Research | "Critique a DeepMind paper" or "present past work" | ✅ Reported — candidate report [105] |
| Code review | Calibration review of unfamiliar code / PR-style exercise | ✅ Reported — candidate report [103] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After "implement a transformer block": *"How does the memory footprint change with grouped-query attention at Gemini scale?"*
- After the paper critique: *"What experiment would you run to falsify the paper's central claim?"*
- In the code-review round: *"Rank these three bugs by blast radius and justify the order."*

## What they're really testing

- **Genuine research literacy.** The paper-critique round rewards people who read papers adversarially and can propose the missing experiment.
- **Calibration.** Both the multimodal-eval question and the new code-review round test whether your confidence tracks reality.
- **Fundamentals you can implement, not just cite.** A transformer block from scratch is expected.
- **Scale intuition.** "Vector DB for Gemini-scale" wants sharding, recall/latency trade-offs, and cost — not a toy index.
- **Fit with a decentralized org.** Different teams weight these differently; tailor to the team you're matching with.

## How to prep

- **Transformer block from scratch** → [content/01](../content/01-llm-fundamentals.md).
- **Calibrated / multimodal evaluation** → [answers/eval-pipeline.md](../answers/eval-pipeline.md) and [content/04](../content/04-evals-and-llm-as-judge.md).
- **Vector DB at scale** → [answers/semantic-search.md](../answers/semantic-search.md) (ANN/HNSW, sharding, recall vs latency).
- **Research presentation** → prepare a 10-minute past-work talk *and* a critique of one recent DeepMind paper, with the falsifying experiment ready.
- **Code review** → practice ranking bugs by blast radius; see [questions/](../questions/).

## Culture signals

**Responsibility, safety, innovation, and benefiting humanity** [78]. Research maturity — knowing a result's limits — reads better than raw enthusiasm.

## Cross-links

- 🛠️ Worked design: [Semantic search / vector DB at scale](../answers/semantic-search.md)
- 🛠️ Worked design: [Evaluation pipeline (calibrated, multimodal)](../answers/eval-pipeline.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Google DeepMind. MIT-licensed.</sub>
