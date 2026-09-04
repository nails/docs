---
description: A centralised, semantic class for feeding information back to the user.
---

# User Feedback

The user feedback service provides a handful of semantic methods for feeding information back to the user. This class is made available globally throughout [controllers](../key-concepts/controllers.md) via the `$this->oUserFeedback` property but can be loaded at any point using:

```php
use Nails\Factory;
use Nails\Common\Service\UserFeedback;

/** @var UserFeedback $oUSerFeedback */
$oUserFeedback = Factory::service('UserFeedback');
```

## Setting messages

The following setter methods are available to you:

```php
$oUserFeedback->success('An action was successful');
$oUserFeedback->error('An error occurred');
$oUserFeedback->warning('Something needs attention');
$oUserFeedback->info('Something to be aware of');
```

{% hint style="info" %}
Messages will persist through a single redirect when calling `redirect()` which makes them useful for success/error messages when doing `POST` actions which end with a redirect.
{% endhint %}

## Rendering messages

To render messages you can make use of the getter methods:

```php
$oUserFeedback->getSuccess();
$oUserFeedback->getError();
$oUserFeedback->getWarning();
$oUserFeedback->getInfo();

// The above are shortcut methods for the following, where $sType
// is one of the UserFeedback::TYPE_* constants
$oUserFeedback->get($sType);
```

These methods return a `Nails\Common\Factory\Service\UserFeedback\Message` object which when cast as a string will return the appropriate message.

You can combine the above with the service's `getTypes()` method to make a dynamic alert block for your app:

```php
foreach ($oUserFeedback->getTypes() as $sType) {

    $sMessage = (string) $oUserFeedback->get($sType);
    
    if (!empty($sMessage)) {
        echo sprintf(
            '<p class="alert alert--%s">%s</p>',
            strtolower($sType),
            $sMessage
        );
    }
}
```
