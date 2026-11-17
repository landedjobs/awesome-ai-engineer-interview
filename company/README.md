<div align="center">

<img src="https://static.b100x.ai/email/landed-wordmark.png" width="300" alt="Landed" />

# Company-by-Company AI Engineer Interviews (2026)

**Real loops, verbatim questions, and what each lab is actually testing — one file per company.**

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-6C2BD9.svg)](../LICENSE)
[![Updated](https://img.shields.io/badge/updated-2026--11-00A86B)](#whats-new)
[![Companies](https://img.shields.io/badge/companies-17-6C2BD9)](#the-companies)
[![Visit Landed](https://img.shields.io/badge/Visit-Landed-6C2BD9?logo=rocket&logoColor=white)](https://landed.jobs)

*Maintained by [Landed](https://landed.jobs) — daily AI-native job matches, agent help with every application, and mock-interview prep.*

</div>

---

Every AI lab claims it hires "the best AI engineers." In practice they hire **three different jobs**, and the loop tells you which one. Read the philosophy first — it predicts the round that decides your offer.

> ⭐ **Star this repo** if a company page saves you a round. New candidate reports welcome via PR.

← Back to the [main repo README](../README.md) · Jump to a [company](#the-companies).

## The three hiring philosophies

A rough way to read a loop: what the company ships tends to predict the round that decides the offer. Use it as a lens, not a rule; each page has the evidence.

| Philosophy | What you ship | Loops tend to center on | Companies |
|---|---|---|---|
| 🧠 **Write the model** | Models, training, alignment | ML fundamentals, debugging model code, research depth | OpenAI · Anthropic · Google DeepMind · Meta AI · Mistral · Cohere |
| 🏗️ **Ship the platform** | The infrastructure models run on | Serving, distributed systems, system design | xAI · Databricks · Nvidia |
| 🚀 **Build the product** | The app users touch | Applied coding on real code, take-homes and work trials, product sense | Cursor · Perplexity · Sierra · Harvey · Glean · Notion · Replit · Scale AI (FDE) |

**The tensions worth knowing (each one sourced on its page):**
- **AI rules differ by company and by round.** Meta runs an AI-enabled coding round next to an unaided one (second-hand), Notion's AI Applications Engineer loop has an AI-enabled backend round (official), Anthropic asks for no AI in live interviews or take-homes unless told otherwise (official), and Cursor's first technical screens allow no AI beyond autocomplete (June 2025 reporting). Prepare both: see [the AI-assisted coding round](../content/10-ai-assisted-coding-round.md).
- **Product companies test with real work.** Sierra's agent take-home plus a presentation, Cursor's two-day in-office project and Replit's live-or-take-home exercise all grade how you build and defend something. See [take-homes and work trials](../content/11-take-homes-and-work-trials.md).
- **Official timelines are long.** Google DeepMind says 4-10 weeks and Databricks two to three months; OpenAI's guide gives about a week for application review and a decision within about a week of finals.
- **Many popular "facts" about these loops have no public source.** We removed or softened them in the Nov 2026 refresh (for example a "22% pass rate" for Mistral). If a page says *not publicly confirmed*, we looked and could not find a source we could open.

## Canonical comparison

| Company | Loop (best public evidence) | Signature round | Evidence | Philosophy | Guide |
|---|---|---|---|---|---|
| **OpenAI** | Review (~1 wk) → intro calls → skills assessment → 4-6h finals over 1-2 days | Skills assessment that varies by team | Official guide + first-person | 🧠 | [openai.md](openai.md) |
| **Anthropic** | Recruiter → 60-90 min coding challenge → 4-5h onsite (five 1h sessions incl. values) | Values round; published performance take-home | Official AI policy + second-hand | 🧠 | [anthropic.md](anthropic.md) |
| **Google DeepMind** | Initial → skills → final interviews → offer; 4-10 weeks | Role-specific; rounds not publicly confirmed | Official guide | 🧠 | [google-deepmind.md](google-deepmind.md) |
| **Meta (AI / GenAI)** | Onsite with one AI-enabled and one unaided coding round (since Oct 2025) | AI-enabled coding | Second-hand + first-person | 🧠 | [meta.md](meta.md) |
| **Mistral** | First conversation → 2-5 technical exercises → values discussion → references | Exercises that reflect real work | Official only | 🧠 | [mistral.md](mistral.md) |
| **Cohere** | Recruiter → possible take-home → hiring manager → team (one SWE: 4 rounds, 5 weeks) | Take-home (streaming server, SWE account) | Official + first-person | 🧠 | [cohere.md](cohere.md) |
| **xAI (Grok)** | Engineer call → live phone screen → 2-3 onsite rounds | Applied coding + serving design | Second-hand only | 🏗️ | [xai.md](xai.md) |
| **Databricks** | Screen → onsite of 4-6 interviews; 2-3 months | Applied AI and customer case (FDE) | Official + second-hand | 🏗️ | [databricks.md](databricks.md) |
| **Nvidia** | Recruiter → 1h phone screen → 4-5h onsite; 6-8 weeks | LLM-heavy PyTorch screen; C++ for CUDA roles | Second-hand | 🏗️ | [nvidia.md](nvidia.md) |
| **Cursor / Anysphere** | Screens (no AI beyond autocomplete) → two-day in-office project with a demo | Two-day work trial | Second-hand | 🚀 | [cursor.md](cursor.md) |
| **Perplexity** | Reply within 2 weeks → technical assessment → finals → answer within a week | Founder or VP round for senior roles (guide) | Official + second-hand + first-person | 🚀 | [perplexity.md](perplexity.md) |
| **Scale AI** | FDE: assessment → screen → ~5-round onsite; about two months | Debugging and messy-data rounds (FDE) | Second-hand + first-person, mixed roles | 🏗️ / 🚀 | [scale.md](scale.md) |
| **Sierra** 🆕 | Recruiter → API coding → agent debugging → agent take-home + 60-min presentation → HM | Build-debug-defend | First-person (one account) | 🚀 | [sierra.md](sierra.md) |
| **Harvey** 🆕 | No Agents-role loop public; adjacent roles: recruiter → coding → architecture panel | Architecture panel (adjacent role) | Official posting + first-person, adjacent roles | 🚀 | [harvey.md](harvey.md) |
| **Glean** 🆕 | New grad: screen → recruiter → deep dive → 2h design assignment + algorithms | AI-focused exercise (AI Engineer posting) | First-person (new grad) + official posting | 🚀 | [glean.md](glean.md) |
| **Notion** 🆕 | Recruiter → 60-min screen → 4-interview onsite incl. AI-enabled backend coding | AI-enabled backend coding | Official guides | 🚀 | [notion.md](notion.md) |
| **Replit** 🆕 | Recruiter → HM → live or take-home exercise reviewed together → panel | Exercise you review with the team | Official (all roles) | 🚀 | [replit.md](replit.md) |

## The companies

- [**OpenAI**](openai.md): 🧠 official process from OpenAI's interview guide, plus first-person research-engineer questions.
- [**Anthropic**](anthropic.md): 🧠 official AI-use policy, a published performance take-home, five-session onsite with a values round.
- [**Google DeepMind**](google-deepmind.md): 🧠 four-stage official process, 4-10 weeks, role-specific prep.
- [**Meta (AI / GenAI)**](meta.md): 🧠 the AI-enabled coding round next to an unaided one.
- [**Mistral**](mistral.md): 🧠 two to five technical exercises and a values discussion, from Mistral's own page.
- [**Cohere**](cohere.md): 🧠 official stages plus one senior SWE's four-round account.
- [**xAI (Grok)**](xai.md): 🏗️ second-hand guides only; where they disagree, the page says so.
- [**Databricks**](databricks.md): 🏗️ official onsite of four to six interviews over two to three months.
- [**Nvidia**](nvidia.md): 🏗️ LLM-heavy phone screens, C++ for CUDA-heavy roles.
- [**Cursor / Anysphere**](cursor.md): 🚀 no AI in first screens, then a two-day project in the office.
- [**Perplexity**](perplexity.md): 🚀 fast official timelines, a founder or VP round for senior roles.
- [**Scale AI**](scale.md): 🏗️🚀 forward-deployed and new-grad loops, labelled by role.
- [**Sierra**](sierra.md) 🆕: 🚀 agent debugging, an agent take-home and a presentation.
- [**Harvey**](harvey.md) 🆕: 🚀 adjacent-role evidence plus the official Agents posting.
- [**Glean**](glean.md) 🆕: 🚀 a new-grad loop and the AI Engineer, Data and Evals posting.
- [**Notion**](notion.md) 🆕: 🚀 official AI Applications Engineer guide with an AI-enabled round.
- [**Replit**](replit.md) 🆕: 🚀 official process with a live or take-home exercise.

## Provenance legend

Every question row in a company file is labeled. We never invent questions; representative follow-ups are separated and marked.

| Emoji | Meaning |
|---|---|
| ✅ **Reported** | A real question from a public candidate report or write-up, linked inline with its evidence type and date. |
| 🔮 **Representative** | *Not* a reported question — a follow-up you should expect given the round. Clearly separated in its own subsection. |
| 🧠 / 🏗️ / 🚀 | Hiring philosophy: write the model / ship the platform / build the product. |

> [!NOTE]
> Pages refreshed since Nov 2026 cite each source as an inline link with its evidence type (official, first-person, second-hand) and date, and carry a "Last checked" date. Where we could not find a public source we can open, the claim is softened or marked not publicly confirmed, and the question is labelled 🔮 Representative. We label the strength of the evidence rather than dress it up.

## What's new

- **2026-11**: Five new companies (Sierra, Harvey, Glean, Notion, Replit). All 12 existing pages re-sourced: every claim now has an inline link, an evidence type and a date; claims without a public source were softened or relabelled 🔮.
- **2026-07**: Initial per-company section shipped: 12 companies, three-philosophy framing, canonical comparison table, provenance-labeled question tables.

---

<div align="center">

### Stop spraying. Get **matched**, get **prepped**, get **Landed**.

[![Get Started](https://img.shields.io/badge/Get%20Started%20Free-→-6C2BD9?style=for-the-badge)](https://landed.jobs)

<sub>Maintained by [Landed](https://landed.jobs) · No affiliation with the companies named. Content MIT-licensed.</sub>

</div>
