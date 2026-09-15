---
description: >-
  This module provides a stable API for defining cleanup routines that are
  automatically discovered, scheduled, and audited to a log file.
---

# Housekeeping

The housekeeping module is an orchestrator for cleanup work: truncating or deleting stale rows, removing files from disk, and any other residue a module or app needs to tidy up.

Routines are discovered the same way cron tasks are: implement an interface, put the class in the right namespace, and it is picked up automatically. Unlike cron tasks, a housekeeping routine **is** the work — there is no separate console command to bind.

A single cron task (`housekeeping:run`, every minute) asks the orchestrator which routines are due and executes them in-process so the audit trail stays in one place.

## Defining a routine

Drop a class under `src/Housekeeping/` that extends `Nails\Housekeeping\Routine\Base`. The app, any module, or housekeeping itself can contribute routines.

```php
namespace App\Housekeeping;

use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\DeletesModelRows;

class Enquiries extends Base
{
    const DESCRIPTION     = 'Deletes enquiries older than 730 days';
    const CRON_EXPRESSION = '@daily';

    protected function model(): \Nails\Common\Model\Base
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

Scaffold an app routine with:

```
nails make:housekeeping:routine Enquiries
```

### Constants

| Constant | Purpose |
|---|---|
| `LABEL` | Short name in admin and `housekeeping:list`. Falls back to `DESCRIPTION`, then the class name. |
| `DESCRIPTION` | Longer explanation of what is removed. |
| `CRON_EXPRESSION` | When the orchestrator should run the routine (`@daily`, `*/15 * * * *`, etc.). |
| `ENVIRONMENT` | Restrict to named environments; empty = all. |
| `ENABLED` | Hard off-switch. |

Custom jobs implement `execute(Context $oContext): Result` instead of using a trait. `Context` exposes the audit logger, console output, and `isDryRun()`.

## Official traits

Use these when the work is a common pattern. They select matching items, write each one to the audit log, then mutate (unless `--dry-run`).

| Trait | What it does |
|---|---|
| `DeletesModelRows` | Batch-delete model rows matching `where()`. Override `auditColumns()`, `batchSize()`, `optimizeAfter()`. Refuses to run with an empty `where()`. |
| `DeletesFiles` | Unlink files under `directory()` matching `pattern()` older than `olderThanDays()`. |
| `TruncatesTable` | Log the current row count, then `TRUNCATE`. Cannot snapshot every row. |

Multi-table jobs (several models, same condition) should call `Factory::service('Deleter', \Nails\Housekeeping\Constants::MODULE_SLUG)` from a custom `execute()` rather than stacking traits.

## Console

```
nails housekeeping:list
nails housekeeping:run
nails housekeeping:run --dry-run
nails housekeeping:run --force
nails housekeeping:run --routine=App\\Housekeeping\\Enquiries
```

`--force` ignores the cron expression (still skips disabled routines unless combined with a named `--routine`). `--dry-run` writes the same audit lines with `dry_run=true` and does not delete.

## Audit log

Each run appends to `application/logs/housekeeping-YYYY-MM-DD.php`:

```
INFO - 2026-09-15 03:00:01 [abc123] - Nails\Cdn\Housekeeping\Tokens --> START
INFO - 2026-09-15 03:00:01 [abc123] - Nails\Cdn\Housekeeping\Tokens --> DELETE id=42 token=... expires=2026-01-01 00:00:00
INFO - 2026-09-15 03:00:02 [abc123] - Nails\Cdn\Housekeeping\Tokens --> SUMMARY processed=150 failed=0 success=true dry_run=false duration_ms=812
INFO - 2026-09-15 03:00:02 [abc123] - Nails\Cdn\Housekeeping\Tokens --> FINISH
```

The session id in brackets groups one invocation. Admin → Utilities → Housekeeping can list routines, run or dry-run them, and tail these log files.

## Events

Fired in the `nails/module-housekeeping` namespace:

- `HOUSEKEEPING:START`
- `HOUSEKEEPING:READY` — discovered routine objects
- `HOUSEKEEPING:ROUTINE:BEFORE` / `AFTER` / `ERROR`
- `HOUSEKEEPING:FINISH`
