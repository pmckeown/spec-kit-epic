# Example: report-export

An invented epic, shown at the state **after `epic.plan`** with work under way.
The product, files and dates are made up; the shapes are real.

The register is shown **as it reads on F2's branch**. `cut` records a cut on
the feature's own branch, so on `main` (the target) F2 still reads `planned`
until it is integrated, and a second `cut` run from `main` recognises it as
already cut by its branch and its `specs/` directory, not by its status.

What to look at:

- **The slug rewrite.** `epic.specify` wrote dependants as provisional slugs.
  `epic.plan` mapped them and rewrote every one; the original slug survives as
  `slug:` on each feature. Here `f:schedules` was split into two features
  (F3 and F4 both carry `slug: schedules`), because a schedule without a way to
  stop it is a mechanism shipped without its alarm.

  | Provisional slug | Became |
  |---|---|
  | `f:export-access` | F1 |
  | `f:csv-download` | F2 |
  | `f:schedules` | F3, F4 (split) |

- **A refusal next to a deferral.** D-EXPORT-002 names the proof that killed
  scraping the screen; D-EXPORT-003 names what would bring other formats back.
- **Every edge's `because` says what breaks**, not that one feature "needs"
  another.
- **Status notes.** F1 records what was observed and how the gate was shown able
  to fail, plus what it did not cover. F2 records an override of spec sign-off.
- **`target: main`.** The operator chose to integrate straight into the default
  branch; another epic might name a release branch. The extension has no
  opinion.
- **`status_note` markers** (`Gate observed`, `Epic review`) that `cut`,
  `review` and `close` read.
- **`external:`**: a fix that landed on the target without being an epic
  feature, and an ordering against another epic, with its `because`.
- **An `[UNGROUNDED]` mark** in `epic.md` that F4's gate depends on, which
  means `epic.cut` will refuse to cut F4 until it is settled.
