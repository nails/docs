---
description: Tab groups for sectioned admin screens.
---

# Tabs

The Tabs plugin turns a control list and a matching set of panels into a tab group. Use it for sectioned forms (Details / Content / SEO). Prefer [`Helper::tabs()`](../helper/tabs.md) rather than writing this markup by hand.

## Markup

A group is three pieces that share a `data-tabgroup` value:

```markup
<input type="hidden" data-tabgroup="tab-group" name="tab-group" value="" />

<ul class="tabs" data-tabgroup="tab-group">
    <li class="tab">
        <a href="#" data-tab="tab-details">Details</a>
    </li>
    <li class="tab">
        <a href="#" data-tab="tab-content">Content</a>
    </li>
</ul>

<section class="tabs" data-tabgroup="tab-group">
    <div class="tab-page tab-details">
        <!-- details fields -->
    </div>
    <div class="tab-page tab-content">
        <!-- content fields -->
    </div>
</section>
```

* Controls live in `ul.tabs > li.tab > a[data-tab]`. The plugin puts `.active` on the `<li>`.
* Panels live in `section.tabs > div.tab-page`. The panel whose class matches the control’s `data-tab` is shown.
* The hidden input stores the active tab so a failed form POST reopens the same tab.

If `data-tabgroup` is omitted, the plugin assigns a unique group per list/section pair.

## Groups

Two lists that share the same `data-tabgroup` stay in sync. Clicking “Content” in either list activates that panel in every group with that name. Independent groups need different `data-tabgroup` values.

## Default tab

On load the plugin shows, in order:

1. The first panel that contains a validation error
2. The tab stored in the hidden input
3. The first tab

A control whose panel contains an error gets `.error` on the `<a>` (rendered as a small indicator). Errors are `.field.error`, `.alert.alert-danger`, `.system-alert.error`, and `.error.show-in-tabs`.

Changing tab calls `refreshUi()` on the admin controller so plugins such as [Select](select.md) can bind fields that were hidden.

## Segmented tabs

Add `tabs--segmented` on the `<ul>` (and on the `<section>` if it is not the next sibling) for a pill switcher whose panels have no card of their own. Use that when a heavier [screen-tab](screen-tabs.md) row already sits above these tabs.

```markup
<ul class="tabs tabs--segmented" data-tabgroup="language">
    <li class="tab">
        <a href="#" data-tab="tab-en">English</a>
    </li>
</ul>
```

`Helper::tabs()` emits the default card style. Add `tabs--segmented` yourself when you write the markup.

## Overflow

A long tab bar scrolls horizontally. The active tab is scrolled into view; the underline stays inside the scroll container.
