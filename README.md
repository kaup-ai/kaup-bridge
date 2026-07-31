# kaup-bridge

A [spec-kit](https://github.com/github/spec-kit) extension that bridges
spec-kit and [superpowers](https://github.com/obra/superpowers) — it replaces
spec-kit's built-in `/speckit-implement` with the superpowers Subagent-Driven
Development (SDD) workflow over `specs/<feature-id>/tasks.md`.

## What

`kaup-bridge` drives the superpowers SDD loop over a spec-kit task list:
it dispatches one fresh implementer subagent per task, runs a separate
reviewer per task, enforces per-task STOP for user approval, and hands
control to the user for `/speckit-converge`. An optional brainstorm→specify
feature-id bridge (date-slug) is provided via the preset.

## Prerequisites

- [spec-kit](https://github.com/github/spec-kit) `specify` CLI >= 0.15.0
  (`specify --version`)
- The coding agent's native
  [superpowers](https://github.com/obra/superpowers) installed — kaup-bridge's
  execute command *references* superpowers skills, it does not bundle them.

## Install

kaup-bridge is a spec-kit extension. Install it into your spec-kit project,
then activate your coding agent's integration:

```bash
specify extension add kaup-bridge --from https://github.com/kaup-ai/kaup-bridge.git
specify integration install claude   # or: codex / opencode
```

For local development / before the repo is pushed, install from a local path
instead:

```bash
specify extension add --dev /path/to/kaup-bridge
```

Then install superpowers for your agent — see the
[superpowers README](https://github.com/obra/superpowers) for the per-harness
command (Claude Code, Codex CLI, OpenCode, and others are all supported by
superpowers natively).

### Verify

```bash
specify extension list | grep kaup-bridge
specify integration status
```

### OpenCode note

OpenCode is not multi-install safe. If another integration is already active,
switch rather than installing alongside it:

```bash
specify integration switch opencode
```

### Multi-harness note

Extension commands render only to the integration active at `specify
extension add` time. If you add a new integration afterward, re-run
`specify extension upgrade` (or `specify extension add` again) to render
commands to the new integration.

## Usage

```text
/speckit-kaup-bridge-execute <feature-id>
```

`<feature-id>` is the directory under `specs/` holding `tasks.md` and
`plan.md`. The command reads `.specify/memory/constitution.md` Articles
IV / V / VI on every run (single SSOT — not inlined).

## Constitution

kaup-bridge ships a 9-article `constitution-template.md`. Initialize your
project's constitution from it, filling the `<PROJECT: ...>` slots. See
`examples/kaup/` for a filled reference (kaup's own instantiation).

## Architecture

See [docs/architecture.md](docs/architecture.md) for how kaup-bridge bridges
spec-kit and superpowers, and how three-platform adaptation works via
spec-kit's `specify integration` mechanism.

## License

Apache-2.0 — see [LICENSE](LICENSE).
