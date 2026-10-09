---
description: "Cut the next feature out of an epic: branch it from the integration target, hand to specify, seed its promoted decisions, and update the register, stopping at the spec"
---

# Cut a Feature from an Epic

## User Input

```text
$ARGUMENTS
```

The user input is optional: the feature id to cut (`F3`), and the epic slug if
more than one directory exists under `epics/`. With no feature id, cut the next
ready one (step 1).

## What this command is for

A feature leaves the epic and becomes an ordinary Spec Kit feature. **This
command is mechanical**: feature branch, core specify, promoted decisions,
register.

**It writes no prose document of its own.** Everything a feature needs to know
about the epic is already in `epic.md` and `epic.yml`, and those are in git, so
"what was promised when this was cut" is
`git show <cut-sha>:epics/<slug>/epic.yml`, not a copy somebody has to remember
to write and could later edit into agreement. The one thing it carries across is
a *pointer* to each promoted decision, because the epic ledger lives at the epic
and nothing working at feature level will open it unprompted.

The file shapes are in `.specify/extensions/epic/FORMATS.md`.

## Refuse before you start

Check every item below and report each one that holds, then stop. Where an item
names a way to continue, follow it instead.

1. **No `epic.yml`, or one with no `features:`.** Name
   `__SPECKIT_COMMAND_EPIC_PLAN__`.
2. **No `target:` in the register.** The integration target is the operator's
   choice and plan asks for it. Ask the user for it now, write it, and continue;
   never assume the default branch.
3. **One feature per invocation.** If the user names several, cut the first and
   say so. Two features cut in one pass share a conversation's context and start
   citing each other instead of the epic.
4. **An incoming edge names a feature that is not yet `integrated`.** Quote the
   edge's `because` (it says what breaks) and ask before proceeding. If the
   upstream feature is in fact merged into `target:` and only the register is
   behind, say so: integrating is a convention (see "Integrating a feature" in
   `.specify/extensions/epic/README.md`), and the register should be corrected
   there rather than worked around here. If the user chooses to proceed with the
   upstream genuinely unmerged, record it in the epic ledger as a `decision` with
   the edge's consequence as its rejected alternative, and `touches:` both
   features. Do not proceed silently.
5. **An incoming edge names a feature that was integrated without meeting the
   integration precondition.** Its `status_note` must have both:
   - a `Gate observed` line, or `GATE NOT OBSERVED` with who accepted that;
   - an `Epic review` line with `0 open`, or with every open finding recorded as
     `OWED:` to the owner or `ACCEPTED:`.

   (Markers in `FORMATS.md`.) Without them, the upstream feature was integrated
   on a claim nobody checked, or unreviewed against the epic, and this feature
   is about to be built on it. Name what is missing and ask for it to be
   recorded. Do not proceed until it is.
6. **The feature has no gate in the register.** A feature cut without a gate is
   a feature whose gate gets written after the code, describing what the code
   happens to do. Name `__SPECKIT_COMMAND_EPIC_PLAN__`.
7. **The feature's gate depends on an `[UNGROUNDED]` claim in `epic.md`.**
   Ground it first: a gate resting on an ungrounded claim is a feature built on
   a false premise. The exception is a mark whose "what would settle it" is
   this feature's own gate. Cut it if the gate states the claim as something it
   proves, and name the mark in the report; if the gate does not, refuse and
   name `__SPECKIT_COMMAND_EPIC_PLAN__`.
8. **`epic.md` or `decisions.md` still holds an `f:<slug>` token.** Plan did not
   finish resolving slugs to features; `touches:` cannot be matched. Name
   `__SPECKIT_COMMAND_EPIC_PLAN__`.
9. **Uncommitted changes that this command would carry or overwrite.** This
   command switches branches and commits. Refuse if any of these is dirty:
   - a file outside `epics/` and `specs/`;
   - `epics/<slug>/` itself (the register is about to be edited, and the edit
     must start from what is committed on `target:`).

   Other dirty files under `epics/` or `specs/` belong to other work: leave them
   alone, name them in the report, and offer to stash them if switching to
   `target:` would fail.
10. **A ledger entry that `touches:` this feature is still `open`.** The feature
    would be specified around a question nobody has answered. List each entry
    with its options and recommendation, and ask the owner to decide it first,
    as a new entry that supersedes it (see `open` in `FORMATS.md`). If the owner
    chooses to cut anyway, record a `CUT WITH OPEN DECISIONS` line in the
    feature's `status_note` in step 7 (markers in `FORMATS.md`), and say so in
    the report. Core `specify` will then likely ask clarification questions
    about the same points: answer each by pointing to its `open` entry, and do
    not settle it in the spec; the decision belongs in the ledger. A
    `provisional` entry does not block: name it in the report.

## Steps

### 1. Select the feature

If the user named one, use it. Otherwise take the first feature that is
`status: planned`, in the earliest milestone, with every incoming edge
satisfied. Say which you picked and why, including any you skipped and what
blocks them.

**`planned` on `target:` does not mean nobody has cut it.** A cut is recorded on
the feature's own branch (step 7) and reaches `target:` only when the feature is
integrated, so a feature cut in parallel still reads `planned` here. Treat a
candidate as **already cut** if either exists:

- a local or remote branch with the name step 3 would give it;
- a directory under `specs/` carrying its slug, on any local or remote branch
  (`git branch -a`, then `git ls-tree` per branch).

Skip it, and say in the report which feature was skipped for this reason and
where its cut lives. If the user named a feature that is already cut, say where
and stop; do not cut it a second time.

### 2. Check the integration target

`target:` names the branch features integrate into. The name is the operator's
decision, recorded by plan; never rename it or pick another. **If the branch
does not exist, create it** from the tip of the branch that holds the epic,
which must be the current branch: `epics/<slug>/epic.yml` must be committed
there with the content you read. If it is not, stop and say which branch holds
the epic. Report that you created the
target, and from which branch and commit.

Then check the target holds the epic: `epics/<slug>/epic.yml` must exist on it
with the same content you read. Every cut reads and writes the register on
`target:`; a target that lacks the epic's files leaves this cut reading a stale
or missing register and the next cut not seeing this one. If it lacks them,
stop and say which branch does have them.

### 3. Branch from the target

The feature **is** an ordinary Spec Kit feature from here on and takes an
ordinary name, so that no core command has to be taught about it:

- **Directory (requested):** `specs/<prefix>-<feature-slug>`, where `<prefix>`
  follows the project's `feature_numbering` in `.specify/init-options.json`
  (sequential `NNN`, or timestamp). Absent the setting, sequential. A
  sequential number is the next after the highest `specs/NNN-*` directory on
  any local or remote branch, found the same way as in step 1: a feature cut in
  parallel holds a number the target cannot see yet. This is a request: step 4
  records what core `specify` actually created.
- **Branch:** the directory name, with the project's branch prefix if it uses
  one. Follow what the repository's existing branches do; do not invent a
  convention.

Check out `target:` and create the feature branch from its tip, **unless** the
project registers a `before_specify` hook that creates branches (check
`.specify/extensions.yml`). In that case do not create it yourself: two branch
creators fight. Pass the branch name through to `__SPECKIT_COMMAND_SPECIFY__` in
step 4 as `GIT_BRANCH_NAME`. Either way, confirm afterwards that the feature
branch's parent is the target's tip.

### 4. Hand to core Spec Kit, and STOP at the spec

Run **`__SPECKIT_COMMAND_SPECIFY__` only**, with `SPECIFY_FEATURE_DIRECTORY`
set to the directory from step 3 (and `GIT_BRANCH_NAME` if step 3 said so). Give
it, as the feature description:

- the register entry for this feature: `title`, `concern`, `gate`;
- the `concern` of each feature on the other side of its edges, with the edge's
  `because`, so the boundary can be stated from both sides;
- the `EF` ids this feature depends on, from the spanning-facts table, with the
  instruction that `spec.md` **cites the ids and does not restate the facts**;
- a pointer to `epics/<slug>/epic.md`, and a note that the promoted decisions
  will be in `decisions.md` beside the spec.

Core `specify` writes for readers who do not need the implementation. Keep the
`EF` and decision-id citations, state the gate as the behaviour it observes, and
leave the mechanism the gate names (calls, adapters, storage) to core `plan`.

**Do not run `__SPECKIT_COMMAND_PLAN__`. Do not run `__SPECKIT_COMMAND_TASKS__`.**
A spec nobody has approved is not a thing to decompose: tasks written against an
unapproved spec give it the appearance of having been settled, and the reviewer
who then reads the spec is arguing with work that already exists. Cutting a
feature and planning it are separate decisions with a human between them, and
this command owns only the first.

When it finishes, **read the feature directory it actually created** from
`.specify/feature.json` (`feature_directory`). Use that path from here on, not
the one you requested. If they differ, say so in the report.

### 5. The one post-processing step: citations, not restatements

This is the only place the extension edits core output, and it edits one thing:
**`spec.md` must cite `EF<n>` ids and not restate the facts behind them.** Turn
any restatement into a citation. Change nothing else in the file.

### 6. Seed the feature's `decisions.md` with the promoted decisions

Write `decisions.md` into the directory from step 4, in the format in
`.specify/extensions/epic/FORMATS.md`. Carry every ledger entry whose `touches:`
includes this feature.

**One line of the rejected alternative, plus the pointer.** Not the argument:
that is a copy and it will drift. Not the pointer alone: a pointer is only
followed by someone who already suspects there is something to find, and the
whole failure mode is a feature re-deriving a refusal that looks obviously right
at feature level.

For a `deferral`, carry the **reviving condition** rather than the refusal. That
distinction is why the ledger has two types. Every type, including an `open`
entry, has its pointer shape in `FORMATS.md`.

If the project's own feature-level decision log already lives somewhere else,
put the section there instead and say where.

### 7. Record what happened in the register, and commit

On the feature branch, update the register for this feature:

- `features.<id>.status: cut`
- `features.<id>.feature:` the directory core `specify` created (step 4)
- `features.<id>.branch:` the feature branch
- Add no `facts:` or `decisions:` list. Those edges stay at the fact and at the
  decision.

The register records what happened, not what was intended, which is why this
comes last.

Commit `epics/<slug>/epic.yml` and the seeded `decisions.md` together, and
nothing else: `spec.md` and anything else core `specify` wrote (such as its
`checklists/`) are the human's to review, and whether core output is committed
is the project's convention, not this command's. Leave them uncommitted and say
so in the report. The cut reaches
`target:` when the feature is integrated; until then, the feature branch is
where it lives.

The feature's next status is `gated`, set by convention when its `gate:` has been
observed, with a `Gate observed` line in its `status_note`. Nothing in this
extension sets it. After that comes `__SPECKIT_COMMAND_EPIC_REVIEW__`, and the
next cut that depends on this feature refuses without both.

### 8. Self-check

- [ ] `target:` exists and holds `epics/<slug>/`; if this command created it,
      the report says so
- [ ] The feature branch's parent is the target's tip
- [ ] No `open` ledger entry touches this feature, or the owner chose to cut
      anyway and the `status_note` has a `CUT WITH OPEN DECISIONS` line
- [ ] `__SPECKIT_COMMAND_PLAN__` and `__SPECKIT_COMMAND_TASKS__` were NOT run
- [ ] The register's `feature:` is the directory core `specify` created
- [ ] `spec.md` cites `EF` ids and restates no fact, and nothing else in it was changed
- [ ] Every decision whose `touches:` names this feature is carried, as a
      pointer plus one line, never the argument
- [ ] Deferrals carry their reviving condition, not a refusal
- [ ] The register has `status`, `feature` and `branch`, and no `facts:` or
      `decisions:` list
- [ ] The commit holds the register and the seeded `decisions.md`, alone
- [ ] This command wrote no prose document of its own

## Report

- Which feature, which target, which branch, which feature directory (and
  whether it differs from the one requested), and why this one was next.
- **Whether this command created the target**, and if so from which branch and
  commit.
- Any feature skipped, and the edge that blocks it.
- The promoted decisions carried, by id and type.
- Any restatement turned into a citation in `spec.md`.
- Any dirty files under `epics/` or `specs/` left alone.
- **Anything you could not source from the register or `epic.md`**: that is the
  epic spec being incomplete, and it is worth more than the cut itself.
- **Next: the spec needs a human.** Name `__SPECKIT_COMMAND_PLAN__` as what
  follows *after sign-off*, and say plainly that it has not been run. Then
  anything else "What comes next" in `FORMATS.md` implies, such as a feature
  that is now due for `__SPECKIT_COMMAND_EPIC_REVIEW__`, or
  `__SPECKIT_COMMAND_EPIC_CLOSE__` if this was the last.
