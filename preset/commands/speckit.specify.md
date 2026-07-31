---
description: >-
  Wrap of /speckit-specify. Bridges the hybrid workflow brainstorm step
  to specify by auto-reusing the brainstorm-preallocated feature-id when
  exactly one specs/<id>/brainstorm.md exists without a spec.md. Otherwise
  defers to the core specify command unchanged.
strategy: wrap
---

## Pre-Execution: brainstorm → specify feature-id bridging (applies only when the project uses date-slug feature-ids)

> **Configurable (D6).** This bridging applies ONLY when the project's
> feature-id policy is `date-slug`. If the project uses spec-kit's native
> sequential or timestamp numbering, SKIP this entire directive and defer
> to `{CORE_TEMPLATE}` unchanged.

Before running the core specify logic that follows (`{CORE_TEMPLATE}`),
determine whether a prior brainstorm step has already pre-allocated the
feature directory for this feature. The hybrid workflow's step 1
(brainstorm) creates `specs/<date-slug>/brainstorm.md`; this wrap makes
step 2 (specify) reuse that same `<date-slug>` id instead of letting
spec-kit auto-number it sequentially or by timestamp.

Perform this detection in order:

1. Scan the `specs/` directory for feature subdirectories whose
   `brainstorm.md` exists but whose `spec.md` does NOT yet exist. Each such
   directory represents a finished brainstorm awaiting specify (the
   brainstorm→specify handoff point).

2. Branch on how many matching directories you find:

   - **Exactly one matching directory** (`specs/<id>/` with `brainstorm.md`
     but no `spec.md`): Treat `SPECIFY_FEATURE_DIRECTORY` as already
     provided and set to `specs/<id>` (the matched directory). Do NOT
     generate a new short name and do NOT invoke spec-kit auto-numbering.
     When the core logic below reaches its "Create the spec feature
     directory" step, the resolution-order branch for an explicitly
     provided `SPECIFY_FEATURE_DIRECTORY` will fire and reuse `specs/<id>`
     as-is. This is the common single-user sequential case.

   - **Zero matching directories**: No brainstorm is pending specify.
     Proceed with the core specify logic unchanged — let spec-kit
     auto-generate the feature directory under `specs/` per the project's
     `feature_numbering` setting. Do not set `SPECIFY_FEATURE_DIRECTORY`.

   - **More than one matching directory**: Multiple brainstorms are
     simultaneously awaiting specify, so the id cannot be inferred
     unambiguously. STOP and warn the user that several
     `specs/<id>/brainstorm.md`-without-`spec.md` directories exist. List
     each candidate directory, then require the user to supply the
     feature-id explicitly by providing `SPECIFY_FEATURE_DIRECTORY` (for
     example `SPECIFY_FEATURE_DIRECTORY=specs/<the-chosen-id>`) before
     proceeding. Do not guess among the candidates.

This directive only selects the feature directory; it does not alter any
other specify behavior. After this directive resolves the directory (or
defers to the core), continue directly into the core specify command body
below.

{CORE_TEMPLATE}
