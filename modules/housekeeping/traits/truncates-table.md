---
description: >-
  Log a table’s current row count, then TRUNCATE it. Use when the whole table
  is disposable and per-row audit is not required.
---

# TruncatesTable

Use `Nails\Housekeeping\Traits\TruncatesTable` when a routine’s job is “empty this table.” Typical cases are scratch tables, import staging, and caches that are cheaper to rebuild than to prune.

The trait logs the current row count, then calls `$oModel->truncate()` (unless `--dry-run`). It cannot snapshot every row — if you need a `DELETE` line per record, use [`DeletesModelRows`](deletes-model-rows.md) instead.

## Example

```php
namespace App\Housekeeping;

use Nails\Common\Model\Base as ModelBase;
use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\TruncatesTable;

class ImportStaging extends Base
{
    use TruncatesTable;

    const LABEL           = 'Import staging';
    const DESCRIPTION     = 'Truncates the nightly import staging table';
    const CRON_EXPRESSION = '0 4 * * *';

    protected function model(): ModelBase
    {
        return Factory::model('ImportStaging', 'app');
    }
}
```

## Required methods

### `model(): \Nails\Common\Model\Base`

The model whose table should be truncated. Use `Factory::model()` so the app can still overload the class.

There is no `where()`. Truncate means the whole table.

## Behaviour

1. `countAll()` on the model.
2. Log `TRUNCATE {table} rows={n} dry_run={true|false}` and a console line with the same numbers.
3. If this is not a dry-run, `$oModel->truncate()`.
4. Return `Result::ok($iCount)` — `processed` is the row count *before* the truncate, so a dry-run and a live run report the same number.

{% hint style="warning" %}
`TRUNCATE` is not a filtered delete. There is no per-row audit trail, and the operation is not recoverable from the housekeeping log. Prefer [`DeletesModelRows`](deletes-model-rows.md) unless the table is genuinely disposable.
{% endhint %}

## Audit log

```
INFO - 2026-09-15 04:00:01 [abc123] - App\Housekeeping\ImportStaging --> START
INFO - 2026-09-15 04:00:01 [abc123] - App\Housekeeping\ImportStaging --> TRUNCATE nails_app_import_staging rows=18420 dry_run=false
INFO - 2026-09-15 04:00:01 [abc123] - App\Housekeeping\ImportStaging --> SUMMARY processed=18420 failed=0 success=true dry_run=false duration_ms=55
INFO - 2026-09-15 04:00:01 [abc123] - App\Housekeeping\ImportStaging --> FINISH
```

On a dry-run the `TRUNCATE` line is still written (`dry_run=true`) and `processed` is still the current count, but the table is left untouched.
