# Speaker notes — token-savings talk

Short talk track for Pong L. Keep claims attributed. Do not promise exact savings %.

## Open (about 30 seconds)

Agents mostly pay for re-reading their own past: history, tool output, and retries.
The clever next prompt is rarely the main cost.

## Walk-through (about 3–4 minutes)

1. **Meter without typing** — Use a live status line and a log tool like `ccusage`. Do not rely on typing `/context` every few minutes.
2. **Context picture** — System rules + MCP tools + memory + messages all share one window.
3. **MCP hygiene** — Cite Anthropic staff “67k+” multi-server setups and Spence’s before/after. Action: turn off unused servers; shorten schemas; defer load when you can.
4. **Sessions** — `/clear` between unrelated tasks. `/compact` mid-task with an explicit keep-list around 60% full, not at the ceiling.
5. **Standing docs** — Keep AGENTS.md / CLAUDE.md short. Load skills only when needed (Verma’s per-turn tax story).
6. **Models** — Cheap model for explore and plan. Strong model for hard implement. Do not switch mid-warm session (cache rebuild). Cite Anthropic’s Haiku-switch lesson and @anshuc Luna+Astra only as self-reported.
7. **Subagents** — Good when they keep noise out of the parent. Bad when you fan out blindly (~7× official / measured caution).
8. **Output** — Concise / STE100-style short sentences cut narration. No proven total-token % for STE100 alone.

## Close (about 30 seconds)

Stack two levers this week: MCP hygiene + clear between tasks.
Log cache hits and weekly `ccusage` totals.
On stage say “practitioners report…” and name the source. Never guarantee a percentage.

## See also

- Main playbook: `NOTES.md`
- Savings table and top patterns live there (ordered by impact).
