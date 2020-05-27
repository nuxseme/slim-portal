# PSR-7 and Value Objects

> Slim supports [PSR-7](https://github.com/php-fig/http-message) interfaces for its Request and Response objects. This makes Slim flexible because it can use *any* PSR-7 implementation. For example, you could return an instance of `GuzzleHttp\Psr7\CachingStream` or any instance returned by the `GuzzleHttp\Psr7\stream_for()` function.
>
> Slim provides its own PSR-7 implementation so that it works out of the box. However, you are free to replace Slim’s default PSR-7 objects with a third-party implementation. Just override the application container’s `request` and `response` services so they return an instance of `Psr\Http\Message\ServerRequestInterface` and `Psr\Http\Message\ResponseInterface`, respectively.

Slim request 和 response 实现了PSR-7，带给了Slim非常灵活的组件选择，只要实现了PSR-7的HTTP request和response都可以替换。

中间件 实现 MiddlewareInface 

处理请求实现 ResquestHandleInterface



## Value objects

值对象不可修改

> Request and Response objects are [*immutable value objects*](http://en.wikipedia.org/wiki/Value_object). They can be “changed” only by requesting a cloned version that has updated property values. Value objects have a nominal overhead because they must be cloned when their properties are updated. This overhead does not affect performance in any meaningful way.
>
> You can request a copy of a value object by invoking any of its PSR-7 interface methods (these methods typically have a `with` prefix). For example, a PSR-7 Response object has a `withHeader($name, $value)` method that returns a cloned value object with the new HTTP header.



```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/foo', function (Request $request, Response $response, array $args) {
    $payload = json_encode(['hello' => 'world'], JSON_PRETTY_PRINT);
    $response->getBody()->write($payload);
    return $response->withHeader('Content-Type', 'application/json');
});

$app->run();
```



The PSR-7 interface provides these methods to transform Request and Response objects:

- `withProtocolVersion($version)`
- `withHeader($name, $value)`
- `withAddedHeader($name, $value)`
- `withoutHeader($name)`
- `withBody(StreamInterface $body)`

The PSR-7 interface provides these methods to transform Request objects:

- `withMethod($method)`
- `withUri(UriInterface $uri, $preserveHost = false)`
- `withCookieParams(array $cookies)`
- `withQueryParams(array $query)`
- `withUploadedFiles(array $uploadedFiles)`
- `withParsedBody($data)`
- `withAttribute($name, $value)`
- `withoutAttribute($name)`

The PSR-7 interface provides these methods to transform Response objects:

- `withStatus($code, $reasonPhrase = '')`

Refer to the [PSR-7 documentation](http://www.php-fig.org/psr/psr-7/) for more information about these methods.



# 总结

实现接口的好处，可以模拟request和response，单元测试

面对接口编程，代码协议

值对象属性   **不可修改**  需要修改重新clone对象返回