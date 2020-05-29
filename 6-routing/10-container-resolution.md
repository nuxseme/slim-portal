# Container Resolution



You are not limited to defining a function for your routes. In Slim there are a few different ways to define your route action functions.

In addition to a function, you may use:

- container_key:method
- Class:method
- Class implementing `__invoke()` method
- container_key

This functionality is enabled by Slim’s Callable Resolver Class. It translates a string entry into a function call. Example:

```php
$app->get('/', '\HomeController:home');
```

Alternatively, you can take advantage of PHP’s `::class` operator which works well with IDE lookup systems and produces the same result:

```php
$app->get('/', \HomeController::class . ':home');
```

In this code above we are defining a `/` route and telling Slim to execute the `home()` method on the `HomeController` class.

Slim first looks for an entry of `HomeController` in the container, if it’s found it will use that instance otherwise it will call it’s constructor with the container as the first argument. Once an instance of the class is created it will then call the specified method using whatever Strategy you have defined.

### Registering a controller with the container

Create a controller with the `home` action method. The constructor should accept the dependencies that are required. For example:

```php
<?php

class HomeController
{
    protected $view;

    public function __construct(\Slim\Views\Twig $view) {
        $this->view = $view;
    }
    
    public function home($request, $response, $args) {
      // your code here
      // use $this->view to render the HTML
      return $response;
    }
}
```

Create a factory in the container that instantiates the controller with the dependencies:

```php
$container = $app->getContainer();
$container->set('HomeController', function (ContainerInterface $c) {
    $view = $c->get('view'); // retrieve the 'view' from the container
    return new HomeController($view);
});
```

This allows you to leverage the container for dependency injection and so you can inject specific dependencies into the controller.

### Allow Slim to instantiate the controller

Alternatively, if the class does not have an entry in the container, then Slim will pass the container’s instance to the constructor. You can construct controllers with many actions instead of an invokable class which only handles one action.

```php
<?php
use Psr\Container\ContainerInterface;

class HomeController
{
   protected $container;

   // constructor receives container instance
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }

   public function home($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }

   public function contact($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

You can use your controller methods like so.

```php
$app->get('/', \HomeController::class . ':home');
$app->get('/contact', \HomeController::class . ':contact');
```

### Using an invokable class

You do not have to specify a method in your route callable and can just set it to be an invokable class such as:

```php
<?php
use Psr\Container\ContainerInterface;

class HomeAction
{
   protected $container;

   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }

   public function __invoke($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

You can use this class like so.

```php
$app->get('/', \HomeAction::class);
```

Again, as with controllers, if you register the class name with the container, then you can create a factory and inject just the specific dependencies that you require into your action class.



官方建议的controller 使用container的方式