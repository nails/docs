---
description: This page describes the CDN's object API.
---

# Objects

{% hint style="warning" %}
🔐This is an [authenticated](../../api/consuming.md#authentication) endpoint - anonymous uploads can be granted through the use of [upload tokens](objects.md#upload-token).
{% endhint %}

## Create an object

<mark style="color:green;">`POST`</mark> `/api/cdn/object/create`

This method allows for an object to be created. It expects the binary data to be in the `$_FILES` array under the key `upload`.&#x20;

#### Headers

| Name         | Type   | Description                             |
| ------------ | ------ | --------------------------------------- |
| X-Cdn-Bucket | string | Which bucket to upload the file to      |
| X-Cdn-Token  | string | A CDN upload token for anonymous upload |

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

### Upload Token

File uploads require authentication. however, to facilitate anonymous uploads an upload token can be generated and used to grant temporary upload permissions to a user.

The upload token is a short-lived and will grant an anonymous user the permission to upload files when it is passed via the `X-Cdn-Token` header at time of upload.

```php
use Nails\Cdn;

/** @var Cdn\Service\Cdn $oCdn */
$oCdn = Factory::service('Cdn', Cdn\Constants::MODULE_SLUG);

/** @var Cdn\Resource\Token */
$oToken = $oCdn->generateToken();

/**
 * {
 *     "token": "xxxx-xxxx-xxxx",
 *     "expires": "2020-03-31 12:34:56"
 * }
 */
```

{% hint style="info" %}
By default tokens are valid for 1 hour. if you wish to specify a custom expiry time, pass a `\DateTime`, `\Nails\Common\Resource\DateTime`, or `Y-m-d H:is` string to `generateToken($mExpiry)`
{% endhint %}

## List objects

<mark style="color:blue;">`GET`</mark> `/api/cdn/object`

Lists objects in the CDN.

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

## Delete an object

<mark style="color:green;">`POST`</mark> `/api/cdn/object/delete`

Deletes an object from the CDN.

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

## Restore an object

<mark style="color:blue;">`GET`</mark> `/api/cdn/object/restore`

Restores an object.

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

## Search objects

<mark style="color:blue;">`GET`</mark> `/api/cdn/objects/search`

Searches the CDN.

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

## List objects in trash

<mark style="color:blue;">`GET`</mark> `/api/cdn/object/trash`

Lists items in the CDN's trash.

{% tabs %}
{% tab title="200 " %}
```
```
{% endtab %}
{% endtabs %}

