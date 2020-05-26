# Deployment

## Disable error display in production

```php
<?php
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// ...

// If you are adding the pre-packaged ErrorMiddleware set `displayErrorDetails` to `false`
//tips可以根据不同的环境加载不同的配置 设置不同的参数
$app->addErrorMiddleware(false, true, true);

// ...

$app->run();
```