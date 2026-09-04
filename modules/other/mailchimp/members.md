---
description: Members are individual subscribers to a MailChimp Audience.
---

# Members

The Member API allows you to list, create, update, archive, unarchive, and delete memebrs from an [audience](audiences.md).

{% hint style="info" %}
See [Audiences](audiences.md) for how to create, udpate, and delete audiences.
{% endhint %}

## Getting the API

Members must belong to an audience, so in order to get the Members API you must request it from the context of an audience. You can do this in two ways, both routes amount to the same thing so the best choice is dependant on your needs.

### From the Client

```php
use Nails\MailChimp;

/** @var Mailchimp\Factory\Audience **/
$oMemberApi = $oClient->members('{AUDIENCE_ID}');
```

{% hint style="warning" %}
The above is a shorthand of the below, and will still result in two API calls.
{% endhint %}

### From the Audience

```php
use Nails\MailChimp;

/** @var Mailchimp\Factory\Audience **/
$oAudienceApi = $oClient->audiences();

/** @var MailChimp\Resource\Audience **/
$oAudience = $oAudienceApi->getById('{AUDIENCE_ID}');

/** @var MailChimp\Factory\Member **/
$oMemberApi = $oAudience->members(); 
```

## Listing all Members

To list all members of the audience use the Member API's `getAll` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Member[] */
$aMembers = $oMemberApi->getAll();
```

## Get a single Member

To get a single Member, use the Member API's `getByEmail(string $sEmail)` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Member */
$oMember = $oMemberApi->getByEmail('{MEMBER_EMAIL}');
```

## Create a new Member

To create a new Member, use the Member API's `create(array $aParameters)` method:

{% hint style="info" %}
See the [MailChimp docs](https://mailchimp.com/developer/reference/lists/list-members/) for an overview of the fields which you can pass to this method.
{% endhint %}

```php
use Nails\MailChimp;

/** MailChimp\Resource\Member */
$oMember = $oMemberApi->create([
    'email_address' => 'bugs.bunny@example.com',
    'email_type'    => 'html',
    'status'        => 'subscribed',
    'merge_fields'  => [
        'FNAME' => 'Bugs',
        'LNAME' => 'Bunny',
    ]
]);
```

## Update a Member

To update an existing Member use the Member API's `update` method:

```php
use Nails\MailChimp;

/** MailChimp\Resource\Member */
$oMember = $oMemberApi->update(
    '{MEMBER_EMAIL}',
    [
        'merge_fields' => [
            'FNAME' => 'Daffy',
            'LNAME' => 'Duck',
        ]
    ]
);
```

## Delete a Member

To delete an existing Member use the Member API's `delete` method:

```php
use Nails\MailChimp;

$oMemberApi->delete('{MEMBER_EMAIL}');
```

## Member Tags

Adding arbritrary text tags to members is a powerful way of organsing your audience members - automations and segments can be configured based on a member's tags.

### Getting the Tags API

Tags must belong to a memver, so in order to get the Tags API you must request it from the context of an member.&#x20;

```php
use Nails\MailChimp;

/** @var MailChimp\Resource\Member **/
$oMember = $oAudience
    ->members()
    ->getByEmail('bugs.bunny@example.com');

/** @var MailChimp\Factory\Tag **/
$oTagsApi = $oMember->tags();
```

### List Tags on a Member

```php
use Nails\MailChimp;

/** @var MailChimp\Resource\Tag[] **/
$aTags = $oMember->tags()->getAll();
```

### Add Tags to a Member

Add Tags using the Tag API's `add` method, note that this action will **not** remove any existing tags.

```php
$oMember
    ->tags()
    ->add([
        'Tag One',
        'Tag Two'
    ]);
```

{% hint style="info" %}
If a tag doesn't yet exist in MailChimp, it will be created automatically.
{% endhint %}

### Remove Tags from a Member

Remove Tags using the Tag API's `remove` method:

```php
$oMember
    ->tags()
    ->remove([
        'Tag One',
        'Tag Two'
    ]);
```

### Set Tags on a member

To set/remove tags in a single operation use the Tag API's `set` method. This method allows you to pass an array of tags and specify explicitly whether that tag is active or inactive.

```php
$oMember
    ->tags()
    ->set([
        ['name' => 'Tag One', 'status' => 'active'],
        ['name' => 'Tag Two', 'status' => 'inactive'],
    ]);
```

{% hint style="info" %}
Existing tags which are not in the list will be left attached to the member.
{% endhint %}

