---
description: >-
  Admin's DefaultController is a highly configurable boilerplate for simplifying
  day-to-day management of your application's entities.
---

# Default Controller

`Nails\Admin\Controller\DefaultController` is a configurable base class that builds a CRUD interface for a model. It handles the views, validation and saving, so a typical controller is just a list of constants.

Extend it in `src/Admin/Controller/` and set, at minimum, `CONFIG_MODEL_NAME`:

```php
<?php
// src/Admin/Controller/Book.php

namespace App\Admin\Controller;

use Nails\Admin\Controller\DefaultController;

class Book extends DefaultController
{
    const CONFIG_MODEL_NAME     = 'Book';
    const CONFIG_MODEL_PROVIDER = 'app';
}
```

{% hint style="info" %}
Generate this boilerplate with `nails make:controller:admin Book`. It writes the controller to `src/Admin/Controller/`.
{% endhint %}

## What you get

| Action  | URL                   | Available when                                                                        |
| ------- | --------------------- | ------------------------------------------------------------------------------------- |
| Browse  | `{base}`              | Always. Paginated, searchable if the model is `Searchable`, and filterable.           |
| Create  | `{base}/create`       | `CONFIG_CAN_CREATE`                                                                   |
| Edit    | `{base}/edit/{id}`    | `CONFIG_CAN_EDIT`                                                                     |
| Delete  | `{base}/delete/{id}`  | `CONFIG_CAN_DELETE`. A soft delete if the model supports it.                          |
| Restore | `{base}/restore/{id}` | `CONFIG_CAN_RESTORE`, for soft-deleted items.                                         |
| Destroy | `{base}/destroy/{id}` | `CONFIG_CAN_DESTROY`. Permanently removes the item.                                   |
| Copy    | `{base}/copy/{id}`    | `CONFIG_CAN_COPY`, and the model uses the `Copyable` trait.                           |
| Sort    | `{base}/sort`         | `CONFIG_CAN_SORT`, and the model uses the `Sortable` trait.                           |
| View    | The item's own URL    | `CONFIG_CAN_VIEW`, and the resource has a `url` property or a `getUrl()` method.      |

Each action also checks its `CONFIG_PERMISSION_*` constant (see [Permissions](#permissions)), and its button is hidden when the user can't use it.

The create and edit forms are built from the model's field definitions (`describeFields()`). Each fieldset becomes a [tab](../helper/tabs.md), and the [save bar](../forms.md#floating-save-bar), [notes](../javascript/notes.md) and [unsaved-changes](../javascript/unsaved-changes.md) warning come built in. Changes are recorded in the [change log](#changelog).

The controller also adapts to the traits the model uses. `Sortable` adds a "Defined Order" sort option. `Localised` adds a locale column and per-locale buttons. `Publishable` adds a publish-state filter. `Nestable` keeps its ordering and breadcrumb columns out of the forms.

If you want different markup, put a view with the same name (`index.php`, `edit.php` or `order.php`) in your controller's [view directory](./#views). Admin uses it instead of the built-in one.

## Configuring

The DefaultController has many configuration constants. All described with their defaults shown below.

{% hint style="info" %}
These constants are used by the controller's `getConfig` method which populates the `$aConfig` class property. Some configurations accepts closures which cannot be set via a constant); to set these you can use [Late Configuration](default-controller.md#late-configuration).
{% endhint %}

### Model/Provider

The following constants define to which model the controller is bound:

```php
const CONFIG_MODEL_NAME     = '';
const CONFIG_MODEL_PROVIDER = 'app';
```

### Permissions

Each action can require a [permission](../user-permissions.md). Set the constant to a permission class name. `null` means any admin user can do it.

```php
const CONFIG_PERMISSION_BROWSE  = null;
const CONFIG_PERMISSION_CREATE  = null;
const CONFIG_PERMISSION_EDIT    = null;
const CONFIG_PERMISSION_VIEW    = null;
const CONFIG_PERMISSION_DELETE  = null;
const CONFIG_PERMISSION_DESTROY = null;
const CONFIG_PERMISSION_RESTORE = null;
const CONFIG_PERMISSION_COPY    = null;
const CONFIG_PERMISSION_SORT    = null;
```

For example:

```php
use App\Admin\Permission;

const CONFIG_PERMISSION_BROWSE = Permission\Book\Browse::class;
const CONFIG_PERMISSION_EDIT   = Permission\Book\Edit::class;
```

If the user doesn't have `CONFIG_PERMISSION_BROWSE`, the controller adds nothing to the sidebar.

### Title

The singular and plural name of the item being managed; defaults to the model name.

```php
const CONFIG_TITLE_SINGLE = '';
const CONFIG_TITLE_PLURAL = '';
```

`CONFIG_TITLE_PLURAL` is also the label for the index crumb in the [breadcrumb trail](breadcrumbs.md) (`Admin › {plural}`). Override `getIndexBreadcrumbLabel()` if you need a different label.

### Sidebar

Where to display this controller in the admin sidebar (defaults to `CONFIG_TITLE_PLURAL`) and which icon to use.

```php
const CONFIG_SIDEBAR_GROUP = '';
const CONFIG_SIDEBAR_ICON  = '';
```

The format for the sidebar link. `%s` is replaced with `CONFIG_TITLE_PLURAL`.

```php
const CONFIG_SIDEBAR_FORMAT = 'Manage %s';
```

Extra keywords that match this group when a user filters the sidebar.

```php
const CONFIG_SIDEBAR_SEARCH_TERMS = [];
```

### Base URL

The base URL for this controller. Leave it empty to use `static::url()`, which is almost always what you want.

```php
const CONFIG_BASE_URL = '';
```

### Controller Behaviour

Specify whether the controller supports item creation.

```php
const CONFIG_CAN_CREATE = true;
```

Specify whether the controller supports item editing.

```php
const CONFIG_CAN_EDIT = true;
```

Specify whether the controller supports linking to the item.

```php
const CONFIG_CAN_VIEW = true;
```

Specify whether the controller supports item deletion

```php
const CONFIG_CAN_DELETE = true;
```

Specify whether the controller supports permanently destroying items

```php
const CONFIG_CAN_DESTROY = true;
```

Specify whether the controller supports item restoration

```php
const CONFIG_CAN_RESTORE = true;
```

Specify whether the controller supports copying items (the model must use `Copyable`)

```php
const CONFIG_CAN_COPY = true;
```

Specify whether the controller supports manual ordering (the model must use `Sortable`)

```php
const CONFIG_CAN_SORT = true;
```

### Index View

The fields to show on the index view. Column name on the left, property name on the right. Dot notation can be used to reach deeper properties, for example items made available via `CONFIG_INDEX_DATA`.

```php
const CONFIG_INDEX_FIELDS = [
    'Label'       => 'label',
    'Modified'    => 'modified',
    'Modified By' => 'modified_by',
];
```

{% hint style="info" %}
If you need to return dynamic data then a closure can be set as the right hand side via [Late Configuration](default-controller.md#late-configuration).
{% endhint %}

Any additional header buttons to add to the index page.

```php
const CONFIG_INDEX_HEADER_BUTTONS = [];
```

Any additional buttons to add to each row on the page.

```php
const CONFIG_INDEX_ROW_BUTTONS = [];
```

Additional data to pass into the `getAll` call on the index view.

```php
const CONFIG_INDEX_DATA = [];
```

The fields on the index view which should be rendered as user cells.

```php
const CONFIG_INDEX_USER_FIELDS = [
    'created_by',
    'modified_by',
    'user_id',
];
```

The fields on the index view which should be rendered as boolean cells.

```php
const CONFIG_INDEX_BOOL_FIELDS = [
    'is_active',
    'is_published',
    'is_deleted',
];
```

The fields on the index view which should be run through `number_format`.

```php
const CONFIG_INDEX_NUMERIC_FIELDS = [
    'id',
];
```

The fields on the index view which should be centred.

```php
const CONFIG_INDEX_CENTERED_FIELDS = [
    'id',
];
```

The ID to give the index page

```php
const CONFIG_INDEX_PAGE_ID = '';
```

Show a [notes](../javascript/notes.md) button on each row, optionally with a count of existing notes

```php
const CONFIG_INDEX_NOTES_ENABLE = false;
const CONFIG_INDEX_NOTES_COUNT  = false;
```

Extra markup to render above and below the index table

```php
const CONFIG_INDEX_HTML_HEADER = '';
const CONFIG_INDEX_HTML_FOOTER = '';
```

The sorting options to give the user on the index view:

```php
const CONFIG_SORT_OPTIONS = [
    'Label'    => 'label',
    'Created'  => 'created',
    'Modified' => 'modified',
];
```

The default sorting order:

```php
const CONFIG_SORT_DIRECTION = 'asc';
```

### Create and Edit views

Fields which should be marked as readonly when creating an item

```php
const CONFIG_CREATE_READONLY_FIELDS = [];
```

The fields to ignore on the create view

```php
const CONFIG_CREATE_IGNORE_FIELDS = [
    'id',
    'slug',
    'token',
    'is_deleted',
    'created',
    'created_by',
    'modified',
    'modified_by',
];
```

The fields to ignore on the edit view

```php
const CONFIG_EDIT_IGNORE_FIELDS = self::CONFIG_CREATE_IGNORE_FIELDS;
```

Fields which should be marked as readonly when editing an item

```php
const CONFIG_EDIT_READONLY_FIELDS = [];
```

Additional data to pass into the getAll call on the edit view

```php
const CONFIG_EDIT_DATA = [];
```

Any additional header buttons to add to the edit page.

```php
const CONFIG_EDIT_HEADER_BUTTONS = [];
```

Specify a specific order for fieldsets. Each fieldset becomes a [tab](../helper/tabs.md) on the edit screen. Fields that are not already inside a `<fieldset>` are wrapped in one so they get card chrome. The sticky [save bar](../forms.md#floating-save-bar) is rendered automatically.

```php
const CONFIG_EDIT_FIELDSET_ORDER = [];
```

The ID to give the edit page

```php
const CONFIG_EDIT_PAGE_ID = '';
```

Extra markup to render above and below the edit form

```php
const CONFIG_EDIT_HTML_HEADER = '';
const CONFIG_EDIT_HTML_FOOTER = '';
```

Warn the user, instead of overwriting, if someone else saved the item after they opened it

```php
const EDIT_MODIFIED_CHECK_ENABLED = true;
```

### Delete and destroy

Additional data to pass into the `getAll` call when loading the item to delete or destroy

```php
const CONFIG_DELETE_DATA  = [];
const CONFIG_DESTROY_DATA = [];
```

### Sorting

{% hint style="info" %}
The following apply to model's which implement the `Sortable` trait.
{% endhint %}

Additional data to pass into the getAll call on the sort view

```php
const CONFIG_SORT_DATA = [];
```

Which column to use for the label when sorting

```php
const CONFIG_SORT_LABEL = 'label';
```

Any additional columns to add to the sort view

```php
const CONFIG_SORT_COLUMNS = [];
```

### Notes

Enable or disable [notes](../javascript/notes.md) in the edit screen's save bar

```php
const CONFIG_EDIT_NOTES_ENABLE = true;
```

For the index equivalent, see `CONFIG_INDEX_NOTES_ENABLE` under [Index View](#index-view).

### Unsaved changes

Whether to stamp `data-unsaved-changes` on the edit form and save bar. When true (the default), the [Unsaved Changes](../javascript/unsaved-changes.md) plugin shows a chip beside Save if the form is dirty.

```php
const EDIT_UNSAVED_CHANGES_ENABLED = true;
```

### Changelog

Whether to record updates in the admin change log

```php
const CHANGELOG_ENABLED = true;
```

An array of fields to ignore when processing change log updates

```php
const CHANGELOG_FIELDS_IGNORE = [
    'id',
    'is_deleted',
    'created',
    'created_by',
    'modified',
    'modified_by',
];
```

Entries are browsable under _Logs → Change Log_. To prune old entries, see [Housekeeping](../housekeeping.md#changelog).

### User feedback messages

Message displayed to user when an item is successfully created

```php
const CREATE_SUCCESS_MESSAGE = 'Item created successfully. %s';
```

Message displayed to user when an item fails to be created

```php
const CREATE_ERROR_MESSAGE = 'Failed to create item.';
```

Message displayed to user when an item is successfully updated

```php
const EDIT_SUCCESS_MESSAGE = 'Item updated successfully. %s';
```

Message displayed to user when an item fails to be updated

```php
const EDIT_ERROR_MESSAGE = 'Failed to update item.';
```

Message displayed to user when an item is successfully deleted

```php
const DELETE_SUCCESS_MESSAGE = 'Item deleted successfully.';
```

Message displayed to user when an item fails to be deleted

```php
const DELETE_ERROR_MESSAGE = 'Failed to delete item.';
```

Message displayed to user when an item is successfully destroyed

```php
const DESTROY_SUCCESS_MESSAGE = 'Item destroyed successfully.';
```

Message displayed to user when an item fails to be destroyed

```php
const DESTROY_ERROR_MESSAGE = 'Failed to destroy item.';
```

Message displayed to user when an item is successfully restored

```php
const RESTORE_SUCCESS_MESSAGE = 'Item restored successfully.';
```

Message displayed to user when an item fails to be restored

```php
const RESTORE_ERROR_MESSAGE = 'Failed to restore item.';
```

Message displayed to user when an items are ordered successfully

```php
const ORDER_SUCCESS_MESSAGE = 'Items ordered successfully.';
```

Message displayed to user when an item fails to be ordered

```php
const ORDER_ERROR_MESSAGE = 'Failed to order items.';
```

Message displayed to user when an item is successfully copied

```php
const COPY_SUCCESS_MESSAGE = 'Item copied successfully.';
```

## Hooks

To add behaviour around saving without replacing whole actions, override these protected methods. They do nothing by default, except `beforeEdit()`, which runs the "modified since you opened it" check. If you override `beforeEdit()`, call `parent::beforeEdit($oItem)`.

| Hook                                                     | Called                                               |
| -------------------------------------------------------- | ---------------------------------------------------- |
| `beforeCreateAndEdit($sMode, ?Resource $oItem)`          | Before a create or an edit, after validation passes. |
| `beforeCreate()` / `beforeEdit(?Resource $oItem)`        | Before a create or an edit.                          |
| `afterCreateAndEdit($sMode, Resource $oNew, ?Resource $oOld)` | After a create or an edit is saved.             |
| `afterCreate(Resource $oNew)` / `afterEdit(Resource $oNew, ?Resource $oOld)` | After a create or an edit is saved. |
| `beforeCopy(Resource $oItem)` / `afterCopy(Resource $oNew, Resource $oOld)` | Around a copy.                       |
| `beforeDelete(Resource $oItem)` / `afterDelete(Resource $oItem)`   | Around a delete.                           |
| `beforeDestroy(Resource $oItem)` / `afterDestroy(Resource $oItem)` | Around a destroy.                          |

`$sMode` is `static::EDIT_MODE_CREATE` or `static::EDIT_MODE_EDIT`. Throwing an exception from a `before*` hook stops the save, and the message is shown to the user.

Other useful extension points:

* `runFormValidation(string $sMode, array $aOverrides = [])` adds or changes validation rules.
* `getPostObject(): array` changes the data passed to the model's `create()` or `update()`.
* `loadEditViewData(?Resource $oItem)` adds data for the edit view.
* `indexDropdownFilters()` and `indexCheckboxFilters()` add [index filters](#filters).
* `addIndexHeaderButton()`, `addEditHeaderButton()` and `addIndexRowButton()` add buttons.

## Breadcrumbs

`DefaultController` builds an opt-in breadcrumb trail with the index as the tip (`Admin › {plural}`). Create, edit, and sort push an extra crumb. Nested actions can call `setBreadcrumbTrail()` and then `addBreadcrumb()`. See [Breadcrumbs](breadcrumbs.md).

## Late Configuration

Late configuration is the process of setting or changing a configuration value after the controller has been initiated. Typically this is used to apply closures to configuration fields which accept them, but might be used to change a config based on some other context.

For example, if you wish to show a column which contains dynamic information about the item you'd add a closure to the `CONFIG_INDEX_FIELDS` config field. The following example shows a count of the total number of reviews plus the average review score.

```php
<?php
// Expand the book reviews
const CONFIG_INDEX_DATA = ['expand' => ['reviews']];

// Define the initial column layout, with a placeholder for `Reviews`
const CONFIG_INDEX_FIELDS = [
    'Label'      => 'label',
    'Reviews'    => '',
    'Created'    => 'created',
    'Created By' => 'created_by',
];

// Overwrite the `Reviews` column
public function __construct()
{
    parent::__construct();
    $this->aConfig['INDEX_FIELDS']['Reviews'] = function($oBook) {
    
        $iTotal = 0;
        foreach ($oBook->reviews->data as $oReview) {
            $iTotal += $oReview->score;
        }
    
        return sprintf(
            '%s total reviews; Average: %s/5',
            $oBook->reviews->count,
            number_format($iTotal / $oBook->reviews->count, 2)
        );
    };
}
```

## Index View

The index view is the overview of all the items managed by the controller. It is searchable, paginated and filterable.

### Filters

The DefaultController offers two types of filters which can be applied to the index data set: Dropdown and checkbox. Both are almost identical in function, however the checkbox filter allows multiple values to be selected.

![Filter area showing a dropdown filter](<../../../.gitbook/assets/Screenshot 2020-05-26 12.37.01.png>)

![Filter area showing a checkbox filter](<../../../.gitbook/assets/Screenshot 2020-05-26 12.36.37.png>)

Both flavours are defined using the `IndexFilter` factory, depending on whether you want the filter be a dropdown or a checkbox then place the `IndexFilter` definition in the controller's `indexDropdownFilters` or `indexCheckboxFilters` method respectively:

```php
protected function indexDropdownFilters(): array
{
    return [
        Factory::factory('IndexFilter', 'nails/module-admin')
            ->setLabel('Status')
            ->setColumn('status')
            ->addOptions([
            
                Factory::factory('IndexFilterOption', 'nails/module-admin')
                    ->setLabel('Published')
                    ->setValue('PUBLISHED')
                    ->setIsSelected(true),
                    
                Factory::factory('IndexFilterOption', 'nails/module-admin')
                    ->setLabel('Draft')
                    ->setValue('DRAFT')
            ]),
    ];
}
```

Multiple distinct filters will be joined together using an `AND` operator. When using checkbox filters, the multiple selected options will be joined using an `OR` operator.

For example, if you had a dropdown filter `Published[Yes|No]` and a checkbox filter `Publisher[ACME Books Ltd|Penguin Publishings]` then the resulting query when both checkbox items were checked and the dropdown was set to `Yes` would be:

```
SELECT *
FROM books
WHERE
(is_published = 'Yes')
AND (publisher = 'ACME Books Ltd' OR publisher = 'Penguin Publishings')
```

{% hint style="info" %}
For advanced filtering, you may set the `IndexFilterOption` item to behave as an SQL query by using `setIsQuery(true)`. This will tell the Filter that the value you supply using `setValue()` should be executed as an SQL query.
{% endhint %}

### Index Row Buttons

Each item on the index has actions which can be applied to it, typically these are `View`, `Edit` and `Delete`; however it is easy to add additional buttons via the `CONFIG_INDEX_ROW_BUTTONS` configuration:

```php
const CONFIG_INDEX_ROW_BUTTONS = [
    [
        // The button's value/label
        'label' => 'The button label',
        
        // The button's URL, relative to the controller. Item
        // properties can be substituted in using Mustache syntax.
        'url' => 'edit/{{id}}',
        
        // Additional classes to apply to the button
        'class' => 'btn-primary',
        
        // Permission class required in order to render
        // the button
        'permission' => Permission\Book\Edit::class,
        
        // Any additional attributes to apply to
        // the button
        'attr' => '',
        
        // Whether the button is enabled or not
        // Only available when the button is created
        // via Late Configuration
        'enabled' => function($oItem) {
            return true;
        },
    ]
];
```

It is also possible to group buttons into a button group by passing in multiple URLs via the button's `url` property. The following example uses [Late Configuration](default-controller.md#late-configuration) to add the button, which will only render for published books:

```php
public function __construct()
{
    parent::__construct();

    $this->aConfig['INDEX_ROW_BUTTONS'][] = [
        'label'   => 'Download',
        'class'   => 'btn-primary',
        'enabled' => function ($oItem) {
            return $oItem->status === 'PUBLISHED';
        },
        'url'     => [
            'As PDF'   => 'download/{{id}}/pdf',
            'As HTML'  => 'download/{{id}}/html',
            'As eBook' => 'download/{{id}}/ebook',
        ],
    ];
}
```

