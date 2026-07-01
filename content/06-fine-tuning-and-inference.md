# Fine-Tuning & Inference Optimization

> When the model won't behave, don't reach for the most expensive lever first. This is the frame — RAG vs fine-tune vs prompt, LoRA/QLoRA, the post-training ladder (SFT → RM → PPO → DPO → GRPO), quantization, and the serving stack — that separates a senior answer from a hand-wave.

Back to [README](../README.md) · [All questions](../questions/README.md)

---

## What this tests

Two things, and interviewers probe both. First, **judgment**: given a failing feature, do you climb the cheapest lever that closes the gap, or default to "let's fine-tune"? Second, **mechanism-and-numbers depth**: can you derive the LoRA update, name the three QLoRA tricks, explain why DPO drops the reward model, and quote the memory math for serving a 70B?

Answer-first version of the whole track: **prompting changes what you ask this turn; RAG changes what the model knows; fine-tuning changes how it behaves.** Serving is where the bill lives — ~40% of a modern LLM-engineer loop is inference, not modelling. Below is the decision frame, the math, the traps, and the interview angles.

---

## The decision — RAG vs fine-tune vs prompt

The reflex answer to an underperforming feature — "let's fine-tune" — is usually the *wrong* first move, and knowing why is the senior signal. The three levers act on three different objects:

- **Prompting** reshapes the *input distribution* at inference. No weights touched. Cost is context tokens; it survives any model upgrade.
- **RAG** keeps the base model frozen and prepends *retrieved evidence*, so the model conditions on fresh, attributable text every query.
- **Fine-tuning** updates the *weights* so the behaviour becomes a prior the model already carries — which is what removes the per-query cost of a long prompt.

The mental model that wins interviews: **behaviour and knowledge are orthogonal failure modes.** If the model doesn't *know* something (yesterday's ticket, a regulation that changed last week), that's a knowledge gap — no amount of fine-tuning reliably injects volatile facts. If the model knows enough but won't *act* right (wrong tone, ignores your JSON schema, can't do the domain's reasoning even with a perfect prompt), that's a behaviour gap, and fine-tuning is the durable fix.

| Lever | Touches | Fixes knowledge? | Fixes behaviour? | Per-query cost | Cold-start cost |
| --- | --- | --- | --- | --- | --- |
| **Prompting** | nothing | weakly | moderately | + context tokens | low (one string) |
| **RAG** | nothing | **yes** (+ citations) | no | + retrieval + chunk tokens | medium (index pipeline) |
| **Fine-tune** | weights | indirectly (stale, no cite) | **yes**, durable | ~0 (merged) | high (data + train + eval) |

**Climb the ladder — don't jump to the top.** Benchmark the best prompt-and-RAG configuration *first*, then fine-tune only the residual error that is a stable behaviour the model can't acquire in context. The order is one of increasing cost and decreasing reversibility.

Put numbers on it: hosted GPT-4.1 fine-tuning runs ~**$25 / 1M training tokens**, so a 20M-token SFT run is ~**$500 of training alone** before the data labelling and eval that dominate the real bill. RAG re-indexes for effectively free; a prompt is one string in version control.

**The evidence that behaviour ≠ scale:** InstructGPT (Ouyang et al., 2022) — outputs from the **1.3B** aligned model were preferred over the **175B** unaligned GPT-3. A 130× smaller model won on human preference purely because its *behaviour* was tuned. "Just use a bigger model" is not a substitute for fine-tuning when the gap is behaviour.

### The example where you would NOT fine-tune despite having data

You have 50k labelled support tickets. Tempting to fine-tune. But if the product also needs to **cite the document it used** and the knowledge base **updates weekly**, fine-tuning is the wrong primary lever: retraining weekly is wasteful (~$500+/run), a fine-tune can't natively cite, and it can't enforce per-user permissions. That's textbook **RAG** — re-index cheaply, attribute to the chunk. Reserve the fine-tune for a residual *behaviour* gap (a refusal to discuss the domain, a strict output format), and keep RAG underneath it for the facts. **The fine-tune sets the floor of competence; RAG sets the ceiling of freshness; the prompt carries the rules of this turn.**

> [!WARNING]
> **"Fine-tune to add knowledge."** No — that's RAG. Fine-tuning encodes facts *lossily* and *statically*: it can't cite, goes stale the moment a doc changes, can't enforce per-user ACL, and surfaces facts as confident paraphrases rather than verifiable quotes. Most "we tried fine-tuning and it didn't help" stories are teams that fine-tuned a knowledge problem. Fine-tuning is for behaviour — style, format, tool use, a skill — not the rows in your wiki.

> [!WARNING]
> **Catastrophic forgetting.** Aggressive domain fine-tuning pulls weights away from general competence — the model gets better at your tickets and quietly worse at everything else. This is a *train-data composition* problem, not an architecture one: mix in ~5–15% replay from the original/general distribution, lower the learning rate, bound the update (LoRA rank, fewer epochs). And you only *see* it if you held out a general eval (MMLU/MT-Bench) alongside your task eval. Gate the deploy on a capability floor: "MMLU must not drop more than ~2 points."

---

## PEFT & LoRA — the update math

Full fine-tuning a 175B model in fp16 needs ~1.2 TB of VRAM just for weights, gradients, and Adam optimizer state — out of reach for almost everyone. **PEFT** (parameter-efficient fine-tuning) is the family that avoids this — LoRA, adapters, prefix tuning, prompt tuning — by training a tiny fraction of parameters and freezing the rest. LoRA is the default.

**The mechanism.** LoRA (Hu et al., 2021) freezes the pretrained weights and injects a trainable **low-rank decomposition** into selected linear layers. For a weight matrix `W` (d×k), instead of learning a full update `ΔW` you learn `ΔW = B·A`, where `A` is r×k and `B` is d×r, and the rank `r` is tiny — often 8 to 64. `A` is initialised from a small Gaussian and **`B` is initialised to zero**, so at step 0 the adapter contributes nothing and the model starts exactly at the base distribution.

```python
    # Minimal LoRA adapter forward (PyTorch-ish)
    # W0: frozen base weight (d x k). A, B: the only trainable params.
    class LoRALinear(nn.Module):
        def __init__(self, W0, r=8, alpha=16):
            super().__init__()
            d, k = W0.shape
            self.W0 = W0.requires_grad_(False)      # frozen base
            self.A = nn.Parameter(torch.randn(r, k) * 0.01)   # Gaussian init
            self.B = nn.Parameter(torch.zeros(d, r))          # ZERO init -> step 0 == base
            self.scale = alpha / r                             # the forward multiplier

        def forward(self, x):
            # h = W0 x + (alpha/r) * B (A x)
            return x @ self.W0.T + self.scale * (x @ self.A.T) @ self.B.T
```

**Why does adding two small matrices approximate full fine-tuning?** You're not approximating `W` — you're approximating the fine-tuning *delta* `ΔW`, and the delta is intrinsically low-rank. The single most striking fact in the LoRA paper: on GPT-3 175B, a rank of **r = 1 or 2** was sufficient even though the attention weights are 12,288-dimensional. The adaptation manifold is shockingly low-rank. Candidates who frame it as *a delta on the update manifold* (not "freezing weights") stand out.

**The three knobs.**

| Knob | Typical | Effect / rule |
| --- | --- | --- |
| **rank `r`** | 16–128 | capacity of the update; too high overfits small data |
| **alpha** | 1–2× r | forward scale is `alpha/r`; `alpha = 2r` keeps effective scale stable across ranks |
| **target modules** | q_proj, v_proj | cheapest; **matches full FT only if you cover ALL linear layers** (QLoRA ablation) |

The target-modules finding is the senior gotcha: q/v-only does **not** match full fine-tuning — you need LoRA on all linear transformer-block layers. Recipe: start `r=16, alpha=32, q_proj+v_proj`, measure on a held-out set, and expand to all-linear and raise rank *only if quality misses the bar*. **Coverage > rank** for matching full FT.

Concrete footprint to have ready: **Llama-2-7B at r=8 trains ~4.19M params ≈ 17 MB** — a ~1000× reduction. That tiny adapter is the entire artifact you ship and version.

**Adapter merging & the inference story.** At deploy you compute `W_merged = W + (alpha/r)·B·A` once, and the result is a dense matrix bit-identical to what full fine-tuning would produce — so a merged LoRA has **zero additional inference latency** (unlike classic adapters that add a forward hop per token). You can also keep adapters *unmerged* and hot-swap them per request — the basis for multi-tenant adapter serving.

> [!TIP]
> LoRA trains ~0.1–1% of the parameters and, once merged, costs *nothing* at inference. The 17 MB adapter is the whole artifact — you can ship a hundred fine-tunes as a hundred tiny files over one shared base.

> [!WARNING]
> **"LoRA = cheaper inference."** The savings are at *training* (no optimizer state for frozen weights — and Adam's two moments per param are the dominant memory term). A *merged* LoRA serves at full dense size and latency. The only inference-memory win is hot-swapping many unmerged adapters over one shared base (multi-tenant). Conflating train-time and inference-time savings is a classic weak answer.

> [!WARNING]
> **"LoRA is lossy vs full FT."** Usually within noise for narrow, behaviour-shaped tasks *if you cover all linear layers at adequate rank*. LoRA genuinely underperforms only for large distribution shifts or new capabilities (new language/modality, heavy domain adaptation) where you're moving the base prior, not nudging it. When a LoRA plateaus below full FT, the culprit is almost always under-coverage — expand target modules and raise rank before concluding PEFT can't reach the bar.

---

## QLoRA — 65B on one 48GB GPU

QLoRA (Dettmers et al., 2023) backpropagates gradients through a **frozen, 4-bit-quantized** base model into **higher-precision (BF16) LoRA adapters**. It compresses the dominant memory line item — base weights — without disturbing gradient flow into the adapters. A strong answer names all three tricks, not "it's 4-bit LoRA":

1. **NF4 (4-bit NormalFloat)** — a data type information-theoretically optimal for *normally-distributed* weights, with bins holding equal probability mass. Fits trained weights' Gaussian shape far better than uniform int4.
2. **Double quantization** — quantize the quantization constants themselves, cutting per-parameter overhead from 0.5 bits to ~0.127 bits (~3 GB saved on a 65B model).
3. **Paged optimizers** — use NVIDIA unified memory to page optimizer state to CPU during the gradient-checkpoint memory spike on long sequences, killing the OOM-on-long-context failure mode.

The forward pass dequantizes `W` (NF4 → BF16) on the fly, computes in BF16, and adds `B·A`. Adapters stay BF16; only the frozen base is NF4.

| Model | QLoRA total (bs=1, seq=512) | Fits single GPU? |
| --- | --- | --- |
| LLaMA-7B | ~5 GB base | yes (24 GB) |
| LLaMA-33B | 24.7 GB | yes (24–32 GB) |
| LLaMA-65B | 45.0 GB | yes (48 GB) |

The tradeoff, and the senior tell: QLoRA pays a **dequantize-on-the-fly tax** — on Llama-2-7B, ~17.86 GB peak vs ~36.66 GB for a two-GPU full FT (**~33% less memory at ~39% more runtime**). Quality is safe: QLoRA "replicates 16-bit LoRA and full-finetuning task performance," and the Guanaco family it produced reached **99.3% of ChatGPT** on Vicuna from a single GPU in 24 hours. Skip QLoRA only when fp16 memory is plentiful and you don't want the per-step dequant tax.

> [!TIP]
> **Two independent dials.** Quantization (QLoRA's NF4 base) sets the *memory floor*; adapter coverage (all-linear layers) sets the *quality ceiling*. You can run 4-bit *and* cover all linear layers to hit full-FT quality — the 4-bit saves memory, the coverage recovers quality. Treating "use QLoRA" and "match full FT" as one decision is what produces under-covered, underperforming adapters.

---

## The post-training ladder — SFT → RM → PPO → DPO → GRPO

SFT makes a base model follow instructions but only ever sees *positive* examples — it can't express "this answer is *better* than that one." Preference optimization is the stage that aligns to human preference over *pairs*: helpfulness, tone, refusal behaviour.

### RLHF: SFT → reward model → PPO

The original three-stage pipeline (InstructGPT, 2022):

1. **SFT** — fine-tune on curated demonstrations to get a reference policy.
2. **Reward model (RM)** — train a scalar `r(x,y)` on pairwise human preferences `{y_w, y_l}` with a Bradley-Terry loss `-log σ(r(x,y_w) − r(x,y_l))`. The RM learns to score any (prompt, response).
3. **PPO** — optimise the policy with RL against the RM, plus a **KL penalty** toward the SFT reference so the policy doesn't drift into gibberish that games the RM. Objective: maximise `r(x,y) − β·KL(policy ‖ SFT_ref)`.

The failure modes interviewers probe hardest: **reward hacking** (Goodhart — the policy finds outputs that score high on the proxy RM but are degenerate to humans), **instability/mode collapse**, and the **four-model infra cost** (policy + reference + reward + value all resident in memory).

### DPO: skip the reward model

DPO (Rafailov et al., 2023): under Bradley-Terry + a KL-bounded objective, the *optimal RLHF policy* can be written in closed form as a function of the reference and the reward — so the reward is **implicit in the log-ratio of the policy to the reference**. DPO eliminates the reward model and the PPO loop entirely, collapsing the whole thing after SFT into **one supervised classification loss** over (chosen, rejected) pairs:

```python
    # DPO loss -- no reward model, no policy sampling, no PPO
    # pi_logratio  = logp_policy(y_w) - logp_policy(y_l)
    # ref_logratio = logp_ref(y_w)    - logp_ref(y_l)    # frozen SFT reference
    loss = -F.logsigmoid(beta * (pi_logratio - ref_logratio)).mean()
    # beta = KL strength (default 0.1; 0.5 for TL;DR). Lower beta => more divergence.
```

No reward model to train, no sampling from the policy during training (PPO's expensive, unstable rollouts are gone), no PPO infra. "Stable, performant, and computationally lightweight." The one finicky knob is **beta**.

### GRPO: group-relative, no value model — the 2026 reasoning default

GRPO (Group Relative Policy Optimization, from DeepSeekMath; made mainstream by **DeepSeek-R1**, 2501.12948) is the reasoning post-training default in 2026. Its key move: **drop the value model.** PPO trains a separate critic to estimate a baseline for the advantage. GRPO instead samples a **group** of G completions per prompt, scores each with a (often rule-based, verifiable) reward, and computes the advantage as the reward **normalized within the group**: `A_i = (r_i − mean(r)) / std(r)`. The group *is* the baseline.

That's why DeepSeek-R1 chose it for reasoning: the reward is sparse, shaped, and *verifiable* (did the math check out? did the code pass?), and GRPO propagates that signal without the cost and instability of a learned critic — halving the resident-model count versus PPO and playing perfectly with programmatic/verifiable rewards at scale.

| | RLHF (PPO) | DPO | GRPO |
| --- | --- | --- | --- |
| **Reward model** | explicit, trained | **none** (implicit in log-ratio) | reward function (often rule-based) |
| **Value/critic model** | yes | no | **no** (group mean is the baseline) |
| **Policy sampling in training** | yes (rollouts) | **no** | yes (a group per prompt) |
| **Stability / cost** | unstable, 4 models resident | stable, cheap, 1 loss | stable, no critic, RL-grade |
| **Best for** | sparse shaped rewards, calibrated online iteration | dense preference over short answers (chat, tone, refusals) | **verifiable long-horizon reasoning** (math, code) |

**Choose by the reward signal's structure.** Dense preference over short answers → **DPO** (chat/style/refusals). Sparse, shaped, long-horizon, *verifiable* reward → **GRPO** (reasoning). Sparse shaped reward where you want a reusable calibrated RM for online eval → **PPO/RLHF**. Measured bounds: InstructGPT outputs preferred over GPT-3 **85±3%**; DPO ~**61%** vs PPO ~**57%** win rate on TL;DR — so DPO isn't just cheaper, it's competitive on preference-shaped tasks.

> [!TIP]
> **GRPO's whole trick in one line:** it deletes the value model. The advantage is just the reward *normalized within a sampled group of completions* — the group's own mean and std are the baseline. No critic to train, no critic to keep in memory, and it feeds naturally on the rule-based verifiable rewards that reasoning tasks provide.

> [!WARNING]
> **"DPO == PPO."** DPO is *not RL*. There is no reward model, no policy sampling, no rollouts — it's a Bradley-Terry classification loss on fixed preference pairs. And the two fail in *different kinds*: PPO reward-hacks an explicit RM (Goodhart); DPO **preference-warps** — it overfits the *shape* of the pairs (stylistically aligned but factually shallow) and is brittle OOD against its frozen reference. "DPO is just simpler RLHF" misses that the choice is workload-dependent.

---

## Quantization — method matters more than bit-width

The senior framing: **quantization quality is a function of method, not bit-width.** Two 4-bit quantizers can differ wildly. Also distinguish **weight-only** quantization (shrink the weight stream — the decode bottleneck) from **activation** quantization (quantize the running activations too, e.g. W8A8, needed to actually hit the hardware's low-precision GEMM path).

| Method | Bits | Kind | Quality vs FP16 | When |
| --- | --- | --- | --- | --- |
| **INT8** (LLM.int8/SmoothQuant) | 8 | weight+act | near-lossless w/ outlier handling | Ampere default; ~2× memory cut |
| **FP8** (E4M3/E5M2) | 8 | weight+act | ~0 ppl delta | **H100/Blackwell default; +33% tok/s, −24% $/Mtok** |
| **AWQ** | 4 | weight-only | protects ~1% salient channels; <0.2 ppl on 70B | GPU, memory-bound; edge is the **fused kernel**, not quality |
| **GPTQ** | 4 (3) | weight-only | +0.15 ppl int4 on 175B; +0.54 at int3 | GPU; Hessian reconstruction; one-shot |
| **NF4** | 4 | weight-only | matches 16-bit for QLoRA | QLoRA base; optimal for Gaussian weights |

The numbers *decouple quality from bit-width* at modern methods: GPTQ int4 on OPT-175B moves C4 perplexity only **10.13 → 10.28**; AWQ matches that and runs **>3× faster than HF FP16** (the win is the kernel). **FP8 on H100** measured **+33% tokens/sec, −24% cost per million tokens** at near-zero perplexity delta — it's the default on Hopper/Blackwell because the GPU runs FP8 GEMMs at ~2× FP16 throughput. GPTQ vs AWQ in practice: similar int4 quality, so **decide by your stack's kernel support**, not accuracy. Per-channel (per-output-channel) scaling is standard for both — one scale per channel slashes error versus a single per-tensor scalar.

> [!WARNING]
> **"Quantization is free."** Down to 4-bit the recoverable fraction is set by the *method*, not the bit count — but two cliffs are real. (1) **Activation outliers**: a few channels have huge magnitudes; naive per-tensor quantization gets destroyed by them (why LLM.int8 splits outliers out and AWQ protects salient channels). (2) **int3 and below**: GPTQ jumps to +0.54 ppl — reserved for "must fit one GPU regardless of quality." And **FP8 is not an option on A100** (no FP8 tensor cores) — naming that hardware constraint is the senior tell. Choosing bits without naming the method or the hardware is the weak answer.

---

## Inference — the levers that actually move the bill

Every optimization maps to one term of the cost equation. A forward pass splits into two phases with different bottlenecks:

- **Prefill** (processing the prompt) is **compute-bound** — one big parallel matmul over all input tokens. Governs **TTFT** (time to first token).
- **Decode** (one token at a time) is **memory-bandwidth-bound** — each new token streams the *entire* weight set + growing KV cache through the GPU for tiny matmuls. Governs **TPOT** (time per output token).

### KV cache

Every generated token attends to all previous ones, so the keys/values for the whole sequence are cached and grow with output length. A 13B model burns ~**1 MB of KV state per token**; on an A100 40GB, after ~26 GB of weights you fit only ~14k tokens of cache — ~7 concurrent requests at 2048-token sequences. The KV cache is the hidden memory hog of decode, and the target of the next two levers.

### PagedAttention / vLLM — kill fragmentation

The classic bug: allocating one *contiguous* max-length KV tensor per request wastes **60–80%** of reserved memory to internal fragmentation and over-reservation. **PagedAttention** (Kwon et al., SOSP 2023) borrows OS virtual memory: split the KV cache into fixed **4–16 token pages**, keep a per-request block table mapping logical → physical pages allocated on demand. Result: **<4% waste** with copy-on-write sharing of identical prefixes. Measured **2–4× throughput at the same latency**. The big multiplier is **prefix caching** — a shared system prompt or RAG document is computed once and reused across requests (enable it for any workload with >5% prefix overlap; note per-user caches aren't shareable for privacy).

### Continuous batching — 23× over static

The single biggest serving win is a *scheduling* change, not a kernel.

| | Static (request-level) | Continuous (iteration-level) |
| --- | --- | --- |
| **Schedule** | fill batch → run all to completion → swap | reschedule **every forward pass** |
| **Waste** | pad to longest seq; short requests idle | evict finished, admit new into freed slots — GPU stays full |
| **Throughput** | ~81 tok/s under length variance | Orca **36.9×** / vLLM **23×** at same latency |

Continuous batching (Orca, OSDI 2022; vLLM) is table-stakes — static implementations are obsolete. It shifts the bottleneck to scheduler quality and introduces one new failure: a long **prefill** shares the batch with tiny **decode** steps and starves them (TTFT spikes). Fix with **chunked prefill** (split long prompts into ~512-token chunks so decode interleaves), or the full **prefill-decode disaggregation** (separate engine pools) when the SLO is asymmetric.

### Speculative decoding — 2–3× at batch 1

A small **draft** proposes *K* tokens; the large **target** verifies all K in a single forward pass via rejection sampling — and the output distribution is **mathematically identical** to vanilla decoding (lossless — state this). Speedup ≈ *K × α* (α = acceptance rate); with K=4–8, α=0.5–0.8 you get **2–3×**.

| Method | Mechanism | Note |
| --- | --- | --- |
| **vanilla** | separate small draft model | 2.6×@T=1 / 3.4×@T=0 (Leviathan, bs=1) |
| **Medusa** | multiple draft *heads* on the target | no separate model; parallel heads |
| **EAGLE-3** | single draft head in the target's forward pass | **the 2026 leader**; 1.4–2× large batch, 10–30% over vLLM single-stream |

**When it evaporates:** speculative decoding helps when the target is bandwidth-bound (small batch) and *hurts* when the target is already throughput-bound (large batch), because the draft compute becomes pure overhead. EAGLE-3.1 in vLLM shows the decay: 2.03× @ concurrency 1 → 1.66× @ 16. Enable it on latency-sensitive batch-1 traffic; measure acceptance rate per workload (templated/RAG prefixes accept more).

### MoE serving — expert parallelism on top of tensor parallelism

A Mixture-of-Experts model routes each token to a few of many FFN "experts," so params are huge but compute per token is small. Serving adds a parallelism axis: on top of **tensor parallelism** (shard each matmul across GPUs) you add **expert parallelism** (place different experts on different GPUs). The hard part is the **all-to-all** token routing between the attention (dense, TP-sharded) and expert (EP-sharded) stages, plus load balance across experts — which is why MoE-serving throughput lives or dies on the interconnect (NVLink/RDMA) and the router. TensorRT-LLM and vLLM both expose EP for large MoEs (DeepSeek, Mixtral).

### Serving engines — vLLM vs TensorRT-LLM vs SGLang

| Stack | Sweet spot | Why |
| --- | --- | --- |
| **vLLM** | greenfield / high-throughput chat, 50+ concurrency | PagedAttention + continuous batching + chunked prefill + prefix cache + broad quant + spec-decode. Beats TGI 3.67× @100 conc, 24× @200 |
| **TGI** | short-batch, low-concurrency, TTFT-bound | 1.3–2× lower TTFT at low concurrency (less scheduler overhead) |
| **SGLang** | multi-call agentic/RAG with heavy prefix reuse + JSON | RadixAttention (52–99% hit rate) + compressed-FSM grammar decoder; up to 5× on multi-call |
| **TensorRT-LLM** | peak throughput on Hopper/Blackwell, ops can build | FP8/SmoothQuant kernels + in-flight batching + MoE EP; cost is 30–60 min per-config engine builds |

> [!WARNING]
> **"More GPUs = faster decode."** Decode is *memory-bandwidth-bound*, not compute-bound — you're streaming weights + KV cache per token, doing tiny matmuls. Throwing GPUs at a serving problem without continuous batching, paged KV, quantization, or the prefill/decode split ignores that the dominant waste is *idle GPU time* (fragmented KV, padded static batches), which PagedAttention and continuous batching reclaim before any hardware spend. And the levers **compose multiplicatively** (~24–32× theoretical) but real traffic lands at single-to-low-double-digit percent because scheduler/network/kernel-launch overhead dominates — always measure the compound, budget against the *production* number, not the corner-case ceiling.

---

## Interview angles

**Q: LoRA vs full fine-tuning?**
LoRA freezes `W` and learns `ΔW = B·A` at tiny rank; forward is `h = Wx + (alpha/r)·B·A·x` with `B` init to zero so step 0 equals the base. You're approximating the fine-tuning *delta*, which is intrinsically low-rank (GPT-3 needed r=1–2). It trains ~0.1–1% of params, the checkpoint is ~17 MB for a 7B at r=8, and merged it has zero inference latency. It *matches* full FT for behaviour-shaped tasks when you cover all linear layers at adequate rank; full FT earns its cost only for large distribution shifts or new capabilities.

**Q: Explain QLoRA / why 4-bit?**
Backprop through a frozen **4-bit NF4** base into **BF16 adapters**. Three tricks: NF4 (info-theoretically optimal for Gaussian weights), double quantization (quantize the quant constants, 0.5 → 0.127 bits/param), paged optimizers (page optimizer state to CPU on the long-seq spike). Puts a 65B fine-tune on one 48 GB GPU (~45 GB at bs=1/seq=512). Cost: ~30–40% slower steps from dequant-on-the-fly. Quality matches 16-bit LoRA (Guanaco = 99.3% of ChatGPT, 24h, one GPU).

**Q: What is PEFT + its techniques?**
Parameter-efficient fine-tuning: train a small fraction, freeze the rest. Family: LoRA (low-rank adapters, the baseline), classic adapters (inserted bottleneck layers), prefix tuning / prompt tuning (learn soft prompt vectors), (IA)³. LoRA dominates because it merges to zero inference cost, unlike adapters that add a forward hop.

**Q: When fine-tune vs RAG vs prompt?**
Classify the failure. Volatile, attributable knowledge → RAG. Stable behaviour/format/skill → fine-tune. Missing instruction → prompt. They compose: fine-tune the floor of competence, RAG the ceiling of freshness, prompt the rules of the turn. Climb the ladder cheapest-first; reserve the sticky, expensive fine-tune for the residual behaviour prompting+RAG provably can't reach.

**Q: Explain RLHF (SFT → RM → PPO).**
SFT to a reference policy, train a Bradley-Terry reward model on pairwise prefs, then PPO the policy against the RM with a KL penalty to the SFT reference. Four models resident. Failures: reward hacking (Goodhart), instability/mode collapse, infra cost.

**Q: DPO vs PPO at the algorithm level?**
Under BT + a KL-bounded objective the optimal policy is a closed-form function of the reference and reward, so the reward is *implicit* in the policy/reference log-ratio. DPO replaces the RM and PPO loop with one supervised classification loss — no reward model, no rollouts, no RL. It's stable and cheap but preference-warps and is brittle OOD against a frozen reference; PPO earns its cost on sparse shaped rewards and gives you a reusable calibrated RM.

**Q: What is GRPO + why did DeepSeek-R1 choose it over PPO?**
Group Relative Policy Optimization drops PPO's value/critic model. Sample a group of G completions per prompt, score each (often with a rule-based *verifiable* reward), and set the advantage to the reward normalized within the group (`(r−mean)/std`) — the group mean *is* the baseline. DeepSeek-R1 chose it because reasoning rewards are sparse, shaped, and verifiable; GRPO propagates that signal with RL-grade credit assignment but no learned critic to train or hold in memory. It's the 2026 reasoning post-training default.

**Q: INT8 vs FP16 / GPTQ vs AWQ vs NF4 — when?**
INT8 halves memory near-losslessly if you handle activation outliers (LLM.int8/SmoothQuant) — the Ampere default. FP8 is the H100/Blackwell default (+33% tok/s, ~free quality) but needs FP8 tensor cores (not A100). At 4-bit, GPTQ and AWQ have similar quality (<0.2 ppl on a 70B) — choose by kernel/stack support; AWQ ships a fused kernel. NF4 is the QLoRA base type. int3 (+0.54 ppl) only when you *must* fit one GPU.

**Q: How does PagedAttention eliminate fragmentation?**
Naive contiguous per-request KV allocation wastes 60–80% to internal fragmentation and over-reservation. PagedAttention pages the KV cache into fixed 4–16 token blocks with a per-request block table (logical → physical), allocated on demand — <4% waste — and shares identical prefixes copy-on-write, so a shared system prompt is stored once.

**Q: vLLM vs TensorRT-LLM — when each?**
vLLM is the greenfield default: PagedAttention + continuous batching + broad quant + spec-decode, best throughput at 50+ concurrency, no build step. TensorRT-LLM is peak throughput on Hopper/Blackwell via FP8/SmoothQuant kernels and in-flight batching, but costs 30–60 min per-(model,GPU,batch) engine builds — worth it for a hot, fixed model where ops can pay the build tax.

**Q: How does speculative decoding work + expected speedup?**
Small draft proposes K tokens; large target verifies all K in one forward pass via rejection sampling — output distribution is identical (lossless). Speedup ≈ K×α; ~2–3× at batch 1 (EAGLE-3 leads). It evaporates at high batch when the target is already throughput-bound and the draft compute becomes overhead.

**Q: Deploy a 70B Llama with continuous batching.**
On 8×A100 40GB: FP16 ≈ 140 GB won't fit, int8 ≈ 70 GB fits, **AWQ/GPTQ int4 ≈ 35 GB** is comfortable with KV headroom (FP8 is off the table — no A100 tensor cores). Serve on **vLLM**: continuous batching + PagedAttention for the throughput win, prefix caching for the shared system prompt, chunked prefill to stop long prompts starving decode, spec-decoding on the batch-1 tail. Autoscale on QPS; add admission control for prompts that exceed the KV budget. Budget against the *measured compound*, not the theoretical ceiling.

---

## 📚 Resources

- 📄 **LoRA** — https://arxiv.org/abs/2106.09685 — low-rank adapters; the PEFT baseline (r=1–2 sufficed on GPT-3).
- 📄 **QLoRA** — https://arxiv.org/abs/2305.14314 — 4-bit NF4 base + double-quant + paged optimizers → 65B on one 48 GB GPU.
- 📄 **DPO** — https://arxiv.org/abs/2305.18290 — preference optimization without a reward model; "your LM is secretly a reward model."
- 📄 **DeepSeek-R1 (GRPO)** — https://arxiv.org/abs/2501.12948 — RL-only reasoning post-training; made GRPO mainstream.
- 📄 **PagedAttention / vLLM** — https://arxiv.org/abs/2309.06180 — KV-cache paging, <4% waste, up to 24× throughput.
- 💻 **vLLM** — https://github.com/vllm-project/vllm — Apache-2.0, ~85k★ — the de-facto serving stack.
- 💻 **HF PEFT** — https://github.com/huggingface/peft — Apache-2.0, ~21k★ — LoRA / adapters / prefix tuning.
- 📰 **Raschka, "State of LLM Reasoning Model Training"** — https://magazine.sebastianraschka.com/p/the-state-of-llm-reasoning-model-training — best practitioner summary of GRPO/DPO/RLHF.

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [Fine-tuning questions](../questions/fine-tuning.md)
