---
description: This module provides a complete and powerful admin UI.
---

# Admin

Admin is the back-office UI. Controllers under `/admin` load a shared chrome: sidebar, topbar, [forms](forms.md), and a plugin system for JS widgets.

Typical edit screens use:

* [DefaultController](controllers/default-controller.md) or a custom view
* [Helper::tabs()](helper/tabs.md) for section tabs, or [screen tabs](javascript/screen-tabs.md) for top-level workspaces
* [Form fields](../../../key-concepts/form-fields.md) (`form_field()`, booleans, dropdowns)
* [Floating save](forms.md#floating-save-bar) via `Helper::floatingControls()`, with optional [unsaved-changes](javascript/unsaved-changes.md)

## Housekeeping

Admin ships routines that prune stale sessions, expired data exports, and old changelog rows. See [Housekeeping](housekeeping.md).
