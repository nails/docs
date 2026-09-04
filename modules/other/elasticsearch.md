---
description: >-
  The Elasticsearch module provides a Nails orientated interface for querying an
  Elasticsearch cluster.
---

# Elasticsearch

Elasticsearch is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. It is great for storing and searching data and querying it _fast_.

The Nails Elasticsearch module provides a Nails-orientated client for querying an Elasticsearch cluster, as well as some utilities to make managing indexes, syncing data, searching, and warming easy.

## Indexes

Indexes are a central concept in Elasticsearch. Indexes are where your documents are stored. If an Elasticsearch cluster is considered a database, then indexes can (very) roughly be considered tables.

This module provides a convenient way of defining indexes through code. Classes in the `App\Elasticsearch\Index` namespace which implement the `Nails\Elasticsearch\Interfaces\Index` interface are Index definitions.

The following example is a definition for a `book` index, with no special settings or mappers.

{% hint style="info" %}
See the Elasticsearch documentation for more information on [indexes](https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html#glossary-index), [mappings](https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html#glossary-mapping), and [settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-modules-settings).
{% endhint %}

```php
namespace App\Elasticsearch\Index;

use Nails\Elasticsearch;
use stdClass;

class Book implements Elasticsearch\Interfaces\Index
{
    /**
     * Defines the name of the index in the cluster - must be unique
     */
    public static function getIndex(): string
    {
        return 'book';
    }

    /**
     * Defines any index settings
     */
    public function getSettings(): stdClass
    {
        return (object) [];
    }

    /**
     * Defines any index mappings
     */
    public function getMappings(): stdClass
    {
        return (object) [];
    }
    
    /**
     * Called when warming the index
     */
    public function warm(Client $oClient, OutputInterface $oOutput)
    {
    }
}
```

Indexes can be created, reset, and warmed using the [Command Line Tool](../../command-line-tool.md)

#### Creating & Resetting indexes

Once the definitions are all in place, you can create the indexes in your Elasticsearch cluster using the following command:

```bash
nails elasticsearch:reset
```

This will delete the index if it exists, then recreate it using the defined mappings and settings.

{% hint style="danger" %}
💥**This is highly destructive!**

If any data exists in an index when it is reset, that data will be deleted.
{% endhint %}

#### Warming

Warming is the act of populating, or refreshing, the contents of an index. This might be something you do once, on deployment, or regularly. It simply calls the index's `warm()` method which is responsible for populating the index with data (see also, [syncing an index with a model](elasticsearch.md#syncing)).

```bash
nails elasticsearch:warm
```

[See below for more information on index warming.](elasticsearch.md#warming)

## The Client

The client is the primary service offered by this module, and is the main means of interacting with the Elasticsearch cluster.

The Elasticsearch client can be loaded via the [Factory](../../key-concepts/factory/):

```php
use Nails\Elasticsearch;
use Nails\Factory;

/** @var Elasticsearch\Client $oClient */
$oClient = Factory::service('Client', Elasticsearch\Constants::MODULE_SLUG);
```

The Nails client abstracts the official [Elasticsearch PHP client](https://github.com/elastic/elasticsearch-php), providing some additional utility methods.

### Methods

The following methods are available:

#### `isAvailable($iTimeout = null): bool`

This method allows you to test if the cluster is available. The `$iTimeout` parameter is the duration (in seconds) before a timeout occurs.

#### `index(Index $oIndex, $mId, $mDocument): self`

This method allows you to index a document. A document is defined as any object which you wish to store in the specified index. `$mId` is  the document's unique ID&#x20;

This method both creates new documents, or updates existing documents.

#### `delete(Index $oIndex, $mId): self`

Deletes a document with ID `$mID` from an index.

#### `search($mQuery, $mIndexes = null): Search`

Performs a search across the supplied [indexes](elasticsearch.md#indexes). See [searching](elasticsearch.md#searching) for further details.

#### `discoverIndexes(): array`

Returns an array of all discovered [indexes](elasticsearch.md#indexes).

#### `getClient(): \Elasticsearch\Client`

Returns the instance of the underlying low-level [official Elasticsearch PHP client](https://github.com/elastic/elasticsearch-php).

#### `destroy(OutputInterface $oOutput = null): self`

Destroys all data in the cluster, optionally logging to an `OutputInterface`.

{% hint style="danger" %}
💥**This is very destructive!**

Use with caution. This will destroy all indexes in the cluster, even indexes which are not managed by the application.
{% endhint %}

#### `reset(OutputInterface $oOutput = null): self`

Deletes all indexes managed by the application (i.e returned by `discoverIndexes()`) and recreates them.

{% hint style="danger" %}
💥**This is very destructive!**

This method will delete all data contained within managed indexes.
{% endhint %}

#### `warm(OutputInterface $oOutput = null): self`

Executes the `warm()` method of each index. Used to populate an index with data. Typically this is managed via [models which sync their data](elasticsearch.md#syncing).

### Configuration

There are two [configurations](../../getting-started/configuration.md) you can set to configure the cluster:

| Config                  | Description                                              | Default              |
| ----------------------- | -------------------------------------------------------- | -------------------- |
| `ELASTICSEARCH_HOSTS`   | The hosts array the client should connect to.            | `["127.0.0.1:9200"]` |
| `ELASTICSEARCH_TIMEOUT` | How long the timeout for `isAvailable()` is, in seconds. | `2`                  |

## Syncing Models

Often, the data we want in an index is the contents of a model. This module provides a utility trait which can be used by a model and will reliably sync changes (creates, updates, and deletes) to an [Index](elasticsearch.md#indexes).

```php
Nails\Elasticsearch\Traits\Model\SyncWithElasticsearch
```

Models which use this trait must define the `syncWithIndex(): Index` method specifies which [Index](elasticsearch.md#indexes) it will sync with.&#x20;

Additionally, models can define a `syncToElasticsearchData(): array` method to manipulate the control array used when querying the model's `getById()` method when indexing an item.

In the following example, we are syncing the `Book` model to the `Book` index, and expanding the book's author when we index the `Book` resource.

```php
namespace App\Model\Book;

use Nails\Common\Model\Base;
use Nails\Common\Helper\Model\Expand;
use Nails\Elasticsearch;

class Book extends Base
{
    use Elasticsearch\Traits\Model\SyncWithElasticsearch;
    
    const TABLE = 'book';
    
    protected function syncWithIndex(): Elasticsearch\Interfaces\Index
    {
        return new App\Elasticsearch\Index\Book();
    }
    
    protected function syncToElasticsearchData(): array
    {
        return [new Expand('author')];
    }
}
```

## Searching

@todo - write up the Search helper class and it's child utilities.

## Warming Indexes

Warming an index means populating it with new data, or refreshing existing data. A warm might happen after a [reset](elasticsearch.md#creating-and-resetting-indexes), or perhaps if data has become corrupted somehow.

Indexes are responsible for warming themselves via the Indexe's`warm()` method. However, in a similar vein to model's syncing themselves with an index, indexes can bind to a specific model to be warmed.

The `Nails\Elasticsearch\Traits\Model\Warm` trait can be used by an Index to signify that it should be warmed by the data represented by a specific model. It does this by returning an instance of the model in a `getModel(): Base` method:

```php
namespace App\Elasticsearch\Index;

use Nails\Common\Model\Base
use Nails\Elasticsearch;
use stdClass;

class Book implements Elasticsearch\Interfaces\Index
{
    use Elasticsearch\Traits\Model\Warm;
    
    /**
     * REturns the model to warm with
     */
    public function getModel(): Base
    {
        return Factory::model('Book', 'app');
    }
    
    /**
     * Defines the name of the index in the cluster - must be unique
     */
    public static function getIndex(): string
    {
        return 'book';
    }

    /**
     * Defines any index settings
     */
    public function getSettings(): stdClass
    {
        return (object) [];
    }

    /**
     * Defines any index mappings
     */
    public function getMappings(): stdClass
    {
        return (object) [];
    }
}
```

{% hint style="success" %}
Note that when using the `Nails\Elasticsearch\Traits\Model\Warm` trait the `warm()` method can be omitted.
{% endhint %}

## Query Helpers

@todo - write up query helpers once they are implemented.

