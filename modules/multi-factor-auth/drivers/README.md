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

## Writing your own

Implement the interface:

```php
namespace Nails\MFA\Interfaces\Authentication;

use Nails\Common\Service\UserFeedback;
use Nails\MFA\Resource\Token;

interface Driver
{
    public function getLabel(): string;

    public function getDescription(): string;

    public function preForm(Token $oToken, UserFeedback $oUserFeedback): void;

    public function postForm(Token $oToken): void;

    public function validate(Token $oToken, string $sCode): void;

    public function canTryAgain(): bool;
}
```

| Method | When it runs |
| --- | --- |
| `getLabel()` / `getDescription()` | Shown in Admin and on the verification page context |
| `preForm()` | GET (and any non-verify/restart POST) of `/mfa`, **before** the form is rendered. Send codes, push notifications, etc. here |
| `postForm()` | Immediately after the form is rendered. Optional cleanup |
| `validate()` | After the user submits **Verify**. Throw `Nails\MFA\Exception\InvalidCodeException` if the code is wrong — the message is shown on the form |
| `canTryAgain()` | When `true`, the form includes **Request another verification code**, which deletes the token and mints a new one |

Store per-challenge secrets on the token with `$oToken->setData()` / `$oToken->getData()`, not in the session. The token already belongs to one user, expires, and is deleted on success or failure.

The controller records a failed attempt **before** it calls `validate()`, so parallel submits cannot bypass the attempt cap. `validate()` should only compare the code.

Register the package in `composer.json` `extra.nails`, following the [Email driver](https://github.com/nails/driver-multi-factor-auth-email) as a template, then [enable it](../#enabling-a-driver).
