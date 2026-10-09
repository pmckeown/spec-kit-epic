---
description: "Review implemented work against the epic, not its own spec: epic changes not propagated, drift from what was promised, refusals reintroduced, shared definitions touched by several features, independent attack and re-attack of guards"
---

# Review against the Epic

## User Input

```text
$ARGUMENTS
```

The user input is optional: the feature id to review (`F3`), and the epic slug
if more than one directory exists under `epics/`. With no feature id, review the
feature on the current branch. With `all`, review the whole epic as it stands on
`target:` (this is what `__SPECKIT_COMMAND_EPIC_CLOSE__` runs).

## What this command is for

Every feature is checked against **its own spec** by whatever the project uses.
Nothing in that checks it against **the epic**: whether the epic changed after
the feature was cut and the feature never heard, whether the feature still does
what was promised, whether it quietly re-decided something an earlier feature
settled, whether it changed something another feature owns. Each feature can be
green on its own and the epic still drift, because the drift is between
features, and no feature's tests look there.

This command looks there. Per feature, it runs after the feature is implemented
and gated, before it is integrated. With `all`, it runs at close. It does not
review code quality, style, or the feature against its own spec.

The file shapes are in `.specify/extensions/epic/FORMATS.md`. Reviewer models,
fan-out and re-attack limits are in `.specify/extensions/epic/epic-config.yml`
if it exists; otherwise use the defaults stated here.

## Refuse before you start

Check every item below and report each one that holds, then stop. Where an item
names a way to continue, follow it instead.

1. **No `epic.yml` with `features:`.** Name `__SPECKIT_COMMAND_EPIC_PLAN__`.
2. **The feature is `planned`.** There is nothing to review. Name
   `__SPECKIT_COMMAND_EPIC_CUT__`.
3. **The feature has no implementation yet** (its branch has no commits beyond
   the cut). Say so and stop.

## Who reviews

**The reviewer should not be the author.** Steps 2 to 7 read; step 8 attacks.
If your environment can run separate agents or fresh sessions, use them, give
each only the gate, the diff and the epic files (never the conversation that
wrote the code), and take the model for each role from `review.reviewers` in the
config. If it cannot, run the steps yourself and say so in the report: an
author attacking their own guard tends to attack the case they already handled.

### What `all` covers

By the time `all` runs, every feature has normally been reviewed on its own,
with attacks and re-attack rounds. Running all of that again is the most
expensive thing close can do, and it re-checks what was already checked. So
`all` covers only what a per-feature review structurally cannot see, across
the whole epic at once:

- step 2, epic changes not propagated;
- step 5, refusals reintroduced;
- step 6, shared definitions touched by several features;
- step 7, cross-feature fixes recorded as discoveries.

Steps 3 and 4 are skipped. Step 8 attacks **only guards changed since each
feature's own `Epic review` line** (including by fixes made during close).
**A feature with no `Epic review` line gets the full review**, every step, as
though it were reviewed on its own. Say in the report which features got which.

**With `all`, fan out.** One reviewer cannot hold a whole epic. Use one
**attacker** per `review.features_per_attacker` features (default 3), grouping
features that share definitions, plus one **cross-feature reviewer** for steps 2
to 7 across the whole epic. A reviewer's report is its final message; it need
not write a file.

## Steps

### 1. Establish what was promised

Find the **cut commit**: the first commit that names this feature's directory
in the register, since only `cut` writes `feature:`:

```sh
git log --reverse --format=%H -S'<feature directory>' -- epics/<slug>/epic.yml | head -1
```

(Searching for `status: cut` matches the plan commit, which wrote every
feature's status.) What was promised is the epic at that commit
(`git show <cut-sha>:epics/<slug>/`), and the feature's `spec.md`.

What was delivered is the feature branch's diff against the point it branched
from `target:`. For `all`, it is every feature's integrated change on `target:`.

### 2. Epic changes that never reached the feature

The epic keeps moving while
features are built: a fact corrected, a decision reversed, a gate tightened, an
acceptance step rewritten, often by the operator, mid-build, in the epic files
alone. A feature cut before the change still carries the old version in its
`spec.md`, its plan, its tasks and its seeded `decisions.md`, and was built to
it.

Diff `epics/<slug>/` between the cut commit and now. For every change since the
cut that touches this feature (by `Features`, `touches:`, an edge, or its own
register entry):

- Is there a ledger entry for it, and does its `owes:` name this feature?
- Does the feature's spec, plan and seeded `decisions.md` state the current
  version, or the old one? Read the prose, not only the pointers: a superseded
  fact restated in a sentence is as stale as a stale pointer.
- Was the feature built to the old version?

An unrecorded change is a finding against the epic, not the feature: add the
ledger entry, with `owes:`. A feature built to the old version is a finding
against the feature.

### 3. Drift from what was promised

Keep this short. Compare the feature's `concern` and `gate` at the cut commit
with what they are now, and both with what the diff does. A gate rewritten to
match what the code does, with no ledger entry, is a finding. So is behaviour
outside the concern: it may be right, it is still unpromised.

### 4. Spanning facts cited, not restated

Any restatement of a spanning fact in the feature's spec, plan or tasks is a
finding, and a restatement that **disagrees** with the fact is a serious one:
the feature was built against its own copy.

### 5. Refused alternatives not reintroduced

For every `refusal` in the ledger, **whether or not its `touches:` names this
feature**: state the refused alternative as the concrete design someone would
write, and search the diff for it. A refusal is most likely to come back in a
feature it was never promoted to, because nobody there was told.

**Include test and scenario commits.** An affordance added to make something
testable (a shortcut, a bypass, a link that skips a step) can reintroduce a
refused design into the product without anyone deciding to.

For every `deferral`, check whether this feature has met its reviving
condition. If so, the deferral should be revisited, not silently built around.

### 6. Shared definitions touched by several features

List every definition this feature
changed (a function, type, interface, shared configuration, stored shape,
anything with one name used in several places) that **another feature in this
epic** also introduced or changed.

Read the **live** definition, from the running system or the built artefact
where the project has one, not the source file that first created it: a later
change may have replaced it. For each, check both features still hold: the
later change did not break the earlier feature's gate, and did not silently undo
it.

### 7. Fixes to other features become discoveries

If the diff changes behaviour another feature introduced (a fix, a correction,
a defect found along the way), append a `discovery` entry to the epic ledger:
**one per root cause**, not one per symptom, with `touches:` naming every
feature on both sides. An unrecorded fix leaves the other feature's register
entry and gate claiming something that was not true.

### 8. Attack the guards, then attack the fixes

Every claim of the form "X is blocked", "X is refused", "only Y can do X", in the
feature's gate or in the epic's facts it depends on, is attacked, not read:
write the attack, run it against a disposable environment, roll it back, record
what happened. `specify` attacked the guards that existed before the epic; this
attacks the ones the features added. If no disposable environment exists, say
so and record each claim as not attacked.

**Then re-attack.** Once findings are fixed, the **same** attacker attacks each
fix, told to get round it by an adjacent route: a record put in place before
the guard runs, a second path to the same effect, an ordering the fix did not
move. In practice, fixes often fail this, and a fix can introduce its own
regression. Repeat until a round finds nothing new at high or medium severity,
or `review.reattack.max_rounds` rounds (default 3) have run. Record each round.

### 9. Classify and route the findings

Each finding gets a severity (high, medium or low) and one of:

- **fix**: a defect against what the epic promised. Fixed, then re-attacked
  (step 8).
- **owner's call**: fixing it changes what the product promises, or needs a
  decision only the owner can make. Never decided by the reviewer, whatever the
  severity. Recorded as `OWED:` to the owner.
- **accept**: left as it is, with a reason. Only findings at or below
  `review.accept_max_severity` (default `low`) may be accepted without the
  owner; above that, acceptance is the owner's call. Recorded as `ACCEPTED:`.

**Check every proposed fix against the ledger before it is made.** A fix that
contradicts a ledger entry (reintroduces what it refused, undoes what it chose,
or changes what it fixed) is a reversal of that decision, not a fix. If the
entry's `by:` names the owner, or it has no `by:`, the fix is an **owner's
call** whatever its severity: queue it as a question, and change nothing. The
re-attack loop makes this likelier, because under pressure to close a finding
the obvious fix is often the very alternative the owner already rejected. If the
owner agrees, or the entry was the agent's own, the reversal is recorded as a
new entry that supersedes the old one in full (`supersedes:`, the old entry's
status, and `owes:`; see Reversing in `FORMATS.md`).

**Where fixes land.** Per feature, on the feature's branch, by its own workflow.
With `all`, every feature is already integrated: fixes land on a branch cut
from `target:` (named per the repository's convention), and reach `target:`
under the operator's usual merge rule. This command changes no code itself.

### 10. Record the review

Append to the feature's `status_note` (with `all`, to each feature reviewed):

```text
Epic review <date>: <n> findings, <m> open, <r> attack rounds.
```

followed by one `OWED:` or `ACCEPTED:` line per finding left so (markers in
`FORMATS.md`). Commit only the register and any ledger entries this review
appended.

## Self-check

- [ ] The cut commit was found by the `feature:` recipe, and the epic read at it
- [ ] Every epic change since the cut that touches the feature was checked for propagation
- [ ] Every refusal in the ledger was searched for, including in test and scenario commits
- [ ] Every shared definition touched by another feature was read live and checked from both sides
- [ ] Every cross-feature fix has one `discovery` entry per root cause
- [ ] Every guard claim was attacked, and every fix re-attacked, or the report says why not
- [ ] The attackers were independent of the author, or the report says they were not
- [ ] No product decision was taken by a reviewer
- [ ] No fix reverses a ledger entry the owner decided; every reversal the
      owner agreed is a superseding entry, with the old entry's status set
- [ ] No code was changed by this command

## Report

- Findings, most serious first: severity, route, what was promised, what was
  delivered, the evidence.
- Epic changes that had not reached a feature, and the `owes:` written for them.
- Discovery entries appended, by id.
- Attack rounds: per round, what was attacked, by whom, what got through.
- Owner's calls, listed together so the owner can take them in one sitting.
- **Next**, from the register, per "What comes next" in `FORMATS.md`.
