# RAG — Retrieval, Chunking, Hybrid Search, Reranking, Scale & Safety

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/rag.md`](../content/02-rag.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **43 questions** (36 self-test checkpoints + 7 reported). RAG is the single most-asked applied topic in a 2026 AI-engineer loop, and "RAG" hides three different systems. The through-line: retrieval is an information-retrieval problem — diagnose with metrics (recall@k / precision@k), fix the cheap levers (chunking, hybrid, reranking) before swapping models, and treat the retrieved channel as an untrusted input.

---

### How to use this file

Answer first, then open the fold. Note the recurring senior moves: **measure before you swap models**, **match technique to query distribution**, **enforce ACLs at retrieval time**, and **catch silent freshness drift with probe-set telemetry**.

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (scale, multi-tenant, security, or end-to-end design)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — Diagnosing retrieval & the "shape" of RAG `[rag-systems] l1`

### Q1. A teammate proposes fixing weak answers by upgrading to the largest available embedding model. What's the senior response?

> **Difficulty:** 🟡 Medium · `[rag-systems] l1` · 🔮 Representative

- Agree — bigger embeddings are the main driver of retrieval quality
- ✅ First measure recall@k / precision@k to locate the failure, then fix chunking/hybrid before swapping models
- Switch to a larger generation model instead

<details><summary>Show answer & why each option</summary>

- **Agree, bigger embeddings** — Model size is rarely the dominant lever; domain fit, chunking, and hybrid search usually move retrieval far more.
- ✅ **Measure, then fix chunking/hybrid** — Retrieval is an IR problem — diagnose with metrics, and the cheapest wins are almost always chunking + hybrid, not a bigger encoder.
- **Larger generation model** — If the right chunk isn't retrieved, no generator can recover it — this is a retrieval problem.
</details>

### Q2. You're building support QA over a fixed 40k-doc knowledge base with a 1.5 s latency budget. Which "shape" framing should drive your architecture?

> **Difficulty:** 🟡 Medium · `[rag-systems] l1` · 🔮 Representative

- Web-shaped — optimize a single engine for seconds-fresh, billion-doc search
- ✅ Chat-shaped — closed corpus, comfortable latency; spend your effort on chunking, reranking, and model choice
- It doesn't matter — all RAG is the same pipeline

<details><summary>Show answer & why each option</summary>

- **Web-shaped** — That's Perplexity's problem, not yours; a closed 40k-doc corpus doesn't need web-scale freshness machinery.
- ✅ **Chat-shaped** — A bounded corpus with a 1.5 s budget is the chat shape; the dominant levers are indexing quality and reranking, not the vector DB.
- **All the same** — Shape sets your latency budget, footprint, and dominant lever; conflating them is how you over- or under-build.
</details>

### Q3. In an interview you're asked "RAG or fine-tuning to make the model answer from our internal wiki?" Strongest opening?

> **Difficulty:** 🟡 Medium · `[rag-systems] l1` · 🔮 Representative

- Fine-tune the model on the wiki so the facts live in the weights
- ✅ RAG — retrieve at query time for freshness, citations, and access control; consider fine-tuning only for style/format or a fixed skill
- Neither — just use a bigger base model

<details><summary>Show answer & why each option</summary>

- **Fine-tune** — Bakes in facts that go stale on every doc edit, gives no citations, and can't enforce per-user permissions.
- ✅ **RAG** — Decouples knowledge from weights: re-index for freshness, cite the chunk, filter by ACL. Fine-tuning is for behaviour/format, not volatile facts.
- **Bigger base model** — Still has no row for your private, post-training data; size doesn't add knowledge it never saw.
</details>

### Q4. Your retriever's recall@20 is stuck at 0.86 and tuning chunking hasn't moved it. You're on HNSW with default params. What's worth trying first?

> **Difficulty:** 🔴 Hard · `[rag-systems] l1` · 🔮 Representative

- ✅ Raise `efSearch` (and/or `M`) — ANN is approximate, so the index config may be the recall ceiling
- Immediately swap to a larger embedding model
- Lower k to 5 so precision improves

<details><summary>Show answer & why each option</summary>

- ✅ **Raise `efSearch`/`M`** — HNSW returns approximate neighbours; a low `efSearch` silently drops true top-k. Raising it (at some latency cost) tests whether the index, not the encoder, caps recall.
- **Swap embedding model** — A full re-embed, rarely the cheapest lever; first rule out the approximate-search ceiling you can change for free.
- **Lower k to 5** — Shrinking k can only lower recall — it removes candidates rather than recovering the missed gold chunk.
</details>

### Q5. You're serving 50M chunks and the vector index won't fit in your RAM budget. Which move shrinks the footprint WITHOUT re-embedding the corpus?

> **Difficulty:** 🔴 Hard · `[rag-systems] l1` · 🔮 Representative

- Re-embed everything with a smaller model
- Increase the chunk size so there are fewer vectors
- ✅ Truncate Matryoshka dimensions and/or switch to product quantization (IVFPQ), recovering quality with a reranker

<details><summary>Show answer & why each option</summary>

- **Re-embed smaller** — That is exactly the re-embed you're trying to avoid — and it invalidates every stored vector.
- **Bigger chunks** — Dilutes the embedding and hurts precision; a quality regression, not a clean memory fix.
- ✅ **Matryoshka truncation / IVFPQ + reranker** — Both shrink the stored vectors in place at a measured recall cost that a reranker on the shortlist restores — no re-embed required.
</details>

---

## Section B — Chunking & parsing `[rag-systems] l2`

### Q6. A retrieved chunk says "revenue grew 3% over the previous quarter," but the model can't tell which company or quarter — and the right chunk often doesn't rank for "ACME Q2 2023 revenue." Strongest fix?

> **Difficulty:** 🔴 Hard · `[rag-systems] l2` · 🔮 Representative

- Use much larger chunks so each one carries more surrounding text
- ✅ Contextual retrieval: prepend an LLM-written context to each chunk before indexing it (embeddings + BM25)
- Switch to a larger generation model

<details><summary>Show answer & why each option</summary>

- **Larger chunks** — Helps a little but dilutes the embedding and bloats the index — and still doesn't reliably inject the missing identifiers.
- ✅ **Contextual retrieval** — Situates every chunk in its document so identifiers survive isolation; Anthropic measured ~35–49% fewer retrieval failures, ~67% with a reranker.
- **Larger generation model** — The identifiers were never retrieved — no generator can recover information that isn't in context.
</details>

### Q7. Your RAG has healthy recall@20, but answers are vague and miss specifics; precision@5 measures 0.55. Best first move?

> **Difficulty:** 🟡 Medium · `[rag-systems] l2` · 🔮 Representative

- Increase chunk size to give each chunk more context
- ✅ Shrink chunks toward ~256 tokens and add a reranker to sharpen the top-5
- Swap the embedding model for a larger one

<details><summary>Show answer & why each option</summary>

- **Increase chunk size** — Larger chunks usually lower precision — more noise per vector — the opposite of what you need.
- ✅ **Smaller chunks + reranker** — precision@5 is the bottleneck; smaller chunks + a cross-encoder tighten exactly what reaches the prompt without losing recall.
- **Larger embedding model** — Rarely the dominant lever; chunk size and reranking move precision far more per dollar.
</details>

### Q8. You need to change your chunk size and contextualization prompt on a live 1 TB index. What's the safe rollout?

> **Difficulty:** 🔴 Hard · `[rag-systems] l2` · 🔮 Representative

- Re-chunk and re-embed in place, one document at a time, during low traffic
- ✅ Build a second index with the new chunker, validate it on a held-out labelled set, then cut traffic over atomically
- Keep both chunkers' vectors in one index and let RRF sort it out

<details><summary>Show answer & why each option</summary>

- **In-place re-embed** — Mixes old and new representations in the same index — recall shears silently while dashboards stay green.
- ✅ **Dual index + atomic cutover** — Keeps generations from mixing; you switch only once the new one validates, and can roll back instantly.
- **Both in one index** — Cross-generation vectors aren't comparable; fusing them hides the sheared distance distribution rather than fixing it.
</details>

### Q9. Indexing a codebase, answers to "where is `parseConfig` defined?" keep missing or returning fragments. Your splitter is fixed-size 512 tokens. Best fix?

> **Difficulty:** 🟡 Medium · `[rag-systems] l2` · 🔮 Representative

- ✅ Split on language-aware boundaries (functions/classes via AST), attaching file path + enclosing symbol as metadata
- Increase the chunk size to 2,048 tokens so more code fits
- Add overlap so fragments are duplicated across chunks

<details><summary>Show answer & why each option</summary>

- ✅ **AST-aware splitting + metadata** — Code's unit of meaning is the function/class, not 512 tokens; AST splitting keeps definitions whole and the metadata makes "where defined" answerable — the Cursor approach.
- **2,048 tokens** — Bigger chunks dilute the embedding and still cut at arbitrary points.
- **Add overlap** — Overlap patches straddling sentences, not a function split off from its signature/imports.
</details>

### Q10. Your corpus is mostly scanned PDFs with tables. Recall is poor and answers garble numeric facts. Where do you look first?

> **Difficulty:** 🟡 Medium · `[rag-systems] l2` · 🔮 Representative

- Swap to a semantic chunker
- Raise k and add a reranker
- ✅ The parsing stage — use layout/OCR-aware extraction and keep tables as Markdown/HTML before chunking

<details><summary>Show answer & why each option</summary>

- **Semantic chunker** — Can't recover structure the parser already destroyed; you'd be tuning the wrong stage.
- **Raise k + reranker** — More candidates can't fix chunks that are interleaved-column noise; the text was corrupted before indexing.
- ✅ **The parsing stage** — No chunker recovers a shredded parse; tables and scans need structure-preserving extraction first, or every downstream stage inherits garbage.
</details>

---

## Section C — Hybrid search & fusion `[rag-systems] l3`

### Q11. A user searches for the exact error code "E1042." Which retriever most reliably surfaces it?

> **Difficulty:** 🟢 Easy · `[rag-systems] l3` · 🔮 Representative

- Dense vector search
- ✅ Keyword / BM25, inside a hybrid (with a sane analyzer)
- A bigger embedding model

<details><summary>Show answer & why each option</summary>

- **Dense search** — Embeddings blur rare exact tokens — "E1042" may not sit near anything useful in vector space.
- ✅ **BM25 in hybrid** — Lexical search matches the exact token; hybrid keeps semantic recall for the rest — just make sure the analyzer doesn't strip the code.
- **Bigger embedding model** — Still an embedding — exact rare tokens remain its weak spot regardless of size.
</details>

### Q12. Why do production systems like Perplexity fuse BM25 + dense and then run a cross-encoder, instead of just doing one bigger vector search?

> **Difficulty:** 🟡 Medium · `[rag-systems] l3` · 🔮 Representative

- Because vector search is too slow at scale
- ✅ Recall and precision are different jobs: fuse cheap recall (BM25+dense), then spend a precise but costly reranker on the shortlist
- To avoid using a vector database at all

<details><summary>Show answer & why each option</summary>

- **Too slow** — ANN is fast; the issue is quality, not speed.
- ✅ **Recall vs precision** — A wide cheap net maximizes the chance the right doc is present; the expensive cross-encoder then re-orders only the shortlist for precision.
- **Avoid vector DB** — They still use vector retrieval — it's one stage, fused with lexical and reranked.
</details>

### Q13. You turned on hybrid, but searches for part numbers like "ACME-X200" still miss even though the docs contain them. Most likely cause?

> **Difficulty:** 🔴 Hard · `[rag-systems] l3` · 🔮 Representative

- ✅ The BM25 analyzer is splitting/stripping the hyphen so "ACME-X200" never becomes a matchable token
- RRF is weighting dense too heavily
- The embedding model is too small

<details><summary>Show answer & why each option</summary>

- ✅ **Analyzer strips the hyphen** — BM25 only matches tokens the analyzer emits; a default analyzer that strips punctuation kills exact-code matching — the classic hybrid gotcha.
- **RRF weighting** — RRF uses ranks; if BM25 never produced the token, no fusion weight can surface it — the failure is upstream in tokenization.
- **Embedding too small** — Exact rare tokens are dense retrieval's weak spot regardless of size; this is a lexical-analyzer problem.
</details>

### Q14. A teammate wants to replace RRF with a tuned weighted fusion (`α·bm25 + (1−α)·dense`) to squeeze a few NDCG points. What's the senior caveat?

> **Difficulty:** 🔴 Hard · `[rag-systems] l3` · 🔮 Representative

- Weighted fusion is always worse than RRF
- ✅ The weight is corpus/query-distribution-specific, so it goes stale on drift and needs a labelled set plus ongoing re-tuning
- Weighted fusion can't combine BM25 and cosine scales

<details><summary>Show answer & why each option</summary>

- **Always worse** — Not true — a calibrated α can beat RRF by a few points; the issue is cost, not a hard ceiling.
- ✅ **Weight goes stale on drift** — A tuned α silently mis-ranks as the corpus/query mix drifts; RRF's zero-maintenance robustness is usually worth more than the small gain.
- **Can't combine scales** — It can, via score normalization; the real problem is maintaining the weight over time.
</details>

### Q15. Queries are pure natural-language paraphrase over a small, clean FAQ with no codes or proper nouns. A teammate insists on adding BM25 + RRF. Best call?

> **Difficulty:** 🟡 Medium · `[rag-systems] l3` · 🔮 Representative

- Add it — hybrid is always better
- Add SPLADE instead, it's strictly superior
- ✅ Skip hybrid here — measure first; pure dense may already be enough for paraphrase-only traffic

<details><summary>Show answer & why each option</summary>

- **Always better** — Hybrid's win comes from exact-token queries; with none, you add an index and latency for little gain.
- **SPLADE** — Shines on vocabulary mismatch with exact-match needs; for pure paraphrase over a clean FAQ it's unnecessary complexity too.
- ✅ **Skip, measure first** — Hybrid isn't free (extra index, latency, analyzer to maintain); with no exact-token needs it often buys nothing — let the metric decide.
</details>

---

## Section D — Reranking & retrieval strategy `[rag-systems] l4`

### Q16. Why not just run the cross-encoder reranker over the whole corpus and skip first-stage retrieval?

> **Difficulty:** 🟡 Medium · `[rag-systems] l4` · 🔮 Representative

- Cross-encoders are less accurate than bi-encoders
- ✅ It's far too slow — a cross-encoder must score every query–doc pair jointly and can't be pre-indexed
- Cross-encoders can't read long text

<details><summary>Show answer & why each option</summary>

- **Less accurate** — They're more accurate — that's exactly why we rerank with them.
- ✅ **Too slow, can't pre-index** — You can only afford it on a small shortlist; first-stage recall narrows millions of docs to ~50 cheaply.
- **Can't read long text** — You truncate to ~512 tokens, but the real blocker is per-query cost, not length.
</details>

### Q17. Users mostly ask narrow factual lookups ("what's the refund window for plan X?"). A teammate wants to switch the whole system to GraphRAG. Best call?

> **Difficulty:** 🟡 Medium · `[rag-systems] l4` · 🔮 Representative

- Yes — GraphRAG is strictly better than vector RAG
- ✅ No — keep vector RAG + rerank for lookups; reserve GraphRAG for whole-corpus sensemaking questions
- Add agentic multi-hop retrieval to every query instead

<details><summary>Show answer & why each option</summary>

- **Strictly better** — It isn't; on lookups vector RAG + rerank beats GraphRAG on quality and cost, and GraphRAG indexing is 5–10× pricier.
- ✅ **Match technique to query distribution** — GraphRAG's strength is "main themes / who are the actors" across a corpus, not narrow fact retrieval.
- **Agentic multi-hop everywhere** — Multiplies latency 4–7× for single-hop questions that don't need it.
</details>

### Q18. recall@50 is broken — the gold chunk often isn't retrieved at all. A teammate adds a cross-encoder reranker to fix it. Will it work?

> **Difficulty:** 🔴 Hard · `[rag-systems] l4` · 🔮 Representative

- ✅ No — a reranker only re-orders the shortlist; it can't recover a chunk that first-stage retrieval never fetched
- Yes — cross-encoders are accurate enough to find anything
- Yes, if you also raise `top_n` to 50

<details><summary>Show answer & why each option</summary>

- ✅ **No — reranking can't recover un-retrieved chunks** — Reranking fixes "missed top-ranked" (in the set but low), not "not retrieved." Broken recall needs chunking/hybrid/ANN-param/ColBERT fixes first.
- **Yes, finds anything** — A cross-encoder only scores candidates it's handed; if the gold chunk isn't in the shortlist, it's invisible.
- **Raise `top_n`** — Keeping more of a shortlist that already excludes the gold chunk still can't surface it.
</details>

### Q19. A grounded factual QA system over your own clean docs has good recall. A teammate proposes HyDE to "boost retrieval." Best call?

> **Difficulty:** 🟡 Medium · `[rag-systems] l4` · 🔮 Representative

- Add HyDE everywhere — it always improves retrieval
- ✅ Skip HyDE here — it helps zero-shot/domain-mismatch corpora, but can backfire on well-grounded factual lookups
- Replace retrieval with HyDE-only

<details><summary>Show answer & why each option</summary>

- **HyDE everywhere** — Its hypothetical answer can hallucinate details that pull retrieval away from the true passage.
- ✅ **Skip on grounded corpora** — HyDE shines when query and docs use different vocabulary; on a clean grounded corpus with good recall it adds a call and risk for little gain.
- **HyDE-only** — HyDE is a query transform feeding retrieval, not a retriever; on factual queries it's the wrong transform anyway.
</details>

### Q20. An interviewer asks why you don't just stuff the whole corpus into a 1M-token context and skip retrieval. Strongest answer?

> **Difficulty:** 🟡 Medium · `[rag-systems] l4` · 🔮 Representative

- Long context is cheaper than running a vector DB
- ✅ It loses on cost (per-token), latency (prefill), recall (lost-in-the-middle), and attribution (no chunk to cite) — retrieve to keep the window small, cheap, ordered, and citable
- Models can't actually read 1M tokens

<details><summary>Show answer & why each option</summary>

- **Cheaper** — The opposite — you pay for every dumped token on every query, which dwarfs retrieving a few chunks.
- ✅ **Loses on cost/latency/recall/attribution** — All four axes favour retrieval for corpus-scale QA; long context is a complement for holistic single-document tasks.
- **Can't read 1M** — They can ingest it, but recall degrades with length and the cost/latency/attribution problems remain.
</details>

---

## Section E — Vector index selection & scale `[rag-systems] l5`

### Q21. You must serve 800M vectors but your RAM budget is tight. Which index choice fits — and what's the cost?

> **Difficulty:** 🔴 Hard · `[rag-systems] l5` · 🔮 Representative

- HNSW — best recall, accept the RAM bill
- ✅ IVFPQ — ~20× less memory, then recover quality with a reranker on the shortlist
- It doesn't matter — all ANN indexes use similar memory

<details><summary>Show answer & why each option</summary>

- **HNSW** — At ~1.4 TB for a billion vectors it may simply not fit; "best recall" is moot if you can't host it.
- ✅ **IVFPQ + reranker** — Trades recall for a huge memory saving (~70 GB vs ~1.4 TB at 1B); reranking restores top-k quality.
- **All the same** — They differ by ~20× in memory at billion scale; index choice is a procurement decision.
</details>

### Q22. A multi-tenant RAG returns correct answers in testing. What's the highest-priority production risk to design against?

> **Difficulty:** 🔴 Hard · `[rag-systems] l5` · 🔮 Representative

- Slightly higher latency from metadata filtering
- ✅ Permission leaks — enforce ACL filters at retrieval time, before rerank/prompt assembly
- Choosing the wrong embedding model

<details><summary>Show answer & why each option</summary>

- **Latency from filtering** — Real but minor next to a data-isolation breach.
- ✅ **Permission leaks** — Cross-tenant leakage (filter bypass or injection that pulls privileged context) is the catastrophic failure; filter before a chunk can ever enter the prompt.
- **Wrong embedding model** — Matters for quality, but isn't the security/isolation risk that defines a multi-tenant system.
</details>

### Q23. Latency, throughput, and individual retrievals all look healthy, but users say answers are subtly out of date. Dashboards are green. What's the likely failure and how would you have caught it?

> **Difficulty:** 🔴 Hard · `[rag-systems] l5` · 🔮 Representative

- A capacity problem — add more replicas
- ✅ Silent freshness drift (stale docs or a vendor checkpoint change); catch it with top-k-overlap telemetry on a fixed probe set
- The reranker is misconfigured

<details><summary>Show answer & why each option</summary>

- **Capacity** — Throughput is healthy; adding replicas does nothing for stale or drifted representations.
- ✅ **Silent freshness drift + probe-set telemetry** — Recall can decay distributionally while per-request metrics stay green; only freshness/overlap telemetry on a probe set reveals it.
- **Reranker misconfigured** — Would show as precision problems on current docs, not systematically outdated answers.
</details>

### Q24. A 4-person startup is launching B2B RAG with ~2M vectors per customer and a strict data-residency requirement. Build or buy the vector store?

> **Difficulty:** 🔴 Hard · `[rag-systems] l5` · 🔮 Representative

- Self-host Vespa from day one for maximum control
- ✅ Buy a managed store, but pick one that supports per-tenant isolation / in-region (or VPC) deployment to satisfy residency
- Use one shared index with a metadata filter and ignore residency for now

<details><summary>Show answer & why each option</summary>

- **Self-host Vespa** — A 4-person team can't absorb the operational load of a self-hosted ANN engine; control isn't the binding constraint, residency is.
- ✅ **Buy managed with isolation/region features** — Below ~10M vectors managed is cheaper and faster to ship; residency is satisfied by isolation/region features, not by self-hosting.
- **Shared index, ignore residency** — Data residency is a legal constraint, not a "later" item, and a shared index risks cross-tenant leakage.
</details>

### Q25. You upgrade your embedding model and push the new vectors into the existing live index alongside the old ones. What happens?

> **Difficulty:** 🔴 Hard · `[rag-systems] l5` · 🔮 Representative

- Recall improves immediately since the new model is better
- Nothing — embeddings from different models are comparable
- ✅ Recall shears silently because old and new vectors aren't comparable; you must full-reindex behind a dual index and cut over

<details><summary>Show answer & why each option</summary>

- **Improves immediately** — New and old vectors live in different spaces; comparing across them is meaningless, so quality degrades.
- **Comparable** — They are not — a new model's vector space is not cosine-comparable to the old one (representation shearing).
- ✅ **Shears silently → dual index + cutover** — Mixing generations corrupts cross-comparisons; the safe path is a full re-embed in a parallel index, validate, then atomic cutover.
</details>

---

## Section F — RAG evaluation & the second input channel `[rag-systems] l6`

### Q26. Answers are fluent but keep stating facts that aren't in the retrieved context. Which metric most directly flags this?

> **Difficulty:** 🟡 Medium · `[rag-systems] l6` · 🔮 Representative

- Answer relevancy
- ✅ Faithfulness / groundedness
- Context recall

<details><summary>Show answer & why each option</summary>

- **Answer relevancy** — Measures whether the answer addresses the question, not whether its claims are supported.
- ✅ **Faithfulness / groundedness** — Checks that every claim is entailed by the retrieved context — though it degrades on numeric/multi-hop, so pair it with a confidence-aware check.
- **Context recall** — Measures whether you fetched the needed context, not whether the answer stuck to it.
</details>

### Q27. You're swapping embedding vendors; offline RAGAS scores improve. Safest way to ship?

> **Difficulty:** 🔴 Hard · `[rag-systems] l6` · 🔮 Representative

- Ship it — offline scores went up
- ✅ Shadow/canary on real production traffic, watching per-span hit@k on the long tail before full rollout
- Trust the vendor's benchmark numbers

<details><summary>Show answer & why each option</summary>

- **Ship it** — Offline sets over-sample easy queries and can be flattered by the new vendor's own tuning; the long tail can still collapse.
- ✅ **Shadow/canary + per-span hit@k** — Embedding swaps are the silent-drift case; only the production distribution + per-span tracing reveals long-tail recall loss.
- **Trust vendor benchmarks** — Vendor benchmarks rarely match your corpus or query distribution.
</details>

### Q28. You build a golden set by having an LLM write Q&A from chunks you hand-picked. Offline recall is 0.91; production recall is 0.40. What went wrong?

> **Difficulty:** 🔴 Hard · `[rag-systems] l6` · 🔮 Representative

- The production retriever is simply worse and needs a bigger embedding model
- ✅ The gold contexts came from hand-picked chunks, not the production retriever, so offline eval measured a corpus the live system never uses
- RAGAS computed faithfulness incorrectly

<details><summary>Show answer & why each option</summary>

- **Retriever is worse** — The gap is a measurement artifact — you graded against contexts the live system never surfaces.
- ✅ **Gold contexts weren't production-derived** — Generate gold questions, but harvest gold contexts from real production retrieval runs — otherwise the eval reports a fantasy that collapses in prod.
- **RAGAS bug** — This is a recall provenance problem, not a faithfulness-metric bug.
</details>

### Q29. A teammate uses GPT-4 as an LLM judge and reports its scores as the deploy gate, with no human comparison. What's the senior objection?

> **Difficulty:** 🔴 Hard · `[rag-systems] l6` · 🔮 Representative

- ✅ An unvalidated judge can be systematically wrong at scale — measure its agreement with human ground truth (Cohen's κ) and calibrate before trusting it to gate
- GPT-4 is too cheap to be a reliable judge
- Judges should always use a 1–10 quality score for nuance

<details><summary>Show answer & why each option</summary>

- ✅ **Calibrate the judge (κ)** — The judge is itself a model output needing evaluation; raw % agreement flatters, so report κ and iterate the judge prompt until it clears a bar.
- **Too cheap** — Cost isn't the issue; an uncalibrated judge's biases (position, verbosity, self-preference) are.
- **Always 1–10** — Backwards — narrow binary judge questions are more reliable than vague 1–10 scores.
</details>

### Q30. Your guardrail reliably blocks jailbreaks in the user prompt, but a study shows injecting documents into its context flips its judgments in a meaningful fraction of cases — and your retrieved chunks are exactly such documents. What's the lesson?

> **Difficulty:** 🔴 Hard · `[rag-systems] l6` · 🔮 Representative

- The guardrail just needs a higher threshold
- Disable retrieval for sensitive queries
- ✅ RAG has two input channels — the user prompt AND retrieved context; you must validate retrieved context (and whitelist tool calls), not just the user message

<details><summary>Show answer & why each option</summary>

- **Higher threshold** — Doesn't address the architectural gap: the retrieved-context channel isn't being checked at all.
- **Disable retrieval** — Guts the product; the fix is to validate the second channel, not remove it.
- ✅ **Validate both input channels** — Indirect prompt injection rides in through documents; guarding only the user side is negligent, since the retrieved channel can flip the guardrail's own judgments.
</details>

---

## Section G — End-to-end RAG design `[rag-systems] l7`

### Q31. Where should you enforce per-user document permissions in a RAG system?

> **Difficulty:** 🟡 Medium · `[rag-systems] l7` · 🔮 Representative

- In the prompt — tell the model to ignore docs the user can't see
- ✅ At retrieval time, as a metadata/ACL filter on the search (before rerank and prompt assembly)
- After generation, by redacting the answer

<details><summary>Show answer & why each option</summary>

- **In the prompt** — Prompt-level rules leak: the chunks were still retrieved and can surface, and injection can override the instruction.
- ✅ **At retrieval time** — If a chunk is never retrieved, it can never leak — enforce permissions in the query, then rerank/generate over the safe set.
- **After generation** — Too late — the model already saw forbidden context and may have used it.
</details>

### Q32. An interviewer asks: "It works in the demo — how do you know it's production-ready?" Best answer?

> **Difficulty:** 🟡 Medium · `[rag-systems] l7` · 🔮 Representative

- The answers are fluent and the stakeholders are happy
- ✅ It passes a split-eval gate on a production-derived golden set, clears injection tests, and meets the p99 latency/cost budget
- It uses the latest models and a managed vector DB

<details><summary>Show answer & why each option</summary>

- **Fluent & happy** — That's the demo. Fluency isn't faithfulness, and a few happy queries don't cover the long tail.
- ✅ **Measurable bar** — Production-ready spans retrieval, generation, safety, and performance — not a vibe check.
- **Latest models & managed DB** — Tooling choices don't prove correctness, safety, or that it won't regress.
</details>

### Q33. An interviewer says "design a Q&A assistant over our internal docs." What's the strongest first move?

> **Difficulty:** 🟡 Medium · `[rag-systems] l7` · 🔮 Representative

- Start drawing embed → retrieve → generate immediately to show you know the pipeline
- ✅ Ask scoping questions first — shape, scale, freshness SLA, tenancy/permissions, stakes — then design to those constraints
- Recommend the latest models and a managed vector DB up front

<details><summary>Show answer & why each option</summary>

- **Draw the pipeline** — Jumping to a generic diagram skips the requirements that determine the whole architecture — it reads as memorized.
- ✅ **Scope first** — "RAG" hides three different systems; clarifying shape/scale/freshness/tenancy/stakes lets you pick index, ingestion, and isolation deliberately.
- **Latest models up front** — Tool choices don't answer the requirements.
</details>

### Q34. Asked to size the index for 50M chunks of 1,536-d embeddings on HNSW, what do you say?

> **Difficulty:** 🔴 Hard · `[rag-systems] l7` · 🔮 Representative

- "A few gigabytes — embeddings are small"
- "It depends on the model"
- ✅ "~50M × 1,536 × 4 bytes ≈ 300 GB raw, ~450–600 GB with the HNSW graph — at this scale I'd weigh IVFPQ + a reranker"

<details><summary>Show answer & why each option</summary>

- **A few GB** — Off by two orders of magnitude; underestimating RAM is exactly the surprise that forces a mid-project re-architecture.
- **It depends** — True but evasive — the interviewer wants the arithmetic, which you can do from dimension and count alone.
- ✅ **Do the arithmetic on the spot** — N × dim × 4B × ~1.8, and connecting it to the index choice, is the senior signal.
</details>

### Q35. Three months in, the assistant's answers are subtly more outdated, but nothing was deployed and all dashboards are green. What does a strong design already have in place?

> **Difficulty:** 🔴 Hard · `[rag-systems] l7` · 🔮 Representative

- Auto-scaling to handle the load
- ✅ Freshness telemetry (top-k-overlap on a probe set) + CDC re-embedding + a last-verified timestamp — so silent drift is caught and corrected
- A bigger embedding model swapped in last week

<details><summary>Show answer & why each option</summary>

- **Auto-scaling** — Not a load problem; scaling does nothing for stale or drifted representations.
- ✅ **Freshness telemetry + CDC + last-verified** — Stale recall is invisible to latency/throughput; only distribution-level freshness metrics catch it, and CDC re-embeds keep the index current.
- **Bigger model last week** — An unplanned in-place swap would shear recall further — and still wouldn't explain drift you can't see.
</details>

### Q36. A new B2B customer asks for the assistant over their private wiki, with exact part-number queries common, ~3M docs, a 1.5 s budget, strict isolation, and weekly doc updates. What's the defensible end-to-end design?

> **Difficulty:** 🔴 Hard · `[rag-systems] l7` · 🔮 Representative

- Single shared index, dense-only, no reranker, nightly full reindex — keep it simple
- ✅ Per-tenant (or VPC) index; structure-aware + contextual chunks with ACL metadata; hybrid BM25+dense+RRF filtered at retrieval; rerank to 5; grounded cited answers; CDC weekly updates; split-eval gate
- GraphRAG over the whole wiki with a 1M-token context fallback

<details><summary>Show answer & why each option</summary>

- **Single shared, dense-only** — Dense-only misses part numbers, a shared index risks cross-tenant leakage, and full reindex wastes money vs CDC — fails on exact-match, isolation, and cost.
- ✅ **The full defensible design** — Matches every requirement: isolation (per-tenant/VPC), exact tokens (BM25 in hybrid), the 1.5 s budget (HNSW + ~120 ms rerank), freshness (CDC), and safety (ACL filter + grounding + eval gate).
- **GraphRAG + 1M context** — Part-number lookups are narrow fact retrieval — GraphRAG is 5–10× pricier to index and long-context blows the latency/cost budget.
</details>

---

## Section H — Reported RAG interview questions

> These are questions observed in real 2026 AI-engineer loops. Source: Adil Shamim, *"Every AI Engineer Interview Question You Need to Know in 2026 — from 100 Real Interviews"* — https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a · plus [KalyanKS-NLP/RAG-Interview-Questions-and-Answers-Hub](https://github.com/KalyanKS-NLP/RAG-Interview-Questions-and-Answers-Hub). No multiple choice — treat these as open prompts and rehearse a 60–90 second answer.

### Q37. How do you evaluate a RAG pipeline? Which metrics (NDCG, MRR, precision@k, recall)?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Separate retrieval from generation. Retrieval → recall@k, precision@k, MRR, NDCG@k against a production-derived golden set. Generation → faithfulness/groundedness + answer relevancy (RAGAS or a calibrated judge). Gate on both offline, then shadow/canary online. See Q26–Q29 above and [`content/rag.md`](../content/02-rag.md).

### Q38. What is hybrid search? Why combine vector + keyword (BM25)?
> **Difficulty:** 🟢 Easy · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Dense handles paraphrase/semantics; BM25 nails exact tokens (codes, SKUs, proper nouns). Fuse with RRF. Hybrid beats either alone by ~15–30% on mixed traffic. Mind the analyzer (Q13). See Q11–Q15.

### Q39. Why re-rank on top of vector retrieval? Cross-encoder vs bi-encoder?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Bi-encoder embeds query and doc independently (pre-indexable, cheap, first-stage recall). Cross-encoder scores the pair jointly (accurate, expensive, no pre-index → shortlist only). Recall and precision are different jobs. See Q12, Q16, Q18.

### Q40. Explain ANN search / HNSW indexing.
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** HNSW = navigable small-world graph, layered; `efSearch`/`M` trade recall for latency/memory (Q4). It's *approximate* — the config can be your recall ceiling. At billion scale, IVFPQ trades ~20× memory for recall you recover with a reranker (Q21). Be able to size the RAM (Q34).

### Q41. Where do embeddings fail — negation, temporal reasoning, precision?
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Dense embeddings blur exact tokens (error codes, part numbers → use BM25), struggle with negation ("not covered" vs "covered") and temporal/quantitative constraints. Mitigate with hybrid search, metadata filters, and contextual retrieval (Q6). See Q11, Q13.

### Q42. What are the key RAG tradeoffs (latency/accuracy, chunk-size/context, cost/quality)?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Smaller chunks → higher precision, lower recall context per chunk (Q7). Reranking → +precision, +latency/cost. Bigger k → +recall, +prompt cost + lost-in-the-middle. Every lever is a dial tied to the query distribution and SLO — name the counter-metric.

### Q43. Is RAG still relevant in the era of long-context LLMs?
> **Difficulty:** 🟡 Medium · ✅ Reported ([DSPy / dev.to system-design set](https://dev.to/arslan_ah/ai-system-design-interview-questions-chatgpt-rag-llm-inference-and-agents-1doi))
> **Senior frame:** Yes — long context loses to retrieval on cost (per-token), latency (prefill), recall (lost-in-the-middle), and attribution (no chunk to cite). Long context is a *complement* for holistic single-document tasks, not a replacement for corpus-scale QA. This is Q20 verbatim as a live prompt.

---

<sub>⬅ [Back to the index](./README.md) · Related: [Evals →](./evals.md) · [System design →](./system-design.md) · [LLMOps →](./llmops.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
