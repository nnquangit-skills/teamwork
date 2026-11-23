---
name: teamwork
description: Coordinate agents and subagents through a project's runner. Use for assigning work, selecting models, monitoring runs, resolving blockers, and verifying deliverables before marking them complete.
---

# Teamwork

## Plan and assign work

1. Establish whether the project owner wants a plan, a simulation, or execution. Start agents once the owner has requested execution. Read the repository instructions, approved requirements, Feature PRD and ARD, current tracker, and Git status. If sources conflict, identify the approved decision and the affected IDs.
2. Keep existing IDs. For a four-level plan, group work by Epic → Feature → Task → Subtask. Schedule items whose dependencies have cleared, taking P0 before P1 and P2. Give each Task one owner who integrates its results. That owner may do all Subtasks or delegate those with a clear output and an independently verifiable boundary.
3. Give each run its ID, parent ID, assignee, dependencies, writable files, deliverable, acceptance criteria, verification commands, and handoff location. Assign ownership of shared interfaces before starting parallel implementation. Give concurrent writers separate Git worktrees through the project's runner. Confirm that the previous writer has stopped before reassigning the same work.

## Start a run and select a model

- Inspect the project's ACP runner, adapter, preset, and available models. If the runner uses `npx`, take the package name, version, and invocation from the project's configuration or runner documentation. Verify that the adapter responds before dispatching a batch. Record the session ID and actual model.
- Use a lightweight model for inventory or bounded documentation edits. Match cross-component design, regression diagnosis, and high-risk review to a model with the relevant capabilities.
- When credits run out, preserve the checkpoint, diff, verification results, and remaining work. Confirm that the old run has stopped writing. Resume the **same work item ID** in a new run on the configured fallback provider, such as `litellm-237`. Diagnose rate limits, authentication failures, and test failures separately. Load credentials from the project's private environment.

## Monitor progress

Use the project's monitoring cadence if it defines one. Otherwise, use these intervals while the coordination session is active.

| When | What the coordinator does |
|---|---|
| On dispatch | Record the start time, deadline, next check, and time zone. |
| Every minute while a run is active | Read runner events or poll the session. Look for observable progress such as command output, a diff, or a completed check. |
| After 5 minutes without progress | Inspect the current command and ask the worker what step it has reached, what is blocking it, and what help it needs. |
| After 10 minutes without progress, or at the deadline | Inspect the process and logs. If a long command is progressing, record the evidence and set the next check. If the run is stuck, preserve its work before reassigning it. |
| On submission, a blocker, or a model change | Notify the Task owner and coordinator immediately. Prioritize review of newly submitted work. Tell the project owner when work is submitted, accepted, or affects delivery. |
| After 10 minutes without a new acceptance | Send the project owner one update with active IDs, observed progress, blockers, and the next check time. |

When resuming an interrupted session, reconcile the tracker, live sessions, Git worktrees, and latest run reports before dispatching again.

## Resolve a faulty plan

A subagent who discovers a false assumption pauses the affected work and sends the Task owner a reproducible example. The owner consults the agent responsible for the interface or the reviewer. The coordinator can reorder, split, or reassign work within approved scope. Ask the project owner for a decision when the fix changes a product commitment, requires additional budget or access, or depends on unresolved requirements. Continue independent IDs while waiting.

## Review and accept work

The worker runs the assigned checks and submits the result. An independent reviewer inspects the diff and repeats the relevant checks. If a check fails, return the same ID with reproduction steps. Once the evidence meets the criteria, the coordinator updates the tracker and reports the result. Accept a Task after checking the integrated outcome, a Feature against its PRD and ARD, and an Epic against its delivery goal.

For projects using the `worktree` skill, follow its handoff and reporting conventions. Otherwise, use the project's tracker and report format.
