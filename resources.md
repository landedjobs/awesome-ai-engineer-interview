<div align="center">

# 📚 Resource Hub — AI Engineer Interview (2026)

**Every link typed, annotated, and license-noted.** The one place to find the canonical paper, doc, video, course, tool, or repo for each topic.

[← Back to README](README.md) · [Questions](questions/README.md) · [Content](content/) · [Worked answers](answers/) · [Company loops](company/)

</div>

---

> **Legend** — 📄 paper · 📘 docs · 📰 article · 🎬 video · 🧑‍🏫 course · 🛠️ tool · 💻 repo
> Repos note **license + approx stars**. We ship MIT and only *adapt* MIT/Apache/CC0 (with attribution); CC-BY / CC-BY-SA (incl. autogen `microsoft/autogen`) are **link-only**. Curated to ~8 per topic — this is a map, not a dump. Updated **2026-07**.

## Contents

- [Start here — the canonical few](#start-here--the-canonical-few)
- [LLM fundamentals & transformers](#llm-fundamentals--transformers)
- [RAG & retrieval](#rag--retrieval)
- [Agents, tool use & MCP](#agents-tool-use--mcp)
- [Evals & LLM-as-judge](#evals--llm-as-judge)
- [LLMOps — cost, latency, serving](#llmops--cost-latency-serving)
- [Fine-tuning & post-training](#fine-tuning--post-training)
- [Security & guardrails](#security--guardrails)
- [System design & interview loops](#system-design--interview-loops)
- [Question banks & competitor repos](#question-banks--competitor-repos)

---

## Start here — the canonical few

If you read nothing else, read these.

- 📘 [AI Engineering (O'Reilly)](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) — Chip Huyen. The most-cited single book: foundations → prompt → RAG → agents → finetune → latency/cost. *(paid)*
- 🎬 [Neural Networks: Zero to Hero](https://karpathy.ai/zero-to-hero.html) — Andrej Karpathy. 9 lectures building GPT from scratch — the base for "implement attention from memory."
- 🧑‍🏫 [Full Stack LLM Bootcamp](https://fullstackdeeplearning.com/llm-bootcamp/) — 8 free lectures: prompting, RAG, agents, UX, evals, deployment.
- 💻 [openai-cookbook](https://github.com/openai/openai-cookbook) — OpenAI · MIT · ~67k★. Largest collection of production LLM patterns.
- 📘 [Claude Cookbook](https://platform.claude.com/cookbook) — Anthropic. Runnable recipes: tool use, MCP, extended thinking, prompt caching, PDF, vision.
- 📰 [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic. First-party reference for 2026's hottest topic.

## LLM fundamentals & transformers

- 🎬 [Intro to Large Language Models](https://youtube.com/watch?v=zjkBMFhNj_g) — Karpathy. The one-hour mental model of the generation loop.
- 🧑‍🏫 [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course) — tokenization, transformers, PEFT/LoRA fine-tuning.
- 📄 [RoFormer / RoPE](https://arxiv.org/abs/2104.09864) — rotary position embeddings; the "why RoPE extends to longer context" answer.
- 📄 [FlashAttention-2](https://arxiv.org/abs/2307.08691) — IO-aware exact attention; why long-context prefill isn't as slow as naive O(T²) suggests.
- 📰 [Coding the KV cache in LLMs](https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms) — Sebastian Raschka. The KV-cache mechanism, hands-on.
- 🛠️ [tiktoken](https://github.com/openai/tiktoken) — OpenAI · MIT. Count tokens before you reason about cost or context.
- 📰 [LLM research papers 2026 (part 1)](https://magazine.sebastianraschka.com/p/llm-research-papers-2026-part1) — Raschka's curated 2026 paper digest.
- 💻 [rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch) — Raschka · Apache-2.0. Build a GPT from scratch — the "from memory" primitives.

## RAG & retrieval

- 📘 [Enhancing RAG with contextual retrieval](https://platform.claude.com/cookbook/capabilities-contextual-embeddings-guide) — Anthropic. BM25 + embeddings + context + rerank; 35–49% fewer retrieval failures.
- 📘 [LangChain RAG docs](https://docs.langchain.com/oss/python/langchain/rag) — the canonical framework reference for the embed→retrieve→generate pipeline.
- 💻 [run-llama/llama_index](https://github.com/run-llama/llama_index) — LlamaIndex · MIT · ~43k★. Most-cited indexing/RAG framework; RAG-eval + agent modules.
- 💻 [vibrantlabsai/ragas](https://github.com/vibrantlabsai/ragas) — Apache-2.0 · ~14.5k★. Faithfulness, answer relevancy, context precision/recall metrics.
- 🎬 [Why Your RAG System Is Broken, and How to Fix It](https://youtube.com/watch?v=wexpoR1R03A) — Jason Liu. The IR-first debugging mindset.
- 🎬 [The 5 Levels of Text Splitting for Retrieval](https://youtube.com/watch?v=8OJC21T2SL4) — Greg Kamradt. Chunking families and when each fails.
- 📰 [Evaluating RAG (series)](https://eugeneyan.com/) — Eugene Yan. Canonical production-eval patterns.
- 🎬 [Structured generation with Outlines](https://youtube.com/watch?v=aNmfvN6S_n4) — Rémi Louf. Constrained decoding = no more parse failures.

## Agents, tool use & MCP

- 📄 [ReAct](https://arxiv.org/abs/2210.03629) — Thought/Action/Observation; the foundational agent loop.
- 📰 [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — Anthropic. Workflows vs agents; de-escalate to the simplest pattern.
- 📰 [Introducing the Model Context Protocol](https://www.anthropic.com/news/model-context-protocol) — Anthropic. "USB-C for AI integrations."
- 📘 [MCP — introduction & spec](https://modelcontextprotocol.io/docs/getting-started/intro) — resources, prompts, tools, sampling, roots, elicitation.
- 💻 [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) — MIT · ~36k★. Graph-structured agent orchestration with checkpoints.
- 💻 [openai/openai-agents-python](https://github.com/openai/openai-agents-python) — MIT · ~27k★. Minimal agents SDK (handoffs, guardrails, tracing).
- 💻 [letta-ai/letta (MemGPT)](https://github.com/letta-ai/letta) — Apache-2.0 · ~23k★. Tiered agent memory beyond the context window.
- 💻 [microsoft/autogen](https://github.com/microsoft/autogen) — **CC-BY-4.0 · ~59k★ · link-only** (research artifact license — study it, don't relicense).

## Evals & LLM-as-judge

- 📘 [Evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices) — OpenAI. Objective → dataset → grader → run + analyze.
- 📄 [Judging LLM-as-a-Judge (MT-Bench / Chatbot Arena)](https://arxiv.org/abs/2306.05685) — Zheng et al. Canonical judge-eval + bias taxonomy.
- 📄 [A Survey on LLM-as-a-Judge](https://arxiv.org/abs/2411.15594) — biases + mitigations, 1600+ citations.
- 📄 [G-Eval](https://arxiv.org/abs/2303.16634) — chain-of-thought NLG scoring.
- 💻 [confident-ai/deepeval](https://github.com/confident-ai/deepeval) — Apache-2.0 · ~16k★. "Pytest for LLMs" — 50+ metrics, CI gates.
- 📰 [Mastering agent evaluation](https://developer.nvidia.com/blog/mastering-agentic-techniques-ai-agent-evaluation/) — NVIDIA. The 3-metric agentic stack: outcome · trajectory · tool/reasoning.
- 📰 [Eval-driven development](https://www.braintrust.dev/articles/eval-driven-development) — Braintrust. Evals as the working spec.
- 🧑‍🏫 [Galileo Eval Engineering](https://docs.galileo.ai/learn/eval-engineering) — free course on metric design, hallucination metrics, agent eval.

## LLMOps — cost, latency, serving

- 📰 [LLM Cost Optimization 2026](https://www.maviklabs.com/blog/llm-cost-optimization-2026) — routing + caching + batching → 47–80% spend cut.
- 📰 [Mastering LLM inference optimization](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/) — NVIDIA. MQA/GQA, KV-cache, batching.
- 📄 [PagedAttention / vLLM](https://arxiv.org/abs/2309.06180) — the paging idea that killed KV-cache fragmentation; up to 24× throughput.
- 📰 [Continuous batching](https://www.anyscale.com/blog/continuous-batching-llm-inference) — Anyscale. 23× over static batching, explained.
- 💻 [langfuse/langfuse](https://github.com/langfuse/langfuse) — MIT · ~30k★. OTel-native LLM tracing + eval; the multi-framework default.
- 📰 [LLM observability with OpenTelemetry](https://opentelemetry.io/blog/2024/llm-observability/) — where to put spans in an agent mesh.
- 💻 [vllm-project/vllm](https://github.com/vllm-project/vllm) — Apache-2.0 · ~85k★. The de-facto open serving stack.
- 📰 [The Tail at Scale](https://research.google/pubs/the-tail-at-scale/) — Dean & Barroso. Hedged requests — the fix for p99 stragglers.

## Fine-tuning & post-training

- 📄 [LoRA](https://arxiv.org/abs/2106.09685) · [QLoRA](https://arxiv.org/abs/2305.14314) — the minimum-cost fine-tuning baseline; why 4-bit works.
- 📘 [PEFT — LoRA conceptual guide](https://huggingface.co/docs/peft/main/en/conceptual_guides/lora) — Hugging Face. The practitioner reference.
- 📄 [DPO](https://arxiv.org/abs/2305.18290) — preference optimization without a reward model or PPO loop.
- 📄 [DeepSeek-R1](https://arxiv.org/abs/2501.12948) · [DeepSeekMath (GRPO)](https://arxiv.org/abs/2402.03300) — GRPO, the center of 2026 reasoning post-training.
- 📰 [The State of RL for LLM Reasoning](https://magazine.sebastianraschka.com/p/the-state-of-llm-reasoning-model-training) — Raschka. Best practitioner summary of PPO → DPO → GRPO.
- 📰 [RAG vs fine-tuning vs prompt engineering](https://www.ibm.com/think/topics/rag-vs-fine-tuning-vs-prompt-engineering) — IBM. The decision framework interviewers probe.
- 💻 [huggingface/trl](https://github.com/huggingface/trl) — Apache-2.0 · ~18k★. SFT / DPO / GRPO trainers.
- 🧑‍🏫 [mlabonne/llm-course](https://github.com/mlabonne/llm-course) — Apache-2.0 · ~80k★. Fine-tuning + serving roadmap and notebooks.

## Security & guardrails

- 📘 [OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/) — the threat-model checklist (LLM01 prompt injection → LLM10).
- 📄 [EchoLeak (CVE-2025-32711)](https://arxiv.org/abs/2509.10540) — first real-world zero-click indirect injection vs M365 Copilot (CVSS 9.3).
- 📰 [The lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) — Simon Willison. Private data + untrusted content + egress = exfiltration.
- 📄 [CaMeL: defeating prompt injection by design](https://arxiv.org/abs/2503.18813) — the dual-LLM / control-flow-separation architecture.
- 💻 [protectai/llm-guard](https://github.com/protectai/llm-guard) — MIT · ~3k★. Input/output scanners (injection, PII, toxicity).
- 📰 [Guardrails tooling comparison](https://www.fuzzylabs.ai/blog-post/guardrails-for-llms-a-tooling-comparison) — NeMo Guardrails vs Guardrails AI, when each.

## System design & interview loops

- 📰 [Generative AI System Design Interview](https://igotanoffer.com/en/advice/generative-ai-system-design-interview) — sample prompts + rubrics.
- 📰 [OpenAI interview process & timeline](https://igotanoffer.com/en/advice/openai-interview-process) — authoritative six-step breakdown.
- 📰 [The Ultimate AI Research Engineer Interview Guide](https://www.sundeepteki.org/advice/the-ultimate-ai-research-engineer-interview-guide-cracking-openai-anthropic-google-deepmind-top-ai-labs) — Sundeep Teki. Cross-lab 2026 loop comparison.
- 📰 [AI system design interview questions](https://dev.to/arslan_ah/ai-system-design-interview-questions-chatgpt-rag-llm-inference-and-agents-1doi) — ChatGPT, RAG, LLM inference, agents.
- 💻 [alirezadir/Machine-Learning-Interviews](https://github.com/alirezadir/machine-learning-interviews) — MIT · ~8.5k★. Established ML system-design taxonomy (classic-ML heavy).
- 💻 [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) — Apache-2.0 · 100+★ runnable RAG + agent apps to build portfolio pieces.

## Question banks & competitor repos

We link the good ones and tell you where they stop — this repo exists because none of them combine per-lab loops + annotated resources + sourced questions + 2026 topics.

- 💻 [amitshekhariitbhu/ai-engineering-interview-questions](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions) — MIT · ~1.9k★. Living AI/GenAI/LLM/agentic question bank.
- 💻 [Devinterview-io/llms-interview-questions](https://github.com/Devinterview-io/llms-interview-questions) — 63 LLM Qs (2026). Good breadth, thin on loops/evals.
- 💻 [KalyanKS-NLP/LLM-Interview-Questions-and-Answers-Hub](https://github.com/KalyanKS-NLP/LLM-Interview-Questions-and-Answers-Hub) — Apache-2.0 · ~1k★. 100+ well-cited LLM Q&A; no 2026 loop guidance.
- 💻 [KalyanKS-NLP/RAG-Interview-Questions-and-Answers-Hub](https://github.com/KalyanKS-NLP/RAG-Interview-Questions-and-Answers-Hub) — Apache-2.0. 100+ RAG Q&A; single-topic depth.
- 💻 [alexeygrigorev/ai-engineering-field-guide](https://github.com/alexeygrigorev/ai-engineering-field-guide) — ~4k★. Closest scope; real Q4'25/Q1'26 practices, but thin Q&A and no MCP/context-eng.

---

<div align="center">

### Stop spraying. Get **matched**, get **prepped**, get **Landed**.

[![Get Started](https://img.shields.io/badge/Get%20Started%20Free-→-6C2BD9?style=for-the-badge)](https://landed.jobs)

<sub>Maintained by [Landed](https://landed.jobs) · No affiliation with the companies named. Content MIT-licensed; linked resources retain their own licenses.</sub>

</div>
