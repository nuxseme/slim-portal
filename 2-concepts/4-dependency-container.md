# Dependency Container

> Slim uses an optional dependency container to prepare, manage, and inject application dependencies. Slim supports containers that implement [PSR-11](http://www.php-fig.org/psr/psr-11/) like [PHP-DI](http://php-di.org/doc/frameworks/slim.html).

## Example usage with PHP-DI

You don’t *have* to provide a dependency container. If you do, however, you must provide an instance of the container to `AppFactory` before creating an `App`.

```php
<?php
use DI\Container;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

// Create Container using PHP-DI
$container = new Container();

// Set container to create App with on AppFactory
AppFactory::setContainer($container);
$app = AppFactory::create();
```

挂载服务到容器

配置也可以单独挂载到配置项

```php
$container->set('myService', function () {
    $settings = [...];
    return new MyService($settings);
});
```

You can fetch services from your container explicitly as well as from inside a Slim application route like this:

闭包被绑定到了容器上 

```php
/**
 * Example GET route
 *
 * @param  ServerRequestInterface $request  PSR-7 request
 * @param  ResponseInterface      $response  PSR-7 response
 * @param  array                  $args Route parameters
 *
 * @return ResponseInterface
 */
$app->get('/foo', function (Request $request, Response $response, $args) {
    $myService = $this->get('myService');

    // ...do something with $myService...

    return $response;
});
```

To test if a service exists in the container before using it, use the `has()` method, like this:

```php
/**
 * Example GET route
 *
 * @param  ServerRequestInterface $request  PSR-7 request
 * @param  ResponseInterface      $response  PSR-7 response
 * @param  array                  $args Route parameters
 *
 * @return ResponseInterface
 */
$app->get('/foo', function (Request $request, Response $response, $args) {
    if ($this->has('myService')) {
        $myService = $this->get('myService');
    }
    return $response;
});
```

## Configuring the application via a container



In case you want to create the `App` with dependencies already defined in your container, you can use the `AppFactory::createFromContainer()` method.

**Example**

```php
<?php

use App\Factory\MyResponseFactory;
use DI\Container;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\Factory\AppFactory;

require_once __DIR__ . '/../vendor/autoload.php';

// Create Container using PHP-DI
$container = new Container();

// Add custom response factory
$container->set(ResponseFactoryInterface::class, function (ContainerInterface $container) {
    return new MyResponseFactory(...);
});

// Configure the application via container
$app = AppFactory::createFromContainer($container);

// ...

$app->run();
```

Supported `App` dependencies are:

- Psr\Http\Message\ResponseFactoryInterface
- Slim\Interfaces\CallableResolverInterface
- Slim\Interfaces\RouteCollectorInterface
- Slim\Interfaces\RouteResolverInterface
- Slim\Interfaces\MiddlewareDispatcherInterface



Slim 4的容器依赖注入在构建Controller的时候需要当着__constract的参数传入，没找到其他优雅的方式

```php
/**
 * @param LoggerInterface $logger
 * @param UserRepository  $userRepository
 */
public function __construct(LoggerInterface $logger, UserRepository $userRepository,Container $container)
{
    parent::__construct($logger);
    $this->userRepository = $userRepository;
    $this->container = $container;
}
```

应用层 Slim的例子是一些Action，绑定到了容器的定义类型属性下面

```php
class ReflectionBasedAutowiring implements DefinitionSource, Autowiring
{
    public function autowire(string $name, ObjectDefinition $definition = null)
    {
        $className = $definition ? $definition->getClassName() : $name;

        if (!class_exists($className) && !interface_exists($className)) {
            return $definition;
        }

        $definition = $definition ?: new ObjectDefinition($name);

        // Constructor
        $class = new \ReflectionClass($className);
        $constructor = $class->getConstructor();
        if ($constructor && $constructor->isPublic()) {
            $constructorInjection = MethodInjection::constructor($this->getParametersDefinition($constructor));
            $definition->completeConstructorInjection($constructorInjection);
        }
      //这里解析了调用的类的构造函数

        return $definition;
    }

    public function getDefinition(string $name)
    {
        return $this->autowire($name);
    }
    ...
 }
```



Tips: Slim 响应response之前 清空了缓冲输出，测试环境调试可以配置此处

```php
<?php
declare(strict_types=1);

namespace App\Application\ResponseEmitter;

use Psr\Http\Message\ResponseInterface;
use Slim\ResponseEmitter as SlimResponseEmitter;

class ResponseEmitter extends SlimResponseEmitter
{
    /**
     * {@inheritdoc}
     */
    public function emit(ResponseInterface $response): void
    {
        // This variable should be set to the allowed host from which your API can be accessed with
        $origin = isset($_SERVER['HTTP_ORIGIN']) ? $_SERVER['HTTP_ORIGIN'] : '';

        $response = $response
            ->withHeader('Access-Control-Allow-Credentials', 'true')
            ->withHeader('Access-Control-Allow-Origin', $origin)
            ->withHeader('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type, Accept, Origin, Authorization')
            ->withHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS')
            ->withHeader('Cache-Control', 'no-store, no-cache, must-revalidate, max-age=0')
            ->withAddedHeader('Cache-Control', 'post-check=0, pre-check=0')
            ->withHeader('Pragma', 'no-cache');
      //测试环境可以根据配置决定是否注释掉已获得打印的调试数据
       // if (ob_get_contents()) {
       //     ob_clean();
       //}
        parent::emit($response);
    }
}
```



Slim DI\Definition\Definition 这个库处理了避免类多次实例化的逻辑...