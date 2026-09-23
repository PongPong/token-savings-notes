# token-savings-notes

Public playbook for cutting LLM and agent token spend.

## Two doors

| You use… | Start here |
| --- | --- |
| **Browser chat only** (ChatGPT / Claude.ai / Gemini / …) | [guides/browser-chat.md](./guides/browser-chat.md) |
| **CLI / coding agents** (Claude Code, Cursor, OpenCode, Codex, …) | [guides/cli-agents.md](./guides/cli-agents.md) |

Both doors share one **core**. The pattern map (browser action vs CLI action vs core link):

→ [guides/MAP.md](./guides/MAP.md)

### No-duplication rule

- Patterns, caveats, attributed figures, and quotes live **once** in core (`NOTES.md`, `SOURCES.md`, companions).
- Guides only apply the idea in that UI and **link** to core pattern IDs.
- `EXAMPLE-PROMPTS.md` stays one file — guides link to sections; do not paste full fences twice.
- Before merge: same quote/% in two files → delete from the guide, link to core.

## Core (shared source of truth)

| File | Role |
| --- | --- |
| [NOTES.md](./NOTES.md) | Do this week, **Savings cheat sheet** (single numbers board), Top patterns, metering, stubs |
| [SOURCES.md](./SOURCES.md) | Named quotes, link list, X primary delta |
| [EXAMPLE-PROMPTS.md](./EXAMPLE-PROMPTS.md) | Pasteable prompts / configs |
| [ORCH-CASES.md](./ORCH-CASES.md) | One vs multi-agent cases (incl. Astra plan + Sol leaf) |
| [MODE-MATRIX.md](./MODE-MATRIX.md) | Ask / Plan / Agent / Debug × context pollution |
| [TYPESAFE-JEV.md](./TYPESAFE-JEV.md) | TypeSafe Jev (+ open-weights Laya; Apple Silicon [laya-mlx](https://github.com/mizorewww/laya-mlx); CUA-S1 / Jev Ultrafast) |
| [SPEAKER-NOTES.md](./SPEAKER-NOTES.md) | Short talk track |

## Weekly refresh (Monday ~09:00 Europe/London)

1. New X/web tips → **SOURCES.md** only (credible accounts; include CN-language when useful).
2. New pattern/figure → **NOTES.md** Savings cheat sheet + pattern section **once**.
3. Update [guides/MAP.md](./guides/MAP.md) row or a **thin** guide lever if the UI action changed.
4. Companions only if content moved (ORCH / MODE / Jev / prompts / speaker).
5. **Never** maintain twin encyclopedias in `guides/`.

Scout rules: creditable X only; not eng-only; include Chinese users; no invented %.

Last edit: 2026-09-24.
