---
description: "Housekeeping routines shipped by the CDN module."
---

# Housekeeping

CDN registers the following [housekeeping](../housekeeping/) routines. They are discovered automatically when `nails/module-housekeeping` is installed and appear under Admin → Utilities → Housekeeping.

The old `cdn:trash:empty` command remains as a deprecated wrapper that delegates to `housekeeping:run --force`.

## Tokens

`Nails\Cdn\Housekeeping\Tokens` deletes expired rows from the CDN token table. It runs every fifteen minutes.

Audit log columns: `id`, `token`, `expires`.

## Trash

`Nails\Cdn\Housekeeping\Trash` permanently destroys objects that have been in the trash longer than `CDN_TRASH_RETENTION` days (default **180**). Each item is logged, then passed to `Cdn::objectDestroy()` so both the trash row and the file are removed. Dry-run logs the same lines and skips destroy.

It runs daily at midnight.

Audit log columns: `id`, human filename, `trashed`.
