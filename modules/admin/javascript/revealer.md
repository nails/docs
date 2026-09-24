---
description: >-
  A plugin which triggers showing and hiding of elements based on the value of a
  control.
---

# Revealer

This plugin allows you to bind the visibility of DOM elements based on the value of `checkbox` and `select` controls.

This functionality is achieved through the use of the `data-revealer` data attribute, where the value is an arbitrary string which groups controls and elements together.

{% hint style="info" %}
Revealer groups must be unique, i.e two controls must cannot share the same group name.
{% endhint %}

For elements which are not a control (i.e not a `checkbox`, or `select`) then the second data attribute `data-reveal-on` is required. This attribute specifies for which value the element should be shown (it is hidden on all non-matching values).

## The Control

### Checkboxes

A checkbox control element will look something like this:

```markup
<input type="checkbox" data-revealer="group-1">
```

### Selects

A select control element will look something like this:

```markup
<select data-revealer="group-1">
    <option value="OPTION_1">Option 1</option>
    <option value="OPTION_2">Option 2</option>
</select>
```

## The Elements

Each element which is to bind to a control must define a `data-revealer` attribute which matches the control, as well as a `data-reveal-on` attribute which defines what value the element will be revealed for.

{% hint style="info" %}
For `checkbox` controls, the only valid values are `true` and `false`.
{% endhint %}

### Multiple values (OR)

`data-reveal-on` (and `data-reveal-not-on`) may list more than one value, separated by commas (or `data-reveal-delimiter`). The element is shown if the control’s current value matches **any** of them.

```markup
<select data-revealer="type">
    <option value="1">Page</option>
    <option value="2">Redirect</option>
    <option value="3">iFrame</option>
</select>

<div data-revealer="type" data-reveal-on="1,3">
    <!-- shown when the selected option is 1 or 3, not 2 -->
</div>
```

### Multiple groups (OR)

An element may list more than one group in `data-revealer`, separated by commas (or `data-reveal-delimiter` if you have changed the delimiter). The element is shown if **any** of those groups currently matches `data-reveal-on`.

Controls still need unique group names; only the target element lists multiple groups.

```markup
<input type="checkbox" data-revealer="password">
<input type="checkbox" data-revealer="user-details">

<div data-revealer="password,user-details" data-reveal-on="true">
    <!-- shown when either checkbox is checked -->
</div>
```

## A working example

### Checkbox

```markup
<!-- The control -->
<input type="checkbox" data-revealer="group-1">

<!-- The elements-->
<div data-revealer="group-1" data-reveal-on="true">
    <!-- shown when the checkbox is checked -->
</div>

<div data-revealer="group-1" data-reveal-on="false">
    <!-- shown when the checkbox is NOT checked -->
</div>
```

### Select

```markup
<!-- The control -->
<select data-revealer="group-1">
    <option value="OPTION_1">Option 1</option>
    <option value="OPTION_2">Option 2</option>
</select>

<!-- The elements -->
<div data-revealer="group-1" data-reveal-on="OPTION_1">
    <!-- shown when the selected option is OPTION_1 -->
</div>

<div data-revealer="group-1" data-reveal-on="OPTION_2">
    <!-- shown when the selected option is OPTION_2 -->
</div>
```

