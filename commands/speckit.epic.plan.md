---
description: "Decompose an epic into features with gates, dependency edges and milestones, and write the epic register"
---

# Plan an Epic

## User Input

```text
$ARGUMENTS
```

The user input is optional: the epic slug, if more than one directory exists
under `epics/`. With one, use it. With several and no slug, list them and ask.

## What this command is for

`epic.md` says what is true and what the epic owns. This command says **what gets
built, in what order, and how anyone will know each piece worked**. Its output is
the completed `epic.yml`, the register every later command reads.

Take every file shape from `.specify/extensions/epic/FORMATS.md`. Read the
epic's own `epic.md`, `decisions.md` and stub `epic.yml` in full: the
decomposition is constrained by them, especially by which concerns a promoted
decision `touches:`.

## Refuse before you start

1. **No `epic.md`.** Name `__SPECKIT_COMMAND_EPIC_SPECIFY__`.
2. **No `decision_prefix` in `epic.yml`.** The ledger's ids already cite one, so
   it was settled. Find it in `decisions.md`, ask the user to confirm it, and
   write it. Never invent a new one: changing it silently rewrites every citation.
3. **`epic.md` has `[UNGROUNDED]` marks that reach the decomposition.** An
   ungrounded claim about a gate or a guard will become a feature built on a false
   premise. List them and ask whether to ground them first or cut around them.
4. **The spanning-facts table is empty.** An epic with no fact spanning two
   features is a sequence of features that happen to be scheduled together, and
   the register would be pure overhead. Say so and name
   `__SPECKIT_COMMAND_SPECIFY__` for each of them, unless
   `__SPECKIT_COMMAND_EPIC_SPECIFY__` simply has not found the facts yet, in which
   case go back to it.
5. **`epic.yml` already has `features:`.** This is a re-plan, not a first plan
   (a stub with only `epic`, `title`, `decision_prefix` and `status` is a first
   plan). Say what is changing and carry forward every feature whose `status` is
   past `planned`: you may re-order and re-scope unbuilt work, never silently
   rewrite the history of work already done. Never renumber an existing `F<n>`.

## Steps

### 1. Cut the features

One feature, **one concern, walkable alone**. The test is not size, it is whether
a person can walk the finished thing and say yes or no.

Start from the provisional slugs (`f:<slug>`) used in `epic.md` and
`decisions.md`. They are the specify step's guess at the concerns, not a
commitment: merge, split, rename or drop them as the tests below require.

**The oversize test:** if a feature would need a numbering scheme inside its own
`tasks.md` to navigate, it is not a feature, it is an epic. Cut it again.

**The undersize test:** if two candidate features can only ever be walked
together, or one's gate is the other's gate, they are one feature. Merge them.
Pay particular attention to a feature that is a *mechanism* and a feature that is
its *alarm*: shipping the mechanism without the alarm is usually worse than
shipping neither, and splitting them is how the alarm gets dropped under schedule
pressure.

Name each feature by its concern, not by its layer. "The write block" is a
concern. "Storage changes" is a layer, and a layer-shaped feature cannot be
walked.

Number the features `F1`, `F2`, ... in intended build order.

### 2. Give every feature a gate

A gate is **what would have to be observed for this feature to be believed**, in
terms a person can check. Write it before the feature is built, because a gate
written afterwards is a description of what the code happens to do.

Three rules:

- **A gate must be able to fail.** "No email is delivered" is not a gate if the
  address is undeliverable by construction: it is true of completely broken
  containment. Ask what a *broken* implementation would do, and if the gate
  passes for it, the gate is wrong. The cheapest proof is a negative control:
  disable the thing under test and confirm the check goes red.
- **A gate for a guard is an attack, not a hand-test.** "Clicking Save refuses"
  is satisfied by a guard that is vacuous everywhere else. Name the shape of the
  attack.
- **A gate that proves a change must also prove the non-change.** Anything
  conditional on a flag needs both arms: the flagged subject blocked *and* the
  unflagged one unaffected, in every cell. A gate that only tests the new
  behaviour cannot see collateral damage.

Where the project cannot test something locally, say so **in the gate** and name
what is testable instead. A constraint discovered during implementation becomes
an excuse; a constraint written into the gate becomes a known limit.

### 3. Write the edges, each with its `because`

An edge is `{ from, to, because }`. The `because` says **what breaks if the order
is violated**, not "B needs A", which is what an arrow already says.

**An edge without a stated consequence is easy to drop when the schedule
tightens.** The consequence is the whole argument for the order, and it is
what someone re-reads when they are tempted to reorder. If you cannot state
it, there is no edge, and saying so is a useful finding.

Edges live at the top level of `epic.yml`, not as a list of ids on each feature,
precisely so that writing one without a `because` is awkward.

### 4. Set milestones

A milestone is **what is true when this set of features has landed**, in one
phrase, from outside. A milestone names a capability somebody outside the team
could notice; "phase 1" is a label, not a milestone.

Then test it: if the phrase overstates what the set actually delivers, either
move a feature or weaken the phrase. Record the weaker phrase: a milestone
that promises more than it delivers is how a "done" epic gets shipped half-built.

**A `means:` is a claim to be checked when the milestone lands**, by whatever
the project has, and `__SPECKIT_COMMAND_EPIC_CLOSE__` walks each one. Write it
so that somebody *could* check it (that is what "from outside" buys), and note
that a milestone is the natural place to attach an end-to-end check, because by
construction no feature owns it. The epic's Acceptance walk is a different thing
and is not the union of these claims.

### 5. Ask for the integration target

Features branch from one branch and integrate back into it: `main`, a shared
`release` branch, a branch made for this epic. **Which one is the operator's
decision, not this command's.** Ask, and record the answer as `target:`. Do not
propose a default as though it were settled, and do not create the branch: if it
does not exist yet, say that the first cut will create it from the branch that
holds the epic.

**The target is often the whole content of an edge's `because`.** A feature's
shared helper, data shape or interface is visible to the next feature only if
the first is *integrated into the target*, so "F2 calls the filter F1 adds" is a
statement about the branch. Write the edge that way.

**Then ask about work outside the epic.** Is anything else landing on the same
target that this epic must be sequenced against: another epic, a sibling branch,
a change already in flight? Is anything being designed alongside it? Record each
as an `external.edges` entry with its `relation` and a `because`, in the shape in
`FORMATS.md`.

### 6. Resolve the provisional slugs and complete the edges

Every `f:<slug>` in `epic.md` and `decisions.md` now gets its `F<n>`.

1. **Write the mapping first**, as a table in your working notes: each slug, the
   feature(s) it became. A slug can map to one feature (the common case), to
   several (it was split: write all of them), or to none (the concern was
   dropped or folded elsewhere).
2. **Rewrite mechanically.** Replace each `f:<slug>` token with its `F<n>` (or
   list) wherever it appears in either file: the spanning-facts table's
   `Features` column, every `touches:` line, and the prose of `epic.md` and of
   ledger entries. This is an id substitution, not an edit to a decision: the
   ledger stays append-only in substance. Change nothing but the token. Where
   two slugs became one feature, remove the duplicate id from the line
   (`F1, F1` becomes `F1`).
3. **A slug that maps to nothing is a finding, not a deletion.** A fact or
   decision that touched a concern no feature now owns either has a dependant you
   missed, or its row is wrong. Report it and ask; do not silently drop the
   reference.
4. **Then complete the edges.** The feature list now exists: fill in every
   dependant the specify step could not name, in both files.
5. Confirm no `f:` token remains in either file.

Record each feature's provisional slug on the feature as `slug:` in the
register, so the rewrite is traceable and a later reader can find the
specify-time discussion of the concern.

**Then look for single dependants, and report them; do not refuse.** A fact with
one dependant may belong in that feature's `spec.md`, or you may have missed a
dependant; a decision whose `touches:` names one feature may belong to that
feature. Either may also gain a second dependant when a feature is split later,
so a refusal here would only force a false entry to get past it. List each by
id, say which explanation you think holds, and leave the decision to the user.

### 7. Complete `epic.yml`

Shape is the register in `.specify/extensions/epic/FORMATS.md`. Keep the four
stub keys exactly as written (`decision_prefix` verbatim), set `status:
in-flight`, and add `target:` and the rest. Record no paths to `epic.md` or
`decisions.md`: they are always beside the register. Every feature starts at
`status: planned` with no `feature:` or `branch:`:
`__SPECKIT_COMMAND_EPIC_CUT__` sets those.

**Do not give a feature a `facts:` or `decisions:` list.** Those edges are
recorded once, at the fact and at the decision: the spanning-facts table's
`Features` column and the ledger's `touches:`. A copy on the feature is a second
hand-edited list of the same edges, and the two drift apart.
`__SPECKIT_COMMAND_EPIC_CUT__` passes them to `__SPECKIT_COMMAND_SPECIFY__` as
the feature description instead, so the feature's own `spec.md` cites the ids
without a second list existing anywhere to fall out of date.

Self-check:

- [ ] Every feature is one concern and walkable alone
- [ ] No feature would need a numbering scheme in its own `tasks.md`
- [ ] No mechanism is separated from its alarm
- [ ] Every feature has a gate that could fail
- [ ] Every gate on a conditional behaviour proves both arms
- [ ] Every edge has a `because` naming a consequence
- [ ] Every milestone phrase is what is true from outside, and is not an overstatement
- [ ] No `f:<slug>` token remains in `epic.md` or `decisions.md`
- [ ] Every spanning fact and promoted decision with a single dependant is reported
- [ ] `target:` is the operator's answer, not an assumed default
- [ ] `decision_prefix` is unchanged from the stub

## Report

- The feature list, one line each: id, concern, milestone, gate.
- The slug -> feature mapping, including every split and every slug that mapped
  to nothing.
- The edges, with their consequences.
- **Any candidate feature you merged or cut again**, and which test it failed.
- Any spanning fact or promoted decision with one dependant, by id, and which
  explanation you think holds.
- The integration target, and whether it exists yet.
- Any external edges recorded, with their consequences.
- Any milestone phrase you had to weaken, and to what.
- Next: `__SPECKIT_COMMAND_EPIC_CUT__`, per "What comes next" in `FORMATS.md`.
