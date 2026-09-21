# Token-savings research pack — cut LLM/agent token spend

**For:** Engineer / Pong L presentation · public pack  
**Prepared:** 17 Sep 2026 (Europe/London); audience split 2026-09-21  
**Rule:** No invented quotes or numbers. % claims marked self-reported. **Savings cheat sheet** = single numbers board.

**Audiences:** [Browser chat guide](./guides/browser-chat.md) · [CLI agents guide](./guides/cli-agents.md) · [Pattern map](./guides/MAP.md) (browser vs CLI actions — no duplicated tips).

**Companions:** [EXAMPLE-PROMPTS.md](./EXAMPLE-PROMPTS.md) · [ORCH-CASES.md](./ORCH-CASES.md) · [SOURCES.md](./SOURCES.md) · [MODE-MATRIX.md](./MODE-MATRIX.md) · [TYPESAFE-JEV.md](./TYPESAFE-JEV.md) · [SPEAKER-NOTES.md](./SPEAKER-NOTES.md)

---

## Contents

- [Do this week](#do-this-week)
- [Savings cheat sheet (rough figures)](#savings-cheat-sheet-rough-figures)
- [Top patterns (by savings impact)](#top-patterns-by-savings-impact)
- [Meter token use without typing /context](#meter-token-use-without-typing-context)
- [TypeSafe Jev (System One)](#typesafe-jev-system-one)
- [Computer-use token hygiene](#computer-use-token-hygiene)
- [Case studies: one agent vs multi-agent](#case-studies-one-agent-vs-multi-agent)
- [Agent modes × context pollution](#agent-modes-context-pollution)
- [Slide bullets (STE100-style)](#slide-bullets-ste100-style)
- [Caveats — anecdotes vs measured; conflicts](#caveats-anecdotes-vs-measured-conflicts)
- [Practical real-world examples (house + field)](#practical-real-world-examples-house-field)
- [Context engineering (PRP workflow)](#context-engineering-prp-workflow)
- [Example prompts (real-world)](#example-prompts-real-world)
- [Measured study: five “token saving” modes (u/bisonbear2, 2026)](#measured-study-five-token-saving-modes-ubisonbear2-2026)
- [Library catalog: llm-engineer-toolkit](#library-catalog-llm-engineer-toolkit)
- [Generative UI token hygiene (json-render)](#generative-ui-token-hygiene-json-render)
- [Sources](#sources)
- [Gaps](#gaps)
- [Team practice (standing)](#team-practice-standing)

## Do this week

Default operating checklist for agents and humans. Details live in the sections below.

1. **Turn off unused MCP / tools** — Biggest easy win. Prefer short schemas or Tool Search; leave Claude Code Tool Search on (default). One high-signal graph tool beats many idle servers.
2. **New session for a new task** — `/clear` between unrelated work. `/compact` mid-task around ~60% full with an explicit keep-list (prefer prune-first / [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) over lossy summary when tool exhaust dominates).
3. **Meter without typing** — Status line while you work; weekly `npx ccusage@latest daily` (see Meter section). Do not live on `/context`.
4. **Dense edits, format once** — Agent emits dense patches; run the project’s CLI formatter once at the end (never LLM pretty-print).
5. **Route before you burn frontier** — Cheap model (or TypeSafe Jev) for triage/guards; strong model only for hard implement. No mid-warm `/model` switch.

Optional: skim [TYPESAFE-JEV.md](./TYPESAFE-JEV.md) if you want classify/route/verify without a full LLM call.

## Savings cheat sheet (rough figures)

**This table is the single numbers board.** Other sections explain *how*; they should not invent new %. If a figure is not here, treat it as unlisted for the deck.

Rough = as reported elsewhere. Say “practitioners report…” on stage. Do **not** guarantee %.

| Technique | Rough savings (as reported) | Evidence | Notes |
| --- | --- | --- | --- |
| MCP hygiene (disable / consolidate / Tool Search) | Schema cut **~60%** (14k→5.7k); Tool Search **~85%** (134k→5k); idle MCP **4–10k** or **67k+** with many servers; PostHog **113k→5k** single exec; Rulestack **~61k→21k** (~**40k** deferred); Cursor DCD **46.9%** fewer total agent tokens (MCP-calling runs) | Spence **self-measured**; Tool Search **Anthropic internal** via VB; Shuttle / staff / PostHog **self-reported**; Rulestack **self-measured**; Cursor **A/B** | Usually the biggest *easy* win |
| Lean AGENTS.md / skills on demand | **~2–3k tokens/turn** when skill not loaded | Verma **self-reported** | Always-on tax every turn |
| `/clear` / new session (unrelated task) | **~30–50%** per-message (community); one case **412k** cleared | Atticus **attributed**; @jcfmunoz **self-reported** | Same task may prefer warm cache |
| `/compact` ~60% util (not 95%) | No universal %; Shuttle autocompact buffer example 45k→176k free | MindStudio tip; Shuttle **self-reported** | Quality lever more than a fixed % |
| Context prune before LLM summary (DCP etc.) | No universal %; cache-hit notes ~85% vs ~90% | DCP docs ecosystem | Free mechanical prune first |
| fast-jev-compaction (Jev keep/drop tools) | No universal token-% in README; reports **char** `reductionRatio`; fallback to LLM summary if too little cut | [tamaratran/fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) README; LiteLLM TypeSafe compaction blog | Verbatim prune — not a summary; measure incl. Jev cost |
| STE100 / Concise / terse output | **No STE100 %**; terse output **30–50%** of *output*; Concise article **40–60%** output | Hasan / @ellen_in_sf **self-reported**; STE100 gists **no figure** | Cuts narration, not whole bill |
| Cheap orch + strong leaf | Quota **40–70%** (was 80–90%); demo **50%→7%** weekly Plus | @anshuc **self-reported**, author-corrected | Premium-quota shape |
| Haiku/cheap Explore leaf (vs inherit Opus) | **~37%** fewer metered tokens (one pair) | Systima **measured** n=1 | Pin leaf model |
| Cheap plan → clear → strong implement | No single %; Terra-style **−49% cost** / **+6% tokens** | Shuttle; bisonbear2 **measured** | Price ≠ fewer tokens |
| Prompt-cache hygiene | Cache reads ~**10%** of input price; mid-session model switch can **raise** cost | Anthropic | Protect prefix |
| Parent + Explore (summary only) | No %; parent context win | Official docs | Total tokens may still rise |
| Dense edit + format-once CLI | No public % | House rule | Stops format loops |
| RTK (shell compress) | Vendor **60–90%** of *command output*; bill often **~0 / +5%** | Vendor vs JetBrains/Quesma/bisonbear2 **disputed** | Don’t cite 60–90% as bill |
| codebase-memory-mcp | Author **10×** / **~99%** vs file-by-file explore | Preprint / README **self-reported** | Verify on your harness |
| Graphify (knowledge graph skill) | Query graph instead of grepping; code AST map uses **0** LLM credits (README benchmark table) | Graphify README / BENCHMARKS — **no total-session token %** | Prefer when docs+code; compare vs codebase-memory-mcp below |
| Jev Ultrafast (indexed DOM + Jev) | Author eval **n=3 pairs** (same Mercury helper): median **9.450 s → 7.092 s** (~**25%** lower **runtime**); TypeSafe reqs **22 → 17**; browser protocol **1,092 → 101**. Demo **7.073 s**. **No TypeSafe billed $ / no universal session token-%** | [jev-ultrafast `docs/performance.md`](https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md) **author / small-n** (sign-test p=0.25) | Runtime & request counts — **not** a token-% |
| tgrep vs rg | Up to **~52×** faster (latency) | microsoft/tgrep benches | Token win is indirect |
| Agent Teams / heavy subagent fan-out | **~7× more** tokens; small fan-out **2.6×–5.9× more** | Anthropic **official**; Systima **measured** | Savings = *avoid* this |
| Stacked levers (MCP + clear + terse + …) | Combined “**~80%**” / $74→$11 weekends | Hasan **self-reported** | Not transferable as a guarantee |

**Footer:** Isolation ≠ cheaper system-wide. Measure with a status line + `ccusage` (or your dashboard). Use `/context` only for a one-shot peek.

## Top patterns (by savings impact)
Sorted by **rough savings impact** (high → low). Stars = impact estimate from attributed NOTES evidence (not a lab score). Say “practitioners report…” — do not guarantee %.

### 1. MCP / tool hygiene (fewer tools, short schemas, defer load) — ★★★★★ (5/5)
**Impact (why 5/5):** Largest easy win: schema cuts ~60% (Spence), Tool Search ~85% internal, idle MCP tens of k, Cursor DCD **46.9%** on MCP-calling runs (product A/B).
**What people do:** Disable unused MCP servers. Consolidate tools (params > many near-duplicate tools). Trim descriptions. Use Tool Search / `defer_loading` so schemas load on demand. Exception: one **high-signal** exploration MCP (e.g. [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) or [Graphify](https://github.com/Graphify-Labs/graphify) — see Targeted exploration) can beat many low-signal file tools — still disable everything else you are not using this session.
**Claude Code `ENABLE_TOOL_SEARCH`:** Leave Tool Search on for first-party hosts (unset or `true`). `auto` / `auto:N` thresholds against **context-window size**, not “keep my prompt small.” On 1M ctx, 5% = 50k, so ~40k defs load upfront and undo the deferral (Rulestack **self-measured**, v2.1.263, 2026-09-09: unset **20,819** vs `false` **60,989** vs `auto:5` **62,319**; ~**40,170** deferred / 88 defs). Claude Code may disable Tool Search on non-first-party `ANTHROPIC_BASE_URL` / older models.
**Cursor Dynamic Context Discovery** ([product blog](https://cursor.com/blog/dynamic-context-discovery), Jediah Katz, 6 Jan 2026): sync MCP tool descriptions to folders; agent gets names then looks up schemas. A/B on runs that called an MCP tool: **46.9%** fewer total agent tokens (statistically significant; high variance by MCP count). Same post: long tool outputs → files; chat history as files for summarization recovery; Agent Skills; terminal sessions as files.
**Why it saves:** Tool defs load before you type. Multi-server setups commonly eat tens of thousands of tokens at session start.


### 2. Fresh session / `/clear` per unrelated task — ★★★★☆ (4/5)
**Impact (why 4/5):** Community ~30–50% per-message; stops dead-context rent. Caveat: same task may prefer warm cache.
**What people do:** New chat when the task changes. Keep related phases in one session. Put durable rules in CLAUDE.md / AGENTS.md.  
**Why it saves:** Stale history rides every turn. Clearing stops paying rent on dead context.  
**Caveat:** Within one task, a warm cache can make one long session cheaper than many cold restarts (see caching).


### 3. Cheap-model routing (small for triage / explore; big for hard steps) — ★★★★☆ (4/5)
**Impact (why 4/5):** Quota/price lever (anshuc 40–70% self-reported; Terra −49% cost). Cold-boundary only.
**What people do:** Haiku / Luna / Gemini for search, plans, scaffolding. Opus / Astra / Sonnet for hard coding and irreversible work. Route at session or cold boundaries — not every turn.  
**Why it saves:** Frontier rates on grep/rename/test-scaffold waste money.  
**Catalog:** Routers in [llm-engineer-toolkit](https://github.com/KalyanKS-NLP/llm-engineer-toolkit) (e.g. RouteLLM) — verify cost claims upstream.
**Caveat:** Mid-session model switches bust prompt cache; uncached cheap can beat cached frontier only if measured.


### 4. Lean standing instructions (short CLAUDE.md / AGENTS.md; skills on demand) — ★★★★☆ (4/5)
**Impact (why 4/5):** ~2–3k tokens/turn avoided when skills stay unloaded (Verma self-reported).
**What people do:** Keep always-on rules thin (phonebook, not manual). Move domain packs into skills that load when triggered.  
**Why it saves:** Always-on files tax *every* turn (even with cache, still a recurring cost). On-demand skills avoid that baseline.
**Related:** Feature-specific context belongs in a **PRP / INITIAL.md**, not in always-on CLAUDE.md — see **Context engineering (PRP workflow)**.


### 5. Prompt-caching hygiene (stable prefix; don’t break it) — ★★★★☆ (4/5)
**Impact (why 4/5):** Cache reads ~10% of input price; one bust forces full-price re-read.
**What people do:** Static system + tools first. Dynamic stuff in messages. Don’t shuffle tools or swap models mid-session. Cache-safe compaction forks.  
**Why it saves:** Cache reads ~10% of input price. One prefix byte change can force full-price re-read of the whole history.
**Catalog:** semantic query cache (e.g. GPTCache) under Cache in [llm-engineer-toolkit](https://github.com/KalyanKS-NLP/llm-engineer-toolkit) — separate from Anthropic prompt-cache prefix hygiene; vendor 10× claim is **self-reported**.


### 6. Context compaction (`/compact`, mid-session summarize) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Cuts recurring history tax; no universal %. Compact ~60% util.
**What people do:** Summarize history on purpose. Keep decisions, active errors, files in scope, constraints. Drop resolved logs and dead tangents.  
**Why it saves:** Full history is resent every turn. A shorter summary cuts the recurring re-read tax.  
**Practice tip:** Compact ~60% utilization (not at 95%). Auto-compact near the ceiling often summarizes already-degraded context.  
**Prefer prune-before-summary when you can:** LLM summaries are lossy (paths, errors, constraints vanish). For tool-heavy sessions, prefer Pattern 7 / [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (Jev keep/drop of tool calls — verbatim text) over a summary-only `/compact`. See [TYPESAFE-JEV.md](./TYPESAFE-JEV.md#use-case-verbatim-context-compaction-fast-jev-compaction).


### 7. Context pruning (drop tool exhaust first; summarize last) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Mechanical prune is free; Jev-guided prune costs a cheap decision call; no single published %.
**What people do:** Remove stale tool outputs, duplicate file reads, old errors *before* LLM summarization. OpenCode DCP: dedup + purge + optional compress. Live DCP README still documents the plugin; it now points at related project **[Sleev](https://sleev.ai)** (`sleev` CLI) — local proxy for Claude Code, Codex, OpenCode. DCP still works; Sleev is a related successor multi-harness proxy. No published savings % here — do not deep-dive without measuring. Atlassian Rovo Dev: structure-aware prune cascade.  
**Concrete tool — [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction)** (`tamaratran/fast-jev-compaction`): npm library + Claude Code plugin. Uses **TypeSafe Jev** to score each historical tool call/result (`noul`: keep call? keep result verbatim?). Actions: keep both, keep call + truncate result (`truncateHeadChars`, default 300), or drop call+result. **Never rewrites** user/assistant text — only deletes/truncates tools. Pins first message + newest `preserveRecentMessages` (default 6). Claude Code hook replaces built-in `/compact` summary when reduction is enough; else falls back to summary. Needs `TYPESAFE_API_KEY`; function hooks flag for Claude Code 2.1.274+. Related: LiteLLM’s TypeSafe Jev compaction guardrail (drop stale tool results before the model call).  
**Why it saves:** Most bloat is machine output, not user intent. Mechanical prune is free (no extra LLM call). Jev prune is cheap vs a frontier summarize pass and keeps exact paths/errors. Summary is the fallback.  
**Caveat:** README publishes `reductionRatio` (chars), not a universal token-%. Include Jev’s own cost when measuring. Do not invent a deck %.


### 8. Targeted exploration (narrow @files; stop early; shrink shell output) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Stops junk entering the prefix; tool claims vary (RTK disputed; graph MCP self-reported).
**What people do:** Point at paths. Prefer indexed / structural search over full-file dumps. Explicit stop rules in AGENTS.md. Concrete tools practitioners use:

- **[RTK](https://github.com/RTK-AI/rtk) (Rust Token Killer)** — wrap shell / CLI output so the agent gets a compressed form (`rtk git status`, hooks rewrite Bash). Use after choosing the right tool; do not let RTK pick the tool. Built-in `Read`/`Grep` often bypass the Bash hook.
- **[tgrep](https://github.com/microsoft/tgrep)** — trigram-indexed grep (client/server) as a **ripgrep/grep replacement** on large trees. Index once (`tgrep serve .`); agents should prefer `tgrep` for repo search when the index is warm (see repo `AGENTS.md`). Speed win on huge monorepos; token win is indirect (faster, tighter search → fewer thrash reads).
- **[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)** — MCP that builds a local code knowledge graph (tree-sitter + optional Hybrid LSP). Structural / graph queries replace dozens of grep+read cycles. Authors claim large token cuts vs file-by-file exploration (see sources; mark self-reported). Prefer this *instead of* blind full-repo greps when installed — still subject to MCP hygiene (don’t pile unrelated MCP servers beside it).
- **[Graphify](https://github.com/Graphify-Labs/graphify)** (`Graphify-Labs/graphify`) — `/graphify` skill maps code (+ docs/PDFs/media) into a **queryable knowledge graph** so the agent can query the graph instead of grepping files. Code path: local tree-sitter AST (deterministic, no LLM credits for code map per README). Docs/media may use a semantic pass. Not a vector index. Install: `uv tool install graphifyy` then `graphify install`. No universal token-% in README — savings mechanism is fewer file crawls.

**Graphify vs codebase-memory-mcp (feature comparison — Pong, 2026-09-17):**

| Feature | Graphify | codebase-memory-mcp |
| --- | --- | --- |
| Primary scope | Multi-modal (code, PDFs, docs, DB schemas) | Code-first with deep structural and type analysis |
| Parsing & language layer | Local tree-sitter (36 languages / 79 extensions per comparison card) | Hybrid LSP + tree-sitter (stronger TS/JSX resolution per comparison card) |
| Query mechanism | Direct assistant mapping / integrated context retrieval (`/graphify`, `graphify query` / path / explain) | Cypher-style / graph query engine for arbitrary traversals (15 MCP tools) |
| Best used for | Projects that need docs, specs, and code mapped together | Deep programmatic code queries, dead-code detection, precise TS typing |

**Pick:** Graphify when the corpus mixes **docs + code**; codebase-memory-mcp when you need **deep code-structure / type-aware** graph queries. Both still count as MCP/skill surface area — keep other unused servers off (pattern: MCP hygiene).

**Why it saves:** Wrong exploration pins junk into the prefix forever. Stopping early and returning only useful hits avoids the re-read tax.

### 9. Short / STE100-style output (cut narration tokens) — ★★☆☆☆ (2/5)
**Impact (why 2/5):** Output/narration only; no solid STE100 total-token %. Softens decode cost.
**What people do:** Concise output style; “lead with result, no preamble”; ASD-STE100-style short sentences for status.  
**Why it saves:** Output tokens are sequential and often priced higher than input. Less chat wrapper = less decode cost.  
**Caveat:** STE100 is clarity-first; gists note it is not a hard length cap on the whole reply.


### 10. Subagents / specialized workers (isolate noise; return summaries) — ★★☆☆☆ (2/5)
**Impact (why 2/5):** Parent-context win; heavy fan-out can cost ~7× total tokens — use selectively.
**Cases:** See **Case studies: one agent vs multi-agent orchestration** (when one agent wins vs cheap orch + strong leaf vs Explore/Haiku).
**What people do:** Spawn workers with clean context for search/research. Parent keeps only the answer. Cheap orchestrator + expensive coding worker is a common Codex pattern (@anshuc).  
**Why it saves (parent):** Exploration junk stays out of the main window.  
**Caveat:** Each subagent re-pays prompts/tools. Anthropic-cited ~7× for heavy fan-out. Use when isolation value > startup cost.


---


## Meter token use without typing /context

You do not need to type `/context` or `/status` all the time. Most stacks already write usage to disk or can show a live meter.

### Live meter (while you work)

- **Claude Code status line** — Set a `statusLine` command in settings. After each turn, Claude Code runs your script with session JSON (context %, tokens, cost, rate limits). Your script prints a bar at the bottom. This is local and does **not** spend API tokens.
- Guides: [status line docs](https://claude-code.mintlify.app/en/statusline), [Andrew Connell write-up](https://www.andrewconnell.com/articles/claude-code-cli-statusline/), [example script](https://github.com/nnaveenraju/claude-code-status-line).
- Caveat: the built-in context % can miss system/MCP overhead ([issue #34537](https://github.com/anthropics/claude-code/issues/34537)). Treat it as a pressure gauge, not a perfect bill.

### History from local logs (after sessions or on a schedule)

Agents already log turns. These tools read those files:

| Tool | What it does | Link |
| --- | --- | --- |
| **ccusage** | Daily / session / 5-hour block reports from local JSONL (Claude Code, Codex, OpenCode, and more) | https://github.com/ryoppippi/ccusage — try `npx ccusage@latest` |
| **TokenTelemetry** | Local dashboard across Claude Code, Cursor, Codex, OpenCode, Copilot, … | https://github.com/onichan64/tokentelemetry |
| **Agent Profiler** | Hooks + SQLite; session shape, tool noise, always-on context audit | https://github.com/cleverb/agent-profiler |
| **Agentlytics** | Unified local analytics for many editors | https://www.npmjs.com/package/agentlytics |
| **Agent Lens** | Multi-agent usage + cache view | https://github.com/opseal/agent-lens |
| **CodeDash** | Session browser + cost from real token fields | https://github.com/EvilFreelancer/codedash |

### Hooks and OpenTelemetry (always-on export)

- **PostToolUse / transcript hooks** — Read `transcript_path` JSONL (`input_tokens`, cache fields, `output_tokens`) and append to your own ledger.
- **Claude Code OpenTelemetry** — `CLAUDE_CODE_ENABLE_TELEMETRY=1` plus OTEL exporters for token and cost metrics (needs a collector such as Grafana).

### Experiments (A/B, not a live UI)

- **Stet** — Replay real repo tasks and compare bills across setup changes ([stet.sh write-up](https://www.stet.sh/blog/gpt-56-token-saving-modes)).
- **Systima** — Metered studies (e.g. subagent tax).

### Simple weekly habit

1. Keep a **status line** on while coding.
2. Once a week run **`npx ccusage@latest daily`** (or your dashboard) and note cache-read share + top sessions.
3. Only use `/context` when you need a one-shot breakdown inside an active chat.

**Also see:** [llm-engineer-toolkit → LLM Monitoring](https://github.com/KalyanKS-NLP/llm-engineer-toolkit#llm-monitoring) for Helicone / Opik / Phoenix / agenttrace and peers (catalog only — verify before adopting).

**Weekly refresh:** re-check links and new metering tools each Monday with the rest of this playbook.


## TypeSafe Jev (System One)

Jev is a **decision** model (not a chat/coding LLM): state + typed questions → probabilities. Use it to replace expensive LLM calls for routing, guardrails, and triage — and for **verbatim context prune** via [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (Patterns 6–7).

**Details and real-world use cases:** [TYPESAFE-JEV.md](./TYPESAFE-JEV.md) (includes **open-weights Laya** alternative).

**One-line fit:** Keep frontier models for writing, hard reasoning, and sparse plans; let Jev handle fast classify / route / verify / keep-or-drop-tool-result / **per-tick action Choice** (e.g. [minecraft-agent](https://github.com/rmalde/minecraft-agent) §9; live web [jev-ultrafast](https://github.com/browser-use/jev-ultrafast) §11) (price list ~$0.042/MTok input, output free — verify live; speed/cost multiples are author ceilings). Open-weights alt: [Laya](https://github.com/NandhaKishorM/laya) (same `choice`/`score`/`noul` shape; Apache 2.0 — see TYPESAFE-JEV).


## Computer-use token hygiene

GUI agents burn tokens when **every** click goes through a frontier model + screenshot. Prefer: plan once with a strong model; score local form/clicks with a **System One** specialist or Jev/Laya Choice over code-listed elements; use structured a11y/DOM over pixels when you can; skip already-filled fields; use an API/connector instead of the GUI when one exists.

**Live web sibling:** [jev-ultrafast](https://github.com/browser-use/jev-ultrafast) — page → indexed element table; Jev picks operation + target in **one** request; a small LLM writes text only on `TYPE_TEXT`; default loop has **no screenshots**. Author runtime/request counts (not a token-%) live on the Savings cheat sheet.

**Details:** [CUA-S1 §10](./TYPESAFE-JEV.md#10-computer-use--specialist-system-one-for-local-gui-decisions-cua-s1) · [Jev Ultrafast §11](./TYPESAFE-JEV.md#11-browser-use-jev-ultrafast--indexed-dom--jev-optarget).

## Case studies: one agent vs multi-agent

When to use one agent vs multi-agent (and which combo). Full cases and sources:

**Details:** [ORCH-CASES.md](./ORCH-CASES.md)

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
| Unsure | **One agent first; add workers only when clutter or parallelism is real** | Default away from tax |

---

## Agent modes × context pollution

Session modes (Ask / Plan / Agent / Debug / Explore…) across Cursor, Claude Code, OpenCode, Codex, Windsurf, Continue, Aider, Copilot — and which ones keep the **main** context clean.

**Full matrix + glossary + per-agent notes:** [MODE-MATRIX.md](./MODE-MATRIX.md)

| Protects main context? | Pattern |
| --- | --- |
| **Best** | Explore / research **subagent** → summary only (Claude Explore, Copilot/OpenCode/Cursor subagents) |
| **Yes** | Chat with **no tools** (Continue Chat); Claude `/btw`; Plan → **fresh** implement (Windsurf @plan file; Cursor mode switch = fresh context) |
| **Partial** | Ask/Plan **in main** with read tools — no writes, but search can still fill the window |
| **No** | Agent/Build/Code in main; **Cursor Debug** (logs/traces); Aider ask→code same thread (by design) |
| **Isolates peers, multiplies spend** | Claude Agent Teams (~**7×** tokens in plan mode — official) |

Do not confuse **approval** modes (Auto-review, acceptEdits, sandbox) with Ask/Plan — they change prompts, not isolation.

## Slide bullets (STE100-style)

- Context engineering: lean always-on rules + examples + PRP blueprint; execute in a clean session.

**Title: Cut agent token spend — what practitioners actually do**

- Context is resent every turn. Junk context is a recurring tax.
- Compact on purpose. Keep decisions, errors, scope. Drop dead logs.
- Prefer prune of tool exhaust before LLM summary.
- Start a new session for a new task. Keep related work together.
- Turn off unused MCP tools. Short schemas beat many near-duplicate tools.
- Prefer on-demand tool search / deferred schemas when available. Leave Claude Code Tool Search on. Treat `auto:N` as a % of the context window.
- Cursor folder-based MCP discovery. Product A/B: 46.9% fewer tokens on MCP-calling runs.
- Keep CLAUDE.md / AGENTS.md short. Load skills only when needed.
- Route cheap models to explore and triage. Keep frontier for hard steps.
- Do not switch models mid-warm session unless you accept a cache miss.
- Use subagents to keep noise out of the parent. Do not fan out blindly.
- Ask for short replies. Lead with the result. Skip preambles.
- Point at files. Prefer tgrep / graph MCP / `rg` over full dumps; wrap shell with RTK. Stop when evidence is enough.
- Protect the prompt-cache prefix. Stable tools and rules first.
- Prefer a status line + weekly `ccusage`. `/context` is optional. Anecdotes ≠ your bill.
- Local “token savers” can raise the *trajectory* bill. Replay your own tasks (bisonbear2).


---

## Caveats — anecdotes vs measured; conflicts

Percentages for slides: use the **Savings cheat sheet** only. This section is for conflicts and trust labels.

### Anecdotes / self-reported (do not present as universal law)
- RTK: vendor demos claim large Bash-output cuts; JetBrains/Quesma and u/bisonbear2 (Reddit 1v9xjh0: RTK **+5%** total tokens geom. mean, more tool calls) — do not cite 60–90% as proven bill savings.
- codebase-memory-mcp: 10× / 99.2% token claims are author/preprint — verify before deck guarantees.
- Graphify: strong “query instead of grep” claim; code AST map billed as 0 LLM credits in their bench — still measure total session tokens on your harness.
- @anshuc 80–90% then corrected to 40–70%; Plus-quota demo 50% vs 7%.
- Shuttle 45k autocompact buffer; Scott Spence 60% MCP schema cut; Verma ~2–3k/turn; Hasan $74→$11 / ~80% combined.
- Aakash Gupta / Anthropic internal Tool Search 134k→5k (85%) — via VentureBeat, not a peer paper.
- Community “30–50% from `/clear`” (Atticus Li) — attributed, not reproduced here.
- Rulestack / Jo Do: `ENABLE_TOOL_SEARCH=auto:N` on a large window can undo Tool Search (1M ctx, `auto:5` loaded ~40k defs). Prefer unset/`true` when the goal is a small prompt.
- Jev Ultrafast: author ~**25%** lower **runtime** (n=3 pairs, sign-test p=0.25); TypeSafe reqs 22→17; protocol 1,092→101. **Not** a session token-%. No TypeSafe billed $ in their note.
- json-render: constrained catalog JSON vs freeform UI dumps — **no published token-%**; do not invent one.

### Measured / product-backed (still context-specific)
- Anthropic staff: 7+ MCP servers → 67k+ tokens.
- Anthropic blog: mid-session Haiku switch can cost more than staying on Opus (cache rebuild).
- OpenCode DCP and Atlassian: prune-first architecture described with engineering rationale; not a single published % for all users.
- Cursor Dynamic Context Discovery (Jediah Katz, 6 Jan 2026): **46.9%** fewer total agent tokens on MCP-calling runs (product A/B; high variance by MCP count).

### Conflicts to flag on slides
1. **Fresh chat vs long session:** Clear between *unrelated* tasks. For *same* task, warm prompt cache can make one long session cheaper than many cold starts (Reddit/Claude Code threads; Anthropic caching docs).
2. **Subagents save parent context but can raise total tokens** (~7× heavy fan-out). Isolation ≠ cheaper system-wide.
3. **Cheap routing vs cache:** Per-turn routers can *increase* spend. Route at cold boundaries or via subagents with handoff.
4. **Auto-compact:** Some practitioners disable it (noise / lost control). Others compact manually at ~60%. Product auto-compact near 95% is late for quality.
5. **STE100 vs Concise:** STE100 = short *sentences* / clarity; Concise = less narration. STE100 alone does not guarantee fewer tokens.
6. **Starving context backfires:** Under-spec prompts cause more exploration and retries (Atticus Li thesis: optimize cost-per-accepted-change, not tokens-per-message).

---

## Practical real-world examples (house + field)

Concrete tactics agents can apply today. Prefer these over abstract “be efficient” advice.

### P.1 Dense edits, then deterministic format once (house rule — Pong, 2026-09-17)

**Problem:** LLM-generated pretty-printing wastes tokens. Newlines, indentation, and blank lines in model output are tokens in every write, re-read, and diff. Asking the model to “make it look nice” is slow, non-deterministic, and often fights the repo style. Mid-edit format-on-save also causes edit/tool loops (agent writes → editor reformats → agent cannot match lines → more tool calls).

**Do this:**
- During multi-file edits, emit dense patches: minimal indentation / blank lines where the language allows (Python still needs indent for syntax). Do **not** spend model tokens on cosmetic layout.
- Do **not** ask the model to format or “clean up style.”
- After **all** planned edits land, run the project’s **pinned deterministic formatter once** via CLI/tooling — e.g. `prettier`, `rustfmt`, `gofmt`/`gofumpt`, `clang-format`, `black`/`ruff format`, `yapf` — matching the repo config (`.prettierrc`, `rustfmt.toml`, etc.).
- Verify with the same tool (`--check` / exit code) before declaring done.
- Turn off editor format-on-save for agent sessions when it fights the agent (Zed/Codex reports: autoformat drives token-burning loops).

**Why it saves:** Fewer model output tokens per edit; shorter tool payloads; one cheap local format pass instead of N LLM formats; stable diffs; fewer retry cycles from whitespace mismatch.

**Caveat:** Committed code must match the project formatter. Dense mid-edit form is for the agent path only; the final tool pass is the source of truth for style.

### P.2 Dense reads for exploration; pretty originals for edits

**Do this:** When only *understanding* code, strip comments / excess blank lines in the context you feed the model (minify-for-read). When *editing*, work from the real file so line numbers and diffs stay honest.

**Field note:** arXiv “Hidden Cost of Readability” (2025) reports ~24.5% fewer input tokens on average when formatting noise is removed without breaking syntax (self-reported experimental mean — not a guarantee). Skills like code-minification exist for read-only explore; avoid minify for compile errors, stack traces, or review.

### P.3 One formatter handshake in AGENTS.md

**Do this:** One short rule: “Before done: run the project’s deterministic formatter on touched files (CLI), never reformat by hand in the model.” Point at the real config (`.prettierrc`, `rustfmt.toml`, …). Align any AGENTS.md style claims with that config.

**Why it saves:** Stops the 10-line change → 300-line prettier fight → agent rewrite loop. Style stays deterministic and cache-friendly.

### P.4 Patch / hunk edits, not full-file rewrites

**Do this:** Prefer search-replace or unified hunks over rewriting whole files. Point at one file; search before full read.

**Why it saves:** Full-file rewrites re-bill the whole buffer every time.

### P.5 Shell: quiet, short, stop early

**Do this:** Prefer `rg -n` over dumping trees. Cap command output. Stop when evidence is enough. Compress logs before pasting into chat.

**Why it saves:** Tool exhaust is the usual context bloat; prune/compact later is more expensive than never fetching the noise.

### P.6 Narration tax

**Do this:** Lead with the result. No “Certainly…”, no restating the task, no play-by-play of every tool. STE100: short sentences, bullets for parallel items.

**Why it saves:** Decode tokens are billable too; fluff multiplies across turns.

### P.7 Subagent boundary = summary only

**Do this:** Worker does the long read/edit. Parent receives a short result (paths changed, tests, blockers). Do not paste the worker’s full tool log into the parent.

**Why it saves:** Isolates noise; parent context stays cheap. Fan-out without summaries can raise *total* tokens (~7× community caveat).




## Context engineering (PRP workflow)

Template and workflow from [coleam00/Context-Engineering-Intro](https://github.com/coleam00/Context-Engineering-Intro) (~13.8k★). **Substance only** (no social share links). Goal: give the coding agent enough structured context to finish features end to end — most agent failures are **context** failures, not model failures (author framing).

**Author slogan (not a token metric):** “Context Engineering is 10x better than prompt engineering and 100x better than vibe coding.” Treat as advocacy, **not** a savings %.

### Prompt engineering vs context engineering

| | Prompt engineering | Context engineering |
| --- | --- | --- |
| Focus | Clever wording of one ask | Full system: docs, examples, rules, patterns, validation |
| Metaphor (author) | Sticky note | Full screenplay |
| Outcome sought | Better phrasing | Consistency, multi-step features, self-correcting loops |

### Workflow (Claude Code–centered; portable)

1. **Global rules** — maintain a lean `CLAUDE.md` / `AGENTS.md` (project conventions, test/style gates). Same spirit as Pattern 4: phonebook, not a novel.
2. **Examples folder** — put real patterns under `examples/` (structure, tests, CLI, agent/tools). Assistants copy patterns they can see.
3. **Feature ask (`INITIAL.md`)** — fill FEATURE / EXAMPLES / DOCUMENTATION / OTHER CONSIDERATIONS (gotchas AIs usually miss).
4. **`/generate-prp INITIAL.md`** — research codebase + docs → write a **PRP** (Product Requirements Prompt): blueprint with steps, validation gates, tests, confidence score. Lives under `PRPs/`.
5. **`/execute-prp PRPs/….md`** — load PRP → plan → implement → validate → iterate until gates pass.

Repo layout (abridged): `.claude/commands/{generate,execute}-prp.md`, `PRPs/templates/prp_base.md`, `examples/`, `CLAUDE.md`, `INITIAL.md`.

### Token-savings fit (how to use without bloating)

| Do | Why |
| --- | --- |
| Keep **always-on** rules thin; put feature context in the **PRP file** for that task | Avoids Pattern 4 always-on tax every unrelated turn |
| Run **generate-prp** and **execute-prp** in **separate sessions** (or clear between) when research dumps are huge | Same as Plan → fresh implement / Pattern 2 |
| Prefer **examples + validation gates** over pasting whole repos into chat | Targeted exploration (Pattern 8) beats blind dumps |
| Treat completion quality as the win; **measure** tokens/cost | More upfront context can raise one-session tokens while cutting failed retries — trajectory bill, not sticky-note bill |

**Not a substitute for:** MCP hygiene, Explore subagents for noisy search, or prune-first compact (Patterns 1, 7, 10).

**INITIAL.md skeleton** (pasteable): see [EXAMPLE-PROMPTS.md](./EXAMPLE-PROMPTS.md#initialmd-context-engineering-skeleton).

## Example prompts (real-world)

Copy-paste prompts and configs (STE100, Concise, MCP off, `/clear`, compact, cheap leaf, lean AGENTS.md, house format-once).

**Full library:** [EXAMPLE-PROMPTS.md](./EXAMPLE-PROMPTS.md)

**Starters (most used):**
1. House format-once + deterministic CLI formatter (never LLM pretty-print) — see companion §1
2. STE100 / Concise output style — companion §2
3. Disable unused MCP before the session — companion §3
4. `/clear` vs `/compact` decision card — companion §4–5
5. Explore pinned to Haiku / cheap orch + strong leaf — companion §6

## Measured study: five “token saving” modes (u/bisonbear2, 2026)

Summary figures that belong on the deck are also on the **Savings cheat sheet** (e.g. Terra cost-only win, RTK disputed).

**Source:** [r/ClaudeAI — I tested 5 popular token saving methods…](https://www.reddit.com/r/ClaudeAI/comments/1v9xjh0/i_tested_5_popular_token_saving_methods_across_10/) · fuller write-up [stet.sh](https://www.stet.sh/blog/gpt-56-token-saving-modes) · author **u/bisonbear2** (building Stet; disclosed). Snapshot also via [sentinel](https://reddit.sentinel-team.org/posts/1v9xjh0/snapshots/2026-07-30T01%3A30%3A02.254894Z).

**Setup (author-reported):** Codex **5.6 Sol** baseline (medium effort) vs six arms on **10** real merged-task replays from one repo; **2** full repeats → **140** agent runs. Graded tests, equivalence, code review, footprint, eight quality dims. Author notes Codex stack but argues transfer to Claude-style agents. **n=2 repeats** — treat as one practitioner measurement, not a meta-analysis.

| Mode | What it is | Total tokens (geom. mean Δ) | Cost Δ | Saved in both runs? |
| --- | --- | ---: | ---: | --- |
| Caveman | terse “caveman” interaction | −2% | −2% | No |
| Ponytail | lazy senior / YAGNI / min output | +7% | +9% | No |
| RTK | compress shell command output | +5% | +1% | No |
| Context Mode | batch/index retrieval; return judged-relevant slices | +68% | +51% | No |
| Mandarin | denser Chinese prompt translation | +46% | +36% | No |
| Terra xhigh | cheaper model, higher reasoning | +6% | **−49%** | Cost only |

**Headline (author):** none of the six cut total tokens in *both* runs; five *increased* average total tokens. Context Mode worst (+68%). Only consistent **dollar** cut was Terra xhigh (−49% cost) while using slightly *more* tokens.

**Behavior findings (blend into patterns):**
- **Output compression ≠ workload compression.** Local savings (shorter replies, thinner CLI output, cheaper retrieval) often change search / validation / patch policy so the *trajectory* bill rises.
- **RTK (pattern 10):** individual commands smaller; agent issued **more** tools (avg **60** vs baseline **49**) and ~2× wall time — aligns with JetBrains/Quesma “disputed bill savings.”
- **Caveman / Ponytail (pattern 8 STE100/concise):** average near-flat or higher total tokens; quality hits on **robustness** / **scope** (graders). Ponytail cut *output* tokens both runs (−3%, −11%) but total tokens still +16% / −1%.
- **Context Mode:** cheap structured retrieval → agent gathered/revisited *more* context (quantity demanded↑). Commenters: may fit as a **scoped retrieval subagent**, not a global session saver.
- **Mandarin denser language:** not a free token win; changed decomposition/delegation; trajectory cost dominated. Commenter note: English droppable scaffolding ≠ agglutinative languages.
- **Terra / cheap strong-effort model (pattern 6):** wins on **price**, not token count — same family as cheap-leaf / cold-boundary routing.

**Quote:** “The bill is for the whole trajectory, not the local surface.” / “Output compression is not workload compression.”

**Deck use:** cite as **measured caution** against installing “token saver” wrappers as automatic wins. Prefer replay-your-own-tasks. Do not present −49% Terra as fewer tokens.


## Library catalog: llm-engineer-toolkit

Curated index of **120+ LLM libraries** by category — not a token-saver itself. Use it to **discover** tools that map onto this playbook.

**Repo:** [KalyanKS-NLP/llm-engineer-toolkit](https://github.com/KalyanKS-NLP/llm-engineer-toolkit) (~10.8k★, Apache-2.0; refreshed often).

| Token-savings angle (NOTES) | Toolkit categories to open first | Examples listed there (catalog descriptions — verify upstream) |
| --- | --- | --- |
| Cheap-model routing | Application Development → **Routers** | RouteLLM (“save LLM costs… route simpler queries to cheaper models”); OpenRoutiQ |
| Prompt-cache / repeat query cost | Application Development → **Cache** | GPTCache (vendor claim in catalog: “Slash Your LLM API Costs by 10x” — **self-reported**, not our measurement) |
| External memory vs stuffing the window | Application Development → **Memory** | mem0, Letta (MemGPT), Memoripy, Tree Ring Memory, Memobase |
| Meter without `/context` | **LLM Monitoring** | Helicone, Opik, Phoenix, Evidently, **agenttrace** (coding-agent traces/costs/tokens), OrcaReplay, traceAI, AgentOps |
| Short / compressed prompts | **LLM Prompts** | LLMLingua, Selective Context, PCToolkit (prompt compression); DSPy / Promptimizer (optimize, not always shorter) |
| Structured / constrained output (less narration waste) | **LLM Structured Outputs** | Instructor, Outlines, Guidance, LMQL, Jsonformer; generative UI: [json-render](#generative-ui-token-hygiene-json-render) (catalog JSON, **no published token-%**) |
| Multi-API + gateways (routing / fallbacks) | Multi API Access | LiteLLM, Portkey AI Gateway, Bifrost |
| Agents / orch (isolation ≠ cheaper) | **LLM Agents** | CrewAI, LangGraph, AutoGen, Smolagents, … — still apply NOTES Agent Teams / Explore caveats |

**How to use with this pack:** Pick a category above → open the toolkit table → only adopt a library after you meter it (status line / `ccusage` / dashboard). Do **not** copy catalog marketing % onto the Savings cheat sheet unless you have an attributed measurement.

**Related hubs by same curator:** [Prompt Engineering Techniques Hub](https://github.com/KalyanKS-NLP/Prompt-Engineering-Techniques-Hub) · [LLM Survey Papers Collection](https://github.com/KalyanKS-NLP/LLM-Survey-Papers-Collection).

## Generative UI token hygiene (json-render)

**Repo:** [vercel-labs/json-render](https://github.com/vercel-labs/json-render) (~17.8k★ as of 2026-09-21, **Apache-2.0**, TypeScript) · [json-render.dev](https://json-render.dev).

Vercel Labs **Generative UI** framework: the model emits JSON constrained to a **predefined component/action catalog** (Zod schemas). Renderers cover React / Vue / Svelte / Solid / RN / Next / PDF / email / ink and peers — the lesson is the catalog, not the package list.

**Token-savings angle:** Constrained structured UI JSON beats freeform HTML / JSX / markdown UI dumps. The catalog shrinks the decision space. SpecStream streams the spec for progressive render. Same family as toolkit **LLM Structured Outputs** and NOTES prompt hygiene.

**House rule:** Prefer a small catalog + JSON spec over asking the model to write UI code. **No published universal token-%** in the README — do not invent one. Optional skeleton: [EXAMPLE-PROMPTS.md](./EXAMPLE-PROMPTS.md#json-render-catalog-prompt-skeleton). Docs also mention **experimental TypeSafe Jev composition** (unreleased) — [json-render.dev/docs/jev](https://json-render.dev/docs/jev).

## Sources

Attributed quotes, link list, and X primary posts.

**Full list:** [SOURCES.md](./SOURCES.md)

For deck percentages, use the **Savings cheat sheet** above as the single numbers board.


---
**Maintenance (weekly, Monday ~09:00 Europe/London):** Refresh X/web in [SOURCES.md](./SOURCES.md). Re-check metering tools, [TYPESAFE-JEV.md](./TYPESAFE-JEV.md), and [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction). Update the **Savings cheat sheet** only when new attributed figures appear. Keep companions + thin `guides/` in sync (map rows only — no tip duplication). `EXAMPLE-PROMPTS.md`, `ORCH-CASES.md`, `SPEAKER-NOTES.md`. Last edit: 2026-09-21. Source of truth: private GitHub `PongPong/token-savings-notes`.

## Gaps

- **X.com / Twitter primary access:** WebSearch `site:x.com` empty; X search URLs redirect to login. Browser could open individual public posts (17 Sep 2026 pass). Nitter mirrors failed. See **X primary delta** in [SOURCES.md](./SOURCES.md).
- **flaviocopes.com** Concise/STE100 article: Cloudflare-blocked on WebFetch; used secondary gists + explainx + search snippets.
- **OpenCode DCP README:** Live README (21 Sep 2026) readable — related **Sleev** pointer; cache-hit ~85% vs ~90% still not on the live file — do not hard-cite those % from the current README.
- **No independent lab** comparing all patterns head-to-head on one harness. Harness overhead differs (HN table: OpenCode vs Claude Code vs others) — do not over-generalize.
- **STE100 token savings:** Practitioners describe clarity and shorter sentences; almost no hard token meters for STE100 alone.
- **Cursor-specific measured savings:** Cursor blog Dynamic Context Discovery now cited — **46.9%** fewer total agent tokens on MCP-calling runs (product A/B, Jan 2026). Forum evidence for tool-filter need remains; still fewer independent before/after audits than Claude Code blogs.


## Team practice (standing)

All of Pong’s agents (including Engineer) apply useful techniques from this file in day-to-day work: MCP/tool hygiene, lean always-on rules, compact/prune at task boundaries, new session per unrelated task, cheap orchestrator + strong leaf, STE100 short replies, selective subagents, prompt-cache hygiene; dense mid-edit then one deterministic formatter CLI pass (never LLM pretty-print). Prefer measured cost-per-accepted-change. Do not invent savings %.

## Scout policy (standing)

- Prefer **creditable** X/Twitter accounts only (known practitioners, product teams, researchers, established creators). Skip anonymous engagement bait.
- **Not eng-only** — include product, research, ops, education when the tip is about saving LLM/agent tokens.
- **Include Chinese users / CN-language posts** (e.g. 省token、上下文压缩、MCP、Claude Code、Cursor、OpenCode).
- Still: no invented quotes/numbers; mark % claims self-reported.

