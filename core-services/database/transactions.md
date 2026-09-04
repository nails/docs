---
description: >-
  Database transactions allow you to safely run groups of queries and commit or
  rollback if something goes wrong.
---

# Transactions

Database transactions allow you to run a query (or multiple queries) and roll back changes if something goes wrong. They are particular useful when working with queries which touch multiple tables and cancelling the group if one of the operations fails.

Transactions in Nails are manual, i.e you must actively commit or roll back a transaction once you reach an outcome. Transactions not committed at the end of the request are rolled back automatically.

Transactions are initiated, committed, or rolled back using the [Database service](./).

```php
use Nails\common\Service\Database;
use Nails\Factory;

/** @var Database $oDb */
$oDb = Factory::service('Database');

try {

    $oDb->trans_begin();
    
    
    // Execute queries
    
    
    $oDb->trans_commit();

} catch (\Exception $e) {
    $oDb->trans_rollback();
}
```

{% hint style="warning" %}
Transactions can be nested, and must be committed or rolled back individually.
{% endhint %}
