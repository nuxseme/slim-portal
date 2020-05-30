# 自动加载

## composer 配置加载

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"  //配置目录映射
    }
},
"autoload-dev": {
    "psr-4": {
        "Tests\\": "tests/"
    }
},
"scripts": {
    "start": "php -S localhost:8080 -t public",
    "test": "phpunit"
}
```



```php
Composer\Autoload\ClassLoader Object
(
    [prefixLengthsPsr4:Composer\Autoload\ClassLoader:private] => Array
        (
            [p] => Array
                (
                    [phpDocumentor\Reflection\] => 25
                )

            [Z] => Array
                (
                    [Zend\Diactoros\] => 15
                )
							.
            	.
            	.
            [C] => Array
                (
                    [Chadicus\Slim\OAuth2\Middleware\] => 32
                    [Chadicus\Slim\OAuth2\Http\] => 26
                    [Chadicus\Psr\Middleware\] => 24
                )

            [A] => Array
                (
                    [App\] => 4
                )

        )
         [prefixDirsPsr4:Composer\Autoload\ClassLoader:private] => Array
        (
            [phpDocumentor\Reflection\] => Array
                (
                    [0] => /Users/nuxse/tms/vendor/composer/../phpdocumentor/reflection-common/src
                    [1] => /Users/nuxse/tms/vendor/composer/../phpdocumentor/reflection-docblock/src
                    [2] => /Users/nuxse/tms/vendor/composer/../phpdocumentor/type-resolver/src
                )
						.
           	.
						.
            [App\] => Array
                (
                    [0] => /Users/nuxse/tms/vendor/composer/../../src
                )

        )


```

1 配置composer 

2 composer install

3 入口文件引用vender 下的composer



# Autoload 文件

php 可以配置多个autoload 注册函数，按注册的顺序查找

类的查找逻辑就是已知class name 和 namespace ,通过建立的映射逻辑找到文件并包含进来

范例如下:

```php

spl_autoload_register(
    function (string $class) {
        static $classes = null;
        if ($classes === null) {
            $classes = [
            ];
        }
        $map_class = $class;
        if (isset($classes[$map_class])) {
            $file = __DIR__ .DIRECTORY_SEPARATOR. $classes[$map_class].'.php';
        }else{
            //处理命名空间的映射关系
            $namespace_class = str_replace('\\',DIRECTORY_SEPARATOR,$class);

            $file = ROOT.DIRECTORY_SEPARATOR.$namespace_class.'.php';
        }
        if(file_exists($file)) {
            require $file;
        }
    }
);

```

如果class name 和 namespce 跟vendor里面冲突可能会导致加载到错误的文件