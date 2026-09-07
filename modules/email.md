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

### The email shell

Every email is wrapped in a shared header and footer which bring unity to all the emails your application sends — a masthead, a card, and a footer wrapped around whatever the body view renders:

```
masthead          logo, or the app name as text
card
    subject banner
    greeting      "Hi {{sentTo.first_name}},"
    ← body view →
    sign off
footer
    view online / unsubscribe links
    postal address
```

You very rarely need to touch the header or footer views directly. There are three ways to customise the shell, cheapest first.

**1. Do nothing.** The masthead looks for a logo — your app's composer `extra.nails.data.logo_url`, then the `APP_LOGO_URL` config value, then `assets/img/logo.{png,jpg,gif}` — and falls back to your app's name as text if none is found.

**2. Set the content settings in Admin.** Under *Admin → Settings → Email → Content*:

| Setting          | Effect                                     |
| ---------------- | ------------------------------------------- |
| Sign Off         | A block rendered below the body of every email |
| Footer Address   | A postal address in the footer              |

Both are omitted entirely (no empty row) when left blank, and both are rendered through the email's Mustache context, so `The {{appName}} Team` works.

**3. Override a slot.** The shell is broken up into small, independently-overridable **slots**. Drop a file into `application/modules/email/views/structure/slots/` using one of the names below and it wins over the module's own copy — no registration required:

| Slot | Default |
| --- | --- |
| `masthead` | The discovered logo, else the app name as text |
| `greeting` | `Hi {{sentTo.first_name}},`, falling back to `Hi,` |
| `signoff` | The *Sign Off* setting, else nothing |
| `footer_links` | View-online and unsubscribe links |
| `footer_address` | The *Footer Address* setting, else nothing |
| `styles` | Nothing — see [Overriding the CSS](email.md#overriding-the-css) |

Each slot (bar `styles`) has a `<slot>_plaintext` counterpart for the plain text part of the email, which you should override alongside it — e.g. `slots/masthead.php` and `slots/masthead_plaintext.php`.

{% hint style="info" %}
Slots are given `$emailObject`, the same object available to body views.
{% endhint %}

#### Overriding the header and footer directly

Slots cover the vast majority of customisation, but if you need to restructure the shell itself, you can still override the two structure views wholesale:

```bash
application/modules/email/views/structure/email_header.php
application/modules/email/views/structure/email_header_plaintext.php
application/modules/email/views/structure/email_footer.php
application/modules/email/views/structure/email_footer_plaintext.php
```

{% hint style="warning" %}
The header and footer are **two halves of one document**, not two independent views — the header opens tags (`<html>`, `<body>`, the layout tables) which the footer closes. If you override one you must override the other, and keep the tags balanced yourself. Prefer overriding a slot instead wherever you can.
{% endhint %}

### Styling emails

Body views are bare HTML fragments — a few paragraphs, a table, a button — styled entirely by classes the shell already provides. You should not open a layout `<table>` or add `style` attributes of your own. The most common classes:

```markup
<p class="alert alert-warning">Something needs your attention.</p>

<a href="{{url}}" class="btn btn-primary">Pay online now</a>

<table class="table table--list">
  <tr><td>Reference</td><td>{{invoice.ref}}</td></tr>
</table>

<div class="panel">
  <div class="panel__header">Order summary</div>
  <div class="panel__body">…</div>
</div>
```

Other components available out of the box include `badge`, `divider`, `hero` and `code`, plus text/spacing utilities such as `text-muted`, `text-center` and the `m-*`/`p-*` spacing scale. See the module's [README](https://github.com/nails/module-email#class-reference) for the full class reference.

#### Overriding the CSS

All styling compiles from Sass down to a single `<style>` block embedded in the header — there's no CSS inliner, and no `<link>` support in most email clients.

`module-email` is a Composer dependency, installed into `vendor/` like any other package — it isn't cloned or rebuilt in place, and nothing under `vendor/` should be edited. To customise the CSS, you compile your **own** stylesheet, in your app, from the framework's Sass, then hand the result to the module through the `styles` slot:

{% stepper %}
{% step %}
### Set your tokens

Every colour, spacing and radius value is a Sass variable prefixed `$email-` and marked `!default`. In your app's own Sass — not the module's — set the ones you want to change, then import the framework:

```scss
// application/assets/sass/email.scss
$email-color-brand: #00a0b0;
$email-radius-card: 0;

@import '../../../vendor/nails/module-email/assets/sass/email';
```

Only need one component? Import its partial directly instead of the whole manifest — e.g. `.../assets/sass/email/components/button` — and you get just that component, complete with its own media-query and dark-mode rules.
{% endstep %}

{% step %}
### Build it with your app's own tooling

This is a plain Sass file that lives in your app, compiled by whatever already builds the rest of your app's CSS. There's no module-specific build step to run, and nothing inside `vendor/nails/module-email` to change or recompile.
{% endstep %}

{% step %}
### Load the compiled CSS through the `styles` slot

Override `application/modules/email/views/structure/slots/styles.php` to inline the file you just built:

```php
// application/modules/email/views/structure/slots/styles.php
<style type="text/css">
    <?php require NAILS_APP_PATH . 'assets/css/email.min.css'; ?>
</style>
```

It has to be an inline `<style>` block rather than a `<link>`, since there is no CSS inliner in the pipeline. This slot renders *after* the module's own default stylesheet, so — depending on whether your build imported the framework's Sass at all — it either cascades a handful of overrides over the top, or replaces the styling outright.
{% endstep %}
{% endstepper %}

{% hint style="info" %}
Sass tokens compile down to literal values rather than CSS custom properties — Outlook's rendering engine has no `var()` support, so this is deliberate rather than an oversight.
{% endhint %}

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
| `preheader`       | The snippet a client shows next to the subject in the message list. Set this via `data()` to fill it in.                     |

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
