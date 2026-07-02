# Nvidia — AI Engineer Interview (2026)

🏗️ *"Ship the platform."* The one lab whose loop clearly *changed* in 2025–2026: ML screens now push down to **hardware-software co-design** and **CUDA-level** implementation.

← Back to [main README](../README.md) · [company index](README.md)

> [!NOTE]
> **The loop at a glance.** Recruiter → phone coding → **3–5 onsite** rounds (coding, system design, ML, and hardware-software co-design). Roughly **5 onsite interviews / ~5 hours**. The distinguishing shift for 2025–2026 is **hardware-software co-design + CUDA-level** questions — the clearest maker-vs-researcher signal in this section, and the only reported *philosophy change* across the labs here.

## The loop

1. **Recruiter call.**
2. **Phone coding.**
3. **Onsite: coding.**
4. **Onsite: system design.**
5. **Onsite: ML** — theory + co-design.
6. **Onsite: hardware-software co-design** — CUDA-level implementation.

Source: candidate reports [114], [115].

## Questions reported in the wild

| Round | Question | Provenance |
|---|---|---|
| ML theory | "Explain the roofline model" | ✅ Reported — candidate report [114] |
| Distributed training | "Distributed training memory optimization" | ✅ Reported — candidate report [114] |
| CUDA kernel | Low-level CUDA kernel implementation | ✅ Reported — candidate report [114] |

### 🔮 Representative follow-ups you should expect

*Not reported — natural extensions given the loop.*

- After the roofline model: *"Is this kernel compute-bound or memory-bound, and what's the fix?"*
- After distributed-training memory: *"Where does ZeRO/activation-checkpointing buy you the most, and at what cost?"*
- After the CUDA kernel: *"Now tile it for shared memory and explain the occupancy trade-off."*

## What they're really testing

- **Hardware-software co-design.** The 2025–2026 shift means you must reason about the model *and* the silicon together.
- **Roofline thinking.** Knowing whether a workload is compute- or memory-bound is the core mental model.
- **CUDA-level fluency.** Low-level kernel implementation separates platform builders from model users.
- **Distributed-training memory.** Sharding, checkpointing, and memory optimization at scale are expected.
- **Maker mindset.** Nvidia's culture rewards building close to the metal.

## How to prep

- **Roofline / compute-vs-memory** → [content/07](../content/06-fine-tuning-and-inference.md); learn to classify a kernel by its bottleneck.
- **Distributed-training memory** → [content/03](../content/06-fine-tuning-and-inference.md) and [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md) (parallelism, checkpointing, ZeRO).
- **CUDA kernels** → practice a tiled matmul kernel from scratch; understand shared memory, occupancy, and coalescing.
- **System design** → [answers/llm-inference-at-scale.md](../answers/llm-inference-at-scale.md).

## Culture signals

**"Speed of light" engineering, maker culture, AI-first infra.** The co-design shift rewards engineers who think about hardware and software as one system.

## Cross-links

- 🛠️ Worked design: [LLM inference at scale / hardware-aware serving](../answers/llm-inference-at-scale.md)

---

<sub>Part of the [Landed](https://landed.jobs) AI-native jobs family · [company index](README.md) · No affiliation with Nvidia. MIT-licensed.</sub>
