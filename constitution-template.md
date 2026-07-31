# Project Constitution

## Core Principles

### I. Library-First (Module-Mapped)

Every feature MUST start as a module with a single, clear responsibility.
In this project, a "library" maps to <PROJECT: your module unit — e.g.
a service, a package, or a module with a single clear responsibility>.
Modules MUST be self-contained, independently testable, and documented. Organizational-only modules (grouping code without a distinct
purpose) are FORBIDDEN.

**Rationale**: Module-level isolation enables parallel SDD task execution
and prevents cross-cutting coupling that would block evidence-driven
review (Article IV).

### II. CLI Interface

Every module MUST expose its functionality through a CLI. The CLI
contract is text in/out: stdin/args → stdout, errors → stderr. Each CLI
MUST support both JSON output (for machine consumption and piping) and
human-readable output (for operator inspection). This project is CLI-first: <PROJECT: your primary CLI / interaction
surface> is the primary interaction surface; <PROJECT: other layers that
wrap the CLI, or "none" if single-CLI>. Delete this Article if the project
is not CLI-first.

**Rationale**: Text I/O guarantees composability, debuggability
(Article IV), and a clean test surface (Article III).

### III. Test-First (NON-NEGOTIABLE)

TDD is mandatory: tests MUST be written and user-approved before
implementation, then fail (RED), then pass (GREEN), then refactor. The
Red-Green-Refactor cycle is strictly enforced for every code change.
Coverage MUST be ≥ 80% for all new or modified code.

**Rationale**: Test-first catches design defects at the cheapest-fix
point and produces the evidence trail Article IV requires.

### IV. Evidence-Driven, No Fake Completion (NON-NEGOTIABLE)

Every completion claim MUST be backed by command output and `file:line`
evidence. It is FORBIDDEN to:
- use `--ignore` / `--deselect` to skip failing or broken tests;
- present a mock or stub as a live system;
- use `test.skip`, placeholder, or TODO as evidence of completion.

The controller MUST independently re-verify Critical and High review
findings against the actual codebase (do not rely on subagent self-report).

### V. Per-Task STOP for User Approval

After each task in tasks.md, execution MUST STOP and wait for explicit
user approval before the next task. This overrides any continuous-execution
default of the SDD workflow. A plan MUST NOT proceed to execution without
a user review gate after planning ("write plan → STOP → user review").

### VI. Cross-Repository SSOT Discipline

<PROJECT: your repo layout — e.g. "an N-repo workspace: <design-repo>
(docs SSOT) / <code-repo-1> / <code-repo-2> (code SSOTs)", or "single-repo".>

- All git / ls / grep commands MUST carry explicit absolute paths
  (`git -C /abs/path/<target-repo>`) to prevent cwd drift.
- <PROJECT: SSOT split rule — e.g. "Design decisions live only in
  <design-repo>; code changes land only in <code-repos>. Never edit code in
  a code repo from the design repo.", or delete this bullet if single-repo.>
- Re-check cwd and `git status` before editing after any resume or compaction.

### VII. Simplicity (YAGNI)

The simplest solution that satisfies the spec MUST be chosen first. Code
MUST NOT be added for hypothetical future requirements. When two
approaches meet the requirement, the simpler one MUST be selected.
Complexity MUST be justified in the plan or deviation log.

**Rationale**: Speculative branches are carry cost that slows delivery and adds
evidence burden under Article IV.

### VIII. Anti-Abstraction

Abstraction MUST be introduced only when two or more concrete consumers
demand it. Speculative generalization — interfaces, plugin hooks, or
config seams ahead of a second real consumer — is FORBIDDEN. Prefer
duplication over premature abstraction until the common shape is proven;
refactor at the third concrete instance, not the first.

**Rationale**: Premature abstractions historically create more rework cost than they
save.

### IX. Integration-First Testing

Integration tests MUST cover every cross-module flow end-to-end.
Mandatory integration-test focus: new module-contract tests, contract
changes between kaup-core services, kaup-frontend ↔ kaup-core
interaction, shared-schema changes (e.g., `kaup-common/schema.sql`),
and cross-repo flows (Article VI). Unit tests alone are insufficient
for flow-level correctness claims.

**Rationale**: The most damaging defects are often integration-level; unit-green
hides contract drift.

## Governance

**Superpowers Bridge Rule**: Implementation of any task
list MUST follow the Superpowers workflow — worktree → TDD
(red-green-refactor) → per-task fresh subagent execution → whole-branch
code review → finish-branch. `/speckit-implement` is replaced by
`/speckit-kaup-bridge-execute`; `/speckit-converge` remains the step-9
acceptance gate.

**Supremacy**: This constitution supersedes all other practices when in conflict.
**Amendment**: Amendments require documentation, user approval, and a semantic
version bump (MAJOR: principle removal/redefinition; MINOR: new principle or
material expansion; PATCH: wording/clarification).
**Compliance**: Every `/speckit-plan` MUST pass the Constitution Check gate
(Phase -1). Denied gates MUST register an exception in the plan's Complexity
Tracking with a justification.

**Version**: 0.1.0-template | **Ratified**: <PROJECT: date> | **Last Amended**: <PROJECT: date>
