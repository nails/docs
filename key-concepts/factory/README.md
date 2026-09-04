# Factory

The `\Nails\Factory` class is a core component in Nails, its purpose is to act as a service/instance container and provide a simple API for loading instances of [services](services.md), [models](models/), [resources](resources.md), and [factories](factories.md) from the application itself and also from installed [components](../components/).

## Defining Factory Items

The application, and any installed components define which services, models, resources and factories they provide through the`application/services/services.php` file. This file returns closures which in turn return class instances.

```php
use App\Service;
use App\Model;
use App\Factory;
use App\Resource;

return [
    'services' => [
        'MyService' => function (): Service\MyService {
            return new Service\MyService();
        },
    ],
    'models' => [
        'MyModel' => function (): Model\MyModel {
            return new Model\MyModel();
        },
    ],
    'resources' => [
        'MyResource' => function ($mObject): Resource\MyResource {
            return new Resource\MyResource($mObject);
        },
    ],
    'factories' => [
        'MyFactory' => function (): Factory\MyFactory {
            return new Factory\MyFactory();
        },
    ],
];
```

## Requesting Factory Items

Items are then loaded at runtime using the Factory's static methods:

```php
use Nails\Factory;

//  Returns the same instance every time
Factory::service($sKey, $sProvider);
Factory::model($sKey, $sProvider);

//  Returns a new instance every time
Factory::resource($sKey, $sProvider, $mObject);
Factory::factory($sKey, $sProvider);
```

Where `$sKey` is the key defined in the `services.php` file and `$sProvider` is the name of the [component](../components/) which provides the item; for example if the above example is in the app, then you'd load a model like so:

```php
use App\Model\MyModel;
use Nails\Factory;

/** @var MyModel $oModel */
$oModel = Factory::model('MyModel', 'app');
```

If the above was provided by the CMS module, it would be loaded like this:

```php
use Nails\Cms\Constants;
use Nails\Cms\Model\Bucket;
use Nails\Factory;

/** @var Bucket $oModel */
$oModel = Factory::model('Bucket', Constants::MODULE_SLUG);
```

## Services

Services are single instance classes which provide specific functionality, usually abstracting something (e.g. a third party service).

{% content-ref url="services.md" %}
[services.md](services.md)
{% endcontent-ref %}

## Models

Models are single instance classes which represent data from the database; usually this is a 1-to1 relationship with a specific database table.

{% content-ref url="models/" %}
[models](models/)
{% endcontent-ref %}

## Resources

Resources represent a single item in a collection, typically emitted by models.

{% content-ref url="resources.md" %}
[resources.md](resources.md)
{% endcontent-ref %}

## Factories

Factories are similar to services, but instead of the same instance being returned a new instance is constructed with each request.

{% content-ref url="factories.md" %}
[factories.md](factories.md)
{% endcontent-ref %}

## Overloading

One of the major benefits of using the Factory is the ability to overload items which are provided by installed components.

{% content-ref url="overloading.md" %}
[overloading.md](overloading.md)
{% endcontent-ref %}

