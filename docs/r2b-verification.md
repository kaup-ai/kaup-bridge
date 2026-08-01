# R2b Verification — execute 真触发 on Codex / OpenCode

> R2b is the only unverified item in the kaup-bridge extraction feature.
> T7 (Claude) + T8 (Codex/OpenCode) already proved install + render + skill-reference
> syntax compatibility (R2a). R2b verifies the LAST mile: actually triggering
> `/speckit-kaup-bridge-execute` in a live Codex / OpenCode session and confirming
> the referenced superpowers skills load and the SDD loop starts.
>
> A subagent cannot drive an interactive REPL, so this MUST be human-driven.

## Prerequisites (~5 min, shared)

```bash
# 1. Install kaup-bridge + activate the agent's integration
#    From GitHub (name + --from url; --from is an option, the name is the positional arg):
specify extension add kaup-bridge --from https://github.com/kaup-ai/kaup-bridge/archive/refs/heads/main.zip
#    Or from a local clone (--dev is a flag, the path is the positional arg, NO name):
#       specify extension add --dev /path/to/kaup-bridge
specify integration install codex          # or: opencode

# 2. Install NATIVE superpowers for that agent (kaup-bridge references, not bundles)
#    Codex:    Codex plugin marketplace — superpowers
#              (T8 confirmed ~/.codex/config.toml has [plugins."superpowers@..."] enabled = true)
#    OpenCode: opencode.json add {"plugin":["superpowers@git+https://github.com/obra/superpowers.git"]}

# 3. Minimal feature (to trigger execute without running a full SDD pass)
mkdir -p specs/000-r2b-smoke
# hand-write specs/000-r2b-smoke/tasks.md with 1 trivial task (e.g. "echo hello")
# + a placeholder specs/000-r2b-smoke/plan.md
```

⚠️ **T8 concern 2**: if you run `extension add` BEFORE `integration install`, the
extension's command does NOT render to the newly-added integration. If the skill
doesn't appear in step 2 below, re-run `specify extension add`, or switch agent
with `specify integration switch <agent>` (which re-renders — verified).

## Codex R2b

Open a Codex CLI session in the project directory.

1. **superpowers loaded?** Ask Codex: `describe your superpowers`
   → expect: brainstorming / subagent-driven-development / test-driven-development / etc.
2. **kaup-bridge execute registered?** Ask: `list your available skills/commands`
   → expect: `speckit-kaup-bridge-execute` in the list
3. **Trigger execute**: `/speckit-kaup-bridge-execute 000-r2b-smoke`
   (or Codex's native skill-invocation syntax)
4. **Observe**:
   - execute.md body loads (Codex reads the SDD SOP)
   - dispatches an implementer subagent for task 1
   - superpowers skills are actually invoked (subagent-driven-development / TDD)
   - **per-task STOP** (Article V — after task 1 it stops and waits for you, does not run on)
5. **Verdict**: all of step 4 observed → **PASS**; skill unknown / syntax error / SDD doesn't start → **FAIL** (record the verbatim error)

## OpenCode R2b

Open an OpenCode session in the project directory.

1. **superpowers loaded?** `use skill tool to list skills`
   → expect: superpowers skills present
2. **kaup-bridge registered?** `use skill tool to list`
   → expect: `speckit-kaup-bridge-execute`
3. **Trigger execute**: `use skill tool to load speckit-kaup-bridge-execute` (per superpowers' `.opencode/INSTALL.md` invocation style), feature-id `000-r2b-smoke`
4. **Observe + Verdict**: same as Codex steps 4-5

## FAIL triage (most likely culprits)

| Symptom | Check |
|---|---|
| skill not in list | `specify extension list \| grep kaup-bridge` + `specify integration status`; to change agent use `specify integration switch <new>` (uninstalls current + installs new + re-renders extension, verified) |
| superpowers not loaded | confirm the agent has superpowers installed (Codex `~/.codex/config.toml` / OpenCode `opencode.json` plugin entry) |
| `superpowers:xxx` reference syntax error | T8 R2a proved the canonical style compatible, but a live trigger may expose an edge case → record the Codex/OpenCode verbatim error and report back; execute.md's skill-reference syntax may need a per-harness adaptation |

## Result record (fill in below after running)

Record per-agent PASS/FAIL + observations/errors right here in this file
(you don't need dry-run-codex-opencode.md — that's the T8 historical evidence;
R2b result stays here):

### Codex R2b — <date>
<!-- PASS/FAIL + what you observed: superpowers loaded? execute triggered? SDD started? per-task STOP? verbatim errors if any -->

### OpenCode R2b — <date>
<!-- same -->

### Verdict
- **Both agents PASS** → R2b resolved. kaup-bridge is genuinely three-platform
  equivalent (including live trigger). The extraction feature is 100% closed.
- **FAIL** → report back; likely a new task to adapt execute.md's skill-reference
  syntax for non-Claude harnesses.
