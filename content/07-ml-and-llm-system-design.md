# ML & LLM System Design

The signature onsite round, reduced to one reusable rubric you can ride down any GenAI prompt — plus the whiteboard math that separates an offer from a "strong on paper."

Back to [README](../README.md) · [All questions](../questions/README.md)

## What this tests

Answer-first: this round is **not** a memorized-architecture quiz. The interviewer is watching whether you can take a vague business goal — "design a support chatbot," "design LLM inference at scale" — and walk a fixed spine under a 60-minute clock, **naming a tradeoff and putting a number on every box**. HelloInterview calls GenAI system design "the wild west," where consistency across companies is "frustratingly low." That is exactly why the *framework* beats any one company's model answer.

Three things get graded, every time, above raw modeling depth:

1. **Structured thinking** — you frame before you draw, and you finish the whole spine.
2. **Tradeoff reasoning** — RAG vs fine-tune, build vs buy, accuracy vs latency vs cost, one big model vs a cascade — named unprompted.
3. **Estimation on the spot** — QPS → tokens/day → $/day → GPU count, sketched live. Interviewers use this as a **filter**; a candidate who can't size the system has usually never run one.

Graded interviewing.io debriefs are blunt about it: a candidate who scored **2/4 technical but 4/4 communication** got the offer, because strong clarification and proactive framing outweighed a shaky high-level design. Communication and framing routinely outvote the 4/4-technical rambler.

---

## The reusable rubric — one spine for every GenAI prompt

Every credible framework collapses to the same stages. Memorize these seven, ride them top to bottom, and defend each with a number and a tradeoff. This is the spine the rest of this file hangs on.

| # | Stage | Key questions to ask | What a strong (senior) answer says |
| --- | --- | --- | --- |
| 1 | **Problem framing & requirements** | Is this even an ML problem? What's the target function? Who's the user? Scale (QPS), latency SLO, quality bar? | Maps the prompt to a known objective (retrieval-conditioned generation, ranking, classification). Volunteers the silent numbers ("I'll assume ~50 QPS, p95 < 2s, hallucination < 5% — confirm?"). ML only if it beats the best heuristic by ~3×. |
| 2 | **Data & eval set** | Where do labels/eval cases come from? How do I measure quality *without* ground truth? What's the golden set? | Treats the **eval set as a first-class deliverable**. Names a reference-free stack (RAGAS faithfulness, LLM-as-judge + human audit), an offline metric tied to an online business metric, and a counter-metric. |
| 3 | **Retrieval / model choice** | RAG vs fine-tune vs prompt? Which model? Embeddings + index? Rerank? Single model or cascade? | Baseline first, then the simplest thing that clears the bar. Hybrid retrieval (BM25 + dense, RRF) by default; a cross-encoder reranker as the cheapest accuracy buy; routes easy queries to a cheap model. |
| 4 | **Serving & latency budget** | Batch/online/streaming? TTFT vs total? Where do the milliseconds go? Continuous batching, prefix caching? | Breaks the SLO into a **per-stage budget that sums** (retrieval + rerank + prefill + decode + network). Names vLLM continuous batching + PagedAttention (8–23×) and prefix caching (~90% cost / ~85% latency). |
| 5 | **Eval & guardrails** | How do I catch hallucination, PII, permission leaks, OOD? What's the refusal path? Citations? | Grounding via mandatory citations, ACL-filtering at retrieval, a refusal classifier for OOD, PII redaction, an audit log. Measures faithfulness *separately* from retrieval recall. |
| 6 | **Monitoring & cost** | What drifts? Retrain cadence? $/day today and at 10×? Cost cliff? | Names drift signals (PSI > 0.2 → retrain), shadow → canary rollout, a $/1k-QPS estimate, and the caching levers that bend the cost curve. |
| 7 | **Scaling** | 10× traffic — what breaks? Autoscaling? Cold start? Multi-region? | Autoscale on **queue depth tied to a latency SLO** (never GPU memory — it's preallocated), a min-replica buffer for cold starts, graceful degradation (cheaper model over a timeout), active-active failover. |

In prose: you **frame** the target and the numbers, define the **eval set** you'll grade against, pick the **retrieval/model** shape (baseline → the simplest thing that works), lay out **serving** with a latency budget, wrap it in **eval & guardrails**, wire up **monitoring & cost**, and close with **scaling** follow-ups. Miss a stage and the interviewer has already penciled in a gap. Seniority shows up as knowing which stages to *abbreviate* — roughly **80/20 breadth/depth at mid, 60/40 senior, 40/60 Staff+** — never as skipping framing.

> [!TIP]
> The eval set is a **first-class deliverable** in a design answer, not an afterthought you mention if asked. "Eval is the new system design" — a senior candidate can be asked to *design the eval harness before the agent*: define metric → build a golden dataset → version it → run regression. If your design has no eval plan, it reads as "I built a demo," not "I run this in production."

## How to open — clarify before you draw a single box

The single most common way to fail is to skip framing and design for the wrong `Y`. Opening with *"I'd use a two-tower DNN with a cross-encoder reranker"* before stating the target function, the metric, or the scale is the fastest way a strong-on-paper candidate underwhelms — it signals you optimize components, not outcomes.

Spend the first ~10 minutes locking four things, then confirm them out loud:

- **Target function** — state the supervised objective in 30 seconds ("retrieval-conditioned generation, maximize answer utility − hallucination penalty − cost − latency").
- **Scale** — QPS, catalog/corpus size, concurrent users. If the prompt is silent, *say the number you're assuming.*
- **Latency SLO** — convert "how fast?" into a millisecond budget *before* picking a model. For LLM chat, split it: TTFT vs total.
- **Quality bar** — the metric and its number ("citation accuracy > 90%, hallucination < 5%"), plus a counter-metric.

Real prompts plant these numbers on purpose: a fraud round opens with "millions of transactions daily, end-to-end < 300ms, inference < 20ms, AUC-ROC and precision@K." A RAG prompt confirms "50M internal docs, p99 < 2s, citation accuracy > 90%." When the prompt stays silent, **stating the number you assume and confirming it** is the single move that separates an ML-engineer candidate from an SWE one.

> [!WARNING]
> **Jumping to architecture before requirements.** Sketching the model in minute one is Sin #1. Every architecture choice is ungrounded until you've locked the target `Y`, the metric, and the scale — and the wrong `Y` poisons every downstream decision. Recovery if you catch yourself: restate the assumed target function explicitly and offer to re-prioritize the agenda.

## Whiteboard estimation — QPS → tokens → $/day → GPU/RAM

Estimation is a **filter**. Interviewers watch whether you can size QPS → tokens → dollars → GPUs on the spot; fumbling here reads as never having run the system. Keep the mental model tiny: **cost = tokens × price** (output ~3–5× input), **GPU count = peak QPS ÷ throughput-per-replica**, and **KV-cache RAM ≈ tokens × 2 × layers × hidden × bytes**. Here's a load-bearing snippet you can reconstruct on a whiteboard:

```python
    # --- WHITEBOARD SIZING: a RAG chatbot on a hosted API, then self-hosted ---

    # 1) Traffic
    dau            = 200_000            # daily active users
    queries_per_u  = 5                  # sessions/user/day
    daily_queries  = dau * queries_per_u          # 1.0M queries/day
    avg_qps        = daily_queries / 86_400        # ~11.6 QPS average
    peak_qps       = avg_qps * 5                    # diurnal peak ~58 QPS

    # 2) Tokens per query (RAG: big cached prefix + short answer)
    prompt_tokens  = 2_000             # system + retrieved context (cacheable)
    output_tokens  = 300               # the answer the user reads
    cache_hit_rate = 0.80              # 80% of the prefix hits the prefix cache

    # 3) $/day on a hosted API (illustrative: $3/M input, $15/M output)
    in_price, out_price = 3.0 / 1e6, 15.0 / 1e6
    cached_price        = in_price * 0.1              # cache read ~0.1x input
    per_query_in  = (prompt_tokens * cache_hit_rate * cached_price
                     + prompt_tokens * (1 - cache_hit_rate) * in_price)
    per_query_out = output_tokens * out_price
    daily_cost    = daily_queries * (per_query_in + per_query_out)
    print(f"{avg_qps:.0f} avg QPS, ${daily_cost:,.0f}/day  (~${daily_cost*365/1e3:,.0f}k/yr)")

    # 4) Self-hosted GPU sizing (70B on 8xH100 node)
    tps_per_replica = 60               # sustained concurrent QPS one node serves
    replicas        = -(-peak_qps // tps_per_replica)   # ceil -> 1 node here
    node_cost_day   = 8 * 3.15 * 24                       # 8xH100 @ ~$3.15/GPU-hr
    print(f"{replicas} node(s), ${node_cost_day*replicas:,.0f}/day GPU floor")

    # 5) KV-cache RAM per in-flight request (sanity check for concurrency)
    layers, hidden, kv_bytes = 80, 8192, 2            # 70B-class, fp16
    ctx = prompt_tokens + output_tokens
    kv_gb = ctx * 2 * layers * hidden * kv_bytes / 1e9  # ~6 GB/req at 2.3k ctx
    print(f"KV cache ~{kv_gb:.1f} GB/request -> paged attention is mandatory")
```

The point isn't precision — it's the **chain**: traffic → tokens → dollars, then a second pass for GPUs and RAM. Two levers fall straight out of the arithmetic: **prefix caching** collapses the input term (~90% of the prefix cost), and **cutting output tokens cuts both latency and cost** because decode is the expensive, slow phase. Quote the numbers, then quote the lever.

## Latency budgets — a per-stage budget that sums to the SLO

Quoting one latency number is a junior tell. A senior gives a **budget that sums** to the SLO, names the stage that dominates, and states the lever that moves it. For LLMs the budget splits into **two numbers**: **TTFT** (time-to-first-token = *prefill*, compute-bound on prompt length) and **total** (TTFT + decode, where decode/ITL is *memory-bandwidth-bound* over the KV cache — the GPU re-reads the whole cache for every token).

| Stage | Bound by | Budget (ms) | First lever to cut it |
| --- | --- | ---: | --- |
| Retrieval (hybrid BM25 + dense ANN) | I/O + ANN search | 120 | Cache; smaller candidate set; managed ANN |
| Rerank (cross-encoder, 50→8) | GPU/model pass | 80 | BGE local (~100ms) vs Cohere API; skip on easy queries |
| Prefill / **TTFT** | compute on prompt len | 400 | **Prefix cache** the stable prefix (~85% off), shrink prompt |
| Decode (300 tokens @ ~40 tok/s) | KV-cache bandwidth | 900 | Fewer output tokens; smaller model; speculative decoding |
| Network + serialization + buffer | round-trips | 200 | Co-locate; stream tokens (perceived latency ≪ total) |
| **Total p95** | | **1,700** | Fits a **2s SLO** with ~300ms headroom |

> [!WARNING]
> **Quoting one latency number** ("it'll be under 2 seconds"). The interviewer wants the *decomposition* — which stage dominates and why. And know the curveball: *"why not just buy a bigger GPU to cut latency?"* A bigger GPU helps **prefill** (compute) and **concurrency**, but per-token **decode** is bandwidth-bound over the KV cache — you cut it by shrinking the KV footprint (quantize, smaller model, speculative decoding), **not** by buying a faster chip. "TTFT and total are optimized with different levers" is the senior LLM-serving signal.

## Tradeoff reasoning — the four you'll be pushed on

Naming the tradeoff *before the interviewer asks* is the cleanest senior tell in the room. Keep these four loaded:

| Decision | Pick the left when… | Pick the right when… | The senior nuance |
| --- | --- | --- | --- |
| **RAG vs fine-tune vs prompt** | Prompt: task is simple, in-distribution. RAG: knowledge is large, changes often, needs citations. | Fine-tune: you need a *behavior/format/tone* or latency shrink, and the knowledge is stable. | They compose. RAG for *facts* (re-index on change — FT bakes stale facts into weights); FT for *form*. Start with prompt, add RAG, fine-tune last. |
| **Build vs buy** | Buy (hosted API): time-to-market, small/medium scale, no SRE. | Build (self-host vLLM): high steady volume, data-residency, cost at scale. | Crossover is a $/token break-even. Self-hosting halves infra cost but demands SRE, autoscaling, and on-call. Say the volume where it flips. |
| **Accuracy vs latency vs cost** | — pick two — | — the third pays — | A cross-encoder reranker *buys accuracy cheaply* ($0.001–0.002/q) and *cuts* cost by shrinking LLM context. Not always a strict trilemma — name the free lunch. |
| **Single model vs cascade** | Single: uniform hard queries, simplicity. | Cascade/router: a fat easy tail. | A router picks the cheapest model that clears the bar. A 5-pass medium model + reranker often **beats** one giant-context call at ~20% of the cost. Cascade the easy tail. |

> [!TIP]
> "Accuracy vs latency vs cost" is often **not** a strict trilemma. The reranker is the classic counter-example: it raises accuracy *and* cuts cost, because narrowing 50–100 candidates to 5–10 before generation shrinks the LLM's (expensive) input context. Spotting where a lever moves two axes the right way at once is a Staff-level signal.

> [!WARNING]
> **One big model for everything.** A single large-context call is expensive (you pay per token, attention is super-linear), and quality *degrades* as the window fills (Lost-in-the-Middle, context rot). "Just stuff 200k tokens and skip retrieval" is the canonical trap. The senior move: narrow the candidate set before generation (retrieve + rerank), route easy queries to a cheap model, and reserve the big-context call for genuinely holistic single-document tasks — never as the default.

---

## Worked design 1 — customer-support RAG chatbot

**Prompt:** "Design a customer-support chatbot grounded in our help docs + past tickets. It must cite sources and rarely hallucinate."

- **1. Framing.** Retrieval-conditioned generation, not open-domain. Target: maximize deflection (resolved without a human) − hallucination penalty − cost. Assume ~1M queries/day (~12 QPS avg, ~60 peak), p95 < 2s, citation accuracy > 90%, hallucination < 5%, thumbs-up > 70%. Confirm the corpus (~1M docs + tickets) and that answers must carry a source ID.
- **2. Data & eval set.** Golden set of ~300 real questions with known-good answers + the doc that should ground them. Offline: Recall@10 for retrieval, RAGAS **faithfulness** + context-precision for generation. Online: citation accuracy (human spot-check), hallucination rate, thumbs-up, deflection rate. Counter-metric: **false-deflection** (chatbot "resolved" but the user re-opens).
- **3. Retrieval / model.** ingest → structure-aware chunk (400-token children, 2000-token parents) → embed → **hybrid** BM25 + dense (RRF) → retrieve 50 → **cross-encoder rerank → 8** → prompt with citations → mid-tier LLM. BM25 catches exact error codes/SKUs; the reranker is the cheapest accuracy buy. No fine-tune yet — RAG covers the facts.
- **4. Serving & latency.** Online synchronous, streamed. Budget as the table above (~1.7s p95). Prefix-cache the stable system-prompt + retrieved-context prefix (stable-first, user-turn-last) → ~90% cost / ~85% latency on the prefix. Keep output short.
- **5. Eval & guardrails.** Every claim must trace to a cited chunk (faithfulness check); refusal path for OOD questions; PII redaction; audit log of (query, retrieved IDs, answer). Bound hallucination with mandatory citations + a low-confidence → human-escalation threshold.
- **6. Monitoring & cost.** Watch retrieval recall, faithfulness, and thumbs-up continuously (LLM-as-judge sampled + human audit). **Freshness drift** is the killer: re-index on doc modification dates or it cites a stale policy. Cost cliff → embedding-query cache + LLM-output cache (>30% hit rate typical for support) + prefix caching. $/day from the estimation snippet.
- **7. Scaling.** 10× → autoscale API concurrency or vLLM replicas on queue-depth-vs-latency; cache absorbs the fat head of repeated questions; degrade to a cheaper model under load rather than time out.

## Worked design 2 — LLM inference at scale

**Prompt:** "Design LLM inference serving a 70B model for millions of daily requests."

- **1. Framing.** This is a *serving/throughput* problem, not a modeling one. Target: maximize sustained QPS per GPU-dollar subject to TTFT/ITL SLOs. Assume ~100 QPS peak, TTFT p99 < 500ms, ITL < 50ms/token, mixed prompt lengths. Confirm self-host vs API (assume self-host — the point of the prompt).
- **2. Data & eval.** "Eval" here is a **load-test harness**: a replayable traffic trace (real prompt-length distribution) measuring throughput, TTFT, ITL, and cost/1k-tokens across configs. Regression-gate every config change against it.
- **3. Model choice.** 70B on an 8×H100 node in **FP8** on Hopper (~2× throughput vs FP16 — wrong precision is a classic pitfall). Route: a router sends the easy tail to a smaller/cheaper model; reserve the 70B for hard queries.
- **4. Serving & latency.** The whole design is here. **Continuous batching + PagedAttention** (vLLM/SGLang) — **8× over naive, up to 23× with paging** — is the single largest throughput lever. **Prefix caching** for repeated system prompts. TTFT = prefill (compute); ITL = decode (KV-bandwidth) — cut ITL with quantization / speculative decoding, not a bigger GPU.
- **5. Guardrails.** Per-request token caps (bound a runaway cost bug), timeout + graceful degradation, request-level logging with **cost alerting** (a silent cost bug with no alert is a real war story), input/output safety filters.
- **6. Monitoring & cost.** Throughput, TTFT/ITL p99, GPU utilization, cost/1k-tokens, cache hit rate. GPU floor: an 8×H100 node is ~$600/day before traffic; a 70B replica runs ~$34k/yr — prefix caching drops steady-state utilization ~70% → ~25%, letting one node serve 2–3× volume.
- **7. Scaling.** **Autoscale on queue depth (or batch size) tied to a latency target** — never GPU memory (vLLM preallocates the KV cache, so memory-used only scales *up*). **Cold start is the hard floor:** an 8×H100 vLLM replica takes tens of seconds (serverless seen at 460s); HPA polls every 15–60s, so keep a **min-replica buffer** sized to the diurnal peak. Multi-region active-active for outage tolerance.

> [!WARNING]
> **The OOM-with-free-memory curveball.** "The cluster OOMs under load but `nvidia-smi` shows free memory — why?" **KV-cache fragmentation** from naive contiguous max-length reservations, not insufficient GPUs. Fix is **PagedAttention** (fixed-size pages on demand, waste 60–80% → <4%), not more hardware. Missing this signals you've never run inference under real load.

## Worked design 3 — support-ticket triage agent

**Prompt:** "Design an agent that reads incoming support tickets, categorizes them, sets priority, and routes/resolves or escalates."

- **1. Framing.** Agentic classification + tool-use, not free generation. Target: correct route + correct priority, maximize auto-resolution while minimizing mis-escalation. Assume ~50k tickets/day, most latency-tolerant (async, seconds OK — not interactive chat), accuracy bar tied to a business cost (a mis-routed P0 is expensive).
- **2. Data & eval set.** Golden set of historically-labeled tickets (category, priority, resolution). The end-state is what matters, so evaluate **trace-based**: did it pick the right category, call the right tools in the right order, and reach the right terminal state? Metric: routing accuracy, priority F1, auto-resolution rate; counter-metric: **mis-escalation / false-auto-resolve** rate.
- **3. Model / architecture.** Planner → executor with **function calling** over a tool set (search KB, look up account, create/route ticket, escalate). RAG for the KB lookups; a small cheap model for the classification tail, escalating to a stronger model only for ambiguous tickets (**cascade**). Memory for multi-turn threads with a summarization + decay policy.
- **4. Serving & latency.** Async / queue-based — spikes absorbed by the queue, seconds-scale latency acceptable. Tool calls dominate wall-clock, not decode; parallelize independent tool calls.
- **5. Eval & guardrails.** Confidence threshold → auto-resolve only above it, else **human-in-the-loop** escalation. Tool-call validation (never let the agent take a destructive action unchecked), PII redaction, full trace/audit log per ticket. Refusal/escalate path for OOD tickets.
- **6. Monitoring & cost.** Watch routing accuracy, auto-resolution rate, mis-escalation, cost/ticket. **Concept drift** is the real risk — new product launches shift ticket categories; PSI on the category distribution > 0.2 → refresh few-shots / retrain the classifier. Trace-level observability (which tool failed, where the plan derailed).
- **7. Scaling.** Queue-based autoscaling on backlog depth; the cheap-model tail keeps cost flat as volume grows; degrade to "route to human queue" under overload rather than mis-resolve.

---

> [!WARNING]
> **No eval plan.** "How do you know it works?" met with silence, or "we'll watch accuracy on a hold-out," is a fail — especially for RAG/agents where there's no ground truth. The eval set (golden cases + reference-free judges like RAGAS/LLM-as-judge + a human audit sample) **is part of the design**. Name it unprompted.

> [!WARNING]
> **Ignoring cost and scale until asked.** A design that stops at the happy path — no $/day, no 10× story, no drift/rollback — reads as a demo. Volunteer the cost estimate and the failure modes (cold start, KV fragmentation, freshness drift, permission leaks) *before* the interviewer probes. Finishing the whole spine beats a perfect retrieval layer with no monitoring story.

## Interview angles

The real design prompts, each with how a senior attacks it via the rubric:

- **Design ChatGPT / a Claude chat service.** Frame as interactive LLM serving. Lead with the two-number latency SLO (TTFT vs ITL), continuous batching + PagedAttention for throughput, prefix caching for the system prompt, KV-cache math, and autoscale-on-queue-depth with a min-replica buffer. Streaming makes perceived latency ≪ total.
- **Design a RAG system.** ingest → chunk → embed → **hybrid** retrieve → **rerank** → generate-with-citations → eval. Lead **evaluation-first** (RAGAS + LLM-as-judge, no ground truth). Hybrid beats pure-vector 15–30%; the reranker is the highest-ROI component.
- **Design LLM inference at scale.** See worked design 2 — quantization (FP8), continuous batching (8–23×), KV cache + PagedAttention, prefix caching, autoscale on queue depth, cold-start buffer. Give a $/1k-token number.
- **Design an agent (ticket triage / research).** Planner/executor, function calling, memory with decay, **trace-based eval**, confidence-gated human-in-the-loop, tool-call validation, observability. Cascade the easy tail to a cheap model.
- **Hallucination-free banking chatbot.** There's no "hallucination-free" — reframe as *bounded* hallucination. Mandatory citations (every claim traces to a source), a refusal path for anything not grounded, faithfulness measured separately from retrieval recall, strict PII/compliance guardrails + audit log, and a human-escalation threshold. Answer "can we ship at 5%?" in *business* terms.
- **Candidate-sourcing over 750M profiles, semantic search, <500ms.** This is a retrieval/latency problem, not generation. Frame the latency budget: ANN index (Faiss/ScaNN/managed) sharded across nodes, pre-computed embeddings, hybrid filter + vector, a cheap rerank on the top slice — budget must sum to <500ms with the index search dominating. Estimate index RAM: 750M × dim × bytes.
- **"How would you open?"** Clarify target `Y`, scale, latency SLO, quality bar — *then* draw. Say the numbers you're assuming and confirm them.
- **"How do you handle a 10× / Black-Friday spike?"** Min-replica buffer pre-warmed to peak, queue-depth autoscaling, shadow any model upgrade, and **degrade gracefully** — serve a cheaper model or a queued response rather than time out. Never "just autoscale it."

## 📚 Resources

- 📰 [Generative AI System Design Interview](https://igotanoffer.com/en/advice/generative-ai-system-design-interview) — IGotAnOffer, 2026. Sample GenAI design prompts plus the rubric interviewers grade against; the closest thing to an answer key for this round.
- 💻 [alirezadir/machine-learning-interviews](https://github.com/alirezadir/machine-learning-interviews) — Apache-2.0, ~8.4k★. The canonical ML system-design taxonomy and 9-step template; classic-ML heavy, so pair it with the LLM/RAG material here.
- 📘 [Chip Huyen, *AI Engineering*](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) — O'Reilly, 2025. The single most-cited book: foundations → prompt → RAG → agents → fine-tune → latency/cost. The backbone for stages 3–6 of the rubric.
- 📰 [HelloInterview — ML / GenAI system design](https://www.hellointerview.com/) — 2026. Worked ML/GenAI design walkthroughs and the breadth/depth-by-level rubric (80/20 → 40/60).
- 📰 [Eugene Yan](https://eugeneyan.com/) — ongoing. Production ML/LLM design patterns, the offline/online 2×2, and eval-without-ground-truth writing from someone who ships.
- 📰 [IGotAnOffer — OpenAI interview process](https://igotanoffer.com/en/advice/openai-interview-process) — 2026. Shows where the AI System Design round (4a) sits in the onsite loop and what each round weights.
- 📄 [The Tail at Scale](https://research.google/pubs/the-tail-at-scale/) — Dean & Barroso, Google, 2013. The foundational latency-budget + tail-tolerance (hedged requests) reasoning behind the serving/scaling stages.
- 💻 [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) — Apache-2.0, ~30k★. 100+ reference architectures (RAG, agents) to sanity-check the shape of your design against what people actually ship.

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [System design rubric](../answers/system-design-rubric.md) · [System design questions](../questions/system-design.md)
