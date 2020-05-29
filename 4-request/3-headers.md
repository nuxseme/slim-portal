# Headers

## The Request Headers

Every HTTP request has headers. These are metadata that describe the HTTP request but are not visible in the request’s body. Slim’s PSR-7 Request object provides several methods to inspect its headers.

### Get All Headers

You can fetch all HTTP request headers as an associative array with the PSR-7 Request object’s `getHeaders()` method. The resultant associative array’s keys are the header names and its values are themselves a numeric array of string values for their respective header name.

```php
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```

### Get One Header

You can get a single header’s value(s) with the PSR-7 Request object’s `getHeader($name)` method. This returns an array of values for the given header name. Remember, *a single HTTP header may have more than one value!*

```php
$headerValueArray = $request->getHeader('Accept');
```

You may also fetch a comma-separated string with all values for a given header with the PSR-7 Request object’s `getHeaderLine($name)` method. Unlike the `getHeader($name)` method, this method returns a comma-separated string.

```php
$headerValueString = $request->getHeaderLine('Accept');
```

### Detect Header

You can test for the presence of a header with the PSR-7 Request object’s `hasHeader($name)` method.

```php
if ($request->hasHeader('Accept')) {
    // Do something
}
```

