---
description: >-
  Unlink files under a directory whose names match a pattern and that are older
  than a retention period.
---

# DeletesFiles

Use `Nails\Housekeeping\Traits\DeletesFiles` when a routine’s job is “remove files from this directory that look like this and are old enough.”

The trait walks the directory recursively, selects files whose *filename* matches `fnmatch(pattern())`, skips anything newer than `olderThanDays()`, and unlinks the rest (unless `--dry-run`).

The housekeeping module’s own `Nails\Housekeeping\Housekeeping\LogFiles` routine is a production example of this trait: it cleans `application/logs/*.php` and `*.php.gz` older than `LOG_RETENTION` days. Pair it with [`ArchivesFiles`](archives-files.md) (the module’s `LogFilesArchive` routine) so uncompressed files are gzipped first and only later unlinked.

## Example

```php
namespace App\Housekeeping;

use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\DeletesFiles;

class ExpiredExports extends Base
{
    use DeletesFiles;

    const LABEL           = 'Expired exports';
    const DESCRIPTION     = 'Deletes generated export files older than 14 days';
    const CRON_EXPRESSION = '@daily';

    protected function directory(): string
    {
        return NAILS_APP_PATH . 'application/uploads/exports/';
    }

    protected function pattern(): string
    {
        return '*';
    }

    protected function olderThanDays(): int
    {
        return 14;
    }
}
```

A retention-aware log cleaner (the pattern the module ships):

```php
namespace Nails\Housekeeping\Housekeeping;

use Nails\Common\Service\Logger as CommonLogger;
use Nails\Config;
use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\DeletesFiles;

class LogFiles extends Base
{
    use DeletesFiles;

    const LABEL                  = 'Log files';
    const DESCRIPTION            = 'Deletes log files older than the configured retention period, including compressed archives';
    const CRON_EXPRESSION        = '0 0 * * *';
    const CONFIG_RETENTION_DAYS  = 'LOG_RETENTION';
    const DEFAULT_RETENTION_DAYS = 180;

    protected function directory(): string
    {
        /** @var CommonLogger $oLogger */
        $oLogger = Factory::service('Logger');
        return $oLogger->getDir();
    }

    /**
     * @return string|string[]
     */
    protected function pattern(): string|array
    {
        return ['*.php', '*.php.gz'];
    }

    protected function olderThanDays(): int
    {
        return (int) Config::get(static::CONFIG_RETENTION_DAYS, static::DEFAULT_RETENTION_DAYS);
    }
}
```

## Required methods

### `directory(): string`

Absolute path of the directory to scan. A missing directory throws `Nails\Housekeeping\Exception\HousekeepingException` rather than failing silently.

Trailing slashes are normalised; you can return either form.

## Optional methods

| Method | Default | Purpose |
|---|---|---|
| `pattern()` | `'*.php'` | `fnmatch()` pattern applied to the **filename**, not the full path. May also return an array of patterns. |
| `olderThanDays()` | `180` | Only files whose mtime is older than this many days are removed. `0` means every matching file, regardless of age. |

The default pattern is aimed at PHP log files. Override it whenever you are not cleaning logs.

## Behaviour

1. Log `DIRECTORY {dir} pattern={pattern} older_than_days={n} dry_run={true|false}`.
2. Recurse the directory (`CHILD_FIRST`, skipping `.` and `..`). Directories themselves are never removed.
3. For each file whose name matches `pattern()` and whose age exceeds `olderThanDays()`:
   * Log `UNLINK {absolute path}`.
   * If this is not a dry-run, `unlink()` it. A failed unlink is counted and logged as `ERROR failed to unlink …`.
4. Return `Result::ok($iProcessed)`, or `Result::fail(…)` if any unlink failed.

Age is computed from `DateTime::diff()` on the file’s mtime, so a file modified *today* is never older than 0 days. With the default of 180, a file must be more than 180 days old to be removed.

{% hint style="info" %}
`pattern()` is matched with `fnmatch()` against `SplFileInfo::getFilename()` only. `*.php` matches `log-2026-01-01.php` in any subdirectory; it will not match `archive.log`. There is no `**` glob — recursion is always on, and the pattern never sees directory components.
{% endhint %}

## Audit log

```
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFiles --> START
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFiles --> DIRECTORY /var/app/application/logs/ pattern=*.php,*.php.gz older_than_days=180 dry_run=false
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFiles --> UNLINK /var/app/application/logs/log-2025-01-01.php.gz
INFO - 2026-09-15 00:00:02 [abc123] - Nails\Housekeeping\Housekeeping\LogFiles --> SUMMARY processed=12 failed=0 success=true dry_run=false duration_ms=40
INFO - 2026-09-15 00:00:02 [abc123] - Nails\Housekeeping\Housekeeping\LogFiles --> FINISH
```
