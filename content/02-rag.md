# Retrieval-Augmented Generation (RAG)

> Ground an LLM in *your* corpus at query time — for freshness, citations, and access control the weights can never give you.

Back to [README](../README.md) · [All questions](../questions/README.md)

---

## What this tests in an interview

Whether you treat retrieval as an **information-retrieval problem with real metrics** — or as a magic box you point at a vector DB. The senior signal is simple: you debug **retrieval before generation**, you can define `recall@k` / `precision@k` and pick the right one, and you can size the thing (RAM, latency, cost) on a whiteboard. Most "we tried RAG and it didn't work" stories are really "we built the wrong *shape* of RAG and never instrumented the retrieval boundary."

---

## Embeddings & vector search

A **bi-encoder** (your embedding model) maps query and document into the same vector space *separately*, so document vectors can be computed offline and indexed — that's what makes search fast, and also why the match is only approximate: query and doc never see each other. Retrieval is then nearest-neighbour search in that space (cosine on normalized vectors; on normalized vectors cosine/dot/L2 rank identically).

At scale you never brute-force it. Exact ("flat") search is `O(N·d)` — one query against 100M 1,536-d vectors is ~150 billion multiply-adds, tens of seconds on a CPU. A vector DB instead builds an **approximate-nearest-neighbour (ANN)** index that trades a few points of recall for 100–1000× speed. The pipeline is always `embed query → ANN → top-k → (rerank) → LLM`.

### ANN index tradeoffs — HNSW vs IVF vs IVFPQ

The dominant index is **HNSW** (Hierarchical Navigable Small World): a layered proximity graph you descend greedily, each hop moving closer to the query. Two knobs set quality/cost — `M` (edges per node: higher = better recall, more RAM) and `efSearch` (candidates explored per query: higher = better recall, more latency). **HNSW is a recall/latency/memory triangle you tune, not a black box.** `IVF` clusters vectors and probes only the nearest cells (`nprobe`); `IVFPQ` adds **product quantization** — each vector is chopped into sub-vectors, each replaced by a codebook centroid id, compressing a ~6 KB vector to a few dozen bytes.

| Index | Memory (1B×128-d) | p99 | Recall | Use when |
|---|---|---|---|---|
| **HNSW** | ~1,408 GB | 32–152 ms | highest | you can afford the RAM |
| **IVF** | ~1,126 GB | < HNSW | medium | memory-tight, recall-tolerant |
| **IVFPQ** | ~70 GB | 106–230 ms | lowest | massive footprint → pair with a reranker |

*(Source: AWS OpenSearch 1B-vector benchmark.)* HNSW vs IVFPQ is a **~20× memory delta** — index choice is a procurement decision, and it sets your p99 and RAM bill far more than the embedding model does.

**The RAM math to do on the whiteboard:** `bytes = N_vectors × dim × 4` (float32). So `50M chunks × 1,536 dim × 4 B ≈ 300 GB` *raw*, before HNSW adds its graph (×1.5–2). That single line is why dimension is a first-order cost decision — and why Matryoshka truncation (keep the first N dims) and product quantization exist. To shrink RAM *without* re-embedding: truncate Matryoshka dims and/or switch to IVFPQ, recovering the lost recall with a reranker on the shortlist.

> [!WARNING]
> **"HNSW returns the true nearest neighbours."** It returns *approximate* ones — that's the "A." At default settings it returns ~95–99% of the true top-k; the missing 1–5% are **silent**. Your retrieval has a recall ceiling set by the *index config*, not just the encoder. When `recall@k` looks stuck, raise `efSearch`/`M` before you blame the embedding model.

---

## The three shapes of RAG — name it before you build

The most common senior mistake is building "a RAG" without deciding which *shape* you have. Shape sets your latency budget, index footprint, freshness SLA, and dominant lever.

| Shape | Example | Budget | Dominant lever | Freshness / ingest |
|---|---|---|---|---|
| **Chat** | Notion, DoorDash, Uber Genie | 300 ms–2 s, modest QPS | chunking + reranking + model choice | daily/weekly → nightly batch |
| **Enterprise-search** | Glean, Elastic, Harvey | sub-second p99 | the hybrid index; multi-tenancy + RBAC | CDC on doc edits, per-tenant isolation |
| **Web** | Perplexity, LinkedIn feed | seconds-fresh, tight latency | one engine fusing lexical + dense + freshness | seconds — tens of thousands of updates/s |

Perplexity runs ~200M queries/day at p50 358 ms; Notion cut chat latency ~2 s → ~350 ms by swapping to a *smaller* fine-tuned model, not by touching retrieval. Same word, three completely different engineering problems.

> [!NOTE]
> **Interview move:** when asked to "design a RAG," name the shape and its freshness SLA *out loud* first. It frames every later decision and shows you've built one, not just read about it.

---

## The seven failure points

Barnett et al. catalogued seven failure modes that recur in real RAG. This taxonomy turns "the bot is dumb" into a precise, stage-localized diagnosis — and each maps to a lever.

| # | Failure | Symptom | Cause | Fix |
|---|---|---|---|---|
| 1 | **Missing content** | answer nowhere in output | not in the corpus | ingestion / coverage |
| 2 | **Missed top-ranked** | present but below cutoff | ranked too low | reranking |
| 3 | **Not in context** | right doc, wrong chunk | split / buried | chunking + contextual retrieval |
| 4 | **Not extracted** | context has it, model misses it | weak grounding | grounding prompt |
| 5 | **Wrong format** | ignored "return JSON" | output shape | structured output |
| 6 | **Wrong specificity** | too vague / too narrow | bad query | query transformation |
| 7 | **Incomplete** | partial multi-part answer | single-hop over compound Q | decomposition |

Failures **1–3 are retrieval** (the text never reached the prompt); **4–7 are generation/orchestration** (the text was there, the pipeline mishandled it). The discipline: *locate the boundary first.* If the gold chunk is in the retrieved set but the answer is wrong, no chunking or reranking helps — you're in generation territory. If it's not in the set, no prompt engineering helps. Most teams burn weeks tuning the wrong half because they never instrumented the boundary.

> [!TIP]
> Treat retrieval as an IR problem. Build a labelled set of ~50–200 `(query, gold-chunk-ids)` pairs and measure `recall@k` (did the gold chunk make top-k?) and `precision@k` (how much of top-k was relevant?) *before* you touch the generator. The first hour of a RAG project is better spent on 50 labelled queries than on swapping the embedding model — every later change is an A/B test against that set. **Provenance rule:** harvest the gold *contexts* from your production retriever's runs, not from chunks you hand-picked, or offline recall reads far higher than reality (one team saw 0.91 offline, 0.40 in prod).

---

## Chunking

Parsing is stage zero — tables, scans, and multi-column PDFs corrupt text *before* any chunker runs; no chunker recovers a shredded parse. After that, chunk size is a **precision/recall tradeoff you measure, not guess**. Default to ~512 tokens with 10–15% overlap, split recursively on structure — then measure: if `precision@5 < 0.7`, drop toward 256.

| Size | Recall@20 | Precision@5 | What breaks |
|---|---|---|---|
| 128 tok | lower | high | context stripped (orphaned refs) |
| 256 tok | good | **highest** | sweet spot for entity lookups |
| 512 tok | high | good | sane default — measure before trusting |
| 1024 tok | highest | lower | signal dilution, noisy top-k |
| 2048+ tok | high | poor | one fact lost in an averaged vector |

The counterintuitive mechanism: a chunk is pooled into *one* vector. A 1,500-token chunk with 30 query-relevant tokens averages that signal against 1,470 irrelevant ones — cosine similarity drops and a tight 200-token competitor outranks it. So **bigger chunks add noise to the vector**, they don't "give the model more context."

**The fixes for orphaned chunks:**

| Technique | How | Cost | When |
|---|---|---|---|
| **Late chunking** | embed the *whole doc* in one pass, then pool per-chunk so each vector carries doc context | one forward pass, no LLM call | long-context encoder + *coherent* docs |
| **Contextual retrieval** | LLM writes a 50–100 tok context per chunk, prepend before indexing (embeddings **and** BM25) | ~$1.02/M tokens, ~90% off with prompt caching | model-agnostic; also feeds BM25 |
| **Small-to-big** | index small (256 tok) children for matching, return the ~1024 tok parent to read | dedup by parent id | precision *and* context |

**Anthropic's Contextual Retrieval** (Sep 2024) stacks cleanly on top-20 retrieval failure rate:

```
Embeddings only ................. 5.7%
+ Contextual embeddings ......... 3.7%   (-35%)
+ Contextual BM25 (hybrid) ...... 2.9%   (-49%)
+ Reranker ...................... 1.9%   (-67%)
```

> [!TIP]
> Three cheap, *stackable* wins — contextual embeddings, contextual BM25, a reranker — take top-20 retrieval failures from **5.7% → 1.9%** (35–49% fewer, 67% with the reranker) with no model change and no chunk-size retune. The order that compounds: reasonable chunks → contextual + hybrid → reranker.

Code and structured corpora chunk on their **own semantic units**, not a token ruler — Cursor splits code on AST function/class boundaries and keys each vector by file path. Metadata (`doc_id`, `acl`, `date`, `last_verified`) is *part* of the chunk: it's what makes it filterable, citable, and expirable, and exact constraints like "only 2024 docs" belong in a metadata predicate, not in the embedding.

> [!WARNING]
> **"The model needs more context, so use bigger chunks."** Backwards — bigger chunks dilute the pooled vector and rank *worse*. The right move is **small-to-big**: small chunks to *match*, parent expansion to *read*. And never change chunking in place on a live index (see the embedding clock, below).

---

## Hybrid search: dense + BM25, fused with RRF

Dense retrieval nails *meaning* but is bad at **exact** tokens — error codes, SKUs, function names, surnames — because a dense vector *compresses* text and rare exact tokens are exactly what a lossy compressor throws away. `BM25` keeps them because it never compresses. They fail on **different queries**, which is why fusing them wins.

| Query | Dense | BM25 | Why hybrid |
|---|---|---|---|
| "how do I cancel my plan" | strong | weak | paraphrase → dense |
| "error E-1042" | weak | strong | exact rare token → BM25 |
| "ACME-X200 firmware" | weak | strong | SKU / part number → BM25 |
| "Dr. Sarah Okonkwo paper" | weak | strong | proper noun → BM25 |

BM25 is **not a raw count** — it's IDF-weighted (rare terms count more), term-saturated (`k1`: the 10th occurrence adds less than the 2nd), and length-normalized (`b`). On many corpora a tuned BM25 ties or beats a generic dense model. The senior gotcha lives in the **analyzer**, not the formula: a default analyzer that strips hyphens turns `E-1042` into nothing matchable and your exact-match advantage silently evaporates.

Fuse with **Reciprocal Rank Fusion (RRF)** — each doc scores `Σ 1/(k + rank)` across retrievers (`k ≈ 60`). It fuses on **rank, not score**, on purpose: BM25 scores are unbounded and cosine sits in [-1, 1], so averaging the raw numbers lets whichever scale is bigger dominate.

```python
def rrf(rankings, k=60):
    scores = {}
    for ranking in rankings:            # e.g. [bm25_ids, vector_ids]
        for rank, doc_id in enumerate(ranking):
            scores[doc_id] = scores.get(doc_id, 0) + 1 / (k + rank)
    return sorted(scores, key=scores.get, reverse=True)

def hybrid_recall(query, k_each=50):
    bm25_ids  = bm25_search(query, k=k_each)    # exact tokens, codes, names
    dense_ids = vector_search(query, k=k_each)  # paraphrase, intent
    fused = rrf([bm25_ids, dense_ids], k=60)     # rank-based, no calibration
    return fused[:k_each]                        # capped shortlist -> reranker
```

On WANDS, BM25 (0.6983) and dense (0.6953) are indistinguishable alone — hybrid RRF jumps to **0.7497 (+7.4%)**; across BEIR, hybrid lifts system NDCG 43.4 → 52.6. A tuned convex combination (`α·norm(bm25) + (1-α)·dense`) can beat RRF by a few points, but `α` is corpus/query-specific and goes stale on drift — RRF has no weight to maintain. Prefer RRF by default; weighted only with a labelled set *and* a re-tuning plan.

> [!WARNING]
> **"Vector search killed keyword search."** BM25 didn't die — it became the half of the system dense retrieval is bad at. Even Anthropic's contextual-retrieval gains come partly from adding *contextual BM25* back in. Skip hybrid only for pure-paraphrase traffic over a small clean corpus with no codes or names.

---

## Reranking: cross-encoder after retrieval (why two stages)

Recall and precision are **different jobs**, so you split them. The first stage (bi-encoder + BM25, fused) casts a wide cheap net; a **cross-encoder** reranker then re-scores only the shortlist for precision.

| | Bi-encoder (recall) | Cross-encoder (precision) |
|---|---|---|
| Input | query and doc encoded *separately* | query + doc concatenated, *one* input |
| Attention | none across the pair | full cross-attention, every q-token ↔ every d-token |
| Indexable | yes — doc vectors precomputed offline | **no** — a fresh forward pass per pair |
| Cost | ms over millions (ANN) | ~ms *per pair* → shortlist only |
| Accuracy | approximate, scalable | accurate, ~1000× costlier |

You can't pre-index a cross-encoder, so scoring the whole corpus per query is seconds-to-minutes — that's why it only runs on the fused top-25–50. **Retrieve 50, rerank to 5.** Cohere reports **+23.4% nDCG over hybrid, +30.8% over BM25** on financial data for a few-hundred-ms add — routinely a bigger win than swapping models.

```python
import cohere
co = cohere.Client()
candidates = hybrid_recall(query, k_each=50)         # wide net (recall)
reranked = co.rerank(model="rerank-v3.5", query=query,
                     documents=[c["text"] for c in candidates],
                     top_n=5)                          # precision: best 5
context = [candidates[r.index] for r in reranked.results]
```

Caveats: cap candidates and truncate passages to ~512 tokens (cross-encoder cost is linear in candidates × length); general rerankers lose **5–10 points** on legal/medical/financial — fine-tune on ~10k triples for a vertical; rerank the *fused* list, not just dense (or you throw away BM25's exact hits); and **degrade gracefully** — a reranker timeout should fall back to first-stage order, not error the query. A reranker only fixes failure #2 ("in the set, ranked too low"); if `recall@50` is broken it can't recover a chunk that was never retrieved — solve recall first. (Related: **ColBERT** / late interaction is a *first-stage retriever*, not a reranker — per-token max-sim, more expressive than a pooled vector but pre-indexable; reach for it at 10M+ passages when single-vector recall is broken.)

---

## Retrieval & generation eval

Grade retrieval and generation **separately** so you can localize a failure. Pick the retrieval metric to match the task:

| Metric | Rewards | Use when |
|---|---|---|
| **Recall@k** | gold chunk anywhere in top-k | first-stage **ceiling** (must-have — a miss is unrecoverable) |
| **Precision@k** | share of top-k that's relevant | context noise / token budget |
| **MRR** | the *first* relevant hit, high | one right answer, position matters |
| **NDCG@k** | graded relevance, position-discounted | multiple relevant; **reranker** quality |

```python
def recall_at_k(eval_set, retrieve, k=20):
    hits = 0
    for query, gold_ids in eval_set:
        got = {r["chunk_id"] for r in retrieve(query, k=k)}
        hits += 1 if got & set(gold_ids) else 0     # did ANY gold chunk make top-k?
    return hits / len(eval_set)
# Rule of thumb: maximize recall@50-100 in stage 1, buy precision@5 with a reranker.
```

On the **generation** side: **faithfulness/groundedness** (is every claim entailed by the context?) and **answer relevancy** (does it address the question?), held with retrieval fixed. Read the 2×2: retrieval-bad + generation-good = faithfully answered the *wrong* context (fix chunking/hybrid/rerank); retrieval-good + generation-bad = right context, model ignored it (fix grounding). **RAGAS** ships these (`context_precision`, `context_recall`, `faithfulness`, `answer_relevancy`) — but treat each metric as *one alarm with a known blind spot*: RAGAS faithfulness **nulls out** on 83.5% of FinanceBench numeric prompts because claim-entailment breaks on multi-step reasoning, and a null is not a pass. Freeze a golden set (contexts from the *production* retriever), gate CI on a per-metric margin, and validate any LLM-judge against human labels via Cohen's κ before trusting it — an unvalidated judge is systematically wrong at scale.

---

## Scale: the RAM, the dual index, and the embedding clock

At tens of millions of vectors, your ANN algorithm is a **memory-budget decision first**: HNSW if you can afford the RAM (~1.4 TB at 1B, best recall), IVFPQ if you can't (~70 GB, ~20× less), recovering recall with a reranker. Harvey found pgvector comfortable only to ~500k embeddings; past ~10M plan for IVFPQ + reranking and budget for periodic recompute. **Buy** a vector DB until one of privacy, freshness, scale, or team-size forces you to build.

The invisible killer at scale is the **embedding clock** (aka the freshness graveyard). Vector similarity ignores *time*, and three clocks drift silently — none of them error, none show on a latency dashboard:

- **Document clock** — the source changed (API docs ~2 weeks, compliance ~6 months).
- **Embedding clock** — you upgraded the encoder; new vectors aren't cosine-comparable to old ones ("representation shearing").
- **Chunking clock** — you changed boundaries, so a vector now represents *different* text.

Because half the index represents old text and half new, the *distribution* of distances shears and recall sags a few points and stays there — one documented case fell **0.92 → 0.74** with nothing in the metrics to explain it. Re-embedding is therefore **never free**: a ~1 TB re-embed runs ~$12k/month. The defenses: CDC-driven incremental re-embedding (only changed rows), a **dual index** (build the new one alongside the old, validate on a held-out set, atomic cutover, instant rollback), and shipping **freshness as a first-class metric** — top-k overlap on a fixed probe set catches drift end-to-end accuracy misses.

> [!WARNING]
> **"Re-embedding is basically free / I'll re-chunk the live index in place."** Re-chunking = full re-embed; mixing generations in one index shears recall while every dashboard stays green. Always re-index behind a dual index with a validated cutover. Saying "in place" is a red flag that you haven't run RAG at scale.

---

## Security: ACL filter *before* similarity, and indirect injection

The scariest B2B RAG bug is one tenant seeing another's data. The single most important rule: the permission check is a **filter on the retrieval query**, so a forbidden chunk is *never fetched*. The tempting anti-pattern — retrieve broadly, then redact — is a breach waiting to happen: once the chunk is in the system, a prompt-injection payload, a citation request, or a summarization step can surface it.

```python
# SAFE: permissions filter the search itself -> forbidden chunks never load.
def retrieve_safe(query, user, k=50):
    return hybrid_search(query, k=k, acl_filter=user.allowed_acls)   # filter AT query time

# UNSAFE: fetch everything, then try to hide it. A clever prompt can surface it.
def retrieve_unsafe(query, user, k=50):
    hits = hybrid_search(query, k=k)                                 # forbidden chunks ALREADY loaded
    return [h for h in hits if h["acl"] in user.allowed_acls]        # too late -- breach risk
# Rule: a chunk that is never retrieved can never leak. Enforce ACLs pre-rerank.
```

The 2026 pattern for multi-tenancy is DB-layer isolation (Postgres + pgvector + **row-level security** + HNSW, namespace-per-tenant, or per-customer VPC), *not* app-layer filtering — "I'd add it to the system prompt" is an instant fail.

**Indirect prompt injection** (OWASP LLM01:2025, the #1 risk) rides in through the *second* input channel: retrieved content. A poisoned "public" doc can flip a guardrail or pull privileged context — She et al. (2025) found inserting documents into a guardrail's context flips its safe/unsafe judgment **~1-in-10 cases**, because the guardrail is itself an LLM. Guarding only the user prompt is negligent: validate retrieved context too, and whitelist any tool calls the model emits.

---

## The 2026 angle: is RAG still relevant with long context?

Yes — and the framing that lands is: **RAG is retrieval + grounding + freshness + ACLs, not a context-window workaround.** "Why not stuff the whole corpus into a 1M-token window?" loses on four production axes:

- **Cost** — you pay for every dumped token on *every* query, vs retrieving a few chunks.
- **Latency** — prefill scales with input length; a full corpus is seconds of TTFT.
- **Recall** — lost-in-the-middle / context-rot means a fact buried in 500k tokens often *isn't* recalled, and it worsens as you fill the window.
- **Attribution** — RAG hands you the exact chunk to cite; a giant context can't tell you which passage it used.

Re-evaluations show long context wins single-hop Wikipedia QA, while RAG wins dialogue, grounded citations, freshness, and cost — and even Anthropic's own guidance is *just-in-time retrieval with sub-agent compaction*, not dumping the corpus in. Long context is a **complement for holistic single-document tasks**, not a replacement for retrieval over a corpus. "A bigger context window solves RAG" is the red-flag answer; "retrieve to keep the window small, cheap, ordered, and citable" is the senior one.

---

## Interview angles

- **"Evaluate a RAG pipeline — what metrics?"** → Split retrieval and generation. Report `recall@k` as the must-pass *ceiling* for the retriever (a miss is unrecoverable) and `NDCG@k` (or MRR for single-answer queries) to judge the reranker. On generation: faithfulness + answer relevancy, held with retrieval fixed. Then the 2×2 to localize a failure. Tie each metric to the failure it exposes; naming one metric for everything is the weak answer.
- **"What is hybrid search and why?"** → Dense compresses away rare exact tokens (codes, SKUs, names); BM25 keeps them. They fail on different queries, so fuse both with RRF (rank-based — BM25 is unbounded, cosine is [-1,1]). Hybrid lifts BEIR NDCG 43.4 → 52.6. Mention the analyzer gotcha: a hyphen-stripping tokenizer kills exact-code matching, and that's a tokenization bug, not a fusion bug.
- **"Reranking — cross-encoder vs bi-encoder?"** → Bi-encoder encodes query and doc separately (indexable, fast, approximate); cross-encoder concatenates them and runs full cross-attention (accurate, ~1000× costlier, un-indexable → shortlist only). Retrieve 50, rerank to 5. It only fixes "in the set, ranked low" — it can't recover broken recall@50.
- **"Explain HNSW."** → Greedy descent through a layered proximity navigable-graph; `M` (edges) and `efSearch` (candidates) trade recall for RAM/latency. It's *approximate* — the index config is a recall ceiling you tune before blaming the encoder.
- **"Where do embeddings fail?"** → Negation, temporal reasoning, and exact match. A dense vector compresses meaning, so rare exact tokens and hard constraints ("only 2024 docs") wash out — push those into BM25 (exact tokens) and metadata predicates (constraints); let the vector handle semantics.
- **"Key RAG tradeoffs?"** → recall vs precision (wide cheap net → costly precise reranker); chunk size (small dilutes less but strips context → small-to-big); RRF vs weighted fusion (maintenance vs a few NDCG points); HNSW vs IVFPQ (~20× RAM); managed vs self-host (privacy/freshness/scale/team).
- **"Is RAG still relevant with long context?"** → Yes — RAG is retrieval + grounding + freshness + ACLs. Long context loses on cost, latency (prefill), recall (lost-in-the-middle), and attribution. It's a complement for single-document holistic tasks, not a corpus replacement.
- **"RAG hallucinates despite the right context in the prompt — fix?"** → That's failure #4 (not extracted), a *generation* problem — no better encoder helps. Enforce the grounding contract: answer only from numbered context, cite the chunk id per claim, allow abstention ("I don't know"), then *verify* cited ids exist and (high-stakes) that the claim is entailed. "Use a better model" is the junior answer.

---

## 📚 Resources

- 📰 [Anthropic — Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) — the canonical fix for the orphaned chunk: BM25 + contextual embeddings + reranker, **35–49% fewer retrieval failures** (67% with the reranker). The single best "read past the README" story in RAG.
- 💻 [run-llama/llama_index](https://github.com/run-llama/llama_index) — MIT, ~43k★ — canonical indexing/RAG framework; small-to-big, hybrid, and RAG-eval primitives out of the box.
- 💻 [vibrantlabsai/ragas](https://github.com/vibrantlabsai/ragas) — Apache-2.0, ~14.5k★ — faithfulness, context precision + recall, answer relevancy; know its numeric/multi-hop null-out blind spot.
- 📰 [Eugene Yan — "Evaluating RAG"](https://eugeneyan.com/) — the canonical production RAG eval series; the split-eval discipline and golden-set provenance in practice.
- 📄 [HNSW paper — Malkov & Yashunin](https://arxiv.org/abs/1603.09320) — the ANN graph index everyone ships; read it to explain `M`/`efSearch` and the recall ceiling mechanically.
- 📘 [LangChain RAG docs](https://docs.langchain.com/oss/python/langchain/rag) — canonical hybrid + rerank + query-transform reference implementations.
- 📄 [Reciprocal Rank Fusion — Cormack et al. (SIGIR '09)](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) — the original RRF paper; why fusing on rank beats tuning a corpus-specific weight.
- 💻 [KalyanKS-NLP/RAG-Interview-Hub](https://github.com/KalyanKS-NLP/RAG-Interview-Questions-and-Answers-Hub) — Apache-2.0, ~521★ — 100+ topically-organized RAG interview Q&A; the most directly-named RAG-interview repo.

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [RAG questions](../questions/rag.md)
