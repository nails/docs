---
description: Generate sitemaps for your application.
---

# Sitemap

Site maps are  important for maintaining healthy SEO, they are also tedious to maintain. This module provides an interface for defining your application's site map in code and refreshing it automatically.

The site map is composed of URLs which are returned by [Generators](sitemap.md#generators) (classes you define which describe your application's URLs). You should create generators to cover each aspect of your site. For example, you might create a generator to cover all the `Articles` on your site, and another for all the `Books`.

The module provides a console command which, when executed, will generate a sitemap which is accessible `/sitemap.xml`.

## Generators

The Sitemap service looks for classes defined at `App\SiteMap\Generator` which implement the `Nails\SiteMap\Interfaces\Generator` interface. These classes should return an array of `Nails\SiteMap\Factory\Url` objects.

For example, using an `Article` model as an example, the generator might look like this:

```php
namespace App\SiteMap\Generator;

use Nails\Common\Exception\FactoryException;
use Nails\Factory;
use Nails\SiteMap;

class Article implements SiteMap\Interfaces\Generator
{
    /**
     * Returns an array of URLs for the sitemap
     *
     * @return SiteMap\Factory\Url[]
     * @throws FactoryException
     */
    public function execute(): array
    {
        /** @var \App\Model\Article $oModel */
        $oModel = Factory::model('Article', 'app');
        $aUrls  = [];

        /** @var \App\Resource\Article $oArticle */
        foreach ($oModel->getAll() as $oArticle) {

            /** @var SiteMap\Factory\Url $oUrl */
            $oUrl = Factory::factory('Url', SiteMap\Constants::MODULE_SLUG);
            $oUrl->setUrl($oArticle->getUrl());

            $aUrls[] = $oUrl;
        }

        return $aUrls;
    }
}
```

{% hint style="info" %}
Some modules define their own generators for you.
{% endhint %}

## Refreshing the sitemap

There are various ways of refreshing the sitemap, the method you use depends on your specific use-case:

### Manually

The following console command is made avaiable by this module which, when executed, will generate the sitemap:

```bash
nails sitemap:generate
```

### Event

Similar to [generated routes](../../key-concepts/routing.md#generated-routes), you can trigger a sitemap refresh by triggering the `xxx` [event](../../core-services/event.md#triggering-events).

### Trait

@todo - trait which triggers a sitemap refresh on create/save.

### Cron

Of course, you can opt to refresh the sitemap on a schedule of your choosing by utilising the [cron module](../cron.md) and creating a task which executes the module's console command.



