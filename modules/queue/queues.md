---
description: >-
  The built-in queues, how to add your own, and how aliases and discovery
  work.
---

# Queues

A queue is a name that jobs are filed under and workers listen to. It is a class that implements `Nails\Queue\Interface\Queue`. The job stores the class name.

Most apps only need the built-in `default` queue. Add a queue when a kind of work needs its own workers: a slow report that should not hold up password-reset emails, or work that needs setup before the first job.

## Built-in queues

| Class | Alias |
|---|---|
| `Nails\Queue\Queue\Queues\DefaultQueue` | `default` |
| `Nails\Queue\Queue\Queues\PriorityQueue` | `priority` |

`push()` uses `default` when no queue is given, and `nails queue:work` listens to `default` when no `--queue` is given.

### Priority

`priority` is a separate queue in name only. The module gives it no extra weight, no head start, and no worker. Nothing starts a worker for it, and a job pushed to it waits until the app runs one.

{% hint style="warning" %}
**The app has to run a separate worker process for `priority`.** One worker that listens to both `default` and `priority` takes the oldest job across the two, exactly as if they were one queue, so a priority job still waits behind everything pushed to `default` before it.
{% endhint %}

Give each queue its own process, so priority work never waits behind a slow `default` job:

```bash
nails queue:work --queue=default
nails queue:work --queue=priority
```

Keep both running with a [process manager](process-managers.md#a-worker-per-queue).

## Defining a queue

A queue has two static hooks. Both may do nothing.

```php
namespace App\Queue\Queues;

use Nails\Queue\Interface\Queue;
use Nails\Queue\Resource\Worker;

class Reports implements Queue
{
    public static function setup(Worker $worker): void
    {
        //  Runs once, when a worker for this queue starts
    }

    public static function refresh(Worker $worker): void
    {
        //  Runs again every QUEUE_WORKER_REFRESH_INTERVAL seconds
    }
}
```

| Hook | When |
|---|---|
| `setup()` | Once as the worker starts, before it takes any job. |
| `refresh()` | Periodically while the worker is running. Use it for work that goes stale, such as refreshing a token or clearing temporary files. |

Both receive the `Nails\Queue\Resource\Worker` that is running. An exception from either is logged and the worker carries on.

`refresh()` is checked once per pass through the worker loop, so a long job delays it until the job finishes. An idle worker passes through the loop at least every five seconds (see [Workers](workers.md#what-a-worker-does)), so the hook runs close to on time.

The hooks are static because the job records only the class name. A queue is built with `new` and no arguments.

## Using a queue

Pass the class name or an instance when pushing a job, and give the class name to the worker:

```php
$manager->push(
    task: new Tasks\Reports\Generate(),
    data: Data\Reports\Generate::make(123),
    queue: \App\Queue\Queues\Reports::class,
);
```

```bash
nails queue:work --queue='App\Queue\Queues\Reports'
```

Quote the class name in the shell so the backslashes survive.

## Aliases

An alias is a short name for a queue: `default` and `priority` are aliases. `push()` and `--queue` accept an alias anywhere they accept a class name, and aliases are not case sensitive. A name resolves in this order:

1. A queue instance is used as it is.
2. A registered alias.
3. A class name.

Anything else throws an `InvalidArgumentException`.

Aliases are not stored. Each PHP process starts with just `default` and `priority`, so a custom alias has to be registered in every process that uses it: the web request that pushes the job, and the worker. The place that runs for both is the Manager. Override it with `App\Queue\Service\Manager`:

```php
namespace App\Queue\Service;

use App\Queue\Queues;
use Nails\Common\Service\Database;
use Nails\Queue\Model;
use Nails\Queue\Service\Manager as BaseManager;

class Manager extends BaseManager
{
    public function __construct(Database $database, Model\Worker $workerModel, Model\Job $jobModel)
    {
        parent::__construct($database, $workerModel, $jobModel);
        $this->addAlias('reports', new Queues\Reports());
    }
}
```

Then `queue: 'reports'` and `--queue=reports` both work. If that is more setup than you want, use the class name.

## Discovery

The Manager finds queues by looking in the `Queue\Queues` namespace of every installed component: the app (`App\Queue\Queues`), `Nails\Queue\Queue\Queues` in this module, and any other module that ships one. A class is discovered when it implements `Queue` and can be instantiated.

Discovery is what makes a queue visible to [admin](admin.md). The overview lists discovered queues, and the dashboard alert warns when a discovered queue has no worker. A queue outside that namespace still works for pushing and running jobs, but admin does not list it.

{% hint style="warning" %}
`DefaultQueue` and `PriorityQueue` are discovered, so the dashboard warns while `priority` has no worker, even if you never push to it. Run a [separate `priority` worker](#priority) to clear the warning.
{% endhint %}

A worker counts for a queue when its heartbeat is recent and it was started with that queue's class. A worker started with `--queue=priority` counts for `PriorityQueue` whichever way the queue was spelled.
