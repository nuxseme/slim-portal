# Route names



Application routes can be assigned a name. This is useful if you want to programmatically generate a URL to a specific route with the RouteParser’s `urlFor()` method. Each routing method described above returns a `Slim\Route` object, and this object exposes a `setName()` method.

```php
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
})->setName('hello');
```

You can generate a URL for this named route with the application RouteParser’s `urlFor()` method.

```php
$routeParser = $app->getRouteCollector()->getRouteParser();
echo $routeParser->urlFor('hello', ['name' => 'Josh'], ['example' => 'name']);

// Outputs "/hello/Josh?example=name"
```

The RouteParser’s `urlFor()` method accepts three arguments:

- `$routeName` The route name. A route’s name can be set via `$route->setName('name')`. Route mapping methods return an instance of `Route` so you can set the name directly after mapping the route. e.g.: `$app->get('/', function () {...})->setName('name')`
- `$data` Associative array of route pattern placeholders and replacement values.
- `$queryParams` Associative array of query parameters to be appended to the generated url.