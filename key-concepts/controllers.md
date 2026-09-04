# Controllers

Controllers and [views](views.md) are the basic building blocks of Nails applications. Controllers are responsible for handling business logic, while the views are responsible for handling presentation.

Controllers and views are bundled in the application as [modules](components/modules.md).

## The Base Controller

Controllers should always extend `App\Controller\Base` class, which in turn should extend `Nails\Common\Controller\Base`. In addition to bootstrapping much of Nails' functionality `Base` also provides an opportunity for the developer to perform global actions on every page request (e.g. loading a menu, loading assets, or checking if the active user has new messages).

## A Sample Controller

The following example is a very basic controller with a method which loads a view using the [View service](../core-services/view.md), top and tailed with the site's global header and footer.&#x20;

### Folder Structure

```
application/
    ↳ modules/
        ↳ books/
            ↳ controllers/
                ↳ books.php
            ↳ views/
                ↳ index.php
```

### Controller

```php
<?php

use App\Controller\Base;
use Nails\Common\Service\View;
use Nails\Factory;

class Books extends Base
{
    public function index()
    {
        /** @var View $oView */
        $oView = Factory::service('View');
        $oView
            ->setData([
                'aBooks' = [
                    (object) [
                        'label'   => 'Treasure Island',
                        'author'  => 'Robert Louis Stevenson',
                        'summary' => 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.'
                    ],
                    (object) [
                        'label'   => '1984',
                        'author'  => 'George Orwell',
                        'summary' => 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.'
                    ],
                ],
            ])
            ->load([
                'structure/header',
                'books/index',
                'structure/footer',
            ]);
    }
}
```

### View

```php
<h1>
    Some great novels
</h1>
<?php

foreach ($aBooks as $oBook) {
    ?>
    <h2><?=$oBook->label?></h2>
    <h3>by <?=$oBook->author?></h3>
    <p><?=$oBook->summary?></p>
    <?php
}

```
