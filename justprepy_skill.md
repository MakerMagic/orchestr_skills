---
name: justprepy-teamwork
description: "Orchestrates scoped software-development tasks using a Controller and a Worker: dependency-based delegation, hard-stop approval gates, bounded retries, evidence-based verification, compact handoffs, and owner-controlled commits. Use for multi-step implementation or audits in any project, including Antigravity Teamwork."
---

# Controller–Worker orchestration

This skill is **project-agnostic** despite its name. Apply it only to the task the owner has explicitly assigned. Discover the current repository's rules, architecture, conventions, and approvals; never assume any project's routes, framework, database, or naming.

## Authority and runtime

- **Owner (human):** sets scope, approves product/data-model decisions and risky operations, controls commits, deployment, and any next milestone.
- **Controller (lead agent):** decomposes the approved task, schedules dependent groups, delegates, reviews evidence/diffs, handles blockers, and sends one concise final report. Normally does not implement code itself.
- **Worker (execution agent):** acts on exactly one delegated group, changes only authorized files, verifies its work, and reports facts. Does not independently expand scope or start the next group.
- **External reviewer (if present, e.g. ChatGPT):** evaluates the Controller's final report and decides whether another milestone prompt should be issued. The skill cannot control the external reviewer; give it sufficient evidence.
- Invoke a **real separate Worker agent/session** using the available delegation mechanism; do not simulate Worker messages or silently have the Controller do the Worker's job. If delegation is unavailable, stop and disclose that the requested two-agent workflow cannot run.
- The desired configuration is **one Controller + one Worker**. Antigravity's built-in `/teamwork` may create additional mandatory internal roles; a Skill cannot cap its agent count. Disclose that limitation before execution if an exact two-agent count is a hard requirement. Do not spawn optional extra agents.
- A Skill is a workflow, **not authorization**. The owner's explicit task boundaries and repository instructions take precedence; no implied permission to edit, deploy, change schemas, or commit.

## 1. One-time preflight (Controller)

1. Read applicable project instructions (e.g. `AGENTS.md`) and the owner's current task/stop conditions.
2. Inspect Git status, relevant code/test entry points, runtime constraints, and concurrent agent file/process ownership. Preserve all existing changes; distinguish pre-existing diffs from this task.
3. Confirm the task's output, acceptance criteria, authorized files, and any required human decisions. If essential facts or approval are missing, **stop before edits** and request only the specific missing item.
4. Do not perform broad repository audits, repeated preflights, or planning documents unless genuinely necessary.

## 2. Plan minimally; delegate incrementally

- Create the **smallest useful dependency graph** of subtask IDs (`T1`, `T2`, ...). Each node has goal, dependencies, owned files, acceptance evidence, and stop conditions.
- Group subtasks only when they share an implementation boundary or must be verified together. Order prerequisite groups first. Independent groups can be scheduled later, but never give the Worker all future tasks at once.
- If the task is atomic, send **one Worker assignment** without artificial decomposition.
- Never start an unapproved product milestone merely because it appears in a roadmap. Complete only the current owner-authorized milestone.
- One Worker owns a given file at a time. Use isolated worktrees/workspaces when concurrent contributors may edit overlapping code or share build caches and ports. Never kill another agent's processes, wipe a shared cache, or overwrite unrelated changes.

## 3. Delegation contract (Controller -> Worker)

Send only enough context to execute the **next ready, related group**:

```text
GROUP: G1 | TASKS: T1[, T2]
GOAL: <one measurable outcome>
DEPENDENCIES: <accepted prior task IDs or none>
ALLOWED FILES: <exact paths or tightly bounded paths>
ACCEPTANCE: <observable checks>
STOP IF: <specific hard-stop conditions>
VERIFY: <focused tests; required group checks>
```

The Worker must inspect relevant existing code before edits, preserve project patterns, use the minimum change, avoid speculative features, and not authorize more files or operations for itself. If a necessary dependency is outside the allowlist, **report and stop** rather than silently expanding it.

## 4. Worker result contract

The Worker returns facts, not a chronological terminal transcript:

```text
TASKS: T1[, T2]
STATUS: completed | blocked | failed
CHANGES: <exact added/modified/deleted files, or none>
EVIDENCE: <tests/checks with observed pass/fail, actual diff summary>
ISSUE: <none or exact failure and impact>
IF BLOCKED: confidence=high|medium|low; reason=<why>; retry_condition=<observable trigger>
```

Never claim a test, browser check, database operation, or external action ran unless it actually ran. Separate static checks, mocked checks, real runtime tests, and unverified assumptions. Never expose secrets, tokens, private records, or passwords in reports.

## 5. Controller review after each group

Inspect the Worker's exact changed-file list, scope, relevant diff, acceptance checks, actual failure evidence, and known concurrent changes. Do not accept a mere `completed` assertion as proof.

- **Accepted:** record the accepted working-tree result and concise evidence; delegate the next dependency-ready group within the **same approved milestone**.
- **Fixable defect:** give the Worker only the smallest correction to the **current** group; do **not** open a new milestone or advance a dependent group.
- **Hard stop:** stop all unsafe/dependent work, preserve state, and request the owner's decision. Do not issue a new Worker task that bypasses the stop.
- **Soft blocker:** only defer when the blocked node is nonessential to the next group and cannot invalidate its acceptance. Mark it incomplete, keep its dependency edges visible, and continue solely with genuinely independent authorized work.

**Never accept an overall completion with unresolved mandatory acceptance criteria.** If group checks are unavailable, label them unverified; do not replace runtime evidence with theoretical expectations.

## 6. Failures, confidence, and retry budget

Treat these as **HARD STOP unless the owner already explicitly approved the exact operation and its safeguards**:

- Selecting or changing database tables, fields, types, constraints, ownership policies, or migrations without product-level owner approval. **STOP and request the proposed field/schema contract first**, even if implementation would be easy.
- Changing remote/production data or schema; deploying; publishing; billing, security, credentials, permissions, or authentication changes beyond the approved scope.
- Destructive or history-rewriting Git actions, deleting user data, overwriting concurrent work, exposing secrets, or requiring changes outside authorized files.
- A confirmed security/integrity failure, unexplained data loss, or a failed mandatory gate that makes further work unsafe.

For a **SOFT blocker**, the Controller asks the Worker (once, only when useful): "How confident are you that a later approved task or environment change will resolve this: high, medium, or low? Give the evidence and exact retry condition."

- **high:** reproducible cause and concrete retry trigger; defer and retry once after the trigger occurs.
- **medium:** plausible but uncertain; retry only if new evidence or conditions justify it.
- **low:** do not spend more tokens retrying automatically; report for owner action/re-scoping.

Confidence alone **never** overrides a hard stop, dependency, or failed acceptance criterion. No retry loop: **maximum one automatic retry after new evidence or a real condition change**, unless the owner explicitly authorizes more. Record initial failure and subsequent resolution for the final report.

## 7. Verification strategy (fast but sufficient)

- Worker: run **focused** checks for the affected code after each group and inspect the focused diff.
- Controller: schedule the full required regression suite, typecheck, build, smoke checks, and/or real database checks at the **appropriate milestone boundary**, not redundantly after each tiny subtask. Run earlier when risk, owner instructions, or dependencies require it.
- Do not skip mandatory gates to save tokens. If a verification cannot run safely, state the blocker and its impact.
- Do not confuse matching SQL text with executing it; do not confuse unit tests with an authenticated browser test; do not claim deployment or remote verification from local results.
- Confirm the final changes stay within approved scope and preserve existing behavior.

## 8. End-of-milestone reporting and external gate

Return one compact milestone report using **exactly one** overall outcome:

- `completed` — all required outcomes and checks actually met.
- `completed with caveat` — acceptance met; nonblocking, explicit limitation remains.
- `initially incomplete, but resolved later` — a documented failure was later fixed; name the issue and verified resolution.
- `incomplete - no solution` — mandatory outcome not achieved; describe failure and remaining impact.
- `blocked - owner action required` — waiting on a human decision, credential setup, field approval, or other owner-controlled prerequisite.

Include only:

```text
STATUS: <one outcome>
DELIVERED: <goal and accepted task IDs>
CHANGES: <exact paths, noting unrelated owner changes separately>
EVIDENCE: <observed checks, counts, failures/skips>
ISSUES: <none or concise caveats/deferred IDs and retry triggers>
OWNER ACTION: <none or one concrete decision/action needed>
GIT: <working-tree status; suggested commit readiness only, NOT approval>
```

After reporting, **STOP**. No automatic next milestone, commit, push, release, migration, or deployment. The owner (and external reviewer, if used) decides if the current milestone is accepted and when to issue the next assignment. If a human step is required, report only that step and await its result; do not append new implementation instructions.

## Token discipline

- One concise plan and small group handoffs, not repeated full-context prompts.
- Reuse accepted task IDs, targeted findings, and test evidence; avoid repeating repository background or logs.
- No redundant agent deliberation, exhaustive scans, speculative cleanups, or separate reviews for each trivial line.
- Prefer one focused Worker for a small change; additional agent roles only when runtime requires them or owner authorizes them.
- Record only material decisions and evidence; preserve enough traceability for external review.
