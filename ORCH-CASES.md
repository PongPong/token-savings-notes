# Case studies: one agent vs multi-agent

Companion to `NOTES.md`. Full cases, STE100 bullets, and attribution map.

Paste-ready for these NOTES (Pong L). STE100-style. Every figure attributed. No invented %. Expand Pattern 7 / §6 — do not duplicate full essays.

**How to read:** Verdict = preferred shape for *that* scenario. Isolation ≠ cheaper system-wide. Mark **self-reported** / **measured** / **official** as labeled.

---

### Case 1 — Noisy explore / tests / logs (multi wins for *parent* context)

- **Verdict:** multi-agent (parent + Explore / research subagent; summary only)
- **Combination:** Claude Code parent + built-in Explore (or custom research worker); pin Explore to Haiku when possible
- **Scenario:** Side task would flood main chat with search hits, test output, docs, or logs you will not re-read.
- **Why it fits:** Subagent keeps verbose tool exhaust out of the parent window. Parent pays only for the returned summary. Official product guidance: use a subagent when a side task would flood the main conversation; Explore is for search/analysis without edits.
- **Source:** Anthropic Claude Code docs — [Create custom subagents](https://code.claude.com/docs/en/sub-agents); [Manage costs](https://code.claude.com/docs/en/costs) (“Delegate verbose operations to subagents”); Claude Academy — [Subagents](https://academy.claude.com/courses/claude-code-101/subagents): “summarizes its findings and returns that summary back to Claude… without the entire journey… cluttering your main context.”
- **Savings:** no figure given (quality/context win; total tokens may rise — see Cases 3–5)

---

### Case 2 — Cheap orch + strong coding leaf (multi wins on *quota* for premium models)

- **Verdict:** multi-agent (cheap orchestrator + strong leaf subagents)
- **Combination:** Luna (cheap orch) + Astra as coding subagents; fresh contexts; leaf stops after implement; orch validates
- **Scenario:** Codex/ChatGPT-style work where Astra-as-main burns weekly Plus quota on large inputs despite caching (e.g. graphics / Dream Loop demos).
- **Why it fits:** Large Astra inputs destroy quota. Orch stays glue (plan steps, call leaf, test). Leaf does narrow coding in empty contexts. Reverse shapes failed author tests: Astra orch + Luna leaf worse; Luna code + Astra critique worse.
- **Source:** @anshuc X thread (UnrollNow mirror) — https://unrollnow.com/status/2098014776886448337 — First: “Saves 80-90% of your quota.” Correction: “savings are more like 40-70%. The 80-90% figure includes separate savings from my skill.” Demo: Astra-only ~50% weekly Plus quota vs Luna+Astra 7%.
- **Savings:** **self-reported** — first claim 80–90% corrected to **40–70%**; demo **~50% → 7%** of weekly Plus quota (includes skill effects on 3D path — author flagged)

---

### Case 3 — Agent teams in plan mode (multi *hurts* tokens — official caution)

- **Verdict:** one agent (or tiny team) unless you truly need peer messaging + shared task list
- **Combination:** single Claude Code session; avoid experimental Agent Teams unless required
- **Scenario:** Coordinated multi-session “team” with teammates in plan mode.
- **Why it fits:** Each teammate is a separate Claude instance with its own context. Official cost docs warn this shape multiplies spend. Teams off by default for a reason.
- **Source:** Anthropic — [Manage costs effectively](https://code.claude.com/docs/en/costs): “Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode, because each teammate maintains its own context window and runs as a separate Claude instance.”
- **Savings:** **official** ~**7×** *more* tokens vs standard session (plan-mode teammates). Not a savings figure.

---

### Case 4 — Modest fan-out on a small task (multi *hurts* — measured)

- **Verdict:** one agent (sequential in same thread)
- **Combination:** single Claude Code session; no Task/Agent fan-out
- **Scenario:** Same small coding task done (a) directly vs (b) fanned to two / five subagents. Correct results either way.
- **Why it fits:** Each subagent is a full agent loop (own system prompt + tools every turn). Parent only ingests compact results — but children re-pay baselines. On small tasks, orchestration overhead also made fan-out *slower* in timed lanes.
- **Source:** Systima — [The Subagent Tax](https://systima.ai/blog/subagent-tax) (22 Jul 2026); companion [Claude Code vs OpenCode token overhead](https://systima.ai/blog/claude-code-vs-opencode-token-overhead): direct **121,000** metered input vs two-subagent **513,000** (**4.2×**); matrix **2.6×–5.9×** vs sequential across Opus/Fable lanes.
- **Savings:** **measured** — multi used **2.6×–5.9×** *more* input tokens than sequential (not a saving). Haiku-pinned vs inherited Opus: **37%** fewer metered tokens in one matched pair (**measured**, n=1 pair).

---

### Case 5 — Blind fan-out per finding (multi *hurts* — lived incident)

- **Verdict:** one agent or few persistent workers (batch findings); not one agent per finding
- **Combination:** capped workers (comment suggests ~4–8) processing lists; stable prompt prefix for cache
- **Scenario:** Large repo audit; orchestrator spawned one fresh agent per small finding (~73 findings).
- **Why it fits:** Startup tax dominated. Homogeneous tiny tasks do not amortize cold agent boot. Prompt prefix differed early (per-finding id) → cache thrash.
- **Source:** GitHub anthropics/claude-code [#77834](https://github.com/anthropics/claude-code/issues/77834) (MehmetUluc): “~47.7K input tokens before… useful work”; “~4.9M total… ~3.5M… repeated startup”; “~71%… initialization.” Commenter: batch + max-subagents ceiling (4–6) in similar GTM setups; **~60%** startup waste cut claim is **self-reported** by commenter.
- **Savings:** **self-reported incident** — multi burned tokens; suggested batching cuts waste (commenter’s **~60%** on their setup)

---

### Case 6 — Subagent-heavy Max sessions (multi *hurts* quota — anecdotal)

- **Verdict:** one agent for routine Pro/Max day work; pin leaf models if you must fan out
- **Combination:** single Sonnet/Opus thread; if subagents — frontmatter `model: sonnet` / `haiku`, not inherit Opus
- **Scenario:** Quota gone mid-day during agent runs; typing feels fine but agent runs drain plan.
- **Why it fits:** Subagents do not share your warm parent cache; each re-reads. Inherit-Opus workers multiply price. ofox attributes plan-drain patterns to fan-out + Opus default.
- **Source:** ofox.ai — [Claude Code Usage Limit Hit Too Fast](https://ofox.ai/blog/claude-code-usage-limit-hit-too-fast-2026/): “Agent teams can use about 7x the tokens… when teammates run in plan mode.” “One developer who blew through Max 20x found 85% of the usage came from subagent-heavy sessions.” (restates Anthropic 7× for teams; 85% is attributed anecdote)
- **Savings:** no savings — **anecdotal** **~7×** / **85%** of usage from subagent-heavy sessions (**self-reported third-party anecdote** via ofox)

---

### Case 7 — Sequential single-file / dependent steps (one agent wins)

- **Verdict:** one agent
- **Combination:** single Claude Code (or Codex) thread; plan-then-edit in same context
- **Scenario:** One-file fix, ordered refactors, or five tasks that must run in series.
- **Why it fits:** Subagent startup (prompt + tools + round trips) dwarfs the work. Dependent steps cannot parallelize — you pay N× bootstrap for zero wall-clock win. Atticus rule: use subagent when main-context clutter prevented is worth more than startup overhead.
- **Source:** Atticus Li — [How to Save Tokens With Claude Code…](https://atticusli.com/blog/posts/save-tokens-claude-code/): “Subagent-heavy workflows can use around 7× the tokens of a single thread”; FAQ: “Use them when they prevent genuine main-context clutter or enable parallel independent work — not for quick sequential steps.” (7× attributed to Anthropic / secondary; treat as **attributed**, not Atticus’s own meter)
- **Savings:** no figure given for the single-agent win; avoid the **~7×** heavy-fan-out tax (**attributed**)

---

### Case 8 — Parallel independent research (multi can win *latency*; tokens still up)

- **Verdict:** multi-agent (parent + N Explore/research workers) when wall-clock > token bill
- **Combination:** parent + several read-only Explore/Haiku workers; parent synthesizes summaries
- **Scenario:** Find references across independent services/modules; or parallel auth/db/API surveys.
- **Why it fits:** Isolation + parallelism. Parent context stays clean. Token total usually higher (Cases 3–4); choose when interactive latency matters. CN practitioner write-ups echo official “noisy output → subagent” rule and Haiku for light retrieve.
- **Source:** Anthropic subagent docs (parallel research pattern); Systima notes fan-out buys speed only for “genuinely independent work on a big surface”; Juejin — [Claude Code 子代理深度实践](https://juejin.cn/post/7644429323654529024): “产出多、复用少” → 独立上下文；“轻量检索路由到 Haiku”.
- **Savings:** no universal %; expect **higher** total tokens; latency win only if work is independent (**no figure given**)

---

### Case 9 — Explore on Haiku after inherit change (multi + cheap leaf)

- **Verdict:** multi-agent (parent strong + Explore Haiku override)
- **Combination:** `~/.claude/agents/Explore.md` (or project) with `name: Explore` + `model: haiku` shadowing built-in
- **Scenario:** Codebase search while parent stays on Sonnet/Opus. After Claude Code ≥2.1.198, built-in Explore inherits parent model (no longer always Haiku).
- **Why it fits:** Exploration stays cheap; parent cache / model unchanged. Anthropic caching lesson: mid-session model switch rebuilds cache — better to hand off to a Haiku subagent than `/model` Haiku mid-thread.
- **Source:** Anthropic subagent docs (Explore inherit + override with `model: haiku`); Anthropic blog — [Prompt caching is everything](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything): “it would actually be more expensive to switch to Haiku than to have Opus answer, because we would need to rebuild the prompt cache for Haiku.” “best way to do it is with subagents… Explore agents, which use Haiku.” NOTES §6 Explore.md pattern (AINews / docs).
- **Savings:** no universal %; Systima matched pair Haiku-pin **37%** fewer metered tokens vs inherited Opus (**measured**, one pair)

---

### Case 10 — Plan on cheap / implement on strong (multi-*model*, often one agent each)

- **Verdict:** multi-model orchestration (not necessarily subagents): cheap planner session → clear → strong implementer
- **Combination:** Gemini (or Haiku) plan.md → `/clear` → Claude Code implement; or frankchu: lock tier per conversation, escalate only at cold boundaries
- **Scenario:** Large codebase research + plan, then focused implementation. Or naive per-step model router that busts prompt cache.
- **Why it fits:** Planning/search is noisy and model-agnostic. Implementation wants stable cache + strong coder. Mid-session downshifts invalidate cache and can cost *more*. Cold boundary (new session / compact restart) is the safe switch point — same economics as “cheap orch / strong leaf,” without mandatory subagent tax.
- **Source:** Shuttle (dcodes) — [Claude Code Best Practices](https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices): Gemini 2.5 Pro for plan/context gather; Claude for implement; `/clear` after 1–3 messages. frankchu — [router + caching trap](https://dev.to/frankchu/i-built-a-router-to-cut-my-claude-code-bill-and-prompt-caching-was-the-whole-problem-3ifl): per-step routing “can pay more with the router than without it”; lock tier; escalate only. Anthropic caching blog (above).
- **Savings:** no figure given for Gemini/Claude split; frankchu — no single % (mechanism)

---

### Case 11 — Permission / blast-radius isolation (multi wins on safety, not tokens)

- **Verdict:** multi-agent (planner read-only + executor write-scoped)
- **Combination:** parent plan / review agent (read-only tools) + leaf with Write/Edit; optional worktree isolation
- **Scenario:** Production changes where planner must not write; or parallel editors need git worktree isolation.
- **Why it fits:** Tool allowlists and worktrees buy safety and less merge conflict — not token thrift. Expect higher total tokens; pay for isolation deliberately.
- **Source:** Anthropic subagent docs (tool restrictions; `isolation: worktree`); Anthropic docs document `isolation: worktree` for parallel file edits.
- **Savings:** no figure given (safety/isolation, not token win)

---

### Case 12 — Astra plans + Sol implements (Brad Groux)

- **Verdict:** Astra coordinates; Sol implements, when the slices are independent
- **Combination:** Codex. GPT-6 Astra (up to Extra High) plans, delegates, and reviews. Three to five GPT-5.6 Sol threads (generally Medium) implement.
- **Scenario:** Existing code, a backlog, and a UI where a correct edit can still be the wrong edit. One Astra chat must understand, plan, write, fix, and hold every rule.
- **Wrong pattern (author):** “Man make fire.” A bigger prompt on the strongest model. One-shot habit on a project that already has code. Hook: “I was asking Eisenhower to storm the beaches of Normandy himself.”
- **Practice (author, Codex):**
  1. Astra reads project docs, the code, and open issues. It builds a dependency-ordered plan. It splits that plan into **3–5 independent** Sol assignments.
  2. Each Sol task gets the relevant context, a defined scope, the files or interfaces it owns, the guardrails that apply (not a vague “follow SOPs”), completion evidence, an isolated branch, and a focused PR.
  3. Astra reviews the diff and the evidence, fixes findings, merges when authorized, and starts the next dependency-ready work. Optional: a quiet 15-minute coordination check. It notifies only for a decision or a meaningful result.
  4. Serialize tasks that would both own the same migration or shared interface.
  5. “Done” means review, then merge when authorized, then update the preview. An open PR is not done. A draft the owner asked to keep open stays a draft.
- **Author honesty:** Work comes back faster because several independent pieces move at once. **3–5 threads means concurrency.** No controlled before-and-after benchmark. Dependencies, review, and integration still take time.
- **Token fit (house):** Same family as Case 2 (cheap orch + strong leaf) and Case 10 (plan, then implement), and as [MODE-MATRIX](./MODE-MATRIX.md): frontier model for plan, review, and coordination; Sol (Medium) for scoped implementation. The article states no price. Avoid one long frontier session that does every role. Parallel leaves can raise total tokens versus one serial session (Cases 3–4; Agent Teams). The author’s claim is workflow, intervention, and concurrency.
- **Source:** @BradGroux (Brad Groux), shown 6:49 PM · 22 Sep 2026 (author-local display) — https://x.com/BradGroux/status/2102455199583334625 — article “I Think I’ve Been Using GPT-6 Astra Wrong” (https://x.com/i/article/2102449008232861696). Row in [SOURCES.md](./SOURCES.md).
- **Savings:** **no controlled benchmark / no token-%**

---

### STE100 bullets (deck)

- One agent wins for sequential, dependent, or tiny work.
- Multi wins when noise would pollute the parent, or work is truly parallel.
- Each subagent re-pays prompts and tools. Isolation is not free.
- Official caution: agent teams ≈ 7× tokens in plan mode.
- Measured caution: small fan-outs used 2.6×–5.9× more input than sequential (Systima).
- Lived caution: one agent per finding paid ~47k startup each; most tokens were boot (issue #77834).
- Cheap orch + strong leaf can save *premium* quota (@anshuc; self-reported 40–70%).
- Astra can plan and review while Sol implements 3–5 independent scopes (Brad Groux). That claim is concurrency. No token-%.
- Pin Explore / workers to Haiku. Do not inherit Opus for search.
- Do not switch models mid-session to “save.” Use a Haiku subagent or a cold boundary.
- Parent must receive summaries only (NOTES P.7). Never paste full worker logs up.

---

### Decision cheat sheet

| Situation | Prefer | Why |
| --- | --- | --- |
| One file / ordered steps / tiny shell | **One agent** | Startup tax > work; no parallelism |
| Search, tests, logs, doc crawl you won’t re-read | **Parent + Explore/research (Haiku)** | Noise stays out of parent |
| Parallel independent modules / services | **Parent + N read-only workers** | Latency; tokens still up — accept trade |
| Premium model quota dying on long main threads | **Cheap orch + strong leaf** (@anshuc shape) | Keep big model off orchestration tokens |
| Agent Teams / plan-mode teammates | **Avoid unless required** | Official ~7× vs standard session |
| Fan-out per tiny finding / audit item | **Few persistent workers + batch** | Amortize ~47k-class startup |
| Want cheaper model mid-task | **Haiku subagent or new session** | Mid-thread `/model` busts cache |
| Plan then implement across models | **Cheap plan session → clear → strong implement** | Cold boundary; Shuttle pattern |
| Safety / write blast radius | **Read-only planner + scoped executor** | Isolation for risk, not thrift |
| Existing backlog; independent slices; one frontier chat does every role | **Astra plan/review + Sol leaves** (Brad Groux, Case 12) | Concurrency and less role-switching. Serialize shared interfaces. No measured token cut. |
| Unsure | **One agent first; add workers only when clutter or parallelism is real** | Default away from tax |

---

### Attribution map (do not drop)

| Claim | Kind | Primary URL |
| --- | --- | --- |
| ~7× agent teams (plan mode) | official | https://code.claude.com/docs/en/costs |
| Subagent = summary; Explore/Haiku override | official | https://code.claude.com/docs/en/sub-agents |
| Mid-session Haiku switch vs Explore subagent | official | https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything |
| Academy summary-only teaching | official | https://academy.claude.com/courses/claude-code-101/subagents |
| 121k→513k / 2.6×–5.9× / Haiku-pin 37% | measured | https://systima.ai/blog/subagent-tax |
| ~47k startup / 71% init / 4.9M run | self-reported bug | https://github.com/anthropics/claude-code/issues/77834 |
| Luna+Astra 40–70%; 50%→7% demo | self-reported | https://unrollnow.com/status/2098014776886448337 |
| 7× / 85% Max anecdote | anecdotal (ofox) | https://ofox.ai/blog/claude-code-usage-limit-hit-too-fast-2026/ |
| ~7× subagent-heavy (Atticus cites Anthropic) | attributed | https://atticusli.com/blog/posts/save-tokens-claude-code/ |
| Gemini plan → Claude implement | practitioner blog | https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices |
| Router vs cache lock | practitioner | https://dev.to/frankchu/i-built-a-router-to-cut-my-claude-code-bill-and-prompt-caching-was-the-whole-problem-3ifl |
| Astra plans; 3–5 Sol threads implement; no benchmark | author practice (X article) | https://x.com/BradGroux/status/2102455199583334625 |
| 省token / Haiku explore / 产出多复用少 | CN practitioner + docs | https://juejin.cn/post/7644429323654529024 |

**Conflicts already in NOTES §4:** Subagents save *parent* context but can raise *total* tokens; cheap per-turn routers can raise spend vs cold-boundary handoff.


