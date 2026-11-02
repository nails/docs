---
description: >-
  Keep queue workers running in production with systemd or Supervisor: start
  on boot, restart on exit, stop cleanly, and restart after a deploy.
---

# Process managers

`nails queue:work` runs until it is stopped. Something has to start it when the server boots, start it again if it crashes or is killed, and stop it cleanly. That is the job of a process manager. This page covers [systemd](#systemd) and [Supervisor](#supervisor). Use one of them, not both.

Whichever you use, set these up the same way:

| Setting | Why |
|---|---|
| Run it as the web server's user | The worker writes logs and files the web app reads. A worker run as `root` creates files the web app can't overwrite. |
| Start in the project root | `nails` finds the application from the working directory. |
| Pass the same configuration as the web app | The worker is a separate process and does not inherit what PHP-FPM or Apache has. Anything set as an environment variable for the web app must be set for the worker too. In particular `ENVIRONMENT` defaults to `DEVELOPMENT`, so a worker without it runs as a development environment. |
| Restart on any exit | A worker that exits, for any reason, should come back. |
| Stop with `SIGTERM` and wait | The worker finishes its current job on `SIGTERM`. The manager's stop timeout has to be longer than your longest job, or the job is killed part-way and run again. This needs the `pcntl` extension: see [Stopping](workers.md#stopping). |
| Use absolute paths to `nails` | Both managers start the process with a minimal environment. Find the path with `command -v nails`. |

{% hint style="info" %}
You do not need a process manager to run `queue:clean`. It is a [cron](../cron.md) task. If cron is running, it is already scheduled.
{% endhint %}

## systemd

Create a template unit. The `@` makes it a template: the text after it in the name is the instance, available as `%i`, so one file runs as many workers as you like.

`/etc/systemd/system/nails-queue@.service`

```ini
[Unit]
Description=Nails queue worker %i
After=network.target

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/var/www/app
Environment=ENVIRONMENT=PRODUCTION
ExecStart=/usr/local/bin/nails queue:work --queue=default
Restart=always
RestartSec=5
KillSignal=SIGTERM
TimeoutStopSec=300
SyslogIdentifier=nails-queue-%i

[Install]
WantedBy=multi-user.target
```

| Line | Effect |
|---|---|
| `WorkingDirectory` | The project root. |
| `Environment` | Config for the worker. For several values use `EnvironmentFile=/etc/nails/app.env`, a file of `KEY=value` lines. |
| `ExecStart` | The worker command. `--queue` names the [queue](queues.md) this worker serves. It is `default` here, and every other queue needs [a worker of its own](#a-worker-per-queue). |
| `Restart=always`, `RestartSec` | Restart five seconds after any exit, except a `systemctl stop`. |
| `TimeoutStopSec` | How long to wait after `SIGTERM` before `SIGKILL`. The default is 90 seconds. Set it above your longest job. |

Load the unit and start two workers:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nails-queue@1 nails-queue@2
```

`enable` starts them at boot, `--now` also starts them now. The `1` and `2` are only labels.

Check on them and read their output:

```bash
systemctl status 'nails-queue@*'
journalctl -u 'nails-queue@*' -f
```

### Run it as one service

If a single worker is enough, drop the `@` from the file name, delete `%i` from the two lines that use it, and use the plain name: `systemctl enable --now nails-queue`.

## Supervisor

Supervisor reads one file per program. On Debian and Ubuntu that is in `/etc/supervisor/conf.d/`; other distributions put it in `/etc/supervisord.d/` with an `.ini` extension.

`/etc/supervisor/conf.d/nails-queue.conf`

```ini
[program:nails-queue]
command=/usr/local/bin/nails queue:work --queue=default
directory=/var/www/app
user=www-data
environment=ENVIRONMENT="PRODUCTION"
process_name=%(program_name)s_%(process_num)02d
numprocs=2
autostart=true
autorestart=true
startsecs=5
startretries=10
stopsignal=TERM
stopwaitsecs=300
redirect_stderr=true
stdout_logfile=/var/log/nails-queue_%(process_num)02d.log
stdout_logfile_maxbytes=20MB
stdout_logfile_backups=5
```

| Line | Effect |
|---|---|
| `directory` | The project root. |
| `environment` | Config for the worker, as a comma-separated list of `KEY="value"` pairs. |
| `numprocs`, `process_name` | How many workers to run. With more than one, `process_name` has to include `%(process_num)02d`. |
| `autorestart=true` | Restart on any exit. The default, `unexpected`, does not restart a worker that exits with status 0. |
| `startsecs`, `startretries` | A worker that exits before `startsecs` counts as a failed start. After `startretries` failed starts Supervisor stops trying and marks it `FATAL`. A high number lets the worker ride out a database that isn't up yet at boot. |
| `stopsignal`, `stopwaitsecs` | Send `SIGTERM`, then `SIGKILL` after this many seconds. The default is 10, which would kill most jobs part-way. Set it above your longest job. |
| `stdout_logfile` | Console output. The worker also writes its own [log](workers.md#log). |

Tell Supervisor about the new file and start the workers:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status 'nails-queue:*'
```

`update` starts a program that is new, because of `autostart=true`.

## A worker per queue

A worker serves only the queues it was started with. Each queue gets its own program, with its own `--queue`, so that a slow queue never holds up a fast one.

The built-in `priority` queue is the case that matters. It is a queue in name only: nothing starts a worker for it, and a single worker given both `default` and `priority` runs jobs oldest first across the two, so priority jobs gain nothing. The app has to configure a separate process for `priority`, or those jobs are never run. See [Priority](queues.md#priority). The same applies to any queue you [define](queues.md#defining-a-queue).

{% tabs %}
{% tab title="systemd" %}
Copy the unit to `/etc/systemd/system/nails-queue-priority@.service`. Change two lines and leave the rest as it was:

```ini
ExecStart=/usr/local/bin/nails queue:work --queue=priority
SyslogIdentifier=nails-queue-priority-%i
```

Then load it and start a worker:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nails-queue-priority@1
```
{% endtab %}

{% tab title="Supervisor" %}
Add a second program to the same file, or to a file of its own:

```ini
[program:nails-queue-priority]
command=/usr/local/bin/nails queue:work --queue=priority
directory=/var/www/app
user=www-data
environment=ENVIRONMENT="PRODUCTION"
process_name=%(program_name)s_%(process_num)02d
numprocs=1
autostart=true
autorestart=true
startsecs=5
startretries=10
stopsignal=TERM
stopwaitsecs=300
redirect_stderr=true
stdout_logfile=/var/log/nails-queue-priority_%(process_num)02d.log
stdout_logfile_maxbytes=20MB
stdout_logfile_backups=5
```

Then load it:

```bash
sudo supervisorctl reread
sudo supervisorctl update
```
{% endtab %}
{% endtabs %}

Priority jobs are usually short, so `TimeoutStopSec` and `stopwaitsecs` can be lower here than for `default`. Set them from the longest job that queue runs.

## Checking it works

1. Open **Queue → Overview** in [admin](admin.md). Each running worker is in the **Workers** table with a recent heartbeat, listing the queues it serves. The dashboard warning for queues with no worker should be gone, which means every [discovered queue](queues.md#discovery), `priority` included, has a worker.
2. Push a `DoNothing` job, as in [Trying it out](tasks.md#trying-it-out), and watch it complete.
3. Restart the worker (below) and check a replacement appears in **Workers**. If you stop a worker without `SIGTERM`, its old row stays until `queue:clean` removes it.

## Deploying

A worker loads your code once, when it starts. After a deploy, restart the workers so they run the new code.

{% tabs %}
{% tab title="systemd" %}
```bash
sudo systemctl restart 'nails-queue*'
```
{% endtab %}

{% tab title="Supervisor" %}
```bash
sudo supervisorctl restart 'nails-queue:*' 'nails-queue-priority:*'
```
{% endtab %}
{% endtabs %}

Both managers send `SIGTERM` and wait for the job in progress, up to the timeout you set, so a restart does not cut a job off. A restart is only as quick as the longest job running when it starts.

## Other environments

The same rules apply anywhere else a worker runs, such as a Docker container or a Kubernetes pod. Run `nails queue:work` as the container's main process, restart the container if it exits, forward `SIGTERM` to the process, and make the grace period before `SIGKILL` longer than your longest job. Give the container the same environment as the web app.
