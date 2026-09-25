---
description: This module provides a complete and powerful admin UI.
---

# Admin

The Admin module (`nails/module-admin`) is the back office for a Nails application. It gives you a logged-in area at `/admin` with a sidebar, a dashboard, settings, a change log, data exports, and a consistent set of form and table components, so that each screen you add only has to describe _what_ it manages.

Admin doesn't hold a list of screens. Any installed component (your app or a module) can add screens, permissions, dashboard widgets and more by putting classes in a known namespace. Admin finds them at runtime and wires them in.

## How it works

Every request under `/admin` goes through one route:

1. `Nails\Admin\Routes` sends every `/admin/...` URL to Admin's router.
2. The router checks the request's IP against the optional [IP whitelist](user-permissions.md#restricting-admin-by-ip). If the IP isn't allowed, it returns a 404. It then checks that the user is an admin, meaning their user group has at least one admin permission. Anyone else gets the unauthorised response.
3. The `Controller` service looks through every installed component for classes in its `Admin\Controller` namespace, matches one to the URL, and calls the method.
4. The controller extends [`Base`](controllers/base-controller.md), which loads Admin's CSS, JS and helpers. It then renders a view inside the admin chrome using [`Helper::loadView()`](helper/).

The sidebar is built the same way. Admin calls each discovered controller's static `announce()` method, which says which sidebar group the controller belongs to and which links it adds.

## Where things live

Discovery works by namespace. In your app, the `App\` namespace maps to `src/`, so each extension point is a directory under `src/Admin/`. Modules use their own namespace in the same way (for example `Nails\Blog\Admin\Controller\...`).

| What                                                          | Namespace (app)                             | Location (app)                       | Implements / extends                                     |
| ------------------------------------------------------------- | ------------------------------------------- | ------------------------------------ | -------------------------------------------------------- |
| [Controllers](controllers/)                                   | `App\Admin\Controller`                      | `src/Admin/Controller/`              | `Nails\Admin\Controller\Base` or `DefaultController`     |
| [Permissions](user-permissions.md)                            | `App\Admin\Permission`                      | `src/Admin/Permission/`              | `Nails\Admin\Interfaces\Permission`                      |
| [Dashboard widgets](dashboard-widgets.md)                     | `App\Admin\Dashboard\Widget`                | `src/Admin/Dashboard/Widget/`        | `Nails\Admin\Interfaces\Dashboard\Widget`                |
| [Dashboard alerts](dashboard-widgets.md#dashboard-alerts)     | `App\Admin\Dashboard\Alert`                 | `src/Admin/Dashboard/Alert/`         | `Nails\Admin\Interfaces\Dashboard\Alert`                 |
| [Data export sources](data-export.md#sources)                 | `App\Admin\DataExport\Source`               | `src/Admin/DataExport/Source/`       | `Nails\Admin\Interfaces\DataExport\Source`               |
| [Data export formats](data-export.md#formats)                 | `App\Admin\DataExport\Format`               | `src/Admin/DataExport/Format/`       | `Nails\Admin\Interfaces\DataExport\Format`               |
| [Scheduled exports](data-export.md#scheduled-exports)         | `App\Admin\DataExport\Schedule`             | `src/Admin/DataExport/Schedule/`     | `Nails\Admin\Interfaces\DataExport\Schedule`             |
| "Create" header button                                        | `App\Admin\Ui\Header\Button\Create`         | `src/Admin/Ui/Header/Button/Create/` | `Nails\Admin\Interfaces\Ui\Header\Button\Create`         |
| Header search section                                         | `App\Admin\Ui\Header\Button\Search\Section` | `src/Admin/Ui/Header/Button/Search/Section/` | `Nails\Admin\Interfaces\Ui\Header\Button\Search\Section` |
| Settings pages                                                | `App\Settings`                              | `src/Settings/`                      | `Nails\Common\Interfaces\Component\Settings`             |

Views are the one exception. They are plain PHP templates, not classes, so they sit in `application/modules/admin/views/` rather than in `src/`. See [Views](controllers/#views).

{% hint style="info" %}
The Admin module builds its own screens (Dashboard, Settings, Utilities, Change Log, Help, Styleguide) the same way. `src/Admin/Controller` and `src/Admin/Permission` in the module's repository are good worked examples.
{% endhint %}

## What's in the box

When the module is installed you get:

* **Dashboard.** A per-user grid of [widgets](dashboard-widgets.md) and any active [alerts](dashboard-widgets.md#dashboard-alerts).
* **Settings.** One page for each component that provides a settings class, under the _Settings_ sidebar group.
* **Utilities.** _Export Data_ ([Data Export](data-export.md)) and _Rewrite Routes_.
* **Logs → Change Log.** An audit trail of create, edit, delete and restore actions made through the [DefaultController](controllers/default-controller.md).
* **Styleguide.** A living reference of the admin components. It has no sidebar link; its controller returns `null` from `announce()`, so you reach it by URL.

## Building a screen

Most admin screens are CRUD screens for a model. For those, extend [`DefaultController`](controllers/default-controller.md), point it at the model, and you get index, create, edit, delete, restore, copy and sort screens with no views to write.

For anything else, extend [`Base`](controllers/base-controller.md) and write your own methods and views. You'll typically use:

* [Form fields](../../key-concepts/form-fields.md) (`form_field()`, booleans, dropdowns) inside admin [form chrome](forms.md)
* [`Helper::tabs()`](helper/tabs.md) for section tabs, or [screen tabs](javascript/screen-tabs.md) for top-level workspaces
* [`Helper::floatingControls()`](forms.md#floating-save-bar) for the sticky save bar, with optional [unsaved-changes](javascript/unsaved-changes.md) warnings
* The [JavaScript plugins](javascript/) for repeaters, dynamic tables, modals and more

New to the module? Start with [Your First Admin Page](getting-started.md).

## Configuration

| Constant                         | Default | Purpose                                                                                                   |
| -------------------------------- | ------- | --------------------------------------------------------------------------------------------------------- |
| `ADMIN_URL`                      | `admin` | The URL prefix Admin is served from.                                                                      |
| `ADMIN_DATA_EXPORT_RETENTION`    | `3600`  | Seconds to keep a generated [data export](data-export.md#configuration).                                  |
| `ADMIN_SESSION_RETENTION`        | `3600`  | Seconds before an idle admin session row is pruned. See [Housekeeping](housekeeping.md#sessions).         |
| `ADMIN_CHANGELOG_RETENTION_DAYS` | unset   | Days to keep change log rows. Unset or `0` keeps them forever. See [Housekeeping](housekeeping.md#changelog). |

_Settings → Admin_ holds the branding colours for the admin UI and the [IP whitelist](user-permissions.md#restricting-admin-by-ip).
