# The AI System-Design Rubric (2026)

> One reusable spine for the 45–60 min AI system-design round — applied across every worked answer in this folder.
> Maintained by [Landed](https://landed.jobs) — scout AI-native roles, get **referred**, and run company-specific mock interviews.

**Answer-first:** the AI system-design round is *not* a modeling quiz. It tests whether you can take a vague business goal and ride a fixed spine — **problem framing → data & eval set → retrieval/model choice → serving & latency → eval & guardrails → monitoring & cost → scaling** — under a clock, naming a **tradeoff and a number** at every box. HelloInterview calls the field "the wild west" precisely because the *framework* beats memorising any one company's answer. This page is that framework. Every design in `answers/` walks these seven stages in order.

> ⭐ **Star this repo** if this rubric saves your onsite.

```mermaid
flowchart LR
    A[1 · Problem framing] --> B[2 · Data & eval set]
    B --> C[3 · Retrieval / model choice]
    C --> D[4 · Serving & latency]
    D --> E[5 · Eval & guardrails]
    E --> F[6 · Monitoring & cost]
    F --> G[7 · Scaling]
    style A fill:#6C2BD9,color:#fff
    style E fill:#6C2BD9,color:#fff
    style G fill:#00A86B,color:#fff
```

---

## Why these seven stages

Every credible framework (Chip Huyen's *Designing ML Systems*, alirezadir's 9-step template, HelloInterview's rubric) collapses to the same spine. **The spine is uncontroversial** — interviewers are not waiting for you to discover it, they are watching whether you can *walk* it without skipping framing, without starting at the model, and without going silent on failure. Seniority shows up as knowing which stages to *abbreviate*, not covering all seven exhaustively. The breadth/depth dial by level: **~80/20 for mid, ~60/40 for senior, ~40/60 for Staff+**.

The single biggest predictor of a hire is **breadth across all seven** with a number on each. Weak candidates silently lose the round in stages 1–2 (framing + data) by minute 10 — before modeling even starts.

---

## Stage 1 — Problem framing

**What a senior does:** locks the target function in the first ~10 minutes, restates the assumed numbers the prompt left silent, and confirms them. Never opens with "I'd use a two-tower DNN with a cross-encoder reranker" — that signals you optimise components, not outcomes.

**Clarifying questions to ask** (scope / scale / freshness / tenancy / stakes):

| Axis | Ask | Why it changes the design |
|------|-----|---------------------------|
| **Scope** | What's the single user-visible task? Retrieval-augmented answer, ranking, classification, generation? | Maps to a known ML objective; wrong `Y` poisons everything downstream. |
| **Scale** | QPS (avg + peak), corpus size (docs/vectors/profiles), tokens/request? | Sets index choice, RAM, GPU count, cost. |
| **Freshness** | How stale can data be — seconds, minutes, nightly? | Batch vs streaming ingestion; re-index cadence. |
| **Tenancy** | Multi-tenant? Per-user ACLs? PII / regulated data? | Metadata filtering, row-level security, data residency. |
| **Stakes** | What's the cost of a wrong answer — annoyance, dollars, legal? | Sets the guardrail budget and the human-in-the-loop band. |
| **Latency** | p50 and p99 budget end-to-end? | Converts "how fast?" into a per-stage millisecond budget *before* picking a model. |

**Numbers to state:** the assumed QPS (e.g. "10M requests/day ≈ 116 avg / ~350 peak QPS"), the p99 budget (e.g. 500 ms), the corpus size, the freshness window. **Say the number you're assuming and confirm it** — that single move separates an AI-engineer candidate from an SWE one.

> [!WARNING]
> **Trap — model-first.** Opening with an architecture before the target function, metric, or scale is the fastest way a strong-on-paper candidate underwhelms. Frame first, metric-tied-to-a-business-unit second, architecture third.

---

## Stage 2 — Data & eval set

**What a senior does:** names where the label/eval signal comes from, its delay and noise, and **builds the golden eval set before the system** — "eval is the new system design." For LLM apps there are rarely ground-truth labels, so you design the eval set explicitly: 50–200 hand-curated (query, expected-answer, expected-sources) triples, versioned in git, expanded from production traces.

**Clarifying questions:**
- Where does the eval signal come from — human annotation, user feedback (thumbs), implicit signals, synthetic generation?
- What's the delay and noise on that signal? (A click ≠ satisfaction; "no complaint" ≠ correct.)
- Do we have a golden set? How big, who owns it, how do we grow it?
- What are the failure classes we care about — wrong answer, missing citation, refusal, latency, cost?

**Numbers to state:** golden set size (**start 50–100, grow to 500+**), the metric per failure class, the annotation budget. For retrieval systems: recall@k target (e.g. **recall@20 ≥ 0.95**). For generation: faithfulness / groundedness target (e.g. **≥ 0.9**).

> [!WARNING]
> **Trap — "we have labels."** For LLM/RAG systems there usually are no clean labels, and evaluation without ground truth has the weakest industrial conventions — this is where candidates improvise badly. The senior move is to *design the eval set on the whiteboard* (golden triples + LLM-as-judge with a rubric + human spot-checks) before drawing the pipeline.

---

## Stage 3 — Retrieval / model choice

**What a senior does:** proposes a **baseline first** (BM25, popularity, a single prompt) so the fancy system has a bar to beat, then defends the *simplest* architecture that hits the metric. For RAG: chunking → embedding → index → hybrid → rerank, each choice with a number. For generation: the cheapest model that clears the quality bar, with routing.

**Clarifying questions:**
- Is this even an LLM/ML problem, or does a rule/heuristic win? (ML must beat the best heuristic by ~3× on an offline metric.)
- Long-context stuffing vs retrieval? (Retrieval + rerank usually beats a giant-context call at ~20% of the cost.)
- What embedding dimension / index / rerank depth?

**Numbers to state:**

| Decision | State this |
|----------|-----------|
| Chunking | ~200–500 tokens, 10–15% overlap; parent-child or contextual |
| Embedding dim | 384 / 768 / 1024 / 1536 / 3072 — drives RAM |
| Index | HNSW (best recall, most RAM) · IVF-PQ (billions on commodity RAM) · flat (<1M) · DiskANN (SSD tail) |
| RAM math | `vectors × dim × 4 bytes` for fp32; ÷4 with int8, ÷32 with PQ |
| Hybrid | dense + BM25 fused via RRF; contextual retrieval cuts retrieval failures **35–49%** |
| Rerank | retrieve 100–150 → cross-encoder rerank → top 5–20 |

> [!WARNING]
> **Trap — no baseline.** Jumping to a fine-tuned model or a 7-stage RAG pipeline without a BM25/popularity/single-prompt baseline means you can't say what the complexity *buys*. Ship the baseline in a day; layer sophistication on the residual.

---

## Stage 4 — Serving & latency

**What a senior does:** breaks the end-to-end p99 into a **per-stage budget that sums to the SLO**, and names the lever that moves each stage. For LLMs, splits the budget into **TTFT (prefill, compute-bound)** and **ITL (decode, memory-bandwidth-bound over the KV cache)** — different levers.

**Clarifying questions:**
- Synchronous (chat), async (queue), batch (nightly), or streaming?
- What's the p99 budget per stage — retrieve, rerank, generate, guardrail?
- Self-hosted or API? (Drives the GPU-economics vs per-token-price tradeoff.)

**Numbers to state** — a worked 500 ms budget:

```
500 ms p99 =  embed query 15ms + ANN 20ms + rerank 60ms
            + LLM TTFT 300ms + guardrails 40ms + network/buffer 65ms
```

- Continuous batching + PagedAttention: **8–23× throughput** over naive batching; KV waste **60–80% → <4%**.
- Prefix caching on repeated prompts: **~90% cost / ~85% latency** on the cached prefix (exact-match — stable part first, user turn last).
- A bigger GPU helps prefill + concurrency, **not** per-token ITL (that's bandwidth-bound over the KV cache).

> [!WARNING]
> **Trap — one latency number.** "It'll be under 500 ms" with no per-stage breakdown is a bucket-6 fail. And "autoscale it" without naming *queue-depth-on-latency* (never GPU memory, which vLLM preallocates) signals you've never run inference under load.

---

## Stage 5 — Eval & guardrails

**What a senior does:** ties the offline eval to an **online metric + a counter-metric**, and layers guardrails as an explicit input/output stage in the diagram. Treats offline metrics as a **gate, not the verdict**.

**Clarifying questions:**
- What's the online metric (deflection, CSAT, task-completion) and its counter-metric (escalation rate, false-refusal, hallucination rate)?
- What must the system *never* do? (Leak PII, give financial/medical advice, follow injected instructions.)
- Input guardrails (prompt-injection, PII, off-topic) vs output guardrails (groundedness, toxicity, schema)?

**Numbers to state:** faithfulness / groundedness ≥ 0.9; hallucination rate < X%; tool-calling accuracy for agents; the A/B plan ("canary 1–5%, MDE 1.5%, run 14 days"). Guardrail latency budget (~30–50 ms). For agents: tool-selection accuracy, task-completion rate, trace-based (not just end-state) eval.

> [!WARNING]
> **Trap — offline AUC as proof.** Offline metrics measure rank-ordering on a frozen set; they do not measure the business outcome. Name an online metric and an A/B plan, and treat offline scores as a gate. For agents, evaluating only the **end state** misses a broken trajectory that happened to land — grade the trace.

---

## Stage 6 — Monitoring & cost

**What a senior does:** monitors the **online business KPI directly** plus label-free drift tripwires, and puts a **dollars-per-1k-requests** number on the design. "Watch accuracy on a hold-out" is the canonical *wrong* answer — in production you rarely have fresh labels.

**Clarifying questions:**
- What's the business KPI, and what proxy signals fire before labels arrive (prediction-distribution drift, calibration, staleness, cost per request)?
- Rollout order (shadow → canary → full) and rollback layers (model / config / feature)?
- What's the cost budget per request / per month?

**Numbers to state** — back-of-envelope cost:
```
cost/req = (input_tokens × in_price) + (output_tokens × out_price ~3–5× in)
monthly  = cost/req × req/day × 30
```
- PSI is the production drift standard (**>0.2 → retrain**); KS is too sensitive at large N.
- Prefix-cache savings: steady-state GPU util 70% → 25%, one node serves 2–3× the volume.

> [!WARNING]
> **Trap — hold-out accuracy.** A plan that hinges on hold-out accuracy signals you've never operated a model. Lead with the business KPI, add drift tripwires (PSI, calibration), run a shadow challenger, and keep a three-layer versioned rollback — and note the online feature/cache store doesn't clear atomically.

---

## Stage 7 — Scaling

**What a senior does:** names what breaks first at 10× and the mitigation. Sharding, replication, index partitioning, min-replica GPU buffers, multi-region, graceful degradation ("serve a cheaper model rather than time out").

**Clarifying questions:**
- What breaks first at 10× traffic — GPU queue, vector index RAM, reranker, the DB?
- Single-region or multi-region? (Single-region = total-downtime risk on a datacenter outage.)
- How do you degrade under a spike rather than fall over?

**Numbers to state:** shard count (index partitioned by tenant/hash), replica count for QPS, min-replica buffer for cold starts (**8×H100 vLLM = tens of seconds; serverless seen at 460 s**), autoscale on queue depth (target 3–5).

> [!WARNING]
> **Trap — happy-path only.** Designing only the happy path and going silent on failure loses points. Pre-empt the 10× spike, the hot shard, the cold start, and the graceful-degradation story *before* the interviewer asks.

---

## How to use this rubric in a 45–60 min round

Budget the clock, check in after each phase ("does this match what you're looking for, or should I drill in?"), and **finish the spine** — completion beats a perfect single stage.

```
60-MIN BUDGET (compress proportionally for 45)
STAGE                     MIN     LOCK THIS                         SKIP THIS
------------------------  ------  --------------------------------  ------------------
1 Framing + eval set      0–12    target task, metric, scale, p99   business essays
2 Data & eval set         12–18   golden set, failure classes       table schemas
3 Retrieval / model       18–32   baseline → index → hybrid → rerank naming every tool
4 Serving & latency       32–42   per-stage ms budget, GPU/routing  k8s manifests
5 Eval & guardrails       42–50   online metric + counter + A/B     metric derivations
6 Monitor & cost          50–56   business KPI, $/1k, drift, rollout re-explaining diagram
7 Scaling + wrap          56–60   what breaks at 10×, 3 tradeoffs   —
```

Three signals that land at the senior bar every time: you **name a baseline by minute ~18**, you **express accuracy in business units** (deflection %, chargeback-$, faithfulness) not log-loss, and you **volunteer a counter-metric** unprompted.

---

## Scoring signal — junior vs senior

| Stage | Junior answer (loses points) | Senior answer (hire signal) |
|-------|------------------------------|-----------------------------|
| **Framing** | "I'll build a RAG system" and starts drawing | Restates task + assumes QPS/p99/corpus/tenancy numbers and confirms |
| **Eval set** | "We'll measure accuracy" | Designs 50–200 golden triples + LLM-judge rubric + human spot-check *first* |
| **Model** | Jumps to fine-tuning or GPT-4 for everything | Baseline (BM25) first; simplest thing that hits the metric; routes cheap↔smart |
| **Index** | "Use a vector DB" | HNSW vs IVF-PQ with RAM math; hybrid + rerank with the 35–49% number |
| **Serving** | "It'll be fast, under 500ms" | Per-stage budget summing to SLO; TTFT vs ITL levers; prefix caching |
| **Guardrails** | "Add a content filter" | Input+output guardrails as a stage; groundedness ≥0.9; injection defense; A/B plan |
| **Monitoring** | "Watch accuracy on a hold-out" | Business KPI + PSI drift + shadow challenger + 3-layer rollback |
| **Cost** | Doesn't mention cost | $/1k-req back-of-envelope + prefix-cache + routing savings |
| **Scaling** | "Add more servers" | Names what breaks at 10×, shards the index, min-replica buffer, graceful degradation |
| **Communication** | Silent, waits to be prompted deeper | Checks in after each phase, verbalises the clock, pre-empts failure modes |

> [!TIP]
> In close calls, **communication and proactive framing routinely outvote raw technical depth** — the consistent lesson from graded interview debriefs. The 4/4-communication candidate gets the offer the 4/4-technical rambler misses.

---

### The worked designs (all use this rubric)

| Design | Provenance |
|--------|-----------|
| [RAG over 10M docs](rag-over-10m-docs.md) | 🔮 Representative (prompt B) |
| [Agentic workflow](agentic-workflow.md) | 🔮 Representative (prompt D) |
| [Semantic search](semantic-search.md) | 🔮 Representative |
| [Content moderation](content-moderation.md) | 🔮 Representative |
| [Eval pipeline](eval-pipeline.md) | 🔮 Representative ("eval is the new system design") |
| [Coding agent](coding-agent.md) | 🔮 Representative |
| [Support bot (RAG)](support-bot.md) | ✅ Reported (prompt G) |
| [LLM inference at scale](llm-inference-at-scale.md) | ✅ Reported (prompt C) |
| [AI candidate sourcing — 750M profiles](ai-candidate-sourcing.md) | ✅ Reported (prompt J) |
| [Hallucination-free banking chatbot](hallucination-free-banking-chatbot.md) | ✅ Reported (prompt I) |

---

<div align="center">

**Nav:** [← README](../README.md) · [This rubric](system-design-rubric.md)

<sub>Maintained by [Landed](https://landed.jobs) · No affiliation with the companies named. Content MIT-licensed. Updated 2026-07.</sub>

</div>
