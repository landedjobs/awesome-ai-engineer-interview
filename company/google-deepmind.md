# Google DeepMind — AI Engineer Interview (2026)

🧠 *"Write the model."* Research depth is the currency here. Expect to explain your own work, think out loud and back your impact with data.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at Google DeepMind](https://www.landed.jobs/companies/google-deepmind?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full Google DeepMind loop guide with a prep plan](https://www.landed.jobs/resources/guides/google-deepmind-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

> [!NOTE]
> **The loop at a glance.** *Official ([Interviewing at Google DeepMind](https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf), checked 17 Nov 2026):* four stages, **initial interviews → skills interviews → final interviews → decision and offer**, with detailed, role-specific prep material sent to each candidate. The process takes **4–10 weeks** depending on the role, and can run longer with interviewer availability. Many interviews are competency-based, and the guide suggests the STAR method. **AI policy:** you may use AI to prepare, but not during live interviews or interview tasks unless told otherwise. The guide does not publish the technical rounds for an AI Engineer role, so the exact rounds are **not publicly confirmed**; ask your recruiter.
>
> Last checked: 17 Nov 2026.

## The loop

1. **Initial interviews** (official).
2. **Skills interviews** (official): role-specific; the guide links a coding-prep video for roles that involve coding.
3. **Final interviews** (official).
4. **Decision and offer** (official): feedback usually within a few days; check with your recruiter after a week.

Source: [Interviewing at Google DeepMind](https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf) (official guide, checked 17 Nov 2026).

What happens inside the skills and final stages for an AI Engineer is **not publicly confirmed**. The earlier version of this page listed a coding screen, an ML-fundamentals deep dive, a research presentation or paper critique, Gemini-scale system design and a newly added code-review round. We could not find a public source for that sequence, so treat those as rounds to ask about, not a confirmed loop.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Coding screen | LeetCode medium/hard, ML-flavored | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML fundamentals | "Implement a transformer block" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML fundamentals | "Calibrated evaluation of multimodal models" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML fundamentals | "Design a vector DB for Gemini-scale infrastructure" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Research | "Critique a DeepMind paper" or "present past work" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Code review | Calibration review of unfamiliar code / PR-style exercise | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After "implement a transformer block": *"How does the memory footprint change with grouped-query attention at Gemini scale?"*
- After the paper critique: *"What experiment would you run to falsify the paper's central claim?"*
- In the code-review round: *"Rank these three bugs by blast radius and justify the order."*

## What they're really testing

- **Genuine research literacy.** The paper-critique round rewards people who read papers adversarially and can propose the missing experiment.
- **Intellectual honesty.** The official guide says they value curiosity and honesty: say when you're not sure, ask questions and explain your reasoning.
- **Calibration.** Questions like multimodal evaluation or a code review test whether your confidence tracks reality.
- **Fundamentals you can implement, not just cite.** A transformer block from scratch is expected.
- **Scale intuition.** "Vector DB for Gemini-scale" wants sharding, recall/latency trade-offs, and cost — not a toy index.
- **Role-specific bars.** The official guide says each candidate gets role-specific prep material; tailor to the team you're matching with.
- **Unaided work.** Live interviews and interview tasks are AI-free unless you're told otherwise.

## How to prep

- **Transformer block from scratch** → [content/01](../content/01-llm-fundamentals.md).
- **Calibrated / multimodal evaluation** → [answers/eval-pipeline.md](../answers/eval-pipeline.md) and [content/04](../content/04-evals-and-llm-as-judge.md).
- **Vector DB at scale** → [answers/semantic-search.md](../answers/semantic-search.md) (ANN/HNSW, sharding, recall vs latency).
- **Research presentation** → prepare a 10-minute past-work talk *and* a critique of one recent DeepMind paper, with the falsifying experiment ready.
- **Code review** → practice ranking bugs by blast radius; see [questions/](../questions/).
- **Official prep steps** → the guide suggests reading the DeepMind blog, trying Gemini in Google AI Studio, reviewing the job description for competencies, and using data to show your impact.
- **AI policy** → prepare with AI if you like, but practise solving without it; if a round does allow AI, see [content/10](../content/10-ai-assisted-coding-round.md).

## Culture signals

**Responsibility, safety, innovation, and benefiting humanity.** DeepMind states its mission as building AI responsibly to benefit humanity ([About Google DeepMind](https://deepmind.google/about/), official). The interviewing guide adds curiosity and intellectual honesty ([official guide](https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf)). Research maturity, knowing a result's limits, reads better than raw enthusiasm.

## Cross-links

- 🛠️ Worked design: [Semantic search / vector DB at scale](../answers/semantic-search.md)
- 🛠️ Worked design: [Evaluation pipeline (calibrated, multimodal)](../answers/eval-pipeline.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Google DeepMind. MIT-licensed.</sub>
