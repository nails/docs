---
description: "Housekeeping routines shipped by the Geo-IP module."
---

# Housekeeping

Geo-IP registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoIp\Housekeeping\Cache` deletes rows from `geoip_cache` older than `CACHE_PERIOD` (`1 HOUR`). It runs hourly. The scheduled routine never truncates the table.

Audit log columns: `id`, `ip`, `created`.

The old `geoip:cache:clear` command remains as a deprecated wrapper. Without `--force` it delegates to the routine. With `--force` it truncates the table (logging the table name and row count) unless `--dry-run` is also set. `--force` is wrapper-only; `housekeeping:run` will not truncate.
