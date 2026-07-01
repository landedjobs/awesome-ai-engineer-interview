<div align="center">

<img src="https://static.b100x.ai/email/landed-wordmark.png" width="300" alt="Landed" />

# Company-by-Company AI Engineer Interviews (2026)

**Real loops, verbatim questions, and what each lab is actually testing — one file per company.**

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-6C2BD9.svg)](../LICENSE)
[![Updated](https://img.shields.io/badge/updated-2026--07-00A86B)](#whats-new)
[![Companies](https://img.shields.io/badge/companies-12-6C2BD9)](#the-companies)
[![Visit Landed](https://img.shields.io/badge/Visit-Landed-6C2BD9?logo=rocket&logoColor=white)](https://landed.jobs)

*Maintained by [Landed](https://landed.jobs) — daily AI-native job matches, agent help with every application, and mock-interview prep.*

</div>

---

Every AI lab claims it hires "the best AI engineers." In practice they hire **three different jobs**, and the loop tells you which one. Read the philosophy first — it predicts the round that decides your offer.

> ⭐ **Star this repo** if a company page saves you a round. New candidate reports welcome via PR.

← Back to the [main repo README](../README.md) · Jump to a [company](#the-companies).

## The three hiring philosophies

Drawn from the cross-lab synthesis in our research corpus. Almost every company below sits primarily in one bucket — and the **signature round** is the tell.

| Philosophy | What you ship | Loop centers on | Labs |
|---|---|---|---|
| 🧠 **Write the model** | The intelligence itself — models, training, alignment | Implement-from-scratch primitives (attention, PPO, VAE), mechanistic understanding, research take-homes | OpenAI · Anthropic · Google DeepMind · Meta AI · Mistral · Cohere |
| 🏗️ **Ship the platform** | The infra models run on | Large-scale serving, distributed training, system design; hardware-software co-design | xAI · Databricks · Nvidia · (Meta GenAI infra) |
| 🚀 **Build the product** | The app users touch | Applied system craft, end-to-end shipping, product sense | Cursor · Perplexity · (Harvey · Glean) |

**The tensions worth knowing:**
- **Cursor bans AI in interviews. Meta added a "Coding with AI" round.** Prepare both a *no-AI* and a *use-AI-well* answer template.
- **Anthropic runs the longest single process (5–10h)** yet openly publishes its mechanistic-interpretability take-home.
- **Mistral's 22% pass rate and OpenAI's 2–8 week loop** signal similar selectivity via very different taxonomies.
- **Nvidia's hardware-software co-design shift** is the only clear philosophy *change* reported across 2025–2026.

## Canonical comparison

| Company | Loop length | Signature round | Pass-rate / selectivity | Philosophy | Guide |
|---|---|---|---|---|---|
| **OpenAI** | 4–6h final · 2–8 wk E2E | HM "first 90 days" deep dive + "Design ChatGPT" | Very selective (2–8 wk) | 🧠 Write the model | [openai.md](openai.md) |
| **Anthropic** | 5–10h · 1–2 days | Mechanistic-interp take-home + dedicated culture round | ~10h, high bar | 🧠 Write the model | [anthropic.md](anthropic.md) |
| **Google DeepMind** | 4–6h · 6–8 wk | Research presentation / critique a DeepMind paper + new code-review round | Selective, decentralized | 🧠 Write the model | [google-deepmind.md](google-deepmind.md) |
| **Meta (AI/GenAI)** | 5 coding + 2 research · 4–8 wk | New **"Coding with AI"** round | Committee-gated | 🧠 Write the model | [meta.md](meta.md) |
| **Scale AI** | 7 rounds · ~14 days median | Take-home ML eval pipeline + FDE messy-data presentation | 7-interview gauntlet | 🏗️ / 🚀 hybrid | [scale.md](scale.md) |
| **xAI (Grok)** | 4–5 rounds · 1–3 wk | 4-question / 70-min CodeSignal + serving system design | Fast, high-throughput | 🏗️ Ship the platform | [xai.md](xai.md) |
| **Databricks** | 5–6 stages · 4–7 wk | Spark-flavored ML system design | "Extremely selective" | 🏗️ Ship the platform | [databricks.md](databricks.md) |
| **Perplexity** | 4–5 rounds · ~11 days avg | In-memory file-system coding + **founder round** | Fast loop, founder-gated | 🚀 Build the product | [perplexity.md](perplexity.md) |
| **Cohere** | 7 rounds · 4–6 wk | 3-hour language-modelling + coding technical assessment | Long, IC3–IC6 | 🧠 Write the model | [cohere.md](cohere.md) |
| **Mistral** | 5+ rounds · 3–5 wk | Founder/leader final | **22% pass · 78% rejected** | 🧠 Write the model | [mistral.md](mistral.md) |
| **Cursor / Anysphere** | 8h paid onsite + 2-day final | **No AI allowed** · 8-hour paid real project | Craft-gated | 🚀 Build the product | [cursor.md](cursor.md) |
| **Nvidia** | 5 onsite · ~5h | Hardware-software co-design + CUDA-level | Co-design bar | 🏗️ Ship the platform | [nvidia.md](nvidia.md) |

## The companies

- [**OpenAI**](openai.md) — 🧠 four-to-six-hour loop, 48h take-home, HM "what would you build in 90 days."
- [**Anthropic**](anthropic.md) — 🧠 longest single loop, open-sourced mech-interp take-home, dedicated culture round.
- [**Google DeepMind**](google-deepmind.md) — 🧠 research presentation + newly added calibration code-review round.
- [**Meta (AI / GenAI)**](meta.md) — 🧠 five coding + two research, plus the new "Coding with AI" round.
- [**Scale AI**](scale.md) — 🏗️🚀 seven-round loop, ML-eval take-home, forward-deployed messy-data presentation.
- [**xAI (Grok)**](xai.md) — 🏗️ recruiter → 4-question CodeSignal → three onsite cores, fast.
- [**Databricks**](databricks.md) — 🏗️ recruiter → coding → take-home → Spark-scale ML system design.
- [**Perplexity**](perplexity.md) — 🚀 in-memory file-system challenge, AI take-home, founder round, ~11 days.
- [**Cohere**](cohere.md) — 🧠 seven rounds with a 3-hour language-modelling technical assessment.
- [**Mistral**](mistral.md) — 🧠 five-plus rounds, 22% pass rate, founder final.
- [**Cursor / Anysphere**](cursor.md) — 🚀 no AI in interviews, 8-hour paid onsite, 2-day in-office final.
- [**Nvidia**](nvidia.md) — 🏗️ roofline, distributed-training memory, CUDA kernels, hardware-software co-design.

## Provenance legend

Every question row in a company file is labeled. We never invent questions; representative follow-ups are separated and marked.

| Emoji | Meaning |
|---|---|
| ✅ **Reported** | A real question from a public candidate report or write-up, carried near-verbatim, with its bracketed source id. Where the source file provided a URL we link it; otherwise we cite `candidate report [id]`. |
| 🔮 **Representative** | *Not* a reported question — a follow-up you should expect given the round. Clearly separated in its own subsection. |
| 🧠 / 🏗️ / 🚀 | Hiring philosophy: write the model / ship the platform / build the product. |

> [!NOTE]
> Bracketed ids (e.g. `[95]`, `[4]`, `[33]`) are the source markers from our research corpus (two deep-research runs completed 2026-07-01). Some markers are candidate posts on Blind/Reddit/Taro/linkjob.ai without a stable canonical URL — those are cited as `candidate report [id]` rather than linked. This is deliberate: we label the strength of the evidence rather than dress it up.

## What's new

- **2026-07** — Initial per-company section shipped: 12 companies, three-philosophy framing, canonical comparison table, provenance-labeled question tables.

---

<div align="center">

### Stop spraying. Get **matched**, get **prepped**, get **Landed**.

[![Get Started](https://img.shields.io/badge/Get%20Started%20Free-→-6C2BD9?style=for-the-badge)](https://landed.jobs)

<sub>Maintained by [Landed](https://landed.jobs) · No affiliation with the companies named. Content MIT-licensed.</sub>

</div>
