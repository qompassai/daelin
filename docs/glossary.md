# Glossary

Terms used across Daelin, defined once.

- **A2A (Agent-to-Agent)** — Google's open protocol (donated to the Linux
  Foundation, 2025) for independently-deployed agents to discover and
  delegate to each other. The *horizontal* layer. See chapter 5.
- **AAIF (Agentic AI Foundation)** — Linux Foundation directed fund (2025)
  governing MCP, A2A, goose, and AGENTS.md under one roof.
- **Agent Card** — A2A's discovery document: a JSON file at
  `/.well-known/agent-card.json` describing an agent's identity, endpoint,
  skills, and auth. Validated on fetch, never executed. See chapter 5.
- **AP2 (Agent Payments Protocol)** — open standard for agents to transact
  via signed mandates; built on A2A + MCP. The commerce layer above the
  messaging protocols.
- **Artifact** — in A2A, a typed output of a task (as opposed to a message,
  which carries conversation). See chapter 5.
- **Elicitation** — MCP mechanism where a server requests information from
  the user mid-interaction (being superseded by MRTR in the 2026 spec).
- **Fan-out** — delegating one task to multiple peer agents and reconciling
  their artifacts. See chapter 5.
- **Host / Client / Server (MCP)** — the host is the application the user
  interacts with (e.g., an editor); the client is the protocol endpoint
  inside the host that manages a server connection; the server exposes
  tools, resources, and prompts. One host, many clients, many servers.
- **JSON-RPC 2.0** — the message envelope both MCP and A2A use: requests
  with `id`/`method`/`params`, responses with `result`/`error`,
  notifications with neither `id` nor response.
- **MCP (Model Context Protocol)** — Anthropic's open protocol (2024) for
  agents to connect to tools and data. The *vertical* layer. See chapters
  1–4.
- **MRTR (Multi Round-Trip Requests)** — the MCP 2026-07-28 pattern
  replacing sampling + elicitation: structured multi-turn server↔client
  exchanges.
- **Prompt (MCP)** — a reusable instruction template a server exposes;
  distinct from a chat prompt.
- **Resource (MCP)** — read-only data a server exposes (files, schemas,
  documents); distinct from tools, which *do* things.
- **Roots** — MCP client feature advertising which filesystem roots the
  server may reference. (Deprecated in the 2026-07-28 spec lifecycle.)
- **Sampling** — MCP mechanism where a server asks the client (the model)
  to generate text. Refused by default in this repo's client (ch. 2).
- **SSE (Server-Sent Events)** — the streaming transport MCP used for HTTP
  before Streamable HTTP; resumability removed in the 2026-07-28 revision.
- **Stdio transport** — MCP over a process's stdin/stdout with
  newline-delimited JSON-RPC. The simplest transport; what this repo's
  implementations use. See chapters 2–3.
- **Streamable HTTP** — the MCP HTTP transport since spec 2025-03-26:
  POST JSON-RPC, stream responses. What Salesforce's hosted servers use
  (ch. 7).
- **Tool (MCP)** — a callable capability a server exposes, with a name,
  description, and input schema. Descriptions are untrusted input — see
  chapter 4.
- **Vetting** — client-side scanning of tool metadata before it enters AI
  context. See chapter 4.
- **Well-known URI** — `/.well-known/agent-card.json`: where A2A agents
  publish their cards for discovery.
- **대리인 (daeriin)** — Korean for *agent*: one who acts on another's
  behalf. The name of this repo, and the idea it teaches.
