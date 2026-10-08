---
description: "Housekeeping routines shipped by the CDN module."
---

# Housekeeping

CDN registers the following [housekeeping](../housekeeping/) routines. They are discovered automatically when `nails/module-housekeeping` is installed and appear under Admin → Utilities → Housekeeping. Run them on demand with `housekeeping:run --routine=…` (add `--dry-run` or `--force` as needed).

## Tokens

`Nails\Cdn\Housekeeping\Tokens` deletes expired rows from the CDN token table. It runs every fifteen minutes.

Audit log columns: `id`, `token`, `expires`.

## Trash

`Nails\Cdn\Housekeeping\Trash` permanently destroys objects that have been in the trash longer than `CDN_TRASH_RETENTION` days (default **180**). Set the config value to `0` to disable deletion; the routine still appears in the list and no-ops. Each item is logged, then passed to `Cdn::objectDestroy()` so both the trash row and the file are removed. Dry-run logs the same lines and skips destroy.

It walks the trash table once per run (`id >` the last id seen). A destroy that fails is logged `ERROR` and counted; the loop moves on and the routine returns `Result::fail` at the end. A missing file is treated as success: the database row is still purged, and the audit log records `MISSING_FILE id=…`.

It runs daily at midnight.

Audit log columns: `id`, human filename, `trashed`.
