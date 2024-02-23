# 魔术方法背后的魔术

PHP 语言允许以过程化和**面向对象**（**OO**）的方式编写代码。虽然过程化方式更像是 PHP 初始版本的遗留物，但我们今天仍然可以编写完全过程化的应用程序。虽然两种方法都有各自的优缺点，但面向对象的方式如今是最主导的，其优势在健壮和模块化的应用程序中更加明显，而这些应用程序几乎不可能使用过程化风格进行工作。

了解 PHP OO 模型的各个特性对于理解、编写和调试现代应用程序至关重要。**魔术方法**是 PHP 语言中更有趣和常常神秘的特性之一。它们是预定义的类方法，PHP 编译器在某些事件下执行，比如对象初始化、对象销毁、对象转换为字符串、对象方法访问、对象属性访问、对象序列化、对象反序列化等等。

在本章中，我们将根据以下章节列表，介绍 PHP 中可用的每个魔术方法的使用：

+   使用 __construct()

+   使用 __destruct()

+   使用 __call()

+   使用 __callStatic()

+   使用 __set()

+   使用 __get()

+   使用 __isset()

+   使用 __unset()

+   使用 __sleep()

+   使用 __wakeup()

+   使用 __toString()

+   使用 __invoke()

+   使用 __set_state()

+   使用 __clone()

+   使用 __debugInfo()

+   跨流行平台的使用统计

PHP 语言将所有以`__`开头的函数名称保留为魔术函数。

# 使用 __construct()

`__construct()`魔术方法代表了 PHP 构造函数概念，类似于其他 OO 语言。它允许开发人员参与对象创建过程。具有`__construct()`方法声明的类，在每个新创建的对象上调用它。这使我们能够处理对象在使用之前可能需要的任何初始化。

以下代码片段显示了`__construct()`方法的最简单可能的用法：

```php
<?php

class User
{
  public function __construct()
  {
    var_dump('__construct');
  }
}

new User;
new User();

```

两个`User`实例将产生相同的`string(11) "__construct"`输出到屏幕上。更复杂的例子可能包括构造函数参数。考虑以下代码片段：

```php
<?php

class User
{
  protected $name;
  protected $age;

  public function __construct($name, $age)
  {
    $this->name = $name;
    $this->age = $age;
    var_dump($this->name);
    var_dump($this->age);
  }
}

new User; #1
new User('John'); #2
new User('John', 34); #3
new User('John', 34, 4200.00); #4

```

在这里，我们看到一个接受两个参数`$name`和`$age`的`__construct()`方法。在`User`类定义之后，我们有四个不同的对象初始化尝试。尝试`#3`是唯一有效的初始化尝试。尝试`#1`和`#2`触发以下错误：

```php
Warning: Missing argument 1 for User::__construct() // #1
Warning: Missing argument 2 for User::__construct() // #1 & #2

```

尝试`#4`，即使无效，也不会触发错误。与其他方法不同，当`__construct()`被额外参数覆盖时，PHP 不会生成错误消息。

`__construct()`方法的另一个有趣的案例是与父类一起使用。让我们考虑以下例子：

```php
<?php

class User
{
  protected $name;
  protected $age;

  public function __construct($name, $age)
  {
    $this->name = $name;
    $this->age = $age;
  }
}

class Employee extends User
{
  public function __construct($employeeName, $employeeAge)
  {
    var_dump($this->name);
    var_dump($this->age);
  }
}

new Employee('John', 34);

```

前面代码的输出如下：

```php
NULL NULL

```

原因是如果子类定义了构造函数，父构造函数不会被隐式调用。要触发父构造函数，我们需要在子构造函数中运行`parent::__construct()`。让我们修改我们的`Employee`类来做到这一点：

```php
class Employee extends User
{
 public function __construct($employeeName, $employeeAge)
 {
   parent::__construct($employeeName, $employeeAge);
   var_dump($this->name);
   var_dump($this->age);
 }
}

```

现在将输出如下：

```php
 string(4) "John" int(34)

```

让我们看下面的例子：

```php
<?php

class User
{
  public function __construct()
  {
    var_dump('__construct');
  }

  public static function hello($name)
  {
    return 'Hello ' . $name;
  }
}

echo User::hello('John');

```

在这里，我们有一个简单的`User`类，有一个魔术`__construct()`和一个静态`hello()`方法。在类定义之后，我们调用静态`hello()`方法。这不会触发`__construct()`方法。

前面例子的唯一输出如下：

```php
Hello John

```

`__construct()`方法只在通过`new`关键字初始化对象时触发。

我们希望保持我们的`__construct()`方法，以及其他魔术方法，只在`public`访问修饰符下。然而，如果情况需要，我们可以自由地将`finally`访问修饰符混合在一起

考虑以下例子：

```php
<?php

class User
{
 public final function __construct($name)
 {
 var_dump($name);
 }
}

class Director extends User
{

}

class Employee extends User
{
 public function __construct($name)
 {
 var_dump($name);
 }
}

new User('John'); #1
new Director('John'); #2
new Employee('John'); #3

```

因此，初始化尝试`#1`和`#2`即使使用`final`访问修饰符也会运行。这是因为`#1`实例化了定义了 final `__construct()`方法的原始`User`类，而`#2`实例化了不尝试实现自己的`__construct()`方法的空`Director`类。初始化尝试`#3`将失败，导致以下错误：

```php
Fatal error: Cannot override final method User::__construct()

```

这实际上是访问修饰符和覆盖的基础，而不是特定于`__construct()`魔术方法本身。然而，值得知道的是，可以使用`final`修饰符与构造函数，因为这可能会派上用场。

除了实例化简单对象外，面向对象编程中`__construct()`方法的实际用途以**依赖注入**的形式出现。如今，注入依赖关系通常被认为是处理依赖关系的一种方法。虽然依赖关系可以通过各种 setter 方法注入到对象中，但在一些主要的 PHP 平台上，如 Magento，使用`__construct()`方法作为主要方法仍然占主导地位。

以下代码块演示了 Magento 的`vendor/magento/module-gift-message/Model/Save.php`文件中的`__construct()`方法：

```php
 public function __construct(
   \Magento\Catalog\Api\ProductRepositoryInterface $productRepository,
   \Magento\GiftMessage\Model\MessageFactory $messageFactory,
   \Magento\Backend\Model\Session\Quote $session,
   \Magento\GiftMessage\Helper\Message $giftMessageMessage
 ) {
   $this->productRepository = $productRepository;
   $this->_messageFactory = $messageFactory;
   $this->_session = $session;
   $this->_giftMessageMessage = $giftMessageMessage;
 }

```

通过`__construct()`方法传递了几个依赖项，这似乎比以前的例子要复杂得多。即便如此，Magento 的大多数`__construct()`方法要比这个更加健壮，向对象传递了数十个参数。

我们可以轻松总结`__construct()`方法的作用，它是一种类签名，表示消费者应该如何完全实例化特定对象。

# 使用 __destruct()

除了构造函数，析构函数是面向对象语言的常见特性。`__destruct()`魔术方法代表了这个概念。一旦没有其他引用指向特定对象，该方法就会被触发。这可能是当 PHP 决定显式释放对象时，也可能是我们使用`unset()`语言构造强制释放对象时发生的。

与构造函数一样，父析构函数不会被 PHP 隐式调用。为了运行父析构函数，我们需要显式调用`parent::__destruct()`。此外，如果子类没有实现自己的析构函数，则子类继承父类的析构函数。

假设我们有一个以下简单的`User`类：

```php
<?php

class User
{
   public function __destruct()
   {
      echo '__destruct';
   }
}

```

有了`User`类，让我们继续查看实例创建示例：

```php
echo 'A';
new User();
echo 'B';

// outputs "A__destructB"

```

这里的`new User();`表达式将`User`类的一个实例实例化为*空气*，因为它没有将新实例化的对象分配给变量。这是 PHP 明确调用`__destruct()`方法的触发器，导致`A__destructB`字符串：

```php
echo 'A';
$user = new User();
echo 'B';

// outputs "AB__destruct"

```

这里的`new User();`表达式将`User`类的一个实例实例化为`$user`变量。这可以防止 PHP 立即触发，因为脚本可能会在路径的后面使用`$user`变量。尽管如此，PHP 在得出结论`$user`变量没有被引用时，会显式调用`__destruct()`方法，导致`AB__destruct`字符串。

```php
echo 'A';
$user = new User();
echo 'B';
unset($user);
echo 'C';

// outputs "AB__destructC"

```

在这里，我们稍微扩展了前面的例子。我们使用`unset()`语言构造来强制销毁表达式之间的`$user`变量。调用`unset()`基本上是 PHP 执行对象的`__destruct()`方法的隐式触发器，导致`AB__destructC`字符串。

```php
echo 'A';
$user = new User();
echo 'B';
exit;
echo 'C';

// outputs "AB__destruct"

```

在这里，我们在`C`字符串输出之前调用`exit()`语言构造。这作为 PHP 的隐式触发器，表明没有更多的引用指向`$user`变量，因此可以执行对象的`__destruct()`方法。结果输出是`AB__destruct`字符串。

某些情况可能会诱使我们从`__destruct()`方法中调用`exit()`构造函数，因为在`__destruct()`中调用`exit()`会阻止剩余的关闭例程执行。同样，从`__destruct()`方法抛出异常只会在脚本终止时触发致命错误。这绝不是处理应用程序状态的方法。

大多数情况下，析构函数不是我们想要或需要自己实现的东西。我们的大多数类可能不需要它，因为 PHP 本身在清理方面做得相当不错。然而，有些情况下，我们可能希望在对象不再被引用后立即释放对象消耗的资源。`__destruct()`方法允许在对象终止时进行某些后续操作。

# 使用 __call()

重载是面向对象编程中一个熟悉的术语。然而，并非所有编程语言都以相同的方式解释它。PHP 中的重载概念与其他面向对象语言大不相同。传统上，重载提供了使用相同名称但不同参数的多个方法的能力，而在 PHP 中，重载意味着动态创建方法和属性。

不幸的是，对术语重载的误用为一些开发人员增加了困惑，因为对于这种类型的功能，更合适的术语可能是*解释器挂钩*。

PHP 中支持方法重载的两个魔术方法是`__call()`和`__callStatic()`。在本节中，我们将更仔细地看看`__call()`方法。

在*对象上下文*中调用不可访问方法时，将触发`__call()`魔术方法。该方法接受两个参数，如下概要所示：

```php
public mixed __call(string $name, array $arguments)

```

然而，`__call()`方法的参数具有以下含义：

+   `$name`：这是被调用的方法的名称

+   `$arguments`：这是一个包含传递给`$name`方法的参数的枚举数组

以下示例演示了在对象上下文中使用`__call()`方法：

```php
<?php

class User
{
 public function __call($name, $arguments)
 {
 echo $name . ': ' . implode(', ', $arguments) . PHP_EOL;
 }

 public function bonus($amount)
 {
 echo 'bonus: ' . $amount . PHP_EOL;
 }
}

$user = new User();
$user->hello('John', 34);
$user->bonus(560.00);
$user->salary(4200.00);

```

`User`类本身只声明了`__call()`和`bonus()`方法。`$user`对象尝试调用`hello()`、`bonus()`和`salary()`方法。这实际上意味着对象试图调用两个缺失的方法：`hello()`和`salary()`。缺失的两个方法会触发`__call()`方法，从而产生以下输出：

```php
__call => hello: John, 34
bonus: 560
__call => salary: 4200

```

我们可以在 Magento 平台中找到`__call()`方法的一个很好的用例示例，如下面从`vendor/magento/framework/DataObject.php`类文件中摘取的条目：

```php
public function __call($method, $args)
{
   switch (substr($method, 0, 3)) {
     case 'get':
       $key = $this->_underscore(substr($method, 3));
       $index = isset($args[0]) ? $args[0] : null;
     return $this->getData($key, $index);
     case 'set':
       $key = $this->_underscore(substr($method, 3));
       $value = isset($args[0]) ? $args[0] : null;
     return $this->setData($key, $value);
     case 'uns':
       $key = $this->_underscore(substr($method, 3));
     return $this->unsetData($key);
     case 'has':
       $key = $this->_underscore(substr($method, 3));
     return isset($this->_data[$key]);
   }
   // ...
}

```

不需要深入了解 Magneto 本身，可以说他们的`DataObject`类在整个框架中充当根数据对象。`__call()`方法中的代码使其能够在对象实例上魔法地*获取*、*设置*、*取消设置*和*检查*属性的存在。这在后续表达式中使用，例如从`vendor/magento/module-checkout/Controller/Cart/Configure.php`文件中摘取的以下条目：

```php
$params = new \Magento\Framework\DataObject();
$params->setCategoryId(false);
$params->setConfigureMode(true);
$params->setBuyRequest($quoteItem->getBuyRequest());

```

好处在于我们可以轻松地为`DataObject`的实例赋予可能存在也可能不存在的魔术方法。例如，`setCategoryId()`是`DataObject`类上不存在的方法。由于它不存在，调用它会触发`__call()`方法。这一点一开始可能不太明显，因此让我们考虑另一个想象的例子，我们的自定义类从`DataObject`继承：

```php
<?php

class User extends \Magento\Framework\DataObject
{

}

$user = new User();

$user->setName('John');
$user->setAge(34);
$user->setSalary(4200.00);

echo $user->getName();
echo $user->getAge();
echo $user->getSalary();

```

注意我们通过`__call()`魔术方法在这里实现的*设置器*和*获取器*的*美丽和简单*。尽管我们的`User`类基本上是空的，但我们已经继承了父类`__call()`实现的魔术。

`__call()`方法赋予我们一些真正有趣的可能性，其中大部分将作为框架或库的一部分。

# 使用 __callStatic()

`__callStatic()`魔术几乎与`__call()`方法相同。`__call()`方法绑定到*对象上下文*，而`__callStatic()`方法绑定到*静态上下文*，这意味着在通过作用域解析运算符（`::`）调用不可访问的方法时会触发此方法。

该方法根据以下概要接受两个参数：

```php
public static mixed __callStatic (string $name, array $arguments)

```

请注意，在方法声明中使用静态访问修饰符，这是静态上下文所需的。以下示例演示了在静态上下文中使用`__callStatic()`方法：

```php
<?php

class User
{
  public static function __callStatic($name, $arguments)
  {
    echo '__callStatic => ' . $name . ': ' . implode(', ', $arguments)
      . PHP_EOL;
  }

  public static function bonus($amount)
  {
  echo 'bonus: ' . $amount . PHP_EOL;
  }
}

```

代码将产生以下输出：

```php
User::hello('John', 34);
User::bonus(560.00);
User::salary(4200.00);

```

`User`类本身只声明了`__callStatic()`和`bonus()`方法。`User`类尝试调用静态`hello()`，`bonus()`和`salary()`方法。这实际上意味着该类试图调用两个缺失的方法：`hello()`和`salary()`。对于缺失的两个方法，`__callStatic()`方法会启动，从而产生以下输出：

```php
__callStatic => hello: John, 34
bonus: 560
__callStatic => salary: 4200

```

在面向对象编程中，静态上下文比对象上下文更少见，这使得`__callStatic()`方法比`__call()`方法更少使用。

# 使用 __set()

除了*方法重载*之外，*属性重载*是 PHP 重载功能的另一个方面。 PHP 中有四个魔术方法支持属性重载：`__set()`，`__get()`，`__isset()`和`__unset()`。在本节中，我们将更仔细地看一下`__set()`方法。

尝试向不可访问的属性写入数据时，将触发`__set()`魔术方法。

该方法根据以下概要接受两个参数：

```php
public void __set(string $name, mixed $value)

```

而`__set()`方法的参数具有以下含义：

+   `$name`：这是正在交互的属性的名称

+   `$value`：这是`$name`属性应该设置的值

让我们看一下以下对象上下文示例：

```php
<?php

class User
{
  private $data = array();

  private $name;
  protected $age;
  public $salary;

  public function __set($name, $value)
  {
    $this->data[$name] = $value;
  }
}

$user = new User();
$user->name = 'John';
$user->age = 34;
$user->salary = 4200.00;
$user->message = 'hello';

var_dump($user);

```

`User`类声明了四个具有不同访问修饰符的属性。它进一步声明了`__set()`方法，该方法拦截对象上下文中的所有属性写入尝试。尝试设置不存在的（`$message`）或不可访问的（`$name`，`$age`）属性会触发`__set()`方法。`__set()`方法的内部工作将不可访问的数据推送到`$data`属性数组中，这在以下输出中可见：

```php
object(User)#1 (4) {
  ["data":"User":private]=> array(3) {
    ["name"]=> string(4) "John"
    ["age"]=> int(34)
    ["message"]=> string(5) "hello"
  }
  ["name":"User":private]=> NULL
  ["age":protected]=> NULL
  ["salary"]=> float(4200)
}

```

`__set()`方法的一个实际用途可能是允许在对象构造期间将属性设置为`true`；否则，抛出异常。

在静态上下文中尝试使用四种属性重载方法（`__set()`，`__get()`，`__isset()`和`__unset()`）将导致以下错误：

```php
PHP Warning: The magic method __set() must have public visibility and cannot be static...

```

# 使用 __get()

尝试从不可访问的属性中读取数据时，将触发`__get()`魔术方法。该方法根据以下概要接受一个参数：

```php
public mixed __get(string $name)

```

`$name`参数是正在交互的属性的名称。

让我们看一下以下对象上下文示例：

```php
<?php

class User
{
  private $data = [
    'name' => 'Marry',
    'age' => 32,
    'salary' => 5300.00,
  ];

  private $name = 'John';
  protected $age = 34;
  public $salary = 4200.00;

  public function __get($name)
  {
    if (array_key_exists($name, $this->data)) {
      echo '__get => ' . $name . ': ' . $this->data[$name] . PHP_EOL;
    } else {
      trigger_error('Undefined property: ' . $name, E_USER_NOTICE);
    }
  }
}

$user = new User();

echo $user->name . PHP_EOL;
echo $user->age . PHP_EOL;
echo $user->salary . PHP_EOL;
echo $user->message . PHP_EOL;

```

`User`类定义了四个不同的属性，跨越三种不同的可见性访问修饰符。由于我们没有获取器方法来访问所有单独的属性，唯一可直接访问的属性是`public $salary`。这就是`__get()`方法派上用场的地方，因为一旦我们尝试访问不存在或无法访问的属性，它就会启动。前面代码的结果输出为以下四行：

```php
__get => name: Marry

__get => age: 32

4200

PHP Notice: Undefined property: message in...

```

`age`和`name`的值是从`$data`属性中获取的，这是`__get()`方法内部工作的结果。

# 使用 __isset()

`__isset()`魔术方法是通过调用`isset()`或`empty()`语言结构来触发的。该方法根据以下概要接受一个参数：

```php
public bool __isset(string $name)

```

`$name`参数是正在交互的属性的名称。

让我们看一下以下对象上下文示例：

```php
<?php

class User
{
  private $data = [
    'name' => 'John',
    'age' => 34,
  ];

  public function __isset($name)
  {
    if (array_key_exists($name, $this->data)) {
      return true;
    }

    return false;
  }
}

$user = new User();

var_dump(isset($user->name));

```

`User`类定义了一个名为`$data`的单个受保护的数组属性，以及一个魔术`__isset()`方法。当前方法的内部工作只是针对`$data`数组键名进行查找，并在数组中找到键时返回`true`，否则返回`false`。示例的结果输出为`bool(true)`。

Magento 平台为`vendor/magento/framework/HTTP/PhpEnvironment/Request.php`类文件的`__isset()`方法提供了一个有趣且实用的用例：

```php
public function __isset($key)
{
  switch (true) {
    case isset($this->params[$key]):
    return true;

    case isset($this->queryParams[$key]):
    return true;

    case isset($this->postParams[$key]):
    return true;

    case isset($_COOKIE[$key]):
    return true;

    case isset($this->serverParams[$key]):
    return true;

    case isset($this->envParams[$key]):
    return true;

    default:
    return false;
  }
}

```

这里的`Magento\Framework\HTTP\PhpEnvironment\Request`类代表了 PHP 环境及其所有可能的请求数据。请求数据可以来自许多来源：查询字符串、`$_GET`、`$_POST`等。`switch`语句遍历了这些源数据变量（`$params`、`$queryParams`、`$postParams`、`$serverParams`、`$envParams`、`$_COOKIE`），以查找并确认请求参数的存在。

# 使用 __unset()

通过调用`unset()`语言构造函数来触发`__unset()`魔术方法，该方法接受一个参数，如下概要所示：

```php
public bool __unset(string $name)

```

`$name`参数是正在交互的属性的名称。

让我们看一下以下对象上下文示例：

```php
<?php

class User
{
  private $data = [
    'name' => 'John',
    'age' => 34,
  ];

  public function __unset($name)
  {
    unset($this->data[$name]);
  }
}

$user = new User();

var_dump($user);
unset($user->age);
unset($user->salary);
var_dump($user);

```

`User`类声明了一个单个私有的`$data`数组属性，以及`__unset()`魔术方法。这个方法本身非常简单；它只是调用`unset()`并传递给它给定数组键的值。我们正在尝试取消`$age`和`$salary`属性。`$salary`属性实际上并不存在，既不是类属性，也不是`data`数组的键。幸运的是，`unset()`不会抛出`Undefined index`类型的错误，因此我们不需要额外的`array_key_exists()`检查。以下的输出显示了从对象实例中删除了`$age`属性：

```php
object(User)#1 (1) {
  ["data":"User":private]=> array(2) {
    ["name"]=> string(4) "John"
    ["age"]=> int(34)
  }
}

object(User)#1 (1) {
  ["data":"User":private]=> array(1) {
    ["name"]=> string(4) "John"
  }
}

```

我们不应该混淆`unset()`构造与`(unset)`转换的用法。这两者是不同的操作，因此`(unset)`转换不会触发`__unset()`魔术方法。

```php
unset($user->age); // will trigger __unset()
((unset) $user->age); // won't trigger __unset()

```

# 使用 __sleep()

对象序列化是面向对象编程的另一个重要方面。PHP 提供了一个`serialize()`函数，允许我们对传递给它的值进行序列化。结果是一个包含可以存储在 PHP 中的任何值的字节流表示的字符串。对标量数据类型和简单对象进行序列化是非常简单的，如下例所示：

```php
<?php

$age = 34;
$name = 'John';

$obj = new stdClass();
$obj->age = 34;
$obj->name = 'John';

var_dump(serialize($age));
var_dump(serialize($name));
var_dump(serialize($obj));

```

结果输出如下：

```php
string(5) "i:34;"
string(11) "s:4:"John";"
string(56) "O:8:"stdClass":2:{s:3:"age";i:34;s:4:"name";s:4:"John";}"

```

即使是一个简单的自定义类也可以很容易地：

```php
<?php

class User
{
  public $name = 'John';
  private $age = 34;
  protected $salary = 4200.00;
}

$user = new User();

var_dump(serialize($user));

```

上述代码的结果如下：

```php
string(81) "O:4:"User":3:{s:4:"name";s:4:"John";s:9:"Userage";i:34;s:9:"*salary";d:4200;}"

```

当我们的类在大小上要么很重要，要么包含资源类型的引用时，就会出现问题。`__sleep()`魔术方法以一种方式解决了这些挑战。它的预期用途是提交未决数据或执行相关的清理任务。当我们有不需要完全序列化的大型对象时，该函数非常有用。

`serialize()`函数会在对象存在时触发对象的`__sleep()`方法。实际触发是在序列化过程开始之前完成的。这使对象能够明确列出它想要允许序列化的字段。`__sleep()`方法的返回值必须是一个包含我们想要序列化的所有对象属性名称的数组。如果该方法不返回可序列化的属性名称数组，则会序列化为`NULL`并发出`E_NOTICE`。

以下示例演示了一个简单的`User`类，其中包含一个简单的`__sleep()`方法的实现：

```php
<?php

class User
{
  public $name = 'John';
  private $age = 34;
  protected $salary = 4200.00;

  public function __sleep() 
  {
    // Cleanup & other operations???
    return ['name', 'salary'];
  }
}

$user = new User();

var_dump(serialize($user));

```

`__sleep()`方法的实现清楚地说明`User`类的唯一两个可序列化属性是`name`和`salary`。请注意，实际名称以字符串形式提供，没有`$`符号，这导致输出如下：

```php
string(60) "O:4:"User":2:{s:4:"name";s:4:"John";s:9:"*salary";d:4200;}"

```

将对象序列化以存储在数据库中是一种危险的做法，应尽可能避免。需要复杂对象序列化的情况很少。即使有这样的情况，也很可能是应用设计不当的标志。

# 使用 __wakeup()

关于可序列化对象的主题如果没有`serialize()`方法的对应方法--`unserialize()`方法，将不完整。如果`serialize()`方法调用触发对象的`__sleep()`魔术方法，那么可以合理地期望反序列化也有类似的行为。因此，在给定对象上调用`unserialize()`方法将触发其`__wakeup()`魔术方法。

`__wakeup()`的预期用途是重新建立在序列化过程中可能丢失的任何资源，并执行其他重新初始化任务。

让我们看下面的例子：

```php
<?php

class Backup
{
  protected $ftpClient;
  protected $ftpHost;
  protected $ftpUser;
  protected $ftpPass;

  public function __construct($host, $username, $password)
  {
    $this->ftpHost = $host;
    $this->ftpUser = $username;
    $this->ftpPass = $password;

    echo 'TEST!!!' . PHP_EOL;

    $this->connect();
  }

  public function connect()
  {
    $this->ftpClient = ftp_connect($this->ftpHost, 21, 5);
    ftp_login($this->ftpClient, $this->ftpUser, $this->ftpPass);
  }

  public function __sleep()
  {
    return ['ftpHost', 'ftpUser', 'ftpPass'];
  }

  public function __wakeup()
  {
    $this->connect();
  }
}

$backup = new Backup('test.rebex.net', 'demo', 'password');
$serialized = serialize($backup);
$unserialized = unserialize($serialized);

var_dump($backup);
var_dump($serialized);
var_dump($unserialized);

```

`Backup`类通过其构造函数接受主机、用户名和密码信息。在内部，它将核心 PHP 的`ftp_connect()`函数设置为建立与 FTP 服务器的连接。成功建立的连接返回一个资源，我们将其存储到类的受保护的`$ftpClient`属性中。由于资源不可序列化，我们确保将其从`__sleep()`方法返回数组中排除。这确保我们的序列化字符串不包含`$ftpHost`属性。我们进一步在`__wakeup()`方法中设置了`$this->connect();`调用，以重新初始化`$ftpHost`资源。整体示例结果如下输出：

```php
object(Backup)#1 (4) {
  ["ftpClient":protected]=> resource(4) of type (FTP Buffer)
  ["ftpHost":protected]=> string(14) "test.rebex.net"
  ["ftpUser":protected]=> string(4) "demo"
  ["ftpPass":protected]=> string(8) "password"
}

string(119) "O:6:"Backup":3:{s:10:"*ftpHost";s:14:"test.rebex.net";s:10:"*ftpUser";s:4:"demo";s:10:"*ftpPass";s:8:"password";}"

object(Backup)#2 (4) {
  ["ftpClient":protected]=> resource(5) of type (FTP Buffer)
  ["ftpHost":protected]=> string(14) "test.rebex.net"
  ["ftpUser":protected]=> string(4) "demo"
  ["ftpPass":protected]=> string(8) "password"
}

```

`__wakeup()`方法在`unserialize()`函数调用期间承担了构造函数的角色。因为对象的`__construct()`方法在反序列化期间不会被调用，所以我们需要小心地实现必要的`__wakeup()`方法逻辑，以便对象可以重建可能需要的任何资源。

# 使用 __toString()

`__toString()`魔术方法在我们将对象用于字符串上下文时触发。它允许我们决定对象在被视为字符串时的反应方式。

让我们看下面的例子：

```php
<?php

class User
{
  protected $name;
  protected $age;

  public function __construct($name, $age)
  {
    $this->name = $name;
    $this->age = $age;
  }
}

$user = new User('John', 34);
echo $user;

```

在这里，我们有一个简单的`User`类，通过其构造方法接受`$name`和`$age`参数。除此之外，没有其他内容表明类应如何响应尝试在字符串上下文中使用它，这正是我们在类声明后立即做的，因为我们试图`echo`对象实例本身。

在其当前形式下，生成的输出将如下所示：

```php
Catchable fatal error: Object of class User could not be converted to string in...

```

`__toString()`魔术方法允许我们简单而优雅地规避这个错误：

```php
<?php

class User
{
  protected $name;
  protected $age;

  public function __construct($name, $age)
  {
    $this->name = $name;
    $this->age = $age;
  }

  public function __toString()
  {
    return $this->name . ', age ' . $this->age;
  }
}

$user = new User('John', 34);
echo $user;

```

通过添加`__toString()`魔术方法，我们能够将对象的结果字符串表示定制为以下代码行：

```php
John, age 34

```

Guzzle HTTP 客户端通过其 PSR7 HTTP 消息接口实现提供了`__toString()`方法的实际用例示例；而一些实现使用了`__toString()`方法。以下代码片段是 Guzzle 的`vendor/guzzlehttp/psr7/src/Stream.php`类文件的部分提取，该文件实现了`Psr\Http\Message\StreamInterface`接口：

```php
 public function __toString()
 {
   try {
     $this->seek(0);
     return (string) stream_get_contents($this->stream);
   } catch (\Exception $e) {
     return '';
   }
 }

```

在逻辑丰富的`__toString()`实现中，`try...catch`块基本上是一种常态。这是因为我们不能从`__toString()`方法中抛出异常。因此，我们需要确保没有错误逃逸。

# 使用 __invoke()

`__invoke()`魔术方法在对象被调用为函数时触发。该方法接受可选数量的参数，并能够返回各种类型的数据，或者根本不返回数据，如下概要所示：

```php
mixed __invoke([ $... ])

```

如果对象类实现了`__invoke()`方法，我们可以通过在对象名称后面加上括号`()`来调用该方法。这种类型的对象称为函数对象或函数对象。

维基百科页面（[`en.wikipedia.org/wiki/Functor`](https://en.wikipedia.org/wiki/Functor)）提供了有关函数子的更多信息。

以下代码块演示了简单的`__invoke()`实现：

```php
<?php

class User
{
  public function __invoke($name, $age)
  {
    echo $name . ', ' . $age;
  }
}

```

`__invoke()`方法可以通过将对象实例作为函数使用或调用`call_user_func()`来触发。

```php
$user = new User();

$user('John', 34); // outputs: John, 34

call_user_func($user, 'John', 34); // outputs: John, 34

```

使用`__invoke()`方法，我们可以将我们的类伪装成。

```php
var_dump(is_callable($user)); // true

```

使用`__invoke()`的好处之一是，它可以创建一个跨语言的标准回调类型。这比在引用函数、对象实例方法或类静态方法时使用字符串、对象和数组的组合更方便，通过`call_user_func()`函数。

`__invoke()`方法作为强大的语言补充，我们认为它提供了新的开发模式的机会；尽管它的滥用可能会导致代码不清晰和混乱。

# 使用 __set_state()

`__set_state()`魔术方法被触发（实际上并没有）用于`var_export()`函数导出的类。该方法接受一个单一的数组类型参数，并返回一个对象，如下概要所示：

```php
static object __set_state(array $properties)

```

`var_export()`函数输出或返回给定变量的可解析字符串表示。它与`var_dump()`函数有些类似，不同之处在于返回的表示是有效的 PHP。

```php
<?php

class User
{
  public $name = 'John';
  public $age = 34;
  private $salary = 4200.00;
  protected $identifier = 'ABC';
}

$user = new User();
var_export($user); // outputs string "User::__set_state..."
var_export($user, true); // returns string "User::__set_state..."

```

这导致了以下输出：

```php
User::__set_state(array(
 'name' => 'John',
 'age' => 34,
 'salary' => 4200.0,
 'identifier' => 'ABC',
))

string(113) "User::__set_state(array(
 'name' => 'John',
 'age' => 34,
 'salary' => 4200.0,
 'identifier' => 'ABC',
))"

```

使用`var_export()`函数实际上不会触发我们的`User`类的`__set_state()`方法。它只产生一个`User::__set_state(array(...))`表达式的字符串表示，我们可以记录、输出或通过`eval()`语言结构进行执行。

以下代码片段是一个更健壮的示例，演示了`eval()`的使用：

```php
<?php

class User
{
  public $name = 'John';
  public $age = 34;
  private $salary = 4200.00;
  protected $identifier = 'ABC';

  public static function __set_state($properties)
  {
    $user = new User();

    $user->name = $properties['name'];
    $user->age = $properties['age'];
    $user->salary = $properties['salary'];
    $user->identifier = $properties['identifier'];

    return $user;
  }
}

$user = new User();
$user->name = 'Mariya';
$user->age = 32;

eval('$obj = ' . var_export($user, true) . ';');

var_dump($obj);

```

这导致了以下输出：

```php
object(User)#2 (4) {
  ["name"]=> string(6) "Mariya"
  ["age"]=> int(32)
  ["salary":"User":private]=> float(4200)
  ["identifier":protected]=> string(3) "ABC"
}

```

了解`eval()`语言结构非常危险，因为它允许执行任意的 PHP 代码，因此不建议使用。因此，除了调试目的之外，使用`__set_state()`本身就变得值得怀疑。

# 使用 __clone()

`__clone()`魔术方法在使用`clone`关键字进行克隆的新克隆对象上触发。该方法不接受任何参数，也不返回任何值，如下概要所示：

```php
void __clone(void)

```

在对象克隆方面，我们倾向于区分深拷贝和浅拷贝。深拷贝会复制所有对象可能指向的对象。浅拷贝尽可能少地复制，将对象引用留作引用。虽然浅拷贝可能对抗循环引用很有用，但不一定是期望的行为，因为它会复制所有属性，无论它们是引用还是值。

以下示例演示了`__clone()`方法的实现和`clone`关键字的使用：

```php
<?php

class User
{
  public $identifier;

  public function __clone()
  {
    $this->identifier = null;
  }
}

$user = new User();
$user->identifier = 'john';

$user2 = clone $user;

var_dump($user);
var_dump($user2);

```

这导致了以下输出：

```php
object(User)#1 (1) {
  ["identifier"]=> string(4) "john"
}

object(User)#2 (1) {
  ["identifier"]=> NULL
}

```

关于`__clone()`方法的重要要点是，它并不是克隆过程的覆盖。正常的克隆过程总是会发生。`__clone()`方法只是承担了修正错误的责任，我们可能对结果不满意时会使用它。

# 使用 __debugInfo()

当调用`var_dump()`函数时，`__debugInfo()`魔术方法会被触发。默认情况下，`var_dump()`函数显示对象的所有公共、受保护和私有属性。但是，如果对象类实现了`__debugInfo()`魔术方法，我们可以控制`var_dump()`函数的输出。该方法不接受任何参数，并返回一个要显示的键值数组，如下概要所示：

```php
array __debugInfo(void)

```

以下示例演示了`__debugInfo()`方法的实现：

```php
<?php

class User
{
  public $name = 'John';
  public $age = 34;
  private $salary = 4200.00;
  private $bonus = 680.00;
  protected $identifier = 'ABC';
  protected $logins = 67;

  public function __debugInfo()
  {
    return [
      'name' => $this->name,
      'income' => $this->salary + $this->bonus
    ];
  }
}

$user = new User();

var_dump($user);

```

这导致了以下输出：

```php
object(User)#1 (2) {
  ["name"]=> string(4) "John"
  ["income"]=> float(4880)
}

```

虽然`__debugInfo()`方法对于定制我们自己的`var_dump()`输出很有用，但这可能不是我们在日常开发中必须做的事情。

# 流行平台上的使用统计

PHP 生态系统可以说是非常庞大的。有数十个免费和开源的 CMS、CRM、购物车、博客和其他平台和库。WordPress、Drupal 和 Magento 可能是在博客、内容管理和购物车解决方案方面最受欢迎的平台之一。它们都可以从各自的网站上下载：

+   WordPress: [`wordpress.org`](https://wordpress.org)

+   Drupal: [`www.drupal.org`](https://www.drupal.org)

+   Magento: [`magento.com/`](https://magento.com/)

考虑到这些流行平台，以下表格对魔术方法的使用进行了一些说明：

| **魔术方法** | **WordPress 4.7****(702 .php files)** | **Drupal 8.2.4****(8199 .php files)** | **Magento CE 2.1.3****(29649 .php files)** |
| --- | --- | --- | --- |
| `__construct()` | 343 | 2547 | 12218 |
| `__destruct()` | 19 | 19 | 77 |
| `__call()` | 10 | 35 | 152 |
| `__callStatic()` | 1 | 2 | 4 |
| `__get()` | 23 | 31 | 125 |
| `__set()` | 15 | 24 | 86 |
| `__isset()` | 21 | 15 | 57 |
| `__unset()` | 11 | 13 | 34 |
| `__sleep()` | 0 | 46 | 103 |
| `__wakeup()` | 0 | 10 | 94 |
| `__toString()` | 15 | 181 | 460 |
| `__invoke()` | 0 | 27 | 112 |
| `__set_state()` | 0 | 3 | 5 |
| `__clone()` | 0 | 32 | 68 |
| `__debugInfo()` | 0 | 0 | 2 |

该表是对整个平台代码库中`function __[magic-method-name]`的粗略搜索结果。很难在此基础上得出任何结论，因为平台在`.php`文件数量上有显著差异。有一件事我们可以肯定——并非所有魔术方法都同样受欢迎。例如，WordPress 似乎甚至没有使用`__sleep()`、`__wakeup()`和`__invoke()`方法，这些方法在面向对象编程中很重要。这可能是因为 WordPress 处理的 OO 组件没有像 Magento 那样多，后者在架构上更多地是一个面向对象的平台。Drupal 在这里有点中庸，在总的`.php`文件数量和使用的魔术方法方面。无论是否有定论，上表概述了 PHP 提供的几乎每个魔术方法的活跃使用。

# 总结

在本章中，我们详细研究了 PHP 提供的每个魔术方法。它们的易用性和它们为语言带来的功能同样令人印象深刻。通过适当命名我们的类方法，我们能够利用对象状态和行为的几乎每个方面。虽然这些魔术方法大多数情况下不是我们日常使用的东西，但它们的存在赋予了我们一些巧妙的架构风格和解决方案，这些解决方案在其他语言中并不容易实现。

未来，我们将进入 CLI 领域和更难以捉摸的 PHP 使用。
