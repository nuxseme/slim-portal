# The Response Status



Every HTTP response has a numeric [status code](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). The status code identifies the *type* of HTTP response to be returned to the client. The PSR-7 Response object’s default status code is `200` (OK). You can get the PSR-7 Response object’s status code with the `getStatusCode()` method like this.

```php
$status = $response->getStatusCode();
```

You can copy a PSR-7 Response object and assign a new status code like this:

```php
$newResponse = $response->withStatus(302);
```

