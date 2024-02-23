# 解决依赖关系

编写松散耦合的代码已经成为任何专业开发人员的必备技能。虽然传统应用程序倾向于将所有内容打包在一起，最终形成一个大的代码块，但现代应用程序采用了更渐进的方法，因为它们在很大程度上依赖于第三方库和其他组件。如今，几乎没有人会构建自己的邮件发送器、日志记录器、路由器、模板引擎等。这些组件中的大部分都可以通过 Composer 等方式被我们的应用程序使用。由于各个组件本身都是由各种社区或商业实体测试和维护的，因此我们的应用程序维护成本大大降低。整体代码质量本身也因为更专业的开发人员处理特定功能而得以提高，这些功能可能超出了我们的专业领域。通过松散耦合的代码实现的和谐。

松散耦合的代码有许多积极的副作用，其中包括以下几点：

+   更容易重构

+   提高代码可维护性

+   更容易跨平台利用

+   更容易跨框架利用

+   对单一职责原则的追求

+   更容易测试

通过利用各种语言特性（如接口）和设计模式（如依赖注入）轻松实现松散耦合的魔法。接下来，我们将通过以下部分来看一下依赖注入的基本方面：

+   减轻常见问题

+   理解依赖注入

+   理解依赖注入容器

# 减轻常见问题

依赖注入是一种既定的软件技术，处理对象依赖的问题，使我们能够编写松散耦合的类。虽然这种模式已经存在了相当长的时间，但 PHP 生态系统直到 Symfony 等主要框架开始实施它之前并没有真正采用它。如今，除了微不足道的应用类型之外，它已经成为事实上的标准。整个依赖问题可以通过一个简单的例子轻松观察到：

```php
<?php   class Customer {
  protected $name;    public function loadByEmail($email)
 {  $mysqli = new mysqli('127.0.0.1', 'foggy', 'h4P9niq5', 'sakila');    $statement = $mysqli->prepare('SELECT * FROM customer WHERE email = ?');
  $statement->bind_param('s', $email);
  $statement->execute();    $customer = $statement->get_result()->fetch_object();    $this->name = $customer->first_name . ' ' . $customer->last_name;    return $this;
 } }   $customer = new Customer(); $customer->loadByEmail('MARY.SMITH@sakilacustomer.org');

```

在这里，我们有一个简单的 `Customer` 类，其中有一个 `loadByEmail()` 方法。令人困扰的部分是 `loadByEmail()` 实例方法对数据库 `$mysqli` 对象的依赖。这导致了紧耦合，降低了代码的可重用性，并为后续代码更改可能引入可能的系统范围副作用打开了大门。为了减轻问题，我们需要将数据库 `$mysqli` 对象注入到 `$customer` 中。

可以从[`dev.mysql.com/doc/sakila/en/`](https://dev.mysql.com/doc/sakila/en/)获取 MySQL Sakila 数据库。

有三种方法可以将依赖注入到对象中*：*

+   通过实例方法

+   通过类构造函数

+   通过实例属性

而实例方法和类构造函数的方法似乎比实例属性注入更受欢迎。

以下示例演示了使用实例方法进行依赖注入的方法：

```php
<?php   class Customer {
  public function loadByEmail($email, $mysqli)
 {  // ...
  } }   $mysqli  = new mysqli('127.0.0.1', 'foggy', 'h4P9niq5', 'sakila');  $customer = new Customer(); $customer->loadByEmail('MARY.SMITH@sakilacustomer.org', $mysqli);

```

在这里，我们通过客户的 `loadByEmail()` 实例方法将 `$mysqli` 对象的实例注入到 `Customer` 对象中。虽然这肯定比在 `loadByEmail()` 方法内部实例化 `$mysqli` 对象的方式更好，但很容易想象，如果我们的类有十几种方法，每种方法都需要传递不同的对象，我们的客户端代码可能会变得笨拙。虽然这种方法似乎很诱人，但通过实例方法注入依赖违反了面向对象编程的封装原则。此外，为了依赖而向方法添加参数绝不是最佳实践的例子。

另一种方法是利用类构造函数方法，如下例所示：

```php
<?php   class Customer {
  public function __construct($mysqli)
 {  // ...
  }    public function loadByEmail($email)
 {  // ...
  } }   $mysqli = new mysqli('127.0.0.1', 'foggy', 'h4P9niq5', 'sakila');   $customer = new Customer($mysqli); $customer->loadByEmail('MARY.SMITH@sakilacustomer.org');

```

在这里，我们通过客户的`__constructor()`方法将`$mysqli`对象的实例注入到`Customer`对象的实例中。无论是注入一个对象还是十几个对象，构造函数注入在这里都是明显的赢家。客户端应用程序有一个单一的入口点用于所有注入，这样就很容易跟踪事物。

没有依赖注入的概念，松散耦合的代码是不可能实现的。

# 理解依赖注入

在介绍部分，我们提到通过类`__construct()`方法传递依赖项。除了传递依赖对象之外，还有更多内容。让我们考虑以下三个看似相似但不同的例子。

尽管 PHP 已经支持类型提示很长一段时间了，但并不罕见遇到以下代码片段：

```php
<?php   class App {
  protected $config;
  protected $logger;    public function __construct($config, $logger)
 {  $this->config = $config;
  $this->logger = $logger;
 }    public function run()
 {  $this->config->setValue('executed_at', time());
  $this->logger->log('executed');
 } }   class Config {
  protected $config = [];    public function setValue($path, $value)
 {  // implementation
  } }   class Logger {
  public function log($message)
 {  // implementation
  } }   $config = new Config(); $logger = new Logger();   $app = new App($config, $logger); $app->run();

```

我们可以看到`App`类的`__construct()`方法没有使用 PHP 类型提示功能。开发人员假定`$config`和`$logger`变量是某种类型。虽然这个例子可以正常工作，但它仍然使我们的类紧密耦合。这个例子和之前在`loadByEmail()`方法中有`$msqli`依赖的例子之间没有太大的区别。

将类型提示添加到混合中允许我们强制传递给`App`类`__construct()`方法的类型：

```php
<?php   class App {
  protected $config;
  protected $logger;    public function __construct(Config $config, Logger $logger)
 {  $this->config = $config;
  $this->logger = $logger;
 }    public function run()
 {  $this->config->setValue('executed_at', time());
  $this->logger->log('executed');
 } }   class Config {
  protected $config = [];    public function setValue($path, $value)
 {  // implementation
  } }   class Logger {
  public function log($message)
 {  // implementation
  } }   $config = new Config(); $logger = new Logger();   $app = new App($config, $logger); $app->run();

```

这个简单的举措使我们的代码松散耦合了一半。虽然我们现在指示我们的可注入对象是一个确切的类型，但我们仍然锁定在一个特定类型上，也就是实现。追求松散耦合不应该让我们锁定在特定的实现上；否则，依赖注入模式就没有太多用处了。

这第三个例子在第一个两个例子中设置了一个重要的区别：

```php
<?php   class App {
  protected $config;
  protected $logger;    public function __construct(ConfigInterface $config, LoggerInterface $logger)
 {  $this->config = $config;
  $this->logger = $logger;
 }    public function run()
 {  $this->config->setValue('executed_at', time());
  $this->logger->log('executed');
 } }   interface ConfigInterface {
  public function getValue($value);    public function setValue($path, $value); }   interface LoggerInterface {
  public function log($message); }   class Config implements ConfigInterface {
  protected $config = [];    public function getValue($value)
 {  // implementation
  }    public function setValue($path, $value)
 {  // implementation
  } }   class Logger implements LoggerInterface {
  public function log($message)
 {  // implementation
  } }   $config = new Config(); $logger = new Logger();   $app = new App($config, $logger); $app->run();

```

偏爱接口类型提示而不是具体类类型提示是编写松散耦合代码的关键要素之一。虽然我们仍然通过类`__construct()`注入依赖项，但现在我们是以*面向接口而不是实现*的方式来做。这使我们能够避免紧密耦合，使我们的代码更具可重用性。

显然，这些例子最终都很简单。我们可以想象当注入的对象数量增加时，事情会变得多么复杂，每个注入的对象可能需要一个、两个，甚至十几个`__construct()`参数本身。这就是依赖注入容器派上用场的地方。

# 理解依赖注入容器

依赖注入容器是一个知道如何自动将类组合在一起的对象。**自动装配**这个术语意味着实例化和正确配置对象。这绝不是一项容易的任务，这就是为什么有几个库在解决这个功能。

Symfony 框架提供的 DependencyInjection 组件是一个整洁的依赖注入容器，可以通过 Composer 轻松安装。

继续前进，让我们创建一个`di-container`目录，在那里我们将执行这些命令并设置我们的项目：

```php
composer require symfony/dependency-injection

```

结果输出表明我们应该安装一些额外的包：

![](img/500c7447-11c1-4eb4-b1dd-54811758adb7.png)

我们需要确保通过运行以下控制台命令添加`symfony/yaml`和`symfony/config`包：

```php
composer require symfony/yaml
composer require symfony/config

```

`symfony/yaml`包安装了 Symfony Yaml 组件。该组件将 YAML 字符串解析为 PHP 数组，反之亦然。`symfony/config`包安装了 Symfony Config 组件。该组件提供了帮助我们从源中查找、加载、合并、自动填充和验证配置值的类，这些源可以是 YAML、XML、INI 文件，甚至是数据库本身。`symfony/dependency-injection`、`symfony/yaml`和`symfony/config`包本身就是松散耦合组件的一个很好的例子。虽然这三个组件共同工作以提供完整的依赖注入功能，但组件本身遵循松耦合的原则。

查看[`symfony.com/doc/current/components/dependency_injection.html`](http://symfony.com/doc/current/components/dependency_injection.html)了解更多关于 Symfony 的 DependencyInjection 组件的信息。

现在让我们继续在`di-container`目录中创建`container.yml`配置文件：

```php
services:
  config:
    class: Config
 logger:
    class: Logger
 app:
    class: App
 autowire: true

```

`container.yml`文件具有特定的结构，以关键字`services`开头。不深入研究，可以说服务容器是 Symfony 对依赖注入容器的称呼，而服务是执行某些任务的任何 PHP 对象--基本上是任何类的实例。

在`services`标签下面，我们有`config`、`logger`和`app`标签。这表示了三个独特服务的声明。我们可以轻松地将它们命名为`the_config`、`the_logger`、`the_app`，或者其他我们喜欢的名称。深入研究各个服务，我们看到`class`标签是所有三个服务共有的。`class`标签告诉容器在请求给定服务实例时实例化哪个类。最后，在`app`服务定义中使用的`autowire`功能允许自动装配子系统通过解析构造函数来检测`App`类的依赖关系。这使得客户端代码非常容易获取`App`类的实例，甚至不需要了解`App`类`__construct()`中的`$config`和`$logger`要求。

有了`container.yml`文件，让我们继续在`di-container`目录中创建`index.php`文件：

```php
<?php   require_once __DIR__ . '/vendor/autoload.php';   use Symfony\Component\DependencyInjection\ContainerBuilder; use Symfony\Component\Config\FileLocator; use Symfony\Component\DependencyInjection\Loader\YamlFileLoader;   interface ConfigInterface { /* ... */}
interface LoggerInterface { /* ... */}
class Config implements ConfigInterface { /* ... */}
class Logger implements LoggerInterface { /* ... */}
class App { /* ... */}   // Bootstrapping $container = new ContainerBuilder();   $loader = new YamlFileLoader($container, new FileLocator(__DIR__)); $loader->load('container.yml');   $container->compile();   // Client code $app = $container->get('app'); $app->run();

```

确保用我们在理解依赖注入部分的第三个示例中的确切代码替换从`ConfigInterface`到`App`的所有内容。

我们首先包含了`autoload.php`文件，以便为我们的依赖容器组件实现自动加载。在`use`语句后面的代码与我们在理解依赖注入部分中的代码相同。有趣的部分在其后。创建了`ContainerBuilder`的实例，并传递给`YamlFileLoader`，后者加载了`container.yml`文件。文件加载后，我们在`$container`实例上调用`compile()`方法。运行`compile()`允许容器识别`autowire`服务标签，以及其他内容。最后，我们在`$container`实例上使用`get()`方法来获取`app`服务的实例。在这种情况下，客户端事先不知道传递给`App`实例的参数；依赖容器根据`container.yml`配置自行处理了所有内容。

使用接口类型提示和容器，我们能够编写更具可重用性、可测试性和解耦性的代码。

查看[`symfony.com/doc/current/service_container.html`](http://symfony.com/doc/current/service_container.html)了解更多关于 Symfony 服务容器的信息。

# 总结

依赖注入是一种简单的技术，它允许我们摆脱紧耦合的枷锁。结合接口类型提示，我们得到了一个强大的技术，可以编写松散耦合的代码。这样可以隔离和最小化可能的未来应用程序设计变化以及其缺陷的影响。如今，甚至在不采用这些简单技术的情况下编写模块化和大型代码库应用程序被认为是不负责任的。

展望未来，我们将更仔细地研究围绕 PHP 包的生态系统的状态，它们的创建和分发。
