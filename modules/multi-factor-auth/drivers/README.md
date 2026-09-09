---
description: >-
  MFA drivers implement the second factor — sending a code, prompting an app,
  and checking what the user submits.
---

# Drivers

The MFA module only orchestrates sign-in. A driver is what actually challenges the user.

Drivers are Composer packages of type `driver` with `subType` `authentication` and `forModule` `nails/module-multi-factor-auth`. They must implement `Nails\MFA\Interfaces\Authentication\Driver`.

## Official drivers

{% content-ref url="email.md" %}
[email.md](email.md)
{% endcontent-ref %}

{% content-ref url="authenticator.md" %}
[authenticator.md](authenticator.md)
{% endcontent-ref %}

## Writing your own

Implement the interface:

```php
namespace Nails\MFA\Interfaces\Authentication;

use Nails\Auth\Resource\User;
use Nails\Common\Service\UserFeedback;
use Nails\MFA\Resource\Token;
use Nails\MFA\Resource\UserMethod;
use stdClass;

interface Driver
{
    public function getLabel(): string;

    public function getDescription(): string;

    public function getSetupDescription(): string;

    public function preForm(Token $oToken, UserFeedback $oUserFeedback): void;

    public function postForm(Token $oToken): void;

    public function validate(Token $oToken, string $sCode): void;

    public function canTryAgain(): bool;

    public function resend(Token $oToken, UserFeedback $oUserFeedback): void;

    public function requiresEnrollment(): bool;

    public function setupStart(User $oUser): stdClass;

    public function setupComplete(User $oUser, string $sCode, stdClass $oPending): stdClass;

    public function reset(User $oUser, UserMethod $oMethod): void;
}
```

| Method | When it runs |
| --- | --- |
| `getLabel()` | Admin labels, chooser headings, and button copy |
| `getDescription()` | Admin / configuration copy, written as if describing the driver to a third party |
| `getSetupDescription()` | First-person copy on the setup chooser (“Use an authenticator app…”) |
| `preForm()` | GET (and any non-verify/resend POST) of `/mfa`, **before** the form is rendered. Send codes, push notifications, etc. here |
| `postForm()` | Immediately after the form is rendered. Optional cleanup |
| `validate()` | After the user submits **Verify**. Throw `Nails\MFA\Exception\InvalidCodeException` if the code is wrong — the message is shown on the form |
| `canTryAgain()` | When `true`, the form includes **Request another verification code** |
| `resend()` | That button’s action. Reissue a code on the **same** token; do not mint a new challenge. Only called when `canTryAgain()` is `true` |
| `requiresEnrollment()` | When `true`, the user must complete setup (QR code, secret, …) before the driver can validate. Email is `false`; Authenticator is `true` |
| `setupStart()` | Begin enrollment. Return data for the setup view (secret, QR SVG, and so on) |
| `setupComplete()` | Confirm enrollment with a code. Return the payload stored on the user-method row |
| `reset()` | Called when a method is removed, before the row is deleted |

Store per-challenge secrets on the token with `$oToken->setData()` / `$oToken->getData()`, not in the session. The token already belongs to one user, expires, and is deleted on success or failure.

Long-lived secrets (an authenticator seed, a hardware key id) belong on the `UserMethod` row via `setupComplete()`, not on the token.

The controller records a failed attempt **before** it calls `validate()`, so parallel submits cannot bypass the attempt cap. `validate()` should only compare the code.

Register the package in `composer.json` `extra.nails`, following the [Email driver](https://github.com/nails/driver-multi-factor-auth-email) as a template, then [enable it](../#enabling-a-driver).
