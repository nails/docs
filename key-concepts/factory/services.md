# Services

In Nails services are singleton classes which provide miscellaneous functionality. Typically, services are used to abstract third parties, or to orchestrate high-level application logic.

For example, a developer might create service to facilitate:

* Interaction with the a third party API
* Abstract in-app logic which requires multiple models, e.g. creating, and charging for a user's subscription.

## Where do services live?

Services live at `src/Service` and in the `App\Service` namespace.

## How are services loaded?

Services should be loaded using the `Factory` and are defined within the `services` element of `services.php`. Each time you load a service you are given the same instance.

Define services in `application/services/services.php`:

```php
use App\Service;

return
    'services' => [
        'ThirdPartyApi' => function (): Service\ThirdParty\Api {
            return new Service\ThirdParty\Api();
        },
    ],
];
```

To use a service, you load it using the [Factory](./):

```php
use App\Service;
use Nails\Factory;

/** @var Service\ThirdParty\Api $oApi */
$oApi = Factory::service('ThirdPartyApi', 'app');
```

