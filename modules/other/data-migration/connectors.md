---
description: >-
  Connecters are the classes which "talk" to the data sources. Their purpose is
  to read and write from a data source (e.g. a CSV file, or a MySQL table).
---

# Connectors

Connectors allow the data migtration to "talk" to a data source or data target. They provide a layer for reading, writing, and counting records in a data store. Typically this will be MySQL to MySQL, but faciliatating CSV to MySQL or ElasticSearch to MySQL could be achieved by writing appropriate Connector classes.

Connectors implement the `\HelloPablo\DataMigration\Interfaces\Connector` trait.

## Bundled Connectors

### MySQL

Connects to a MySQL Table:

```php
new \HelloPablo\DataMigration\Connector\MySQL(
    // The class to use for units of work
    $oUnit,
    
    // The connection's host
    $sHost,
    
    // The connection's username
    $sUsername,
    
    // The connection's password
    $sPassword,
    
    // The connection's port
    $iPort,
    
    // The connection's database
    $sDatabase,
    
    // The connection's table
    $sTable
);
```

{% hint style="success" %}
If you need to alter the behaviour of this connector, for example, join tables or selectively include records then simply extend this class and overload the appropriate methods and constant values.
{% endhint %}
