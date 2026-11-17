# Glean · AI Engineer Interview (2026)

🚀 *"Build the product."* The enterprise-search and agents company still runs a heavy algorithms and low-level-design loop, and its current evals role adds an AI-focused exercise on top.

← Back to [main README](../README.md) · [company index](README.md)

> [!WARNING]
> **Adjacent-role evidence.** The only public full-loop account we could verify is a **new-grad** software engineer loop from **March 2025** (first-person). The AI Engineer evidence is an **official job posting**, which describes requirements and one interview step, not a full loop. Do not read the new-grad rounds as the AI Engineer loop. A full AI Engineer loop is **not publicly confirmed**.

> [!NOTE]
> **The loop at a glance.** *New-grad SWE, one candidate, Mar 2025 (first-person):* a 1-hour algorithm screen → a 30-minute recruiter call → a 1-hour resume deep dive plus math proof → a final of a **2-hour low-level-design assignment** and a 1-hour algorithm round. *AI Engineer, Data and Evals (official posting, live 17 Nov 2026):* the posting says you will complete a **brief AI-focused exercise or discussion** and should expect to discuss it in detail through the process. Round count, durations, timeline and AI-tool policy for the AI Engineer loop are **not publicly confirmed**.
>
> Last checked: 17 Nov 2026.

## The loop

**New-grad software engineer, as one candidate reported it** ([LeetCode post, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience), first-person):

1. **Technical screen (1 hour)** with an engineer: one algorithm question.
2. **Recruiter call (30 min)**: why Glean, why the role.
3. **Technical round (1 hour)**: about 20 minutes of resume and project deep dive, then a math proof with no coding.
4. **Final (two back-to-back interviews)**, about two weeks later:
    - **Assignment round (2 hours)**: a 5-part low-level design problem, coded live as part of the final. This was not a take-home.
    - **Algorithm round (1 hour)**: one medium and one hard problem.

The candidate was rejected the next day, with feedback that the assignment round fell short. The full elapsed time is not stated.

**AI Engineer, Data and Evals** ([official Greenhouse posting](https://job-boards.greenhouse.io/gleanwork/jobs/4746908005), live 17 Nov 2026):

- The posting asks for 5+ years of software engineering with Go and/or Python and backend fundamentals.
- It asks for experience building evalsets and benchmarks for LLM, RAG or agent use cases, and for eval methods such as LLM-as-a-judge, pointwise vs pairwise grading, human labeling and judge calibration.
- It asks for eval and experiment infrastructure at scale (CI/CD, OpenTelemetry-style observability) and SQL-based analysis of quality changes.
- **Interview step (official):** a "brief AI-focused exercise or discussion" that shows how you design and use AI. Its format and length are not stated.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Screen (1h, new grad) | "Given m arrays of size n how do you find the kth largest element" | ✅ Reported · [first-person, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience) |
| Technical (1h, new grad) | Prove why the median is the best value for balancing an array (paraphrased; the candidate did not recall exact wording) | ✅ Reported · [first-person, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience) |
| Assignment (2h, new grad) | Build a table module from scratch in 5 parts: add rows and columns, then joins, then further parts | ✅ Reported · [first-person, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience) |
| Algorithms (1h, new grad) | Sort an array that is sorted except for one out-of-place element | ✅ Reported · [first-person, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience) |
| Algorithms (1h, new grad) | LeetCode 317 (Shortest Distance from All Buildings) | ✅ Reported · [first-person, Mar 2025](https://leetcode.com/discuss/interview-question/6487496/Glean-or-New-Grad-Interview-Full-Loop-Experience) |
| AI exercise (AI Engineer) | A brief AI-focused exercise or discussion; content not described | 📘 Official posting · [Greenhouse, live 17 Nov 2026](https://job-boards.greenhouse.io/gleanwork/jobs/4746908005) |

### 🔮 Representative follow-ups you should expect

*Not reported. These are natural probes for the AI Engineer, Data and Evals role, built from the posting's own requirements.*

- *"Your LLM judge and your human labels disagree on 20% of groundedness cases. How do you find out which one is wrong?"*
- *"You have a pairwise judge and a pointwise judge. Which do you use to gate a release, and why?"*
- *"A retrieval change lifts answer quality but adds 300 ms. How do you report that tradeoff with confidence intervals?"*
- *"Build an eval set for a multi-step enterprise agent when production data has customer permissions on every document."*
- After the kth-largest question: *"Now the arrays are sorted. What changes?"*

## What they're really testing

- **Algorithms still count.** The one verified loop had three algorithm problems and a math proof. AI-era hiring here has not dropped data structures.
- **Speed on a layered design problem.** The 2-hour table module grows part by part; finishing the early parts cleanly and fast is what lets you reach the later ones.
- **Eval engineering, not prompt tweaking** (AI Engineer posting). Judge calibration, slice-level failure analysis and quality vs latency vs cost tradeoffs are named in the requirements.
- **Showing how you design and use AI.** The posting's AI-focused exercise is explicit about this. Expect to defend your choices later in the process.

## How to prep

- **LLM-as-judge, calibration, pass@k, trace-first error analysis** → [content/04](../content/04-evals-and-llm-as-judge.md) and [questions/evals.md](../questions/evals.md).
- **Design an eval platform** → [answers/eval-pipeline.md](../answers/eval-pipeline.md).
- **Enterprise retrieval and permissions** → [content/02](../content/02-rag.md) and [answers/rag-over-10m-docs.md](../answers/rag-over-10m-docs.md).
- **Quality vs latency vs cost** → [content/05](../content/05-llmops-cost-latency.md).
- **The AI-focused exercise** → [content/10](../content/10-ai-assisted-coding-round.md) for how to show your AI workflow while staying in control of it.
- **Algorithms and the math proof** → [questions/coding.md](../questions/coding.md); practice heaps, k-way merge and selection, and be ready to argue why the median minimizes total absolute deviation.
- **Layered low-level design** → build a small in-memory table with insert, select and join in 2 hours, then add one feature at a time against a timer.

## Culture signals

Thin public evidence. The new-grad candidate described engineers and recruiters as friendly and quick to respond (first-person, Mar 2025). The AI Engineer posting is **hybrid, four days a week in office** (official). Anything beyond that is **not publicly confirmed**.

## Cross-links

- 🛠️ Worked design: [Eval pipeline](../answers/eval-pipeline.md)
- 🛠️ Worked design: [RAG over 10M docs](../answers/rag-over-10m-docs.md)
- 🛠️ Worked design: [Semantic search](../answers/semantic-search.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · Last checked: 17 Nov 2026 · No affiliation with Glean. MIT-licensed.</sub>
