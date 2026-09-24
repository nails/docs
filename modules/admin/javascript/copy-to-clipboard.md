---
description: >-
  This plugin provides a simple API for copying data to the user's clipboard,
  either programatically or when an item is clicked.
---

# Copy to Clipboard

This plugin copies text to the user's clipboard when an element is clicked (typically a button). Add the class `js-copy-to-clipboard` to the element. The text to copy is `data-clipboard-text`.

```markup
<button
   class="btn btn-default js-copy-to-clipboard"
   data-clipboard-text="Text to copy"
>
   Copy to Clipboard
</button>
```

A short “Copied” bubble appears above the button. The plugin rebinds on `refreshUi()`, so buttons created inside a [Modal](modal.md) work once the body is set.
