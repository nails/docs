---
description: >-
  Utilise database migrations to alter your app's database schema and content
  programatically.
---

# Migrations

As your app's functionality and requirements change over time so might the database structure or content. Migrations are a system where you can programatically define changes to your database and execute it automatically as part of deployment or build - making changes for production or your team seamless.

In Nails, the database is shared between your app and any installed components. As a result migrations will always be run in the following order:

1. `nails/common`
2. Installed [components](../../key-concepts/components/)
3. The application

## Writing migrations

Nails will look for classes in `App\Database\Migration` which implement the`Nails\Common\Interfaces\Database\Migration` interface and sort them by the return value of the class' `getPriority()` method.

{% hint style="info" %}
Quickly create sequential migrations using the command line tool:

```
nails make:db:migration
```

This will create the next migration in the sequence for you.
{% endhint %}

### Migration Trait

Nails provides a trait to make writing migrations easier:

```
Nails\Common\Traits\Database
```

This trait will handle the majority of the boilerplate code which is required by the interface, leaving you to focus on the actual migration.

The following is a sample migration which uses the trait; it creates a table called `app_my_table`:

```php
namespace App\Database\Migration;

use Nails\Common\Interfaces;
use Nails\Common\traits

class Migration0 extends Interfaces\Database\Migration
{
    use Traits\Database\Migration;
    
    public function execute()
    {
        $this->query('
            CREATE TABLE `{{APP_DB_PREFIX}}my_table` (
                `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
                `key` varchar(50) DEFAULT NULL,
                `value` text,
                PRIMARY KEY (`id`),
            ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
        ');
    }
}
```

#### **Methods**

The trait provides four helper functions for interacting with the database, these are essentially wrappers for a normal `\PDO` connection.

* `$this->query()` - for running plaintext SQL
* `$this->prepare()` - for preparing a `\PDOStatement`
* `$this->lastInsertId()` - returns the ID of the last successful write operation
* `$this->db()` - returns the raw instance of the `\PDO` class, should you need it

It also provides helpers for inspecting the schema, so that a migration can determine whether its changes are already in place:

* `$this->tableExists()`
* `$this->getTableColumns()` - returns the table's columns, keyed by column name
* `$this->columnExists()`
* `$this->indexExists()`
* `$this->getTableForeignKeys()` - returns the table's foreign keys, grouped by the constrained column
* `$this->foreignKeyExists()`

## Repeatable migrations

A migration which implements `Nails\Common\Interfaces\Database\Migration\Repeatable` is evaluated on **every** run of `db:migrate`, regardless of the version recorded for its component. In exchange it must work out for itself whether there is anything to do, and must be safe to execute any number of times.

```php
namespace App\Database\Migration;

use Nails\Common\Interfaces;
use Nails\Common\Traits;

class Migration7 implements Interfaces\Database\Migration\Repeatable
{
    use Traits\Database\Migration;

    public function execute()
    {
        if ($this->columnExists('{{APP_DB_PREFIX}}my_table', 'label')) {
            return;
        }

        $this->query('ALTER TABLE `{{APP_DB_PREFIX}}my_table` ADD `label` VARCHAR(150) NOT NULL DEFAULT "";');
    }
}
```

This exists because a migration's reachability would otherwise depend on its number. `db:migrate` records a single integer per component and runs only those migrations numbered above it, which is fine for a single line of development but breaks down where a component is maintained on more than one branch: the same change can carry a different number on each, so an app moving between branches will resume part-way through the *other* branch's sequence and silently skip whatever sits below that point. Marking a migration repeatable takes it out of that race entirely.

{% hint style="warning" %}
Repeatable migrations run on every migration of every app, forever. Keep the "is there work to do?" check cheap, and put it first.
{% endhint %}

### Maintaining a component on two branches

Where the same component is maintained on two long-lived branches, apply the following so that apps can move from one to the other:

* Any change which exists on only one branch should be repeatable, otherwise an app arriving from the other branch can skip it.
* Any change which exists on both branches must be numbered **at least as high** on the branch apps upgrade *to* as it is on the branch they upgrade *from*. Numbering it lower means an incoming app never runs it.
* Because a shared change can be numbered higher on the destination branch, an arriving app will re-run it. Shared migrations must therefore be guarded with the schema helpers above rather than issuing bare DDL.

{% hint style="warning" %}
Things to be aware of when using the trait:

**The name of the class is important**\
The priority of the migration will be inferred from the Class' name. It is expected that you will name classes in the format `/^.+\d+%/` - e.g. `Migration0`, `Migration1`, etc.

**You can substitute constants into queries**\
When calling `query()` or `prepare()` the string will be processed and constant names wrapped in `{{}}` will be substituted for the constant value, e.g. `{APP_DB_PREFIX}}`.
{% endhint %}

## Running migrations

To apply any new migrations to the app's database issue the following command:

```
nails db:migrate
```

This will apply, in order, any as yet unapplied migrations.
