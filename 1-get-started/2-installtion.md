# Installation

## System Requirements

- Web server with URL rewriting
- PHP 7.2 or newer

## Step 1: Install Composer

## Step 2: Install Slim

已有MCV代码了

```bash
composer require slim/slim
```

初始化项目建议以下

```bash
compser create-project slim/slim-skeleton:dev-master [my-app-name]
```

## Step 3: Install a PSR-7 Implementation and ServerRequest Creator

这一步必须要，Slim源码中只定义了默认的类，但是没有安装第三方的类库

### [Slim PSR-7](https://github.com/slimphp/Slim-Psr7)

建议安装Slim 官方的，其他的不知道后续是不是继续维护

```bash
composer require slim/psr7
```

## Step 4: Hello World

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/', function (Request $request, Response $response, $args) {
    $response->getBody()->write("Hello world!");
    return $response;
});

$app->run();
```