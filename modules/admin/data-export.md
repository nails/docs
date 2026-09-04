# Data Export

Exporting Data (or "reports" as they are often known) is simple using Admin's Data Export system. Simply navigate to `Utilities › Export Data`, select the source you'd like and the format you'd like it in.

## Sources

Sources are classes which return data from the database and return it in a consistent way which is understood by the [format](data-export.md#formats) classes. Any component in a Nails application can provide a source.

Sources should have the namespace `App\DataExport\Source` and implement the `Nails\Admin\Interfaces\DataExport\Source` interface.

{% hint style="info" %}
Use the console to quickly create Data Export Sources:

`nails make:admin:dataexport`
{% endhint %}

The example below shows a simple export which exports items from the `Book` model.

```php
namespace App\DataExport\Source;

use App\Model;
use App\Resource;
use Nails\Admin\DataExport\SourceResponse;
use Nails\Admin\Interfaces\DataExport\Source;
use Nails\Factory;

class Books implements Source
{
    public function getLabel(): string
    {
        return 'Books';
    }

    public function getFileName(): string
    {
        return 'books';
    }

    public function getDescription(): string
    {
        return 'Exports all books';
    }
    
    public function getOptions(): array
    {
        return [];
    }
    
    public function isEnabled(): bool
    {
        return true;
    }

    public function execute($aOptions = [])
    {
        /** @var Model\Book $oBook */
        $oModel = Factory::model('Book', 'app');
        $aBooks = $oModel->getAll();
        
        $aData = [];
        
        /** @var Resource\Book $oBook */
        foreach ($aBooks as $oBook) {
            $aData[] = [
                $oBook->id,
                $oBook->label,
                $oBook->author,
                $oBook->date_published,
            ];
        }
        
        /** @var SourceResponse $oResponse */
        $oResponse = Factory::factory('DataExportSourceResponse', 'nails/module-admin');
        $oResponse
            ->setLabel($this->getLabel())
            ->setFileName($this->getFilename())
            ->setFields([
                'ID',
                'Title',
                'Author',
                'Published',
            ])
            ->setData($aData);

        return $oResponse;
    }
}
```

### Handling large data sets

The example above generates a new array and passes it into the `SourceResponse`'s `setData()` method. This is fine, but can eat up a lot of memory if the data set is large.

Also available on the `SourceResponse` object is the `setSource()` method. This method accepts a raw database query response object (as returned by the [Database service's](../../core-services/database/) `query()` method, or a [model's](../../key-concepts/factory/models/) `getAllRawQuery()` method) and efficiently compiles the export.

```php
public function execute($aOptions = [])
{
    /** @var Model\Book $oBook */
    $oModel  = Factory::model('Book', 'app');
    $oResult = $oModel->getAllRawQuery([
        'select' => [
            'id',
            'label',
            'author',
            'date_published'
        ],
    ]);
    
    /** @var SourceResponse $oResponse */
    $oResponse = Factory::factory('DataExportSourceResponse', 'nails/module-admin');
    $oResponse
        ->setLabel($this->getLabel())
        ->setFileName($this->getFilename())
        ->setFields([
            'ID',
            'Title',
            'Author',
            'Published',
        ])
        ->setSource($oResult);

    return $oResponse;
}
```

### Combining multiple files in a single Source

Sometimes it is necessary to bundle multiple files together in a single source, the use case is usually a one-to-many relationship (e.g. `book` and `book_review`). This is easily accomplished by returning an array of `SourceResponse` objects in the `execute()` method. Each of these items will be fed individually to the chosen formatter and then all zipped together into a single archive.

```php
public function execute($aOptions = [])
{
    /** @var Model\Book $oBookModel */
    $oBookModel = Factory::model('Book', 'app');
    /** @var Model\Book\Review $oReviewModel */
    $oReviewModel = Factory::model('BookReview', 'app');
    
    $oResultBooks = $oBookModel->getAllRawQuery([
        'select' => [
            'id',
            'label',
            'author',
            'date_published'
        ],
    ]);
    
    $oResultReviews = $oReviewModel->getAllRawQuery([
        'select' => [
            'id',
            'book_id',
            'reviewer',
            'review',
            'date_published'
        ],
    ]);

    return [
    
        Factory::factory('DataExportSourceResponse', 'nails/module-admin')
            ->setLabel('Books')
            ->setFileName('books')
            ->setFields([
                'ID',
                'Title',
                'Author',
                'Published',
            ])
            ->setSource($oResultBooks),
        
        Factory::factory('DataExportSourceResponse', 'nails/module-admin')
            ->setLabel('Reviews')
            ->setFileName('reviews')
            ->setFields([
                'ID',
                'Book ID',
                'Reviewer',
                'Review',
                'Published',
            ])
            ->setSource($oResultReviews),
    ];
}
```

The above will result in a zip file being created containing two files: `book` and `book_review`.

### Options

@todo - complete this

## Formats

Formatters take the data returned by a [source](data-export.md#sources) and compile it down to a string which is then saved to a file and sent to the user.

By default, Nails provides CSV and a JSON formatters.

If you need to create a new formatter, then you should do so in the `App\DataExport\Format` namespace, and it should implement the `Nails\Admin\Interfaces\DataExport\Format` interface.

## Exporting Data Programatically

To export data programatically (e.g. in a cron job) you may use the `DataExport` service:

```php
use Nails\Admin\Service\DataExport;
use Nails\Factory;

/** @var DataExport $oDataExport */
$oDataExport = Factory::service('DataExport', 'nails/module-admin');
```

This model provides you with the following methods:

| Method                                                                                                                                                                                    | Description                                                                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `getAllSources(): array`                                                                                                                                                                  | Returns an array of all available Sources.                                                                                                             |
| `getSourceBySlug(string $sSlug): ?Source`                                                                                                                                                 | Returns a single Source object.                                                                                                                        |
| `getAllFormats(): array`                                                                                                                                                                  | Returns an array of all available Formats.                                                                                                             |
| `getFormatBySlug(string $sSlug): ?Format`                                                                                                                                                 | Returns a single Format object.                                                                                                                        |
| <p><code>export(</code></p><p>    <code>string $sSourceSlug,</code></p><p>    <code>string $sFormatSlug,</code></p><p>    <code>array $aOptions = []</code></p><p><code>): int</code></p> | Executes a DateExport source then passes to a DataExport format. Once complete the resulting file is uploaded to the CDN and the object's ID returned. |
