# Route callbacks



Each routing method described above accepts a callback routine as its final argument. This argument can be any PHP callable, and by default it accepts three arguments.

- `Request` The first argument is a `Psr\Http\Message\ServerRequestInterface` object that represents the current HTTP request.
- `Response` The second argument is a `Psr\Http\Message\ResponseInterface` object that represents the current HTTP response.
- `Arguments` The third argument is an associative array that contains values for the current route’s named placeholders.

### Writing content to the response

There are two ways you can write content to the HTTP response. First, you can simply `echo()` content from the route callback. This content will be appended to the current HTTP response object. Second, you can return a `Psr\Http\Message\ResponseInterface` object.

echo 这种方式会被ob_clean()  违背Slim的设计思想

### Closure binding

If you use a [dependency container](https://www.slimframework.com/docs/v4/concepts/di.html) and a `Closure` instance as the route callback, the closure’s state is bound to the `Container` instance. This means you will have access to the DI container instance *inside* of the Closure via the `$this` keyword:

```php
$app->get('/hello/{name}', function ($request, $response, $args) {
    // Use app HTTP cookie service
    $this->get('cookies')->set('name', [
        'value' => $args['name'],
        'expires' => '7 days'
    ]);
});
```

> **Heads Up!**
>
> Slim does not support `static` closures.

闭包绑定在容器上，具名class method来自定义器，在反射的时候根据__construct()的参数动态绑定，所以在应用层的方法中需要使用容器需要构造函数包含Container $container类型的参数 

## Redirect helper

You can add a route that redirects `GET` HTTP requests to a different URL with the Slim application’s `redirect()` method. It accepts three arguments:

1. The route pattern (with optional named placeholders) to redirect `from`
2. The location to redirect `to`, which may be a `string` or a [Psr\Http\Message\UriInterface](https://www.php-fig.org/psr/psr-7/#35-psrhttpmessageuriinterface)
3. The HTTP status code to use (optional; `302` if unset)

```php
$app->redirect('/books', '/library', 301);
```

`redirect()` routes respond with the status code requested and a `Location` header set to the second argument.