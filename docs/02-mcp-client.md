# Chapter 2 — How an MCP Client Works

*Implementation: Diver `lua/ai/rose/mcp.lua` and `lua/ai/mcp/` (client, registry,
tools, discovery, ui, commands). This chapter explains the design; the code is
the reference.*

## The idea in one paragraph

An MCP client is the agent's hands. The agent (the "host") decides *what* to do;
the client handles *how*: spawning server processes, framing messages,
matching responses to requests, and presenting tools to the agent. The protocol
itself is JSON-RPC 2.0 — the client's job is everything around it: transport,
lifecycle, and trust boundaries.

## Layer 1: the transport (`ai.rose.mcp`)

The foundation is deliberately boring: **one JSON-RPC object per line** over a
child process's stdin/stdout.

```lua
-- the entire transport contract, simplified:
M.start({ cmd = { "/usr/bin/python3", "server.py" } }, function(err, client)
    client:request("tools/list", {}, function(err, result) ... end)
    client:call_tool("read_file", { path = "x.lua" }, function(err, result) ... end)
end)
```

Key design decisions, each earning its place:

- **argv-only spawning, never shell strings.** The command is an explicit array
  (`{"/usr/bin/python3", "server.py"}`), validated element by element (no NUL
  bytes, no empty commands). A shell string is an injection surface; an argv
  array isn't. This is the single most important line of defense and it costs
  nothing.
- **Newline-delimited framing, not Content-Length.** Unlike LSP, MCP-over-stdio
  has no length prefix — each line is one message. Simpler to implement,
  simpler to debug (you can `tail` the wire), and adequate because the 8 MiB
  per-message cap bounds the worst case.
- **The iron rule of stdout: MCP bytes only.** Anything the server prints that
  isn't a JSON-RPC object is a protocol error. Logs go to stderr. This is what
  makes the stream parseable without a state machine.
- **Server-to-client requests are refused except `ping`.** The spec allows
  servers to request sampling, elicitation, and roots from the client. This
  client answers `ping` and returns "unsupported" (-32601) for everything else.
  A server that can make your editor do things is a server that owns your
  editor; the default is no.
- **Timeouts and cancellation are first-class.** Every request gets a timer
  (30s default, 10s for initialize). Cancel sends `notifications/cancelled`
  before completing the pending call. A hung server degrades to an error, never
  to a hung editor.

The handshake: `initialize` with a protocol version (the client accepts
`2024-11-05` through `2025-11-25`), then `notifications/initialized`. Note the
timeline from chapter 1: the 2026-07-28 spec *removed* this handshake for the
stateless core — this client targets the stateful era, which is what the vast
majority of deployed servers still speak.

## Layer 2: session management (`ai.mcp.client`)

One layer up, the harness-level client owns **one session per server name**:
process lifetime, the pending-request table (bounded at 64), and teardown.
Teardown is idempotent and total — `VimLeavePre` closes every session so no
server outlives the editor.

A nice architectural touch: this layer *probes* for `ai.rose.mcp` and reuses
its transport when available, falling back to its own native implementation.
The probe is best-effort and never raises — if the rose module is absent or
its shape changed, the native path just works. Two implementations of the same
contract, one seam.

## Layer 3: the registry (`ai.mcp.registry`)

Clients need to know *which* servers exist. The registry is a JSON file
(`servers.json` under Neovim's data dir) mapping names to `{command, args, env,
cwd, enabled}` — with paranoia appropriate to something that spawns processes:

- **Atomic writes** (tmp file + rename) so a crash mid-write can't corrupt it.
- **Bounded everything**: 256 KiB file cap, 128 entries max, 64-char names,
  32 args max. A registry is a trust store; unbounded trust stores are how you
  get surprises.
- **Validation on load and before launch.** A corrupt file is backed up and the
  registry starts empty — never a crash, never a half-loaded entry.
- **Risk flags** on entries, so the UI can show which servers are more
  dangerous than others *before* they're spawned.

## Layer 4: tools (`ai.mcp.tools`)

`tools/list`, schema inspection, and `tools/call` — but the call path always
passes through an **explicit confirmation**: the security layer when present,
`vim.ui.select` otherwise. The agent can *propose* a tool call; a human (or a
policy) *approves* it. This is the human-in-the-loop checkpoint from the best
practices (chapter 8), implemented as a chokepoint rather than a suggestion.

## Layer 5: discovery (`ai.mcp.discovery`)

Finding new servers: SearXNG first (reusing the existing search config), Brave
Search API only when SearXNG is unconfigured. The important part isn't the
search engine — it's the install gate: **a registry entry is written only after
typed confirmation of the full argv.** You see the exact command that will be
spawned, character for character, and you type to confirm. No "click to
install" that hides what's being installed.

## The UI (`ai.mcp.ui`, `commands.lua`)

A floating-window browser over servers and tools, in the style of the A2A UI
(chapter 5) — one visual language for both protocols. User commands (`:Mcp*`)
are registered by requiring `commands.lua`; requiring any module has no side
effects until `setup()` runs, and `setup()` is idempotent.

## What this teaches

1. **Transports are the easy part; trust boundaries are the work.** The
   JSON-RPC framing is ~100 lines. The argv validation, confirmation gates,
   and vetting (chapter 4) are where the real engineering lives.
2. **Refuse by default.** The client supports the smallest subset of the spec
   that gets work done (ping-only server requests, explicit argv). Every
   capability you *don't* implement is an attack surface you don't have.
3. **Layer the client.** Transport → session → registry → tools → discovery.
   Each layer has one job and one trust story. When something goes wrong, you
   know exactly which layer to blame.
