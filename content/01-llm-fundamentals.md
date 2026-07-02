# LLM Fundamentals

The one lesson everything else on this track is built on: what an LLM *actually does* when you call it, and the transformer internals interviewers open with.

Back to [README](../README.md) · [All questions](../questions/README.md)

## What this tests in an interview

Whether you treat an LLM like a function (return value, free, instant) or like what it is: a **probabilistic text generator** with **finite working memory you rent by the token** and **latency that scales with how much it writes**. Almost every foundational LLM question — and every internals-round opener — is a direct consequence of those three facts plus the transformer that produces them. Lead with the mechanism, then the production number; that's the whole rubric.

---

## The generation loop: prefill, then decode

An LLM is **autoregressive**. Given the tokens so far it emits a *logit* for every token in its ~100k–200k vocabulary, softmaxes those into probabilities, **samples** one, appends it, and repeats until a stop token or a length cap. Everything we call "reasoning" or "instruction-following" is emergent behavior on top of that one loop. First engineering consequence: your output is a *sample from a distribution*, not a return value.

A single call runs in **two phases with inverted costs**:

| Phase | What it does | Bound by | Sets | Lever to improve |
| --- | --- | --- | --- | --- |
| **Prefill** | Ingests the whole prompt in **one parallel forward pass**, builds the KV cache | **Compute** (big parallel matmuls) | Time-to-first-token (TTFT) | Shrink or prompt-cache the **prompt** |
| **Decode** | Generates output tokens **one at a time**, each pass reads the growing KV cache | **Memory bandwidth** (stream the cache from HBM) | Total time | Shrink the **output**, cut KV bytes, faster model |

```
ONE LLM CALL = PREFILL + DECODE
    prefill ── all N prompt tokens in ONE parallel pass ──► build KV cache
              cost ∝ input length; this is most of your TTFT
    decode  ── tok ── tok ── tok ── ...
              each = one forward pass reusing the KV cache; ~constant per token
    total decode time ∝ NUMBER OF OUTPUT TOKENS
    TTFT  ≈ prefill time                          (shrink the PROMPT)
    total ≈ TTFT + out_tokens × inter-token-latency (shrink the OUTPUT)
```

This is why "why is the first token slow but the rest stream out fast?" is a classic warm-up: the wait before token one is prefill over your whole prompt; after that each token is a cheap incremental step. It's also why a 4,000-in / 50-out call feels snappy while a 500-in / 2,000-out call feels slow — **output length dominates total latency, input length dominates TTFT.**

> [!TIP]
> You optimize the two latencies with *different* levers. To cut **TTFT**, shrink or cache the prompt (prefill). To cut **total time**, shrink the output, stream it, or pick a faster-decoding model. Conflating them is the most common cost/latency mistake juniors make.

## Tokens: the unit of cost, latency, and limits

Models don't see words or characters — they see **tokens**, sub-word pieces from a byte-pair-encoding (BPE) vocabulary. Rule of thumb for English: **~4 characters ≈ 1 token** (~0.75 words/token). Code, JSON, and non-English text tokenize *less* efficiently — 2–4× more tokens per character — which quietly inflates cost and eats context budget. Tokens are the unit of *everything* commercial: pricing, latency, rate limits, and the context-window cap are all counted in tokens.

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4o")

def n_tokens(text: str) -> int:
    return len(enc.encode(text))

prompt = open("contract.txt").read()
print(n_tokens(prompt))          # e.g. 9,142 -> ~$0.02 input on a mid-tier model

# tokenization is why models "can't spell": they never see characters.
enc.encode("strawberry")         # -> a few sub-word fragments, not 10 letters
# so "how many r's in strawberry?" is a tokenization artifact, not a reasoning
# failure. Same root cause as shaky digit-by-digit arithmetic.
```

Two pricing facts to memorize. **(1) Output tokens cost ~3–5× more than input tokens** on most providers — because decode is sequential and compute-heavy while prefill is parallelized. **(2)** A long stable prefix can be **prompt-cached** for up to ~90% off its input cost. Together these mean the cheapest feature is one with a *big cached prompt and a short output* — exactly backwards from "shorter prompts are cheaper."

**Whiteboard the cost/latency estimate.** "Roughly what will this feature cost per month, and what's its latency?" is a standard senior screen — they're testing whether you think in tokens. State your assumptions out loud (model, token counts, price per million); that's what's graded.

```python
# Back-of-envelope a support assistant: 200k requests/day, mid-tier model
reqs_per_day = 200_000
in_tokens, out_tokens = 1_200, 250
price_in, price_out = 2.50 / 1e6, 10.00 / 1e6   # note: output ~4x input

daily = reqs_per_day * (in_tokens * price_in + out_tokens * price_out)
print(round(daily), round(daily * 30))          # ~$1,100/day  ~$33k/month
# latency: TTFT tracks the 1,200-token prefill; total tracks the 250 output tokens.
# Cut output -> cut total time AND cost.
```

> [!WARNING]
> **"Just give it a bigger model to count letters / do arithmetic."** Tokens are sub-word byte fragments; the model never sees individual characters, and numbers fragment by corpus frequency (`127` is often one token, `677` two). The fix is never a bigger model — it's a *tool* (code execution, a calculator) for character/number work, or a digit-splitting tokenizer policy. You're fighting the vocabulary, not the reasoning.

## The context window is working memory — and it rots

The context window (e.g. 200k tokens) is the model's **working memory for one call** — system prompt, conversation history, retrieved documents, tool definitions, tool results, *and* the output all share that one budget. It is not long-term memory; nothing persists across calls unless you put it back in. Treating the window like a database is the root of a whole class of bugs.

Two empirical facts every senior must know:

- **Lost in the Middle** (Liu et al., 2023): recall is **U-shaped** — facts at the *start* or *end* of a long context are recalled reliably; facts buried in the *middle* are frequently missed.
- **Context Rot** (Chroma, 2025): answer quality degrades as input grows *even in frontier models*, and it's a **reasoning** degradation, not just a retrieval miss.

The upshot: **more context can make answers worse, not just slower.** There's a cost reason too — attention grows super-linearly with sequence length and the KV cache grows linearly in memory, so a 100k-token context is far more expensive to serve than its input-token price suggests. GitHub Copilot deliberately caps prompts around **6,000 characters** to keep fast models inside its latency envelope — treating the window as scarce.

> [!WARNING]
> **"Bigger context window = better."** Bigger context is not free memory. You pay for every token in cost *and* latency, and the model reasons *worse* as you fill it (Lost-in-the-Middle + Context Rot). Treat the window as a scarce, **ordered** resource — curate what goes in and put what matters at the *edges*, not the middle.

**"The doc is bigger than the window — what do you do?"** The wrong answer is "use a bigger-context model." Walk the menu and pick by constraint: **chunk + retrieve (RAG)** the few relevant passages (default; cheapest, best recall on targeted questions); **map-reduce** summarize when the question needs the whole document; **hierarchical** summaries for huge corpora; reserve **long-context stuffing** for when the whole doc genuinely fits *and* the task is holistic.

```python
# map-reduce summarization: the answer to "summarize a doc bigger than the window"
def summarize_large(doc, chunk):
    parts = [summarize(c) for c in chunk(doc)]          # MAP: per chunk (parallel)
    while n_tokens("\n".join(parts)) > WINDOW_BUDGET:
        parts = [summarize("\n".join(g)) for g in groups_of(parts, 5)]  # REDUCE: fold
    return summarize("\n".join(parts))                  # final pass over folded summaries
```

## Sampling: temperature, top-k, top-p

Sampling is the only knob end users see, and it's three layers on the *same* softmax over logits.

| Knob | What it does | Typical use |
| --- | --- | --- |
| **Temperature** | Divides logits before softmax: `softmax(logits/T)`. `T<1` sharpens toward argmax (repetitive); `T>1` flattens (varied); `T=0` = greedy | Tune this first |
| **Top-k** | Keeps only the `k` highest-probability tokens, then samples (`k=1` = greedy) | Coarse cap |
| **Top-p (nucleus)** | Keeps the smallest set of tokens whose cumulative probability exceeds `p` — adaptive: small nucleus when one token dominates, large when flat | Leave near default |

They compose: apply temperature, then top-k/top-p, then renormalize and sample. **Tune temperature *or* top-p, not both at once** — they interact and make behavior hard to reason about. Practical settings: extraction/classification/structured output → 0–0.2; balanced assistant/Q&A → ~0.7; brainstorming/creative → 0.9–1.2.

> [!WARNING]
> **"temperature=0 is deterministic."** It makes decoding *greedy* (always take the argmax) — but greedy ≠ deterministic. Real systems still vary run-to-run from **batched inference, floating-point non-associativity on GPUs, mixture-of-experts routing, and silent provider model updates.** A `seed` helps reproducibility but is best-effort, not a contract. Never write `assert output == expected_string` — enforce a **schema**, validate, and gate changes with **evals**.

## Reasoning models & the knobs that came with them

Modern **reasoning models** (extended-thinking modes) generate a hidden chain of "thinking" tokens before the visible answer. They lift accuracy on hard, multi-step problems — but you **pay for those hidden tokens**, they add latency, and streaming them is a common source of surprise bills. Newer APIs expose a **reasoning-effort** knob (low/medium/high) to trade quality for cost/speed. Senior instinct: reasoning models for genuinely hard planning/math/code; standard models for the latency-sensitive, high-volume, "just extract this" majority — and you can **route** per request rather than choosing globally.

> [!NOTE]
> **Silent regressions are real.** Anthropic's April 2026 Claude Code postmortem traced two months of "it got dumber" reports to *three interacting changes* shipped weeks apart — a default reasoning-effort cut (high → medium), a caching bug that dropped older thinking from idle sessions, and a verbosity-reduction prompt tweak. Individually small; together they read as broad intelligence loss. The safeguard: **pin model and prompt versions, and run an eval on every change.**

---

# Transformer internals

Everything above is *behavior*. This is the machine that produces it — the part interviewers open an internals round with. Be able to write `softmax(Q·Kᵀ/√d_k)·V` and the variance argument on a whiteboard.

## Tokenization & BPE

Modern GPT-style models use **byte-level byte-pair encoding (BPE)**. The base alphabet is not letters — it's the **256 raw UTF-8 bytes**. Training starts from those 256 single-byte tokens, repeatedly finds the *most frequent adjacent pair*, merges it into a new symbol, and records the merge — until it hits a target vocab size. GPT-2's vocab is **50,257** = 256 bytes + ~50,000 learned merges + 1 end-of-text token. Decode replays the recorded merges strictly in learned order.

Byte-level gives one enormous property: **zero out-of-vocabulary, ever** — any string, any language, any emoji, any binary blob is representable (worst case: fall back to single bytes). The cost: the smallest nameable unit is one byte, and most non-ASCII characters are multiple UTF-8 bytes, so CJK/Arabic/emoji run 2–4× hotter than English. The tokenizer emits `(B, T)` integer ids; the model's first act is a lookup into the **token embedding table** `(vocab_size, d_model)` → `(B, T, d_model)`. That table is usually **tied** to the output unembedding (for GPT-2 124M it's 50,257 × 768 ≈ 38.6M params — a third of the model), so vocab size is a real parameter cost.

Failure modes that ship to prod and survive every prompt tweak because they live in the vocab: inconsistent number splits (wrecks arithmetic), non-ASCII cost blowup, code fragmentation, and **glitch tokens** (under-trained entries like `SolidGoldMagikarp` that produce unsteerable output). The senior intervention: inspect token ids of representative inputs before assuming character-level behavior, and consider retraining BPE on your domain.

## Self-attention, from scratch

The full equation, from Vaswani et al. (2017): `Attention(Q, K, V) = softmax(Q·Kᵀ / √d_k)·V`. Read it as a soft, content-based dictionary lookup. For each token we form three vectors by learned linear projections of the *same* input: a **query** ("what am I looking for?"), a **key** ("what do I contain?"), a **value** ("what do I deliver if attended to?"). `q·k` scores how well a query matches each key; softmax over the keys turns scores into weights summing to 1; we take that weighted sum of value vectors. **Self**-attention: Q, K, V from the same sequence. **Cross**-attention: queries from one sequence, keys/values from another.

Why is this a leap over the RNN it replaced? An RNN compresses all prior context into one fixed hidden state passed step by step, so token 1 reaches token 500 only after 499 lossy sequential hops. Attention deletes the bottleneck: token 500's query reads token 1's value *directly*, in one hop — and every position computes in parallel during training, which made scale tractable. The **causal mask** (`mask[i,j] = -inf for j > i`) is what makes a decoder (GPT) autoregressive vs an encoder (BERT); it's the mechanical expression of "decode left-to-right."

### Why divide by √d_k — the exact variance argument

This is where interviewers separate "normalization trick" (weak) from a real derivation (strong). Assume each component of `q` and `k` is i.i.d. with mean 0, variance σ². Then `q·k = Σ qᵢkᵢ` is a sum of `d_k` independent zero-mean terms, so it has mean 0 and **variance `d_k·σ²`** — it grows linearly with head dimension. Feed large-variance logits into softmax and it **saturates**: one logit dominates, the distribution goes near-one-hot, and the gradient through softmax vanishes. Dividing by **√d_k** (the standard deviation of the sum) rescales variance back to σ², keeping softmax in its responsive, trainable regime.

```
WHY 1/sqrt(d_k), NOT 1/d_k, NOT 1
    q, k components: i.i.d., mean 0, variance s^2
    q . k  = sum of d_k terms -> mean 0, VARIANCE = d_k * s^2  (grows with d_k)
    divide by sqrt(d_k):  Var(q . k / sqrt(d_k)) = s^2   (back to unit scale)
    too big a logit -> softmax saturates -> ~one-hot -> gradient ~ 0 -> no learning
    Vaswani: large d_k "pushes softmax into regions of extremely small gradients."
```

### Multi-head: a shape split, not more compute

`MultiHead(Q,K,V) = Concat(head₁,…,head_h)·Wᴼ`, where each head projects into a smaller `d_k = d_model/h` subspace (Vaswani base: `d_model=512`, `h=8`, `d_k=64`), runs scaled dot-product attention in parallel, then all heads concatenate back to `d_model` and mix through `Wᴼ`. Total params and per-token FLOPs **match a single d_model-wide attention exactly.** What you gain is **resolution**: a single softmax must average all attended positions into one output, blurring fine-grained multi-relation structure; `h` smaller heads let each specialize (one tracks subject→verb, another local n-grams, another a delimiter/first-token "sink"). Training drives heads to differentiate because identical heads waste capacity.

> [!WARNING]
> **"Multi-head attention costs more compute — it's several attentions for more power."** False premise. Concatenated `d_k=64` heads at `h=8` use the *same* parameters and FLOPs as one `d_model=512` attention. The win is representational, not computational — per-head resolution — and it's exactly the property MQA/GQA trade away for serving speed.

### The exact tensor shapes through one block

Whiteboard these until automatic — "implement MHA" is a standard coding round, and the bugs they watch for are *shape* bugs: wrong softmax axis, missing transpose, forgetting `.contiguous()`, a double-softmax.

```
TENSOR SHAPES, ONE BLOCK (GPT-2 124M: d_model=768, n_head=12, d_head=64)
    x                (B, T, 768)
    -> Q,K,V proj    each (B, T, 768)
    -> reshape heads (B, 12, T, 64)          split 768 = 12 * 64
    -> Q @ K^T       (B, 12, T, T) / sqrt(64), causal mask, softmax over LAST dim (keys)
    -> @ V           (B, 12, T, 64)
    -> merge heads   (B, T, 768)
    -> output proj   (B, T, 768)
    -> residual add  (B, T, 768)
```

```python
import torch

def attention(q, k, v, causal=True):        # q,k,v: (B, n_head, T, d_head)
    d_head = q.size(-1)
    att = (q @ k.transpose(-2, -1)) / d_head**0.5        # (B, n_head, T, T)
    if causal:
        T = q.size(-2)
        mask = torch.tril(torch.ones(T, T, device=q.device)).bool()
        att = att.masked_fill(~mask, float("-inf"))       # block attending to the future
    att = att.softmax(dim=-1)                              # over KEYS (last dim)
    return att @ v                                         # (B, n_head, T, d_head)
```

### The quadratic cost — two distinct phenomena

Attention has **two** quadratic costs, and conflating them is the most common production-analysis bug. **Compute** is `O(T²·d)`: forming `Q·Kᵀ` materializes a `T×T` matrix, `(att·V)` multiplies it back. **Memory I/O** is `O(T²·d + T·d²)`: a naive kernel writes the `T×T` matrix to GPU HBM after softmax and reads it back, so even with free FLOPs the traffic is quadratic. As sequences grew, the binding constraint *migrated* from "too much math" to "too much memory movement."

```
THE QUADRATIC, BY THE NUMBERS (per layer, T = sequence length)
    T x T attention matrix grows with T^2:
    T = 1k  ->     1,000,000 entries / head
    T = 8k  ->    64,000,000 entries / head   (64x the 1k case)
    T = 32k -> 1,073,741,824 entries / head   (~1B, per head, per layer)
    compute:  O(T^2 * d)           the matmuls
    HBM I/O:  O(T^2 * d + T*d^2)    writing/reading the T x T matrix (the real wall)
    vs RNN:   O(T * d^2) total, but SEQUENTIAL -> the trade we made.
```

> [!WARNING]
> **"Attention is O(n²), so decode is compute-bound."** Two errors in one. First, transformer attention is *massively parallel* on a GPU — an RNN's O(n) work is strictly sequential, so transformers train far faster despite more FLOPs. Second, the long-context wall is **memory I/O** (writing the T×T matrix to HBM), not FLOPs — and at **decode** the bottleneck is streaming the KV cache from HBM, i.e. **bandwidth-bound**, not compute-bound. FlashAttention (exact, not approximate) wins 2–3× precisely by never materializing that matrix.

## Positional encoding & RoPE

Self-attention is **permutation-invariant**: shuffle the tokens and the set of `q·k` scores is identical, because the dot product doesn't know *where* either token sits. Language is ordered — so every transformer injects position somewhere. The scheme you pick decides whether the model can run *longer* than it trained on, with zero extra params or a brittle retrain.

| Scheme | How position enters | Relative? | Extend context? |
| --- | --- | --- | --- |
| **Sinusoidal** (Vaswani) | Add fixed sin/cos vector to embedding | approx | Degrades past train length |
| **Learned-absolute** (GPT-2) | Add trainable `PE[pos]` | no | **No row past max_len → retrain** |
| **RoPE (rotary)** | Rotate Q,K by `m·θ` per dim-pair | **yes (exact)** | **Yes, via base/freq scaling** |
| **ALiBi** | Linear distance penalty on scores | yes | Strong length extrapolation |

**RoPE** (RoFormer, Su et al. 2021) is the 2023–2026 default (Llama, Mistral, Gemma, Qwen). Instead of *adding* a position vector, it **rotates** each query and key by an angle that scales with position: group `d` dims into `d/2` pairs; pair `i` at position `m` is rotated by `m·θᵢ` where `θᵢ = 10000^(−2i/d)`. The identity that makes it brilliant: after rotation, `⟨R_m·q, R_n·k⟩ = ⟨q, R_(n−m)·k⟩` — the score depends **only on the relative offset (m − n)**. So RoPE applies *absolutely* (each token rotated by its own index) yet makes attention behave *relatively*, with **zero parameters and no position table**, plus built-in long-range decay.

```python
import torch
# RoPE: rotate each (even,odd) dim-pair of q/k by an angle that scales with position.
def rope(x, base=10000.0):                    # x: (B, T, n_head, d_head), d_head even
    B, T, H, D = x.shape
    pos = torch.arange(T).float()[:, None]    # (T,1)
    i = torch.arange(0, D, 2).float()         # pair index
    theta = base ** (-i / D)                  # (D/2,)
    ang = pos * theta                         # (T, D/2) = m * theta_i
    cos, sin = ang.cos(), ang.sin()
    x1, x2 = x[..., 0::2], x[..., 1::2]       # even, odd dims
    xr1 = x1 * cos[None, :, None, :] - x2 * sin[None, :, None, :]
    xr2 = x1 * sin[None, :, None, :] + x2 * cos[None, :, None, :]
    return torch.stack([xr1, xr2], dim=-1).flatten(-2)   # back to (B,T,H,D)
# apply to q and k BEFORE the dot product; never to v. 0 learned params.
```

Because RoPE's frequencies are tied to position, you can **extend context after training** by rescaling them — **Position Interpolation (PI)**, **NTK-aware** scaling, and **YaRN** (reaches 100k+ with ~10× fewer training tokens than PI). The trap: naive scaling **degrades retrieval before it degrades perplexity** — validate with **needle-in-a-haystack**, not perplexity.

> [!WARNING]
> **"Positional info is free / RoPE just rotates Q and K, that's it."** RoPE rotates **q** and **k** — never **v** — because position must influence *which* tokens attend (the score), not the content aggregated. And position is baked into the architecture: absolute schemes have no representation past their trained length; RoPE can be *scaled* but naive scaling silently wrecks long-range retrieval while perplexity looks fine. Extending context is a retrain-or-scale decision validated with needle-in-a-haystack, not a flag.

## The transformer block: pre-norm, RMSNorm, FFN

A modern block (nanoGPT / Llama / Mistral shape) is two **pre-norm** residual sublayers: `x = x + attn(norm(x))` then `x = x + ffn(norm(x))`. The **residual stream** `x` flows straight through, untouched, and each sublayer reads a *normalized copy* and adds a delta back. That clean additive highway is (1) a direct route for gradients backward (no vanishing through dozens of layers) and (2) easier optimization — each sublayer learns a *correction*, not a full transform.

```python
import torch.nn as nn
class Block(nn.Module):                        # the modern (pre-norm) transformer block
    def __init__(self, dim, n_head):
        super().__init__()
        self.norm1 = nn.RMSNorm(dim)           # RMSNorm, not LayerNorm
        self.attn = CausalSelfAttention(dim, n_head)
        self.norm2 = nn.RMSNorm(dim)
        self.ffn = SwiGLU(dim)                  # gated FFN, ~2/3*4*dim inner
    def forward(self, x):
        x = x + self.attn(self.norm1(x))        # pre-norm; residual add AFTER
        x = x + self.ffn(self.norm2(x))
        return x                                 # unnormalized x is the highway forward
```

**Pre-norm vs post-norm** is not stylistic. The 2017 transformer used post-norm `LayerNorm(x + Sublayer(x))`; every modern LLM uses pre-norm `x + Sublayer(LayerNorm(x))`. Xiong et al. (2020) showed post-norm's early-layer gradients are unstable and *require* learning-rate warm-up to train at all; pre-norm keeps a clean unnormalized residual path, so gradients behave at init and **warm-up becomes optional**. At 70B this is often the difference between "trains" and "diverges."

| | LayerNorm | RMSNorm (modern default) |
| --- | --- | --- |
| Formula | `(x − mean)/√(var+ε)·γ + β` | `x/√(mean(x²)+ε)·γ` |
| Drops | — | mean-centering **and** the `β` bias |
| Cost | baseline | one fewer mean, subtraction, bias per norm site |
| Both are | per-token across features (**not** per-batch → why neither is BatchNorm) | |

The **FFN** transforms each token *independently* — a 4× inner expansion (`768 → 3072 → 768` for GPT-2) — and is where **most params and FLOPs actually live** (attention decides *what to look at*; the FFN does the *per-token thinking*). Modern LLMs swap ReLU for **SwiGLU** (gated, three matrices, inner ≈ `2/3·4·d_model`). The crystallized 2023–2026 stack: **pre-norm, RMSNorm, RoPE, SwiGLU, bias-free layers** — each removes a training-stability or efficiency failure that bites at scale. **MoE** is a twist: many expert FFNs with a router activating a few per token, decoupling parameter count from active FLOPs (cheap compute, huge memory) at the cost of expert-parallel serving.

> [!TIP]
> Multi-head attention is the bridge between the math and the serving lever two sections down: heads exist to preserve relational resolution, and **MQA/GQA are precisely a controlled sacrifice of key/value resolution to shrink the KV cache.** Understand heads and you already understand why GQA-8 barely hurts quality.

## KV cache: the memory formula

The **KV cache** stores K and V for every previous token at every layer, so decode doesn't recompute them — turning generation from O(T²) cumulative recompute into O(T) per step. HuggingFace measured **61s → 11.7s (5.21×)** on a T4 just by enabling `use_cache=True`. It exists only at inference: training is one parallel teacher-forced pass, so there's no token-by-token reuse to cache. Memorize the formula:

```
KV CACHE = 2 * L * B * T * H_kv * D_h * dtype_bytes        (factor 2 = K and V)
    Llama 7B (L=32, H_kv=32, D_h=128, FP16):
        per token = 2*32*32*128*2 = 524,288 bytes = 0.5 MB
        T=4096, B=1 -> 2 GB        T=4096, B=8 -> 16 GB
    Llama 2 70B (L=80, H_kv=8 via GQA, D_h=128, FP16):
        per token = 2*80*8*128*2  = 327,680 bytes ~= 0.31 MB
        T=128k, B=1 -> ~39 GB (KV cache ALONE -> the floor for 128k serving)
    Note: 70B has FEWER KV bytes/token than 7B -> GQA (8 vs 32 heads).
```

Sit with that last point: a **70B has fewer KV bytes per token than a 7B**, because it uses GQA with 8 KV heads vs 32. That single architecture choice — KV head count — not parameter count, is the dominant serving-cost lever for long context. At long context the question stops being "how big is the model" and becomes "how big is the cache."

> [!WARNING]
> **"The KV cache is a speedup, so it's basically free."** It avoids *recompute* but costs *memory* — linear in context and batch, with a heavy per-token constant (0.5 MB/token for Llama 7B). A 70B at 128k needs ~39 GB of cache alone, and **decode is then bottlenecked streaming it from HBM** (~2 TB/s on an A100 vs ~19 TB/s on-chip SRAM). Long context is paid for in KV memory and bandwidth, not saved by the cache.

## MHA → GQA → MQA, FlashAttention, speculative decoding

**Cut the cache at the source with fewer KV heads.** Standard MHA gives every query head its own K/V. **MQA** keeps all `H` query heads but shares a *single* K/V head (cache ÷ H). **GQA** interpolates: `G` groups, each of `H/G` query heads sharing one K/V head.

| Variant | Query heads | KV heads | Cache vs MHA | Quality | Used in |
| --- | --- | --- | --- | --- | --- |
| **MHA** | H | H | 1× (baseline) | baseline | orig Transformer, BERT |
| **MQA** | H | 1 | 1/H | slight drop | PaLM, Falcon |
| **GQA** | H | G (e.g. 8) | G/H | ≈ MHA | Llama 2 70B, Mistral |
| **MLA** | H | latent/compressed | lowest | ≈ MHA | DeepSeek-V2 |

GQA-8 recovers essentially all of MHA's quality at near-MQA speed, so it's the default for new long-context models; you can **uptrain** an MHA checkpoint into GQA (mean-pool per-group K/V weights) for only **~5% of pre-training compute** — decide *before* fine-tuning.

**FlashAttention** (Dao et al., 2022) is not an approximation and not cleverer math — it's the *same exact attention*, reorganized to minimize HBM traffic. A naive kernel materializes the `T×T` matrix in HBM; FlashAttention **tiles** Q/K/V into blocks that fit in fast SRAM, computes softmax incrementally, and **never writes the full T×T matrix to HBM**. Output is bit-for-bit identical. FA-1 gave 3× on GPT-2; FA-2 hit 50–73% of A100 peak FLOPs; FA-3 reaches ~740 TFLOPS FP16 on Hopper. **The win is IO-awareness, not smarts** — memory movement, not FLOPs.

**Speculative decoding** attacks the same bandwidth wall from the other side. A small cheap *draft* model proposes the next `k` tokens; the big *target* model verifies all of them in a **single parallel forward pass** (verification is parallel, like prefill). Accepted tokens are kept, the first rejection truncates. You get **2–3× throughput** with *identical* output distribution — it's exact, not lossy — as long as the draft is much cheaper and agrees often. The applied-scientist rubric bundles this: **"KV caching + speculative decoding (2–3×) + INT8/FP8 quantization."**

> [!TIP]
> Speculative decoding is a direct attack on the bandwidth wall: it amortizes one expensive target-model pass over several tokens by letting a cheap draft guess ahead. The win scales with the draft's acceptance rate, and unlike most speedups it changes *nothing* about the output distribution.

## Putting it together: one forward pass, one deployment

```
FORWARD PASS: "the cat sat" -> next-token logits  (GPT-2 124M, FP16)
    text "the cat sat"
      | tokenizer (byte-level BPE, ~4 chars/token)
      v ids (B, T) = (1, 3)
      | token embedding table (50257 x 768), tied to unembed
      v x (1, 3, 768)
      | + position info (RoPE rotates Q,K)
      v
    -- repeat for each of 12 blocks -------------------------------
      x -> RMSNorm (1, 3, 768)             pre-norm
        -> Q,K,V (1, 3, 768) -> heads (1, 12, 3, 64)
        -> Q @ K^T / sqrt(64) (1, 12, 3, 3), mask, softmax, @V, merge, W^O
      x = x + attn(...)  (1, 3, 768)       residual add
      x = x + ffn(norm(x))  768->3072->768 (SwiGLU)
    ---------------------------------------------------------------
      | final norm + unembed (768 -> 50257)
      v logits (1, 3, 50257) -> take last row, sample
```

The `(B, n_head, T, T)` score matrix is the quadratic object FlashAttention refuses to materialize; the FFN holds most per-token FLOPs; the final unembed produces `(B,T,vocab)`, and you sample from the *last* position. At **decode** the new token is a single position `T_new=1` whose query attends over all cached keys → score `(B, n_head, 1, T_cached)`, not a full `T×T` — same diagram, inverted cost profile.

**Sizing a real deployment** — Llama-2-70B, GQA-8, FP16, 32k context: weights ~140 GB (multi-GPU regardless); KV/token ≈ `2·80·8·128·2 = 0.31 MB`; at 32k that's ~10 GB *per request*, ~40 GB at batch 4. **The KV cache — not the weights — is what scales with your concurrency and context.** Levers to fit an SLO: FP8 weights (140 → ~70 GB on Hopper), FP8/INT8 KV cache, GQA (already on), **PagedAttention** (fragmentation waste 60–80% → <4%), **continuous batching** (Orca 36.9×), FlashAttention.

> [!TIP]
> The internals aren't separate topics — they're five interfaces to the *same* forward pass. Pick a tokenizer and you've picked your worst-case tasks *and* your token cost; pick a position scheme and you've picked your context ceiling; pick an attention scheme (MHA/GQA/MQA) and you've picked your serving-cost curve. Quality and cost are the same forward-pass decision.

---

## Interview angles

A menu of the questions you'll actually get. Lead with the mechanism, then the production number.

- **"Walk me through what happens when you call an LLM."** → tokenize → **prefill** (parallel, builds the KV cache, compute-bound, sets TTFT) → **decode** (one token/pass, reads the KV cache, bandwidth-bound, sets total time) → stop. Output is a *sample*, not a return value.
- **"Why is the first token slow but streaming fast?"** → prefill processes the whole prompt in one parallel pass (TTFT); each decode step is a cheap incremental pass reusing the cache. It's not weight-loading — weights are already resident.
- **"Explain self-attention."** → content-based mixing → `q·k` scores → softmax over keys → weighted sum of values; every token reaches every other in one hop, killing the RNN recurrence bottleneck; decoders **mask the future**. Volunteer the masking variant before they ask.
- **"Why divide by √d_k?"** → `q·k` has variance `d_k·σ²`; large logits saturate softmax into its near-zero-gradient regime; √d_k restores unit scale so gradients flow. (Not "it keeps numbers from blowing up.")
- **"Self-attention vs multi-head — why 12 heads of 64 not one of 768, same FLOPs?"** → **resolution**: one softmax averages all positions into mush; separate heads specialize on different relations at *identical* cost. It's a shape split, not more compute.
- **"GQA vs MHA — what does GQA give up?"** → some key/value resolution (distinct K/V subspaces per head) in exchange for an H/G× smaller KV cache; query-head diversity is preserved, which is why GQA-8 barely hurts quality. Uptrain MHA→GQA for ~5% compute.
- **"Why do transformers need positional encoding?"** → self-attention is permutation-invariant; without position, `q·k` doesn't depend on *where* tokens sit, so order is invisible. RoPE rotates Q,K so the score depends only on the offset `m−n` — relative effect from an absolute operation, zero params, extensible via YaRN.
- **"Implement attention from memory."** → walk the shapes: `(B,T,d_model)` → per-head `(B,h,T,d_head)` → scores `(B,h,T,T)` / √d_k → causal mask → **softmax over keys (last dim)** → `@V` → merge → `Wᴼ` → residual. Name the `√d_k` scale and the common bugs (wrong softmax axis, missing transpose, double-softmax).
- **"How does tokenization affect performance?"** → ~4 chars/token English; code/JSON/non-English run 2–4× hotter (cost + context budget); spelling/arithmetic failures are tokenizer artifacts, not reasoning — give it a tool. Vocab size is a real param cost (tied embedding/unembedding).
- **"What are the limitations of a big context window?"** → Lost-in-the-Middle (U-shaped recall) + Context Rot (reasoning degrades as it fills) mean *more context can mean worse answers*; plus super-linear attention cost and linear KV growth. Curate and order; don't stuff.
- **"Temperature & top-p?"** → temperature rescales logits before softmax; top-p keeps a cumulative-mass nucleus; tune one, not both. **temperature 0 is greedy, not deterministic** — design for variation with schemas + eval gates.
- **"What is the KV cache?"** → per-token K/V storage at every layer to avoid O(T²) recompute; `2·L·B·T·H_kv·D_h·dtype_bytes`; linear in context and batch, heavy per token; at long context it's the dominant serving cost, and decode is bandwidth-bound streaming it from HBM.
- **"Reasoning model vs standard?"** → hard multi-step / verification-heavy → reasoning; latency/cost/volume-sensitive and shallow → standard; route per request rather than choosing globally.
- **"Users say it got dumber but you shipped nothing."** → suspect a silent provider model update (or interacting internal changes); pin model + prompt versions and gate every change with a golden eval + needle-in-a-haystack.

## 📚 Resources

- 🧑‍🏫 [Karpathy — Neural Networks: Zero to Hero](https://karpathy.ai/zero-to-hero.html) — builds GPT from scratch, token by token; the canonical "implement attention / build the tokenizer" interview prep. *(2026)*
- 🧑‍🏫 [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course) — hands-on tokenization, BPE, and transformer internals with runnable notebooks. *(2026)*
- 🛠️ [tiktoken](https://github.com/openai/tiktoken) — MIT; OpenAI's fast byte-level BPE tokenizer — encode real strings and see actual token counts and the "strawberry" split (~14k★). *(2026)*
- 📄 [RoPE — RoFormer (Su et al., 2021)](https://arxiv.org/abs/2104.09864) — rotary positional embeddings; the 2026 default, and the source of the relative-from-absolute identity. *(paper)*
- 📄 [FlashAttention-2 (Dao, 2023)](https://arxiv.org/abs/2307.08691) — IO-aware *exact* attention; the definitive "memory movement, not smarts" reference for why it's 2–3×. *(paper)*
- 📰 [KV caching, explained](https://huggingface.co/blog/not-lain/kv-caching) — why decode caches K/V and the measured speedup; pair with Raschka's [Coding the KV cache](https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms) for a from-scratch implementation. *(articles)*
- 📰 [Sebastian Raschka — Ahead of AI](https://magazine.sebastianraschka.com/) — LLM internals, GQA/MoE deep-dives, and 2026 paper roundups written for engineers. *(article)*
- 📄 [PagedAttention / vLLM (Kwon et al., 2023)](https://arxiv.org/abs/2309.06180) — OS-style KV-cache paging that cut fragmentation 60–80% → <4% and fixed serving throughput. *(paper)*

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [LLM fundamentals questions](../questions/fundamentals.md)
