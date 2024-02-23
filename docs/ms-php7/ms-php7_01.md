# 全新的 PHP

如今编程语言不胜枚举。新的语言不时地出现。选择适合工作的语言远不止是其功能清单的一部分。有些针对特定的问题领域，其他则试图定位更广泛的使用。这说明软件开发是一个动态的生态系统，语言需要不断适应不断变化的行业，以保持对其消费者的相关性。这些变化对于已经建立的语言（如 PHP）尤其具有挑战性，因为向后兼容性是一个重要的考虑因素。

PHP 最初由 Rasmus Lerdorf 于 1995 年左右创建，起初只是用 C 语言编写的一些 CGI 程序。那时，它是一个简单的脚本解决方案，使开发人员能够轻松构建动态 HTML 页面。无需编译，开发人员可以轻松地将几行代码放入文件中，并在浏览器中查看结果。这使得它早期非常受欢迎。二十年后，PHP 发展成为一个适用于 Web 开发的丰富通用脚本语言。在这些年里，PHP 成功地在每个新版本中提供了令人印象深刻的功能集，同时保持了可靠的向后兼容性水平。如今，其大量的核心扩展最终简化了与文件、会话、Cookie、数据库、Web 服务、加密和许多其他 Web 开发常见功能的工作。它对面向对象编程（OOP）范式的出色支持使其真正与其他领先的行业语言竞争。

PHP 5 十年的统治在 2015 年 12 月 PHP 7 的发布中被推翻。它带来了全新的执行引擎 Zend Engine 3.0，显著提高了性能并减少了内存消耗。这个简单的软件更新现在使我们能够为更多并发用户提供服务，而无需添加任何物理硬件。开发者的接受程度几乎是瞬间的，尤其是因为向后不兼容性很小，使得迁移尽可能轻松。

在本章中，我们将详细了解 PHP 7 和 7.1 版本中引入的一些新功能：

+   标量类型提示

+   返回类型提示

+   匿名类

+   生成器委托

+   生成器返回表达式

+   空合并运算符

+   太空船操作符

+   常量数组

+   统一的变量语法

+   可抛出的

+   组使用声明

+   类常量可见性修饰符

+   捕获多个异常类型

+   可迭代伪类型

+   可空类型

+   无返回类型

正是这些特性注定会在下一代 PHP 框架和库以及我们编写自己的代码的方式上留下深刻印记。

# 标量类型提示

按分类，PHP 是一种动态类型和弱类型的语言。这是两个经常混在一起的不同概念。动态类型的语言不需要在使用之前显式声明变量。弱类型的语言是指变量不属于任何特定的数据类型，也就是说，它的类型可以通过不同的值类型重新分配而改变。

让我们看看以下例子：

```php
// dynamic typed (no specific type defined, directly assigning value)
$name = "Branko"; // string
$salary = 4200.00; // float
$age = 33; // int

// weak typed (variable value reassigned into different type)
$salary = 4200.00; // float
$salary = $salary + "USD"; // float
$salary = $salary . "USD"; // string

```

在上述代码中，我们看到使用了三个不同的变量，其中没有一个预定义为特定类型。我们只是将值声明到它们中。PHP 然后在运行时确定类型。即使确定了变量类型，也可以通过简单地分配另一种类型的值来更改它。这是两个非常强大的概念，当明智地使用时，可以为我们节省大量代码行。

然而，这些强大的特性往往间接地鼓励了不良设计。这在编写函数时特别明显，要么是通过强制函数设计者进行多个数据类型检查，要么是强制他们进行多个函数返回类型。

让我们看看以下例子：

```php
function addTab($tab) {
  if (is_array($tab)) {

  } elseif (is_object($tab)) {

  } elseif (is_string($tab)) {

  } else {

  } 
}

```

考虑到输入参数的类型不确定性，`addTab`函数被迫分支其逻辑。同样，同一个函数可能决定根据逻辑分支返回不同类型的数据。这样的设计通常是因为函数试图做太多事情。真正的问题甚至不在函数本身，而是在使用函数的开发人员那一边。如果发生开发人员对传递参数类型不够了解，可能会导致意外的结果。

为了帮助我们编写更正确和自我描述的程序，PHP 引入了**类型提示**。

PHP 从 5.0 版本开始支持函数参数类型提示，但仅限于对象，从 5.1 版本开始也支持数组。PHP 7 开始，标量类型也可以进行类型提示，这使其成为该版本中更令人兴奋的功能之一。以下是 PHP 支持的标量类型提示：

+   `int`

+   `float`

+   `string`

+   `bool`

现在我们可以以以下两种方式编写函数：

+   可以是`function register($email, $age, $notify) { /* ... */}`

+   可以是`function register($email, int $age, $notify) { /* ... */}`

+   可以是`function register(string $email, int $age, bool $notify) { /* ... */}`

然而，仅仅对标量类型进行提示是不够的，因为类型声明默认情况下不会被强制执行。PHP 会尝试将其转换为指定的类型而不会抱怨。通过在 PHP 文件的第一条语句中添加`declare(strict_types=1);`指令，我们可以强制执行严格的类型检查行为。值得注意的是，该指令只影响它所在的特定文件，并不影响其他包含的文件。**文件级别**指令用于保持与众多扩展和内置 PHP 函数的向后兼容性。

让我们看下面的例子：

```php
declare(strict_types=1);

function register(string $email, int $age, bool $notify) {
 // body
}

register('user@mail.com', '33', true);

```

打开严格类型指令后，尝试将不正确的数据类型传递给提示的标量参数将导致`\TypeError`异常，如下所示：

```php
Fatal error: Uncaught TypeError: Argument 2 passed to register() must be of the type integer, string given, called in /test.php on line 11 and defined in /test.php:5 Stack trace: #0 /test.php(11): register('user@mail.co...', '33', true) #1 {main} thrown in /test.php on line 5.

```

标量类型提示是 PHP 语言的一个强大的新功能。它们在运行时为开发人员提供了额外的保护层，而不会真正牺牲一般的弱类型系统。

# 返回类型提示

类型提示功能不仅限于函数参数；从 PHP 7 开始，它们还扩展到函数返回值。适用于函数参数提示的规则也适用于函数返回类型提示。要指定函数返回类型，我们只需在参数列表后面加上冒号和返回类型，如下例所示：

```php
function register(string $user, int $age) : bool {
  // logic ...
  return true;
}

```

开发人员仍然可以编写带有多个条件`return`语句的函数；只是在这种情况下，每个达到的`return`语句都必须匹配提示的返回类型，否则会抛出`\TypeError`。

函数返回类型提示与超类型很好地配合。让我们看下面的例子：

```php
class A {}
class B extends A {}
class C extends B {}

function getInstance(string $type) : A {
    if ($type == 'A') {
       return new A();
       } elseif ($type == 'B') {
           return new B();
       } else {
           return new C();
       }
  }

getInstance('A'); #object(A)#1 (0) { }
getInstance('B'); #object(B)#1 (0) { }
getInstance('XYZ'); #object(C)#1 (0) { }

```

我们看到该函数对所有三种类型都执行得很好。鉴于`B`直接扩展了`A`，而`C`又扩展了`B`，该函数接受它们作为返回值。

考虑到 PHP 的动态特性，函数返回类型可能一开始看起来似乎是朝错误的方向迈出的一步，更何况因为很多 PHP 代码已经使用了 PHPDoc 的`@return`注释，这与现代 IDE 工具（如 PhpStorm）很好地配合。然而，`@return`注释只是提供信息，它在运行时并不强制实际返回类型，而且只有在强大的 IDE 中才有意义。使用函数返回类型提示可以确保我们的函数返回我们打算返回的内容。它们并不妨碍 PHP 的动态特性；它们只是从函数使用者的角度丰富了它。

# 匿名类

从类中实例化对象是一个非常简单的操作。我们使用`new`关键字，后面跟着类名和可能的构造函数参数。类名部分意味着之前定义的类的存在。虽然很少见，但有些情况下类只在执行期间使用。这些罕见的情况使得在我们知道类只被使用一次时，强制单独定义一个类变得冗长。为了解决这种冗长的挑战，PHP 引入了一个名为**匿名类**的新功能。虽然匿名类的概念在其他语言中已经存在了相当长的时间，但 PHP 在 PHP 7 版本中才引入了它。

匿名类的语法非常简单，如下所示：

```php
$obj = new class() {};
$obj2 = new class($a, $b) {
   private $a;
   private $b;
   public function __construct($a, $b) {
     $this->a = $a;
     $this->b = $b;
   }
};

```

我们使用`new`关键字，后面跟着`class`关键字，然后是可选的构造函数参数，最后是用大括号包裹的类体。两个对象都被实例化为`class@anonymous`类型。通过匿名类实例化的对象的功能与通过命名类实例化的对象没有区别。

与命名类相比，匿名类几乎是相等的，它们可以传递构造函数参数，扩展其他类，实现接口，并使用特性。然而，匿名类不能被序列化。尝试序列化匿名类的实例，如下面的代码片段所示，会抛出一个致命的`Serialization of class@anonymous is not allowed…`错误。

在使用匿名类时，需要牢记一些注意事项。在另一个类中嵌套匿名类会隐藏该外部类的私有和受保护的方法或属性。为了规避这个限制，我们可以将外部类的私有和受保护的属性传递到匿名类的构造函数中，如下所示：

```php
interface Salary {
      public function pay();
   }

   trait Util {
      public function format(float $number) {
         return number_format($number, 2);
      }
   }

   class User {
      private $IBAN;
      protected $salary;
      public function __construct($IBAN, $salary) {
         $this->IBAN = $IBAN;
         $this->salary = $salary;
      }

      function salary() {
       return new class($this->IBAN, $this->salary) implements Salary {
         use Util;
         private $_IBAN;
         protected $_salary;

         public function __construct($IBAN, $salary) {
            $this->_IBAN = $IBAN;
            $this->_salary = $salary;
         }

        public function pay() {
           echo $this->_IBAN . ' ' . $this->format($this->_salary);
        }
     };
   } 
 }
 $user = new User('GB29NWBK60161331926819', 4500.00);
 $user->salary()->pay();

```

在这个简化的`User`类示例中，我们有一个返回匿名类的`salary`方法。为了展示匿名类更强大的用法，我们让它实现`Salary`接口并使用`Util`特性。`Salary`接口强制匿名类实现`pay`方法。我们的`pay`方法的实现需要外部类的`IBAN`和`salary`成员值。由于匿名类不允许访问外部类的私有和受保护成员，我们通过匿名类构造函数传递这些值。虽然整体示例当然不反映出良好的类设计概念，但它展示了如何绕过成员可见性限制。

匿名类还有一个选项，可以通过扩展外部类本身来获取外部类的私有和受保护成员。然而，这需要匿名类的构造函数正确实例化外部类；否则，我们可能会遇到警告，比如`User::__construct()`缺少参数。

尽管它们没有名字，匿名类仍然有一个内部名称。在匿名类的实例上使用核心 PHP `get_class`方法，可以得到这个名称，如下面的例子所示：

```php
class User {}
class Salary {}

function gen() {
  return new class() {};
}

$obj = new class() {};
$obj2 = new class() {};
$obj3 = new class() extends User {};
$obj4 = new class() extends Salary {};
$obj5 = gen();
$obj6 = gen();

echo get_class($obj); // class@anonymous/var/www/index.php0x27fe03a
echo get_class($obj2); // class@anonymous/var/www/index.php0x27fe052
echo get_class($obj3); // class@anonymous/var/www/index.php0x27fe077
echo get_class($obj4); // class@anonymous/var/www/index.php0x27fe09e
echo get_class($obj5); // class@anonymous/var/www/index.php0x27fe04f
echo get_class($obj6); // class@anonymous/var/www/index.php0x27fe04f

for ($i=0; $i<=5; $i++) {
  echo get_class(new class() {}); // 5 x   
    class@anonymous/var/www/index.php0x27fe2d3
}

```

观察这些输出，我们可以看到在相同位置（函数或循环）创建的匿名类将产生相同的内部名称。具有相同名称的匿名类对等号（`==`）运算符返回`true`，对身份运算符（`===`）返回`false`，这是一个重要的考虑因素，以避免潜在的错误。

对匿名类的支持为一些有趣的用例打开了大门，比如模拟测试和进行内联类覆盖，这两者在明智使用时可以提高代码质量和可读性。

# 生成器委托

在任何编程语言中，遍历项目列表是最常见的事情之一。PHP 通过`foreach`结构使得遍历各种数据集合变得容易。许多语言区分各种类型的集合数据，如字典、列表、集合、元组等。然而，PHP 并不过多关注数据结构，大多数情况下简单地使用`array()`或`[]`结构来表示集合。这反过来可能会对创建大型数组在内存中产生负面影响，可能导致超出内存限制甚至增加处理时间。

除了原始的*array*类型外，PHP 还提供了`ArrayObject`和`ArrayIterator`类。这些类将数组转变为面向对象应用程序中的一等公民。

生成器允许我们编写使用`foreach`来遍历一组数据而无需构建数组的代码。它们就像一个产出尽可能多值的函数，而不是只返回一个值，这使它们具有类似迭代器的行为。虽然生成器从 PHP 5.5 就存在，但它们缺乏更高级的功能。**生成器委托**是 PHP 7 发布后提供的改进之一。

让我们看下面的例子：

```php
function even() {
   for ($i = 1; $i <= 10; $i++) {
     if ($i % 2 == 0) {
        yield $i;
     }
   }
}

function odd() {
    for ($i = 1; $i <= 10; $i++) {
       if ($i % 2 != 0) {
          yield $i;
       }
    }
}

function mix() {
   yield -1;
   yield from odd();
   yield 17;
   yield from even();
   yield 33;
}

// 2 4 6 8 1 0
foreach (even() as $even) {
  echo $even;
}

// 1 3 5 7 9
foreach (odd() as $odd) {
  echo $odd;
}

// -1 1 3 5 7 9 17 2 4 6 8 10 33
foreach (mix() as $mix) {
  echo $mix;
}

```

在这里，我们定义了三个生成器函数：`even`、`odd`和`mix`。`mix`函数通过使用`yield` from `<expr>`演示了生成器委托的概念。而`<expr>`是任何评估为可遍历对象或数组的表达式。我们可以看到通过循环遍历`mix`函数的结果，会输出它自身以及`even`和`odd`函数的所有产出值。

生成器委托语法允许将`yield`语句分解为更小的概念单元，使生成器具有类似方法对类的组织功能。谨慎使用时，这可以提高我们的代码质量和可读性。

# 生成器返回表达式

尽管 PHP 5.5 通过引入生成器函数功能丰富了语言，但它缺乏`return`表达式以及它们的产出值。生成器函数无法指定返回值的能力限制了它们在协程中的实用性。PHP 7 版本通过添加对`return`表达式的支持解决了这个限制。生成器基本上是可中断的函数，其中`yield`语句标志着中断点。让我们来看一个简单的生成器，以自调用匿名函数的形式编写：

```php
$letters = (function () {
  yield 'A';
  yield 'B';
  return 'C';
})();

// Outputs: A B
foreach ($letters as $letter) {
  echo $letter;
}

// Outputs: C
echo $letters->getReturn();

```

尽管`$letters`变量被定义为自调用匿名函数，但`yield`语句阻止了立即函数执行，将函数转变为生成器。生成器本身保持静止，直到我们尝试对其进行迭代。一旦迭代开始，生成器产出值`A`，然后是值`B`，但不是`C`。这意味着在`foreach`结构中使用时，迭代将仅包括产出值，而不是返回值。一旦迭代完成，我们可以调用`getReturn()`方法来检索实际的返回值。在迭代生成器结果之前调用`getReturn()`方法无法获取未返回异常的生成器的返回值。

生成器的好处在于它们不是单向通道；它们不仅限于产出值，还可以接受值。作为`\Generator`类的实例，它们可以使用几个有用的方法，其中两个是`getReturn`和`send`。`send`方法使我们能够将值发送回生成器，将生成器与调用者之间的单向通信转变为双向通道，有效地将生成器转变为协程。添加`getReturn`方法赋予生成器`return`语句，为协程提供更灵活的功能。

# 空合并运算符

在 PHP 中使用变量非常容易。变量的声明和初始化是通过单个表达式完成的。例如，表达式`$user['name'] = 'John';`将自动声明类型为数组的变量`$user`，并初始化该数组，其中包含一个键名为`name`，值为`John`。

日常开发通常包括检查各种分支决策的变量值的存在，比如`if ($user['name'] =='John') { … } else { … }`。当我们自己编写代码时，我们倾向于确保我们的代码不使用未声明的变量和未初始化的数组键。然而，有时变量来自外部，因此我们无法保证它们在运行时的存在。在`$user`未设置或设置但键不是 name 时调用`$user['name']`将导致未定义索引的通知--`name`。像代码中的任何意外状态一样，通知是不好的，更糟糕的是它们实际上不会破坏你的代码，而是允许它继续执行。当发生通知时，除非我们将`display_errors`配置设置为`true`，并配置错误报告以显示`E_ALL`，否则我们甚至不会在浏览器中看到通知。

这是不好的，因为我们可能依赖不存在的变量和它们的值。这种依赖甚至可能没有在我们的代码中处理，我们甚至不会注意到，因为代码将继续执行，除非放置了特定的变量检查。

PHP 语言有一定数量的预定义变量，称为**超全局变量**，我们可以从任何函数、类或文件中使用它们，而不受范围的限制。最常用的可能是`$_POST`和`$_GET`超全局变量，它们用于获取通过表单或 URL 参数提交的数据。由于我们无法保证在这种情况下`$_GET['name']`的存在，因此我们需要检查它。通常，这是通过 PHP 中的`isset`和`empty`函数来完成的，如下面的代码块所示：

```php
// #1
if (isset($_GET['name']) && !empty($_GET['name'])) 
   {
     $name = $_GET['name'];
   } 
else {
     $name = 'N/A';
     }

// #2
if (!empty($_GET['name'])) 
   {
     $name = $_GET['name'];
   } 
else {
       $name = 'N/A';
     }

// #3

$name = ((isset($_GET['name']) && !empty($_GET['name']))) ? $_GET['name'] : 'N/A';

// #4
$name = (!empty($_GET['name'])) ? $_GET['name'] : 'N/A';

```

第一个示例是最健壮的，因为它同时使用了`isset`和`empty`函数。这些函数并不相同，因此了解它们各自的功能是很重要的。`empty`函数的好处是，如果我们尝试传递一个可能未设置的变量给它，比如`$_GET['name']`，它不会触发通知，而是简单地返回`true`或`false`。这使得`empty`函数在大多数情况下都是一个不错的辅助工具。然而，即使是第四个示例，通过使用三元运算符编写，也是相当健壮的。

PHP 7 引入了一种新类型的运算符，称为**null coalesce**（`??`）运算符。它赋予我们编写更短表达式的能力。下面的示例演示了它的使用优雅之处：

```php
$name = $_GET['name'] ?? 'N/A';

```

如果第一个操作数存在且不为 null，则返回其结果，否则返回第二个操作数。换句话说，从左到右读取，将返回第一个存在且不为 null 的值。

# 太空船操作符

比较两个值是任何编程语言中频繁的操作。我们使用各种语言运算符来表示我们希望在两个变量之间执行的比较类型。在 PHP 中，这些运算符包括相等（`$a == $b`），全等（`$a === $b`），不相等（`$a != $b`或`$a <> $b`），不全等（`$a !== $b`），小于（`$a < $b`），大于（`$a > $b`），小于或等于（`$a <= $b`），和大于或等于（`$a >= $b`）比较。

所有这些比较运算符的结果都是布尔值`true`或`false`。然而，有时候存在需要进行三路比较的情况，在这种情况下，比较的结果不仅仅是布尔值`true`或`false`。虽然我们可以通过各种表达式使用各种运算符来实现三路比较，但解决方案却并不优雅。

随着 PHP 7 的发布，引入了一个新的太空船`<=>`运算符，其语法如下：

```php
(expr) <=> (expr)

```

太空船`<=>`运算符提供了组合比较。比较后，它遵循以下条件：

+   如果两个操作数相等，则返回`0`

+   如果左操作数大，则返回`1`

+   如果右操作数大，则返回`-1`

用于产生上述结果的比较规则与现有比较运算符使用的规则相同：`<`、`<=`、`==`、`>=`和`>`。

新运算符的实用性在排序函数中尤为明显。没有它，排序函数就会变得相当复杂，如下例所示：

```php
$users = ['branko', 'ivana', 'luka', 'ivano'];

usort($users, function ($a, $b) {
  return ($a < $b) ? -1 : (($a > $b) ? 1 : 0);
});

```

我们可以通过应用新的运算符来缩短上面的例子，如下所示：

```php
$users = ['branko', 'ivana', 'luka', 'ivano'];

usort($users, function ($a, $b) {
  return $a <=> $b;
});

```

应用太空船`<=>`运算符（如果适用）可以使表达式简洁而优雅。

# 常量数组

PHP 中有两种常量，**常量**和**类常量**。常量可以在几乎任何地方使用定义构造定义，而`class`常量是使用`const`关键字在各个类或接口中定义的。

虽然我们不能说一种常量类型比另一种更重要，但 PHP 5.6 通过允许具有数组数据类型的类常量来区分这两种类型。除了这种差异，这两种类型的常量都支持标量值（整数、浮点数、字符串、布尔值或 null）。

PHP 7 发布通过将数组数据类型添加到常量中来解决了这种不平等，使以下表达式成为有效表达式：

```php
// The class constant - using 'const' keyword
class Rift {
  const APP = [
    'name' => 'Rift',
    'edition' => 'Community',
    'version' => '2.1.2',
    'licence' => 'OSL'
  ];
}

// The class constant - using 'const' keyword
interface IRift {
  const APP = [
    'name' => 'Rift',
    'edition' => 'Community',
    'version' => '2.1.2',
    'licence' => 'OSL'
  ];
}

// The constant - using 'define' construct
define('APP', [
  'name' => 'Rift',
  'edition' => 'Community',
  'version' => '2.1.2',
  'licence' => 'OSL'
]);

echo Rift::APP['version'];
echo IRift::APP['version'];
echo APP['version'];

```

尽管具有数组数据类型的常量可能不是一种令人兴奋的功能，但它为整体常量使用增添了一定的风味。

# 统一变量语法

新的变量语法可能是 PHP 7 发布中最具影响力的功能之一。它为变量解引用带来了更大的秩序。然而，影响部分不仅对更好的变化产生影响，它还引入了某些**向后兼容性**（**BC**）破坏。这些变化的主要原因之一是与*变量变量*语法的不一致性。

观察`$foo['bar']->baz`表达式，首先获取一个名为`$foo`的变量，然后从结果中取出`bar`偏移量，最后访问`baz`属性。这是正常的变量访问解释，从左到右。然而，*变量变量*语法违反了这个原则。观察`$$foo['baz']`变量，首先获取`$foo`，然后是它的`baz`偏移量，最后查找结果名称的变量。

新引入的统一变量语法解决了这些不一致性，如下例所示：

```php
/*** expression syntax ***/
$$foo['bar']['baz']

// PHP 5.x meaning
${$foo['bar']['baz']}

// PHP 7.x meaning
($$foo)['bar']['baz']

/*** expression syntax ***/
$foo->$bar['baz']

// PHP 5.x meaning
$foo->{$bar['baz']}

// PHP 7.x meaning
($foo->$bar)['baz']

/*** expression syntax ***/
$foo->$bar['baz']()

// PHP 5.x meaning
$foo->{$bar['baz']}()

// PHP 7.x meaning
($foo->$bar)['baz']()

/*** expression syntax ***/
Foo::$bar['baz']()

// PHP 5.x meaning
Foo::{$bar['baz']}()

// PHP 7.x meaning
(Foo::$bar)['baz']()

```

除了解决上述的不一致性，还添加了几种新的语法组合，使以下表达式现在有效：

```php
$foo()['bar']();
[$obj1, $obj2][0]->prop;
getStr(){0}
$foo['bar']::$baz;
$foo::$bar::$baz;
$foo->bar()::baz()
// Assuming extension that implements actual toLower behavior
"PHP"->toLower();
[$obj, 'method']();
'Foo'::$bar;

```

这里有很多不同的语法。虽然其中一些可能看起来令人不知所措，难以找到用途，但它为新的思维方式和代码使用打开了一扇门。

# 可抛出的

PHP 中的异常并不是一个新概念。自从 PHP 5 发布以来，它们一直存在。然而，它们并没有包括 PHP 所有的错误处理，因为错误并不被视为异常。当时的 PHP 有两种错误处理系统。这使得处理起来很棘手，因为传统错误无法通过`try...catch`块捕获异常。某些技巧是可能的，其中一个可以使用`set_error_handler()`函数来设置一个用户定义的错误处理程序函数，基本上监听错误并将其转换为异常。

让我们看下面的例子：

```php
<?php class Mailer {
  private $transport;

  public function __construct(Transport $transport)
 {  $this->transport = $transport;
 } } $transport = new stdClass();  try {
  $mailer = new Mailer($transport); } catch (\Exception  $e) {
  echo 'Caught!'; } finally {
  echo 'Cleanup!'; }

```

PHP 5 将无法捕获这个错误，而是抛出`可捕获的致命错误`，如下所示：

```php
Catchable fatal error: Argument 1 passed to Mailer::__construct() must be an instance of Transport, instance of stdClass given, called in /index.php on line 18 and defined in /index.php on line 6.

```

通过在此代码之前添加`set_error_handler()`的实现，我们可以将致命错误转换为异常：

```php
set_error_handler(function ($errno, $errstr) {
  throw new \Exception($errstr, $errno);
});

```

有了上述代码，`try...catch...finally`块现在会按预期启动。然而，有一些错误类型无法通过`set_error_handler`捕获，例如`E_ERROR`、`E_PARSE`、`E_CORE_ERROR`、`E_CORE_WARNING`、`E_COMPILE_ERROR`、`E_COMPILE_WARNING`，以及在调用`set_error_handler`的文件中引发的大多数`E_STRICT`。

PHP 7 发布通过引入`Throwable`接口和将错误和异常移至其下，改进了整体的错误处理系统。它现在是通过`throw`语句抛出的任何对象的基本接口。虽然我们不能直接扩展它，但我们可以扩展`\Exception`和`\Error`类。`\Exception`是所有 PHP 和用户异常的基类，`\Error`是所有内部 PHP 错误的基类。

我们现在可以轻松地将我们之前的`try...catch...finally`块重写为以下之一：

```php
<?php   // Case 1 try {
  $mailer = new Mailer($transport); } catch (\Throwable $e) {
  echo 'Caught!'; } finally {
  echo 'Cleanup!'; }   // Case 2 try {
  $mailer = new Mailer($transport); } catch (\Error $e) {
  echo 'Caught!'; } finally {
  echo 'Cleanup!'; }

```

注意在第一个示例的`catch`块中使用了`\Throwable`。尽管我们不能扩展它，但我们可以将其用作在单个`catch`语句中捕获`\Error`和`\Exception`的简写。

实现`\Throwable`带来了非常需要的错误和异常之间的对齐，使得它们更容易理解。

# 组使用声明

PHP 在 5.3 版本中引入了命名空间。它提供了一种将相关类、接口、函数和常量分组的方式，从而使我们的代码库更有组织和可读。然而，处理现代库通常涉及大量冗长的`use`语句，用于从各种命名空间导入类，如下例所示：

```php
use Magento\Backend\Block\Widget\Grid;
use Magento\Backend\Block\Widget\Grid\Column;
use Magento\Backend\Block\Widget\Grid\Extended;

```

为了解决这种冗长，PHP 7 发布引入了组使用声明，允许以下语法：

```php
use Magento\Backend\Block\Widget\Grid;
use Magento\Backend\Block\Widget\Grid\{
  Column,
  Extended
};

```

在这里，我们将`Column`和`Extend`压缩到一个声明下。我们可以进一步使用以下复合命名空间来压缩这个：

```php
use Magento\Backend\Block\Widget\{
  Grid
  Grid\Column,
  Grid\Extended
};

```

组使用声明充当缩写，使得以简洁的方式导入类、常量和函数稍微更容易。尽管它们的好处似乎有些边缘，但它们的使用是完全可选的。

# 捕获多个异常类型

引入了可抛出对象后，PHP 基本上围绕错误检测、报告和处理进行了调整。开发人员可以使用`try...catch...finally`块根据自己的意愿处理异常。使用多个`catch`块可以更好地控制对某些类型异常的响应。然而，有时我们希望对一组异常做出相同的响应。在 PHP 7.1 中，异常处理进一步得到了改进以适应这一挑战。

让我们看一下以下的 PHP 5.x 示例：

```php
try {
      // ...
    } 
catch (\InvalidArgumentException $e) 
    {
      // ...
    } 
catch (\LengthException $e)
    {
      // ...
    }
catch (Exception $e) 
   {
     // ...
   } 
finally 
  {
    // ...
  }

```

在这里，我们处理了三种异常，其中两种异常非常具体，第三种异常是在前两种异常不匹配时捕获。`finally`块只是一个清理，如果需要的话。现在想象一下，对于`\InvalidArgumentException`和`\LengthException`块，需要相同的响应。解决方案要么是将一个异常块中的整个代码块复制到另一个异常块中，要么是最好的情况下编写一个包装响应代码的函数，然后在每个异常块中调用该函数。

新增的异常处理语法可以捕获多个异常类型。通过使用单个竖线(`|`)，我们可以为`catch`参数定义多个异常类型，如下所示的 PHP 7.x 示例：

```php
try {
      // ...
    } 
catch (\InvalidArgumentException | \LengthException $e)
   {
     // ...
   }  
catch (\Exception $e) 
   {
     // ...
   }
 finally 
   {
     // ...
   }

```

除了一丝优雅外，新的语法直接影响了代码重用的效果更好。

# 类常量可见性修饰符

PHP 中有五种访问修饰符：`public`、`private`、`protected`、`abstract`和`final`。通常称为**可见性修饰符**，它们并非都同样适用。它们的使用分布在类、函数和变量之间，如下所示：

+   **函数**：`public`、`private`、`protected`、`abstract`和`final`

+   **类**：`abstract`和`final`

+   **变量**：`public`、`private`和`protected`

然而，类常量不在此列表中。PHP 的旧版本不允许在类常量上使用可见性修饰符。默认情况下，类常量仅被分配为公共可见性。

PHP 7.1 版本通过引入`public`、`private`和`protected`类常量可见性修饰符来解决了这个限制，如下例所示：

```php
class Visibility 
 {
   // Constants without defined visibility
   const THE_DEFAULT_PUBLIC_CONST = 'PHP';

   // Constants with defined visibility
   private const THE_PRIVATE_CONST = 'PHP';
   protected const THE_PROTECTED_CONST = 'PHP';
   public const THE_PUBLIC_CONST = 'PHP';
 }

```

与旧行为类似，没有明确可见性的类常量默认为`public`。

# 可迭代伪类型

在 PHP 中，函数通常接受或返回一个数组或实现`\Traversable`接口的对象。虽然这两种类型都可以在`foreach`结构中使用，但从根本上说，数组是一种原始类型；对象不是。这使得函数难以理解这些类型的迭代参数和返回值。

PHP 7.1 通过引入可迭代伪类型来解决这个问题。其想法是在参数或返回类型上使用它作为类型声明，以指示该值是`iterable`。`iterable`类型接受任何数组，任何实现 Traversable 的对象和生成器。

以下示例演示了将`iterable`用作函数参数的用法：

```php
function import(iterable $users) 
 {
   // ...
 }

function import(iterable $users = null) 
 {
   // ...
 }

function import(iterable $users = []) 
 {
   // ...
 }

```

尝试将值传递给前面的`import`函数，而不是 Traversable 的数组实例或生成器，会抛出`\TypeError`。然而，如果分配了默认值，无论是 null 还是空数组，函数都会起作用。

以下示例演示了将`iterable`用作函数返回值的用法：

```php
 function export(): iterable {
   return [
     'Johny',
     'Tom',
     'Matt'
   ];
 }

 function mix(): iterable {
   return [
     'Welcome',
      33,
      4200.00
   ];
 }

 function numbers(): iterable {
    for ($i = 0; $i <= 5; $i++) {
       yield $i;
    }
 }

```

需要注意的一点是，在 PHP 中，`iterable`被实现为一个保留的类名。这意味着任何名为`iterable`的用户类、接口或特性都会抛出错误。

# 可空类型

许多编程语言允许某种可选或可空类型，具体取决于术语。PHP 动态类型已经通过内置的 null 类型支持了这个概念。如果变量被赋予了常量值 null，它没有被赋予任何值，或者使用`unset()`构造函数取消了赋值，那么变量被认为是 null 类型。除了变量，null 类型也可以用于函数参数，通过将它们赋予 null 的默认值。

然而，这带来了一定的限制，因为我们无法声明一个可能为 null 的参数，而不同时将其标记为可选。

PHP 7.1 通过在类型前加上一个问号符号(`?`)来解决了这个限制，以指示类型可以为 null，除非明确赋予其他值。这也意味着类型可以同时为 null 和必需。这些可空类型现在几乎可以在任何允许类型声明的地方使用。

以下是带有强制参数值的可空类型的示例：

```php
function welcome(?string $name) {
   echo $name;
}

welcome(); // invalid
welcome(null); // valid

```

对`welcome`函数的第一次调用会抛出`\Error`，因为它的声明使参数成为了必需的。这说明可空类型不应该被误解为将`null`作为值传递。

以下是一个带有可选参数值的可空类型的示例，可选的意思是它已经被赋予了默认值`null`：

```php
function goodbye(?string $name = null)
 {
   if (is_null($name)) 
     {
       echo 'Goodbye!';
     } 
   else
     { 
       echo "Goodbye $name!";
     }
 }

goodbye(); // valid
goodbye(null); // valid
goodbye('John'); // valid

```

以下是使用可空返回类型声明函数的示例：

```php
function welcome($name): ?string 
  {
    return null; // valid
  }

function welcome($name): ?string 
  {
    return 'Welcome ' . $name; // valid
  }

function welcome($name): ?string 
 {
   return 33; // invalid
 }

```

可空类型适用于标量类型（布尔值、整数、浮点数、字符串）和复合类型（数组、对象、可调用）。

# Void 返回类型

在 PHP 7 中引入的函数参数类型和函数返回类型的强大功能中，`mix`函数中缺少了一件事。虽然函数返回类型允许指定所需的返回类型，但它们不允许指定缺少返回值。为了解决这一不一致性，PHP 7.1 版本引入了`void`返回类型功能。

为什么这很重要，我们可能会问自己？与前面提到的函数返回类型一样，这个特性对于文档和错误检查目的非常有用。由于 PHP 的性质，它在函数定义中不需要`return`语句，因此一开始不清楚函数是执行某些操作还是返回一个值。使用`void`返回类型使得函数的目的更清晰，即执行一个动作，而不是产生一个结果。

让我们看下面的例子：

```php
function A(): void {
   // valid
}

function B(): void {
   return; // valid
}

function C(): void {
   return null; // invalid
}

function D(): void {
   return 1; // invalid
}

```

`function A`和`function B`方法展示了`void`类型参数的有效用法。`function A`方法没有明确设置返回值，但这没关系，因为 PHP 隐式地总是返回`null`。`function B`方法简单地使用了`return`语句，后面没有任何类型，这也是有效的。`function C`方法有点奇怪，乍看起来可能是有效的，但实际上不是。为什么`function C`无效，而`function A`方法却有效，即使它们做的事情是一样的？尽管在 PHP 中，`return`和`return null`在技术上是等价的，但它们并不完全相同。返回类型的存在或缺失表示了函数的意图。指定返回值，即使是`null`，都意味着这个值是重要的。对于 void 返回类型，返回值是无关紧要的。因此，使用`void`返回类型表示一个不重要的返回值，在函数调用后不会在任何地方使用。

显式 void 和隐式 null 返回之间的区别可能有些模糊。这里的要点是，使用 void 返回类型传达了函数不应该返回任何类型的值。虽然它们对代码本身没有什么重大影响，它们的使用是完全可选的，但它们确实为语言带来了一定的丰富性。

# 总结

PHP 7 和 7.1 版本引入了许多变化。其中一些变化使语言超越了 PHP 曾经的样子。虽然仍然保持动态类型系统，但现在可以严格定义函数参数和返回类型。这改变了我们查看和处理函数的方式。在与函数相关的变化中，还有其他一些针对改进 PHP 5 十多年历史的变化。整个生态系统需要一些时间来适应。对于有 PHP 5 经验的开发人员来说，这些变化不仅仅是技术上的，它们需要改变思维方式，以成功应用现在可能的东西。

接下来，我们将研究 PHP 标准的当前状态，由谁定义它们，它们描述了什么，以及我们如何从中受益。
