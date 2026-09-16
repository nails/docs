---
description: >-
  Gzip files under a directory whose names match a pattern and that are older
  than an archive threshold.
---

# ArchivesFiles

Use `Nails\Housekeeping\Traits\ArchivesFiles` when a routine’s job is “compress files in this directory that look like this and are old enough.” Typical cases are log files and generated exports that should stay easy to grep while they are still hot, then shrink on disk until a later retention pass deletes them.

The trait walks the directory recursively, selects files whose *filename* matches `fnmatch(pattern())`, skips anything newer than `olderThanDays()` and anything that already has the archive suffix, gzip-compresses the rest next to the original, copies the original mtime onto the archive, and unlinks the uncompressed file (unless `--dry-run`).

The housekeeping module’s own `Nails\Housekeeping\Housekeeping\LogFilesArchive` routine is a production example of this trait: it compresses `application/logs/*.php` older than `LOG_ARCHIVE` days. Pair it with [`DeletesFiles`](deletes-files.md) (the module’s `LogFiles` routine) so files go **hot → cold → purged** on independent thresholds.

## Example

```php
namespace App\Housekeeping;

use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\ArchivesFiles;

class ExpiredExportsArchive extends Base
{
    use ArchivesFiles;

    const LABEL           = 'Export archives';
    const DESCRIPTION     = 'Compresses generated export files older than 14 days';
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

A retention-aware log compressor (the pattern the module ships):

```php
namespace Nails\Housekeeping\Housekeeping;

use Nails\Common\Service\Logger as CommonLogger;
use Nails\Config;
use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Routine\Context;
use Nails\Housekeeping\Routine\Result;
use Nails\Housekeeping\Traits\ArchivesFiles;

class LogFilesArchive extends Base
{
    use ArchivesFiles {
        execute as archiveFiles;
    }

    const LABEL                = 'Log file archives';
    const DESCRIPTION          = 'Compresses log files older than the configured archive threshold';
    const CRON_EXPRESSION      = '0 0 * * *';
    const CONFIG_ARCHIVE_DAYS  = 'LOG_ARCHIVE';
    const DEFAULT_ARCHIVE_DAYS = 14;

    protected function directory(): string
    {
        /** @var CommonLogger $oLogger */
        $oLogger = Factory::service('Logger');
        return $oLogger->getDir();
    }

    protected function pattern(): string
    {
        return '*.php';
    }

    protected function olderThanDays(): int
    {
        return (int) Config::get(static::CONFIG_ARCHIVE_DAYS, static::DEFAULT_ARCHIVE_DAYS);
    }

    public function execute(Context $oContext): Result
    {
        $iDays = $this->olderThanDays();
        if ($iDays < 1) {
            $oContext
                ->writeln('Log archive disabled')
                ->log('DISABLED ' . static::CONFIG_ARCHIVE_DAYS . '=0');

            return Result::ok(0, 'Log archive disabled');
        }

        return $this->archiveFiles($oContext);
    }
}
```

`LOG_ARCHIVE=0` on that routine disables compression so logs stay hot until `LOG_RETENTION` purges them. The trait itself treats `0` as “every matching file,” the same as [`DeletesFiles`](deletes-files.md).

## Required methods

### `directory(): string`

Absolute path of the directory to scan. A missing directory throws `Nails\Housekeeping\Exception\HousekeepingException` rather than failing silently.

Trailing slashes are normalised; you can return either form.

## Optional methods

| Method | Default | Purpose |
|---|---|---|
| `pattern()` | `'*.php'` | `fnmatch()` pattern applied to the **filename**, not the full path. May also return an array of patterns. |
| `olderThanDays()` | `14` | Only files whose mtime is older than this many days are compressed. `0` means every matching file, regardless of age. |
| `archiveSuffix()` | `'.gz'` | Appended to the original path. Files that already end with this suffix are skipped. |

The default pattern is aimed at PHP log files. Override it whenever you are not archiving logs.

## Behaviour

1. Log `DIRECTORY {dir} pattern={pattern} older_than_days={n} suffix=.gz dry_run={true|false}`.
2. Recurse the directory (`CHILD_FIRST`, skipping `.` and `..`). Directories themselves are never touched.
3. Skip files whose name already ends with `archiveSuffix()`.
4. For each remaining file whose name matches `pattern()` and whose age exceeds `olderThanDays()`:
   * Log `ARCHIVE {absolute path} -> {absolute path}.gz`.
   * If this is not a dry-run, gzip the file, copy the original mode and mtime onto the archive, then `unlink()` the original.
   * If `{path}.gz` already exists from an interrupted previous run, the original is unlinked and the existing archive is left in place.
   * A failed compress or unlink is counted and logged as `ERROR failed to compress …` / `ERROR failed to unlink …`.
5. Return `Result::ok($iProcessed)`, or `Result::fail(…)` if any file failed.

Age is computed from `DateTime::diff()` on the file’s mtime, so a file modified *today* is never older than 0 days. With the default of 14, a file must be more than 14 days old to be compressed.

Preserving mtime matters: a later [`DeletesFiles`](deletes-files.md) pass still sees the file as 15 days old (or whatever it was), not as created the moment it was gzipped.

The zlib extension is required. It is listed as `ext-zlib` on `nails/module-housekeeping`.

{% hint style="info" %}
`pattern()` is matched with `fnmatch()` against `SplFileInfo::getFilename()` only. `*.php` matches `log-2026-01-01.php` in any subdirectory; it will not match `log-2026-01-01.php.gz`. There is no `**` glob — recursion is always on, and the pattern never sees directory components.
{% endhint %}

## Audit log

```
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFilesArchive --> START
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFilesArchive --> DIRECTORY /var/app/application/logs/ pattern=*.php older_than_days=14 suffix=.gz dry_run=false
INFO - 2026-09-15 00:00:01 [abc123] - Nails\Housekeeping\Housekeeping\LogFilesArchive --> ARCHIVE /var/app/application/logs/log-2026-08-01.php -> /var/app/application/logs/log-2026-08-01.php.gz
INFO - 2026-09-15 00:00:02 [abc123] - Nails\Housekeeping\Housekeeping\LogFilesArchive --> SUMMARY processed=12 failed=0 success=true dry_run=false duration_ms=40
INFO - 2026-09-15 00:00:02 [abc123] - Nails\Housekeeping\Housekeeping\LogFilesArchive --> FINISH
```
