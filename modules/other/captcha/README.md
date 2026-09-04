---
description: >-
  The Captcha module provides applications with a simple, unified API for
  protecting forms from bots.
---

# Captcha

## Installation

Install this module using composer:

```bash
compsoer require nails/module-captcha
```

## Configuration

This module is configured in [Admin](../../admin/). From the Captcha settings page you can select your desired driver, and configure it accordingly.

## Usage

Implementing a captcha check on a form is a two step process:

1. Display a challenge to the user on the frontend.
2. Verify the challenge upon form submission in the backend

### Generating a captcha

To generate a new captcha call the `Captcha` service's `generate()` method. This will return a new instance of `Nails\Captcha\Factory\CaptchaForm` which exposes two methods: `getHtml()` and `getLabel()` – you can use these in your forms.

```php
use Nails\Captcha\Service\Catcha;
use Nails\Factory;

/** @var Captcha $oCaptcha */
$oCaptcha = Factory::service('Captcha', 'nails/module-captcha');
echo $oCaptcha->generate()->getHtml();
```

Include this within your form's open/close tags and submit along with the rest of your data.

### Verifying a captcha

To verify a captcha use the service's `verify(string $sToken = null):bool` method:

```php
use Nails\Captcha\Service\Catcha;
use Nails\Common\Exception\ValidationException;
use Nails\Factory;

/** @var Captcha $oCaptcha */
$oCaptcha = Factory::service('Captcha', 'nails/module-captcha');

if (!$oCaptcha->verify()) {
    throw new ValidationException('Failed captcha test');
}
```

{% hint style="info" %}
By default, drivers will inspect the request's `$_POST` data, but you should you need to you can pass the token directly to the `verify()` method.
{% endhint %}

### Helpers

The module provides helper functions for the above, which may be easier to use in templates:

* `captchaGenerate(): Nails\Captcha\Factory\CaptchaForm`
* `captchaVerify(string $sToken = null):bool`

## Drivers

The actual logic for implementing and verifying captchas is provided by drivers.

{% content-ref url="drivers/google-recaptcha.md" %}
[google-recaptcha.md](drivers/google-recaptcha.md)
{% endcontent-ref %}

{% hint style="warning" %}
Note that the module does not provide any driver by default.
{% endhint %}



*
