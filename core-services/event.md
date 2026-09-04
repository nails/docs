---
description: >-
  The Event service is responsible for broadcasting and listening for events
  which occur during the application's runtime.
---

# Event

Events in Nails are a way of broadcasting to other parts of the system that something happened, and provides an interface for code to react. For example; you may wish to update a mailing list database every time a user is created, or send a notification to an administrator whenever somebody makes a purchase.

Your code can subscribe to events triggered by either installed modules or other parts of your app easily using the Event service.

The Event service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\Event;
use Nails\Factory;

/** @var Event $oEvent */
$oEvent = Factory::service('Event');
```

## Announcing Events

In order for an event to be useable, it first must be announced. This is simply a case of creating a class in the root namespace of the component called `Events`. This class must extend `Nails\Common\Events\Base` and is simply a collection of constants and DocBlocs.

```php
namespace App;

use Nails\Common\Events\Base;

class Events extends Base
{
    /**
     * Fired when a user yells
     *
     * @param integer $iUserId  The User's ID
     * @param string  $sYelling What they're yelling
     */
    const USER_YELL = 'USER_YELL';
    
    /**
     * Fired when a user whispers
     *
     * @param integer $iUserId     The User's ID
     * @param string  $swhispering What they're whispering
     */
    const USER_WHISPER = 'USER_WHISPER';
}
```

In the above example, we're declaring two events (`\App\Events::USER_YELL` and `\App\Events::USER_WHISPER`), as well as providing some documentation about when the event is fired and what arguments are passed to subscribers. The DocBloc is parsed and is what is used by the `nails events:list` [console command](../modules/console.md).

{% hint style="info" %}
The values of the constants do not matter so long as they're unique for the class
{% endhint %}

## Triggering Events

Events are triggered using the `trigger($sEvent, $sNamespace, $aData)` method on the `Event` service. Using the above declaration as an example:

```php
use Nails\Common\Service\Event;
use Nails\Factory;

/** @var Event $oEventService */
$oEventService = Factory::service('Event');
$oEventService->trigger(
    \App\Events::USER_YELL,
    \App\Events::getEventNamespace(),
    [$iUserId, $sYelling]
);
```

Each listener subscribed to the `\App\Events::USER_YELL` event in the `app` namespace will be called (in the order they were subscribed) and will be passed `$iUserId` and `$sYelling` as two separate arguments.

## Subscribing to Events

### Automatic

To automatically subscribe to events on boot, you should add the `autoload()` method to `\App\Events`. This method should return an of instances which extend the `Nails\Common\Events\Subscription` object.

Additionally, you can also configure the subscription to only fire once, even if the event is triggered many times.

#### Example

```php
namespace App;

use App\Event\Listener;
use Nails\Factory;

class Events
{
    public function autoload(): array
    {
        return [
            new Listener\User\Yelled(),
            new Listener\User\Whispered(),
            new Listener\System\Ready(),
        ];
    }
}
```

Example event listeners for the above:

```php
namespace App\Event\Listener\User;

use App\Events;
use Nails\Common\Events\Subscription;

class Yelled extends Subscription
{
    public function __construct()
    {
        $this
            ->setEvent(Events::USER_YELL)
            ->setNamespace(Events::getEventNamespace())
            ->setCallback([$this, 'execute']);
    }

    public function execute(int $iUserId, string $sYelling): void
    {
        //  The user with ID $iUserId yelled "$sYelling"
    }
}
```

```php
namespace App\Event\Listener\User;

use App\Events;
use Nails\Common\Events\Subscription;

class Whispered extends Subscription
{
    public function __construct()
    {
        $this
            ->setEvent(Events::USER_WHISPERED)
            ->setNamespace(Events::getEventNamespace())
            ->setCallback([$this, 'execute']);
    }

    public function execute(int $iUserId, string $sWhispering): void
    {
        //  The user with ID $iUserId whispered "$sWhispering"
    }
}
```

```php
namespace App\Event\Listener\System;

use Nails\Common\Events;
use Nails\Common\Events\Subscription;

class Ready extends Subscription
{
    public function __construct()
    {
        $this
            ->setEvent(Events::SYSTEM_READY)
            ->setNamespace(Events::getEventNamespace())
            ->setCallback([$this, 'execute']);
    }

    public function execute(): void
    {
        //  The system is ready
    }
}
```

{% hint style="info" %}
You can subscribe the same callback to multiple events in the same namespace by passing an array of events to the `setEvent()` method.
{% endhint %}

### Manual

If you only need a subscription to be attached in certain circumstances, then you can add a new subscription on-the-fly using the `Event` service's `subscribe()` method.

```php
$oEventService = Factory::service('Event');
$oEventService->subscribe(
    \App\Events::USER_YELL,
    'app',
    [$this, 'myCallback'],
    true // Only subscribe once
);
```

## Model Events

Models automatically trigger events for their `create()`, `update()`, `delete()`, and `destroy()` methods. You can subscribe to them in the same way as the above and use the model's `::gtEventNamespace()` method to set the namespace. The event names are available as constants on the model itself.

Example model event listener:

```php
namespace App\Event\Listener\MyModel;

use App\Model\Books;
use Nails\Common\Events\Subscription;

class Created extends Subscription
{
    public function __construct()
    {
        $this
            ->setEvent(Books::EVENT_CREATED)
            ->setNamespace(Books::getEventNamespace())
            ->setCallback([$this, 'execute']);
    }

    public function execute(int $iId): void
    {
        //  The model "Books" created a new item
    }
}
```

All events will pass the item's ID to the handler, with the `EVENT_DELETE` method also passing the deleted item as the second parameter.
