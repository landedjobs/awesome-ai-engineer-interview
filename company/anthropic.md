# Anthropic — AI Engineer Interview (2026)

🧠 *"Write the model."* The safety-first lab runs a long loop with a values round that many candidates find hardest, and it has published an old take-home for anyone to try.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at Anthropic](https://www.landed.jobs/companies/anthropic?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full Anthropic loop guide with a prep plan](https://www.landed.jobs/resources/guides/anthropic-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

> [!NOTE]
> **The loop at a glance.** *Second-hand, 2026 ([interviewing.io](https://interviewing.io/anthropic-interview-questions), based on conversations with Anthropic engineers):* a **30-minute** recruiter call → a **60–90 minute** coding challenge (most reported a 90-minute CodeSignal take-home; referrals may skip it) → a **4–5 hour onsite**, typically five one-hour sessions: hiring manager, CodeSignal coding, system design, role-specific coding and a **company values** round. About **3–4 weeks** in all, by that page's estimate, not Anthropic's. *Official AI policy ([Anthropic candidate AI guidance](https://www.anthropic.com/candidate-ai-guidance), checked 17 Nov 2026):* no AI in live interviews or take-homes **unless Anthropic says otherwise**; AI is encouraged for preparation. The same second-hand page reports an AI-assisted interview being piloted; that does not change the official default.
>
> Last checked: 17 Nov 2026.

## The loop

1. **Recruiter call (30 min).** The second-hand source says candidates can fail it.
2. **Coding challenge (60–90 min).** Most reported a 90-minute CodeSignal take-home; a 60-minute live version is also mentioned. One engineer described a two-hour, four-level assessment.
3. **Onsite (4–5 hours, typically five one-hour sessions):**
    - Hiring manager call.
    - Coding in CodeSignal.
    - System design in a shared Google Doc.
    - Role-specific second coding round.
    - **Company values** round with non-technical interviewers.
4. **References + offer.**

Source: [interviewing.io](https://interviewing.io/anthropic-interview-questions) (second-hand, 2026). Exact composition varies by role.

*Published take-home (official):* Anthropic released a version of its original **performance-engineering take-home** on [GitHub](https://github.com/anthropics/original_performance_takehome). The README says it was once a 4-hour test, then a 2-hour one, and that Anthropic moved to a different base for time-limited take-homes after Claude Opus 4.5. It is a performance-optimization task, not a mechanistic-interpretability one, and the repo does not say which roles received it.

*One first-person account ([Aced, entry-level MLE (L3), US](https://www.aced.io/experiences/anthropic-machine-learning-engineer-interview-43e051); page shows relative dates only):* recruiter screen → online assessment (extract a clean dataset with SQL and Python) → technical interview (clean and analyse it in Python without NLP libraries) → a panel scenario → a final round with data cleaning and a debugging task. About 3 weeks; rejected at the final technical stage. This is an entry-level process, not the AI Engineer loop.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Coding take-home | Implement a bank with multiple transaction types (widely circulated, so interviewers may watch for rehearsed answers) | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/anthropic-interview-questions) |
| System design | "Design a Claude chat service" | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/anthropic-interview-questions) |
| System design | "Design a system that enables a GPT to handle multiple questions in a single thread" | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/anthropic-interview-questions) |
| System design | "Design a banking app" | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/anthropic-interview-questions) |
| Values (1h) | Reflective questions about past experiences and hypothetical ethical scenarios, with follow-ups on how you felt then and now | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/anthropic-interview-questions) |
| Panel (entry-level MLE) | A conversational model deployed on sensitive topics gives confident wrong answers in high-risk contexts: how do you investigate and mitigate? | ✅ Reported · [first-person, Aced](https://www.aced.io/experiences/anthropic-machine-learning-engineer-interview-43e051) |
| Final (entry-level MLE) | Find and fix bugs in a ~200-line prewritten model-training data pipeline | ✅ Reported · [first-person, Aced](https://www.aced.io/experiences/anthropic-machine-learning-engineer-interview-43e051) |
| Take-home (published) | Speed up starter code on a simulated machine, scored in clock cycles | ✅ Official · [Anthropic on GitHub](https://github.com/anthropics/original_performance_takehome) |
| Coding | "Rotate a matrix" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Coding | "Implement a trie" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Coding | "Merge sorted lists" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Coding | "Build a rate limiter" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| System design | "Design an LLM chat system incorporating retrieval and tool-use" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML / research | "Mechanics of PPO, RLHF safety constraints" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML / research | "Mechanistic interpretability: analyze attention heads for inducing circuits" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions of the rounds above.*

- After the performance take-home: *"Which optimization bought the most cycles, and how did you check the tests were untouched?"*
- After "Design an LLM chat system": *"Where do tool-use and retrieval race, and how do you make the agent's failures observable?"*
- In the culture round: *"Defend a position you hold that most people you respect disagree with."*

## What they're really testing

- **Measured optimization.** The published take-home is scored in clock cycles, so it rewards profiling and careful iteration over clever guesses.
- **Clean, practical code under a timer.** The reported CodeSignal challenge is a progressive build (such as a bank with several transaction types), with no AI unless Anthropic says otherwise.
- **Systems that expose their own failures.** The chat-with-tools design question wants observability and safe termination, not just a happy path.
- **Real-time reasoning about hard, ambiguous questions.** The second-hand source says most candidates fail the values round, and that interviewers value skepticism and thoughtful pushback over agreement.
- **Defensibility of hard truths.** Mission-first and safety-first values are probed directly.

## How to prep

- **Performance take-home** → try the [published version](https://github.com/anthropics/original_performance_takehome) under a 2-hour limit, without AI, to match the default policy.
- **AI policy** → read the [official candidate AI guidance](https://www.anthropic.com/candidate-ai-guidance); if a round allows AI, see [content/10](../content/10-ai-assisted-coding-round.md).
- **Mechanistic interpretability** (for interp-leaning roles) → [content/03](../content/06-fine-tuning-and-inference.md) and the interp section of [content/01](../content/01-llm-fundamentals.md).
- **PPO / RLHF safety constraints** → [content/03](../content/06-fine-tuning-and-inference.md).
- **"Design an LLM chat system with retrieval + tool-use"** → [answers/agentic-workflow.md](../answers/agentic-workflow.md) and [answers/support-bot.md](../answers/support-bot.md).
- **Coding warm-ups** (trie, matrix rotate, merge, rate limiter) → [questions/](../questions/) coding tier; time yourself at CodeSignal pace.
- **RAG** → [content/02](../content/02-rag.md).
- **Values round** → rehearse a real disagreement you reasoned through. The second-hand source suggests reading Dario Amodei's *Machines of Loving Grace* and Anthropic's *Core Views on AI Safety*.

## Culture signals

Anthropic's published values include **"Put the mission first"**, **"Ignite a race to the top on safety"** and **"Be helpful, honest, and harmless"** ([Anthropic company page](https://www.anthropic.com/company), official). The values round asks what you value and how you reason through ethical scenarios ([interviewing.io](https://interviewing.io/anthropic-interview-questions), second-hand, 2026). Vague or performative answers read as a miss.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / chat with tools + retrieval](../answers/agentic-workflow.md)
- 🛠️ Worked design: [Support bot](../answers/support-bot.md)
- 📄 Note: Anthropic has published a version of its [original performance take-home](https://github.com/anthropics/original_performance_takehome). It is a performance-engineering task, not an interp one.
- 🤖 AI-assisted rounds: [content/10](../content/10-ai-assisted-coding-round.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Anthropic. MIT-licensed.</sub>
