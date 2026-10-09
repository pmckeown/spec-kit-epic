---
description: "Close an epic: review the whole epic, walk the acceptance and milestones, run the project's checks on the target after the fixes, settle external edges, and bring the register to ready or closed"
---

# Close an Epic

## User Input

```text
$ARGUMENTS
```

The user input is optional: the epic slug, if more than one directory exists
under `epics/`, and any change to the step order or the steps to skip for this
run (for example "acceptance first, skip milestones").

## What this command is for

The last feature being integrated is not the epic being done. Each feature was
proved against its own gate; nothing yet has proved the whole: the
features checked against each other, the acceptance journey walked across all of
them, the project's checks run on the target with every fix in, and the work
outside the epic that has to land in the right order around it.

**It does not push, and it does not merge onward.** Both are the operator's
decisions. This command brings the epic to `ready` (everything done that does
not need the operator) or `closed` (nothing left at all), and says which.

The file shapes are in `.specify/extensions/epic/FORMATS.md`; the step order and
reviewer settings are in `.specify/extensions/epic/epic-config.yml` if it
exists.

## Refuse before you start

Check every item below and report each one that holds, then stop. Where an item
names a way to continue, follow it instead.

1. **A feature is not `integrated`.** List each, with its status. A feature the
   operator has decided not to build is not a reason to wait: record it in the
   ledger as a `decision` (what was dropped, why, and what the epic no longer
   claims as a result), remove it from its milestone, and continue.
2. **A feature reached `integrated` without meeting the integration
   precondition.** Its `status_note` needs `Gate observed` (or `GATE NOT
   OBSERVED` with who accepted it) **and** an `Epic review` line with `0 open`,
   or with every open finding recorded as `OWED:` to the owner or `ACCEPTED:`
   (markers in `FORMATS.md`). A feature with no `Epic review` line is reviewed in
   full in the `review` step rather than refused; one with open findings and
   neither marker is refused. If the register predates the markers or
   the observation was written in some other form, **back-fill it from the
   evidence** (commit messages, run records, an earlier note) rather than
   re-observing, and mark it `(back-filled)`. Only where no evidence exists, ask
   for it to be observed now or for `GATE NOT OBSERVED` with who accepted that.
3. **A ledger entry names anything in `owes:`.** A feature or a section of
   `epic.md` still states a superseded ruling. Bring it current first. An entry
   with no `owes:` line owes nothing.
4. **Uncommitted changes outside `epics/` and `specs/`.** This command commits.

## The steps, and their order

Close is a set of steps, not a fixed sequence. **The order is the operator's
choice**, because the steps differ a great deal in cost: an independent
whole-epic review with re-attack rounds can cost many times what an acceptance
walk does. Take the order from the user input, else from `close.steps` in the
config, else this default:

| Step | What it does |
|---|---|
| `review` | Cross-feature review of the whole epic, its fixes, and re-attack rounds |
| `acceptance` | The acceptance walk, its fixes, and a re-walk |
| `milestones` | Each milestone's `means:` walked on `target:` |
| `checks` | The project's own full checks, on `target:`, after every fix above |
| `external` | Work outside the epic and external edges |
| `unpushed` | What exists only locally, and the onward merge |

Two orders work in practice. The default is the **economical** one,
`[acceptance, review, milestones, checks, external, unpushed]`: the acceptance
walk is the cheap step, and a person going through the whole journey finds the
defects that exist only where features meet. `review` then runs on the walk's
fixes as well as the features. The **thorough** order puts `review` first, so
the walk runs on code already reviewed across features; it costs more and finds
the same kinds of defect later. Say which you are running, and why.

**A step may be skipped, never silently.** Record each skipped step in the
closing note as `SKIPPED: <step>, <why>, by <who>`. `checks` runs after the last
step that changes code, whatever the order: the fixes are exactly what the
project's checks must cover.

Set the epic's `status: closing` and commit it before starting.

## Steps

### `review`

Run `__SPECKIT_COMMAND_EPIC_REVIEW__ all`. It covers what per-feature reviews
cannot see (epic changes not propagated, refusals reintroduced, shared
definitions, cross-feature fixes), attacks only guards changed since each
feature's own review, reviews in full any feature that has no review, routes
every finding (fix, owner's call or accept), and re-attacks every fix.
Fixes land on a branch cut from `target:` and reach it under the operator's
merge rule, and each is checked against the ledger first: a fix that reverses
an owner's decision is an owner's call, not a fix. Continue when no finding
routed **fix** is open; owner's calls and acceptances carry into the closing
note.

### `acceptance`

Walk the epic's Acceptance section end to end, on `target:`, by one person or
agent. It is not the union of the milestone walks; it is the whole journey,
including the parts that only appear once everything is in place.

- **Write the walk's notes as the walk goes**, step by step, so an interrupted
  walk can resume where it stopped rather than start again.
- **Nobody edits the tree during a walk.** Commit fixes and evidence between
  walks, not during one. If the project walks with a tool, follow that tool's
  own rules about what a single run may span.
- **A step that fails because the acceptance text is stale** (it describes a
  design the features decided against) is a finding against `epic.md`, not the
  product: correct the text, with a ledger entry, and say so.
- **A walk failure whose obvious fix reverses a ledger entry** is routed the
  same way as in review: an owner's call if the owner decided it, and a
  superseding entry, never a quiet edit, if the reversal is agreed.
- **Fix, then re-walk.** A failure that a fix resolves can fail again for a
  second, deeper cause. Walk the failed steps again after the fixes, and the
  whole walk again if a fix touched shared definitions.

### `milestones`

Walk each milestone's `means:` on `target:`, by whatever means the project has,
and record the result per milestone. A claim that does not hold is fixed, or
weakened with the weaker phrase recorded.

### `checks`

Run the project's full checks on `target:`, with every fix integrated. Features
that each passed alone can fail together, and the fixes made during close are
the least-tested code in the epic.

### `external`

Read `external:` in the register.

- **`work:`**: confirm each item is still meant to be on `target:`, and that the
  review covered it.
- **`edges:`**: check each `because` still holds and say what it now requires.
  An `after` edge is now due to its owner; a `before` edge that has not landed
  stops the close.
- **Anything this epic and a sibling both place in an order**, by number or by
  time (sequence numbers, identifiers, timestamps, version numbers): check it
  against every external edge's branch. A collision is obvious; an item that
  sorts on the wrong side of a sibling's is not, and can be worse. Name each,
  and who has to give way.

### `unpushed`

List anything that exists only locally: unpushed commits on `target:` or a
feature or fix branch. **Do not push.** Record each as `OWED: push <branch>,
the operator's decision`.

If `target:` is not where the work finally lives (it is a release branch that
merges onward), record `OWED: onward merge of <target> into <branch>, the
operator's decision`. The onward merge is always an `OWED:` line, never a step
this command takes.

### Bring the register to `ready` or `closed`

Write the epic-level `status_note`: each step's result in the order run, every
`SKIPPED:`, every `ACCEPTED:`, every owner's call as `OWED:`, the external
edges now due, and the reviewers' attack rounds in summary.

- Anything `OWED:` to the operator or owner (a push, the onward merge, a
  product call) -> `status: ready`. This is the normal outcome, because this
  command never pushes or merges.
- Nothing owed -> `status: closed`.

From `ready`, the operator reaches `closed` by hand: push, merge onward, settle
the owner's calls, then set `status: closed` and remove the `OWED:` lines that
are now done.

Commit only the register.

## Self-check

- [ ] Every feature is `integrated`, or recorded as dropped in the ledger
- [ ] Every integrated feature has a gate observation, back-filled from evidence where needed
- [ ] Nothing is owed in any ledger `owes:`
- [ ] The step order is stated, with why, and every skipped step is recorded
- [ ] Every fix was re-attacked or re-walked
- [ ] No fix reversed an owner's decision without the owner
- [ ] `checks` ran after the last step that changed code
- [ ] Every external edge was checked, including order by time, not only by number
- [ ] Nothing was pushed or merged onward by this command
- [ ] Status is `ready` if anything is owed to the operator or owner, else `closed`

## Report

- `ready` or `closed`, and why. If `ready`, exactly what the operator owes.
- The order run, each step's result, and anything skipped.
- Owner's calls, together, so they can be taken in one sitting.
- Acceptances, with their reasons.
- External edges now due, and to whom.
- **Next**, per "What comes next" in `FORMATS.md`.
