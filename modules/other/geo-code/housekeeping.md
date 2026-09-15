---
description: "Housekeeping routines shipped by the Geo-code module."
---

# Housekeeping

Geo-code registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoCode\Housekeeping\Cache` deletes rows from `geocode_cache` older than `CACHE_PERIOD` (`6 MONTH`). It runs hourly. The scheduled routine never truncates the table.

Audit log columns: `id`, `address`, `created`.

The old `geocode:cache:clear` command remains as a deprecated wrapper. Without `--force` it delegates to the routine. With `--force` it truncates the table (logging the table name and row count) unless `--dry-run` is also set. `--force` is wrapper-only; `housekeeping:run` will not truncate.
