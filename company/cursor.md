# Cursor / Anysphere — AI Engineer Interview (2026)

🚀 *"Build the product."* The AI-code-editor lab reportedly runs its **first technical screens without AI** (autocomplete aside) and ends with a **two-day project in the office with the team**. The loop itself is a statement about craft.

← Back to [main README](../README.md) · [company index](README.md)

*Last checked: 17 Nov 2026*

> [!NOTE]
> **The loop at a glance.** Cursor's [careers page](https://cursor.com/careers) (official, checked Nov 2026) does not publish a process. **June 2025 reporting** on the CEO's podcast remarks says first technical screens allow no AI other than autocomplete, and the final step is **two days in the office** on a real project with the team, ending in a demo. A later prep guide built from candidate reports describes a **45-minute** recruiter call → **1-3 technical screens of 60 minutes** → that two-day project, or a shorter remote version of **roughly 8 hours**. Whether the project is paid is **not publicly confirmed**. It's close to the inverse of Meta's "Coding with AI" round.

## The loop

What the CEO said, as reported ([Business Insider, 12 Jun 2025](https://www.businessinsider.com/cursor-interview-process-no-ai-on-site-project-coding-tool-2025-6), second-hand, reporting on Michael Truell's Y Combinator podcast remarks):

- First technical screens run **without AI other than autocomplete**. He framed AI-free programming as a time-boxed test of skill.
- The final step is not a traditional interview. Shortlisted candidates spend **two days in the office**, work on a real project with the team, join meals, and **demo** what they built.
- The article does not give the full round sequence, pay, or AI rules during the two-day project.

What a prep guide says ([Aced Cursor SWE guide](https://www.aced.io/guides/cursor-software-engineer-interview), second-hand, built from candidate and interviewer reports, updated about Sep 2026):

1. **Recruiter call (about 45 min)**: informal, on your interest in Cursor. Sometimes run by the hiring manager.
2. **Technical screens (1-3 × 60 min)**: building on a real Cursor repository with an engineer or the hiring manager. AI access varies: the earliest screen may allow only editor autocomplete.
3. **Project onsite**: for most engineering hires, a two-day in-person project with the team. Some candidates, including new grads, report a remote version of roughly 8 hours. The guide says AI tools are open at the onsite.

These two sources disagree on AI use during the project. Ask your recruiter. Pay for the project: **not publicly confirmed** in either source. For how to approach a multi-day trial, see [take-homes and work trials](../content/11-take-homes-and-work-trials.md).

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Technical screen | Build a hash tree to organize data in a repository | ✅ Reported · [Aced guide, SWE, 2026](https://www.aced.io/guides/cursor-software-engineer-interview) |
| Technical screen | Find duplicate files in a file system | ✅ Reported · [Aced guide, SWE, 2026](https://www.aced.io/guides/cursor-software-engineer-interview) |
| Onsite | Brainstorm a product idea with the help of AI | ✅ Reported · [Aced guide, SWE, 2026](https://www.aced.io/guides/cursor-software-engineer-interview) |
| Onsite | How would you handle hallucinations in a deployed generative AI model? | ✅ Reported · [Aced guide, SWE, 2026](https://www.aced.io/guides/cursor-software-engineer-interview) |
| Final (two days in office) | Build a real project with the team and demo it at the end | ✅ Reported · [Business Insider, Jun 2025](https://www.businessinsider.com/cursor-interview-process-no-ai-on-site-project-coding-tool-2025-6) |
| Phone technical | Implement a flexible rate-limiter for API endpoints using per-user rules and interfaces | 🔮 Representative (earlier reported label removed: no source we could open) |
| Phone technical | Build an AI-driven code-completion system | 🔮 Representative (earlier reported label removed: no source we could open) |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After a rate-limiter: *"Now support burst allowances and per-tier rules without rewriting the interface."*
- In a code-completion discussion: *"How do you keep latency under 100ms while ranking multiple completions?"*
- In the project: *"Ship a working slice by the end of day one. What do you cut, and why?"*
- After duplicate files: *"The repo is 50 GB. Hash everything, or filter by size first?"*

## What they're really testing

- **Craft without a copilot.** Keeping AI out of the first screens is the point: they want to see how *you* think, design interfaces, and debug.
- **Working in a real codebase.** The guide describes screens built on a real Cursor repository, so reading unfamiliar code matters.
- **Real-project execution.** The two-day project measures shipping a working slice under real constraints, not toy problems.
- **Team fit.** Two days in the office, meals included, is as much about collaboration as code.
- **Applied AI-systems fluency.** Code completion and low-latency serving are the domain — know the product's problems.

## How to prep

- **Clean interfaces** → [questions/](../questions/) coding tier; practice a token-bucket limiter with per-user rules and a clean interface.
- **AI code completion / applied systems** → [answers/agentic-workflow.md](../answers/agentic-workflow.md) and [content/05](../questions/coding.md).
- **Latency-sensitive serving** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md).
- **No-AI muscle** → practice implementing from scratch, no autocomplete — the opposite of your day job.
- **The two-day project** → [take-homes and work trials](../content/11-take-homes-and-work-trials.md); confirm dates, pay and AI rules before you block two days.

## Culture signals

An **applied research lab** obsessed with AI-code-editor craft. **Keeping AI out of first screens is itself the signal.** The CEO said they would rather hire strong programmers without AI-tool experience and teach them ([Business Insider, Jun 2025](https://www.businessinsider.com/cursor-interview-process-no-ai-on-site-project-coding-tool-2025-6)). Whether this still holds in late 2026 is not publicly confirmed.

## Cross-links

- 🛠️ Worked design: [Agentic workflow / applied AI systems](../answers/agentic-workflow.md)
- 🛠️ Worked design: [LLM inference at scale / low-latency serving](../answers/llm-inference-at-scale.md)
- 📘 Concept page: [Take-homes and work trials](../content/11-take-homes-and-work-trials.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Cursor / Anysphere. MIT-licensed.</sub>
