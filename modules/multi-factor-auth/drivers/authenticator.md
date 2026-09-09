---
description: >-
  Let the user confirm sign-in with a time-based code from an authenticator app
  on their phone.
---

# Authenticator

This driver is a TOTP second factor. After a successful password login, the user types a six-digit code from an app such as Google Authenticator, 1Password, or Authy.

Unlike [Email](email.md), the secret lives with the user. They must [enrol](#enrollment) before the driver can validate a sign-in.

## Installation

```bash
composer require nails/driver-multi-factor-auth-authenticator
```

This depends on `nails/module-multi-factor-auth`. [Enable the driver](../#enabling-a-driver) after install.

## Enrollment

`requiresEnrollment()` is `true`. Until the user has an Authenticator method, sign-in (when the group policy requires a challenge) sends them through setup:

1. `setupStart()` generates a 160-bit secret and an `otpauth://totp/…` URI (six digits, 30-second period).
2. The setup page shows a QR code and the secret for manual entry.
3. They confirm with a code from the app. `setupComplete()` checks it and stores the secret (and the matching time-step) on their user-method row.
4. Authenticator becomes their default method. They can change that later at `/mfa/manage`.

Enrollment cannot be done from the console: the secret has to be shown to the user.

## What happens on `/mfa`

1. `preForm()` shows the configured prompt. Nothing is sent; the app already has the secret.
2. `validate()` loads the enrolled secret, accepts the current 30-second window plus one step either side, and rejects a code that has already been used (`last_period`).
3. **Request another verification code** is not shown (`canTryAgain()` is `false`). `resend()` is a no-op.

## Settings

Configure the driver in [Admin](../../admin/) under **Settings → MFA: Authenticator**.

| Setting | Default | Meaning |
| --- | --- | --- |
| **Issuer name** | *(application name)* | Label shown in authenticator apps. Leave blank to use `APP_NAME` |
| **Prompt** | `Enter the code from your authenticator app.` | Flash message on the verification form |
| **Invalid Code Entered** | `Invalid code entered. Please try again.` | Flash message on a bad or reused code |
