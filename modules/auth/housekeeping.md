---
description: "Housekeeping routines shipped by the Auth module."
---

# Housekeeping

Auth registers the following [housekeeping](../housekeeping/) routines. They are discovered automatically when `nails/module-housekeeping` is installed and appear under Admin → Utilities → Housekeeping.

Run them on demand with `housekeeping:run --routine=…` (add `--dry-run` or `--force` as needed). `auth:user:import:process` is unchanged.

## User imports

`Nails\Auth\Housekeeping\UserImports` is the scheduled cleaner for user import jobs. It runs every fifteen minutes and does three things, in order:

1. **Release orphans** — clear `claim_token` / `claimed` on jobs whose claim is older than `AUTH_USER_IMPORT_STALE_CLAIM` seconds (default 900). This is an update, not a delete. Each released id is written to the audit log. Status is left alone so a job can resume from its cursor.
2. **Reap drafts** — delete `DRAFT` jobs whose `modified` is older than `AUTH_USER_IMPORT_DRAFT_TTL` seconds (default 86400). Cap `MAX_PER_RUN` (100). After the row is gone, the job's CDN objects are destroyed.
3. **Rotate finished** — the same delete path for terminal statuses (`COMPLETE`, `PARTIAL`, `FAILED`) older than `AUTH_USER_IMPORT_RETENTION` seconds (default 2592000).

Audit log columns: `id`, `status`, `modified`. Release lines are logged as `RELEASE` rather than `DELETE`.

## API access tokens

`Nails\Auth\Housekeeping\AccessTokens` deletes rows from `user_auth_access_token` whose `expires` is in the past. Tokens with a null expiry are left alone. It runs daily.

Audit log columns: `id`, `user_id`, `expires`. The token secret (or its hash) is never logged.

There was no scheduled cleaner for this table before; expired tokens were only skipped at use.

## Legacy 2FA tokens

`Nails\Auth\Housekeeping\TwoFactorTokens` deletes expired rows from `user_auth_two_factor_token` (the tokens minted by `Authentication::mfaTokenGenerate()` with a ten-minute TTL). It runs every fifteen minutes.

Audit log columns: `id`, `user_id`, `expires`. `token` and `salt` are not logged.

There was no scheduled cleaner for this table before.

## User events

`Nails\Auth\Housekeeping\UserEvents` deletes rows from `user_event` older than `AUTH_USER_EVENT_RETENTION_DAYS`. Unset or `0` disables deletion; the routine still appears in the list and no-ops. There is no module default — apps that want a policy set the config.

It runs daily.

Audit log columns: `id`, `created_by`, `type`, `created`.
