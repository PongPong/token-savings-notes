# Pattern map — browser chat vs CLI agents

**Rule:** Patterns, quotes, and % live once in core ([NOTES.md](../NOTES.md), [SOURCES.md](../SOURCES.md), [EXAMPLE-PROMPTS.md](../EXAMPLE-PROMPTS.md)).  
Guides only say **what to click / type in that UI**. If a figure appears twice, delete it from the guide and link here.

| ID | Core pattern (NOTES) | Browser chat action | CLI / coding-agent action |
| --- | --- | --- | --- |
| P1 | [MCP / tool hygiene](../NOTES.md#top-patterns-by-savings-impact) | N/A (no MCP). Disable unused **GPTs / apps / connectors** in the product settings. | Disable unused MCP servers; short schemas; Tool Search / defer load. |
| P2 | [Fresh session / clear](../NOTES.md#top-patterns-by-savings-impact) | **New chat** when the task changes. Do not stack unrelated asks in one thread. | `/clear` or new session between unrelated tasks. |
| P3 | [Cheap-model routing](../NOTES.md#top-patterns-by-savings-impact) | Use the product **model picker**: cheap/fast for triage; frontier only for hard writing/code. | Pin Explore/leaf to Haiku (etc.); cold-boundary route; no mid-warm `/model`. |
| P4 | [Lean standing instructions](../NOTES.md#top-patterns-by-savings-impact) | Keep **Custom instructions / Project instructions** short. Put task detail in the message, not always-on. | Thin `AGENTS.md` / `CLAUDE.md`; skills on demand. |
| P5 | [Prompt-cache hygiene](../NOTES.md#top-patterns-by-savings-impact) | Keep long fixed instructions at the **top** of Project/custom text; don’t reshuffle them every turn. | Stable system+tools prefix; don’t shuffle tools mid-session. |
| P6 | [Context compaction](../NOTES.md#top-patterns-by-savings-impact) | No `/compact`. Start a **new chat** and paste a short keep-list (decisions, errors, files). | `/compact` ~60% util with an explicit keep-list. |
| P7 | [Context pruning](../NOTES.md#top-patterns-by-savings-impact) | Don’t paste whole logs/PDFs. Paste **only the failing slice**. | Prune tool exhaust first; DCP / fast-jev-compaction when available. |
| P8 | [Targeted exploration](../NOTES.md#top-patterns-by-savings-impact) | Attach **one file** or quote a short span. Ask for a path before a dump. | Narrow `@files`; `rg` / tgrep / graph MCP; stop early. |
| P9 | [Short / STE100 output](../NOTES.md#top-patterns-by-savings-impact) | Ask for **bullets / STE100**. “No preamble.” | Output-style / Concise rules — [EXAMPLE-PROMPTS](../EXAMPLE-PROMPTS.md). |
| P-CU | [Computer-use hygiene](../NOTES.md#computer-use-token-hygiene) / [CUA-S1 §10](../TYPESAFE-JEV.md#10-computer-use--specialist-system-one-for-local-gui-decisions-cua-s1) / [Jev Ultrafast §11](../TYPESAFE-JEV.md#11-browser-use-jev-ultrafast--indexed-dom--jev-optarget) | Rare in pure chat. Prefer vendor “fill form” / connectors over pasting screenshots. | Frontier plans; S1/Jev scores local actions; structured DOM ≫ screenshots; skip filled fields. Live web: indexed element table + Jev op/target (jev-ultrafast). |
| P-UI | [Generative UI / json-render](../NOTES.md#generative-ui-token-hygiene-json-render) | Rare in pure chat. If the product must emit UI, prefer a **small component catalog** over pasting HTML. | Prefer **catalog JSON** (Zod/schema) over freeform HTML/JSX/markdown UI dumps. |
| P-OW | [Open-weights / offline](../TYPESAFE-JEV.md#open-weights-alternative-laya) · [laya-mlx](../TYPESAFE-JEV.md#apple-silicon-laya-mlx) | N/A (local runtime). | On Apple Silicon: `pip install laya-mlx`, then `laya-mlx predict` for local Laya-shaped decisions. Figures stay in TYPESAFE-JEV. |
| P10 | [Subagents / workers](../NOTES.md#top-patterns-by-savings-impact) | Rare in pure chat. Use a **second chat** for research; paste only the summary back. | Explore/Plan subagents; return summary only. Isolation ≠ cheaper (see Agent Teams). |
| P-AS | [Astra plans + Sol implements](../ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux) | No Codex thread pin. Optional: a frontier chat plans; a separate chat does one scoped task; paste the summary back. | Codex: pin **Astra** for plan/review/coord and **GPT-5.6 Sol** (Medium) for leaf impl. [Case 12](../ORCH-CASES.md#case-12-astra-plans-sol-implements-brad-groux). **No token-%.** |

**Also:** [MODE-MATRIX.md](../MODE-MATRIX.md) (Ask/Plan/Agent/Debug) · [ORCH-CASES.md](../ORCH-CASES.md) · [TYPESAFE-JEV.md](../TYPESAFE-JEV.md) · [SPEAKER-NOTES.md](../SPEAKER-NOTES.md).

Last edit: 2026-09-24.
