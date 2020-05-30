# Route

路由的解析到实例化对应的应用层回调流程

```php
大致流程

应用入口 App->handle()
中间件调度器 MiddlewareDispatcher->handle()
最靠近核心的路由中间件  路由解决方案
App 构造函数
$this->routeResolver = $routeResolver ?? new RouteResolver($this->routeCollector);
$routeRunner = new RouteRunner($this->routeResolver, $this->routeCollector->getRouteParser(), $this);

if (!$middlewareDispatcher) {
  $middlewareDispatcher = new MiddlewareDispatcher($routeRunner, $this->callableResolver, $container);
} else {
  $middlewareDispatcher->seedMiddlewareStack($routeRunner);
}

$this->middlewareDispatcher = $middlewareDispatcher;


RouteRunner handle
public function handle(ServerRequestInterface $request): ResponseInterface
    {
        // If routing hasn't been done, then do it now so we can dispatch
        if ($request->getAttribute(RouteContext::ROUTING_RESULTS) === null) {
            $routingMiddleware = new RoutingMiddleware($this->routeResolver, $this->routeParser);
            $request = $routingMiddleware->performRouting($request);
        }

        if ($this->routeCollectorProxy !== null) {
            $request = $request->withAttribute(
                RouteContext::BASE_PATH,
                $this->routeCollectorProxy->getBasePath()
            );
        }

        /** @var Route $route */
        $route = $request->getAttribute(RouteContext::ROUTE);
        return $route->run($request);
    }

Route run()
   /**
     * {@inheritdoc}
     */
    public function run(ServerRequestInterface $request): ResponseInterface
    {
        if (!$this->groupMiddlewareAppended) {
            $this->appendGroupMiddlewareToRoute();
        }

        return $this->middlewareDispatcher->handle($request);
    }

Route 也具备中间件概念，最靠近路由器的中间件路由解决中间件
   public function __construct(
        array $methods,
        string $pattern,
        $callable,
        ResponseFactoryInterface $responseFactory,
        CallableResolverInterface $callableResolver,
        ?ContainerInterface $container = null,
        ?InvocationStrategyInterface $invocationStrategy = null,
        array $groups = [],
        int $identifier = 0
    ) {
        $this->methods = $methods;
        $this->pattern = $pattern;
        $this->callable = $callable;
        $this->responseFactory = $responseFactory;
        $this->callableResolver = $callableResolver;
        $this->container = $container;
        $this->invocationStrategy = $invocationStrategy ?? new RequestResponse();
        $this->groups = $groups;
        $this->identifier = 'route' . $identifier;
        $this->middlewareDispatcher = new MiddlewareDispatcher($this, $callableResolver, $container);
    }

Route 的handle 
  if ($this->callableResolver instanceof AdvancedCallableResolverInterface) {
            $callable = $this->callableResolver->resolveRoute($this->callable);
        } else {
            $callable = $this->callableResolver->resolve($this->callable);
        }
        $strategy = $this->invocationStrategy;
返回路由器匹配的回调
  
  
  CallableResolver 
   private function resolveByPredicate($toResolve, callable $predicate, string $defaultMethod): callable
    {
  			//匿名函数绑定到容器上
        if (is_callable($toResolve)) {
            return $this->bindToContainer($toResolve);
        }
        $resolved = $toResolve;
        if ($predicate($toResolve)) {
            $resolved = [$toResolve, $defaultMethod];
        }
        if (is_string($toResolve)) {
            [$instance, $method] = $this->resolveSlimNotation($toResolve);
            if ($predicate($instance) && $method === null) {
                $method = $defaultMethod;
            }
            $resolved = [$instance, $method ?? '__invoke'];
        }
        $callable = $this->assertCallable($resolved, $toResolve);
        return $this->bindToContainer($callable);
    }
//匿名函数挂载到容器上  内部的this指向容器
private function bindToContainer(callable $callable): callable
    {
        if (is_array($callable) && $callable[0] instanceof Closure) {
            $callable = $callable[0];
        }
        if ($this->container && $callable instanceof Closure) {
            $callable = $callable->bindTo($this->container);
        }
        return $callable;
    }

 //非匿名函数 
 private function resolveSlimNotation(string $toResolve): array
    {
        preg_match(CallableResolver::$callablePattern, $toResolve, $matches);
        [$class, $method] = $matches ? [$matches[1], $matches[2]] : [$toResolve, null];
				//非匿名函数实例由容器获取
        if ($this->container && $this->container->has($class)) {
            $instance = $this->container->get($class);
        } else {
            if (!class_exists($class)) {
                throw new RuntimeException(sprintf('Callable %s does not exist', $class));
            }
            $instance = new $class($this->container);
        }
        return [$instance, $method];
    }

public function has($name)
    {
        if (! is_string($name)) {
            throw new InvalidArgumentException(sprintf(
                'The name parameter must be of type string, %s given',
                is_object($name) ? get_class($name) : gettype($name)
            ));
        }

        if (array_key_exists($name, $this->resolvedEntries)) {
            return true;
        }

        $definition = $this->getDefinition($name);
        if ($definition === null) {
            return false;
        }

        return $this->definitionResolver->isResolvable($definition);
    }
private function getDefinition($name)
    {
        // Local cache that avoids fetching the same definition twice
        if (!array_key_exists($name, $this->fetchedDefinitions)) {
            $this->fetchedDefinitions[$name] = $this->definitionSource->getDefinition($name);
        }

        return $this->fetchedDefinitions[$name];
    }
SourceChain 
public function getDefinition(string $name, int $startIndex = 0)
    {
        $count = count($this->sources);
        for ($i = $startIndex; $i < $count; ++$i) {
            $source = $this->sources[$i];

            $definition = $source->getDefinition($name);

            if ($definition) {
                if ($definition instanceof ExtendsPreviousDefinition) {
                    $this->resolveExtendedDefinition($definition, $i);
                }

                return $definition;
            }
        }

        return null;
    }
ReflectionBasedAutowiring 
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

        return $definition;
    }

  ObjectCreator 
  此处解析配置文件对应的应用层的回调函数，根据类的构造函数参数依赖实例化类
  private function createInstance(ObjectDefinition $definition, array $parameters)
    {
        // Check that the class is instantiable
        if (! $definition->isInstantiable()) {
            // Check that the class exists
            if (! $definition->classExists()) {
                throw InvalidDefinition::create($definition, sprintf(
                    'Entry "%s" cannot be resolved: the class doesn\'t exist',
                    $definition->getName()
                ));
            }

            throw InvalidDefinition::create($definition, sprintf(
                'Entry "%s" cannot be resolved: the class is not instantiable',
                $definition->getName()
            ));
        }

        $classname = $definition->getClassName();
        $classReflection = new ReflectionClass($classname);

        $constructorInjection = $definition->getConstructorInjection();

        try {
            $args = $this->parameterResolver->resolveParameters(
                $constructorInjection,
                $classReflection->getConstructor(),
                $parameters
            );

            $object = new $classname(...$args);

            $this->injectMethodsAndProperties($object, $definition);
        } catch (NotFoundExceptionInterface $e) {
            throw new DependencyException(sprintf(
                'Error while injecting dependencies into %s: %s',
                $classReflection->getName(),
                $e->getMessage()
            ), 0, $e);
        } catch (InvalidDefinition $e) {
            throw InvalidDefinition::create($definition, sprintf(
                'Entry "%s" cannot be resolved: %s',
                $definition->getName(),
                $e->getMessage()
            ));
        }

        return $object;
    }
```



```php
<?php
declare(strict_types=1);

use App\Application\Actions\User\ListUsersAction;
use App\Application\Actions\User\ViewUserAction;
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\App;
use Slim\Interfaces\RouteCollectorProxyInterface as Group;

return function (App $app) {
    $app->options('/{routes:.*}', function (Request $request, Response $response) {
        // CORS Pre-Flight OPTIONS Request Handler
        return $response;
    });

    $app->get('/', function (Request $request, Response $response) {
        $response->getBody()->write('Hello world!');
        return $response;
    });

    $app->group('/users', function (Group $group) {
        $group->get('', ListUsersAction::class);
        $group->get('/{id}', ViewUserAction::class);
    });

    $app->group('/user',function(\Slim\Interfaces\RouteCollectorProxyInterface $group){
        $group->post('/login', \App\Application\Controller\UserController::class.':login');
        $group->get('/logout', \App\Application\Controller\UserController::class.':logout');
    });

    $app->group('/ship',function(\Slim\Interfaces\RouteCollectorProxyInterface $group){
        $group->get('/list', \App\Application\Controller\ShipController::class.':list');
        $group->post('/createTable', \App\Application\Controller\ShipController::class.':createTable');
    });

    $app->get('/user/info[/{id}]', \App\Application\Controller\UserController::class.':info');

    $app->get('/menu', \App\Application\Controller\MenuController::class.':list');

    $app->get('/clear', \App\Application\Controller\ClearController::class.':handle');


    $app->post('/rating/handle', \App\Application\Controller\RatingController::class.':handle');


    $app->get('/pdf/download', \App\Application\Controller\PdfController::class.':download');

    $app->get('/pdf/preview', \App\Application\Controller\PdfController::class.':preview');

    $app->get('/log/info', \App\Application\Controller\LogController::class.':info');
};

```



路由分组 

```php
$app->group('/users', function (Group $group) {
        $group->get('', ListUsersAction::class);
        $group->get('/{id}', ViewUserAction::class);
    });
```

路由参数

```php
$app->get('/user/info[/{id}]', \App\Application\Controller\UserController::class.':info');

```

针对单个或者分组都可以挂载中间件



对于中间件 除了某个路由之外的绑定

## 检测uri

```php
$sessionId = $request->getCookieParams()['PHPSESSID'] ?: null;

$session = $_SESSION[$sessionId] ?? null;

if($request->getUri()->getPath() == '/user/login') {
return $handler->handle($request);
}
//比如对于不是登陆接口的请求都需要登录态检测，可以在中间件获取当前的path 
if (empty($session)) {
return $this->unAuthorization();
```

可以配置一个ignore  url 

## Access+IgnoreAcess 

```php
$app->group('',function(Group $group){
        $group->get('/log/info', \App\Application\Controller\LogController::class.':info')->setName('this is set name');
    })->addMiddleware(new  \App\Application\Middleware\AccessMiddleware());

    $app->group('',function(Group $group){
        $group->get('/log/info', \App\Application\Controller\LogController::class.':info')->setName('this is set name');
    })->addMiddleware(new  \App\Application\Middleware\IgnoreAccessMiddleware());
```

可以在Access外层配置一个IgnoreAccess中间件，分组的时候可以对于不需要access的分组到一组并加载IgnoreAccessMiddleware 

```php
class IgnoreAccessMiddleware implements Middleware
{
    /**
     * {@inheritdoc}
     */
    public function process(Request $request, RequestHandler $handler): Response
    {
        echo  __CLASS__,__METHOD__;
        $request->withAttribute('ignore_access',true);
				//设置属性 在Access阶段检测
        return $handler->handle($request);
    }
}
```



2中方案都可以，暂时没找到其他优雅的方案 