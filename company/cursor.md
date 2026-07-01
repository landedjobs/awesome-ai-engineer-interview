# Cursor / Anysphere — AI Engineer Interview (2026)

🚀 *"Build the product."* The AI-code-editor lab **bans AI in its interviews** and runs an **8-hour paid onsite** on a real project — the loop itself is a statement about craft.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** Recruiter (30 min) → **1–3 technical phone screens** (60 min, applied AI-systems) → an **8-hour paid onsite** building a real project with the team → a **2-day in-office final**. The distinguishing signal *is* the format: **no AI allowed in interviews**, and you're paid to build something real. It's the exact inverse of Meta's "Coding with AI" round.

## The loop

1. **Recruiter call (30 min).**
2. **Technical phone screens (1–3 × 60 min)** — applied AI-systems.
3. **8-hour paid onsite** — a real project, with the team, no AI assistance.
4. **2-day in-office final** — building projects with the team.

Source: candidate reports [40], [41], [42], [43], [44]; "no AI in interviews" [44].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Phone technical | "Implement a flexible rate-limiter for API endpoints using per-user rules and interfaces" | ✅ Reported — candidate report [41] |
| Phone technical | Applied AI-systems (build an AI-driven code-completion system) | ✅ Reported — candidate report [43] |
| Onsite paid project | 8-hour real project with the team | ✅ Reported — candidate report [43] |
| Final 2-day office | Building projects with the team | ✅ Reported — candidate report [44] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After the rate-limiter: *"Now support burst allowances and per-tier rules without rewriting the interface."*
- In the code-completion round: *"How do you keep latency under 100ms while ranking multiple completions?"*
- In the paid onsite: *"Ship a working slice by hour 4 — what do you cut, and why?"*

## What they're really testing

- **Craft without a copilot.** Banning AI is the whole point: they want to see how *you* think, design interfaces, and debug.
- **Applied AI-systems fluency.** Code completion and low-latency serving are the domain — know the product's problems.
- **Real-project execution.** The 8-hour paid onsite measures shipping a working slice under real constraints, not toy problems.
- **Team fit.** The 2-day in-office final is as much about collaboration as code.
- **Interface design.** The rate-limiter question is explicitly about clean interfaces and extensibility.

## How to prep

- **Rate limiter / clean interfaces** → [questions/](../questions/) coding tier; practice a token-bucket limiter with per-user rules and a clean interface.
- **AI code completion / applied systems** → [answers/agentic-workflow.md](../answers/agentic-workflow.md) and [content/05](../content/05-ai-assisted-engineering.md).
- **Latency-sensitive serving** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md).
- **No-AI muscle** → practice implementing from scratch, no autocomplete — the opposite of your day job.

## Culture signals

An **applied research lab** obsessed with AI-code-editor craft. **"No AI in interviews" is itself the signal** — they hire engineers who can build without leaning on the tool.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / applied AI systems](../answers/agentic-workflow.md)
- 🛠️ Worked design: [LLM inference at scale / low-latency serving](../answers/llm-inference-at-scale.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Cursor / Anysphere. MIT-licensed.</sub>
