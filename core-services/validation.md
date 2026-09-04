---
description: >-
  The FormValidation service validates arbitrary data sets against a set of
  rules. It has no dependency on the request, so it works in web requests, on
  the command line and in tests.
---

# Validation

The `FormValidation` service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\FormValidation;
use Nails\Factory;

/** @var FormValidation $oValidation */
$oValidation = Factory::service('FormValidation');
```

Validation is performed by a `Validator` object: a set of rules keyed by field, some optional messages and labels, and the data to check. You can build one on the fly with `buildValidator()`, or define one as a class (see [Validator classes](validation.md#validator-classes)).

## Quick start

```php
use Nails\Common\Exception\ValidationException;
use Nails\Common\Service\FormValidation;
use Nails\Factory;

/** @var FormValidation $oValidation */
$oValidation = Factory::service('FormValidation');

try {

    $oValidator = $oValidation
        ->buildValidator(
            //  The rules, keyed by field
            [
                'name'  => [FormValidation::RULE_REQUIRED, FormValidation::rule(FormValidation::RULE_MAX_LENGTH, 150)],
                'email' => [FormValidation::RULE_REQUIRED, FormValidation::RULE_VALID_EMAIL],
                'age'   => [FormValidation::RULE_INTEGER],
            ],
            //  Optional: message overrides, keyed by rule
            [
                FormValidation::RULE_INTEGER => 'Whole numbers only, please.',
            ],
            //  Optional: the data to validate; defaults to $_POST
            $aData
        )
        ->run();

    //  Valid. Use the validated data rather than the raw input: rules such as
    //  `trim` mutate values, and the validated copy carries those changes.
    $aClean = $oValidator->getValidatedData();

} catch (ValidationException $e) {

    //  $e->getMessage() is a generic "please check the highlighted fields" line;
    //  $e->getData() is [field => message] for each failing field.
    $aErrors = $e->getData();
}
```

`run()` accepts a data array as its argument if you'd rather supply the data at run time. On success it returns the validator; on failure it throws `Nails\Common\Exception\ValidationException`, with each field's first failing message available through `getData()` on the exception and `getErrors()` on the validator.

{% hint style="info" %}
`buildValidatorFromModel($oModel)` builds a validator from a model's `describeFields()` definitions, which is how the admin's default controllers validate create/edit forms.
{% endhint %}

## Rules

Each field takes an array of rules (a pipe-separated string such as `'required|max_length[150]'` also works). A rule may be any of:

| Form | Example |
| ---- | ------- |
| A rule name | `'required'`, `'max_length[150]'` |
| A `RULE_*` constant on `FormValidation` | `FormValidation::RULE_VALID_EMAIL` |
| A constant with parameters, via `rule()` | `FormValidation::rule(FormValidation::RULE_IN_LIST, 'a,b,c')` |
| A rule class name | `\Nails\Common\Validation\Rule\Required::class` |
| A rule instance | `new Required()` |
| A closure | `function ($mValue, Context $oContext) { ... }` |

`FormValidation::rule()` joins any additional arguments with a `.`, so `rule(RULE_IS_UNIQUE, 'user', 'email', 5)` produces `is_unique[user.email.5]`. Each rule documents how it reads its parameter; most take a single value, some split it themselves (see the [rule reference](validation.md#rule-reference)).

### Closures

A closure receives the value and a `Nails\Common\Validation\Context`. It fails by throwing a `ValidationException`, whose message is used verbatim, or by returning `false`. Any other outcome passes.

```php
use Nails\Common\Exception\ValidationException;
use Nails\Common\Validation\Context;

'password' => [
    function ($sPassword, Context $oContext) use ($oPasswordModel) {
        $iGroupId = $oContext->getValue('group_id');
        if (!$oPasswordModel->isAcceptable($iGroupId, $sPassword)) {
            throw new ValidationException('Password does not meet the group\'s requirements.');
        }
    },
],
```

The `Context` is how a rule sees beyond its own value:

| Method | Purpose |
| ------ | ------- |
| `getField()`, `getLabel()` | The field being validated and its label |
| `getParam()`, `getParams('.')` | The raw text inside `[...]`, or split on a delimiter |
| `getValue('other')`, `hasField('other')` | Read another field from the data being validated (after any mutation) |
| `getData()` | The whole data set |
| `getIndex()` | The element key, when validating one element of an array value |
| `setValue($mNew)` | Replace the value (see mutating rules below) |
| `translate('key')` | Look up a language line |

{% hint style="warning" %}
Closures run even when the value is empty, so guard for that yourself if the field is optional. Named rules other than `required`, `isset` and `matches` are skipped for empty values.
{% endhint %}

### How rules run

* Rules run in the order given, except `required` and `isset`, which always run first.
* A field stops at its first failing rule; one message per field is reported.
* Empty values (`null` or `''`) skip every rule except `required`, `isset`, `matches` and closures. An optional field therefore needs no special handling: `['valid_email']` passes when blank.
* Bracketed field names are supported: `'tags[]'` validates each element of the array in turn, `'address[0][postcode]'` addresses a nested value. A rule that accepts whole arrays (`is_array`, `item_count`) receives the array itself.
* Some rules mutate the value rather than test it (`trim`, `prep_url`, `strip_image_tags`, `encode_php_tags`, `prep_for_form`). Later rules see the mutated value, and `getValidatedData()` returns the data with those changes applied. The original input is never modified.
* An unknown rule name throws `Nails\Common\Validation\Exception\UnknownRuleException` rather than failing the field silently. A name that matches a plain PHP function taking one argument (for example `strtolower`) is called as a rule; a short list of dangerous functions is refused.

## Messages

When a rule fails, the message is the first of:

1. A per-field, per-rule message, via `setFieldMessages(['email' => ['required' => '...']])`.
2. A per-rule message, via the second argument to `buildValidator()` or `setMessages(['required' => '...'])`.
3. The `fv_<rule>` language line (for example `fv_required`), which apps may override in their own `nails_lang.php`.
4. The rule's built-in default.

Messages may contain `{field}` and `{param}`; `{field}` is the field's label, or its name if no label was given via `setLabels()`. Messages thrown from closures are used as-is.

```php
$oValidation
    ->buildValidator(['dob' => [FormValidation::RULE_VALID_DATE]])
    ->setLabels(['dob' => 'Date of birth'])
    ->setFieldMessages(['dob' => [FormValidation::RULE_VALID_DATE => 'Please give your {field} as YYYY-MM-DD.']])
    ->run();
```

## Validator classes

`Validator` is designed to be extended. Give a rule set a class when the same rules are used in more than one place, when the rules depend on something worth naming (a model, a service, the record being edited), or when you want to unit test them. A single form with a handful of fields is better left inline.

```php
namespace App\Validator\User;

use Nails\Common\Factory\Service\FormValidation\Validator;
use Nails\Common\Service\FormValidation;
use Nails\Factory;

class Identity extends Validator
{
    //  Dependencies are constructor arguments. Make them optional and resolve them
    //  from the Factory when absent, so production code needs no wiring and tests
    //  can pass doubles.
    public function __construct(private readonly ?int $iIgnoreUserId = null)
    {
        parent::__construct();
    }

    protected function rules(): array
    {
        $oUserModel = Factory::model('User', 'nails/module-auth');

        return [
            'email' => [
                'trim',
                FormValidation::RULE_REQUIRED,
                FormValidation::RULE_VALID_EMAIL,
                $this->iIgnoreUserId
                    ? FormValidation::rule(FormValidation::RULE_IS_UNIQUE, $oUserModel->getTableName(), 'email', $this->iIgnoreUserId)
                    : FormValidation::rule(FormValidation::RULE_IS_UNIQUE, $oUserModel->getTableName(), 'email'),
            ],
        ];
    }

    protected function messages(): array
    {
        return [
            FormValidation::RULE_IS_UNIQUE => 'That email address is already registered.',
        ];
    }
}
```

Override `rules()` (required) and, as needed, `messages()`, `labels()` and `fieldMessages()`. Compute rules in `rules()` rather than the constructor, so constructing a validator is free and configuration is read when it runs. A validator must never read the request itself; the caller supplies the data.

Callers construct it with `new` and pass the data to `run()`. Anything set at runtime is merged over the class-defined values, per field, so a caller can extend or override the class's rules:

```php
(new Identity($oUser->id))
    ->addRules(['first_name' => [FormValidation::RULE_REQUIRED]])   // add fields
    ->setRules(['username' => []])                                   // switch a class-defined field off
    ->run($oInput->post());
```

Class hierarchies work as you'd expect: a subclass can call `parent::rules()` and merge its own.

### Testing a validator

Because the engine is independent of CodeIgniter, a validator can be run in a plain PHPUnit test against an array. The test suite only needs Nails booted, which `tests/bootstrap.php` does with:

```php
require 'vendor/autoload.php';

\Nails\Testing::bootstrapModule(__FILE__);
```

(along with a `PRIVATE_KEY` constant defined in `phpunit.xml`). That makes the Factory, services, models and language files available without CodeIgniter or a database.

Rules that need a database (`is_unique`, `unique_if_diff`, `is_id`) can be replaced for a single validator with `stubRule()`. The stub keeps the real rule's name, aliases, empty-value behaviour and default message; it receives the same `Context`, so you can assert the parameters the rule was built with:

```php
use Nails\Common\Validation\Context;

$oValidator = (new Identity())
    ->stubRule(FormValidation::RULE_IS_UNIQUE, function ($mValue, Context $oContext) {
        [$sTable, $sColumn] = $oContext->getParams();
        return $mValue !== 'taken@example.com';
    });

try {
    $oValidator->run(['email' => 'taken@example.com']);
    self::fail('Expected a ValidationException');
} catch (ValidationException $e) {
    self::assertSame(['email'], array_keys($e->getData()));
}
```

`setEngine()` swaps the whole engine if you need more control.

## Custom rules

Rules are classes implementing `Nails\Common\Interfaces\Validation\Rule`; most extend `Nails\Common\Validation\AbstractRule`, which only asks for a name, a default message and `apply()`. They are discovered automatically from the `Validation\Rule` namespace of every component: `App\Validation\Rule\*` in the app, `Nails\Cdn\Validation\Rule\*` in a module, and so on. No registration is needed.

```php
namespace App\Validation\Rule;

use Nails\Common\Validation\AbstractRule;
use Nails\Common\Validation\Context;

/**
 * Rule: `valid_sku[prefix]` — the value must be a SKU beginning with the given prefix
 */
class ValidSku extends AbstractRule
{
    public const NAME            = 'valid_sku';
    public const DEFAULT_MESSAGE = 'The {field} field must be a valid SKU beginning with {param}.';

    public function apply(mixed $mValue, Context $oContext): bool
    {
        $sPrefix = $oContext->getParam() ?? '';
        return (bool) preg_match('/^' . preg_quote($sPrefix, '/') . '\d{6}$/', (string) $mValue);
    }
}
```

Once the class exists, `'valid_sku[ABC]'` works anywhere a rule name does. Add an `fv_valid_sku` line to your language file to make the message translatable.

The interface's other methods have sensible defaults on `AbstractRule` and may be overridden:

| Method | Default | Override when |
| ------ | ------- | ------------- |
| `getAliases()` | `[]` | The rule should answer to other names too |
| `runsOnEmpty()` | `false` | The rule must run for empty values, as `required` does |
| `acceptsArrays()` | `false` | The rule wants the whole array rather than each element, as `item_count` does |

Rules may fail by returning `false` (the resolved message is used), or by throwing a `ValidationException` to supply a specific message. A rule that transforms its value calls `$oContext->setValue($mNew)` and returns `true`.

Names are global. When two components declare the same name, the app wins over drivers and skins, which win over modules, which win over `nails/common`; declaring `App\Validation\Rule\ValidEmail` with the name `valid_email` therefore replaces the built-in everywhere.

## Views

The `set_value()`, `form_error()`, `set_select()`, `set_radio()` and `set_checkbox()` helpers read the most recent validation run, so a view rendered after a failed `run()` repopulates fields and shows inline errors without any extra wiring. When no validation has run they fall back to `$_POST`.

## Legacy API

Before the current engine, `FormValidation` wrapped CodeIgniter's form validation library and exposed its methods (`set_rules()`, `set_message()`, `set_data()`, `run()`, `error_array()`, `callback_*` rules, and so on). These continue to work but are deprecated and will be removed in a future major version; convert them to `buildValidator()` or a validator class. Two behaviours differ from the old library: closures no longer run before named rules, and an unknown rule name throws rather than silently failing the field.

## Rule reference

Parameters go inside square brackets after the name, for example `max_length[150]`. Rules marked *mutates* change the value rather than test it. Rules marked *empty* run even when the value is empty; all others are skipped for empty values.

| Rule | Parameter | Description |
| ---- | --------- | ----------- |
| `required` | | The value must not be empty. *Empty.* |
| `isset` | | The value must be present, even if empty. *Empty.* |
| `matches` | Another field | The value must equal the other field's value. *Empty.* |
| `differs` | Another field | The value must not equal the other field's value. |
| `is` | A value | The value must be exactly the parameter. |
| `in_list` | Comma-separated values | The value must be one of the list. |
| `in_range` | `low-high` | The value must be a number within the range. |
| `min_length` | Integer | Minimum string length. |
| `max_length` | Integer | Maximum string length. |
| `exact_length` | Integer | Exact string length. |
| `maxWords` (alias `max_words`) | Integer | Maximum number of words. |
| `regex_match` | A regular expression | The value must match the expression. |
| `alpha` | | Letters only. |
| `alpha_numeric` | | Letters and digits only. |
| `alpha_numeric_spaces` | | Letters, digits and spaces only. |
| `alpha_dash` | | Letters, digits, underscores and dashes only. |
| `alpha_dash_period` | | As `alpha_dash`, plus periods. |
| `numeric` | | Any number, including signed decimals. |
| `integer` | | A whole number, optionally signed. |
| `decimal` | | A decimal number with a fractional part. |
| `is_natural` | | A non-negative whole number (0, 1, 2 ...). |
| `is_natural_no_zero` | | A positive whole number (1, 2, 3 ...). |
| `is_bool` | | `true`, `false`, `1`, `0`, `'1'` or `'0'`. |
| `greater_than` | Number | The value must be greater than the parameter. |
| `greater_than_equal_to` | Number | The value must be greater than or equal to the parameter. |
| `less_than` | Number | The value must be less than the parameter. |
| `less_than_equal_to` | Number | The value must be less than or equal to the parameter. |
| `valid_email` | | A single email address. |
| `valid_emails` | | A list of email addresses separated by commas, semicolons or new lines. |
| `valid_url` | | A URL. |
| `valid_ip` | `ipv4` or `ipv6` (optional) | An IP address. |
| `valid_mac` | | A MAC address. |
| `valid_base64` | | A Base64-encoded string. |
| `valid_postcode` | | A UK postcode. |
| `validTimecode` | | A timecode in `hh:mm:ss` format. |
| `supportedLocale` | | One of the app's supported locales. |
| `valid_date` | Format (default `Y-m-d`) | A date in the given format. |
| `date_future` | Format | A date after today. |
| `date_past` | Format | A date before today. |
| `date_today` | Format | Today's date. |
| `date_before` | `other_field.format` | A date before the date in another field. |
| `date_after` | `other_field.format` | A date after the date in another field. |
| `valid_datetime` | Format (default `Y-m-d H:i:s`) | A date and time in the given format. |
| `datetime_future` | Format | A date and time in the future. |
| `datetime_past` | Format | A date and time in the past. |
| `datetime_before` | `other_field.format` | Before the date and time in another field. |
| `datetime_after` | `other_field.format` | After the date and time in another field. |
| `valid_time` | Format (default `H:i:s`) | A time in the given format. |
| `time_future` | Format | A time later today. |
| `time_past` | Format | A time earlier today. |
| `time_before` | `other_field.format` | Before the time in another field. |
| `time_after` | `other_field.format` | After the time in another field. |
| `is_unique` | `table.column.ignore_id.ignore_column` | The value must not already exist in `table.column`, optionally ignoring one row (`ignore_column` defaults to `id`). |
| `unique_if_diff` | `table.column.old_value` | As `is_unique`, but only checked when the value differs from `old_value`. |
| `is_id` | `Model.provider` | The value must be the ID of an existing item in the model (provider defaults to `app`). |
| `is_array` | | The value must be an array; also marks the field as accepting whole arrays. |
| `item_count` | `min,max` | The array must contain between `min` and `max` items; wrap in parentheses, `item_count[(0,5)]`, for exclusive bounds. |
| `trim` | | *Mutates:* trims whitespace. |
| `prep_url` | | *Mutates:* prefixes `http://` when no scheme is present. |
| `prep_for_form` | | *Mutates:* escapes quotes and angle brackets for re-rendering in a form. |
| `strip_image_tags` | | *Mutates:* replaces `<img>` tags with their `src`. |
| `encode_php_tags` | | *Mutates:* encodes `<?` and `?>`. |

Modules may add rules of their own; for example `nails/module-cdn` provides `cdnObjectPickerMultiObjectRequired`, `cdnObjectPickerMultiLabelRequired` and `cdnObjectPickerMultiAllRequired` for its multi-object picker.
