# Route strategies



The route callback signature is determined by a route strategy. By default, Slim expects route callbacks to accept the request, response, and an array of route placeholder arguments. This is called the RequestResponse strategy. However, you can change the expected route callback signature by simply using a different strategy. As an example, Slim provides an alternative strategy called `RequestResponseArgs` that accepts request and response, plus each route placeholder as a separate argument.

Here is an example of using this alternative strategy:

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Handlers\Strategies\RequestResponseArgs;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * Changing the default invocation strategy on the RouteCollector component
 * will change it for every route being defined after this change being applied
 */
$routeCollector = $app->getRouteCollector();
$routeCollector->setDefaultInvocationStrategy(new RequestResponseArgs());

$app->get('/hello/{name}', function ($request, $response, $name) {
    return $response->write($name);
});
```

Alternatively you can set a different invocation strategy on a per route basis:

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Handlers\Strategies\RequestResponseArgs;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();
$routeCollector = $app->getRouteCollector();

$route = $app->get('/hello/{name}', function ($request, $response, $name) {
    return $response->write($name);
});
$route->setInvocationStrategy(new RequestResponseArgs());
```

You can provide your own route strategy by implementing the `Slim\Interfaces\InvocationStrategyInterface`.

