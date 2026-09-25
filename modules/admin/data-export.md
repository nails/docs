# Data Export

Admin's Data Export system produces downloadable files, often called reports, from your data. An admin user goes to _Utilities › Export Data_, chooses a **source** (what to export) and a **format** (CSV, JSON and so on), and submits the form.

Exports don't run during the request. The request is queued, and the `admin:dataexport:process` command runs it: `nails/module-cron` schedules that command every minute. When the file is ready it's uploaded to the CDN and the user gets an email with a download link. The _Export Data_ page lists recent exports and their status.

Using the _Export Data_ screen needs the `Nails\Admin\Admin\Permission\Utilities\DataExport\Generate` [permission](user-permissions.md).

## Sources

Sources are classes which return data from the database and return it in a consistent way which is understood by the [format](data-export.md#formats) classes. Any component in a Nails application can provide a source.

Admin discovers sources in each component's `Admin\DataExport\Source` namespace. For your app, that's `App\Admin\DataExport\Source` in `src/Admin/DataExport/Source/`. Sources implement `Nails\Admin\Interfaces\DataExport\Source`.

{% hint style="info" %}
Use the console to quickly create Data Export Sources:

`nails make:admin:dataexport`
{% endhint %}

The example below shows a simple export which exports items from the `Book` model.

```php
<?php
// src/Admin/DataExport/Source/Books.php

namespace App\Admin\DataExport\Source;

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

    public function getDescriptionExtended(): string
    {
        return 'One row per book, including unpublished books.';
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

The above will result in a zip file being created containing two files: `books` and `reviews`.

### Options

`getOptions()` returns form fields, as arrays in the same shape the [form field helpers](../../key-concepts/form-fields.md) accept. They're shown when the user picks the source, and the submitted values are passed to `execute()` as `$aOptions`, keyed by each field's `key`:

```php
public function getOptions(): array
{
    return [
        [
            'key'     => 'status',
            'label'   => 'Status',
            'type'    => 'dropdown',
            'options' => [
                ''          => 'All',
                'PUBLISHED' => 'Published',
                'DRAFT'     => 'Draft',
            ],
        ],
    ];
}

public function execute($aOptions = [])
{
    $aData = [];
    if (!empty($aOptions['status'])) {
        $aData['where'][] = ['status', $aOptions['status']];
    }

    // ...
}
```

### Enabling and disabling

Return `false` from `isEnabled()` to hide a source, for example when a feature is turned off or the user lacks a [permission](user-permissions.md).

### Slugs

Every source and format has a slug made from the component slug and the class name relative to the `Source` or `Format` namespace, for example `app::Books` or `nails/module-admin::Csv`. Use these slugs with the console commands and the `DataExport` service.

## Formats

Formatters take the data returned by a [source](data-export.md#sources) and compile it down to a string which is then saved to a file and sent to the user.

By default, Nails provides CSV and a JSON formatters.

To add a format, create a class in `App\Admin\DataExport\Format` (`src/Admin/DataExport/Format/`) that implements `Nails\Admin\Interfaces\DataExport\Format`. It needs:

| Method                                  | Purpose                                                                           |
| --------------------------------------- | --------------------------------------------------------------------------------- |
| `getLabel()`                            | The name shown to the user.                                                       |
| `getDescription()`                      | A short description.                                                              |
| `getFileExtension()`                    | The extension for generated files, for example `csv`.                             |
| `execute($oSourceResponse, $rFile)`     | Write the `SourceResponse`'s data to the open file handle `$rFile`.               |

`Nails\Admin\Admin\DataExport\Format\Csv` and `Json` in the Admin module are worked examples.

## Scheduled exports

To send a report to people on a schedule, add a class in `App\Admin\DataExport\Schedule` (`src/Admin/DataExport/Schedule/`) that implements `Nails\Admin\Interfaces\DataExport\Schedule`:

```php
<?php
// src/Admin/DataExport/Schedule/WeeklyBooks.php

namespace App\Admin\DataExport\Schedule;

use Nails\Admin\Interfaces\DataExport\Schedule;
use Nails\Auth\Constants as AuthConstants;
use Nails\Factory;

class WeeklyBooks implements Schedule
{
    public function getCronExpression(): string
    {
        return '0 7 * * 1';     // 07:00 every Monday
    }

    public function getSource(): string
    {
        return 'app::Books';
    }

    public function getFormat(): string
    {
        return 'nails/module-admin::Csv';
    }

    public function getOptions(): array
    {
        return ['status' => 'PUBLISHED'];
    }

    public function getUsers(): array
    {
        return Factory::model('User', AuthConstants::MODULE_SLUG)->getByIds([1, 2]);
    }

    public function getTTL(): int
    {
        return 604800;          // Keep the file for a week
    }
}
```

Each time `admin:dataexport:process` runs, it queues one export per user for any schedule that is due. The export is then processed like any other. `getTTL()` is how long, in seconds, the export is kept before [housekeeping](housekeeping.md#data-export) removes it.

## Console commands

| Command                     | Purpose                                                                     |
| --------------------------- | --------------------------------------------------------------------------- |
| `make:admin:dataexport`     | Create a new source in `src/Admin/DataExport/Source/`.                      |
| `admin:dataexport:list`     | List the available sources and formats, with their slugs.                   |
| `admin:dataexport:run`      | Run a source immediately.                                                   |
| `admin:dataexport:process`  | Queue due scheduled exports, then process pending ones. Runs every minute from cron. |

## Exporting Data Programatically

To export data programatically (e.g. in a cron job) you may use the `DataExport` service:

```php
use Nails\Admin\Service\DataExport;
use Nails\Factory;

/** @var DataExport $oDataExport */
$oDataExport = Factory::service('DataExport', 'nails/module-admin');
```

The service provides the following methods:

| Method                                                                                                                                                                                    | Description                                                                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `getAllSources(): array`                                                                                                                                                                  | Returns an array of all available Sources.                                                                                                             |
| `getSourceBySlug(string $sSlug): ?Source`                                                                                                                                                 | Returns a single Source object.                                                                                                                        |
| `getAllFormats(): array`                                                                                                                                                                  | Returns an array of all available Formats.                                                                                                             |
| `getFormatBySlug(string $sSlug): ?Format`                                                                                                                                                 | Returns a single Format object.                                                                                                                        |
| <p><code>export(</code></p><p>    <code>string $sSourceSlug,</code></p><p>    <code>string $sFormatSlug,</code></p><p>    <code>array $aOptions = []</code></p><p><code>): int</code></p> | Executes a DataExport source then passes to a DataExport format. Once complete the resulting file is uploaded to the CDN and the object's ID returned. |

## Configuration

| Constant                      | Default | Purpose                                                          |
| ----------------------------- | ------- | ---------------------------------------------------------------- |
| `ADMIN_DATA_EXPORT_RETENTION` | `3600`  | How long, in seconds, an export requested through admin is kept. |
| `ADMIN_DATA_EXPORT_URL_TTL`   | `300`   | How long, in seconds, a signed download link stays valid.        |

## Cleanup

Completed exports expire and are removed by [housekeeping](housekeeping.md#data-export). Deleting an export row through the model also destroys its CDN download, but only when no other export still points at that file (identical requests share one object). CDN failures are logged; they do not resurrect the row.
