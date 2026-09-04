# Widgets

Widgets are the underlying work horse of the CMS and range from being incredibly simple to extremely complex. There is no real limit to what widgets can and cannot do.

## Anatomy of a Widget

Widgets are essentially a single class with two companion views: one for the front end and one for the back end. Users can place widgets into widget areas and define properties based on the `<form>` contents of `views/editor.php`.

In addition, widgets can supply some custom JS to enhance the admin experience (e.g. dynamic fields). A basic widget directory tree looks like this:

```
application/
    cms/
        widgets/
            MyWidget/
                widget.php
                screenshot.png
                views/
                    editor.php
                    render.php
                js/
                    dropped.js

```

`widget.php` is the [Widget Definition](./#widget-definition).

`screenshot.php` is an optional screenshot to display along side the widget in the editor sidebar; this should be around 500px wide and show the widget with as little surrounding content as possible.

`views/editor.php` is the[ Editor View](./#editor-view).

`views/render.php` is the [Render View](./#render-view).

`js/dropped.js` is the editor's [javascript](./#javascript).

## Creating Widgets

The easiest way to create a widget is to use the Command Line Tool:

```bash
nails make:cms:widget MyWidget
```

This will create a widget called `MyWidget` for you, with stubs for you to alter as necessary.

### Widget Definition

This is a class which matches the widget's name and is in the App\Cms\Widget namespace. It declares your widget's details.

```php
// application/modules/cms/widgets/MyWidget.php

namespace App\Cms\Widget;

use Nails\Cms\Widget\WidgetBase;

class MyWidget extends WidgetBase
{
    public function __construct()
    {
        parent::__construct();

        $this->label       = 'My Widget';
        $this->grouping    = 'Generic';
        $this->description = 'A short description about the widget';
        $this->keywords    = 'some,searchable,keywords';
        
        // Keys defined here are available as variables in both views
        $this->data = [
            'sBody' => '<p>Default body text</p>',
        ];
    }
}
```

### Editor View

The editor view is optional, but if provided is a means for you to offer the user some configurable options (e.g. text input or select  option). The structure of this file is up to you - the wiget editor view will automatically parse out form input elements and save them as variables. Once saved they are made available to the view, so they can be repopulated.

```php
// application/modules/cms/widgets/views/editor.php

echo form_field_textarea([
    'key'     => 'sBody',
    'label'   => 'Body',
    'default' => $sBody
]);
```

### Render View

This view is what is rendered in the front end when a widget area is rendered. It is a basic PHP view which is passed the form input elements from the editor as variables which match the input names.

```php
// application/modules/cms/widgets/views/render.php

if (!empty($sBody)) {
    ?>
    <div class="cms-widget">
        <?=$sBody?>
    </div>
    <?
}
```

### Javascript

Additional Javascript is optional, but if available will be called each time a new instance of the widget is added to the widget editor interface. It will be called within a closure which makes the DOM element available via a variable called `domElement`; use this to bind custom actions to items within the editor interface.

```javascript
// application/modules/cms/widgets/js/dropped.js

domElement
    .querySelector('.some-class')
    .addEventListener(...);javascript
```

## Default Widgets

There are a number of commonly used widgets which are bundled with the module. If you're stuck, this can be a good place to look.

```
vendor/nails/module-cms/cms/widgets/*
```

## Widget Helpers

The helper `cmsWidget(string $sSlug, array $aData = [])` is available for rendering widgets on their own; the first parameter is the widget's slug/classname and the second is any key:value data you wish to pass to the render view.

```php
<h1>An Example</h1>
<?=cmsWidget('MyWidget', ['body' => '<p>This is some body text.</p>'])?>
```

## Overriding Module Widgets

Widgets provided by the app are loaded last; if the slug matches exactly then will override any module-provided widgets. This gives you an opportunity to alter the behaviour of the widget, or it's views. A common scenario is to set the `DISABLED` constant to `true` so that the widget is not offered to the end user.

## Deprecating Widgets

Over time widgets designs evolve and widgets may become outdated and shouldn't be used. Highlight that a widget is deprecated by setting its `DEPRECATED` constant to `true`. Additionally, if another widget should be used instead you can specify alternative(s) by setting the `ALTERNATIVE` constant. The widget editor will highlight that the widget is deprecated and inform the user which alternative to use instead.

```php
// application/modules/cms/widgets/MyWidget.php

namespace App\Cms\Widget;

use Nails\Cms\Widget\WidgetBase;

class MyWidget extends WidgetBase
{
    const DEPRECATED  = true;
    const ALTERNATIVE = 'MyOtherWidget or SomeOtherWidget';

    public function __construct()
    {
        parent::__construct();

        $this->label       = 'My Widget';
        $this->grouping    = 'Generic';
        $this->description = 'A short description about the widget';
        $this->keywords    = 'some,searchable,keywords';
        
        // Keys defined here are available as variables in both views
        $this->data = [
            'sBody' => '<p>Default body text</p>',
        ];
    }
}
```

