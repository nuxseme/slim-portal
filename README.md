# Introduction

Silm 是一个php轻框架，旨在提供快速开发出强大的web应用和接口



```php
<?php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

require __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

$app->get('/hello/{name}', function (Request $request, Response $response, array $args) {
    $name = $args['name'];
    $response->getBody()->write("Hello, $name");
    return $response;
});

$app->run();
```



# 安装

初次创建项目 

```bash
$ composer create-project slim/slim-skeleton:dev-master [my-app-name]
```

自建项目目录之后直接compose 引入

```bash
$ composer  require  slim/slim
```



> 学习或者初始项目建议按第一种方式安装，里面已经包含了比较规范的目录结构，参照DDD模型设计，还包含了测试用例，路由示例，中间件示例，基础设置持久化示例等等
>

# 特色

### HTTP Router 

Slim provides a fast and powerful router that maps route callbacks to specific HTTP request methods and URIs. It supports parameters and pattern matching.

> #### HTTP 路由
>
> Slim 提供了一个强大快速的路由器，能快速映射HTTP请求和URIs，支持参数设置和模式匹配

### Middleware

Build your application with concentric middleware to tweak the HTTP request and response objects around your Slim app.

> #### 中间件
>
> 围绕http请求和响应的中间件层来构建应用

### PSR-7 Support

Slim supports any PSR-7 HTTP message implementation so you may inspect and manipulate HTTP message method, status, URI, headers, cookies, and body.

> #### PSR-7 规范支持
>
> Slim 广泛实现了PSR-7设计的HTTPP相关的接口，使其可以方便使用实现接口的第三方库



### Dependency Injection

Slim supports dependency injection so you have complete control of your external tools. Use any Container-Interop container.

> #### 依赖注入
>
> Slim 提供了容器，方便依赖注入



# 核心架构图示

<img src="assets/middleware.png" style="zoom:50%;" />

# 总结



> Slim 相对其他的轻框架比较更加高效，扩展能力更强，容易集成第三方类库的使用，相对YII这种大型框架比较灵活，框架束缚比较少，相应的提供的能力没那么强。Slim更多处理简易项目和API接口项目比较方便，中间件的灵活使用也使得应用结构简单，更加模块化。