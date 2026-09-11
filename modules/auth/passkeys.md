---
description: >-
  Let users sign in with a fingerprint, face, screen lock, or security key
  instead of a password.
---

# Passkeys

A passkey is a WebAuthn credential stored by the user's device or password manager. The private key never leaves the authenticator and the browser will only offer it back to the site that created it, so a passkey cannot be phished, guessed, reused, or leaked in a database breach.

Auth stores passkeys, runs both WebAuthn ceremonies, and exposes them in three places: the login page, a self-service page at `auth/passkeys`, and a tab in Admin when editing a user.

{% hint style="info" %}
A passkey login where the user verified themselves (biometric, PIN, or screen lock) also satisfies a [Multi-Factor Auth](../multi-factor-auth/) challenge, so those users are not asked for a second factor. The user has already presented two factors: the device, and whatever unlocked it.
{% endhint %}

## Requirements

* `ext-openssl` and `ext-mbstring`. `ext-sodium` is optional, and only needed for authenticators which use Ed25519 keys.
* A **secure context**: HTTPS, or `localhost` for development. Browsers refuse WebAuthn anywhere else.
* A single host per app. See [Relying Party ID](#relying-party-id).

## Enabling

Run [migrations](../../core-services/database/migrations.md) so the `user_passkey` table exists, then turn passkeys on in Admin under **Settings → Authentication → Login**.

Nothing changes for existing users until they register a passkey; password login is untouched.

## What users see

{% stepper %}
{% step %}
### They sign in with a password

If they have no passkey yet, and this browser has not been asked before, they are offered one on a short interstitial page. Declining sets a cookie and they are not asked again on that browser.
{% endstep %}

{% step %}
### They add a passkey

From the nudge, or at any time from `auth/passkeys`. They can name each one so they can tell their laptop from their phone.
{% endstep %}

{% step %}
### They sign in with it

The login page shows a **Sign in with a passkey** button, and the identifier field offers saved passkeys in its own autofill dropdown. Either route signs them straight in; no username or password is typed.
{% endstep %}
{% endstepper %}

Users manage their own passkeys at `auth/passkeys`: rename, remove, and see when each was last used. Passkeys which sync through the user's password manager are marked, so they know which ones exist on more than one device.

## Relying Party ID

The Relying Party (RP) ID is the domain a passkey is bound to. It defaults to the host of `BASE_URL` and is shown, read-only, on the settings page.

{% hint style="danger" %}
Changing the RP ID invalidates every passkey already registered. Authenticators bind credentials to the RP ID, so after a change the browser will not offer the old ones back and users must register again.
{% endhint %}

Override it only when you have a good reason, such as sharing passkeys across `app.example.com` and `www.example.com` by setting the RP ID to the registrable parent domain:

```php
// config/app.php
define('AUTH_PASSKEY_RP_ID', 'example.com');
```

Ceremonies are also checked against a list of permitted origins, derived from `BASE_URL` and `SECURE_BASE_URL`. Add more if the app is served from another origin:

```php
define('AUTH_PASSKEY_ALLOWED_ORIGINS', ['https://app.example.com']);
```

## Overridden views

An app which has replaced `auth/views/login/form.php` owns its own markup and assets, so the module does not add the button or the JavaScript to it. Opt back in with the `passkey` helper:

```php
<?php

loadPasskeyAssets();

echo passkeyLoginButton($return_to);
```

`passkeysEnabled()` and `passkeyRegisterButton()` are available too. Each returns an empty string when passkeys are disabled, so they are safe to call unconditionally.

For the button to work, the surrounding `<form>` needs `id="login-form"` and `data-passkey-site-url` (the value of `siteUrl()`); the identifier input needs `autocomplete="username webauthn"` and `data-passkey-conditional` for autofill.

## Admin

Editing a user shows a **Passkeys** tab listing their credentials, when each was added and last used, and a checkbox to revoke. Revoking is immediate and cannot be undone; the user must register the device again.

Registrations and revocations are recorded as `did_add_passkey` and `did_remove_passkey` user events, and a passkey login is recorded as `did_log_in` with `provider` set to `passkey`.

## API

The browser talks to these endpoints; they are not intended as a public API. All of them require a JSON request body and a same-origin request, and all return 404 when passkeys are disabled.

| Endpoint | Auth | Purpose |
| --- | --- | --- |
| `GET api/auth/passkey/index` | Logged in | List the user's passkeys |
| `POST api/auth/passkey/register` | Logged in | Begin registration |
| `POST api/auth/passkey/attest` | Logged in | Finish registration |
| `POST api/auth/passkey/challenge` | Public | Begin sign-in |
| `POST api/auth/passkey/assert` | Public | Finish sign-in |
| `POST api/auth/passkey/rename` | Logged in | Rename one |
| `POST api/auth/passkey/revoke` | Logged in | Remove one |

## In code

The `Passkey` service is the entry point. The pure ceremony methods touch neither the database nor the session, so they can be unit tested directly.

```php
use Nails\Auth\Constants;
use Nails\Auth\Service\Passkey;
use Nails\Factory;

/** @var Passkey $oPasskey */
$oPasskey = Factory::service('Passkey', Constants::MODULE_SLUG);

if ($oPasskey->isEnabled()) {
    $oOptions = $oPasskey->createRegistrationOptions($oUser);
}
```

Every failure extends `Nails\Auth\Exception\Passkey\PasskeyException`, so a single catch covers a whole ceremony. Catch the subclasses when you need to tell a stale challenge from a bad signature.

To sign a user in from an assertion you have already collected, use `Authentication::loginWithPasskey()`, which applies the same lockout and suspension rules as password login.

## Notes on the design

* **Sign counters.** Authenticators which keep a counter have it checked on every login; one that fails to advance signals a cloned credential and the login is refused. Many platform authenticators keep no counter and always report zero; those are accepted, as the specification intends.
* **Public keys are stored in plaintext.** A public key is public by definition, and encrypting it would tie every passkey to `PRIVATE_KEY` rotation.
* **User handles are derived, not stored on the user.** The handle the authenticator sees is an HMAC of the user's ID, so no column was added to the `user` table. It is written to each passkey row at registration, so rotating `PRIVATE_KEY` does not break credentials which already exist.
* **No captcha on passkey login.** A passkey is phishing-resistant and the same lockout rules apply, so a captcha adds friction without adding protection.
* **A passkey login skips the temporary and expired password checks**, because no password took part in it.
