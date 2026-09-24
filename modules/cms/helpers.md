# Helpers



The following helpers are made available by the CMS module.

### Loading the helper

```php
use Nails\Factory;
Factory::helper('cms', 'nails/module-cms');
```

### Admin: widget editor field

`form_field_cms_widgets()` (and model type `Nails\Cms\Helper\Form::FIELD_WIDGETS`) renders a button that opens the widget editor. It is a [form field](../../../key-concepts/form-fields.md), so `tip`, `required`, and `info` work the same as other helpers.

```php
echo form_field_cms_widgets([
    'key'   => 'body',
    'label' => 'Body',
    'tip'   => 'Widgets are rendered in order on the front end.',
]);
```

### Front end

#### cmsBlock($sSlug)

> @todo - write up this helper

#### cmsSlider($sIdSlug)

> @todo - write up this helper

#### cmsMenu($mIdSlug)

> @todo - write up this helper

#### cmsMenuNested($mIdSlug)

> @todo - write up this helper

#### cmsPage($mIdSlug)

> @todo - write up this helper

#### cmsArea($mIdSlug)

> @todo - write up this helper

#### cmsAreaWithData($aData)

> @todo - write up this helper

#### cmsWidget($sSlug, $aData = array())

> @todo - write up this helper
