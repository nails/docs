---
description: This module provides an interface for sending structured email.
---

# Email

Sending email in Nails is straight forward and structured. Emails are defined in advance, and use templates for their content. [Sending an email at runtime](email.md#sending-email) is achieved through email [Factories](../key-concepts/factory/factories.md).

## Email Definitions

In order to send an email, its type has to first be defined. Defining an email tells the system details about what the email is used for as well as what template to use when sending.

### Creating Email Definitions

Create new email definitions using the [Command Line Tool](../command-line-tool.md).

```bash
nails make:email
```

This will create the following templates:

```bash
# nails make:email Book\\Reviewed
application/modules/email/views/book/reviewed.php
application/modules/email/views/book/reviewed_plaintext.php
```

Additionally it will configure a [Factory](../key-concepts/factory/factories.md) to make sending email easy and object orientated:

```bash
# nails make:email Book\\Reviewed
src/Factory/Email/Book/Reviewed.php

# This can be loaded using:
# \Nails\Factory::factory('EmailBookReviewed', 'app');
```

Finally, it will add the definition to `./application/config/email_types.php`, where you can customise various aspects of the email.

{% hint style="success" %}
Remember to update the description and subject line in `email_types.php`!
{% endhint %}

## Sending Email

Send email using the Factory created when you built the definition. The Email factories have utility methods for specifying who the email should be sent to (`to()`, `cc()` and `bcc()`), as well as passing in dynamic data which can be used by the [templates](email.md#templates).

An example of sending an email to a user after they have submitted a review of a book:

```php
/* @var \App\Factory\Email\Book\Reviewd $oEmail */
$oEmail = Factory::factory('EmailBookReviewed', 'app');
$oEmail
    // The recipient's email, user object, or user ID
    ->to(activeUser())
    // Data to make available to the templates
    ->data([
        'book'     => [
            'label'  => $oBook->label,
            'author' => $oBook->author,
        ],
        'review' => [
            'title'    => $oReview->title,
            'body'     => $oReview->body,
            'reviewer' => $oReview->reviewer
        ],
    ])
    ->send();
```

## Templates

Nails encourages compatibility with email clients by offering both HTML and plaintext templates.

For the example above, the templates would be located at:

```bash
application/modules/email/views/book/reviewed.php
application/modules/email/views/book/reviewed_plaintext.php
```

### Header and Footer

Your application can specify global header and footer templates to use. These will top-and-tail all emails and bring unity to all the emails sent by your application. To override the default header and footer you can add the following files:

```bash
# Header views
application/modules/email/views/structure/email_header.php
application/modules/email/views/structure/email_header_plaintext.php

# Footer views
application/modules/email/views/structure/email_footer.php
application/modules/email/views/structure/email_footer_plaintext.php
```

### Template Data

Data passed to the template via the `data()` method can be rendered in the template using [Mustache](https://mustache.github.io/) templating. For example, the data in the above example might be rendered like this:

```markup
<p>
    Thanks, {{review.reviewer}}!
<p>
<p>
    We have received your review of {{book.label}}.
</p>
```

In addition to the user-supplied data, Nails will populate the following data variables:

| Key               | Description                                                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `emailType`       | Details about the type of email being sent.                                                                                   |
| `emailRef`        | The email's unique reference.                                                                                                 |
| `sentFrom.name`   | The sender's name                                                                                                             |
| `sentFrom.email`  | The sender's email                                                                                                            |
| `sentTo`          | Details about the recipient. If the recipient is a known user then this will contain user details such as their ID, name, etc |
| `appName`         | The value of `APP_NAME` configuration                                                                                         |
| `url.viewOnline`  | The URL where the email can be viewed in a browser.                                                                           |
| `url.unsubscribe` | The URL where the suer can unsubscribe from this email type. (If the email cannot be unsubscribed from, this will be blank)   |
| `url.trackerImg`  | The URL of the tracker image for that URL.                                                                                    |

### PHP in templates

It is **strongly** recommended to avoid using PHP in email templates. Templates which contain PHP cannot be easily overridden in [Admin](admin/) (a feature to allow your clients to edit email templates).

It is possible, however to call simple PHP functions in templates using a Mustache-style syntax. Functions which accept a single parameter (e.g. `siteUrl()`, `asset()` or `date()`) can be rendered as follows:

```markup
{{ siteUrl('some/url') }}
{{ asset('img/avatar.jpg') }}
{{ date('Y-m-d') }}
```

{% hint style="warning" %}
If using PHP is unavoidable then email data is available via the `$emailObject->data` variable. If PHP is detected in the template Admin will prevent templates being overridden.
{% endhint %}
