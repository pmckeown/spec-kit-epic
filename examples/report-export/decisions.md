# Epic ledger: report-export

### D-EXPORT-001 · 2026-02-24 · discovery · decided
by:         the agent
refs:       epic.md § What is now wrong, item 1
proven:     src/reports/view.ts:88 (visibility filter applied in the list view only)
touches:    F1, F2, F3
supersedes: —
owes:       —

The brief said exports could reuse "the report's existing permission check".
There is none on the data path: the filter is applied by the list view after the
query returns, so anything calling the query directly gets every row. Every
feature that produces a file must call the filter itself, which is why it became
F1 and EF1.

### D-EXPORT-002 · 2026-02-24 · refusal · decided
by:         the owner
refs:       epic.md § Boundary, Out item 2
proven:     src/reports/view.ts:88; attack transcript in epic.md § What exists today
touches:    F2, F3
supersedes: —
owes:       —

Generating the export by rendering the on-screen report and scraping its table
was REFUSED. It looks right because the screen already shows exactly the right
rows, so it seems to inherit the permission check for free. It does not: the
screen paginates at 100 rows and the scrape would silently truncate, and a
scheduled export (F3) has no screen to render. Rejected alternative stands even
for F2 alone, where the first objection still holds.

Promoted because it is the first design anybody reaches for.

### D-EXPORT-003 · 2026-02-25 · deferral · decided
by:         the owner
refs:       epic.md § Boundary, Out item 3
proven:     —
touches:    F2, F3
supersedes: —
owes:       —

Spreadsheet formats other than CSV are DEFERRED. CSV covers every request in the
source material. Revive when a user asks for a format CSV cannot carry (merged
cells, multiple sheets, typed dates): at that point the export writer in F2 is
the place to add it, and F3 inherits it.

### D-EXPORT-004 · 2026-02-25 · decision · decided
by:         the owner
refs:       epic.md § Facts that span features, EF2
proven:     —
touches:    F3, F4
supersedes: —
owes:       —

A schedule is owned by the user who created it, and runs with that user's
visibility at the moment of each run, not at the moment of creation. Rejected:
snapshotting the visibility at creation, which would keep sending a report to
someone who has since lost access to it.
