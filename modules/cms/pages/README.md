---
description: Nestable pages built from a template and widget areas.
---

# Pages

CMS pages are nestable pages edited in admin and rendered by a [template](templates.md). Editors work under CMS → Pages. A page keeps a draft and a published copy of its title, slug, parent, template, widget data, template options, and SEO fields (title, description, keywords, image).

Publishing copies the draft onto the published columns and registers the published slug as a route:

```php
$route['about/team'] = 'cms/render/page/42';
```

A previous slug redirects to the current one with a 301, via `cms/render/legacy_slug/{id}`. Unpublished and deleted pages are not routed. Delete is soft.

The render controller sets:

| Variable | Contents |
| --- | --- |
| `$oCmsPage` | The page resource |
| `$oCmsPageData` | The published data, or the draft when previewing |
| `$page_data` | The same data object, kept for older templates |

Preview (`cms/render/preview/{id}`) is limited to users who can edit pages and renders the draft. A published page with unpublished edits shows a warning to those users.

`cmsPage()` returns the page resource, by id or slug, without rendering it.

```php
$oPage = cmsPage('about/team');
echo $oPage->published->title;
```

`$oPage->render()` returns the template HTML. Pass `false` to render the draft.

Publishing and unpublishing fire `PAGE:PUBLISHED` and `PAGE:UNPUBLISHED` (`Nails\Cms\Events`), with the page id.

To serve a page at `/`, see [Homepage](homepage.md).
