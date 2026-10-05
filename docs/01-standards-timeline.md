# Chapter 1 — The Standards Timeline: MCP and A2A, 2024–2026

*Researched from live primary sources on 2026-09-30. Every entry carries its
source. Claims marked [secondary] come from corroborating coverage rather than
a loaded primary page; claims marked [disputed] had sources that disagreed.
An interactive version of this timeline ships as [`timeline.html`](../timeline.html).*

## The shape of the story

Two protocols, two directions. **MCP** (Model Context Protocol, created by
Anthropic) standardizes how *one agent* reaches *down* into tools and data.
**A2A** (Agent-to-Agent, created by Google) standardizes how *one agent* reaches
*sideways* to *another agent*. They were designed as complements — Google said
so explicitly at A2A's launch — and in 2026 they came under one governance
roof while remaining separate specs.

## Timeline

### 2024 — MCP is born

| Date | Event | Source |
|------|-------|--------|
| 2024-11-05 | MCP specification first published (v2024-11-05): JSON-RPC 2.0, hosts/clients/servers; server features (tools, resources, prompts); client features (roots, sampling); transports stdio and HTTP+SSE. | [spec changelog](https://modelcontextprotocol.io/specification/2025-03-26/changelog) |
| 2024-11-25 [secondary] | Anthropic open-sources MCP: the spec, Python/TypeScript SDKs, Claude Desktop local server support, and an open-source server repo (Google Drive, Slack, GitHub, Git, Postgres, Puppeteer). Created by David Soria Parra and Justin Spahr-Summers. Early adopters: Block, Apollo; dev-tool partners: Zed, Replit, Codeium, Sourcegraph. | [Anthropic announcement](https://www.anthropic.com/news/model-context-protocol) |
| 2024-12-19 | Anthropic publishes "Building effective agents": the workflows-vs-agents taxonomy (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer, autonomous agents). Core advice: simplest solution first, extensive sandboxed testing, pause for human feedback at checkpoints. | [Anthropic engineering](https://www.anthropic.com/engineering/building-effective-agents) |

### 2025 — A2A arrives, both protocols harden

| Date | Event | Source |
|------|-------|--------|
| 2025-01 [secondary] | OpenAI publishes "A Practical Guide to Building Agents" (30-page PDF): agent = model + tools + instructions; single- vs multi-agent orchestration; guardrails; "set up evals to establish a performance baseline." | [OpenAI PDF](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| 2025-03-26 | MCP spec 2025-03-26. Headlines: OAuth 2.1 authorization framework; HTTP+SSE replaced by **Streamable HTTP**; JSON-RPC batching (later removed); tool annotations (read-only/destructive hints). | [spec changelog](https://modelcontextprotocol.io/specification/2025-03-26/changelog) |
| 2025-03 [secondary] | MCP maintainers signal intent to build a central registry. OpenAI adopts MCP across products. | [MCP blog](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/) |
| 2025-04-09 | **Google announces A2A** on the Google Developers Blog: open protocol for agents from different vendors to collaborate; 50+ launch partners; explicitly "complements Anthropic's Model Context Protocol." Core concepts: Agent Card discovery, task lifecycle, messages, UX negotiation via parts. | [Google Developers Blog](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/) |
| 2025-05-27 | A2A spec v0.2.1: authenticated extended cards, referenceTaskIds. | [A2A changelog](https://github.com/a2aproject/a2a/blob/HEAD/CHANGELOG.md) |
| 2025-06-18 | MCP spec 2025-06-18. Headlines: batching removed; structured tool output; servers classified as OAuth Resource Servers; clients MUST implement Resource Indicators (RFC 8707) so malicious servers can't steal tokens; new Security Best Practices page; elicitation; `MCP-Protocol-Version` header. | [spec changelog](https://modelcontextprotocol.io/specification/2025-06-18/changelog) |
| 2025-06-23 | Google donates A2A to the **Linux Foundation** (AWS, Cisco, Google, Microsoft, Salesforce, SAP, ServiceNow founding). 100+ companies supporting. | [Google Developers Blog](https://developers.googleblog.com/en/google-cloud-donates-a2a-to-linux-foundation/) |
| 2025-07-30 | A2A v0.3.0 (breaking): Agent Card well-known URI changes `agent.json` → `agent-card.json`; signed cards; mTLS. | [A2A changelog](https://github.com/a2aproject/a2a/blob/HEAD/CHANGELOG.md) |
| ~2025-08-25 [secondary] | IBM's Agent Communication Protocol (ACP, BeeAI team) merges into A2A; the ACP repo is archived. A2A consolidates as the de-facto agent-to-agent standard. | secondary (multiple sources agree) |
| 2025-09-08 | **MCP Registry launches in preview**: open catalog + API for discovering public MCP servers at registry.modelcontextprotocol.io. | [MCP blog](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/) |
| 2025-09-16 [disputed: 09-17 / 09-29] | Google announces the **Agent Payments Protocol (AP2)**: open standard for agents to transact via cryptographically signed mandates; 60+ launch partners. Built on A2A + MCP. | [Linux Foundation press](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year) |
| 2025-11-25 | MCP spec 2025-11-25 ("One Year of MCP"): OIDC discovery for auth servers; tool/resource icons; incremental scope consent; tool-name guidance; OAuth Client ID Metadata Documents; experimental Tasks (durable async). | [spec changelog](https://modelcontextprotocol.io/specification/2025-11-25/changelog) |
| 2025-12-09 [secondary] | Anthropic donates MCP to the **Agentic AI Foundation (AAIF)**, a Linux Foundation directed fund co-founded by Anthropic, Block, and OpenAI. MCP joins Block's goose and OpenAI's AGENTS.md as founding projects. Claimed: 10,000+ public servers, 97M+/month SDK downloads. | [Anthropic announcement](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation) |

### 2026 — Stable specs, shared governance

| Date | Event | Source |
|------|-------|--------|
| 2026-03-12 | **A2A v1.0.0** — first stable spec (breaking vs 0.x): protocol separated from transport bindings (JSON-RPC, REST, gRPC); modernized OAuth 2.0; native multi-tenancy; signed Agent Cards; `tasks/list`. | [A2A changelog](https://github.com/a2aproject/a2a/blob/HEAD/CHANGELOG.md) |
| 2026-04-09 | Linux Foundation one-year A2A press release: 150+ organizations, 22,000+ GitHub stars, SDKs in five languages, native integration in Azure AI Foundry / Copilot Studio / Amazon Bedrock AgentCore. Restates the complementarity: "A2A defines how agents communicate across organizational boundaries, while MCP defines how agents connect to internal tools and data sources." | [Linux Foundation press](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year) |
| 2026-05-26 | A2A v1.0.1 (patch). Latest 1.x as of research date. | [A2A changelog](https://github.com/a2aproject/a2a/blob/HEAD/CHANGELOG.md) |
| 2026-07-28 | MCP spec 2026-07-28 — "the largest revision since launch": **stateless core** (sessions and the `initialize` handshake removed; every request carries version + capabilities in `_meta`); new `server/discover` RPC; `ping`/`logging/setLevel` removed; experimental Tasks moved to an official extension; Multi Round-Trip Requests replace sampling + elicitation; SSE resumability removed. | [spec changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) |
| 2026-08-17 | A2A accepted as a Growth Stage project at AAIF — **both protocols now share a governance home**, specs remaining separate. | [A2A blog](https://a2a-protocol.org/latest/blog/2026/08/27/a-new-chapter-for-a2a-joining-the-agentic-ai-foundation/) |

## Adjacent efforts (one line each)

- **IBM ACP** — merged into A2A ~Aug 2025; historical, do not adopt independently. [secondary]
- **Coral Protocol** — decentralized "Internet of Agents" with on-chain payments; an economic layer, not a wire-protocol competitor. [secondary]
- **ANP (Agent Network Protocol)** — community decentralized networking with DID identity; low adoption vs A2A. [secondary]
- **AP2 (Agent Payments Protocol)** — the commerce layer above A2A+MCP; donated to FIDO Alliance Apr 2026. [secondary]

## Why this matters for the rest of this repo

Everything that follows is an implementation of ideas from this timeline:
chapters 2–4 implement MCP (client, server, security), chapter 5 implements
A2A, chapter 6 shows them composed in a real system. When the timeline says
"stateless core" or "signed Agent Cards," you'll see what those decisions cost
— and buy — in actual code.
