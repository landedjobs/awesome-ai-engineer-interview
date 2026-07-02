# Anthropic — AI Engineer Interview (2026)

🧠 *"Write the model."* The safety-first lab runs the **longest single loop** in the industry — and is candid enough to have open-sourced its take-home.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** Recruiter → a **55-minute CodeSignal** coding screen → technical phone screen → **take-home OR pair-coding** → a **4–5 interview onsite** over 1–2 days → references + offer. Total interviewing time is **~5–10 hours** (one candidate reported ~10h) — the most of any lab here. The distinguishing features: a **mechanistic-interpretability take-home** (which Anthropic has published) and a **dedicated 45-minute culture round** that tests whether you can reason through hard questions in real time.

## The loop

1. **Recruiter call.**
2. **CodeSignal screen (55 min)** — pure algorithmic problem-solving.
3. **Technical phone screen.**
4. **Take-home OR pair-coding** — the mech-interp assignment lands here for many candidates.
5. **Onsite (4–5 interviews, 1–2 days)** — coding, ML/research, and system design.
6. **Culture round (45 min)** — values and real-time reasoning.
7. **References + offer.**

Source: candidate reports [79], [8]; CodeSignal screen [62]; take-home [61], [63]; culture round [74].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| CodeSignal 55-min | (algorithmic problem-solving) | ✅ Reported — candidate report [62] |
| Coding | "Rotate a matrix" | ✅ Reported — candidate report [79] |
| Coding | "Implement a trie" | ✅ Reported — candidate report [79] |
| Coding | "Merge sorted lists" | ✅ Reported — candidate report [79] |
| Coding | "Build a rate limiter" | ✅ Reported — candidate report [79] |
| System design | "Design an LLM chat system incorporating retrieval and tool-use" | ✅ Reported — candidate report [79] |
| ML / research | "Mechanics of PPO, RLHF safety constraints" | ✅ Reported — candidate report [79] |
| ML / research | "Mechanistic interpretability: analyze attention heads for inducing circuits" | ✅ Reported — candidate report [79] |
| Take-home | Mechanistic-interpretability assignment | ✅ Reported — candidate reports [61], [63] |
| Culture (45 min) | "What you value, whether you can reason through hard questions in real time" | ✅ Reported — candidate report [74] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions of the rounds above.*

- After the mech-interp take-home: *"Which head did you find induces the circuit, and what ablation confirms it?"*
- After "Design an LLM chat system": *"Where do tool-use and retrieval race, and how do you make the agent's failures observable?"*
- In the culture round: *"Defend a position you hold that most people you respect disagree with."*

## What they're really testing

- **Mechanistic understanding, not just usage.** The interp take-home rewards people who can reason about *why* a model behaves, at the circuit level.
- **Clean fundamentals under a timer.** The CodeSignal screen is deliberately pure problem-solving — no LLM trivia to hide behind.
- **Systems that expose their own failures.** The chat-with-tools design question wants observability and safe termination, not just a happy path.
- **Real-time reasoning about hard, ambiguous questions.** The culture round is a genuine filter, not a formality.
- **Defensibility of hard truths.** Mission-first and safety-first values are probed directly.

## How to prep

- **Mechanistic interpretability** → [content/03](../content/06-fine-tuning-and-inference.md) and the interp section of [content/01](../content/01-llm-fundamentals.md); actually run an attention-head ablation before the take-home.
- **PPO / RLHF safety constraints** → [content/03](../content/06-fine-tuning-and-inference.md).
- **"Design an LLM chat system with retrieval + tool-use"** → [answers/agentic-workflow.md](../answers/agentic-workflow.md) and [answers/support-bot.md](../answers/support-bot.md).
- **Coding warm-ups** (trie, matrix rotate, merge, rate limiter) → [questions/](../questions/) coding tier; time yourself at CodeSignal pace.
- **RAG** → [content/02](../content/02-rag.md).
- **Culture round** → rehearse a real disagreement you reasoned through; Anthropic scores the *process*, not the answer.

## Culture signals

**Mission-first, safety-first, and the defensibility of hard truths** [77], [74]. Expect to be asked what you value and to defend it live — vague or performative answers read as a miss.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / chat with tools + retrieval](../answers/agentic-workflow.md)
- 🛠️ Worked design: [Support bot](../answers/support-bot.md)
- 📄 Note: Anthropic has publicly shared a version of its mechanistic-interpretability take-home — practicing on real interp tooling is directly on-target.

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Anthropic. MIT-licensed.</sub>
