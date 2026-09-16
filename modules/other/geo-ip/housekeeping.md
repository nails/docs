---
description: "Housekeeping routines shipped by the Geo-IP module."
---

# Housekeeping

Geo-IP registers the following [housekeeping](../../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Cache

`Nails\GeoIp\Housekeeping\Cache` deletes rows from `geoip_cache` older than `GEO_IP_CACHE_PERIOD` seconds (default **3600**). It runs hourly. It never truncates the table; unexpired rows stay until they age out. The same value is used when looking up a cached result, so the service and the routine cannot drift.

Audit log columns: `id`, `ip`, `created`.
