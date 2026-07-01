# OpenAI — AI Engineer Interview (2026)

🧠 *"Write the model."* The lab that made LLMs a product still interviews like a research org — implement-from-scratch primitives, then defend how you'd ship them at ChatGPT scale.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** A resume screen → recruiter → phone coding → a **4–6 hour final loop** (coding, system design, ML, behavioral, and a hiring-manager deep dive) → references → offer. End-to-end **2–8 weeks** (data-science roles 4–6 weeks). The distinguishing round is the **hiring-manager "first 90 days" deep dive** paired with a **"Design ChatGPT"**-class system-design question — OpenAI wants to see both from-scratch ML fluency and product judgment about the thing you'd actually build.

## The loop

1. **Resume screen** — bar is high; impact evidence matters more than pedigree.
2. **Recruiter call (~30 min)** — motivation, comp, role-fit, timeline.
3. **Phone coding (45 min)** — standard DSA / applied coding.
4. **Final loop (4–6h over 1–2 days)** — coding, system design, ML/research coding, behavioral, and the hiring-manager deep dive.
5. **References.**
6. **Offer & compensation.**

Source: candidate reports [10], [95]; timeline [14].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Phone coding | "Implement an LRU cache" | ✅ Reported — candidate report [95] |
| Coding onsite | "Design a word-frequency counter in a stream" | ✅ Reported — candidate report [93] |
| Coding onsite | "Implement a tokenizer" | ✅ Reported — candidate report [93] |
| ML coding (research) | "Implement attention [block]" / "implement PPO" / "implement VAE" | ✅ Reported — candidate report [4] |
| ML research | "Debug a sudden 5pp regression in your eval?" | ✅ Reported — candidate report [4] |
| ML research | "Design an RLHF pipeline for a 70B model" | ✅ Reported — candidate report [4] |
| System design | "Design ChatGPT" / "Design an evaluation pipeline" | ✅ Reported — candidate report [95] |
| System design | "Design the serving stack for ChatGPT" | ✅ Reported — candidate report [93] |
| System design | "How would you make ChatGPT safe from jailbreaks?" | ✅ Reported — candidate report [93] |
| ML domain | "How do you evaluate a generative model after fine-tuning?" | ✅ Reported — candidate report [93] |
| ML applications | "How to evaluate ChatGPT and mitigate jailbreak/harm behavior" | ✅ Reported — candidate report [95] |
| Hiring manager | "What would you build at OpenAI in the first 90 days?" | ✅ Reported — candidate report [93] |

### 🔮 Representative follow-ups you should expect

*Not reported — these are the natural next probes given the rounds above.*

- After "implement attention": *"Now add KV-cache and explain the memory math for a 70B decode step."*
- After "Design the serving stack": *"Where does continuous batching help and where does it hurt tail latency?"*
- After the jailbreak question: *"How would you build the eval set that proves your mitigation works before shipping?"*

## What they're really testing

- **From-scratch ML is table stakes, not the differentiator.** Attention / PPO / VAE from memory only qualifies you; the offer is decided elsewhere.
- **Can you reason about *your own* system under failure?** The "5pp eval regression" question rewards a debugging method, not a lucky guess.
- **Product judgment at frontier scale.** "Design ChatGPT" and the 90-day question test whether you'd build the *right* thing, not just a correct thing.
- **Safety as an engineering constraint.** Jailbreak/harm questions want eval-backed mitigations, not hand-waving.
- **Alignment with the Charter.** Behavioral rounds probe mission fit and collaboration, not just competence.

## How to prep

- **Attention / transformer from memory** → work through [content/01](../content/01-transformers-and-attention.md); rehearse writing multi-head attention with a KV-cache cold.
- **RLHF / PPO / VAE** → [content/03](../content/03-training-and-alignment.md) for the RLHF pipeline mental model at 70B scale.
- **"Design ChatGPT" / serving stack** → the worked design in [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) (batching, KV-cache, quantization, autoscaling) and [answers/support-bot.md](../answers/support-bot.md) for the product-shaped variant.
- **Evaluation pipeline** → [answers/eval-pipeline.md](../answers/eval-pipeline.md); be ready to design the eval *before* the system.
- **RAG fundamentals** → [content/02](../content/02-rag.md).
- **Question bank** → [questions/](../questions/) for the LLM-fundamentals and system-design tiers.

## Culture signals

Alignment with the **OpenAI Charter**, concrete **impact evidence**, and **collaboration** [75], [95]. Come with a crisp story of something you shipped and can defend end-to-end.

## Cross-links

- 🛠️ Worked design: [Design ChatGPT / LLM inference at scale](../answers/llm-inference-at-scale.md)
- 🛠️ Worked design: [Support bot / "Design ChatGPT" product framing](../answers/support-bot.md)
- 🛠️ Worked design: [Evaluation pipeline](../answers/eval-pipeline.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with OpenAI. MIT-licensed.</sub>
