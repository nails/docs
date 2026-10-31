---
description: Define RSS feeds in code and serve them from your application.
---

# RSS

This module lets you describe RSS feeds as classes. The module finds them, compiles each one to XML on a schedule, serves it at its own URL, and adds a `<link rel="alternate">` tag to the page `<head>` so browsers and feed readers can discover it.

```bash
composer require nails/module-rss
```

The module depends on the [CDN](../cdn/README.md), [Console](../console.md), and [Cron](../cron.md) modules.

## Defining a feed

The module discovers any instantiable class in a component's `Rss\Feeds` namespace that implements `Nails\Rss\Interfaces\Feed`. Discovery covers the app (`App\Rss\Feeds\…` in `src/Rss/Feeds/`) and every installed module. You do not register feeds anywhere.

The interface is large. Most feeds use the `Nails\Rss\Traits\Feed` trait, which implements everything except `getItems()`:

```php
namespace App\Rss\Feeds;

use Nails\Factory;
use Nails\Rss\Constants;
use Nails\Rss\Factory\Item;
use Nails\Rss\Factory\ItemCollection;
use Nails\Rss\Interfaces;
use Nails\Rss\Traits;

class Blog implements Interfaces\Feed
{
    use Traits\Feed;

    public function getDescription(): string
    {
        return 'The latest posts from our blog';
    }

    public function getItems(): ItemCollection
    {
        /** @var ItemCollection $items */
        $items = Factory::factory('ItemCollection', Constants::MODULE_SLUG);

        /** @var \App\Model\Post $model */
        $model = Factory::model('Post', 'app');

        foreach ($model->getAll(null, 25) as $post) {
            /** @var Item $item */
            $item = Factory::factory('Item', Constants::MODULE_SLUG);
            $item
                ->setTitle($post->label)
                ->setLink($post->url)
                ->setDescription($post->excerpt ?? '')
                ->setPubDate($post->published)
                ->setImage($post->cover_id);

            $items->add($item);
        }

        return $items;
    }
}
```

After you add, remove, or rename a feed, [rewrite the routes](../../key-concepts/routing.md#generated-routes) so its URL resolves:

```bash
nails routes:rewrite
```

### Defaults supplied by the trait

Override any of these methods on your feed class.

| Method | Default |
|---|---|
| `getSlug()` | Built from the class name. Everything up to and including `Rss\Feeds\` is collapsed into the vendor prefix, then the name is split on case changes and lower-cased: `App\Rss\Feeds\Blog` → `app-blog`, `App\Rss\Feeds\Insights\En` → `app-insights-en`. Slugs must be unique. |
| `getUrl()` | `rss/{slug}`, which is served by the module's controller. |
| `getTitle()` | The class name after `Rss\Feeds\`, split into words: `App\Rss\Feeds\LatestNews` → `Latest News`. |
| `getDescription()` | Empty. |
| `getLanguage()` | The language code of the app's default locale, for example `en`. |
| `getImage()` | `null`. |
| `getCategory()` | Empty. |
| `getCopyright()` | `Copyright {current year}, {APP_NAME}`. |
| `getXmlPath()` | `rss/feeds/{slug}.xml`, relative to the [FileCache](../../core-services/cache.md). |
| `shouldLink()` | `true`. |

Empty values are left out of the XML. Every channel also gets `<lastBuildDate>`, set when the feed is compiled, and `<ttl>60</ttl>`.

### Items

`getItems()` must return a `Nails\Rss\Factory\ItemCollection` of `Nails\Rss\Factory\Item` objects. Every setter is chainable and every field is optional.

| Setter | Output |
|---|---|
| `setTitle(string)` | `<title>` |
| `setLink(string)` | `<link>` and `<guid>`. The value is passed through `siteUrl()`, so a relative path becomes absolute. |
| `setPubDate(string\|DateTime)` | `<pubDate>` in RFC 2822 format. A string is parsed into a `Nails\Common\Resource\DateTime`. |
| `setDescription(string)` | `<description>` |
| `setAuthor(string)` | `<author>` |
| `setCategory(string)` | `<category>` |
| `setImage(string\|int\|CdnObject\|null)` | `<enclosure>`. See [images](#images). |

A value containing `<`, `>`, `&`, `"`, `'`, or `]]>` is wrapped in a CDATA section, so you can pass HTML in a description.

### Images

`getImage()` on the feed and `setImage()` on an item accept the same values:

* A string is treated as a URL and passed through `siteUrl()`.
* An integer is treated as a CDN object ID.
* A `Nails\Cdn\Resource\CdnObject` is used as-is.

A feed image becomes a channel `<image>`. A URL gives it a `<url>`; a CDN image gives it `<url>`, `<title>`, `<width>`, and `<height>`.

An item image becomes an `<enclosure>`. A URL gives it `url` and `length="0"`; a CDN image gives it `url`, `length` (the file size in bytes), and `type` (the MIME type).

A CDN object that is not an image, or an ID that does not exist, produces an empty `<image>` or an `<enclosure>` with only `length="0"`.

### Linking from the page head

When the system is ready, the module adds this tag to the page `<head>` for every feed whose `shouldLink()` returns `true`:

```html
<link rel="alternate" type="application/rss+xml" title="{title}" href="{url}">
```

Override `shouldLink()` to decide per request. For example, a site with one feed per language can link only the feed for the language being viewed:

```php
public function shouldLink(): bool
{
    return ACTIVE_LANGUAGE_ID === $this->getLanguageId();
}
```

## Compiling feeds

Feeds are compiled to XML and written to the FileCache. The `Nails\Rss\Cron\Task\Compile` cron task runs `rss:compile` every hour.

Compile every feed by hand with:

```bash
nails rss:compile
```

Pass `--feed` (or `-f`) one or more times to compile specific feeds. The value is the feed's fully-qualified class name:

```bash
nails rss:compile --feed="App\Rss\Feeds\Blog"
```

The console command prints the cache path of each compiled file.

## Serving feeds

A request to a feed's URL is handled by the module's `rss/feed` controller. It returns the cached XML with a `Content-Type` of `application/rss+xml`. If no cached file exists yet, the controller compiles the feed first. An unknown feed returns a 404.

Because the controller serves the cached file, items added since the last compile do not appear until the next hourly run. Run `rss:compile` after a change that must appear straight away.

## The Rss service

The `Rss` service discovers feeds once, when it is first built.

```php
/** @var \Nails\Rss\Service\Rss $rss */
$rss = \Nails\Factory::service('Rss', \Nails\Rss\Constants::MODULE_SLUG);

$rss->getFeeds();                         // FeedCollection of every discovered feed
$rss->getFeed(\App\Rss\Feeds\Blog::class); // a single feed, or null
```

`getFeed()` accepts a class name or a feed instance. It returns the first discovered feed that is an instance of that class.

## Customising the feed

The trait builds the XML in small protected methods. Override them on your feed class to change the output without rewriting `getXml()`:

| Method | Purpose |
|---|---|
| `getRssNodeAttributes()` | Attributes for the root `<rss>` element. Defaults to `['version' => '2.0']`. Add namespaces here, such as `xmlns:atom`. |
| `getChannelNodeChildren($doc)` | The elements inside `<channel>`, before the items. Return `null` entries to skip them. |
| `getItemNodeChildren($doc, $item)` | The elements inside each `<item>`. |
| `createElement($doc, $key, $value, $attributes)` | Creates an element, wrapping the value in CDATA when needed. |

## Overriding the service and factories

The module's service and factories can be replaced by app classes. If the class exists, the module builds it instead of its own:

| Name | App class |
|---|---|
| `Rss` service | `App\Rss\Service\Rss` |
| `FeedCollection` factory | `App\Rss\Factory\FeedCollection` |
| `Item` factory | `App\Rss\Factory\Item` |
| `ItemCollection` factory | `App\Rss\Factory\ItemCollection` |

Each app class should extend the module class it replaces.
