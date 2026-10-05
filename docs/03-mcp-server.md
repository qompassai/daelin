# Chapter 3 — How an MCP Server Works

*Implementation: Diver `lua/ai/mcp/server/` (init, stdio, protocol, policy,
tools). The mirror image of chapter 2.*

## The idea in one paragraph

If the client is the agent's hands, the server is the world's API in a shape
agents can use. An MCP server exposes **tools** (things the agent can do),
**resources** (things the agent can read), and **prompts** (reusable instruction
templates) over the same JSON-RPC wire the client speaks. This chapter covers a
server that runs *inside the editor's ecosystem*: a headless Neovim process
serving MCP over stdio.

## The shape

```
:Mc pServer start
      │
      ▼
nvim --headless -l lua/ai/mcp/server/stdio.lua --serve
      │  stdin: JSON-RPC lines in
      │  stdout: JSON-RPC lines out (MCP bytes ONLY)
      │  stderr: logs, warnings, diagnostics
      ▼
protocol engine → policy gate → tools
```

`:McpServer start` spawns the headless child with pipes held open; `stop`
terminates it; `status` reports on it. `setup()` is idempotent and registering
the command is its only side effect.

## The transport (`stdio.lua`)

Newline-delimited JSON-RPC over the process's stdio — the exact mirror of the
client's transport, with the same iron rule enforced from the other side:
**stdout carries ONLY MCP bytes.** Logs, warnings, and diagnostics go to
stderr. An oversized line is dropped (and logged to stderr) rather than parsed.
The server exits when the client closes stdin — no orphaned processes.

Two details worth studying:

- **The transport is injectable for tests.** `start(server, deps)` takes the
  stdin/stdout/stderr/exit functions as a table, so tests drive the entire
  stack with fakes. `real_deps()` wires the real `vim.uv` pipes. The transport
  was designed test-first, not test-after — and the test suite is split evenly
  between validation (does the happy path work) and adversarial cases (what
  happens on garbage input, oversized lines, mid-stream EOF).
- **Framing bounds are mirrored.** The 8 MiB line cap matches the client's, so
  both sides agree on the maximum message size. A protocol is a contract; both
  sides must read the same contract.

## The policy gate (`policy.lua`)

Before any tool executes, the policy gate decides whether it's *allowed* to.
This is where the server's own security posture lives: which tools exist, what
arguments they accept, and what preconditions must hold. The gate runs before
dispatch, not after — a tool that shouldn't run never starts running.

## The tools (`tools.lua`)

The actual capabilities the server exposes. Each tool declares a name, a
description, and an input schema (JSON Schema) — the same metadata shape the
client's `tools/list` returns. Descriptions are written for the *model*, not
for humans: they need to say what the tool does, what it needs, and what it
returns, precisely enough that an agent can use the tool correctly without
having seen its implementation.

This is also where chapter 4's warning bites from the other direction: these
descriptions will be read by a model that *trusts* them. Write them as if an
adversary will try to make them say something else — because on a hostile
server, that's exactly what happens.

## Client and server, together

The elegant property of this design: **the client and server are the same
protocol with opposite postures.** The client is paranoid about what the server
can make it do (refuse server requests, confirm tool calls). The server is
paranoid about what the client can make it do (policy gate, bounded tools).
Neither trusts the other; both trust the wire format. That's the whole
security model of MCP in one sentence: *trust the protocol, verify the peer.*

## What this teaches

1. **A server is a client with the trust arrows reversed.** Same JSON-RPC,
   same framing, opposite paranoia. If you understand chapter 2, you
   understand 80% of this chapter — the remaining 20% is the policy gate.
2. **Stdout discipline is a protocol feature.** "Only MCP bytes on stdout"
   sounds like a style rule; it's actually what makes the stream parseable,
   debuggable, and testable. Conventions at the byte level become guarantees
   at the system level.
3. **Testability is designed, not added.** The injectable transport means the
   entire server — framing, protocol, policy, tools — runs in tests without a
   subprocess. If your transport can't be faked, your tests can't be thorough.
