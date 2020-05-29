# Body

## The Request Body

Every HTTP request has a body. If you are building a Slim application that consumes JSON or XML data, you can use the PSR-7 Request object’s `getParsedBody()` method to parse the HTTP request body into a native PHP format. Note that body parsing differs from one PSR-7 implementation to another.

You may need to implement middleware in order to parse the incoming input depending on the PSR-7 implementation you have installed. Here is an example for parsing incoming `JSON` input:

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface as RequestHandler;

class JsonBodyParserMiddleware implements MiddlewareInterface
{
    public function process(Request $request, RequestHandler $handler): Response
    {
        $contentType = $request->getHeaderLine('Content-Type');

        if (strstr($contentType, 'application/json')) {
            $contents = json_decode(file_get_contents('php://input'), true);
            if (json_last_error() === JSON_ERROR_NONE) {
                $request = $request->withParsedBody($contents);
            }
        }

        return $handler->handle($request);
    }
}
```

```php
$parsedBody = $request->getParsedBody();
```

Technically speaking, the PSR-7 Request object represents the HTTP request body as an instance of `Psr\Http\Message\StreamInterface`. You can get the HTTP request body `StreamInterface` instance with the PSR-7 Request object’s `getBody()` method. The `getBody()` method is preferable if the incoming HTTP request size is unknown or too large for available memory.

```php
$body = $request->getBody();
```

The resultant `Psr\Http\Message\StreamInterface` instance provides the following methods to read and iterate its underlying PHP `resource`.

- getSize()
- tell()
- eof()
- isSeekable()
- seek()
- rewind()
- isWritable()
- write($string)
- isReadable()
- read($length)
- getContents()
- getMetadata($key = null)

## 