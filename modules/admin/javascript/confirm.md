---
description: Add modal confirmations to links.
---

# Confirm

This module pops up a modal when links with the class `confirm` are clicked, allowing the user to confirm whether they wish to continue with the action or not.

The title and the body of the modal can be configured via the `data-title` and `data-body` attributes.

```markup
<a href="/dangerous/action"
   class="btn btn-danger confirm"
   data-body="This action cannot be undone, are you sure you wish to continue?"
>
   do a dangerous thing
</a>
```

{% hint style="warning" %}
This plugin only works with normal hyperlinks which would cause a page reload.
{% endhint %}
