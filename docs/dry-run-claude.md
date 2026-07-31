# Dry-run: Claude Code install + render (T7)

Date: 2026-07-31
Project: /tmp/kaup-bridge-drill
Spec-kit CLI: specify 0.15.0
Integration: claude (default)
Extension source: ~/workspace/kaup/kaup-bridge (installed via `--dev`, symlinks point into `.specify/extensions/kaup-bridge/.specify-dev/`)

## Step 1 specify init

Command: `specify init . --integration claude` (run in a clean `/tmp/kaup-bridge-drill` after `rm -rf .specify .claude`)

Exit code: 0 (non-interactive; did not block)

Scaffold log (verbatim):
```
Specify Project Setup
  Project         kaup-bridge-drill
  Working Path    /private/tmp/kaup-bridge-drill

Selected coding agent integration: claude
Selected script type: sh
Initialize Specify Project
├── ● Check required tools (ok)
├── ● Select coding agent integration (claude)
├── ● Select script type (sh)
├── ● Install integration (Claude Code)
├── ● Install shared infrastructure (scripts (sh) + templates)
├── ● Ensure scripts executable (5 updated)
├── ● Install bundled workflow (speckit installed)
└── ● Finalize (project ready)

Project ready.
```

Next-steps panel enumerated the default `/speckit-*` skills (constitution, specify, plan, tasks, implement, converge) — baseline skills are present before extension install.

## Step 2 extension add

Command: `specify extension add --dev ~/workspace/kaup/kaup-bridge`

Exit code: 0

Output (verbatim):
```
✓ Extension installed successfully!

Kaup Superpowers Bridge (v1.0.0)
  Bridge replacing /speckit-implement with Superpowers SDD execution of tasks.md

Provided commands:
  • speckit.kaup-bridge.execute - Execute specs/<feature-id>/tasks.md via
Superpowers SDD (replaces /speckit-implement)

⚠  Configuration may be required
   Check: .specify/extensions/kaup-bridge/
```

`specify extension list` (exit 0), kaup-bridge lines:
```
  ✓ Kaup Superpowers Bridge (v1.0.0)
     kaup-bridge
     Bridge replacing /speckit-implement with Superpowers SDD execution of
tasks.md
     Commands: 1 | Hooks: 0 | Priority: 10 | Status: Enabled
```

## Step 3 render

SKILL.md path: `/tmp/kaup-bridge-drill/.claude/skills/speckit-kaup-bridge-execute/SKILL.md`

Symlink (default for `--dev` installs):
```
SKILL.md -> ../../../.specify/extensions/kaup-bridge/.specify-dev/agent-commands/claude/speckit-kaup-bridge-execute/SKILL.md
```
Resolved target (realpath) exists:
```
/private/tmp/kaup-bridge-drill/.specify/extensions/kaup-bridge/.specify-dev/agent-commands/claude/speckit-kaup-bridge-execute/SKILL.md  (19592 bytes)
```

SKILL.md head (first 6 lines, verbatim):
```
---
name: speckit-kaup-bridge-execute
description: Execute specs/<feature-id>/tasks.md via Superpowers SDD (replaces /speckit-implement)
compatibility: Requires spec-kit project structure with .specify/ directory
metadata:
  author: github-spec-kit
```

`superpowers:` references: 6 (>=4 required). Lines:
- L80: `superpowers:using-git-worktrees`
- L91: `superpowers:subagent-driven-development`
- L95: `superpowers:test-driven-development`
- L211: `superpowers:requesting-code-review`
- L265: `superpowers:finishing-a-development-branch`
- L329: `superpowers:finishing-a-development-branch`

Kaup project-assumption grep (`kaup-core|kaup-frontend|kaup-design|Closing-wave|001-sequence`):
0 matches (grep exit 1 = clean). The execute SKILL.md is fully de-kaup'd — no project-specific paths, wave names, or example feature IDs leaked into the rendered skill.

## Step 4 constitution-template

Source file (shipped in the extension repo, not an install artifact):
```
/Users/linyang/workspace/kaup/kaup-bridge/constitution-template.md  (5481 bytes, 120 lines)
```

`<PROJECT:` slot count: 7 (>=4 required). Slots at lines 8, 22, 23, 60, 65, 94, 120 — cover module-unit mapping, primary CLI surface, repo layout, SSOT split rule, integration-test focus, and version/ratify/amended dates.

Article structure (9 articles, I–IX, matches the design recorded in kaup-design stage-2a memory):
```
### I. Library-First (Module-Mapped)
### II. CLI Interface
### III. Test-First (NON-NEGOTIABLE)
### IV. Evidence-Driven, No Fake Completion (NON-NEGOTIABLE)
### V. Per-Task STOP for User Approval
### VI. Cross-Repository SSOT Discipline
### VII. Simplicity (YAGNI)
### VIII. Anti-Abstraction
### IX. Integration-First Testing
```

`specify integration status` (exit 0):
```
Integration status: OK
Default integration: claude
Installed integrations: claude
Multi-install safe: yes
Shared templates target alignment: claude
Modified managed files: 0
Missing managed files: 0
Invalid manifest paths: 0
Unchecked manifests: 0
```

## Verdict

| Check | Result |
|-------|--------|
| install (specify init + extension add --dev) | PASS |
| render (SKILL.md appears under .claude/skills, symlink resolves) | PASS |
| execute SKILL.md de-kaup'd (0 project assumptions, 6 superpowers refs) | PASS |
| constitution template (7 slots, 9 articles, 120 lines) | PASS |

No-regression note: this is the first end-to-end install verification of the
extracted `kaup-bridge` repo under a clean spec-kit 0.15 project. All four
verdicts pass. The `extension.yml` manifest is consumed correctly
(schema_version 1.0, requires.speckit_version ">=0.15.0" satisfied), the
`speckit.kaup-bridge.execute` command renders to a Claude Code skill at the
expected path, the execute body retains all 6 `superpowers:*` skill references
needed for SDD execution, and the constitution template ships intact with 7
`<PROJECT:` slots and 9 articles. No source files in `kaup-bridge` were
modified during this dry-run; only this evidence document was added.

Observed, non-blocking detail: the rendered SKILL.md `metadata.author` reads
`github-spec-kit` (a spec-kit CLI default) rather than the `author: kaup-ai`
declared in `extension.yml`. This is a spec-kit render convention for the
managed skill frontmatter and does not affect command discovery or execution;
flagging for awareness only.
