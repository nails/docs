# WidgetEditor (v2) editor

The editor is a full-screen overlay on the existing widget field. Pages, areas, and any `form_field_cms_widgets()` consumer keep the same button and the same stored JSON. The v1 editor stays in place. Which one opens is a session choice.

## Which editor opens

`CMS_WIDGET_EDITOR_ENABLE` lists the versions, in the same way as `CDN_MEDIA_MANAGER_ENABLE`. An empty or missing value enables both, so the first time a browser opens an editor it gets v1, and that choice is stored for the session. An app sets `['v1']` or `['v2']` to keep one. An app that is not ready for v2 should set `['v1']`. Existing v1 widgets open in v2 once they are bridged (see [Bridging v1 widgets](v2.md#bridging-v1-widgets)).

The choice is `CMS_WIDGET_EDITOR_DEFAULT` in the session (`Session::setUserData()`). It lapses with the session. A stored choice for a version that is no longer enabled is replaced by the first enabled version.

With both versions enabled, v1 shows **Try the new editor**. That stores v2 and reloads. v2 shows **Use the previous editor**, which stores v1 and reloads. `PUT api/cms/widgeteditor/version` with `{"version": "v1"}` or `{"version": "v2"}` is the call both buttons make.

The page embeds the enabled versions and the active one in `#cms-widget-editor-config`. When v2 is enabled, the field also loads Vue 3 (`Asset::vue3()`), `admin.widget-editor.min.js` / `admin.widget-editor.min.css`, and the editor assets of every installed [field type](#field-types). Vue is a webpack external, so the bundle does not include it. The bundle is not on admin pages that have no widget field.

## What the overlay does

Opening any widget field, or a page area, opens every area on that form. The bar shows a widgets icon and the active area's field label; with more than one area, the label is a menu that switches between them. Undo, redo, **Cancel** and **Done** are on the right. The same icon is on the button that opens the editor. **Done** keeps the edits. **Cancel**, and Escape when nothing else is open, closes without them. If anything has changed since the editor opened, it asks first; discarding writes each area back to how it was.

The outline on the left lists instances with a number, label and a line of their text. Its collapse control sits before the label, and stays put when the column closes. The inspector on the right is titled the same way, with its control on the right edge, and it collapses the same way. A container's children sit under it, indented, with a vertical line tying them to the parent. A nested row's highlight starts on that line. Each slot has its own collapse control. Hovering the boundary between two rows, including the top of the list and the end of a slot, reveals a centred **+** that inserts there; it takes no space, so the rows do not move.

Dragging a row moves it, and a container's slots fade with the row. The marker is only as wide as the list it lands in, so a drop inside a container reads as inside it. While the drag is over a place, the marker is a down arrow rather than a **+**. A place the widget cannot land in, such as inside itself, uses the same line with a ban, and the drop is refused. The upper half of a row is the place before it. The lower half of a leaf, or of a container whose slots are collapsed, is the place after it. The lower half of an open container is the start of its first open slot, because nothing can be inserted between a container and its children. Within the same session a row can move to another area. Duplicate and remove are on each row. The canvas does not repeat them.

A change the editor refuses is reported under the bar. The notice can be dismissed, and it goes away on its own after a few seconds. Removing a widget asks for confirmation in an admin modal. The modal's actions hold Cancel and Remove. When the widget contains others, that warning is an `alert-warning`. Delete or Backspace asks for the selected widget. Undo and redo cover insert, duplicate, remove and move. Consecutive edits to one instance are a single undo step.

An empty page, and an empty outline, each offer **Add a widget** to place the first one. A **+** between rows in the outline and a **+** between widgets on the canvas open the insert drawer. The drawer lists widgets the active area allows, with favourites first; hovering one shows its screenshot and description. A widget chosen from a **+** goes at that position. Enter in the search inserts the first match. Hidden and deprecated widgets stay out of the organiser; an instance of one that is already saved can still be edited. A disabled or unknown instance is locked: it can be removed or reordered in its own list, and its stored object is kept. Its outline row carries a warning icon, and the canvas and inspector each show a warning naming the slug.

The inspector on the right is titled Inspector. The selected widget's label, group and description sit above its settings. A widget with no settings says so and points to editing its text on the page.

An HTML-form widget loads its form from `POST …/inspector` and writes the field values back with the same rules as the v1 serialiser. A bridged v1 widget's `dropped` script runs once its form is placed, and its `removed` script runs before the widget is removed (see [Scripts](v2.md#scripts)). **Open in modal** moves that form into a dialog for wide controls, and closing the dialog moves it back with its values. A widget that declares fields gets its form from [Fields](#fields) instead. Nothing is written for an instance the user has not edited, so opening and closing an area leaves its JSON untouched.

Every change is written to the widget field straight away, and a page writes its template data on the same change. A copy is also kept in `localStorage` under `cms-widget-editor:` plus the area slugs, so a crash can be restored. **Done** drops that copy. **Cancel** drops it only after discarding changes, so an unrestored copy from an earlier session is left alone when nothing was edited.

## Area rules

`form_field_cms_widgets()` accepts `allowed` and `disallowed`. A template area accepts the same through `TemplateArea::setAllowed()` and `setDisallowed()`. Entries are slugs, aliases, or `group:<label>`. The button carries them as `data-cms-rules`. The organiser hides what they forbid, and an insert or a move that would break them is refused. The server still accepts a save that breaks them.

## Canvas

The canvas shows each top-level instance's server-rendered preview in a same-origin iframe. The iframe is sandboxed without `allow-scripts`, so preview markup does not run script. Previews refresh 300ms after a change, and only instances whose slug or data changed are re-requested. While a preview is rendering again, the one already on the canvas stays and is dimmed under **Generating preview**. The first preview, before there is anything to keep, says **Loading preview**. A preview that fails shows an error card in place of the widget. A locked instance shows a warning there instead of a preview.

The canvas is drawn as a page with a maximum width. The iframe is sized to its content and the stage around it scrolls. The iframe loads once. A change to an instance replaces only that instance's markup, and the field being edited inline is left alone until editing ends. Links in the canvas do not navigate.

A frame with no script does not run event listeners for real pointer or keyboard input, so the canvas does not listen inside it. An overlay in the admin page covers the iframe, finds the widget or field under the pointer with `elementFromPoint()`, and draws the hover and selection lines and the **+** inserters between widgets. Each widget is padded inside its own box, so the text sits in from the paper and the widget's line stays on the left edge of the page. The inserter is inset from that edge and fades in as one control. Hovering a row in the outline shows that line, and hovering the widget on the canvas marks the same row. Clicking a widget selects it in the outline and the inspector. Clicking a row scrolls that widget into view. A heading marked `data-cms-action="toggle"` opens and closes the panel it belongs to, and that open or closed state is the panel's **Starts open** value.

### Inline editing

A widget marks its editable fields in its preview markup (see [Editable fields](#editable-fields)). In the canvas:

- A **text** field is edited in place: clicking it puts an editable copy, styled like the field, over it with the caret where the click was. Enter or Escape ends editing, as does pressing outside the field. The click that ends editing does not start it again. The text is one line, and pasted text is reduced to plain text. Each change is written to the instance straight away, and no preview is requested for it, because the canvas already shows the text.
- A **rich** field is edited in place too. Clicking it puts a Tiptap editor over the field. The text does not move: the menu is a bubble that appears when text is selected, with undo and redo, a paragraph or a heading (levels 1 to 6), lists, a blockquote, a code block, bold, italic and a link. A field marked `rich-basic` uses the formatting-only bubble: undo and redo, bold, italic and a link. The editor uses the preview's type, including the site's fonts and the same font smoothing, and stays inside the widget. Escape or a press outside the field ends editing, and that click does not start it again. An empty field shows "Write something…" on the canvas so it can be clicked; the public page stays empty until the field has content. A blank document is stored as an empty string. The HTML is written only when it changes, so opening and closing a field does not normalise stored markup the schema does not keep.

### Rich text

The canvas editor and the fallback modal share one Tiptap schema: paragraphs, bold, italic, links, lists, headings (levels 1 to 6), blockquotes and code blocks. The menubar is the one from Tiptap's simple editor template, limited to that schema. On the canvas it floats over the selection. In the modal it stays attached above the text, so the dialog does not depend on a selection. A rich text field declared with `basic()` — the same choice as CKEditor's wysiwyg basic — uses a smaller schema and menu: paragraphs, bold, italic and links. On the canvas that field is edited as `rich-basic`. A widget that does not mark the field for the canvas is edited from the inspector, which opens the same editor in a modal, including the basic menu when the field asks for it. Save, in the modal's actions, is enabled only after the user changes the text. Save writes the HTML Tiptap produces, which normalises it (for example `<b>` becomes `<strong>`). Escape or Cancel closes the modal without writing.

With no site stylesheet, the canvas sets a slightly looser default line height (1.45 for text, 1.3 for headings) and plain styles for quotes and code. Any rule from a site stylesheet wins over those defaults. The editor copies the preview's computed type, so the two stay matched.

While a modal is open, keys belong to it. Backspace in the rich text editor does not remove the selected widget.

### Editable fields

A widget renders the field with `editable()` in `render()`:

```php
return $this->editable('sTitle', 'h2', $data['sTitle'], ['class' => 'title'])
    . $this->editable('body', 'div', $data['body'], ['class' => 'body']);
```

When `getFields()` declares the key, the field type decides how it is edited: a `Text` is `'text'`, a `Richtext` is `'rich'`, and a `Richtext` declared with `basic()` is `'rich-basic'`, the formatting-only editor. Passing the last argument is optional, and it throws if it disagrees with the type. A type that is edited only in the inspector (a `Textarea`, a `Select`, and so on) throws too, so a multi-line field cannot be edited as one line by mistake. A key the widget does not declare, as in an HTML-form widget, is edited as the last argument says, `'text'` by default.

Outside the canvas the markup is the same as a plain element. In the canvas the element also carries `data-cms-field`, `data-cms-edit` and `data-cms-path`, which the canvas uses to write the edit back to the right instance. A child in a slot has the path of its slot and index, so a nested widget's fields are edited the same way as a top-level one. A repeater row is named `list/0/child`: the repeater's key, the row index and the child's key. The child's type decides how it is edited, and the value is written to that row. The repeater stays in the inspector. `'text'` values are escaped; `'rich'` and `'rich-basic'` values are output as HTML, so use them only for content the editor stores.

Inline editing needs `editable()` in `render()`. A widget whose `render()` has no editable fields is still previewed, and it is edited through its inspector.

Scripts do not run in the canvas (see D20). A widget whose preview depends on JavaScript overrides `preview()` with static markup.

### Fields

A widget that declares `getFields()` gets its inspector from the list, and `getEditorForm()` is not used:

```php
use Nails\Cdn\Cms\Fields\ObjectPicker;
use Nails\Cms\Cms\Fields;

public function getFields(): ?array
{
    return [
        Fields\Text::make('sHeading')->label('Heading')->required(),
        Fields\Toggle::make('bShowLink')->label('Show link'),
        Fields\Text::make('sLinkText')->label('Link text')->visibleWhen(['bShowLink' => true]),
        Fields\Select::make('sTone')->label('Tone')->options(['light' => 'Light', 'dark' => 'Dark']),
        ObjectPicker::make('iImageId')->label('Image'),
        Fields\Repeater::make('aItems')->label('Items')->min(1)->max(3)->fields([
            Fields\Text::make('sText')->label('Text')->required(),
        ]),
    ];
}
```

Each field is an instance of a field type, made with `make()` and the data key (a PHP-style name: letters, digits and underscores). Every type takes `label()` (defaults to the key), `tip()`, `subtitle()`, `info()`, `required()`, `wide()`, `visibleWhen()` and `default()`. The types Nails ships are:

| Class | Type name | Control | Own settings |
| --- | --- | --- | --- |
| `Nails\Cms\Cms\Fields\Text` | `text` | single line; can be edited on the canvas | `maxLength()` |
| `Nails\Cms\Cms\Fields\Textarea` | `textarea` | multi-line text | `maxLength()` |
| `Nails\Cms\Cms\Fields\Richtext` | `richtext` | Tiptap text, edited on the canvas; the modal is the fallback | `basic()`: formatting only (undo, bold, italic, link) instead of the simple editor menu (headings 1 to 6, lists, blockquote, code block, bold, italic, link) |
| `Nails\Cms\Cms\Fields\Select` | `select` | drop-down | `options()`, a map of value to label, shown in the order given; the first is the default |
| `Nails\Cms\Cms\Fields\Toggle` | `toggle` | the admin switch | |
| `Nails\Cms\Cms\Fields\Number` | `number` | number input | `min()`, `max()` |
| `Nails\Cms\Cms\Fields\Repeater` | `repeater` | a list of rows, each with its own fields | `fields()`, `min()`, `max()`; rows cannot hold another repeater |
| `Nails\Cms\Cms\Fields\Table` | `table` | a grid of text cells, stored as a list of rows of strings | `size($rows, $columns)`: the empty grid a new instance starts with (3 × 3 by default) |
| `Nails\Cdn\Cms\Fields\ObjectPicker` | `cdn_object` | CDN object picker, stores the object id (module-cdn) | `bucket()` (the bucket slug) |

Rules:

- **`tip`** is the question-mark hint, using the same hint as the rest of admin. It opens to the right, so the text stays inside the inspector.
- **`subtitle`** is quiet text under the label.
- **`info`** is text under the control. The second argument is a class, for example `alert alert-info`, the same as a form field's info class.
- **`visibleWhen`** shows the field only while every named sibling has the given value, compared strictly. A hidden field keeps its value, so it comes back when the condition does. A name that is not a sibling is a schema error.
- **`required`** shows an asterisk. It never blocks a save. A toggle is never empty.
- **`wide`** shows a short summary in the inspector, clamped to three lines, with **Edit** for the control in a modal. The modal holds the control only; its title is the label. Changes are written straight away; **Done** closes it. Rich text is already a summary with the same **Edit** button, so `wide` does not change it.
- **`min` and `max`** are a hint. A number outside them is kept; nothing is clamped.
- A repeater's **Add item** stops at `max`, and **Remove** stops at `min`. With no minimum the last row can be removed. An empty list is a dashed **Add an item** button. A row is titled from its first field's text, or that field's selected option, and that title collapses the row. Rows move with the up and down icons. A select uses Select2, which admin already loads, without the search box. Select2 reports the choice through jQuery's change event, and the control listens for that, so the stored value matches the drop-down.
- A table has **Add row** and **Add column**. The × above a column removes it, and the bin at the end of a row removes that row. Removing the last column empties the table, which shows a dashed **Add a row** button. Rows are filled to the widest row when the widget renders. The summary is the size, for example "3 rows × 2 columns"; a table whose cells are all blank counts as empty for `required`.
- A bad schema (something in the list that is not a field, an invalid or duplicate key, a select without options, a `default` that is not an option, a repeater without fields or inside a repeater) throws `InvalidArgumentException` when the editor loads its catalogue, so it shows on the first admin request. It is a developer error, not a user one.

A new instance starts with each field's default; a key in the widget's `$defaults` overrides it. Before a widget renders and before its inspector loads, `normalise()` fills any missing defaults, including those of each repeater row. So a field added to a widget later shows its default on existing content, and nothing is saved until the user edits it.

When the stored shape changes, override `upgrade()` to convert the older data. It must be pure, and it must return data that is already in the current shape unchanged. `normalise()` applies it before filling defaults, so `render()` and `preview()` always see the current shape. A widget that overrides it is flagged `upgrades` in the catalogue. When one is selected, the editor asks `…/inspector` for the upgraded data and shows that in the inspector. The first edit starts from the upgraded data, so the saved instance holds the current shape only. An instance nobody edits is saved exactly as it was.

```php
public function upgrade(array $data): array
{
    if (!array_key_exists('title', $data) && array_key_exists('sTitle', $data)) {
        $data['title'] = $data['sTitle'];
    }
    unset($data['sTitle']);

    return $data;
}
```

A field can also be edited inline: call `editable()` for it in `render()`. The inspector leaves that field out, because the page is where it is changed. A field the preview does not mark, including one inside a repeater row, stays in the inspector. A repeater row has no path to address it, so it cannot be edited inline.

### Field types

A field type is a PHP class and an inspector control, and any component can supply one. module-cms ships the types above, apart from `cdn_object`, which module-cdn ships with its own control.

The class implements `Nails\Cms\WidgetEditor\Interfaces\Field`, usually by extending `Nails\Cms\WidgetEditor\Field\Base`, and lives in the component's `Cms\Fields` namespace (`App\Cms\Fields\Colour`, `Nails\Cdn\Cms\Fields\ObjectPicker`), where it is discovered like a widget. It names its type, adds its own settings, and points at the script that registers its control:

```php
namespace App\Cms\Fields;

use Nails\Cms\WidgetEditor\Field\Base;

class Colour extends Base
{
    protected string $palette = 'brand';

    public static function getType(): string
    {
        return 'colour';
    }

    public static function getAssets(): array
    {
        return [['admin.colour-field.min.js', 'app']];
    }

    public function palette(string $palette): static
    {
        $this->palette = $palette;
        return $this;
    }

    protected function defaultValue(): mixed
    {
        return '';
    }

    protected function config(): array
    {
        return ['palette' => $this->palette];
    }
}
```

`config()` adds to what the control receives as `field`. `validate()` throws for a setting the editor cannot use. `fill()` fills what a stored value is missing (a repeater fills its rows). `getCanvasEdit()` returns `'text'`, `'rich'` or `'rich-basic'` for a type that can be edited on the canvas, and null otherwise.

Each type name belongs to one class. A class that extends a type, keeps its name and lives in a higher-precedence component (the app before modules) replaces it, along with its assets. Two unrelated classes with one name throw when the editor loads.

The script registers a Vue 3 component under the same type name. Build it with `vue` as a webpack external, as module-cms does, so it uses the editor's copy of Vue. It can load before or after the editor:

```js
import ColourControl from './ColourControl.vue';

(window.NAILS_CMS_WIDGET_EDITOR_FIELDS = window.NAILS_CMS_WIDGET_EDITOR_FIELDS || []).push({
    type: 'colour',
    component: ColourControl, // props: field, value. Emits `update` with the new value.
    isEmpty: (value) => value === '',        // optional: what counts as missing for required()
    coerce: (raw) => String(raw || ''),      // optional: what is stored for what the control reports
    summary: (value) => value || 'None',     // optional: the one line a wide field shows
    layout: 'label',                         // optional: 'block' for a control with several buttons or inputs
});
```

The editor draws the label, tip, required marker and warning around the component, and the summary and modal for a wide field. A later definition for a type replaces an earlier one, including the built-ins. `window.NAILS_CMS_WIDGET_EDITOR.components` holds `FieldControl`, which draws one field (a control can nest fields with it), and `Modal`, the editor's dialog. A field whose type has no registered control shows a warning in the inspector rather than a control.

### Kitchen sink

`Nails\Cms\Cms\Widgets\KitchenSink` uses every field type and setting Nails ships: text, both rich text editors on the canvas and in the inspector, a wide textarea and repeater, conditional fields, number limits, and CDN pickers at the top level and in a row. Use it to check the inspector and the canvas after a change. It is in the Developer group, and hidden in production.

## Not in this editor yet

Content the rich text schema does not keep (tables, embeds and unknown blocks) is not shown in the editor. The raw HTML view that warns about this is not built yet. The field is written back only if the user edits it, so such content is lost only when the user saves an edit. A basic field also drops headings, lists, quotes and code blocks. Links are not in the field types yet; `cdn_object` covers images and files only.

Automated tests: `yarn test:js` in `module-cms` covers the document tree, the serialiser, the canvas document and its helpers, the area rules, the session commands, the field type registry, and the field rules (visibility, required warnings, repeater rows, coercion, summaries). `yarn test:js` in `module-cdn` covers the `cdn_object` rules. `phpunit` covers `editable()` in preview and render mode, the edit mode each field type allows, the paths of nested instances, the field types and their discovery, and the kitchen sink. The inspector, the picker's event handling and the modal were checked by hand in a browser against a test area; there is no automated browser test.
