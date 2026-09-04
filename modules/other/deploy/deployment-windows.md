---
description: >-
  Deployment windows are defined periods of time where it is acceptable to
  deploy the application.
---

# Deployment Windows

Defining deployment windows allows your application explicltiy declare periods of times where it is acceptable/permitted to deploy.

The following command can be used to determine if the current time falls within a permitted deployment window, and if not, can be used to signal that a deployment should be aborted:

```bash
nails deploy:window
```

This command will discover [defined deployment windows](./#defining-deployment-windows) for the environment and will exit with either a `0` or `1` exit code if a deployment is within a window, or not. Your deployment process canuse this output to determine whetehr to continue with the deployment or to abort.

{% hint style="info" %}
The tool leans towards allowing deployments. If no windows are defined for a given environment it assumes that there should be no restrictions.
{% endhint %}

{% hint style="warning" %}
This tool in and of itself does not impact whether your deployment process continues or not - it is up to you to make that decision using the tool's output.
{% endhint %}

## Defining Deployment Windows

The console commands `nails deploy:window` and `deploy:window:list` will look for window classes in the `App\Deploy\Window` namespace which implement the `Nails\Deploy\Interfaces\Window` interface.

A sample deployment window configuration which restricts `PRODUCTION` deployments to within business hours (9am - 5pm, Monday to Friday) might look like this:

```php
namespace App\Deploy\Window;

use Nails\Deploy\Interfaces\Window;
use Nails\Environment;

class BusinessHours implements Window
{
    public function getEnvironments(): array
    {
        // An array of environments the window applies to
        // If empty then window applies to all environments
        return [Environment::ENV_PROD];
    }

    public function getDays(): array
    {
        // An array of days the window applies to
        // If empty, then window applies every day
        return ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'];
    }

    public function getOpen(): ?string
    {
        // The time the window opens, in 24 hour format HH:MM:SS
        // If null, defaults to 00:00:00
        return '09:00:00';
    }

    public function getClose(): ?string
    {
        // The time the window closes, in 24 hour format HH:MM:SS
        // If null, defaults to 23:59:59
        return '17:00:00';
    }
}

```

{% hint style="info" %}
Note that the tool simply determines if the current time falls within the boundaries of any defined window, it does not perform sanity checking on times, timezones, or overlaps.
{% endhint %}

## Listing Deployment Windows

>

List configured deployment windows for all environments using:

```bash
nails deploy:window:list
```
