---
description: Add modal confirmations to links.
---

# Confirm

This plugin intercepts clicks on `a.confirm` and asks the user to continue using the [Modal](modal.md) plugin.

The title and the body of the modal can be configured via the `data-title` and `data-body` attributes. They default to “Are you sure?” and a generic confirmation sentence.

```markup
<a href="/dangerous/action"
   class="btn btn-danger confirm"
   data-title="Delete this item?"
   data-body="This action cannot be undone, are you sure you wish to continue?"
>
   do a dangerous thing
</a>
```

OK follows the link’s `href` (including `target="_blank"`, `_parent`, and `_top`). Cancel closes the modal.

{% hint style="warning" %}
This plugin only works with normal hyperlinks. It does not submit a form for you.
{% endhint %}
