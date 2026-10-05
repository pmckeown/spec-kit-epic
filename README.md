# spec-kit-epic

A [Spec Kit](https://github.com/github/spec-kit) extension for work too big for
one feature.

| | |
|---|---|
| Extension id | `epic` |
| Commands | `speckit.epic.specify`, `speckit.epic.plan`, `speckit.epic.cut`, `speckit.epic.review`, `speckit.epic.close` |
| Requires | Spec Kit 0.12.x (tested on 0.12.9), git |
| Status | 0.1.0 |

Command ids are written with dots and rendered for your agent with its own
separator (`speckit.epic.specify` becomes `/speckit.epic.specify` or
`/speckit-epic-specify`). The commands are plain prompts over files. They need
no executor, scripts or specific agent. `review` can use separate agents and a
model per reviewer role where your environment supports them, and runs the same
steps in one session where it does not, saying so in its report.

## Install

```sh
specify extension add --dev /path/to/spec-kit-epic     # from a local checkout
```

Or download a release archive and pass it to `specify extension add --from
<url>`. The extension installs to `.specify/extensions/epic/`, and its commands
are registered for every agent integration in the project.

## Purpose

Core Spec Kit takes one feature from `specify` to `implement`. It has
nowhere to put a fact, a decision or a gate that belongs to several features
at once, and the default (duplicating it into each `spec.md`) is how they
drift.

The `epic` extension owns that level, and only that. It holds:

- the facts more than one feature depends on, in one place;
- the decisions, especially the refusals, that a feature must not re-derive;
- the dependency edges between features, with what breaks if one is violated;
- an acceptance walk that no single feature owns.

It knows nothing about how a feature is built, tested, or reviewed against its
own spec. Everything below the epic is core Spec Kit and whatever else a
project already uses.

### What the commands do

#### `specify`

Writes the epic spec (`epic.md`), the ledger (`decisions.md`) and a stub
register (`epic.yml`). The spec holds what does not fit in any one feature: the
re-grounded picture of what exists today, the boundary, the facts that span
features, the decisions no feature may re-derive (especially the refusals) and
the acceptance walk no feature owns. Re-grounding is the command's main job: it
treats the source material as a stale hypothesis, checks every claim against
the code, and lists the places where the source is now wrong. Each "X is
blocked" claim is attacked against a disposable environment. A claim it cannot
ground is marked `[UNGROUNDED]`.

#### `plan`

Decides what gets built, in what order, and how anyone will know each piece
worked. It splits the epic into features, gives each a gate that must be able
to fail, writes the dependency edges with what breaks if each is violated, sets
milestones and asks for the integration target. It then rewrites the
provisional feature slugs into `F<n>` ids and completes `epic.yml`, the register
every later command reads.

#### `cut`

Turns the next feature into an ordinary Spec Kit feature. It branches from the
integration target, creating the target first if it does not exist and saying
so, hands to core `specify` and stops at the spec, seeds the
feature's `decisions.md` with a pointer to each promoted decision, and records
what happened in the register. It writes no prose of its own: what was promised
at cut time is already in git. Its one post-processing step turns any spanning
fact the new spec restated into a citation.

#### `review`

Checks implemented work against the epic, which a feature's own checks do not
look at. Per feature, it runs after the feature is implemented and gated, before
it is integrated; with `all`, it runs at close. It looks for epic changes that
never reached the feature, drift from what was promised, spanning facts restated
instead of cited, refused alternatives reintroduced, shared definitions changed
by several features and fixes to other features made in passing. It attacks the
guards with reviewers that did not write the code, re-attacks every fix, and
records the review in the register. It does not review code quality, style or
the feature against its own spec.

#### `close`

Checks the whole epic once the last feature is integrated. It reviews the
features against each other, walks the acceptance journey and the milestones,
runs the project's checks on the target with every fix in, and settles the work
outside the epic that has to land in the right order around it. It does not push
or merge onward, because both are the operator's decisions. It brings the epic
to `ready` (everything done that does not need the operator) or `closed`
(nothing left at all), and says which.

## Principles

1. An epic spec is a re-grounding, not a summary. Its central section is
   "what exists today, precisely", and every claim in it carries a code
   citation. An epic assembled from an issue and a conversation is an epic built
   on whatever was true when the issue was filed.
2. Record every decision with the alternative it rejected. Without it, the
   decision reads as an accident, and the next person to meet the same fork
   takes the other branch in good faith.
3. A refusal is not a deferral. "We will not do this" and "not yet"
   propagate differently to features and must be labelled differently.
4. A fact that spans features lives at the epic and is cited by the features,
   never copied into them. This applies to the extension's own files too:
   every feature-fact and feature-decision edge is recorded exactly once, at the
   fact or the decision. See [One place per edge](#one-place-per-edge).
5. A dependency edge states what breaks if it is violated. An edge without a
   stated consequence is easy to drop when the schedule tightens.
6. Every feature's gate proves the feature's own concern; the epic keeps a gate
   no feature owns. Passing every feature's gate does not show that the whole
   journey works.
7. The register stores links and structure, never anything an issue tracker can
   be asked. Pull-request and issue state is read at the moment it is wanted.
8. A command names only a command that exists. It never suggests installing
   one.

## When an epic is warranted

An epic is warranted when the work has facts, decisions or gates that outlive
any one feature. It is not warranted by size, effort, or how many files it
touches.

The clearest signal is a `tasks.md` that needs a numbering scheme to
navigate. A single feature whose task list has grown sub-sections has stopped
being one concern, and reordering the file will not fix that.

These refusals enforce the boundary from the other side:

- `epic.specify` refuses when the goal decomposes into a single feature, and
  names core `specify` instead.
- `epic.plan` refuses when the spanning-facts table is empty. An epic with no
  fact spanning two features is a sequence of features that happen to be
  scheduled together, and the register would be pure overhead.

### How big an epic holds up

An epic holds up well at 4 to 10 features. Beyond that, lean on
milestones: a milestone's `means:` statement (what is true from outside when its
features land) is what sequences the work and keeps a long epic legible. If an
epic is heading well past ten features, consider whether it is two epics with an
edge between them.

## Workflow

```text
epic.specify --> epic.plan --> epic.cut --> core specify --> (sign-off) --> core plan / tasks / implement
                                  ^                                                      |
                                  |                                          observe the gate (convention)
                                  |                                                      |
                                  +-- next feature <-- integrate (convention) <-- epic.review
                                                                |
                                                      last feature integrated
                                                                |
                                                           epic.close
```

Every command ends by naming the step the register now implies, from one table
in [FORMATS.md](FORMATS.md#what-comes-next), so a step that is due is said out
loud rather than remembered.

- One feature per cut. `cut` refuses to cut two at once: two features cut in
  one pass share context and start citing each other instead of the epic.
- The operator names the integration target. `plan` asks for it and records
  it as `target:`: `main`, a shared `release` branch, a `release/<name>` branch
  for this epic, whatever the project's branch strategy is. The extension has no
  opinion about which; `cut` branches from it and features integrate into it.
  If the branch does not exist yet, the first `cut` creates it from the branch
  that holds the epic, and says so in its report.
- `cut` stops at the spec. It runs core `specify` and nothing after it. A
  human approves the spec before core `plan` runs.

## Artifacts

All under `epics/<slug>/`: `epic.md`, `decisions.md` and `epic.yml`. Their exact
shapes are in [FORMATS.md](FORMATS.md), which is what the commands read; this
section is the reasoning behind them. A worked example at the post-plan state is
in [examples/report-export/](examples/report-export/).

Beside `specs/`, not inside it. Location carries the type, so nothing else
has to: `specs/` keeps meaning exactly one thing, which is what every core Spec
Kit command assumes, and the extension's footprint stays one removable
directory.

Epics have no number. Features are many, short-lived and cited constantly, so
each gets a counter with the slug as a label on it. Epics are few and
long-lived, and are cited by name. Avoid `<slug>-NNN` for epics: it differs from
a feature id only in which end the number is on.

### The spanning-facts table

This is the artifact features cite. A feature's `spec.md` cites `EF2`; it
does not restate it, because a restated fact is a copy, and copies drift with
nothing to detect it.

Provisional slugs before plan. `epic.specify` runs before the feature list
exists, so it names dependants by provisional slug (`f:import`). `epic.plan`
decides the features, assigns `F<n>` ids and rewrites every slug mechanically,
recording each feature's original slug as `slug:`. Writing numbers at specify
time would only mean renumbering every list by hand once plan has decided.

A correction to a fact keeps its EF id. A change of substance gets a new id,
with the old row marked superseded. Features cite ids, so a silently repointed
id is a feature that has drifted without any record of the change. Either way a
ledger entry is required: that is what makes the change discoverable to the
features in the row.

### The ledger

The decision prefix is declared, not derived from the slug. A derivation
rule ("uppercase, truncate at the first hyphen") drifts the moment a slug has
three words, and this is a stable citation key. Declaring it also lets the
directory stay descriptive while the id stays short. `epic.specify` writes it
into a stub `epic.yml` before the first id exists, so no ledger id ever cites a
prefix declared nowhere. It refuses a prefix another epic already uses, because
shared prefixes make ids ambiguous.

The id should look different from whatever your features use. Both
levels get cited in the same running prose, so the shape is what tells a
reader which one they are looking at. If your feature ledger keys on a number
(`D-123-004`), this keying on a word (`D-EXAMPLE-007`) is the distinction doing
its job rather than an inconsistency to tidy away.

A feature's id must not embed its epic. Parentage is recorded once, in the
register; an id that also carries it is a second copy of that edge inside a
stable citation key, so a re-parented feature or a renamed epic makes every
citation either wrong or in need of rewriting. Features also exist without
epics, and their ids have to mean the same thing when they do.

Every entry says who decided it (`by:`), and reversing one is a new entry.
The pressure to reverse a decision is highest when closing out findings, and
the obvious fix is often the alternative the owner already rejected. A fix that
contradicts an owner's entry is the owner's call, never the reviewer's. An
agreed reversal is a new entry that supersedes the old one formally (its
status, the new entry's `supersedes:`, and `owes:` for everything still citing
it) rather than editing it in place.

`refusal` and `deferral` are separate types, and that is principle 3 made
mechanical. A refusal names the proof that killed the alternative; a deferral
names the condition that would revive it. Collapse them and you either revive a
disproven option or lose one that was only premature.

A decision is promoted when its `touches:` lists the features it reaches.
`touches:` is also the canonical record of the feature-decision edge: the
answer to "what else does this change?". A decision reaching only one feature
usually belongs to that feature, and plan reports it. It does not refuse it,
because a decision touching one feature today may touch two once a feature is
split, and a refusal would only force a false entry to get past it.

### The register

`status_note` is a field, not a comment, and it is the most useful thing in
the register for whoever picks up an epic later. A one-word `status` is never
the whole truth: what was observed, what is still owed, what the gate did *not*
cover. Read the status notes before anything else when resuming an epic. A
trailing `#` comment wraps at a deep indent and is invisible to anything that
parses the register; a block scalar is neither.

`edges` is a top-level list rather than a `depends_on` array on each feature,
so that each edge can carry its `because`. A list of ids has nowhere to put
one, so the shape enforces principle 5.

The register records no paths it could derive. `epic.md` and `decisions.md` are
always beside it; the slug is the pointer.

### One place per edge

A feature-fact edge lives in the spanning-facts table. A feature-decision edge
lives in the ledger's `touches:`. Neither is repeated in the register.

This is principle 4 applied to the extension's own files, and it is a deliberate
cost: `epic.yml` cannot answer "what does F1 depend on?" without reading two
other files. The alternative, a `facts:` and `decisions:` list on each feature,
is two hand-edited copies of the same edge set, which is exactly the drift this
extension exists to prevent. Anything that needs the reverse index should
derive it, not store it.

The same rule rules out a per-feature charter. It is tempting to have `cut`
write a frozen copy of a feature's concern, gate, facts, decisions and edges
into the feature, so a later review can compare the implementation against what
was promised. The promise already has a home (`epic.md` and `epic.yml`), and a
snapshot of it already exists in git: what was promised at cut time is
`git show <cut-sha>:epics/<slug>/epic.yml`. That is more reliable than a
hand-written snapshot, because nobody has to remember to write it and it cannot
be edited after the fact.

## Commands

| Command | Reads | Writes | Done when | Next |
|---|---|---|---|---|
| `speckit.epic.specify` | source material, and the code it claims things about | `epic.md`, `decisions.md`, stub `epic.yml` | every claim grounded or marked, every decision carries its alternative | `speckit.epic.plan` |
| `speckit.epic.plan` | `epic.md`, `decisions.md`, stub `epic.yml` | `epic.yml`; rewrites `f:<slug>` -> `F<n>` in the other two | every feature is one walkable concern with a gate, every edge has a `because` | `speckit.epic.cut` |
| `speckit.epic.cut` | `epic.yml`, `decisions.md` | the feature branch, the feature's seeded `decisions.md`, the register; runs core `specify` | the feature has a `spec.md` awaiting sign-off | core `plan`, **after a human approves the spec** |
| `speckit.epic.review` | the cut commit, the feature's diff, the epic files, other features' history, `epic-config.yml` | `discovery` and `owes:` entries, a review line in `status_note`; no code | epic changes since cut propagated, every refusal searched for, every shared definition checked live from both sides, every guard attacked and every fix re-attacked | integrate (convention) |
| `speckit.epic.close` | everything on `target:`, `epic-config.yml` | the register's closing note; `status: ready` or `closed` | every step run or recorded as skipped, checks run after the last fix, external edges settled | what is owed to the operator: push, onward merge, owner's calls |

`cut` is mechanical. It branches from `target:`, runs core `specify` with the
feature directory set explicitly, seeds the promoted decisions as one-line
pointers into the directory core `specify` actually created, and records what
happened in the register. If the project registers a `before_specify` hook that
creates branches (such as the core `git` extension), `cut` lets that hook
create the branch rather than fighting it.

It does not run core `plan` or `tasks`: planning or writing tasks against an
unapproved spec makes it look settled. Cutting a feature and planning it are
separate decisions with a human between them.

This extension does not replace any core command and registers no hooks. The
obvious hook (`before_specify`, loading an epic's facts and decisions into every
feature) is not registered because `cut` already writes those pointers into the
feature; a hook would be a second, untested mechanism for a job already done.

It post-processes core output in exactly one place: after core `specify`, `cut`
turns any spanning fact that `spec.md` restated back into a citation of its `EF`
id. That is the one thing about a feature's spec this extension is responsible
for, and it changes nothing else in the file.

`review` checks a feature against the epic, which the feature's own checks do
not. It diffs the epic files since the feature's cut, so a decision reversed or
a fact corrected mid-build reaches a feature already cut through `owes:`. It
searches the feature for designs the ledger refused, checks every shared
definition from both sides, and attacks guards the same way `specify` does,
after implementation rather than before, with reviewers that did not write the
code. It then re-attacks every fix, by an adjacent route, until a round finds
nothing new: in practice, first fixes to a guard were often bypassed.

`close` is a set of steps, and the operator orders them. The default walks
the acceptance first: it is the cheap step, and it finds the defects that only
exist where features meet over the journey. `review all` then covers only what
per-feature reviews structurally cannot see, and attacks only guards changed
since each feature was reviewed, rather than repeating every feature's review.
An operator can reorder or skip steps; what is not fine is skipping one without
saying so, so every skip is recorded.

`close` normally ends at `ready`, not `closed`. It never pushes and never
merges onward, so unless the operator has already done both, something is
always owed. `closed` is reached by hand: push, merge onward, settle the owner's
calls, set `status: closed`, and remove the `OWED:` lines now done.

### Configuration

Nothing needs configuring: every command runs on its defaults. To change them,
copy the installed template to the config file and edit it:

```sh
cp .specify/extensions/epic/config-template.yml .specify/extensions/epic/epic-config.yml
```

(Spec Kit 0.12 installs the template but does not create the config file from
it.) Every key is optional:

- `review.reviewers.<role>.model`: the model for each reviewer role (attacker,
  cross-feature), in whatever form your agent accepts. Empty means the current
  session's model. Attackers do the costly work; the cross-feature reviewer
  mostly reads, and is the natural place for a cheaper model.
- `review.features_per_attacker`: fan-out for `review all`.
- `review.reattack.max_rounds`: the cap on re-attack rounds.
- `review.accept_max_severity`: the highest severity a reviewer may accept
  without the owner.
- `close.steps`: the default order of close's steps. The user input to
  `close` overrides it for one run.

## Steps that are conventions, not commands

Verifying a feature, signing off its spec, integrating it and checking status
are real steps, and each is one obvious action: verifying is whatever the
project already uses, sign-off is a line in `status_note`, integrating is a git
merge plus a two-line YAML edit, and status is reading one file. A command
wrapping any of them would be a second way to do something that has one.

They still matter. `cut` and `close` read the states and the `status_note`
markers these conventions set, and refuse when they are missing, so keep them
true. The markers are in [FORMATS.md](FORMATS.md#status_note).

### Verifying a feature

A feature is `gated` when its `gate:` has been observed, by whatever means the
project has: a test, a walk, a review, a person. That observation is a
precondition of integrating the feature, not a step this extension owns,
because the moment it owns it, it has an opinion about how features are tested
and the purpose statement above stops being true. Set `status: gated` and add a
`Gate observed` line saying what was observed and how it was shown able to fail.
If the gate genuinely cannot be observed, say so with `GATE NOT OBSERVED` and
who accepted that; a later `cut` refuses an upstream with neither.

Walking scenarios works well for this. A scenario a person or an agent can
follow step by step, and judge pass or fail at each step, observes a gate in the
terms it was written in, and the same scenarios serve the milestone claims and
the acceptance walk later.

Milestones are checked the same way. A milestone's `means:` is a claim to be
walked when the milestone lands; `close` walks every one. The epic's
Acceptance walk is not the union of the milestone claims: it is the whole
journey, including the parts that only appear once everything is in place.

### Integrating a feature

1. Check the integration precondition. The `status_note` has a `Gate
   observed` line (or an accepted `GATE NOT OBSERVED`) and an `Epic review` line
   with `0 open`, or with every open finding recorded as `OWED:` to the owner or
   `ACCEPTED:`. If not, the feature is not ready, whatever the branch looks like.
   `cut` and `close` check the same thing and refuse without it.
2. Merge the feature branch into `target:`. Every feature branch edits
   `epic.yml`, so features built in parallel conflict there. On a conflict in
   `epic.yml`, keep `target:`'s version and re-apply only this feature's own
   lines (its `status` and `status_note`); never take the feature branch's copy
   of anyone else's entry.
3. On `target:`, set `status: integrated` and extend the `status_note` with the
   merge commit and anything the gate did not cover.

The order of integration is often the whole content of an edge's `because`: a
feature's shared helper, data shape or interface is visible to the next feature
only once the first is on `target:`, so "F2 calls the filter F1 adds" is a
statement about the branch.

### Recording a sign-off, or an override of it

`cut` stops at the spec so that a human signs it off before planning. When a
spec is signed off, or someone decides to build without sign-off, say so in the
feature's `status_note` in the register, not in the feature's own spec, with who
decided and when. The two forms are in [FORMATS.md](FORMATS.md#status_note).

### Checking status

Read `epic.yml`: the `status` and `status_note` of each feature, in register
order, are the epic's state. Read the notes before anything else. "What comes
next" in [FORMATS.md](FORMATS.md#what-comes-next) says what the state implies.

### Changing the epic mid-build

Facts get corrected, decisions get reversed, gates get tightened and acceptance
steps get rewritten after features are cut, often by the operator, often in the
epic files alone. A feature already cut does not find out by itself. Its
`spec.md`, its plan and its seeded `decisions.md` still state the old version,
it reads as current, and the feature is built to it.

So every change to the epic after features are cut is a ledger entry, and the
entry carries `owes:`: every feature (spec, plan prose and pointers) and every
section of `epic.md` that still states the old version. The list is cleared one
item at a time as each is brought current. `review` diffs the epic since each
feature's cut to catch changes nobody recorded, and `close` refuses while
anything is owed.

### Work outside the epic

An epic rarely has its target to itself. Other branches and epics often land on
the same target, sometimes in a required order, and some work has to be
designed alongside it. The register's `external:` section records those: `work`
that landed on the target without being an epic feature, and `edges` to other
branches or epics, each with a `because`, like every other edge. `plan` asks
about them, and `close` checks every one.

## Layout

Installed into a project:

```text
.specify/extensions/epic/
  extension.yml
  README.md                     this file
  FORMATS.md                    the file shapes; the commands read this
  config-template.yml           copy to epic-config.yml to change reviewer models, fan-out, re-attack limit, close order
  commands/speckit.epic.*.md    the commands, rendered into each agent's command directory
```

Produced by the commands, in the project:

```text
epics/<slug>/                   epic.md, decisions.md, epic.yml
specs/<feature>/decisions.md    seeded by cut with pointers to promoted decisions
```

## License

MIT. See [LICENSE](LICENSE).
