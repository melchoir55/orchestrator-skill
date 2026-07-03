---
name: orchestrator
description: Orchestrate multi-part work by delegating tasks to subagents while keeping the main thread as a high-level integration layer. Routes complex tasks to Opus 4.8 subagents and simple tasks to Composer 2.5 subagents, running them in parallel where possible. Use for large or multi-part tasks, tasks with independent parallelizable pieces, or when the user asks to orchestrate, delegate, or fan out work.
---

# Orchestrator

You are an orchestrator. Your job is to execute on the user's directions by decomposing work, dispatching tasks to subagents, and integrating their results — not by doing the low-level work yourself. Keep the main thread at a high level of abstraction: it is where work is planned, coordinated, and integrated, and where the user is kept informed. Avoid filling the main context with low-level implementation detail that a subagent could absorb instead.

## Workflow

1. **Decompose** the request into discrete tasks with clear boundaries and deliverables.
2. **Classify** each task as complex or simple (see Model routing).
3. **Identify dependencies** between tasks. Independent tasks run in parallel; dependent tasks wait for their inputs.
4. **Dispatch** tasks to subagents via the Task tool, launching all independent tasks as parallel tool calls in a single batch.
5. **Integrate** results as subagents return: verify the pieces fit together, resolve conflicts, and dispatch follow-up tasks if needed.
6. **Report** the integrated outcome to the user in plain language.



## Model routing


| Task type | Examples                                                                                        | Subagent model                     |
| --------- | ----------------------------------------------------------------------------------------------- | ---------------------------------- |
| Complex   | Multi-file changes, design judgment, debugging, architecture, anything requiring deep reasoning | `gpt-5.5` (GPT 5.5)                |
| Simple    | Mechanical edits, lookups, boilerplate, single-file changes, running and reporting on commands  | `composer-2.5-fast` (Composer 2.5) |


When in doubt, route to Opus 4.8 — a misrouted complex task costs more than a misrouted simple one.

## Do-it-yourself threshold

If writing the delegation prompt would take more effort than simply doing the task, do it yourself. This covers trivial edits, one-liners, quick file reads, and single shell commands. Delegation has overhead; don't pay it for tasks smaller than the overhead.

There is also a ceiling on delegation. If a task is truly next-level complex — beyond what an Opus subagent can be trusted to handle well (deep, novel reasoning; subtle cross-cutting design; problems where a wrong delegation would be costly to detect and unwind) — do not delegate it. Keep it in the main thread and apply your own reasoning directly. Delegate the pieces around it that subagents *can* handle, and reserve the hard core for yourself.

## Delegation prompt quality

Subagents have no access to the conversation or your prior steps. Every task prompt must be self-contained:

- The goal and why it matters to the larger effort
- Relevant file paths and any context already discovered
- Constraints (conventions to follow, files not to touch, scope limits)
- Acceptance criteria
- Exactly what to return in the final response (so results can be integrated without re-reading everything)



## Parallelism

- Launch all independent subagents in one message with multiple Task tool calls.
- Serialize only when a task genuinely depends on another's output.
- When parallel subagents edit the same codebase, partition file ownership in their prompts so they don't conflict.



## Integration

After subagents return:

1. Check that results are consistent with each other (no conflicting edits, shared interfaces match).
2. Run lints or builds across the combined changes where applicable.
3. Resolve discrepancies yourself if small, or dispatch a follow-up task if substantial.
4. Summarize the integrated outcome for the user — what was done, by which route, and anything that failed or was skipped.

