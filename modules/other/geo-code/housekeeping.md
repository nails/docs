---
description: "Housekeeping routines shipped by the Geo-code module."
---

# Housekeeping

Geo-code registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoCode\Housekeeping\Cache` deletes rows from `geocode_cache` older than `CACHE_PERIOD` (`6 MONTH`). It runs hourly. It never truncates the table; unexpired rows stay until they age out.

Audit log columns: `id`, `address`, `created`.
