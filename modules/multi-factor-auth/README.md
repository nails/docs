---
description: >-
  Add a second step to sign-in so a password alone is not enough to access an
  account.
---

# Multi-Factor Auth

The Multi-Factor Auth (MFA) module sits in front of [Auth](../auth.md) and asks for a second proof of identity after a successful password (or social) login. Until that second step succeeds, the user is not actually signed in.

How the second step works is left to [drivers](drivers/). The module itself handles the login intercept, the verification page, tokens, rate limits, and “remember this device”.

{% hint style="warning" %}
The module does not ship a driver. You must install and enable at least one, otherwise sign-in will fail once MFA is in the project. The only official driver today is [Email](drivers/email.md).
{% endhint %}

## What users see

1. They submit the normal login form.
2. Auth accepts the credentials and fires its login event.
3. MFA logs them back out, stores a short-lived token in a cookie, and redirects them to `/mfa`.
4. The enabled driver runs (for email, that means sending a code).
5. They enter the code. Optionally they tick **Don't ask again on this device**.
6. On success they are logged in and sent to the original `return_to` URL, or to their group homepage.

If they are already trusted on this browser (the privileged cookie is still valid), step 3 is skipped and they go straight through.

Admin impersonation (`wasAdmin()`) also skips MFA, so staff who “log in as” a user are not challenged.

## Installation

The module needs [Auth](../auth.md) and [Email](../email.md).

```bash
composer require nails/module-multi-factor-auth
composer require nails/driver-multi-factor-auth-email
```

Run [migrations](../../core-services/database/migrations.md) so the `mfa_token` table exists, then [enable a driver](#enabling-a-driver).

## Enabling a driver

Enabled authentication drivers are stored as the `enabled_driver_authentication` app setting for `nails/module-multi-factor-auth`. Multiple drivers can be enabled; **the verification page currently uses the first one**.

There is no Admin screen for this yet, so enable a driver in code (a one-off during setup is enough):

```php
use Nails\Factory;
use Nails\MFA\Constants;

/** @var \Nails\MFA\Service\AuthenticationDriver $oDrivers */
$oDrivers = Factory::service('AuthenticationDriver', Constants::MODULE_SLUG);
$oDrivers->saveEnabled([
    'nails/driver-multi-factor-auth-email',
]);
```

The slug is the Composer package name of the driver.

{% hint style="info" %}
Driver-specific options (code format, user-facing copy, and so on) live on each driver’s own settings page in [Admin](../admin/). For email, that page is labelled **MFA: Email**.
{% endhint %}

## The verification page

The challenge is served at `/mfa` (`mfa/index`). The page:

* Requires the `mfa-token` cookie. If cookies are disabled, sign-in cannot continue.
* Sends `Cache-Control: no-store` and `Referrer-Policy: no-referrer`.
* Shows a code field (`autocomplete="one-time-code"`), a remember-this-device checkbox, **Verify**, and — when the driver allows it — **Request another verification code**.

To replace the markup, drop your own view at:

```text
application/modules/mfa/views/form.php
```

If that file exists, the module will not load the default `nails.min.css` for the page, so you can style it with the rest of the app.

## Limits and cookies

These values are constants on `Nails\MFA\Service\MultiFactorAuth`.

| Limit | Default | Purpose |
| --- | --- | --- |
| Token lifetime | 5 minutes | How long the user has to complete the challenge |
| Incorrect codes | 5 | After this the token is deleted and they must sign in again |
| New tokens per user per hour | 5 | Caps how often a code can be minted, including “request another code”. Used tokens still count for the hour |
| Remember this device | 14 days | Lifetime of the privileged cookie when the checkbox is ticked |

Cookies:

| Cookie | Role |
| --- | --- |
| `mfa-token` | Encrypted salt + token for the current challenge. HttpOnly, Secure, `SameSite=Lax`, 5 minute TTL |
| `mfa-is-privileged` | Encrypted hash of the user (site `PRIVATE_KEY`, user id, user salt). Marks the browser as trusted |

If the privileged cookie is missing or does not match the logged-in user, MFA runs again on the next login.

Expired, malformed, or exhausted tokens send the user back to login with a short explanation. Hitting the hourly mint cap surfaces as “Please wait and try again later.”

## Using the service

Load it from the [Factory](../../key-concepts/factory/):

```php
use Nails\Factory;
use Nails\MFA\Constants;
use Nails\MFA\Service\MultiFactorAuth;

/** @var MultiFactorAuth $oMfa */
$oMfa = Factory::service('MultiFactorAuth', Constants::MODULE_SLUG);
```

Useful methods:

| Method | What it does |
| --- | --- |
| `authenticate(User $oUser, bool $bIsRemembered, bool $bForce = false)` | Starts a challenge and redirects to `/mfa` when MFA is required, or when `$bForce` is `true` |
| `isAuthenticated()` | `true` when the user is logged in **and** privileged |
| `requiresAuthentication()` | Inverse of the above, except admin impersonation is never challenged |
| `setIsPrivileged(User $oUser, bool $bRemember = true)` | Trusts this browser. `$bRemember = false` makes it a session cookie |
| `isPrivileged()` | Reads the privileged cookie for the active user |

To protect a sensitive action (for example changing an email address) you can force a fresh challenge even if the device is remembered:

```php
$oMfa->authenticate(activeUser(), false, true);
```

## Logging

MFA writes to `application/logs/mfa-YYYY-MM-DD.php`. Each request gets a `uniqid()` in the line format so you can follow one sign-in attempt. The logger is the `Logger` service on the MFA module and can be overloaded as `App\MFA\Service\Logger`.

## Customising behaviour

Services, the token model, and the token resource follow the usual [overloading](../../key-concepts/factory/overloading.md) convention:

* `App\MFA\Service\MultiFactorAuth`
* `App\MFA\Service\AuthenticationDriver`
* `App\MFA\Service\Logger`
* `App\MFA\Model\Token`
* `App\MFA\Resource\Token`

To change how a code is delivered, write or configure a [driver](drivers/) rather than forking the module.

## Drivers

{% content-ref url="drivers/" %}
[drivers](drivers/)
{% endcontent-ref %}

{% content-ref url="drivers/email.md" %}
[email.md](drivers/email.md)
{% endcontent-ref %}
