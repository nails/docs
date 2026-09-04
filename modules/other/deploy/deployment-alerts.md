---
description: Send alerts to interested parties before, or after, a deployment.
---

# Deployment Alerts

Send alerts to nominated email addresses when a deployment starts, or completes, by defining deployment alert groups.

The following command can be used to send alerts before and after a deployment, respectively:

```bash
nails deploy:alert:pre
nails deploy:alert:post
```

This command will discover defined alert groups for the environment and will send an alert to each email defined therein.

{% hint style="warning" %}
This tool is a dumb implementation in that it has no context of the actual deployment process. Your deployment tooling should call the appropriate command at the appropriate point.
{% endhint %}

## Defining Deployment Alerts

The following console commands...

```bash
nails deploy:alert:pre
nails deploy:alert:pre:list
nails deploy:alert:post
nails deploy:alert:post:list
```

... will look for alert classes in the `App\Deploy\Alert` namespace which implement the `Nails\Deploy\Interfaces\Alert` interface.

A sample deployment alert definition which sends an email to `sally.ceo@example.com` before and after each deployment might look like this:

```php
namespace App\Deploy\Alert;

use Nails\Deploy\Interfaces\Alert;
use Nails\Environment;

class Admins implements Alert
{
    public function getEnvironments(): array
    {
        // An array of environments the alert applies to
        // If empty then alert applies to all environments
        return [Environment::ENV_PROD];
    }

    public function getEmails(): array
    {
        // An array of email addresses to send the alert to
        // You may wish to populate this automatically, e.g all the admins
        return [
            'sally.ceo@example.com',
        ];
    }

    public function isPre(): bool
    {
        // Whether the group should bind to the :pre commands
        return true;
    }

    public function isPost(): bool
    {
        // Whether the group should bind to the :pre commands
        return true;
    }
}

```

## Listing Deployment Alerts

List configured deployment alerts for all environments using:

```bash
nails deploy:alert:pre:list
nails deploy:alert:post:list
```

## Change Email Copy

The default deployment alert text is very succinct. If you wish to be more verbose in your email copy then you can modify the text in one of two ways:

### In admin

The templates allow template editing via the [admin](../../admin/) interface.

### Override the view

For more granular control of the template, you can overload the views by supplying a view at one or more of the following locations:

```bash
./application/modules/deploy/views/Email/templates/alert/post.php
./application/modules/deploy/views/Email/templates/alert/post_plaintext.php
./application/modules/deploy/views/Email/templates/alert/pre.php
./application/modules/deploy/views/Email/templates/alert/pre_plaintext.php
```
