---
description: >-
  A database-backed job queue. Push work from a request, run it in a
  long-lived worker, and retry it when it fails.
---

# Queue

The queue module moves slow or fragile work out of the request. The app pushes a job, a worker process picks it up and runs it, and a failed job is retried with a growing delay. Jobs live in MySQL, so there is no broker to install.

```bash
composer require nails/module-queue
nails db:migrate
```

The module needs `nails/module-console` for the worker commands and [`nails/module-cron`](cron.md) to schedule its cleanup. Reserving a job uses `SELECT … FOR UPDATE SKIP LOCKED`, so the database must be MySQL 8.0 or newer, or MariaDB 10.6 or newer.

The migration creates two tables, prefixed with `NAILS_DB_PREFIX` as usual:

| Table | Holds |
|---|---|
| `nails_queue_job` | Every job, from the moment it is pushed until it is rotated out |
| `nails_queue_worker` | One row per running worker, with its last heartbeat |

Both tables are hidden from [Find & Replace](find-replace/exclusions.md).

## How it fits together

| Piece | What it is |
|---|---|
| [Task](queue/tasks.md) | A class with a `run()` method. It is the work. |
| Data | The JSON payload stored with a job and handed to `run()`. |
| [Queue](queue/queues.md) | A named stream of jobs. A worker listens to one or more queues. |
| [Worker](queue/workers.md) | The `nails queue:work` process. It runs one job at a time. |
| [Manager](queue/tasks.md#pushing-a-job) | The service the app uses to push jobs. Workers use it to reserve them. |

A job moves through four statuses:

| Status | Meaning |
|---|---|
| `PENDING` | Waiting for a worker. A job with a future `available_at` is _scheduled_ and is still `PENDING`. |
| `RUNNING` | A worker has reserved it and is running the task. |
| `COMPLETE` | The task returned without throwing. |
| `FAILED` | The task threw and has no retries left. The errors are kept on the job. |

A failed attempt that still has retries left goes back to `PENDING` with a later `available_at`.

1. The app calls `push()` with a task, a payload, and optionally a queue and a start time.
2. A [worker](queue/workers.md) reserves the oldest job that is `PENDING` and due, in the queues it was started for.
3. The worker calls `run()` on the task with the payload.
4. The job becomes `COMPLETE`, is rescheduled for a retry, or becomes `FAILED`.
5. [`queue:clean`](queue/workers.md#maintenance), run by cron, removes dead workers, recovers stuck jobs, and deletes old rows.

{% hint style="warning" %}
Nothing runs a job until a worker is running. Jobs sit in `PENDING` and the admin dashboard shows a warning. Keep a worker alive with a [process manager](queue/process-managers.md). Each [queue](queue/queues.md) a worker should serve is named when it starts, and the built-in `priority` queue needs a [worker of its own](queue/queues.md#priority).
{% endhint %}

## Pages

* [Tasks](queue/tasks.md): write a task, push a job, and how retries work
* [Queues](queue/queues.md): the built-in queues, your own queues, and aliases
* [Workers](queue/workers.md): run workers, read their log, stop them, and the maintenance command
* [Process managers](queue/process-managers.md): keep workers running with systemd or Supervisor
* [Admin](queue/admin.md): the overview screen, the dashboard alert, and the API behind them

## Configuration

Set these as [config values](../getting-started/configuration.md), usually environment variables. All are optional.

| Config | Default | Effect |
|---|---|---|
| `QUEUE_WORKER_WAIT_TIME` | `500` | Milliseconds an idle worker first sleeps between polls. The wait doubles on each empty poll, up to 5,000 ms, and starts again after a job. A random 0 to 200 ms is added each time. |
| `QUEUE_WORKER_REFRESH_INTERVAL` | `300` | Seconds between calls to each queue's `refresh()` hook. |
| `QUEUE_WORKER_HEARTBEAT_DEBOUNCE` | `5` | Minimum seconds between heartbeat writes from one worker. |
| `QUEUE_WORKER_HEARTBEAT_STALE` | `300` | Seconds without a heartbeat before a worker counts as dead. |
| `QUEUE_JOB_ROTATE_COMPLETE_DAYS` | `7` | Days to keep `COMPLETE` jobs. `0` keeps them forever. |
| `QUEUE_JOB_ROTATE_FAILED_DAYS` | `30` | Days to keep `FAILED` jobs. `0` keeps them forever. |

{% hint style="danger" %}
A worker records its heartbeat between jobs, not during one. If a single job can run for longer than `QUEUE_WORKER_HEARTBEAT_STALE`, the worker looks dead, [`queue:clean`](queue/workers.md#maintenance) deletes it, and the job is put back in `PENDING` while it is still running. Another worker then runs it a second time. Set `QUEUE_WORKER_HEARTBEAT_STALE` higher than your longest job.
{% endhint %}

## Overriding

The Manager, both models, both resources, and the `Data` factory can each be replaced by defining a class of the same name under `App\Queue`: `App\Queue\Service\Manager`, `App\Queue\Model\Job`, `App\Queue\Factory\Data`, and so on. See [Overloading](../key-concepts/factory/overloading.md).
