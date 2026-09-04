# Components

Components are simply packages which are installed via [Composer](https://getcomposer.org). What makes them a _Nails_ component is the presence of the `extra.nails` property in the package's `composer.json` file. This property is used to define the component's type as well as other settings.

Components are divided into three types:&#x20;

1. [Modules](./#modules)
2. [Drivers](./#drivers)
3. [Skins](./#skins)

Sometimes it's not practical or possible to distribute a component via Composer; for example, a skin might only be used by a specific project. For this reason it is also possible to [bundle component-like items within the application](./#app-components).

## Modules

Modules typically bring about complete functionality, but can also simply provide a few services which expose additional tools and behaviours. For example:

* The [CMS module](../../modules/cms/) brings about admin interfaces which allow the user to generate static pages, manage menus, create snippets as well as a front end mechanic for rendering these pages
* The [CDN module](../../modules/cdn/) provides API endpoints, services, admin controllers, and helpers for storing and serving files either from the disk, or from a file storage service like S3.
* The [PDF module](../../modules/other/pdf.md) provides a service for creating and saving PDFs, either to disk, or via the CDN module if it's available.

{% content-ref url="modules.md" %}
[modules.md](modules.md)
{% endcontent-ref %}

## Drivers

Drivers are much more restricted components which power a very specific bit of functionality provided by a module. These are organised via a Driver service provided by the module, and typically implement a trait enforced by the module. For example

* The [AWS driver](../../modules/cdn/drivers/aws-s3.md) for the [CDN module](../../modules/cdn/) allows files to be stored on S3.
* The [Stripe driver](../../modules/invoice/drivers/stripe.md) for the [Invoice module](../../modules/invoice/) provides support for Stripe when taking payments.

{% content-ref url="drivers.md" %}
[drivers.md](drivers.md)
{% endcontent-ref %}

## Skins

Skins are similar to drivers, but are orientated around user-facing components. Most modules allow you to bring-your-own-GUI entirely, but sometimes it's easier to leverage skins. For example, the [Invoice module](../../modules/invoice/) utilises a skin in order to generate the invoice PDF.

{% content-ref url="skins.md" %}
[skins.md](skins.md)
{% endcontent-ref %}

## App Components

App components are identical to Composer distributed components, except they are stored outside the vendor folder. You can utilise Composer's `repositories` key to symlink these packages into your vendor folder so that Nails recognises them.

Your component should reside in its own directory, and have its own `composer.json` file. At minimum this file should contain a `name` key as well as the relevant `extra.nails` data to declare it as a [module](modules.md), [driver](drivers.md), or [skin](skins.md).

You must then define the `repositores` key in your app's `composer.json` file and `require` the component as you would any other component.

Sample component, at `./components/custom-skin/composer.json`:

```javascript
{
    "name": "app/custom-skin",
    "description": "A custom skin for nails/module-example.",
    "extra": {
        "nails": {
            "name": "Custom Skin",
            "type": "skin",
            "forModule": "nails/module-example"
        }
    }
}
```

Your app's `composer.json` file:

```javascript
{
    ...
    "repositories": [
        {
            "type": "path",
            "url": "./components/custom-skin"
        }
    ],
    "require": [
        ...
        "app/custom-skin": "*",
        ...
    ]
}
```

Remember to execute `composer update` to install the component.

{% hint style="info" %}
Remember that this component is a first class Composer package and can take advantage of all of Composer's features, for exanple autoloading.
{% endhint %}
