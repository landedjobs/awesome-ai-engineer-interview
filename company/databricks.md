# Databricks — AI Engineer Interview (2026)

🏗️ *"Ship the platform."* Data-platform DNA runs through the whole loop — system design is **Spark-flavored** and the bar is "extremely selective."

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** For a Senior Applied AI Engineer role: recruiter → phone coding → **take-home** (some tracks) → **system design** → **ML/Spark** → final. **5–6 stages** over **4–7 weeks**, described by candidates as "extremely selective." The distinguishing round is a **Spark-scale ML system design** — feature stores, online inference, and clusters at 1M QPS.

## The loop

1. **Recruiter call.**
2. **Phone coding.**
3. **Take-home** (included on some tracks).
4. **System design** — platform/infra scale.
5. **ML / Spark** — ML system design with Spark.
6. **Final round.**

Source: candidate reports [37], [38], [39].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| System design | "Design a feature store for online inference" | ✅ Reported — candidate report [37] |
| System design | "How do you scale a Spark cluster to 1M QPS?" | ✅ Reported — candidate report [37] |
| ML system design | ML System Design (with Spark) | ✅ Reported — candidate report [38] |
| Take-home | Some tracks include a take-home | ✅ Reported — candidate report [38] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After "feature store for online inference": *"How do you guarantee train/serve consistency and bound staleness?"*
- After "Spark to 1M QPS": *"Where does shuffle become the bottleneck, and what do you change first?"*
- On the ML system design: *"Where does the eval set live, and how does it gate a model push?"*

## What they're really testing

- **Platform-scale system design.** Feature stores, online inference, and 1M-QPS clusters are the core filter — think data infra, not just models.
- **Spark fluency.** The ML rounds assume you understand shuffles, partitioning, and cluster scaling.
- **Train/serve consistency.** Feature-store questions probe whether you've actually run online inference in production.
- **Take-home execution.** Where present, it rewards clean, runnable, defensible work.
- **Selectivity endurance.** 5–6 stages over 4–7 weeks favor consistency across a long loop.

## How to prep

- **Feature store / online inference** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) and [content/06](../content/06-data-and-eval-sets.md) for train/serve consistency.
- **Spark scaling** → know shuffle, partitioning, broadcast joins, and where clusters bottleneck.
- **ML system design** → [answers/eval-pipeline.md](../answers/eval-pipeline.md) and [answers/semantic-search.md](../answers/semantic-search.md).
- **Coding warm-ups** → [questions/](../questions/) coding tier.

## Culture signals

Data-platform first; **"extremely selective."** Show that you reason about data infrastructure and scale, not just model quality.

## Cross-links

- 🛠️ Worked design: [LLM inference at scale / online inference](../answers/llm-inference-at-scale.md)
- 🛠️ Worked design: [Eval pipeline](../answers/eval-pipeline.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Databricks. MIT-licensed.</sub>
