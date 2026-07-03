# orchestrator-skill

A [Cursor Agent Skill](https://cursor.com/docs/agent/skills) that turns the agent into an orchestrator. Instead of doing every step in the main thread, the agent decomposes multi-part work, delegates tasks to parallel subagents with cost-aware model routing, and keeps the main conversation focused on planning and integration.

## What it does

When this skill is active, the agent acts as a coordinator rather than a solo implementer. The main thread stays at a high level: breaking down the request, dispatching work, checking that pieces fit together, and reporting back. Low-level implementation detail lives in subagent contexts, not in your chat.

The workflow follows six steps:

1. **Decompose** — Split the request into discrete tasks with clear boundaries and deliverables.
2. **Classify** — Label each task as complex or simple for model routing.
3. **Identify dependencies** — Independent tasks run in parallel; dependent tasks wait for their inputs.
4. **Dispatch** — Send tasks to subagents, launching all independent work in a single parallel batch.
5. **Integrate** — As results return, verify consistency, resolve conflicts, and dispatch follow-ups if needed.
6. **Report** — Summarize the integrated outcome in plain language.

## Model routing

Tasks are routed to different subagent models based on complexity:

| Task type | Examples | Default model |
| --------- | -------- | ------------- |
| Complex | Multi-file changes, design judgment, debugging, architecture, deep reasoning | `gpt-5.5` |
| Simple | Mechanical edits, lookups, boilerplate, single-file changes, running commands | `composer-2.5-fast` |

When in doubt, route to the stronger model — a misrouted complex task costs more than a misrouted simple one. The model names in `orchestrator/SKILL.md` can be edited to whatever models you prefer or have available in your Cursor setup.

## Do-it-yourself threshold

The skill defines when *not* to delegate:

- **Floor** — If writing the delegation prompt would take more effort than doing the task yourself, just do it. Trivial edits, one-liners, quick file reads, and single shell commands fall below the delegation overhead.
- **Ceiling** — If a task is too complex to trust a subagent with (novel reasoning, subtle cross-cutting design, high cost of a wrong answer), keep it in the main thread. Delegate the surrounding pieces and reserve the hard core for direct reasoning.

## Delegation prompts

Subagents have no access to the conversation or prior steps. Every task prompt must be self-contained: the goal, relevant file paths, constraints, acceptance criteria, and exactly what to return so results can be integrated without re-reading everything.

## Parallelism

Independent subagents launch in one message with multiple parallel Task tool calls. Tasks serialize only when one genuinely depends on another's output. When parallel subagents edit the same codebase, file ownership is partitioned in their prompts to avoid conflicts.

## Install

Copy the `orchestrator/` folder into your Cursor skills directory:

```bash
cp -r orchestrator ~/.cursor/skills/
```

The skill file should end up at `~/.cursor/skills/orchestrator/SKILL.md`.

Invoke it in Cursor chat with `/orchestrator`, or let the agent pick it up automatically when you give it a large or multi-part task.
