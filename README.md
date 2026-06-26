# Awesome AI Engineer Interview [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> The open prep guide for **AI Engineer interviews** — real questions, what's actually tested, and how to answer.
> Maintained by [Landed](https://landed.b100x.ai) — scout roles, get **referred**, and run company-specific mock interviews.

⭐ **Star this** to prep smarter. PRs with new questions welcome.

---

## What an AI Engineer interview actually tests

Most AI Engineer loops (2026) test five things — usually **not** ML theory from scratch:

1. **LLM application design** — RAG, agents, tool use, when to fine-tune vs prompt.
2. **Practical coding** — Python, APIs, data wrangling; ship a small working thing.
3. **Evals & reliability** — how you measure and improve an LLM system.
4. **System design** — retrieval, caching, latency, cost, guardrails.
5. **Product sense** — does your solution actually help the user.

---

## Core questions (with what they're looking for)

### LLM & RAG
- **Walk me through a RAG pipeline you'd build for X.** → chunking, embeddings, vector store, retrieval, re-ranking, grounding, citations.
- **How do you reduce hallucinations?** → grounding in retrieved context, citations, constrained decoding, evals, human-in-loop.
- **When would you fine-tune instead of prompt/RAG?** → stable task, style/format control, latency/cost at scale; RAG for fresh/factual.
- **How do you chunk documents well?** → semantic vs fixed, overlap, metadata, eval on retrieval quality.

### Agents
- **Design an agent that does X.** → tools/function-calling, planning, memory, termination conditions, failure handling, cost control.
- **How do you keep an agent from looping or going off-task?** → step limits, validation, guardrails, evals.

### Evals
- **How would you evaluate an LLM feature?** → offline eval sets, LLM-as-judge (with caveats), human review, regression tests, production monitoring.
- **Your accuracy dropped after a model upgrade. What do you do?** → eval harness, diff outputs, prompt/version pinning, rollback.

### Coding / practical
- Build a small endpoint that takes a query, retrieves context, and answers with citations.
- Parse messy data into a clean structured output with an LLM + validation.

### System design
- **Design a production question-answering system over 10M documents.** → ingestion, embeddings, vector DB, retrieval, caching, latency/cost, monitoring, guardrails.

### Behavioral
- A time you shipped fast under ambiguity; a project you're proud of; how you handle being wrong.

---

## How to prepare (2-week plan)

1. Build one **RAG app** and one **agent** end-to-end (see [projects-to-land-an-ai-job](https://github.com/landedjobs/projects-to-land-an-ai-job)).
2. Write an **eval set** for each — be ready to talk about how you measure quality.
3. Practice explaining trade-offs out loud (prompt vs RAG vs fine-tune).
4. Run **company-specific mock interviews** on [Landed](https://landed.b100x.ai).

---

## Company-specific guides

Landed maintains interview guides + real candidate experiences per company and role. **[Find your company's AI Engineer guide →](https://landed.b100x.ai)**

---

## Resources

- LLM app patterns, eval frameworks, vector DB docs (community-maintained — PRs welcome).

## FAQ

**Do I need to grind LeetCode?** Some companies still include a coding round, but AI Engineer loops lean toward practical LLM building + system design. Balance both.

**No production LLM experience — can I still pass?** Yes, if you've **built and can explain** real projects. Demonstrated work + clear reasoning about trade-offs is what interviewers want.

---

<sub>Maintained by [Landed](https://landed.b100x.ai). Prep with company-specific guides + mock interviews.</sub>
