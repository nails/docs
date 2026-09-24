---
description: The admin modal plugin, used by Confirm and app code.
---

# Modal

Admin’s modal is a small JS API plus optional markup. [Confirm](confirm.md), dashboard widget removal, and Reset Nav all use it. Prefer this over Bootstrap’s `$.modal()`.

## Create from JS

```javascript
let modal = window.NAILS.ADMIN.getInstance('Modal').create();

modal
    .setTitle('Copy preview link')
    .setBody('<textarea class="form-control">https://example.com/preview</textarea>')
    .clearActions()
    .addAction('Copy', ['btn-primary'], (event, modal) => {
        // ...
        modal.hide();
    })
    .addAction('Close', ['btn-default'], (event, modal) => {
        modal.hide();
    })
    .show();
```

`setBody()` accepts a string, an element, or an array of elements, then calls `refreshUi()` so plugins inside the body (Select, Copy to Clipboard, Revealer) bind.

### Instance methods

| Method                         | Description                                              |
| ------------------------------ | -------------------------------------------------------- |
| `setTitle(html)`               | Title bar                                                |
| `setBody(html\|element\|[])`   | Body; refreshes UI                                       |
| `addAction(label, classes, cb)`| Footer button. `classes` defaults to `['btn-default']`   |
| `clearActions()`               | Remove footer buttons                                    |
| `setActions(html\|element\|[])`| Replace the footer wholesale                             |
| `show()` / `hide()` / `isShown()` | Open, close, query                                   |
| `onShow(cb)` / `onHide(cb)`    | Callbacks                                                |
| `scrollToTop()` / `scrollToBottom()` | Scroll the inner panel                         |

The action bar is hidden when it has no buttons. Escape closes the open modal.

[Copy to Clipboard](copy-to-clipboard.md) buttons work inside the body and the action bar.

## Existing markup

Any `.modal` in the page is upgraded on `refreshUi()`:

```markup
<div class="modal">
    <div class="modal__inner">
        <div class="modal__close">&times;</div>
        <div class="modal__title">Title</div>
        <div class="modal__body">Body</div>
        <div class="modal__actions"></div>
    </div>
</div>
```

`Helper::addModal($sTitle, $sBody, $bIsOpen)` queues a modal to render in the admin footer.

## Related

* [Confirm](confirm.md) — `a.confirm` links
* [Modalize](modalize.md) — edit a hidden chunk of a form in a modal
