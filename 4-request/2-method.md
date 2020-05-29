# Method

## The Request Method

Every HTTP request has a method that is typically one of:

- GET
- POST
- PUT
- DELETE
- HEAD
- PATCH
- OPTIONS

You can inspect the HTTP request’s method with the Request object method appropriately named `getMethod()`.

```php
$method = $request->getMethod();
```

It is possible to fake or *override* the HTTP request method. This is useful if, for example, you need to mimic a `PUT` request using a traditional web browser that only supports `GET` or `POST` requests.

**Heads Up!**

To enable request method overriding the [Method Overriding Middleware](https://www.slimframework.com/docs/v4/middleware/method-overriding.html) must be injected into your application.



There are two ways to override the HTTP request method. You can include a `METHOD` parameter in a `POST` request’s body. The HTTP request must use the `application/x-www-form-urlencoded` content type.

```http
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 22

data=value&_METHOD=PUT
```

You can also override the HTTP request method with a custom `X-Http-Method-Override` HTTP request header. This works with any HTTP request content type.

```Http
POST /path HTTP/1.1
Host: example.com
Content-type: application/json
Content-length: 16
X-Http-Method-Override: PUT

{"data":"value"}
```

## The Request URI



Every HTTP request has a URI that identifies the requested application resource. The HTTP request URI has several parts:

- Scheme (e.g. `http` or `https`)
- Host (e.g. `example.com`)
- Port (e.g. `80` or `443`)
- Path (e.g. `/users/1`)
- Query string (e.g. `sort=created&dir=asc`)

You can fetch the PSR-7 Request object’s [URI object](https://www.php-fig.org/psr/psr-7/#35-psrhttpmessageuriinterface) with its `getUri()` method:

```php
$uri = $request->getUri();
```

The PSR-7 Request object’s URI is itself an object that provides the following methods to inspect the HTTP request’s URL parts:

- getScheme()
- getAuthority()
- getUserInfo()
- getHost()
- getPort()
- getPath()
- getBasePath()
- getQuery() (returns the full query string, e.g. `a=1&b=2`)
- getFragment()
- getBaseUrl()

You can get the query parameters as an associative array on the Request object using `getQueryParams()`.

> **Base Path**
>
> If your Slim application's front-controller lives in a physical subdirectory beneath your document root directory, you can fetch the HTTP request's physical base path (relative to the document root) with the Uri object's `getBasePath()` method. This will be an empty string if the Slim application is installed in the document root's top-most directory.

## 