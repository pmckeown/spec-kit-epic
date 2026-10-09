# Changelog

## 0.1.4

- `FORMATS.md` has seeded-pointer shapes for every entry type: `decision`,
  `discovery` and `open` join `refusal` and `deferral`.
- New `status_note` marker `CUT WITH OPEN DECISIONS`, written when the owner
  cuts past refusal 10; `review` treats each id still `open` as an owner's
  call.
- `cut` numbers a sequential feature after the highest `specs/NNN-*` on any
  branch, not only the target, so parallel cuts do not collide.
- `cut` creates a missing target from the branch that holds the epic, which
  must be the current branch.
- `cut` says that `spec.md` and core `specify`'s other output, such as
  `checklists/`, stay uncommitted for the human's review.
- `cut` tells core `specify` to keep `EF` and decision citations but leave the
  gate's mechanism to core `plan`.
- After a cut with open decisions, core `specify`'s clarification questions
  are answered by pointing to the `open` entries, not settled in the spec.

## 0.1.3

- Supports Spec Kit 1.x: `speckit_version` is now `>=0.12.9.dev0,<1.2.0`,
  tested on 0.12.9 and 1.1.2.
- Every command checks all its refusals and reports each one that holds, rather
  than stopping at the first. A refusal that names a way to continue is
  followed instead.
- `cut` refusal 7 allows a gate that grounds its own premise: an
  `[UNGROUNDED]` mark settled by the feature's own gate, when the gate states
  the claim as something it proves.
- The README's configuration note covers Spec Kit 1.x, which creates
  `epic-config.yml` on install.
- Shorter manifest description, for the catalogue.

## 0.1.2

- "What comes next" in `FORMATS.md` no longer names `cut` for a feature an
  `open` ledger entry touches, which `cut` refuses since 0.1.1. A new row names
  the owner's decision as the next step.

## 0.1.1

- `cut` refuses a feature while an `open` ledger entry touches it (refusal 10),
  unless the owner chooses to cut anyway, recorded in the feature's
  `status_note`. A `provisional` entry is reported, not refused.
- `FORMATS.md` defines an `open` entry: options and a recommendation instead of
  a rejected alternative, answered by a new entry that supersedes it.
- `plan` rewrites `f:<slug>` tokens in prose as well as in the `Features` column
  and `touches:` lines, and removes duplicate ids left by merged slugs.
- `specify` checks that every "not yet" Out item has a matching `deferral`
  entry.

## 0.1.0

First public release.

- `speckit.epic.specify`, `speckit.epic.plan`, `speckit.epic.cut`,
  `speckit.epic.review`, `speckit.epic.close`.
- `specify` names dependants by provisional slug (`f:<slug>`); `plan` assigns
  `F<n>` ids and rewrites the slugs mechanically, recording each as `slug:`.
- `specify` writes a four-key stub `epic.yml` holding `decision_prefix`, so the
  prefix is declared before any decision id uses it, and refuses a prefix
  another epic already uses.
- The integration target (`target:`) is the operator's choice, asked for by
  `plan`; `cut` branches from it, and creates it if it does not exist yet,
  saying so in its report.
- `cut` runs core `specify` first, then seeds `decisions.md` into the directory
  core `specify` actually created and records that in the register.
- File shapes live in `FORMATS.md`, which the commands read; the README holds
  the reasoning.
- `review` checks a feature against the epic rather than its own spec: drift
  since cut, refusals reintroduced, restated facts, shared definitions touched
  by several features, fixes to other features recorded as discoveries, and
  guards attacked by a reviewer independent of the author.
- `close` reviews the whole epic, walks each milestone and the acceptance, runs
  the project's checks on the target, settles external edges, and closes the
  register. It does not merge onward.
- Verifying, sign-off, integrating and status are documented conventions; the
  `status_note` markers they write are read by `cut` and `close`.
- `cut` refuses an integrated upstream with no gate observation.
- Superseding a decision or fact lists the features owing a pointer update in
  `owes:`; `close` refuses while any is owed.
- `external:` in the register records work on the target outside the epic, and
  ordering edges to other branches or epics.
- One "What comes next" table that every command's report reads.
- `review` checks first that changes to the epic since a feature's cut reached
  it; `owes:` covers any such change, naming feature specs, pointers and
  `epic.md` sections. Fans out to independent reviewers for `all`, re-attacks
  every fix, and routes each finding as fix, owner's call, or accept.
- `close` runs its steps in an operator-chosen order (default: acceptance walk
  first), records every skip, runs the project's checks after the last fix,
  never pushes or merges onward, and normally ends at `ready`; `closed` is set
  by hand once the operator's `OWED:` lines are done.
- `review all` covers only what per-feature reviews cannot see, attacks only
  guards changed since each feature's review, and reviews in full any feature
  that was never reviewed.
- `cut` recognises a feature already cut on another branch, and refuses an
  integrated upstream without both a gate observation and an epic review.
- Optional `epic-config.yml`: reviewer models per role, fan-out, re-attack cap,
  acceptance threshold, close step order.
- `ACCEPTED:`, `SKIPPED:` and back-filled `Gate observed` markers.
- Ledger entries carry `by:`; a review or close fix that contradicts an
  owner's entry is routed to the owner, and an agreed reversal supersedes the
  old entry formally, including setting its status.
- Worked example at the post-plan state in `examples/report-export/`.
