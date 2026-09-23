# Agent modes × context pollution

Companion to `NOTES.md`. Research date **2026-09-18 (Europe/London)**. Official docs preferred. No invented modes — gaps marked *not found*. Codex Astra/Sol cross-link added **2026-09-24**.

**Pollution** = tool exhaust (search hits, file dumps, shell/test logs, traces) that lands in the **main** conversation every later turn re-pays.

| Tag | Meaning |
| --- | --- |
| **Low** | Little/no tool exhaust in main, or child/subagent returns a summary only, or fresh-session handoff |
| **Medium** | Tools run in main but scoped (reads/search); plan text stays in main; writes gated |
| **High** | Full agent loop in main (edits + shell + tests + MCP); debug logs/traces; multi full windows |

**Isolation ≠ cheaper system-wide.** Protecting the parent can still raise *total* tokens (see Agent Teams ~7×).

---

## Mode glossary (what each mode means)

| Mode | What it does | Typical main-thread pollution |
| --- | --- | --- |
| **Ask / Chat (read-only)** | Answer questions; explore; no (or minimal) writes | **Low–Medium** — Medium if search/read tools dump into the same chat |
| **Plan** | Research + propose; defer implementation until approval | **Medium** in main, or **Low** if research is offloaded / plan file + fresh implement |
| **Agent / Build / Code** | Autonomous multi-step: read, edit, shell, iterate | **High** |
| **Edit / Manual** | Controlled edits (scoped files / accept diffs) | **Medium–High** |
| **Debug** | Instrument runtime, collect logs/traces, then fix | **High** — log/trace volume |
| **Explore (subagent)** | Read-only research in a **child** context; return summary | **Low** for parent |
| **Architect** | Design pass then apply (often dual-model) | **Medium–High** |
| **Review** | Read-only critique / PR review | **Low–Medium** |
| **Agent Teams / multi** | Multiple full sessions with own windows | **High system-wide** — isolation ≠ savings |
| **Permission / approval** | When to prompt (Auto-review, acceptEdits, bypass…) | **N/A** as session mode — do not confuse with Ask/Plan |

---

## Agent × Mode matrix

Legend: **Yes** = documented first-class · **Partial** = related / renamed / permission-adjacent · **—** = not found / unclear

| Agent | Ask/Chat RO | Plan | Agent/Build | Edit/Manual | Debug | Explore (sub) | Architect | Review | Teams / multi |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Cursor** | **Yes** (Ask) | **Yes** | **Yes** (Agent) | **—** (Manual largely removed) | **Yes** | **Yes** (subagents) | **—** | custom | Cloud / multi-agent (≠ Claude Teams) |
| **Claude Code** | **Partial** (`/btw`; Manual=reads-only permission) | **Yes** (`plan` permission) | **Yes** (default) | **Partial** (`acceptEdits`) | **—** as UI mode | **Yes** (Explore) | **—** | custom | **Yes** (Agent Teams, experimental) |
| **OpenCode** | **—** named Ask | **Yes** (Plan primary) | **Yes** (Build) | **—** | **—** | **Yes** (Explore) | **—** | custom | Child sessions (not Teams) |
| **Codex** | **—** named Ask; use `/permissions` RO | **Yes** (`/plan`) | **Yes** | **—** | **—** | **Yes** (subagents) | **—** | `/review` | Parallel `/agent` threads |
| **Windsurf / Cascade** | **Yes** (Ask) | **Yes** | **Yes** (Code) | **—** (legacy “Write”≈Code) | **—** | **—** | **—** | **—** | **—** |
| **Continue.dev** | **Yes** (Chat = **no tools**) | **Yes** (RO tools) | **Yes** (Agent) | **Yes** (Edit) | **—** | **—** | **—** | **—** | **—** |
| **Aider** | **Yes** (`ask`) | **Partial** (ask→code) | **Yes** (`code`) | **—** | **—** | **—** | **Yes** | **—** | **—** |
| **GitHub Copilot** | **Yes** (Ask) | **Yes** | **Yes** (Agent) | **Yes** (Edit) | **—** | **Yes** (own window) | **—** | custom | Subagents / custom |

---

## Which modes protect the main context? (ranked)

| Rank | Pattern | Protects main? | Watch-outs | Best examples |
| --- | --- | --- | --- | --- |
| 1 | **Explore / research subagent → summary only** | **Yes — best** | Large summaries still tax parent | Claude Explore/Plan subagents; Copilot subagents; OpenCode Explore; Cursor subagents |
| 2 | **Chat / Ask with no tools** | **Yes** | User-pasted blobs still pollute | Continue **Chat** |
| 3 | **Plan → implement in fresh session / @plan file** | **Yes** | Plan that keeps all research in one thread is weaker | Windsurf plan file + fresh implement; Cursor “each mode own context”; Copilot Plan → Start Implementation |
| 4 | **Read-only Ask/Plan in main (tools on)** | **Partial** | Read/search can still fill the window | Cursor Ask; Copilot Ask; Windsurf Ask; Claude `plan` without Explore |
| 5 | **Aider ask→code same thread** | **No** (intentional) | History retained on purpose | Aider |
| 6 | **Full Agent/Build/Code in main** | **No** | Highest ongoing pollution | All products’ default agent |
| 7 | **Debug / log-instrumentation** | **No** | Traces/logs in main | **Cursor Debug** (clearest case) |
| 8 | **Agent Teams / many full windows** | Isolates peers, **not** a saver | Official **~7×** tokens (Claude plan-mode teams) | Claude Agent Teams |

### Does not pollute the parent (prefer these)

- Claude / OpenCode / Copilot / Cursor **Explore-style subagents** (exhaust stays in child; summary returns)
- Continue **Chat** (no tools)
- Claude **`/btw`** (side question without adding to history)
- Windsurf **Plan file + @mention → fresh implement context**
- Cursor **mode switch = fresh context** (official: each mode has its own context)

### Does pollute (or multiplies spend)

- **Agent / Build / Code** in the main thread
- **Cursor Debug** (instrumentation + collected logs in the Debug chat)
- **Claude Agent Teams** in plan mode (~7× official) — isolation ≠ savings
- **Aider ask→code** in one conversation (by design)

---

## Per-agent notes (short)

### Cursor
Official modes: **Agent, Ask, Plan, Debug**. Ask = read-only, no edits. Debug adds log statements and reviews collected logs — highest log-exhaust risk among products surveyed. Subagents: own context window, return result to main. **“Each mode uses its own context, so switching modes starts a fresh context window.”** Do not confuse with Auto-review / Allowlist / Run Everything (approval modes). Manual largely removed from current docs.

Sources: [Ask](https://cursor.com/help/ai-features/ask-mode) · [Agent](https://cursor.com/help/ai-features/agent) · [Plan](https://cursor.com/docs/agent/plan-mode) · [Debug](https://cursor.com/docs/agent/debug-mode)

### Claude Code
Permission modes include **`plan`** (research, no source edits until approved) and Manual/default (reads without ask). Built-in **Explore** and **Plan** subagents keep exploration **out of the main conversation**. `/btw` answers without adding to history. Agent Teams: each teammate is a separate instance; official costs doc ~**7×** tokens when teammates run in plan mode. No Cursor-style Debug UI mode found.

Sources: [permission-modes](https://code.claude.com/docs/en/permission-modes) · [subagents](https://code.claude.com/docs/en/subagents) · [costs](https://code.claude.com/docs/en/costs)

### OpenCode
Primaries: **Build** (default, all tools) and **Plan** (restricted writes). **Explore** subagent = fresh child context. Scout was in v1 docs; **absent** from V2 builtins table (version gap). Tab switch = new context? **Unclear**.

Source: [opencode.ai/docs/agents](https://opencode.ai/docs/agents/)

### Codex
**Plan** via `/plan` or Shift+Tab — gather context and plan; not execute until ended. No named Ask mode; use `/permissions` read-only. Subagents offload bounded work. `/review` for diffs. Debug mode **not found**.

**Practitioner cross-link (not a product mode):** @BradGroux pins Astra for plan, review, and coordination, and GPT-5.6 Sol for implementation threads. See [ORCH Case 12](./ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux). **No token-%.** Parallel threads can raise total spend (rank 8 below).

Source: [best practices](https://developers.openai.com/codex/learn/best-practices)

### Windsurf / Cascade
**Code, Plan, Ask**. Plan can live in an external Markdown file; **@mention plan → continue with fresh context**. Ask cannot make changes.

Source: [Cascade modes](https://docs.devinenterprise.com/desktop/cascade/modes)

### Continue.dev
**Chat = no tools** (cleanest ask). Plan = RO tools. Agent = all tools. Edit = highlight-scoped.

Source: [how it works](https://docs.continue.dev/ide-extensions/agent/how-it-works)

### Aider
`/ask` never edits; `/code` edits; **architect** = two LLM requests (higher cost). Recommended ask↔code in **one** conversation — planning pollutes the same window on purpose.

Source: [modes](https://aider.chat/docs/usage/modes.html)

### GitHub Copilot Chat
Ask / Plan / Agent / Edit. Plan does not change code until approved; Start Implementation → Agent. Subagents: own context window, return final result without mixing contexts. Debug **not found**.

Source: [chat in IDE](https://docs.github.com/en/copilot/how-tos/chat-with-copilot/chat-in-ide)

---

## Practical defaults (token hygiene)

1. Prefer **Explore-style subagents** for grepping / test dumps / doc fetches.
2. Prefer **Plan that writes an external plan + new chat for Build** when the product supports it.
3. Treat **Agent Teams** as a spend multiplier, not a context saver for the lead.
4. Use **Continue Chat** or Claude **`/btw`** for side questions without growing history.
5. Avoid **Cursor Debug** (or any verbose log mode) when the goal is token hygiene — unless the bug needs runtime evidence.
6. Do not confuse **approval modes** with Ask/Plan — they change prompts, not isolation.

---

## Gaps

- Cursor Manual: removed / not in current official mode table
- Exact Plan→Build exhaust handoff in Cursor: partially documented
- OpenCode Tab = new context?: unclear
- OpenCode Scout: version-dependent
- Codex named Ask: not found
- First-class Debug Mode outside Cursor: not found
- Universal % token savings per mode: not published (only qualitative + Claude Teams ~7×)

Raw research scratch: `mode-matrix-research.md` (local workspace; not required in the private repo).
