# Agents — Tool Design, Multi-Agent, MCP & Long-Running State

<sub>⬅ [Back to the Question Bank index](./README.md) · 📖 Deep dive: [`content/agents.md`](../content/03-agents-and-tool-use.md) · Part of **[awesome-ai-engineer-interview](../README.md)** — maintained by [Landed](https://landed.jobs)</sub>

> **29 questions** (20 self-test checkpoints + 9 reported). The agent round is where 2026 loops separate builders from prompt-tinkerers. Core theses: **the tool surface is the model's real API** (optimize tools before prompts), **errors compound multiplicatively over long trajectories**, **de-escalate to the simplest pattern that meets the bar** (workflow before autonomous agent), **MCP is untrusted by default**, and **the window is a cache — durable state lives on disk**.

---

### How to use this file

Commit to an answer, then open the fold. For agent-security questions see also [evals.md](./evals.md) (the lethal trifecta and injection containment live there since they're graded as security evals).

**Difficulty legend** · 🟢 Easy · 🟡 Medium · 🔴 Hard (scale, security, cost, long-horizon)
**Provenance legend** · 🔮 Representative (Landed checkpoints) · ✅ Reported (real interview, source linked)

---

## Section A — Tool design & the agent-computer interface `[agents-evals-llmops] ag1`

### Q1. Your agent keeps picking the wrong tool and passing odd arguments. Anthropic's guidance says the highest-leverage fix is usually to…

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag1` · 🔮 Representative

- Rewrite the system prompt to be more forceful about tool choice
- ✅ Improve the tools themselves — clearer descriptions, poka-yoke schemas (enums, absolute paths), examples, and explicit boundaries
- Add more tools so the agent always has an exact match

<details><summary>Show answer & why each option</summary>

- **Rewrite the prompt** — Prompt tweaks help least here; the tool surface is the model's real API and is where confusion originates.
- ✅ **Improve the tools (ACI)** — Anthropic: "optimizing tools is often more impactful than optimizing the overall prompt." A well-designed ACI removes whole classes of wrong calls.
- **Add more tools** — More tools mean noisier decisions and more tokens — it usually makes selection worse.
</details>

### Q2. A tool hits a transient upstream 503. What's the right thing for it to return so the loop behaves well?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag1` · 🔮 Representative

- Raise a bare exception and let the loop crash
- ✅ A structured error tagging it `TRANSIENT` with a `retry_after`, so the agent can retry within budget (vs give up / escalate)
- A plain string "something went wrong"

<details><summary>Show answer & why each option</summary>

- **Bare exception** — The agent can't reason about an unstructured crash; it can't tell transient from permanent.
- ✅ **Structured error with `TRANSIENT` + `retry_after`** — Structured errors let the agent choose retry / give-up / escalate instead of confusing a logical failure with a transient one.
- **Plain string** — Ambiguous — the agent often retries non-retryable failures or gives up on transient ones.
</details>

### Q3. An interviewer asks why your agent's monthly bill grew much faster than the number of steps per task. Best answer?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag1` · 🔮 Representative

- ✅ The full transcript is re-sent every step, so input tokens grow super-linearly with trajectory length; we cut steps, trimmed observations, and cached the stable prefix
- Output tokens are billed at 4× input, so longer answers dominated the bill
- The provider raised prices mid-month

<details><summary>Show answer & why each option</summary>

- ✅ **Transcript re-sent every step** — Cost is roughly quadratic in trajectory length — the dominant agent cost driver. Caching the stable system+tools prefix is the structural fix.
- **Output at 4× input** — True for single calls, but for agents the re-sent input transcript — not final answer length — is what compounds.
- **Provider raised prices** — A pricing change is uniform; it doesn't explain cost growing with trajectory length.
</details>

### Q4. Your agent retrieves a 50k-token document early in a task, and three steps later it seems to forget its original goal. Most likely cause and fix?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag1` · 🔮 Representative

- The model's reasoning degraded — switch to a larger model
- The context window filled up — raise `max_tokens`
- ✅ Context pollution: the un-trimmed blob buried the goal (Lost-in-the-Middle / Context Rot) — compress the observation to a handle/summary and re-inject the goal each turn

<details><summary>Show answer & why each option</summary>

- **Larger model** — Model strength isn't the issue; the goal is now buried in a long, un-curated context.
- **Raise `max_tokens`** — `max_tokens` caps output, not input context, and a bigger window still rots; this misreads the mechanism.
- ✅ **Context pollution → compress + re-inject** — Every observation re-enters context; a giant un-trimmed return degrades the next decision. Context engineering (compress, prune, externalise, re-inject) is the fix.
</details>

### Q5. You add Anthropic's "think" tool to a policy-heavy refund agent but see almost no improvement. Most likely reason?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag1` · 🔮 Representative

- The think tool only helps on coding tasks, not policy tasks
- ✅ You added the no-op slot but didn't tell the model WHAT to reason about (the relevant policy), so the reasoning step has nothing to chew on
- Temperature was too low for the think step to vary

<details><summary>Show answer & why each option</summary>

- **Only coding tasks** — The largest reported lift (pass^1 0.370→0.570) was on tau-bench Airline — a policy-heavy task.
- ✅ **No-op slot with no direction** — The think-tool win is contingent on a prompt that directs the reasoning at the policy; a bare "think more" slot does much less.
- **Temperature** — Doesn't govern whether a mid-trajectory reasoning slot is grounded in policy; that's a prompting issue.
</details>

---

## Section B — Single vs multi-agent & control flow `[agents-evals-llmops] ag2`

### Q6. You're building a coding agent where several agents would edit the same repository in parallel. Single-agent or multi-agent?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag2` · 🔮 Representative

- Multi-agent — parallelism will make it faster
- ✅ Single-agent — the writes are shared and must be self-consistent
- Multi-agent, but have them message constantly to stay in sync

<details><summary>Show answer & why each option</summary>

- **Multi-agent (faster)** — Parallel writers make conflicting implicit choices (style, naming, error handling) — Cognition's exact anti-pattern for shared writes.
- ✅ **Single-agent** — When writes are shared, one serial thread preserves consistency; multi-agent helps for parallel *reads* with one synthesising writer.
- **Multi-agent + constant chatter** — Inter-agent chatter is itself a documented failure mode and doesn't reconstruct the implicit decisions a single thread keeps coherent.
</details>

### Q7. When does adding Reflexion-style self-critique actually improve an agent?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag2` · 🔮 Representative

- Always — letting the model critique itself is free quality
- ✅ When there's a real verifier (tests, schema, calibrated judge) and a bounded retry budget
- Only on creative writing tasks

<details><summary>Show answer & why each option</summary>

- **Always free** — A model critiquing itself with no external signal often just confirms its own error — "the self-critique paradox."
- ✅ **Real verifier + bounded retries** — Reflexion converges when the critique is grounded in a real scorer and capped; ungrounded or unbounded, it spirals.
- **Only creative writing** — Those lack a verifier — exactly where self-critique is least reliable.
</details>

### Q8. A PM asks you to spec "an agent" that takes an incoming support email and either answers it, issues a refund, or escalates to a human. What's the right first architecture?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag2` · 🔮 Representative

- ✅ A routing workflow: a cheap classifier sends the email to one of three specialised handlers; add an autonomous loop only if the task space turns out open-ended
- A fully autonomous agent with a large tool set so it can decide everything itself
- A multi-agent system with one agent per action type

<details><summary>Show answer & why each option</summary>

- ✅ **Routing workflow** — A workflow in disguise — distinct input types with distinct handlers. Routing is predictable and 5–20× cheaper than an autonomous agent; de-escalate to the simplest pattern that meets the bar.
- **Fully autonomous** — Over-engineering: hands control flow to the model (high cost, less predictable) for a task with three known branches.
- **Multi-agent per action** — Actions are mutually exclusive per email and some are writes; parallel agents add coordination cost and conflict risk for no benefit.
</details>

### Q9. Each step of your ReAct agent is ~95% reliable in isolation, yet end-to-end success on 15-step tasks is poor. What's going on, and the structural fix?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag2` · 🔮 Representative

- The model is too small for 15-step tasks — upgrade it
- Random variance — re-run until it passes
- ✅ Errors compound multiplicatively (0.95^15 ≈ 46%) — shorten trajectories, add verification gates that catch an error before it propagates, and use resumable checkpoints

<details><summary>Show answer & why each option</summary>

- **Too small** — A stronger model raises per-step reliability marginally but doesn't change the multiplicative compounding.
- **Random variance** — Re-running is pass@k thinking; it hides the structural fragility.
- ✅ **Multiplicative compounding → shorten + verify + checkpoint** — Long autonomous trajectories are fragile even with a strong model; the fixes are structural, not a bigger model.
</details>

### Q10. Your agent occasionally runs up a large bill by re-issuing slightly reworded versions of the same failing search for many steps. Your harness already has a max-steps cap. What's missing?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag2` · 🔮 Representative

- Nothing — the step cap will eventually stop it
- ✅ Semantic loop detection plus a per-task cost ceiling and progress detection (no new info in K steps → stop/escalate)
- A larger retry budget so it tries more variations

<details><summary>Show answer & why each option</summary>

- **Step cap is enough** — A step cap bounds count but not spend; many cheap-looking steps with growing re-sent context can still be expensive with no progress.
- ✅ **Semantic loop detection + cost ceiling + progress check** — Exact-match loop detection misses semantically-identical reworded calls; these catch the expensive, non-productive looping a step cap allows.
- **Larger retry budget** — Makes it worse — more permission to keep re-issuing variants that aren't working.
</details>

---

## Section C — MCP & tool integration `[agents-evals-llmops] ag3`

### Q11. You're building one embedded agent with a single, latency-sensitive tool surface and sensitive local credentials. MCP or in-process tools?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag3` · 🔮 Representative

- MCP — it's the modern standard, always use it
- ✅ In-process tools — you don't need cross-client reuse, and you avoid the latency and trust-boundary overhead
- Neither — agents shouldn't use tools with credentials

<details><summary>Show answer & why each option</summary>

- **MCP always** — MCP adds per-call latency and a credential/trust boundary you don't need for a single embedded agent — its value is multi-client/multi-vendor reuse.
- ✅ **In-process tools** — Lower-latency and keeps credentials out of a separate server process.
- **No credentialed tools** — Plenty of production agents use credentialed tools; the question is the integration layer.
</details>

### Q12. You're connecting a third-party MCP server. What's the right security posture?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag3` · 🔮 Representative

- Trust it — MCP is an official protocol
- ✅ Treat it as untrusted: least-privilege tool scope, an allow-list, version pinning, a sandboxed agent process, and per-call auditing
- Add a system-prompt rule telling the model not to misuse the tools

<details><summary>Show answer & why each option</summary>

- **Trust it** — The protocol being official doesn't make a given server trustworthy; tool-poisoning and CVEs are documented.
- ✅ **Untrusted + defense-in-depth** — A third-party server is an untrusted neighbourhood — least-privilege scoping, allow-list, pinning, sandbox, audit.
- **System-prompt rule** — Prompt-level rules don't contain a malicious or poisoned tool.
</details>

### Q13. An MCP server you approved months ago silently changes one tool's description to instruct the agent to attach a secret file to every call. Which control specifically defends against this "rug pull"?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag3` · 🔮 Representative

- ✅ Pinning the server version so an approved server can't silently change tool behaviour or descriptions
- A larger context window so the malicious description is diluted
- Lowering temperature so the model ignores the new instruction

<details><summary>Show answer & why each option</summary>

- ✅ **Version pinning** — A rug pull is a post-approval change; version pinning (plus re-review on upgrade) is the control aimed at it. An allow-list controls which tools, not which version.
- **Larger context** — More context doesn't neutralise an instruction the model reads; it just costs more.
- **Lower temperature** — Doesn't govern whether the model follows instructions embedded in a tool description.
</details>

### Q14. You need to let 50 engineers safely use ~30 internal and community MCP servers. What's the scalable architecture?

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag3` · 🔮 Representative

- Have each agent connect directly to whatever servers it needs, and document a policy
- Whitelist the community servers in a shared config and trust them
- ✅ Front everything with an MCP gateway/registry that proxies servers: central auth, allow-listing, version pinning, rate limiting, and per-call audit

<details><summary>Show answer & why each option</summary>

- **Direct connections + policy** — No central enforcement point; policy without a control plane is unenforceable at 50×30.
- **Static whitelist** — Doesn't handle auth, version pinning, rate limits, or per-call audit — and community servers are the highest-risk to trust wholesale.
- ✅ **MCP gateway/registry** — The API-gateway pattern for agents — one place to enforce least privilege, pin versions, and log every `tools/call`.
</details>

### Q15. After connecting eight MCP servers, your single agent's tool selection gets noticeably worse and its prompt grew. Best explanation?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag3` · 🔮 Representative

- The servers are too slow and the agent is timing out
- ✅ All eight servers' tool schemas were dumped into the prompt, re-inflating the tool-overload problem — enable servers per-workspace and route to the relevant few
- MCP is incompatible with single agents

<details><summary>Show answer & why each option</summary>

- **Too slow** — Latency would slow responses, not degrade which tool the agent picks; the symptom is selection quality plus prompt growth.
- ✅ **Tool-schema overload → scope/route** — Connecting many servers loads all their tools into context; past ~20–40 tools selection degrades and the prompt bloats.
- **Incompatible with single agents** — MCP works fine with a single agent; the issue is loading too many tool schemas at once.
</details>

---

## Section D — Memory & long-running state `[agents-evals-llmops] ag7`

### Q16. Your agent needs to work on a task that spans far more than one context window. The right architecture?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag7` · 🔮 Representative

- Use a model with a bigger context window and keep everything in the prompt
- ✅ Persist structured state on disk (feature list + progress log + git) and resume from it each session — stateless model, stateful harness
- Spawn many parallel agents to cover more ground

<details><summary>Show answer & why each option</summary>

- **Bigger window** — Context still rots and exhausts; "bigger window" delays the wall rather than removing it.
- ✅ **Persist state on disk** — Every long-running production agent does this: the window is a cache, files are the database, and a fresh session resumes from the handoff artifact.
- **Many parallel agents** — Doesn't solve persistence, and parallel writers conflict — the shared-write anti-pattern.
</details>

### Q17. An interviewer asks how you know your agent is production-ready. Best answer?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag7` · 🔮 Representative

- It completed the target task successfully in a demo
- ✅ It clears pass^k on a trajectory-graded golden set, emits span traces, gates writes behind approval, and has no lethal trifecta
- It uses the newest model and an MCP server

<details><summary>Show answer & why each option</summary>

- **Demo success** — That's pass@1 on a happy path — it hides variance, regressions, and the attack surface.
- ✅ **Measurable bar** — Production-ready spans reliability, observability, and security — not a single successful run.
- **Newest model + MCP** — Tooling choices don't prove reliability or safety, and MCP adds a trust boundary you must still secure.
</details>

### Q18. "Your research agent costs about $124 per deep report and the PM wants it 50% cheaper. What do you change, and in what order?"

> **Difficulty:** 🔴 Hard · `[agents-evals-llmops] ag7` · 🔮 Representative

- Switch to a smaller model for the whole task
- Add more agents to parallelise and finish faster
- ✅ Cache the stable system+tools prefix (0.1× reads), trim/compress observations, cut steps, then route easy turns to a cheaper model — cheapest-impact levers first

<details><summary>Show answer & why each option</summary>

- **Smaller model everywhere** — A blunt last lever that can tank quality; per-task cost is dominated by re-sent transcript across steps, which a cheaper model doesn't fix.
- **More agents** — Naive multi-agent runs ~10× via compounding handoffs and duplicated work — it raises cost.
- ✅ **Prefix caching → trim → cut steps → route** — Per-task cost is super-linear in trajectory length because the transcript re-sends each step; attack the dominant driver before a model swap.
</details>

### Q19. Your long-running agent occasionally crashes mid-task and loses hours of work; restarting begins from scratch. Best fix?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag7` · 🔮 Representative

- ✅ Persist a structured plan + append-only progress log + git checkpoints to disk and resume from them each session — stateless model, stateful harness
- Increase `max_steps` so it has time to redo the lost work
- Hold the entire task in a bigger context window so nothing is lost

<details><summary>Show answer & why each option</summary>

- ✅ **Persist plan + log + checkpoints** — The window is a cache, not durable memory; externalising state makes the agent resumable after a crash or context reset with no lost work.
- **Increase `max_steps`** — Doesn't prevent loss on the next crash and inflates cost.
- **Bigger window** — Still evaporates on a crash and rots as it fills; a cache, not a database.
</details>

### Q20. Two weeks after launch, users report your agent "got worse," though you shipped nothing. What in your harness both prevents this and now tells you what happened?

> **Difficulty:** 🟡 Medium · `[agents-evals-llmops] ag7` · 🔮 Representative

- A higher retry budget so it recovers from the degradation
- ✅ Pinned model+prompt versions with a CI eval gate on a golden set, plus production span traces and cost/steps alarms
- A larger context window to absorb the change

<details><summary>Show answer & why each option</summary>

- **Higher retry budget** — Retries mask symptoms and inflate cost; they neither prevent nor diagnose a provider-side change.
- ✅ **Pinning + eval gate + traces + cost alarms** — Pinning + an eval gate catch a silent provider checkpoint change before ship; production traces and cost/steps alarms localise it after — the Anthropic-postmortem pattern.
- **Larger window** — Unrelated to a behavioural regression from a model update.
</details>

---

## Section E — Reported agent interview questions

> Observed in real 2026 loops. Sources: Adil Shamim, *"Every AI Engineer Interview Question… from 100 Real Interviews"* — https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a · [amitshekhariitbhu/ai-engineering-interview-questions](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions) · NVIDIA/TechEon 2026 agentic-eval guides. Rehearse a crisp answer; the checkpoints above are the drill.

### Q21. What is an AI agent vs a simple LLM call? What makes a system truly *agentic* (and what doesn't)?
> **Difficulty:** 🟢 Easy · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** An agent runs a loop where the *model* controls the flow — it decides which tool to call, reads the observation, and decides the next action — over multiple turns toward a goal. A single LLM call (or a fixed prompt chain / router) is a *workflow*: the control flow is coded, not model-decided. "Agentic" = the model owns the branch/loop decisions. De-escalate to a workflow whenever the branches are known (Q8).

### Q22. When is agentic the WRONG solution?
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** When the task has a small, known set of branches (use a router/workflow — cheaper, predictable, testable), when latency/cost budgets are tight, or when errors compound over many steps with no verifier (Q9). Autonomy buys flexibility you pay for in cost, latency, and reliability.

### Q23. When would you choose multi-agent over single-agent? How do you debug failures across interacting agents?
> **Difficulty:** 🔴 Hard · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** Multi-agent wins for *parallel reads* with one synthesising writer (research fan-out). It fails for *shared writes* (Q6). Orchestration (a central coordinator) is easier to debug than choreography (peer messaging). Debug with span-level traces per agent + a shared task ledger; the classic failure is duplicated/conflicting work and inter-agent chatter.

### Q24. Your agent keeps picking the wrong tool — how do you improve tool selection?
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** Fix the tools first (Q1): clearer descriptions, poka-yoke schemas (enums, absolute paths). Then reduce the surface — route/retrieve the top-k relevant tools when you exceed ~20–40 (Q15, and `fundamentals` Q14). Prompt exhortation is the weakest lever.

### Q25. An agent deleted a prod DB. How do you prevent irreversible actions?
> **Difficulty:** 🔴 Hard · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions))
> **Senior frame:** Gate writes behind human approval; least-privilege tool scopes; make destructive tools require an explicit confirmation token; dry-run/plan-then-apply; and put irreversible actions outside the agent's autonomous authority. This is the write-gate half of breaking the lethal trifecta (see [evals.md](./evals.md)).

### Q26. Implement a ReAct agent that calls two tools and recovers from a tool error.
> **Difficulty:** 🟡 Medium · ✅ Reported ([amitshekhariitbhu](https://github.com/amitshekhariitbhu/ai-engineering-interview-questions)) — see also [coding.md](./coding.md)
> **Senior frame:** Loop: Thought → Action(tool, args) → Observation → repeat until answer. Tools return *structured* errors (Q2) so the loop can branch retry/give-up/escalate. Bound the loop (max steps + cost ceiling + progress check, Q10). Runnable skeleton in [`content/agents.md`](../content/03-agents-and-tool-use.md) and the coding file.

### Q27. Explain MCP. When would you use MCP over a custom tool-bus?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a), [MCP spec](https://modelcontextprotocol.io/docs/getting-started/intro))
> **Senior frame:** MCP is an open JSON-RPC client/server protocol ("USB-C for AI integrations") exposing resources, prompts, tools, and sampling. Use it when tools are reused across *multiple clients/vendors* or you want an ecosystem of servers. For a single embedded agent with hot-path latency and local creds, in-process tools win (Q11). At org scale, front servers with a gateway (Q14).

### Q28. Walk me through an MCP handshake between a Claude client and a Postgres MCP server.
> **Difficulty:** 🔴 Hard · ✅ Reported ([research/01 §5.1 emerging topics](https://modelcontextprotocol.io/specification/draft/architecture))
> **Senior frame:** Client initializes over stdio/HTTP, negotiates protocol version + capabilities; server advertises its tools/resources; client lists tools, the model chooses one, the client issues `tools/call` with typed args; the server executes the query and returns a structured result the model reads. Security posture: pin the version, least-privilege the DB role, audit every call (Q12–Q13).

### Q29. Orchestration vs choreography for multi-agent systems?
> **Difficulty:** 🟡 Medium · ✅ Reported ([Shamim](https://adilshamim8.medium.com/every-ai-engineer-interview-question-you-need-to-know-in-2026-from-100-real-interviews-b5b7ae4b961a))
> **Senior frame:** Orchestration = a central coordinator dispatches sub-agents and merges results (easier to trace, control, and bound). Choreography = peers react to each other's messages (more flexible, far harder to debug and prone to chatter loops). Default to orchestration unless you have a strong reason; keep one writer for shared state.

---

<sub>⬅ [Back to the index](./README.md) · Related: [Evals →](./evals.md) · [LLMOps →](./llmops.md) · [System design →](./system-design.md)</sub>

<sub>Part of the **[Landed](https://landed.jobs)** AI-native jobs family. No affiliation with the companies named. Content MIT-licensed.</sub>
