# Epic: Scheduled report export

## Why this epic

Users can see reports but cannot take them anywhere. The claim is that any user
can get a report they can already see out of the product, once or on a
schedule, without ever receiving a row they could not see on screen. What
prevents it today is that no code path produces a file, and the one permission
check that exists is not where the brief thinks it is.

## What exists today

- Reports are built by one query, `buildReport()` in `src/reports/query.ts:41`,
  which returns every matching row regardless of who asks.
- The visibility filter is applied **after** the query, in the list view:
  `src/reports/view.ts:88`. Nothing else calls it.
- Attack, run against a disposable copy: calling `buildReport()` directly as a
  user with access to 3 of 10 projects returned rows from all 10. Transcript
  kept in the epic's working notes. The "existing permission check" the brief
  relies on does not exist on the data path.
- Outbound email goes through one sender, `src/mail/send.ts:12`, which already
  writes an unsubscribe header. `[UNGROUNDED: whether the header's link works
  for a recipient who is not signed in; settle by sending one to a fresh
  inbox.]`
- There is no job scheduler. Background work today runs from one nightly task,
  `jobs/nightly.ts:5`.

## What is now wrong

1. The brief says exports can "reuse the report's permission check". There is
   none on the data path (see above). This changes what gets built: the filter
   becomes its own feature, first, and every export path calls it. See
   D-EXPORT-001.
2. The brief assumes a scheduler exists. It does not. F3 owns adding one, and
   its gate has to prove a schedule does not fire early, which the nightly task
   would make easy to get wrong.

## Boundary

**In:** CSV export on demand; scheduled email delivery; unsubscribe and expiry.

**Out:**

1. **Exports of raw tables, not reports.** Not at all: every request in the
   source material is for a report a user already looks at, and a raw-table
   export has no permission model to inherit.
2. **Rendering the screen and scraping it.** Not at all; refused in
   D-EXPORT-002.
3. **Formats other than CSV.** Not yet; deferred in D-EXPORT-003 with its
   reviving condition.

The line: this epic owns getting a visible report out of the product. It does
not own what reports exist or who can see them.

## Facts that span features

| Id  | Fact | Features |
|-----|------|----------|
| EF1 | Every export path calls the visibility filter itself, with the requesting user, before writing any row. The list view's filter is not on the data path. | F1, F2, F3 |
| EF2 | A schedule runs as its owner, with the owner's visibility at the moment of the run. | F3, F4 |
| EF3 | Every delivered email carries a working unsubscribe link that needs no sign-in. | F3, F4 |

## Decisions

| Id | Type | One line |
|----|------|----------|
| D-EXPORT-001 | discovery | The permission check is not on the data path |
| D-EXPORT-002 | refusal | Do not render-and-scrape the on-screen report |
| D-EXPORT-003 | deferral | Formats other than CSV, until one is asked for |
| D-EXPORT-004 | decision | A schedule runs with its owner's visibility at run time |

## Acceptance

One user, from the beginning: they open a report covering projects they can and
cannot see, export it, and find only their own projects in the file. They set it
to arrive weekly. Nothing arrives early. It arrives on the day. An administrator
then removes their access to one project, and the next delivery no longer
contains it, with nobody touching the schedule. They press unsubscribe in the
email from a device where they are not signed in, and nothing further arrives.
