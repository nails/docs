---
description: >-
  Units represent a single item being transferred. A Pipeline transfers multiple
  units.
---

# Units

Units represent a single migration as it proceeds from a source to a target within a Pipeline. As well as being the object which transports the data a Unit object  also  exposes information about the object throughout its lifescycle as well as provides the ability to look ahead to determine if the unit should be migrated.

Units implement the `\HelloPablo\DataMigration\Interfaces\Unit` trait.

## Bundled Unit

The bundled unit provides basic functionality and serves as a "dumb" transport mechaniusm, i.e it does not perform any checks as to whether the source item has, or should, be migrated.

```
\HelloPablo\DataMigration\Unit
```

## Looking Ahead

If needed, you can craft your own Unit object. This is useful, for example, if you need to perform a check to ensure that you do not migrate duplicate items.

The following methods are useful when crafting your own Unit classes:

#### `shouldMigrate()`

Tests whether the source unit should be migrated. If it should not be migrated then `SkipException` should be thrown. You may, for example, wish to exclude items based on the value of a property.

#### `isMigrated(Interfaces\Pipeline $oPipeline)`

Determines if an item has already been migrated; returns the migrated item's ID if so, null if not. It is passed the Pipeline which is currently being executed so you have context as to what source/target needs checked,
