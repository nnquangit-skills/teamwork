---
name: teamwork
description: Coordinate agents and subagents through a project's runner. Use for assigning work, discovering and classifying available models, selecting a model for each assignment, monitoring runs, resolving blockers, and verifying deliverables before marking them complete.
---

# Teamwork

## Plan and assign work

1. Establish whether the project owner wants a plan, a simulation, or execution. Start agents once the owner has requested execution. Read the repository instructions, approved requirements, Feature PRD and ARD, current tracker, and Git status. If sources conflict, identify the approved decision and the affected IDs.
2. Keep existing IDs. For a four-level plan, group work by Epic → Feature → Task → Subtask. Schedule items whose dependencies have cleared, taking P0 before P1 and P2. Give each Task one owner who integrates its results. That owner keeps dependent work in sequence and coordinates independent Subtasks using the parallel dispatch rule below.
3. Give each run its ID, parent ID, assignee, dependencies, writable files, deliverable, acceptance criteria, verification commands, and handoff location. Confirm that the previous writer has stopped before reassigning the same work.

## Dispatch independent work to two ACP agents

1. **Choose a ready pair.** When two Tasks or Subtasks can proceed independently, dispatch them concurrently to two distinct ACP agents. If one Task naturally splits into two independently testable deliverables, give each a Subtask ID and keep one owner for integration. Check that neither item needs the other's unfinished output and that the two write scopes do not overlap. Settle shared contracts first. Give shared files such as lockfiles, migrations, and generated manifests a single writer. If that leaves only one ready item, run it immediately and recheck the queue when dependencies clear.
2. **Start both runs.** Select a suitable model for each assignment. Use the project's configured ACP runner to start two separate sessions from the same recorded base revision, each on its own branch and Git worktree. Give each worker its ID, allowed paths, agreed interface, and checks. Launch the second session while the first is running. Record both session IDs, branches, and worktree paths so either run can be monitored or resumed.
3. **Handle newly discovered overlap.** If a worker needs a file owned by its peer or needs to change their shared contract, pause that part and notify the coordinator before editing. The coordinator assigns one writer, updates both briefs, and keeps unaffected work moving.
4. **Integrate and verify.** Review each diff against its assigned paths and criteria. The designated integrator applies one reviewed change at a time, resolves any conflicts with the relevant owners, and runs the checks that exercise both changes together. Report each item as it passes review and close the parent only after the integrated result passes.

## Discover and classify models before dispatch

At the start of each execution batch, the coordinator must scan the available models across the agents being considered, including configured fallback providers. Reuse the inventory across assignments in that batch. Refresh the affected entries when access, configuration, quota, or the work requirements change.

1. **Discover candidates.** Inspect each agent's ACP adapter and presets. Use the runner's documented model-list command or the configured provider's model catalog under the credentials available to that agent. If the runner uses `npx`, take the package, version, and invocation from the project's configuration. For adapters without enumeration, inspect their configured presets and record the limits of the scan.
2. **Search for missing information.** When a capability is unclear or the initial list has no suitable candidate, search the provider's current official catalog, model cards, and adapter documentation for alternatives. Cross-check candidates against the models selectable through that agent. Record catalog-only candidates as unverified. Keep credentials and private endpoints out of search queries and reports.
3. **Classify each candidate by work it can perform.** Record agent, provider, model ID or preset, access status, suitable task types, and the evidence behind the classification. Use the criteria below. A model may fit several classes. Record unknowns explicitly, including context capacity, tool support, or pricing when they affect the decision.

| Work class | What to look for |
|---|---|
| Inventory and bounded edits | Accurate extraction or editing on representative inputs, sufficient context, low cost or latency for repeated work. |
| Implementation and testing | Reliable code edits, tool support through the actual adapter, and the ability to run and interpret the Task's checks. |
| Architecture and difficult debugging | Reasoning across components, enough context for the affected interfaces, and evidence from comparable design or diagnosis work. |
| Independent review | Ability to trace requirements, inspect the diff, reproduce failures, and verify acceptance criteria in the review environment. |

## Select a model for each assignment

- For each dispatched Task, separately delegated Subtask, and review, match the acceptance criteria to the inventory. Filter by required inputs, tools, context, and project constraints. Among candidates that meet those requirements, use relevant prior results, cost, and latency to choose. Record the work ID, chosen agent/provider/model, reason, and suitable fallback if available.
- Confirm that the selected route responds through the intended adapter. An existing successful session or the first bounded step of the assignment can serve as the check. Record the session ID and actual model reported by the runner. If the runner hides the underlying model, record the observed preset and label the model unknown. If no accessible candidate meets a required capability, report that gap and continue independent work.

## Hand off an active run when quota is exhausted

1. **Checkpoint and release the work.** Confirm the runner reports exhausted quota or credits. Handle short rate limits, authentication errors, and failed tests according to their cause. Stop the old writer and confirm its session and child processes have released the workspace. If the agent cannot reply, recover its revision, uncommitted changes (including new files), completed checks, outstanding work, and next verification command from the worktree and runner logs.
2. **Find an available replacement agent.** Refresh the inventory across configured agents and fallback routes, including `litellm-237` where configured. Exclude routes sharing the exhausted quota. Check each candidate against the tools, context, access, and budget needed for the remaining work. Preserve reviewer independence when reassigning a review.
3. **Look up comparable public results.** Choose an evaluation that measures the remaining work. For repository issue resolution, compare the percentage resolved on the same SWE-bench split and scaffold. For bounded code edits, compare Aider Polyglot pass rates with comparable edit settings. For other work, find an evaluation of that task type. Record the source URL, lookup date, metric, benchmark version or split, model revisions, agent harness, and scores for the outgoing and candidate agents. A model-only result is a proxy when the deployed harness differs. [SWE-bench](https://www.swebench.com/) · [Aider Polyglot](https://aider.chat/docs/leaderboards/)
4. **Try equivalent agents first, then step down.** Prefer another agent running the same verified model revision with comparable tools and settings on available quota, or a candidate in the same performance band on the relevant benchmark. Use the benchmark's reported uncertainty or the project's agreed tolerance to interpret small score differences. If no equivalent is available, try the closest lower-scoring eligible candidate first, then work downward. When comparable scores are unavailable, mark the ranking unknown and use a bounded, representative check to establish task fit. For a lower-ranked or unranked replacement, verify its first checkpoint against the original acceptance criteria before handing over the rest. If no candidate qualifies, record the blocker and continue independent IDs.
5. **Resume and report.** Give the replacement the checkpoint and a new run ID under the **same work item ID**. Have it confirm the revision, outstanding changes, and next step before editing. Update the assignee and tell the Task owner and project owner which agent took over, why it was selected, where the benchmark evidence is recorded, and which check comes next. Keep both runs' evidence linked to the item.

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
