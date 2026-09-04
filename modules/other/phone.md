---
description: >-
  This module provides a unified API for formatting and validating phone
  numbers.
---

# Phone

This module provides a simple way of validating and verifying phone numbers for various geographic areas.

The module provides the `Nails\Phone\Service\Phone` service, which is the simplest way of interacting with the parsers, formatters and validators.

The Phone service can be loaded like so:

```php
use Nails\Phone;
use Nails\Factory;

/** @var Phone\Service\Phone $oService */
$oService = Factory::service('Phone', Phone\Constants::MODULE_SLUG);
```

## Supported Countries

Phone numbers can be parsed, formatted and validated for multiple countries. The currently supported countries are:

| Country        | ISO Code |
| -------------- | -------- |
| United Kingdom | GB       |
| USA            | US       |

{% hint style="info" %}
A is possible to create your own [parsers](phone.md#parsing), [formatters](phone.md#formatting), and [validators](phone.md#validating) by supplying instances with the appropriate interface.
{% endhint %}

## Parsing

To parse a phone number into its discreet parts use the Phone service's `parse(string $sNumber)` method.

```php
use Nails\Phone;

/** @var Resource\Phone $oPhone */
$oPhone = $oService
    ->parse(
        '0207 1234 5678', 
        Phone\Constants::COUNTRY_GB
    );
```

The parse method accepts two parameters: the number to parse, and the ISO code pf country to parse for, i.e the format in which the supplied number will be supplied in.

This function will return a `Nails\Phone\Resource\Phone` resource. These resources expose two methods: `format()` and `validate()` which can be used to [format](phone.md#formatting-a-phone-number) and [validate](phone.md#validating-a-phone-number) the number respectively.

{% hint style="info" %}
Supply your own parser by implementing the `Nails\Phone\Interfaces\Parser` interface.
{% endhint %}

If you wish to supply the different segments of the number explicitly you can use the Phone service's `build()` method:

```php
/** @var Resource\Phone $oPhone */
$oPhone = $oService
    ->build(
        'GB',      // The country the number belongs to
        44,        // The coutry calling code
        '0207',    // The area segment
        '7296043', // The local segment
        '123'      // The extension
    );
```

## Formatting

Once you have a Phone resource you can format it in various ways depending on your required need:

### As a string

Formats the number in accordance with the local style:

```php
echo (string) $oPhone->formatted();
echo (string) $oPhone;

//  +44 (0) 207 729 6043
```

### As a URL

Formats the number using the `tel://` protocol, suitable for links:

```php
echo $oPhone->formatted()->asUrl();

//  tel://+442077296043
```

### As an array

If you need the discreet components you can format as an array:

```php
echo $oPhone->formatted()->asArray();

//  [
//    'country'      => 'GB',
//    'country_code' => 44,
//    'area'         => '0207',
//    'local'        => '729',
//    'extension'    => '6043',
//  ]
```

### As JSON

Returns the phone number as a JSON object:

```php
echo $oPhone->formatted()->asJson();

//  {
//    "country": "GB",
//    "country_code": 44,
//    "area": "0207",
//    "local": "729",
//    "extension": "6043",
//  }
```

{% hint style="info" %}
Supply your own formatter by implementing the `Nails\Phone\Interfaces\Formatter` interface.
{% endhint %}

## Validating

Once you have a Phone resource, you can validate it by calling either the `isValid(): bool` method, or the `validate()` method. The former will return  a boolean as to whether the Phone object is valid, the latter will throw a `Nails\Phone\Exception\ValidationException` if it is invalid.

```php
use Nails\Phone\Exception\ValidationException;

if (!$oPhone->isValid()) {
    echo 'Invalid phone number';
}

try {

    $oPhone->validate();

} catch (ValidationException $e) {
}
```
