---
description: >-
  Add a second step to sign-in so a password alone is not enough to access an
  account.
---

# Multi-Factor Auth

The Multi-Factor Auth (MFA) module sits in front of [Auth](../auth.md) and asks for a second proof of identity after a successful password (or social) login. Until that second step succeeds, the user is not actually signed in.

How the second step works is left to [drivers](drivers/). The module itself handles the login intercept, group policy, enrollment, the verification pages, tokens, rate limits, and “remember this device”.

{% hint style="warning" %}
The module does not ship a driver. You must install and enable at least one, otherwise a group that requires MFA cannot finish sign-in. Official drivers are [Email](drivers/email.md) and [Authenticator](drivers/authenticator.md).
{% endhint %}

## What users see

1. They submit the normal login form.
2. Auth accepts the credentials and fires its login event.
3. If their [group policy](#group-policy) requires a challenge, MFA stores a short-lived token in a cookie, clears the half-finished login, and redirects them to `/mfa`.
4. If they have no enrolled method and the policy is **Required**, they are sent through setup first.
5. Otherwise they verify with their default method, or pick one if they have several and no default is set.
6. They enter the code. Optionally they tick **Trust this device**.
7. On success they are logged in and sent to the original `return_to` URL, or to their group homepage.

If they are already trusted on this browser (the privileged cookie is still valid), the challenge is skipped and they go straight through.

Admin impersonation (`wasAdmin()`) also skips MFA, so staff who “log in as” a user are not challenged.

A **user-verified passkey login skips the challenge** for that sign-in. A passkey is already a phishing-resistant second factor, so re-challenging adds nothing. This is decided per-login from a signal Auth records at sign-in ([`Authentication::getLoginMethod()`](../auth/passkeys.md)); it is not a trusted-device cookie, so a later password login on the same browser is still challenged. Apps that override `mfa/views/form.php` must adopt the `instanceof Interactive` snippet (see [Drivers](drivers/#interactive-drivers)) for the passkey factor to render on `/mfa` at all.

Users can review and change their methods at `/mfa/manage` when their group policy is Optional or Required and there is something they can actually change.

## Installation

The module needs [Auth](../auth.md). The [Email](drivers/email.md) driver also needs the [Email](../email.md) module.

```bash
composer require nails/module-multi-factor-auth
composer require nails/driver-multi-factor-auth-email
composer require nails/driver-multi-factor-auth-authenticator
```

Run [migrations](../../core-services/database/migrations.md) so the `mfa_token`, `mfa_group_policy`, and `mfa_user_method` tables exist, then [enable a driver](#enabling-a-driver) and set [group policy](#group-policy).

## Group policy

Each user group has a mode. Groups with no row default to **Disabled**.

| Mode | Sign-in | Management |
| --- | --- | --- |
| `DISABLED` | No challenge | Users cannot enrol methods |
| `OPTIONAL` | Challenged only if they already have a method | Users can add, remove, and choose a default |
| `REQUIRED` | Always challenged (unless the device is trusted) | Users can add methods and change the default. They cannot remove their last method |

Set policy in Admin when editing a group (the **MFA** tab), or from the console:

```bash
php vendor/nails/module-console/console.php mfa:group:policy --group=staff --mode=REQUIRED
```

{% hint style="info" %}
A user with an Optional policy and no enrolled methods signs in with a password only. The moment they enrol a method, later logins from untrusted devices will ask for it.
{% endhint %}

## Enabling a driver

Enabled authentication drivers are stored as the `enabled_driver_authentication` app setting for `nails/module-multi-factor-auth`. Multiple drivers can be enabled. At sign-in the user’s **default** enrolled method is used; if they have no default they are offered a chooser. During setup, a single enabled driver is selected automatically.

Enable a driver from the console:

```bash
php vendor/nails/module-console/console.php mfa:driver:enable --driver=nails/driver-multi-factor-auth-email
```

Or in code:

```php
use Nails\Factory;
use Nails\MFA\Constants;

/** @var \Nails\MFA\Service\AuthenticationDriver $oDrivers */
$oDrivers = Factory::service('AuthenticationDriver', Constants::MODULE_SLUG);
$oDrivers->saveEnabled([
    'nails/driver-multi-factor-auth-email',
    'nails/driver-multi-factor-auth-authenticator',
]);
```

The slug is the Composer package name of the driver.

{% hint style="info" %}
Driver-specific options (code format, issuer name, user-facing copy, and so on) live on each driver’s own settings page in [Admin](../admin/). Email is labelled **MFA: Email**; Authenticator is **MFA: Authenticator**.
{% endhint %}

## The verification and setup pages

Challenges are served at `/mfa`. Management is at `/mfa/manage`. Both:

* Require the `mfa-token` cookie for a live challenge. If cookies are disabled, sign-in cannot continue.
* Send `Cache-Control: no-store` and `Referrer-Policy: no-referrer`.
* Use Nails’ blank header and footer by default.

To wrap the pages in your own shell, provide:

```text
application/modules/mfa/views/structure/header.php
application/modules/mfa/views/structure/footer.php
```

Individual views (`form.php`, `setup.php`, `setup_confirm.php`, `manage.php`) can be overridden in the same directory. If an app-level `form.php` exists, the module will not load the default `nails.min.css` for that page, so you can style it with the rest of the app.

**Request another verification code** calls the current driver’s `resend()` on the same token. It does not mint a new challenge. That button only appears when `canTryAgain()` is `true` (Email yes, Authenticator no), and is capped per token.

## Limits and cookies

These values are constants on `Nails\MFA\Service\MultiFactorAuth`. Override them by extending the service at app level (`App\MFA\Service\MultiFactorAuth`). Trusted-device behaviour can also be set as config properties (for example in `config/app.php`).

| Limit | Default | Purpose |
| --- | --- | --- |
| Token lifetime (`TOKEN_TTL`) | 5 minutes | How long the user has to complete the challenge |
| Incorrect codes (`MAX_VERIFICATION_ATTEMPTS`) | 5 | After this the token is deleted and they must sign in again |
| Resends per token (`MAX_RESENDS_PER_TOKEN`) | 3 | How many times they can request another code on the same challenge |
| New tokens per user per hour (`MAX_TOKEN_MINTS_PER_HOUR`) | 5 | Caps how often a *new* challenge is minted. Repeating login while a live token still has at least 30 seconds left *reuses* that token instead of counting another mint |
| Remember this device | 14 days | Lifetime of the privileged cookie when the checkbox is ticked. Override with `MFA_TRUSTED_DEVICE_TTL` (seconds) |

Cookies:

| Cookie | Role |
| --- | --- |
| `mfa-token` | Encrypted salt + token for the current challenge. HttpOnly, Secure, `SameSite=Lax`, token TTL |
| `mfa-is-privileged` | Encrypted hash of the user (site `PRIVATE_KEY`, user id, user salt). Marks the browser as trusted |

A malformed privileged cookie is discarded rather than throwing; the next login is challenged again. Signing out always drops `mfa-token`. It also drops `mfa-is-privileged` unless `MFA_TRUST_SURVIVES_LOGOUT` is `true`.

Expired, malformed, or exhausted tokens send the user back to login with a short explanation. Hitting the hourly mint cap surfaces as “Please wait and try again later.”

## Admin

When editing a **user group**, an **MFA** tab sets that group’s policy.

When editing a **user**, an **MFA** tab shows enrolled methods, the group policy, and lets staff reset a method or change the default.

## Console

```bash
php vendor/nails/module-console/console.php mfa:config
```

| Command | Purpose |
| --- | --- |
| `mfa:config` | Show installed/enabled drivers and group policies |
| `mfa:driver:enable --driver=<package>` | Enable an installed driver |
| `mfa:driver:disable --driver=<package>` | Disable a driver while retaining user enrollments |
| `mfa:driver:setting --driver=<package> [--key=<key> [--value=<value>]]` | Inspect or update driver app settings. Use `--json` for structured values |
| `mfa:group:policy --group=<id-or-slug> [--mode=DISABLED\|OPTIONAL\|REQUIRED]` | Inspect or update a group policy |
| `mfa:user:status --user=<id-email-or-username>` | Show a user's effective policy and enrolled methods |
| `mfa:user:method:add --user=<user> --driver=<package> [--default]` | Enrol a non-interactive driver such as Email |
| `mfa:user:method:remove --user=<user> --driver=<package>` | Remove an enrollment |
| `mfa:user:method:default --user=<user> --driver=<package>` | Change the user's default method |

Omit `--driver`, `--user`, or `--group` in an interactive terminal to be prompted. `--no-interaction` skips prompts and requires those options to be set. Mutating commands request confirmation; pass `--force` for unattended execution.

Drivers which hold a user secret, such as Authenticator, must be enrolled by the user (or an interactive setup flow) so the secret and QR code are delivered directly to them.

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
| `userRequiresChallenge(User $oUser)` | Whether this user’s group policy and enrollments mean they should be challenged |
| `userCanConfigureMethods(User $oUser)` | Whether `/mfa/manage` (or a link to it) is worth showing |
| `setIsPrivileged(User $oUser, bool $bRemember = true)` | Trusts this browser. `$bRemember = false` makes it a session cookie |
| `isPrivileged()` | Reads the privileged cookie for the active user |

To protect a sensitive action (for example changing an email address) you can force a fresh challenge even if the device is remembered:

```php
$oMfa->authenticate(activeUser(), false, true);
```

To offer management from your own account screen:

```php
if ($oMfa->userCanConfigureMethods(activeUser())) {
    // link to siteUrl('mfa/manage')
}
```

## Logging

MFA writes to `application/logs/mfa-YYYY-MM-DD.php`. Each request gets a `uniqid()` in the line format so you can follow one sign-in attempt. The logger is the `Logger` service on the MFA module and can be overloaded as `App\MFA\Service\Logger`.

## Customising behaviour

Services, models, and resources follow the usual [overloading](../../key-concepts/factory/overloading.md) convention:

* `App\MFA\Service\MultiFactorAuth`
* `App\MFA\Service\AuthenticationDriver`
* `App\MFA\Service\Logger`
* `App\MFA\Model\Token`
* `App\MFA\Model\GroupPolicy`
* `App\MFA\Model\UserMethod`
* `App\MFA\Resource\Token`
* `App\MFA\Resource\GroupPolicy`
* `App\MFA\Resource\UserMethod`

To change how a code is delivered, write or configure a [driver](drivers/) rather than forking the module.

## Drivers

{% content-ref url="drivers/" %}
[drivers](drivers/)
{% endcontent-ref %}

{% content-ref url="drivers/email.md" %}
[email.md](drivers/email.md)
{% endcontent-ref %}

{% content-ref url="drivers/authenticator.md" %}
[authenticator.md](drivers/authenticator.md)
{% endcontent-ref %}
