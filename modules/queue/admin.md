---
description: >-
  The queue overview screen, the dashboard warning for unworked queues, and
  the API behind them.
---

# Admin

**Queue → Overview** shows what the queue is doing now: how many jobs are in each state, which workers are alive, and which jobs failed. It needs `nails/module-admin`.

The screen loads its data from the [API](#api) and asks again every ten seconds, so it can stay open on a second monitor.

## Permission

| Permission | Class | Allows |
|---|---|---|
| Can view queue overview | `Nails\Queue\Admin\Permission\Overview\View` | See the screen and its data, and retry failed jobs |

Grant it on a user group, as described in [User Permissions](../admin/user-permissions.md). The one permission covers retrying a failed job as well as looking at it.

## Overview

### Totals

A row of tiles across the top:

| Tile | Meaning |
|---|---|
| Workers (total, active, stale) | Worker rows. A worker is stale when its heartbeat is older than `QUEUE_WORKER_HEARTBEAT_STALE`. |
| Jobs (total) | Every job in every status. |
| Jobs pending | `PENDING` jobs that are due and waiting for a worker. |
| Jobs scheduled | `PENDING` jobs whose start time is still in the future. |
| Jobs running, complete, failed | Jobs in that status. Complete jobs disappear after [rotation](workers.md#maintenance). |
| Oldest queued age | How long the oldest due `PENDING` job has been waiting. A figure that keeps growing means the workers are not keeping up, or are not running. |
| Avg. latency (24h) | The average time between a job becoming available and a worker starting it. |
| Avg. duration (24h) | The average time a job takes to run, for jobs that finished. |
| Throughput (1h, 24h) | Jobs finished in that period, both `COMPLETE` and `FAILED`. |

### Queues

One row per [discovered queue](queues.md#discovery), with its job counts by status, its average latency and duration over the last hour, and its throughput over the last hour and the last 24 hours. A queue you have not [put in the discovery namespace](queues.md#discovery) is missing here.

### Workers

Every row in the worker table, with its ID, token, queues, how long it has been running, and when it last reported. A stale worker is highlighted. It is either dead or in the middle of a job longer than `QUEUE_WORKER_HEARTBEAT_STALE`; see [the warning](../queue.md#configuration) on that setting. `queue:clean` removes it.

### Failed jobs

The latest 100 `FAILED` jobs, newest first. Each shows its queue, task, payload, when it failed, the number of attempts, and the message from the last error. **Error Trace** opens the stack trace of the last error.

**Retry** puts the job back in `PENDING`. It waits the usual [backoff](tasks.md#retries) for its next attempt number before it runs, so it is not instant. A retried job gets one more attempt: its retries are already used up, so if it fails again it is `FAILED` straight away. To give a task more tries, raise its `getMaxRetries()`.

The old errors stay on the job.

## Dashboard alert

The admin dashboard shows the warning **Some queues are not being processed** when a [discovered queue](queues.md#discovery) has no live worker. It lists the queues without one. A worker is live when its heartbeat is newer than `QUEUE_WORKER_HEARTBEAT_STALE` and it was started for that queue's class.

It is the same check [Find & Replace](../find-replace.md#queue) uses to warn about its own queue. The check is also available to your code:

```php
/** @var \Nails\Queue\Service\Manager $manager */
$manager = Factory::service('Manager', \Nails\Queue\Constants::MODULE_SLUG);

$manager->unworkedQueues();          //  Queue[]: discovered queues with no live worker
$manager->isProcessed('priority');   //  bool: does a live worker cover this queue?
```

## API

The screen reads and writes through two routes. Both need a logged-in user with the permission above. See [API](../api/README.md) for how routes and responses work.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/queue/overview` | Everything on the screen |
| `POST` | `/api/queue/overview/retry` | Retry a failed job |

`GET` returns four keys in `data`:

| Key | Holds |
|---|---|
| `kpis` | The tiles: a `label`, `value`, `hint`, and `type` (`number`, `age`, or `duration`) each |
| `queues` | One entry per discovered queue, with `label`, `jobs`, `latency`, `duration`, and `throughput` |
| `workers` | One entry per worker, with `id`, `token`, `queues`, `created`, `heartbeat`, and `stale` |
| `failed` | Up to 100 failed jobs, with `id`, `queue`, `task`, `payload`, `finished`, `attempts`, and `errors` |

`POST` takes `job_id`. It answers `400` if `job_id` is missing or the job is not `FAILED`, and `404` if there is no such job.
