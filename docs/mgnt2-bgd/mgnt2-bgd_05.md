# 第五章. 使用依赖注入

**依赖注入**是一种软件设计模式，通过该模式，一个或多个依赖项被注入或通过引用传递到对象中。这在实际层面上究竟意味着什么，以下两个简单的示例将展示：

```php
public function getTotalCustomers()
{
    $database = new \PDO( … );
    $statement = $database->query('SELECT …');
    return $statement->fetchColumn();
}
```

在这里，您将看到一个简化的 PHP 示例，其中`$database`对象是在`getTotalCustomers`方法中创建的。这意味着对数据库对象的依赖被锁定在对象实例方法中。这导致了紧密耦合，具有诸如可重用性降低和由于对代码某些部分的更改可能引起的系统级影响等几个缺点。

解决这个问题的方法是通过将依赖注入到方法中来避免具有这些依赖关系的方法，如下所示：

```php
public function getTotalCustomers($database)
{
    $statement = $database->query('SELECT ...');
    return $statement->fetchColumn();
}
```

在这里，一个`$database`对象被传递（注入）到方法中。这就是依赖注入的全部内容——一个简单的概念，它使得代码松散耦合。虽然这个概念很简单，但在像 Magento 这样的大型平台上实现它可能并不容易。

Magento 有其自己的对象管理器和依赖注入机制，我们将在以下部分详细探讨：

+   对象管理器

+   依赖注入

+   配置类偏好

+   使用虚拟类型

### 注意

要跟踪和测试以下部分给出的代码示例，我们可以使用在[`github.com/ajzele/B05032-Foggyline_Di`](https://github.com/ajzele/B05032-Foggyline_Di)可用的代码。要安装它，我们只需下载并将其放入`app/code/Foggyline/Di`目录。然后，在 Magento 根目录的命令行中运行以下命令集：

```php
php bin/magento module:enable Foggyline_Di
php bin/magento setup:upgrade
php bin/magento foggy:di
```

在以下部分提供的代码片段测试时，最后一个命令可以重复使用。当运行`php bin/magento foggy:di`时，它将在`DiTestCommand`类的`execute`方法中运行代码。因此，我们可以从`DiTestCommand`类内部以及`di.xml`文件本身作为**DI**的游乐场使用`__construct`和`execute`方法。

# 对象管理器

在 Magento 中，对象的初始化是通过所谓的**对象管理器**来完成的。对象管理器本身是`Magento\Framework\ObjectManager\ObjectManager`类的实例，该类实现了`Magento\Framework\ObjectManagerInterface`接口。`ObjectManager`类定义了以下三个方法：

+   `create($type, array $arguments = [])`: 这将创建一个新的对象实例

+   `get($type)`: 这将检索一个缓存的实例对象

+   `configure(array $configuration)`: 这将配置`di`实例

对象管理器可以实例化一个 PHP 类，这可能是一个模型、助手或块对象。除非我们正在工作的类已经接收到了对象管理器的实例，否则我们可以通过将`ObjectManagerInterface`传递到类构造函数中来接收它，如下所示：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager
)
{
    $this->_objectManager = $objectManager;
}
```

通常，在 Magento 中我们不需要关心构造函数参数的顺序。以下示例也将使我们能够获取对象管理器的一个实例：

```php
public function __construct(
    $var1,
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $var2 = []
)
{
    $this->_objectManager = $objectManager;
}
```

尽管我们仍然可以使用普通的 PHP 来实例化一个对象，例如`$object = new \Foggyline\Di\Model\Object()`，但通过使用对象管理器，我们可以利用 Magento 的高级对象特性，如自动构造函数依赖注入和对象代理。

这里有一些使用对象管理器的`create`方法创建新对象的示例：

```php
$this->_objectManager->create('Magento\Sales\Model\Order')
$this->_objectManager->create('Magento\Catalog\Model\Product\Image')
$this->_objectManager->create('Magento\Framework\UrlInterface')
$this->_objectManager->create('SoapServer', ['wsdl' => $url, 'options' => $options])
```

下面是一些使用对象管理器的`get`方法创建新对象的示例：

```php
$this->_objectManager->get('Magento\Checkout\Model\Session')
$this->_objectManager->get('Psr\Log\LoggerInterface')->critical($e)
$this->_objectManager->get('Magento\Framework\Escaper')
$this->_objectManager->get('Magento\Sitemap\Helper\Data')
```

对象管理器的`create`方法总是返回一个新的对象实例，而`get`方法返回一个单例。

注意传递给`create`和`get`的一些字符串参数实际上是接口名称，而不是严格的类名称。我们很快就会看到为什么它既适用于类名称也适用于接口名称。现在，只需说它之所以有效，是因为 Magento 的依赖注入实现。

# 依赖注入

到目前为止，我们已经看到了对象管理器如何控制依赖项的实例化。然而，按照惯例，对象管理器不应该在 Magento 中直接使用。相反，它应该用于系统级别的初始化工作。我们被鼓励使用模块的`etc/di.xml`文件来实例化对象。

让我们分析现有的`di.xml`条目之一，例如在`vendor/magento/module-admin-notification/etc/adminhtml/di.xml`文件下为`Magento\Framework\Notification\MessageList`类型找到的条目：

```php
<type name="Magento\Framework\Notification\MessageList">
    <arguments>
        <argument name="messages" xsi:type="array">
            <item name="baseurl" xsi:type="string"> Magento\AdminNotification\Model\System \Message\Baseurl</item>
            <item name="security" xsi:type="string"> Magento\AdminNotification\Model\System\ Message\Security</item>
            <item name="cacheOutdated" xsi:type="string"> Magento\AdminNotification\Model\System\ Message\CacheOutdated</item>
            <item name="media_synchronization_error" xsi:type="string">Magento\AdminNotification\Model\ System\Message\Media\Synchronization\Error</item>
            <item name="media_synchronization_success" xsi:type="string">Magento\AdminNotification\Model\ System\Message\Media\Synchronization\Success</item>
        </argument>
    </arguments>
</type>
```

基本上，这意味着每当创建`Magento\Framework\Notification\MessageList`的实例时，`messages`参数都会传递给构造函数。`messages`参数被定义为数组，该数组进一步由其他字符串类型项组成。在这种情况下，这些字符串类型属性的值是类名称，如下所示：

+   `Magento\Framework\ObjectManager\ObjectManager`

+   `Magento\AdminNotification\Model\System\Message\Baseurl`

+   `Magento\AdminNotification\Model\System\Message\Security`

+   `Magento\AdminNotification\Model\System\Message\CacheOutdated`

+   `Magento\AdminNotification\Model\System\Message\Media\Synchronization\Error`

+   `Magento\AdminNotification\Model\System\Message\Media\Synchronization\Success`

如果你现在查看`MessageList`的构造函数，你会看到它是以下方式定义的：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $messages = []
)
{
    //Method body here...
}
```

如果我们按照以下方式修改`MessageList`的构造函数，代码将能够正常工作：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $someVarX = 'someDefaultValueX',
    $messages = []
)
{
    //Method body here...
}
```

修改后：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $someVarX = 'someDefaultValueX',
    $messages = [],
    $someVarY = 'someDefaultValueY'
)
{
    //Method body here...
}
```

然而，如果我们将`MessageList`的构造函数更改为以下变体之一，代码将无法正常工作：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $Messages = []
)
{
    //Method body here...
}
```

另一种变体如下：

```php
public function __construct(
    \Magento\Framework\ObjectManagerInterface $objectManager,
    $_messages = []
)
{
    //Method body here...
}
```

在 PHP 类的构造函数中，`$messages`参数的名称必须与`di.xml`中参数列表中的参数名称完全匹配。构造函数中参数的顺序并不像它们的命名那样重要。

在进一步查看`MessageList`构造函数时，如果我们在其内部某处执行`func_get_args`，则`$messages`参数中的项目列表将与`vendor/magento/module-admin-notification/etc/adminhtml/di.xml`中显示的列表相匹配并超过它。这是因为列表不是最终的，因为 Magento 从整个平台收集 DI 定义并将它们合并。因此，如果另一个模块正在修改`MessageList`类型，修改将反映出来。

如果我们在整个 Magento 代码库的所有`di.xml`文件中进行字符串搜索，搜索`<type name="Magento\Framework\Notification\MessageList">`，这将产生一些额外的`di.xml`文件，它们对`MessageList`类型有自己的添加，如下所示：

```php
//vendor/magento/module-indexer/etc/adminhtml/di.xml
<type name="Magento\Framework\Notification\MessageList">
    <arguments>
        <argument name="messages" xsi:type="array">
            <item name="indexer_invalid_message" xsi:type="string">Magento\Indexer\Model\Message \Invalid</item>
        </argument>
    </arguments>
</type>

//vendor/magento/module-tax/etc/adminhtml/di.xml
<type name="Magento\Framework\Notification\MessageList">
    <arguments>
        <argument name="messages" xsi:type="array">
            <item name="tax" xsi:type="string">Magento \Tax\Model\System\Message\Notifications</item>
        </argument>
    </arguments>
</type>
```

这意味着`Magento\Indexer\Model\Message\Invalid`和`Magento\Tax\Model\System\Message\Notifications`字符串项被添加到`messages`参数中，并在`MessageList`构造函数中可用。

在前面的 DI 示例中，我们只定义了`$messages`参数作为`array`类型的一个参数，其余的都是其数组项。

让我们看看另一个类型定义的 DI 示例。这次是在`vendor/magento/module-backend/etc/di.xml`文件下找到的，定义如下：

```php
<type name="Magento\Backend\Model\Url">
    <arguments>
        <argument name="scopeResolver" xsi:type="object"> Magento\Backend\Model\Url\ScopeResolver</argument>
        <argument name="authSession" xsi:type="object"> Magento\Backend\Model\Auth\Session\Proxy</argument>
        <argument name="formKey" xsi:type="object"> Magento\Framework\Data\Form\FormKey\Proxy</argument>
        <argument name="scopeType" xsi:type="const"> Magento\Store\Model\ScopeInterface::SCOPE_STORE </argument>
        <argument name="backendHelper" xsi:type="object"> Magento\Backend\Helper\Data\Proxy</argument>
    </arguments>
</type>
```

在这里，您将看到传递给`Magento\Backend\Model\Url`类构造函数的几个不同参数的类型。如果您现在查看`Url`类的构造函数，您将看到它是以以下方式定义的：

```php
public function __construct(
    \Magento\Framework\App\Route\ConfigInterface $routeConfig,
    \Magento\Framework\App\RequestInterface $request,
    \Magento\Framework\Url\SecurityInfoInterface $urlSecurityInfo,
    \Magento\Framework\Url\ScopeResolverInterface $scopeResolver,
    \Magento\Framework\Session\Generic $session,
    \Magento\Framework\Session\SidResolverInterface $sidResolver,
    \Magento\Framework\Url\RouteParamsResolverFactory $routeParamsResolverFactory,
    \Magento\Framework\Url\QueryParamsResolverInterface $queryParamsResolver,
    \Magento\Framework\App\Config\ScopeConfigInterface $scopeConfig,
    $scopeType,
    \Magento\Backend\Helper\Data $backendHelper,
    \Magento\Backend\Model\Menu\Config $menuConfig,
    \Magento\Framework\App\CacheInterface $cache,
    \Magento\Backend\Model\Auth\Session $authSession,
    \Magento\Framework\Encryption\EncryptorInterface $encryptor,
    \Magento\Store\Model\StoreFactory $storeFactory,
    \Magento\Framework\Data\Form\FormKey $formKey,
    array $data = []
) {
    //Method body here...
}
```

这里的`__construct`方法显然比`di.xml`文件中定义的参数要多。这意味着`di.xml`中的类型参数条目并不一定涵盖所有类的`__construct`参数。在`di.xml`中定义的参数只是简单地强加了 PHP 类本身中定义的各个参数的类型。只要`di.xml`中的参数类型相同或为同一类型的子类型，这种方法就可以工作。

理想情况下，我们不会将类类型而是接口传递给 PHP 构造函数，然后在`di.xml`中设置类型。这就是`type`、`preference`和`virtualType`在`di.xml`中发挥重要作用的地方。我们已经看到了`type`的作用。现在，让我们继续看看`preference`是如何工作的。

# 配置类偏好

许多 Magento 的核心类在构造函数中传递接口。这种做法的好处是，对象管理器在`di.xml`的帮助下可以决定为给定的接口实际实例化哪个类。

让我们想象一个具有构造函数的`Foggyline\Di\Console\Command\DiTestCommand`类，如下所示：

```php
public function __construct(
    \Foggyline\Di\Model\TestInterface $myArg1,
    $myArg2,
    $name = null
)
{
    //Method body here...
}
```

注意`$myArg1`是如何被指定为`\Foggyline\Di\Model\TestInterface`接口的。对象管理器知道它需要在整个`di.xml`中查找可能的`preference`定义。

我们可以在模块的`di.xml`文件中定义`preference`，如下所示：

```php
<preference
        for="Foggyline\Di\Model\TestInterface"
        type="Foggyline\Di\Model\Cart"/>
```

在这里，我们基本上是在说，当有人请求 `Foggyline\Di\Model\TestInterface` 的实例时，给它一个 `Foggyline\Di\Model\Cart` 对象的实例。为了使这可行，`Cart` 类必须自己实现 `TestInterface`。一旦 `preference` 定义到位，先前的例子中显示的 `$myArg1` 就变成了 `Cart` 类的对象。

此外，`preference` 元素不仅限于指出某些接口的首选类。我们可以用它来设置某些其他类的首选类。

现在，让我们看看带有构造函数的 `Foggyline\Di\Console\Command\DiTestCommand` 类：

```php
public function __construct(
    \Foggyline\Di\Model\User $myArg1,
    $myArg2,
    $name = null
)
{
    //Method body here...
}
```

注意 `$myArg1` 现在已作为 `\Foggyline\Di\Model\User` 类进行了类型提示。就像在先前的例子中一样，对象管理器会在 `di.xml` 中查找可能的 `preference` 定义。

让我们在模块的 `di.xml` 文件中定义 `preference` 元素，如下所示：

```php
<preference
    for="\Foggyline\Di\Model\User"
    type="Foggyline\Di\Model\Cart"/>
```

这个 `preference` 定义的意思是，每当请求 `User` 类的实例时，传递一个 `Cart` 对象的实例。这只有在 `Cart` 类继承自 `User` 类的情况下才会工作。这是一种重写类的便捷方式，其中类直接传递到另一个类的构造函数中，而不是接口。

由于 `__construct` 参数可以类型提示为类或接口，并且可以通过 `di.xml` 的 `preference` 定义进一步操作，因此会引发一个问题：使用接口还是具体类更好？虽然答案可能并不完全明确，但始终更倾向于使用接口来指定我们注入到系统中的依赖项。

# 使用虚拟类型

除了 `type` 和 `preference`，`di.xml` 还有一个我们可用的强大功能。`virtualType` 元素使我们能够定义虚拟类型。创建虚拟类型就像创建现有类的子类一样，只是它是在 `di.xml` 中而不是在代码中完成的。

**虚拟类型** 是一种在不影响其他类的情况下将依赖项注入到一些现有类中的方法。为了通过实际例子解释这一点，让我们看看在 `app/etc/di.xml` 文件中定义的以下虚拟类型：

```php
<virtualType name="Magento\Framework\Message\Session\Storage" type="Magento\Framework\Session\Storage">
    <arguments>
        <argument name="namespace" xsi:type="string"> message</argument>
    </arguments>
</virtualType>
<type name="Magento\Framework\Message\Session">
    <arguments>
        <argument name="storage" xsi:type="object"> Magento\Framework\Message\Session\Storage</argument>
    </arguments>
</type>
```

先前的例子中的 `virtualType` 定义是 `Magento\Framework\Message\Session\Storage`，它继承自 `Magento\Framework\Session\Storage` 并覆盖了 `namespace` 参数到 `message` 字符串值。在 `virtualType` 中，`name` 属性定义了虚拟类型的全局唯一名称，而 `type` 属性与虚拟类型基于的实际 PHP 类相匹配。

现在，如果你查看 `type` 定义，你会看到其 `storage` 参数被设置为 `Magento\Framework\Message\Session\Storage` 对象。`Session\Storage` 文件实际上是一个虚拟类型。这允许 `Message\Session` 被定制，而不会影响也声明了对 `Session\Storage` 依赖的其他类。

虚拟类型允许我们在特定类中使用依赖项时有效地改变其行为。

# 摘要

在本章中，我们探讨了对象管理器和依赖注入，它们是 Magento 对象管理的基础。我们学习了依赖注入中`type`和`preference`元素的含义以及如何使用它们来操作类构造参数。尽管关于 Magento 中的依赖注入还有很多可以说的，但所提供的信息应该足够，并帮助我们了解 Magento 的其他方面。

在下一章中，我们将通过插件的概念扩展我们的旅程到`di.xml`。
