# Request Helpers



Slim’s PSR-7 Request implementation provides these additional proprietary methods to help you further inspect the HTTP request.

### Detect XHR requests



You can detect XHR requests by checking if the header `X-Requested-With` is `XMLHttpRequest` using the Request’s `getHeaderLine()` method.

```bash
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 7
X-Requested-With: XMLHttpRequest

foo=bar
```

Figure 13: Example XHR request.

```php
if ($request->getHeaderLine('X-Requested-With') === 'XMLHttpRequest') {
    // Do something
}
```

### Content Type



You can fetch the HTTP request content type with the Request object’s `getHeaderLine()` method.

```php
$contentType = $request->getHeaderLine('Content-Type');
```

### Content Length



You can fetch the HTTP request content length with the Request object’s `getHeaderLine()` method.

```php
$length = $request->getHeaderLine('Content-Length');
```

### Request Parameter



To fetch single request parameter value. You will need to use `getServerParams()`

For example, to get a single Server Parameter:

```php
$params = $request->getServerParams();
$authorization = isset($params['HTTP_AUTHORIZATION']) ? $params['HTTP_AUTHORIZATION'] : null;
```

## 