# Evals — Agent Evaluation, LLM-as-Judge, Injection & Security Evals

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/evals.md`](../content/04-evals-and-llm-as-judge.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **19 questions** (10 self-test checkpoints + 9 reported). *"Eval is the new system design"* is the loudest 2026 hiring signal — evals are first-class deployment artifacts with gates, not offline reports. Theses: **error analysis before metrics**, **trajectory eval sees what output-only eval hides**, **pass^k not pass@k for an SLA**, **an LLM judge is itself a model output that must be calibrated against humans**, and **security tasks invert the grader — a correct refusal is a pass**.

---

### How to use this file

Answer, then open the fold. Judge-calibration questions also appear in [rag.md Q29](./rag.md) (RAG judge as deploy gate); the lethal-trifecta and injection-containment questions are graded as **security evals**, so they live here.

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (calibration, security, inverted graders)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — Agent & trajectory evals `[agents-evals-llmops] ag4`

### Q1. Final answers look fine, but your agent's cost is creeping up and some runs take many extra tool calls. Which eval surfaces this?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag4` · 🔮 Representative

- Output-only eval on the final answer
- ✅ Trajectory eval with step-efficiency scoring
- A bigger model

<details><summary>Show answer & why each option</summary>

- **Output-only eval** — It marks both the lean and the bloated run "success" — it can't see the wasted steps.
- ✅ **Trajectory eval + step-efficiency** — Grading the transcript exposes redundant/extra tool calls (e.g. 7 where 3 suffice) that output-only evals hide.
- **Bigger model** — Doesn't diagnose anything; the issue is inefficiency you must first measure.
</details>

### Q2. You're reporting reliability for a customer-facing support agent. Which metric should the SLA use?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag4` · 🔮 Representative

- pass@k — it shows the agent can succeed
- ✅ pass^k — all k attempts must succeed
- pass@1 — simplest to compute

<details><summary>Show answer & why each option</summary>

- **pass@k** — Rewards "at least one of k worked," which overstates reliability for a user who gets one shot.
- ✅ **pass^k** — Production reliability means consistent success; pass^k captures "fails every third request isn't good enough."
- **pass@1** — A single shot hides variance; a smoke test, not an SLA metric.
</details>

### Q3. You inherit an agent with no evals and a vague "make it better" mandate. What's the right first move?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag4` · 🔮 Representative

- ✅ Read 50–100 real traces, open-code what went wrong in each, cluster into failure modes, and build a small targeted eval for the most frequent mode
- Add an LLM-as-judge that rates overall quality 1–5 on every response
- Upgrade to the newest model and re-test by hand

<details><summary>Show answer & why each option</summary>

- ✅ **Error analysis first** — Tells you what actually breaks and how often, so you measure (and fix) the dominant failure modes — not metrics you invented in the abstract.
- **Coarse 1–5 judge** — A quality score with no error analysis or calibration can't drive a specific fix and hides process failures — the classic anti-pattern.
- **Upgrade + hand-test** — You can't tell "better" from "differently broken" without an eval set.
</details>

### Q4. A security eval for prompt injection reports 0% pass@100 on a task where the agent should NOT comply with an injected instruction. Most likely explanation?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag4` · 🔮 Representative

- The agent is completely incapable and needs a bigger model
- The provider silently downgraded the model
- ✅ The grader is scoring a correct refusal as a failure — the agent is doing the right thing and the eval is inverted/broken

<details><summary>Show answer & why each option</summary>

- **Incapable → bigger model** — A 0% pass@100 almost never means incapability; it points at the task or grader.
- **Provider downgraded** — A provider change shifts many evals gradually, not a clean 0% on one refusal task.
- ✅ **Inverted grader** — On a security task the desired behaviour is non-compliance; a grader that marks refusal as "task incomplete" turns a working control into a fake 0%. Read the transcripts and fix the grader.
</details>

### Q5. Your LLM judge disagrees with your spot-checks often, and you used the same model family to generate and to judge. Best fix?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag4` · 🔮 Representative

- Switch the judge to a 1–5 quality scale for more granularity
- ✅ Use a different model family to judge, rewrite the rubric as binary criterion-specific pass/fail with justifications, randomise order, and calibrate against human labels (Cohen's κ)
- Raise the judge's temperature so it considers more options

<details><summary>Show answer & why each option</summary>

- **1–5 scale** — More granularity on an uncalibrated, biased judge adds noise; the problem is calibration and self-preference.
- ✅ **Cross-family + binary criteria + randomise + calibrate** — Cross-family removes self-preference bias; binary criteria + justifications + order randomisation + measured human agreement are the documented playbook.
- **Raise temperature** — Makes the judge less consistent; judges should be near-deterministic and calibrated.
</details>

---

## Section B — Security & injection evals `[agents-evals-llmops] ag6`

### Q6. Which combination makes data exfiltration from an agent trivially exploitable (the "lethal trifecta")?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag6` · 🔮 Representative

- A large context window + many tools + high temperature
- ✅ Access to private data + exposure to untrusted content + an external communication channel — in one agent
- Using MCP instead of in-process tools

<details><summary>Show answer & why each option</summary>

- **Context + tools + temperature** — Those are capability/latency knobs, not the exfiltration condition.
- ✅ **Private data + untrusted content + egress** — With all three, an attacker can inject via untrusted content, have the agent read private data, and send it out. Break the trifecta to fix it.
- **MCP** — MCP adds a trust boundary to manage, but the trifecta is about data + untrusted input + egress, regardless of transport.
</details>

### Q7. Your agent reads customer-uploaded documents and can send emails. What's the most durable defense against indirect prompt injection?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag6` · 🔮 Representative

- A strong system prompt instructing it to ignore malicious instructions in documents
- ✅ Break the trifecta and gate writes: separate the data/egress planes, least-privilege tools, human approval on sends
- Lower the temperature so the model is less suggestible

<details><summary>Show answer & why each option</summary>

- **Strong system prompt** — Prompt-level defenses cut ASR but never to zero (73.2% baseline); a determined injection gets through.
- ✅ **Break the trifecta + gate writes** — Architectural separation + a write gate removes the exploit path itself, rather than hoping the model resists every injection.
- **Lower temperature** — Doesn't govern instruction-following from retrieved content; irrelevant to injection.
</details>

### Q8. An interviewer asks why "just tell the model to ignore malicious instructions in documents" can't work even in principle. Best answer?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag6` · 🔮 Representative

- ✅ An LLM has no boundary between instructions and data — system prompt, user message, and retrieved content are one token stream — so your "ignore injections" rule is just another instruction in the same channel the attacker uses
- It can work if the instruction is placed last in the system prompt so it has priority
- It works only on weaker models; frontier models are immune

<details><summary>Show answer & why each option</summary>

- ✅ **No instruction/data boundary** — Prompt injection is unsolved at the model layer precisely because there's no separation; a meta-instruction can't create a boundary the architecture doesn't have.
- **Place it last** — Position doesn't create a boundary; the model still can't distinguish your meta-instruction from an injected one.
- **Only weaker models** — The 73.2% baseline ASR spans GPT-4, Claude 2.1, and Llama 2 — it's architectural.
</details>

### Q9. In GeminiJack, what actually performs the data exfiltration once the injected instruction is followed?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag6` · 🔮 Representative

- The user clicking a malicious link in the summary
- A separate malware payload downloaded by the agent
- ✅ The act of rendering an attacker-controlled `<img>` tag whose URL carries the stolen data in its query string — a GET to the attacker's server

<details><summary>Show answer & why each option</summary>

- **Clicking a link** — GeminiJack is zero-click — no link is clicked; that's what makes it notable.
- **Malware payload** — No malware; the exploit uses only the agent's own legitimate rendering capability.
- ✅ **Rendering an attacker `<img>`** — The injected instruction encodes private data into an image URL; rendering fires the request, exfiltrating with zero clicks. Egress is the trifecta leg to cut.
</details>

### Q10. You must let an agent read untrusted web pages AND call tools that can act. Which architecture best contains injection?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag6` · 🔮 Representative

- Raise the input classifier's threshold and log everything
- ✅ A CaMeL/dual-LLM split: a privileged LLM plans from the trusted request and a quarantined LLM reads untrusted content but cannot choose actions — untrusted data fills values, never redirects control flow
- Give the agent every tool but lower the temperature

<details><summary>Show answer & why each option</summary>

- **Classifier + logging** — A classifier reduces but never removes injection (8.7% residual); logging is detection, not containment.
- ✅ **CaMeL / dual-LLM split** — The durable fix denies untrusted data the ability to select actions; separating the planes means an injection can't add or redirect tool calls.
- **Every tool + low temperature** — Temperature doesn't govern instruction-following, and a broad tool surface widens the blast radius.
</details>

---

## Section C — Reported eval & security interview questions

> Observed in real 2026 loops. Sources: [amitshekhariitbhu/ai-engineering-interview-questions](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), NVIDIA *Mastering Agentic Techniques: AI Agent Evaluation* (https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation/), OWASP LLM Top 10 (https://genai.owasp.org/llm-top-10/), EchoLeak CVE-2025-32711. Rehearse an answer; the checkpoints above and in [rag.md](./rag.md) are the drill.

### Q11. How do you build an evaluation harness for an agent? Design an eval suite for a coding agent (three metric categories).
> **Difficulty:** 🔴 Hard · ✅ Reported ([NVIDIA agent-eval playbook](https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation/), [Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Three-metric stack — **outcome** (Task Success Rate / end-state correctness), **trajectory** (steps & tokens per success, Tool-Call Accuracy — Q1), **tool/reasoning** (schema compliance, reasoning soundness). Static benchmarks (HumanEval/GSM8K) measure the *model*; dynamic trajectory benchmarks (SWE-bench, GAIA, WebArena) measure the *system*. Build the golden set from real traces (Q3); gate on pass^k (Q2).

### Q12. Outcome vs trajectory eval — when does each fail?
> **Difficulty:** 🟡 Medium · ✅ Reported ([NVIDIA](https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation/))
> **Senior frame:** Outcome-only misses inefficiency and lucky-right-for-wrong-reasons (Q1). Trajectory-only can penalise a valid alternative path. Use both: outcome as the gate, trajectory for cost/efficiency and debugging. End-state-only success is right when only the final DB/file state matters and the path is free.

### Q13. Explain LLM-as-a-Judge — biases and mitigations. When do you trust GPT-4-as-judge vs a human?
> **Difficulty:** 🔴 Hard · ✅ Reported ([Zheng et al. MT-Bench](https://arxiv.org/abs/2306.05685), [Survey on LLM-as-a-Judge](https://arxiv.org/abs/2411.15594))
> **Senior frame:** Biases — position, verbosity/length, self-preference (same-family). Mitigations — cross-family judge, binary criterion-specific rubrics with justifications, order randomisation, and calibration against human labels (Cohen's κ) — Q5 and [rag.md Q29](./rag.md). Trust the judge only after it clears an agreement bar on a ~50-example human golden set; keep humans for high-stakes and periodic re-calibration.

### Q14. How do you build a regression test set for a weekly-shipping LLM feature?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Braintrust EDD](https://www.braintrust.dev/articles/eval-driven-development), [DeepEval](https://deepeval.com/blog/eval-driven-development))
> **Senior frame:** Harvest failures from production/shadow (Q3), decontaminate, version the golden set, and run it as a CI gate on every prompt/model change — pinned model+prompt versions so a silent provider change is caught (see [llmops.md](./llmops.md)). Rings: deterministic checks → judge → human spot-check.

### Q15. Why does AlpacaEval 2.0 use length-controlled win-rate?
> **Difficulty:** 🔴 Hard · ✅ Reported ([research/04 eval set](https://arxiv.org/abs/2306.05685))
> **Senior frame:** LLM judges have a verbosity bias — longer answers win more, independent of quality. Length-controlled win-rate regresses out response length so the score reflects quality, not word count. It's a concrete instance of the judge-bias problem (Q13).

### Q16. What is hallucination and how do you mitigate it?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Fluent claims not entailed by evidence. Mitigations — ground in retrieved context and require citations (measure faithfulness, [rag.md Q26](./rag.md)); constrain outputs to a schema; allow "not present" paths (`fundamentals` Q15); validate cited spans exist; and gate with an eval that scores groundedness and hallucination rate against targets.

### Q17. Name three prompt-injection classes and a defense for each.
> **Difficulty:** 🔴 Hard · ✅ Reported ([OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/))
> **Senior frame:** (1) **Direct** (user overrides system) → delimit + treat user text as data (`fundamentals` Q8); (2) **Indirect** (injection via retrieved docs/tools) → validate the second input channel + break the trifecta (Q6–Q7); (3) **Data exfiltration / egress** (e.g. rendered-image GET, Q9) → cut the egress leg, gate writes, dual-LLM control-flow separation (Q10). Prompt rules alone never reach 0% (Q8).

### Q18. Why is indirect injection harder than direct? Cite a real incident.
> **Difficulty:** 🔴 Hard · ✅ Reported ([EchoLeak CVE-2025-32711](https://arxiv.org/abs/2509.10540))
> **Senior frame:** Direct injection is in the user turn you can wrap and distrust; indirect rides in through *content you retrieved or a tool returned*, which most guardrails never inspect ([rag.md Q30](./rag.md)). EchoLeak (CVE-2025-32711, CVSS 9.3) was the first real-world zero-click indirect injection against M365 Copilot — exfiltration with no user action. Defense is architectural, not a prompt.

### Q19. How do you evaluate a reasoning model differently from an instruction-tuned model?
> **Difficulty:** 🟡 Medium · ✅ Reported ([research/01 §5.3](https://openai.com/index/learning-to-reason-with-llms/))
> **Senior frame:** Reasoning models bill hidden thinking tokens and reshape trajectories, so measure cost/latency including thinking, not just the answer; grade the *final* answer plus (where visible) reasoning soundness; use harder, contamination-controlled benchmarks (AIME/GPQA-style) since easy sets saturate; and watch over-thinking on shallow tasks (route those to a standard model — [llmops.md](./llmops.md)).

---

<sub>⬅ [Back to the index](./README.md) · Related: [Agents →](./agents.md) · [RAG →](./rag.md) · [LLMOps →](./llmops.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
