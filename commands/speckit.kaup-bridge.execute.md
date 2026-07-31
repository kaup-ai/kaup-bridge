---
description: "Execute specs/<feature-id>/tasks.md via Superpowers SDD (replaces /speckit-implement)"
---

## User Input

```text
$ARGUMENTS
```

`$ARGUMENTS` is the **feature-id** — the directory name under `specs/` that
holds `tasks.md` and `plan.md` for the feature
(e.g. `2026-07-22-sequence-abort-cli`). It resolves to
`specs/$ARGUMENTS/tasks.md`.

You **MUST** consider the user input before proceeding (if not empty).

This command is the kaup-bridge replacement for `/speckit-implement`: it
drives the Superpowers Subagent-Driven Development (SDD) workflow over a
spec-kit task list, then hands control back to the user for step 9
(`/speckit-converge`).

## Pre-Execution Checks

**Check for extension hooks (before implementation)** — simplified per
spec-kit convention:

- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the
  `hooks.before_implement` key.
- If the YAML cannot be parsed or is invalid, skip hook checking silently
  and continue normally.
- Filter out hooks where `enabled` is explicitly `false`. Treat hooks
  without an `enabled` field as enabled by default.
- For each remaining hook, do **not** attempt to interpret or evaluate
  hook `condition` expressions — leave condition evaluation to the
  HookExecutor implementation.
- For each executable hook, emit the corresponding block per spec-kit
  convention:
  - **Optional hook** (`optional: true`): emit `## Extension Hooks` with
    `**Optional Pre-Hook**: {extension}`, `Command: /{command}`,
    `Description`, `Prompt`, and `To execute: /{command}`.
  - **Mandatory hook** (`optional: false`): emit `## Extension Hooks`
    with `**Automatic Pre-Hook**: {extension}`,
    `Executing: /{command}`, and `EXECUTE_COMMAND: {command}`, then
    actually invoke the hook and wait for it to finish before proceeding.
- If no hooks are registered or `.specify/extensions.yml` does not exist,
  skip silently.

kaup-bridge itself does not register any `before_implement` hooks —
Article V gates the task loop manually via STOP-and-wait, not via an
extension hook. This section is kept for spec-kit interface compatibility
so future extensions can plug in without rewriting this command.

## Outline

1. **Load context** (REQUIRED reads + constitution reference, not inline):

   - **REQUIRED**: Read `specs/$ARGUMENTS/tasks.md` for the complete task
     list and execution plan.
   - **REQUIRED**: Read `specs/$ARGUMENTS/plan.md` for tech stack,
     architecture, and file structure.
   - **IF EXISTS**: Read `specs/$ARGUMENTS/data-model.md`,
     `specs/$ARGUMENTS/contracts/`, `specs/$ARGUMENTS/research.md`,
     `specs/$ARGUMENTS/quickstart.md`, and
     `specs/$ARGUMENTS/checklists/` for supporting context.
   - **REQUIRED**: Read .specify/memory/constitution.md Articles
     IV / V / VI for governance constraints (evidence-driven / per-task
     STOP / cross-repo SSOT). The Articles are the **single SSOT** — do
     not paste their text into this run; re-read the file on each task.

2. **Branch decision** — worktree (default) vs main-line:

   - **Default**: create a git worktree per
     `superpowers:using-git-worktrees` and run all task execution inside
     it.
   - **Main-line (exception)**: only when the user explicitly authorises
     linear-main execution (e.g. a project-authorised linear-main convention). Record the
     authorisation in the task log before proceeding.
   - This decision becomes the branch context for the whole-branch review
     in step 5.

3. **Per-task SDD loop** — for each task in `tasks.md`:

   - **Dispatch ONE fresh subagent per task** per
     `superpowers:subagent-driven-development`. The implementer starts
     with a clean context, reads the task brief, verifies `git status`
     and cwd (absolute paths — Article VI) before any edit, and finishes
     by marking the task `[X]` in `tasks.md`.
   - **TDD red-green-refactor** per `superpowers:test-driven-development`:
     write a failing test → write the minimal implementation that turns
     it green → refactor. Tests are real — no `test.skip`, no
     `--ignore` / `--deselect`, no mock or stub presented as a live
     system (Article IV).
   - **Reviewer pass** (FR-013, per-task tier = **haiku**): each task
     gets a separate reviewer subagent (fresh context, distinct from the
     implementer) at the **haiku** tier that files findings with
     `file:line` evidence. The controller independently re-verifies
     every Critical / High finding against the actual codebase — do not
     trust subagent self-report (Article IV). Rationale: per-task review
     is slice-scoped (each reviewer sees only one task's diff), so a
     fast model suffices to catch contract gaps and verify the reverse-
     TDD cycle within that slice. The fresh-subagent-per-task model
     carries an inherent blind spot — no single-task reviewer sees the
     cross-task picture — which is why the whole-branch **opus** layer
     in step 5 exists as the single backstop for cross-slice breaks.
   - **STOP for user approval** before the next task (Article V).
     Messages like "continue W4-N" are planning authorisation, not
     execution authorisation; wait for an explicit per-task go.
   - **Mark `[X]` in `tasks.md`** only after the reviewer pass completes
     with zero unresolved Critical / High findings.
   - **Controller Discipline (C1)** — every bash verification command the
     controller runs MUST carry an explicit target-repo absolute path:
     either `cd /abs/path/<target-repo> && <cmd>` or
     `git -C /abs/path/<target-repo> ...`. MUST NOT rely on residual cwd.
     Rationale: the dry-run produced 2 cwd-drift incidents that emitted
     false-green evidence; this rule operationalises Article IV
     (evidence-driven, no fake completion) and Article VI (cross-repo
     absolute paths). Also: re-check cwd and `git status` after any
     resume / compaction / session restart before editing.

   - **Controller Direct-Action Policy (B2/B3)** — the default set by the
     "Dispatch ONE fresh subagent per task" bullet above is **dispatch**;
     the controller executing a task directly (skipping the
     fresh-implementer dispatch) is a **controlled exception**, never the
     rule. Two scenarios permit direct action:
     - **B2 — dispatch-failure fallback (FR-010)**: when a
       fresh-implementer dispatch FAILS (quota / enforcer 403, connection
       death, provider error) AND retry is exhausted or impractical, the
       controller MAY execute the task directly as a fallback rather than
       block. Because the controller now becomes the author, it MUST
       dispatch a separate fresh reviewer subagent (distinct from itself)
       to preserve the author ≠ reviewer separation — the dispatch failure
       does not exempt a real task from review.
     - **B3 — small change (FR-011)**: the controller MAY execute
       directly WITHOUT dispatch when the change is small, defined as ALL
       of: < 20 lines, single file, no design decision (e.g. typo,
       help-text, doc wording, a config line, a spike / verification with
       no committed artifact). B3 changes are non-logic / trivial, so the
       per-task fresh-reviewer dispatch is ALSO skipped — its cost would
       exceed its benefit (the very problem B3 exists to solve). Review
       instead relies on the controller's objective self-verification
       (grep / diff) plus the step 5 whole-branch opus review as backstop.
       This is the intentional B2 / B3 distinction: a B2 real task that
       failed to dispatch still warrants a fresh reviewer; a B3 trivial
       change does not. Non-code B3 changes (typo / doc wording) are also
       exempt from the TDD red-green-refactor cycle (the L90-94 TDD
       mandate applies to code changes).
     - **Ceiling (FR-011)**: a LARGE change — logic, cross-file, or any
       design decision — MUST be dispatched to a fresh implementer + a
       separate reviewer. The controller MUST NOT absorb a large change
       directly. If a large change cannot be dispatched (persistent
       failure), STOP and escalate to the user rather than silently doing
       it.
     - **Mandatory deviation record (FR-012)**: EVERY controller-direct
       action (whether B2 fallback or B3 small change) MUST be logged as
       a deviation entry in `specs/<ARGUMENTS>/CHANGELOG.md` with the
       task ID, the reason (B2 dispatch-failure / B3 small-change), and
       the scale (lines / files / what). This is the same CHANGELOG
       mechanism step 4 ("Deviation handling") describes for small
       deviations — it is not a separate ledger. A controller-direct
       action is never invisible.

     Rationale (FR-010 / FR-011 / FR-012, constitution Article IV): the
     dry-run (a prior feature) produced DEV-02 (quota
     403 → B2 dispatch-failure fallback) and DEV-03 / DEV-04 (small-change
     dispatch where the dispatch cost exceeded the benefit → B3) — both
     real occurrences, not hypothetical. This policy formalises existing
     practice: the present feature's execute pass used controller-direct
     action repeatedly (spikes, B3 micro-fixes). The per-task STOP
     (Article V) is NOT waived by direct-action — the controller still
     gates the next task on explicit user approval.

4. **Deviation handling** (two layers):

   - **Small deviation** (typo, path correction, detail nuance, missing
     import, etc.): append an entry to
     `specs/$ARGUMENTS/CHANGELOG.md` with task ID + delta + reason. Do
     **not** modify `tasks.md` inline — the plan is the execution SSOT
     and stays stable across the pass.
   - **Large deviation** (logic error in plan, missing or extra task,
     scope change, architectural surprise): **STOP and escalate to the
     user**. Present two paths: (a) go back to the spec-kit phase to fix
     `tasks.md` and re-plan, or (b) park the gap in a follow-up file and
     finish the current pass. Do not silently absorb large deviations.

5. **Whole-branch code review** (FR-013 / FR-017, whole-branch tier =
   **opus**) — when every task in `tasks.md` is marked `[X]`:

   - **Non-optionality (FR-017)** — this whole-branch review is
     **mandatory and MUST NOT be skipped for ANY multi-task feature,
     regardless of size**. Necessity is unrelated to feature size: the
     dry-run feature (a prior small feature) still produced the
     cross-slice breaks (integration / E2E / contract drift) detailed
     in the opus-dispatch bullet below — catches that only whole-branch
     review made. Any feature with more than one task carries
     cross-slice-break risk because the fresh-subagent-per-task blind
     spot is structural, not size-dependent; the whole-branch pass
     therefore runs for every multi-task feature. There is no "too
     small to bother" threshold, and feature size is never a reason to
     skip this layer. (This declares that the pass MUST happen; the
     opus-dispatch bullet below says WHAT the reviewer does; the GATE
     audit bullet further down says what ELSE it does — three distinct
     concerns.)
   - Dispatch a fresh **opus** reviewer per
     `superpowers:requesting-code-review` over the **entire branch diff**,
     not per-task. **opus** is the deep, cross-task integration tier —
     the only layer that sees the full branch diff and can catch
     cross-slice breaks no single-task reviewer could see. Dry-run
     evidence: per-task haiku reviewers each Approved their own slice,
     yet the whole-branch opus pass caught cross-slice breaks (integration,
     E2E, contract drift) — breaks that confirm the
     fresh-subagent-per-task blind spot is real, not hypothetical.
   - Findings bucketed Critical / High / Medium / Low. Critical and High
     are blockers; Medium and Low may be deferred to a follow-up with
     user sign-off.
   - The controller re-verifies every Critical / High finding
     independently (Article IV — do not trust subagent self-report).
   - **Fallback (FR-014)** — model-tier degradation policy (asymmetric
     by design):
     - **Per-task reviewer (haiku)**: if `haiku` is unavailable, the
       per-task reviewer drops to **sonnet**. Slice review remains
       useful at the sonnet tier — the per-task layer is fast feedback,
       not the integration backstop.
     - **Whole-branch reviewer (opus)**: does **NOT** drop. Integration
       review is the single backstop for the per-task blind spot, so if
       `opus` is unavailable, BLOCK and wait for opus rather than
       downgrading — do not silently let cross-task breaks slip. The
       asymmetry is intentional: a degraded per-task pass still adds
       value, but a degraded whole-branch pass would negate the entire
       reason the two-layer model exists (the dry-run M1/M3/M4 catches
       were opus-only).
     - Models use tier aliases (`haiku` / `sonnet` / `opus`) resolved by
       the provider.
   - **Constitution GATE audit (FR-015 / FR-016)** — the whole-branch
     opus reviewer has an ADDITIONAL duty beyond the branch diff: it MUST
     also re-review the feature's `plan.md` Constitution Check +
     Complexity Tracking section and verify GATE honesty. The
     constitution GATE is a declarative principle-check (Phase -1 of
     `/speckit-plan`, evaluating the plan against all 9 Articles) — a
     semantic judgment, not a script-checkable gate. Per the
     constitution's Compliance section, *"Denied gates MUST register an
     exception in the plan's Complexity Tracking with a justification"* —
     this is the EXISTING public-audit mechanism the reviewer audits, not
     a new one. The forgery-catch rule (FR-016): if the plan ACTUALLY
     violates a constitution Article but its Complexity Tracking does NOT
     register the violation, that is a GATE forgery — the reviewer flags
     it as a **Critical** finding. Rationale (FR-015 / FR-016, Article IV,
     brainstorm D8): the GATE is enforced by honesty + reviewer backstop,
     exactly parallel to Article IV ("the controller MUST independently
     re-verify Critical and High review findings ... do not rely on
     subagent self-report") — the GATE likewise does not trust the plan's
     self-reported "PASS". Automating the GATE with a keyword grep would
     only catch surface violations; semantic compliance (e.g. whether a
     plan's design truly violates Article VIII Anti-Abstraction) is a
     judgment only an LLM reviewer can make. This reuses the plan.md
     Complexity Tracking mechanism the constitution already defines — no
     new mechanism, no constitution amendment.

6. **Finish branch** per `superpowers:finishing-a-development-branch`:

   - Linear-ff or squash per repo convention (follow the project's branch
     convention).
   - Commit message follows `<type>(<scope>): ...` (types: `feat` / `fix`
     / `refactor` / `docs` / `test` / `chore` / `perf` / `ci`). No
     `Co-Authored-By` attribution — globally disabled in
     `~/.claude/settings.json`, and pre-commit hooks reject it.
   - **Commit Discipline (C2 / C3)** — before `git commit`, run the
     target repo's formatter to avoid pre-commit-hook abort loops
     (FR-005 / C2): run the target repo's formatter (project-configured — e.g.
     `ruff format`, `pnpm format`, `gofmt`, or none for a pure-docs repo). When capturing the commit
     hash for the ledger, first check the commit's exit code
     (FR-006 / C3): `COMMIT_EXIT=$?`; only if `COMMIT_EXIT -eq 0` run
     `HASH=$(git rev-parse --short HEAD)`. If the commit failed, do NOT
     write a stale hash to the ledger — diagnose the failure first.
     Rationale: the dry-run produced 1 hash-misread incident; this rule
     keeps the evidence chain intact (Article IV).
   - **Do not `git push` automatically** — push is the user's independent
     decision. If the project has a pre-push hook (e.g. a full test + type-check
     suite), confirm it is green before proposing push.
   - **Explicit non-action**: this command does **not** run
     `/speckit-converge`. Converge is step 9 of the hybrid workflow and
     is invoked separately by the user after this command reports
     completion.
   - **Converge defer-judgment SOP (FR-018, F1)** — although execute
     does not invoke converge, it hands off to converge, so the
     classification converge MUST apply is documented here for
     continuity. When converge (the post-execute acceptance
     reconciliation) compares the codebase against the spec, it MUST
     distinguish two kinds of findings:
     - **Spec gap** — a hard spec requirement (an FR or Acceptance
       Scenario) that is NOT implemented in the codebase → **append a
       task to `tasks.md`**. The feature is not truly done while a spec
       hard-requirement is missing; appending keeps the `tasks.md`
       "完成 / `[X]`" semantics honest.
     - **Phase-2 enhancement** — an improvement that is NOT a spec
       hard-requirement (environment hardening, concurrency / load
       tuning, performance optimisation, coverage-tooling additions,
       DX ergonomics) → record in `docs/superpowers/follow-ups/` (or
       `specs/<feature-id>/follow-ups/`), **NOT in `tasks.md`**.
     Rationale (FR-018, brainstorm D9): converge is an acceptance
     reconciliation, not an enhancement backlog. Misclassifying a
     Phase-2 enhancement as a spec gap pollutes `tasks.md`'s "完成"
     semantics — once an enhancement is listed as a task, "all tasks
     `[X]`" no longer means "spec satisfied" but "spec satisfied +
     optional extras done", which is contract-rigidity creep. The
     dry-run converge was 0-finding, empirically validating this split:
     coverage-tooling absence and the D5 cross-connection were
     correctly judged Phase-2 enhancements, NOT spec gaps.

Note: This command assumes a complete task breakdown exists in
`specs/$ARGUMENTS/tasks.md`. If tasks are incomplete or missing, suggest
running `/speckit-tasks` first to regenerate the task list.

## Done When

- [ ] All tasks in `specs/$ARGUMENTS/tasks.md` completed and marked `[X]`
- [ ] Each task had a fresh implementer subagent + separate reviewer + TDD red-green-refactor + explicit user approval gate (Article V)
- [ ] All Critical / High review findings resolved (controller re-verified per Article IV); Medium / Low either fixed or deferred to a follow-up with user sign-off
- [ ] Whole-branch `opus` review completed over the full branch diff
- [ ] Whole-branch reviewer re-audited the feature's `plan.md` Constitution GATE for honesty (FR-015 / FR-016); any GATE forgery (an actual Article violation NOT registered in the plan's Complexity Tracking) flagged as Critical
- [ ] Small deviations logged to `specs/$ARGUMENTS/CHANGELOG.md`; large deviations escalated and resolved (plan fix or follow-up) — `tasks.md` itself not silently edited
- [ ] Cross-repo discipline upheld (Article VI): all git / ls / grep commands carried explicit absolute paths, cwd and `git status` re-checked after any resume, compaction, or session restart
- [ ] Branch finished per `superpowers:finishing-a-development-branch`; push deferred to the user
- [ ] `/speckit-converge` explicitly **not** invoked by this command (step 9, separate invocation)
