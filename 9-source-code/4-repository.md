# Repository



```php
<?php
declare(strict_types=1);

use App\Domain\User\UserRepository;
use App\Infrastructure\Persistence\User\InMemoryUserRepository;
use DI\ContainerBuilder;

return function (ContainerBuilder $containerBuilder) {
    // Here we map our UserRepository interface to its in memory implementation
    $containerBuilder->addDefinitions([
        UserRepository::class => \DI\autowire(InMemoryUserRepository::class),
    ]);
};
```

这是Slim  create project给的引用模型仓库的示例

对于匿名函数，传入container实例，并将模型对应仓库类绑定到容器上

在应用层的构造函数传入，仓库依赖就可以在应用层使用

```php
abstract class UserAction extends Action
{
    /**
     * @var UserRepository
     */
    protected $userRepository;

    /**
     * @param LoggerInterface $logger
     * @param UserRepository  $userRepository
     */
    public function __construct(LoggerInterface $logger, UserRepository $userRepository)
    {
        parent::__construct($logger);
        $this->userRepository = $userRepository;
    }
}
```



仓库依赖写入了配置文件，由反射自动解析并加载

如果模型过多，仓库配置文件会很臃肿

