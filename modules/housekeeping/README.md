---
description: >-
  This module provides a stable API for defining cleanup routines that are
  automatically discovered, scheduled, and audited to a log file.
---

# Housekeeping

The housekeeping module is an orchestrator for cleanup work: truncating or deleting stale rows, archiving or removing files from disk, and any other residue a module or app needs to tidy up.

A single cron task (`housekeeping:run`, every minute) asks the orchestrator which routines are due and executes them in-process so the audit trail stays in one place. Unlike [cron](../cron.md) tasks, a housekeeping routine **is** the work — there is no separate console command to bind.

```bash
composer require nails/module-housekeeping
```

## Defining a routine

Any instantiable class under `src/Housekeeping/` that implements `Nails\Housekeeping\Interfaces\Routine` is picked up automatically. That applies to the app (`App\Housekeeping\…`), any installed module (`Nails\Cdn\Housekeeping\…`, and so on), and the housekeeping module itself (`Nails\Housekeeping\Housekeeping\…`).

You do not register routines anywhere. Discovery is:

1. Walk every available component.
2. Find classes in that component’s `Housekeeping` namespace (the `src/Housekeeping/` directory).
3. Keep those that implement `Routine` and can be instantiated.

Most authors extend `Nails\Housekeeping\Routine\Base` rather than implementing the interface from scratch. `Base` is a bootstrapper: it maps a handful of class constants onto the interface getters. It does **not** do the cleanup. You still have to populate `execute()`.

```php
namespace App\Housekeeping;

use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Routine\Context;
use Nails\Housekeeping\Routine\Result;

class AnonymiseGuests extends Base
{
    const DESCRIPTION     = 'Removes personal data from guest orders older than two years';
    const CRON_EXPRESSION = '@daily';

    public function execute(Context $oContext): Result
    {
        // Honour dry-run: log what you would do, but do not mutate.
        if ($oContext->isDryRun()) {
            $oContext->log('Would anonymise expired guest orders');
            return Result::ok();
        }

        // …do the work, writing an audit line per item with $oContext->log()…

        $oContext->log('Anonymised expired guest orders');

        return Result::ok(12);
    }
}
```

That class, dropped into `src/Housekeeping/AnonymiseGuests.php`, is enough. `housekeeping:list` will show it, the orchestrator will run it when `@daily` is due, and Admin → Utilities → Housekeeping will list it.

Scaffold the same shape with:

```
nails make:housekeeping:routine AnonymiseGuests
```

### Constants

`Base` reads these. Override only what you need.

| Constant | Purpose |
|---|---|
| `LABEL` | Short name in admin and `housekeeping:list`. Falls back to `DESCRIPTION`, then the class name. |
| `DESCRIPTION` | Longer explanation of what is removed. |
| `CRON_EXPRESSION` | When the orchestrator should run the routine (`@daily`, `*/15 * * * *`, `0 0 * * *`, …). Must be a valid cron expression; an empty value is a configuration error. |
| `ENVIRONMENT` | Restrict to named environments (`['PRODUCTION']`, or `[\Nails\Environment::ENV_PROD]`). Empty = all. |
| `ENABLED` | Hard off-switch. Defaults to `true`. |

`getKey()` is the fully-qualified class name. That is how `--routine` and the last-run stamp identify a routine.

### `Context` and `Result`

`execute()` receives a `Context` and must return a `Result`.

`Context` is the routine’s window onto the run:

| Method | Purpose |
|---|---|
| `isDryRun()` | `true` when the run was started with `--dry-run` (or the admin dry-run button). Do not mutate. |
| `log($sMessage)` | Append an audit line for this routine. |
| `writeln($sLine)` | Write to the console when one is attached (no-op in admin). |
| `logger()` | The housekeeping `Logger` service, if you need more than `log()`. |
| `output()` | The Symfony console output, or `null`. |

`Result` reports what happened. The orchestrator writes a `SUMMARY` line from it and records last-run metadata (except on dry-run).

```php
return Result::ok($iProcessed);                          // success
return Result::fail('Could not delete rows', $iProcessed, $iFailed);
```

Throwing is fine too: the orchestrator catches it, logs `ERROR`, and records a failed result.

## Official traits

For the common jobs you do not need to write `execute()` yourself. Official traits supply it, as long as you fill in the few abstract methods they ask for.

They all go through the `Deleter` service: select matching items, write each one to the audit log, then mutate (unless `--dry-run`).

A short `DeletesModelRows` routine looks like this:

```php
namespace App\Housekeeping;

use Nails\Common\Model\Base as ModelBase;
use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\DeletesModelRows;

class Enquiries extends Base
{
    use DeletesModelRows;

    const DESCRIPTION     = 'Deletes enquiries older than 730 days';
    const CRON_EXPRESSION = '@daily';

    protected function model(): ModelBase
    {
        return Factory::model('Enquiry', 'app');
    }

    protected function where(): array
    {
        $oNow = Factory::factory('DateTime');
        $oNow->modify('-730 days');

        return [
            ['created <', $oNow->format('Y-m-d H:i:s')],
        ];
    }
}
```

{% hint style="info" %}
Each trait defines `execute()`, so you cannot `use` more than one on the same class. For several models or mixed work, keep a custom `execute()` and call the `Deleter` service yourself.
{% endhint %}

{% content-ref url="traits/" %}
[traits](traits/)
{% endcontent-ref %}

{% content-ref url="traits/deletes-model-rows.md" %}
[deletes-model-rows.md](traits/deletes-model-rows.md)
{% endcontent-ref %}

{% content-ref url="traits/deletes-files.md" %}
[deletes-files.md](traits/deletes-files.md)
{% endcontent-ref %}

{% content-ref url="traits/archives-files.md" %}
[archives-files.md](traits/archives-files.md)
{% endcontent-ref %}

{% content-ref url="traits/truncates-table.md" %}
[truncates-table.md](traits/truncates-table.md)
{% endcontent-ref %}

## Console

```
nails housekeeping:list
nails housekeeping:run
nails housekeeping:run --dry-run
nails housekeeping:run --force
nails housekeeping:run --routine=App\\Housekeeping\\AnonymiseGuests
```

| Invocation | What runs |
|---|---|
| `housekeeping:run` | Enabled routines whose cron expression is due in the current environment |
| `housekeeping:run --force` | Every discovered routine, including disabled ones and those restricted to another environment |
| `housekeeping:run --routine=X` | That routine if it is enabled (due-ness is ignored) |
| `housekeeping:run --routine=X --force` | That routine even if it is disabled |

`--routine` accepts a fully-qualified class name or the short class name. `--dry-run` writes the same audit lines with `dry_run=true` and does not delete, unlink, gzip, or truncate. Last-run metadata is not updated on a dry-run.

## Audit log

Each run appends to `application/logs/housekeeping-YYYY-MM-DD.php`. The orchestrator always writes `START`, `SUMMARY`, and `FINISH`; the routine (or its trait) writes the lines in between.

```
INFO - 2026-09-15 03:00:01 [abc123] - Nails\Cdn\Housekeeping\Tokens --> START
INFO - 2026-09-15 03:00:01 [abc123] - Nails\Cdn\Housekeeping\Tokens --> DELETE id=42 token=... expires=2026-01-01 00:00:00
INFO - 2026-09-15 03:00:02 [abc123] - Nails\Cdn\Housekeeping\Tokens --> SUMMARY processed=150 failed=0 success=true dry_run=false duration_ms=812
INFO - 2026-09-15 03:00:02 [abc123] - Nails\Cdn\Housekeeping\Tokens --> FINISH
```

The session id in brackets groups one invocation.

Admin → Utilities → Housekeeping lists routines, runs or dry-runs them, and tails these log files.

## Events

Fired in the `nails/module-housekeeping` namespace:

| Event | When | Payload |
|---|---|---|
| `HOUSEKEEPING:START` | Runner begins | — |
| `HOUSEKEEPING:READY` | After discovery | Discovered routine objects |
| `HOUSEKEEPING:ROUTINE:BEFORE` | Immediately before `execute()` | The routine |
| `HOUSEKEEPING:ROUTINE:AFTER` | After a returned result | The routine and its `Result` |
| `HOUSEKEEPING:ROUTINE:ERROR` | After `execute()` throws | The routine and the exception |
| `HOUSEKEEPING:FINISH` | Runner ends | — |

## Configuration

Retention is config-only. There is no Admin UI for these values — set them in `.env` or via `Config::set()`. Unset keys fall back to the module default. Do not call `Config::default()` for these keys: that defines a constant, which makes `Config::isSet()` permanently true and would skip any remaining fallback.

| Key | Default | Module |
|---|---|---|
| `EMAIL_ARCHIVE_RETENTION_DAYS` | `0` (disabled) | [Email](../email.md#housekeeping) |
| `ADMIN_CHANGELOG_RETENTION_DAYS` | `0` (disabled) | [Admin](../admin/housekeeping.md) |
| `ADMIN_SESSION_RETENTION` | `3600` | [Admin](../admin/housekeeping.md) |
| `AUTH_USER_EVENT_RETENTION_DAYS` | `0` (disabled) | [Auth](../auth/housekeeping.md) |
| `AUTH_USER_IMPORT_STALE_CLAIM` | `900` | [Auth](../auth/housekeeping.md) |
| `AUTH_USER_IMPORT_DRAFT_TTL` | `86400` | [Auth](../auth/housekeeping.md) |
| `AUTH_USER_IMPORT_RETENTION` | `2592000` | [Auth](../auth/housekeeping.md) |
| `CDN_TRASH_RETENTION` | `180` (days) | [CDN](../cdn/housekeeping.md) |
| `LOG_ARCHIVE` | `14` (days) | [ArchivesFiles](traits/archives-files.md) |
| `LOG_RETENTION` | `180` (days) | [DeletesFiles](traits/deletes-files.md) |
| `GEO_IP_CACHE_PERIOD` | `3600` | [Geo-IP](../other/geo-ip/housekeeping.md) |
| `GEO_CODE_CACHE_PERIOD` | `15552000` (180 days) | [Geo-code](../other/geo-code/housekeeping.md) |

The Email archive previously used the `retention_period` app setting. That value is still honoured when `EMAIL_ARCHIVE_RETENTION_DAYS` is not set, but using it emits a deprecation notice. Config always wins when both are present.

## First-party routines

Official modules ship their own routines under `src/Housekeeping/`. The orchestrator discovers them automatically. What each one removes lives in that module's docs:

- [Admin](../admin/housekeeping.md) — sessions, expired data exports, changelog
- [Auth](../auth/housekeeping.md) — user imports, API access tokens, legacy 2FA tokens, user events
- [CDN](../cdn/housekeeping.md) — expired tokens, trash
- [Email](../email.md#housekeeping) — archive
- [Geo-code](../other/geo-code/housekeeping.md) — cache
- [Geo-IP](../other/geo-ip/housekeeping.md) — cache
- [Multi-Factor Auth](../multi-factor-auth/housekeeping.md) — challenge tokens

The housekeeping module's own log routines are documented with the traits they use:

- `Nails\Housekeeping\Housekeeping\LogFilesArchive` — [ArchivesFiles](traits/archives-files.md); compresses `*.php` older than `LOG_ARCHIVE` days (default 14). Set `LOG_ARCHIVE=0` to keep logs uncompressed until they are purged.
- `Nails\Housekeeping\Housekeeping\LogFiles` — [DeletesFiles](traits/deletes-files.md); deletes `*.php` and `*.php.gz` older than `LOG_RETENTION` days (default 180).

Together they take log files **hot** (uncompressed, easy to parse) → **cold** (gzipped, kept in case) → **purged**. The two thresholds are independent: a file past retention is deleted even if it was never archived, and `LOG_ARCHIVE=0` leaves everything hot until purge. Both run at midnight; discovery order runs the purge first so files already past retention are not compressed only to be deleted the next day.

