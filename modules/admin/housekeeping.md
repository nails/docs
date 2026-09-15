---
description: "Housekeeping routines shipped by the Admin module."
---

# Housekeeping

Admin registers the following [housekeeping](../housekeeping/) routines. They are discovered automatically when `nails/module-housekeeping` is installed and appear under Admin → Utilities → Housekeeping. Run them on demand with `housekeeping:run --routine=…` (add `--dry-run` or `--force` as needed). `admin:dataexport:process` is unchanged.

## Sessions

`Nails\Admin\Housekeeping\Sessions` deletes rows from `admin_session` whose `heartbeat` is older than one hour. It runs every five minutes.

Audit log columns: `id`, `user_id`, `heartbeat`.

## Data export

`Nails\Admin\Housekeeping\DataExport` deletes expired rows from `admin_export` and, when `download_id` is set, destroys the matching CDN object. Each item is handled in a database transaction, the same as the old cleaner. It runs every fifteen minutes.

Audit log columns: `id`, `download_id`, `expires`.

## Changelog

`Nails\Admin\Housekeeping\ChangeLog` deletes rows from `admin_changelog` older than `ADMIN_CHANGELOG_RETENTION_DAYS`. Unset or `0` disables deletion; the routine still appears in the list and no-ops. There is no module default — apps that want a policy set the config.

It runs daily.

Audit log columns: `id`, `user_id`, `created`.

There was no cleaner for this table before.
