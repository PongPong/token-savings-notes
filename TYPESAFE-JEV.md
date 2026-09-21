# TypeSafe Jev (System One) — for token-aware agents

Short companion to `NOTES.md`. Keep the main playbook thin; put detail here.

**What it is:** Jev is TypeSafe AI’s **System One** model. It does **not** write prose or code. You send **state** (text or JSON) plus typed **questions**. It returns choices, scores, or yes/no probabilities — in parallel, usually in tens to hundreds of milliseconds.

**Official:** [Introducing System One Models & Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) (15 Sep 2026) · [typesafe.ai](https://typesafe.ai) · skill: [typesafe-ai/skills](https://github.com/typesafe-ai/skills)

**API shape:** `POST /v1/systemone` with `model` (e.g. `jev-latest`), `state`, and `questions` (`noul` / `choice` / `score`). Also via Vercel AI Gateway as `typesafe-ai/jev`.

## Why it matters for token spend

Coding agents and chat LLMs burn tokens on **every** routing, guardrail, and “is this urgent?” step. Jev is meant to be a **cheap decision primitive** so the frontier LLM only runs when you need generation or deep reasoning.

| Claim | Source | How to treat it |
| --- | --- | --- |
| Input **$0.042 / MTok**; output **free** | TypeSafe launch blog | Price list — verify live |
| End-to-end **~70–500 ms** (often ~100 ms) | TypeSafe launch blog | Latency from their region; measure yours |
| Up to **~193.6× faster / ~444.6× cheaper** on workflow evals | TypeSafe homepage / blog | **Author ceiling**; they say high end of real-world |
| Batching many questions in one call can be **~12× cheaper / ~10× faster** than one-question-per-call | Flavio / TypeSafe cookbook pattern (speculative fan-out) | Depends on long shared state |
| Doom bot ~**10 queries/s** ≈ **~$7/hour** | TypeSafe fun demos | Demo cost, not your bill |

**House rule:** Jev cuts the **decision** slice of the bill. If most spend is long coding context, savings stay small. Measure cost **per solved task**, not only per token (same lesson as NOTES metering + bisonbear2).


## Open-weights alternative: Laya

**Repo:** [NandhaKishorM/laya](https://github.com/NandhaKishorM/laya) (~4.0k★, **Apache 2.0**) · PyPI `laya` · HF [convaiinnovations/laya](https://huggingface.co/convaiinnovations/laya).

**What it is:** Self-hostable **non-autoregressive System 1** decision engine — same primitive family as TypeSafe Jev (`choice` / `score` / `noul` over a state) in **one forward pass**, no text generation. Multilingual Router picks a checkpoint per request. Position it as an **open-weights / $0 self-hosted** alternative when you cannot or will not call TypeSafe’s closed API.

| | TypeSafe Jev (NOTES default) | Laya (open weights) |
| --- | --- | --- |
| Weights / license | Closed API | **Apache 2.0** checkpoints |
| Cost shape | ~$0.042/MTok input (verify live) | **Self-host** (GPU/CPU your bill) |
| API shape | `state` + typed questions | Same idea: `predict(state, questions)` |
| Latency (author / 3P) | Blog ~70–500 ms; 3P p50 often cited ~236–276 ms | README T4: ~**33–40 ms** / 1 q; multilingual ~**7.2 ms/q** batched |
| Languages | Product focus EN (check live docs) | English + **100+** via `laya-multilingual` + `Router` |
| High-cardinality Choice | Strong (README: Jev up to ~255 options) | Weaker at default head budget (Banking77); raise `head_max_len` or hierarchical Choice |

### Checkpoints + Router

| Checkpoint | Encoder | Params | Context | Use |
| --- | --- | --- | --- | --- |
| `laya` | ModernBERT-large | 421M | 512 | English |
| `laya-multilingual` | mmBERT-base | 322M | 1024 | 100+ languages (faster) |
| `laya-typed-decisions` | ModernBERT-large | 421M | 1024 | Typed-decisions workflows |
| **`Router(preload=True)`** | — | — | — | Detects script/language (&lt;1 ms) → dispatches checkpoint |

**Presets (map to NOTES / Jev use cases):** `laya.router_questions()` (cheap vs frontier), `guard_questions()`, `moderation_questions()`, `triage_questions()`.

### How it maps onto Jev use cases here

Same patterns as §1–§9 and fast-jev-compaction: swap the decision backend when self-hosting is required. Keep **code-listed** Choice options (minecraft-agent style). For compaction / keep-drop tools, you’d reimplement the asker against `agent.predict` / `Router` — Laya is not a drop-in Claude Code plugin.

### Honest limits (from Laya README — do not oversell)

- **Base checkpoints ≈ chance** on typed-decisions zero-shot; fine-tune for domain (typed-decisions FT **0.766** vs published Jev **0.727** — **author table**, different sample/prompt setup; Jev numbers “never measured here”).
- **&gt;~20–50 Choice options:** default token budget per label is tiny; raise `head_max_len` / `max_len` or split coarse→fine. Jev currently easier out-of-the-box for 50+ options.
- English checkpoint **collapses** on non-Latin (e.g. Khmer 0.000 @ high confidence) — **always route**; don’t trust raw confidence alone.
- Fit **temperature** for calibration before gating on confidence (ECE improves a lot after fit).
- Soft distribution matching and raw ECE: places where published Jev still leads — see their [BENCHMARKS.md](https://github.com/NandhaKishorM/laya/blob/main/BENCHMARKS.md).

**House rule:** Prefer TypeSafe Jev when you want a managed API and high-cardinality Choice without tuning. Prefer **Laya** when you need open weights, offline/VPC, multilingual routing, or $0 inference at the model layer. Measure your own latency and accuracy; don’t put either side’s leaderboard Δ on the Savings cheat sheet as a token-%.

```bash
pip install laya
# Router(preload=True) for production; or laya.load("convaiinnovations/laya")
```

## Real-world use cases (agents + products)

### 1. Model / task routing (agent middleware)

**Scenario:** Some user asks need a cheap/fast model; others need a strong reasoner.  
**Pattern:** Jev Choice (+ optional Score for ambiguity) picks `fast` vs `powerful` (or `deterministic_script` / `human`). Code starts the right agent.  
**Why it fits:** Avoids a full LLM call just to choose a model. Matches NOTES “cheap orch / cold-boundary routing” without mid-session `/model` cache busts.  
**Source:** [LangChain — Building a Harness with Jev](https://www.langchain.com/blog/building-a-harness-with-jev) (`ModelRouterMiddleware`).

### 2. Tool / shell guardrails (“Auto Mode”)

**Scenario:** Agent wants to run `bash` / delete / deploy.  
**Pattern:** Before the tool runs, Jev scores `read_only` / `reversible` / `irreversible` (and related Nouls). Low confidence → ask a human.  
**Why it fits:** Same idea as closed harness safety classifiers, but cheap enough to run on every tool call.  
**Source:** LangChain `AutoModeMiddleware`; Flavio early shadow test of ambiguous `rm -rf` (log first, then automate).

### 3. Support / inbox triage

**Scenario:** Tickets or emails pile up; not every message needs a frontier LLM.  
**Pattern:** One call with many questions: department Choice, urgency Noul, severity Score, refund Noul, etc. Code routes to billing, eng, product, or human.  
**Why it fits:** Speculative fan-out — ask everything once; ignore unused answers. Order-status lookups can skip the LLM entirely.  
**Sources:** TypeSafe quickstart / support examples; Flavio intent-routing walkthrough; LangChain support-ticket urgency example.

### 4. Verify LLM output (claim / citation checks)

**Scenario:** An LLM wrote a summary, PR comment, or podcast notes.  
**Pattern:** For each claim, a Noul: “Does the source support this?” Flag low probability for review.  
**Why it fits:** Keeps the generative model for writing; uses Jev for cheap checks (NOTES: verify before trust).  
**Sources:** Flavio “verification” section; TypeSafe cookbooks (citation / injection scoring).

### 5. PR / code risk prefilter

**Scenario:** Large PR; you do not want Opus on every hunk.  
**Pattern:** Chunk diffs → Jev Scores/Nouls for security, complexity, bad practices → rank → send only hot chunks to a coding model.  
**Why it fits:** Same “filter then frontier” pattern as blog maintenance / log triage.  
**Source:** Flavio “semantic linter” / PR risk matrix pattern.

### 6. Browser / tool-pick loops

**Scenario:** Agent must click one of many page elements, or pick one tool from a list.  
**Pattern:** Planner LLM sets the goal; Jev Choice over **candidates code already listed** (never invent a DOM node).  
**Why it fits:** High-cardinality Choice without hallucinated options; TypeSafe Wikiracing demo stress-tests this.  
**Sources:** TypeSafe Wikiracing / Doom demos; Flavio browser-automation and tool-picker demos (Browserbase cited in LangChain post).

### 7. Bulk labeling / map-reduce

**Scenario:** Label thousands of rows (papers, listings, events).  
**Pattern:** One Jev call per row (or batched questions); classical code aggregates.  
**Early reports (treat as anecdotes):** ~1k papers labeled for ~$0.08 (plus LLM summary prep cost); large listing batches in minutes — Flavio “What people are building.”  
**Why it fits:** TypeSafe’s “map-reducing over big data” pitch; cheap input price dominates.

### 8. Real-time UX judgments

**Scenario:** Score tone / urgency / “looks AI-written” while the user types.  
**Pattern:** Call Jev on debounce; update UI from Scores.  
**Why it fits:** Sub-second latency makes per-keystroke AI feasible where a frontier LLM would not.  
**Source:** Flavio editor / feed-filter demos; TypeSafe real-time use-case list.



### 9. Minecraft agent — Astra plans, Jev picks the next action

**Repo:** [rmalde/minecraft-agent](https://github.com/rmalde/minecraft-agent) (~259★). Astra/Sol plans sparsely; Jev (`typesafe/jev-1.13`) selects one **code-listed** Mineflayer action each step. OpenRouter via local relay; model IDs in README.

**Latest verified result (author README, `nether-final-08`):** empty inventory → Nether → six bed explosions → exit, full health, no deaths; **8m 43.3s** (~**40%** shorter than prior video — **self-reported**). **131 Jev decisions** / **35 Astra calls**. Peaceful surveyed seed; recordings local/out of git.

#### How the prompts and I/O are designed (`models.mjs`)

**Role split (hard rule in both prompts):** Astra/Sol = high-level **plan only**. Jev = **every** player action. Planner must not emit keypresses or invent observations.

**1) Planner input / output (Astra or Sol)**

| Piece | Design |
| --- | --- |
| **Endpoint** | `/api/v1/chat/completions` · `PLANNER_MODEL` default `openai/gpt-6-astra` |
| **System prompt** | Long, seed-specific. Three variants keyed off `knownSeed.requiresEyes` / `requiresNether` / default. States difficulty (Peaceful vs Easy), kit targets, surveyed coords, “no cheats/teleports”, camera limits, and that **Jev chooses all actions**. |
| **User message** | `JSON.stringify(compactObservation(state))` — same compacted game state Jev sees (plus plan-trigger fields when calling `plan`). |
| **Constraints** | `reasoning.effort: 'low'`, `max_tokens: 1800`, `response_format: json_object` |
| **Required JSON out** | `{ objective: string, targets: { [item]: minCount }, waypoint: {x,y,z}\|null, notes: string }` — validated (`objective` must be string). |
| **Cadence** | Not every tick — milestones, stage change, failures, arrival, or periodic async refresh (`async-planner` / `planTrigger`). Plans refresh **while Jev keeps acting**. |

Prompt themes (paraphrase of source, not FB): keep objectives short; one kit objective not per-recipe spam; follow `kitNeeds` deficits; travel uses surveyed `travelRoute.next`; End = beds from cover; never invent observations.

**2) Jev input (state + Choice question)**

Code builds a candidate list `options[]` (`{ key, description, fn }`) — only **legal** moves for this tick (travel, mine one block, loot, craft, boat, escape breath, bed attack, wait, …). Then:

```text
POST /api/alpha/decisions
model: typesafe/jev-1.13
state: JSON.stringify(compactObservation(gameState))   # stringified structured state
questions.action:
  type: choice
  instructions: <fixed controller brief — see below>
  criteria: { "a0": "<desc>", "a1": "<desc>", ... }   # from options[i].description
```

**`compactObservation` (shrink state before both models):** keep recent actions to last **5** (`action/result/position` only); trim mobs to **16** within 48 blocks; drops within 24 blocks; strip `hotbar`, `equipment`, `xp`, recording flags, raw `looted` list noise; compress bastion loot counts. Goal: small relevant state (TypeSafe “context rot” rule).

**State typically includes:** `position`, `dimension`, `health`, `food`, `inventory`, `kitNeeds`, `stage`, `plan` (current Astra JSON), `knownSeed` / route, `battle` / dragon observations, `recent` action results, `difficulty`, run counters — **structured Mineflayer/sensor data, not screenshots**.

**Fixed Choice `instructions` (controller brief):** advance planner objective; **FIRST** take offered escape if breath cloud threatens (respect `safeAtCurrentHeight`); never eat/wait inside cloud; after safety, eat if health &lt; 16 until food bar full; survival &gt; item reserves; prefer boat for long water; don’t craft/collect past target; avoid failed/needless waits; movement/mining may use pathfinding but **Jev owns the choice**.

**3) Jev output → game**

| Field | Meaning |
| --- | --- |
| `answers.action.choice` | Key like `a3` |
| Harness | `options[Number(choice.slice(1))]` → run that option’s `fn` |
| Reject | Missing/invalid choice → throw `Invalid JEV action` (no free-form action inventing) |

**Design takeaway for agents:** (1) **Code enumerates** the action set each step — Jev only ranks. (2) **Stringify one shared state** for plan + decide after compaction. (3) **Planner prompt = JSON contract + world rules**; **Jev prompt = short Choice instructions + criteria map**. (4) Sparse frontier calls, dense Jev calls → token/cost shape of cheap leaf / tool-pick (§6).

**Caveats:** Seed/route surveyed; Peaceful for that recording. Related demos cited by author: [typesafe-minecraft-demo](https://github.com/ellistev/typesafe-minecraft-demo), [typesafe-mario](https://github.com/fhshaik/typesafe-mario), [tsai-sc](https://github.com/phyous/tsai-sc).



### 10. Computer use — specialist System One for local GUI decisions (CUA-S1)

**Problem:** Computer-use agents are token-hungry. A common loop is: screenshot (or huge a11y dump) → frontier LLM thinks → one click → repeat. Most steps are **local** decisions (“put this value in this box”, CHECK / CLICK / SKIP), not novel planning.

**Pattern ([trycua/cua](https://github.com/trycua/cua) · Show HN [CUA-S1](https://news.ycombinator.com/item?id=49767564)):** Keep a **System 2** frontier model for plan / explore / recover. Hand **narrow, recurring** decisions to a **System One** scorer that only ranks options you already listed — no token-by-token generation, no inventing field values, no screenshot in the specialist path.

**CUA-S1-FORMS (first release, author claims):**
- ~**706k** params · ~**2.8 MB** checkpoint · MIT under `libs/cua-s1`
- Input: structured form elements + candidate values from a document (not screenshots)
- Output: per-element scores → USE value / CHECK / CLICK / SKIP; your code orders actions; Cua Driver executes
- vs hosted Jev on their form task (**author eval, specialist fine-tuned for this convention**): whole set **99.7%** vs **83.6%**; action steps **100%** vs **96%**; leave-filled-alone **100%** vs **74%**
- Latency (**author**): **7–9 ms** local score vs **260–280 ms** hosted Jev call (network included; not end-to-end form time)

**Token-savings fit (computer use generally):**

| Lever | Why it cuts spend |
| --- | --- |
| **Frontier only for plan / stuck** | Don’t re-pay Astra/Opus for every checkbox |
| **Code-listed actions + S1 Choice** | Same as minecraft-agent / Jev tool-pick — model cannot invent DOM nodes |
| **Structured state ≫ full screenshots** | Accessibility / DOM / form schema beats pixels for fill loops |
| **Skip already-filled** | Cheap SKIP policy beats another LLM turn |
| **Prefer connector/API over GUI** | When a form has an API, skip computer-use entirely |
| **Crop / lower res / one focused window** | If you must send pixels, shrink the image tax |

**Design takeaway:** Space between brittle scripts and full agent loops — layout varies enough that scripts hurt, but the **decision set stays narrow**. General agent hits novelty → specialists (forms, later other apps) score local steps.

**Caveats:** Forms-only today; specialist beat Jev after **task-specific** training — not a free general Jev replacement. No universal token-% in the Show HN. Measure your own frontier-call count per form completion.

Related in this pack: §6 browser/tool-pick · §9 minecraft Astra+Jev · [Laya](#open-weights-alternative-laya) for self-hosted S1 · NOTES Patterns 3 / 8 / 10.

## Use case: Verbatim context compaction (fast-jev-compaction)

**Problem:** Built-in `/compact` asks an LLM to **summarize** old turns. Summaries are lossy — file paths, exact errors, constraints, and commands can disappear even when still needed.

**Pattern:** [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (`tamaratran/fast-jev-compaction`) — npm library + Claude Code plugin. Jev scores each historical **tool call** and **tool result** (`noul` keep-call / keep-result). Then: keep both, keep call + truncate result, or drop both. **User and assistant text stay verbatim and in order.** Newest `preserveRecentMessages` (default 6) and the first message are pinned. Claude Code hook swaps in the pruned transcript for `/compact` / auto-compact when reduction is enough; otherwise falls back to the built-in summary.

**Also:** LiteLLM documents a TypeSafe Jev compaction guardrail that can replace stale tool results with a short notice before the model call ([blog](https://docs.litellm.ai/blog/typesafe-jev-compaction)).

**Why it fits NOTES:** Same family as Pattern 7 (prune tool exhaust first) and a safer alternative to Pattern 6’s summary-only compact. Uses Jev as the cheap decision layer instead of a frontier summarize pass.

**Measure:** Library reports character `reductionRatio` and stats — **no universal token-% in the README**. Include Jev request cost when comparing on/off. Do not invent deck percentages.

**Install (Claude Code plugin, from README):**
```bash
# needs Claude Code 2.1.274+ function hooks
# ~/.claude/settings.json env: CLAUDE_CODE_ENABLE_FUNCTION_HOOKS=1, TYPESAFE_API_KEY=...
claude plugin marketplace add tamaratran/fast-jev-compaction
claude plugin install fast-jev-compaction@fast-jev-compaction
```
Or `npm install fast-jev-compaction` and call `compactMessages(...)` from your own harness.

## How it maps onto this playbook

| NOTES idea | Jev angle |
| --- | --- |
| Cheap-model routing | Route with Jev **or open-weights [Laya](https://github.com/NandhaKishorM/laya)**, then run Luna/Haiku vs Opus/Astra |
| Cheap orch + strong leaf | [minecraft-agent](https://github.com/rmalde/minecraft-agent): Astra JSON plan sparsely; Jev Choice over code-built `a0…an` actions |
| Computer-use GUI loops | [CUA-S1](https://github.com/trycua/cua) forms specialist (+ Jev/Laya-shaped S1): frontier plans; S1 scores local fill/check/skip — avoid screenshot→LLM every field |
| MCP / tool hygiene | Jev is **not** another fat MCP schema dump — call it from code/middleware with thin state |
| Subagents | Prefer Jev for tiny judgments; reserve subagents for real isolation / parallelism |
| Context prune / `/compact` | Prefer [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction) (keep/drop tools, verbatim text) over lossy LLM summary when tool exhaust dominates |
| Metering | Log `usage.input_tokens` from System One responses in the same weekly `ccusage` habit |
| Open-weights / offline decisions | [Laya](https://github.com/NandhaKishorM/laya) (`choice`/`score`/`noul`, Apache 2.0) as Jev-shaped alternative — fine-tune; route multilingual |
| STE100 / Concise | Orthogonal — Jev returns no narration to trim |

## Limits (do not ignore)

- No text generation, summarization, or free-form SQL/code inventing.
- Weak at math, counting, and date arithmetic — do that in code.
- State should be **small and relevant** (context rot still applies).
- Early access / waitlist; pin versioned model IDs when you tune thresholds.
- For an **open-weights** path, see [Laya](#open-weights-alternative-laya) — not a managed API; plan for GPU preload and fine-tuning.
- Author speed/cost multiples are **ceilings** — measure your pipeline.

## Install pointers (agents)

```bash
# Claude Code plugin
claude plugin marketplace add typesafe-ai/skills
claude plugin install typesafe@typesafe-ai

# Other agents
npx skills add typesafe-ai/skills --skill typesafe-ai
```

Suggested explore prompt (TypeSafe): ask the agent, with the skill loaded, where intelligent judgment can replace fragile parsing — then review questions/thresholds in one file before shipping.

## Weekly refresh

Re-check pricing, model aliases, LangChain middleware APIs, [fast-jev-compaction](https://github.com/tamaratran/fast-jev-compaction), [rmalde/minecraft-agent](https://github.com/rmalde/minecraft-agent), [Laya](https://github.com/NandhaKishorM/laya), LiteLLM Jev compaction, and new cookbooks each Monday with the rest of the token-savings notes. Also [CUA-S1 / trycua](https://github.com/trycua/cua) computer-use specialists.

