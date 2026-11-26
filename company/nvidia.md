# Nvidia — AI Engineer Interview (2026)

🏗️ *"Ship the platform."* A conventional coding and design loop that bends toward LLM inference and, for CUDA-heavy roles, C++ and GPU architecture.

← Back to [main README](../README.md) · [company index](README.md)

**On landed.jobs:** [open roles at Nvidia](https://www.landed.jobs/companies/nvidia?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-jobs) · [the full Nvidia loop guide with a prep plan](https://www.landed.jobs/resources/guides/nvidia-ai-interview-guide?utm_source=github&utm_medium=referral&utm_campaign=github-awesome-ai-engineer-interview&utm_content=company-guide)

> [!NOTE]
> **The loop at a glance.** *Second-hand ([interviewing.io](https://interviewing.io/nvidia-interview-questions), 2026, based on conversations with current and former Nvidia engineers):* a **30-minute** recruiter call (skippable with a strong referral) → a **1-hour** technical phone screen → sometimes a 30-minute hiring-manager call → a **4–5 hour onsite**: two 1-hour coding rounds in CoderPad, one or two 1-hour system-design rounds, and a 1-hour hiring-manager round. About **6–8 weeks** in all. The page says Nvidia is focused on hiring engineers with LLM experience, that some phone screens are entirely LLM questions in PyTorch, and that CUDA-heavy roles may interview in C++. The earlier version of this page described a 2025–2026 shift to hardware-software co-design rounds; we could not find a public source for that, so it is **not publicly confirmed**. No official Nvidia description of the loop or its AI-tool policy was found.
>
> Last checked: 17 Nov 2026.

## The loop

1. **Recruiter call (30 min).**
2. **Technical phone screen (1 hour)**: about 15 minutes of chat, then a 45-minute technical part. For some roles it is all LLM questions in PyTorch.
3. **Hiring-manager call (30 min)**: not always.
4. **Onsite: coding** (two 1-hour rounds, CoderPad). C++ is likely for CUDA-heavy roles.
5. **Onsite: system design** (1 hour; some candidates get two).
6. **Onsite: hiring manager** (1 hour).

Source: [interviewing.io](https://interviewing.io/nvidia-interview-questions) (second-hand, 2026). Composition varies by team and role. A dedicated hardware-software co-design round is **not publicly confirmed**.

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| Phone screen (LLM) | Implement beam search for LLM inference and give the time complexity of each operation | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/nvidia-interview-questions) |
| Phone screen (LLM topics) | Mixture-of-experts, autoregressive decoding with KV-cache, LoRA, high-level distributed computing | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/nvidia-interview-questions) |
| System design | Design a chatbot for a website | ✅ Reported · [second-hand, interviewing.io, 2026](https://interviewing.io/nvidia-interview-questions) |
| ML theory | "Explain the roofline model" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| Distributed training | "Distributed training memory optimization" | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |
| CUDA kernel | Low-level CUDA kernel implementation | 🔮 Representative · no public source found (re-labelled 17 Nov 2026) |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After the roofline model: *"Is this kernel compute-bound or memory-bound, and what's the fix?"*
- After distributed-training memory: *"Where does ZeRO/activation-checkpointing buy you the most, and at what cost?"*
- After the CUDA kernel: *"Now tile it for shared memory and explain the occupancy trade-off."*

## What they're really testing

- **LLM inference fluency.** The second-hand source says some screens are all LLM questions, such as beam search and KV-cache decoding.
- **Hardware awareness.** For CUDA-heavy roles, expect C++ and questions that need you to reason about the model *and* the GPU together; the same source suggests learning CPU/GPU architecture for design rounds.
- **Roofline thinking.** Knowing whether a workload is compute- or memory-bound is the core mental model.
- **CUDA-level fluency.** Low-level kernel implementation separates platform builders from model users.
- **Distributed-training memory.** Sharding, checkpointing, and memory optimization at scale are expected.
- **Maker mindset.** Nvidia's culture rewards building close to the metal.

## How to prep

- **LLM screen** → implement beam search and KV-cache decoding in PyTorch, and state the complexity of each step. See [content/01](../content/01-llm-fundamentals.md).
- **Roofline / compute-vs-memory** → [content/07](../content/06-fine-tuning-and-inference.md); learn to classify a kernel by its bottleneck.
- **Distributed-training memory** → [content/03](../content/06-fine-tuning-and-inference.md) and [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) (parallelism, checkpointing, ZeRO).
- **CUDA kernels** → practice a tiled matmul kernel from scratch; understand shared memory, occupancy, and coalescing.
- **System design** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md).

## Culture signals

**"Speed of light" engineering, maker culture, AI-first infra.** CUDA-heavy roles reward engineers who think about hardware and software as one system.

## Cross-links

- 🛠️ Worked design: [LLM inference at scale / hardware-aware serving](../answers/llm-inference-at-scale.md)


<!-- landed-apply-cta -->
<table><tr><td>
<b>⚡ <a href="https://app.landed.jobs/signup?from=github&amp;cta=github-guide-apply&amp;item=awesome-ai-engineer-interview&amp;utm_source=github&amp;utm_medium=referral&amp;utm_campaign=github-awesome-ai-engineer-interview&amp;utm_content=guide-apply-file">Auto-apply to AI engineer jobs that match you →</a></b><br>
Upload your resume once. <a href="https://www.landed.jobs">landed</a> finds the roles that fit you and applies within hours of them going live, so you actually hear back.
</td></tr></table>

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Nvidia. MIT-licensed.</sub>
