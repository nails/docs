---
description: >-
  This module provides a framework for importing data into your application from
  external sources, like an old database or CSV.
---

# Data Migration

This module is a Nails orientated wrapper around [HelloPablo's Data Migration framework](https://github.com/hellopablo/data-migration). It exposes console commands for running migrations as well as imposes some structure within Nails applications. Install using Composer:

```
composer require nails/module-data-migration
```

## Key Concepts

There are five key concepts in the data migration module:

### Connectors

Connecters are the classes which "talk" to the data sources. Their purpose is to read and write from a data source (e.g. a CSV file, or a MySQL table).

{% content-ref url="connectors.md" %}
[connectors.md](connectors.md)
{% endcontent-ref %}

### Pipelines

Pipelines represent a single migration, from a source [Connector](./#connectors), to a target [Connector](./#connectors). Pipelines define the source and destination [Connectors](./#connectors), as well as the [Recipe](./#recipes) to use when transferring [Units](./#units) of data. They also allow the developer to define a priority, as well as offering hooks at various points throughout the lifecycle of a migration.

{% content-ref url="pipelines.md" %}
[pipelines.md](pipelines.md)
{% endcontent-ref %}

### Recipes

Recipes deifne how a [Unit](./#units) of data is transformed once it leaves the source [Connector](./#connectors), and before it is sent to the target [Connector](./#connectors).

{% content-ref url="recipes.md" %}
[recipes.md](recipes.md)
{% endcontent-ref %}

### Transformers

Transformers mutate a single piece of data within a [Recipe](recipes.md).

{% content-ref url="transformers.md" %}
[transformers.md](transformers.md)
{% endcontent-ref %}

### Units

Units represent a single item being transferred. A [Pipeline](./#pipelines) transfers multiple units.

{% content-ref url="units.md" %}
[units.md](units.md)
{% endcontent-ref %}

## Running Migrations

Run Pipelines using the following console command:

```
nails datamigration:run 
```

The console accepts no arguments and the following options:

```
--dry-run              Whether to perform a dry-run or not
-f, --filter=FILTER    Filter pipelines (only include matches) (multiple values allowed)
-e, --exclude=EXCLUDE  Exclude matches (multiple values allowed)
-d, --debug            Run in debug mode
-m, --memory=MEMORY    Set memory limit (in MB)
-s, --stop-on-error    Stop on first error, rather than summarrise
```

## Example Migration

Sometimes it's easier to learn by example.

{% content-ref url="simple-example.md" %}
[simple-example.md](simple-example.md)
{% endcontent-ref %}



