---
name: delegate-to-luna-adaptive
description: Delegate ordinary bounded non-trivial repository work to GPT-6 Luna with per-task reasoning effort and quick, standard, or deep verification. The primary agent retains consequential decisions and acceptance. Keep unclear requirements, tightly coupled, and high-risk work with the primary agent.
---

# Delegate to Luna Adaptive

The primary agent scopes requirements, risk, relevant files, acceptance, and boundaries, then hands ordinary bounded non-trivial work to Luna before detailed diagnosis. Luna is the preferred executor and may choose a suitable local pattern or diagnose and fix within one boundary. The primary keeps unclear requirements, consequential decisions, integration, and acceptance. Route Luna explicitly and choose effort per task.

## Select the workflow and controls

Preserve `policy.allow_implicit_invocation: false`. An explicit invocation selects this workflow for the task over the automatic Max workflow. Do not combine delegation workflows. Merely mentioning, comparing, or editing a skill does not invoke it. A later explicit user selection controls the affected work.

These are prompt conventions, not native tool parameters:

- `effort=low|medium|high|xhigh|max`: pin the supported child effort when specified. Otherwise choose per task.
- `verification=quick|standard|deep`: default to `quick` to prioritize low usage and latency. Honor an explicitly selected mode.
- `verification=false` or `驗證=false` means `quick`; `verification=true` or `驗證=true` means `standard`.

Treat effort and verification as separate controls. Preserve the selected controls for continuing work until the user changes them. If conflicting values remain after considering the latest explicit instruction, clarify only the unresolved setting. State chosen verification mode and planned child effort briefly before delegation.

## Spend less on coordination

Prefer one child for a bundled deliverable. At most three Luna children may be active concurrently for this task, counting every child role including verifiers. Choose one, two, or three only when each lane has independent useful work and disjoint files or resources; this is a ceiling, never a quota. Parallelism can increase total usage, so bundle related edits and targeted checks instead of splitting investigator, writer, and tester roles. Avoid speculative OPTIONAL lanes and routine review agents.

Keep briefs around 150-250 words and ordinary returns within 150 words when sufficient; never omit required boundaries or evidence to meet these targets. Send paths, relevant symbols, and changed facts rather than whole files, logs, skills, or repeated history. Batch independent scoped reads; reuse current evidence instead of rediscovering it. Store long logs in artifacts and return only outcomes, relevant errors, and paths.

## Choose useful work and effort

Delegate ordinary bounded non-trivial work by default when its goal and acceptance can be stated compactly, including short, straightforward implementation, tests, documentation, and safe investigation. Skip only truly trivial work when the full coordination cost exceeds its benefit. Luna may implement, investigate, test, debug, refactor, or document. Keep unclear user requirements, consequential architecture decisions, high-risk or irreversible actions, and tightly coupled work with the primary agent; safe bounded evidence-gathering toward those decisions may still go to Luna. Higher effort does not make unsuitable work suitable.

Without a user-pinned effort, choose the lowest credible supported level:

| Effort | Suitable reasoning demand |
| --- | --- |
| `low` | Clerical or deterministic read-only work; avoid production-code changes. |
| `medium` | Clear local patterns: documentation, simple tests, bounded implementation. |
| `high` | Non-trivial logic, debugging, refactoring, or test design. |
| `xhigh` | Difficult but bounded multi-file tracing, state interactions, or concurrency. |
| `max` | Exceptional bounded reasoning beyond the lower levels. |

Use `medium` for ordinary bounded implementation and `low` for deterministic read-only tasks. Select `high` or above only when the brief identifies the concrete reasoning difficulty. Choose from task complexity, not file count, verification mode, or unused capacity.

## Set verification once

- `quick`: No optional tests or independent verifier. Run checks required by the user, repository, deliverable format, or safety constraints. The primary agent inspects the artifact/diff once and reports untested behavior.
- `standard`: The writer runs the smallest relevant targeted checks once. The primary agent reviews changes and evidence. Add an independent verifier only for a concrete concern that remains unresolved.
- `deep`: Add broader relevant checks when they provide new evidence. Consider a read-only independent verifier for subtle behavior or weak coverage; it is not automatic.

All modes retain acceptance criteria, mandatory checks, ownership review, and concern resolution. Assign each check one owner. A passing check remains usable unless later changes affect what it tested, evidence is missing, or a specific risk justifies repeating it. Failed checks require focused repair, not a restart of all verification.

A verifier receives original requirements, relevant files/diff, and its checks without the writer's conclusions. It never writes files. Choose verifier effort independently unless the user pinned effort; it supplements the primary agent's acceptance review.

## Plan small waves and boundaries

Use successive waves within the three-child task cap and live available capacity. Distinguish total-agent limits (including the primary agent) from child-only limits. Queue excess work; never spawn merely to fill slots. The cap includes all concurrently active Luna roles, including writers, investigators, and verifiers.

Before assigning writers, record existing changes using a scoped Git diff/status or file snapshots outside Git. Parallel writers need disjoint files and mutable resources. Shared files, generated outputs, services, and dependent tasks require sequencing. Preserve pre-existing and concurrent changes. The primary agent may work on independent tasks but must not duplicate an active child's investigation, edits, or checks.

Track each child's id, owned files/resources, effort, priority, dependencies, and status. Keep delegation one level deep unless the user explicitly authorizes nested delegation.

## Bound decisions before execution

For implementation, the primary agent supplies fixed requirements, acceptance criteria, safety boundaries, known relevant files, and any helpful local reference; it does not need to choose every implementation detail. Hand off before detailed diagnosis or solving. Luna may inspect the allowed files, use the first local pattern that fits, and combine diagnosis and repair within one explicit boundary. Keep unresolved user requirements and consequential architecture or risk decisions with the primary agent; Luna may gather bounded evidence and return it before crossing those decisions. Use a separate investigation lane only when its question and evidence are independently useful or a primary-owned decision must be made before implementation.

Apply these defaults unless the brief justifies a different limit:

- Follow the first existing pattern that meets acceptance. Do not compare alternatives, redesign abstractions, or reopen settled choices without contradictory evidence. If the chosen approach conflicts with correctness or safety, report the conflict instead of following it blindly.
- Investigations assess at most two evidence-backed hypotheses. Each further read must answer a named unresolved question within Allowed reads; stop discovery once enough evidence supports the assigned result. Report remaining uncertainty when the search limit is reached.
- After a failed assigned check, allow one focused repair and rerun the affected check once. If still failing, return the partial result and failure evidence; the primary agent chooses the next step under the existing retry limit.
- Reopen a decision only for a new requirement, concrete contradictory evidence, or a failed check. Record hypothetical concerns briefly; they do not authorize more work. Material uncertainty blocking acceptance returns BLOCKED, never DONE.

These are observable workflow limits, not enforceable caps on internal reasoning tokens or elapsed time. High effort does not authorize additional scope. Once acceptance and assigned checks pass, return immediately.

## Give a compact brief

Read enough authoritative context to define the boundary; leave scoped execution discovery to the child.

```text
Goal: one deliverable and observable acceptance criteria
Context: exact relevant paths and existing/concurrent changes
Decisions: fixed requirements; allowed local choices; optional known reference; decisions reserved for the primary agent; or one bounded investigation question
Allowed reads: bounded paths, dependencies, commands, or sources
Allowed writes: exclusively owned files/resources, or none
Boundaries: prohibited actions; preserve others' changes; no nested delegation
Stop/budget: acceptance met and assigned checks complete; bounded search/check scope; report BLOCKED when exhausted
Checks: verification mode, specific checks and owners, or none if quick permits
Dependencies: prerequisite results, or none; REQUIRED unless explicitly OPTIONAL
```

Merge fields when useful; do not paste unrelated history. Include known local dependencies and applicable instructions in the read boundary without authorizing broad discovery. If the child needs more access or finds an ownership conflict, it returns `BLOCKED` with the path/action and reason. The primary agent may amend the brief within existing user authorization; only missing authority or requirements require user input.

Stop after the deliverable and assigned checks; no opportunistic cleanup or open-ended verification. Require this concise return:

```text
Status: DONE | DONE_WITH_CONCERNS | BLOCKED | FAILED
Files: changed paths or none
Result: behavior or findings
Checks: command, outcome, and relevant artifact/output; or none
Concern: unresolved issue, assumption, missing scope, or none
```

`DONE_WITH_CONCERNS` means the deliverable and assigned checks are complete with a specific residual concern. Incomplete acceptance or missing mandatory checks require `BLOCKED` or `FAILED` and the partial result. A completed investigation may report an unresolved finding.

## Route and retry deliberately

Inspect the live collaboration schema and explicitly set:

```text
fork_turns="none"
model="gpt-6-luna"
reasoning_effort=<selected supported effort>
```

Use a small positive history fork only when essential recent context cannot be expressed in the brief and the tool supports explicit overrides with it. Never use a full-history fork or omit routing. If the model/effort is unavailable, disclose the limitation and take over in the primary agent; do not silently substitute a model or change pinned effort.

For capacity or transient rate-limit failure, wait for capacity or the indicated retry window, then retry once with the same routing. Do not blindly retry unsupported parameters/models. Tool acceptance confirms requested routing; prove actual model execution with available runtime metadata when needed, not the child's self-description.

After a blocked or failed task, classify the cause before any retry:

- Missing context or unclear criteria: amend the brief and retry only unfinished work at the same effort.
- Demonstrated reasoning difficulty: if effort was not user-pinned and work remains suitable, raise it one supported level for the unfinished portion.
- Broad scope, coupling, architecture, safety, permissions, or environment blockers: split only genuinely independent work, take over, or report the blocker.

Allow at most one revised execution attempt per deliverable. When the remaining work is safe and bounded, prefer a focused correction with the same child within that limit; do not take over merely for convenience. If the revised attempt fails or scope, coupling, or risk changes, the primary takes over. Never redo completed work merely to increase confidence. Before changing ownership, confirm the previous writer stopped. If the follow-up tool cannot change effort, spawn a new explicitly routed child with the current partial state and remaining work; never pretend a message changes model settings.

## Join, then accept

Every child is joined work. Review a completed lane and release dependent work once its prerequisites are terminal and accepted, provided active lanes have disjoint files/resources. Unrelated slow lanes do not create an all-wave barrier. A progress message or timeout is not completion. Prefer event-driven waits over repeated polling; bound waits to allow user updates. Inspect status only when completion or ownership is unclear.

If available status/activity does not explain a delay, send one non-interrupting progress request or scope correction, then allow a response at a message boundary. Do not repeatedly ping, duplicate the investigation, or interrupt solely for slowness.

Use `send_message` for a running child. Use `followup_task` or the live equivalent that starts a turn when an idle/terminal child needs more work; a message alone may not restart it. If a terminal child lacks a report, inspect artifacts and check evidence first, then recover only the missing evidence once if needed.

The primary agent reviews the actual diff/artifact against the baseline, acceptance criteria, ownership, and user changes; inspects assigned check results; and resolves material concerns. Do not redo correct work or passing checks. Missing evidence, suspicious logic, scope violations, or failures need focused follow-up before claiming completion.

If the user cancels or replaces work, stop affected children. Before finalizing, collect every required result and explicitly stop unneeded OPTIONAL work. Confirm none of this request's children remain active; a requested interruption is not itself proof they stopped.

Briefly report the result, verification mode, actual child count/efforts, changed files, performed/skipped checks, and material limitations. Mention retries or capacity fallback only if they occurred. Do not claim measured speed or token savings without evidence.
