# Chapter 6 — Case Study: Editor ↔ Agent Runtime over MCP

*Systems: rose.nvim (in-editor AI, now part of Diver's `lua/ai/rose/`) and
phlow (the agent runtime). This is where chapters 2 and 3 stop being
components and start being a system.*

## The idea in one paragraph

An editor is where the human works; an agent runtime is where the agent works.
The problem: how do they talk to each other without the editor becoming an
agent framework and without the agent runtime becoming an editor plugin? The
answer this system gives: **MCP over stdio between them, with a private
backchannel for editor operations.** The editor hosts the MCP client; the
runtime serves MCP; tools flow one way, editor actions flow back through a
separate, deliberate channel.

## The shape

```
┌─ Neovim (diver) ──────────────┐      ┌─ phlow ───────────────────┐
│  rose.nvim                    │      │  agent runtime             │
│  ├─ chat / planner→coder→     │ MCP  │  ├─ MCP server (stdio)     │
│  │   validation→reviewer      │◄────►│  ├─ bounded checks         │
│  ├─ ai.rose.mcp (client,      │stdio │  └─ operator approvals     │
│  │   ch. 2)                   │      │                             │
│  └─ private socket ───────────┼──────┘                             │
│     (editor tools backchannel)│                                    │
└──────────────────────────────┘
```

## Why MCP between them

The alternative is a bespoke protocol — and bespoke protocols are where
coupling goes to hide. By using MCP:

- **The runtime is just a server.** Any MCP client can talk to it, not only
  rose. The runtime doesn't know or care that its client lives in Neovim.
- **The editor is just a client.** Any MCP server can plug into rose, not
  only phlow (chapter 7 is exactly this: Salesforce's server plugging into
  the same client).
- **The wire format is the contract.** Changes at the seam — MCP framing,
  socket protocol, config keys, approved checks — must keep both sides'
  formats stable. The protocol version negotiation from chapter 2 is what
  makes this enforceable rather than aspirational.

## The backchannel: why not everything over MCP

MCP is deliberately *not* used for everything. Editor operations — buffer
edits, cursor moves, diagnostics — go through a **private socket** with editor
tools. Why the split?

Because MCP's trust model (chapter 4) says: the client doesn't let the server
make it do things. But an agent runtime *needs* to make the editor do things
— that's its job. Routing editor actions through MCP would mean punching holes
in the client's refusal posture. The separate backchannel keeps the MCP
client's paranoia intact while giving the runtime a *different*, explicitly
gated path for editor effects. Two channels, two trust stories, no confusion
between them.

## The bounded workflow

rose runs a **bounded planner → coder → validation → reviewer** pipeline:

- **Planner** decomposes the task.
- **Coder** acts (via MCP tools served by phlow, or local editor tools).
- **Validation** runs the gates — linters, tests, diagnostics. The same gates
  a human would run, automated.
- **Reviewer** checks the result against the request.

Each stage is bounded: timeouts, iteration caps, explicit handoff contracts.
An unbounded agent is a liability; a bounded one is a tool. The bounds are
part of the design, not a limitation discovered later.

Operator-approved checks gate the dangerous transitions — the human-in-the-loop
checkpoint from chapter 8's best practices, placed where the agent's autonomy
meets the world's mutability.

## What this teaches

1. **Use the standard protocol at system seams.** The moment two systems need
   to talk, reach for MCP before inventing a wire format. Standards are
   leverage: every future client and server comes for free.
2. **Don't force one channel to do two trust jobs.** MCP for tools (client
   paranoid), private socket for editor effects (separately gated). When the
   trust stories differ, the channels should differ.
3. **Bound the agent, then trust the bounds.** Timeouts, iteration caps,
   validation gates, operator approvals — the pipeline is trustworthy because
   its limits are structural, not because the model is well-behaved.
