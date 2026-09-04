---
description: >-
  The Validation service is provides a simple interface for validating arbitrary
  data sets.
---

# Validation

The `FormValidation` service provides a simple API for validating datasets.

```php
use Nails\Common\Service\FormValidation;
use Nails\Factory;

/** @var FormValidation $oValidation */
$oValidation = Factory::service('FormValidation');
```

There are two distinct ways to validate data:

1. [Procedurally](validation.md#procedural-validation)
2. [Validators](validation.md#validators)

Both are valid, Procedural validation is easier to understand, however Validators are more portable and more object orientated.

## Procedural Validation

In procedural validation you define your rulesets on-the-fly and validate it there and then. Typically you are validating `$_POST` data, so this is the default data source.

A basic example:

```php
$oValidation->setRule(
    'label',
    [
        $oValidation::RULE_REQUIRED,
    ]
);

$oValidation->setRule(
    'email',
    [
        $oValidation::RULE_REQUIRED,
        $oValidation::RULE_VALID_EMAIL,
    ]
);

if ($oValidation->run()) {
    echo 'POST contains valid data';
} else {
    echo 'POST contains INVALID data';
    print_r($oValidation->errors());
}
```

Many rules accept parameters, for example, the `RULE_MAX_LENGTH` accepts an integer specifying what the maximum string length should be. Use the `rule()` static method to build these:

```php
$oValidation->setRule(
    'label',
    [
        $oValidation::rule(
            $oValidation::RULE_MAX_LENGTH,
            150
        )
    ]
);
```

Additionally you can pass `\Closure` objects as rules which will be evaluated and should throw a `\Nails\Common\Exception\ValidationException` exception if the data is invalid.

```php
use Nails\Common\Exception\ValidationException;

$oValidation->setRule(
    'label',
    [
        function($sValue) {
            if ($sValue === 'INVALID') {
                throw new ValidationException(
                    'This field is invalid'
                );
            }
        }
    ]
);
```

## Validator Objects

Validators are objects which wrap up the logic of defining rules and validating them against a dataset. They allow you to define a ruleset and then validate some data against it in an organised, repeatable fashion.

To create a validator on the fly, you may use the `buildValidator()` method:

```php
use Nails\Common\Exception\ValidationException;
use Nails\Common\Factory\Service\FormValidation\Validator;
use Nails\Common\Service\FormValidation;
use Nails\Factory;

/** @var FormValidation $oValidation */
$oValidation = Factory::service('FormValidation');

//  The ruleset as a key value pair
$aRules = [
    'email'    => [
      $oValidation::RULE_REQUIRED,
      $oValidation::RULE_VALID_EMAIL,
      function($mValue) {
          if (preg_match('/.*@example\.com$/', $mValue) === false) {
              throw new ValidationException(
                  'Only @example.com email addresses are allowed.'
              );
          }
      }
    ],
    'username' => [
      $oValidation::RULE_REQUIRED,
      $oValidation::rule(
          $oValidation::RULE_MAX_LENGTH,
          150
      ),
    ],
];

//  Any error message overrides
$aMessages = [
    $oValidation::RULE_REQUIRED => 'This field is required',
];

//  The data to validate
$aData = [
    'email'    => 'baz@example.com',
    'username' => ''
];

/** @var Validator $oValidator */
$oValidator = $oValidation
    ->buildValidator(
        $aRules,
        $aMessages
    );

try {

    $oValidator->run($aData);
    
} catch (ValidationException $e) {
    //  Handle failures
}
```

This will return a `Nails\Common\Factory\Service\FormValidation\Validator` object, which exposes a `run()` method. When executed, if validation fails, a `Nails\Common\Exception\ValidationException` will be thrown with the specific errors being exposed via the exception's `getData()` method as well as via the Validator's `getErrors()` method.

{% hint style="info" %}
The `run()` method accepts an array as the dataset to validate, if no dataset is passed then `$_POST` is assumed.
{% endhint %}

### Portability

Validators can be defined as a class which extends the `Nails\Common\Factory\Service\FormValidation\Validator` class, defining the ruleset therein. This is useful if you wish to centralise a validation object for use throughout the application. For example, you may repeatedly save a similar type of data in different parts of a system, but require the same validation rules to be applied.

For example, consider this validator object:

```php
namespace App\Factory\FormValidation\Validator;

use Nails\Common\Factory\Service\FormValidation\Validator;

class User extends Validator
{
    public function __construct(
        array $aRules = [],
        array $aMessages = [],
        array $aData = []
    ) {
        parent::__construct($aRules, $aMessages, $aData);
        $this->aRules = array_merge(
            $this->aRules,
            [
                'user_name'  => ['required'],
                'user_email' => ['required', 'valid_email'],
                'user_phone' => [],
            ]
        );
    }
}
```

Assuming this is properly defined as a [Factory](../key-concepts/factory/factories.md), the calling code might look like this:

```php
use Nails\Common\Factory\Service\FormValidation\Validator;

/** @var Validator $oValidator */
$oValidator = Factory::factory('FormValidationValidatorUser', 'app');
$oValidator->run();
```

This would pass any `$_POST` data through validation and throw a `ValiationException` exception if validation failed. This approach means your ruleset is defined once, but can be used anywhere. Additionally it is straightforward to use class hierarchy to extend the ruleset even further:

```php
namespace App\Factory\FormValidation\Validator\User;

use App\Factory\FormValidation\Validator\User;

class Administrator extends User
{
    public function __construct(
        array $aRules = [],
        array $aMessages = [],
        array $aData = []
    ) {
        parent::__construct($aRules, $aMessages, $aData);
        $this->aRules = array_merge(
            $this->aRules,
            [
                'password' => ['required', 'min_length[10]'],
            ]
        );
    }
}
```

The above, when instantiated, results in a ruleset which looks like this:

```php
[
    'user_name'  => ['required'],
    'user_email' => ['required', 'valid_email'],
    'user_phone' => [],
    'password'   => ['required', 'min_length[10]'],
]
```

## Rule Reference

The following validation rules are available:

| Rule  | Parameters | Description |
| ----- | ---------- | ----------- |
| @todo | Complete   | This        |
