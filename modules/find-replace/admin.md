---
description: >-
  Queue a search in admin, filter the results, export them, and replace or
  revert your own changes.
---

# Admin

Find & Replace is under **Utilities**. **Find & Replace** is the search list. **My changes** is the replacements you applied from admin.

A search belongs to the admin who queued it. The list, the results, the export, and **My changes** show only that person's rows.

## Permissions

Both permissions are in the Find & Replace group. Grant them on a user group, as described in [User Permissions](../admin/user-permissions.md).

| Permission | Class | Allows |
|---|---|---|
| Search | `Nails\FindReplace\Admin\Permission\Search` | Open the screens, queue searches, view results, export CSV, and keep or delete your own searches |
| Replace | `Nails\FindReplace\Admin\Permission\Replace` | Apply replacements and revert your own admin changes |

The navigation item is shown to anyone with Search. Writing also checks Replace, and still requires Search. Someone with Replace alone does not see the screens.

## Starting a search

**Find** is required. **Replace with** is the text that will be written later; leave it blank to delete the term. **Case sensitive** defaults to yes.

**Advanced** limits the scan. Put one entry per line. The entries use the same syntax as [exclusions](exclusions.md#entries). **Limit to tables** with a blank value searches every searchable column. **Exclude tables** is applied after that.

Submitting queues the search on the [configured queue](../find-replace.md#queue). It stays **Pending** until a worker for that queue runs it, then **Running**, then **Complete** or **Failed**. The list refreshes on its own. Progress is the number of tables scanned against the number of tables in this search.

The index shows a warning while that queue has no live worker, using the same check as the queue dashboard alert. Searches can still be submitted. They stay pending until a worker is processing the queue. If the configured queue does not exist, the warning says so and the search is not saved.

When the search finishes, the user who queued it is emailed.

| Email | Subject | When |
|---|---|---|
| Find & Replace: Search complete | Find and replace search complete | The search finished. The message links to the results. |
| Find & Replace: Search failed | Find and replace search failed | The search failed. The message includes the error. |

The search row is already updated if the email cannot be sent. The job is not retried. **Run again** clears the saved results and queues the same term, replacement, case sensitivity, and table limits.

**Keep** holds the search past the [90-day housekeeping](housekeeping.md) window. **Stop keeping** puts it back on that window, counted from the last run. **Delete** asks for confirmation, then removes the search and its results. Change rows are kept until their own housekeeping run, and can still be reverted from **My changes** or the command line.

## Results

The results screen shows the term, the replacement, whether the search was case-sensitive, and any table limits. A running search shows a progress meter. A failed search shows the error and **Run again**.

When resolvers have stored facets, each facet with at least one value is a filter. **Any** leaves that facet out. Several filters combine: a row must match all of them. The counts are outstanding rows (results that have not been replaced yet). The list is 50 rows per page.

Each row shows the record, the column, a snippet of the match, and a preview of the replacement. Badges come from resolvers. The line under the title is the entity label and id, or the table and primary key when no resolver named the row. Read-only rows show their reason and have no checkbox. Action buttons are the links the resolver attached.

**Export CSV** downloads the outstanding rows for the current filters. The file is `find-replace.csv`, with the columns described on the [console](console.md#find-replacesearch) page.

## Replacing

Replace is available once the search is complete and the user has the Replace permission.

| Button | What it updates |
|---|---|
| Replace selected | The ticked rows. The button stays disabled until one is ticked. |
| Replace all | Every outstanding result for this search, on every page. |
| Replace matching filters | Every outstanding result that matches the current filters, on every page. This label replaces **Replace all** while a filter is set. |

A confirmation dialog states the scope. Each cell is then read again and skipped for the [reasons on the overview](../find-replace.md#what-a-replacement-will-skip). Applied rows leave the outstanding list and stay stored until the search is deleted or run again. The screen reports how many were applied, and lists skipped and failed rows.

## My changes

**Utilities → Find & Replace → My changes** groups the replacements you applied in admin, newest first, under their run id. Revert one row, or revert the run. The revert runs immediately. A cell that has changed since the replacement is refused.

A revert is stored as a new run. Revert that run to put the replacement back, while the cell is still the restored value.

Command-line replacements are omitted here. Revert those with [`find-replace:revert`](console.md#find-replacerevert) and the run id the command printed.

## Search API

The search list talks to an admin API. Every route requires an admin session and the Search permission. The API queues, reruns, keeps, and deletes searches. It does not apply replacements.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/find-replace/search` | List your searches |
| `GET` | `/api/find-replace/search/{id}` | Fetch one search you own |
| `POST` | `/api/find-replace/search` | Queue a search |
| `POST` | `/api/find-replace/search/rerun` | Clear results and queue that search again |
| `POST` | `/api/find-replace/search/persistent` | Keep the search, or stop keeping it |
| `POST` | `/api/find-replace/search/delete` | Delete the search and its results |

A new search accepts `term` (required), `replacement`, `case_sensitive`, `include_tables`, and `exclude_tables`. The table fields are a newline-separated string or a list of entries. `case_sensitive` and `persistent` are booleans; `1` and `"1"` also count as true. The response is the search, with status `201` when it was just created.

A search payload contains `id`, `term`, `replacement`, `status` (`PENDING`, `RUNNING`, `COMPLETE`, or `FAILED`), `tables_scanned`, `tables_total`, `result_count`, `persistent`, `last_run`, `error`, and `results_url`. On a complete search, `result_count` is the number of results still outstanding.
