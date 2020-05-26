# Slim 4 Documentation

## Welcome

> Slim is a PHP micro framework that helps you quickly write simple yet powerful web applications and APIs. At its core, Slim is a dispatcher that receives an HTTP request, invokes an appropriate callback routine, and returns an HTTP response. That’s it.

Slim 是一个轻型的php框架，旨在帮助你快速构建强大的web application 和 APIs。Slim的核心是一个调度器，接受HTTP请求，唤醒合适的回调逻辑处理之后返回一个HTTP 响应。这就是全部。

Slim 核心代码提供的功能有限，比较适合轻型项目和接口类型的项目。

## What’s the point?

 [Symfony](https://symfony.com/) or [Laravel](https://laravel.com/) 是非常好的框架，但是对于轻型项目没必要。正常的一个http请求从客户端发起到服务器接受，经过路由处理解析成对应的controller 和 action 以及参数，执行完成业务逻辑之后返回一个HTTP响应给到客户端，这就是整个基本的核心逻辑。其中还有很多逻辑，比如登录态校验，参数校验，拦截器，权限校验等等大型框架集成的东西，在一个微型的项目中可能用不到。所以Slim 秉承实现最小的原则，仅仅实现了核心的路由解析调度，中间件调度器的逻辑，其他逻辑开发者可以自行扩展。Slim生态中有很多开源的中间件，基本都遵循prs-7,prs-15等接口规范，开箱即用。开发者可以自己在核心之外，中间件的链路上集成对请求和响应的处理逻辑。框架之外，Slim并不限制项目的架构模式，可以单纯的函数，也可以是简易的MVC，也可以是比较复杂的DDD架构（Slim create project 就是推荐这种架构模式）。开发者可以灵活地调整合适的架构。

​	**并没有什么完美的架构，恰当及完美**

Slim的开发者多次强调了这段话，可以细品

<font size="5">***At its core, Slim is a dispatcher that receives an HTTP request, invokes an appropriate callback routine, and returns an HTTP response. That’s it***</font>

> 原文
>
> Slim is an ideal tool to create APIs that consume, repurpose, or publish data. Slim is also a great tool for rapid prototyping. Heck, you can even build full-featured web applications with user interfaces. More importantly, Slim is super fast and has very little code. In fact, you can read and understand its source code in only an afternoon!
>
> You don’t always need a kitchen-sink solution like [Symfony](https://symfony.com/) or [Laravel](https://laravel.com/). These are great tools, for sure. But they are often overkill. Instead, Slim provides only a minimal set of tools that do what you need and nothing else.

## How does it work?

> First, you need a web server like Nginx or Apache. You should [configure your web server](https://www.slimframework.com/docs/v4/start/web-servers.html) so that it sends all appropriate requests to one “front-controller” PHP file. You instantiate and run your Slim app in this PHP file.
>
> A Slim app contains routes that respond to specific HTTP requests. Each route invokes a callback and returns an HTTP response. To get started, you first instantiate and configure the Slim application. Next, you define your application routes. Finally, you run the Slim application. It’s that easy. Here’s an example application:

本地开发或者调试可以使用browser或者IDE的HTTP-Client + 内置的web服务器

```bash
php -S  
```

推荐部署nginx服务器，可以解决后端源码，前端静态资源目录，路径设置等可能引起的问题。

类似所有的框架一样，必然有一个入口文件，比如index.php 整个请求由此开始...

Slim应用主要就是处理请求并生成一个响应。每一个路由唤醒的调用逻辑都接受一个Request的类，处理完成之后返回一个Respose的类。

通常index.php 实现了以下逻辑:

1 定义常量或者引入常量文件

2 引入自动加载文件，可以多个

> ​	按加载的顺序查找，相同的文件一般用namespace区分

3 如果需要，实例化容器，将初始化的类或者参数绑定到容器

4 实例化Slim App

5 引入或者设置 当前app的路由配置

6 引入或者设置中间件配置文件

除了5,6需要绑定app必须在实例化之后，很多其他的逻辑，比如日志，渲染器等等的使用视情况而定



```php
<?php
use Slim\Factory\AppFactory;
use DI\Container;

//自动加载
require __DIR__ . '/vendor/autoload.php';
require __DIR__.'/application/config/autoload.php';

//session_start();

defined("ROOT") or  define("ROOT",__DIR__);

//实例化容器
$container = new Container();
AppFactory::setContainer($container);

$app = AppFactory::create();

//加载路由配置
require __DIR__ . '/application/config/routes.php';

//加载中间件配置
require __DIR__ . '/application/config/middleware-test.php';


$app->run();
```

> 官方的例子和说明

```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

/**
 * Instantiate App
 *
 * In order for the factory to work you need to ensure you have installed
 * a supported PSR-7 implementation of your choice e.g.: Slim PSR-7 and a supported
 * ServerRequest creator (included with Slim PSR-7)
 */
$app = AppFactory::create();

// Add Routing Middleware
$app->addRoutingMiddleware();

/**
 * Add Error Handling Middleware
 *
 * @param bool $displayErrorDetails -> Should be set to false in production
 * @param bool $logErrors -> Parameter is passed to the default ErrorHandler
 * @param bool $logErrorDetails -> Display error details in error log
 * which can be replaced by a callable of your choice.
 
 * Note: This middleware should be added last. It will not handle any exceptions/errors
 * for middleware added after it.
 */
$errorMiddleware = $app->addErrorMiddleware(true, true, true);

// Define app routes
$app->get('/hello/{name}', function (Request $request, Response $response, $args) {
    $name = $args['name'];
    $response->getBody()->write("Hello, $name");
    return $response;
});

// Run app
$app->run();
```

## Request and response

 官方注解，比较能反映Slim的设计哲学，明白这个之后，理解源码就比较轻松了

> When you build a Slim app, you are often working directly with Request and Response objects. These objects represent the actual HTTP request received by the web server and the eventual HTTP response returned to the client.
>
> Every Slim app route is given the current Request and Response objects as arguments to its callback routine. These objects implement the popular [PSR-7](https://www.slimframework.com/docs/v4/concepts/value-objects.html) interfaces. The Slim app route can inspect or manipulate these objects as necessary. Ultimately, each Slim app route **MUST** return a PSR-7 Response object.