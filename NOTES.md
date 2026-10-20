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
- [Measured study: 12 Claude Code tips, 945 runs (Kensei, 2026)](#measured-study-12-claude-code-tips-945-runs-kensei-2026)
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
| MCP hygiene (disable / consolidate / Tool Search) | Schema cut **~60%** (14k→5.7k); Tool Search **~85%** (134k→5k); idle MCP **4–10k** or **67k+** with many servers; PostHog **113k→5k** single exec; Rulestack **~61k→21k** (~**40k** deferred); Cursor DCD **46.9%** fewer total agent tokens (MCP-calling runs); Command+K schema **~75%** (84-tool, **self-measured**); Agent Planners **171k→~4.8k** (**self-measured**) | Spence **self-measured**; Tool Search **Anthropic internal** via VB; Shuttle / staff / PostHog **self-reported**; Rulestack **self-measured**; Cursor **A/B**; Command+K **self-measured**; Agent Planners **self-measured** | Usually the biggest *easy* win |
| Cursor harness efficiency (system prompt + dynamic built-ins + cache breakpoints + sparse Read nums) | Aggregate **~7%** lower user token cost; static tool-description tokens **−60%**; cold-cache misses **−20%**; cache-read tokens **−1.6%**; system prompt trimmed **~66%** (harness-internal) | Cursor blog 23 Sep 2026 — **product A/B / production** | Shipped in Cursor — not a user toggle. MCP DCD **46.9%** remains the separate MCP-calling figure already on this sheet. |
| Lean AGENTS.md / skills on demand | **~2–3k tokens/turn** when skill not loaded | Verma **self-reported** | Always-on tax every turn |
| `/clear` / new session (unrelated task) | **~30–50%** per-message (community); one case **412k** cleared | Atticus **attributed**; @jcfmunoz **self-reported** | Same task may prefer warm cache |
| `/compact` ~60% util (not 95%) | No universal %; Shuttle autocompact buffer example 45k→176k free | MindStudio tip; Shuttle **self-reported** | Quality lever more than a fixed % |
| Kensei 945-run Claude Code tips (Sonnet 5.5, code-reading questions) | `/clear` before unrelated work: next 3 questions **−47% / −76% / −79%** after 5 / 15 / 30 turns; `/compact` **−39% / −58% / −60%** (compact call counted); 16 KB rules in a skill vs in CLAUDE.md **−31%** per question; Opus 5.5 effort low vs medium **−17%**; Sonnet 5.5 high vs medium **+20%**; subagent, MCP off, caveman: **no clear effect** | Kensei **self-measured**, n=945, 95% CI, [public logs](https://github.com/kenseii1/claude-code-limits-tests) | One Python repo; lookups, not writing code; tokens, not plan meter. [Study](#measured-study-12-claude-code-tips-945-runs-kensei-2026) |
| Context prune before LLM summary (DCP etc.) | No universal %; cache-hit notes ~85% vs ~90% | DCP docs ecosystem | Free mechanical prune first |
| fast-jev-compaction (Jev keep/drop tools) | No universal token-% in README; reports **char** `reductionRatio`; fallback to LLM summary if too little cut | [tamaratran/fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) README; LiteLLM TypeSafe compaction blog | Verbatim prune — not a summary; measure incl. Jev cost |
| STE100 / Concise / terse output | **No STE100 %**; terse output **30–50%** of *output*; Concise article **40–60%** output | Hasan / @ellen_in_sf **self-reported**; STE100 gists **no figure** | Cuts narration, not whole bill |
| Cheap orch + strong leaf | Quota **40–70%** (was 80–90%); demo **50%→7%** weekly Plus | @anshuc **self-reported**, author-corrected | Premium-quota shape |
| Astra plan/review + Sol impl (3–5 Codex threads) | **No token-%** (author: concurrency, not a benchmark) | @BradGroux **author practice** | Workflow and less role-switching. Fan-out may raise total tokens. [Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux) |
| Haiku/cheap Explore leaf (vs inherit Opus) | **~37%** fewer metered tokens (one pair) | Systima **measured** n=1 | Pin leaf model |
| Haiku 5.5 short-prompt tier (API billing) | Stay ≤100K with `/autocompact 100k` (per-model, incl. Haiku subagents). Prices $0.10/$0.50 vs $0.50/$2.50 over 100K. Anthropic **~75%** avg vs Haiku 4.5 / **90%** list ≤100K — **vendor** | Anthropic launch + pricing + CHANGELOG 2.1.293; tip @lydiahallie (**staff**) | Not a universal bill %; prompt length counts cache reads and writes |
| Cheap plan → clear → strong implement | No single %; Terra-style **−49% cost** / **+6% tokens** | Shuttle; bisonbear2 **measured** | Price ≠ fewer tokens |
| Prompt-cache hygiene | Cache reads ~**10%** of input price (Anthropic); mid-session model switch can **raise** cost; OpenAI up to **90%** off cached input (**vendor**); customer-reported GitHub **>50%** less fresh processing / Manus **85%→90%+** hit / unnamed **~20%** cost; Sonnet 5.5 cache reads **$0.10**/M (was $0.20) → ~**20%** cheaper most agentic (**vendor**, 7 Oct) | Anthropic; OpenAI 22 Sep 2026 (**vendor** + **customer-reported**); Anthropic 7 Oct 2026 **vendor** | Protect prefix; OpenAI 30-min window; changing reasoning effort no longer breaks cache. Sonnet cache-read cut is **vendor**, not a universal bill % |
| Append-only history + batch pruning (Asana browser agent) | Same model (Model B): **29×** lower cost per run, **4×** faster vs original production setup; on GPT-6.1 Sol: **76×** lower, **5×** faster, **89%** of input from cache, $0.47/run | Asana engineering + OpenAI story — **company-measured** (144 runs + 12-run follow-up; 3 runs per condition) | The 76× also includes a model change. Caching history *without* batch pruning cost more on 3 of 4 models. [Pattern 5](#top-patterns-by-savings-impact) |
| Shared cached first request + fewer calls + model/effort A/B (Cloudflare AI code reviewer) | **$2.43 → $0.77** per review (**−68%**); median **4m 09s → 1m 32s**; code changes **$0.65** and model/effort **$1.06** of the $1.67 drop; first-request cache hit was **14–26%** before the fix | [Cloudflare blog](https://blog.cloudflare.com/obsessive-inference-optimization/) 19 Oct 2026 — **company-measured** (before/after per release; authors: not a controlled experiment) | Medium effort lost real findings on some reviewers. Check quality at the effort you pay for. [Pattern 5](#top-patterns-by-savings-impact) |
| Parent + Explore (summary only) | No %; parent context win | Official docs | Total tokens may still rise |
| Dense edit + format-once CLI | No public % | House rule | Stops format loops |
| RTK (shell compress) | Vendor **60–90%** of *command output*; bill often **~0 / +5%** | Vendor vs JetBrains/Quesma/bisonbear2 **disputed** | Don’t cite 60–90% as bill |
| codebase-memory-mcp | Author **10×** / **~99%** vs file-by-file explore | Preprint / README **self-reported** | Verify on your harness |
| Graphify (knowledge graph skill) | Query graph instead of grepping; code AST map uses **0** LLM credits (README benchmark table) | Graphify README / BENCHMARKS — **no total-session token %** | Prefer when docs+code; compare vs codebase-memory-mcp below |
| Jev Ultrafast (indexed DOM + Jev) | Author eval **n=3 pairs** (same Mercury helper): median **9.450 s → 7.092 s** (~**25%** lower **runtime**); TypeSafe reqs **22 → 17**; browser protocol **1,092 → 101**. Demo **7.073 s**. **No TypeSafe billed $ / no universal session token-%** | [jev-ultrafast `docs/performance.md`](https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md) **author / small-n** (sign-test p=0.25) | Runtime & request counts — **not** a token-% |
| tgrep vs rg | Up to **~52×** faster (latency) | microsoft/tgrep benches | Token win is indirect |
| Agent Teams / heavy subagent fan-out | **~7× more** tokens; small fan-out **2.6×–5.9× more** | Anthropic **official**; Systima **measured** | Savings = *avoid* this |
| Fresh session + transcript recall (funes) vs keep / handoff / compact | **3–8×** fewer weighted tokens than keep-context; compact **5–8×** recall cost; handoff ≈ keep (n=3 tasks) | Corvoysier HF **self-measured** | Trap-knowledge tasks; author’s funes tool; one model (claude-opus-4-8) |
| Effort tier (max vs xhigh) | Sonnet 5.5 max **68** / **$14.19**; Sonnet 5.5 xhigh **63** / **$3.33**; GPT-6.1 Sol xhigh **63** / **$1.04**; Sol medium **61** / **$0.70** (per task) | Artificial Analysis Coding Agent Index ~2 Oct 2026 — **third-party measured** (API list price) | Raw $/task + points — no derived % |
| Stacked levers (MCP + clear + terse + …) | Combined “**~80%**” / $74→$11 weekends | Hasan **self-reported** | Not transferable as a guarantee |

**Footer:** Isolation ≠ cheaper system-wide. Measure with a status line + `ccusage` (or your dashboard). Use `/context` only for a one-shot peek.

## Top patterns (by savings impact)
Sorted by **rough savings impact** (high → low). Stars = impact estimate from attributed NOTES evidence (not a lab score). Say “practitioners report…” — do not guarantee %.

### 1. MCP / tool hygiene (fewer tools, short schemas, defer load) — ★★★★★ (5/5)
**Impact (why 5/5):** Largest easy win: schema cuts ~60% (Spence), Tool Search ~85% internal, idle MCP tens of k, Cursor DCD **46.9%** on MCP-calling runs (product A/B). Host-side retrieval and server-side discovery figures are on the cheat sheet (**self-measured**).
**What people do:** Disable unused MCP servers. Consolidate tools (params > many near-duplicate tools). Trim descriptions. Use Tool Search / `defer_loading` so schemas load on demand. Exception: one **high-signal** exploration MCP (e.g. [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) or [Graphify](https://github.com/Graphify-Labs/graphify) — see Targeted exploration) can beat many low-signal file tools — still disable everything else you are not using this session.
**Claude Code `ENABLE_TOOL_SEARCH`:** Leave Tool Search on for first-party hosts (unset or `true`). `auto` / `auto:N` thresholds against **context-window size**, not “keep my prompt small.” On 1M ctx, 5% = 50k, so ~40k defs load upfront and undo the deferral (Rulestack **self-measured**, v2.1.263, 2026-09-09: unset **20,819** vs `false` **60,989** vs `auto:5` **62,319**; ~**40,170** deferred / 88 defs). Claude Code may disable Tool Search on non-first-party `ANTHROPIC_BASE_URL` / older models.
**Claude Code `alwaysLoad: false`:** Per MCP server, defers all of that server’s tools behind tool search (changelog 2.1.287). Pair with Tool Search / MCP hygiene.
**Host-side per-turn retrieval (distinct from Tool Search / `alwaysLoad`):** [Command+K](https://commandplusk.com/blog/too-many-mcp-tools-per-turn-retrieval) (9 Oct 2026) embeds the tool catalogue and, above a 24-tool threshold, sends a top 20 plus pins (use-case, identity, recently used), failing open to the full set. The schema-token result on that deployment is on the cheat sheet (**self-measured**).
**Server-side discovery:** [Agent Planners](https://www.agentplanners.com/blog/mcp-server-too-many-tools-on-demand-discovery) (2 Oct 2026) exposes search / describe / call instead of a full tools list above ~100 tools. Before/after is on the cheat sheet (**self-measured**).
**MCP description cap (Claude Code 2.1.296):** Default limit on MCP tool descriptions sent up front and on MCP server instructions is now **4,096** characters (was 2,048). Still trim. Do not treat the raise as free — it can increase upfront schema tokens.
**ToolSearch cache-miss fix (Claude Code 2.1.302):** Changelog fix for a prompt cache miss when a tool added mid-session was loaded again through ToolSearch or after compaction. If you use Tool Search, update. No token-%.
**Cursor Dynamic Context Discovery** ([product blog](https://cursor.com/blog/dynamic-context-discovery), Jediah Katz, 6 Jan 2026): sync MCP tool descriptions to folders; agent gets names then looks up schemas. A/B on runs that called an MCP tool: **46.9%** fewer total agent tokens (statistically significant; high variance by MCP count). Same post: long tool outputs → files; chat history as files for summarization recovery; Agent Skills; terminal sessions as files.
**Why it saves:** Tool defs load before you type. Multi-server setups commonly eat tens of thousands of tokens at session start.


### 2. Fresh session / `/clear` per unrelated task — ★★★★☆ (4/5)
**Impact (why 4/5):** Community ~30–50% per-message; stops dead-context rent. Caveat: same task may prefer warm cache.
**What people do:** New chat when the task changes. Keep related phases in one session. Put durable rules in CLAUDE.md / AGENTS.md.  
**Why it saves:** Stale history rides every turn. Clearing stops paying rent on dead context.  
**Caveat:** Within one task, a warm cache can make one long session cheaper than many cold restarts (see caching).
**Measured:** In the [Kensei 945-run study](#measured-study-12-claude-code-tips-945-runs-kensei-2026), `/clear` before unrelated work was the biggest cut. `/compact` was next when the next task builds on the last. Figures on the cheat sheet.
**Resume warning (Claude Code 2.1.304, Pro/Max):** Claude Code now warns when resuming an inactive conversation would use a noticeable share of your 5-hour limit. Choose resume, or start a new conversation; Claude can look up the old one if you refer to it. **Vendor — no %.**


### 3. Cheap-model routing (small for triage / explore; big for hard steps) — ★★★★☆ (4/5)
**Impact (why 4/5):** Quota/price lever (anshuc 40–70% self-reported; Terra −49% cost). Cold-boundary only.
**What people do:** Haiku / Luna / Gemini for search, plans, scaffolding. Opus / Astra / Sonnet for hard coding and irreversible work. Route at session or cold boundaries — not every turn.  
**Codex variant (Brad Groux):** Pin Astra (up to Extra High) to plan, review, and coordination. Hand scoped implementation to GPT-5.6 Sol (generally Medium), in 3–5 independent threads. **No token-%.** Full case: [ORCH-CASES Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux). Fan-out can still raise total tokens.  
**Why it saves:** Frontier rates on grep/rename/test-scaffold waste money.  
**Effort tier (Artificial Analysis Coding Agent Index, ~2 Oct 2026):** Effort and harness move cost per task a lot; max is a premium. Third-party measured (API list price): Sonnet 5.5 max **68** / **$14.19**; Sonnet 5.5 xhigh **63** / **$3.33**; GPT-6.1 Sol xhigh **63** / **$1.04**; Sol medium **61** / **$0.70**. No derived %. Via [vibecoding.tech mirror](https://vibecoding.tech/news/2026/10/02/codex-claude-coding-agent-cost) (X blocked); methodology [artificialanalysis.ai/agents/coding-agents](https://artificialanalysis.ai/agents/coding-agents).
**Haiku 5.5 short-prompt tier (API billing, 7 Oct 2026):** Default Haiku on the Anthropic API is `claude-haiku-5-5`. A request over 100,000 input tokens — cache reads and cache writes included — pays the higher tier. On API billing, `/model haiku` then `/autocompact 100k` (saved per model, including Haiku subagents) stays in the cheaper tier. Vendor average and list comparison vs Haiku 4.5 are on the cheat sheet. Staff tip: [@lydiahallie](https://x.com/lydiahallie/status/2107898889508700209).
**Production model and effort A/B ([Cloudflare](https://blog.cloudflare.com/obsessive-inference-optimization/), 19 Oct 2026):** Split each reviewer 50/50 between two models or two effort levels, then compare cost *and* findings. Sonnet 5.5 cost 47–62% less per run than Sonnet 5 at the same per-token price, so the cut came from fewer tokens. On code quality, high effort cost 61% more per run than medium and posted 58% more serious findings; medium lost real findings. **Company-measured**, short windows. Headline figures on the cheat sheet.
**Catalog:** Routers in [llm-engineer-toolkit](https://github.com/KalyanKS-NLP/llm-engineer-toolkit) (e.g. RouteLLM) — verify cost claims upstream.
**Caveat:** Mid-session model switches bust prompt cache; uncached cheap can beat cached frontier only if measured.


### 4. Lean standing instructions (short CLAUDE.md / AGENTS.md; skills on demand) — ★★★★☆ (4/5)
**Impact (why 4/5):** ~2–3k tokens/turn avoided when skills stay unloaded (Verma self-reported).
**What people do:** Keep always-on rules thin (phonebook, not manual). Move domain packs into skills that load when triggered.  
**Why it saves:** Always-on files tax *every* turn (even with cache, still a recurring cost). On-demand skills avoid that baseline.
**Related:** Feature-specific context belongs in a **PRP / INITIAL.md**, not in always-on CLAUDE.md — see **Context engineering (PRP workflow)**.
**Measured:** The [Kensei study](#measured-study-12-claude-code-tips-945-runs-kensei-2026) moved 16 KB of unneeded rules from CLAUDE.md into a skill with a one-line pointer. Figure on the cheat sheet.
**Split, don’t delete ([SwiftLee](https://www.avanderlee.com/ai-development/reduce-token-usage-claude-code-codex-cursor/), Antoine van der Lee, 28 Sep 2026):** Move scoped knowledge into sub-folder `AGENTS.md` files or docs the agent opens only when needed. Author: RocketTrace root `AGENTS.md` 8,677 → 1,454 bytes. He also saw a Codex log where a project `AGENTS.md` passed its 32 KB budget and the rest was truncated without notice. Keep one skill per framework; duplicate skills load twice.


### 5. Prompt-caching hygiene (stable prefix; don’t break it) — ★★★★☆ (4/5)
**Impact (why 4/5):** Cache reads ~10% of input price; one bust forces full-price re-read.
**What people do:** Static system + tools first. Dynamic stuff in messages. Don’t shuffle tools or swap models mid-session. Cache-safe compaction forks.  
**Why it saves:** Cache reads ~10% of input price. One prefix byte change can force full-price re-read of the whole history.
**Sonnet 5.5 cache reads (7 Oct 2026):** Anthropic cut the cache-read price. Claude Code 2.1.296 prices `/cost`, the status line, `--max-budget-usd`, and the SDK at the new rate. The vendor note for most agentic tasks is on the cheat sheet (**vendor**). [Launch](https://www.anthropic.com/claude-haiku-5-5).
**OpenAI (GPT-6+, 22 Sep 2026):** Up to **90%** off cached input tokens (**vendor**); eligible shared prefixes reused within a **30-minute** window; changing reasoning effort no longer breaks cache. Use an explicit breakpoint after the stable prefix (`prompt_cache_breakpoint` / `prompt_cache_options.mode: explicit`); change effort via an appended update. Check misses with [cache diagnostics](https://developers.openai.com/api/docs/guides/prompt-caching/diagnostics) (`tools_changed`, `input_changed`, …). Customer-reported: GitHub **>50%** less fresh prompt processing; Manus hit rate **85%→90%+**; unnamed **~20%** cost cut — not a universal bill %. [Announcement](https://openai.com/index/better-prompt-caching-for-gpt-6).
**Codex vendor-side cache reuse (Tibo, OpenAI, 16 Oct 2026):** OpenAI says Codex & Work now reuse context across subagents, keep context cached longer, and route requests to increase cache hits. This is server-side, not a user setting. **No %.** Keep subagent and session prefixes stable so the cache can hit. [Post](https://x.com/thsottiaux/status/2111244596131111253).
**Claude Code 1h cache and idle sessions:** Idle compaction (Pattern 6) runs before the one-hour prompt cache expires.
**Append-only history, batch pruning ([Asana](https://asana.com/inside-asana/cut-browsers-agent-cost), Oct 2026):** Asana’s browser agent cached its tools and system prompt, but not its growing history. It also removed the last screenshot at every step, so the cached prefix broke on almost every call. Fix: put a cache marker on the latest tool result. Keep history append-only. Prune screenshots in batches (keep up to 20, then cut to 1). Raise the history budget so old text is not trimmed. Caching history without batch pruning cost more on 3 of 4 models. Keep caps on steps, tokens and cost per run. Figures on the cheat sheet (**company-measured**). [OpenAI story](https://openai.com/index/asana-browser-agent) · pointer [@daniel_mac8](https://x.com/daniel_mac8/status/2109079519252361651).
**Gateway session affinity ([Vercel AI Gateway](https://vercel.com/changelog/ai-gateway-logs-now-show-session-affinity), 18 Oct 2026):** A gateway can send turn 2 to a different provider than turn 1. That provider has no cache for your prompt. Vercel now keeps each turn on the provider that served the last one (default for text and chat). Send the same `x-session-id` header, or `providerOptions.gateway.sessionId`, on every request in a conversation. Without one, it groups by a fingerprint of the conversation’s opening. On other gateways, pin a conversation to one provider. **Vendor — no %.** Pointer [@russano](https://x.com/russano/status/2111605981075411125).
**Byte-stable prefix, volatile suffix ([ClawCodex](https://github.com/agentforce314/clawcodex), MIT, Python rebuild of Claude Code):** On DeepSeek, it keeps `system + tools + history` byte-identical across turns. It moves per-request parts (environment, the mutable `MEMORY.md` body, plan mode) to a block after the history ([FAQ](https://clawcodex.app/faq.html)). Then the cached prefix does not break. The same layout works on any provider with prefix caching. Its “×-cheaper” figures are price-sheet math, not a measured bill cut.
**Check the first request ([Cloudflare AI code reviewer](https://blog.cloudflare.com/obsessive-inference-optimization/), 19 Oct 2026):** Every review began with the same 11k–21k tokens of tools and agent prompt, but the first call rarely hit the cache. Cause: OpenCode v1 put the agent prompt and an environment section (working directory, date) in one block, with the only breakpoint at its end. Fix: a marker at the end of the shared prompt and a hook that splits the block, so the shared half gets its own breakpoint. Keep repo-specific parts (skill list, changing tool descriptions) out of that half. Three parallel reviewers send an identical first request: one writes the cache, the other two wait (up to 20 s) for its response to start, then read. Also: put files every session reads by tool call into the prompt (one reviewer: 15.9 → 9.6 calls per run); remove tools a subagent cannot use; read files in parallel; skip reviewers with nothing new on re-reviews; stop after the result is posted (no extra summary turn). Figures on the cheat sheet (**company-measured**). Pointer [@Cloudflare](https://x.com/Cloudflare/status/2112280767308918908).
**Cache lifetime per role (Claude Code 2.1.243, late catch):** API-key and cloud-provider users can set `promptCacheTtl` and `subagentPromptCacheTtl`. Keep the 1-hour cache on the main conversation and 5 minutes on subagents. 1-hour writes cost more than 5-minute writes, so use them only where you return within the hour. 2.1.304 also fixed self-paced `/loop` wakeups at the 1-hour maximum firing just after the cache expired. **Vendor — no %.** Pointer [@colerepo](https://x.com/colerepo/status/2112237172963078599).
**Catalog:** semantic query cache (e.g. GPTCache) under Cache in [llm-engineer-toolkit](https://github.com/KalyanKS-NLP/llm-engineer-toolkit) — separate from Anthropic prompt-cache prefix hygiene; vendor 10× claim is **self-reported**.


### 6. Context compaction (`/compact`, mid-session summarize) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Cuts recurring history tax; no universal %. Compact ~60% util.
**What people do:** Summarize history on purpose. Keep decisions, active errors, files in scope, constraints. Drop resolved logs and dead tangents.  
**Why it saves:** Full history is resent every turn. A shorter summary cuts the recurring re-read tax.  
**Practice tip:** Compact ~60% utilization (not at 95%). Auto-compact near the ceiling often summarizes already-degraded context.  
**Claude Code 1M default (2.1.287+):** On Bedrock / Vertex / Foundry / Claude apps gateway, Opus 4.7+ and Fable use **1M** context by default (no `[1m]` suffix). Opt out: `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` (keeps 200K) or `/autocompact 200k`. From 2.1.288, `/autocompact` window is saved **per model**. 宝玉 (@dotey, ~10 Sep, via mirror) recommends disabling 1M especially with Fable 5.1 — **self-reported, no %**.
**Haiku API `/autocompact 100k`:** On API billing, set the Haiku 5.5 window to 100K so that model stays in the short-prompt tier (Pattern 3). Saved per model, including Haiku subagents.
**Subagent `autoCompactWindow` (2.1.296):** Set `autoCompactWindow` in subagent frontmatter or an `--agents` definition so the subagent can auto-compact earlier than the parent conversation. No token-%.
**Idle compaction (Claude Code 2.1.299; `/config` row 2.1.302):** In the terminal, a 200K+ token conversation left idle for about 54 minutes is compacted before its one-hour prompt cache expires. Your next message then reprocesses the summary, not the full history. Change the 200K minimum with `CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS`. Turn it off with `"idleCompaction": false` in settings.json or the `Idle compaction` row in `/config`. Trade-off (Lydia Hallie, Anthropic): Claude then continues from a summary, not your full context. Keep it on for cost. Turn it off for long debug sessions where detail matters. **Vendor feature — no token-%.**
**Compaction action-order (2.1.293, quality):** Changelog fix for treating actions from just before compaction as already finished, then retracting or redoing that work. No token-%.
**`CLAUDE_CODE_AUTO_COMPACT_WINDOW` suffix fix (2.1.304):** Before 2.1.304, `=200k` silently set a **100K** window. The variable now takes `k` and `M` suffixes like `/autocompact`, and warns when a value is out of range or ignored. If you set it, update and re-check the window. Compaction now re-sends the skill list; `/reload-skills` sends only new or changed skills. **Vendor — no %.**
**Caveat (Corvoysier HF, 1 Oct 2026):** On trap-knowledge tasks, compaction can cost **5–8×** more weighted tokens than transcript recall, and may drop the numbers you need. Prefer fresh session + recall tool when measured better than keep/handoff/compact (cheat sheet).
**Prefer prune-before-summary when you can:** LLM summaries are lossy (paths, errors, constraints vanish). For tool-heavy sessions, prefer Pattern 7 / [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (Jev keep/drop of tool calls — verbatim text) over a summary-only `/compact`. See [TYPESAFE-JEV.md](./TYPESAFE-JEV.md#use-case-verbatim-context-compaction-fast-jev-compaction).


### 7. Context pruning (drop tool exhaust first; summarize last) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Mechanical prune is free; Jev-guided prune costs a cheap decision call; no single published %.
**What people do:** Remove stale tool outputs, duplicate file reads, old errors *before* LLM summarization. OpenCode DCP: dedup + purge + optional compress. Live DCP README still documents the plugin; it now points at related project **[Sleev](https://sleev.ai)** (`sleev` CLI) — local proxy for Claude Code, Codex, OpenCode. DCP still works; Sleev is a related successor multi-harness proxy. No published savings % here — do not deep-dive without measuring. Atlassian Rovo Dev: structure-aware prune cascade.  
**Concrete tool — [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction)** (`tamaratran/fast-jev-compaction`): npm library + Claude Code plugin. Uses **TypeSafe Jev** to score each historical tool call/result (`noul`: keep call? keep result verbatim?). Actions: keep both, keep call + truncate result (`truncateHeadChars`, default 300), or drop call+result. **Never rewrites** user/assistant text — only deletes/truncates tools. Pins first message + newest `preserveRecentMessages` (default 6). Claude Code hook replaces built-in `/compact` summary when reduction is enough; else falls back to summary. Needs `TYPESAFE_API_KEY`; function hooks flag for Claude Code 2.1.274+. Related: LiteLLM’s TypeSafe Jev compaction guardrail (drop stale tool results before the model call).  
**Why it saves:** Most bloat is machine output, not user intent. Mechanical prune is free (no extra LLM call). Jev prune is cheap vs a frontier summarize pass and keeps exact paths/errors. Summary is the fallback.  
**Caveat:** README publishes `reductionRatio` (chars), not a universal token-%. Include Jev’s own cost when measuring. Do not invent a deck %.

**Research pointer — FOCUS** ([arXiv 2609.37590](https://arxiv.org/abs/2609.37590), 29 Sep 2026): training-free decision-preserving context compression; authors report up to **48%** less peak context (and 73% dependency) with up to +8.9 pp task success — **research self-reported**; peak context ≠ bill; no shipped CLI tool. Not on the cheat sheet.
**Research pointer — UT Austin, “Beyond Token Savings”** ([arXiv 2609.32961](https://arxiv.org/abs/2609.32961), 26 Sep 2026): on Terminal-Bench with Qwen, policies using roughly one-third as many tokens can take **20–80%** longer than the uncompressed agent. **Research self-reported**; measure latency and cost, not tokens alone. Pair with FOCUS. A Copilot production figure in the intro is a **third-party cite inside the paper**, not our measurement (quote in [SOURCES](./SOURCES.md)). Not on the cheat sheet. No shipped lever.
**Research pointer — “How Do AI Agents Spend Your Money?”** ([arXiv 2604.22750](https://arxiv.org/abs/2604.22750), Bai et al., University of Michigan, Stanford and others; first posted 24 Apr, v3 2 Oct 2026): OpenHands on SWE-bench Verified, eight models, four runs per task. Input tokens, mostly cache reads, drive cost even with caching. Abstract: runs on the same task differ by up to 30× in total tokens; on average the most expensive run is about 2× the cheapest. Accuracy peaks at intermediate cost. Expensive runs re-view and re-edit the same files. Models underestimate their own token use. **Research self-reported**, older models (Sonnet 4.5, GPT-5 era). Use: treat repeated reads or edits of one file as a stop signal; cap runs. Not on the cheat sheet. CN pointer [@byCanen](https://x.com/byCanen/status/2112396976046182427).
**Search results: passages, not pages ([Exa 3.0](https://exa.ai/blog/exa-3), 19 Oct 2026):** Dynamic Highlights picks passages across all results. It gives more to strong pages and drops repeated facts. Set `dynamic: true`; the launch says GA, but the [docs](https://exa.ai/docs/search/highlights) still list a beta header, so check. Exa: 95% fewer tokens than full page text (**vendor-measured**). [Artificial Analysis](https://x.com/ArtificialAnlys/status/2112237553059582092): 157k vs 299k total tokens per task against standard highlights, but 48% slower per task (**third-party measured**, one model and harness). Same idea for any search tool: return passages, not whole pages. Not on the cheat sheet.
**Sandbox / index tool exhaust (context-mode):** Route Bash/Read/WebFetch-scale work through a sandbox that returns summaries or BM25 hits ([mksglu/context-mode](https://github.com/mksglu/context-mode)). Author benches claim ~**98%** *tool-output* cuts — treat like RTK: measure bill impact on your harness; license ELv2.


### 8. Targeted exploration (narrow @files; stop early; shrink shell output) — ★★★☆☆ (3/5)
**Impact (why 3/5):** Stops junk entering the prefix; tool claims vary (RTK disputed; graph MCP self-reported).
**What people do:** Point at paths. Prefer indexed / structural search over full-file dumps. Explicit stop rules in AGENTS.md. Concrete tools practitioners use:

- **[RTK](https://github.com/RTK-AI/rtk) (Rust Token Killer)** — wrap shell / CLI output so the agent gets a compressed form (`rtk git status`, hooks rewrite Bash). Use after choosing the right tool; do not let RTK pick the tool. Built-in `Read`/`Grep` often bypass the Bash hook.
- **`/eco` in [ClawCodex](https://github.com/agentforce314/clawcodex)** — deterministic Bash-output filters ported from RTK’s method set: failure-focused test summaries, `git`/`pip`/`npm` noise strip, log dedup, head-cap. A never-worse guard keeps the raw form when a filter does not help. Lossy cuts write the full output to disk with a recovery command. **Self-measured:** 27 replayed command outputs, 92,989 → 17,767 tokens of *command output* (not a bill %). Same caveat as RTK: also measure tool-call count and total session tokens.
- **[OpenViking](https://github.com/volcengine/OpenViking)** (Volcengine, AGPL-3.0) — context database for agent memory, knowledge and skills behind a `viking://` file system. Each folder carries an L0 one-line abstract (`.abstract.md`) and an L1 overview (`.overview.md`). The agent opens L2 full content only when needed. You can copy the idea by hand: a short summary file per folder that the agent reads first. **Vendor-measured** (LoCoMo user memory, Doubao models): input tokens 34.3–91.0% lower than native agent memory. Not on the cheat sheet. CN pointer: [@Huahuazo](https://x.com/Huahuazo/status/2112161843909128627).
- **[tgrep](https://github.com/microsoft/tgrep)** — trigram-indexed grep (client/server) as a **ripgrep/grep replacement** on large trees. Index once (`tgrep serve .`); agents should prefer `tgrep` for repo search when the index is warm (see repo `AGENTS.md`). Speed win on huge monorepos; token win is indirect (faster, tighter search → fewer thrash reads).
- **[codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp)** — MCP that builds a local code knowledge graph (tree-sitter + optional Hybrid LSP). Structural / graph queries replace dozens of grep+read cycles. Authors claim large token cuts vs file-by-file exploration (see sources; mark self-reported). Prefer this *instead of* blind full-repo greps when installed — still subject to MCP hygiene (don’t pile unrelated MCP servers beside it).
- **[Graphify](https://github.com/Graphify-Labs/graphify)** (`Graphify-Labs/graphify`) — `/graphify` skill maps code (+ docs/PDFs/media) into a **queryable knowledge graph** so the agent can query the graph instead of grepping files. Code path: local tree-sitter AST (deterministic, no LLM credits for code map per README). Docs/media may use a semantic pass. Not a vector index. Install: `uv tool install graphifyy` then `graphify install`. No universal token-% in README — savings mechanism is fewer file crawls.
- **Global build-output hook ([SwiftLee](https://www.avanderlee.com/ai-development/reduce-token-usage-claude-code-codex-cursor/), Antoine van der Lee, 28 Sep 2026)** — One `PreToolUse` hook script for Cursor, Claude Code and Codex. It appends `2>&1 | xcsift -f toon -w` to unfiltered `xcodebuild` / `swift build` / `swift test`. The Codex variant denies the raw command and returns the exact retry command. Reason: an `AGENTS.md` filter rule was skipped on 751 of 2,141 builds. Hooks enforce; instructions only suggest. Author audit of 1,674 Cursor sessions: 94.2% cache reads, 0.6% output. Author clean test run: ~406,500 → ~168 estimated tokens of *command output* (not a bill %). The same hook shape fits any noisy CLI. Rules: [EXAMPLE-PROMPTS](./EXAMPLE-PROMPTS.md#swiftlee-global-agent-rules-10-lines).

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
**What people do:** Spawn workers with clean context for search/research. Parent keeps only the answer. Cheap orchestrator + expensive coding worker is a common Codex pattern (@anshuc). The inverse Codex shape is also in use: Astra plans and reviews; Sol implements independent slices ([Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux), @BradGroux). That claim is concurrency, not a measured token cut.  
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
- **Cache writes by lifetime (2.1.300):** The OpenTelemetry `api_request` event now has `cache_creation_5m_tokens` and `cache_creation_1h_tokens`. Use them to see which cache lifetime you pay to write.
- **`/usage` warnings (2.1.299):** `/usage` now warns about max effort, workflows, always-on context, and large tool results. Check it before you add a new meter.
- **Gateway sessions (Vercel AI Gateway):** The Sessions view shows cache hit rate, cost, models and providers per conversation, tagged by client (Claude Code, Codex, Cursor). Sort by cost; a long session with a low hit rate keeps paying to reread history. See Pattern 5.

### Experiments (A/B, not a live UI)

- **Stet** — Replay real repo tasks and compare bills across setup changes ([stet.sh write-up](https://www.stet.sh/blog/gpt-56-token-saving-modes)).
- **Systima** — Metered studies (e.g. subagent tax).

**Claude Code receipt caveat (Corvoysier):** the `usage` field on the result event can undercount a turn badly (author: **14×–442×** on some receipts). Prefer `modelUsage` when metering from session JSON.
### Simple weekly habit

1. Keep a **status line** on while coding.
2. Once a week run **`npx ccusage@latest daily`** (or your dashboard) and note cache-read share + top sessions.
3. Only use `/context` when you need a one-shot breakdown inside an active chat.

**Also see:** [llm-engineer-toolkit → LLM Monitoring](https://github.com/KalyanKS-NLP/llm-engineer-toolkit#llm-monitoring) for Helicone / Opik / Phoenix / agenttrace and peers (catalog only — verify before adopting).

**Weekly refresh:** re-check links and new metering tools each Monday with the rest of this playbook.


## TypeSafe Jev (System One)

Jev is a **decision** model (not a chat/coding LLM): state + typed questions → probabilities. Use it to replace expensive LLM calls for routing, guardrails, and triage — and for **verbatim context prune** via [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (Patterns 6–7).

**Details and real-world use cases:** [TYPESAFE-JEV.md](./TYPESAFE-JEV.md) (includes **open-weights Laya** alternative).

**One-line fit:** Keep frontier models for writing, hard reasoning, and sparse plans; let Jev handle fast classify / route / verify / keep-or-drop-tool-result / **per-tick action Choice** (e.g. [minecraft-agent](https://github.com/rmalde/minecraft-agent) §9; live web [jev-ultrafast](https://github.com/browser-use/jev-ultrafast) §11) (price list ~$0.042/MTok input, output free — verify live; speed/cost multiples are author ceilings). Open-weights alt: [Laya](https://github.com/NandhaKishorM/laya) (same `choice`/`score`/`noul` shape; Apache 2.0 — see TYPESAFE-JEV). On Apple Silicon: [laya-mlx](https://github.com/mizorewww/laya-mlx) (independent MLX port; author latency ms / **0 output tokens** — not a session token-%; [Apple Silicon subsection](./TYPESAFE-JEV.md#apple-silicon-laya-mlx)).

**Decision backends:** Jev, djev, Laya, OpenJev, and SemIf are compared in [Alternatives landscape](./TYPESAFE-JEV.md#alternatives-landscape-decision-backends) (HF blog, 23 Sep 2026). JevBench composite / calibration / latency — **not** a token-%. [Nimble](./TYPESAFE-JEV.md#nimble) ([bespokelabsai/nimble](https://github.com/bespokelabsai/nimble)) is the self-hosted open recipe in that section; author holdout agreement and latency are **not** a token-%.

**Local scoring:** Avi Chawla’s 22 Sep 2026 walkthrough is a SGLang `/v1/score` tutorial in [Local scoring with SGLang](./TYPESAFE-JEV.md#local-scoring-sglang) — mechanism and an author latency demo, **not** a token-%.

**StartLux-Decision:** Open 0.8B–27B decision models with a `/v1/systemone`-compatible server sit in [StartLux-Decision](./TYPESAFE-JEV.md#startlux-decision) — weights CC BY-NC 4.0; author eval, **not** a token-%.


## Computer-use token hygiene

GUI agents burn tokens when **every** click goes through a frontier model + screenshot. Prefer: plan once with a strong model; score local form/clicks with a **System One** specialist or Jev/Laya Choice over code-listed elements; use structured a11y/DOM over pixels when you can; skip already-filled fields; use an API/connector instead of the GUI when one exists.

**Live web sibling:** [jev-ultrafast](https://github.com/browser-use/jev-ultrafast) — page → indexed element table; Jev picks operation + target in **one** request; a small LLM writes text only on `TYPE_TEXT`; default loop has **no screenshots**. Author runtime/request counts (not a token-%) live on the Savings cheat sheet.

**Details:** [CUA-S1 §10](./TYPESAFE-JEV.md#10-computer-use--specialist-system-one-for-local-gui-decisions-cua-s1) · [Jev Ultrafast §11](./TYPESAFE-JEV.md#11-browser-use-jev-ultrafast--indexed-dom--jev-optarget).

**Screenshot pruning breaks cache:** Prune screenshots in batches, not at every step. See Pattern 5 (Asana).

**Send only what changed ([agent-browser](https://github.com/vercel-labs/agent-browser), Vercel Labs, v0.38.0):** `snapshot --delta` returns the full accessibility tree once, then `unchanged` or a compact structural delta. It falls back to full state after a URL change. `--delta --full` refreshes the baseline, for example when compaction dropped the earlier tree. `screenshot --if-changed` skips unchanged captures. `--delta` resets between sessions. For checks hours apart, save a snapshot and run `diff snapshot --baseline <file>`. Maintainer demo: 73× fewer tokens filling one form (**author demo**, not a bill %). [Post](https://x.com/ctatedev/status/2111930462176432333).

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
| Existing backlog; independent slices; one frontier chat does every role | **Astra plan/review + Sol leaves** (Brad Groux) | Concurrency and less role-switching. Serialize shared interfaces. No measured token cut. |
| Unsure | **One agent first; add workers only when clutter or parallelism is real** | Default away from tax |

**Astra plans + Sol implements (Brad Groux):** In Codex, Astra (up to Extra High) reads docs, code, and open issues, then assigns 3–5 independent tasks to GPT-5.6 Sol (generally Medium). Astra reviews, merges when authorized, updates the preview, and starts the next ready work. Author: faster because independent work moves at once; **no controlled benchmark**. House fit: frontier model for plan, review, and coordination; Sol (Medium) for scoped implementation — same family as cheap orch and [MODE-MATRIX](./MODE-MATRIX.md). The article states no price. Parallel threads can raise total tokens (Agent Teams). Full write-up: [ORCH-CASES Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux). Source row: [SOURCES](./SOURCES.md).

**Sol orchestrates, Astra reviews only (Md Ismail Sojal):** Sol runs the tree, Luna explores, and Astra runs only for an independent review — [ORCH-CASES Case 13](./ORCH-CASES.md#case-13-sol-orchestrates-astra-review-only); “1/5th the cost” is an author cost claim, **not** a token-%.

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
- @BradGroux (shown 6:49 PM · 22 Sep 2026, author-local): Astra plans and reviews; 3–5 Sol threads implement. Faster from concurrency. **No controlled benchmark. No token-%.** Parallel threads can cost more total tokens than one serial session.

### Measured / product-backed (still context-specific)
- Anthropic staff: 7+ MCP servers → 67k+ tokens.
- Anthropic blog: mid-session Haiku switch can cost more than staying on Opus (cache rebuild).
- OpenCode DCP and Atlassian: prune-first architecture described with engineering rationale; not a single published % for all users.
- Cursor Dynamic Context Discovery (Jediah Katz, 6 Jan 2026): **46.9%** fewer total agent tokens on MCP-calling runs (product A/B; high variance by MCP count).

### Conflicts to flag on slides
1. **Fresh chat vs long session:** Clear between *unrelated* tasks. For *same* task, warm prompt cache can make one long session cheaper than many cold starts (Reddit/Claude Code threads; Anthropic caching docs).
2. **Subagents save parent context but can raise total tokens** (~7× heavy fan-out). Isolation ≠ cheaper system-wide.
3. **Cheap routing vs cache:** Per-turn routers can *increase* spend. Route at cold boundaries or via subagents with handoff.
4. **Auto-compact:** Some practitioners disable it (noise / lost control). Others compact manually at ~60%. Product auto-compact near 95% is late for quality. Idle compaction (2.1.299) also summarizes a long idle session; turn it off when full detail matters.
5. **STE100 vs Concise:** STE100 = short *sentences* / clarity; Concise = less narration. STE100 alone does not guarantee fewer tokens.
6. **STE100 always-on vs thinking quality (S Anand, 1 Aug 2026):** Asking ChatGPT (GPT-5.6 Sol) to “Answer in ASD-STE100” reduced thinking quality across six tasks (fewer sources; red on most eval axes). Prefer: let the model reason, **then** ask for STE100/simple explanation — or keep STE100 for status/orchestration replies only. **No token-%** in that post.
7. **Starving context backfires:** Under-spec prompts cause more exploration and retries (Atticus Li thesis: optimize cost-per-accepted-change, not tokens-per-message).
8. **Fewer tokens ≠ faster or cheaper (UT Austin, 26 Sep 2026):** Compression can cut tokens and still raise latency or cost. Measure latency and cost, not tokens alone. Research pointer under Pattern 7; **not** a cheat-sheet savings %.

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
6. Astra plan + Sol leaf stubs — companion Astra/Sol section (skeletons; full text stays on the X article)

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


## Measured study: 12 Claude Code tips, 945 runs (Kensei, 2026)

Headline figures are on the **Savings cheat sheet** (row: Kensei 945-run). This section holds setup, mechanism and limits.

**Source:** X article “Stop Hitting Claude Code Limits: 7 Best Practices, Tested on 945 Runs” by [@0xKense1](https://x.com/0xKense1/status/2111105138290987198) (Kensei, 16 Oct 2026). Raw logs, harness and charts: [kenseii1/claude-code-limits-tests](https://github.com/kenseii1/claude-code-limits-tests) (MIT). Pointer: [@daniel_mac8](https://x.com/daniel_mac8/status/2111501909337210911) (Dan McAteer, 17 Oct). Small account; trust rests on the public raw logs.

**Setup (author-reported):** A renamed copy of Flask (“Tessel”), so the model must open files. 10 “how does X work in this repo” questions with an answer key. A fresh `claude -p` per run with explicit `--model` and `--effort`. Arms paired per question, in shuffled order. Default: Sonnet 5.5, effort medium. Tokens from `modelUsage` (input, output, cache reads, cache writes, subagents). 118M tokens in total. Each figure has a 95% CI; a CI that crosses zero counts as “no clear effect”. Claude Code 2.1.296–2.1.301, 14–16 Oct 2026.

**Where tokens go (author):** A bare “ok” cost 19,100 tokens before any file read. The average question cost ~110k tokens, while the last call held only ~24k. Each call resends the base plus everything read so far. 87% of tokens were cache reads; ~1% was the answer. So only two levers work: a smaller base, or less history carried.

**Other findings (author):**
- Flat notes map + “answer from the note” rule: 32% fewer tokens, but 2 of 20 answers wrong (a note missed a detail). Use only with maintained notes.
- Name the file and function: 13% more tokens, but 20/20 right vs 8/20 for the vague prompt. A cheap wrong answer costs more.
- Haiku 5.5 vs Sonnet 5.5 on lookups: 33% more tokens, about 12× cheaper per question at list price, same accuracy. How plan limits count this is not published.
- Opus 5.5 vs Sonnet 5.5 on lookups: +9.9% tokens (CI crosses zero), about 2.2× list cost per question ($0.18 vs $0.08), 20/20 vs 19/20 correct (repo `results/ANALYSIS.md`). Kensei’s [follow-up](https://x.com/0xKense1/status/2112256782315708667) (19 Oct) uses this to test the “use the most capable model” tip, which he quotes from Boris Cherny on Lenny’s Podcast (Feb, about Opus 4.6). Lookups only, not long builds.
- Subagent for a test-suite diagnosis: no clear change in total. The main chat stayed smaller (22k vs 32k on the heavy task).
- MCP connectors off: bare “ok” 1,691 tokens lighter; no clear effect on real questions (tools are deferred by default).
- Deny rule on a vendor folder: kept searches out (13/20 → 1/20 runs). No token effect.
- Caveman skill: fewer output tokens, no clear total cut. The short style faded after turn 2.
- Obsidian notes via MCP vs plain files: +34% / +42% (smaller earlier test, not part of the 945).

**Limits (author):** Mostly Sonnet 5.5. One Python repo. Code reading, not writing. Raw tokens, not the plan meter. 3 chains per session length.

**Deck use:** Strongest public measurement so far for `/clear`, `/compact` and skills over CLAUDE.md. Also a measured caution: subagents, MCP-off and caveman did not cut totals here. Pairs with u/bisonbear2 (Codex): output-side tricks touch only ~1% of the bill. Measure-one-tip loop: [EXAMPLE-PROMPTS](./EXAMPLE-PROMPTS.md#kensei-measure-one-tip-loop).

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
**Maintenance (weekly, Monday ~09:00 Europe/London):** Refresh X/web in [SOURCES.md](./SOURCES.md). Re-check metering tools, [TYPESAFE-JEV.md](./TYPESAFE-JEV.md), [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction), [laya-mlx](https://github.com/mizorewww/laya-mlx), the [Cursor harness efficiency post](https://cursor.com/blog/improved-token-efficiency), the [TinyFish context guide](https://www.tinyfish.ai/blog/claude-code-context-window), and the STE100 always-on caveat. Update the **Savings cheat sheet** only when new attributed figures appear (laya-mlx ms stay latency, not a token-%; Brad Groux Astra/Sol stays **no token-%**; JevBench composite / cal / latency stays off this sheet; Nimble holdout agreement / latency stays off this sheet; the SGLang scoring tutorial’s author latency stays off this sheet; TinyFish / ECC / context-mode % stay off this sheet as universal figures; OpenAI cache customer figures stay labeled **customer-reported**; Corvoysier recall row stays **self-measured** n=3; AA effort $/task stays raw **third-party** with no derived %; FOCUS peak-context stays off this sheet; Haiku 5.5 average/list cuts and the Sonnet 5.5 cache-read note stay **vendor**; Command+K and Agent Planners stay **self-measured**; UT Austin latency and the in-paper Copilot cite stay off this sheet; StartLux-Decision author eval stays off this sheet; the Sojal “1/5th the cost” stays an author cost claim; idle compaction, the ToolSearch fix, the OTel cache-write split, `/usage` warnings, and Codex cache reuse stay **vendor, no %**; Kensei 945-run figures stay **self-measured** (code-reading questions, mostly Sonnet 5.5); Asana 29× / 76× stay **company-measured** vs the original setup (76× includes a model change); SwiftLee audit counts stay **author audit**, not a bill %; Vercel session affinity stays **vendor, no %**). Keep companions + thin `guides/` in sync (map rows only — no tip duplication). `EXAMPLE-PROMPTS.md`, `ORCH-CASES.md`, `SPEAKER-NOTES.md`. Last edit: 2026-10-19. Source of truth: private GitHub `PongPong/token-savings-notes`.

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

