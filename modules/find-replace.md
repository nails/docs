---
description: >-
  Search text columns across the database and replace matches from admin or
  the command line.
---

# Find & Replace

Find & Replace searches text columns in the application database and can replace the matches. Searching only reads. A replacement writes the cell and stores the previous value so it can be reverted while that cell is still the value that was written.

```bash
composer require nails/module-find-replace
```

Use it from [Admin](find-replace/admin.md) or the [command line](find-replace/console.md). An application hides or locks columns with [exclusions](find-replace/exclusions.md), and names the records in the results with [resolvers](find-replace/resolvers.md).

## What is searched

The module reads the live schema. A column is included when:

- The table is a base table. Views are left out.
- The type is `char`, `varchar`, `tinytext`, `text`, `mediumtext`, `longtext`, or `json`.
- No [exclusion](find-replace/exclusions.md) hides the column.

Tables are scanned in name order. A table with exactly one primary-key column can be updated. A table with no primary key, or with a composite primary key, is still listed and marked read-only (`no primary key`).

## How a term matches

The term is literal text. `%` and `_` match themselves. Matching is case-sensitive unless the search says otherwise.

The same term is also searched in the forms a stored column might already use:

| Encoding | What is searched |
|---|---|
| Plain text | The term as given |
| HTML | The term passed through `htmlspecialchars()` |
| JSON | The term as the contents of a JSON string, without the surrounding quotes |

A cell that matched the HTML form is updated with the HTML form of the replacement. A JSON match is updated with the JSON form. A plain-text cell receives the replacement as typed. Every occurrence in the cell is replaced.

The snippet kept with a hit is a window around the match, 80 characters by default. The replacement is applied to the whole cell. The command line can change the window with `--context-length`. Admin always uses 80.

## What a replacement will skip

Before writing, the cell is read again. The row is skipped when that read does not allow the change. The same reasons appear in admin and on the command line.

| Reason | When |
|---|---|
| `read-only` | An exclusion marks the column read-only |
| `no primary key` | The table does not have exactly one primary-key column |
| `serialised value — edit via its own screen` | The cell is a PHP-serialised value |
| `column is excluded` | The column is hidden, or is no longer a searchable column |
| `row not found` | The row has been deleted |
| `cell no longer contains the term` | The cell changed after the search |
| `cell has changed since the replacement` | A revert found a cell that is no longer the recorded value |

A serialised cell is included in the results so it can be found. The value is left for the screen that owns it. A cell counts as serialised when it starts with a PHP serialised-type marker and `unserialize()` accepts it, including the value `b:0;`.

{% hint style="warning" %}
A revert writes the previous value only when the live cell is still exactly the recorded value. A later edit in that cell is left as it is.
{% endhint %}

## Queue

Admin searches run as a queue job. The queue is `default` unless `FIND_REPLACE_QUEUE` names another alias or queue class, such as `priority` or `App\Queue\Queues\Reports`. A blank value also uses `default`.

`nails queue:work` listens to `default`. Pass `--queue` when the search uses another queue.

The Find & Replace index shows a warning when that queue has no live worker. A worker is live when its heartbeat is within `QUEUE_WORKER_HEARTBEAT_STALE` seconds (300 by default) and it was started for that queue. This is the same check as the queue dashboard alert. A stale worker, or a worker listening to a different queue, leaves the warning in place. The form can still be submitted; the search stays pending until a worker picks it up. If `FIND_REPLACE_QUEUE` does not name a queue, the index says so and the search is not saved.

## Who can use it

Admin access is two permissions in the Find & Replace group, described on the [Admin](find-replace/admin.md) page. The command line has no separate permission: anyone who can run `nails` against the application can search, replace, and revert.

## Where to go next

| Page | What it covers |
|---|---|
| [Admin](find-replace/admin.md) | Saved searches, results, export, replace, and revert |
| [Console](find-replace/console.md) | `find-replace:search`, `find-replace:replace`, `find-replace:revert`, and `find-replace:fields` |
| [Exclusions](find-replace/exclusions.md) | Columns that are hidden or read-only |
| [Resolvers](find-replace/resolvers.md) | Record titles, facets, badges, and edit links |
| [Housekeeping](find-replace/housekeeping.md) | How long searches and change rows are kept |
