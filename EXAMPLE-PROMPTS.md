# Example prompts (real-world)

Companion to `NOTES.md`. Full pasteable blocks live here so the main playbook stays short.

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

- **Label:** paraphrased (config values from Claude Code docs / Rulestack measurement)
- **Source:** Claude Code Tool Search env; Rulestack / Jo Do measurement https://dev.to/rulestack/tool-search-hid-40170-tokens-from-our-first-request-auto5-put-them-all-back-2jin
- **Prompt:**
```json
"env": { "ENABLE_TOOL_SEARCH": true }
```
Values: **unset** / **`true`** = defer schemas (prefer this when the goal is a small first prompt). **`false`** = load all tool defs upfront. **`auto`** / **`auto:N`** = threshold vs **context-window size** (default `auto` = 10%). On a 1M window, `auto:5` is 50k — ~40k defs can load upfront and undo Tool Search. Claude Code may disable Tool Search on non-first-party `ANTHROPIC_BASE_URL` / older models; leave it on for first-party hosts.
- **When:** Claude Code `settings.json` when MCP/tool defs bloat the first request. Prefer default unset/`true`; use `auto`/`auto:N` only if you understand the % of *your* window.

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

**Count:** 27 prompts (1: 3 · 2: 5 · 3: 5 · 4: 4 · 5: 3 · 6: 5 · 7: 5). The Astra/Sol block below is three skeletons. It is not three more full fences, and it is not the article text.

### Astra/Sol orchestration templates (skeleton)

**Source:** [@BradGroux](https://x.com/BradGroux/status/2102455199583334625) (Brad Groux) — [I Think I’ve Been Using GPT-6 Astra Wrong](https://x.com/i/article/2102449008232861696). Shown 6:49 PM · 22 Sep 2026 (author-local display). Adapt these stubs to your repo. The full templates are in that article. **No token-%.** Case: [ORCH-CASES Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux).

**1. Read the project and divide the work** — Astra

```text
Read the repo docs, the code, and the open issues.
Build a dependency-ordered plan.
Name 3–5 independent tasks: outcome, scope, owned files, guardrails, checks.
Keep the current UI. Raise product decisions.
Continue items that do not need those answers.
```

**2. Delegate to Sol Medium**

```text
Delegate ready tasks to GPT-5.6 Sol with Medium reasoning.
State the boundaries, the files owned, and the rules that apply.
Serialize tasks that share a migration or an interface.
Require evidence and a focused pull request.
```

**3. Finish the workflow and keep moving** — Astra

```text
Review the diff and the evidence. Fix findings.
Merge when that merge is already authorized. Update the preview.
A requested draft stays a draft.
Start the next ready task during review.
Use a quiet scheduled check-in. Notify only for a decision or a meaningful result.
```

### INITIAL.md (context-engineering skeleton)

**Source:** [coleam00/Context-Engineering-Intro](https://github.com/coleam00/Context-Engineering-Intro) — `INITIAL.md` / `INITIAL_EXAMPLE.md`. Not a Facebook share.

```markdown
## FEATURE:
[Specific functionality and requirements — not a one-liner]

## EXAMPLES:
[Point at files under examples/ and what to mimic (structure, not blind copy)]

## DOCUMENTATION:
[API docs, library guides, MCP resources, schemas]

## OTHER CONSIDERATIONS:
[Auth, rate limits, gotchas AI assistants commonly miss]
```

Then: `/generate-prp INITIAL.md` → `/execute-prp PRPs/….md` (prefer separate sessions if research is noisy). Full workflow in NOTES **Context engineering (PRP workflow)**.

### json-render catalog prompt (skeleton)

**Source:** [vercel-labs/json-render](https://github.com/vercel-labs/json-render) — `catalog.prompt()` builds a full system prompt from your Zod catalog. Skeleton only. **No savings %.**

```text
Emit UI as JSON constrained to this catalog. Do not emit freeform HTML, JSX, or markdown UI.

Components: <names + one-line props>
Actions: <names>

Return a spec { root, elements }. Each element has type, props, children.
Use only listed types. Prefer the smallest spec that solves the ask.
```

