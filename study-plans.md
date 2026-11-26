# 🗓️ Study plans: six weeks to an AI Engineer loop

<sub>⬅ [Back to the main README](README.md) · [Question bank](questions/README.md) · [Content](content/) · [Worked designs](answers/) · [Company loops](company/README.md) · Part of **[awesome-ai-engineer-interview](README.md)**, maintained by [Landed](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=study-plans)</sub>

Six weeks, one deliverable per week, and a self-check you can answer yes or no. Then two real candidates' public write-ups, with what each one does and does not prove.

> [!NOTE]
> **The six-week plan is ours.** It is an editorial plan built from the task types that show up in public 2025-2026 candidate accounts. It is not a schedule any successful candidate published. Change the weights once your recruiter tells you the real rounds.

*Last checked: 17 Nov 2026.*

---

## Contents

- [Before you start: ask your recruiter](#before-you-start-ask-your-recruiter)
- [The six-week plan](#the-six-week-plan)
- [Narrow it for your role](#narrow-it-for-your-role)
- [Case study: Alisa Liu](#case-study-alisa-liu-foundations-implementation-and-ai-off-practice)
- [Case study: Yuan Meng](#case-study-yuan-meng-prepare-for-the-rounds-you-actually-have)
- [What these accounts do and do not establish](#what-these-accounts-do-and-do-not-establish)

---

## Before you start: ask your recruiter

Loops differ by company, team and level, so the cheapest prep is a precise question. Ask these on the first call. Not every employer will answer all of them, but each answer changes what you study.

1. What are the rounds, how long is each, and what is each one graded on, for this exact role and level?
2. Which rounds allow AI tools? Which tools or models are available, and is internet access allowed?
3. Is the coding algorithmic, service-style, work inside an existing repository, ML implementation, debugging, or a mix?
4. Is there a take-home or a paid work trial? How much effort is expected, by when, is it paid, and how is it reviewed?
5. Will there be rounds on evaluation, safety, model serving, or customer and product work?
6. What should I prepare for the project deep dive, and how much detail on my own contribution do you expect?
7. What is the timeline to a decision, and can you work around deadlines from other processes?

> [!TIP]
> Write the answers into a one-page matrix: company, role, each round, AI allowed or not, and what you will practise for it. Week 1 below starts from that page. For what is already public about each company's loop, see [company/](company/README.md).

---

## The six-week plan

| Week | Focus | Deliverable | Self-check | Study in this repo |
|---|---|---|---|---|
| **1** | Map the target loop and get a baseline | The company and role matrix from your recruiter answers. One timed coding problem with no autocomplete and no AI. A small tested Python or TypeScript service. | Can you write, run and explain a solution with every assistant switched off? | [company/](company/README.md) · [questions/coding](questions/coding.md) (C9 to C11, C15) |
| **2** | LLM fundamentals | Small exercises: attention with a causal mask, a tokenizer round trip, temperature and top-p sampling, and a KV-cache memory worksheet. | Can you state tensor shapes, explain masking and sampling, and show your memory arithmetic with its assumptions? | [content/01](content/01-llm-fundamentals.md) · [questions/fundamentals](questions/fundamentals.md) · [questions/coding](questions/coding.md) (C1 to C3, C16) |
| **3** | RAG | An app that answers with citations, a simple retrieval baseline, and a held-out question set. | Can you tell a retrieval failure from a generation failure, and show one change that improved the right metric? | [content/02](content/02-rag.md) · [questions/rag](questions/rag.md) · [answers/rag-over-10m-docs](answers/rag-over-10m-docs.md) |
| **4** | Agents and tool use | A bounded tool-using workflow with injected failures and saved traces. | Does it handle a timeout, invalid tool output, a duplicate action and a clear stop condition? | [content/03](content/03-agents-and-tool-use.md) · [questions/agents](questions/agents.md) (Q30) · [answers/agentic-workflow](answers/agentic-workflow.md) |
| **5** | Evaluation, serving and cost | An eval harness with error analysis, notes from a small load test, and a cost breakdown per completed task. | Can you defend a ship or no-ship decision on quality, safety, latency and cost together? | [content/04](content/04-evals-and-llm-as-judge.md) · [content/05](content/05-llmops-cost-latency.md) · [content/09](content/09-security-and-guardrails.md) · [answers/eval-pipeline](answers/eval-pipeline.md) · [answers/llm-inference-at-scale](answers/llm-inference-at-scale.md) |
| **6** | Rehearse the exact formats | Mocks of each round type you will face: coding with no AI, AI-assisted work in a repository, debugging, system design and a project deep dive. | Can you finish the real round type, explain the tradeoffs, and recover when the AI suggests something wrong? | [content/10](content/10-ai-assisted-coding-round.md) · [answers/system-design-rubric](answers/system-design-rubric.md) · [questions/system-design](questions/system-design.md) · [questions/behavioral](questions/behavioral.md) |

**Each week, the same loop:** read the concept page, answer the checkpoints before opening the folds, build the deliverable, then explain it out loud in two minutes. Keep a short log of every mistake by type (wrong algorithm, messy structure, concurrency, tensor shapes, missing tests, weak explanation) and pick the next exercise to fix the most common one. The log is our suggestion; it is not taken from either case study.

> [!WARNING]
> **Don't let the assistant hide your gaps.** Do Week 1 and part of every week with AI switched off. Both candidates below say this separately: Alisa Liu advises practising with AI assistance completely off, and Yuan Meng found a mock interview exposed design problems her passing tests had hidden.

---

## Narrow it for your role

| If the role is... | Spend more time on | Spend less time on |
|---|---|---|
| **Research or model-facing** (research engineer, member of technical staff) | PyTorch fluency, implementing architectures and decoding from scratch, maths, distributed training ([fundamentals Q52](questions/fundamentals.md), [content/06](content/06-fine-tuning-and-inference.md)) | Customer requirements and product framing |
| **Applied or product AI engineer** | APIs, repository work, evals, cost, and turning a customer need into requirements ([questions/evals](questions/evals.md), [questions/llmops](questions/llmops.md)) | Writing training code from scratch |
| **Customer-facing** (forward deployed, solutions) | Explaining an architecture to an executive and handling pushback ([behavioral Q10](questions/behavioral.md)), plus the applied list above | Deep model internals |

This is a preparation recommendation. It is not a claim about how any employer scores its rounds.

---

## Case study: Alisa Liu, foundations, implementation and AI-off practice

**Source:** [Notes on the Industry Job Search](https://alisawuffles.github.io/blog/job-search), her own blog post, dated 20 Jun 2026. Evidence type: **first-person, named**.

**Who and what.** She searched for Research Scientist / Member of Technical Staff roles at the end of an NLP PhD at the University of Washington. She reports 11 companies and 57 interviews. Her [public post on X](https://x.com/alisawuffles/status/2068765723569324462) says she was joining OpenAI the following week, and [her homepage](https://alisawuffles.github.io/) now lists her as a researcher at OpenAI.

**What she did, in order:**

1. Watched all the lectures of [Stanford CS336, Language Modeling from Scratch](https://cs336.stanford.edu/spring2025/) to see the full breadth of topics.
2. Studied one concept at a time through papers, blog posts and talking to AI models, and implemented things from scratch.
3. Turned implementing and debugging a transformer into muscle memory, using CS336 homework 1 ([assignment repo](https://github.com/stanford-cs336/assignment1-basics)).
4. Practised coding with AI assistance completely off, because otherwise you underestimate how much you rely on it.
5. Prepared for each interview separately, using the round description, the company's interests and hints from the recruiter.
6. Slept properly before interviews and wrote notes after each one.

**Rounds she describes:** ML coding (the most common), general coding, technical discussion, research discussion, behavioral, maths and a job talk. Questions from her post are in this repo as [fundamentals Q51 and Q52](questions/fundamentals.md), [coding C16](questions/coding.md) and [behavioral Q11](questions/behavioral.md).

**Use this when** your loop has ML coding or deep technical discussion: learn a mechanism, implement it, explain it, then test yourself with no help. In this repo: [content/01](content/01-llm-fundamentals.md), [content/06](content/06-fine-tuning-and-inference.md), and [coding C1 to C4](questions/coding.md).

---

## Case study: Yuan Meng, prepare for the rounds you actually have

**Source:** *MLE Interview 2.0: Research Engineering and Scary Rounds*, her own blog post dated 1 Feb 2026. The original address on yuan-meng.com did not resolve when we checked on 17 Nov 2026, so we link the [post's source file in her public website repository](https://github.com/Yuan-Meng/personal_website_2.0/blob/main/content/posts/mle_interviews_2.0/index.md). Evidence type: **first-person, named**.

**Who and what.** An ML engineer who interviewed for applied ML engineer and research engineer roles at the end of 2025 and the start of 2026. She reports onsites at between 5 and 9 companies and offers from all but one; the onsite she failed was for a backend role. The post does not say which offer she accepted.

**What she did:**

1. **Asked which rounds first.** She starts preparing only once phone screens are booked and recruiters have explained each round, because different companies test very different things.
2. **Prepared per round type:** algorithm coding, multi-level object-oriented coding, ML coding, AI-assisted coding, ML model design, ML infra design, ML fundamentals and behavioral.
3. **For Meta's AI-assisted coding round,** practised the example in the candidate portal, made a plan with the interviewer, got the unit tests passing, and could explain why tests failed and why her changes fit the intended behaviour. She reports passing it.
4. **Used a mock interview to find a hidden weakness.** A mock with a backend engineer showed her object-oriented code passed tests but made design choices a backend engineer would not.

Questions from her post are in this repo as [coding C13, C14 and C16](questions/coding.md).

**Use this when** you have several loops with different formats. Write down the rounds per company, prepare for those, and use mocks to find what tests alone do not show. In this repo: [content/10](content/10-ai-assisted-coding-round.md) for the AI-assisted round and [company/meta.md](company/meta.md) for the Meta loop.

---

## What these accounts do and do not establish

| Account | Documented method | Reported outcome | Does not establish |
|---|---|---|---|
| [Alisa Liu](https://alisawuffles.github.io/blog/job-search) (Jun 2026) | CS336 lectures, then one concept at a time, implementation, AI-off practice, per-interview prep | Joining OpenAI, per [her post on X](https://x.com/alisawuffles/status/2068765723569324462) and [her homepage](https://alisawuffles.github.io/) | That applied or product AI roles need a PhD-level curriculum, or that this method caused or guarantees an offer. Her search was for research roles. |
| [Yuan Meng](https://github.com/Yuan-Meng/personal_website_2.0/blob/main/content/posts/mle_interviews_2.0/index.md) (Feb 2026) | Ask for the rounds, prepare per round type, practise in the candidate portal, use mocks to find design gaps | Offers from all but one of 5 to 9 onsites; passed Meta's AI-assisted coding round | That more AI use, a fixed number of practice problems, or a fixed number of hours is enough. She describes ML engineer and research engineer loops, not every AI engineer loop. |

> [!NOTE]
> **The common thread** is preparation driven by feedback and matched to a real loop. Neither account gives a week-by-week calendar, and neither shows which single activity led to an offer. That is why the six-week plan above is labelled as ours.


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>⬅ [Back to the main README](README.md) · [Question bank](questions/README.md) · [Company loops](company/README.md)</sub>

<sub>Part of the **[Landed](https://www.landed.jobs/?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=study-plans)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed; linked write-ups belong to their authors.</sub>
