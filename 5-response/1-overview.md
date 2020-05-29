# Response

Your Slim app’s routes and middleware are given a PSR-7 response object that represents the current HTTP response to be returned to the client. The response object implements the [PSR-7 ResponseInterface](https://www.php-fig.org/psr/psr-7/#33-psrhttpmessageresponseinterface) with which you can inspect and manipulate the HTTP response status, headers, and body.

## How to get the Response object



The PSR-7 response object is injected into your Slim application routes as the second argument to the route callback like this:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello', function (ServerRequest $request, Response $response) {
    $response->getBody()->write('Hello World');
    return $response;
});

$app->run();
```

