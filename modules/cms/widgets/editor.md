# WidgetEditor (v2) editor

The editor is a full-screen overlay on the existing widget field. Pages, areas, and any `form_field_cms_widgets()` consumer keep the same button and the same stored JSON. The v1 editor stays in place. Which one opens is a session choice.

## Which editor opens

`CMS_WIDGET_EDITOR_ENABLE` lists the versions, in the same way as `CDN_MEDIA_MANAGER_ENABLE`. An empty or missing value enables both, so the first time a browser opens an editor it gets v1, and that choice is stored for the session. An app sets `['v1']` or `['v2']` to keep one. Until the legacy bridge ships, an app that has this code and still edits existing content should set `['v1']`.

The choice is `CMS_WIDGET_EDITOR_DEFAULT` in the session (`Session::setUserData()`). It lapses with the session. A stored choice for a version that is no longer enabled is replaced by the first enabled version.

With both versions enabled, v1 shows **Try the new editor**. That stores v2 and reloads. v2 shows **Use the previous editor**, which stores v1 and reloads. `PUT api/cms/widgeteditor/version` with `{"version": "v1"}` or `{"version": "v2"}` is the call both buttons make.

The page embeds the enabled versions and the active one in `#cms-widget-editor-config`. When v2 is enabled, the field also loads Vue 3 (`Asset::vue3()`) and `admin.widget-editor.min.js` / `admin.widget-editor.min.css`. Vue is a webpack external, so the bundle does not include it. The bundle is not on admin pages that have no widget field.

## What the overlay does

Opening any widget field, or a page area, opens every area on that form as tabs. The outline lists instances and their slots. Dragging a row moves it, including into another slot or, within the same session, another area. Insert, duplicate, remove, and up/down are in the bar. Undo and redo cover those commands. Consecutive edits to one instance are a single undo step.

The organiser lists widgets the active area allows, with favourites first. Hidden and deprecated widgets stay out of the organiser; an instance of one that is already saved can still be edited. A disabled or unknown instance is locked: it can be removed or reordered in its own list, and its stored object is kept.

An HTML-form widget loads its form from `POST …/inspector` and writes the field values back with the same rules as the v1 serialiser. A fields-schema widget has no form here yet. Nothing is written for an instance the user has not edited, so opening and closing an area leaves its JSON untouched.

Every change is written to the widget field straight away, and a page writes its template data on the same change. A copy is also kept in `localStorage` under `cms-widget-editor:` plus the area slugs, so a crash can be restored. Closing the editor drops that copy.

## Area rules

`form_field_cms_widgets()` accepts `allowed` and `disallowed`. A template area accepts the same through `TemplateArea::setAllowed()` and `setDisallowed()`. Entries are slugs, aliases, or `group:<label>`. The button carries them as `data-cms-rules`. The organiser hides what they forbid, and an insert or a move that would break them is refused. The server still accepts a save that breaks them.

## Not in this editor yet

The canvas shows each top-level instance's server-rendered preview in a same-origin iframe. The iframe is sandboxed without `allow-scripts`, so preview markup does not run script. Previews refresh 300ms after a change, and only instances whose slug or data changed are re-requested. A preview that fails shows an error card in place of the widget.

Inline editing, rich text, and the fields-schema inspector are not in this editor yet. A widget with a fields-schema inspector can still be added, moved and removed, and its saved data is left as it is. There is no browser test. `yarn test:js` in `module-cms` covers the document tree, the serialiser, the canvas document, the area rules, and the session commands.
