# Evals & LLM-as-Judge

> The eval set is the spec. If you can't measure a regression, you can't ship a probabilistic system — you can only hope.

Back to [README](../README.md) · [All questions](../questions/README.md)

---

## What this tests

Whether you've *shipped* an evaluated LLM system or just prompted one. The tell is what you reach for first. Junior answer: "add an LLM judge that rates quality 1–5." Senior answer: "read 50 traces, cluster the failure modes, write the cheapest check that catches each, then gate CI on a regression suite." Interviewers in 2026 probe four things: can you **start an eval program from nothing** (error analysis), **pick the right metric for the reliability bar** (pass^k vs pass@k), **grade the process not just the answer** (trajectory eval), and **make an LLM judge trustworthy** (binary, cross-family, calibrated against humans). Lead with the failure modes you'd measure, then the harness — never the reverse.

The 2026 framing to internalize: **eval is the new system design.** Static benchmarks (GSM8K, HumanEval) measure the *model*. Dynamic trajectory benchmarks (GAIA, SWE-bench, WebArena) measure the *system*. You design the eval harness before you design the agent.

---

## The rings of testing

Not everything needs an LLM. Most regressions are catchable with a string assertion. Use the **cheapest ring that catches the bug** — deterministic checks run on every request; the expensive judge you *sample*.

| Ring | What it catches | Cost | Determinism |
|---|---|---|---|
| **1 · Deterministic assertions** | exact string, regex, exact match, tool-call name/args, numeric tolerance | ~free, milliseconds | fully deterministic |
| **2 · Schema / property checks** | valid JSON, required fields present, types, ranges, no PII leaked, output parses | cheap, no model call | deterministic |
| **3 · LLM-as-judge** | faithfulness, relevance, plan quality, "did it actually answer" — fuzzy criteria a regex can't express | a model call per grade; must be calibrated | probabilistic, biased |
| **4 · Human review** | ground truth, judge calibration labels, novel failure modes, taste | slow, expensive, the scarce resource | the gold standard |

The layering rule: run rings 1–2 on **100% of traffic** because they're nearly free; run ring 3 on a **sample**; spend ring 4 on the calibration set and the disputed cases only. A passing eval whose grader never checked the trajectory can hide both a bug and a security exploit — cheap deterministic checks are where you catch the boring, high-frequency regressions.

> [!TIP]
> Start at the cheapest ring. A shocking fraction of "we need an LLM judge" problems dissolve into a `assert "refund" in output and order_id in tool_args`. Reach for ring 3 only for criteria you genuinely cannot express deterministically.

---

## LLM-as-judge, done right

An LLM judge scales evaluation to criteria no regex can express. But it is a **noisy, biased classifier** — a model you must *also* evaluate. Two upgrades do most of the work.

**1 · Stop asking for a number.** A "rate this 1–5" judge is uncalibrated and too coarse to drive any fix. Replace it with a **binary, criterion-specific** judge: a written pass/fail definition, a required justification, and an explicit **UNKNOWN** escape hatch so it doesn't hallucinate a verdict when the transcript is ambiguous. This decomposes a fuzzy quality score into concrete yes/no questions that map directly onto your error-analysis failure modes.

**2 · Calibrate against humans.** Label a few dozen examples yourself, measure agreement with **Cohen's κ**, and only trust the judge above **κ ≈ 0.6–0.8**. Re-calibrate every time you edit the rubric — a rubric tweak that passes validation can silently shift judgments elsewhere (*rubric-induced preference drift*).

### The bias table

| Bias | Symptom | Mitigation |
|---|---|---|
| **Position** | prefers option A over B by slot, not merit; robustness drops below 0.5 with 3–4 options | randomize candidate order, average over permutations |
| **Verbosity** | longer answer wins regardless of correctness | length-control the comparison (see AlpacaEval 2.0's length-controlled win-rate) |
| **Self-preference / self-enhancement** | rates *its own* outputs higher (self-enhancement error ~16 for one model, ~9 for another) | never let a model judge its own generations |
| **Family / self-enhancement drift** | same model *family* generating and judging inflates and drifts calibration | use a **cross-family** judge (e.g. Claude generates → GPT judges) |
| **Authority / sycophancy** | confident tone or cited "sources" sway the verdict | rubric forces evidence from the transcript, not the tone |

> [!WARNING]
> "The LLM judge is objective." No — it is a biased classifier with a documented taxonomy of failures. An uncalibrated judge is not a measurement; it's a vibe with a JSON schema. Measure Cohen's κ against human labels before you let it gate anything.

> [!WARNING]
> "Use the same model as judge and system." Self-preference and family drift make this the classic tell of someone who hasn't shipped an evaluated system. Judge ≠ generator family, always.

### A judge you can trust (framework-agnostic)

```python
    import json

    JUDGE_PROMPT = (
        "You are grading whether the agent COMPLETED THE REFUND CORRECTLY.\n"
        "PASS only if ALL hold: (1) it verified the order exists, "
        "(2) it checked the refund window, (3) the refund amount matches the order.\n"
        "If you cannot tell from the transcript, answer UNKNOWN -- do not guess.\n"
        'Return JSON: {"verdict": "PASS|FAIL|UNKNOWN", "reason": "..."}\n\n'
        "TRANSCRIPT:\n{transcript}"
    )

    def judge(transcript, call_model):
        # call_model must be a DIFFERENT family than the agent (no self-preference)
        raw = call_model(JUDGE_PROMPT.format(transcript=transcript), temperature=0)
        out = json.loads(raw)
        return out["verdict"], out["reason"]

    def cohen_kappa(judge_labels, human_labels):
        # agreement corrected for chance; want >= 0.6-0.8 before you trust the judge
        n = len(judge_labels)
        cats = set(judge_labels) | set(human_labels)
        po = sum(j == h for j, h in zip(judge_labels, human_labels)) / n
        pe = sum(
            (judge_labels.count(c) / n) * (human_labels.count(c) / n)
            for c in cats
        )
        return (po - pe) / (1 - pe) if pe != 1 else 1.0

    # calibrate on a hand-labelled set BEFORE trusting the judge in CI:
    #   k = cohen_kappa(judge_verdicts, my_verdicts)
    #   if k < 0.6: tighten the criteria, re-label, re-measure.
```

The judge is binary (not 1–5), criterion-specific (three explicit conditions), justified (it must cite a reason), has an UNKNOWN escape hatch, runs at temperature 0, and is only trusted after `cohen_kappa` clears the bar. That's the whole playbook.

---

## pass@k vs pass^k

The metric you pick encodes your reliability bar. Get this wrong and a coin-flip looks production-ready.

| Metric | Meaning | Use for |
|---|---|---|
| **pass@1** | single-shot success | cheap smoke test; hides variance |
| **pass@k** | *≥1 of k* attempts succeed | best-case ceiling; dev-time screening |
| **pass^k** | *ALL k* attempts succeed | production reliability / SLA bar |

The gap is enormous because attempts compound multiplicatively. If one attempt is **90% reliable**:

- `pass@5 = 1 − 0.10^5 = 99.999%` — looks flawless.
- `pass^5 = 0.90^5 = 59%` — the user-facing truth over five turns.

A real user doesn't get five tries; they get one, then another, then another, and they experience the **conjunction**. Ship customer-facing agents on **pass^k**. "A support agent that fails every third request isn't production-ready" — and pass@k will happily tell you it's fine.

> [!WARNING]
> "pass@k is high, so it's reliable." That's not reliability — that's a best-case ceiling. Reliability is pass^k, and it's brutally lower. Reporting pass@k as an SLA is the metric-literacy version of survivorship bias.

**Diagnostic to memorize:** a **0% pass@100** almost never means an incapable model. It means a **broken task** (impossible grader, missing tool, wrong setup) — or, on a security eval, the model **correctly refusing** a malicious request while an inverted grader scores the refusal as failure. Read the transcripts before you blame the model.

---

## Outcome vs trajectory eval

Two ways to grade an agent run. Mature suites use **both**, because each is blind to what the other catches.

| | **Outcome (reference-based)** | **Trajectory (reference-free)** |
|---|---|---|
| **Grades** | the end state vs a golden answer | the path: tool choice, args, order, step efficiency, safety checks |
| **Cheap & unambiguous** | yes | no — needs rubric or deterministic step checks |
| **Catches** | wrong final answer | wrong-but-lucky, right-but-wasteful, skipped-safety-step |
| **Fails when** | agent got lucky, or took 7 tool calls where 3 would do — both marked "success" | over-penalizes a valid *alternative* route the grader didn't anticipate |

Outcome grading blesses an agent that skipped a permission check, made six redundant tool calls, and got lucky. Trajectory grading catches that — Notion went from **3 → 30 fixes/day** by switching to trajectory grading, because the transcript told them *precisely which step broke*. Braintrust's **step-efficiency** metric makes it concrete: 7 tool calls where 3 suffice scores 43%, a regression an output-only eval marks "success."

The nuance that shows experience: also **read the transcripts**, because some "failures" are valid solutions the grader didn't anticipate. So layer deterministic checks (tool/argument correctness) with model-based rubrics (plan quality), and don't over-punish creative-but-correct paths.

> [!TIP]
> Your eval set is the real spec. The prompt is downstream of what the evals tell you — which is why the eval set, built from real failures, is the most valuable artifact you own. More valuable than the prompt.

---

## The agentic 3-metric stack

The 2026 standard (NVIDIA's agent-eval playbook) for grading an agent is three orthogonal categories. Report all three; a single number is not an eval.

| Category | Measures | Concrete metrics |
|---|---|---|
| **Outcome** | did it accomplish the task? | Task Success Rate, pass^k |
| **Trajectory** | was the path efficient and correct? | steps-per-success, tokens-per-success, tool-call accuracy, step efficiency |
| **Tool / reasoning** | were the mechanics sound? | schema compliance, argument correctness, reasoning soundness |

Outcome tells you *if* it worked. Trajectory tells you *how expensively*. Tool/reasoning tells you *why* it worked or broke. Cost-per-resolved-task and steps-per-task are the sensitive leading indicators — a "small" prompt tweak that adds one tool call shows up in tokens-per-task long before it dents answer quality.

---

## Error analysis: read the traces first

The hardest part of evals is not the harness — it's knowing *what to measure*. Metrics come **after** you've seen the data. Hamel Husain's widely-taught method:

```
    ERROR-ANALYSIS -> EVAL pipeline
    1. collect 50-100 real (or realistic) traces
    2. OPEN-CODE: one free-text note per trace -- what went wrong, specifically
    3. AXIAL-CODE: cluster the notes into failure modes
         "skipped the auth check"        42%
         "wrong tool for date math"      18%
         "looped on a 404 (as if 503)"   15%
         "hallucinated an order id"       12%
    4. build a small, TARGETED eval per dominant mode (start with the 42%)
    5. fixes lower that mode's frequency -> re-read traces -> new modes surface
    measure what actually breaks; don't invent metrics that never fail.
```

This is the antidote to the two most common eval mistakes: **inventing metrics in the abstract** (which measure things that never fail) and **adding a generic "helpfulness 1–5" judge** (too coarse to drive any decision). When asked "how would you start evaluating an agent with no evals?", the senior answer is *read 50 traces, label the failure modes, build a small eval per dominant mode* — not "write a quality judge."

---

## Eval-driven development

Evals aren't a QA afterthought; they are the **development loop**. Anthropic's framing splits them:

- **Capability evals** ask "what can it do?" and should **start low** — a mountain to climb (Claude went 40% → 80%+ on SWE-bench Verified; Opus 4.5 went 42% → 95% on CORE-Bench after a bug fix). Saturation is a signal, not success.
- **Regression evals** ask "does it still do everything it used to?" and should sit near **100%**, gating CI. Any change that breaks existing behaviour fails the gate.

The lifecycle: ship a small capability eval (20–50 tasks from *real failures*); once it saturates, **graduate it into the regression suite**. Then **gate CI**: every prompt edit, tool change, model upgrade, and provider checkpoint runs the regression suite, and a drop below the bar blocks the merge.

The discipline that makes this real:
- **Dataset = spec.** Version it in git. Golden-set **provenance** matters — every case should trace back to a real trace or a deliberate design decision.
- **Pin model + prompt versions**, so a silent provider checkpoint that degrades quality fails the gate *before* users notice.
- **Online → offline flywheel.** Online evals score live production traces; the failures they surface get **promoted into the offline regression set**. Production reveals a new failure mode → capture the trace → it becomes a permanent test → the next change that reintroduces it fails CI.

> [!WARNING]
> "We'll add evals after launch." Without evals you can't tell "it got better" from "it quietly got worse at something it used to do." A probabilistic, multi-step system changes behaviour when you touch a prompt, a tool, or when the provider silently updates the model. Evals-after-launch means users are your regression suite.

> [!TIP]
> Graduate capability evals into regression evals. The capability eval is the mountain you're climbing; the regression eval is the guardrail that stops you sliding back — and a silent provider model update is exactly the slide it catches.

---

## Eval is the new system design

Static benchmarks measure the model; dynamic trajectory benchmarks measure the *system*. When you build an agent in 2026, the eval harness is a design artifact you build **first**, not a report you generate last. GSM8K tells you the model can do arithmetic; **GAIA / SWE-bench / WebArena** tell you whether your *system* — with its tools, prompts, retries, and guardrails — actually completes real multi-step tasks. Design the harness, then the agent will have somewhere to prove itself.

---

## Interview angles

**Q: How would you evaluate a RAG system?**
Three axes, each a separate metric. **Faithfulness** — is the answer grounded in the retrieved context, or did the model invent it? (An LLM judge checks each claim against the context.) **Answer relevance** — does the response actually address the question? **Context precision & recall** — of what you retrieved, how much was relevant (precision), and of what was relevant, how much did you retrieve (recall)? Faithfulness catches hallucination; context recall catches retrieval gaps; context precision catches noisy retrieval that dilutes the prompt. Tools like Ragas operationalize exactly these. Report them separately — a system can be faithful but irrelevant, or relevant but built on missing context.

**Q: RAG is hallucinating despite retrieving the right context — how do you fix it?**
The context is right but the model isn't *using* it, so the fix is downstream of retrieval. Check faithfulness per-claim first to confirm the diagnosis. Then: tighten the prompt to instruct "answer *only* from the provided context; if it's not there, say so"; add a grounding/citation requirement so each claim maps to a passage; reduce context that dilutes the signal (high recall, low precision drowns the right chunk); and consider a smaller, more instruction-following generation model for the synthesis step. If the model still drifts, add a faithfulness judge as a guardrail that blocks ungrounded answers. The key insight for the interviewer: right context + wrong answer is a *generation* problem, not a retrieval one.

**Q: How do you evaluate a fine-tuned model?**
Never on the fine-tuning objective alone. Hold out a test set the model never saw. Measure (1) task performance on your real distribution, (2) a **regression check against the base model** — fine-tuning routinely degrades general capabilities you didn't test for (catastrophic forgetting), so keep a broad capability suite, and (3) behaviour on adversarial / out-of-distribution inputs. Compare head-to-head with the base model using a cross-family LLM judge or human labels, not a self-judge. And watch for the classic trap: a fine-tune that memorized the training distribution and scores beautifully on cases that look like training data but collapses on the long tail.

**Q: Explain LLM-as-a-judge biases and mitigations.**
Position (prefers a slot → randomize order and average), verbosity (longer wins → length-control), self-preference and family drift (rates its own family higher → cross-family judge, never self-judge), authority/sycophancy (confident tone sways it → rubric forces transcript evidence). The meta-mitigations: make it **binary and criterion-specific** with justifications and an UNKNOWN option, and **calibrate against human labels** with Cohen's κ ≥ 0.6–0.8, re-calibrating after every rubric edit. A judge is a model you must also eval.

**Q: Design an eval suite for a coding agent.**
Three metric categories. **Outcome:** task success rate (tests pass, PR merges clean) — reported as pass^k for reliability, not pass@k. **Trajectory:** steps and tokens per success, tool-call accuracy, step efficiency (did it edit 3 files or thrash through 15?), and whether it ran the required checks (tests, lint) before declaring done. **Tool/reasoning:** schema compliance on tool calls, argument correctness, reasoning soundness. Seed the suite from error analysis on 50–100 real agent traces; graduate saturated capability tasks into a regression gate on SWE-bench-style tasks; pin model + prompt versions and block merges on regression.

**Q: Outcome vs trajectory eval — when does each fail?**
Outcome grading fails on wrong-but-lucky (right answer, unsafe or nonsensical path) and right-but-wasteful (7 tool calls where 3 would do) — both scored "success." Trajectory grading fails by over-penalizing a *valid alternative route* the grader didn't anticipate — it can mark a creative-but-correct solution wrong. So you run both: deterministic step checks plus a model-based plan-quality rubric for the trajectory, reference-based final-state checks for the outcome, and you *read transcripts* to catch graders that punished valid paths. Neither alone is sufficient.

**Q: Build a regression test set for a feature that ships weekly.**
Start from real failures: mine last week's production traces, error-analyze, and add the dominant failure modes as cases. Version the dataset in git alongside the code, with provenance on every case. Pin model and prompt versions. Gate the weekly release on the regression suite in CI — sitting near 100%, so any change that breaks prior behaviour fails before merge. Run cheap deterministic checks on every case and a sampled cross-family judge for the fuzzy ones. Each week, promote new production failures into the set: the suite is a maintained dataset that rots if you don't feed it.

**Q: When do you trust GPT-4-as-judge vs a human?**
Trust the judge for high-volume, well-decomposed, binary criterion-specific grades that you've **calibrated** — Cohen's κ ≥ 0.6–0.8 against human labels on a held-out set, cross-family (don't judge GPT outputs with GPT), position-randomized. Fall back to humans for the calibration labels themselves, for novel or ambiguous failure modes the judge hasn't been validated on, for anything where the judge shows self-preference, and for taste/subjective calls that resist a written rubric. The judge scales *validated* criteria; humans define ground truth and catch what the judge can't.

**Q: How do you evaluate a reasoning model differently?**
Its value is in the *process*, so don't grade only the final answer — grade reasoning soundness and whether the "thinking" actually reached the conclusion (models can reason correctly then answer wrong, or reason garbage and get lucky). Watch cost and latency: thinking tokens reshape trajectories and blow up token-per-task, so trajectory metrics matter more. Test on harder, multi-step tasks where reasoning pays off — a reasoning model on trivial tasks just burns tokens. And use pass^k: reasoning models can be higher-variance across attempts, and the conjunction is what the user feels.

---

## 📚 Resources

- 📄 [MT-Bench / Chatbot Arena (Zheng et al., 2023)](https://arxiv.org/abs/2306.05685) — the canonical LLM-as-judge eval and bias taxonomy (position, verbosity, self-enhancement). Start here.
- 📄 [A Survey on LLM-as-a-Judge (2024)](https://arxiv.org/abs/2411.15594) — biases and mitigations, heavily cited (1600+). The reference for the bias table.
- 📄 [G-Eval (2023)](https://arxiv.org/abs/2303.16634) — chain-of-thought NLG scoring with a judge; the "make the judge reason before it grades" technique.
- 💻 [deepeval](https://github.com/confident-ai/deepeval) — Apache-2.0, ~16.6k★ — pytest-for-LLMs, 50+ metrics including faithfulness and G-Eval. The fastest way to a CI-gated eval suite.
- 🛠️ [Braintrust — eval-driven development](https://www.braintrust.dev/articles/eval-driven-development) — the methodology (dataset = spec, gate on regressions) plus a platform with trajectory/step-efficiency grading.
- 📰 [NVIDIA — AI Agent Evaluation playbook](https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation/) — the 2026 3-metric agentic stack: outcome · trajectory · tool/reasoning.
- 📘 [OpenAI — Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices) — objective → dataset → grader → run + analyze, the practical loop.
- 💻 [EleutherAI lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) — MIT — the standard runner for static benchmarks (the ones that measure the model, not the system).

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [Evals questions](../questions/evals.md)
