# Building

## Standard Controllers

API Controllers should exist in the `Api/Controller` namespace. Additionally, they should also extend the base controller `Nails\Api\Controller\Base`.

Methods should return instances of `Nails\Api\Factory\ApiResponse`, setting the payload via the `setData()` method, if required.

Methods are prefixed with the HTTP method they should bind to, e.g `GET`, `POST`, `PUT`, `DELETE`, etc.

```php
namespace Nails\Api\App;

use App\Model;
use App\Resource;
use Nails\Api\Controller\Base;
use Nails\Api\Factory\ApiResponse;
use Nails\Factory;

class Books extends Base
{
    public function getIndex()
    {
        /** @var Model\Book $oModel */
        $oModel = Factory::model('Book', 'app');
        /** @var Resource\Book[] $aBooks */
        $aBooks = $oModel->getAll();
    
        /** @var ApiResponse $oResponse */
        $oResponse = Factory::factory('ApiResponse', 'nails/module-api');
        $oResponse-setData($aBooks);
        
        return $oResponse;
    }

    public function postReview()
    {
    
        /**
         * Do something, such as create a new review
         */
         
        /** @var ApiResponse $oResponse */
        $oResponse = Factory::factory('ApiResponse', 'nails/module-api');
        return $oResponse;
    }
}
```

### URL Structure

Similar to the main URL structure, API routes follow the following URL structure:

```
api/<module>/<controller>/<method>
```

{% hint style="info" %}
For controllers provided by the application, the \<module> will be app. Other components which provide API controllers [register their own API namespace](building.md#api-registration).
{% endhint %}

The previous example provides two endpoints:

```
GET  /api/app/books
POST /api/app/books/review
```

### Authentication

If you need to restrict the endpoint to only users who are logged in then you can set the class' `REQUIRE_AUTH` constant to `true`. If using access tokens, you can specify the scope an access token should have by setting the `REQUIRE_SCOPE` constant.

For more refined authentication, controllers can override the `isAuthenticated($sHttpMethod, $sMethod)` method to provide custom logic to determine whether a request is authenticated.

### Logging

If you need to write to the API log you can do so using the `writeLog($sLine)` method provided by the `Base` class.

### Errors

Any thrown `\Nails\Api\Exception\ApiException` exceptions will be caught by the router and relayed to the user as a typical API response. All other exceptions will not be caught and will be handled by the active [ErrorHandler](../../core-services/error-handler/).

### Rate Limiting

@todo - implement this

## CRUD Controllers

The CRUD controller allows you to bind an endpoint to a model and provides typical Create, Read, Update, and Delete functionality.

```php
namespace Nails\Api\App;

use Nails\Api\Controller\CrudController;

class Book extends CrudController
{
    const CONFIG_MODEL_NAME     = 'Book';
    const CONFIG_MODEL_PROVIDER = 'app';
}
```

{% hint style="info" %}
The `CrudController` is highly configurable via class constants. [Inspect the source](https://github.com/nails/module-api/blob/master/src/Controller/CrudController.php) to learn more.
{% endhint %}

## URL/Controller Mapping

By default, the URL maps to the controller class on disk, however there may be certain circumstances where you may wish to use a URL which does not map to the named file (e.g. if you wish to use a reserved word).

This is achieved by specifying the `controller-map` property in the module's `composer.json` file under the `nails/module-api` namespace:

```javascript
{
  "extra": {
    "nails" : {
      "data": {
        "nails/module-api": {
          "controller-map": {
            "Object": "MyObject"
          }
        }
      }
    }
  }
}
```

In the above example, the url `api/app/object` would resolve to the controller `App\Api\Controller\MyObject`.

## Module Development

{% hint style="warning" %}
This section only applies if you're building a module which will be added as a dependency.
{% endhint %}

### API Registration

Modules should register their intent to provide API controllers by specifying the `nails/module-api.namespace` property in their `composer.json` file. This field is what the API will use to bind URLs to their controllers; e.g. setting this to `store` would expose the module's API controllers at `/api/store/{{controller}`.

Example `composer.json` file (with most fields redacted for brevity):

```
{
    "extra":
    {
        "nails" :
        {
            "moduleName": "store",
            "type": "module",
            "namespace": "Nails\\Store\\",
            "data": {
                "nails/module-api": {
                    "namespace": "store"
                }
            }
        }
    }
}
```
