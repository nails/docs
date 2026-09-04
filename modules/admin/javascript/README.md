---
description: >-
  The section covers loading JS in admin, the admin JS plugin system, and the
  bundled plugins.
---

# Javascript

## Auto-loading

Your app can auto-load JS (and CSS) in admin via `composer.json`:

```javascript
{
    // ... the rest of composer.json has been omitted
    "extra": {
        "nails": {
            "data": {
                "nails/module-admin": {
                    "autoload": {
                        "assets": {
                            "js": ["admin.js"],
                            "css": ["admin.css"]
                        }
                    }
                }
            }
        }
    }
}
```

This will load `admin.js` and `admin.css` from the app's assets directory via the [Asset Service](../../../core-services/asset.md).

## Plugins

Admin has a simple plugin interface for loading JS into the UI. A plugin is defined as simply some JS which is loaded and instantiated on each page load in admin, and is [registered](./#registering-plugins) using the Admin JS Controller.

### The Admin JS Object

Admin loads a globally accessible JS object which provides unified API for all plugins, and is available under the `window.NAILS.ADMIN` global variable.

### Registering plugins

Register plugins using the `registerPlugin` method of the admin JS Object. The following is an example of `admin.js` (which might be auto-loaded, as explained above):

```javascript
import MyPlugin from './components/MyPlugin.js';

window.NAILS.ADMIN.registerPlugin(
    'app',         // The namespace in which to register the plugin
    'MyPlugin',    // The name to give your plugin, must be unique in the namespace
    new MyPlugin() // The plugin instance
);
```

### Refreshing the UI

Plugins which manipulate the DOM can request a UI refresh, i.e. announce that there is new UI and provide an opportunity for other plugins to interact accordingly. For example once a new row is created by the [Dynamic Table](dynamic-table.md) plugin, we want the [Select](select.md) plugin to instantiate any new elements. This is achieved through the JS Object's `refreshUI()` method:

```javascript
/**
 * Having performed some action which has added DOM elements
 * your plugin can call refreshUi()
 */

window.NAILS.ADMIN.refreshUi();
```

The above will announce to all plugins that they should look for new UI and bind to it, if applicable.

When writing your own plugins, it is highly recommended to leverage the `refreshUi` event when constructing your plugin via the JS Object's `onRefreshUi()` method:

```javascript
class MyPlugin
{
    constructor(adminController) {
        adminController
            .onRefreshUi(() => {
                this.init();
            })
    }
    
    init() {
        let nodes = document.querySelectorAll('.my-items:not(.processed)');
        [...nodes].forEach((element) => {
        
            // Do something with `element`
            // Prevent the same element being processed again
            element.classList.add('processed');
        });
    }
}

export default MyPlugin;
```

{% hint style="danger" %}
When using `refreshUi` event remember that it might be called multiple times, ensure your plugin won't bind against the same element twice, as is in the example above.
{% endhint %}

## Bundled Plugins

The following plugins are bundled with admin, and are available for you to use:

* [Copy to Clipboard](copy-to-clipboard.md) – easily copy text to the user's clipboard
* [Dynamic Table](dynamic-table.md) – render dynamic tables easily
* [Notes](notes.md) – keep notes about anything
* [Repeater](repeater.md) – build repeatable blocks using templates
* [Searcher](searcher.md) – populate your forms with entities
* [Select](select.md) – searchable drop downs
* [Sortable](sortable.md) – allow drag and drop sorting
* [Tabs](tabs.md) – build tabbed interfaces
