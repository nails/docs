---
description: "Housekeeping routines shipped by the Admin module."
---

# Housekeeping

Admin registers the following [housekeeping](../housekeeping/) routines. They are discovered automatically when `nails/module-housekeeping` is installed and appear under Admin → Utilities → Housekeeping. Run them on demand with `housekeeping:run --routine=…` (add `--dry-run` or `--force` as needed). `admin:dataexport:process` is unchanged.

## Sessions

`Nails\Admin\Housekeeping\Sessions` deletes rows from `admin_session` whose `heartbeat` is older than `ADMIN_SESSION_RETENTION` seconds (default **3600**). It runs every five minutes.

Audit log columns: `id`, `user_id`, `heartbeat`.

## Data export

`Nails\Admin\Housekeeping\DataExport` deletes expired rows from `admin_export`. The CDN download is removed by `Nails\Admin\Event\Listener\Export\Deleted`, which listens for `DELETED` on the export model so any `delete()` / `deleteMany()` (housekeeping included) cascades the file. The object is destroyed only when no remaining row still references that `download_id`. Identical requests share a file, so the last sibling is the one that removes it. CDN failures are logged and do not fail the delete or the routine.

Dry-run logs the rows and skips `delete()`, so the listener does not run.

It runs every fifteen minutes.

Audit log columns: `id`, `download_id`, `expires`.

## Changelog

`Nails\Admin\Housekeeping\ChangeLog` deletes rows from `admin_changelog` older than `ADMIN_CHANGELOG_RETENTION_DAYS`. Unset or `0` disables deletion; the routine still appears in the list and no-ops. There is no module default — apps that want a policy set the config.

It runs daily.

Audit log columns: `id`, `user_id`, `created`.

There was no cleaner for this table before.
