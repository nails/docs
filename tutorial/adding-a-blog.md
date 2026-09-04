---
description: >-
  We'll put our new knowledge to practical use and build a basic blogging
  platform to learn about models and database migrations.
---

# Adding a simple blog

Next we are going to add some functionality to our website in the form of a simple blogging platform. This will demonstrate the basic functionality of [Models](../key-concepts/factory/models/) and [Resources](../key-concepts/factory/resources.md) as well as allow us to create some controllers to display our exciting news posts to the world.

In this section our aim is to create a blog index page at `/blog` as well as use dynamic routes to map `blog/{slug}` to a controller for rendering single news posts.&#x20;

## Create the controllers

First we need to design our routes and controllers; we want to define the URLs which will be used for listing our blog posts as well as the controller for rendering the single blog post.

We want to create the following URL structure:

```
# List the blog posts, in chronological order
/blog

# Render a single blog post based on a generated slug
/blog/{slug}
```

This will be achieved by creating controllers within a module named `blog`. The Nails Command Line Tool provides some generators to make this easier for you.

As we're using Docker, we'll need to enter the web server container to execute these next commands - do so using `make ssh`. you will be taken to the application's root directory, `/var/www/html` within the context of the web server container.

```bash
make ssh
```

{% hint style="info" %}
From here on out `nails` commands should be executed within the context of the web server container - we won't continue to remind you to do this!
{% endhint %}

Next we'll use the command line tool's `make:controller` utility.

```
nails make:controller
```

We need to pass the generator the name of the controller we wish to generate as the first argument, and any methods as the second argument - we need to pass it in the format:

```
{module}/{controller} {methods}
```

If `{methods}` is not passed it defaults to `index`, similarly, if `{controller}` is not passed it defaults to the value of `{module}`.

So, to create our blog index controller we need only execute:

```bash
nails make:controller blog
```

This will create a controller and some views in the `./www/application/modules/blog` directory. Now if you visit [https://localhost/blog](https://localhost/blog) in your browser you should see the placeholder page:

![](<../.gitbook/assets/Screenshot 2020-03-02 15.11.07.png>)

Great! This doesn't do very much yet but we'll come back to that. Next we want to add a controller to handle the `/blog/{slug}` route. Repeating the above, we'll issue:

```bash
nails make:controller blog/single
```

Navigating to [https://localhost/blog/single](https://localhost/blog/single) should bring up a similar placeholder page as before 🎉

## Configuring dynamic routes

At the moment we're relying on [automatic routes](../key-concepts/routing.md#automatic-routes) to resolve our controller. To make our `Single` controller respond to our desired route structure to `/blog/{slug}` we need to define some [explicit routes](../key-concepts/routing.md#explicit-routes).

If you haven't already, have a quick read of how Nails handles routing, then come back and continue.

{% content-ref url="../key-concepts/routing.md" %}
[routing.md](../key-concepts/routing.md)
{% endcontent-ref %}

To support our desired URL structure we are going to add the following explicit route to the app's `applocation/config/routes.php` file:

```php
$route['blog/(.+)'] = 'blog/single/index';
```

To test if this works, visit [https://localhost/blog/my-blog-post](https://localhost/blog/my-blog-post) in your browser, if you see the same placeholder page as before you know it's worked.

## Adding the model and database table

Now we have our controllers and routes in place, let's create our models and design the database table we'll use to save our blog posts in.

### Create the models

Creating models is similar to creating controllers: there's a Nails Command Line Utility for that:

Execute the following command to generate a blog post model in the App's `App\Model` namespace:

```
nails make:model Blog/Post
```

{% hint style="info" %}
Either forward or backslashes can be used to define the namespace segments, but if you use backslashes you must escape them using double backslashes.
{% endhint %}

Read what the tool says it will create, and confirm by hitting enter. You have now created a new `App\Model\Blog\Post` model, associated resource – both have been automatically added to your app's `./www/application/services.php` file.

Additionally a default `app_blog_post` table and seeder have also been created for you.

### Configure the model

Open the newly created model in your IDE, it lives here:

```
./www/src/Model/Blog/Post.php
```

Here you can see the `App\Model\Blog\Post` model definition. It is configured to bind to the `app_blog_post` table, and to dispatch `App\Resource\Blog\Post` resources.

We want to enable automatic slug generation, so go ahead and add the following class constant:

```php
const AUTO_SET_SLUG = true;
```

This tells the model to automatically generate a slug when a new blog post item is created; it will use the blog post's `label` property to do so.

By default, slugs will not be automatically updated each time the item is saved. In this tutorial we want to enable automatic updates of the slug each time a blog post is saved. We can enable this by setting the following constant:

```php
const AUTO_SET_SLUG_IMMUTABLE = false;
```

{% hint style="info" %}
Most of the model's functionality is configured using class constants. In your own time, take a look through all the options available to you in `Nails\Common\Model\Base`
{% endhint %}

### Design the database schema

Now that we have a properly defined model we should ensure that our database table is set up properly. Connect to the MySQL database using your GUI of choice and view the structure for the `app_blog_post` table which was created for you, it should look something like this:

```
CREATE TABLE `app_blog_post` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `slug` varchar(150) DEFAULT NULL,
  `label` varchar(150) DEFAULT NULL,
  `is_deleted` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `created` datetime NOT NULL,
  `created_by` int(11) unsigned DEFAULT NULL,
  `modified` datetime NOT NULL,
  `modified_by` int(11) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `created_by` (`created_by`),
  KEY `modified_by` (`modified_by`),
  CONSTRAINT `app_blog_post_ibfk_1` FOREIGN KEY (`created_by`) REFERENCES `nails_user` (`id`) ON DELETE SET NULL,
  CONSTRAINT `app_blog_post_ibfk_2` FOREIGN KEY (`modified_by`) REFERENCES `nails_user` (`id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

All these fields are useful to us, so we'll leave them there. We're also going to want:

* A `body` to store the content of our blog post
* An `excerpt` field to store a quick summary for the index page
* A `date_published` date
* An `is_published` boolean flag

Add them using your GUI, or with these `ALTER` statements:

```
ALTER TABLE `app_blog_post` ADD COLUMN `excerpt` varchar(255) DEFAULT NULL AFTER `label`;
ALTER TABLE `app_blog_post` ADD COLUMN `body` text DEFAULT NULL AFTER `excerpt`;
ALTER TABLE `app_blog_post` ADD COLUMN `date_published` datetime DEFAULT CURRENT_TIMESTAMP AFTER `body`;
ALTER TABLE `app_blog_post` ADD COLUMN `is_published` tinyint(1) DEFAULT 1 AFTER `date_published`;
```

{% hint style="success" %}
Credentials and database access details for the Docker Environment are set in `docker-compose.override.yml` - more details about the Database container can be found in the [Docker docs](https://docker.nailsapp.co.uk/containers/database).
{% endhint %}

### Add a migration

Now that we have a table, we'll want to make sure we persist it  between deployments by writing a [database migration](../core-services/database/migrations.md) which will create this table the next time `make build` is executed.

If you haven't already, now is a good time to read about how Nails handles database migrations:

{% content-ref url="../core-services/database/migrations.md" %}
[migrations.md](../core-services/database/migrations.md)
{% endcontent-ref %}

On the command line create a new migration using the CLI tool:

```bash
nails make:db:migration
```

This will generate a new blank migration, taking into consideration any existing migrations. In this tutorial it will create the `0` migration and _because_ it is the `0` migration it will automatically populate it with `CREATE` statements for every `app_*` table present in the database.

{% hint style="info" %}
This behaviour is particularly useful when creating multiple tables on a project during development. Add as many models as you like, tweak them to your heart's content.

When you are ready to regenerate the zero migration you can use the following:

```
rm application/migrations/0.php
nails make:db:migration 0
```

This can save you a lot of time and helps to avoid developer error when manually crafting `CREATE` statements.
{% endhint %}

## Seeding the database

At this stage we now have controllers, models, resources, and a database table in place. Next we want to seed this table with some data, this will make building out our frontend controllers and views a bit easier.

The seeder process is covered extensively [here](../core-services/database/seeders.md), the following is a quick introduction but assumes some basic knowledge of how the seeder system works.

{% content-ref url="../core-services/database/seeders.md" %}
[seeders.md](../core-services/database/seeders.md)
{% endcontent-ref %}

During the model creation process a seeder was automatically created for you, it lives here:

```
./www/application/src/Seed/BlogPost.php
```

If you inspect this in your IDE you can see this seeder is bound to the `App\Model\Blog\Post` model via the `CONFIG_MODEL_NAME` constant. It is using the `Nails\Common\Console\Seed\DefaultSeed` parent class which will automatically generate items based on the field types of the table.

To generate seeded content execute `make seed` from _outside_ of the web server container.

Once seeders have been run, look at the contents of the `app_blog_post` table, you should see it filled with \~20 records.

{% hint style="info" %}
Have a look at `Nails\Common\Console\Seed\DefaultSeed` to see how you can reconfiguring the seeder's behaviour, or override the `generate()` method to customise the generated object.
{% endhint %}

{% hint style="info" %}
`make seed` is a wrapper for `./www/application/scripts/seed.sh` – inspect that file to see what it is doing.
{% endhint %}

### Configure the resource

A [Resource](../key-concepts/factory/resources.md) is a representation of a single item dispensed by a model. Resources are typically map to database field names, but provide an opportunity to add additional behaviour through methods or additional class fields.

Open the Resource in your IDE:

```
./www/src/Resource/Blog/Post.php
```

Here we can see an empty Resource, which extends the base `Nails\Common\Resource\Entity` resource. The base resource defines the `$id`, `$created`, `$created_by`, `$modified`, and `$modified_by` class properties – we should add the additional ones in line with the database schema we created previously.

Add the following class properties:

```php
/** @var string */
public $slug;

/** @var string */
public $label;

/** @var string */
public $excerpt;

/** @var string */
public $body;

/** @var \Nails\Common\Resource\DateTime */
public $date_published;

/** @var bool */
public $is_published;
```

{% hint style="info" %}
Some field types are automatically cast, for example `tinyint(1)` fields are considered booleans, and DateTime fields are converted into instances of `\Nails\Common\Resource\DateTime`
{% endhint %}

The resource is a representation of a blog post, so it is sensible to define the post's URL here also, so that calling code can refer to it when generating things like links. Let's do this by adding a `url()` helper method:

```php
public function url(): string
{
    return siteUrl('blog/' . $this->slug);
}
```

## Building the index controller

Now we have seeded data, a means to retrieve it, and a structured representation of our data we will build the index controller. This controller will query the `App\Model\Blog\Post` model for published posts, sorted by published date.

### Loading published blog posts

Open the blog index controller in your IDE:

```
./www/application/modules/blog/controllers/Blog.php
```

Add the following code just above where the Views are loaded, around line 17:

```php
$oNow   = Factory::factory('DateTime');
$oModel = Factory::model('BlogPost', 'app');
$aPosts = $oModel->getAll([
    'where' => [
        ['is_published', true],
        ['date_published <', $oNow->format('Y-m-d H:i:s')],
    ],
]);
```

Here we are fetching blog posts from the model where `is_published` is true, and `date_published` is in the past as an array and assigning it to the `$aPosts` variable.

{% hint style="success" %}
If at any time you wish to inspect an object, use the debug  helper `dd()` to print out a summary of any passed arguments.
{% endhint %}

We then want to make this data available to the views, do this by using the [View service's](../key-concepts/views.md) `setData()` method:

```php
Factory::service('View')
    ->setData([
        'aPosts' => $aPosts,
    ])
    ->load([
        'structure/header',
        'blog/index',
        'structure/footer',
    ]);
```

### Rendering the posts

Open the blog index view in your IDE:

```
./www/application/modules/blog/views/index.php
```

Replace the contents of this file with the following code:

```php
<?php
/**
 * @var App\Resource\Blog\Post[] $aPosts
 */
?>
<ul>
    <?php
    foreach ($aPosts as $oPost) {
        ?>
        <li>
            <a href="<?=$oPost->url()?>">
                <?=$oPost->label?>
            </a>
            <br>
            <small>
                <?=$oPost->date_published->formatted?>
            </small>
        </li>
        <?php
    }
    ?>
</ul>
```

Here we are looping over the data passed into the view and rendering properties of the Blog\Post Resource, as well as utilising the `url()` method we created earlier.

Test it out in your browser by visiting [https://localhost/blog](https://localhost/blog), you should see something like this:

![](<../.gitbook/assets/Screenshot 2020-03-02 17.04.17.png>)

## Building the single post controller

The final step of our blogging platform journey is to complete the "single blog post" controller. This controller will check the URL for a post slug, and render a page if it is valid.

Open the blog single controller in your DIE:

```
./www/application/modules/blog/controllers/Single.php
```

First we need to check the URL for a slug, and query the model. Add the following code, before the Views are loaded:

```php
/** @var \Nails\Common\Service\Uri $oUri */
$oUri = \Nails\Factory::service('Uri');
/** @var \App\Model\Blog\Post $oModel */
$oModel = \Nails\Factory::model('BlogPost', 'app');

/** @var \App\Resource\Blog\Post $oPost */
$oPost = $oModel->getBySlug(
    $oUri->segment(2)
);

if (empty($oPost)) {
    show404();
}
```

This code, fetches the second segment of the URL and passes that to the model's `getBySlug()` method. If a post is found the `$oPost` will be an instance of `\App\Resource\Blog\Post`, `null` if it is not found. If no post is found then trigger a 404 error and halt any further execution.

As before in the index view, we should pass the post to the Views:

```php
Factory::service('View')
    ->setData([
        'oPost' => $oPost,
    ])
    ->load([
        'structure/header',
        'blog/Single/index',
        'structure/footer',
    ]);

```

Now open the single view in your IDE:

```
./www/application/modules/blog/views/Single/index.php
```

In this view we have the `App\Resource\Blog\Post` resource to play with and it is entirely up to you as the developer to decide how the page should be styled. For this tutorial however, replace the contents of the view with the following:

```php
<?php
/**
 * @var App\Resource\Blog\Post $oPost
 */
?>
<h1>
    <?=$oPost->label?>
</h1>
<p>
    <?=$oPost->excerpt?>
</p>
<hr>
<?=auto_typography($oPost->body)?>
```

In your browser, navigate to a single blog page from the index we created earlier. You should see a page which looks something like this:

![](<../.gitbook/assets/Screenshot 2020-03-02 17.16.45.png>)

And that's it - you have successfully created a very simple, albeit not very author friendly, blogging platform! 💪🏻

Next we'll learn how to add some relational data to our blog posts, and then how to build a GUI for administering your blog.
