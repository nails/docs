---
description: This plugin provides an interface for making items sortable.
---

# Sortable

To make elements sortable (using a drag and drop interface) add the class `js-admin-sortable` to a containing element. Immediate children will be made sortable.

## Configuration

The following data attributes are available to configure the Sortable item:

| Attribute          | Description                                                      | Default  |
| ------------------ | ---------------------------------------------------------------- | -------- |
| `data-handle`      | Defines a selector to use as the handle (i.e what an be grabbed) | `null`   |
| `data-axis`        | What axis the sorter will work on.                               | `y`      |
| `data-containment` | What container boundaries the sorters will be restricted to.     | `parent` |

## Saving the order

After sorting has stopped, the plugin will look for items with the class `js-admin-sortable__order` and iterate over them setting their value to their index. Typically these items would be hidden form fields which can be used to communicate the explicit order to the backend.

## A working example

The following example shows a `ul` with two items. These can be sorted using the `.handle` element, and after sorting each items `order` value will be updated.

```markup
<ul class="js-admin-sortable" data-handle=".handle">
    <li>
        <div class="handle"></div>
        <input type="hidden" name="books[0][id]" value="123">
        <input type="hidden" name="books[0][order]" class="js-admin-sortable__order" value="0">
        <input type="hidden" name="books[0][label]" value="Treasure Island">
    </li>
    <li>
        <div class="handle"></div>
        <input type="hidden" name="books[1][id]" value="123">
        <input type="hidden" name="books[1][order]" class="js-admin-sortable__order" value="1">
        <input type="hidden" name="books[1][label]" value="1984">
    </li>
</ul>
```

{% hint style="info" %}
This plugin works well with the [Dynamic Table](dynamic-table.md) and [Repeater](repeater.md) plugins.
{% endhint %}
