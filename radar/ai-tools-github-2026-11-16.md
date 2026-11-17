# AI tools on GitHub, 16 Nov 2026

Source: [@themianoo on X](https://x.com/themianoo/status/2122293999096779181), posted 16 Nov 2026, 11:41 Europe/London. The post ranked these ten repos by monthly star gain. This file sorts them by GitHub stars, highest first.

Star counts checked with the GitHub API on 17 Nov 2026, 10:33 Europe/London. Monthly gains are the post's figures, not a new measurement. For T3 Code the post says "nearly +33k".

| # | Tool | Stars | Monthly gain (post) | What it does |
|---|------|------:|--------------------:|--------------|
| 1 | [Matt Pocock's Skills](https://github.com/mattpocock/skills) | 326,294 | +34k | AI skills a TypeScript teacher uses on real work. Best known: `grill-me`, which questions you about the goal before it starts. Installs into Codex with one command. |
| 2 | [Orca](https://github.com/stablyai/orca) | 127,238 | +33k | Workbench for AI coding. Runs agents such as Claude Code and Codex at once, each in its own workspace. Splits a large task, hands out the parts, then merges the results. |
| 3 | [taste-skill](https://github.com/Leonxlnx/taste-skill) | 117,257 | +20k | Gives an agent design taste for company sites (the post compares this to Figma or x.ai): one block per screen, clean alignment, real photos. Checks the page in a real browser at desktop, tablet and phone widths. |
| 4 | [VoiceStudio](https://github.com/debpalash/VoiceStudio) | 112,281 | +46k | Local AI voice studio. Clone a voice from a clean sample, design a new voice from a text description, dub videos, transcribe, and read audiobooks. 646 languages, free, runs offline, no account. The author says to get consent before cloning someone's voice. |
| 5 | [Impeccable](https://github.com/pbakaus/impeccable) | 111,712 | +29k | Flags the "made by AI" look in web pages (Inter font, purple-blue gradients, cards inside cards). Say "bolder" or "quieter" to adjust. Has 62 deterministic checks that flag AI style with no model. |
| 6 | [HyperFrames](https://github.com/heygen-com/hyperframes) | 85,542 | +21k | From HeyGen. Turns web pages into video: describe a video to Claude Code, it writes the page, animates it and exports MP4. There is also a free desktop app. |
| 7 | [security-audit-skill](https://github.com/cloudflare/security-audit-skill) | 74,291 | +43k | From Cloudflare. Turns an agent into a security audit team: maps the project, sends several agents to hunt bugs, has a fresh agent try to disprove each finding, then reports confirmed, needs checking, and ruled out. |
| 8 | [T3 Code](https://github.com/pingdotgg/t3code) | 63,405 | nearly +33k | One interface for Claude Code, Codex and Cursor on your own subscriptions. Start a task on your computer and approve the next step from your phone. |
| 9 | [Hallmark](https://github.com/Nutlope/hallmark) | 56,482 | +25k | From Together AI. 50 ready design themes (paper, fonts, colours, texture), layout based on the page's purpose, checked in a real browser. `hallmark redesign` restyles an existing page without changing the content. |
| 10 | [agent-desktop](https://github.com/vercel-labs/agent-desktop) | 24,808 | +22k | From Vercel Labs. Lets an agent use desktop apps by reading every button and element from the accessibility tree and giving each a ref, instead of guessing from screenshots, then re-reads to confirm the click. Mac, Windows and Linux. |

## Token-saving angles

These claims come from each project's README. They are vendor or author claims. Neither README gives a savings percentage.

- **agent-desktop** ([README](https://github.com/vercel-labs/agent-desktop)): gives an agent computer use "without screenshots or pixel matching". It reads the accessibility tree and returns JSON, with a ref for every element the agent can act on. The agent acts on a ref, then reads the UI again to check the result. The tool has no model inside. See [Computer-use token hygiene](../NOTES.md#computer-use-token-hygiene).
- **Impeccable** ([README](https://github.com/pbakaus/impeccable)): its slop detector is "62 deterministic rules" that flag common AI-style tells and quality problems. The README says "No LLM, no API key."
