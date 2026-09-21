# Sources and X primary posts

Companion to `NOTES.md`. Named quotes, link list, and X primary delta.

Canonical **numbers** for the deck live in NOTES **Savings cheat sheet** — do not duplicate % here without updating that table.

## Named sources — handle + link + short quote

| Source | Link | Quote / claim (exact or closely attributed) | Savings |
| --- | --- | --- | --- |
| Claude Academy (official) | https://academy.claude.com/courses/claude-code-101/context-management | “Use `/compact` when you're working on a specific feature… Use `/clear` when you want to start a new feature.” Subagents return “just a summary” to keep primary context clean. | no figure given |
| MindStudio — `/compact` at 60% | https://www.mindstudio.ai/blog/claude-code-compact-command-context-management | “The better approach: compact at around 60% context utilization, before quality starts degrading.” | no figure given |
| Atlassian — Rovo Dev pruning | https://www.atlassian.com/blog/development/rovo-dev-keeps-long-sessions-useful | “structure-aware pruning is the better default… LLM-based compaction still has a place as a more aggressive fallback.” | no figure given |
| OpenCode DCP (GitHub / npm) | https://github.com/Opencode-DCP/opencode-dynamic-context-pruning · https://www.npmjs.com/package/@tarquinen/opencode-dcp | Prunes stale tool content before LLM send; author notes ~85% cache hit with DCP vs ~90% without (plugin docs / secondary summaries — **not** on live README as of 21 Sep 2026). Live README related project: **[Sleev](https://sleev.ai)** (`sleev` CLI) — local proxy for Claude Code, Codex, OpenCode. DCP still works; Sleev is a related successor. | cache-hit delta self-reported in docs ecosystem; no universal %; Sleev **no % cited here** |
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
| Graphify Labs | https://github.com/Graphify-Labs/graphify | “maps your entire project (code, docs, PDFs, images, videos) into a knowledge graph you can query instead of grepping through files.” Code: local tree-sitter; “LLM credits **0**” for graph build vs per-token systems (benchmark table). | no universal session-token %; code-map credits **0** (author bench) |
| fast-jev-compaction (tamaratran) | https://github.com/tamaratran/fast-jev-compaction | “never rewrites anything… only deletes tool calls and tool results Jev says are no longer needed”; Claude Code plugin replaces compaction summary when reduction enough; else fallback. `reductionRatio` = char cut, not a published token-%. | No universal token-% in README |
| llm-engineer-toolkit (KalyanKS-NLP) | https://github.com/KalyanKS-NLP/llm-engineer-toolkit | “curated list of 120+ LLM libraries category wise.” Token-relevant categories: Routers, Cache, Memory, Monitoring, Prompts (compression), Structured Outputs. | Catalog only — no toolkit-wide savings % |
| coleam00 Context-Engineering-Intro | https://github.com/coleam00/Context-Engineering-Intro | Context engineering template: lean CLAUDE.md + examples/ + INITIAL.md → `/generate-prp` → `/execute-prp`. “Most agent failures aren't model failures - they're context failures.” Slogan 10x/100x is advocacy, not a token %. | Quality / completion workflow; measure trajectory cost |
| rmalde/minecraft-agent | https://github.com/rmalde/minecraft-agent | Astra/Sol → JSON plan; Jev Choice `a0…an` + `compactObservation` state (`models.mjs`). nether-final-08: 131 Jev / 35 Astra (**self-reported**). | Call-ratio / timing self-reported; seed surveyed |
| Laya (NandhaKishorM / Convai Innovations) | https://github.com/NandhaKishorM/laya | Open-weights non-autoregressive System 1 (`choice`/`score`/`noul`); Apache 2.0; Router + EN/multilingual/typed-decisions checkpoints. Author vs published Jev table (latency/accuracy) — Jev figures third-party, not re-measured in that README. | Self-host alternative to TypeSafe Jev; fine-tune for domain |
| Jo Do / Rulestack (DEV) | https://dev.to/rulestack/tool-search-hid-40170-tokens-from-our-first-request-auto5-put-them-all-back-2jin | Claude Code v2.1.263, 2026-09-09, four MCP servers + browser extension, 1M ctx. `ENABLE_TOOL_SEARCH` unset **20,819**; `false` **60,989**; `auto:5` **62,319**. Delta default vs false ≈ **40,170** deferred (88 tool defs). `auto:N` thresholds vs context-window size (1M × 5% = 50k → ~40k defs load upfront). | **self-measured** first-request input; not a universal % |
| Cursor — Dynamic Context Discovery (Jediah Katz) | https://cursor.com/blog/dynamic-context-discovery | MCP: sync tool descriptions to folders; agent gets names then looks up schemas. A/B on runs that called an MCP tool: “this strategy reduced total agent tokens by 46.9% (statistically significant, with high variance based on the number of MCPs installed).” Also: long tool outputs → files; chat history as files; Agent Skills; terminals as files. | **46.9%** total agent tokens, MCP-calling runs — **Cursor A/B** (product-backed) |
| Sleev | https://sleev.ai | Local proxy / `sleev` CLI for Claude Code, Codex, OpenCode (and other harnesses). Pointed from live OpenCode DCP README as related project. | no % cited here — do not deep-dive without measuring |
| ClaudeCodeLab (CN) | https://claudecode-lab.com/zh/blog/claude-code-token-optimization/ | `/usage` metering; lean CLAUDE.md; MCP off when CLI suffices; subagent caveat (isolates noise, still spends its own context). | scout / CN — **no new cheat-sheet %** |
| ofox.ai (CN) — 5 strategies 2026 | http://ofox.ai/zh/blog/claude-code-token-optimization-5-strategies-2026/ | Stacked levers (cache, model tier, context hygiene, thinking budget, hooks/subagents/Batch). Author claims bill to **10–40%** of prior (title also says 60–90%). | **self-reported stacked** — do **not** put 60–90% on the cheat sheet as a new universal figure (same class as Hasan stacked caveat) |
| CUA-S1 / trycua (Show HN) | https://news.ycombinator.com/item?id=49767564 · https://github.com/trycua/cua | System One specialist for computer-use form decisions (USE/CHECK/CLICK/SKIP); ~706k params; author: 7–9 ms local vs 260–280 ms hosted Jev; form-task accuracy claims vs Jev (**specialist fine-tuned**). | Computer-use token pattern; no universal session token-% |
| Jev Ultrafast (Browser Use × TypeSafe) | https://github.com/browser-use/jev-ultrafast (~14.5k★, MIT) · [docs/performance.md](https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md) | Dynamic indexed action space: page → element table; Jev picks operation + target in one request; small LLM writes text only on `TYPE_TEXT`; default loop **no screenshots**. Demo Zürich→London **7.073 s**. Matched n=3: median **9.450 s → 7.092 s** (~25% lower **runtime**); TypeSafe reqs **22 → 17**; protocol **1,092 → 101**; sign-test p=0.25. Recording: 17 Jev reqs; median Jev ~178 ms; TypeSafe 90,558 in / 6,325 out tokens; **no billed TypeSafe $**; OpenRouter helper **$0.00006272** for two calls. | **Author eval / small-n**; runtime & request counts — **not** a session token-% |
| json-render (Vercel Labs) | https://github.com/vercel-labs/json-render (~17.8k★, Apache-2.0) · https://json-render.dev | Generative UI: AI emits JSON constrained to a predefined component/action catalog (Zod). SpecStream. Constrained catalog JSON ≫ freeform HTML/JSX/markdown UI dumps. | **No published universal token-%** — do not invent one |

---


## Source list (markdown)

- [Context management · Claude Academy](https://academy.claude.com/courses/claude-code-101/context-management) — undated course page — docs  
- [How to Use the /compact Command… (MindStudio)](https://www.mindstudio.ai/blog/claude-code-compact-command-context-management) — 2 Apr 2026 — blog  
- [Agent Context Pruning · Atlassian Rovo Dev](https://www.atlassian.com/blog/development/rovo-dev-keeps-long-sessions-useful) — 30 Mar 2026 — blog  
- [opencode-dynamic-context-pruning](https://github.com/Opencode-DCP/opencode-dynamic-context-pruning) — GitHub (`@tarquinen/opencode-dcp`); live README related: [Sleev](https://sleev.ai)  
- [Graphify](https://github.com/Graphify-Labs/graphify) — GitHub (knowledge graph skill)  
- [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) — GitHub  
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
- [Tool search hid 40,170 tokens · Jo Do / Rulestack (DEV)](https://dev.to/rulestack/tool-search-hid-40170-tokens-from-our-first-request-auto5-put-them-all-back-2jin) — ~11 Sep 2026 — blog (**self-measured**)
- [Dynamic context discovery · Cursor (Jediah Katz)](https://cursor.com/blog/dynamic-context-discovery) — 6 Jan 2026 — product blog (**46.9%** A/B)
- [Sleev](https://sleev.ai) — local multi-harness context proxy (`sleev` CLI); related to OpenCode DCP
- [Claude Code Token 优化指南 · ClaudeCodeLab (CN)](https://claudecode-lab.com/zh/blog/claude-code-token-optimization/) — Apr 2026 (updated Jun 2026) — `/usage`, lean CLAUDE.md, MCP off, subagent caveat
- [Claude Code Token 优化 2026：5 个策略 · ofox.ai (CN)](http://ofox.ai/zh/blog/claude-code-token-optimization-5-strategies-2026/) — 13 May 2026 — stacked levers; author **10–40% of prior** (**self-reported stacked**; not a new cheat-sheet figure)  
- [Jev Ultrafast](https://github.com/browser-use/jev-ultrafast) — GitHub (Browser Use × TypeSafe; indexed DOM + Jev op/target); [performance.md](https://github.com/browser-use/jev-ultrafast/blob/main/docs/performance.md) author eval
- [json-render](https://github.com/vercel-labs/json-render) — GitHub (Vercel Labs Generative UI); [json-render.dev](https://json-render.dev)  

---



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


## X/web primary delta (21 Sep 2026)

Web search pass (Europe/London). `site:x.com` still empty; X search URLs still redirect to login. No new X primary quotes this week.

**Web (merged into NOTES / this file):**
- Rulestack / Jo Do DEV (~11 Sep 2026) — `ENABLE_TOOL_SEARCH` first-request tokens **self-measured** (20,819 / 60,989 / 62,319; ~40,170 deferred). `auto:N` vs window-size caveat.
- Cursor Dynamic Context Discovery (Jediah Katz, 6 Jan 2026) — product A/B **46.9%** fewer total agent tokens on MCP-calling runs. Was missing from NOTES; now on the cheat sheet.
- OpenCode DCP live README — related project **Sleev** (`sleev` CLI). DCP still works; no Sleev % cited.
- CN scout (reaffirm only; **no new cheat-sheet %**): [ClaudeCodeLab](https://claudecode-lab.com/zh/blog/claude-code-token-optimization/) (`/usage`, lean CLAUDE.md, MCP off, subagent caveat); [ofox.ai ZH](http://ofox.ai/zh/blog/claude-code-token-optimization-5-strategies-2026/) stacked levers, author bill to 10–40% of prior (**self-reported stacked** — do not promote 60–90% as a new universal figure).
- [jev-ultrafast](https://github.com/browser-use/jev-ultrafast) (~14.5k★) — indexed DOM + Jev op/target; author runtime/request counts on NOTES cheat sheet (**not** a token-%).
- [json-render](https://github.com/vercel-labs/json-render) (~17.8k★) — catalog-constrained generative UI JSON; **no published token-%**.

---
**Maintenance (weekly, Monday ~09:00 Europe/London):** Refresh X/web deltas. Re-check metering tools and **[TYPESAFE-JEV.md](./TYPESAFE-JEV.md)**. Update savings figures only when attributed. Keep `SPEAKER-NOTES.md` in sync if the talk changes. Last edit: 2026-09-21. Source of truth: private GitHub `PongPong/token-savings-notes`.

