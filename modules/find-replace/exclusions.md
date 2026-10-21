---
description: >-
  Hide columns from search, or show them and refuse to replace them, by
  dropping a class under src/FindReplace/Exclusion.
---

# Exclusions

An exclusion decides which columns Find & Replace may see. Hidden columns are left out of search, replace, revert, and `find-replace:fields`. Read-only columns are searched and shown, and a replacement or revert skips them with the reason `read-only`.

You do not register the class. Any concrete class under `src/FindReplace/Exclusion` that implements `Nails\FindReplace\Interfaces\Exclusion` and has no required constructor arguments is picked up. That applies to the app (`App\FindReplace\Exclusion\…`) and to every installed module.

Hidden wins when a column matches a hidden entry and a read-only entry.

## A class

```php
namespace App\FindReplace\Exclusion;

use Nails\FindReplace\Interfaces\Exclusion;

class Content implements Exclusion
{
    public function hidden(): array
    {
        return [
            'app_session',
            'app_user.password',
            '/\.hash$/',
        ];
    }

    public function readOnly(): array
    {
        return [
            'app_page.slug',
            'app_log.*',
        ];
    }
}
```

`hidden()` and `readOnly()` each return a list of strings. An entry that is not a string, or a regex that is not valid PCRE, is an error (`Invalid find-replace exclusion regex` for a bad pattern). The error is raised when the registry is built, and again when someone types the same pattern into a search.

## Entries

Each entry is one of:

| Entry | Covers |
|---|---|
| `app_article` | Every column on that table |
| `app_article.*` | Every column on that table |
| `app_article.title` | That column |
| `/_lang$/` | Any `table.column` the pattern matches |

A regex starts with `/`, includes its closing delimiter, and may include flags (`/pattern/i`). It is tested against `table.column`, for example `app_article.title`.

The same syntax is used for a single search, which does not change these exclusions. On the command line, pass `--table` and `--exclude-table`. In admin, put one entry per line under **Limit to tables** and **Exclude tables**. An empty include list means every column the exclusions still allow. Exclude entries are applied after include entries, so a column that matches both is left out.

## Built-in exclusion

The module always applies its own exclusion, on top of any class you add:

- Its own search, result, facet, and change tables are hidden.
- A column whose name ends in `.token` or `.ref` is hidden (`/\.(token|ref)$/`).
- A column whose name ends in `.slug` is read-only (`/\.slug$/`).

A table that does not have exactly one primary-key column is read-only as well, with the reason `no primary key`. That rule is applied by the schema scan, separate from exclusion classes.

## Shipped with Nails

Each Nails module that owns searchable columns ships a `Package` class under `src/FindReplace/Exclusion`. A module that declares an exclusion lists `nails/module-find-replace` in `require-dev` so tests and phpstan load the interface. Those classes follow the same rules an app should use:

- Hide secrets, sessions, caches, logs, archives, and operational tables (queues, migrations, ephemeral previews, dashboard widgets, email blockers). A search leaves those columns out.
- Hide or lock functional columns such as hashes, drivers, mime types, and JSON that the module's own screens must keep valid.
- Mark contact data (email, telephone, IP, VAT number) and similar identifiers read-only. A search shows the hit; replacement is left to the screen that owns the row.
- Leave user-authored content searchable and replaceable: names, labels, bodies, quotes, CMS copy, form answers, comments, chat messages.

Module classes rely on the built-in `.slug`, `.token`, and `.ref` rules. Columns that only look similar (`draft_slug`, `token_stats`, `claim_token`) are listed explicitly. App-owned columns, including those on `user_meta_app`, stay the app's exclusion.
