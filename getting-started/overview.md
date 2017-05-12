## Overview

- [What is Nails](#intro)
- [How does it work](#intro-how)
- [What are components](#intro-components)
- [Where do you get components](#intro-packagist)
- [Anatomy of an application](#intro-anatomy-application)
- [Anatomy of a component](#intro-anatomy-component)

--

<a name="intro"></a>
### What is Nails

Nails is an open-source PHP framework. Super quick and easy to install and work with, it contains numerous handy modules to suit the most common use cases.
Nails is **not** an out-of-the-box, plug-and-play CMS, blogging platform or shopping platform (like WordPress, Drupal or ExpressionEngine); it is a tool for developers, allowing them to build any type of site efficiently and effectively without coercing a tool that doesn't quite fit their needs.
Nails sits on top of the also open-source framework CodeIgniter, as well as utilising components from Symfony; its goal is to make developing powerful websites simple by focusing on innovation over repetition (i.e not re-inventing the wheel), all whilst giving the developer a huge amount of flexibility so as to meet the needs of the business. In addition, all user-facing interfaces are built with user experience in mind to make managing the content a pleasure, not a chore.
Out of the box, Nails does very little besides managing the essentials (like session management, admin, logging, error catching, routing and a whole bunch of handy helpers and tools) - a deliberate decision in order to keep the operating footprint as low as possible. Its real power comes through the use of the powerful component system (which leverages Composer), allowing complete chunks of functionality to be brought in so time and money can be spent on building the “important stuff”.Nails is a young framework, but one which is actively developed with features, fixes and updates being pushed almost every day. 



<a name="intro-how"></a>
### How does it work

> @todo - complete this section
> 
> - beef this section out



<a name="intro-components"></a>
### What are components

> @todo - complete this section
> 
> - composer packages
> - contain the `nails` property in composer.json



<a name="intro-packagist"></a>
### Where do you get components

Nails is distributed via Composer so it is natural for components to be delivered this way, too.

Official components, as well as any which are intended to be used by the wider community, use [Packagist](http://packagist.org). While this is the preferred method of distribution, it iss also perfectly aceptable to distribute using your own VCS - if composer can see it, it can be used. 

> @todo - complete this section
> 
> - bundled components (i.e not pulled in using composer)



<a name="intro-anatomy-application"></a>
### Anatomy of an application

> @todo - complete this section
> 
> - application directory
> - modules directory
> - src directory
> - services.php
> - logging
> - routes
> - console



<a name="intro-anatomy-component"></a>
### Anatomy of a component

> @todo - complete this section
> 
> - composer.json
> - src directory
> - public directory
