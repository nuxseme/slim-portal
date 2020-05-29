# Route middleware



You can also attach middleware to any route or route group.

```php
$app->group('/foo', function (RouteCollectorProxy $group) {
    $group->get('/bar', function ($request, $response, $args) {

    })->add(new RouteMiddleware());
})->add(new GroupMiddleware());
```

