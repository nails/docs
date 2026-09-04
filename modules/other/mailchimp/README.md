---
description: >-
  The MailChimp Module provides a simple API for interfacing with MailChimp
  Audiences, Members, and Member Tags
---

# MailChimp

The MailChimp module provides a simple abstraction for interfacing with [Audiences](audiences.md), and [Members](members.md) in an object-orientated way.

## The Client

The MailChimp client is loaded using the [Factory](../../../key-concepts/factory/):

```php
use Nails\MailChimp;

/** @var MailChimp\Service\Client **/
$oClient = Factory::service('Client', MailChimp\Constants::MODULE_SLUG);
```

The client provides two utility methods for accesing [Audiences](audiences.md), and [Members](members.md):

```php
use Nails\MailChimp;

/** @var MailChimp\Factory\Audience **/
$oAudienceApi = $oClient->audiences();

/** @var MailChimp\Factory\Member **/
$oMemberApi   = $oClient->members();
```

In addition to this it provides methods for performing manual operations against the MailChimp API – these will call the API using the [configured credentials](./#configuring):

```php
$oClient->get(string $sEndpoint, array $aParameters);
$oClient->post(string $sEndpoint, array $aParameters);
$oClient->delete(string $sEndpoint, array $aParameters);
$oClient->patch(string $sEndpoint, array $aParameters);
```

## Configuring

The Client is configured by setting the following [configurations](../../../getting-started/configuration.md):

| Key                 | Description             |
| ------------------- | ----------------------- |
| `MAILCHIMP_API_KEY` | Your MailChimp API key. |

## Audiences

Audiences (historically known as lists), are where MailChimp stores your mailing list contacts.

{% content-ref url="audiences.md" %}
[audiences.md](audiences.md)
{% endcontent-ref %}

## Members

Members belong to audiences, and represent a single subscriber.

{% content-ref url="members.md" %}
[members.md](members.md)
{% endcontent-ref %}

