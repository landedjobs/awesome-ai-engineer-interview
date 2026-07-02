# Meta (AI / GenAI) — AI Engineer Interview (2026)

🧠 *"Write the model."* A FAANG-style coding gauntlet — but 2026 added the industry's first mainstream **"Coding with AI"** round, so you're graded on how well you *use* the assistant.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** MLE track: recruiter (45 min) → phone (2 LeetCode-medium, ~45 min) → onsite of **5 coding + 1 system design + 1 behavioral**. Research Scientist adds **2 deep research interviews** and the **new "Coding with AI" round**. End-to-end **4–8 weeks**, committee-gated. The distinguishing round is **"Coding with AI"** — the mirror image of Cursor's no-AI policy.

## The loop

1. **Recruiter call (45 min).**
2. **Phone screen** — two LeetCode-medium questions (~45 min).
3. **Onsite** — 5 coding + 1 system design + 1 behavioral.
4. **Research Scientist add-ons** — 2 deep research interviews + a **"Coding with AI"** round.

Source: candidate reports [55], [56], [57].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Coding (each) | "Two LeetCode-style questions (medium difficulty)" | ✅ Reported — candidate report [57] |
| System design | "Design a recommender system for Instagram Reels" | ✅ Reported — candidate report [59] |
| System design | "Design Instagram feed" or "Design Messenger" | ✅ Reported — candidate report [59] |
| ML theory | "Difference between L1 and L2 regularization" | ✅ Reported — candidate report [59] |
| ML theory | "Bias/variance, precision-recall" | ✅ Reported — candidate report [59] |
| Research | Deep-dive presentation on past work | ✅ Reported — candidate report [58] |
| "Coding with AI" | Code review / style fix using AI assistant tools | ✅ Reported — candidate report [55] |
| Behavioral | "Tell me about a time you disagreed with a coworker" | ✅ Reported — candidate report [59] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- In "Coding with AI": *"The assistant produced this diff — which part do you trust, which do you verify, and how?"*
- After "Design Reels recommender": *"Where does the ranking model's latency budget go, and what do you cache?"*
- After the L1/L2 question: *"When would you pick neither and reach for early stopping instead?"*

## What they're really testing

- **Raw coding throughput.** Five coding rounds means speed and correctness under repetition, not one clever insight.
- **Judgment *with* an AI assistant.** The new round grades whether you can review, verify, and correct AI output — the skill Meta bets its engineers now need.
- **Classic ML fluency.** L1/L2, bias/variance, precision-recall are still live at Meta.
- **Product-scale system design.** Reels/feed/Messenger want ranking, caching, and latency at billions of users.
- **Move-fast collaboration.** The behavioral round probes disagreement and openness.

## How to prep

- **Coding volume** → [questions/](../questions/) coding tier; drill LeetCode-medium at phone-screen pace.
- **"Coding with AI"** → practice reviewing AI-generated diffs: what to trust, what to test. See [content/05](../questions/coding.md) and [answers/agentic-workflow.md](../answers/agentic-workflow.md).
- **Recommender / feed system design** → [answers/semantic-search.md](../answers/semantic-search.md) for retrieval-and-ranking patterns; [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) for latency/caching.
- **ML theory** → [content/00](../content/01-llm-fundamentals.md) (L1/L2, bias/variance, precision-recall).
- **Research deep-dive** → prepare a crisp past-work presentation.

## Culture signals

**Move Fast, Be Bold, Be Open, Build Social Value** [58]. The behavioral round wants concrete stories of shipping fast and disagreeing openly.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / AI-assisted coding](../answers/agentic-workflow.md)
- 🛠️ Worked design: [Semantic search & ranking](../answers/semantic-search.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Meta. MIT-licensed.</sub>
