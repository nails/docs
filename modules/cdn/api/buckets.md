---
description: This page describes the CDN's bucket API.
---

# Buckets

{% hint style="info" %}
This endpoint leverages the [CRUD API Controller](../../api/building.md#crud-controllers) and is available at `/api/cdn/bucket/{id}`
{% endhint %}

{% hint style="warning" %}
🔐This is an [authenticated](../../api/consuming.md#authentication) endpoint, and is only intended to be used by Administrators with the `admin:cdn:manager:bucket:create` [permission](../../admin/user-permissions.md).
{% endhint %}

## List bucket contents

<mark style="color:blue;">`GET`</mark> `/api/cdn/bucket`

This endpoint lists the contents of a bucket, in paginated form.

#### Query Parameters

| Name       | Type    | Description                 |
| ---------- | ------- | --------------------------- |
| bucket\_id | integer | The bucket's ID             |
| page       | integer | The desired page of results |

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

sss
