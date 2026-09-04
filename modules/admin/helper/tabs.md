---
description: An overview of how to sue Admin's tab helper to generate tabbed content.
---

# Tabs

Generate tabs using the `tabs()` static method on the `\Nails\Admin\Helper` class.

## Defining Tabs

The `tabs()` method accepts an array of tab definitions as it's first argument, eachd efinitoon contains two properties: `label` and `content`.

The `label` property is what will appear in the tab itself, where as the `content` property is the contents which is revealed when the tab is clicked.

```php
echo \Nails\Admin\Helper::tabs([
    [
        'label'   => 'Tab Number 1',
        'content' => 'The contents of tab 1'
    ],
    [
        'label'   => 'Tab Number 2',
        'content' => 'The contents of tab 2'
    ],
t]);
```

{% hint style="info" %}
The `content` element can also be a `\Closure` with the closure's return value being what is rendered.
{% endhint %}

## Tab Groups

If multiple tabs appear on a page they can be grouped using the second argument of the `tabs` method. Tabs which share a group are controlled together, i.e clicking `Tab Number 2` in any group will activate that tab in all groups.

By default, all tab groups work independently.

## Tabs and POST

Each tab group contains a hidden input element which stores the current tab. This is pre-populated using `POST` data and defines which tab to default to. This allows the tab group to open at the same tab it was on when the containing form was submitted. The primary use case here is submitting a form and the page reopening having failed validation, we'd like the user to stay on the same tab to avoid confusion.

## Tabs and errors

By default the tab which is revealed by default is the first in the list. However, if any of the tabs contain validation errors (i.e. elements with the `.error` class) then the tab button will highlight this and the user will be taken to the first tab containing an error.
