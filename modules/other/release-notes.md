---
description: >-
  This module brings a means of extracting release notes from your GitHub tags
  and rendering them for your customers in Admin.
---

# Release Notes

If you're following a good deployment protocol then you'll likely be tagging your releases in your repository and writing release notes as part of the tag. If you wish to give these notes to your customers then you might copy and paste them into a weekly email.

This module allows you to take these tags and display them, automatically, as both a [dashbaord widget](../admin/dashboard-widgets.md) and a full [admin controller](../admin/controllers/). Tag messages are parsed as markdown and presented to the user in full HTML glory allowing you to add content such as links, lists, and basic formatting.

Additionally, [nominated people can be notified](release-notes.md#notifications) whenever a new release is fetched.

{% hint style="info" %}
This module uses the GitHub API. If your repo is hosted elsewhere then, unfortunately, this module will not work.
{% endhint %}

![The dashboard widget](<../../.gitbook/assets/Screenshot 2021-08-12 at 15.30.53.png>)

## Configuration

This module is simple to configure; all required settings are available via [Admin](../admin/) under `Settings` in the admin sidebar.

| Property  | Description                                                                                                                                                                                                                             |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Repo**  | This should be the repo you wish to query.                                                                                                                                                                                              |
| **User**  | <p>This is the user to authenticate as.</p><p><em>Authentication is optional, however you may find you hit rate limits quickly if using anonymous queries. If your repo is private then this field will be required.</em></p>           |
| **Token** | <p>This is the access token to authenticate with.</p><p><em>Authentication is optional, however you may find you hit rate limits quickly if using anonymous queries. If your repo is private then this field will be required.</em></p> |

## Fetching

The module will automatically fetch new tags every hour using a [cron task](../cron.md). You can fetch on-demand using the [console command](../cdn/console.md) `nails releasenotes:fetch`. Each time the task executes new tags will be fetched and stored in the database.

{% hint style="info" %}
Note that fetches only happen once. If the tag is updated in the repo then the update will **not** be included.
{% endhint %}

Once stored in the database, tags can be viewed via the Dashboard Widget and/or via the admin controller. The widget is limited to the 10 most recent tags.

## Editing

If you wish, you can edit tags in admin once they're in the database. You may wish to do this if the original tag is badly formed, missing details, or simply requires tweaking.

{% hint style="info" %}
When editing the tag you are presented with the raw tag body, including things like GPG signatures.
{% endhint %}

## Notifications

After [fetching](release-notes.md#fetching) new tags, the console command will look for notification definitions in order to send a summary email with the newly discovered tags.

Notification definitons are classes which exist at `App\ReleaseNotes\Notification` and implement the `Nails\ReleaseNotes\Interfaces\Notifiucation` interface:

```php
namespace App\ReleaseNotes\Notification;

use Nails\ReleaseNotes\Interfaces\Notification;

class Admin implements Notification
{
    public function getEmails(): array
    {
        return [
            'admin@example.com',
        ];
    }
}
```
