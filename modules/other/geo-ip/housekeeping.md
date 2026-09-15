---
description: "Housekeeping routines shipped by the Geo-IP module."
---

# Housekeeping

Geo-IP registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoIp\Housekeeping\Cache` deletes rows from `geoip_cache` older than `CACHE_PERIOD` (`1 HOUR`). It runs hourly. It never truncates the table; unexpired rows stay until they age out.

Audit log columns: `id`, `ip`, `created`.
