---
description: >-
  Batch-delete model rows matching a where() clause, writing each row to the
  housekeeping audit log first.
---

# DeletesModelRows

Use `Nails\Housekeeping\Traits\DeletesModelRows` when a routine’s job is “delete rows from this model that match this condition.”

The trait selects matching rows in batches, writes each one to the audit log, then deletes them (unless `--dry-run`). It refuses to run with an empty `where()` so a misconfigured routine cannot empty a table.

The CDN module’s `Nails\Cdn\Housekeeping\Tokens` routine is a production example of this trait.

## Example

```php
namespace App\Housekeeping;

use Nails\Common\Model\Base as ModelBase;
use Nails\Factory;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Traits\DeletesModelRows;

class Enquiries extends Base
{
    use DeletesModelRows;

    const LABEL           = 'Old enquiries';
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

## Required methods

### `model(): \Nails\Common\Model\Base`

The model whose rows should be deleted. Use `Factory::model()` so the app can still overload the class.

### `where(): array`

Conditions identifying the rows to delete, in the same shape `Model::getAll()` accepts:

```php
return [
    ['created <', $sCutoff],
    ['status', 'draft'],
];
```

Must not be empty. An empty array throws `Nails\Housekeeping\Exception\HousekeepingException` rather than deleting the whole table.

## Optional methods

| Method | Default | Purpose |
|---|---|---|
| `auditColumns()` | `['id']` | Extra columns written to each `DELETE` audit line. `id` is always included. |
| `batchSize()` | `200` | Rows fetched (and deleted) per iteration. Must be at least `1`. |
| `optimizeAfter()` | `false` | When `true`, runs `OPTIMIZE TABLE` after a successful non-dry-run that deleted at least one row. |

```php
protected function auditColumns(): array
{
    return ['id', 'email', 'created'];
}

protected function batchSize(): int
{
    return 500;
}

protected function optimizeAfter(): bool
{
    return true;
}
```

## Behaviour

1. Log `TABLE {table} batch_size={n} dry_run={true|false}`.
2. Fetch a page of matching rows, ordered by id, selecting only the audit columns.
3. Write a `DELETE id=… col=…` line (and a console line) for each row.
4. If this is not a dry-run, `deleteMany()` those ids, then fetch page 1 again (the previous rows are gone). On dry-run, advance the page instead.
5. Repeat until a page comes back empty.
6. Optionally `OPTIMIZE TABLE`.
7. Return `Result::ok($iProcessed)`.

If `deleteMany()` fails, the trait returns `Result::fail(…)` immediately with the number processed so far and the size of the failed batch.

{% hint style="warning" %}
`where()` is the only thing standing between the routine and the rest of the table. Keep it specific (a date cutoff, an expiry column, a status) and cover it with a dry-run before enabling the schedule.
{% endhint %}

## Audit log

```
INFO - 2026-09-15 03:00:01 [abc123] - App\Housekeeping\Enquiries --> START
INFO - 2026-09-15 03:00:01 [abc123] - App\Housekeeping\Enquiries --> TABLE nails_app_enquiry batch_size=200 dry_run=false
INFO - 2026-09-15 03:00:01 [abc123] - App\Housekeeping\Enquiries --> DELETE id=42 email=jane@example.com created=2023-01-04 09:12:00
INFO - 2026-09-15 03:00:02 [abc123] - App\Housekeeping\Enquiries --> SUMMARY processed=150 failed=0 success=true dry_run=false duration_ms=812
INFO - 2026-09-15 03:00:02 [abc123] - App\Housekeeping\Enquiries --> FINISH
```

## Several models

The trait binds `execute()` to a single model. For the same condition across several tables, do not stack traits — call `Deleter::deleteRows()` from a custom `execute()` instead. See [When not to use a trait](README.md#when-not-to-use-a-trait).
