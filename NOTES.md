# Token-savings research pack — cut LLM/agent token spend

**For:** Engineer / Pong L presentation  
**Prepared:** 17 Sep 2026 (Europe/London)  
**Method:** WebSearch + WebFetch of practitioner posts, product blogs, GitHub READMEs. X.com often blocked; used mirrors / articles that cite tweets.  
**Rule:** No invented quotes or numbers. % claims marked self-reported where source is personal.

---


## Scout policy (standing)

- Prefer **creditable** X/Twitter accounts only (known practitioners, product teams, researchers, established creators). Skip anonymous engagement bait.
- **Not eng-only** — include product, research, ops, education when the tip is about saving LLM/agent tokens.
- **Include Chinese users / CN-language posts** (e.g. 省token、上下文压缩、MCP、Claude Code、Cursor、OpenCode).
- Still: no invented quotes/numbers; mark % claims self-reported.

## 1. Top patterns (5–10)

### 1. Context compaction (`/compact`, mid-session summarize)
**What people do:** Summarize history on purpose. Keep decisions, active errors, files in scope, constraints. Drop resolved logs and dead tangents.  
**Why it saves:** Full history is resent every turn. A shorter summary cuts the recurring re-read tax.  
**Practice tip:** Compact ~60% utilization (not at 95%). Auto-compact near the ceiling often summarizes already-degraded context.

### 2. Context pruning (drop tool exhaust first; summarize last)
**What people do:** Remove stale tool outputs, duplicate file reads, old errors *before* LLM summarization. OpenCode DCP: dedup + purge + optional compress. Atlassian Rovo Dev: structure-aware prune cascade.  
**Why it saves:** Most bloat is machine output, not user intent. Mechanical prune is free (no extra LLM call). Summary is the fallback.

### 3. Fresh session / `/clear` per unrelated task
**What people do:** New chat when the task changes. Keep related phases in one session. Put durable rules in CLAUDE.md / AGENTS.md.  
**Why it saves:** Stale history rides every turn. Clearing stops paying rent on dead context.  
**Caveat:** Within one task, a warm cache can make one long session cheaper than many cold restarts (see caching).

### 4. MCP / tool hygiene (fewer tools, short schemas, defer load)
**What people do:** Disable unused MCP servers. Consolidate tools (params > many near-duplicate tools). Trim descriptions. Use Tool Search / `defer_loading` so schemas load on demand. Exception: one **high-signal** exploration MCP (e.g. [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) — see pattern 10) can beat many low-signal file tools — still disable everything else you are not using this session.  
**Why it saves:** Tool defs load before you type. Multi-server setups commonly eat tens of thousands of tokens at session start.

### 5. Lean standing instructions (short CLAUDE.md / AGENTS.md; skills on demand)
**What people do:** Keep always-on rules thin (phonebook, not manual). Move domain packs into skills that load when triggered.  
**Why it saves:** Always-on files tax *every* turn (even with cache, still a recurring cost). On-demand skills avoid that baseline.

### 6. Cheap-model routing (small for triage / explore; big for hard steps)
**What people do:** Haiku / Luna / Gemini for search, plans, scaffolding. Opus / Astra / Sonnet for hard coding and irreversible work. Route at session or cold boundaries — not every turn.  
**Why it saves:** Frontier rates on grep/rename/test-scaffold waste money.  
**Caveat:** Mid-session model switches bust prompt cache; uncached cheap can beat cached frontier only if measured.

### 7. Subagents / specialized workers (isolate noise; return summaries)
**Cases:** See **Case studies: one agent vs multi-agent orchestration** (when one agent wins vs cheap orch + strong leaf vs Explore/Haiku).
**What people do:** Spawn workers with clean context for search/research. Parent keeps only the answer. Cheap orchestrator + expensive coding worker is a common Codex pattern (@anshuc).  
**Why it saves (parent):** Exploration junk stays out of the main window.  
**Caveat:** Each subagent re-pays prompts/tools. Anthropic-cited ~7× for heavy fan-out. Use when isolation value > startup cost.

### 8. Short / STE100-style output (cut narration tokens)
**What people do:** Concise output style; “lead with result, no preamble”; ASD-STE100-style short sentences for status.  
**Why it saves:** Output tokens are sequential and often priced higher than input. Less chat wrapper = less decode cost.  
**Caveat:** STE100 is clarity-first; gists note it is not a hard length cap on the whole reply.

### 9. Prompt-caching hygiene (stable prefix; don’t break it)
**What people do:** Static system + tools first. Dynamic stuff in messages. Don’t shuffle tools or swap models mid-session. Cache-safe compaction forks.  
**Why it saves:** Cache reads ~10% of input price. One prefix byte change can force full-price re-read of the whole history.

### 10. Targeted exploration (narrow @files; stop early; shrink shell output)
**What people do:** Point at paths. Prefer indexed / structural search over full-file dumps. Explicit stop rules in AGENTS.md. Concrete tools practitioners use:

- **[RTK](https://github.com/RTK-AI/rtk) (Rust Token Killer)** — wrap shell / CLI output so the agent gets a compressed form (`rtk git status`, hooks rewrite Bash). Use after choosing the right tool; do not let RTK pick the tool. Built-in `Read`/`Grep` often bypass the Bash hook.
- **[tgrep](https://github.com/microsoft/tgrep)** — trigram-indexed grep (client/server) as a **ripgrep/grep replacement** on large trees. Index once (`tgrep serve .`); agents should prefer `tgrep` for repo search when the index is warm (see repo `AGENTS.md`). Speed win on huge monorepos; token win is indirect (faster, tighter search → fewer thrash reads).
- **[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)** — MCP that builds a local code knowledge graph (tree-sitter + optional Hybrid LSP). Structural / graph queries replace dozens of grep+read cycles. Authors claim large token cuts vs file-by-file exploration (see sources; mark self-reported). Prefer this *instead of* blind full-repo greps when installed — still subject to MCP hygiene (don’t pile unrelated MCP servers beside it).

**Why it saves:** Wrong exploration pins junk into the prefix forever. Stopping early and returning only useful hits avoids the re-read tax.

---

## 2. Named sources — handle + link + short quote

| Source | Link | Quote / claim (exact or closely attributed) | Savings |
| --- | --- | --- | --- |
| Claude Academy (official) | https://academy.claude.com/courses/claude-code-101/context-management | “Use `/compact` when you're working on a specific feature… Use `/clear` when you want to start a new feature.” Subagents return “just a summary” to keep primary context clean. | no figure given |
| MindStudio — `/compact` at 60% | https://www.mindstudio.ai/blog/claude-code-compact-command-context-management | “The better approach: compact at around 60% context utilization, before quality starts degrading.” | no figure given |
| Atlassian — Rovo Dev pruning | https://www.atlassian.com/blog/development/rovo-dev-keeps-long-sessions-useful | “structure-aware pruning is the better default… LLM-based compaction still has a place as a more aggressive fallback.” | no figure given |
| OpenCode DCP (GitHub) | https://github.com/opencode-dcp/opencode-dynamic-context-pruning | Prunes stale tool content before LLM send; author notes ~85% cache hit with DCP vs ~90% without (plugin docs / secondary summaries). | cache-hit delta self-reported in docs ecosystem; no universal % savings |
| Thariq Shihipar / Anthropic (via VentureBeat) | https://venturebeat.com/orchestration/claude-code-just-got-updated-with-one-of-the-most-requested-user-features | “Users were documenting setups with 7+ servers consuming 67k+ tokens.” | 67k+ tokens MCP overhead (Anthropic staff, quoted) |
| Aakash Gupta (via VentureBeat, cites X) | same VentureBeat article | “from ~134k to ~5k in Anthropic’s internal testing. That’s an 85% reduction” (Tool Search). | **self-reported / Anthropic internal** via secondary article |
| Scott Spence | https://scottspence.com/posts/optimising-mcp-server-context-usage-in-claude-code | Before: 20 tools / 14,214 tokens; after consolidate: 8 tools / 5,663 tokens. “Reduction: 60% fewer tokens!” | **self-measured** 60% on one MCP server; also saw ~66k–82k MCP tokens with many servers enabled |
| Cursor forum — michael.siebert | https://forum.cursor.com/t/add-the-possibility-to-filter-mcp-tools/76776 | Hit ~40-tool ceiling; many tools “totally irrelevant”; wanted selective enable. | no figure given |
| Nishchya Verma | https://medium.com/@www.nishchyaverma/your-agents-md-might-be-ruining-your-ai-budget-c833d7755f20 | “Typical savings: roughly ~2 – 3k tokens per turn that you no longer pay for when the skill isn’t loaded.” | **self-reported** ~2–3k tokens/turn |
| dcodes @ Shuttle | https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices | Auto-compact buffer example: “Went from 45k tokens… to having 176k tokens of free space.” MCP: “Some MCP servers take 4-10k tokens just sitting there.” `/clear` after 1–3 messages; Gemini for plan, Claude for implement. | **self-reported** 45k→176k free; MCP 4–10k idle |
| Atticus Li | https://atticusli.com/blog/posts/save-tokens-claude-code/ | `/clear` “biggest single lever — community measurements put the per-message savings… at roughly 30–50%.” Subagent-heavy workflows “around 7× the tokens.” Keep CLAUDE.md under ~150 lines. | 30–50% community / 7× Anthropic-noted — treat as attributed, not independently verified here |
| Md Nasimul Hasan | https://saadh393.github.io/blog/balance-ai-optimize-token-usage | Same-task weekends: “$74” → “$11.” “Terse output… cuts output volume by 30 to 50%” (own week log). Combined levers “closer to 80%.” | **self-reported** |
| Anthropic — prompt caching lessons | https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything | “it would actually be more expensive to switch to Haiku than to have Opus answer, because we would need to rebuild the prompt cache for Haiku.” | no universal %; qualitative cost inversion |
| @anshuc (X thread via UnrollNow) | https://unrollnow.com/status/2098014776886448337 | First: “Saves 80-90% of your quota.” Correction: “savings are more like 40-70%. The 80-90% figure includes separate savings from my skill.” Demo: Astra-only ~50% weekly Plus quota vs Luna+Astra 7%. | **self-reported**; author corrected 80–90% → 40–70% |
| Ammar Najjar | https://ammar-najjar.com/blog/local-ai-coding-toolkit/ | “Give the agent the smallest amount of information necessary…” Fresh session for fresh tasks; compact mid-task; stop rules in AGENTS.md. | no figure given |
| ASD-STE100 output styles (gists) | https://gist.github.com/toppa/bf7ff49d6fc44fd4fc3337248f8f2a7e · https://gist.github.com/L1nefeed/4164ecaaf77879e76dca3c06f142f1c2 | “Maximum 20 words… instruction… Maximum 25 words… descriptive.” “Length is not terseness… Clarity is the goal, not concision.” | no token figure given |
| Claude Code Concise (announced Aug 2026; secondary) | https://explainx.ai/blog/claude-code-concise-output-style-config-august-2026 | Concise: “leads with the result, keeps responses short… still gives full detail when you ask.” | no figure given |
| frankchu — router + caching trap | https://dev.to/frankchu/i-built-a-router-to-cut-my-claude-code-bill-and-prompt-caching-was-the-whole-problem-3ifl | Per-step model routing “can pay more with the router than without it.” Lock tier per conversation; downshift only at cold boundaries. | no single %; mechanism claim |
| ofox.ai — Claude usage limits | https://ofox.ai/blog/claude-code-usage-limit-hit-too-fast-2026/ | “Agent teams can use about 7x the tokens… One developer… found 85% of the usage came from subagent-heavy sessions.” | 7× / 85% **anecdotal / attributed** — verify before citing as fact |
| RTK (Rust Token Killer) | https://github.com/RTK-AI/rtk · Ammar Najjar toolkit https://ammar-najjar.com/blog/local-ai-coding-toolkit/ | Compresses shell/CLI output before it hits the agent; “Choose the right tool → run through RTK → return only useful output.” README / demos claim large Bash-output cuts. | Vendor demo claims often 60–90% on *command output*; JetBrains / Quesma benchmarks find little or no **bill** savings (hook misses Read/Grep; ~≤3% input ceiling) — treat as **disputed** |
| microsoft/tgrep | https://github.com/microsoft/tgrep | “Tools like `grep` and `ripgrep` scan every file on every search… tgrep pre-builds a trigram index so searches only touch the small set of files that could match.” Used as grep replacement for agents (repo `AGENTS.md`; Copilot CLI integration). | Latency up to ~52× vs ripgrep on large repos (project benchmarks); no universal token-% claim |
| DeusData/codebase-memory-mcp | https://github.com/DeusData/codebase-memory-mcp · arXiv:2603.27277 | “10× fewer tokens, 2.1× fewer tool calls vs. file-by-file exploration” (31 repos). README: five structural queries “~3,400 tokens vs ~412,000” (**99.2%** reduction claim). | **Self-reported / preprint**; verify on your harness |

---

## 3. Slide bullets (STE100-style)

**Title: Cut agent token spend — what practitioners actually do**

- Context is resent every turn. Junk context is a recurring tax.
- Compact on purpose. Keep decisions, errors, scope. Drop dead logs.
- Prefer prune of tool exhaust before LLM summary.
- Start a new session for a new task. Keep related work together.
- Turn off unused MCP tools. Short schemas beat many near-duplicate tools.
- Prefer on-demand tool search / deferred schemas when available.
- Keep CLAUDE.md / AGENTS.md short. Load skills only when needed.
- Route cheap models to explore and triage. Keep frontier for hard steps.
- Do not switch models mid-warm session unless you accept a cache miss.
- Use subagents to keep noise out of the parent. Do not fan out blindly.
- Ask for short replies. Lead with the result. Skip preambles.
- Point at files. Prefer tgrep / graph MCP / `rg` over full dumps; wrap shell with RTK. Stop when evidence is enough.
- Protect the prompt-cache prefix. Stable tools and rules first.
- Measure with `/context` and cache-hit logs. Anecdotes ≠ your bill.
- Local “token savers” can raise the *trajectory* bill. Measure your own tasks (bisonbear2).

**One-liner ranking for slides:**  
MCP hygiene → clear/fresh sessions → lean standing docs → compact/prune → short output → smart routing → selective subagents → cache hygiene.

---

## 4. Caveats — anecdotes vs measured; conflicts

### Anecdotes / self-reported (do not present as universal law)
- RTK: vendor demos claim large Bash-output cuts; JetBrains/Quesma and u/bisonbear2 (Reddit 1v9xjh0: RTK **+5%** total tokens geom. mean, more tool calls) — do not cite 60–90% as proven bill savings.
- codebase-memory-mcp: 10× / 99.2% token claims are author/preprint — verify before deck guarantees.
- @anshuc 80–90% then corrected to 40–70%; Plus-quota demo 50% vs 7%.
- Shuttle 45k autocompact buffer; Scott Spence 60% MCP schema cut; Verma ~2–3k/turn; Hasan $74→$11 / ~80% combined.
- Aakash Gupta / Anthropic internal Tool Search 134k→5k (85%) — via VentureBeat, not a peer paper.
- Community “30–50% from `/clear`” (Atticus Li) — attributed, not reproduced here.

### Measured / product-backed (still context-specific)
- Anthropic staff: 7+ MCP servers → 67k+ tokens.
- Anthropic blog: mid-session Haiku switch can cost more than staying on Opus (cache rebuild).
- OpenCode DCP and Atlassian: prune-first architecture described with engineering rationale; not a single published % for all users.

### Conflicts to flag on slides
1. **Fresh chat vs long session:** Clear between *unrelated* tasks. For *same* task, warm prompt cache can make one long session cheaper than many cold starts (Reddit/Claude Code threads; Anthropic caching docs).
2. **Subagents save parent context but can raise total tokens** (~7× heavy fan-out). Isolation ≠ cheaper system-wide.
3. **Cheap routing vs cache:** Per-turn routers can *increase* spend. Route at cold boundaries or via subagents with handoff.
4. **Auto-compact:** Some practitioners disable it (noise / lost control). Others compact manually at ~60%. Product auto-compact near 95% is late for quality.
5. **STE100 vs Concise:** STE100 = short *sentences* / clarity; Concise = less narration. STE100 alone does not guarantee fewer tokens.
6. **Starving context backfires:** Under-spec prompts cause more exploration and retries (Atticus Li thesis: optimize cost-per-accepted-change, not tokens-per-message).

---

## 5. Optional — 1-page speaker notes

**Open (30s):** Agents bill for re-reading their own past. Most spend is not “the next clever prompt.” It is history, tools, and retries.

**Pattern walk (3–4 min):**  
1) Show `/context` mental model: system + MCP + memory + messages.  
2) MCP: cite Shihipar 67k+; Spence before/after. Action: disable unused; defer load.  
3) Session: `/clear` between tasks; `/compact` mid-task with a keep-list.  
4) Docs: short AGENTS.md; skills on demand (Verma).  
5) Models: cheap explore / expensive implement; warn about cache bust. Cite Anthropic Haiku-switch example and @anshuc Luna+Astra *as self-reported*.  
6) Subagents: protect parent; watch total fan-out.  
7) Output: Concise / STE100-ish bullets cut decode cost.

**Close (30s):** Stack levers. They multiply. Pick two this week: MCP hygiene + clear between tasks. Log cache hits. Re-check after product updates.

**Do not say on stage:** Exact % savings as guaranteed. Say “practitioners report…” and name the source.

---

## Source list (markdown)

- [Context management · Claude Academy](https://academy.claude.com/courses/claude-code-101/context-management) — undated course page — docs  
- [How to Use the /compact Command… (MindStudio)](https://www.mindstudio.ai/blog/claude-code-compact-command-context-management) — 2 Apr 2026 — blog  
- [Agent Context Pruning · Atlassian Rovo Dev](https://www.atlassian.com/blog/development/rovo-dev-keeps-long-sessions-useful) — 30 Mar 2026 — blog  
- [opencode-dynamic-context-pruning](https://github.com/opencode-dcp/opencode-dynamic-context-pruning) — GitHub  
- [Claude Code MCP Tool Search · VentureBeat](https://venturebeat.com/orchestration/claude-code-just-got-updated-with-one-of-the-most-requested-user-features) — 15 Jan 2026 — news (cites @Thariq / @AakashGupta / Boris Cherny on X)  
- [Optimising MCP Server Context Usage · Scott Spence](https://scottspence.com/posts/optimising-mcp-server-context-usage-in-claude-code) — 30 Sep 2025 — blog  
- [Filter MCP tools · Cursor forum](https://forum.cursor.com/t/add-the-possibility-to-filter-mcp-tools/76776) — Apr–May 2025 — forum  
- [Your AGENTS.md Might Be Ruining Your AI Budget · Nishchya Verma](https://medium.com/@www.nishchyaverma/your-agents-md-might-be-ruining-your-ai-budget-c833d7755f20) — 22 Jun 2026 — blog  
- [Claude Code Best Practices · Shuttle](https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices) — 16 Oct 2025 — blog  
- [How to Save Tokens With Claude Code · Atticus Li](https://atticusli.com/blog/posts/save-tokens-claude-code/) — 3 Jul 2026 — blog  
- [Why AI Coding Agents Cost So Much · Md Nasimul Hasan](https://saadh393.github.io/blog/balance-ai-optimize-token-usage) — 14 May 2026 — blog  
- [Prompt caching is everything · Anthropic / Thariq Shihipar](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything) — 30 Apr 2026 — blog  
- [@anshuc Astra-as-subagent thread (UnrollNow mirror)](https://unrollnow.com/status/2098014776886448337) — ~11 Sep 2026 — X mirror  
- [Local AI Coding Toolkit · Ammar Najjar](https://ammar-najjar.com/blog/local-ai-coding-toolkit/) — 23 Aug 2026 — blog  
- [ASD-STE100 output style gist (toppa)](https://gist.github.com/toppa/bf7ff49d6fc44fd4fc3337248f8f2a7e) — GitHub gist  
- [ASD-STE100 output style gist (L1nefeed)](https://gist.github.com/L1nefeed/4164ecaaf77879e76dca3c06f142f1c2) — GitHub gist  
- [Claude Code Concise output style · explainx.ai](https://explainx.ai/blog/claude-code-concise-output-style-config-august-2026) — Aug 2026 — secondary blog  
- [Router vs prompt caching · frankchu / DEV](https://dev.to/frankchu/i-built-a-router-to-cut-my-claude-code-bill-and-prompt-caching-was-the-whole-problem-3ifl) — DEV blog  
- [Claude usage limit hit too fast · ofox.ai](https://ofox.ai/blog/claude-code-usage-limit-hit-too-fast-2026/) — blog (anecdotal 7× / 85%)  

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




## Example prompts (real-world)

Copy-pasteable practitioner prompts / configs. Not a re-listing of Gaps, the X delta table, or P.1–P.7 prose. Blocks below are installable text not already fenced in this file (short STE100 table quotes in §2 are not repeated as standalone entries).

### Crushing C.O.D.E — ASD-STE100 `output-style.md`
**Source:** https://crushingcode.nisrulz.com/blog/the-lazy-developers-guide-to-token-efficiency/  
**Attribution:** Nishant Srivastava (Crushing C.O.D.E), 12 Sep 2026 — drop as `~/.config/opencode/rules/output-style.md` (wire via OpenCode `instructions`)

```markdown
- Write in ASD-STE100 Simplified Technical English.
- One idea per sentence. Keep sentences under 20 words.
- Use active voice and present tense. No filler, no hedging.
- Use common words with one meaning. No jargon, slang, or idioms.
- Answer the question. Skip preamble, narration, and closing summaries.
- Default to the smallest output that fully solves the task. Expand only when asked.
- Prefer lists over paragraphs for steps and options.
```

### toppa — Claude Code ASD-STE100 output style (full installable)
**Source:** https://gist.github.com/toppa/bf7ff49d6fc44fd4fc3337248f8f2a7e  
**Attribution:** Michael Toppa — place under `.claude/output-styles/` or `~/.claude/output-styles/`, select via `/config`

```markdown
---
name: ASD-STE100
description: Simplified Technical English — one meaning per word, active voice, simple tense, short sentences, small noun clusters.
keep-coding-instructions: true
---

You are an interactive CLI tool that helps users with software engineering tasks.

Write all English in ASD-STE100 Simplified Technical English. STE is a controlled
language. The aerospace industry built it so that a reader who cannot ask a follow-up
question still reads the text one way only. Its rules are countable, so check your
prose against them as you write it.

## Precedence

These rules set the default shape of the English you write. Any more specific
instruction takes precedence on whatever it addresses. This includes an instruction
from the user, from project instructions, from an invoked skill, or from an established
convention in the file you edit. Where the more specific instruction is silent, these
rules apply.

Follow the more specific instruction without comment. Do not cite this style as a
reason to override it. Do not ask permission.

This exception applies to an explicit instruction only. Do not relax these rules
because a topic feels casual or because other prose seems friendlier.

## Never apply these rules to

- Code. This includes identifiers, syntax, and string literals.
- Quoted material. This includes error output, command output, file contents, and
  another person's words. To rewrite a quotation is falsification, not simplification.
- Text where the exact wording carries the meaning. This includes a command to run, an
  API name, a config key, and an exact error string.

## Rules

| Rule | Limit |
| --- | --- |
| Noun clusters | Maximum 3 words stacked as a modifier. Break a longer stack apart and name the relationship. |
| Main clause first | State the subject and the main verb before any qualifier. Move a relative clause to after the main verb where you can. |
| Sentence length | Maximum 20 words for an instruction or a procedure. Maximum 25 words for descriptive text. |
| One instruction per sentence | Do not join two instructions with "and" or "then". |
| Active voice | Use the passive voice in descriptive text only, and only when the actor is unknown or irrelevant. |
| Simple tenses only | Use the infinitive, the imperative, the simple present, the simple past, and the simple future. Use a past participle as an adjective only. Do not use the present perfect, the past perfect, or a compound auxiliary. |
| No `-ing` verb forms | Use an `-ing` word as a technical noun, or as part of one, only. |
| No hedge stacking | Do not chain modal verbs, as in "may have been caused by". State the uncertainty as its own plain sentence: "The cause is not confirmed." |
| One word, one meaning | Use one term for one concept and repeat it. Do not rotate synonyms for the same idea. |
| Plainest available word | Prefer the short common word to the formal or rare word. |
| Define domain terms | Define a term that is not common English at its first use. Do not carry undefined shorthand forward. |
| No ellipsis | Keep the subject, the verb, and the article explicit, even when the sentence reads longer. |
| Paragraphs | One topic. Maximum 6 sentences. |
| Vertical lists | Use a numbered or bulleted list for 3 or more steps or conditions. |

## Project vocabulary

STE permits a project to define its own approved vocabulary of technical nouns and
verbs. A `CONTEXT.md` file at a repository root is that vocabulary.

If the project has a `CONTEXT.md`, use its terms exactly as it defines them, in the
part of speech it defines. Never substitute a synonym for a term it defines. Never use
a word that its `_Avoid_` lines reject. Do not redefine its terms inline, because the
glossary is the definition.

If the project has no `CONTEXT.md`, do not invent one. Do not present any term as
already established. The rules above apply without change: define a term at first use,
prefer the plainest word, and use one term for one concept.

## Length is not terseness

The caps apply to each sentence, not to the response. Clarity is the goal, not
concision. A long answer in short sentences is correct.

Never drop a fact, a condition, a caveat, or a scope qualifier to meet a limit. Split
the sentence instead.
```

### L1nefeed — Claude Code ASD-STE100 output style (alternate full gist)
**Source:** https://gist.github.com/L1nefeed/4164ecaaf77879e76dca3c06f142f1c2  
**Attribution:** L1nefeed — same install path; stronger “lead with the result / no pad” structure rules

```markdown
---
name: ASD-STE100
description: Simplified Technical English for agent output — one meaning per word, active voice, simple tenses, short sentences
keep-coding-instructions: true
---

# ASD-STE100 Output Style (Simplified Technical English)

You write all prose output in Simplified Technical English, adapted from the ASD-STE100 standard (Issue 9). The aerospace industry built this standard so that a reader cannot misread an instruction. Apply the same discipline to everything you write to the user: answers, summaries, status updates, explanations, and instructions.

This file defines an adaptation, not the standard itself. Output that follows this file is "STE style", not certified STE. Example: the STE dictionary does not approve the pronoun "I" (Part 1, rule GR-3). This style permits "I" because you report your own actions to the user. When this file and the standard disagree, follow this file.

## Scope

Every piece of text you output falls into one of these targets. Each target has one rule:

- **Prose you write yourself** (answers, summaries, status updates, explanations, instructions): apply the rules in this file.
- **Code, commands, file paths, identifiers, and error messages**: reproduce them verbatim.
- **Text you quote from files, documentation, or other sources**: reproduce it verbatim.
- **Code comments and commit messages inside a repository**: match the style of the repository.

"Verbatim" means: copy the text exactly, character for character. The rules in this file apply to one target only: the prose you write yourself.

Accuracy always wins over style. Never remove a fact, a condition, a number, or a scope qualifier to make a sentence shorter. If a rule and precision conflict, keep the precision.

## Word rules

- One word, one meaning. Use each word with only one meaning in a response.
- One action, one verb. Pick one verb for an action and use it every time. Do not rotate synonyms.
- Prefer the plain, short, common word over the formal or rare synonym.
- Use these standard verbs consistently:
  - "check" (not: verify, confirm, validate, inspect)
  - "make sure" (not: ensure, guarantee)
  - "start" (not: initiate, launch, commence)
  - "stop" (not: terminate, halt, cease)
  - "use" (not: utilize, leverage, employ)
  - "show" (not: display, present, exhibit)
  - "find" (not: locate, discover, identify)
  - "change" (not: modify, alter, adjust)
  - "remove" (not: eliminate, delete — but keep "delete" when it names the literal operation)
  - "need" (not: require, necessitate)
- Keep necessary technical terms (API names, tool names, domain nouns). Use each one the same way every time. Define a term once if it is not common English.

## Grammar rules

- Use the active voice. Name the actor: "The test writes a temporary file", not "A temporary file is written".
- Passive voice is permitted only in descriptions, and only when the actor is unknown or does not matter.
- Use only simple tenses: simple present, simple past, simple future, infinitive, and imperative.
- Do not use the perfect tenses. Write "I changed the file", not "I have changed the file".
- Do not use auxiliary verb constructions ("would have been", "could be being").
- Use a past participle only as an adjective ("the changed file"), not to build compound tenses.
- Use the imperative for instructions to the user: "Run the tests", not "You should run the tests" or "The tests should be run".
- Avoid "-ing" verb forms where a simple form works: "before you commit", not "before committing".

## Sentence rules

- Maximum 20 words per sentence in instructions and procedures.
- Maximum 25 words per sentence in descriptions and explanations.
- One instruction per sentence. Split "open the file and check line 3" into two sentences.
- Do not omit words to save space. Keep the subject, the verb, and the articles. "The files that are not backed up" is clear; "files not backed up" is not.
- Limit noun clusters to 3 words. Write "the handler that sets task-queue priority", not "the task queue priority handler".
- Start a warning or a safety-critical note with the command or the condition, not with background: "Do not run this on main. It rewrites history." Not: "Because it rewrites history, you may not want to run this on main."

## Structure rules

- One topic per paragraph. Maximum 6 sentences per paragraph.
- Use a numbered list for a sequence of 3 or more steps. Use a bulleted list for 3 or more parallel items or conditions.
- Do not bury a sequence or a set of conditions inside one prose sentence.
- Lead with the result. The first sentence of a response answers the question or states what happened.
- Stop when the content is complete. Do not pad. No introductions, no restatements of the question, no closing summaries that repeat the body.

## Examples

| Not STE style | STE style |
|---|---|
| "I've gone ahead and updated the configuration, which should hopefully resolve the issue you were seeing." | "I updated the configuration. This corrects the error." |
| "The deployment process will be initiated once validation has completed." | "The system starts the deployment after the validation completes." |
| "Files not matching the pattern are skipped." | "The script skips the files that do not match the pattern." |
| "You might want to consider possibly running the migration script." | "Run the migration script." |
| "the user authentication token refresh mechanism" | "the mechanism that refreshes the authentication token" |
```

### Codex / Prettier — AGENTS.md formatter handshake
**Source:** https://aitoolsguidebook.com/en/articles/codex-agent-conflict-with-prettier/  
**Attribution:** AI Productivity Guide Team (AI Tools Guidebook), updated 17 Jun 2026

```text
Run `pnpm exec prettier --write` on every file you touch.
Then `pnpm exec prettier --check` must return 0 before you finish.
```

Longer variant from the same article:

```text
After every file edit, run:
  pnpm exec prettier --write <files-edited>

Confirm: pnpm exec prettier --check <files-edited>  returns 0.
If non-zero, fix and re-run before declaring done.
```

### Claude Concise — CLAUDE.md verbosity workaround (pre–built-in Concise)
**Source:** https://explainx.ai/blog/claude-code-concise-output-style-config-august-2026  
**Attribution:** Yash Thakker / explainx.ai (secondary); author presents this as a typical community CLAUDE.md workaround before Anthropic shipped `"outputStyle": "Concise"` (Aug 2026)

```markdown
## Output style

- Lead with the result. Skip narrating what you're about to do.
- Keep responses short unless I ask for detail.
- No "I'll now..." or "Let me..." preambles.
```

Built-in settings.json equivalent (config, not a prose prompt): `"outputStyle": "Concise"`.

### OpenCode — compaction config snippets (prompt/config examples)
**Source:** https://opencode.ai/v2/docs/compaction  
**Attribution:** OpenCode docs (V2)

Minimal (auto + keep + buffer):

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "compaction": {
    "auto": true,
    "keep": { "tokens": 15000 },
    "buffer": 20000
  }
}
```

Manual-leaning example from the same page:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "compaction": {
    "auto": false,
    "keep": { "tokens": 24000 },
    "buffer": 16000
  }
}
```

Local checkpoint summary shape the docs show the model is asked for (excerpt):

```md
## Objective
Finish the authentication migration.

## Next Move
- Update the callback handler.
- Verify the login flow.
```

### Crushing C.O.D.E — older OpenCode compaction fields
**Source:** https://crushingcode.nisrulz.com/blog/the-opencode-config-that-actually-works-and-ships/  
**Attribution:** Nishant Srivastava, 28 Jul 2026 — older field names (`auto` / `prune` / `reserved`); verify against current OpenCode schema before copy-paste

```json
"compaction": {
  "auto": true,
  "prune": true,
  "reserved": 6400
}
```

### Shuttle (dcodes) — `plan.md` slash-command prompt
**Source:** https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices  
**Attribution:** dcodes (DevRel @ Shuttle), 16 Oct 2025

```text
Search the codebase for the relevant files that might be needed for the given task below and generate a `plan.md` file in the `plans/<plan-name>/plan.md` file. The file must include these sections:

- **Spec**: A high-level overview of the finished task, which can be a description of the final result or an end-user perspective (1-3 paragraphs max).
- **Plan**: What's your approach? How are you trying to solve this prompt or achieve this goal? (1-3 paragraphs)
- **Tasks**: A list of checkboxes that split the feature into multiple smaller tasks that can be executed separately (as many as needed).
- **Context**: A list of relevant file paths and descriptions of what they are responsible for and what they do.
```

### Atticus Li — plan-before-execute (verbatim quote from article)
**Source:** https://atticusli.com/blog/posts/save-tokens-claude-code/  
**Attribution:** Atticus Li, 3 Jul 2026

```text
Search around, figure out what's causing this, and tell me the plan. Don't write any code yet.
```

### (house — Pong) Dense edits + deterministic format once
**Source:** house rule (Pong L, 2026-09-17) — short prompt form of P.1; not previously present as a fenced prompt block

```text
During multi-file edits: emit dense patches (minimal cosmetic whitespace; keep language-required indent). Do not pretty-print or "clean up style" in the model.

After all planned edits land: run the repo's pinned deterministic formatter once via CLI on touched files (prettier / rustfmt / gofmt / black / ruff format / clang-format — match project config). Then `--check` / exit 0 before done.

Never reformat by hand in the model. Prefer patch/hunk edits over full-file rewrites.
```



### More copy-paste prompts (MCP / clear / compact / routing / AGENTS)
From earlier draft — categories not duplicated in the STE100/formatter blocks above. Labels: verbatim vs paraphrased.

### 3. MCP hygiene

- **Label:** paraphrased
- **Source:** dcodes @ Shuttle — “Claude Code Best Practices”, https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices (session habit; blog is prose)
- **Prompt:**
```text
Before this session: disable every MCP server I am not using for this task.
Toggle with @<servername> or remove unused servers.
Prefer project-scoped MCP over global so tools only load when the project needs them.
```
- **When:** Session start; especially after `/context` shows large MCP share.

- **Label:** verbatim
- **Source:** Scott Spence — McPick workflow, https://scottspence.com/posts/optimising-mcp-server-context-usage-in-claude-code
- **Prompt:**
```bash
npx mcpick
# Toggle: enable only MCP servers relevant to the current task
```
- **When:** Before a Claude Code session when third-party MCP servers are bloated and you cannot edit their schemas.

- **Label:** verbatim
- **Source:** PostHog — connect-time scoping + single-exec lesson, https://posthog.com/newsletter/tokenminning
- **Prompt:**
```text
# Scope MCP at connect time (PostHog example pattern):
# features-only:
https://mcp.posthog.com/mcp?features=flags,dashboards
# single tool:
https://mcp.posthog.com/mcp?tools=dashboard-get

# Provider lesson (for MCP authors / config choice):
# Prefer a single exec / deferred schema path over registering every endpoint as a tool.
```
- **When:** Configuring MCP URLs; choosing CLI/exec mode over full tools mode.

- **Label:** verbatim
- **Source:** AINews (citing Claude Code subagent frontmatter), https://www.ainews.tech/blog/route-claude-code-subagents-to-cheaper-models
- **Prompt:**
```yaml
---
name: safe-researcher
description: Read-only research agent, no external service access
disallowedTools: mcp__github, mcp__linear
---
```
- **When:** Custom research subagent that must not pull MCP schemas for unused servers (`mcp__*` strips all MCP tools).

### 4. /clear / new session

- **Label:** paraphrased
- **Source:** Claude Academy — Context management, https://academy.claude.com/courses/claude-code-101/context-management (rule of thumb already summarized in NOTES; here as a pasteable decision card)
- **Prompt:**
```text
/clear — start a new feature / unrelated task (no previous-session bias).
/compact — same feature, need room, keep memory of prior work.
Put durable cross-session facts in CLAUDE.md / AGENTS.md, not chat history.
```
- **When:** End of a feature or task switch.

- **Label:** paraphrased
- **Source:** Atticus Li — “How to Save Tokens With Claude Code…”, https://atticusli.com/blog/posts/save-tokens-claude-code/
- **Prompt:**
```text
Unrelated task → /clear (wipe working context; keep CLAUDE.md).
Same thread, phase done → /compact at a clean checkpoint.
Look first → /context before deciding.
```
- **When:** Zombie-session prevention.

- **Label:** verbatim
- **Source:** Ammar Najjar — “My Local AI Coding Toolkit…”, https://ammar-najjar.com/blog/local-ai-coding-toolkit/
- **Prompt:**
```text
Related work with useful shared context → continue the session
New or substantially different task → start a fresh session
```
```text
Investigate the OAuth token refresh failure in src/auth/.
Identify the cause, make the smallest necessary change,
and verify the build.
```
- **When:** Session boundary policy + specific bounded task prompt (author’s contrast to “Fix the login issue”).

- **Label:** paraphrased
- **Source:** dcodes @ Shuttle, https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices
- **Prompt:**
```text
Chop work into small chunks. After Claude finishes a chunk: /clear and start a new chat.
Default: clear after 1–3 messages (allow more only if the same task is unfinished).
If a plan is fundamentally wrong: copy the plan to markdown, /clear, regenerate without the failed-attempt noise.
```
- **When:** Aggressive context hygiene / failed-plan reset.

### 5. Compact / prune

- **Label:** verbatim
- **Source:** MindStudio — “How to Use the /compact Command…”, https://www.mindstudio.ai/blog/claude-code-compact-command-context-management
- **Prompt:**
```text
/compact Keep the following: current file structure with all modified paths, the decision to use PostgreSQL instead of SQLite and why, the three functions we identified as needing refactoring, and the current error in auth.ts that we haven't fixed yet.
```
- **When:** Manual compact with an explicit keep-list (author recommends ~60% utilization, not at auto-compact ceiling).

- **Label:** paraphrased
- **Source:** MindStudio (same), preservation checklist turned into a reusable template
- **Prompt:**
```text
/compact Keep:
1. Architectural decisions (not obvious from code alone)
2. Active problems / open errors / what we already tried
3. Files in scope + areas deliberately left alone
4. Constraints (runtime, public API, build requirements)
Drop: resolved logs, dead tangents, replaced drafts, stale tool exhaust.

After compact: Summarize where we are and what we're working on next.
```
- **When:** Mid-task compaction with verification turn.

- **Label:** verbatim
- **Source:** Ammar Najjar, https://ammar-najjar.com/blog/local-ai-coding-toolkit/
- **Prompt:**
```text
Compact when the task is still ongoing but the session has grown substantially.
Start a new session when switching to a completely different task.
```
- **When:** Choosing compact vs clear.

### 6. Cheap orch + strong leaf

- **Label:** verbatim
- **Source:** AINews Explore override (Claude Code docs pattern), https://www.ainews.tech/blog/route-claude-code-subagents-to-cheaper-models — save as `.claude/agents/Explore.md` or `~/.claude/agents/Explore.md`
- **Prompt:**
```markdown
---
name: Explore
description: Fast, read-only search agent for locating code. Use for grep-style lookups, file discovery, and codebase search — not modification.
tools: Read, Grep, Glob, Bash
model: haiku
---

You are a fast, read-only code search agent. Locate files, symbols, and
patterns. Report what you find. Do not modify anything.
```
- **When:** Pin Explore to Haiku after v2.1.198 (built-in Explore inherits parent model).

- **Label:** verbatim
- **Source:** same AINews / Claude Code env lever
- **Prompt:**
```bash
CLAUDE_CODE_SUBAGENT_MODEL=haiku claude
```
- **When:** One-off bulk fan-out where every subagent should be cheap; unset afterward (overrides frontmatter pins).

- **Label:** paraphrased
- **Source:** @anshuc X thread via UnrollNow, https://unrollnow.com/status/2098014776886448337 — session orchestration rules (author later corrected savings %; keep the procedure)
- **Prompt:**
```text
Orchestrator = cheap model (glue only). Coding leaf = strong model as subagent.
- Do not fork the thread; always fresh subagent contexts.
- Strong leaf stops after implementation; cheap orch does testing/validation.
- Cheap orch must not write long technical prompts for the leaf — hand the next plan step only; leaf chooses approach.
- Hard planning first on the strong model when needed; then cheap orch drives steps.
```
- **When:** Codex/ChatGPT-style cheap orch + strong coding subagent (Luna+Astra pattern; swap model names to your stack).

- **Label:** verbatim
- **Source:** dcodes @ Shuttle — plan-generation slash command, https://www.shuttle.dev/blog/2025/10/16/claude-code-best-practices
- **Prompt:**
```text
Search the codebase for the relevant files that might be needed for the given task below and generate a `plan.md` file in the `plans/<plan-name>/plan.md` file. The file must include these sections:

- **Spec**: A high-level overview of the finished task, which can be a description of the final result or an end-user perspective (1-3 paragraphs max).
- **Plan**: What's your approach? How are you trying to solve this prompt or achieve this goal? (1-3 paragraphs)
- **Tasks**: A list of checkboxes that split the feature into multiple smaller tasks that can be executed separately (as many as needed).
- **Context**: A list of relevant file paths and descriptions of what they are responsible for and what they do.
```
- **When:** Cheap/plan model gathers context → fresh Claude Code session implements from `plan.md`.

- **Label:** paraphrased
- **Source:** Anthropic / Thariq Shihipar — “Prompt caching is everything”, https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything
- **Prompt:**
```text
Do not switch models mid-session for a “cheap” follow-up (cache rebuild can cost more than staying on the current model).
To use a cheaper model: spawn a subagent / hand-off message instead of /model mid-thread.
Do not add/remove tools mid-session (breaks cache prefix). Prefer defer_loading / tool search stubs over removing tools.
```
- **When:** Cache-safe routing discipline.

### 7. AGENTS.md lean snippets

- **Label:** verbatim
- **Source:** Ammar Najjar — explicit stopping rules in AGENTS.md, https://ammar-najjar.com/blog/local-ai-coding-toolkit/
- **Prompt:**
```text
Stop searching once enough evidence exists.

Do not inspect additional files merely for completeness.

Do not repeat a successful search using another tool.

Do not continue repository exploration after finding the relevant implementation.

After successful verification, stop unless additional verification is justified.
```
- **When:** Always-on AGENTS.md “know when to stop” block.

- **Label:** paraphrased
- **Source:** Ammar Najjar — tool-selection map (same post)
- **Prompt:**
```text
Text             → rg
Code structure   → ast-grep
Code semantics   → Serena

Choose the right tool → run through RTK when wrapping shell → return only useful output.
Locate first, read second. Extract with jq/yq instead of loading whole documents.
```
- **When:** Local toolkit AGENTS.md environment section.

- **Label:** paraphrased
- **Source:** PostHog tokenminning — what AGENTS.md should / should not include, https://posthog.com/newsletter/tokenminning
- **Prompt:**
```markdown
## AGENTS.md include
- Discovery shortcuts (unguessable CLIs, how we lint/test/build)
- Undiscoverable intent (direction of travel, bans, taste/policy)
- Landmines (things agents get wrong until CI/prod)

## AGENTS.md exclude
- Lint-enforced style (camelCase, locale spelling)
- Obvious model defaults (“follow existing patterns”, “write tests”)
- One ls/grep answers (directory trees, package lists, huge reference dumps)
- Human docs (mission statements, contributing etiquette)
```
- **When:** Curating a lean always-on file.

- **Label:** verbatim
- **Source:** tmolavi/lean-agent-skills README — “token-frugal rules” snippet (via search/index of https://github.com/tmolavi/lean-agent-skills); confirm against repo before committing
- **Prompt:**
```markdown
## Context hygiene
- New task = new session; compact/summarize when context grows heavy.
- Read only the files you need; prefer search/grep over reading whole files.
- Never re-read a file that has not changed since you last read it.

## Scope discipline
- Make minimal, surgical changes; do not touch files outside the task.
- For anything beyond a small edit: present a short plan first, then execute.

## Execution
- Run targeted tests, not the whole suite, unless asked.
- If an installed skill covers the task, use it and follow its scripts.
- Keep diffs, comments and answers concise; no boilerplate.
```
- **When:** Starter lean AGENTS.md (~20 lines) for multi-harness installs.

- **Label:** house
- **Source:** NOTES.md Practical P.3 (formatter handshake only)
- **Prompt:**
```markdown
## Formatter handshake
Before done: run the project’s deterministic formatter on touched files (CLI), never reformat by hand in the model.
Config of record: <path to .prettierrc | rustfmt.toml | …>. Align any style claims in this file with that config.
```
- **When:** One-liner AGENTS.md addition tied to category 1.

---

**Gaps / not found as pasteable prompts**
- Crushing C.O.D.E STE100 article: search hit unrelated Kotlin Native post; no STE100/concise prompt retrieved.
- Medium (Nishchya Verma AGENTS.md budget): WebFetch timeout; no new fenced prompt beyond NOTES quotes.
- flaviocopes Concise article: WebFetch timeout; Concise rules covered via wmedia + explainx instead.
- Category 1: no strong public fenced “dense edit then prettier once” prompt found on the fetched blogs — **house** fills that gap.

**Count:** 26 prompts (1: 3 · 2: 5 · 3: 4 · 4: 4 · 5: 3 · 6: 5 · 7: 5)


## Case studies: one agent vs multi-agent orchestration

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

### STE100 bullets (deck)

- One agent wins for sequential, dependent, or tiny work.
- Multi wins when noise would pollute the parent, or work is truly parallel.
- Each subagent re-pays prompts and tools. Isolation is not free.
- Official caution: agent teams ≈ 7× tokens in plan mode.
- Measured caution: small fan-outs used 2.6×–5.9× more input than sequential (Systima).
- Lived caution: one agent per finding paid ~47k startup each; most tokens were boot (issue #77834).
- Cheap orch + strong leaf can save *premium* quota (@anshuc; self-reported 40–70%).
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
| 省token / Haiku explore / 产出多复用少 | CN practitioner + docs | https://juejin.cn/post/7644429323654529024 |

**Conflicts already in NOTES §4:** Subagents save *parent* context but can raise *total* tokens; cheap per-turn routers can raise spend vs cold-boundary handoff.


## Measured study: five “token saving” modes (u/bisonbear2, 2026)

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

## Gaps

- **X.com / Twitter primary access:** WebSearch `site:x.com` empty; X search URLs redirect to login. Browser could open individual public posts (17 Sep 2026 pass). Nitter mirrors failed. See **X primary delta** below.
- **flaviocopes.com** Concise/STE100 article: Cloudflare-blocked on WebFetch; used secondary gists + explainx + search snippets.
- **OpenCode DCP README:** GitHub fetch returned thin stub; relied on search/docs mirrors for cache-hit notes — confirm on live README before hard-citing 85%/90%.
- **No independent lab** comparing all patterns head-to-head on one harness. Harness overhead differs (HN table: OpenCode vs Claude Code vs others) — do not over-generalize.
- **STE100 token savings:** Practitioners describe clarity and shorter sentences; almost no hard token meters for STE100 alone.
- **Cursor-specific measured savings:** Forum evidence for tool-filter need; fewer hard before/after token audits than Claude Code blogs.


## X primary delta (17 Sep 2026 browser pass)

X search URLs all hit login. Individual public posts stayed readable. Nitter fallbacks failed (`nitter.net` connection closed; `nitter.poast.org` DNS fail). New handles/quotes not in the UnrollNow / VentureBeat set:

| Pattern | Handle | Date | Link | Quote | Savings |
| --- | --- | --- | --- | --- | --- |
| Fresh session / `/context` | @DataChaz | 17 May 2026 | https://x.com/DataChaz/status/2055929071733743693 | “Run `/context` in a fresh session and watch the savings roll in 👀” | No personal figure; lists others’ tool claims |
| Claude Code tips | @meta_alchemist | 31 Mar 2026 | https://x.com/meta_alchemist/status/2039065623557501373 | “these tips will reduce your Claude Code token spending by at least 70-80%” | **self-reported** 70–80% |
| Progressive disclosure / Skills | @dani_avila7 | 17 Oct 2025 | https://x.com/dani_avila7/status/1979349357636767986 | “They auto-load only when needed with progressive disclosure and unlimited references.” | none stated |
| Small tasks + `/clear` | @jcfmunoz | 29 Mar 2026 | https://x.com/jcfmunoz/status/2038288124577562714 | “new task? /clear to save 412K tokens” | **self-reported/tool** 412K |
| Cheap submodels via Agent tool | @amydeng_ | 1 Apr 2026 | https://x.com/amydeng_/status/2039436019326759114 | “The `Agent` tool allows the main agent to choose which model to use as the subagent” | none stated |
| MCP → single exec tool | @posthog | Aug 18 (year not visible) | https://x.com/posthog/article/2089766857071051056 | “A single exec tool. This brought our MCP server down from 113,843 tokens to 5,000.” | **self-reported** 113,843 → 5,000 |
| OpenCode DCP | @bcardarella | 16 Dec 2025 | https://x.com/bcardarella/status/2001007670749466651 | “The @openode dynamic context pruning extension is a must-have” | none stated |
| Layered compaction | @himanshustwts | 31 Mar 2026 | https://x.com/himanshustwts/status/2039228053671547072 | “Claude Code is not doing ‘one compaction’. It has three layers…” | none stated |
| Official auto-compact | @bcherny | 11 Oct 2025 | https://x.com/bcherny/status/1977163445205450783 | “We always auto-compacted near 155k tokens so there's enough buffer.” | none; “not to save costs” (reliability) |
| Concise output / routing | @ellen_in_sf | 30 May 2026 | https://x.com/ellen_in_sf/status/2060824482252833074 | “40-60% fewer output tokens on reasoning tasks.” | article reports 40–60% |

**Slide-worthy adds:** @posthog MCP 113k→5k; @jcfmunoz `/clear` 412K; @bcherny auto-compact is reliability not cost; @meta_alchemist 70–80% (self-reported); @ellen_in_sf 40–60% output.

---
**Maintenance:** Weekly refresh by Engineer (Monday ~09:00 Europe/London). Last baseline: 2026-09-17. Practical examples added 2026-09-17 (format-once + dense edits). X primary delta merged 2026-09-17. Example prompts (real-world) section added 2026-09-17; categories 3–7 (MCP/clear/compact/routing/AGENTS) merged same day (Crushing C.O.D.E STE100, toppa/L1nefeed gists, Prettier handshake, Concise CLAUDE.md, OpenCode compaction, Shuttle plan.md, Atticus plan quote, house—Pong). Source of truth: private GitHub `PongPong/token-savings-notes` (mirror: `/workspace/token-scout/NOTES.md`).


## Team practice (standing)

All of Pong’s agents (including Engineer) apply useful techniques from this file in day-to-day work: MCP/tool hygiene, lean always-on rules, compact/prune at task boundaries, new session per unrelated task, cheap orchestrator + strong leaf, STE100 short replies, selective subagents, prompt-cache hygiene; dense mid-edit then one deterministic formatter CLI pass (never LLM pretty-print). Prefer measured cost-per-accepted-change. Do not invent savings %.
