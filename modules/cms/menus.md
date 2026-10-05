---
description: Nestable menus of CMS pages and custom URLs.
---

# Menus

A menu is a named, nestable list of links. Editors manage them under CMS → Menus. Each menu has a label, a slug, and a description. Each item has a label and a target: a CMS page, or a custom URL. Items can be nested and reordered in the editor.

`cmsMenu()` returns the menu resource (or `null`). It does not render HTML. The optional second argument is passed to the menu model.

```php
$oMenu = cmsMenu('main');

if ($oMenu) {
    foreach ($oMenu->items()->data as $oItem) {
        ?>
        <a href="<?=$oItem->getUrl()?>"><?=htmlspecialchars($oItem->label)?></a>
        <?php
        // Nested items: $oItem->children()->data
    }
}
```

`getUrl()` uses the linked page's published URL when the item points at a page, and the custom URL otherwise.

Pass the id or the slug:

```php
$oMenu = cmsMenu(12);
$oMenu = cmsMenu('main', ['expand' => ['items']]);
```
