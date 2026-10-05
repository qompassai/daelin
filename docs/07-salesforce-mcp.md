# Chapter 7 — Case Study: Salesforce's MCP Servers

*Researched 2026-10-05. This chapter is the payoff of chapter 6's design: a
third-party MCP server plugging into the same client, zero bridge code.*

## The idea in one paragraph

Salesforce ships MCP servers two ways: **hosted** (Salesforce runs the server;
your agent connects over HTTP with OAuth) and **local** (you run the server;
it talks to your org with your existing CLI credentials). The local one —
the **DX MCP Server** — speaks stdio. Which means it plugs directly into the
client from chapter 2 with no adapter, no bridge, no new code. This chapter
documents both flavors and the five-minute investigation that concluded the
bridge didn't need to exist.

## The hosted servers

Salesforce hosts MCP servers at predictable URLs:

```
https://api.salesforce.com/platform/mcp/v1/{servername}                          # simple
https://api.salesforce.com/platform/mcp/v1/d/{mydomain}/develop/{servername}      # My Domain (recommended)
```

Available servers include scoped SObject access (`sobject-reads`,
`sobject-mutations`, `sobject-deletes`, `sobject-all`), `flows`,
`invocable-actions`, `data-cloud-queries`, `prompt-builder`, and more. The
scoping is the security story: there are **read-only, mutation-only,
delete-only, and full-access** variants — you grant the agent the smallest
server that does the job, which is least-privilege (chapter 8) implemented as
server selection.

Auth is per-user OAuth 2.0; every tool call honors the org's object
permissions, field-level security, and sharing rules. The agent sees only what
the authenticated identity is entitled to see — governance inherited from the
platform, not reimplemented in the protocol.

Status as of 2026-10-05: generally available for Enterprise Edition and above;
some servers still carry beta terms, and Salesforce publishes an end-of-life
page for hosted servers — check both before depending on them.

## The DX MCP Server (the local one)

```
npx -y @salesforce/mcp --orgs <username-or-alias> --toolsets orgs,metadata,data,users
```

This is a **stdio MCP server** (binary name `sf-mcp-server`). It authenticates
with your existing Salesforce CLI auth — no separate OAuth dance — and
exposes toolsets: `core`, `data`, `orgs`, `metadata`, `testing`, `users`,
`devops`, `code-analysis`, `enrichment`, and more. `--allow-non-ga-tools`
opts into pre-GA tools; `--dynamic-tools` enables dynamic toolsets.

Because it's stdio, the diver client connects to it as just another registry
entry:

```json
{
  "salesforce-dx": {
    "command": "/path/to/npx",
    "args": ["-y", "@salesforce/mcp", "--orgs", "my-alias",
             "--toolsets", "orgs,metadata,data,users"]
  }
}
```

No bridge. No shim. The argv the client already spawns, the JSON-RPC the
client already speaks. The investigation that established this took one
`--help` invocation — which is the point: **when both sides implement the
standard, integration is configuration.**

## The bridge that wasn't built

The original plan was a stdio→HTTP bridge: a shim process translating the
client's newline-delimited JSON-RPC into Streamable HTTP with OAuth for the
*hosted* servers. The investigation killed it in favor of the DX server:

- The DX server covers the developer use case (orgs, metadata, data, users,
  testing, code analysis) with zero new code.
- The hosted servers are the right choice for *external* agents (Claude,
  ChatGPT) — but for an *editor-native* agent that already holds CLI auth,
  the local server is simpler, faster, and one fewer credential to manage.
- If the hosted path is ever needed, the bridge design is straightforward
  (stdio in, Streamable HTTP + bearer token out) — but "straightforward"
  is not "necessary," and unnecessary code is a liability.

This is the Tiger Style lesson applied to architecture: **the simplest
solution that satisfies the requirement wins, and "use the existing standard
component" beats "build a new component" every time.**

## What this teaches

1. **Standards compound.** Chapter 6 put MCP at the seam so *any* server
   could plug in. This chapter is the payoff: a vendor server plugs in with
   a JSON registry entry. The leverage was designed in chapters ago.
2. **Check for the existing component before building.** The bridge was a
   fine design — and entirely unnecessary. One `--help` saved a project.
3. **Scope servers like you scope permissions.** Salesforce's
   read/mutate/delete/full server variants are least-privilege as a product
   decision. Choose the smallest server that does the job.
