## Returning JSON



In it’s simplest form, JSON data can be returned with a default 200 HTTP status code.

```php
$data = array('name' => 'Bob', 'age' => 40);
$payload = json_encode($data);

$response->getBody()->write($payload);
return $response
          ->withHeader('Content-Type', 'application/json');
```

Figure 15: Returning JSON with a 200 HTTP status code.

We can also return JSON data with a custom HTTP status code.

```php
$data = array('name' => 'Rob', 'age' => 40);
$payload = json_encode($data);

$response->getBody()->write($payload);
return $response
          ->withHeader('Content-Type', 'application/json')
          ->withStatus(201);
```

Figure 16: Returning JSON with a 201 HTTP status code.

**Reminder**

The Response object is immutable. This method returns a *copy* of the Response object that has a new Content-Type header. **This method is destructive**, and it *replaces* the existing Content-Type header.

## Returning a Redirect



You can redirect the HTTP client by using the `Location` header.

```php
return $response
  ->withHeader('Location', 'https://www.example.com')
  ->withStatus(302);
```

