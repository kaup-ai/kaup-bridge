# Dry-run: Codex + OpenCode R1/R2/R3 evidence (T8)

Date: 2026-07-31
Spec-kit CLI: specify 0.15.0
Extension source: ~/workspace/kaup/kaup-bridge (installed via `--dev`)
Throwaway projects: `/tmp/kaup-bridge-codex`, `/tmp/kaup-bridge-opencode` (not committed)
Harness CLIs present: codex-cli 0.146.0, opencode 1.18.10

This run gathers the FIRST empirical evidence for the three risks in spec §2 that
the T7 claude case could not cover. R2b (live execute trigger) is explicitly out
of subagent scope and marked follow-up; everything else was exercised for real.

## Catalog facts (baseline, `specify integration list`)

| Key | Multi-install Safe | CLI Required |
|-----|--------------------|--------------|
| codex | yes | yes |
| opencode | no | yes |
| claude (T7) | yes | yes |

## R1-Codex: PASS

Sequence (verbatim exits): `specify init . --integration codex` → exit 0;
`specify extension add --dev ~/workspace/kaup/kaup-bridge` → exit 0;
`specify integration install codex` → exit 0, "Integration 'codex' is already
installed" (init had already installed it; no-op).

Command surface for codex is `.agents/skills/<skill-id>/SKILL.md` with `$skill`
invocation (init next-steps panel showed `$speckit-constitution`,
`$speckit-specify`, ...). `specify integration status` (exit 0):

```
Integration status: OK
Default integration: codex
Installed integrations: codex
Multi-install safe: yes
Shared templates target alignment: codex
Modified managed files: 0
Missing managed files: 0
Invalid manifest paths: 0
Unchecked manifests: 0
```

kaup-bridge execute rendered at:
```
/tmp/kaup-bridge-codex/.agents/skills/speckit-kaup-bridge-execute/SKILL.md
```
Listed alongside the 10 bundled speckit-* skills in `.agents/skills/`. Render
verification:

- File type: regular file (19592 bytes). Unlike the claude `--dev` case (which
  symlinks into `.specify-dev/agent-commands/claude/`), the codex render is a
  flat COPY with no `.specify-dev/agent-commands/` tree generated. Editing the
  extension source after install will NOT propagate to codex without a re-render
  ( behavioural difference from claude's live symlink; flag for awareness).
- Frontmatter (verbatim, first 7 lines):
  ```
  ---
  name: speckit-kaup-bridge-execute
  description: Execute specs/<feature-id>/tasks.md via Superpowers SDD (replaces /speckit-implement)
  compatibility: Requires spec-kit project structure with .specify/ directory
  metadata:
    author: github-spec-kit
    source: kaup-bridge:commands/speckit.kaup-bridge.execute.md
  ---
  ```
- `superpowers:` references: 6 (L80 using-git-worktrees, L91
  subagent-driven-development, L95 test-driven-development, L211
  requesting-code-review, L265 finishing-a-development-branch, L329
  finishing-a-development-branch). Same six as the claude render in T7.
- Kaup-specific grep (`kaup-core|kaup-frontend|kaup-design|Closing-wave|001-sequence`):
  0 matches. De-kaup'd.
- `~/.codex/skills/` and project `.codex/skills/`: no kaup-bridge entries (codex
  discovers skills via the project `.agents/skills/` directory, not `~/.codex/`).

**Verdict: PASS.** The `speckit.kaup-bridge.execute` extension command renders
into the codex command surface at `.agents/skills/speckit-kaup-bridge-execute/`
with correct content (6 superpowers refs, 0 kaup-specific refs, 19592 bytes).
The `extension.yml` manifest is consumed identically to the claude case; only
the render target directory and invocation prefix differ (`.agents/skills/` +
`$` vs `.claude/skills/` + `/`).

## R1-OpenCode: PASS

Sequence: `specify init . --integration opencode` → exit 0;
`specify extension add --dev ~/workspace/kaup/kaup-bridge` → exit 0.

Command surface for opencode is `.opencode/commands/<cmd>.md` with `/cmd`
invocation (init next-steps panel showed `/speckit.constitution`,
`/speckit.specify`, ...). `specify integration status` (exit 0):

```
Integration status: OK
Default integration: opencode
Installed integrations: opencode
Multi-install safe: yes
Shared templates target alignment: opencode
Modified managed files: 0
Missing managed files: 0
Invalid manifest paths: 0
Unchecked manifests: 0
```

(Note: the "Multi-install safe: yes" field here reports on the single-installed
state; it flips to "no" once a second integration is added — see R3.)

kaup-bridge execute rendered at:
```
/tmp/kaup-bridge-opencode/.opencode/commands/speckit.kaup-bridge.execute.md
```
Render verification:

- File type: symlink, same `--dev` mechanism as claude:
  ```
  .opencode/commands/speckit.kaup-bridge.execute.md ->
    ../../.specify/extensions/kaup-bridge/.specify-dev/agent-commands/opencode/speckit.kaup-bridge.execute.md
  ```
  Target exists (19467 bytes). OpenCode DOES get a harness-specific
  `.specify-dev/agent-commands/opencode/` directory, paralleling claude's
  `claude/` — so edits to the extension source propagate live via the symlink,
  matching the claude `--dev` behaviour and contrasting with codex's flat copy.
- Frontmatter (verbatim, first 3 lines):
  ```
  ---
  description: Execute specs/<feature-id>/tasks.md via Superpowers SDD (replaces /speckit-implement)
  ---
  ```
  Simpler than codex's (no `name`/`metadata` block); the body carries
  `<!-- Extension: kaup-bridge -->` / `<!-- Config: ... -->` HTML comments.
- `superpowers:` references: 6 (L78, L89, L93, L209, L263, L327 — same six skills
  as claude/codex, line numbers shifted by the shorter frontmatter).
- Kaup-specific grep: 0 matches. De-kaup'd.

**Verdict: PASS.** The `speckit.kaup-bridge.execute` command renders into the
opencode command surface at `.opencode/commands/speckit.kaup-bridge.execute.md`
as a live symlink, with correct content (6 superpowers refs, 0 kaup-specific
refs). Harness-specific `.specify-dev/agent-commands/opencode/` is generated.

## R3-OpenCode Multi-install Safe = no: CONFIRMED (constraint is real and enforced)

Starting from the R1 opencode project (only opencode installed), attempted
`specify integration install claude`:

Exit code: 1. Verbatim error:
```
Error: Installed integrations: opencode.
Default integration: opencode.
Installing multiple integrations is only automatic when all involved
integrations are declared multi-install safe.
To replace the default integration, run specify integration switch claude.
To install 'claude' alongside the existing integrations anyway, retry the same
install command with --force.
```

Retried with `--force` (exit 0). Claude installed, but the project then enters
ERROR state. `specify integration status` after `--force`:
```
Integration status: ERROR
Default integration: opencode
Installed integrations: opencode, claude
Multi-install safe: no
Shared templates target alignment: opencode
Modified managed files: 0
Missing managed files: 0
Invalid manifest paths: 0
Unchecked manifests: 0

Findings:
- error unsafe-multi-install: Installed integrations are not all declared multi-install safe: opencode
```

Two escape hatches exist (`switch` to replace, `--force` to override) but the
safe path is blocked and `--force` leaves the project in an explicit ERROR
state with a named finding.

**Bonus finding (extension command propagation):** after force-installing
claude alongside opencode, the kaup-bridge execute command did NOT render into
`.claude/skills/` — only the pre-existing opencode render at
`.opencode/commands/` remained. Extension commands are generated for the
integration(s) active at `extension add` time; adding a new integration later
does not retroactively render extension commands for it. A user who needs the
kaup-bridge command on a second harness must re-run `specify extension add` (or
`specify extension upgrade`) after switching/forcing the new integration.

**Verdict: constraint confirmed.** OpenCode's `Multi-install Safe = no` is a
real, enforced property: co-installing claude or codex alongside opencode is
blocked without `--force`, and `--force` leaves the project in ERROR status.
The kaup-bridge extension itself needs no change (it renders correctly into a
single-harness opencode project); the constraint is a property of the opencode
integration catalog entry, not of kaup-bridge.

## R2a — superpowers skill-reference mechanism: COMPATIBLE

READ-ONLY research; no live session driven. The execute.md body references
six superpowers skills by the `superpowers:<skill-id>` text convention (e.g.
`superpowers:subagent-driven-development`). The question is whether codex and
opencode resolve that same convention.

Evidence that the convention is harness-agnostic:

1. **`superpowers:<id>` is superpowers' own canonical cross-skill reference
   style**, not something kaup-bridge invented. `using-superpowers/SKILL.md`
   uses it verbatim (L30-31):
   ```
   - "Let's build X" → superpowers:brainstorming first, then implementation skills.
   - "Fix this bug" → superpowers:systematic-debugging first, then domain skills.
   ```
   Every superpowers skill that names another skill uses this same prefix.

2. **Per-harness adapters register the skills with each harness's native
   discovery mechanism; none of them rewrite the reference syntax.**
   - Codex: `.codex-plugin/plugin.json` declares `"skills": "./skills/"` —
     codex discovers the skills directory directly. The codex adapter
     (`skills/using-superpowers/references/codex-tools.md`) only documents
     TOOL mapping (enable `[features] multi_agent = true` in `~/.codex/config.toml`
     so `spawn_agent`/`wait_agent`/`close_agent` exist for SDD); it does NOT
     change how skills are named or invoked.
   - OpenCode: `.opencode/plugins/superpowers.js` injects the skills directory
     into the live OpenCode config (`config.skills.paths.push(...)`) and emits
     an explicit tool-substitution table (L77-87):
     ```
     - Invoke a skill → OpenCode's native `skill` tool
     - `Subagent (general-purpose):` → `task` with subagent_type: "general"
     ```
     Again, the `superpowers:<id>` reference is resolved by the agent reading
     the text and loading the named skill via the native `skill` tool — the
     adapter does not rewrite the reference tokens.
   - `using-superpowers/SKILL.md` "Platform Adaptation" (L52-58) lists Codex /
     Pi / Antigravity as having harness-specific reference files; OpenCode is
     handled by its plugin shim rather than a references file.

3. **Superpowers is independently installed on each harness** (it is NOT
   bundled with kaup-bridge or spec-kit — kaup-bridge only REFERENCES
   superpowers skills). On this machine, codex has it registered:
   ```
   # ~/.codex/config.toml
   [plugins."superpowers@superpowers-dev"]
   enabled = true
   ```
   So when codex reads `superpowers:subagent-driven-development` inside the
   rendered kaup-bridge execute SKILL.md, the named skill is discoverable via
   codex's plugin-discovered skills directory. The opencode install path is
   the `opencode.json` `plugin` array per `.opencode/INSTALL.md`.

**Verdict: COMPATIBLE.** The `superpowers:<id>` tokens in execute.md match how
codex and opencode discover and invoke superpowers skills: each harness ships
an adapter that registers the skills natively, and the reference tokens
themselves are harness-agnostic text that the LLM resolves to the native
invocation (Skill tool on claude, `skill` tool on opencode, `$skill` /
plugin-discovered skill on codex). No syntax rewrite is needed for execute.md
to be portable.

**Documented dependency (not a FAIL, but a real precondition):** the
references only resolve if superpowers is independently installed on the
target harness. kaup-bridge's `extension.yml` does not declare superpowers as
a `requires` (there is no such field for cross-plugin skills in spec-kit
0.15). The constitution template and the execute body both assume superpowers
is present; a project that installs kaup-bridge without superpowers will get
a rendered command whose skill references are dead links at runtime. This is
a documentation/onboarding concern, not a code defect — flag for the README
or a follow-up note, not a T10+ code task.

## R2b — live execute trigger on codex/opencode: FOLLOW-UP (out of scope)

A subagent cannot drive an interactive Codex or OpenCode REPL, so the actual
end-to-end trigger of `speckit.kaup-bridge.execute` (agent reads the SKILL,
dispatches an implementer per `superpowers:subagent-driven-development`, loads
`superpowers:test-driven-development` for the red-green-refactor loop, etc.)
was NOT exercised in this run. R2a establishes that the mechanism is
compatible on paper; R2b is the empirical confirmation that a live session on
each harness actually resolves the references and completes one SDD task loop.

**Owner:** human-driven session in a real codex and opencode REPL. Suggested
acceptance check: in a project with both kaup-bridge and superpowers
installed, invoke the execute command against a trivial 1-task feature and
confirm the agent (a) loads `superpowers:subagent-driven-development`, (b)
dispatches a subagent that loads `superpowers:test-driven-development`, and
(c) returns control to the user at the per-task STOP. This is the same shape
as the claude dry-run that already passed in production use; R2a gives
high confidence it will pass on codex/opencode, but the live trigger remains
open evidence.

## T8 outcome

| Risk | Verdict | Evidence |
|------|---------|----------|
| R1-codex | PASS | `.agents/skills/speckit-kaup-bridge-execute/SKILL.md`, 19592 bytes, 6 superpowers refs, 0 kaup-specific |
| R1-opencode | PASS | `.opencode/commands/speckit.kaup-bridge.execute.md` → symlink into `.specify-dev/agent-commands/opencode/`, 6 superpowers refs, 0 kaup-specific |
| R3 (opencode multi-install) | CONFIRMED | `specify integration install claude` on an opencode project exits 1; `--force` leaves `Integration status: ERROR` with `unsafe-multi-install: opencode` finding |
| R2a (skill-ref mechanism) | COMPATIBLE | `superpowers:<id>` is superpowers' own canonical style; per-harness adapters register skills natively without rewriting the tokens; codex has superpowers plugin enabled |
| R2b (live execute trigger) | UNKNOWN → FOLLOW-UP | Out of subagent scope; requires human-driven codex/opencode REPL session |

**Does any FAIL trigger a new T10+ task? No.** No FAIL outcomes. R1 passes on
both new harnesses, R3 is confirmed but is a property of the opencode catalog
entry (not a kaup-bridge defect), and R2a is compatible. The only follow-up
is R2b (live session), which was always scoped as human-driven and does not
indicate a code problem — it is empirical confirmation of what R2a already
established mechanistically.

**Key non-blocking concerns to record:**

1. **Codex `--dev` render is a copy, not a symlink.** Unlike claude/opencode
   (which symlink into `.specify-dev/agent-commands/<harness>/`), the codex
   render is a flat copy at `.agents/skills/<id>/SKILL.md` with no
   `.specify-dev/agent-commands/codex/` generated. Extension authors iterating
   on the source must re-run the install to refresh the codex render. Not a
   bug — a spec-kit behavioural difference worth documenting.
2. **Extension commands do not retroactively render for integrations added
   after `extension add`.** Observed in the R3 `--force` path: claude was
   force-installed but kaup-bridge did not appear under `.claude/skills/`.
   Users adopting a second harness must re-add or upgrade the extension.
3. **superpowers is an undeclared runtime dependency.** kaup-bridge references
   six `superpowers:<id>` skills but does not (and, in spec-kit 0.15, cannot)
   declare superpowers as a `requires`. Projects that install kaup-bridge
   without superpowers get a rendered command with unresolved skill references.
   Recommend a README callout; no code change needed.

No source files in `kaup-bridge` were modified during this dry-run; only this
evidence document was added.
