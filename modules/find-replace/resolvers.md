---
description: >-
  Name the record behind a hit, attach facets and edit links, and choose which
  facet values appear as badges.
---

# Resolvers

A resolver turns a matching row into a result a person can recognise: a label, a title, facets for filtering, and links to the screen that edits the record. Without one, the result is the table, column, and primary key.

You do not register the class. Any concrete class under `src/FindReplace/Resolver` that implements `Nails\FindReplace\Interfaces\Resolver` and has no required constructor arguments is picked up. That applies to the app (`App\FindReplace\Resolver\…`) and to every installed module.

`tables()` lists every table this resolver can describe, including a localised table when the text lives there. The first resolver whose list contains the table is the one that runs. Give each table to one resolver.

`resolve()` receives every match from that table and returns hits keyed by primary key. Load the rows once, then fill the hits. A primary key you leave out falls back to the module's `Nails\FindReplace\Resolver\TableColumn` resolver: the title is `table.column #id`, the entity label is empty, and the Resolver facet is `Unresolved`. `find-replace:fields` lists `TableColumn` with no tables of its own, because it is that fallback.

One hit describes the row. Each matching column is still its own result, and it reuses the title, label, facets, and actions you returned for that primary key.

## A class

```php
namespace App\FindReplace\Resolver;

use Nails\FindReplace\Factory\Action;
use Nails\FindReplace\Factory\Hit;
use Nails\FindReplace\Interfaces\Resolver;
use Nails\FindReplace\Interfaces\Resolver\Badges;
use Nails\FindReplace\Interfaces\Resolver\FacetValues;
use Nails\FindReplace\Interfaces\Resolver\Facets;

class Article implements Resolver, Facets, FacetValues, Badges
{
    public function tables(): array
    {
        return ['app_article'];
    }

    public function resolve(string $table, array $matches): array
    {
        $hits = [];

        foreach ($matches as $match) {
            $id = $match->getId();

            if (isset($hits[$id])) {
                continue;
            }

            $hits[$id] = new Hit(
                $table,
                $id,
                entityLabel: 'Article',
                entityId: $id,
                title: 'Loaded title',
                actions: [
                    new Action('Edit', '/admin/articles/edit/' . $id),
                ],
                facets: [
                    'status' => 'PUBLISHED',
                ],
            );
        }

        return $hits;
    }

    public function facets(): array
    {
        return [
            'status' => 'Status',
        ];
    }

    public function values(): array
    {
        return [
            'status' => ['PUBLISHED', 'DRAFT'],
        ];
    }

    public function badges(): array
    {
        return ['status'];
    }
}
```

Set `title`, `entityLabel`, `entityId`, `facets`, and `actions` from the loaded record. `Action` takes a label and a URL. An optional CSS class is added to the button; a colour class such as `btn-default` replaces the default primary style. An optional target is written onto the link.

`Facets`, `FacetValues`, and `Badges` are optional. Implement the ones this resolver needs.

## Facets

`facets()` returns `key => label`. The keys are what `--facet=key:value` and the admin filter dropdowns use. The module always adds a Resolver facet (`resolver`), labelled "Resolver", whose value is the entity label or `Unresolved`.

`values()` lists the accepted values for a closed facet. Omit a key to leave that facet open: any non-empty string is accepted. When several resolvers declare the same key, the last one sets the label. If any of them omits the key from `values()`, the facet stays open. Otherwise the allowed values are the sorted union of the lists. A value outside a closed list is dropped from the hit, so it never appears as a filter. An empty key or an empty value is rejected.

Admin shows a dropdown for each facet that has values on the outstanding results, with a count beside each value. The command line checks `--facet` against the same lists and stops with `key must be one of …` when a closed facet refuses the value.

## Badges

`badges()` returns facet keys to show on an admin result row, in that order. The first resolver to name a key wins the position; a later duplicate is skipped. The `resolver` key is ignored, because the row already shows the entity label. A hit with no value for a key shows nothing for it.

Badges take their colour from the value, matching the admin index pages: `PUBLISHED` is green, `DRAFT` amber, and `DELETED` red, in any letter case. Every other value is grey.

## Filtering on what a resolver named

`--resolved-only` keeps hits whose entity label is set, and drops the `table.column` fallback. Combine it with `--facet` when a replace should stay inside one kind of record:

```bash
nails find-replace:search "old name" --resolved-only --facet=status:PUBLISHED
```

`find-replace:fields` prints the facet keys, whether each one is closed, and the tables each resolver claims.

## Shipped with Nails

These modules ship resolvers under `src/FindReplace/Resolver`. Each hit links to the admin screen that edits the record.

| Module | Entity labels | Tables | Title |
|---|---|---|---|
| `nails/module-auth` | User, User Group | `user`, `user_group` | Name and primary email; group label |
| `nails/module-cdn` | CDN Object, CDN Bucket | `cdn_object`, `cdn_bucket` | Display filename; bucket label. Opens the Media Manager on that object or bucket |
| `nails/module-cms` | CMS Page, CMS Area, CMS Block, CMS Menu, CMS Menu Item | `cms_page`, `cms_area`, `cms_block`, `cms_menu`, `cms_menu_item` | Page title, or label. A menu item is titled `Menu › Item` and opens its menu |
| `nails/module-custom-forms` | Custom Form, Custom Form Notification | `custom_form`, `custom_form_notification` | Form label. A notification is titled by its form and opens the form editor |
| `nails/module-form-builder` | Form Field, Form Field Option | `formbuilder_form_field`, `formbuilder_form_field_option` | `Owner › Field › Option` |
| `nails/module-redirect` | Redirect | `redirect` | `old URL → new URL` |
| `nails/module-survey` | Survey, Survey Response, Survey Answer | `survey_survey`, `survey_response`, `survey_response_answer` | Survey label; `Survey › Respondent` for responses and answers |

CMS pages declare a closed `status` facet (`PUBLISHED`, `DRAFT`, `DELETED`) and show it as a badge. A deleted page has no edit link.

A form has no name or screen of its own, so a form field is described by the record that owns its form. The resolver looks for foreign keys that reference `formbuilder_form.id`. It takes the first row that uses the form and asks the resolver for that row's table for a title and links. A survey's fields read `Survey: Customer feedback › Email address` and link to the survey; a custom form's fields read `Custom Form: Contact us › Email address` and link to the form. If no resolver claims the owning table, the title starts with that table and row (`app_contact_form #3`), and the hit has no links. If nothing references the form, the title starts with `Form #id`. An owner is only found through a foreign key constraint on its form column, so an app table that stores a form ID needs that constraint.

A resolver receives every match in a table at once. The shipped resolvers query their IDs in chunks of 500, because CodeIgniter's query builder fails on a very long `where_in` list. Do the same in an app resolver for a large table.
