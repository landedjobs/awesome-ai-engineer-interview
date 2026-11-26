# Notion · AI Engineer Interview (2026)

🚀 *"Build the product."* Notion publishes its own interview guides, and its AI Applications Engineer loop has the clearest public AI policy of any company here: AI is banned in every coding round except one, where it is strongly encouraged.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at Notion](https://www.landed.jobs/companies/notion?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full Notion loop guide with a prep plan](https://www.landed.jobs/resources/guides/notion-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

> [!NOTE]
> **Evidence type.** Everything on this page comes from Notion's **official, undated interview guides**, read on 17 Nov 2026: the [AI Applications Engineer guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c) and the [general Software Engineering guide](https://notion.notion.site/Software-Engineering-Interviews-63876cbf0aa9455fa0f0ffc854b40f6a). We found **no verifiable candidate reports** for this role, so there are no ✅ Reported questions. The two guides describe different tracks; they are kept separate below.

> [!NOTE]
> **The loop at a glance** (AI Applications Engineer, official). Recruiter (~20 min) → a 60-minute backend coding screen where you run and test your code → a virtual onsite of **four interviews**: career history, a second coding interview, software design, and an **AI-enabled backend coding interview** → a 30-minute values and motivation conversation with the Head of Data → references and a background check. All interviews after the recruiter call are on Google Meet. A take-home is **not part of the published loop**. Total elapsed time is **not publicly confirmed**.
>
> Last checked: 17 Nov 2026.

## The loop

**AI Applications Engineer** ([official guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c)):

1. **Initial interview (~20 min)** with a recruiter: your background, what you want next, what Notion is building.
2. **Technical screen, backend coding (60 min)**: pair with a Notion engineer on a real-world problem; you run and test the code. Agenda: 5 min intros, 45 min problem solving, 5 to 10 min Q&A ([screen prep page](https://notion.notion.site/020644ad2c944fb29b9ae1c744e12b94d)).
3. **Virtual onsite (4 interviews):**
    - **Career history and experience**: non-technical; achievements, challenges, how you worked with others.
    - **Additional coding interview**: implement an abstract interface; no specific library knowledge tested; write and run code.
    - **Software design**: an open-ended problem in a shared doc, with optional diagrams. Agenda: 5 min intros, 45 min problem, 5 to 10 min Q&A ([design page](https://notion.notion.site/b94dfd91ac804139bb8b67688ce858a4fc7)).
    - **AI-enabled coding interview (backend)**: AI coding tools allowed and "strongly encouraged". Language agnostic, no UI ([AI-enabled coding page](https://notion.notion.site/264efdeead058094b3c3efa618b9fe95)).
4. **Post-onsite (30 min)**: past experience, values and motivations with the Head of Data.
5. **Reference checks**, and an offer contingent on a background check.

**General software engineering track** ([official guide](https://notion.notion.site/Software-Engineering-Interviews-63876cbf0aa9455fa0f0ffc854b40f6a)), for comparison: initial interview with Talent → a live coding technical assessment → panel of **two 60-minute coding interviews**, a **60-minute design** interview to a rough spec, and a **60-minute career history** interview → a final interview with a leadership team member → references. This guide does not list an AI-enabled round.

### AI tool policy (official)

| Round | AI coding tools | Source |
|---|---|---|
| Technical screen and standard coding | Not allowed to generate code. Docs, search and Google AI Overviews for lookups are fine. | [screen prep page](https://notion.notion.site/020644ad2c944fb29b9ae1c744e12b94d) |
| AI-enabled coding (backend) | Allowed and strongly encouraged: an AI IDE such as Cursor or VS Code with Copilot, or an agentic CLI such as Claude Code next to an editor | [AI-enabled coding page](https://notion.notion.site/264efdeead058094b3c3efa618b9fe95) |
| Everything else | Not allowed to write code for you; syntax lookup is fine | [AI Applications Engineer guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c) |

## Questions reported in the wild

No candidate-reported questions for this role could be verified. The rows below are Notion's **own descriptions** of each round's task, labeled 📘 Official.

| Round | What Notion says you'll do | Provenance |
|---|---|---|
| Technical screen | Build a solution to a real-world problem, then run and test it | 📘 Official · [AI Applications Engineer guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c) |
| Additional coding | Write code that implements an abstract interface | 📘 Official · [AI Applications Engineer guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c) |
| Software design | An open-ended software design problem | 📘 Official · [design page](https://notion.notion.site/b94dfd91ac804139bb8b67688ce858a4fc7) |
| AI-enabled coding | Backend problem with AI tools; you prompt, iterate, validate and debug AI code, then make it production-ready | 📘 Official · [AI-enabled coding page](https://notion.notion.site/264efdeead058094b3c3efa618b9fe95) |
| Career history | Highlights and lowlights of each major role, who you worked with, how teams operated | 📘 Official · [AI Applications Engineer guide](https://notion.notion.site/2b7efdeead05807bb43de0444435812c) |

### 🔮 Representative follow-ups you should expect

*Not reported. Natural probes given the official round descriptions.*

- In the AI-enabled round: *"The agent wrote this function. What would you change before you'd merge it?"*
- In the AI-enabled round: *"Why did you stop prompting here and write this part by hand?"*
- In the interface round: *"Add a second implementation of the interface. What did your first design make hard?"*
- In software design: *"Design an AI feature that answers questions over a user's workspace and respects page permissions."*
- In software design: *"How do you know the AI feature got better after a model swap?"*
- In career history: *"What exactly did you build on that project, as opposed to your team?"*

## What they're really testing

- **Two coding modes in one loop.** You must solve problems cleanly with no AI help, and also show you can steer AI tools well. Notion says it restricts AI elsewhere to see how you solve problems yourself.
- **Judgment over AI output.** The AI-enabled guide focuses on validating and debugging AI-generated code and turning it into maintainable code, not on how fast the agent types.
- **Running, tested code.** The screen expects you to run and test within the hour. Tests are not required, but asserting correctness is encouraged.
- **Depth over breadth in your history.** Notion asks for fewer things in more detail, with outcomes and impact, and with your own contribution clear.

## How to prep

- **AI-enabled coding round** → [content/10](../content/10-ai-assisted-coding-round.md). Rehearse with the tool you'll use (Cursor, Copilot or Claude Code) on a backend task, and practice reading and fixing what it writes out loud.
- **No-AI coding rounds** → [questions/coding.md](../questions/coding.md); practice implementing a small interface (a cache, a queue, a key-value store) and running it within 45 minutes.
- **Software design for AI features** → [content/07](../content/07-ml-and-llm-system-design.md), [answers/system-design-rubric.md](../answers/system-design-rubric.md) and [questions/system-design.md](../questions/system-design.md).
- **Retrieval over a workspace** → [content/02](../content/02-rag.md) and [answers/semantic-search.md](../answers/semantic-search.md).
- **Showing an AI feature works** → [content/04](../content/04-evals-and-llm-as-judge.md).
- **Career history** → [questions/behavioral.md](../questions/behavioral.md); prepare highlights and lowlights per role, with numbers.

## Culture signals

Notion asks candidates to read its **operating values** before the final interview, and says interviews are **collaborative by design**: ask clarifying questions and talk through your approach (official guides). It also says it wants to see what working with you would be like. Notion calls itself a big fan of AI, yet restricts it in all but one round to keep the process "consistent and fair".

## Cross-links

- 🛠️ Worked design: [Semantic search](../answers/semantic-search.md)
- 🛠️ Worked design: [Coding agent](../answers/coding-agent.md)
- 🛠️ Rubric: [System design rubric](../answers/system-design-rubric.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · Last checked: 17 Nov 2026 · No affiliation with Notion. MIT-licensed.</sub>
