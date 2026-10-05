<!--------------- /qompassai/daelin/README.md -------------->
<!-------------------- Qompass AI Daelin -------------------->
<!-- Copyright (C) 2026 Qompass AI, All rights reserved -->
<!-- ----------------------------------------------------->

<h1 align="center">Qompass AI 대리인</h1>

<h2 align="center">Daelin — learning the agent protocols by studying real implementations</h2>

**대리인** (daeriin) is the Korean word for *agent* — one who acts on another's behalf.
This repository is exactly that idea, taught: a guided tour of the two protocols
that define how AI agents act in 2026 — **MCP** (Model Context Protocol) and **A2A**
(Agent-to-Agent) — built around real, working implementations rather than abstract
spec summaries.

## ⏱ Start here: the interactive timeline

<div align="center">

### [Open the timeline →](https://qompassai.github.io/daelin/timeline.html)

*The full MCP → A2A chronology, 2024 to today — every entry linked to its primary
source. **Read it first, in order** — everything in this repo follows the same
chronology. Once you've walked the timeline, the chapters below will feel like
a deeper cut of a story you already know.*

</div>

<details>
<summary><p align="center"><strong>📖 Contents — 9 chapters + glossary (click to expand)</strong></p></summary>

| Chapter | File | What you'll learn |
|---------|------|-------------------|
| 1 | [docs/01-standards-timeline.md](docs/01-standards-timeline.md) | The full MCP/A2A chronology, 2024–2026, sourced |
| 2 | [docs/02-mcp-client.md](docs/02-mcp-client.md) | How an MCP client works: transport, sessions, registry, tools |
| 3 | [docs/03-mcp-server.md](docs/03-mcp-server.md) | How an MCP server works: the headless stdio side |
| 4 | [docs/04-mcp-security.md](docs/04-mcp-security.md) | Vetting, least privilege, and the trust model |
| 5 | [docs/05-a2a.md](docs/05-a2a.md) | Agent Cards, discovery, tasks, and multi-agent fan-out |
| 6 | [docs/06-rose-phlow.md](docs/06-rose-phlow.md) | Case study: editor ↔ agent-runtime over MCP |
| 7 | [docs/07-salesforce-mcp.md](docs/07-salesforce-mcp.md) | Case study: Salesforce hosted + DX MCP servers |
| 8 | [docs/08-best-practices.md](docs/08-best-practices.md) | The 2026 consensus checklist, with sources |
| 9 | [docs/09-skill-spec.md](docs/09-skill-spec.md) | The Agent Skills open specification: SKILL.md format, progressive disclosure |
| — | [docs/glossary.md](docs/glossary.md) | Terms, defined once |

**New to all of this?** Walk the [timeline](https://qompassai.github.io/daelin/timeline.html) first, then read chapters 1 → 2 → 5 → 8.
**Here for the code?** Chapters 2–4 and 6–7 are the implementation deep dives.

</details>

<details>
<summary><p align="center"><strong>What this is (click to expand)</strong></p></summary>

Most protocol documentation describes what the spec *says*. Daelin shows what the
spec *means* by walking through code that actually implements it:

- **The standards timeline** — how MCP and A2A evolved from 2024 to today, with
  primary sources for every claim. Start here if you're new.
- **A working MCP client** — stdio transport, JSON-RPC framing, session management,
  server registry, tool discovery — as built for the Diver Neovim environment.
- **A working MCP server** — the other side of the same wire: a headless process
  speaking newline-delimited JSON-RPC.
- **A working A2A implementation** — Agent Cards, discovery, task lifecycle,
  fan-out orchestration across peer agents.
- **The security model** — why tool descriptions are untrusted input, how to vet
  them, and where the confirmation gates go.
- **Production case studies** — including wiring Salesforce's hosted MCP servers
  into an editor-native agent.

Each chapter opens with the plain-language idea, then goes into the mechanism.
No prior protocol knowledge is assumed; curiosity is.

</details>

<details>
<summary><p align="center"><strong>⚙ Working configurations (click to expand)</strong></p></summary>

This repo ships the real thing, not just the explanation:

- **[`agents/`](agents/)** — agent runtime configurations (Claude Code, Codex,
  OpenCode, OpenShell): settings, permissions, and project instructions as
  actually used.
- **[`skills/`](skills/)** — the working skill library: thirteen Tiger Style
  language skills plus `mcp-builder`, `skill-creator`, and more, each following
  the [Agent Skills open specification](https://agentskills.io/specification).
  See [chapter 9](docs/09-skill-spec.md) for the spec itself — the SKILL.md
  format, progressive disclosure, and why skills compose with MCP rather than
  competing with it.

</details>

<details>
<summary><p align="center"><strong>The one-paragraph version (click to expand)</strong></p></summary>

MCP and A2A are complementary layers, not competitors. **MCP is the vertical
connection**: one agent reaching down through MCP clients into MCP servers to
call tools, read resources, and load prompts. **A2A is the horizontal
connection**: one agent reaching sideways to another independently-deployed
agent — discovered via a signed Agent Card — to delegate tasks and exchange
results. A real system uses both: tools over MCP, delegation over A2A. Since
2026 both protocols share a governance home in the Linux Foundation's Agentic
AI Foundation, but their specs remain separate.

</details>

<details>
<summary><p align="center"><strong>Sources and honesty (click to expand)</strong></p></summary>

The standards timeline (chapter 1) was researched from live primary sources on
2026-09-30 — spec changelogs, official announcements, foundation press
releases. Every entry carries its source; claims that couldn't be verified or
where sources disagreed are labeled as such rather than smoothed over. If you
spot something stale, that's a bug — the protocols move fast.

</details>

## License

See [LICENSE](./LICENSE) (repository template license files retained from the
project scaffold).

---

*Daelin is a Qompass AI educational project. The implementations described here
live in the [Diver](https://github.com/qompassai/diver) Neovim configuration
and the [phlow](https://github.com/qompassai/phlow) agent runtime.*
