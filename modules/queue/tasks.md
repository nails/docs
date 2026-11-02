---
description: >-
  Write a task, push a job with a payload, and how failed jobs are retried.
---

# Tasks

A task is the work. It implements `Nails\Queue\Interface\Task` and has two methods: how many times a failure may be retried, and `run()`.

```php
namespace App\Queue\Tasks\Reports;

use App\Model;
use Nails\Factory;
use Nails\Queue\Interface\Data;
use Nails\Queue\Interface\Task;

class Generate implements Task
{
    public static function getMaxRetries(): int
    {
        return 3;
    }

    public function run(Data $data): void
    {
        $payload = $data->get();

        /** @var Model\Report $model */
        $model  = Factory::model('Report', 'app');
        $report = $model->getById($payload->report_id);

        //  Do the work. Throw to fail the attempt.
    }
}
```

`run()` returns nothing. Returning normally completes the job. Throwing anything, including an `Error`, fails the attempt.

A worker builds the task with `new`, so a task cannot have required constructor arguments. Everything the task needs comes from the payload.

## Payload

The payload is wrapped in a `Data` object. Build one with the `Data` factory:

```php
use Nails\Factory;
use Nails\Queue\Constants;

$data = Factory::factory('Data', Constants::MODULE_SLUG, (object) [
    'report_id' => 123,
]);
```

It accepts an array, string, int, float, bool, `stdClass`, or `null`. It is stored as JSON, so everything in it must survive `json_encode()`.

The worker reads the JSON back with `json_decode()`, so what `get()` returns in `run()` is not always what went in:

| Pushed | `get()` returns |
|---|---|
| `stdClass` | `stdClass` |
| Associative array | `stdClass` |
| List array | List array |
| Scalar or `null` | The same scalar or `null` |

Push an object and read properties with `->`, as in the example above.

Keep payloads small. Pass an ID and load the row in `run()`, not the row itself. The job may wait, and the row may change before it runs.

### Your own Data class

A subclass of `Nails\Queue\Factory\Data` is a good place to validate a payload before it is stored:

```php
namespace App\Queue\Data\Reports;

use InvalidArgumentException;
use Nails\Queue\Factory\Data;

class Generate extends Data
{
    public static function make(int $reportId): self
    {
        if ($reportId <= 0) {
            throw new InvalidArgumentException('A report ID is required');
        }

        return new self((object) ['report_id' => $reportId]);
    }
}
```

The class is only used to build the payload. A worker always reads a job back through the `Data` factory, so the task receives the plain `Data` class and not your subclass. To change what the worker builds, define `App\Queue\Factory\Data`.

## Pushing a job

Resolve the Manager and call `push()`:

```php
use App\Queue\Data;
use App\Queue\Tasks;
use Nails\Factory;
use Nails\Queue\Constants;
use Nails\Queue\Service\Manager;

/** @var Manager $manager */
$manager = Factory::service('Manager', Constants::MODULE_SLUG);

$job = $manager->push(
    task: new Tasks\Reports\Generate(),
    data: Data\Reports\Generate::make(123),
);
```

| Argument | Default | Effect |
|---|---|---|
| `task` | Required | The task to run. Only its class name is stored. |
| `data` | Required | The payload. |
| `queue` | `null` | A [queue](queues.md) instance, alias, or class name. `null` means `default`. |
| `availableAt` | Now | A `DateTimeInterface`. No worker runs the job before then. |

`push()` returns the stored `Nails\Queue\Resource\Job`. A job with a future `availableAt` stays `PENDING` and is counted as _scheduled_ in [admin](admin.md).

```php
$manager->push(
    task: new Tasks\Reports\Generate(),
    data: Data\Reports\Generate::make(123),
    queue: 'priority',
    availableAt: new DateTimeImmutable('+10 minutes'),
);
```

{% hint style="info" %}
Pushing does not start anything. If no [worker](workers.md) is running for the queue the job waits. A job pushed inside a database [transaction](../../core-services/database/transactions.md) is invisible to workers until the transaction commits, so a job that depends on the transaction's changes never runs ahead of them.
{% endhint %}

### Order

A worker takes the lowest job `id` among the jobs that are `PENDING` and due. Jobs run first in, first out. A retried job keeps its `id`, so it goes back to its place once its delay has passed. A worker listening to several queues takes the lowest `id` across all of them. See [Queues](queues.md#priority).

## Retries

`getMaxRetries()` is the number of times a failed task is tried again. A task with `3` runs up to four times: the first run and three retries. `0` means the first failure is final.

After a failure with retries left the job goes back to `PENDING`, its `attempts` count goes up by one, and `available_at` moves into the future. The delay doubles each time, starting at five seconds, is capped at five minutes, and has up to 20% random jitter either way.

| Retry | Wait before it, roughly |
|---|---|
| 1 | 5 s |
| 2 | 10 s |
| 3 | 20 s |
| 4 | 40 s |
| 5 | 80 s |
| 6 | 160 s |
| 7 and later | 300 s |

When the retries are used up the job is `FAILED`. A failed job is never run again unless someone [retries it from admin](admin.md#failed-jobs) or calls `retryJob()`. It is deleted after `QUEUE_JOB_ROTATE_FAILED_DAYS`.

Every failure is appended to the job's `errors`, so a failed job carries the whole history. Each entry has the exception class, message, code, file, line, a stack trace cut to 2,000 characters, and when it happened.

### Write tasks that can run twice

A job can run more than once even when it succeeded the first time. If a worker is killed mid-job, or a job outlasts `QUEUE_WORKER_HEARTBEAT_STALE`, [`queue:clean`](workers.md#maintenance) puts it back in `PENDING` and another worker runs it from the start. Make `run()` safe to repeat. Check whether the work is already done before doing it, and prefer writes that set a value to ones that add to it.

## Trying it out

`Nails\Queue\Tasks\DoNothing` is a task that does nothing and never retries. Use it to check that workers are picking up jobs:

```php
$manager->push(
    task: new \Nails\Queue\Tasks\DoNothing(),
    data: Factory::factory('Data', Constants::MODULE_SLUG, null),
);
```

The job should go `PENDING`, `RUNNING`, `COMPLETE` within a second or so of a worker starting. Watch the worker output or the [overview](admin.md).
