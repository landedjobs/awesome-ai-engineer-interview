# LLMOps — Reliable Clients, Cost & Latency, Testing & Observability

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/llmops.md`](../content/05-llmops-cost-latency.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **29 questions** (20 self-test checkpoints + 9 reported). The operational half of the loop: building a client that survives 429-storms and provider drift, cutting cost/latency without wrecking quality, testing probabilistic output with a ring model, and getting decision-level observability. Theses: **jitter + limiter + breaker beats more retries**, **the stable prefix is your biggest cost lever**, **ship deterministic checks first**, and **cost-per-resolved-task is the metric tied to value**.

---

### How to use this file

Answer, then open the fold. Prompt-versioning and CI eval gates cross-reference [evals.md](./evals.md); the observability questions (Section D) are the LLMOps side of agent evals.

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (scale, drift, cost regressions)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — Reliable clients: retries, timeouts, fallbacks `[llm-features] llm4`

### Q1. Right after a launch, the provider starts returning 429s; your clients retry immediately, and the problem gets worse, not better. What's the fix?

> **Difficulty:** 🟡 Medium · `[llm-features] llm4` · 🔮 Representative

- Increase the retry count so requests eventually succeed
- ✅ Add capped backoff + jitter, a client-side rate limiter that refuses over-budget calls, and a circuit breaker
- Switch every request to a second provider permanently

<details><summary>Show answer & why each option</summary>

- **Increase retries** — More retries amplify the storm and double the bill; the issue is synchronised, immediate retries.
- ✅ **Backoff + jitter + limiter + breaker** — Jitter de-synchronises retries; the client-side limiter and breaker stop hammering a recovering provider — the textbook anti-storm stack.
- **Permanent failover** — Overkill and risky — you move the storm and inherit quality drift; fallback is for exhaustion, not the first 429.
</details>

### Q2. Which of these should your client NOT retry?

> **Difficulty:** 🟢 Easy · `[llm-features] llm4` · 🔮 Representative

- A 503 Service Unavailable
- ✅ A 400 caused by the prompt exceeding the context window
- A 429 with a `Retry-After` header

<details><summary>Show answer & why each option</summary>

- **503** — Transient — retry with backoff.
- ✅ **400 context-window overflow** — Non-retryable: the same oversized prompt will fail identically every time. Fix the input (truncate/RAG), don't retry.
- **429 with `Retry-After`** — Retryable — wait for the interval, then retry.
</details>

### Q3. Your client streams responses. A call dies after streaming ~800 of an expected ~1,000 tokens. Your generic retry decorator re-fires the whole request. What's the problem and the better design?

> **Difficulty:** 🔴 Hard · `[llm-features] llm4` · 🔮 Representative

- ✅ Treat a mid-stream failure as a distinct case — fail it cleanly (or resume if supported), and don't count the partial output as a free re-run
- No problem — retrying is always correct for transient failures
- Lower the temperature so the stream doesn't fail

<details><summary>Show answer & why each option</summary>

- ✅ **Handle mid-stream failure distinctly** — Blind retry of a partially-streamed call doubles cost and can duplicate output; partial progress already cost money, so you classify and handle it.
- **Always retry** — For a half-streamed, per-call-expensive request it isn't: you re-pay for the full generation and the user may see duplicated text.
- **Lower temperature** — Has nothing to do with a dropped connection mid-stream.
</details>

### Q4. An interactive feature has a fat tail: p50 TTFT is 600ms but p99 is ~8s, and users hate the stragglers. Cost has some headroom. Strongest lever?

> **Difficulty:** 🔴 Hard · `[llm-features] llm4` · 🔮 Representative

- Raise `max_retries` so slow calls get retried
- ✅ Hedge the slow tail: if no first token by ~p95, fire a second request and take the winner — bounding the hedge rate so extra cost stays small
- Increase the read timeout so slow calls have more time

<details><summary>Show answer & why each option</summary>

- **Raise `max_retries`** — A slow-but-succeeding call isn't an error; retrying doesn't fire until it fails, so it does nothing for the tail.
- ✅ **Hedge the tail** — Hedged requests (The Tail at Scale) collapse the long tail toward p95 by racing a duplicate only on the slow fraction; cap cost by hedging the tail, not every call — and skip it for non-idempotent tool calls.
- **Increase read timeout** — Lets stragglers finish but does nothing to reduce the tail latency users complain about.
</details>

### Q5. Your primary model is down so the breaker fails over to a cheaper second-provider model. Availability recovers, but a week later you learn answer quality quietly dropped during the outage. What was missing?

> **Difficulty:** 🔴 Hard · `[llm-features] llm4` · 🔮 Representative

- A longer circuit-breaker cooldown
- A higher retry count on the primary
- ✅ A fallback path with its own light eval and a prompt validated for that model, plus a quality metric alert — not just an availability check

<details><summary>Show answer & why each option</summary>

- **Longer cooldown** — Affects when you probe the primary, not whether the fallback's answers were any good.
- **Higher retry count** — More retries on a down primary just delay the inevitable fallback.
- ✅ **Fallback with its own eval + quality alert** — Failover keeps you up but can silently serve worse answers (quality drift, prompt non-portability); a real fallback needs its own eval and a quality signal.
</details>

---

## Section B — Cost & latency engineering `[llm-features] llm5`

### Q6. Your prompt-cache hit-rate is near 0% despite a large, seemingly stable system prompt. Most likely cause?

> **Difficulty:** 🟡 Medium · `[llm-features] llm5` · 🔮 Representative

- The provider disabled caching for your account
- ✅ A dynamic value (timestamp, user id, request id) sits inside the cached prefix, so the prefix differs every request
- Your prompt is over 1,000 tokens, which is too long to cache

<details><summary>Show answer & why each option</summary>

- **Provider disabled it** — Caching is automatic/available; the usual culprit is your own prefix changing every call.
- ✅ **Dynamic value in the prefix** — Caching matches a stable prefix; any per-call token before the breakpoint invalidates the match. Move dynamic content into the message body.
- **Too long to cache** — Backwards — long stable prefixes are exactly what caching rewards; ≥~1k tokens is the *minimum* to qualify on OpenAI.
</details>

### Q7. Which workload is semantic response caching most likely to help?

> **Difficulty:** 🟡 Medium · `[llm-features] llm5` · 🔮 Representative

- An open-ended creative-writing assistant
- ✅ A support/FAQ bot over a stable knowledge base with repetitive questions
- A tool-calling agent that mutates account state

<details><summary>Show answer & why each option</summary>

- **Creative writing** — Long-tail, low-repeat prompts rarely clear the similarity threshold — hit rates stay 5–10%.
- ✅ **Support/FAQ bot** — High-duplicate, non-stateful traffic is where semantic caching earns its 40–80% hit rate at high accuracy.
- **State-mutating agent** — State-mutating calls must not be served from cache — a stale hit causes real damage.
</details>

### Q8. A feature sends a 1,200-token prompt (1,000 of it a stable system prefix) and returns ~250 tokens, on a frontier model, 200k req/day. Where is the biggest cost win?

> **Difficulty:** 🔴 Hard · `[llm-features] llm5` · 🔮 Representative

- Trim the 200 dynamic input tokens down to 100
- ✅ Enable prompt caching on the 1,000-token stable prefix (0.1× input) and route the easy majority to a cheaper model
- Raise `max_tokens` so answers finish in one call

<details><summary>Show answer & why each option</summary>

- **Trim dynamic input** — Saves a sliver of the cheaper input side and nothing on output — the smallest lever, and it risks the cache prefix.
- ✅ **Cache the prefix + route easy traffic** — The stable prefix is ~83% of input and cacheable at ~90% off; routing the easy traffic off the frontier model cuts price_in and price_out — together far larger than trimming.
- **Raise `max_tokens`** — Output is already the pricier side; raising the ceiling increases cost and latency.
</details>

### Q9. A high-volume extraction endpoint (shallow task, strict structured output) is on a reasoning model "for accuracy," and both cost and p99 latency are too high. Best move?

> **Difficulty:** 🔴 Hard · `[llm-features] llm5` · 🔮 Representative

- ✅ Move the shallow, high-volume majority to a standard (non-reasoning) model and reserve the reasoning model for the genuinely hard tail, routing per request
- Turn on streaming so the reasoning model feels faster
- Raise the temperature to speed up generation

<details><summary>Show answer & why each option</summary>

- ✅ **Route: standard for the majority, reasoning for the tail** — Reasoning models bill hidden thinking tokens and add latency — wasteful on a shallow, high-volume task; routing cuts both cost and tail latency.
- **Streaming** — Improves perceived latency only; you still pay for and wait on the hidden thinking tokens.
- **Raise temperature** — Affects sampling, not throughput, cost, or whether the model emits hidden reasoning tokens.
</details>

### Q10. You add a cheap→expensive cascade and savings look great at launch, but two months later spend is back up though traffic is flat. Most likely cause and the right instrumentation?

> **Difficulty:** 🔴 Hard · `[llm-features] llm5` · 🔮 Representative

- The cheap model got more expensive — switch providers
- ✅ The escalation rate crept up (harder traffic or scorer drift), so more calls hit the expensive model — track escalation rate and per-request cost as trend lines, then re-tune the threshold/router
- Prompt caching stopped working — disable it

<details><summary>Show answer & why each option</summary>

- **Cheap model got pricier** — A flat-traffic spend rise with an unchanged price list points to behaviour, not price.
- ✅ **Escalation-rate drift** — Cascades silently degrade when the escalation rate drifts up; you're now paying two bills more often. Measure escalation rate + cost over time and re-tune.
- **Caching stopped** — A broken cache raises input cost, but disabling caching makes spend worse; and it doesn't explain a cascade whose escalation behaviour changed.
</details>

---

## Section C — Testing probabilistic output `[llm-features] llm6`

### Q11. You're writing the first tests for the service's probabilistic output. Which ring do you ship first?

> **Difficulty:** 🟢 Easy · `[llm-features] llm6` · 🔮 Representative

- An LLM-as-judge rubric scoring overall quality
- ✅ Deterministic checks: schema validation, JSON validity, exact-match on closed fields
- Byte-for-byte snapshot comparison against a saved output

<details><summary>Show answer & why each option</summary>

- **LLM-as-judge** — Valuable, but the most expensive and least stable ring — add it last, calibrated against humans and sampled.
- ✅ **Deterministic checks** — The cheapest, most reliable ring — it catches the highest-frequency failures for free and is the foundation the other rings build on.
- **Byte-for-byte snapshot** — Probabilistic output makes exact snapshots flaky — a correct rephrasing fails the test.
</details>

### Q12. Two weeks after launch, users report the service "got worse," though you changed nothing. What would have protected you — and now tells you what happened?

> **Difficulty:** 🟡 Medium · `[llm-features] llm6` · 🔮 Representative

- Higher temperature for more diverse outputs
- ✅ A pinned model/prompt version plus a CI eval gate on a golden set that runs on every change
- A bigger context window

<details><summary>Show answer & why each option</summary>

- **Higher temperature** — Irrelevant to a provider-side change, and it adds variance, not protection.
- ✅ **Pinned version + CI eval gate** — Providers silently update models (Anthropic's own postmortem); pinning + a golden-set eval both prevents and detects the regression.
- **Bigger window** — Unrelated to a behavioural regression from a model update.
</details>

### Q13. A user uploads a 400-page contract — well over the context window — and asks for a one-paragraph summary. Best approach for this service?

> **Difficulty:** 🟡 Medium · `[llm-features] llm6` · 🔮 Representative

- Switch to the largest-context model and stuff the whole document in
- ✅ Map-reduce: summarize chunks in parallel, fold the partial summaries until they fit, then a final pass
- Truncate the contract to the first N pages that fit

<details><summary>Show answer & why each option</summary>

- **Stuff it in** — Even when it fits, you pay Lost-in-the-Middle, context rot, and a big cost/latency bill — and many documents still won't fit.
- ✅ **Map-reduce** — A summary is a holistic task, so map-reduce scales past the window at bounded cost; for targeted extraction you'd instead chunk+retrieve.
- **Truncate** — Silently drops most of the document — the summary misses everything after the cut.
</details>

### Q14. Strict structured output is on, so JSON is always valid — yet the extracted `owner` field is wrong on ~30% of documents. Where is the problem and the fix?

> **Difficulty:** 🔴 Hard · `[llm-features] llm6` · 🔮 Representative

- ✅ It's an accuracy/grounding problem, not a structure one — ground the field in a cited source span, add a validator that the span exists, and order a reasoning field before `owner`
- Constrained decoding is failing — disable strict mode
- Raise `max_tokens` so the model has room to get it right

<details><summary>Show answer & why each option</summary>

- ✅ **Accuracy/grounding problem** — Strict mode guarantees structure, not truth; a consistently-wrong field is a prompt/grounding issue — cite the span, validate it, use CoT-in-schema ordering, then eval.
- **Disable strict mode** — Strict mode is producing valid JSON; disabling it reintroduces parse failures and does nothing for accuracy.
- **Raise `max_tokens`** — A one-name field isn't token-starved; budget has nothing to do with correctness.
</details>

### Q15. You add an LLM-as-judge as ring 3 and trust its scores to gate deploys. A month later it's passing changes that users dislike. Most likely cause and fix?

> **Difficulty:** 🔴 Hard · `[llm-features] llm6` · 🔮 Representative

- The judge model is too small — always use the biggest model as judge
- ✅ The judge is uncalibrated and has drifted (length/position bias) — anchor it to a ~50-example human-labelled golden set and monitor judge-vs-human agreement
- Switch the gate to byte-for-byte snapshot matching instead

<details><summary>Show answer & why each option</summary>

- **Too small** — Model size isn't the core issue; an uncalibrated judge of any size drifts and carries biases.
- ✅ **Uncalibrated + drifted → anchor to human labels** — An LLM judge must be calibrated against human labels and re-checked, or it silently drifts; agreement monitoring keeps the gate honest.
- **Byte-for-byte snapshot** — Flaky on probabilistic output — a correct rephrasing fails — replacing one problem with a worse one.
</details>

---

## Section D — Observability & rollout `[agents-evals-llmops] ag5`

### Q16. Your agent passes every staging test but misbehaves in production. Most likely cause and fix?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag5` · 🔮 Representative

- The model is too small — upgrade it
- ✅ Production feeds untrusted content that perturbs trajectories; add span-level traces and a shadow-mode rollout on real traffic
- Add more retries

<details><summary>Show answer & why each option</summary>

- **Too small** — Model size rarely explains a staging/prod gap; the gap is in the inputs and your visibility into trajectories.
- ✅ **Untrusted content + traces + shadow mode** — Curated staging inputs hide the trajectories real content triggers; decision-level traces + shadow mode surface them before full rollout.
- **More retries** — Don't reveal why the agent chose the wrong path — and can mask the problem while inflating cost.
</details>

### Q17. What's typically the earliest signal that a prompt change quietly regressed your agent?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag5` · 🔮 Representative

- A drop in the final-answer quality score
- ✅ A jump in tokens / cost per resolved task (an extra tool call or two)
- An increase in HTTP 500s

<details><summary>Show answer & why each option</summary>

- **Quality-score drop** — Quality often shifts later; by then users may already be affected.
- ✅ **Cost-per-resolved-task jump** — A small prompt tweak that adds a tool call shows up in cost before quality — alert on cost-per-resolved-task.
- **HTTP 500s** — That's an infra signal; an agent regression usually leaves the service "healthy" while behaviour drifts.
</details>

### Q18. Cost-per-call on your agent has been flat for weeks, yet the monthly bill is climbing and tasks resolve at the same rate. What's happening?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag5` · 🔮 Representative

- Cost-per-call is flat, so nothing changed — the finance team miscounted
- ✅ The agent is making more tool calls per resolved task (cost-per-resolved-task and steps-per-task are up); a change nudged extra calls and the re-sent transcript amplified the cost
- Output tokens got more expensive

<details><summary>Show answer & why each option</summary>

- **Nothing changed** — The bill can climb on flat cost-per-call when the agent makes more calls per task; the right metric isn't cost-per-call.
- ✅ **More calls per task** — Cost-per-call hides per-task regressions; cost-per-resolved-task and steps-per-task are the metrics tied to value, and super-linear transcript cost amplifies an extra step.
- **Output pricier** — A price change is uniform and would move cost-per-call; this pattern points at more calls per task.
</details>

### Q19. You're about to ship a new agent version and want to measure its real-world behaviour with zero user risk before serving any of its output. Which rollout stage?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag5` · 🔮 Representative

- Canary at 5% of live traffic
- A full A/B test
- ✅ Shadow mode: run the agent on real inputs but log its outputs instead of serving them, then mine disagreements with the incumbent for your golden set

<details><summary>Show answer & why each option</summary>

- **Canary at 5%** — Canary already serves output to real users; the question asks for zero user risk first.
- **Full A/B** — Serves both arms to users; not zero-risk and comes after shadow mode.
- ✅ **Shadow mode** — Measures behaviour on the production distribution with no user exposure, and surfaces the real failures you promote into your golden/regression set.
</details>

### Q20. Your team wants to keep full prompt/response traces of every production agent run forever for debugging. What's the senior concern?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag5` · 🔮 Representative

- ✅ Traces are large (full transcripts re-sent every step) and contain PII — sample (keep all failures, a fraction of successes), set retention, and redact at the SDK before storage
- No concern — storage is cheap, keep everything
- Only keep the final answers to save space

<details><summary>Show answer & why each option</summary>

- ✅ **Sample + retention + SDK-side redaction** — Unbounded full-fidelity retention of PII-bearing transcripts is a cost and compliance liability; sampling, retention, and redaction are the controls.
- **Keep everything** — Agent traces are unusually large and carry PII, so it's a cost and compliance problem, not a free debugging win.
- **Only final answers** — Dropping the trajectory spans defeats the purpose — the decision tree is exactly what you need to debug an agent.
</details>

---

## Section E — Reported LLMOps interview questions

> Observed in real 2026 loops. Sources: [amitshekhariitbhu/ai-engineering-interview-questions](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), Mavik Labs *LLM Cost Optimization 2026* (https://www.maviklabs.com/blog/llm-cost-optimization-2026), Langfuse/Arize/LangSmith docs, OpenTelemetry-for-LLMs (https://opentelemetry.io/blog/2024/llm-observability/). Rehearse a crisp answer; the checkpoints above are the drill.

### Q21. What is LLMOps vs MLOps?
> **Difficulty:** 🟢 Easy · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** MLOps manages models *you train* (data, training, versioning, drift). LLMOps manages systems *around a model you mostly call*: prompt versioning + CI eval gates (Q12), token cost/latency (Section B), semantic/prompt caching, provider drift and pinning, and trace-based observability for probabilistic output. The unit of change is often the prompt, not the weights.

### Q22. How do you serve LLMs in production? Explain model quantization.
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), [NVIDIA inference optimization](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/))
> **Senior frame:** For self-hosting: vLLM/SGLang/TGI with continuous batching + PagedAttention; scale on queue depth, not GPU memory ([system-design.md](./system-design.md)). Quantization (INT8/INT4/FP8/AWQ/GPTQ) cuts memory and, on Hopper, compute — see [fine-tuning.md](./fine-tuning.md) for the precision-choice reasoning. For hosted APIs, LLMOps is about routing, caching, and reliability instead.

### Q23. How does prompt caching work? Implement semantic caching.
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), [Mavik](https://www.maviklabs.com/blog/llm-cost-optimization-2026))
> **Senior frame:** *Prompt (prefix) caching* reuses the KV of an exact, stable prefix — order the template stable-first, variable-last, keep dynamic values out of the prefix (Q6, Q8). *Semantic caching* embeds the query and serves a stored answer when similarity clears a threshold — great for repetitive FAQ traffic (40–80% hit), never for state-mutating calls (Q7). Implement: embed → ANN lookup → threshold → serve-or-generate → write-back with TTL.

### Q24. Build a prompt-versioning system.
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** Prompts are deployable artifacts: store in source control, tag a `prompt_version`, log it with every request, gate changes with a CI golden-set eval that blocks regressions, and support one-flip rollback (`fundamentals` Q10, Q12 here). This is what turns a silent console edit into a bisectable PR.

### Q25. Design tiered model-routing that drops cost 50% without regressing quality.
> **Difficulty:** 🔴 Hard · ✅ Reported ([Mavik LLM Cost Optimization 2026](https://www.maviklabs.com/blog/llm-cost-optimization-2026))
> **Senior frame:** Cheap→expensive cascade: a cheap model (or classifier) answers the easy majority; escalate the hard tail to the frontier model on a confidence/verifier signal. Add prompt + semantic caching and tool-batching (routing + caching + batching → 47–80% cut). **Instrument escalation rate + cost-per-request as trend lines** or the cascade silently drifts back up (Q10).

### Q26. LangSmith vs Langfuse vs Arize Phoenix for a multi-framework stack — justify.
> **Difficulty:** 🟡 Medium · ✅ Reported ([research/04 observability set](https://opentelemetry.io/blog/2024/llm-observability/))
> **Senior frame:** Langfuse is OTel-native and the default for a *multi-framework* mesh; Arize Phoenix is OSS tracing+eval on OpenTelemetry; LangSmith gives the deepest LangChain-native experience but is best for pure-LangChain shops. Pick on framework spread + open-standard (OTel) portability, not brand.

### Q27. Where do you put an OpenTelemetry collector in a multi-agent mesh, and what spans do you emit?
> **Difficulty:** 🔴 Hard · ✅ Reported ([OpenTelemetry for LLMs](https://opentelemetry.io/blog/2024/llm-observability/))
> **Senior frame:** A collector per service/sidecar exporting to a central backend. Emit a span per LLM call (model, prompt_version, tokens in/out, cost, latency, cache-hit), per tool call (name, args hash, structured error, retry_after), and per agent step (decision, observation size) — the decision tree you need to debug (Q16). Sample by keeping all failures + a fraction of successes, and redact PII at the SDK (Q20).

### Q28. How would you evaluate the reliability of an MCP wrapper around an internal API?
> **Difficulty:** 🔴 Hard · ✅ Reported ([research/01 §5.1](https://toloka.ai/blog/the-importance-of-mcp-evaluations-in-agentic-ai/))
> **Senior frame:** MCP-specific evals: tool-correctness (does the right tool get called with valid args), schema adherence on returns, structured-error behaviour on upstream failures (Q2 in [agents.md](./agents.md)), latency budget per call, and a security posture check (version pinning, least-privilege, per-call audit — [agents.md Q12–Q14](./agents.md)). Trace every `tools/call` and gate on pass^k.

### Q29. Your cheap→expensive cascade's savings evaporated over two months at flat traffic. Diagnose and instrument.
> **Difficulty:** 🔴 Hard · ✅ Reported ([Mavik](https://www.maviklabs.com/blog/llm-cost-optimization-2026)) — live version of checkpoint Q10
> **Senior frame:** Escalation rate crept up (harder traffic or scorer drift), so more calls hit the expensive model. Track escalation rate and cost-per-request as trend lines and re-tune the threshold/router — don't switch providers blindly or disable caching. Verbatim of Section B Q10 as an open prompt.

---

<sub>⬅ [Back to the index](./README.md) · Related: [Evals →](./evals.md) · [Agents →](./agents.md) · [System design →](./system-design.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
