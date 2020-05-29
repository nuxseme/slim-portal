# The Response Headers



Every HTTP response has headers. These are metadata that describe the HTTP response but are not visible in the response’s body. The PSR-7 Response object provides several methods to inspect and manipulate its headers.

### Get All Headers



You can fetch all HTTP response headers as an associative array with the PSR-7 Response object’s `getHeaders()` method. The resultant associative array’s keys are the header names and its values are themselves a numeric array of string values for their respective header name.

```php
$headers = $response->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```

 

### Get One Header



You can get a single header’s value(s) with the PSR-7 Response object’s `getHeader($name)` method. This returns an array of values for the given header name. Remember, *a single HTTP header may have more than one value!*

```php
$headerValueArray = $response->getHeader('Vary');
```

 

You may also fetch a comma-separated string with all values for a given header with the PSR-7 Response object’s `getHeaderLine($name)` method. Unlike the `getHeader($name)` method, this method returns a comma-separated string.

```php
$headerValueString = $response->getHeaderLine('Vary');
```

 

### Detect Header



You can test for the presence of a header with the PSR-7 Response object’s `hasHeader($name)` method.

```php
if ($response->hasHeader('Vary')) {
    // Do something
}
```

Figure 8: Detect presence of a specific HTTP header.

### Set Header



You can set a header value with the PSR-7 Response object’s `withHeader($name, $value)` method.

```php
$newResponse = $oldResponse->withHeader('Content-type', 'application/json');
```

 

> **Reminder**
>
> The Response object is immutable. This method returns a *copy* of the Response object that has the new header value. **This method is destructive**, and it *replaces* existing header values already associated with the same header name.

### Append Header



You can append a header value with the PSR-7 Response object’s `withAddedHeader($name, $value)` method.

```php
$newResponse = $oldResponse->withAddedHeader('Allow', 'PUT');
```

 

> **Reminder**
>
> Unlike the `withHeader()` method, this method *appends* the new value to the set of values that already exist for the same header name. The Response object is immutable. This method returns a *copy* of the Response object that has the appended header value.

### Remove Header



You can remove a header with the Response object’s `withoutHeader($name)` method.

```php
$newResponse = $oldResponse->withoutHeader('Allow');
```

 

> **Reminder**
>
> The Response object is immutable. This method returns a *copy* of the Response object that has the appended header value.