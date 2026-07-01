# System Design — ML & LLM System Design + The 10 Design Prompts

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/system-design.md`](../content/07-ml-and-llm-system-design.md) · Worked answers: [`answers/`](../answers/) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **40 items** (30 self-test checkpoints + 10 design prompts). The signature onsite round (60 min, heavily graded). The spine that survives every prompt: **framing first** (business goal → target function → success metric + counter-metric → scale + latency), then data/features → model → serving/latency → eval/guardrails → monitoring/cost → scaling. The reusable rubric lives in [`content/system-design.md`](../content/07-ml-and-llm-system-design.md); the 8–15 worked designs live in [`answers/`](../answers/).

---

### How to use this file

Sections A–D are self-test checkpoints on the design *muscles* (framing, data/features, serving, monitoring, funnel/fraud patterns, enterprise RAG). Section E is the **10 concrete design prompts** you'll actually get on a whiteboard — each with a "how a senior frames it" pointer into [`answers/`](../answers/).

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (the graded tradeoff, leakage, scale)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — Framing & the interview spine `[ml-system-design] msd1`

### Q1. An interviewer says "design a system to recommend products." You have 45 minutes. What's the strongest first move?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd1` · 🔮 Representative

- Sketch a two-tower retrieval model and a DCN ranker, then explain the embeddings
- Ask for the catalog size and QPS so you can pick an ANN index
- ✅ Clarify the business goal, lock the target function and success metric (plus a counter-metric), and confirm scale and latency — before drawing anything
- Propose a popularity baseline and start coding it

<details><summary>Show answer & why each option</summary>

- **Sketch the model** — This is Sin 2 — starting at the model. Every architecture choice is ungrounded until you know what you're optimising, for whom, at what scale.
- **Ask catalog size/QPS** — Scale matters, but a single infra question isn't framing; you still haven't locked the target function or metric.
- ✅ **Frame first** — The non-skippable opening move. The target Y and metric determine labels, model, and serving budget; getting them wrong poisons the whole design.
- **Popularity baseline** — The right *second* step — after framing, so you know what the baseline must beat.
</details>

### Q2. Your ranker lifts offline NDCG@10 by 4 points. The interviewer asks, "how do you know this helps the business?" Which response is the WEAKEST (the one to avoid)?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd1` · 🔮 Representative

- ✅ Claim that NDCG@10 is the gold-standard ranking metric, so a 4-point lift is a clear win on its own
- Ship to 1–5% behind a canary and measure the online metric against an MDE over ~2 weeks
- Watch a counter-metric like diversity alongside the primary online metric during the A/B
- Run the new ranker in shadow first to catch latency or distribution regressions

<details><summary>Show answer & why each option (the ✅ here marks the WEAKEST answer — the trap)</summary>

- ✅ **NDCG is gold-standard → clear win** — This is the weakest answer and the trap. Offline NDCG is a *gate, not the verdict* — it can rise while engagement falls. Treating it as proof of business value is the offline/online-gap mistake.
- **Canary + MDE** — A strong answer — ties the offline lift to a real online measurement.
- **Counter-metric** — A senior move, not a red flag.
- **Shadow first** — Sound operational hygiene.
</details>

### Q3. You're interviewing at the Staff+ level. With 45 minutes, how should you allocate depth vs breadth compared to a mid-level candidate?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd1` · 🔮 Representative

- Same as mid-level — cover every stage of the spine equally
- ✅ Spend more time on the hardest tradeoff (≈40/60 breadth/depth), brushing past commodity pieces like caches and regional replication
- Skip framing entirely since a Staff engineer is expected to know the objective
- Maximise breadth and avoid going deep, to demonstrate range

<details><summary>Show answer & why each option</summary>

- **Equal coverage** — The mid-level (~80/20) profile. Staff+ is graded on nuance; equal coverage reads as not knowing what to prioritise.
- ✅ **~40/60 depth, brush past commodity** — HelloInterview's level rubric runs ~40/60 for Staff+. Going deep on the genuinely hard tradeoff is the senior signal.
- **Skip framing** — Never skippable — that's Sin 1. Staff+ still states the target function.
- **Maximise breadth** — The inverse of the Staff+ profile.
</details>

### Q4. Halfway through, you've spent 20 of 45 minutes on feature engineering and haven't touched serving or monitoring. What's the right recovery?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd1` · 🔮 Representative

- Keep going on features — depth there shows expertise
- Quietly skip serving and hope the interviewer doesn't notice
- Restart the design with a tighter scope
- ✅ Say "let me move on to serving and monitoring to keep us end-to-end," summarise features in one line, and budget the remaining time deliberately

<details><summary>Show answer & why each option</summary>

- **Keep going** — Exactly how candidates "fail to complete the design." Over-investing leaves the interviewer without serving/monitoring signal.
- **Quietly skip** — Worse than abbreviating — reads as a blind spot.
- **Restart** — Burns the time you don't have.
- ✅ **Verbalise the clock and move on** — The senior move: finishing the spine beats a perfect features section with no serving or monitoring story.
</details>

### Q5. The prompt is "detect spam in comments." Volume is low and a keyword blocklist already catches most of it. Strongest senior response?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd1` · 🔮 Representative

- Design a fine-tuned transformer classifier immediately — spam detection is a classic ML task
- ✅ Note that ML is only justified if it beats the keyword baseline by a meaningful margin on an offline metric; propose shipping the rule now and layering ML on the residual tail
- Refuse the problem because rules are sufficient
- Use an unsupervised anomaly detector since labels are scarce

<details><summary>Show answer & why each option</summary>

- **Heavy model immediately** — Ignores the cost/impact tradeoff and skips the "is this even ML?" check.
- ✅ **Rules-first floor, ML on the tail** — The decision rule is to compare ML against the best heuristic; a rules floor with ML on the long tail is the canonically correct shape.
- **Refuse** — Over-corrects — ML can catch the adaptive tail the blocklist won't.
- **Unsupervised** — Possible later, but it skips the framing step.
</details>

---

## Section B — Labels, features & the feature store `[ml-system-design] msd2`

### Q6. You're designing a recommendation feed and propose training on "click = positive, no-click = negative." The interviewer pushes back. Strongest revision?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd2` · 🔮 Representative

- ✅ Predict a weighted multi-reaction label (e.g. watch-time or like/comment/share weighted, with hide/block as negatives), because clicks reward clickbait and don't capture satisfaction
- Keep clicks but raise the classification threshold to reduce false positives
- Use an unsupervised model so you don't need labels at all
- Add more features to compensate for the noisy click label

<details><summary>Show answer & why each option</summary>

- ✅ **Weighted multi-reaction label** — YouTube's exact reasoning — they predict expected watch time, not click probability. Captures real satisfaction and lets the model learn what to suppress.
- **Raise threshold** — Doesn't fix that the label itself is a poor proxy.
- **Unsupervised** — You still need a target aligned with the goal.
- **More features** — No feature set rescues a misaligned label.
</details>

### Q7. Your fraud model has 0.99 offline AUC but performs poorly in production. Most likely root cause given fraud's label dynamics?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd2` · 🔮 Representative

- The model is underfit and needs more parameters
- ✅ You treated "no chargeback yet" as a confirmed negative; chargebacks arrive 30–120 days later, so recent fraud is mislabeled as legitimate, and a leaked or future-looking feature inflated AUC
- The online store has higher latency than the offline store
- Fraudsters changed tactics overnight

<details><summary>Show answer & why each option</summary>

- **Underfit** — 0.99 AUC is the opposite of underfit — a near-perfect offline score on a hard problem is itself the warning sign.
- ✅ **Label censoring + leakage** — Fraud labels are delayed and censored. Treating unmatured transactions as negatives, plus a future-looking feature, produces a beautiful offline number that collapses.
- **Store latency** — Affects serving speed, not offline-vs-online accuracy.
- **Tactics overnight** — Concept drift wouldn't produce a 0.99 AUC that fails immediately.
</details>

### Q8. The interviewer asks why you'd put a feature store between your pipelines and your model. Strongest answer?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd2` · 🔮 Representative

- It's a faster database for features
- It lets you store embeddings and serve them as a vector database
- ✅ It enforces point-in-time correctness at training and freshness at serving — eliminating training-serving skew — and lets teams reuse features across models
- It removes the need for a streaming pipeline

<details><summary>Show answer & why each option</summary>

- **Faster DB** — Speed is one property but misses the core reason.
- **Vector DB** — A feature store is explicitly *not* a vector database; embeddings belong in a dedicated ANN index alongside it.
- ✅ **Point-in-time correctness + freshness + reuse** — Train/serve consistency (no skew) is exactly why DoorDash, Uber, and Netflix converged on feature stores.
- **Removes streaming** — Streaming jobs materialise fresh features *into* the store; they're complementary.
</details>

### Q9. You need a "number of distinct countries this card was used in over the past hour" feature for fraud scoring. Which pipeline is right, and why?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd2` · 🔮 Representative

- A nightly batch job, because batch is cheaper and simpler
- Precompute it once at signup and cache it
- ✅ A streaming job (Flink/Spark Streaming) that joins recent transaction events with point-in-time-correct windowed aggregation and writes to the online store
- Compute it synchronously inside the model server on every request by querying the transaction DB

<details><summary>Show answer & why each option</summary>

- **Nightly batch** — Makes the feature up to 24 hours stale — useless for an in-progress attack.
- **At signup** — A velocity feature changes continuously; a static signup value is meaningless.
- ✅ **Streaming + windowed aggregation → online store** — Velocity features have a seconds-to-minutes window, so compute in streaming and materialise for sub-100ms reads.
- **Synchronous live join** — Blows the latency budget under burst; pre-materialise via streaming.
</details>

### Q10. A teammate wants to ship a new feature purely because it lifts held-out AUC by 6 points. What should you check first?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd2` · 🔮 Representative

- ✅ Whether the feature is available — with that exact value — at prediction time, i.e. that it isn't leakage from the future or the label
- Whether the model needs regularisation to handle the new feature
- Whether the feature improves training speed
- Whether it increases model size

<details><summary>Show answer & why each option</summary>

- ✅ **Leakage check** — A large, surprising offline lift is the classic leakage signature. Verify the feature is computable at serving time and doesn't encode future/label information.
- **Regularisation** — Comes later; first confirm the lift is real.
- **Training speed** — Irrelevant to whether the lift is trustworthy.
- **Model size** — A serving concern, not why a 6-point lift should be suspect.
</details>

---

## Section C — LLM serving & latency `[ml-system-design] msd3`

### Q11. Your interactive LLM chat has acceptable TTFT but users complain text "types out" too slowly. Which change targets the right bottleneck?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd3` · 🔮 Representative

- Shrink or prefix-cache the prompt to speed up prefill
- Add more replicas behind the load balancer
- ✅ Reduce the KV-cache footprint — quantize, use a smaller model, or speculative decoding — because ITL is memory-bandwidth-bound over the KV cache
- Increase the context window so the model has more room

<details><summary>Show answer & why each option</summary>

- **Speed up prefill** — That improves TTFT (the wait before the first token), but the complaint is per-token speed after the first — an ITL problem.
- **More replicas** — Raise concurrency/throughput but don't lower a single user's per-token latency.
- ✅ **Reduce KV-cache footprint** — Inter-token latency is bounded by reading the whole KV cache per token; shrinking it is the lever.
- **Larger window** — Makes the KV cache bigger, worsening ITL.
</details>

### Q12. Your self-hosted LLM cluster starts rejecting requests with OOM errors under load, but `nvidia-smi` shows several GB free per GPU. Most likely cause and fix?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd3` · 🔮 Representative

- The model weights are too large; switch to a smaller model
- ✅ KV-cache fragmentation — naive serving reserves contiguous max-length blocks per request, leaving unusable holes; switch to PagedAttention (vLLM/SGLang/TGI)
- A memory leak in the tokenizer; restart the pods on a schedule
- Insufficient replicas; scale on GPU memory utilization

<details><summary>Show answer & why each option</summary>

- **Weights too large** — Weights are resident and fixed; the OOM appears only under concurrent load, pointing at the KV cache.
- ✅ **KV-cache fragmentation → PagedAttention** — The canonical "OOM with free memory" pattern. Paged attention allocates KV in fixed-size pages on demand, cutting waste from 60–80% to under 4%.
- **Tokenizer leak** — Scheduled restarts mask the real cause.
- **Scale on GPU memory** — Explicitly the wrong signal for preallocated vLLM/TGI; doesn't address fragmentation within a replica.
</details>

### Q13. A RAG chat assistant reuses a 1,500-token system prompt and retrieved context on every call. How do you cut cost and latency most effectively?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd3` · 🔮 Representative

- Switch to a smaller model for everything
- Lower the temperature to reduce token generation
- ✅ Engineer the prompt so the stable prefix (system + retrieved context) comes first and the variable user turn last, then enable prefix caching for ~90% cost / ~85% latency on the prefix
- Increase max output tokens so fewer follow-up calls are needed

<details><summary>Show answer & why each option</summary>

- **Smaller model** — Trades quality globally and ignores the biggest lever for a repeated-prefix workload.
- **Lower temperature** — Controls sampling, not token count or prefill cost.
- ✅ **Stable-first ordering + prefix caching** — The highest-ROI lever for repeated-prefix workloads, but it requires an exact prefix match — so order the template stable-first, variable-last.
- **Raise max output** — Output is the most expensive part; raising the cap increases cost and latency.
</details>

### Q14. You're configuring autoscaling for a vLLM deployment on GKE. Which scaling signal should you use?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd3` · 🔮 Representative

- GPU memory utilization (`DCGM_FI_DEV_FB_USED`)
- GPU duty cycle / utilization
- ✅ Server queue depth (or batch size) tied to a target latency, starting the queue target around 3–5 and tuning to hit the SLO
- Requests per second with a fixed replica count

<details><summary>Show answer & why each option</summary>

- **GPU memory** — vLLM preallocates the KV cache, so memory-used stays high — "only works for scaling up, won't scale down."
- **Duty cycle** — Measures how long the GPU is active, not how much work is queued.
- ✅ **Queue depth / batch size vs latency target** — Google's GKE guidance is explicit: scale on queue depth or batch size against a latency target. The citable senior answer.
- **RPS + fixed replicas** — A fixed replica count isn't autoscaling, and raw RPS ignores variable prompt lengths.
</details>

### Q15. "To halve per-token decode latency, would buying H200s instead of H100s help?" Strongest answer?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd3` · 🔮 Representative

- Yes — a faster GPU always lowers latency proportionally
- ✅ Only partially — a bigger/faster GPU mainly helps prefill and concurrency; ITL is memory-bandwidth-bound over the KV cache, so the bigger lever is shrinking the KV footprint (quantization, smaller model, speculative decoding)
- No — GPU choice never affects latency
- Yes — bigger memory means a smaller KV cache

<details><summary>Show answer & why each option</summary>

- **Always proportional** — Per-token decode is bounded by memory bandwidth, not raw compute.
- ✅ **Only partially — ITL is bandwidth-bound** — Decode reads the entire KV cache per token; more compute helps TTFT and concurrency, but cutting ITL means reducing KV size or speculative decoding.
- **Never affects** — Overcorrecting: GPU choice does affect prefill and concurrency.
- **Bigger memory → smaller KV** — More memory holds a *larger* cache; it doesn't shrink the per-token read.
</details>

---

## Section D — Monitoring, rollout & the funnel/fraud patterns `[ml-system-design] msd4–msd6`

### Q16. An interviewer asks how you'd monitor a deployed ranking model in production. Strongest answer?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd4` · 🔮 Representative

- ✅ Track the online business KPI directly (e.g. engagement/revenue) plus ML drift signals — prediction- and feature-distribution drift, calibration, staleness — and run a shadow challenger; treat labels as lagging confirmation
- Compute accuracy on a held-out test set every night
- Alert only when the model server's error rate spikes
- Re-run offline NDCG weekly and ship if it's stable

<details><summary>Show answer & why each option</summary>

- ✅ **KPI + label-free drift tripwires + shadow challenger** — In production you rarely have fresh labels; failure often shows as a KPI regression with unchanged inputs. The senior monitoring story.
- **Nightly hold-out** — A frozen snapshot of the past; you usually lack fresh production labels.
- **Server error rate** — Catches infra failures, not quality regressions — the model can be available and quietly wrong.
- **Weekly offline NDCG** — Says nothing about whether the live distribution moved.
</details>

### Q17. You set up a Kolmogorov-Smirnov drift test on a feature with ~200,000 daily samples. It fires a drift alarm almost every day. What's going on and the fix?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd4` · 🔮 Representative

- The feature genuinely drifts daily; retrain every day
- ✅ KS is too sensitive at large N — it flags ~1% drift at n=100k+; switch to PSI (retrain only when >0.2) or widen the window / add a threshold
- Your sample size is too small; collect more data
- The test is broken; remove drift monitoring

<details><summary>Show answer & why each option</summary>

- **Retrain daily** — Overfitting-to-noise trap; the alarm sensitivity, not real drift, is the problem.
- ✅ **KS too sensitive at large N → PSI** — The Evidently study is explicit; PSI is the production standard for "major changes," stopping the false alarms while still catching real shifts.
- **Too small** — 200k is already large — that's the cause.
- **Remove monitoring** — The opposite of the fix.
</details>

### Q18. You want to validate a new fraud model on real traffic distributions without risking a single wrong customer decision. Which rollout step fits?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd4` · 🔮 Representative

- Canary it to 5% of live traffic immediately
- Full rollout with a fast rollback ready
- ✅ Shadow deployment — mirror production traffic to the new model and record its outputs without returning them to users
- Run it offline on last month's data again

<details><summary>Show answer & why each option</summary>

- **Canary 5%** — Serves the new model's decisions to real users — risks wrong decisions.
- **Full rollout** — Exposes everyone; a rollback can't un-decline a legitimately blocked customer.
- ✅ **Shadow deployment** — Validates against real distributions and latency with zero user risk. The correct first step before any canary.
- **Offline again** — You've already validated offline; the goal is real production distributions.
</details>

### Q19. You roll back a bad model by swapping the weights to the previous version, but predictions stay degraded for several minutes. Most likely cause?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd4` · 🔮 Representative

- The load balancer is still routing to old pods
- ✅ The online feature store still serves the new model's materialized feature values, which don't revert atomically — they linger until they TTL out or you force-flush
- The model weights didn't actually change
- GPU memory needs to be cleared

<details><summary>Show answer & why each option</summary>

- **Load balancer** — Routing issues cause inconsistent versions, not uniform multi-minute degradation after a clean swap.
- ✅ **Feature store holds materialized values** — Feature rollback is the slow, dangerous layer: the online store holds values that don't clear atomically, so a rolled-back model reads poisoned features until TTL or force-flush.
- **Weights didn't change** — You'd see behavior unchanged, not a transient that resolves after minutes.
- **GPU memory** — Nothing to do with stale materialized feature values.
</details>

### Q20. For a fraud model, the interviewer asks how often you'd retrain. Which answer shows the most judgment?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd4` · 🔮 Representative

- Continuously / hourly to stay maximally fresh
- ✅ Tie cadence to label maturation and drift rate — roughly monthly for fraud (newer data buys ~+0.5pp recall), with a drift alarm able to trigger an off-cycle retrain; faster isn't always better
- Once a year, since fraud models are stable
- Only when accuracy on a hold-out set drops

<details><summary>Show answer & why each option</summary>

- **Hourly** — Fraud labels (chargebacks) mature over 30–120 days; retraining hourly overfits to noise before labels confirm.
- ✅ **Tie cadence to label maturation + drift** — Cites the +0.5pp-from-newer-data result and the counterintuitive ceiling — the senior answer.
- **Once a year** — Far too slow for an adversarial domain.
- **Only on hold-out drop** — Fresh labels are delayed, so hold-out accuracy lags the incident.
</details>

### Q21. For a 100M-DAU video feed over a 10M-item catalog with a 120ms p99, why use a two-stage funnel rather than one big ranking model over the whole catalog?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd5` · 🔮 Representative

- One model is fine; just make it bigger
- ✅ Retrieval needs ANN over millions of items (a lightweight two-tower with precomputed embeddings narrows to ~500); ranking can then afford a heavy cross-feature DNN on that small set
- Two stages reduce model accuracy but save money
- It lets you skip embeddings entirely

<details><summary>Show answer & why each option</summary>

- **One big model** — Scoring a heavy cross-feature model against 10M items per request blows the latency budget by orders of magnitude.
- ✅ **Recall stage (ANN) + precision stage (DNN)** — The funnel splits a recall problem (cheap, over the full catalog) from a precision problem (expensive, over ~500 candidates). The canonical two-stage rationale.
- **Reduces accuracy** — Not a sacrifice — each stage is optimized for its job.
- **Skip embeddings** — The opposite — retrieval depends on precomputed item embeddings.
</details>

### Q22. Your recommendation feed maximizes CTR and engagement climbs, but the catalog collapses into the same popular items for everyone. Right fix and where?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd5` · 🔮 Representative

- Retrain the ranker on more data
- Switch the retrieval index from HNSW to IVF-PQ
- ✅ Add explicit diversity (and freshness/creator-boost) in the re-ranking layer, and track a filter-bubble counter-metric — the reranker is the cheapest place to fix it
- Lower the candidate-generation recall so fewer items compete

<details><summary>Show answer & why each option</summary>

- **More data** — Reinforces the popularity loop on a pure-relevance objective.
- **HNSW → IVF-PQ** — An index recall/memory tradeoff; nothing to do with the filter-bubble collapse.
- ✅ **Diversity in the re-ranker + counter-metric** — Pure-relevance rankers create filter bubbles; the reranker is the canonical, cheapest fix.
- **Lower recall** — Shrinks the candidate pool and worsens the collapse.
</details>

### Q23. Designing fraud detection, you're asked how to set the decision threshold. Strongest answer?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd5` · 🔮 Representative

- Pick the threshold that maximizes F1 on the validation set
- Use 0.5 as a neutral default
- Maximize recall to catch all fraud
- ✅ Set it at the break-even precision derived from chargeback economics (e.g. Stripe's ~5.07%), tuned per vertical since high-AOV merchants tolerate more false positives

<details><summary>Show answer & why each option</summary>

- **Maximize F1** — Weights precision and recall equally, but fraud costs are asymmetric and measured in dollars.
- **0.5 default** — Ignores the asymmetric cost and per-vertical economics.
- **Maximize recall** — Ignores false declines and the customer-friction cost.
- ✅ **Break-even precision per vertical** — The threshold is an economic decision: block when expected fraud loss exceeds the friction cost. Cites Stripe's worked number.
</details>

### Q24. An interviewer asks why Stripe Radar uses logistic regression, GBMs, and random forests rather than a large transformer. Best answer?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd5` · 🔮 Representative

- Transformers don't work on tabular data at all
- ✅ Fraud labels are delayed and censored (chargebacks arrive 30–120 days later), so over-parameterized models overfit to outdated patterns; a simpler ensemble plus rich network features generalizes better and retrains cheaply
- Random forests are always more accurate than neural nets
- Transformers are too slow to serve under 250ms

<details><summary>Show answer & why each option</summary>

- **Don't work on tabular** — They can be applied; the reason is label dynamics, not impossibility.
- ✅ **Label dynamics cap complexity** — The rate of label arrival caps useful model complexity; the model pack is deliberately simple, with network features doing the heavy lifting.
- **RF always more accurate** — Not a general truth.
- **Too slow** — Small transformers can serve fast; the binding issue is overfitting to stale labels.
</details>

### Q25. A new merchant joins your payments platform with zero transaction history. How do you score their transactions for fraud on day one?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd5` · 🔮 Representative

- Block all transactions until enough history accumulates
- Wait weeks to train a merchant-specific model before scoring anything
- ✅ Lean on network-level features — card history, BIN/IP risk, and prior fraud-cluster matches across the whole platform — which bootstrap a model with as little as ~100 events
- Use only the merchant's own data with a high default threshold

<details><summary>Show answer & why each option</summary>

- **Block all** — Destroys a legitimate new merchant's business.
- **Wait weeks** — Leaves the merchant unprotected; the network already has signal.
- ✅ **Network-level features** — The network moat: a card or pattern new to the merchant isn't new to the network. Bootstraps from ~100 events.
- **Own data + high threshold** — With zero own-data there's nothing to learn from; wastes the network signal.
</details>

### Q26. Your enterprise RAG assistant retrieves well on conceptual questions but fails on exact error codes and SKU IDs. What's the fix?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd6` · 🔮 Representative

- Switch to a larger embedding model
- ✅ Add BM25 lexical search alongside dense retrieval (hybrid search, fused via RRF) — BM25 nails exact-token matches like codes and IDs, dense handles paraphrase; hybrid beats either alone by 15–30%
- Increase the number of retrieved chunks to 200
- Lower the chunk size to 64 tokens

<details><summary>Show answer & why each option</summary>

- **Larger embedding** — A bigger dense embedding still matches on meaning, not exact tokens.
- ✅ **Hybrid BM25 + dense (RRF)** — Exact-token queries are precisely where dense fails and BM25 excels. The documented fix and senior default. (See [rag.md](./rag.md) Q11–Q15.)
- **200 chunks** — Doesn't help if none contain the exact token; bloats context.
- **64-token chunks** — Doesn't change semantic-vs-lexical matching.
</details>

### Q27. "How do you measure the accuracy of this RAG system when you have no labeled answers for 50M documents?" Strongest answer?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd6` · 🔮 Representative

- Report the average cosine similarity of retrieved chunks
- ✅ Use a reference-free eval stack — RAGAS faithfulness and context precision/recall offline, plus LLM-as-judge with a rubric and human spot-checks online — and track citation accuracy and hallucination rate against targets
- Wait until you can hand-label all 50M documents
- Trust the LLM since modern models rarely hallucinate

<details><summary>Show answer & why each option</summary>

- **Cosine similarity** — Measures retrieval closeness, not answer correctness.
- ✅ **Reference-free eval stack** — RAGAS + LLM-as-judge + human audit is how you measure RAG without labels, separating retrieval quality from generation faithfulness. (See [evals.md](./evals.md).)
- **Hand-label 50M** — Infeasible; the point is label-free eval.
- **Trust the LLM** — Assuming correctness is the failure the question probes.
</details>

### Q28. A teammate proposes skipping retrieval entirely and stuffing all relevant docs into a 200k-token context window per query. Why push back?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd6` · 🔮 Representative

- Large-context models don't exist yet
- It's more accurate but slower
- ✅ You pay for every token (attention is super-linear), quality degrades as the window fills (context rot / Lost-in-the-Middle), and a medium model + reranker often beats it at ~20% of the cost
- Citations become impossible with retrieval

<details><summary>Show answer & why each option</summary>

- **Don't exist** — They do; the objection is cost and quality.
- **More accurate but slower** — Usually neither more accurate nor cheaper — quality degrades as the window fills.
- ✅ **Super-linear cost + context rot** — The senior trap: expensive and lower-quality than narrowing with retrieval + reranking. Reserve big-context for holistic single-doc tasks.
- **Citations impossible** — The opposite — retrieval makes citations easy.
</details>

### Q29. Your assistant answers HR and finance questions for all employees from one shared index. A junior employee got an answer citing a restricted compensation doc. Root cause and fix?

> **Difficulty:** 🔴 Hard · `[ml-system-design] msd6` · 🔮 Representative

- The LLM leaked training data; switch models
- ✅ No ACL filtering at retrieval — the candidate set wasn't filtered by the user's document permissions before the LLM saw it; fix with per-tenant/permission metadata filters (or index isolation) applied at retrieval time, plus an audit log
- The reranker scored the restricted doc too high; lower its weight
- The chunk size was too large; reduce it

<details><summary>Show answer & why each option</summary>

- **Training-data leak** — The restricted content came from your own index via retrieval, not the model.
- ✅ **No ACL filtering at retrieval** — The canonical enterprise-RAG failure: filter the candidate set by the asking user's ACLs before generation. (See [rag.md](./rag.md) Q22, Q31.)
- **Reranker weight** — Irrelevant if the doc should never have been a candidate.
- **Chunk size** — Nothing to do with authorization.
</details>

### Q30. Users report the assistant confidently cites an outdated policy that was revised last month. What production gap does this reveal?

> **Difficulty:** 🟡 Medium · `[ml-system-design] msd6` · 🔮 Representative

- The LLM needs fine-tuning on the new policy
- ✅ A freshness / recall-drift gap — when documents change, the index and reranker silently decay; fix with incremental re-indexing keyed on chunk modification dates plus recall telemetry
- The hybrid search weights are wrong; retune RRF
- The context window is too small to hold the new policy

<details><summary>Show answer & why each option</summary>

- **Fine-tune** — Fine-tuning bakes facts into weights (stale the moment a doc changes) — the fix is re-indexing.
- ✅ **Freshness / recall-drift gap** — Stale citations are the classic freshness failure; incremental sync on modification dates + recall monitoring keeps grounding current. (See [rag.md](./rag.md) Q35.)
- **RRF weights** — Balances lexical vs semantic; doesn't make a stale index reflect an updated doc.
- **Window too small** — The new policy was never re-indexed, so it isn't retrieved regardless of window size.
</details>

---

## Section E — The 10 concrete design prompts (whiteboard)

> These are the actual prompts handed to candidates in 2026 loops. Each is **✅ Reported** with its source. For each, a 2–4 line "how a senior frames it" pointer — the full worked design (using the rubric in [`content/system-design.md`](../content/07-ml-and-llm-system-design.md)) lives in [`answers/`](../answers/). **Universal opener for all ten: frame first** (Q1) — business goal, target function, success + counter-metric, scale, latency — before you draw a box.

### Prompt A — Design ChatGPT
> **Difficulty:** 🔴 Hard · ✅ Reported ([dev.to system-design set](https://dev.to/arslan_ah/ai-system-design-interview-questions-chatgpt-rag-llm-inference-and-agents-1doi), [Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **How a senior frames it:** Multi-turn chat: model serving (continuous batching, PagedAttention, KV cache — Q11–Q14), context/conversation state, streaming UX (TTFT vs ITL), safety guardrails on both input channels, cost via prefix caching + routing, and eval/monitoring. The graded tradeoff is usually serving latency vs cost at scale. → [`answers/design-chatgpt.md`](../answers/)

### Prompt B — Design a RAG system (ingestion, retrieval, reranking, eval)
> **Difficulty:** 🔴 Hard · ✅ Reported ([dev.to](https://dev.to/arslan_ah/ai-system-design-interview-questions-chatgpt-rag-llm-inference-and-agents-1doi))
> **How a senior frames it:** Scope the shape/scale/freshness/tenancy first ([rag.md](./rag.md) Q33). Ingestion (parse → chunk → contextualize → embed → index), hybrid retrieval + RRF + rerank, ACL filtering at retrieval, grounded cited generation, and a split-eval gate on a production-derived golden set. → [`answers/design-rag.md`](../answers/)

### Prompt C — Design LLM inference at scale (quantization, batching, KV cache, vLLM, autoscaling)
> **Difficulty:** 🔴 Hard · ✅ Reported ([dev.to](https://dev.to/arslan_ah/ai-system-design-interview-questions-chatgpt-rag-llm-inference-and-agents-1doi))
> **How a senior frames it:** Precision choice from hardware (Q… / [fine-tuning.md](./fine-tuning.md) Q21), continuous batching + PagedAttention (Q12), KV-cache math, prefix caching, and autoscaling on queue depth not GPU memory (Q14). Separate TTFT (prefill) from ITL (bandwidth-bound decode, Q11, Q15). → [`answers/design-llm-inference.md`](../answers/)

### Prompt D — Design an agent (planner/executor, MCP, function-calling, memory, observability)
> **Difficulty:** 🔴 Hard · ✅ Reported ([Sundeep Teki cross-lab guide](https://www.sundeepteki.org/advice/the-ultimate-ai-research-engineer-interview-guide-cracking-openai-anthropic-google-deepmind-top-ai-labs))
> **How a senior frames it:** De-escalate to the simplest pattern ([agents.md](./agents.md) Q8); tool design as the real API; MCP with a gateway + security posture; durable state on disk (stateless model, stateful harness); span-level observability; break the lethal trifecta and gate writes ([evals.md](./evals.md) Q6–Q7). Grade on pass^k. → [`answers/design-agent.md`](../answers/)

### Prompt E — Design an AI tutoring platform with RAG-based Q&A
> **Difficulty:** 🟡 Medium · ✅ Reported ([HelloInterview / Oracle](https://www.hellointerview.com/community/questions/ai-tutoring-platform/cm75445ag00053b64gz80nijf))
> **How a senior frames it:** Per-student personalization, curriculum-grounded RAG with citations, safety for minors, evals on pedagogical quality (not just faithfulness), and cost at classroom scale. The tradeoff: grounding/accuracy vs conversational helpfulness. → [`answers/design-ai-tutor.md`](../answers/)

### Prompt F — Design a Claude chat service
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **How a senior frames it:** Like Prompt A but foreground safety/constitutional guardrails, long-context handling (map-reduce vs retrieval), tool-use/MCP, and multi-tenant isolation. Serving spine identical (Q11–Q14). → [`answers/design-claude-chat.md`](../answers/)

### Prompt G — Design a customer-support chatbot using RAG
> **Difficulty:** 🟡 Medium · ✅ Reported ([Anthropic customer-support cookbook pattern](https://platform.claude.com/cookbook))
> **How a senior frames it:** Routing workflow (answer / act / escalate — [agents.md](./agents.md) Q8), RAG over the KB with freshness (CDC), ACL/tenant isolation, deflection-rate + CSAT as the business metric, and human handoff. Fine-tune the voice/format, RAG the KB ([fine-tuning.md](./fine-tuning.md) Q26). → [`answers/design-support-bot.md`](../answers/)

### Prompt H — Design a real-time chatbot API
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **How a senior frames it:** SLA-driven: streaming, TTFT/ITL budgets, rate limiting + backoff (client reliability — [llmops.md](./llmops.md) Q1–Q4), hedging the tail, autoscaling, and graceful degradation/fallback with its own eval (Q5 there). → [`answers/design-realtime-chat-api.md`](../answers/)

### Prompt I — Design a hallucination-free banking chatbot
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **How a senior frames it:** "Hallucination-free" = grounding + refusal discipline, not a promise. Strict RAG with cited spans + span-existence validators ([llmops.md](./llmops.md) Q14), nullable/"not present" paths, deterministic guardrails, human-in-the-loop for actions, full audit, and a hallucination-rate SLA. The tradeoff: coverage vs safe refusal. → [`answers/design-banking-bot.md`](../answers/)

### Prompt J — Design an AI candidate-sourcing system: 750M profiles, semantic search, <500ms
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **How a senior frames it:** Billion-scale ANN (IVFPQ + rerank, not HNSW — do the RAM math, [rag.md](./rag.md) Q21, Q34), hybrid semantic + structured filters, a two-stage funnel (Q21) to hit <500ms, freshness via CDC, and fairness/bias counter-metrics on sourcing. The tradeoff: recall at 750M vs the latency budget. → [`answers/design-candidate-sourcing.md`](../answers/)

---

<sub>⬅ [Back to the index](./README.md) · Related: [Coding →](./coding.md) · [RAG →](./rag.md) · [Fine-tuning →](./fine-tuning.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
