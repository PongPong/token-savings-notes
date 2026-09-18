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

## How it maps onto this playbook

| NOTES idea | Jev angle |
| --- | --- |
| Cheap-model routing | Route with Jev, then run Luna/Haiku vs Opus/Astra |
| MCP / tool hygiene | Jev is **not** another fat MCP schema dump — call it from code/middleware with thin state |
| Subagents | Prefer Jev for tiny judgments; reserve subagents for real isolation / parallelism |
| Metering | Log `usage.input_tokens` from System One responses in the same weekly `ccusage` habit |
| STE100 / Concise | Orthogonal — Jev returns no narration to trim |

## Limits (do not ignore)

- No text generation, summarization, or free-form SQL/code inventing.
- Weak at math, counting, and date arithmetic — do that in code.
- State should be **small and relevant** (context rot still applies).
- Early access / waitlist; pin versioned model IDs when you tune thresholds.
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

Re-check pricing, model aliases, LangChain middleware APIs, and new cookbooks each Monday with the rest of the token-savings notes.
