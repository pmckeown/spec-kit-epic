---
description: "Write the epic spec: the re-grounded picture of what exists, the boundary, the facts that span features, and the decisions (especially the refusals) no feature may re-derive"
---

# Specify an Epic

## User Input

```text
$ARGUMENTS
```

The user input **is** the epic description, or a pointer to the document that
shaped it (an issue, a brief, a ticket). Assume it is available in this
conversation even if `$ARGUMENTS` appears literally above. Do not ask the user to
repeat it.

## What this command is for

An epic spec is the home for what does not fit in any one feature: a fact that
several features depend on, a decision one feature took that the others must not
re-take, a gate that only the finished thing can pass. Read
`.specify/extensions/epic/FORMATS.md` before writing anything: it holds the
exact shape of every file this command writes.

**This command is not a summariser.** If you find yourself condensing the source
material, you are doing the wrong job. See step 2.

**The epic spec is not the brief, and the pull to make it one is strong.** A
brief is a point-in-time *argument*: it carries history, it addresses a reader
who held the previous view, and it says things like "the earlier version could
not have known this". An epic spec is a standing reference that features cite
for as long as the code lives. Keep what is true now, and cite the source
document for how it was settled.

## Refuse before you start

Check these first and stop if any holds. Each names what to do instead.

1. **The goal decomposes into one feature.** An epic is warranted by facts,
   decisions or gates that outlive a single feature, not by size, effort, or how
   many files it touches. If there is one concern with one gate, say so and name
   `__SPECKIT_COMMAND_SPECIFY__`.
2. **An epic spec already exists at the target path.** Say which, and ask whether
   this is a revision (edit in place, and record what changed and why) or a new
   epic (different slug). In a revision made after features are cut, every
   entry that changes the epic fills `owes:` with everything still stating the
   old version: features' specs and pointers, and `epic.md` sections (see `owes:`
   in `FORMATS.md`). The report lists them.
3. **There is no source material and no code to ground against.** An epic spec
   written from a conversation alone is a list of intentions. Ask for the issue,
   brief or ticket that occasioned it.

## Steps

### 1. Establish the epic directory and its stub register

Slug the epic in 2-4 words. The directory is `epics/<slug>/`: beside
`specs/`, not inside it, and carrying no number. `specs/` keeps meaning one
thing, which every core Spec Kit command assumes. An epic is cited by name, so
it needs no number; avoid `<slug>-NNN` in particular, which differs from a
feature id only in which end the number is on.

**Settle the decision prefix now, and write it down before any id uses it.** It
is the short uppercase token the ledger's ids are built from
(`decision_prefix: EXAMPLE` -> `D-EXAMPLE-007`). It is declared rather than
derived from the slug, because it is a stable citation key and a derivation rule
is what drifts the moment a slug has three words. Keep it short: a long one gets
abbreviated in practice and the abbreviation quietly becomes the real id.

**The prefix must be unique across epics.** Read `decision_prefix` from every
`epics/*/epic.yml` in the repository. If the one you chose is already used,
refuse it and choose another: two epics sharing a prefix make every
`D-<PREFIX>-NNN` ambiguous, and nothing downstream can tell them apart.

Write the **stub register** `epics/<slug>/epic.yml` holding only these four keys
and nothing else:

```yaml
epic:   <slug>
title:  <title>
decision_prefix: <PREFIX>   # D-<PREFIX>-NNN; declared, never derived
status: shaping             # __SPECKIT_COMMAND_EPIC_PLAN__ completes this file
```

The prefix lives in the register from the first moment a decision id exists, so
there is never a ledger whose ids cite a prefix declared nowhere. Everything else
in `epic.yml` belongs to `__SPECKIT_COMMAND_EPIC_PLAN__`; do not write features,
edges or milestones here.

Read `.specify/memory/constitution.md` if it exists. Constitutional constraints
bind an epic more tightly than a feature, because an epic is where a violation
gets established and then inherited by every feature.

### 2. Re-ground every claim: this is the command's substance

**The source material is a hypothesis about the codebase, and it is stale.**
Issues are written once and the code moves. The single most valuable thing this
command produces is the list of places where the source material is now wrong,
because each one is a feature that would otherwise be built against a false
premise.

For every factual claim in the source material about what the system does today:

- Find it in the code. Cite `file:line`, a configuration file, or a definition
  read from the running system: not a search result you did not open.
- Classify it: **still true**, **has moved** (true then, false now, here is the
  current value) or **is now wrong** (was never true, or the conclusion drawn
  from it does not follow).
- A claim you cannot ground gets written down as `[UNGROUNDED: what would settle
  it]`. Never smooth it into prose. An ungrounded claim that reads like a grounded
  one is worse than an absent one, because nobody will check it again.

**Attack, do not read, any claim about a guard.** A guard that is merely read
looks like it works, and a guard can pass every hand-test through the user
interface while being inert on the code path it was written for: privileged
execution contexts, cached identities and role changes all make the obvious check
answer a different question from the one it appears to ask. So if a claim is
"X is blocked", write the attack, run it against a disposable environment, roll
it back, and record the transcript. If no disposable environment exists, say
so and mark the claim `[UNGROUNDED: attack not run, needs <what>]`.

Where a re-grounding changes what gets built, say so in the finding itself. That
sentence is what a feature reads.

### 3. Draw the boundary

In and Out. **Every Out item carries a reason.** A reason says what depends on
the item and what it depends on; "out of scope for v1" is a placeholder that
the next reader deletes.

Distinguish the two kinds of Out:

- **Not at all**: it is not part of the product. Say why the journey does not
  need it.
- **Not yet**: name what would bring it back, and record it as a `deferral` in
  step 5, not as a refusal.

Finish with "the line": one sentence saying what the epic owns and one saying
what it does not. If you cannot write it, the boundary is not drawn yet.

### 4. Write the facts that span features

A table, one row per fact, each with a stable id (`EF1`, `EF2`, ...) and the
features that depend on it. A fact belongs here when **more than one feature
depends on it**: that is the whole test.

This table exists because the alternative is copying the fact into each feature's
`spec.md`, where the copies drift and nothing detects it. A feature cites `EF2`.
It does not restate it.

**The `Features` column is the canonical record of that edge.** It is not
repeated in the register, so this table is the one place to keep it right.

**Name dependants by provisional slug, never by number.** The feature list does
not exist yet; `__SPECKIT_COMMAND_EPIC_PLAN__` decides it and assigns the
`F<n>` ids. Write each dependant as `f:<slug>` (`f:import`, `f:audit-trail`), a
2-3 word name for the concern you expect a feature to own. Plan rewrites every
`f:<slug>` to its `F<n>` mechanically, so a number written here would only be
renumbered by hand later. Use the same slug for the same concern everywhere in
both files; a slug spelled two ways is two features to plan.

Write the dependants you can name and leave the row; plan completes them.

**When a fact later changes**, a correction keeps its id and a change of
substance gets a new one, with the old row left in place and marked
`superseded by EF<n>`. Features cite ids, so a silently repointed id is a feature
that has drifted with nothing to show for it. Either way the change needs a
ledger entry: that is what makes it discoverable to the features named in the
row.

### 5. Record the decisions, each with its rejected alternative

Write `decisions.md` using the entry format in
`.specify/extensions/epic/FORMATS.md`. Ids are `D-<PREFIX>-NNN`, `<PREFIX>` being
the `decision_prefix` written into the stub register in step 1, in the order the
decisions were taken.

These rules:

- **A decision with no rejected alternative is refused.** Write what was not
  chosen and why not. Without it the entry reads as an accident, and the next
  person to reach the same fork takes the other branch in good faith.
- **`refusal` and `deferral` are different types.** A refusal names the proof that
  killed the alternative. A deferral names the condition that would revive it.
  Collapsing them either resurrects something disproven, or buries something that
  was merely early.
- **`by:` is mandatory on every entry**: who took the decision (the owner, the
  operator, or the agent). It decides who may later reverse it, and a later
  fix that contradicts an owner's entry goes back to the owner.
- **`touches:` is mandatory on every promoted entry**: the features the decision
  reaches, written as `f:<slug>` exactly as in step 4, and the canonical record of
  that edge, kept nowhere else. A refusal that lives only in the feature that
  made it gets re-derived by the next feature, at whose level it looks obviously
  right, and quite possibly shipped.

Decisions made outside this command still belong here, with their reasoning. Cite
the source document section rather than restating the argument.

**A refusal earns promotion when the rejected alternative is the one somebody
would reach for anyway.** An alternative nobody would propose is not worth a
feature's attention; the default approach that happens to be wrong here very much
is. Write that sentence into the entry: the reason a refusal is *promoted*
differs from the reason it was *refused*, and a feature needs the first one.

**`epic.md` may carry a decisions summary table, and it holds id, type and one
line only.** Never the reasoning, never the rejected alternative. Two copies of an
argument in one repository is one copy that will be wrong, and the table is the
one people will edit because it is the one they can see.

### 6. Write the acceptance walk no feature owns

One walk, end to end, of the finished thing, from the point of view of whoever
the epic is for.

**Refuse to write it as a restatement of a feature's gate.** Every feature gate
proves its own concern; the epic's acceptance is the thing none of them touches:
the whole journey, in order, by one person, including the parts where nothing is
supposed to happen, and the parts that only appear over time. If what you have
written could be moved into a feature's `spec.md` without loss, it is not the
epic's acceptance.

### 7. Write `epic.md`

Sections in the order given in `.specify/extensions/epic/FORMATS.md`. Prose in the
voice of the surrounding documents; no invented headings.

Then check it yourself, and report the result:

- [ ] Every current-behaviour claim carries a citation or an `[UNGROUNDED]` mark
- [ ] Every claim about a guard was attacked, not read
- [ ] Every Out item carries a reason
- [ ] Every decision carries its rejected alternative
- [ ] `refusal` and `deferral` are distinguished
- [ ] Every promoted decision carries `touches:`
- [ ] Every spanning fact has an `EF` id
- [ ] Every dependant and every `touches:` entry is an `f:<slug>`, never an `F<n>`
- [ ] `epic.yml` holds the four stub keys and nothing else
- [ ] `decision_prefix` is used by no other `epics/*/epic.yml`
- [ ] The acceptance walk is not a restatement of a feature's gate
- [ ] Nothing in the spanning-facts table is also restated in prose

## Report

- The epic directory and the three files written.
- **Every place the source material was wrong**, one line each, and whether it
  changes what gets built. Lead with this; it is the most useful output.
- Every `[UNGROUNDED]` mark and what would settle it.
- Every guard claim you attacked, and what the attack showed.
- The provisional slugs used, one line each, so plan starts from the same list.
- Checklist results, with any unticked box named.
- Next: `__SPECKIT_COMMAND_EPIC_PLAN__`.
