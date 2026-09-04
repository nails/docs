---
description: Show parts of the admin UI in a modal.
---

# Modalize

This plugin provides an interface for showing parts of the UI in a modal - this is especially useful when the part you wish to show is hidden, for example, in a complex form.

For example, in the following complex form – a video entity - the "cuepoints" interface could quickly become quite messy as more fields are added. We can hide the extra fields behind the edit button, which when clicked will show the fields in a modal.

![The base form, showing minimal fields for the "cuepoints" tab](<../../../.gitbook/assets/Screenshot 2020-01-16 18.12.19.png>)

![The modal, shown once the "edit" button is clicked, containing the additional fields.](<../../../.gitbook/assets/Screenshot 2020-01-16 18.14.17.png>)

## The Button

Define your button using the `js-admin-modalize` class and specify the target's ID in the `data-target-id` attribute.

```markup
<button class="js-admin-modalize" data-target-id="hidden-content">
    Edit
</button>
```

## The Target

There is nothing special required for the target element, just that it exists. It is up to you what you show in the modal.

When shown, the contents of this element will be cloned and shown in the modal where the user is free to make changes. When the modal closes, this element will be _replaced_ with the contents of the modal (unless the cancel button was clicked).

```markup
<div id="hidden-content" class="hidden">
    <!-- any markup, including form elements -->
</div>
```

## A working example

```markup
<!-- @todo - a working example -->
```
