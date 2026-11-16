---
name: orca-parallel
description: >-
  Use when the user explicitly asks to run or delegate the current work in parallel
  (for example, “병렬 작업으로 진행해”, “동시에 나눠서 실행해”, or “run these in parallel”).
  Give independent tasks to Orca workers, choose a Gemini 4 Hydrogen tier for each,
  and respect the requested maximum concurrent sessions (default 10 per Orca Run).
---

# Orca Parallel

## Decide whether to use Orca

- Act on the user's request to execute the current work in parallel. Treat parallelism mentioned as a topic, quoted text, a prohibition, or a planning-only request as ordinary input. If the session already has an Orca Task and Dispatch assignment, follow that assignment's worker instructions.
- Split work that can proceed independently into tasks with a target, scope, and evidence of completion. Keep dependent work ordered and coordinate overlapping writes through Orca's worktree guidance. If there are fewer than two independent tasks, use the normal sequential workflow.
- Use a positive whole number explicitly given as the concurrent-session maximum, or **10** if omitted. A limit of one executes the delegated tasks one at a time. Clarify a supplied limit that is invalid or ambiguous before launching.

## Resolve Orca and choose models

1. Use the installed `orca-cli` skill. If the host does not list it, resolve the Orca executable using `ORCA_CLI_COMMAND` when set, `orca-dev` for an `ORCA_DEV_REPO_ROOT` dev session, `orca-ide` on Linux outside Orca-managed terminals, and `orca` otherwise. Load `skills get orca-cli --json` and `skills get orchestration --json` with that executable. Follow their version-matched instructions. Use a local Orca coordinator session that can receive Orca wake-ups. If runtime access or the coordinator binding is unavailable, report the blocker before dispatch.
2. When the user enables this skill as their parallel-work policy, use Antigravity workers and choose the lowest Hydrogen tier that fits each task. Follow an explicit worker or model override from the user. If a different worker type is requested without a model, use that worker's ordinary model selection. Keep the same session limit.

| Task | Model ID |
| --- | --- |
| Known call sites, a narrow inspection, or a check against clear criteria | `gemini-4-hydrogen-low` |
| A bounded change in one module with an established verification path | `gemini-4-hydrogen-medium` |
| An uncertain regression across several components or a complex verification path | `gemini-4-hydrogen-high` |
| A difficult diagnosis with competing causes, such as a race that needs substantial verification | `gemini-4-hydrogen-xhigh` |

3. For Antigravity workers, check the selected IDs in `agy models` on the worker host. Dispatch each task only when its selected model is available. Report the affected tasks and the observed error if an ID or the catalog is unavailable. Pass the full model ID as `--model` to Orca's `orchestration worker-start` with `--agent antigravity`. Use the launch receipt and available worker evidence to distinguish requested, applied, and unverified model selection.

## Dispatch within the limit

1. Use the existing `orchestration` skill and its coordinator-loop reference to create one Run for the request and register the tasks, including those that must wait. Keep the Run ID and session limit in the coordinator context across wakes. Keep each task's model choice with its task spec. Use the Run explicitly when listing tasks or starting workers.
2. Before starting a worker, confirm capacity for this Run using `orchestration worker-list --run <run_id>` and any start requests whose outcome is still uncertain. Read all pages when Orca returns a cursor. Count each live session, including retained sessions, workers waiting for input, and sessions pending or awaiting proof of release. Count an uncertain launch until Orca confirms it did not create a session or the session has been released. Apply the cap to these sessions, not to the number of registered tasks. Have this coordinator perform all launches for the Run. Tell each worker in its task spec to finish the assignment without starting nested workers.
3. Start ready tasks up to the free capacity. Submit starts one at a time and account for each receipt before issuing another start. This lets the workers run concurrently while each new start counts against the same limit. When you reach the limit or there are no ready tasks, follow Orca's instructions to end the turn and receive a wake-up.
4. On a wake-up, handle completion, questions, and recovery through `orchestration`. Follow the installed guide's `next` action when present. When work is queued and at the cap, use `orchestration worker-release --dispatch <dispatch_id>` for a settled, reclaimable session so its slot can reopen on this turn. Respect sessions the user keeps open. A `worker_done` message settles a task. Start more ready tasks only after Orca confirms a slot has freed. When accounting or a launch is uncertain, follow the receipt's recovery action and keep the reserved slot. Report which tasks remain queued if Orca cannot confirm capacity.
5. Use Orca's existing result collection. Report what each task produced, the available verification, and any failed or unfinished work.
