# 突出的面向对象编程特性

“面向对象”（OO）这个术语自 70 年代以来就存在，当时由计算机科学家 Alan Kay 创造。该术语代表了基于对象概念的编程范式。当时，Simula 是第一种展示面向对象特性的语言，如对象、类、继承、子类型等。1977 年标准化为 Simula 67 后，它成为后来语言的灵感来源。其中一种受到启发的语言是 Smalltalk，由 Xerox 的 Alan Kay 领导的研究团队创建。与 Simula 相比，Smalltalk 极大地改进了整体的面向对象概念。随着时间的推移，Smalltalk 成为了最有影响力的面向对象编程语言之一。

虽然这些早期日子还有很多值得说的地方，但重点是面向对象编程是出于特定的需求而诞生的。Simula 使用静态对象来对建模现实世界的实体，而 Smalltalk 使用动态对象作为计算的基础，可以创建、更改或删除。

MVC 模式，是最常见的面向对象软件设计模式之一，是在 Smalltalk 中引入的。

将物理实体映射为类描述的对象的便利性显然影响了开发人员对面向对象范式的整体流行度。然而，对象不仅仅是关于各种属性的映射实例，它们还涉及消息和责任。虽然我们可能基于第一个前提接受面向对象编程，但我们肯定开始欣赏后者，因为构建大型和可扩展系统的关键在于对象通信的便利性。

PHP 语言包含多种范式，最为突出的是：命令式、函数式、面向对象、过程式和反射。然而，PHP 中的面向对象支持直到 PHP 5 发布之前才完全启动。PHP 7 的最新版本带来了一些微小但值得注意的改进，现在被认为是一个稳定和成熟的 PHP 面向对象模型。

在本章中，我们将探讨 PHP 面向对象的一些突出特性：

+   对象继承

+   对象和引用

+   对象迭代

+   对象比较

+   特征

+   反射

# 对象继承

面向对象范式将对象置于应用程序设计的核心，其中对象可以被视为包含各种属性和方法的单元。这些属性和方法之间的交互定义了对象的内部状态。每个对象都是从称为类的蓝图构建的。在基于类的面向对象编程中，没有类的对象，至少不是一个对象。

我们区分基于类的面向对象编程（PHP、Java、C#，...）和基于原型的面向对象编程（ECMAScript / JavaScript、Lua，...）。在基于类的面向对象编程中，对象是从类创建的；在基于原型的面向对象编程中，对象是从其他对象创建的。

构建或创建新对象的过程称为实例化。在 PHP 中，与许多其他语言一样，我们使用`new`关键字从给定的类实例化一个对象。让我们看看以下示例：

```php
<?php

class JsonOutput
{
  protected $content;

  public function setContent($content)
  {
    $this->content = $content;
  }

  public function render()
  {
    return json_encode($this->content);
  }
}

class SerializedOutput
{
  protected $content;

  public function setContent($content)
  {
    $this->content = $content;
  }

  public function render()
  {
    return serialize($this->content);
  }
}

$users = [
  ['user' => 'John', 'age' => 34],
  ['user' => 'Alice', 'age' => 33],
];

$json = new JsonOutput();
$json->setContent($users);
echo $json->render();

$ser = new SerializedOutput();
$ser->setContent($users);
echo $ser->render();

```

在这里，我们定义了两个简单的类，`JsonOutput`和`SerializedOutput`。我们之所以说简单，仅仅是因为它们有一个属性和两个方法。这两个类几乎是相同的——它们只在`render()`方法中的一行代码上有所不同。一个类将给定的内容转换为 JSON，而另一个类将其转换为序列化字符串。在我们的类声明之后，我们定义了一个虚拟的$users 数组，然后将其提供给`JsonOutput`和`SerializedOutput`类的实例，即`$json`和`$ser`对象。

虽然这远非理想的类设计，但它作为继承的一个很好的介绍。

允许类和因此对象继承另一个类的属性和方法。术语如超类、基类或父类用于标记用作继承基础的类。术语如子类、派生类或子类用于标记继承类。

PHP 的`extends`关键字用于启用继承。继承有其限制。我们一次只能从一个类扩展，因为 PHP 不支持多重继承。但是，具有继承链是完全有效的：

```php
// valid
class A {}
class B extends A {}
class C extends B {}

// invalid
class A {}
class B {}
class C extends A, B {}

```

在有效示例中显示的`C`类最终将继承类`B`和`A`的所有允许的属性和方法。当我们说允许时，我们指的是属性和方法的可见性，即访问修饰符：

```php
<?php   error_reporting(E_ALL);   class A {
  public $x = 10;
  protected $y = 20;
  private $z = 30;    public function x()
 {  return $this->x;
 }    protected function y()
 {  return $this->y;
 }    private function z()
 {  return $this->z;
 } }   class B extends A {   }   $obj = new B(); var_dump($obj->x); // 10 var_dump($obj->y); // Uncaught Error: Cannot access protected property B::$y var_dump($obj->z); // Notice: Undefined property: B::$z var_dump($obj->x()); // 10 var_dump($obj->y()); // Uncaught Error: Call to protected method A::y() from context var_dump($obj->z()); // Uncaught Error: Call to private method A::z() from context

```

在对象上下文中，访问修饰符的行为与前面的示例一样，这基本上是我们所期望的。无论它是类`A`的实例还是类`B`的实例，对象都会表现出相同的行为。让我们观察子类内部工作中访问修饰符的行为：

```php
class B extends A
{
  public function test()
  {
    var_dump($this->x); // 10
    var_dump($this->y); // 20
    var_dump($this->z); // Notice: Undefined property: B::$z
    var_dump($this->x()); // 10
    var_dump($this->y()); // 20
    var_dump($this->z()); // Uncaught Error: Call to private method 
      A::z() from context 'B'
  }
}

$obj = new B();
$obj->test();

```

我们可以看到，`public`和`protected`成员（属性或方法）可以从子类中访问，而私有成员则不行——它们只能从定义它们的类中访问。

`extends`关键字也适用于接口：

```php
<?php   interface User {}  interface Employee extends User {}

```

能够继承类和接口的属性和方法构成了一个强大的整体对象继承机制。

了解这些简单的继承规则，让我们看看如何使用继承将我们的`JsonOutput`和`SerializedOutput`类重写为更方便的形式：

```php
<?php   class Output {
  protected $content;    public function setContent($content)
 {  $this->content = $content;
 }    public function render()
 {  return $this->content;
 } }   class JsonOutput extends Output {
  public function render()
 {  return json_encode($this->content);
 } }   class SerializedOutput extends Output {
  public function render()
 {  return serialize($this->content);
 } }

```

我们首先定义了一个`Output`类，其内容几乎与之前的`JsonOutput`和`SerializedOutput`类相同，只是将其`render()`方法更改为简单地返回内容。然后，我们以这样的方式重写了`JsonOutput`和`SerializedOutput`类，它们都扩展了`Output`类。在这种设置中，`Output`类成为父类，而`JsonOutput`和`SerializedOutput`成为子类。子类重新定义了`render()`方法，从而覆盖了父类的实现。`$this`关键字可以访问所有的公共和受保护的修饰符，这使得访问`$content`属性变得容易。

虽然继承可能是将代码结构化为方便的父/子关系链的快速而强大的方法，但应避免滥用或过度使用。在更大的系统中，这可能特别棘手，我们可能会花更多的时间来处理大型类层次结构，而不是实际维护子系统接口。因此，我们应该谨慎使用它。

# 对象和引用

在代码中有两种传递参数的方式：

+   **按引用传递**：这是调用者和被调用者都使用相同的变量作为参数。

+   **按值传递**：这是调用者和被调用者都有自己的变量副本作为参数。如果被调用者决定更改传递参数的值，调用者将不会注意到它。

按值传递是默认的 PHP 行为，如下例所示：

```php
<?php

class Util
{
  function hello($msg)
  {
    $msg = "<p>Welcome $msg</p>";
    return $msg;
  }
}

$str = 'John';

$obj = new Util();
echo $obj->hello($str); // Welcome John

echo $str; // John

```

查看`hello()`方法的内部，我们可以看到它将`$msg`参数值重置为另一个字符串值，该值包含在 HTML 标记中。默认的 PHP 按值传递行为阻止了这种变化在方法范围之外传播。在函数定义中的参数名称之前使用`&`运算符，我们可以强制进行引用传递行为：

```php
<?php

class Util
{
  function hello(&$msg)
  {
    $msg = "<p>Welcome $msg</p>";
    return $msg;
  }
}

$str = 'John';

$obj = new Util();
echo $obj->hello($str); // Welcome John

echo $str; // Welcome John

```

能够做某事并不一定意味着我们应该这样做。应谨慎使用引用传递行为，只有在真正有很好的理由时才应该这样做。前面的例子清楚地显示了内部`hello()`方法对外部范围内的简单标量类型值的副作用。对象实例方法，甚至纯函数，不应该对外部范围产生这种类型的副作用。

一些 PHP 函数，如`sort()`，使用`&`运算符来强制对给定数组参数进行引用传递行为。

说了这么多，对象在哪里适用呢？PHP 中的对象倾向于传递引用行为。当对象作为参数传递时，它仍然被传递为值，但被传递的值不是对象本身，而是对象标识符。因此，将对象作为参数传递的行为更像是通过引用传递：

```php
<?php

class User
{
  public $salary = 4200;
}

function bonus(User $u)
{
  $u->salary = $u->salary + 500;
}

$user = new User();
echo $user->salary; // 4200
bonus($user);
echo $user->salary; // 4700 

```

由于对象比标量值更大，通过引用传递大大减少了内存和 CPU 占用。

# 对象迭代

PHP 数组是 PHP 中最常用的集合结构。我们几乎可以将任何东西都放入数组中，从标量值到对象。使用`foreach`语句轻松遍历这种结构的元素。然而，数组并不是唯一可迭代的类型，对象本身也是可迭代的。

让我们来看下面的基于数组的例子：

```php
<?php

$user = [
  'name' => 'John',
  'age' => 34,
  'salary' => 4200.00
];

foreach ($user as $k => $v) {
  echo "key: $k, value: $v" . PHP_EOL;
}

```

现在让我们来看下面的基于对象的例子：

```php
<?php

class User
{
  public $name = 'John';
  public $age = 34;
  public $salary = 4200.00;
}

$user = new User();

foreach ($user as $k => $v) {
  echo "key: $k, value: $v" . PHP_EOL;
}

```

在控制台上执行这两个例子，将得到相同的输出：

```php
key: name, value: John 
key: age, value: 34 
key: salary, value: 4200

```

默认情况下，迭代仅适用于公共属性，不包括列表中的任何私有或受保护属性。

PHP 提供了一个`Iterator`接口，使我们能够指定要使其可迭代的值。

```php
Iterator extends Traversable {
  abstract public mixed current(void)
  abstract public scalar key(void)
  abstract public void next(void)
  abstract public void rewind(void)
  abstract public boolean valid(void)
} 

```

以下示例演示了一个简单的`Iterator`接口实现：

```php
<?php

class User implements \Iterator
{
  public $name = 'John';
  private $age = 34;
  protected $salary = 4200.00;

  private $info = [];

  public function __construct()
  {
    $this->info = [
      'name' => $this->name,
      'age' => $this->age,
      'salary' => $this->salary
    ];
  }

  public function current()
  {
    return current($this->info);
  }

  public function next()
  {
    return next($this->info);
  }

  public function key()
  {
    return key($this->info);
  }

  public function valid()
  {
    $key = key($this->info);
    return ($key !== null && $key !== false);
  }

  public function rewind()
  {
    return reset($this->info);
  }
}

```

通过这种实现，我们似乎现在能够迭代 User 类的私有和受保护属性。尽管如此，实际情况并非如此。发生的是，通过构造函数，类正在用我们希望迭代的所有其他属性的数据填充`$info`参数。然后，接口规定的方法确保我们的类与`foreach`结构良好地配合。

对象迭代是 PHP 在日常开发中经常被忽视的一个很好的特性。

# 对象比较

PHP 语言提供了几个比较运算符，允许我们比较两个不同的值，结果要么是`true`，要么是`false`：

+   `==`: 等于

+   `===`: 相同

+   `!=`: 不等于

+   `<>`: 不等于

+   `!==`: 不相同

+   `<`: 小于

+   `>`: 大于

+   `<=`: 小于或等于

+   `>=`: 大于或等于

虽然所有这些运算符同样重要，让我们更仔细地看看在对象的上下文中相等（`==`）和相同（`===`）运算符的行为。

让我们来看下面的例子：

```php
<?php

class User {
  public $name = 'N/A';
  public $age = 0;
}

$user = new User();
$employee = new User();

var_dump($user == $employee); // true
var_dump($user === $employee); // false

```

在这里，我们有一个简单的`User`类，其中有两个属性设置为一些默认值。然后我们有同一个类的两个不同实例，`$user`和`$employee`。鉴于这两个对象都具有相同的属性，并且具有相同的值，相等（`==`）运算符返回`true`。另一方面，相同（`===`）运算符返回 false。尽管对象是同一个类的，且在这些属性中具有相同的属性和值，但相同运算符将它们视为不同。

让我们来看下面的例子：

```php
<?php

class User {
  public $name = 'N/A';
  public $age = 0;
}

$user = new User();
$employee = $user;

var_dump($user == $employee); // true
var_dump($user === $employee); // true

```

相同（`===`）运算符只有当两个对象引用同一个类的同一个实例时才认为它们是相同的。相同的运算符行为也适用于对应的运算符，即不等（`<>`或`!=`）和不相同（`!==`）运算符。

除了对象，相同运算符也适用于任何其他类型：

```php
<?php   var_dump(2 == 2); // true var_dump(2 == "2"); // true var_dump(2 == "2ABC"); // true   var_dump(2 === 2); // true var_dump(2 === "2"); // false var_dump(2 === "2ABC"); // false

```

从前面的例子中可以清楚地看出相同运算符的重要性。`2 == "2ABC"`表达式的计算结果为 true，这让人感到困惑。我们甚至可能认为这是 PHP 语言本身的一个 bug。虽然依赖 PHP 的自动类型转换大多数情况下都没问题，但有时会出现意外的 bug，影响我们的应用逻辑。使用相同运算符可以重新确认比较，确保我们考虑的不仅是值，还有类型。

# 特征

我们之前提到 PHP 是一种单继承语言。在 PHP 中，我们不能使用`extends`关键字来扩展多个类。这个特性实际上是一种罕见的商品，只有少数编程语言支持，比如 C++。无论好坏，多重继承允许我们对代码结构进行一些有趣的调整。

PHP Traits 提供了一种机制，通过它我们可以在代码重用或功能分组的上下文中实现这些结构。使用`trait`关键字声明 Trait，如下所示：

```php
<?php

trait Formatter
{
  // Trait body
}

```

Trait 的主体可以是我们在类中放置的任何东西。虽然它们类似于类，但我们不能实例化 Trait 本身。我们只能从另一个类中使用 Trait。为此，我们在类主体中使用`use`关键字，如下例所示：

```php
class Ups
{
  use Formatter;

  // Class body (properties & methods)
}

```

为了更好地理解 Trait 如何有助于，让我们看看以下示例：

```php
<?php   trait Formatter {
  public function formatPrice($price)
 {  return sprintf('%.2F', $price);
 } }   class Ups {
  use Formatter;   private $price = 4.4999; // Base shipping price   public function getShippingPrice($formatted = false)
 {  // Shipping cost calc... $this->price = XXX    if ($formatted) {
  return $this->formatPrice($this->price);
 }    return $this->price;
 } }   class Dhl {
  use Formatter;    private $price = 9.4999; // Base shipping price    public function getShippingPrice($formatted = false)
 {  // Shipping cost calc... $this->price = XXX    if ($formatted) {
  return $this->formatPrice($this->price);
 }    return $this->price;
 } }   $ups = new Ups(); echo $ups->getShippingPrice(true); // 4.50   $dhl = new Dhl(); echo $dhl->getShippingPrice(true); // 9.50

```

前面的例子演示了在代码重用上下文中使用 trait 的情况，其中两个不同的运输类`Ups`和`Dhl`使用相同的 trait。trait 本身包装了一个很好的`formatPrice()`辅助方法，用于将给定的价格格式化为两个小数位。

与类一样，traits 可以访问`$this`，这意味着我们可以轻松地将`Formatter` trait 的先前`formatPrice()`方法重写如下：

```php
<?php   trait Formatter {
  public function formatPrice()
 {  return sprintf('%.2F', $this->price);
 } }

```

然而，这严重限制了我们对 trait 的使用，因为它的`formatPrice()`方法现在期望有一个`$price`成员，而一些使用`Formatter` trait 的类可能没有。

让我们看另一个例子，在这个例子中，我们在功能分组的上下文中使用 traits：

```php
<?php   trait SalesOrderCustomer {
  public function getCustomerFirstname()
 { /* body */
  }    public function getCustomerEmail()
 { /* body */
  }    public function getCustomerGender()
 { /* body */
  } }   trait SalesOrderActions {
  public function cancel()
 { /* body */
  }    public function complete()
 { /* body */
  }    public function hold()
 { /* body */
  } }   class SalesOrder {
  use SalesOrderCustomer;
  use SalesOrderActions;    /* body */ }

```

我们在这里所做的不过是将我们的类代码剪切并粘贴到两个不同的 traits 中。我们将所有与可能的订单操作相关的方法分组到一个`SalesOrderActions` trait 中，将所有与订单客户相关的方法分组到`SalesOrderCustomer` trait 中。这让我们回到了可能并不一定是可取的哲学。

使用多个 traits 有时可能会导致冲突，即在多个 trait 中可以找到相同的方法名。我们可以使用`insteadof`和`as`关键字来缓解这些类型的冲突，如下例所示：

```php
<?php   trait CsvHandler {
  public function import()
 {  echo 'CsvHandler > import' . PHP_EOL;
 }   public function export()
 {  echo 'CsvHandler > export' . PHP_EOL;
 } }   trait XmlHandler {
  public function import()
 {  echo 'XmlHandler > import' . PHP_EOL;
 }    public function export()
 {  echo 'XmlHandler > export' . PHP_EOL;
 } }   class SalesOrder {
  use CsvHandler, XmlHandler {
 XmlHandler::import insteadof CsvHandler;
 CsvHandler::export insteadof XmlHandler;
 XmlHandler::export as exp;
 }    public function initImport()
 {  $this->import();
 }    public function initExport()
 {  $this->export();
  $this->exp();
 } }   $order = new SalesOrder(); $order->initImport(); $order->initExport();   //XmlHandler > import //CsvHandler > export //XmlHandler > export

```

`as`关键字也可以与`public`、`protected`或`private`关键字一起使用，以更改方法的可见性：

```php
<?php   trait Message {
  private function hello()
 {  return 'Hello!';
 } }   class User {
  use Message {
 hello as public;
 } }   $user = new User(); echo $user->hello(); // Hello!

```

更有趣的是，traits 可以进一步由其他 traits 组成，甚至支持`abstract`和`static`成员，如下例所示：

```php
<?php

trait A
{
  public static $counter = 0;

  public function theA()
  {
    return self::$counter;
  }
}

trait B
{
  use A;

  abstract public function theB();
}

class C
{
  use B;

  public function theB()
  {
    return self::$counter;
  }
}

$c = new C();
$c::$counter++;
echo $c->theA(); // 1
$c::$counter++;
$c::$counter++;
echo $c->theB(); // 3

```

除了不能实例化外，traits 与类共享许多特性。虽然它们为我们提供了一些有趣的代码结构工具，但它们也很容易违反单一责任原则。对 trait 使用的整体印象通常是扩展常规类，这使得很难找到正确的用例。我们可以使用它们来描述许多但不是必要的特征。例如，喷气发动机并非每架飞机都必需，但很多飞机都有它们，而其他飞机可能有螺旋桨。

# 反射

反射是每个开发人员都应该警惕的一个非常重要的概念。它表示程序在运行时检查自身的能力，从而允许轻松地反向工程类、接口、函数、方法和扩展。

我们可以从控制台快速了解 PHP 反射的能力。PHP CLI 支持几个基于反射的命令：

+   `--rf <*function name*>`：显示有关函数的信息

+   `--rc <*class name*>`：显示有关类的信息

+   `--re <*extension name*>`：显示有关扩展的信息

+   `--rz <*extension name*>`：显示有关 Zend 扩展的信息

+   `--ri <*extension name*>`：显示扩展的配置

以下输出演示了`php --rf str_replace`命令的结果：

```php
Function [ <internal:standard> function str_replace ] {
  - Parameters [4] {
     Parameter #0 [ <required> $search ]
     Parameter #1 [ <required> $replace ]
     Parameter #2 [ <required> $subject ]
     Parameter #3 [ <optional> &$replace_count ]
  }
}

```

输出反映了`str_replace()`函数，这是一个标准的 PHP 函数。它清楚地描述了参数的总数，以及它们的名称和必需或可选的分配。

反射的真正力量，开发人员可以利用的力量，来自反射 API。让我们看看以下例子：

```php
<?php

class User
{
  public $name = 'John';
  protected $ssn = 'AAA-GG-SSSS';
  private $salary = 4200.00;
}

$user = new User();

echo $user->name = 'Marc'; // Marc

//echo $user->ssn = 'BBB-GG-SSSS';
// Uncaught Error: Cannot access protected property User::$ssn

//echo $user->salary = 5600.00;
// Uncaught Error: Cannot access private property User::$salary

var_dump($user);
//object(User)[1]
// public 'name' => string 'Marc' (length=4)
// protected 'ssn' => string 'AAA-GG-SSSS' (length=11)
// private 'salary' => float 4200

```

我们首先定义了一个`User`类，其中包含三个不同可见性的属性。然后我们实例化了一个`User`类的对象，并尝试更改所有三个属性的值。通常，定义为`protected`或`private`的成员不能在对象外部访问。尝试以读取或写入模式访问它们会抛出一个无法访问的错误。这是我们认为的正常行为。

使用 PHP 反射 API，我们可以绕过这种正常行为，从而可以访问私有和受保护的成员。反射 API 本身为我们提供了几个可用的类：

+   反射

+   ReflectionClass

+   ReflectionZendExtension

+   ReflectionExtension

+   ReflectionFunction

+   ReflectionFunctionAbstract

+   ReflectionMethod

+   ReflectionObject

+   ReflectionParameter

+   ReflectionProperty

+   ReflectionType

+   ReflectionGenerator

+   Reflector（接口）

+   ReflectionException（异常）

这些类中的每一个都公开了各种功能，使我们能够玩弄其他类、接口、函数、方法和扩展的内部。假设我们的目标是从前面的例子中更改`protected`和`private`属性的值，我们可以使用`ReflectionClass`和`ReflectionProperty`，如下例所示：

```php
<?php

// ...

$user = new User();

$reflector = new ReflectionClass('User');

foreach ($reflector->getProperties() as $prop) {
  $prop->setAccessible(true);
  if ($prop->getName() == 'name') $prop->setValue($user, 'Alice');
  if ($prop->getName() == 'ssn') $prop->setValue($user, 'CCC-GG-SSSS');
  if ($prop->getName() == 'salary') $prop->setValue($user, 2600.00);
}

var_dump($user);

//object(User)[1]
// public 'name' => string 'Alice' (length=5)
// protected 'ssn' => string 'CCC-GG-SSSS' (length=11)
// private 'salary' => float 2600

```

我们首先实例化了一个`User`类的对象，就像在前面的例子中一样。然后我们创建了一个`ReflectionClass`的实例，将`User`类的名称传递给它的构造函数。新创建的`$reflector`实例允许我们通过其`getProperties()`方法获取`User`类的所有属性列表。逐个循环遍历属性，我们启动了反射 API 的真正魔力。每个属性（`$prop`）都是`ReflectionProperty`类的一个实例。`ReflectionProperty`类的两个方法，`setAccessible()`和`setValue()`，为我们提供了恰到好处的功能，使我们能够达到我们的目标。使用这些方法，我们能够设置原本无法访问的对象属性的值。

另一个简单但有趣的反射示例是文档注释提取：

```php
<?php

class Calc
{
  /**
  * @param $x The number x
  * @param $y The number y
  * @return mixed The number z
  */
  public function sum($x, $y)
  {
    return $x + $y;
  }
}

$calc = new Calc();

$reflector = new ReflectionClass('Calc');
$comment = $reflector->getMethod('sum')->getDocComment();

echo $comment;

```

仅用两行代码，我们就能够反映`Calc`类并从其`sum()`方法中提取文档注释。虽然反射 API 的实际用途可能一开始并不明显，但正是这些功能使我们能够构建强大而动态的库和平台。

phpDocumentor 工具使用 PHP 反射功能自动生成源代码的文档。流行的 Magento v2.x 电子商务平台广泛使用 PHP 反射功能自动实例化被`__construct()`参数类型提示的对象。

# 总结

在本章中，我们看了一些 PHP 面向对象编程中最基本但不太为人知的特性，这些特性有时在我们日常开发中并没有得到足够的关注。如今，大多数主流工作都集中在使用框架和平台，这些框架和平台往往会将一些概念抽象化。了解对象的内部工作对于成功开发和调试更大型的系统至关重要。反射 API 在操作对象时提供了很大的力量。结合我们在第四章中提到的魔术方法的力量，*魔术方法背后的魔术*，PHP 面向对象模型似乎相当丰富多彩。

接下来，我们将假设我们已经有一个可用的应用程序，并专注于优化其性能。
