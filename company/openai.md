# OpenAI — AI Engineer Interview (2026)

🧠 *"Write the model."* The lab that made LLMs a product still interviews like a research org — implement-from-scratch primitives, then defend how you'd ship them at ChatGPT scale.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at OpenAI](https://www.landed.jobs/companies/openai?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full OpenAI loop guide with a prep plan](https://www.landed.jobs/resources/guides/openai-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

> [!NOTE]
> **The loop at a glance.** *Official ([OpenAI interview guide](https://openai.com/interview-guide), checked 17 Nov 2026):* application review (about **one week**) → introductory calls with a recruiter or hiring manager → a **skills-based assessment** whose format varies by team (pair coding, take-home, technical test) → **4–6 hours of final interviews with 4–6 people over 1–2 days** → a decision within about **one week**, when references may be requested. A single end-to-end duration is not published; one second-hand guide puts it at **6–8 weeks** ([interviewing.io, 2026](https://interviewing.io/openai-interview-questions)). AI-tool rules **vary by interview**: some formats allow AI, others test you without it, and the prep materials say which. A second-hand guide reports an **agentic coding round in beta** for some candidates ([interviewing.io, 2026](https://interviewing.io/openai-interview-questions)); see [content/10](../content/10-ai-assisted-coding-round.md).
>
> Last checked: 17 Nov 2026.

## The loop

1. **Application and résumé review** (official): typically about one week.
2. **Introductory calls** (official): a recruiter or the hiring manager; expect to discuss your work, motivations and goals.
3. **Skills-based assessment** (official): pair coding, a take-home project or a technical test, depending on the team. Some roles get more than one.
4. **Final interviews** (official): 4–6 hours with 4–6 people over 1–2 days, focused on your area. For engineering, the guide says they look for well-designed solutions, high-quality code, optimal performance and good test coverage.
5. **Decision and references** (official): expect to hear within about a week; the recruiter may ask for references.

Source: [OpenAI interview guide](https://openai.com/interview-guide) (official, checked 17 Nov 2026).

*Second-hand, 2026 ([interviewing.io](https://interviewing.io/openai-interview-questions)):* a 30-minute recruiter call, a ~1-hour technical phone screen, sometimes a 1-hour system-design screen, then a 4–6 hour onsite that can include a 45-minute project presentation, coding, system design, behavioral rounds and, for some candidates, a **beta agentic coding round** on an existing codebase. The same page says AI use is otherwise prohibited. That conflicts in part with the official guide, which says rules vary by interview, so ask your recruiter.

*One first-person account ([Aced, Research Engineer, US](https://www.aced.io/experiences/openai-machine-learning-engineer-interview-c7b6a2); page shows relative dates only, posted about spring 2026):* referral → recruiter screen → two 60-minute technical interviews → three final rounds. About 7 weeks in all; the candidate was rejected before the hiring-manager chat. This is one Research Engineer process, not the AI Engineer loop.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Final (Research Engineer) | Find the bug in each of four marked regions of a transformer implementation, then implement KV caching | ✅ Reported · [first-person, Aced, ~2026](https://www.aced.io/experiences/openai-machine-learning-engineer-interview-c7b6a2) |
| Final (Research Engineer) | Implement all_gather across multiple nodes; bound the rounds needed over noisy channels | ✅ Reported · [first-person, Aced, ~2026](https://www.aced.io/experiences/openai-machine-learning-engineer-interview-c7b6a2) |
| Final (Research Engineer) | Serialize and deserialize a key-value store; restore state after shutdown | ✅ Reported · [first-person, Aced, ~2026](https://www.aced.io/experiences/openai-machine-learning-engineer-interview-c7b6a2) |
| Technical (Research Engineer) | Given a Python package, find the latest Python version that supports it | ✅ Reported · [first-person, Aced, ~2026](https://www.aced.io/experiences/openai-machine-learning-engineer-interview-c7b6a2) |
| Technical (Senior Research Engineer) | "Write code for a system that assigns tasks to humans and AI" under a list of requirements | ✅ Reported · [first-person, Aced, ~2026](https://www.aced.io/experiences/openai-senior-machine-learning-engineer-interview-9b7da4) |
| Coding (second-hand) | KL divergence for continuous distributions, with the formula given | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/openai-interview-questions) |
| Phone coding | "Implement an LRU cache" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Coding onsite | "Design a word-frequency counter in a stream" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Coding onsite | "Implement a tokenizer" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML coding (research) | "Implement attention [block]" / "implement PPO" / "implement VAE" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML research | "Debug a sudden 5pp regression in your eval?" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML research | "Design an RLHF pipeline for a 70B model" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| System design | "Design ChatGPT" / "Design an evaluation pipeline" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| System design | "Design the serving stack for ChatGPT" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| System design | "How would you make ChatGPT safe from jailbreaks?" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML domain | "How do you evaluate a generative model after fine-tuning?" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| ML applications | "How to evaluate ChatGPT and mitigate jailbreak/harm behavior" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Hiring manager | "What would you build at OpenAI in the first 90 days?" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |

### 🔮 Representative follow-ups you should expect

*Not reported — these are the natural next probes given the rounds above.*

- After "implement attention": *"Now add KV-cache and explain the memory math for a 70B decode step."*
- After "Design the serving stack": *"Where does continuous batching help and where does it hurt tail latency?"*
- After the jailbreak question: *"How would you build the eval set that proves your mitigation works before shipping?"*

## What they're really testing

- **From-scratch ML you can debug.** One Research Engineer final asked for transformer bug-finding plus a KV cache, not a recital. Expect to read and fix model code, not only write it.
- **Can you reason about *your own* system under failure?** The "5pp eval regression" question rewards a debugging method, not a lucky guess.
- **Product judgment at frontier scale.** Design questions about a ChatGPT-like product test whether you'd build the *right* thing, not just a correct thing.
- **Safety as an engineering constraint.** Jailbreak/harm questions want eval-backed mitigations, not hand-waving.
- **Alignment with the Charter.** The official guide names collaboration, communication, openness to feedback and alignment with the mission and values.
- **Your process, with or without AI.** The official guide says some interviews allow AI and some do not. Practise both modes; see [content/10](../content/10-ai-assisted-coding-round.md).

## How to prep

- **Attention / transformer from memory** → work through [content/01](../content/01-llm-fundamentals.md); rehearse writing multi-head attention with a KV-cache cold, and practise finding bugs in someone else's transformer code.
- **Agentic coding round** → [content/10](../content/10-ai-assisted-coding-round.md); confirm with your recruiter whether your loop includes it.
- **Official reading list** → the guide recommends the OpenAI Charter, OpenAI research posts, the *Deep Learning* book and *Spinning Up in Deep RL*.
- **RLHF / PPO / VAE** → [content/03](../content/06-fine-tuning-and-inference.md) for the RLHF pipeline mental model at 70B scale.
- **"Design ChatGPT" / serving stack** → the worked design in [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) (batching, KV-cache, quantization, autoscaling) and [answers/support-bot.md](../answers/support-bot.md) for the product-shaped variant.
- **Evaluation pipeline** → [answers/eval-pipeline.md](../answers/eval-pipeline.md); be ready to design the eval *before* the system.
- **RAG fundamentals** → [content/02](../content/02-rag.md).
- **Question bank** → [questions/](../questions/) for the LLM-fundamentals and system-design tiers.

## Culture signals

Alignment with OpenAI's mission and values, plus **collaboration**, **effective communication** and **openness to feedback** ([OpenAI interview guide](https://openai.com/interview-guide), official). The guide also says OpenAI is not credential-driven and values people who ramp up fast in a new domain. Come with a crisp story of something you shipped and can defend end-to-end.

## Cross-links

- 🛠️ Worked design: [Design ChatGPT / LLM inference at scale](../answers/llm-inference-at-scale.md)
- 🛠️ Worked design: [Support bot / "Design ChatGPT" product framing](../answers/support-bot.md)
- 🛠️ Worked design: [Evaluation pipeline](../answers/eval-pipeline.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with OpenAI. MIT-licensed.</sub>
