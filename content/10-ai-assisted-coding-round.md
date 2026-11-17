# The AI-Assisted Coding Round

> Some companies now hand you an AI assistant in the coding round. They are not grading the assistant. They are grading whether you stay in charge of the code it writes.

Back to [README](../README.md) · [All questions](../questions/README.md)

*Last checked: 17 Nov 2026*

---

## What this tests in an interview

Whether you can **own code you did not type**. In an AI-assisted (or "AI-enabled") round you get a multi-file codebase or an open product problem plus a coding assistant. The grade comes from the same places as a normal coding round: problem solving, code quality, testing and communication. The new part is that you must also read, check and correct what the assistant produces, and explain every line you keep.

The senior signal: you read the code before you prompt, you state your plan before the assistant writes anything, you ask for small pieces you can review, you run tests after each piece, and you reject a wrong suggestion out loud. The junior tell is asking the assistant "how do I solve this?" and pasting the answer.

> [!WARNING]
> **"AI is allowed, so I don't need to code unaided any more."** At the companies below, the AI round sits *next to* unaided rounds, not instead of them. Several employers still ban AI in live interviews unless they say otherwise. Prepare both modes.

---

## What changed in 2025-2026

Before 2025, a coding round meant you, an editor and no assistant. Three things changed:

1. **Some employers redesigned a round around an AI assistant.** Canva and Meta are the clearest public examples. The problems got bigger and messier, because a plain LeetCode problem is solved by the assistant in seconds.
2. **Interview platforms shipped AI modes.** CodeSignal and Codility now sell assessments with a built-in assistant and record the candidate's prompts for the reviewer. Your prompts can become part of the evidence.
3. **Other employers wrote down that AI is still banned** unless they say otherwise. The policy is now per company and often per round.

This is evidence of changed formats at particular companies. It is not a measured market-wide adoption rate.

### Who runs it, and what they say

Evidence labels follow the repo's rules: **official** (the company's own page), **first-person** (a candidate's own account), **second-hand** (an article or prep site).

| Company | What the source says | Date | Evidence |
|---|---|---|---|
| **Canva** | Piloted an "AI-Assisted Coding" competency that replaces its Computer Science Fundamentals screen for backend and frontend roles. Backend, ML and frontend candidates are *expected* to use tools like Copilot, Cursor and Claude in technical interviews. Questions were made more complex and ambiguous. Candidates are told ahead of time. ([Canva Engineering](https://www.canva.dev/blog/engineering/yes-you-can-use-ai-in-our-interviews)) | 11 Jun 2025 | official |
| **Canva** (interviewer guide) | Grades three areas: engineering problem solving, technical depth and ownership, and AI collaboration. Lists three weak patterns: "AI Showcase", "Feature Marathon" and "Hands-Off". ([Canva Engineering](https://www.canva.dev/blog/engineering/ai-interview-success)) | 20 Oct 2025 | official |
| **Meta** | One onsite coding round became "AI-enabled" for SWE and EM roles up to E7 and M2; one classic coding round with no AI remains. 60 minutes in a three-panel CoderPad with a choice of models. The assistant answers in chat and cannot edit your files. Three phases on one problem: fix a bug, implement the core feature, optimise for larger inputs. ([Hello Interview](https://www.hellointerview.com/blog/meta-ai-enabled-coding)) | rollout reported from Oct 2025; page read 17 Nov 2026 | second-hand (prep site, based on candidate conversations and a quoted internal Meta message). Meta has not published its own description that we could find. |
| **Anthropic** | Live interviews and take-homes are done without AI "unless we indicate otherwise". AI is encouraged for preparing and for polishing an application you drafted yourself. ([Anthropic](https://www.anthropic.com/candidate-ai-guidance)) | page last updated 10 Jul 2025 | official |
| **Anthropic** (pilot) | A prep site reports that Anthropic is piloting an AI-assisted interview; outside that, AI use is prohibited. ([interviewing.io](https://interviewing.io/anthropic-interview-questions)) | published 7 Apr 2026, updated 13 May 2026 | second-hand. The official policy above wins until your recruiter says otherwise. |
| **Google DeepMind** | AI is fine for preparation, but "unless told otherwise" no AI tools in live interviews or interview tasks. ([Google DeepMind PDF](https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf)) | read 17 Nov 2026 | official |
| **Cursor (Anysphere)** | The CEO said first technical screens are done without AI other than autocomplete. The final step is a two-day on-site project with the team (see [take-homes and work trials](11-take-homes-and-work-trials.md)). ([Business Insider](https://www.businessinsider.com/cursor-interview-process-no-ai-on-site-project-coding-tool-2025-6)) | 12 Jun 2025 | second-hand (reporting on a podcast). Whether it still holds in late 2026 is not publicly confirmed. |

The tooling side, from the vendors' own pages:

| Platform | What it offers | Date | Evidence |
|---|---|---|---|
| **CodeSignal** | An in-IDE assistant ("Cosmo") with a full co-pilot mode and a lighter guided mode. Every AI-assisted session ships with a transcript of candidate-AI chat and a session replay. ([CodeSignal](https://codesignal.com/blog/introducing-ai-assisted-coding-assessments-interviews)) | 30 May 2025 | official (vendor) |
| **Codility** | A shared VS Code environment with Claude Code or Codex in the terminal. Replay shows each prompt, terminal command and code change in order; a human scores against a rubric. ([Codility](https://www.codility.com/engineering-skills/agentic-coding-interviews)) | read 17 Nov 2026 | official (vendor) |

A vendor offering a feature does not tell you which employer uses it, or with which settings. Ask.

> [!TIP]
> If a round runs on CodeSignal or Codility with AI on, assume the reviewer can read every prompt you sent. Write prompts you would be happy to have read aloud: specific, scoped, with your plan stated first.

See also the company pages: [Meta](../company/meta.md), [Anthropic](../company/anthropic.md), [Google DeepMind](../company/google-deepmind.md), [Cursor](../company/cursor.md).

---

## What interviewers grade

Put Canva's official rubric next to the criteria Hello Interview reports for Meta and they line up. Neither is about prompt tricks.

| What is graded | What it looks like in the room | Where it comes from |
|---|---|---|
| **Problem solving before prompting** | You clarify requirements, name the algorithm or design, and state a plan before the assistant writes code | Canva's "planning phase"; Meta's "Problem Solving" (second-hand) |
| **Ownership of the code** | You can explain any line on request; you would put your name on it as a pull request | Canva's "technical depth & ownership"; Meta's "Code Quality" (second-hand) |
| **Reading and correcting AI output** | You catch a wrong suggestion, say why it is wrong, and fix or rewrite it | Canva asks whether candidates can "identify and fix issues in AI-generated code" |
| **Testing and verification** | Prompt, review, run, confirm, move on. You run tests often and add edge cases | Meta's "Verification" (second-hand) |
| **Explaining choices** | You narrate what you are doing and why, and end with what you would improve | Canva's "discussion phase"; Meta's "Communication" (second-hand) |
| **Using AI for the right tasks** | Boilerplate, known data structures, unfamiliar APIs: yes. The core insight: yours | Canva's "AI collaboration effectiveness" |

> [!WARNING]
> **"Finishing every phase is the goal."** Canva's interviewer guide names the "Feature Marathon" (rushing features by accepting AI output fast) as a weak pattern. Hello Interview reports Meta candidates who did not finish the last phase and still got offers. Clean, tested, explained progress beats a long list of half-checked features.

### The assistant may be weaker than you expect

Hello Interview reports several Meta candidates found the interview assistant much less helpful than in the practice environment: wrong code on easy problems, or describing code without pointing out bugs. Its theory is that the system prompt limits the assistant; that is unconfirmed. The lesson holds either way: **if you cannot solve the core problem yourself, the assistant will not rescue you.** Plan for a weak assistant and treat a strong one as a bonus.

---

## A good session, step by step

```
1. Orient     read the task, the files, the data models and the tests (5 min)
2. Baseline   run the tests; see what passes and what fails
3. Plan       say the approach out loud: algorithm, data structures, edge cases
4. Delegate   ask for one small, named piece ("a trie with insert and prefix search")
5. Review     read the diff; check types, edge cases, off-by-ones, complexity
6. Verify     run the tests; add one for the case you are worried about
7. Repeat     next piece; keep talking
8. Wrap up    what you built, why, what is weak, what you'd do with more time
```

Prompt quality matters, but only as a means. A strong prompt states context, the exact interface you want and the constraint that matters: "Implement `find_path(grid, start, end)` using BFS with a visited set. Gates `>` and `<` block movement in one direction. Return the path as a list of coordinates or `None`." A weak prompt is "fix this" or "solve this problem".

> [!TIP]
> When the interviewer asks you a question, answer it yourself first, then check with the assistant if you need to. Canva's guide asks for exactly this: tell us what you think before getting help from AI.

---

## Practice plan: AI on and AI off

Practice both modes every week. The unaided reps keep the fundamentals sharp; the assisted reps train review and narration. Alisa Liu's account of her 2026 research-role search (she then joined OpenAI) advises practising with AI assistance completely off, because you will otherwise underestimate how much you rely on it ([Alisa Liu](https://alisawuffles.github.io/blog/job-search); first-person, research roles, not the AI-engineer role).

| Week | AI off (timed, plain editor) | AI on (assistant allowed) | Self-check |
|---|---|---|---|
| 1 | 4 medium problems: BFS/DFS, heaps, intervals, hashing | Clone a small unfamiliar repo; ask the assistant to explain it, then check its explanation against the code | Could you have found the entry point without it? |
| 2 | 2 medium + 1 hard: backtracking, tries, greedy | Fix a planted bug in a 4-5 file project, then add a feature with tests | Did you read before you prompted? |
| 3 | Re-solve week 1-2 misses from memory | 60-minute, three-phase drill: fix, implement, optimise for a large input | Did you finish phase 2 with tests green? |
| 4 | One full mock unaided, out loud | One full mock with a weak or restricted assistant (small model, or chat only, no edits) | Can you reject a wrong suggestion and still finish? |

Keep a short error log after each session: missed requirement, wrong algorithm, accepted bad AI code, untested edge case, silent stretch. Pick the next drill from the log.

---

## Common mistakes

| Mistake | Why it hurts | Instead |
|---|---|---|
| Prompting before reading the code | You miss the data model and the bug hides in plain sight | Spend the first five minutes reading files and tests |
| Asking the assistant for the whole solution | Interviewers watch for candidates who prompt their way out; Canva calls it "Hands-Off" | Name the approach yourself; delegate one piece at a time |
| Pasting without reading | You cannot explain it, and AI bugs (wrong order of checks, off-by-one, type casts) slip in | Read every diff; explain it in one sentence before running it |
| Going quiet while working with the assistant | Half the signal is your reasoning; Meta candidates report the dual conversation is the hardest part | Narrate: "I'm asking for X because Y; now checking Z" |
| Showing off prompting | Canva's "AI Showcase" pattern: tool tricks, not engineering | Keep the task at the centre |
| Never practising unaided | A weak interview assistant leaves you stranded | Weekly AI-off sets (see plan) |
| Assuming AI is allowed because the company sells AI | Anthropic and Google DeepMind ban it in live rounds unless told otherwise; Cursor screens without it | Ask the recruiter for the rule per round, in writing |
| Skipping environment setup (bring-your-own-tool formats) | Canva's guide says not to waste interview time setting up a project | Have your editor, assistant and an empty project ready |

---

## 🔮 Representative practice prompts

These are written for practice. None is a reported question from a named company. P2 and P4 borrow problem types (grid paths with gates, maximising unique characters) that [Hello Interview](https://www.hellointerview.com/blog/meta-ai-enabled-coding) lists for Meta; the prompts and twists are ours.

### P1. Fix, then extend, a rate-limited client

> **Difficulty:** 🟡 Medium · **ai-assisted coding** · 🔮 Representative

You get a 4-file Python package: an HTTP client, a token-bucket rate limiter, a retry helper and tests. Two tests fail. Fix them, then add per-endpoint limits. AI on.

<details><summary>Answer outline</summary>

- Read `limiter.py` and the tests before prompting. Run the suite to see which fail.
- Likely bug class: refill computed with integer division, or time measured in ms vs s. Fix it yourself; say why.
- Plan the feature: a dict of buckets keyed by endpoint, a default bucket, thread safety if the client is shared.
- Ask the assistant for the dict-of-buckets change only. Review it for a race on bucket creation (use a lock or `setdefault` under a lock).
- Add tests: two endpoints do not share a budget; unknown endpoint falls back to default; refill after a sleep (use a fake clock, not `time.sleep`).
- Wrap up: retries must not double-spend tokens; mention idempotency for retried POSTs.

</details>

### P2. The assistant is confidently wrong

> **Difficulty:** 🟡 Medium · **ai-assisted coding** · 🔮 Representative

You ask for BFS shortest path on a grid with one-way gates. The assistant's code marks cells visited when they are popped, not when they are pushed, and ignores gate direction on the way back. Tests pass on the small sample. What do you do?

<details><summary>Answer outline</summary>

- Say out loud that passing small tests does not prove correctness. Name both issues.
- Marking on pop lets the same cell enter the queue many times: correct result, but blow-up on large grids. Mark on push.
- Gate direction: the move from A to B must check the gate on the edge you are crossing, in that direction. Write the check yourself.
- Add a test with a gate that blocks the direct route, and a large open grid with a time budget.
- Point to the general lesson: you own the code, so you review it as you would a colleague's PR.

</details>

### P3. Open product problem, Canva style

> **Difficulty:** 🔴 Hard · **ai-assisted coding** · 🔮 Representative

"Build a booking service for meeting rooms." 60 minutes, your own editor and assistant.

<details><summary>Answer outline</summary>

- First 8-10 minutes: questions. Recurring bookings? Time zones? Overlap rules (is end exclusive)? Cancellation? Concurrent requests?
- Scope out loud: in-memory store, `book`, `cancel`, `list_free(room, day)`; recurring bookings as a stretch.
- Design the core yourself: sorted intervals per room, half-open `[start, end)`, overlap check, a lock per room.
- Delegate: data classes, a CLI or HTTP shell, test scaffolding. Review each piece.
- Tests: back-to-back bookings allowed; overlapping rejected; two threads booking the same slot, one wins.
- Close: what is not production-ready (persistence, auth, time zones), and how you would add each.

</details>

### P4. Optimise for the bigger input

> **Difficulty:** 🔴 Hard · **ai-assisted coding** · 🔮 Representative

Your phase-2 solution picks words to maximise unique characters by trying all subsets. Phase 3 gives two data files: many short words, and few long words. Both time out.

<details><summary>Answer outline</summary>

- Look at the data first: the two files stress different things, so one fix may not suit both.
- Represent each word as a 26-bit mask; drop words with repeated letters; dedupe masks.
- Backtracking with pruning (stop when remaining words cannot beat the best) handles the short-word file.
- For few long words, plain search over masks is already small; the cost was string handling, not search.
- Ask the assistant to write a benchmark harness for both files; you choose and explain the approach.
- If time runs out, state the full plan anyway; credit comes from showing you understand it.

</details>

---

## Interview angles

- **"How do you decide what to hand to the assistant?"** → Known, checkable pieces (data structures, boilerplate, API calls you will read) go to the assistant. The approach, the tricky invariant and the final review stay with you.
- **"The AI's code passes the tests. Are you done?"** → No. Tests are a sample. Read the diff for complexity, edge cases and hidden assumptions, then add the test you are most worried about.
- **"You disagree with the assistant. Now what?"** → Say why it is wrong, write the fix or a sharper prompt, and add a test that would have caught it.
- **"Would you use AI here if it were allowed?"** → Depends on the round's purpose. Follow the stated rule; if unclear, ask before the clock starts.

---

## 📚 Resources

- 📰 [Canva: Yes, you can use AI in our interviews](https://www.canva.dev/blog/engineering/yes-you-can-use-ai-in-our-interviews): the clearest official account of why a company redesigned a round around AI, and what it grades.
- 📰 [Canva: AI interview success, an interviewer's guide](https://www.canva.dev/blog/engineering/ai-interview-success): the three graded areas and the three weak patterns, from an interviewer.
- 📰 [Hello Interview: Meta's AI-enabled coding interview](https://www.hellointerview.com/blog/meta-ai-enabled-coding): second-hand but detailed: environment, three phases, reported problem types, the weak-assistant warning.
- 📘 [Anthropic: guidance on candidates' AI usage](https://www.anthropic.com/candidate-ai-guidance): an official example of a "no AI unless we say so" policy, with allowed and not-allowed uses.
- 📘 [Google DeepMind: interviewing guide (PDF)](https://storage.googleapis.com/deepmind-media/DeepMind.com/Assets/Docs/interviewing-at-google-deepmind.pdf): official process notes, including the AI-use answer in its FAQ.
- 🛠️ [CodeSignal: AI-assisted assessments](https://codesignal.com/blog/introducing-ai-assisted-coding-assessments-interviews) and [Codility: agentic coding interviews](https://www.codility.com/engineering-skills/agentic-coding-interviews): what the platforms record, so you know your prompts may be reviewed.
- 📰 [Alisa Liu: notes on the industry job search](https://alisawuffles.github.io/blog/job-search): a named 2026 account; read it for the "practise with AI off" advice.

---

Back to [README](../README.md) · [All questions](../questions/README.md) · [Coding questions](../questions/coding.md) · Next: [Take-homes and work trials](11-take-homes-and-work-trials.md)
