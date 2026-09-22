---
name: delegate-to-luna-adaptive
description: Delegate independent, bounded repository work to GPT-6 Luna agents while choosing each agent's reasoning effort from task complexity. Luna may implement changes; the current primary agent retains decomposition, integration, and risk-based verification. Skip trivial, ambiguous, tightly coupled, architecture-wide, and high-risk work.
---

# Delegate to Luna Adaptive

The current primary agent orchestrates this workflow regardless of its model or reasoning setting. Do not inspect, assume, require, or claim a particular primary model. This skill does not change the primary model. It explicitly routes only suitable child work to GPT-6 Luna and selects reasoning effort separately for every child.

## Workflow Precedence

When the user explicitly invokes this skill, it owns the delegation workflow for that turn. Do not also apply `$delegate-to-luna-max` or `$luna-task-owner` to the same work. If the user explicitly selects another delegation skill instead, follow that skill.

Keep this skill explicit-only in `agents/openai.yaml` so the user can choose it deliberately.

## Set Verification Depth

Read the verification setting from the user's request. Accept `verification=quick|standard|deep`; treat `verification=false` or `驗證=false` as `quick`, and `verification=true` as `standard`. Default to `standard` when unspecified. This is a prompt convention interpreted by the agent, not a built-in skill parameter. The setting controls verification effort, not the child's reasoning effort or the acceptance criteria.

- `quick`: Use when time is tight. Skip optional tests and independent verification agents. The writer runs only checks explicitly required by the user, repository instructions, or the deliverable's format or safety constraints. The primary agent inspects the diff or artifact once, checks the acceptance criteria and file ownership, and reports behavior that was not tested.
- `standard`: The writer runs the smallest relevant targeted checks once. The primary agent reviews the diff and the writer's evidence; it does not rerun passing checks without a specific unresolved risk. Add an independent verifier only for a concrete concern the primary agent cannot resolve with that evidence.
- `deep`: Use when the user asks for stronger assurance or the task warrants it. Add broader relevant checks and consider an independent read-only verifier for subtle or higher-impact changes. Assign each check an owner and avoid repeating a passing check without a reason.

The mode does not waive explicit user requirements, repository-required checks, or basic inspection of changed files. If a required check cannot be completed, report that limit instead of implying the change was verified. State the chosen mode before delegating so children receive the same budget.

## Decide What Luna Owns

Delegate when work can be expressed as independent, bounded deliverables with observable success criteria. Luna may investigate, implement, test, debug, refactor, or document within an exact scope. Prefer delegating meaningful execution rather than using Luna only for advice.

Keep ambiguous requirements, architecture-wide decisions, security-critical changes, irreversible migrations, and tightly coupled cross-cutting work with the primary agent. A higher reasoning effort does not make an unsuitable task suitable for Luna.

Use successive waves. Spawn multiple children only when their work is genuinely independent. Determine wave size from the independent task count and currently available child slots; do not hard-code a primary-model-dependent limit and do not fill capacity without useful work. Keep delegation one level deep: Luna children must not spawn their own agents unless the user explicitly requests nested delegation.

Maintain a parent-side lane ledger containing each lane's task id, role, owned files, dependencies, requested effort, required/optional status, current state, and whether a final payload was received. Queue excess lanes instead of treating the configured concurrency limit as a target.

Before spawning writers, inspect and record the working tree's starting state. Parallel writers must have disjoint file ownership. Tell every writer that it shares the workspace, must preserve pre-existing and concurrent changes, must not revert work it did not create, and must stop on an ownership conflict. Tasks that touch the same file, depend on unfinished results, or mutate shared state run sequentially. The primary agent must not edit a child's owned files while that child is active.

## Select Reasoning Effort Per Task

Inspect the live `spawn_agent` schema and explicitly set the model and reasoning effort for every child. Choose the lowest effort that is credible for the task, based on reasoning complexity and verification burden rather than file count alone:

- `low`: clerical or deterministic read-only work with almost no inference. Avoid for production-code changes.
- `medium`: repository mapping, reference searches, documentation, simple tests, and mechanical changes with strong local patterns.
- `high`: default for bounded implementation, debugging, refactoring, test design, and review that requires following non-trivial logic.
- `xhigh`: difficult multi-file tracing, subtle state behavior, concurrency analysis, or edge cases that remain clearly bounded.
- `max`: exceptional bounded work requiring Luna's deepest available reasoning. Do not use it by default or as a substitute for primary ownership of ambiguous or high-risk work.

Different children in the same wave may use different efforts. If the task remains unsuitable even at `max`, keep it with the primary agent.

## Build a Compact Task Brief

The primary agent reads enough authoritative context to define the boundary, then sends only the context the child needs. Every brief includes:

```text
Goal:
<one primary deliverable>

Relevant files:
<exact files or narrowly scoped directories>

Context:
<minimum architecture and behavior context>

Starting state and coexistence:
<relevant pre-existing changes, concurrent lanes, and preservation rules>

Allowed reads:
<exact paths, commands, or data sources>

Allowed writes:
<exact files exclusively owned by this child, or "none">

Do not:
<prohibited files, systems, actions, and scope>

Scope expansion gate:
If anything outside Allowed reads or Allowed writes is needed, stop and return BLOCKED with the missing scope and reason.

Expected result:
<observable behavior or findings>

Acceptance criteria:
<specific requirements that must each be demonstrated>

Verification mode:
quick | standard | deep

Stop when:
<completion and verification conditions>

Verification:
<checks required by the mode and task, with one owner per check; state "none" when quick mode has no required checks>

Result priority:
REQUIRED | OPTIONAL
```

Do not include unrelated chat history. A child stops after the requested result and verification instead of adding opportunistic work.

Require this return contract:

```text
Status: DONE | DONE_WITH_CONCERNS | BLOCKED | FAILED
Reasoning effort used: low | medium | high | xhigh | max
Changed files: <paths or none>
Behavior implemented or Findings: <summary>
Tests/checks run: <commands and outcomes>
Assumptions: <assumptions or none>
Unresolved issues: <issues or none>
Evidence: <paths, commands, outputs, or hashes>
```

## Spawn Luna Explicitly

For each delegation, set:

```text
fork_turns="none"
model="gpt-6-luna"
reasoning_effort=<selected low | medium | high | xhigh | max>
```

Use `fork_turns="none"` by default. A small positive history fork is allowed only when essential recent context cannot be expressed compactly. Never use a full-history fork for convenience. If explicit Luna routing or the selected effort is unavailable, do not silently fall back to an inherited model; disclose the fallback and let the primary agent take over or choose a supported effort.

On capacity or rate-limit errors, reduce the active wave, queue excess lanes, and retry the spawn once without changing the requested model or effort. If that fails, keep the lane with the primary agent or report the limitation; do not create an unbounded retry loop.

## Join Every Wave

Record every child and whether its result is REQUIRED or OPTIONAL. Wait for all REQUIRED children to reach a terminal state before integration or dependent work. A timeout is a progress checkpoint, not failure. Inspect status before steering, and send at most one concise course correction when a child is drifting or its progress is genuinely unclear.

Progress messages and status without a final payload are not completion. Attempt one targeted recovery of a missing final report using the available status or follow-up tools; if it remains missing, rerun only that lane with a narrower brief or let the primary agent take over.

Do not leave children running when finalizing. If user input replaces or cancels the work, stop affected children when supported.

## Handle Blocks and Effort Escalation

Do not blindly rerun a blocked or failed lane. Classify the cause first:

- Missing context or unclear acceptance criteria: correct the brief and retry once at the same effort.
- Demonstrated reasoning difficulty: if the task remains bounded and suitable for Luna, retry once at the next supported effort level.
- Scope too broad or coupled: split it into smaller independent lanes, or return it to the primary agent.
- Architecture, safety, permissions, or environment blocker: keep the decision with the primary agent or report the blocker.

If the same cause repeats after the revised attempt, stop escalating and let the primary agent take over. Never rerun completed work at a higher effort merely because confidence is low; use targeted verification instead.

## Add Independent Verification When Worthwhile

In `quick` mode, do not add an independent verifier. In `standard` mode, add one only for a concrete unresolved concern. In `deep` mode, consider a fresh read-only Luna verifier for subtle behavior, higher-impact code, weak test coverage, or a change whose writer raised concerns. Skip this lane when the primary review and existing evidence are sufficient.

Give the verifier the original requirements, acceptance criteria, relevant diff or files, and checks to run, but not the writer's conclusions. Select verifier effort independently; use the same effort as the implementation or one level higher only when the verification itself requires more reasoning. The verifier checks specification compliance first, then correctness, regressions, and test gaps. It reports evidence and never edits files.

An independent Luna verifier supplements but never replaces the primary agent's final acceptance gate.

## Apply a Risk-Based Quality Gate

Luna may be the primary writer for suitable delegated files. The primary agent does not redo correct work merely because Luna produced it. For every verification mode, it performs one baseline acceptance review:

1. Inspect the actual diff or artifact, not only the child's summary.
2. Check every acceptance criterion before reviewing style or polish.
3. Confirm file ownership, requested behavior, preservation of starting-state changes, and absence of unrelated edits.
4. Inspect the results of checks assigned to the writer or verifier; rerun only a missing or failed check, or one affected by a concrete new risk.
5. Evaluate unresolved concerns and assumptions against the original request.

Scale work beyond that baseline to the chosen mode and actual risk:

- `quick`: Stop after the baseline review and any mandatory checks; disclose skipped behavior tests.
- `standard`: Review ordinary implementation logic and regressions against the targeted checks already run.
- `deep`: Review subtle or higher-impact logic in more detail and use additional verification where it adds evidence.
- In any mode, failed checks, suspicious logic, scope violations, or material uncertainty require a focused repair or follow-up before claiming completion.

Do not wait for a user-visible bug before reviewing. Conversely, do not spend primary-model tokens reimplementing a change that has passed proportionate verification. Keep critical architecture and high-risk corrections with the primary agent.

## Final Report

Report the verification mode, actual number of waves and children, each child's role and reasoning effort, material Luna-authored changes, outputs accepted, modified, or rejected, checks performed and skipped, any retries or effort escalation, and any capacity or routing fallback. Never claim an agent, effort, or check ran when it did not.

```text
User -> current primary agent -> bounded briefs with per-task effort
     -> one or more independent Luna writers/investigators
     -> joined results -> risk-based primary verification and integration
```
