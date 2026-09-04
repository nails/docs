---
description: >-
  This plugin allows for the creation of powerful, searchable dropdowns powered
  by CRUD API controllers.
---

# Searcher

This plugin allows you to generate dynamic, searchable dropdowns which are bound to a [CRUD API](../../api/building.md#crud-controllers) endpoint.

Searchers are made searchable by applying the `js-searcher` class and providing a `data-api` attribute which points to the base of the API endpoint in question (in the example below this would resolve to`http://example.com/api/app/books`)

```php
echo form_field([
    'key'   => 'book_id',
    'label' => 'Book',
    'class' => 'js-searcher',
    'data'  => [
        'api' => 'app/books',
    ],
])
```

When submitted, this searcher's value will be the ID of the selected item.

The following additional data attributes are also supported for further configurations:

| Attribute     | Description                                                                                 | Default              |
| ------------- | ------------------------------------------------------------------------------------------- | -------------------- |
| `multiple`    | Allows multiple items to be selected, the final value will be a comma separated list of IDs | `false`              |
| `clearable`   | Whether the selection can be cleared.                                                       | `true`               |
| `placeholder` | The place holder text.                                                                      | `Search for an item` |
| `min-length`  | The minimum length of the search string.                                                    | `2`                  |
| `get-param`   | The query parameter to use when calling the API.                                            | `search`             |
| `prop-id`     | The property to use for the ID (from the API)                                               | `id`                 |
| `prop-label`  | The property to use for the label (from the API)                                            | `label`              |

