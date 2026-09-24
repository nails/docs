---
description: Opt-in collapsible fieldset cards.
---

# Collapsible Fieldsets

A fieldset with `data-collapse` turns its legend into a disclosure button so the group can be folded away. This is opt-in; fieldsets without the attribute stay open.

```markup
<fieldset data-collapse>
    <legend>Associations</legend>
    <!-- open on load -->
</fieldset>

<fieldset data-collapse="closed">
    <legend>Related content</legend>
    <!-- closed on load -->
</fieldset>
```

| Attribute value | Initial state |
| --------------- | ------------- |
| (bare), `open`, `expanded`, `true`, `1` | Open |
| `closed`, `close`, `collapsed`, `false`, `0` | Closed |

The closed default is applied in CSS from the attribute so the body is hidden before JS runs.

## Errors

A fieldset that contains a validation error always starts open, whatever the attribute asks for.

## Limits

The legend’s contents are moved into a `<button>`. A fieldset is ignored (and a console warning is logged) when:

* it has no `<legend>`
* the legend already contains a control (`a`, `button`, `input`, `select`, `textarea`, `label`, …)

Opening a fieldset calls `refreshUi()` so plugins inside it can measure themselves.
