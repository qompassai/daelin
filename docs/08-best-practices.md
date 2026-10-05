# Chapter 8 — Best Practices: The 2026 Consensus

*Distilled from the canonical vendor guides and OWASP, researched 2026-09-30.
Each practice carries its best primary source — verify, don't trust.*

The vendor guides this synthesizes: Anthropic's ["Building effective
agents"](https://www.anthropic.com/engineering/building-effective-agents),
OpenAI's ["A Practical Guide to Building
Agents"](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf),
Google's [ADK docs](https://google.github.io/adk-docs), and the [OWASP Top 10
for Agentic Applications
2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/).

| # | Practice | In two sentences | Source |
|---|----------|------------------|--------|
| 1 | **Evals first** | Set up evaluations to establish a performance baseline *before* optimizing cost or latency. Add complexity only when it demonstrably improves measured outcomes. | [OpenAI guide](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf) |
| 2 | **Least-privilege tool access** | Give the agent only the tools it needs. MCP requires explicit user consent before tool invocation and supports incremental OAuth scope consent. | [MCP spec](https://modelcontextprotocol.io/specification/2026-07-28) |
| 3 | **Human-in-the-loop / approval gates** | Agents should pause for human feedback at checkpoints or when encountering blockers. Keep agents safe, predictable, and effective with explicit guardrails. | [Anthropic](https://www.anthropic.com/engineering/building-effective-agents) |
| 4 | **Sandboxing and network egress control** | Test extensively in sandboxed environments with appropriate guardrails. Constrain which networks and data an agent can reach before production. | [Anthropic](https://www.anthropic.com/engineering/building-effective-agents) |
| 5 | **Prompt-injection defenses** | Treat tool descriptions as untrusted input (chapter 4 is the implementation of this sentence). Validate agent actions against declared intent. | [MCP spec](https://modelcontextprotocol.io/specification/2026-07-28) + [OWASP](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) |
| 6 | **Secret handling** | Never embed static secrets in discovery documents — use out-of-band dynamic credentials. Pass credentials via standard HTTP auth headers, not protocol messages. | [A2A docs](https://a2a-protocol.org/latest/topics/agent-discovery/) |
| 7 | **Observability / tracing** | Propagate trace context across service boundaries (MCP 2026-07-28 documents OpenTelemetry conventions). You can't secure or debug what you can't see. | [MCP changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) |
| 8 | **Deterministic builds / pinned dependencies** | Pin and audit the agent supply chain: models, MCP servers, skills, SDK versions. Plan upgrades deliberately against the protocols' deprecation policies. | [OWASP](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) |
| 9 | **Staged rollouts** | Ship behind version negotiation with defined migration paths. Roll out new agent versions incrementally, with evals gating each stage. | [Linux Foundation](https://www.linuxfoundation.org/press/a2a-protocol-surpasses-150-organizations-lands-in-major-cloud-platforms-and-sees-enterprise-production-use-in-first-year) |

## How Daelin's implementations map to this list

| Practice | Where it's implemented here |
|----------|----------------------------|
| Evals first | Half of every test suite is adversarial by policy (chs. 2–3) |
| Least privilege | Scoped MCP servers, risk flags, minimal toolsets (chs. 2, 7) |
| Human-in-the-loop | Confirmation chokepoint on every tool call (ch. 2), operator approvals (ch. 6) |
| Sandboxing | argv-only spawning, bounded everything (chs. 2–4) |
| Prompt-injection defenses | `mcp_vet.lua` — the full chapter 4 |
| Secret handling | Env-only credentials, never in files or cards (chs. 2, 5) |
| Observability | stderr discipline keeps protocol bytes parseable (ch. 3) |
| Pinned dependencies | Protocol version negotiation, registry validation (ch. 2) |
| Staged rollouts | Idempotent setup, lazy server start, atomic registry writes (ch. 2) |

The checklist isn't aspirational — it's a description of what the preceding
seven chapters already do. That's the point of building the practices into
the architecture instead of bolting them on after.
