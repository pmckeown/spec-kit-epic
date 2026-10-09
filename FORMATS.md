# spec-kit-epic: file formats

The shapes the commands write and read. The reasoning behind each rule is in
[README.md](README.md); this file is only what a command needs to get the shape
right.

All files live under `epics/<slug>/`, beside `specs/`, carrying no number.

## `epic.md`: the epic spec

Written by `epic.specify`. Sections, in order:

| Section | Holds | Refusal that guards it |
|---|---|---|
| Why this epic | The claim being made, and what currently prevents making it | — |
| What exists today | Re-grounded findings, each with `file:line` or an equivalent citation | An ungrounded claim is marked `[UNGROUNDED: <what would settle it>]`, never smoothed |
| What is now wrong | Where the source material is contradicted by the code | — |
| Boundary | In, and **Out with a reason per item**, then "the line" | An Out item with no reason is refused |
| Facts that span features | The spanning-facts table | — |
| Decisions | Optional summary: id, type and one line each; the reasoning lives only in the ledger | A decision with no rejected alternative is refused |
| Acceptance | The walk no feature owns | Refused if it restates a feature's gate |

### The spanning-facts table

```
| Id  | Fact                                                      | Features   |
|-----|-----------------------------------------------------------|------------|
| EF1 | <the single fact, stated once>                            | F1–F4      |
| EF2 | <a mechanism several features must implement identically> | F1, F3     |
| EF3 | <a constraint imposed from outside the epic>              | F2, F4     |
```

- The `Features` column is the **only** record of the feature↔fact edge.
- Before `epic.plan`, dependants are provisional slugs: `f:<slug>` (`f:import`,
  `f:audit-trail`). After it, they are `F<n>`, and no `f:` token remains.
- A **correction** keeps the id; edit the row. A **change of substance** gets a
  new id, and the old row stays, marked `superseded by EF<n>`. Either way, add a
  ledger entry.
- A fact with one dependant is reported by `epic.plan`, not refused: it may gain
  a second when a feature is split, or it may belong in that feature's `spec.md`.

## `decisions.md`: the epic ledger

Append-only. One entry per decision:

```
### D-<PREFIX>-007 · <date> · refusal · decided
by:         <who took it: the owner, the operator, or the agent>
refs:       <where in epic.md this was settled>
proven:     <the evidence, or a pointer to it>
touches:    F1, F2, F4
supersedes: —
owes:       —

<prose: what was chosen, what was rejected, why, and what follows>
```

- `type`: `decision` · `discovery` · `refusal` · `deferral`
- `status`: `decided` · `provisional` · `open` · `superseded-by D-<PREFIX>-nnn`
- `<PREFIX>` is `decision_prefix` from `epic.yml`, verbatim. It is unique across
  `epics/*/epic.yml`.
- A `refusal` names the proof that killed the alternative. A `deferral` names the
  condition that would revive it.
- An `open` entry is a question still waiting for a decision. Its prose lists
  the options with a recommendation, and `by:` names who must decide; it has no
  rejected alternative yet. The answer is a new entry that supersedes it and
  carries the chosen option and the rejected alternative like any other
  decision. `epic.cut` refuses a feature that an `open` entry `touches:`.
- `touches:` is the **only** record of the feature↔decision edge. `f:<slug>`
  before plan, `F<n>` after. A decision touching one feature is reported by
  `epic.plan`, not refused.
- **`owes:` is how a change to the epic reaches work already cut.** Any entry
  that changes the epic after features are cut (superseding a decision,
  superseding or correcting a spanning fact, tightening a gate, rewriting an
  acceptance step) lists in `owes:` everything that still states the old
  version:
  - each feature past `planned` that the change touches, meaning its `spec.md`
    and plan prose as well as its seeded `decisions.md` pointers;
  - each section of `epic.md` that restates it, by name (`epic.md § Acceptance`).

  Bring each one current (for a pointer, add a line under it:
  `SUPERSEDED by D-<PREFIX>-nnn: <one line>`), then remove it from `owes:`.
  `—`, or no `owes:` line at all (entries written before the field existed),
  means nothing is owed. `close` refuses while anything is owed.
- **`by:` says whose decision it is**, and so who may reverse it. An entry
  whose `by:` names the owner can be reversed only by the owner. An entry with
  no `by:` line (written before the field existed) is treated as the owner's
  unless its prose plainly says otherwise.
- **Reversing a decision is a new entry, never an edit.** The new entry names
  the old one in `supersedes:`, fills `owes:`, and the old entry's `status`
  becomes `superseded-by D-<PREFIX>-nnn`. All three, or the rest of the epic
  keeps citing a ruling that no longer holds.
- Three edits to an existing entry are permitted, and nothing else: plan's slug
  rewrite, which changes ids only; clearing names from `owes:`; and setting
  `status: superseded-by D-<PREFIX>-nnn` when a later entry supersedes it, so a
  reader of the old entry can tell it is dead.

## `epic.yml`: the register

`epic.specify` writes the first four keys as a stub; `epic.plan` completes the
file; feature status changes are made by hand (see the README's conventions).

```yaml
epic:   <slug>
title:  <title>
decision_prefix: <SHORT>        # D-<SHORT>-007; declared, never derived
status: in-flight               # shaping | in-flight | closing | ready | closed
target: <branch>                # the operator's integration target: main, release, release/<x>, ...

milestones:
  M1: { means: "<what is true from outside when this set lands>", features: [F1, F2] }

features:
  F1:
    title:     <short name>
    slug:      <the provisional slug from epic.specify>
    concern:   "<the one thing a person walking it says yes or no to>"
    milestone: M1
    status:    planned
    status_note: |              # optional; what the status does not say
      <free text>
    feature:   ~                # set by epic.cut: the directory core specify created
    branch:    ~                # set by epic.cut
    gate:      "<what must be observed for this feature to be believed>"

edges:
  - from: F1
    to:   F2
    because: >-
      <what breaks if this order is violated, not "F2 needs F1">

external:                       # optional; work outside the epic that touches it
  work:                         # landed on target: without being an epic feature
    - ref:  <branch, change request or feature directory>
      why:  "<why it is on target:, and which features it affects>"
  edges:                        # ordering against other branches or epics
    - with:      <branch, epic or feature outside this epic>
      relation:  before | after | alongside   # the other lands before this epic, after it, or is designed with it
      because: >-
        <what breaks if the relation is violated>
```

- The paths of `epic.md` and `decisions.md` are not recorded: they are always
  `epics/<epic>/epic.md` and `epics/<epic>/decisions.md`.
- No feature carries a `facts:` or `decisions:` list.
- `edges` is top-level, and every edge has a `because`. So does every
  `external.edges` entry.
- The epic itself may carry a `status_note`; `close` writes one.
- Epic `status: ready` means close found nothing left except what is owed to the
  operator or owner (a push, the onward merge, a product call). `closed` means
  nothing is owed, including the onward merge. `close` normally ends at
  `ready`; the operator sets `closed` by hand once the `OWED:` lines are done.

### Feature `status`

| Status | Set by | Means |
|---|---|---|
| `planned` | `epic.plan` | Decomposed, gated, not yet a feature |
| `cut` | `epic.cut` | An ordinary Spec Kit feature exists at `feature:` |
| `gated` | convention | The feature's `gate:` has been **observed**, by whatever means the project has |
| `integrated` | convention | Merged into `target:` |

A project may add its own states between `cut` and `gated`; nothing here reads
them.

### `status_note`

A block scalar, never a trailing `#` comment. Free text, except that **lines
starting with these markers are read by the commands**, so write them exactly:

| Marker | Written by | Read by |
|---|---|---|
| `Spec signed off <date> by <who>.` | convention | — |
| `BUILDING UNSIGNED: <who> decided <date> ...` | convention | — |
| `Gate observed <date>: <what was run, and the result>.` | convention | `cut` and `close` refuse an `integrated` feature without it |
| `Gate observed <date> (back-filled): <evidence it was taken from>.` | `close`, or by hand | as above; for registers written before the markers existed |
| `GATE NOT OBSERVED: <reason>, accepted by <who> <date>.` | convention | accepted by `cut` and `close` in place of the above |
| `Epic review <date>: <n> findings, <m> open, <r> attack rounds.` | `review` | `cut` and `close` refuse an `integrated` feature without it, unless `<m>` is 0 or every open finding has an `OWED:` (to the owner) or `ACCEPTED:` line; `review all` attacks only what changed after it |
| `PROVEN FALSIFIABLE: <how the gate was shown able to fail>.` | convention | — |
| `OWED: <what is still to do>, <whose it is>.` | anyone | `close` sets `ready` while any is owed to the operator or owner |
| `ACCEPTED: <finding>, <why it stays>, by <who> <date>.` | `review`, `close` | — ; done, not owed |
| `SKIPPED: <close step>, <why>, by <who>.` | `close` | — |

`OWED` means still to do. `ACCEPTED` means deliberately left as it is. A
reviewer may accept only findings at or below `review.accept_max_severity`; a
product call is always `OWED` to the owner, whatever its severity. Markers are
read at the start of a line only; a register written before them is back-filled
from the evidence, not re-observed.

**This list is meant to stay short.** `status_note` is free text for a person
that happens to carry a few parsed prefixes. If another marker is ever needed,
move the parsed ones to structured keys on the feature (`gate_observed:`,
`epic_review:`) and leave `status_note` as free text, rather than adding
another prefix.

```yaml
    status: integrated
    status_note: |
      Spec signed off <date> by <who>.
      Gate observed <date>: <what was run, and the result>.
      PROVEN FALSIFIABLE: <how it was shown able to fail>.
      Epic review <date>: 3 findings, 0 open.
      Integrated <date>, merge <sha>.
      OWED: <what the gate did not reach, and whose it is>.
```

An override of spec sign-off is recorded in the register, not in the feature's
spec:

```yaml
    status_note: |
      BUILDING UNSIGNED: <who> decided <date> to proceed without spec
      sign-off, because <reason>. Sign-off owed before integration.
```

## What comes next

Every command ends its report with the step the register now implies. One table,
so the commands agree:

| The register shows | Next |
|---|---|
| A stub `epic.yml` (no `features:`) | `speckit.epic.plan` |
| A feature `cut`, spec not signed off | a human signs off the spec |
| A feature `cut`, spec signed off or building unsigned | core `plan`, `tasks`, `implement` |
| A feature implemented, no `Gate observed` | observe the gate (convention) |
| A feature with `Gate observed`, no `Epic review` | `speckit.epic.review` |
| Review findings open, routed fix | fix (on the feature's branch, or at close a branch from `target:`), then re-attack |
| A feature reviewed with `0 open` | integrate it (convention) |
| Anything in a ledger `owes:` | bring the named spec, pointer or `epic.md` section current |
| A feature `planned` with every incoming edge `integrated`, and no `open` ledger entry touching it | `speckit.epic.cut` |
| A feature `planned` with an `open` ledger entry touching it | the owner decides each `open` entry, as a new entry that supersedes it |
| Every feature `integrated` | `speckit.epic.close` |
| Acceptance walk failed | fix, then re-walk the failed steps |
| The epic `ready` | the operator: each `OWED:` in the epic's `status_note` (push, onward merge, owner's calls), then `status: closed` by hand |
| The epic `closed` | nothing |

When several rows hold, name each.

## `specs/<feature>/decisions.md`: the seeded pointers

Written by `epic.cut` into the directory core `specify` created:

```
## Promoted from the epic: do not re-derive

### D-<PREFIX>-007 · refusal · epic ledger
<The alternative, named in one line.> REFUSED: <the one-line reason, including
what makes it look right.>
Full entry and proof: epics/<slug>/decisions.md

### D-<PREFIX>-009 · deferral · epic ledger
<The alternative, named in one line.> DEFERRED UNTIL: <the reviving condition.>
Full entry: epics/<slug>/decisions.md
```

One line plus the pointer, never the argument.
