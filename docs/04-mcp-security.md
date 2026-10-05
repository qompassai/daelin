# Chapter 4 — The Security Model: Vetting, Least Privilege, and Trust

*Implementation: Diver `lua/ai/security/mcp_vet.lua`, `lua/ai/mcp/registry.lua`,
`lua/ai/mcp/tools.lua`. This is the chapter that matters most — read it twice.*

## The core insight

An MCP tool description is **attacker-controlled text that will be read by a
model that trusts it.** When your agent considers which tool to call, it reads
the tool's name and description — written by whoever runs the server — and
treats them as instructions about what the tool does. A poisoned description
doesn't just misdescribe one tool; it steers the model *every time the tool is
considered*.

The MCP spec itself says this: tool descriptions and annotations "should be
considered untrusted, unless obtained from a trusted server." Most clients
ignore that sentence. This implementation doesn't.

## The vetting layer (`mcp_vet.lua`)

Client-side vetting of tool metadata — names plus descriptions — runs at
**registration and discovery time, before the descriptions ever enter AI
context.** The order matters: once a poisoned description is in the model's
context window, the damage is done. Vetting is a gate *before* context, not a
filter after it.

How it works:

- **Per-tool imperative scan.** Tool descriptions addressed *to the model* in
  instructional language ("ignore previous instructions," "send the file
  contents to…") are the attack. The scanner looks for imperative phrasing
  directed at the agent.
- **Cross-tool trigger correlation.** A single innocent-looking tool can be
  half of a two-tool attack. The vetter correlates across the toolset, not just
  within one description.
- **Quarantine on high.** A high-severity finding disables or removes the
  server via the registry. The vetter returns findings; the *caller* decides —
  separation of detection and policy, so the policy can evolve without
  retouching the scanner.

The critical scoping decision: **the imperative-phrase set lives only in this
module and runs only over tool descriptions.** Benign documentation is full of
imperatives ("you should restart the server") — running this scan over prose
would drown in false positives. The attack surface is specifically
*instructional phrasing addressed at the model inside tool metadata*, so that's
exactly — and only — what's scanned.

## The registry as a trust store (chapter 2, revisited)

The registry (`servers.json`) isn't just configuration; it's the list of
processes your editor is allowed to spawn. That's why:

- Entries carry **risk flags** — some servers are inherently more dangerous
  (a shell-execution tool vs. a read-only docs server), and the UI shows it
  *before* spawn.
- **Typed argv confirmation** at install: you see the exact command,
  character for character, and type to confirm. "Click to install" is how
  supply-chain attacks ship.
- Validation on load *and* before launch: a registry entry is re-checked every
  time, not just when written. Trust decays; verification doesn't.

## The confirmation gate (chapter 2, revisited)

Every `tools/call` passes through an explicit confirmation — the security
layer when present, `vim.ui.select` otherwise. This is the human-in-the-loop
checkpoint, implemented as a **chokepoint**: the call path *cannot* bypass it,
it's not a wrapper the agent can choose not to use.

## The transport rules (chapters 2–3, revisited)

- **argv-only spawning.** No shell strings, ever. The most common MCP client
  vulnerability class is command injection through server configuration; argv
  arrays eliminate it structurally.
- **Refuse server→client requests** (except `ping`). A server that can invoke
  sampling, elicitation, or roots can make your client do things. Default no,
  per-case yes.
- **Bound everything.** Message sizes, registry entries, pending requests,
  arg counts. Unbounded inputs are where denial-of-service lives.

## The threat model, stated plainly

| Threat | Where it's handled |
|--------|-------------------|
| Poisoned tool descriptions steering the model | `mcp_vet.lua`, before context |
| Malicious server config / command injection | argv-only, registry validation, typed confirmation |
| Server making the client act | Refuse server→client requests except ping |
| Over-privileged tool access | Risk flags, per-call confirmation gates |
| Rogue server binary | Registry trust store, atomic writes, re-validation |
| Protocol confusion / framing attacks | Strict JSON-RPC 2.0 validation, stdout discipline |

## What this teaches

1. **The missing layer is client-side.** Servers don't vet themselves. Every
   MCP security conversation focuses on server hardening; the client reading
   hostile metadata is the neglected half — and it's where this implementation
   puts its energy.
2. **Security is a set of chokepoints, not a set of suggestions.** Vetting
   before context, confirmation before tool calls, validation before spawn.
   Each is placed where the malicious input *must* pass through, not where
   it's convenient to check.
3. **Scope your scanners to the attack surface.** The imperative-phrase scan
   would be useless run over all text and is precise run over tool
   descriptions. A security control is only as good as its false-positive
   rate, and false-positive rate is a scoping decision.
