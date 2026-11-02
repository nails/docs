---
description: >-
  Run workers, read their log, stop them cleanly, run one job by hand, and
  keep the tables tidy with queue:clean.
---

# Workers

A worker is the `nails queue:work` command. It is a long-running process that polls for jobs and runs them, one at a time.

```bash
nails queue:work
```

Run it from the project root. In production, don't start it by hand: use a [process manager](process-managers.md) so it starts on boot and comes back if it exits.

## Choosing queues

| Option | Effect |
|---|---|
| `--queue` | A [queue](queues.md) alias or class name to work. Repeat it to work several. Defaults to `default`. |

```bash
#  The default queue
nails queue:work

#  By alias, two queues in one worker
nails queue:work --queue=default --queue=reports

#  By class name
nails queue:work --queue='App\Queue\Queues\Reports'
```

A worker only takes jobs from the queues it was started with. A job on a queue that no worker was started for is never run. That includes the built-in `priority` queue: it is not served by the default worker, so [start one for it](queues.md#priority).

## What a worker does

1. Registers itself in `nails_queue_worker` and prints its ID and token.
2. Calls `setup()` on each of its queues once.
3. Loops until told to stop:
   1. Reserves the oldest `PENDING` job that is due, from any of its queues. Two workers never get the same job.
   2. If there is one, marks it `RUNNING`, runs the task, then marks it `COMPLETE`, schedules a [retry](tasks.md#retries), or marks it `FAILED`.
   3. If there isn't one, sleeps for `QUEUE_WORKER_WAIT_TIME` milliseconds plus up to 200 ms of random jitter, then doubles the wait for the next empty poll, up to 5,000 ms. The wait starts again after the next job. A job pushed to an idle worker can wait up to about five seconds.
   4. Calls `refresh()` on each queue when `QUEUE_WORKER_REFRESH_INTERVAL` seconds have passed.
   5. Writes its heartbeat, at most once every `QUEUE_WORKER_HEARTBEAT_DEBOUNCE` seconds.
4. Removes its own row from `nails_queue_worker` on the way out.

A worker runs one job at a time. To run more jobs at once, run more workers. They can share a queue, and they can share a machine.

{% hint style="info" %}
A worker reads your code once, when it starts. After a deploy, restart the workers or they keep running the old code. See [Deploying](process-managers.md#deploying).
{% endhint %}

### Log

The worker prints to the terminal and also writes the same lines to `queue-worker-YYYY-MM-DD.php` in the application log directory, next to the [cron](../cron.md) log. Each line starts with `[worker:#ID]`. The file switches to the new day's name the next time `refresh()` is due.

```
[worker:#12] Worker ID:
[worker:#12] #12:web-01:48213:9f3a1c2e
[worker:#12] Working queues:
[worker:#12] Nails\Queue\Queue\Queues\DefaultQueue
[worker:#12] Waiting for jobs:
[worker:#12] [App\Queue\Tasks\Reports\Generate:#481] ......... RUNNING
[worker:#12] [App\Queue\Tasks\Reports\Generate:#481] ......... COMPLETE [0.2314s]
[worker:#12] [App\Queue\Tasks\Reports\Generate:#482] ......... RETRY 1/3 (next at 14:02:07) [0.0412s]
```

A job line ends in `COMPLETE`, `RETRY n/max` with the time the retry becomes due, or `FAILED`, followed by how long the attempt took. The error itself is not in the log. It is stored on the job: see [Failed jobs](admin.md#failed-jobs).

## Stopping

Send the worker `SIGTERM` or `SIGINT` (`Ctrl+C` in a terminal). It finishes the job it is running, takes no more, removes its row, and exits with status 0. An idle worker stops within about a tenth of a second.

This needs the `pcntl` PHP extension. Without it the signal kills the process at once, in the middle of a job, and the worker's row stays behind until [`queue:clean`](#maintenance) removes it.

A process manager that stops a worker has to wait for the job to end. If it gives up and sends `SIGKILL` first, the job is cut off and later run again. Set the stop timeout above your longest job. The [examples](process-managers.md) do.

## Running one job by hand

`queue:run` takes a job ID and runs its task in the current process. Use it to reproduce a failure with a debugger or `var_dump()`.

```bash
nails queue:run 481
```

It prints the job's ID, queue, task, and payload, registers a short-lived worker for the job's queue, calls `setup()`, runs the task, and removes the worker.

It does not change the job. The status, attempts, and errors are left as they were, a task that throws prints `ERROR: …` and the command still exits 0, and the job can be in any status, including `COMPLETE`. Running a job that is `PENDING` can run it twice if a worker takes it at the same time.

## Maintenance

`queue:clean` repairs what a crashed or killed worker leaves behind and removes old rows.

```bash
nails queue:clean
```

It does three things, in this order:

| Step | What it does |
|---|---|
| Stale workers | Deletes workers whose heartbeat is more than `QUEUE_WORKER_HEARTBEAT_STALE` seconds old. |
| Stuck jobs | Puts `RUNNING` jobs that no longer belong to a worker back in `PENDING`. A job loses its worker when the worker's row is deleted, so the previous step feeds this one. `attempts` is not changed. |
| Old jobs | Deletes `COMPLETE` jobs older than `QUEUE_JOB_ROTATE_COMPLETE_DAYS` and `FAILED` jobs older than `QUEUE_JOB_ROTATE_FAILED_DAYS`, counted from when they finished. `0` turns that deletion off. |

The module registers this as a [cron](../cron.md) task that runs every five minutes, so there is nothing to schedule as long as the cron runner is already running on the server.

### When a worker dies

If a worker is killed with `SIGKILL`, loses its machine, or runs out of memory, the job it was running stays `RUNNING`. It is recovered by `queue:clean`: the worker is deleted once its heartbeat is stale, and the job is reset and run again by another worker. That takes at least `QUEUE_WORKER_HEARTBEAT_STALE` seconds and then up to five minutes more, for the next `queue:clean`, so budget five to ten minutes with the defaults.

Nothing is lost, but the job runs again from the start. See [Write tasks that can run twice](tasks.md#write-tasks-that-can-run-twice).
