---
description: This plugin provides a simple way of generating repeating interfaces.
---

# Repeater

The Repeater plugin allows you to create dynamic, repeatable interfaces for your site administrators.

The Repeater is comprised of 4 main elements:

1. [The container](repeater.md#the-container)
2. [The template](repeater.md#the-template)
3. [The target](repeater.md#the-target)
4. [The controls](repeater.md#the-controls)

## The Container

The container defines the boundaries of the repeater, and is typically a div with the trigger class: `js-admin-repeater`.

```markup
<div class="js-admin-repeater"></div>
```

All of the following items must be contained within the container.

## The Template

The template is the part of the repeater which is _repeated_. Each time a new item is added the template is rendered and then added to the DOM.&#x20;

The template is defined using the following class: `js-admin-repeater__template`.

```markup
<script type="text/template" class="js-admin-repeater__template">
    <!-- this mark up will be repeated -->
</script>
```

{% hint style="warning" %}
Note the use of `<script type="text/template">`. This ensures that the template is interpreted literally and not parsed by the DOM, which can cause unintended side-effects.
{% endhint %}

The template will be parsed using [Mustache](https://github.com/janl/mustache.js), with the template's index being available using `{{index}}`. See [Loading Data](repeater.md#loading-data) for more information on working with data objects.

{% hint style="info" %}
Each time the template is rendered the admin's UI is refreshed, so templates can also contain other complex behaviours, such as [Dynamic Tables](dynamic-table.md), or [Searchers](searcher.md).
{% endhint %}

## The Target

The target is where new instances of the template will be rendered and should be given the class `js-admin-repeater__target`. This is expected to be a `<ul>`.

```markup
<ul class="js-admin-repeater__target"></ul>
```

{% hint style="success" %}
Make your objects sortable by using the [Sortable](sortable.md) plugin.
{% endhint %}

## The Controls

The main interaction points for the repeater are the controls which add and remove instances.

### Adding

A new template will be rendered each time the DOM element with the class `js-admin-repeater__add` is clicked. Typically this will be a button.

```markup
<button class="js-admin-repeater__add">
    &plus; Add
</button>
```

### Removing

DOM elements with the class `js-admin-repeater__remove` will remove the instance they reside in when clicked, i.e these controls should be placed within [the template](repeater.md#the-template).

```markup
<button class="js-admin-repeater__remove">
    &times; Remove
</button>
```

## Loading Data

Often you will have data saved which you would like to use to populate each instance. The repeater will accept a JSON array of objects via a `data-data` attribute on [the container](repeater.md#the-container), with a new instance being rendered for each object in the array.

The data can be accessed using [Mustache](https://github.com/janl/mustache.js) notation.

## A working example

The following is a complete working example which renders a simple form, pre-loaded with some saved data.

```php
<?php

$aItems = [
    (object) [
        'label'  => 'Treasure Island',
        'author' => 'Robert Louis Stephenson',
    ],
    (object) [
        'label'  => '1984',
        'author' => 'George Orwell',
    ],
];

?>
<div class="js-admin-repeater" data-data="<?=htmlspecialchars(json_encode($aItems))?>">

    <div class="js-admin-repeater__template">
        <input type="hidden" name="books[{{index}}][id]" value="{{id}}" />
        <input type="text" name="books[{{index}}][label]" value="{{label}}" />
        <input type="text" name="books[{{index}}][author]" value="{{author}}" />
        <button class="js-admin-repeater__remove">
            &plus; Remove
        </button>
    </div>

    <ul class="js-admin-repeater__target"></ul>

    <button class="js-admin-repeater__add">
        &plus; Add
    </button>
</div>
```

The above example, the resulting form is a property called `books` which is a multi-dimensional array. The `{{index}}` property ensures that each input in the rendered instance is within the same scope.
