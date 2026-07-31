# kaup-bridge architecture

## What kaup-bridge bridges

kaup-bridge is the bridge between two workflows with different philosophies:

| | spec-kit | superpowers |
|---|---|---|
| spec is | executable source (code serves spec) | design SSOT (human-readable) |
| strength | contract definition (specify/plan/tasks) + gates (constitution/checklist/analyze/converge) | decision traceability (brainstorm) + execution discipline (SDD: fresh subagent / TDD / per-task STOP / whole-branch review) |

The hybrid workflow uses spec-kit's 9-step production path as the skeleton,
injecting superpowers at head and tail:

- Step 1: superpowers brainstorming (lock decisions) -> `brainstorm.md`
- Steps 2-7: spec-kit contract definition (specify/clarify/plan/checklist/tasks/analyze)
- Step 8: superpowers SDD replaces `/speckit-implement` (this is kaup-bridge's execute command)
- Step 9: spec-kit converge acceptance reconciliation

## Two bridge points

- **Bridge A (brainstorm -> specify):** kaup-bridge's specify wrap (preset)
  reuses a pre-allocated date-slug feature-id when the project opts into
  date-slug ids (configurable, D6).
- **Bridge B (execute -> converge):** kaup-bridge's execute command reads
  spec-kit's `tasks.md` / `plan.md` / `constitution.md` and drives the
  superpowers SDD loop, then hands off to converge.

## Three-platform adaptation

kaup-bridge does NOT carry per-harness plugin directories. Multi-platform
adaptation is delegated to spec-kit's `specify integration` mechanism
(spec-kit 0.15.0 supports 37+ coding agents):

| layer | command | role |
|---|---|---|
| extension | `specify extension add <name> --from <zip-url>` | registers the execute command source |
| integration | `specify integration install claude\|codex\|opencode` | activates the target agent, renders commands to that agent's command surface |

Both ends are native tools, installed separately per harness:

- spec-kit `specify` CLI (Python, cross-platform) — contract generation + gates + acceptance
- superpowers (multi-harness skill set) — the SDD/brainstorm skills that kaup-bridge's execute command references

For the full design (decisions D1-D9, risks R1-R3, evidence), see the design
spec in the kaup-design repo:
`docs/superpowers/specs/2026-07-31-kaup-bridge-extraction-design.md`.
