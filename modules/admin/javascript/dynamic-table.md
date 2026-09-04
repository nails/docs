---
description: >-
  This plugin provides a mechanism for creating tables which can have rows added
  or removed by the user. Very useful for generating forms which the user can
  control.
---

# Dynamic Table

Dynamic tables are comprised of four main components:

* [The table container](dynamic-table.md#the-container)
* [The template](dynamic-table.md#the-template)
* [The body](dynamic-table.md#the-body)
* [Row controls](dynamic-table.md#row-controls)

## The container

The container element should be an element with the class `js-admin-dynamic-table`, typically this will be a `<table>` element, however this is not a requirement. Feel free to include normal table elements (e.g. `<thead>`) if it makes sense for your use-case:

```markup
<table class="js-admin-dynamic-table">
    <!-- other elements excluded for brevity -->
    <thead>
        <tr>
            <th>Label</th>
            <th>Description</th>
        </tr>
    </thead>
</table>
```

## The template

The template is the markup which is repeated for each row, it should be a non-rendering child element of the container, e.g. a `<script>` element. It should be given the class `js-admin-dynamic-table__template`.

```markup
<table class="js-admin-dynamic-table">
    <!-- other elements excluded for brevity -->
    <script type="text/x-template" class="js-admin-dynamic-table__template">
    <tr>
        <td>
            <input name="foo[{{index}}][label]" value="{{label}}"/>
        </td>
        <td>
            <input name="foo[{{index}}][description]" value="{{description}}"/>
        </td>
    </tr>
    </script>
</table>
```

{% hint style="info" %}
Each time the template is rendered it will be parsed using [Mustache](https://mustache.github.io) syntax; an `{{index}}` property will always be made available, along with any [custom data you provide](dynamic-table.md#populating-with-data).
{% endhint %}

## The body

The body is the target element where rows will be rendered, this should be given the class `js-admin-dynamic-table__body`.

```markup
<table class="js-admin-dynamic-table">
    <!-- other elements excluded for brevity -->
    <tbody class="js-admin-dynamic-table__body"></tbody>
</table>
```

## Row Controls

### Add a row

Any element with the class `js-admin-dynamic-table__add` will, when clicked, add a new row to the body. Typically this will be in the `<tfoot>`:

```markup
<table class="js-admin-dynamic-table">
    <!-- other elements excluded for brevity -->
    <tfoot>
        <tr>
            <td colspan="2">
                <button class="js-admin-dynamic-table__add">
                    Add Row
                </button>
            </td>
        </tr>
    </tfoot>
</table>
```

### Remove a row

Any element contained within a row with the class `js-admin-dynamic-table__remove` will remove the closes `<tr>` element when clicked. A template with a remove button might look like this:

```markup
<table class="js-admin-dynamic-table">
    <!-- other elements excluded for brevity -->
    <script type="text/x-template">
    <tr>
        <td>
            <input type="text" name="foo[{{index}}][label]" value="{{label}}" />
        </td>
        <td>
            <input type="text" name="foo[{{index}}][description]" value="{{description}}" />
        </td>
        <td>
            <button class="js-admin-dynamic-table__remove">
                &times;
            </button>
        </td>
    </tr>
    </script>
</table>
```

## Populating with data

To populate the table with data you can provide a `data` data attribute on the container. This should be a JSON array of objects, each will represent a row and be avaialble to the rendering of the template:

```php
<?php

$aData = [
    ['foo' => 'bar', 'fizz' => 'buzz'],
    ['foo' => 'bar', 'fizz' => 'buzz'],
];

$sData = json_encode($aData);

// Important to escape quotes
$sData = htmlspecialchars($aData);

?>
<table class="js-admin-dynamic-table" data-data="<?=$sData?>">
    <!-- other elements excluded for brevity -->
</table>
```

## Sorting

This plugin plays well with the [Sortable plugin](sortable.md). Add the class `js-admin-sortable` to the root element to allow the user to sort the records.

## Complete Example

This is a complete working example of a Dynamic Table, which also uses the Sortable plugin:

```php
<?php

$aData = [
    ['id' => 1, 'order' => 0, 'label' => 'foo', 'description' => 'bar'],
    ['id' => 2, 'order' => 1, 'label' => 'foo', 'description' => 'bar'],
];

$sData = json_encode($aData);
$sData = htmlspecialchars($aData);

?>
<table class="js-admin-dynamic-table" data-data="<?=$sData?>">
    <thead>
        <tr>
            <th colspan="2">Label</th>
            <th colspan="2">Description</th>
        </tr>
    </thead>

    <!-- the body tag plus sortable plugin, configured to use .handle for dragging -->
    <tbody class="js-admin-dynamic-table__body js-admin-sortable" data-handle=".handle"></tbody>

    <!-- tfoot element for adding new rows -->
    <tfoot>
        <tr>
            <td colspan="4">
                <button class="btn btn-sm btn-success js-admin-dynamic-table__add">
                    &plus; Add Item
                </button>
            </td>
        </tr>
    </tfoot>

    <!-- the row template -->
    <script type="text-template" class="js-admin-dynamic-table__template">
        <tr>
            <!-- sorting handle and hidden fields for item ID and sort order -->
            <td class="handle" style="width: 15px;">
                <b class="fa fa-bars"></b>
                <input type="hidden" name="foo[{{index}}][id]" value="{{id}}" />
                <input type="hidden" name="foo[{{index}}][order]" value="{{order}}" />
            </td>

            <!-- label and description cells -->
            <td>
                <input type="text" name="foo[{{index}}][label]" value="{{label}}" />
            </td>
            <td>
                <input type="text" name="foo[{{index}}][label]" value="{{label}}" />
            </td>

            <!-- remove row -->
            <td style="width: 15px;">
                <button class="btn btn-sm btn-danger js-admin-dynamic-table__remove">
                    &times;
                </button>
            </td>
        </tr>
    </script>
</table>
```
