# LLMOps: Cost, Latency & Reliability

> The two numbers that kill features in production are the bill and the tail — both are engineered, not fixed properties of "the model." This is the lever stack that cuts spend 47–80% and the reliable client that stays up through the provider's weather.

Back to [README](../README.md) · [All questions](../questions/README.md)

---

## What this tests

Whether you think in **tokens and percentiles**, not vibes. A senior AI engineer can (1) estimate a feature's monthly cost on a whiteboard, (2) name the cost levers in order with the catch on each, (3) set latency SLOs from human impact rather than a GPU median, and (4) build a client that survives a flaky, slow, per-call-expensive upstream. The 2026 framing (Mavik): routing + caching + batching routinely cut LLM spend **47–80%** — and the teams that don't do it are usually the ones stamping a timestamp into their cached prefix and wondering why the bill won't drop.

Answer-first for every angle below: **decompose the problem** (TTFT vs total, input vs output, cached vs uncached), reach for the lever that moves the dominant term, name its catch, and budget on a percentile.

---

## Cost math first: the back-of-envelope

Cost is the one thing you can compute before you write a line of code. The formula:

```
cost = input_tokens × price_in  +  output_tokens × price_out
       (cacheable! 0.1x on a hit)   (output ~3-5x input; never cached)
```

Two facts that decide everything:

- **Output tokens dominate.** Output is priced ~3–5× input. "I'd shorten the prompt to make it cheaper" is a junior tell when output is the driver and the prompt is cacheable — you'd save pennies and lose the ~90% caching win.
- **The 4:1 in:out intuition.** A typical RAG/chat call is prompt-heavy on input (retrieved context + system prompt) but the *output* still carries the cost because of the price multiplier. Estimate both, then attack the term that's actually big.

### The worked monthly estimate (base vs prompt-cached)

State your assumptions out loud — the reasoning is what's graded, not the exact dollar figure.

```python
    # Worked monthly estimate — the math that should precede a model choice
    reqs_per_day = 200_000
    in_tokens    = 1_200          # stable 1,000-token system prefix + ~200 dynamic
    out_tokens   = 250
    price_in     = 2.50  / 1_000_000   # $/input token (mid-tier model)
    price_out    = 10.00 / 1_000_000   # $/output token

    base = reqs_per_day * 30 * (in_tokens * price_in + out_tokens * price_out)

    # prompt caching: ~1,000 of the 1,200 input tokens are a cached prefix at 0.1x
    cached_in = (1_000 * price_in * 0.1) + (200 * price_in)
    cached    = reqs_per_day * 30 * (cached_in + out_tokens * price_out)

    print(round(base), round(cached))   # ~$33k vs ~$20k/mo -- caching pays the rent
```

One lever, applied to a stable prefix, drops the bill from **~$33k → ~$20k/month**. That's why caching is lever *one*.

> [!TIP]
> **Prompt caching pays the rent.** A stable system prefix billed at **0.1×** on a hit is often the largest single cost win available — and on OpenAI it's automatic and free. It's the first lever because it compounds with every lever below it.

---

## The cost levers, in order (cheapest effort first)

| # | Lever | Typical win | Cost / catch |
|---|-------|-------------|--------------|
| 1 | **Prompt caching** | ~90% input, ~80% TTFT | Free; protect the byte-stable prefix from dynamic tokens |
| 2 | **Semantic response cache** | 18–60% hit rate (RAG) | Wrong-answer risk if threshold low; never for stateful/fresh queries |
| 3 | **Batch API** | ~50% off | 24h SLA → offline only |
| 4 | **Routing / cascade** | up to 98% cost (FrugalGPT), ~85% keeping ~95% quality (RouteLLM) | Escalation-rate creep, drift, two models to run + eval |
| 5 | **Small fine-tune / self-host** | ~13× cheaper (Character.ai Kaiju) | Data + hosting + eval burden; fails on the long tail; narrow case only |

**1 — Prompt caching.** A cache hit lets the provider skip recomputing the prefill for the shared prefix, saving both compute (latency) and the input charge. Anthropic uses explicit breakpoints (reads at 0.1× base, writes at 1.25×/2×); OpenAI does it automatically on stable prefixes ≥ ~1k tokens. The one rule: keep dynamic content (timestamps, user ids, request ids) **after** the breakpoint — a single per-call token inside the cached prefix sends your hit-rate to zero.

**2 — Semantic response caching.** Serve a stored answer when a new query is *semantically* close to a past one (embedding similarity ≥ a threshold). Reported hit rates: 18–60% in RAG, ~20% in open Q&A. It skips the model entirely on a hit, so it cuts cost and latency to near zero. Layer it *after* provider caching, and never serve stateful or freshness-sensitive answers from it ("what's my balance," "today's incidents"). The threshold is the dangerous knob: too low and you serve a confidently-wrong cached answer to a subtly-different question.

**3 — Batching.** The batch APIs give ~50% off in exchange for a 24h SLA — offline work only: bulk embeddings, eval runs, backfills, nightly summarisation. Never on an interactive path. (Serving-side continuous batching is a different animal — see the resources; it's a throughput lever, not a discount.)

**4 — Routing / cascade.** Most traffic is easy; a minority is hard. A **cascade** runs cheap→expensive sequentially (you sometimes pay twice but never under-serve); a **router** classifies up front (one bill, but a misroute under-serves). FrugalGPT matches GPT-4 quality at up to 98% lower cost by trying a cheap model first and escalating only when a scorer says the answer is weak; RouteLLM cuts cost ~85% while keeping ~95% quality with a learned router. **Watch the escalation rate** — it's the number that tells you your headroom, and when it drifts up your savings evaporate.

**5 — Small fine-tune / self-host.** When the output distribution is *narrow* and volume is *high*, a small fine-tuned model beats a frontier API. Character.ai's Kaiju serves at ~13× lower cost by engineering the inference stack (MQA, sliding-window attention, cross-layer KV sharing, int8). The *last* lever — you take on data curation, eval, hosting, and the risk that the narrow model fails on the long tail the frontier model handled for free.

> [!WARNING]
> **The escalation rate is a live metric, not a launch number.** A cheap→expensive cascade looks great at launch, then two months later spend is back up though traffic is flat. The cause: the escalation rate crept from 20% → 60% (harder traffic or scorer drift), so more calls hit the expensive model — and you may now be paying **two bills**. Track escalation rate *and* cost-per-request as trend lines and re-tune the threshold; don't guess.

> [!WARNING]
> **"Cache saves latency" — which cache?** Two different mechanisms hide behind one word. *Prompt caching* skips prefill recompute on a shared prefix (saves TTFT + input cost, model still runs). *Semantic caching* skips the model entirely on a near-duplicate query (saves everything, but risks a wrong answer at a low threshold). Conflating them in an interview signals you've only read the marketing.

---

## Latency: SLOs from human impact, not your GPU's median

Latency decomposes differently from cost:

```
latency = prefill(input_tokens)  +  output_tokens × inter-token-latency
          \__ sets TTFT (prompt len) __/   \__ sets TOTAL time (output len) __/
```

**TTFT** (time-to-first-token) is set by prefill ∝ prompt length; **total time** is set by output length × ITL. You cut them with *different* levers: shrink/cache the prompt for TTFT, shrink output / stream / pick a faster model for total.

Perception thresholds are well established: TTFT under ~200ms feels instant, ~500ms responsive, >1s users notice, >2s they abandon. Budget per surface, on a **percentile**, not an average:

| Use case | TTFT (p99) | ITL | Stream? |
|----------|-----------|-----|---------|
| Voice agent | ~150 ms | 30 ms | yes |
| Interactive chat | ~300 ms | 50 ms | yes |
| Inline code complete | ~500 ms | 25 ms | yes |
| RAG-augmented chat | 1500 ms+ | 80 ms | yes (prefill budget covers retrieval) |
| Batch / async agent | n/a | — | no |

**Streaming is a perception lever, not a wall-clock one.** Server-sent events cut *perceived* latency by up to ~80% by showing the first token immediately — but total completion time is unchanged. Default it for chat and inline completions. Two traps: with agents/tools, buffer tool-call argument deltas and validate structured output only at the end; with reasoning models, you still pay for (and wait on) hidden thinking tokens even when streaming — the most common source of surprise bills. Always pair streaming with server-side cancellation, or an abandoned tab keeps spending your money.

> [!TIP]
> **Size timeouts from p99.9, not the median.** A read timeout near the provider's p99.9 latency gives ~0.1% false-timeout rate while it's healthy. Sizing off the median trips constantly on normal slow-but-successful calls.

> [!WARNING]
> **"Streaming / lower temperature makes it faster."** Neither touches wall-clock. Streaming improves *perceived* latency only; temperature affects sampling, not throughput. And "p50 is 600ms" while p99 is 8s is the answer that loses the round — the tail is what users feel and remember.

---

## The reliable LLM client: the four-layer stack

The provider is not your dependency — it's your **weather**. A model API *will* time out, return 429s after a hot launch, throw 5xx, and occasionally have a full outage that looks identical to a rate limit from the outside. What's different from a normal RPC: calls are **seconds not milliseconds** (a hung call ties up a worker), each call **costs real money** (a blind retry doubles the bill), failures are **often partial** (a stream that dies at token 800/1,000), and an outage looks *identical* to a rate limit. Every layer below is the classic resilience pattern, tuned for those four facts.

| Layer | Job | LLM-specific twist |
|-------|-----|--------------------|
| 1. **Timeout** (connect + read) | Bound how long any single attempt can hang | Size read from p99.9; for streams use TTFT + idle/inter-token timeout, not one whole-response read; cancel server-side on abandon |
| 2. **Retry** (backoff + jitter) | Re-attempt *only* retryable errors | Every retry doubles cost; a mid-stream failure already burned tokens — don't blindly re-fire |
| 3. **Fallback / rate-limit / circuit breaker** | Reach a healthy endpoint; stop hammering a dead one | Each fallback model needs its **own eval** — failover keeps you up but can silently serve worse answers |
| 4. **Hedging + idempotency + gateway** | Tame the tail; dedup writes; centralise | Hedging multiplies cost on an expensive upstream — bound the rate, skip non-idempotent tool calls |

**Order matters.** The circuit breaker sits *above* retries and *below* fallbacks: it stops you retrying a corpse and frees the fallback chain to reach a healthy endpoint. Retry handles a *blip*; the breaker handles a *sustained* failure; fallback handles "this endpoint is down but another is up." Each layer is independently testable — fault-inject one, assert the next engages.

### Retryable vs non-retryable errors

| Class | Codes | Action |
|-------|-------|--------|
| **Retryable** | 429, 500, 502, 503, 504 | Capped exponential backoff + jitter; honour `Retry-After` on a 429 |
| **Non-retryable** | 400 (incl. context-length overflow), 401, 403, 404 | Fix the input / auth — they fail identically next time; retrying amplifies overload |

### Retry with backoff, jitter, and error classification

```python
    import httpx
    from tenacity import (retry, stop_after_attempt,
                          wait_exponential_jitter, retry_if_exception)

    RETRYABLE = {429, 500, 502, 503, 504}

    def is_retryable(e) -> bool:
        return (isinstance(e, httpx.HTTPStatusError)
                and e.response.status_code in RETRYABLE)

    @retry(stop=stop_after_attempt(3),
           wait=wait_exponential_jitter(initial=1, max=30),   # backoff + JITTER
           retry=retry_if_exception(is_retryable),
           reraise=True)
    def call_llm(client: httpx.Client, payload: dict):
        r = client.post("/v1/chat/completions", json=payload,
                        timeout=httpx.Timeout(connect=3.0, read=45.0))  # connect + read
        if r.status_code == 429 and "retry-after" in r.headers:
            # honour the header before re-raising to the retry layer
            raise httpx.HTTPStatusError("rate limited", request=r.request, response=r)
        r.raise_for_status()
        return r.json()
```

**Why jitter specifically** — a favourite interview probe. Without it, every client that failed at the same instant computes the *same* backoff (1s, 2s, 4s…) and re-hits the recovering provider in synchronised waves — the "thundering herd." AWS's Builders Library shows **full jitter** (sleep a random amount in `[0, backoff]`) flattens those waves. The counter-intuitive lesson: adding randomness makes the system *more* stable.

**Circuit breaker** — a small state machine worth drawing: **closed** (requests flow; count failures over a window) → trips to **open** after threshold (e.g. 5–10 failures in 60s; fail fast for a 30–60s cooldown) → **half-open** (allow one probe; success closes it, failure re-opens). Its whole point is to stop spending retries + timeouts on a known-dead endpoint — the spend that turns one provider blip into your outage.

**Rate limits** — two budgets must converge: the provider's TPM/RPM (signalled by 429 + `Retry-After`) and a **client-side token bucket** that refuses an over-budget request *before* it leaves your process. That client-side limiter — not more retries — is what actually stops a rate-limit storm.

**Hedging** ("The Tail at Scale", Dean & Barroso) — if a call hasn't returned a first token by ~p95, fire a *second* request and take whichever responds first, cancelling the loser. This collapses the long tail toward p95 at the cost of a few percent duplicate calls. The catch on LLMs: it multiplies cost on a per-call-expensive upstream and is unsafe for non-idempotent tool calls — reserve it for read-style, latency-critical paths and **bound the hedge rate**. Knowing when *not* to hedge is the senior signal.

**Idempotency & the gateway** — for write-side calls use an idempotency key (hash model + prompt + request id) and a dedup table so two retrying workers collapse into one upstream call. Once more than one feature uses LLMs, stand up an **AI gateway** (LiteLLM, Portkey, Cloudflare AI Gateway): it centralises routing, retries, caching, spend caps, and the metrics that matter — so provider portability is a config change, not a rewrite.

> [!WARNING]
> **"Retry everything."** Stopping at "retries + backoff" signals you haven't run this in production. It omits jitter (thundering herd), a breaker (retrying a dead endpoint into the ground), a client-side limiter (storming a recovering provider), and error classification (retrying un-retryable 400s forever). Worse, a 400 from a context-length overflow will fail identically every time — retrying it amplifies overload for zero gain.

> [!WARNING]
> **"The fallback model is fine."** Your primary is down, the breaker fails over to a cheaper second-provider model, availability recovers — and a week later you learn answer quality quietly dropped during the outage. Failover keeps you *up* but can silently serve worse answers (quality drift, prompt non-portability). Every fallback path needs its **own light eval** and a prompt validated for that model, plus a quality-metric alert — not just an availability check.

---

## Observability: the one span that explains a request

Instrument every operation as a **span** — generation, tool call, retrieval — and a trace is a tree of spans (one agent task = one trace; generations and tool calls = child spans). Align to OpenTelemetry's GenAI semantic conventions so attribute names (model, token counts, tool names) are portable across backends rather than locked to one vendor's SDK — and so an agent trace lives in the same system as your service traces, letting you correlate "the agent looped" with "the downstream API was slow" in one view.

The load-bearing span attributes: **inputs/outputs** (redacted), **token usage + cost** per call, **latency split into TTFT and total**, **tool name + arguments**, the **structured error** if any, and a **session/user id**. Scale gotchas: traces are large (full prompts re-sent every step) so **sample** (keep 100% of failures, a fraction of successes); prompts contain PII so **redact at the SDK** before it leaves your process.

**Cost is your earliest regression signal.** A "small" prompt tweak that nudges the agent into one extra tool call shows up in tokens-per-task *before* it shows up in answer quality — and because the transcript re-sends every step, a one-step regression moves cost by far more than 1/N. Alert on **cost-per-resolved-task** (tied to business value), not cost-per-call, which can stay flat while cost-per-resolved-task climbs.

---

## Silent regressions: pin the model, gate the bump

The drift that bites most isn't a change you made — a provider silently updates a model checkpoint and quality sags weeks later. Anthropic's own postmortem traced *two months* of "it got dumber" complaints to three small interacting changes, with no single git commit as the culprit; only production-distribution traces could isolate it.

The defence:

- **Pin model versions.** Un-pinned aliases (`gpt-4o`, `claude-3-5-sonnet-latest`) drift under you. Pin to a dated snapshot.
- **Regression-gate every bump.** When you deliberately move to a new version, run your offline eval suite in CI as a gate — the same way you'd gate a code change. A version bump is a code change.
- **Watch the production distribution.** Shadow mode → canary → A/B → full rollout, with online-eval alarms (cost-per-resolved-task, steps-per-task, tool-call distribution) standing watch for the drift that isn't in your git log.

> [!WARNING]
> **"We didn't pin the model."** An un-pinned version is a silent dependency that a third party rewrites without telling you. APM stays green, the bill barely moves, and quality sags for weeks. Pin to a dated snapshot and regression-gate every deliberate bump — otherwise your "stable" feature is riding a moving target.

---

## Interview angles

**Q: What is LLMOps vs MLOps?**
MLOps manages *models you train* — data pipelines, training runs, model registries, drift on your own features. LLMOps manages *models you mostly call* — prompt versioning, eval gates, caching, routing, cost/latency SLOs, and resilience against an upstream you don't control. The centre of gravity shifts from training reproducibility to **prompt/eval/observability as first-class artifacts** and from feature drift to *provider* drift (silent checkpoint updates). Same discipline of gates and monitoring; different failure surface.

**Q: How would you serve LLMs in production?**
Behind a gateway that owns the hardened client: timeout → classified retry (backoff + jitter, honour `Retry-After`) → fallback chain → circuit breaker, plus a client-side rate limiter, idempotency dedup, prompt/semantic caching, spend caps, and per-request cost/latency spans. Set SLOs per surface on a percentile. If the workload is narrow + high-volume, consider a self-hosted small model (vLLM with continuous batching) behind the same gateway.

**Q: How does prompt caching work?**
On a cache hit the provider skips recomputing the prefill for the shared prefix — you save both the compute (TTFT, ~80%) and the input-token charge (~90%, billed at 0.1×). It matches a *byte-stable* prefix, so the one rule is keeping dynamic tokens (timestamps, ids) after the breakpoint. Debugging a 5% hit-rate: a dynamic value is sitting *inside* the cached prefix — move it to the message body.

**Q: Implement semantic caching.**
Embed the incoming query, nearest-neighbour search against a vector store of past (query → answer) pairs, and serve the stored answer if similarity ≥ a tuned threshold (start ~0.95). Skip the cache for stateful/fresh queries. The threshold is the whole game: too low and you serve a confidently-wrong answer to a subtly-different question — so measure precision of hits, not just hit rate, and never cache state-mutating calls.

**Q: Build a prompt-versioning system.**
Prompts are code: version them in a registry with a content hash, tie each version to an eval run, and gate promotion on the eval suite passing. Pin the *model version* alongside the prompt version (a prompt validated on one snapshot isn't validated on another). Log the prompt-version id on every span so a quality/cost regression links straight to the change that caused it.

**Q: Design tiered routing that drops cost 50% without regressing quality.**
Route the easy majority (~60%) to a cheap model, escalate the hard tail. Cascade (cheap→expensive with a scorer) never under-serves but sometimes pays twice; a learned router classifies up front for one bill but a misroute under-serves. Guard quality with a per-tier eval and a scorer calibrated against human labels; guard savings by tracking the **escalation rate** as a trend line — if it drifts up, re-tune the threshold before the savings evaporate.

**Q: Build a reliable client.**
The ordered four-layer stack (above), each layer independently tested. Lead with *why* the order matters: breaker above retries (stop retrying a corpse), below fallback (free the chain to reach a healthy endpoint). Name the LLM twists: cost-doubling retries, partial streams that already cost money, fallback quality drift, wasted tokens on abandoned calls. Add hedging for the slow tail — bounded, never on non-idempotent calls.

**Q: Where does the OTel collector go, and what spans?**
Sidecar/agent collector per service (or per node in a mesh), exporting to a central backend — so redaction and sampling happen close to the source before PII leaves the process. Spans: one root per task, child spans per generation and per tool call, with model, token counts, cost, TTFT/total latency, tool name + args, and structured error as attributes. Sample 100% of failures, a fraction of successes.

**Q: Estimate monthly cost on a whiteboard.**
State assumptions out loud: model, price/million in and out, req/day, tokens in/out. Then `req/day × 30 × (in×price_in + out×price_out)`. Note output dominates (~3–5×), then show the caching delta (~1,000-token stable prefix at 0.1×) — that's the ~$33k → ~$20k move. The reasoning is graded, not the exact figure.

---

## 📚 Resources

- 📰 [Mavik — LLM Cost Optimization 2026](https://www.maviklabs.com/blog/llm-cost-optimization-2026) — the 2026 framing: routing + caching + batching cut spend 47–80%; the lever stack in one place.
- 📄 [The Tail at Scale (Dean & Barroso)](https://research.google/pubs/the-tail-at-scale/) — the hedging + tail-latency discipline behind layer 4; why you engineer p99, not the median.
- 💻 [vLLM](https://github.com/vllm-project/vllm) — Apache-2.0, ~85k★ — PagedAttention + continuous batching; the serving backbone when you self-host the narrow, high-volume case.
- 💻 [langfuse](https://github.com/langfuse/langfuse) — MIT, ~30k★ — OTel-native LLM tracing/cost/observability; the span backbone for cost-per-resolved-task and drill-to-span debugging.
- 📰 [OpenTelemetry for LLMs](https://opentelemetry.io/blog/2024/llm-observability/) — the GenAI span model: portable attribute names for model, tokens, tools.
- 📘 [Anthropic prompt caching docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) — cached-prefix pricing/mechanics (0.1× reads, breakpoints); lever 1, from the source.
- 📰 [NVIDIA — Mastering LLM Inference Optimization](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/) — batching, KV-cache, MQA/GQA; the serving-side latency levers.
- 📰 [Anyscale — Continuous batching for LLM inference](https://www.anyscale.com/blog/continuous-batching-llm-inference) — the ~23× throughput intuition behind modern serving batchers.

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [LLMOps questions](../questions/llmops.md)
