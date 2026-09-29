---
description: Timezone-aware datetime pickers in admin.
---

# DateTime

Admin binds jQuery UI datepicker / timepicker onto `input.date`, `input.datetime`, and `input.time`. `form_field_date()`, `form_field_datetime()`, and `form_field_time()` emit those classes.

## Timezone-aware fields

A datetime is timezone-aware only when you opt in. The picker is still dumb: it posts whatever string is in the box. Your save handler converts that string from the user's timezone to the app timezone.

```php
echo form_field_datetime([
    'key'           => 'date_published',
    'label'         => 'Schedule',
    'default'       => $oItem->date_published
        ? toUserDatetime((string) $oItem->date_published, 'Y-m-d H:i:s')
        : '',
    'timezoneAware' => true,
    'tip'           => 'Leave blank to publish immediately.',
]);
```

`timezoneAware => true` makes `form_field_datetime()`:

* Add `data-timezone-aware`, `data-user-timezone`, and `data-app-timezone` on the input
* Add `.timezone-aware` on the field row
* Sit a one-line channel under the input naming the user's timezone in English (for example `Europe - London`). The name is a [confirm](confirm.md) link (`hint--top hint--medium`) to their account edit screen. Confirm copy: “Continue to edit your account and update your timezone.” / “You will lose unsaved changes.”
* Queue the IANA → English catalogue once per page via the Asset service as `window.NAILS.TIMEZONE_CATALOGUE` for the picker

Use `tip` for field-specific guidance (when the date applies, blank means immediately, and so on). Do not repeat the timezone in `info`.

## Alternate clocks

When a timezone-aware picker opens, the popup can list other timezones and the same instant in each. The list is stored in `localStorage` under `nails.admin.datetime.alternateTimezones`. If that key is missing, the list starts empty. The user's own timezone is not listed again. Add or remove zones from a searchable Select2 dropdown; the choice survives reloads. UTC is labelled `UTC/GMT`.

The input string is read as wall time in `data-user-timezone`, not the browser zone. Changing the date or time in the picker refreshes those clocks; the footer is remounted after datepicker redraws the popup.

{% hint style="info" %}
Do not enable `timezoneAware` on every datetime. Created/modified stamps and values already stored in the app timezone should keep the plain picker.
{% endhint %}
