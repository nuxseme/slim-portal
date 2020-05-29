# Route expressions caching



It’s possible to enable router cache via `RouteCollector::setCacheFile()`. See examples below:

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

/**
 * To generate the route cache data, you need to set the file to one that does not exist in a writable directory.
 * After the file is generated on first run, only read permissions for the file are required.
 *
 * You may need to generate this file in a development environment and comitting it to your project before deploying
 * if you don't have write permissions for the directory where the cache file resides on the server it is being deployed to
 */
$routeCollector = $app->getRouteCollector();
$routeCollector->setCacheFile('/path/to/cache.file');
```