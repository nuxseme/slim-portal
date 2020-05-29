# Request

Your Slim app’s routes and middleware are given a PSR-7 request object that represents the current HTTP request received by your web server. The request object implements the [PSR-7 ServerRequestInterface](https://www.php-fig.org/psr/psr-7/#321-psrhttpmessageserverrequestinterface) with which you can inspect and manipulate the HTTP request method, headers, and body.

## How to get the Request object

The PSR-7 request object is injected into your Slim application routes as the first argument to the route callback like this:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello', function (Request $request, Response $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->run();
```

The PSR-7 request object is injected into your Slim application *middleware* as the first argument of the middleware callable like this:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->add(function (ServerRequestInterface $request, RequestHandler $handler) {
   return $handler->handle($request);
});

// ...define app routes...

$app->run();
```



