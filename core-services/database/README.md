---
description: The Database service is responsible for interfacing with the MySQL database.
---

# Database

The database is primarily exposed via [Models](https://github.com/nails/site-main/blob/develop/docs/intro/factory/models.md), but can also be queried using the `Database` service. The service is loaded using the [Factory](../../key-concepts/factory/):

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');
```

## Configuring

The database connection can be configured using the following constants:

| Constant             | Default     |
| -------------------- | ----------- |
| `DEPLOY_DB_HOST`     | `127.0.0.1` |
| `DEPLOY_DB_USERNAME` | `null`      |
| `DEPLOY_DB_PASSWORD` | `null`      |
| `DEPLOY_DB_DATABASE` | `null`      |
| `DEPLOY_DB_PORT`     | `3306`      |
| `APP_DB_PREFIX`      | `app_`      |
| `NAILS_DB_PREFIX`    | `nails_`    |

{% hint style="info" %}
The configs `APP_DB_PREFIX` and `NAILS_DB_PREFIX` allow you to keep clean separation between your app's tables and the tables required by any installed components. It also allows you to share a database with other Nails installations - if that's your thing.
{% endhint %}

## Querying

Examples are usually easiest to follow, notice how the method names correspond to the part of the query:

### Selecting Data

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

//  Select a particular sub set of columns
$oDb->select(['id', 'column_1', 'column_2']);

//  Apply conditionals
$oDb->where('column_1', 'foo');
$oDb->where('column_2 !=', 'bar');
$oDb->where_in('id', [1, 2, 3]);
$oDb->like('column_3', 'baz'); // == LIKE '%baz%'

$oDb->limit(10, 10);    //  Number per page, offset
$oDb->order_by('column_2', 'desc'); // Column, order

//  Execute the query
$oQuery = $oDb->get('table_name');

//  Returns the result set as an associative array
$aResult = $oQuery->result();
```

### Inserting Data

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

//  Set the value of a particular column
$oDb->set('column_name', 'foo');

//  Can also be set as an associative array
$oDb->set([
    'column_name' => 'foo'
]);

$oDb->insert('table_name');
$iId = $oDb->insert_id();
```

### Updating Data

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

//  Set the value of a particular column
$oDb->set('column_name', 'foo');

//  Can also be set as an associative array
$oDb->set([
    'column_name' => 'foo'
]);

// Set the ID of the record to update
$oDb->where('id', 123);

$oDb->update('table_name');
```

### Deleting Data

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

// Set the ID of the record to delete
$oDb->where('id', 123);

$oDb->delete('table_name');
```

### Manual Queries

```php
use Nails\Common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

$aResults = $oDb->query('

    SELECT * FROM table;

')->result();
```
