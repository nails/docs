---
description: Understanding the request lifecycle of a typical application.
---

# Request Lifecycle

The following diagram shows a simplified version of how a request is handled by Nails, and the various steps it goes through before the output is returned to the browser:

```
Incoming Request (GET, POST, PUT, DELETE, etc)

↓
index.php

↓
Nails\Bootstrap::run();
    ↳ Configuration
    ↳ Set up Error Handler
    ↳ Set up Factory
    ↳ Auto-load

↓
Routing
    ↳ Resolve component
    ↳ Resolve controller

↓
️Execute Controller
    ↳ Base controller
    ↳ App Base Controller
    ↳ Execute controller method

↓
Output to browser
    ↳ Headers
    ↳ Compiled Views

↓
Nails\Bootstrap::shutdown();
```

## Bootstrap

The Nails Bootstrapper is primarily responsible for setting up the environment. It loads all the configuration files as well as auto loads any [Factory](key-concepts/factory/) items.

Once the Bootstrapper has run you are guaranteed to have your configs set as well access to anything which is loaded via the Factory,  that is almost every thing.

## Routing

The router determines which component is responsible for satisfying the request, and which controller therein should be constructed.

{% content-ref url="key-concepts/routing.md" %}
[routing.md](key-concepts/routing.md)
{% endcontent-ref %}

## Controller

Most controllers are children of the App's `App\Controller\Base` class, which in turn is a child of the `Nails\Common\Controller\Base` class.&#x20;

### Nails\Common\Controller\Base::\_\_construct()

The base controller is responsible for setting up the response. It initiates various post-system-ready behaviour and performs common functions which are applicable on every response.

### App\Controller\Base::construct()

The App base controller is the app's hook into the request lifecycle, and the first point of the request which is not internal. Here the application can perform global functions appropriate for the app, such as [load global CSS, JS](core-services/asset.md), or  query the [CMS module's for the primary menu](modules/cms/menus.md).

### Resolved Controller

Finally, the resolved controller's constructor is called and then the method which is handling the response is executed.

## Output

Throughout execution any [Views](key-concepts/views.md) which have been compiled are kept in a buffer and sent to the browser, along with any headers, at the end of the request.

## Events

Throughout the lifecycle various events are triggered, all of which can be listened to and used to add additional behaviour.

{% content-ref url="core-services/event.md" %}
[event.md](core-services/event.md)
{% endcontent-ref %}

