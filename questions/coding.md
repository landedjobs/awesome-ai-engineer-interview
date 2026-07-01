# Coding Round — Implement-From-Memory Patterns

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive & runnable code: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **8 patterns.** The coding screen (Round 2) and the AI/ML specialized onsite (Round 4c) ask you to *implement from memory* — attention, a transformer layer, a LoRA adapter — or *build a small service*. These reward muscle memory and clean reasoning under time pressure, not novelty. Each entry below gives the **prompt · difficulty · what it tests · a senior approach outline** (not a full solution — the runnable reference lives in [`content/`](../content/)).

---

### How to use this file

Don't read the outline first. Set a timer, write the code from memory, then compare. The outline names the *checkpoints an interviewer is listening for* — the parts that separate "compiles" from "understands." For the conceptual "why" behind each (scaling factor, softmax axis, low-rank delta), drill [fundamentals.md](./fundamentals.md) and [fine-tuning.md](./fine-tuning.md) first.

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (numerical care, shapes, or a service under constraints)
**Provenance legend** · ✅ Reported (real coding-round pattern, source linked)

---

### C1. Implement scaled dot-product attention (single head) from memory
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim §3.6](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [Karpathy Zero-to-Hero](https://karpathy.ai/zero-to-hero.html))

**What it tests:** shapes, the `1/√d_k` scaling, the softmax *axis*, causal masking, and whether you know *why* each piece is there.

**Senior approach outline:**
- Signature: `attention(Q, K, V, mask=None)` with `Q,K,V` shaped `(B, T, d_k)` (or `(B, h, T, d_k)`).
- `scores = Q @ K.transpose(-2, -1) / sqrt(d_k)` — **name the scaling out loud**: without it, logits have variance `d_k·σ²`, softmax saturates, gradients vanish ([fundamentals.md](./fundamentals.md) Q21).
- Apply the mask *before* softmax by setting future/pad positions to `-inf` (causal mask = upper-triangular).
- `weights = softmax(scores, dim=-1)` — **the last dim (keys)**; the classic bug is normalizing the wrong axis → uniform weights, flat loss ([fundamentals.md](./fundamentals.md) Q23).
- `return weights @ V`. Interviewer follow-ups: dropout on weights, and the `(B, n_head, T, T)` memory at scale (quadratic — [fundamentals.md](./fundamentals.md) Q24, Q46).
- Runnable, tested reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C2. Implement Multi-Head Attention from memory
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim §3.6](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))

**What it tests:** the split/merge reshape gymnastics, projection matrices, and the *resolution* argument for why heads beat one wide head at equal FLOPs.

**Senior approach outline:**
- Learned projections `W_q, W_k, W_v` (each `d_model → d_model`) and an output `W_o`.
- Reshape to `(B, T, n_head, d_head)` then transpose to `(B, n_head, T, d_head)`; `d_head = d_model / n_head`.
- Run C1 per head in parallel (batched over the head dim), concat back to `(B, T, d_model)`, apply `W_o`.
- **Say the why:** heads specialize on distinct relational subspaces a single averaged softmax would blur — same params/FLOPs as one d_model head ([fundamentals.md](./fundamentals.md) Q22).
- Follow-ups: GQA/MQA (share K/V heads to shrink the KV cache — [fundamentals.md](./fundamentals.md) Q41, Q49), and FlashAttention (never materialize the T×T matrix — Q42).
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C3. Implement a full Transformer (decoder) block from memory
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim §3.6](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [Karpathy](https://karpathy.ai/zero-to-hero.html))

**What it tests:** pre-norm vs post-norm, the residual stream, the FFN expansion, and correct wiring order.

**Senior approach outline:**
- **Pre-norm** block (the frontier default): `x = x + attn(norm1(x)); x = x + ffn(norm2(x))`. The residual carries the *unnormalized* `x` ([fundamentals.md](./fundamentals.md) Q35).
- FFN = `Linear(d → 4d) → activation → Linear(4d → d)`; mention SwiGLU as the modern gated variant, where most params/FLOPs live ([fundamentals.md](./fundamentals.md) Q33).
- Use RMSNorm/LayerNorm (per-token, batch-independent — Q32), a causal mask, and RoPE for position (relative-via-rotation — Q27).
- **Say the why for pre-norm:** stable gradients at init, scales to 80+ layers without warm-up ([fundamentals.md](./fundamentals.md) Q31, Q34).
- Follow-up: stack N blocks + embedding (tied to unembedding — Q19) + final norm + LM head. Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C4. Implement a LoRA adapter from scratch
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim §3.6](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [LoRA paper](https://arxiv.org/abs/2106.09685))

**What it tests:** freezing the base, the low-rank `B·A` factorization, scaling `α/r`, and understanding you approximate the *delta*, not the weight.

**Senior approach outline:**
- Wrap a frozen `Linear`: `A` shape `(r, in)`, `B` shape `(out, r)`; init `A` random (Kaiming), `B` **zeros** so the adapter starts as a no-op.
- Forward: `y = frozen(x) + (alpha / r) * (x @ A.T @ B.T)`. Only `A, B` require grad.
- **Say the why:** you're approximating the fine-tuning delta `ΔW`, which is intrinsically low-rank (LoRA found r=1–2 sufficed on GPT-3) — a rank-r factorization can't represent arbitrary `W` but *can* capture the update ([fine-tuning.md](./fine-tuning.md) Q11).
- Follow-ups: target *all* linear layers (coverage matches full FT — [fine-tuning.md](./fine-tuning.md) Q12); merging for inference gives *no* serving-memory win, only multi-adapter hot-swap does (Q13); QLoRA quantizes the frozen base to 4-bit (Q14).
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C5. Fix a bug in a given transformer/attention implementation
> **Difficulty:** 🟡 Medium · ✅ Reported ([OpenAI/Reddit loop write-up, Round 2](https://www.reddit.com/r/InterviewCoderHQ/comments/1rhfjpw/openai_swe_interview_experience_full_loop/))

**What it tests:** debugging under pressure, reading shapes, and recognizing the classic failure signatures.

**Senior approach outline:**
- The usual planted bugs: **softmax over the wrong dim** (→ near-uniform weights, flat loss — [fundamentals.md](./fundamentals.md) Q23); **missing `/√d_k`** (→ saturated softmax, tiny gradients — Q21); mask applied *after* softmax; wrong transpose in Q·Kᵀ; forgetting the causal mask (→ leakage from future tokens).
- Method: print tensor shapes at each step, check attention-weight rows sum to 1 over the key axis, verify a single-token gradient flows.
- Narrate the *symptom → cause* mapping — that's the signal, not just the patch.
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C6. Implement a ReAct agent loop that calls two tools and recovers from a tool error
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))

**What it tests:** the Thought→Action→Observation loop, structured tool errors, and bounding the loop.

**Senior approach outline:**
- Loop: model emits a `Thought` + `Action(tool_name, args)`; you dispatch the tool; feed the `Observation` back; repeat until the model emits a final answer.
- Tools return **structured errors** (`{status: "TRANSIENT", retry_after: N}` vs permanent) so the loop can branch retry / give-up / escalate ([agents.md](./agents.md) Q2, Q26).
- **Bound it:** max steps + a per-task cost ceiling + progress detection (no new info in K steps → stop/escalate) — a step cap alone isn't enough ([agents.md](./agents.md) Q10).
- Follow-ups: prefix-cache the stable system+tools prefix (cost is super-linear in trajectory — [agents.md](./agents.md) Q3); tool-schema overload past ~20–40 tools (Q15).
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C7. Build a gRPC service for financial report generation
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim §3.6](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))

**What it tests:** production API design around an LLM — proto schema, streaming, reliability, and structured output.

**Senior approach outline:**
- Define the `.proto`: a `GenerateReport` RPC with a request (params, template id) and a **server-streaming** response (stream tokens for TTFT/UX).
- Wrap the LLM call with the reliable-client stack: capped backoff + jitter, a client-side rate limiter, a circuit breaker, and a fallback with its own eval ([llmops.md](./llmops.md) Q1–Q5).
- Enforce **strict structured output** for the report's numeric fields (constrained decoding — [fundamentals.md](./fundamentals.md) Q11), with a validator that cited figures trace to a source span ([llmops.md](./llmops.md) Q14) — "hallucination-free" finance means grounding, not a promise.
- Add tracing spans (model, prompt_version, tokens, cost, latency), retries only on retryable status codes ([llmops.md](./llmops.md) Q2), and a timeout.
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

### C8. Implement semantic caching for an LLM endpoint
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), [Mavik cost guide](https://www.maviklabs.com/blog/llm-cost-optimization-2026))

**What it tests:** embeddings + ANN lookup, threshold discipline, and knowing what must *never* be cached.

**Senior approach outline:**
- On request: embed the query → ANN lookup in a small vector store → if top match ≥ similarity threshold, serve the stored answer; else call the model and write back with a TTL.
- **Threshold discipline:** too low serves wrong answers; tune against a labeled set. Best on repetitive FAQ traffic (40–80% hit); poor on creative/long-tail ([llmops.md](./llmops.md) Q7).
- **Never cache state-mutating calls** — a stale hit causes real damage ([llmops.md](./llmops.md) Q7). Contrast with *prefix* caching (exact-prefix KV reuse — [system-design.md](./system-design.md) Q13).
- Follow-ups: cache-key on normalized query + user scope; metrics on hit-rate and false-hit rate.
- Reference: [`content/coding.md`](../content/08-prompt-engineering-and-structured-output.md).

---

<sub>⬅ [Back to the index](./README.md) · Related: [Fundamentals →](./fundamentals.md) · [Fine-tuning →](./fine-tuning.md) · [Agents →](./agents.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
