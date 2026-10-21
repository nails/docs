---
description: >-
  Search, replace, revert, and list searchable fields from the command line.
---

# Console

Four commands share the application database with [Admin](admin.md). A command-line replacement is recorded as its own run. Revert it with the run id the command prints. Those runs are omitted from **My changes** in admin.

Long output is sent to `less` or `more` when the terminal is taller than the screen. Set `PATH` so one of those is available, or the command pages in place (`Enter` for the next page, `q` to stop).

## Shared filters

`find-replace:search` and `find-replace:replace` accept the same filters. Entries use the [exclusion syntax](exclusions.md#entries): a table, `table.column`, `table.*`, or a `/regex/` tested against `table.column`.

| Option | Effect |
|---|---|
| `--table` | Keep columns this entry covers. Repeat the option to add entries. An empty list searches every searchable column. |
| `--exclude-table` | Drop columns this entry covers, after `--table` is applied. Repeatable. |
| `--case-insensitive` | Match without regard to case. The default is case-sensitive. |
| `--facet=key:value` | Keep hits whose facet equals this pair. Repeat the option; every pair must match. |
| `--resolved-only` | Keep hits a resolver named, and drop unresolved `table.column` hits. |
| `--limit` | Stop after this many hits. |
| `--context-length` | Characters of snippet to keep around each match. Default `80`. |

`--facet` and `--resolved-only` run after the scan, and `--limit` then counts what remains. Without those two filters, `--limit` stops the scan once enough hits have been found. A closed facet rejects a value outside its list and the command stops. See [Resolvers](resolvers.md) for facet keys, and list them with `find-replace:fields`.

## find-replace:search

Search for one term and print the hits. Nothing is written.

```bash
nails find-replace:search "old name"
nails find-replace:search "old name" --table=app_article --case-insensitive
nails find-replace:search "old name" --facet=status:PUBLISHED --resolved-only
```

The table columns are table, column, id, title, snippet, and the read-only reason when there is one.

`--format` and `--output` are passed together. `--format` is `csv` or `json`.

```bash
nails find-replace:search "old name" --format=json --output=results.json
nails find-replace:search "old name" --format=csv --output=results.csv
```

CSV columns are `table`, `column`, `primary_key`, `title`, `encoding`, `term`, `snippet`, `read_only`, `reason`, and `facets` (`key=value` pairs separated by `; `).

JSON is version `1` and is the file `find-replace:replace --from` reads back. Each hit records the table, primary key, column, encoding, the term that matched, the full cell, title, entity label and id, facets, read-only state, and actions.

## find-replace:replace

Replace one term, or the rows in a search JSON file. The second argument is the replacement and may be empty, which deletes the term.

```bash
nails find-replace:replace "old name" "new name"
nails find-replace:replace "old name" "new name" --dry-run
nails find-replace:replace "old name" "new name" --yes
nails find-replace:replace "old name" "new name" --from=results.json
```

The command prints the rows it would change and the rows it would skip, then asks `Apply these replacements? [y/N]`. `--yes` skips the question. `--dry-run` writes nothing.

`--from` takes the rows, the term on each row, and the case sensitivity from the JSON file. `--case-insensitive` is ignored when that file records a case-sensitive search. `--table`, `--exclude-table`, `--facet`, `--resolved-only`, and `--limit` still narrow the file. Each cell is read again before it is written, and skipped for the [reasons on the overview](../find-replace.md#what-a-replacement-will-skip).

Applied, skipped, and failed counts are printed at the end. When at least one row is applied, the command prints a run id. Pass that id to `find-replace:revert`. The command exits with a failure code when any row fails. Skipped rows are reported and do not, on their own, fail the command.

## find-replace:revert

Restore every cell from a replace run whose live value is still the recorded replacement. There is no confirmation prompt. Each restored cell is stored as a new run, and that run id is printed so it can be reverted the same way.

```bash
nails find-replace:revert 3f1c0a2e-4b6d-4e1a-9c80-0a1b2c3d4e5f
```

A run id with no change rows is an error. A cell that has been edited since the replacement is refused (`cell has changed since the replacement`). Restored, refused, and failed counts are printed. A failed row fails the command.

## find-replace:fields

List the facet keys, resolvers, and tables the filters can use.

```bash
nails find-replace:fields
nails find-replace:fields -v
```

Facets are sorted by key. A facet with a value list accepts only those values. A facet that accepts any value says so. Resolvers are listed with the tables they claim. `-v` adds each column, whether it is searchable or read-only, and the read-only reason. Hidden columns are omitted. The module's own `TableColumn` resolver lists no tables; it is the fallback described in [Resolvers](resolvers.md).
