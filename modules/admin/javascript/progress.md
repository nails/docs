---
description: Markup-driven progress bars that redraw when their value or max changes.
---

# Progress

Add the class `js-admin-progress` to an empty element. The current amount is `data-value`. The total is `data-max`, which defaults to `100` when omitted. A `data-max` of `0` draws an empty bar.

```markup
<div class="js-admin-progress" data-value="4" data-max="20"></div>
```

The plugin paints a track, fill, and `4 / 20` label, and sets `role="progressbar"` with `aria-valuemin`, `aria-valuemax`, and `aria-valuenow` on the track.

Bars in the page HTML are bound on the initial `refreshUi()`. For a bar you insert later, call `refreshUi()` on that element (or a parent) after it is in the DOM.

To move the bar, set the attributes on the host. The plugin watches `data-value` and `data-max` and redraws. Write the attributes (`dataset.value`, `setAttribute`) rather than jQuery `.data()`, which does not update the attribute.

```javascript
meter.dataset.max = String(total);
meter.dataset.value = String(scanned);
```

`hidden` on the host is left to the caller.
