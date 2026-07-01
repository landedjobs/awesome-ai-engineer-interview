# Fine-Tuning & Inference — RAG-vs-FT, Data, LoRA/QLoRA, DPO/RLHF & Serving

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/fine-tuning.md`](../content/06-fine-tuning-and-inference.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **40 questions** (30 self-test checkpoints + 10 reported). The LLM-engineer depth round. Theses: **fine-tune behaviour, RAG the volatile knowledge, prompt the per-turn rules — classify the failure mode per layer**; **data quality beats volume (QDC frontier)**; **LoRA saves training memory, not serving memory**; **DPO for dense short-answer preference, RL for sparse long-horizon reward**; and **reason from memory + hardware for precision choices, then re-eval the artifact you actually serve**.

---

### How to use this file

Answer, then open the fold. The serving-side questions here overlap the transcript-internals in [fundamentals.md](./fundamentals.md) (KV cache, quantization) and the system-design serving round in [system-design.md](./system-design.md) — this file is the *fine-tuning-and-precision* lens.

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (memory math, alignment tax, hardware constraints)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — RAG vs fine-tuning vs prompting `[fine-tuning-inference] d1`

### Q1. A team wants a support assistant that answers from a knowledge base that changes weekly and must cite the document it used. What's the strongest primary lever?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] d1` · 🔮 Representative

- Fine-tune the model weekly on the latest knowledge base
- ✅ RAG — retrieve at query time for freshness and chunk-level citations; fine-tune only if a behaviour gap remains
- A larger base model so it memorises more of the knowledge base

<details><summary>Show answer & why each option</summary>

- **Fine-tune weekly** — Wasteful (~$500+ per run at GPT-4.1 rates) and a fine-tune can't natively cite — wrong tool for volatile, attributable knowledge.
- ✅ **RAG** — Volatile + citable knowledge is the textbook RAG case: re-index cheaply, attribute to the chunk. Fine-tuning is reserved for behaviour.
- **Larger base model** — Still has no row for their private, post-training docs and still can't cite or stay fresh.
</details>

### Q2. A model answers factually fine but won't reliably emit your strict JSON tool schema and ignores your house tone, even with a careful prompt and examples. Best durable fix?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] d1` · 🔮 Representative

- ✅ Fine-tune on examples of the correct schema and tone — this is a behaviour gap
- Add the company knowledge base via RAG
- Switch providers and hope the new model formats better

<details><summary>Show answer & why each option</summary>

- ✅ **Fine-tune the behaviour** — Format/tone/tool-call reliability are stable behaviours the base prior won't reach in context; encoding them in weights is the durable fix (and lets you drop the long prompt).
- **Add RAG** — RAG fixes knowledge, not behaviour — the model already knows the facts.
- **Switch providers** — Unpredictable and re-opens the same behaviour gap.
</details>

### Q3. Your fine-tune lifted task accuracy on support tickets but MMLU dropped 6 points. What does a senior do?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] d1` · 🔮 Representative

- Ship it — MMLU is irrelevant to the support task
- Conclude fine-tuning was the wrong call and revert to prompting
- ✅ Treat it as catastrophic forgetting: add ~5–15% general-distribution replay, lower LR / bound the update, and gate on a capability floor (e.g. MMLU −2 max)

<details><summary>Show answer & why each option</summary>

- **Ship it** — Dismissing the regression ignores the multi-objective reality; a 6-point general drop can surface as odd off-task failures.
- **Revert to prompting** — Over-correction — the task gain is real; the fix is data composition and a capability floor.
- ✅ **Catastrophic forgetting → replay + bounded update + floor** — Forgetting is a train-data composition problem; replay + a bounded update recover general skill, and a held-out general eval makes the tradeoff explicit.
</details>

### Q4. A latency-critical, high-volume classifier currently uses a 2,000-token few-shot prompt per call. The labels are stable. What's the strongest optimisation?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] d1` · 🔮 Representative

- Cache the prompt and call it a day
- Move to RAG so the examples are retrieved instead of in the prompt
- ✅ Fine-tune a small model on the labelled examples and drop the few-shot prompt entirely

<details><summary>Show answer & why each option</summary>

- **Cache the prompt** — Helps input cost but you still pay output tokens and the prompt still ships.
- **Move to RAG** — Adds retrieval latency and still puts tokens in context — wrong tool when behaviour is fixed and you want the prompt gone.
- ✅ **Fine-tune a small model, drop the prompt** — Encodes behaviour into weights, so you delete the 2,000-token prompt: near-zero extra latency and lower per-call cost at volume.
</details>

### Q5. "Why not just fine-tune everything — wouldn't a model that knows our domain and behaves right be best?" Strongest response?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] d1` · 🔮 Representative

- Agree — one fine-tuned model is simpler than maintaining three systems
- ✅ Separate the failure modes: fine-tune behaviour, RAG the volatile knowledge, prompt the per-turn rules — and only fine-tune the residual that prompting+RAG can't reach
- Fine-tune for knowledge and prompt for behaviour

<details><summary>Show answer & why each option</summary>

- **One fine-tuned model** — Conflates knowledge and behaviour: facts go stale in weights and can't be cited, and you re-pay the whole cost on every base-model upgrade.
- ✅ **Separate the failure modes** — Behaviour and knowledge are orthogonal; use the cheapest lever per failure and reserve the sticky, expensive fine-tune for true behaviour gaps.
- **Fine-tune knowledge, prompt behaviour** — Backwards — fine-tuning is poor at volatile knowledge; behaviour is what weights encode well.
</details>

---

## Section B — Data quality, diversity & contamination `[fine-tuning-inference] s2`

### Q6. You have 2M scraped instruction pairs and a competitor shipped a strong model on ~8k curated ones. Where do you start?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] s2` · 🔮 Representative

- Train on all 2M — more data dominates curation
- ✅ Curate a clean, stratified subset, judge-filter and decontaminate it, train, eval, then add data where error analysis says it's thin
- Use the 2M as-is but raise the learning rate to absorb it faster

<details><summary>Show answer & why each option</summary>

- **Train on all 2M** — Raw volume with no quality/diversity control slides down the QDC frontier (low diversity, likely contamination); small-and-clean beats large-and-noisy.
- ✅ **Curate + decontaminate + iterate** — Quality-over-quantity plus an eval-driven loop is the senior recipe; the 2M is a pool to sample from, not a thing to train on wholesale.
- **Raise LR** — Parameter tuning can't fix a data problem.
</details>

### Q7. Your synthetic SFT set scores great on your in-distribution eval but the model is brittle on slightly-unusual real queries. Most likely cause?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] s2` · 🔮 Representative

- The model is too small — scale it up
- ✅ Diversity collapse — the generator over-preferred one style, so the set lacks OOD coverage; add a diversity gate (embedding spread) and diversify seeds
- Quality is too low — filter harder with the judge

<details><summary>Show answer & why each option</summary>

- **Too small** — Size doesn't fix a data-coverage problem; the symptom points at the diversity axis.
- ✅ **Diversity collapse → diversity gate** — On the QDC frontier, diversity drives OOD generalisation; an unguarded generator narrows to one style. Embedding-spread filtering and diverse seeds restore coverage.
- **Filter harder** — Harder quality filtering often *reduces* diversity further, worsening the OOD brittleness.
</details>

### Q8. After a fine-tune the model sometimes never stops generating, or echoes the prompt back. The data and loss curve looked fine. First thing to check?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] s2` · 🔮 Representative

- ✅ The chat template / token masking — wrong template, missing EOS, or training on prompt tokens
- The learning rate is too high
- The dataset is too small

<details><summary>Show answer & why each option</summary>

- ✅ **Chat template / token masking** — Silent format-drift bugs: a mismatched template or unmasked prompt tokens teach the model to never stop or to echo prompts, with a healthy-looking loss.
- **LR too high** — Causes instability/divergence, not specifically run-on or echoing.
- **Dataset too small** — Size doesn't produce run-on generation or prompt-echoing; the formatting layer does.
</details>

### Q9. Two expert annotators persistently disagree on ~12% of your legal SFT labels. What's the senior move?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] s2` · 🔮 Representative

- Keep all examples — disagreement adds useful variety
- Average the two labels into one
- ✅ Quantify with Cohen's kappa, re-label conflicts with a senior third reviewer, and drop examples that never converge

<details><summary>Show answer & why each option</summary>

- **Keep all** — Labels experts can't agree on are noise, not diversity; training on them teaches confident wrong answers.
- **Average** — Averaging contradictory expert judgments fabricates a label neither endorsed — still noise.
- ✅ **Kappa + escalate + drop non-converging** — Measuring agreement, escalating conflicts, and dropping non-converging items removes label noise without inventing ground truth.
</details>

### Q10. Your fine-tune reports a big jump on the task eval, but you suspect the win is inflated. Most likely culprit and the check?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] s2` · 🔮 Representative

- Overfitting — train for fewer epochs
- ✅ Contamination — eval examples (or near-duplicates) leaked into training; decontaminate with exact + fuzzy match against the held-out set and re-measure
- The judge model is too lenient

<details><summary>Show answer & why each option</summary>

- **Overfitting** — Possible but wouldn't specifically *inflate the held-out eval*; the classic cause of too-good numbers is leakage.
- ✅ **Contamination → exact + fuzzy decontamination** — Leakage is the standard reason eval scores look too good; exact+fuzzy dedup against a sacred held-out set is the fix and the proof.
- **Lenient judge** — Would inflate judged outputs generally, not produce a jump tied to this specific fine-tune.
</details>

---

## Section C — LoRA / QLoRA / PEFT `[fine-tuning-inference] l3`

### Q11. An interviewer asks why a rank-8 `B·A` update can approximate a full fine-tune of a 12,288-dimensional attention weight. Strongest answer?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] l3` · 🔮 Representative

- Because rank-8 matrices can represent any 12,288-dim matrix
- ✅ You're approximating the fine-tuning delta ΔW, not W — and the delta is intrinsically low-rank, so a small-r B·A captures it (LoRA found r=1–2 sufficed on GPT-3)
- Because the base weights are frozen, the update can be any size

<details><summary>Show answer & why each option</summary>

- **Represent any matrix** — False — a rank-8 factorisation can't represent an arbitrary full-rank matrix; the point is what you're approximating.
- ✅ **Approximating the low-rank delta ΔW** — The PEFT thesis: the adaptation manifold is low-dimensional, so the update (not the weight) factorises at tiny rank.
- **Frozen weights** — Freezing W is unrelated to why a low rank suffices.
</details>

### Q12. Your LoRA fine-tune (q_proj + v_proj, r=16) consistently lands ~3 points below a full fine-tune on the held-out set. The data is clean. What do you try first?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] l3` · 🔮 Representative

- ✅ Expand target modules to all linear layers (and raise rank if needed) — coverage, not rank alone, is what matches full FT
- Switch to full fine-tuning immediately
- Lower the learning rate to stabilise training

<details><summary>Show answer & why each option</summary>

- ✅ **Expand target modules** — The QLoRA ablation shows q/v-only doesn't match full FT; LoRA on all linear layers does. Coverage is the usual culprit for a plateau.
- **Switch to full FT** — Premature — you haven't turned the two dials (coverage, rank) that usually close the gap.
- **Lower LR** — A stability tweak doesn't address a capacity/coverage ceiling.
</details>

### Q13. A teammate says "we used LoRA, so serving will use much less GPU memory than the base model." Correct them.

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] l3` · 🔮 Representative

- They're right — LoRA models are smaller at inference
- ✅ The savings are at training (no optimizer state for frozen weights); a merged LoRA serves at full size — the only inference win is hot-swapping many adapters over one shared base
- Only if you also quantize the adapter to 4-bit

<details><summary>Show answer & why each option</summary>

- **Smaller at inference** — Wrong — a merged LoRA is a full dense matrix; serving memory and latency match the original.
- ✅ **Savings are at training; multi-adapter is the only serving win** — Train-time memory drops ~1000× via fewer trainable params; merged inference is unchanged. Multi-tenant adapter swapping is the lone serving-memory benefit.
- **Quantize the adapter** — The ~17 MB adapter is negligible; quantizing it changes nothing about the dense base.
</details>

### Q14. You need to fine-tune a 33B model and you have a single 48GB GPU. What's the defensible approach, and what cost should you flag?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] l3` · 🔮 Representative

- Full fine-tuning in fp16 — it's the highest quality
- ✅ QLoRA (4-bit NF4 base + BF16 adapters, paged optimizer) — ~24.7GB fits 48GB; flag the ~30–40% runtime tax from dequantizing on the fly and cover all linear layers for quality
- Plain LoRA in fp16 — skip quantization

<details><summary>Show answer & why each option</summary>

- **Full fp16** — A 33B fp16 full fine-tune needs hundreds of GB; it won't fit on 48GB.
- ✅ **QLoRA** — The memory math fits 33B on one 48GB GPU; the tradeoff is slower steps, and all-linear coverage recovers full-FT-level quality.
- **Plain LoRA fp16** — A 33B fp16 base plus activations/gradients still blows past 48GB; quantizing the base (QLoRA) is what makes it fit.
</details>

### Q15. When is full (or near-full) fine-tuning genuinely worth its cost over LoRA?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] l3` · 🔮 Representative

- Always — full fine-tuning is strictly better
- Never — LoRA is mathematically equivalent
- ✅ For large distribution shifts or new capabilities (new language/modality-style, heavy domain adaptation) where you're moving the base prior, not nudging behaviour

<details><summary>Show answer & why each option</summary>

- **Always** — LoRA matches full FT on behaviour-shaped tasks with adequate coverage/rank; "always" ignores the parity result and the cost.
- **Never / equivalent** — Not equivalent: a low-rank update can't capture a genuinely high-rank change.
- ✅ **Large distribution shifts / new capabilities** — Capability-shaped, high-rank changes can exceed what a small-r adapter captures even at full coverage — that's where full FT earns its cost.
</details>

---

## Section D — Preference tuning: DPO vs RLHF `[fine-tuning-inference] p4`

### Q16. You're aligning a customer-support chat model to a preferred tone and refusal style, with a small team and limited GPU budget. Which approach fits best and why?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] p4` · 🔮 Representative

- Full RLHF with PPO and a trained reward model
- ✅ SFT then DPO — a dense style/refusal preference is exactly DPO's sweet spot: one stable loss, no reward model, no PPO infra
- Prompt engineering only — never tune preferences

<details><summary>Show answer & why each option</summary>

- **Full RLHF/PPO** — Overkill for a dense preference over short answers: PPO needs a reward model and four resident models, and is unstable.
- ✅ **SFT then DPO** — The chosen/rejected signal over short answers is dense, so DPO is stable, cheap, and competitive or better than PPO here.
- **Prompt only** — A prompt can nudge tone but won't durably encode a consistent refusal/style preference.
</details>

### Q17. In an interview you explain DPO. What's the one-sentence core that signals real understanding?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] p4` · 🔮 Representative

- DPO doesn't need a reward model
- ✅ Under Bradley-Terry + a KL-bounded objective, the optimal policy is a closed-form function of the reference and reward, so the reward is implicit in the policy/reference log-ratio — letting DPO replace the RM and PPO with one classification loss
- DPO uses reinforcement learning more efficiently than PPO

<details><summary>Show answer & why each option</summary>

- **No reward model** — True but shallow — it's the conclusion, not the mechanism, and the exact weak answer interviewers flag.
- ✅ **The derivation chain** — "Your LM is secretly a reward model" — this is what separates a strong answer from reciting "no reward model."
- **Efficient RL** — DPO is not RL at all — it's a supervised classification loss with no policy sampling.
</details>

### Q18. You're building a model that must solve multi-step math and verify tool-call success. Why might PPO/RLHF beat DPO here?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] p4` · 🔮 Representative

- PPO trains faster than DPO
- ✅ The reward is sparse and shaped (correctness over long horizons); a reward model can propagate credit across steps in ways a binary preference loss can't
- DPO can't be used on math problems at all

<details><summary>Show answer & why each option</summary>

- **PPO faster** — False — PPO is heavier and slower; speed is DPO's advantage.
- ✅ **Sparse, shaped, long-horizon reward** — DPO's dense pairwise signal suits short-answer preference; sparse long-horizon rewards (reasoning, tools) are where RL/RM-based methods earn their cost.
- **Can't use DPO on math** — DPO can be applied, but it's a poorer fit for sparse long-horizon correctness — the point is fit, not impossibility.
</details>

### Q19. After heavy DPO the model is more on-tone but its answers feel repetitive and slightly shallower factually. What's happening and what do you do?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] p4` · 🔮 Representative

- Nothing — preference tuning only improves the model
- ✅ It's the alignment tax (reduced diversity + DPO preference warping); diversify/refresh preference pairs, don't over-train, adjust sampling, and gate on a general eval to bound the regression
- Raise beta to 1.0 to make the model more confident

<details><summary>Show answer & why each option</summary>

- **Nothing** — Ignores the alignment tax: RLHF/DPO can reduce diversity and warp toward preference-pair shape over substance.
- ✅ **Alignment tax → diversify + restraint + general-eval gate** — QDC shows alignment narrows diversity; DPO can overfit pair shape. Fix with data freshness/diversity, restraint, and a gate — measure the trade.
- **Beta to 1.0** — Beta controls divergence from the reference, not confidence; cranking it may worsen drift.
</details>

### Q20. Which statement about RLHF failure modes vs DPO failure modes is correct?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] p4` · 🔮 Representative

- Both fail identically — there's no meaningful difference
- ✅ PPO/RLHF fails by reward hacking (gaming an explicit reward model); DPO fails by preference warping and OOD brittleness against a frozen reference
- DPO is immune to over-optimisation because it has no reward model

<details><summary>Show answer & why each option</summary>

- **Fail identically** — They differ in kind, and knowing that is the reading-depth signal.
- ✅ **Reward hacking vs preference warping** — Explicit-RM methods are gamed (Goodhart); DPO overfits the shape of fixed preference pairs and is brittle if the reference drifts.
- **DPO immune** — The DPO paper explicitly flags reward over-optimisation and OOD generalisation as open limitations.
</details>

---

## Section E — Serving & precision `[fine-tuning-inference] sv5`

### Q21. You must deploy a 70B model on 8×A100 40GB (320GB total). Walk through the precision choice. What's the strongest plan?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] sv5` · 🔮 Representative

- FP8 — it's the most cost-efficient precision and halves memory
- ✅ AWQ or GPTQ int4 (~35GB weights) — fits comfortably with KV-cache headroom; FP8 is off the table on A100, and int4 holds <0.2 ppl delta on a 70B
- Keep FP16 and shard across all 8 GPUs

<details><summary>Show answer & why each option</summary>

- **FP8** — A100 has no FP8 tensor cores; FP8 is native only on H100/Blackwell. Naming this hardware constraint is the point.
- ✅ **AWQ/GPTQ int4** — Reason from memory and hardware: FP16≈140GB won't leave KV headroom comfortably, int4≈35GB is comfortable; int4 quality is near-lossless on 100B-class models.
- **FP16 sharded** — 140GB fits in 320GB but leaves little room for KV cache at scale and throws away a free 4× memory cut for ~0.15 ppl.
</details>

### Q22. Your chatbot shares a long system prompt across most requests and TTFT is high under load. Which lever gives the biggest, most direct win?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] sv5` · 🔮 Representative

- ✅ Prefix caching on a paged KV store — the shared prompt's KV is computed once and reused, so each request only pays for its delta
- Speculative decoding to generate tokens faster
- Switch to int3 quantization to free memory

<details><summary>Show answer & why each option</summary>

- ✅ **Prefix caching** — Shared prefixes are exactly what PagedAttention's copy-on-write / prefix caching targets; with >5% overlap it pays for itself within one round trip (1.7× first-token latency in the Vicuna prod number).
- **Speculative decoding** — Helps TPOT at small batch, not the prefill cost of re-encoding a shared prompt.
- **int3** — Adds real quality loss (+0.54 ppl) and doesn't address redundant prefill.
</details>

### Q23. You enabled speculative decoding and saw a nice latency win at low traffic, but at peak (high concurrency) it got slightly slower. Why?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] sv5` · 🔮 Representative

- The draft model's acceptance rate drops to zero at high load
- Speculative decoding silently changes the output distribution at scale
- ✅ At high batch the target is already throughput-bound, so the extra draft compute becomes overhead rather than savings — spec-decoding helps when decode is bandwidth-bound (small batch)

<details><summary>Show answer & why each option</summary>

- **Acceptance rate → zero** — Acceptance is workload/prefix-driven, not a function of batch size; it doesn't collapse to zero because concurrency rose.
- **Changes the distribution** — False — draft-and-verify is mathematically lossless at any batch size.
- ✅ **Throughput-bound at high batch** — The benefit comes from amortising per-token bandwidth at small batch; at high batch the draft compute is pure overhead.
</details>

### Q24. An interviewer asks why vLLM reports ~23× over naive batching. What's the strongest mechanism-level answer?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] sv5` · 🔮 Representative

- It batches more requests together onto the GPU
- ✅ Static batching pads to the longest sequence and runs the whole batch to completion (short requests idle); continuous batching reschedules every forward pass, evicting finished requests and admitting new ones — and PagedAttention removes the 60–80% KV waste, so the 23× is both together
- It uses int4 quantization to run faster

<details><summary>Show answer & why each option</summary>

- **Batches more** — Too shallow and it's what naive batching already does.
- ✅ **Continuous batching + PagedAttention** — Names static-vs-continuous, the padding/idle waste, the iteration-level reschedule, and that the headline number includes PagedAttention (Orca's 36.9×, vLLM's 23×).
- **int4** — A separate, orthogonal lever; the 23× is a scheduling win, not precision.
</details>

### Q25. Your traffic is mostly multi-call agentic programs that reuse the same prompts in slightly varying forms and emit strict JSON. Which serving stack fits best?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] sv5` · 🔮 Representative

- TensorRT-LLM, because it has the highest peak throughput
- TGI, because it has the lowest TTFT
- ✅ SGLang — RadixAttention exploits the heavy shared-prefix reuse (50–99% hit rates) and its compressed-FSM decoder handles JSON/grammar; up to 5× faster than vLLM on multi-call Llama-7B

<details><summary>Show answer & why each option</summary>

- **TensorRT-LLM** — Wins raw throughput on a fixed workload but requires per-config engine rebuilds and isn't specialised for prefix-tree reuse or grammar-constrained JSON.
- **TGI** — Its low-concurrency TTFT edge doesn't address heavy prefix reuse or structured JSON decoding.
- ✅ **SGLang** — Multi-call structured generation with high prefix sharing is exactly its niche; RadixAttention + FSM decoding are built for it.
</details>

---

## Section F — The full fine-tuning capstone `[fine-tuning-inference] cp6`

### Q26. For the support-assistant scenario (house voice + strict format + tool schema, over weekly-changing tickets), what's the right primary architecture?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] cp6` · 🔮 Representative

- ✅ Fine-tune the voice/format/tool behaviour, keep RAG for the live ticket knowledge, and put per-turn rules in the prompt — classify the failure mode per layer
- Fine-tune on everything including the ticket contents so the model knows it all
- RAG only — retrieve everything and skip fine-tuning

<details><summary>Show answer & why each option</summary>

- ✅ **Layer by failure mode** — Voice/format/tool reliability are stable behaviour gaps (→ fine-tune); weekly-changing tickets are volatile knowledge (→ RAG); rules are per-turn (→ prompt).
- **Fine-tune everything** — Facts that change weekly go stale in weights, can't be cited, and force constant retraining.
- **RAG only** — RAG fixes knowledge, not behaviour; it won't durably enforce house voice, strict format, or reliable tool-schema emission.
</details>

### Q27. You have 50k support tickets. What's the strongest data move before tuning the 7B?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] cp6` · 🔮 Representative

- Train on all 50k to maximise signal
- ✅ Stratify by ticket type/product/outcome, judge-filter for quality, drop low-kappa examples, and decontaminate against the held-out eval — then train a clean subset and let error analysis say where to add data
- Generate 200k synthetic tickets to add volume

<details><summary>Show answer & why each option</summary>

- **Train on all 50k** — Raw volume with no quality/diversity gating risks diversity collapse and contamination (Guanaco beat larger Alpaca).
- ✅ **Stratify + filter + drop low-kappa + decontaminate** — Coverage, quality filtering, kappa-based noise removal, and decontamination are the four-step recipe; quality-over-quantity plus an eval-driven loop.
- **200k synthetic** — Unguarded synthetic generation collapses diversity to one style and adds contamination risk.
</details>

### Q28. How do you prove the fine-tune got better, not just different?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] cp6` · 🔮 Representative

- Run 20 prompts through both models and read the outputs
- Compare task-eval scores only; if higher, ship
- ✅ Three layers vs the same base pipeline: capability benchmarks (regression), a decontaminated golden set (in-domain), and a calibrated LLM-as-judge — gated on a capability floor and a task ceiling

<details><summary>Show answer & why each option</summary>

- **Eyeball 20 prompts** — The exact weak answer the rubric flags — no hold-out, no decontamination, no regression detection, no calibration.
- **Task-eval only** — Blind to catastrophic forgetting and contamination — a higher number can be leakage or hide a regression.
- ✅ **Three-layer eval + floor/ceiling gate** — Disambiguates better-vs-different; decontamination makes the number real; the floor+ceiling gate makes the ship decision multi-objective.
</details>

### Q29. The fine-tune lifts support-task accuracy by 9 points but MMLU drops 6. What do you do?

> **Difficulty:** 🔴 Hard · `[fine-tuning-inference] cp6` · 🔮 Representative

- Ship it — MMLU doesn't matter for support
- ✅ Treat it as catastrophic forgetting: add ~5–15% general-instruction replay, lower the LR / bound the LoRA update, re-train and re-gate on the capability floor (MMLU −2 max) and task ceiling
- Abandon fine-tuning and go back to prompting

<details><summary>Show answer & why each option</summary>

- **Ship it** — Dismissing a 6-point general drop ignores the multi-objective reality; off-distribution failures surface in production.
- ✅ **Replay + bounded update + re-gate** — Forgetting is a train-data composition problem; replay + a bounded update recover general skill, and the floor+ceiling gate makes the tradeoff explicit and shippable.
- **Abandon** — Over-correction — the 9-point gain is genuine; the fix is data composition and a capability floor.
</details>

### Q30. The BF16 fine-tune passed your gates. You quantize to int4 for serving. What's the disciplined final step?

> **Difficulty:** 🟡 Medium · `[fine-tuning-inference] cp6` · 🔮 Representative

- Ship the quantized model — int4 quality loss is negligible so re-eval is unnecessary
- ✅ Re-run the golden set (and capability check) on the actual quantized artifact, and freeze the golden set as a regression CI gate for every future change
- Trust the BF16 eval numbers since quantization only affects speed

<details><summary>Show answer & why each option</summary>

- **Ship without re-eval** — Even small per-model quality shifts from quantization can move a borderline gate; "negligible in general" isn't "negligible for this model on this task."
- ✅ **Re-eval the quantized artifact + freeze the golden set** — You evaluate the artifact you actually serve; freezing the golden set as a CI gate guards against silent regressions on the next base/data/requant change.
- **Trust BF16 numbers** — Quantization affects quality, not just speed; the served int4 model is a different artifact.
</details>

---

## Section G — Reported fine-tuning & inference interview questions

> Observed in real 2026 loops. Sources: [Devinterview-io/llms-interview-questions](https://github.com/Devinterview-io/llms-interview-questions), [KalyanKS-NLP/LLM-Interview-Questions-and-Answers-Hub](https://github.com/KalyanKS-NLP/LLM-Interview-Questions-and-Answers-Hub), [amitshekhariitbhu/ai-engineering-interview-questions](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions), Sebastian Raschka *State of RL for LLM Reasoning* (https://magazine.sebastianraschka.com/p/the-state-of-llm-reasoning-model-training). Rehearse an answer; the checkpoints above are the drill.

### Q31. What is LoRA vs full fine-tuning? Explain QLoRA / why 4-bit?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Devinterview](https://github.com/Devinterview-io/llms-interview-questions), [LoRA paper](https://arxiv.org/abs/2106.09685))
> **Senior frame:** LoRA freezes W and trains a low-rank `B·A` update to the *delta* (Q11); ~1000× fewer trainable params → train-time memory win, not serving (Q13). QLoRA quantizes the frozen base to 4-bit NF4 with BF16 adapters + a paged optimizer so a 33B fits one 48GB GPU (Q14) — at a ~30–40% runtime tax. Cover all linear layers to match full FT (Q12).

### Q32. What is PEFT and its techniques (LoRA, adapters, prefix, prompt tuning)?
> **Difficulty:** 🟡 Medium · ✅ Reported ([HF PEFT guide](https://huggingface.co/docs/peft/en/index))
> **Senior frame:** Parameter-efficient fine-tuning trains a small set of new/selected params while freezing the base: LoRA (low-rank deltas), adapters (bottleneck layers), prefix/prompt tuning (learned soft tokens). Trade coverage/capacity for cost; LoRA on all linear layers is the strong default (Q12, Q15).

### Q33. Explain RLHF (SFT → RM → PPO). DPO vs PPO at the algorithm level?
> **Difficulty:** 🔴 Hard · ✅ Reported ([DPO paper](https://arxiv.org/abs/2305.18290), [Raschka RLHF vs DPO](https://sebastianraschka.com/faq/docs/rlhf-vs-dpo.html))
> **Senior frame:** RLHF = SFT, then a reward model on human preference pairs, then PPO to maximise reward under a KL penalty (four models, unstable). DPO derives a closed-form optimal policy under Bradley-Terry + KL and collapses RM+PPO into one classification loss (Q17). DPO for dense short-answer preference; RL for sparse shaped long-horizon reward (Q18).

### Q34. What is GRPO and why did DeepSeek-R1 use it over PPO?
> **Difficulty:** 🔴 Hard · ✅ Reported ([DeepSeek-R1](https://arxiv.org/abs/2501.12948), [DeepSeekMath](https://arxiv.org/abs/2402.03300))
> **Senior frame:** GRPO (Group Relative Policy Optimization) drops PPO's separate value/critic model — it estimates advantage from the *relative* reward of a group of sampled outputs per prompt. Cheaper and more stable for reasoning post-training where the reward is a verifiable correctness signal; it's the center of 2026 reasoning training. Contrast with DPO (no sampling) and PPO (critic + rollouts).

### Q35. When do you fine-tune vs RAG vs better prompts? Give an example where you'd NOT fine-tune despite having data.
> **Difficulty:** 🟡 Medium · ✅ Reported ([IBM RAG-vs-FT](https://www.ibm.com/think/topics/rag-vs-fine-tuning-vs-prompt-engineering))
> **Senior frame:** RAG for volatile/citable knowledge; fine-tune for stable behaviour (format/tone/tools) or a capability shift; prompt for per-turn rules (Q1–Q5). Don't fine-tune a weekly-changing knowledge base even with data — it goes stale, can't cite, and forces constant retraining (Q26). Fine-tune only the residual prompting+RAG can't reach.

### Q36. What is quantization and why? INT8 vs FP16? GPTQ vs AWQ vs GGUF vs BnB-NF4 — when?
> **Difficulty:** 🟡 Medium · ✅ Reported ([vrlatech quantization 2026](https://vrlatech.com/llm-quantization-explained-int4-int8-fp8-awq-and-gptq-in-2026/))
> **Senior frame:** Lower-precision weights/activations cut memory and (on the right hardware) compute. INT8 halves memory vs FP16 with tiny loss; INT4 (GPTQ/AWQ) is near-lossless on 100B-class models (Q21). GGUF = CPU/llama.cpp; BnB-NF4 = QLoRA training base; AWQ/GPTQ = GPU serving. Match to hardware — FP8 needs Hopper/Blackwell, not A100 (Q21).

### Q37. How does FP8 give ~33% throughput on H100?
> **Difficulty:** 🔴 Hard · ✅ Reported ([Baseten FP8](https://www.baseten.co/blog/33-faster-llm-inference-with-fp8-quantization/))
> **Senior frame:** Hopper has native FP8 tensor cores, so FP8 GEMMs roughly double matmul throughput *and* halve weight/activation/KV memory with minimal accuracy work — a both-axes win, unlike weight-only INT4 (which saves memory but keeps FP16 activations). Pairs with FlashAttention-3. Off the table on Ampere/A100 (`fundamentals` Q45, Q21 here).

### Q38. What is the KV cache and why does it matter? How does PagedAttention eliminate fragmentation? How does FlashAttention reduce memory?
> **Difficulty:** 🔴 Hard · ✅ Reported ([PagedAttention](https://arxiv.org/abs/2309.06180), [FlashAttention-2](https://arxiv.org/abs/2307.08691))
> **Senior frame:** KV cache stores past K/V so decode is incremental — it dominates per-token memory and bandwidth (`fundamentals` Q37–Q38). PagedAttention allocates KV in fixed-size pages on demand, cutting 60–80% contiguous-reservation waste (and the "OOM with free memory" pattern, [system-design.md](./system-design.md)). FlashAttention tiles Q/K/V in SRAM and never materializes the T×T matrix in HBM — exact output, I/O-bound win (`fundamentals` Q42).

### Q39. vLLM vs TensorRT-LLM — when each? Deploy a 70B Llama with continuous batching.
> **Difficulty:** 🔴 Hard · ✅ Reported ([Modal vLLM vs TGI](https://modal.com/blog/vllm-vs-tgi-article))
> **Senior frame:** vLLM = flexible, PagedAttention + continuous batching, great default; TensorRT-LLM = highest peak throughput on a fixed config but per-config engine rebuilds. SGLang for heavy prefix reuse + grammar JSON (Q25). Deploy 70B: quantize to int4 on A100 (Q21), continuous batching + PagedAttention, scale on queue depth ([system-design.md](./system-design.md)).

### Q40. How does speculative decoding work and what speedup should you expect? Static vs dynamic (continuous) batching?
> **Difficulty:** 🔴 Hard · ✅ Reported ([EAGLE](https://arxiv.org/abs/2401.15077), [Anyscale continuous batching](https://www.anyscale.com/blog/continuous-batching-llm-inference))
> **Senior frame:** A small draft model proposes k tokens, the target verifies them in one pass — lossless, wins at *small batch* where decode is bandwidth-bound (Q23); EAGLE-3 > Medusa > vanilla. Static batching pads to the longest seq and runs to completion (idle waste); continuous batching reschedules every forward pass (Orca 36.9×, vLLM 23× with PagedAttention, Q24).

---

<sub>⬅ [Back to the index](./README.md) · Related: [Fundamentals →](./fundamentals.md) · [System design →](./system-design.md) · [LLMOps →](./llmops.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
