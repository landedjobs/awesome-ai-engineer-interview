# Meta (AI / GenAI) — AI Engineer Interview (2026)

🧠 *"Write the model."* A FAANG-style coding loop that, since late 2025, swaps one coding round for an **AI-enabled coding** round, so you're graded on how well you *use* the assistant.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** *Second-hand ([Hello Interview](https://www.hellointerview.com/blog/meta-ai-enabled-coding), undated; [interviewing.io](https://interviewing.io/meta-interview-questions), 2026):* Meta began rolling out an **AI-enabled coding** round in **October 2025**. In the onsite it replaces one of the two classic coding rounds; the other stays **unaided**, and the recruiter tells you which is which. The round is **60 minutes** in a CoderPad setup with a multi-file codebase, tests and an AI chat whose model you can switch. Hello Interview reports it for SWE and EM onsites up to E7/M2; interviewing.io describes it as a pilot that not every candidate gets. *One first-person MLE account ([Aced, Senior MLE (L5), US](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7)):* a 45-minute phone screen with two coding problems, then four finals: coding, **AI-enabled debugging/coding**, behavioral and ML design. Total elapsed time and an MLE-wide round list are **not publicly confirmed**. More on the format: [content/10](../content/10-ai-assisted-coding-round.md).
>
> Last checked: 17 Nov 2026.

## The loop

*SWE loop, second-hand ([interviewing.io](https://interviewing.io/meta-interview-questions), 2026):*

1. **Recruiter call.**
2. **Phone screen (~45 min)**: two LeetCode easy/medium questions in CoderPad with code execution off.
3. **Onsite**: at least two coding rounds, design rounds and one behavioral. Where the AI-enabled round applies, it replaces one coding round ([Hello Interview](https://www.hellointerview.com/blog/meta-ai-enabled-coding)).
4. **Team matching, then offer.**

*Senior MLE (L5), one candidate's account ([Aced](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7), first-person; page shows relative dates only; rejected):*

1. **Sourcer contact and a written pre-screen questionnaire** (role, projects, tools, AI usage).
2. **Phone screen (45 min)**: two coding problems.
3. **Final 1, coding**: two problems.
4. **Final 2, AI-enabled debugging/coding**: run the tests, debug a small multi-file codebase, implement the missing core function, with an LLM available.
5. **Final 3, behavioral.**
6. **Final 4, ML design.**

Research Scientist add-ons (deep research interviews) were listed on the earlier version of this page; we could not find a public source, so they are **not publicly confirmed**.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Phone screen | Simplify a path string; merge two sorted interval lists | ✅ Reported · [first-person, Aced, Senior MLE](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7) |
| Coding | Lowest common ancestor in a tree with parent pointers; next element greater than k in a sorted array | ✅ Reported · [first-person, Aced, Senior MLE](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7) |
| AI-enabled coding | Debug a small multi-file codebase and implement the missing core function, using an LLM if helpful | ✅ Reported · [first-person, Aced, Senior MLE](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7) |
| AI-enabled coding | Phases of bug fixing, core implementation (AI use expected) and optimization against larger inputs | ✅ Reported · [second-hand, Hello Interview, rollout Oct 2025](https://www.hellointerview.com/blog/meta-ai-enabled-coding) |
| ML design | Design TikTok's video recommendation system, with offline vs online evaluation follow-ups | ✅ Reported · [first-person, Aced, Senior MLE](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7) |
| Behavioral | "A time you disagreed with someone and were wrong"; a time you disagreed with your manager | ✅ Reported · [first-person, Aced, Senior MLE](https://www.aced.io/experiences/meta-facebook-senior-machine-learning-engineer-interview-8719a7) |
| System design | "Design a recommender system for Instagram Reels" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| System design | "Design Instagram feed" or "Design Messenger" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML theory | "Difference between L1 and L2 regularization" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML theory | "Bias/variance, precision-recall" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Research | Deep-dive presentation on past work | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- In AI-enabled coding: *"The assistant produced this diff. Which part do you trust, which do you verify, and how?"*
- After "Design Reels recommender": *"Where does the ranking model's latency budget go, and what do you cache?"*
- After the L1/L2 question: *"When would you pick neither and reach for early stopping instead?"*

## What they're really testing

- **Raw coding throughput.** Two problems in a 45-minute screen, often with code execution off, rewards speed and correctness.
- **Judgment *with* an AI assistant.** Hello Interview reports the same four competencies as classic coding (problem solving, code quality, verification, communication), and that the in-interview AI was sometimes less helpful than in practice. One MLE candidate reports the LLM invented helper functions that were not in the codebase. You must catch that.
- **Classic ML fluency.** Recommendation design with offline vs online evaluation came up in one MLE final; L1/L2 and bias/variance are fair game.
- **Product-scale system design.** Reels/feed/Messenger want ranking, caching, and latency at billions of users.
- **Move-fast collaboration.** The behavioral round probes disagreement and openness.

## How to prep

- **Coding volume** → [questions/](../questions/) coding tier; drill LeetCode-medium at phone-screen pace.
- **AI-enabled coding** → read [content/10](../content/10-ai-assisted-coding-round.md); practise on an unfamiliar multi-file codebase with tests, and reject a wrong suggestion out loud. See also [questions/coding.md](../questions/coding.md) and [answers/agentic-workflow.md](../answers/agentic-workflow.md). Hello Interview says Meta offers a practice environment before the round.
- **Recommender / feed system design** → [answers/semantic-search.md](../answers/semantic-search.md) for retrieval-and-ranking patterns; [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) for latency/caching.
- **ML theory** → [content/00](../content/01-llm-fundamentals.md) (L1/L2, bias/variance, precision-recall).
- **Research deep-dive** → prepare a crisp past-work presentation.

## Culture signals

Meta's current values are **Move fast**, **Build awesome things**, **Be direct and respect your colleagues**, **Focus on long-term impact**, **Live in the future** and **Meta, Metamates, me** ([Meta Careers culture page](https://www.metacareers.com/culture), official). The behavioral questions in one MLE account centered on disagreement and conflict, so bring concrete stories of shipping fast and disagreeing openly.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / AI-assisted coding](../answers/agentic-workflow.md)
- 🛠️ Worked design: [Semantic search & ranking](../answers/semantic-search.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Meta. MIT-licensed.</sub>
