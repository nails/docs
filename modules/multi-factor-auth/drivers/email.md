---
description: >-
  Send a one-time login code to the user’s email address after they enter their
  password.
---

# Email

This driver is the default (and currently only) official second factor. After a successful password login, it emails a short code to the account’s address. The user types that code on `/mfa` to finish signing in.

The email is sent with the [Email](../../email.md) module. Delivery, branding, and “view online” behaviour are the same as every other Nails email.

## Installation

```bash
composer require nails/driver-multi-factor-auth-email
```

This depends on `nails/module-multi-factor-auth`. [Enable the driver](../#enabling-a-driver) after install.

## What happens on `/mfa`

1. `preForm()` looks on the MFA token for an existing code.
2. If none is stored, it generates one, saves it on the token, emails it, and shows a success message.
3. If a code is already stored (the user refreshed the page), it does **not** send again. A warning explains that a code is already on its way.
4. `validate()` compares the submitted value with the stored code. A mismatch throws `InvalidCodeException` with the configured “invalid code” copy.
5. **Request another verification code** is available (`canTryAgain()` is `true`). That path mints a new token, so a new email goes out — subject to the [hourly mint cap](../#limits-and-cookies).

The code lives only on the token. It is never written to the user record.

## Settings

Configure the driver in [Admin](../../admin/) under **Settings → MFA: Email**.

| Setting | Default | Meaning |
| --- | --- | --- |
| **Format** | `DDDDDD` | Shape of the generated code |
| **Code has been sent** | `A code has been sent to your email address. Please enter it below.` | Flash message after a new email |
| **Code has already been sent** | `A code has already been sent to your email address. Please enter it below.` | Flash message on refresh |
| **Invalid Code Entered** | `Invalid code entered. Please try again.` | Flash message on a bad code |

### Code format

`Strings::generateToken()` interprets the format string:

| Character | Result |
| --- | --- |
| `D` | A digit, `0–9` |
| `C` | An uppercase letter, `A–Z` |
| `A` | Either a digit or an uppercase letter |

So `DDDDDD` is six digits (for example `482193`), and `CCC-DDD` is something like `QKM-047`. Keep codes short enough to type on a phone.

## The email

The message type is defined by the MFA **module** (not the driver):

| | |
| --- | --- |
| Slug | `mfa_email_code` |
| Factory | `EmailCode` on `nails/module-multi-factor-auth` |
| Subject | `Your login verification code: {{code}}` |
| HTML body | `mfa/email/code` |
| Plaintext body | `mfa/email/code_plaintext` |
| Unsubscribe | Disabled |

Template data includes `code`. The stock copy tells the recipient not to share it, that it is valid for five minutes, and links to the forgotten-password flow if they did not request it.

Override the bodies in the app:

```text
application/modules/mfa/views/email/code.php
application/modules/mfa/views/email/code_plaintext.php
```

Subject and other type metadata can be edited in Admin like any other [email definition](../../email.md).

To send a test message:

```bash
php nails email:test you@example.com -t mfa_email_code
```

The factory’s test payload uses `code` `123456`.

## Overloading the email factory

The factory is `Nails\MFA\Factory\Email\Code`. The module also looks for:

```text
App\Auth\MultiFactorAuth\Driver\Email\Factory\Email\Code
```

Extend `Nails\Email\Factory\Email` and keep `$sType = 'mfa_email_code'` if you replace it.
