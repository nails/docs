---
description: >-
  Official traits that supply execute() for common cleanup jobs: deleting model
  rows, unlinking files, and truncating tables.
---

# Official traits

A housekeeping routine only has to implement `execute()`. For the three jobs that come up constantly, official traits provide that method so the class can describe *what* to clean rather than *how*.

Use a trait when the work matches one of these patterns. Keep a custom `execute()` when it does not.

| Trait | Use when | You provide |
|---|---|---|
| [`DeletesModelRows`](deletes-model-rows.md) | Stale rows in one model should be deleted | `model()`, `where()` |
| [`DeletesFiles`](deletes-files.md) | Files on disk matching a name pattern should be unlinked | `directory()` |
| [`TruncatesTable`](truncates-table.md) | A whole table should be emptied | `model()` |

All three:

* Honour `--dry-run` (audit lines are written; nothing is mutated).
* Go through `Factory::service('Deleter', \Nails\Housekeeping\Constants::MODULE_SLUG)`.
* Define `execute()`, so you cannot `use` more than one on the same class.

{% content-ref url="deletes-model-rows.md" %}
[deletes-model-rows.md](deletes-model-rows.md)
{% endcontent-ref %}

{% content-ref url="deletes-files.md" %}
[deletes-files.md](deletes-files.md)
{% endcontent-ref %}

{% content-ref url="truncates-table.md" %}
[truncates-table.md](truncates-table.md)
{% endcontent-ref %}

## When not to use a trait

Write `execute()` yourself when:

* More than one model or directory is involved.
* The work is not a delete / unlink / truncate (archive, anonymise, rebuild, …).
* You need to branch on data that the trait does not expose.

Call the `Deleter` service from that custom `execute()` for the parts that *are* a common job:

```php
namespace App\Housekeeping;

use Nails\Factory;
use Nails\Housekeeping\Constants;
use Nails\Housekeeping\Routine\Base;
use Nails\Housekeeping\Routine\Context;
use Nails\Housekeeping\Routine\Result;
use Nails\Housekeeping\Service\Deleter;

class Enquiries extends Base
{
    const DESCRIPTION     = 'Deletes enquiries and their notes older than 730 days';
    const CRON_EXPRESSION = '@daily';

    public function execute(Context $oContext): Result
    {
        /** @var Deleter $oDeleter */
        $oDeleter = Factory::service('Deleter', Constants::MODULE_SLUG);

        $oNow = Factory::factory('DateTime');
        $oNow->modify('-730 days');
        $aWhere = [
            ['created <', $oNow->format('Y-m-d H:i:s')],
        ];

        $oEnquiries = $oDeleter->deleteRows(
            $oContext,
            Factory::model('Enquiry', 'app'),
            $aWhere
        );
        if (!$oEnquiries->isSuccess()) {
            return $oEnquiries;
        }

        $oNotes = $oDeleter->deleteRows(
            $oContext,
            Factory::model('EnquiryNote', 'app'),
            $aWhere
        );

        return Result::ok($oEnquiries->getProcessed() + $oNotes->getProcessed());
    }
}
```
