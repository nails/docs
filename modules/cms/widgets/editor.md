# WidgetEditor (v2) editor

The editor is a full-screen overlay on the existing widget field. Pages, areas, and any `form_field_cms_widgets()` consumer keep the same button and the same stored JSON. The v1 editor stays in place. Which one opens is a session choice.

## Which editor opens

`CMS_WIDGET_EDITOR_ENABLE` lists the versions, in the same way as `CDN_MEDIA_MANAGER_ENABLE`. An empty or missing value enables both, so the first time a browser opens an editor it gets v1, and that choice is stored for the session. An app sets `['v1']` or `['v2']` to keep one. An app that is not ready for v2 should set `['v1']`. Existing v1 widgets open in v2 once they are bridged (see [Bridging v1 widgets](v2.md#bridging-v1-widgets)).

The choice is `CMS_WIDGET_EDITOR_DEFAULT` in the session (`Session::setUserData()`). It lapses with the session. A stored choice for a version that is no longer enabled is replaced by the first enabled version.

With both versions enabled, v1 shows **Try the new editor**. That stores v2 and reloads. v2 shows **Use the previous editor**, which stores v1 and reloads. `PUT api/cms/widgeteditor/version` with `{"version": "v1"}` or `{"version": "v2"}` is the call both buttons make.

The page embeds the enabled versions and the active one in `#cms-widget-editor-config`. When v2 is enabled, the field also loads Vue 3 (`Asset::vue3()`) and `admin.widget-editor.min.js` / `admin.widget-editor.min.css`. Vue is a webpack external, so the bundle does not include it. The bundle is not on admin pages that have no widget field.

## What the overlay does

Opening any widget field, or a page area, opens every area on that form as tabs. The outline lists instances and their slots. Dragging a row moves it, including into another slot or, within the same session, another area. Insert, duplicate, remove, and up/down are in the bar. Undo and redo cover those commands. Consecutive edits to one instance are a single undo step.

The organiser lists widgets the active area allows, with favourites first. Hidden and deprecated widgets stay out of the organiser; an instance of one that is already saved can still be edited. A disabled or unknown instance is locked: it can be removed or reordered in its own list, and its stored object is kept.

An HTML-form widget loads its form from `POST …/inspector` and writes the field values back with the same rules as the v1 serialiser. A bridged v1 widget's `dropped` script runs once its form is placed, and its `removed` script runs before the widget is removed (see [Scripts](v2.md#scripts)). **Open in modal** moves that form into a dialog for wide controls, and closing the dialog moves it back with its values. A widget that declares fields gets its form from [Fields](#fields) instead. Nothing is written for an instance the user has not edited, so opening and closing an area leaves its JSON untouched.

Every change is written to the widget field straight away, and a page writes its template data on the same change. A copy is also kept in `localStorage` under `cms-widget-editor:` plus the area slugs, so a crash can be restored. Closing the editor drops that copy.

## Area rules

`form_field_cms_widgets()` accepts `allowed` and `disallowed`. A template area accepts the same through `TemplateArea::setAllowed()` and `setDisallowed()`. Entries are slugs, aliases, or `group:<label>`. The button carries them as `data-cms-rules`. The organiser hides what they forbid, and an insert or a move that would break them is refused. The server still accepts a save that breaks them.

## Canvas

The canvas shows each top-level instance's server-rendered preview in a same-origin iframe. The iframe is sandboxed without `allow-scripts`, so preview markup does not run script. Previews refresh 300ms after a change, and only instances whose slug or data changed are re-requested. A preview that fails shows an error card in place of the widget.

The iframe loads once. A change to an instance replaces only that instance's markup, and an instance with focus is left alone until focus leaves, so the caret is not lost. Links in the canvas do not navigate.

### Inline editing

A widget marks its editable fields in its preview markup (see [Editable fields](#editable-fields)). In the canvas:

- A **text** field is edited in place. Enter is ignored and pasted text is reduced to plain text. Each change is written to the instance straight away, and no preview is requested for it, because the canvas already shows the text.
- A **rich** field opens in the rich text modal when clicked.

### Rich text

The rich text modal is Tiptap. Its schema is paragraphs, bold, italic, links, lists and headings. Save is enabled only after the user changes the text, so a field that is opened and closed is never written back, even if its stored HTML has markup the schema does not keep. Save writes the HTML Tiptap produces, which normalises it (for example `<b>` becomes `<strong>`). Escape or Cancel closes the modal without writing.

While a modal is open, keys belong to it. Backspace in the rich text editor does not remove the selected widget.

### Editable fields

A widget renders the field with `editable()` in `render()`:

```php
return $this->editable('sTitle', 'h2', $data['sTitle'], ['class' => 'title'])
    . $this->editable('body', 'div', $data['body'], ['class' => 'body'], 'rich');
```

Outside the canvas the markup is the same as a plain element. In the canvas the element also carries `data-cms-field`, `data-cms-edit` and `data-cms-path`, which the canvas uses to write the edit back to the right instance. A child in a slot has the path of its slot and index, so a nested widget's fields are edited the same way as a top-level one. `'text'` values are escaped; `'rich'` values are output as HTML, so use them only for content the editor stores.

Inline editing needs `editable()` in `render()`. A widget whose `render()` has no editable fields is still previewed, and it is edited through its inspector.

Scripts do not run in the canvas (see D20). A widget whose preview depends on JavaScript overrides `preview()` with static markup.

### Fields

A widget that declares `getFields()` gets its inspector from the list, and `getEditorForm()` is not used:

```php
public function getFields(): ?array
{
    return [
        ['key' => 'sHeading', 'type' => 'text', 'label' => 'Heading', 'required' => true],
        ['key' => 'bShowLink', 'type' => 'toggle', 'label' => 'Show link'],
        ['key' => 'sLinkText', 'type' => 'text', 'label' => 'Link text', 'visibleWhen' => ['bShowLink' => true]],
        ['key' => 'sTone', 'type' => 'select', 'label' => 'Tone', 'options' => ['light' => 'Light', 'dark' => 'Dark']],
        ['key' => 'aItems', 'type' => 'repeater', 'label' => 'Items', 'min' => 1, 'max' => 3, 'fields' => [
            ['key' => 'sText', 'type' => 'text', 'label' => 'Text', 'required' => true],
        ]],
    ];
}
```

Each field has a `key` (a PHP-style name, letters, digits and underscores), a `type`, and optionally `label` (defaults to the key), `tip`, `required`, `wide`, `visibleWhen` and `default`. The types are:

| Type | Control | Extra keys |
| --- | --- | --- |
| `text` | single line | |
| `textarea` | multi-line text | |
| `richtext` | Tiptap text, edited in the rich text modal | |
| `select` | drop-down | `options`, a map of value to label; the first is the default |
| `toggle` | checkbox | |
| `number` | number input | `min`, `max` |
| `cdn_object` | CDN object picker, stores the object id | `bucket` (the bucket slug) |
| `repeater` | a list of rows, each with its own `fields` | `min`, `max`; rows cannot hold another repeater |

Rules:

- **`visibleWhen`** shows the field only while every named sibling has the given value, compared strictly. A hidden field keeps its value, so it comes back when the condition does. A name that is not a sibling is a schema error.
- **`required`** shows a warning until the field has a value. It never blocks a save. A toggle is never empty.
- **`wide`** shows the control as a one-line summary in the inspector, with **Open in modal** for the full control.
- **`min` and `max`** are a hint. A number outside them is kept; nothing is clamped.
- A repeater's **Add item** stops at `max`, and **Remove** stops at `min`. Rows move with **Up** and **Down**.
- A bad schema (an unknown type, a duplicate or invalid key, a select without options, a `default` that is not an option, a repeater inside a repeater) throws `InvalidArgumentException` when the editor loads its catalogue, so it shows on the first admin request. It is a developer error, not a user one.

Before a widget renders and before its inspector loads, `normalise()` fills any missing defaults, including those of each repeater row. So a field added to a widget later shows its default on existing content, and nothing is saved until the user edits it.

A field can also be edited inline: call `editable()` for it in `render()`. A field inside a repeater row cannot, because there is no path to address the row, so it is edited in the inspector.

## Not in this editor yet

Content the rich text schema does not keep (tables, embeds and unknown blocks) is not shown in the modal. The raw HTML view that warns about this is not built yet. The field is written back only if the user edits it, so such content is lost only when the user saves an edit. Links are not in the field types yet; `cdn_object` covers images and files only.

Automated tests: `yarn test:js` in `module-cms` covers the document tree, the serialiser, the canvas document and its helpers, the area rules, the session commands, and the field rules (visibility, required warnings, repeater rows, coercion). `phpunit` covers `editable()` in preview and render mode, the paths of nested instances, and the field schema. The inspector, the picker's event handling and the modal were checked by hand in a browser against a test area; there is no automated browser test.
