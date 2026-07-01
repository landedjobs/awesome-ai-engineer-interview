# Contributing

Thanks for helping build the best **AI Engineer interview-prep** repo on GitHub. This is a curated, high-trust resource — contributions are welcome, but they clear a quality bar. This file *is* that bar.

Maintained by [Landed](https://landed.jobs). No affiliation with any company named in this repo.

---

## What we accept

- **Questions** — real (reported) or representative AI Engineer interview questions, topic-grouped, difficulty-tagged. See [`questions/`](questions/).
- **Resources** — papers, docs, videos, courses, tools, repos — typed and annotated. See [`resources.md`](resources.md).
- **Content** — improvements to the one-concept-per-file explainers in [`content/`](content/).
- **Worked answers** — new or sharper system-design walkthroughs in [`answers/`](answers/).
- **Company loops** — updates to [`company/`](company/) as processes change (with a source).
- **Fixes** — broken links, wrong explanations, stale 2026 facts, mislabeled provenance.

## The non-negotiables

### 1. Provenance on every "real" question

Every interview question is labeled:

- ✅ **Reported** — actually seen in a loop, **with a checkable source** (URL, or a dated public candidate report). We keep the question verbatim / near-verbatim.
- 🔮 **Representative** — plausible and on-distribution, but not a verbatim reported question.

No source → it ships as 🔮 Representative. We never dress up a made-up question as reported.

### 2. Every link is typed and annotated

Never a bare link dump. Tag the type inline and add a one-line *why*:

> 📄 paper · 📘 docs · 📰 article · 🎬 video · 🧑‍🏫 course · 🛠️ tool · 💻 repo

For **repos and tools**, note the **license** and **approximate stars**. Cap ~8 links per topic — curate, don't accumulate.

### 3. License discipline

This repo ships **MIT** (see [`LICENSE`](LICENSE)).

| Source license | What you may do |
|---|---|
| MIT · Apache-2.0 · CC0 | **Adapt** text/code, with attribution (`License · Source · Author`). |
| CC-BY-4.0 (incl. autogen research artifacts) | **Link + commentary only.** Do not copy verbatim. |
| CC-BY-SA (e.g. applied-ml, awesome-mlops) | **Link + commentary only.** Never copy verbatim (share-alike would relicense us). |
| Unknown / proprietary | Link only, and only if it's publicly readable. |

When in doubt: **link, don't copy.**

### 4. Style

- **Answer-first.** Lead with the conclusion, then the mechanism, then the number.
- **Senior voice** — concrete, no fluff, no hype. Name the tradeoff.
- **Tables over prose** for anything comparable.
- **Callouts** — use `> [!WARNING]` for senior traps / misconceptions and `> [!TIP]` for aha-moments.
- **Dated 2026** — reasoning models, MCP, agentic eval, context engineering, GRPO/DPO, FP8, "eval is the new system design." Don't ship a fact you can't check.
- Match the look of the [umbrella repo](https://github.com/landedjobs/awesome-ai-native-jobs).

## How to contribute

1. Open an issue using a [template](.github/ISSUE_TEMPLATE/) (question / resource / bug), or go straight to a PR for small fixes.
2. Fork, branch, edit.
3. Keep internal links relative (`../content/02-rag.md`) and verify they resolve — CI runs [lychee](.github/workflows/links.yml) on every PR.
4. Open a PR; fill in the [PR checklist](.github/PULL_REQUEST_TEMPLATE.md).

## Question format (MCQ entries)

Multiple-choice entries keep the options **and** a one-line "why" for *each* option (right and wrong), inside a `<details>` block so readers can self-test first:

```markdown
### Q12. <the scenario / question>

> **Difficulty:** 🟡 Medium · **rag** · 🔮 Representative

<details><summary>Show answer & why each option</summary>

- ❌ Option A — *why it's wrong / tempting*
- ✅ Option B — *why it's right*
- ❌ Option C — *why it's wrong*

</details>
```

The per-option "why" is the gold. A question without it will be sent back.

## Recognition

Contributors are credited in release notes. Good PRs get referenced from the [Landed](https://landed.jobs) community.

Stop spraying. Get **matched**, get **prepped**, get **Landed**.
