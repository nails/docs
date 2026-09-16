---
description: "Housekeeping routines shipped by the Geo-code module."
---

# Housekeeping

Geo-code registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoCode\Housekeeping\Cache` deletes rows from `geocode_cache` older than `GEO_CODE_CACHE_PERIOD` seconds (default **15552000**, 180 days). It runs hourly. It never truncates the table; unexpired rows stay until they age out. The same value is used when looking up a cached result, so the service and the routine cannot drift.

Audit log columns: `id`, `address`, `created`.
