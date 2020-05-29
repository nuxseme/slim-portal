# Route Object

Sometimes in middleware you require the parameter of your route.

In this example we are checking first that the user is logged in and second that the user has permissions to view the particular video they are attempting to view.

```php
<?php
$app->get('/course/{id}', Video::class.":watch")->add(PermissionMiddleware::class);
```



```php
<?php
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Routing\RouteContext;

class PermissionMiddleware {
    public function __invoke(Request $request, RequestHandler $handler) {
        $routeContext = RouteContext::fromRequest($request);
        $route = $routeContext->getRoute();
        
        $courseId = $route->getArgument('id');
        
        // ...do permission logic...
        
        return $handler->handle($request);
    }
}
```

## Obtain Base Path From Within Route

To obtain the base path from within a route simply do the following:

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Routing\RouteContext;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function($request, $response) {
    $routeContext = RouteContext::fromRequest($request);
    $basePath = $routeContext->getBasePath();
    
    // ...
    
    return $response;
});
```



## Attributes

With PSR-7 it is possible to inject objects/values into the request object for further processing. In your applications middleware often need to pass along information to your route closure and the way to do it is to add it to the request object via an attribute.

Example, Setting a value on your request object.

```php
$app->add(function ($request, $handler) {
    // add the session storage to your request as [READ-ONLY]
    $request = $request->withAttribute('session', $_SESSION);
    return $handler->handle($request);
});
```

Example, how to retrieve the value.

```php
$app->get('/test', function ($request, $response, $args) {
    $session = $request->getAttribute('session'); // get the session from the request
    return $response->write('Yay, ' . $session['name']);
});
```

The request object also has bulk functions as well. `$request->getAttributes()` and `$request->withAttributes()`



中间件参数传递 采用request->withAttributes(key,value);

