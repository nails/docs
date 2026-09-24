---
description: Top-level workspaces on an admin edit screen.
---

# Screen Tabs

Screen tabs are the heavier workspace switcher that sits above any nested [tab](tabs.md) groups — for example Microsite | Pages. Controls are rendered in the view; they are not inferred from fieldset legends.

## Markup

```markup
<nav class="screen-tabs js-screen-tabs" role="tablist">
    <button type="button" class="screen-tabs__tab js-screen-tab" data-screen="details">
        Details
    </button>
    <button type="button" class="screen-tabs__tab js-screen-tab" data-screen="related">
        Related
        <span class="screen-tabs__count">3</span>
    </button>
</nav>

<div class="screen-tabs__panel js-screen-panel" data-screen="details">
    <!-- details workspace -->
</div>
<div class="screen-tabs__panel js-screen-panel" data-screen="related">
    <!-- related workspace -->
</div>
```

The nav and its panels must share a common ancestor (the surrounding `<form>` is used when present). A group with fewer than two workspaces is left alone.

## Hash

The URL hash deep-links to a workspace (`#related`). The first workspace is the empty hash. After a redirect, send the user to `#pages` to land on that workspace.

Set `data-hash="off"` on the nav to leave the hash alone (several groups on one page, as on the styleguide).

## Saving

A panel with `screen-tabs__panel--no-save` hides `.admin-floating-controls` while it is active. Use that for read-only workspaces that should not submit the form.

## Default workspace

On load the plugin shows, in order:

1. The first panel that contains a validation error (that tab also gets `has-error`)
2. The workspace encoded in the URL hash
3. The first workspace

Changing workspace calls `refreshUi()` on the newly shown panel.
