---
description: This module provides an explicit API for scheduling console commands.
---

# Cron

The runner writes to `application/logs/cron-YYYY-MM-DD.php`. A minute in which no task executes produces no log lines. Tasks that run are logged with their exit status and duration.

A task class can set `const LOG_OUTPUT = false` (the default is `true`) when the command already writes its own log. The cron log then keeps the start line, exit code, and duration, and copies the command output only when the exit code is not 0. `Nails\Housekeeping\Cron\Task\Run` sets this.

While a task still holds its process slot, the runner logs the skip once when it starts and once, with a count and duration, when the slot frees. It does not log the skip on the minutes in between.

Captured output written to the log has ANSI colour codes and banner rules removed, and every entry ends with a newline.

