---
description: The Rollbar error handler sends errors to Rollbar as they happen.
---

# Rollbar Handler

This handler is a middle actor in the error handling chain. Errors and exceptions will be forwarded to [Rollbar](https://rollbar.com) as they happen, and then will be bubbled to the [Default Handler](default-handler.md).

## Installation

```bash
composer require nails/driver-error-handler-rollbar
```

## Configuration

This handler accepts the following [configuration](../../getting-started/configuration.md) values:

| Config                 | Default |
| ---------------------- | ------- |
| `ROLLBAR_ACCESS_TOKEN` | `null`  |

