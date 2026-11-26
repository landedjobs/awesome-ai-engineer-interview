# Harvey: AI Engineer Interview (2026)

🚀 *"Build the product."* Harvey builds AI agents for legal and professional work. Its Agents posting reads like an applied-AI job description: evals, tools, context windows, model choice. The public interview evidence is thinner and comes from adjacent roles.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at Harvey](https://www.landed.jobs/companies/harvey?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full Harvey loop guide with a prep plan](https://www.landed.jobs/resources/guides/harvey-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

*Last checked: 17 Nov 2026*

> [!IMPORTANT]
> **Role and evidence.** No public first-person account of the **Software Engineer, Agents** loop could be verified. This page uses two anonymous first-person accounts from **adjacent roles** (a Senior Software Engineer in 2026 and a Principal MLOps Engineer in about late 2025), plus Harvey's own job posting and hiring FAQ. A job posting describes the job, not the interview. The full current Agents loop, round durations, take-home and AI-assistant policy are **not publicly confirmed**. Ask your recruiter.

> [!NOTE]
> **The loop at a glance (two different candidates, not one loop).** A 2026 Senior SWE candidate reports a **recruiter screen** then a **multi-part coding interview** with follow-ups, two weeks in total. A Principal MLOps candidate from about a year earlier reports **five rounds on separate days**: recruiter, online assessment, live coding, a **three-person architecture panel**, and a director chat, about a month in total. The distinguishing detail is in the MLOps account: live coding that mixed an algorithm problem with **tokenization, embeddings and vector database** questions.

## The loop

What Harvey says officially:

- The [Software Engineer, Agents posting](https://jobs.ashbyhq.com/harvey/fc038666-be6f-4365-be8d-fb46e520473d) (official, published May 2026, San Francisco; listed as Staff, but the text says it hires across Mid, Senior and Staff levels) describes the work: designing environments and actions for agents, model selection, context window management, tool design, and building evals to iterate faster. It asks for Python and experience with LLM APIs and agent frameworks. It does not describe interview rounds.
- The [hiring FAQ by Harvey's COO](https://www.harvey.ai/blog/landing-a-job-at-harvey) (official, 5 Aug 2026) covers values, office expectations and whether you need legal experience. It does not list interview rounds.

What candidates report (each is one person; do not combine them):

**Account A: Senior Software Engineer** ([Aced](https://www.aced.io/experiences/harvey-ai-senior-software-engineer-interview-da6149), first-person, anonymous, US, hybrid, interview about Aug 2026, rejected, two weeks)

1. **Recruiter screen**: your experience, your role on your most recent team, and what you want next.
2. **Technical interview**: a coding question with multiple parts and follow-ups. The candidate's main advice is to be fast in your own language. Duration not stated.

**Account B: Principal MLOps Engineer** ([Aced](https://www.aced.io/experiences/harvey-ai-principal-machine-learning-engineer-interview-2cd0ae), first-person, anonymous, US, remote, interview about late 2025, offer, about one month)

1. **Recruiter screen**: formulaic; compensation came up right away.
2. **Online assessment.**
3. **Live coding**: an algorithmic problem mixed with tokenization, embeddings and vector database questions.
4. **Architecture panel (three interviewers)**: pushed on tradeoffs and latency, and on explaining the same architecture to technical and non-technical people.
5. **Director chat.**

Take-home: **not publicly confirmed** in either account. AI-assistant policy: **not publicly confirmed**.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Recruiter (Senior SWE) | Tell me about your experience and role on your most recent team. What are you looking for in this job? | ✅ Reported · [Aced, Senior SWE, 2026](https://www.aced.io/experiences/harvey-ai-senior-software-engineer-interview-da6149) |
| Technical (Senior SWE) | A multi-part coding question with follow-ups (exact problem not public) | ✅ Reported · [Aced, Senior SWE, 2026](https://www.aced.io/experiences/harvey-ai-senior-software-engineer-interview-da6149) |
| Recruiter (MLOps) | What is your expected compensation range, and is that the full package? | ✅ Reported · [Aced, Principal MLOps, about 2025](https://www.aced.io/experiences/harvey-ai-principal-machine-learning-engineer-interview-2cd0ae) |
| Live coding (MLOps) | An algorithmic problem combined with tokenization, embeddings and vector database questions | ✅ Reported · [Aced, Principal MLOps, about 2025](https://www.aced.io/experiences/harvey-ai-principal-machine-learning-engineer-interview-2cd0ae) |
| Architecture panel (MLOps) | Present an architecture; defend tradeoffs and latency; explain it to technical and non-technical listeners | ✅ Reported · [Aced, Principal MLOps, about 2025](https://www.aced.io/experiences/harvey-ai-principal-machine-learning-engineer-interview-2cd0ae) |

### 🔮 Representative questions for the Agents role

*Not reported. These follow from the work the Agents posting describes, so they are worth preparing, but no candidate has confirmed them.*

- *"Design an agent that answers diligence questions across hundreds of thousands of deal documents. How do you retrieve, cite and verify?"*
- *"A lawyer says the agent's contract review is wrong 1 time in 20. How do you build an eval that captures what 'correct' means to them?"*
- *"Your agent's context window fills up on a long matter. What do you keep, summarise or drop?"*
- *"When would you switch models for one step of a workflow, and how would you prove it was worth it?"*
- *"How would you cut agent latency with caching, parallel tool calls or subagents, and what can go wrong with each?"*
- After the embeddings question: *"Why did retrieval miss this clause, and what would you change first: chunking, the embedding model, or the query?"*

## What they're really testing

Based on the two accounts and the official posting, with the caveat that neither account is for the Agents role:

- **Coding speed under follow-ups.** The Senior SWE account's one lesson is speed on a multi-part problem.
- **Retrieval fundamentals inside coding.** The MLOps account mixed tokenization, embeddings and vector search into the coding round rather than a separate ML round.
- **Architecture you can explain to anyone.** The panel asked for the same design explained to technical and non-technical people. At Harvey the end users are lawyers.
- **Evals that match expert judgment.** The posting puts practical evaluation at the center of the Agents role: defining what "excellent" means with customers and PMs, then using evals to drive quality.
- **Agent engineering craft.** Model choice, tool design, context management, latency and observability are all named in the posting.

## How to prep

- **Agents, tools and context windows** → [content/03](../content/03-agents-and-tool-use.md) and [answers/agentic-workflow.md](../answers/agentic-workflow.md).
- **Retrieval over large document sets** → [content/02](../content/02-rag.md) and [answers/rag-over-10m-docs.md](../answers/rag-over-10m-docs.md). Practice explaining tokenization and embeddings while you code; see [content/01](../content/01-llm-fundamentals.md).
- **Evals with domain experts** → [content/04](../content/04-evals-and-llm-as-judge.md) and [answers/eval-pipeline.md](../answers/eval-pipeline.md). Anthropic's [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (📰 article) is a useful frame.
- **Grounded answers with no invented facts** → [answers/hallucination-free-banking-chatbot.md](../answers/hallucination-free-banking-chatbot.md) for citation and verification patterns that carry over to legal work.
- **Latency and cost** → [content/05](../content/05-llmops-cost-latency.md).
- **Coding speed** → [questions/coding.md](../questions/coding.md). Time yourself on multi-part problems in your strongest language.
- **Take-home or work sample** → not publicly confirmed for Harvey. If one comes up, read [content/11](../content/11-take-homes-and-work-trials.md). A legal-document agent built from the [portfolio project guides](https://github.com/landedjobs/ai-engineer-portfolio-projects/tree/main/guides) (💻 repo) also gives you a project to walk through.
- **Architecture panel and behavioral** → [questions/system-design.md](../questions/system-design.md) and [questions/behavioral.md](../questions/behavioral.md).

## Culture signals

Harvey names three values: **decisiveness, simplicity and "Job's Not Finished"** ([hiring FAQ](https://www.harvey.ai/blog/landing-a-job-at-harvey), official, Aug 2026; repeated in the [Agents posting](https://jobs.ashbyhq.com/harvey/fc038666-be6f-4365-be8d-fb46e520473d)). The FAQ says most staff work from an office three or more days a week, and that legal experience is not required but curiosity about customers' daily work is. It is also candid that the job is not a classic 9 to 5.

## Cross-links

- 🛠️ Worked design: [Agentic workflow](../answers/agentic-workflow.md)
- 🛠️ Worked design: [RAG over 10M docs](../answers/rag-over-10m-docs.md)
- 🛠️ Worked design: [Evaluation pipeline](../answers/eval-pipeline.md)
- 📄 Guide: [Take-homes and work trials](../content/11-take-homes-and-work-trials.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Harvey. MIT-licensed. Last checked: 17 Nov 2026.</sub>
