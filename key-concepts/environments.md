# Environments

Environments are a simple way of adjusting the behaviour or configuration of the application depending on context. Nails supports three distinct environments:

## Supported Environments

### `DEVELOPMENT`

{% hint style="info" %}
This is the default environment.
{% endhint %}

The development environment is the most verbose in terms of logging and error reporting. It is also sensitive to contact with the outside world and some modules (e.g. [Email](../modules/email.md)) will restrict what they do when running in the `DEVELOPMENT` environment.

### `STAGING`

Similar to `DEVELOPMENT`  logging and errors are verbose but intended to be closer to `PRODUCTION`.

### `PRODUCTION`

The live environment, keys and credentials are all for the real thing.

## Protecting Environments

Sometimes it can be desirable to restrict access to a particular environment, e.g. a stopping the public from accessing a `STAGING` instance. Nails offers a simple mechanism for implementing basic authentication across the entire application when it detects a specific environment.

{% hint style="warning" %}
This is not designed to be a cryptographically secure method of protection.
{% endhint %}

### Defining users and passwords

Users (and their corresponding passwords) are defined in a JSON file at the web-root, which takes the following form:

```
protect.{{ENVIRONMENT}}.users.json
```

So, for example, to protect the `STAGING` environment you'd create:

```
protect.staging.users.json
```

{% hint style="info" %}
Note that the environment is lowercase.
{% endhint %}

The JSON is a series of key/value pairs, where the key is the username and the value is a `sha256` encoded password (with no salt). For example, for two users (`john` and `amy`) with passwords `password` and `something` respectively, the JSON would look like this:

```javascript
{
    "john": "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8",
    "amy": "3fc9b689459d738f8c88a3a48aa9e33542016b7a4052e001aaa536fca74813cb"
}
```

### Generating the hash

There are many online tools to do this, but it is recommended to use a local system when encrypting secrets. You can use PHP to encode a string on the command line as follows, and copy it to the clipboard:

```
php -r 'echo hash("sha256", "password-to-encode");' | pbcopy
```

### Whitelisting IPs

Whitelisting an IP (i.e not requiring a password) is straightforward. Similar to the above, create a JSON file at the web-root which contains an array of IP and IP Ranges.

```
protect.{{ENVIRONMENT}}.whitelist.json
```

So, for example, to define a whitelist for the `STAGING` environment you'd create:

```
protect.staging.whitelist.json
```

An example of the contents of this file might look like this:

```javascript
[
    '123.456.78.0/15',
    '123.456.79.1',
]
```

## Supplying credentials

By default, when a protected environment is detected the basic auth headers are sent and the browser will pop up a username/password dialogue.

In some instances you might want to bypass the protection, but not know, or be able to whitelist the IP addresses - e.g. using a proxy, or CDN.

You can pass the credentials into the URL, as is standard for Basic Authentication:

```
https://username:password@example.com
```

Or, you can send the credentials as headers:

```
X-Auth-User: username
X-Auth-Password: my-password
```

{% hint style="warning" %}
These methods are not encrypted, so only do this over a secure connection.
{% endhint %}

