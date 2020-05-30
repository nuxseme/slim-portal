

# Container



```php
<?php

declare(strict_types=1);

namespace DI;

use DI\Compiler\Compiler;
use DI\Definition\Source\AnnotationBasedAutowiring;
use DI\Definition\Source\DefinitionArray;
use DI\Definition\Source\DefinitionFile;
use DI\Definition\Source\DefinitionSource;
use DI\Definition\Source\NoAutowiring;
use DI\Definition\Source\ReflectionBasedAutowiring;
use DI\Definition\Source\SourceCache;
use DI\Definition\Source\SourceChain;
use DI\Proxy\ProxyFactory;
use InvalidArgumentException;
use Psr\Container\ContainerInterface;

/**
 * Helper to create and configure a Container.
 *
 * With the default options, the container created is appropriate for the development environment.
 *
 * Example:
 *
 *     $builder = new ContainerBuilder();
 *     $container = $builder->build();
 *
 * @api
 *
 * @since  3.2
 * @author Matthieu Napoli <matthieu@mnapoli.fr>
 */
class ContainerBuilder
{
    /**
     * Name of the container class, used to create the container.
     * @var string
     */
    private $containerClass;

    /**
     * Name of the container parent class, used on compiled container.
     * @var string
     */
    private $containerParentClass;

    /**
     * @var bool
     */
    private $useAutowiring = true; //默认采用反射自动装配器

    /**
     * @var bool
     */
    private $useAnnotations = false;

    /**
     * @var bool
     */
    private $ignorePhpDocErrors = false;

    /**
     * If true, write the proxies to disk to improve performances.
     * @var bool
     */
    private $writeProxiesToFile = false;

    /**
     * Directory where to write the proxies (if $writeProxiesToFile is enabled).
     * @var string|null
     */
    private $proxyDirectory;

    /**
     * If PHP-DI is wrapped in another container, this references the wrapper.
     * @var ContainerInterface
     */
    private $wrapperContainer;

    /**
     * @var DefinitionSource[]|string[]|array[]
     */
    private $definitionSources = [];

    /**
     * Whether the container has already been built.
     * @var bool
     */
    private $locked = false;

    /**
     * @var string|null
     */
    private $compileToDirectory;

    /**
     * @var bool
     */
    private $sourceCache = false;

    /**
     * @var string
     */
    protected $sourceCacheNamespace;

    /**
     * Build a container configured for the dev environment.
     */
    public static function buildDevContainer() : Container
    {
        return new Container;
    }

    /**
     * @param string $containerClass Name of the container class, used to create the container.
     */
    public function __construct(string $containerClass = Container::class)
    {
        $this->containerClass = $containerClass;
    }

    /**
     * Build and return a container.
     *
     * @return Container
     */
    public function build()
    {
        $sources = array_reverse($this->definitionSources);

        if ($this->useAnnotations) {
            $autowiring = new AnnotationBasedAutowiring($this->ignorePhpDocErrors);
            $sources[] = $autowiring;
        } elseif ($this->useAutowiring) {
          //这里解析链 默认采用反射
            $autowiring = new ReflectionBasedAutowiring;
            $sources[] = $autowiring;
        } else {
            $autowiring = new NoAutowiring;
        }

        $sources = array_map(function ($definitions) use ($autowiring) {
            if (is_string($definitions)) {
                // File
                return new DefinitionFile($definitions, $autowiring);
            } elseif (is_array($definitions)) {
                return new DefinitionArray($definitions, $autowiring);
            }

            return $definitions;
        }, $sources);
        $source = new SourceChain($sources);

        // Mutable definition source
        $source->setMutableDefinitionSource(new DefinitionArray([], $autowiring));

        if ($this->sourceCache) {
            if (!SourceCache::isSupported()) {
                throw new \Exception('APCu is not enabled, PHP-DI cannot use it as a cache');
            }
            // Wrap the source with the cache decorator
            $source = new SourceCache($source, $this->sourceCacheNamespace);
        }

        $proxyFactory = new ProxyFactory(
            $this->writeProxiesToFile,
            $this->proxyDirectory
        );

        $this->locked = true;

        $containerClass = $this->containerClass;

        if ($this->compileToDirectory) {
            $compiler = new Compiler($proxyFactory);
            $compiledContainerFile = $compiler->compile(
                $source,
                $this->compileToDirectory,
                $containerClass,
                $this->containerParentClass,
                $this->useAutowiring || $this->useAnnotations
            );
            // Only load the file if it hasn't been already loaded
            // (the container can be created multiple times in the same process)
            if (!class_exists($containerClass, false)) {
                require $compiledContainerFile;
            }
        }

        return new $containerClass($source, $proxyFactory, $this->wrapperContainer);
    }

   .
   .
}

```

挂在到App上

```php
AppFactory::setContainer($container);
$app = AppFactory::create();
```



```php
<?php

declare(strict_types=1);

namespace DI;

use DI\Definition\Definition;
use DI\Definition\Exception\InvalidDefinition;
use DI\Definition\FactoryDefinition;
use DI\Definition\Helper\DefinitionHelper;
use DI\Definition\InstanceDefinition;
use DI\Definition\ObjectDefinition;
use DI\Definition\Resolver\DefinitionResolver;
use DI\Definition\Resolver\ResolverDispatcher;
use DI\Definition\Source\DefinitionArray;
use DI\Definition\Source\MutableDefinitionSource;
use DI\Definition\Source\ReflectionBasedAutowiring;
use DI\Definition\Source\SourceChain;
use DI\Definition\ValueDefinition;
use DI\Invoker\DefinitionParameterResolver;
use DI\Proxy\ProxyFactory;
use InvalidArgumentException;
use Invoker\Invoker;
use Invoker\InvokerInterface;
use Invoker\ParameterResolver\AssociativeArrayResolver;
use Invoker\ParameterResolver\Container\TypeHintContainerResolver;
use Invoker\ParameterResolver\DefaultValueResolver;
use Invoker\ParameterResolver\NumericArrayResolver;
use Invoker\ParameterResolver\ResolverChain;
use Psr\Container\ContainerInterface;

/**
 * Dependency Injection Container.
 *
 * @api
 *
 * @author Matthieu Napoli <matthieu@mnapoli.fr>
 */
class Container implements ContainerInterface, FactoryInterface, InvokerInterface
{
    /**
     * Map of entries that are already resolved.
     * @var array
     */
    protected $resolvedEntries = [];

    /**
     * @var MutableDefinitionSource
     */
    private $definitionSource;

    /**
     * @var DefinitionResolver
     */
    private $definitionResolver;

    /**
     * Map of definitions that are already fetched (local cache).
     *
     * @var (Definition|null)[]
     */
    private $fetchedDefinitions = [];

    /**
     * Array of entries being resolved. Used to avoid circular dependencies and infinite loops.
     * @var array
     */
    protected $entriesBeingResolved = [];

    /**
     * @var InvokerInterface|null
     */
    private $invoker;

    /**
     * Container that wraps this container. If none, points to $this.
     *
     * @var ContainerInterface
     */
    protected $delegateContainer;

    /**
     * @var ProxyFactory
     */
    protected $proxyFactory;

    /**
     * Use `$container = new Container()` if you want a container with the default configuration.
     *
     * If you want to customize the container's behavior, you are discouraged to create and pass the
     * dependencies yourself, the ContainerBuilder class is here to help you instead.
     *
     * @see ContainerBuilder
     *
     * @param ContainerInterface $wrapperContainer If the container is wrapped by another container.
     */
    public function __construct(
        MutableDefinitionSource $definitionSource = null,
        ProxyFactory $proxyFactory = null,
        ContainerInterface $wrapperContainer = null
    ) {
        $this->delegateContainer = $wrapperContainer ?: $this;

        $this->definitionSource = $definitionSource ?: $this->createDefaultDefinitionSource();
        $this->proxyFactory = $proxyFactory ?: new ProxyFactory(false);
        $this->definitionResolver = new ResolverDispatcher($this->delegateContainer, $this->proxyFactory);

        // Auto-register the container
        $this->resolvedEntries = [
            self::class => $this,
            ContainerInterface::class => $this->delegateContainer,
            FactoryInterface::class => $this,
            InvokerInterface::class => $this,
        ];
    }

    /**
     * Returns an entry of the container by its name.
     *
     * @param string $name Entry name or a class name.
     *
     * @throws DependencyException Error while resolving the entry.
     * @throws NotFoundException No entry found for the given name.
     * @return mixed
     */
    public function get($name)
    {
      //根据类名获取实例
        // If the entry is already resolved we return it
        if (isset($this->resolvedEntries[$name]) || array_key_exists($name, $this->resolvedEntries)) {
            return $this->resolvedEntries[$name];
        }

        $definition = $this->getDefinition($name);
        if (! $definition) {
            throw new NotFoundException("No entry or class found for '$name'");
        }

        $value = $this->resolveDefinition($definition);

        $this->resolvedEntries[$name] = $value;

        return $value;
    }

    /**
     * @param string $name
     *
     * @return Definition|null
     */
    private function getDefinition($name)
    {
       //定义的解析链解析
        // Local cache that avoids fetching the same definition twice
        if (!array_key_exists($name, $this->fetchedDefinitions)) {
            $this->fetchedDefinitions[$name] = $this->definitionSource->getDefinition($name);
        }

        return $this->fetchedDefinitions[$name];
    }

   
    /**
     * Test if the container can provide something for the given name.
     *
     * @param string $name Entry name or a class name.
     *
     * @throws InvalidArgumentException The name parameter must be of type string.
     * @return bool
     */
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

   .
   .
   .

    private function createDefaultDefinitionSource() : SourceChain
    {
        $source = new SourceChain([new ReflectionBasedAutowiring]);
        $source->setMutableDefinitionSource(new DefinitionArray([], new ReflectionBasedAutowiring));

        return $source;
    }
}

```



## 容器与对象缓存器

容器用来解决实例化加载的问题，是一个全局的对象树 

可以预先将对象实例化之后，句柄挂载到容器上，在应用层就可以直接使用 

容器是一个对象的挂载树，所以在对象实例查找，获取，设置的过程需要判断当前对象是否已经存在，避免多次实例化

容器自身也是一个对象，存储于内存中，所以在使用过程中避免了对象的多次实例化销毁的逻辑

容器加载如果是常驻内存的模型，需要防止对象的加载过多导致内存不断持续升高

容器挂载对象，不能代替缓存，比如可以把Logger 挂载到容器，但是不能将UserId = 100的User这个模型挂载到容器。容器跟对象存储器有很大区别

容器挂载的通常是框架，服务，基础设施等相关的实例化的对象，常常在初始请求的阶段或者预热阶段就实例化了

对象存储器主要是在整个项目的生命周期用于缓存对象，非常驻模式，常常挂载到静态变量即可，或者挂载到外部比如APCU,进程间共享。确保读取写入原子？ 常驻模式下，需要挂载到外部，防止不断增长导致内存溢出

> 由非常驻进程模型转向常驻进程模型主要的难点在于项目文件的修改，比如php通常使用类的静态变量作为对象的缓存，转向swoole常驻进程模型就需要修改所有涉及到statis 变量的声明和使用。暂时没找到优雅的解决方案，比如php定义扩展代替static语法，在底层解析处理，或者写入特性直接引入就可以修改static的逻辑