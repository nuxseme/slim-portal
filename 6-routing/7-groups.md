# Route groups



To help organize routes into logical groups, the `Slim\App` also provides a `group()` method. Each group’s route pattern is prepended to the routes or groups contained within it, and any placeholder arguments in the group pattern are ultimately made available to the nested routes:

```php
$app->group('/users/{id:[0-9]+}', function (RouteCollectorProxy $group) {
    $group->map(['GET', 'DELETE', 'PATCH', 'PUT'], '', function ($request, $response, $args) {
        // Find, delete, patch or replace user identified by $args['id']
    })->setName('user');
    
    $group->get('/reset-password', function ($request, $response, $args) {
        // Route for /users/{id:[0-9]+}/reset-password
        // Reset the password for user identified by $args['id']
    })->setName('user-password-reset');
});
```

The group pattern can be empty, enabling the logical grouping of routes that do not share a common pattern.

```php
$app->group('', function (RouteCollectorProxy $group) {
    $group->get('/billing', function ($request, $response, $args) {
        // Route for /billing
    });
    
    $group->get('/invoice/{id:[0-9]+}', function ($request, $response, $args) {
        // Route for /invoice/{id:[0-9]+}
    });
})->add(new GroupMiddleware());
```

Note inside the group closure, Slim binds the closure to the container instance.

- inside route closure, `$this` is bound to the instance of `Psr\Container\ContainerInterface`