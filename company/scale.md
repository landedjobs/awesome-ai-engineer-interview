# Scale AI — AI Engineer Interview (2026)

🏗️🚀 *Ship the platform / build the product.* A seven-round gauntlet built around **data quality**, an **ML-eval take-home**, and a forward-deployed **messy-data presentation**.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** For an Applied AI Engineer role: recruiter → technical OOP → manager → **take-home** → debate-your-solution + technical probe → system design + ML data-modeling → hiring committee — **7 rounds**, ~**14 days median** to decision. The Forward Deployed Engineer variant runs "3–4 rounds, 6–7 interviews." The distinguishing pieces: a **build-an-ML-evaluation-pipeline take-home** and an **FDE presentation** where you're handed a messy dataset and must build a working analysis.

## The loop

1. **Recruiter call.**
2. **Technical OOP round.**
3. **Manager round.**
4. **Take-home** — build an ML evaluation pipeline.
5. **Debate solutions + technical probe.**
6. **System design + ML data-modeling.**
7. **Hiring committee.**

Source: candidate reports [51], [52], [53], [118]; timeline [116]; FDE [50].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Coding 1 | "Parsing data, data transformations, statistics about the data" | ✅ Reported — candidate report [52] |
| Coding 2 (ML) | "Reading unfamiliar code, spotting conceptual bugs, tradeoffs under time" | ✅ Reported — candidate report [52] |
| OOP design | OOP design round | ✅ Reported — candidate report [51] |
| Take-home | "Build an ML evaluation pipeline" | ✅ Reported — candidate report [118] |
| FDE presentation | "Handed a messy dataset, build a working analysis" | ✅ Reported — candidate report [53] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After the eval take-home: *"Which metric would you gate a model release on, and what's its failure mode?"*
- In the debate round: *"Argue for the solution you rejected — where would it beat your chosen one?"*
- In the FDE presentation: *"The client only trusts one number — which do you show, and how do you caveat it?"*

## What they're really testing

- **Data quality obsession.** Scale's whole business is data; the parsing/stats and messy-data rounds are the core filter.
- **Eval as a first-class artifact.** The take-home rewards a real pipeline — dataset, metric, regression gate — not a notebook.
- **Reasoning under time on unfamiliar code.** Coding 2 is about spotting conceptual bugs fast and naming trade-offs.
- **Communication with non-experts.** The FDE presentation is graded on clarity and defensibility, not model complexity.
- **Conviction you can defend.** The debate round tests whether you can argue both sides.

## How to prep

- **ML eval pipeline take-home** → [answers/eval-pipeline.md](../answers/eval-pipeline.md) is the canonical worked design; build a runnable version before the loop.
- **Data wrangling / stats** → [content/06](../content/04-evals-and-llm-as-judge.md); practice parsing messy CSVs into clean, defensible statistics.
- **Reading unfamiliar code** → [questions/](../questions/) coding tier; time yourself finding conceptual bugs.
- **FDE presentation** → rehearse a 10-minute "here's the dataset, here's what I found, here's what I'd caveat" talk.
- **System design + data modeling** → [answers/semantic-search.md](../answers/semantic-search.md).

## Culture signals

**Data quality, AI safety, and mission** [54]. Precision about data and honest error bars beat flashy modeling.

## Cross-links

- 🛠️ Worked design: [Eval pipeline (the take-home)](../answers/eval-pipeline.md)
- 🛠️ Worked design: [Semantic search / data modeling](../answers/semantic-search.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Scale AI. MIT-licensed.</sub>
