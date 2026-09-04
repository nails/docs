---
description: >-
  MailChimp audiences, historically known as lists, are the objects which
  contain your mailing list subscribers.
---

# Audiences

The Audience API allows you to list, create, update, and delete audiences.

{% hint style="info" %}
See [Members](members.md) for how to manage the subscribers in an audience.
{% endhint %}

## Getting the API

The Audience API can be retrieved using the [Client's](./#the-client) `audiences` method:

```php
use Nails\MailChimp;

/** @var Mailchimp\Factory\Audience **/
$oAudienceApi = $oClient->audiences();
```

## Listing all Audiences

To list all audiences in the configured account use the Audience API's `getAll` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Audience[] */
$aAudiences = $oAudienceApi->getAll();
```

## Get a single Audience

To get a single Audience, use the Audience API's `getById(string $sId)` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Audience */
$oAudience = $oAudienceApi->getById('{AUDIENCE_ID}');
```

## Create a new Audience

To create a new Audience, use the Audience API's `create(array $aParameters)` method:

{% hint style="info" %}
See the [MailChimp docs](https://mailchimp.com/developer/reference/lists/) for an overview of the fields which you can pass to this method.
{% endhint %}

```php
use Nails\MailChimp;

/** MailChimp\Resource\Audience */
$oAudience = $oAudienceApi->create([
    'name'                => 'My Audience',
    'contact'             => [
        'company'  => 'An Example Company',
        'address1' => '123 Main street',
        'city'     => 'Glasgow',
        'state'    => 'Glasgow',
        'zip'      => 'G20 8LR',
        'country'  => 'Scotland',
    ],
    'permission_reminder' => 'This is a test list, you were added by mistake',
    'campaign_defaults'   => [
        'from_name'  => 'Test Name',
        'from_email' => 'module-mailchimp@nailsapp.co.uk',
        'subject'    => 'This is a test',
        'language'   => 'en-gb',
    ],
    'email_type_option'   => false,
]);
```

## Update an Audience

To update an existing Audience use the Audience API's `update` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Audience */
$oAudience = $oAudienceApi->update(
    '{AUDIENCE_ID}',
    [
        'name' => 'My Audience - Updated',
    ]
);
```

## Delete an Audience

To delete an existing Audience use the Audience API's `delete` method:

```php
use Nails\MailChimp;

$oAudienceApi->delete('{AUDIENCE_ID}');
```
