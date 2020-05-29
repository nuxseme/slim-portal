# How to create routes



You can define application routes using proxy methods on the `Slim\App` instance. The Slim Framework provides methods for the most popular HTTP methods.

### GET Route

You can add a route that handles only `GET` HTTP requests with the Slim application’s `get()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->get('/books/{id}', function ($request, $response, $args) {
    // Show book identified by $args['id']
});
```

### POST Route

You can add a route that handles only `POST` HTTP requests with the Slim application’s `post()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->post('/books', function ($request, $response, $args) {
    // Create new book
});
```

### PUT Route

You can add a route that handles only `PUT` HTTP requests with the Slim application’s `put()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->put('/books/{id}', function ($request, $response, $args) {
    // Update book identified by $args['id']
});
```

### DELETE Route

You can add a route that handles only `DELETE` HTTP requests with the Slim application’s `delete()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->delete('/books/{id}', function ($request, $response, $args) {
    // Delete book identified by $args['id']
});
```

### OPTIONS Route

You can add a route that handles only `OPTIONS` HTTP requests with the Slim application’s `options()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->options('/books/{id}', function ($request, $response, $args) {
    // Return response headers
});
```

### PATCH Route

You can add a route that handles only `PATCH` HTTP requests with the Slim application’s `patch()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->patch('/books/{id}', function ($request, $response, $args) {
    // Apply changes to book identified by $args['id']
});
```

### Any Route

You can add a route that handles all HTTP request methods with the Slim application’s `any()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```php
$app->any('/books/[{id}]', function ($request, $response, $args) {
    // Apply changes to books or book identified by $args['id'] if specified.
    // To check which method is used: $request->getMethod();
});
```

Note that the second parameter is a callback. You could specify a Class which implementes the `__invoke()` method instead of a Closure. You can then do the mapping somewhere else:

```php
$app->any('/user', 'MyRestfulController');
```

### Custom Route

You can add a route that handles multiple HTTP request methods with the Slim application’s `map()` method. It accepts three arguments:

1. Array of HTTP methods
2. The route pattern (with optional named placeholders)
3. The route callback

```php
$app->map(['GET', 'POST'], '/books', function ($request, $response, $args) {
    // Create new book or list all books
});
```

##  