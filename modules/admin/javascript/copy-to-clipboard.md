---
description: >-
  This plugin provides a simple API for copying data to the user's clipboard,
  either programatically or when an item is clicked.
---

# Copy to Clipboard

This module copies text to the user's clipboard when an element is clicked (typically a button). To use, add the class `js-copy-to-clipboard` to elements. The actual text to copy is defined using the `data-clipbopard-text` attribute.

```markup
<button
   class="btn btn-default js-copy-to-clipboard"
   data-clipboard-text="Text to copy"
>
   Copy to Clipboard
</button>
```
