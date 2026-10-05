# Chapter 10 — Muse and Context Language Models

*Two things this repo's story would be incomplete without: the personal-agent
product this documentation was written inside of, and the September 2026
research paper that reframes how agents manage memory. One is engineering
you can use today; the other is where the field is heading.*

## Part 1 — Muse: the personal agent

[Muse](https://muse.ai) is Meta's personal AI agent product, launched
September 8, 2026 (US and Canada). The core idea is simple and worth stating
plainly: **every user gets their own agent, running on its own dedicated
computer that stays with them between conversations.** The agent has a
filesystem, a terminal, a browser, tools, and memory — it does work on your
behalf rather than just answering questions.

Concretely, Muse is:

- **Strictly personal.** Every conversation is between one user and their own
  agent. There are no group or shared chats.
- **A computer, not a chatbot.** The agent runs shell commands, reads and
  writes files, browses the web, and delegates to subagents — the same
  primitives this repo's earlier chapters describe for MCP and agent
  orchestration.
- **Multi-surface.** The user reaches the same agent through the web app,
  iOS/Android apps, a Mac app, and messaging channels like WhatsApp.
- **Structured output.** Beyond chat, the product has an Artifacts tab
  (documents, pages, apps the agent builds), a Feed (scheduled editorial
  posts), an Ideas tab (suggestions the user can accept), a Goals tab
  (durable outcomes with progress), and a Library (collected files and media).
- **Connected.** Connectors plug in external services (email, calendar,
  shopping, and more); scheduled jobs and watches run while the user is
  away; payments flow through a wallet with per-purchase approval.
- **Powered by Muse Spark**, from Meta's Muse model family (first launched
  April 8, 2026).

Why does this belong in a protocols repo? Because Muse is a worked example
of the architecture this repo teaches: tool-calling agents (MCP-style),
packaged procedures (skills, chapter 9), scheduled background work, and
persistent memory — composed into one product. When chapter 8's best
practices say "confirm before irreversible actions" or "scope every
background job," Muse's approval cards and per-job scoping are what that
looks like in production.

## Part 2 — Context Language Models (arXiv:2609.37725)

On September 29, 2026, researchers at the University of Washington and Meta
Superintelligence Labs (with co-authors at MIT and Trillium Labs) published
[*Context Language Models*](https://arxiv.org/abs/2609.37725) — Rulin Shao,
Pang Wei Koh, Luke Zettlemoyer, Mike Lewis, and colleagues. It tackles the
problem every long-running agent hits: **the context window fills up, and the
harness has to decide what survives.**

### The core idea

Stop treating context as an append-only transcript. **Treat the context as a
file the model itself can edit.** The model gets ordinary file-editing
capabilities over its own context — delete, compress, reorganize, rewrite —
and edits are synchronized back so the next model call runs on the edited
version. Context management moves from *external harness policy* to
*intrinsic model behavior*: the model learns what is worth keeping.

### What the paper reports

Built zero-shot on top of existing models (no retraining required to start),
CLMs beat the context-management strategies the authors tested against:

| Benchmark | Result vs. strongest baseline |
|-----------|-------------------------------|
| BrowseComp-Plus (deep research) | **+11.4% accuracy, 21.5% fewer FLOPs** |
| EdgeBench (12-hour) | **+5% score, 59% fewer FLOPs** |
| 24-hour multi-repo agent-swarm | **65% greater improvement, same compute** |

And because the strategy is now model behavior rather than harness rules, it
can keep improving: natural-language instructions evolved through a
**skill-optimization loop** improved held-out accuracy by up to 35.9 points,
and an online reinforcement-learning method improved Qwen3.5-9B by 47.6%
while using 12% fewer FLOPs. A co-designed serving optimization (Suffix
Cache Reuse) cut server-side compute 35% relative to standard SGLang at
matched performance.

### Why it matters for this repo's readers

Three connections back to what you've already read:

1. **Memory, done by the model.** Chapters 2–4 show the harness side of
   context management (what the client keeps, what the server returns).
   CLMs flip the responsibility: the agent curates its own working memory.
   Note the scope, though — the context file is thrown away when the task
   ends. Anything that must survive to next week is still your job
   (durable memory, `.ai/memory/` handoffs, the patterns in chapter 8).
2. **Skills steering the strategy.** The paper improves CLM behavior through
   a skill-optimization loop — the same SKILL.md packages from chapter 9,
   used to evolve the model's context-management instructions. Skills and
   CLMs compose: skills carry the procedure, the model applies it to its
   own memory.
3. **Multi-agent, naturally.** Multiple agent contexts coexist as files, so
   the same mechanism extends to agent swarms — the A2A world of chapter 5,
   where each peer manages its own state.

### The honest caveat

The paper's own safety section warns that the same freedom lets a model
**plant instructions for itself** — edits to the context file are
self-modifying, which means prompt-injection and persistence risks move
*inside* the context boundary. Chapter 4's vetting mindset applies here
too: if the model can rewrite its own memory, the provenance of what lands
in that file matters. This is active research, not settled practice —
exactly the kind of sharp edge this repo documents rather than smooths over.

## The thread tying it together

Muse shows what a personal agent looks like when the protocols, skills,
scheduling, and memory patterns in this repo are composed into a product.
Context Language Models show where the memory layer of that composition is
heading: from harness-managed transcripts to model-managed working memory,
steered by skills, with the safety questions still open. Read the paper
([arXiv:2609.37725](https://arxiv.org/abs/2609.37725),
[PDF](https://arxiv.org/pdf/2609.37725)) — it's the rare paper that's both
immediately applicable and genuinely new.
