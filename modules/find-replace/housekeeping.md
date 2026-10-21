---
description: >-
  Daily routines that delete expired find-replace searches and change rows
  after 90 days.
---

# Housekeeping

Find & Replace registers two [housekeeping](../housekeeping/) routines. Both use [DeletesModelRows](../housekeeping/traits/deletes-model-rows.md), run daily, and appear under **Admin → Utilities → Housekeeping** when `nails/module-housekeeping` is installed. Run one on demand with `nails housekeeping:run --routine=…`, and add `--dry-run` to log the rows without deleting them.

## Searches

`Nails\FindReplace\Housekeeping\Searches` deletes searches that are not marked **Keep**, 90 days after `last_run`. Deleting a search removes its result rows and facet rows with it.

A search marked **Keep** in admin is stored with `persistent` set, and this routine leaves it alone. **Stop keeping** puts it back under the 90-day rule. **Run again** updates `last_run` when the worker starts, so the window is counted from that run.

Audit log columns: `id`, `last_run`, `persistent`.

## Changes

`Nails\FindReplace\Housekeeping\Changes` deletes change rows whose `created` time is older than 90 days. That includes admin replacements, command-line replacements, and the rows written by a revert.

Change rows expire after 90 days even when the search is marked **Keep**. Once they are gone, `find-replace:revert` reports that the run id has no changes, and **My changes** no longer lists them.

Audit log columns: `id`, `created`.
