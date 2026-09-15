---
description: "Housekeeping routines shipped by the Multi-Factor Auth module."
---

# Housekeeping

MFA registers the following [housekeeping](../housekeeping/) routine. It is discovered automatically when `nails/module-housekeeping` is installed and appears under Admin → Utilities → Housekeeping.

## Challenge tokens

`Nails\MFA\Housekeeping\Tokens` deletes rows from `mfa_token` whose `expires` is in the past. Challenge tokens last 300 seconds (`MultiFactorAuth::TOKEN_TTL`); the routine runs every fifteen minutes so expired rows do not accumulate.

Audit log columns: `id`, `user_id`, `expires`.

There was no scheduled cleaner for this table before; expired tokens were only rejected at use.
