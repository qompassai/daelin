# Chapter 9 — The Agent Skills Specification

*The open standard behind every `SKILL.md` in this repo's [`skills/`](../skills/)
directory. Spec: [agentskills.io/specification](https://agentskills.io/specification)
· [github.com/agentskills/agentskills](https://github.com/agentskills/agentskills).
Originated at Anthropic, published as an open, implementation-agnostic standard.*

## What a skill is

A skill is a **directory** containing, at minimum, a `SKILL.md` file — YAML
frontmatter (metadata) plus Markdown instructions an agent follows to perform
a task. Skills can also bundle scripts, reference docs, templates, and other
resources:

```
skill-name/
├── SKILL.md      # Required: metadata + instructions
├── scripts/      # Optional: executable code
├── references/   # Optional: documentation loaded on demand
├── assets/       # Optional: templates, resources
└── ...           # Any additional files or directories
```

Skills package **procedural knowledge** — how to do something, step by step.
That distinguishes them from their neighbors: MCP gives an agent *tool access*
(what it can call), RAG gives *factual knowledge* (what is true), and a skill
gives *judgment* (when to use the tools and what to do with them). Skills often
use MCP tools for execution.

## The SKILL.md format

YAML frontmatter, then a Markdown body. No rigid body structure is required;
the recommended shape is Overview → When to Use → Steps/Process → Examples →
Edge cases.

**Required frontmatter:**

| Field | Rule |
|-------|------|
| `name` | 1–64 chars; lowercase `a-z`, `0-9`, hyphens only; no leading, trailing, or consecutive hyphens; must match the directory name |
| `description` | 1–1024 chars; must describe both *what* the skill does and *when* to use it — this is the primary trigger mechanism |

**Optional frontmatter:** `license`, `compatibility` (max 500 chars),
`metadata` (arbitrary key-value map), `allowed-tools` (experimental,
space-separated).

## Progressive disclosure: the core idea

Agents load skills in three tiers, so installing many skills costs almost
nothing until one is actually used:

| Tier | What's loaded | When | Cost |
|------|---------------|------|------|
| 1. Discovery | `name` + `description` | Session start, for every skill | ~50–100 tokens/skill |
| 2. Activation | Full `SKILL.md` body | When the description matches the task | < 5,000 tokens recommended |
| 3. Execution | `scripts/`, `references/`, `assets/` | Only when the instructions reference them | Varies |

Practical consequences: keep `SKILL.md` under ~500 lines; anything longer
belongs in a `references/` file that the body links to with guidance on when
to read it. Bundled files consume context only when loaded.

## Discovery paths

The spec standardizes the *package shape*, not a single universal discovery
path. In practice:

- `.agents/skills/` — the shared project path read by OpenCode, Codex,
  Cursor, Copilot, Gemini, and others
- `.claude/skills/` — required by Claude Code (which also reads the shared
  path via some clients)

## Skills vs. MCP, in one paragraph

MCP is the *protocol* an agent uses to call tools; a skill is the *packaged
judgment* for when and how to use them. A skill's instructions may tell the
agent to call MCP tools — the skill provides the procedure, MCP provides the
execution. They compose: skills for the "how," MCP for the "do."

## The skills in this repo

The [`skills/`](../skills/) directory contains the working skill library this
repo's agent workflows actually use — thirteen Tiger Style language skills
(C, C++, Go, Kotlin, Lua, Mojo, Nix, Odin, Python, Rust, Scala, TypeScript,
Zig), plus `mcp-builder`, `skill-creator`, and others. Each follows the spec
above; the Tiger Style skills pair a concise `SKILL.md` (operational core)
with a `references/` guide (full depth) — progressive disclosure applied to
documentation itself.

The [`agents/`](../agents/) directory holds the agent runtime configurations
(Claude Code, Codex, OpenCode, OpenShell) that consume these skills.
