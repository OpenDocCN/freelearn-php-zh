# 第三章：使用 PHP 函数式编程

在本章中，我们将涵盖以下主题：

+   开发函数

+   数据类型提示

+   使用返回值数据类型

+   使用迭代器

+   使用生成器编写自己的迭代器

# 介绍

在本章中，我们将考虑利用 PHP 的**函数式编程**能力的方法。函数式或**过程式**编程是在 PHP 版本 4 引入**面向对象编程**（**OOP**）之前编写 PHP 代码的传统方式。函数式编程是将程序逻辑封装到一系列离散的**函数**中，这些函数通常存储在单独的 PHP 文件中。然后可以在任何未来的脚本中包含此文件，从而允许随意调用定义的函数。

# 开发函数

最困难的部分是决定如何将编程逻辑分解为函数。另一方面，在 PHP 中开发函数的机制非常简单。只需使用`function`关键字，给它一个名称，然后跟着括号。

## 如何做...

1.  代码本身放在大括号中，如下所示：

```php
function someName ($parameter)
{ 
  $result = 'INIT';
  // one or more statements which do something
  // to affect $result
  $result .= ' and also ' . $parameter;
  return $result; 
}
```

1.  您可以定义一个或多个**参数**。要使其中一个参数变为可选，只需分配一个默认值。如果不确定要分配什么默认值，请使用`NULL`：

```php
function someOtherName ($requiredParam, $optionalParam = NULL)
  { 
    $result = 0;
    $result += $requiredParam;
    $result += $optionalParam ?? 0;
    return $result; 
  }
```

### 注意

您不能重新定义函数。唯一的例外是在不同的命名空间中定义重复的函数。这个定义会生成一个错误：

```php
function someTest()
{
  return 'TEST';
}
function someTest($a)
{
  return 'TEST:' . $a;
}
```

1.  如果不知道将向函数提供多少参数，或者想要允许无限数量的参数，请使用`...`后跟一个变量名。提供的所有参数将出现在变量中的数组中：

```php
function someInfinite(...$params)
{
  // any params passed go into an array $params
  return var_export($params, TRUE);
}
```

1.  函数可以调用自身。这被称为**递归**。以下函数执行递归目录扫描：

```php
function someDirScan($dir)
{
  // uses "static" to retain value of $list
  static $list = array();
  // get a list of files and directories for this path
  $list = glob($dir . DIRECTORY_SEPARATOR . '*');
  // loop through
  foreach ($list as $item) {
    if (is_dir($item)) {
      $list = array_merge($list, someDirScan($item));
    }
  }
  return $list;
}
```

### 注意

在函数内使用`static`关键字已经有 12 年以上的历史了。`static`的作用是在函数调用之间保留变量的值。

如果需要在 HTTP 请求之间保留变量的值，请确保已启动 PHP 会话并将值存储在`$_SESSION`中。

1.  在 PHP **命名空间**中定义函数时受到限制。这个特性可以用来为函数库之间提供额外的逻辑分离。为了*锚定*命名空间，您需要添加`use`关键字。以下示例放置在单独的命名空间中。请注意，即使函数名称相同，它们也不会发生冲突，因为它们彼此之间不可见。

1.  我们在命名空间`Alpha`中定义了`someFunction()`。我们将其保存到一个单独的 PHP 文件`chap_03_developing_functions_namespace_alpha.php`中：

```php
<?php
namespace Alpha;

function someFunction()
{
  echo __NAMESPACE__ . ':' . __FUNCTION__ . PHP_EOL;
}
```

1.  然后我们在命名空间`Beta`中定义了`someFunction()`。我们将其保存到一个单独的 PHP 文件`chap_03_developing_functions_namespace_beta.php`中：

```php
<?php
namespace Beta;

function someFunction()
{
  echo __NAMESPACE__ . ':' . __FUNCTION__ . PHP_EOL;
}
```

1.  然后我们可以通过在函数名前加上命名空间名称来调用`someFunction()`：

```php
include (__DIR__ . DIRECTORY_SEPARATOR 
         . 'chap_03_developing_functions_namespace_alpha.php');
include (__DIR__ . DIRECTORY_SEPARATOR 
         . 'chap_03_developing_functions_namespace_beta.php');
      echo Alpha\someFunction();
      echo Beta\someFunction();
```

### 提示

**最佳实践**

最佳实践是将函数库（以及类！）放入单独的文件中：一个命名空间一个文件，一个类或函数库一个文件。

可以在单个命名空间中定义许多类或函数库。将开发到单独的命名空间的唯一原因是如果要促进功能的逻辑分离。

## 它是如何工作的...

最佳实践是将所有逻辑相关的函数放入一个单独的 PHP 文件中。创建一个名为`chap_03_developing_functions_library.php`的文件，并将这些函数（前面描述的）放入其中。

+   `someName()`

+   `someOtherName()`

+   `someInfinite()`

+   `someDirScan()`

+   `someTypeHint()`

然后将此文件包含在使用这些函数的代码中。

```php
include (__DIR__ . DIRECTORY_SEPARATOR . 'chap_03_developing_functions_library.php');
```

要调用`someName()`函数，请使用名称并提供参数。

```php
echo someName('TEST');   // returns "INIT and also TEST"
```

你可以像这样调用`someOtherName()`函数使用一个或两个参数：

```php
echo someOtherName(1);    // returns  1
echo someOtherName(1, 1);   //  returns 2
```

`someInfinite()`函数接受无限（或可变）数量的参数。以下是调用这个函数的一些例子：

```php
echo someInfinite(1, 2, 3);
echo PHP_EOL;
echo someInfinite(22.22, 'A', ['a' => 1, 'b' => 2]);
```

输出如下：

![它是如何工作的...](img/B05314_03_01.jpg)

我们可以这样调用`someDirScan()`：

```php
echo someInfinite(1, 2, 3);
echo PHP_EOL;
echo someInfinite(22.22, 'A', ['a' => 1, 'b' => 2]);
```

输出如下：

![它是如何工作的...](img/B05314_03_02.jpg)

# 数据类型提示

在开发函数时，许多情况下你可能会在其他项目中重用相同的函数库。此外，如果你与团队合作，你的代码可能会被其他开发人员使用。为了控制你的代码的使用，使用**类型提示**可能是合适的。这涉及到指定函数对于特定参数期望的数据类型。

## 如何做...

1.  函数中的参数可以加上类型提示。以下类型提示在 PHP 5 和 PHP 7 中都可用：

+   数组

+   类

+   可调用的

1.  如果调用函数，并传递了错误的参数类型，将抛出`TypeError`。以下示例需要一个数组、一个`DateTime`的实例和一个匿名函数：

```php
function someTypeHint(Array $a, DateTime $t, Callable $c)
{
  $message = '';
  $message .= 'Array Count: ' . count($a) . PHP_EOL;
  $message .= 'Date: ' . $t->format('Y-m-d') . PHP_EOL;
  $message .= 'Callable Return: ' . $c() . PHP_EOL;
  return $message;
}
```

### 提示

你不必为每个参数提供类型提示。只有在提供不同的数据类型会对函数处理产生负面影响时才使用这种技术。例如，如果你的函数使用`foreach()`循环，如果你没有提供一个数组，或者实现了`Traversable`的东西，就会产生一个错误。

1.  在 PHP 7 中，假设适当的`declare()`指令已经被声明，**标量**（即整数、浮点数、布尔值和字符串）类型提示是允许的。另一个函数演示了如何实现这一点。在包含你希望使用标量类型提示的函数的代码库文件的顶部，在开头的 PHP 标记之后添加这个`declare()`指令：

```php
declare(strict_types=1);
```

1.  现在你可以定义一个包含标量类型提示的函数：

```php
function someScalarHint(bool $b, int $i, float $f, string $s)
{
  return sprintf("\n%20s : %5s\n%20s : %5d\n%20s " . 
                 ": %5.2f\n%20s : %20s\n\n",
                 'Boolean', ($b ? 'TRUE' : 'FALSE'),
                 'Integer', $i,
                 'Float',   $f,
                 'String',  $s);
}
```

1.  在 PHP 7 中，假设已经声明了严格的类型提示，布尔类型提示与其他三种标量类型（即整数、浮点数和字符串）有些不同。你可以提供任何标量作为参数，不会抛出`TypeError`！然而，一旦传递到函数中，传入的值将自动转换为布尔数据类型。如果传递的数据类型不是标量（即数组或对象），将抛出`TypeError`。这是一个定义`boolean`数据类型的函数的例子。请注意，返回值将自动转换为`boolean`：

```php
function someBoolHint(bool $b)
{
  return $b;
}
```

## 它是如何工作的...

首先，你可以将`someTypeHint()`、`someScalarHint()`和`someBoolHint()`这三个函数放在一个单独的文件中以供包含。在这个例子中，我们将文件命名为`chap_03_developing_functions_type_hints_library.php`。不要忘记在顶部添加`declare(strict_types=1)`！

在我们的调用代码中，你需要包含这个文件：

```php
include (__DIR__ . DIRECTORY_SEPARATOR . 'chap_03_developing_functions_type_hints_library.php');
```

要测试`someTypeHint()`，调用函数两次，一次使用正确的数据类型，第二次使用不正确的类型。这将抛出一个`TypeError`，因此你需要将函数调用包装在`try { ... } catch () { ...}`块中：

```php
try {
    $callable = function () { return 'Callback Return'; };
    echo someTypeHint([1,2,3], new DateTime(), $callable);
    echo someTypeHint('A', 'B', 'C');
} catch (TypeError $e) {
    echo $e->getMessage();
    echo PHP_EOL;
}
```

从这个子部分末尾显示的输出中可以看出，当传递正确的数据类型时没有问题。当传递不正确的类型时，将抛出`TypeError`。

### 注意

在 PHP 7 中，某些错误已经转换为`Error`类，这与`Exception`的处理方式有些相似。这意味着你可以捕获`Error`。`TypeError`是`Error`的一个特定子类，当向函数传递不正确的数据类型时抛出。

所有 PHP 7 的`Error`类都实现了`Throwable`接口，`Exception`类也是如此。如果你不确定是否需要捕获`Error`还是`Exception`，你可以添加一个捕获`Throwable`的块。

接下来，您可以测试`someScalarHint()`，用正确和不正确的值调用它，将调用包装在`try { ... } catch () { ...}`块中：

```php
try {
    echo someScalarHint(TRUE, 11, 22.22, 'This is a string');
    echo someScalarHint('A', 'B', 'C', 'D');
} catch (TypeError $e) {
    echo $e->getMessage();
}
```

如预期的那样，对该函数的第一次调用有效，而第二次调用会抛出`TypeError`。

当对布尔值进行类型提示时，传递的任何标量值都*不会*导致抛出`TypeError`！相反，该值将被解释为其布尔等价值。如果随后返回此值，则数据类型将更改为布尔值。

要测试这一点，调用之前定义的`someBoolHint()`函数，并将任何标量值作为参数传入。`var_dump()`方法显示数据类型始终是布尔值：

```php
try {
    // positive results
    $b = someBooleanHint(TRUE);
    $i = someBooleanHint(11);
    $f = someBooleanHint(22.22);
    $s = someBooleanHint('X');
    var_dump($b, $i, $f, $s);
    // negative results
    $b = someBooleanHint(FALSE);
    $i = someBooleanHint(0);
    $f = someBooleanHint(0.0);
    $s = someBooleanHint('');
    var_dump($b, $i, $f, $s);
} catch (TypeError $e) {
    echo $e->getMessage();
}
```

如果您现在尝试相同的函数调用，但传入非标量数据类型，则会抛出`TypeError`：

```php
try {
    $a = someBoolHint([1,2,3]);
    var_dump($a);
} catch (TypeError $e) {
    echo $e->getMessage();
}
try {
    $o = someBoolHint(new stdClass());
    var_dump($o);
} catch (TypeError $e) {
    echo $e->getMessage();
}
```

这是整体输出：

![它是如何工作的...](img/B05314_03_03.jpg)

## 另请参阅

PHP 7.1 引入了一个新的类型提示`iterable`，它允许数组、`Iterators`或`Generators`作为参数。有关更多信息，请参阅：

+   [`wiki.php.net/rfc/iterable`](https://wiki.php.net/rfc/iterable)

有关标量类型提示实现背后的原理的背景讨论，请参阅本文：

+   [`wiki.php.net/rfc/scalar_type_hints_v5`](https://wiki.php.net/rfc/scalar_type_hints_v5)

# 使用返回值数据类型

PHP 7 允许您为函数的返回值指定数据类型。然而，与标量类型提示不同，您不需要添加任何特殊声明。

## 如何做...

1.  这个例子向您展示了如何为函数返回值分配数据类型。要分配返回数据类型，首先像通常一样定义函数。在右括号后面，加一个空格，然后是数据类型和一个冒号：

```php
function returnsString(DateTime $date, $format) : string
{
  return $date->format($format);
}
```

### 注意

PHP 7.1 引入了一种称为**可空类型**的返回数据类型的变体。您需要做的就是将`string`更改为`?string`。这允许函数返回`string`或`NULL`。

1.  函数返回的任何东西，无论在函数内部的数据类型如何，都将被转换为声明的数据类型作为返回值。请注意，在这个例子中，将`$a`、`$b`和`$c`的值相加以产生一个单一的总和，然后返回。通常您会期望返回值是一个数字数据类型。然而，在这种情况下，返回数据类型被声明为`string`，这将覆盖 PHP 的类型转换过程：

```php
function convertsToString($a, $b, $c) : string

  return $a + $b + $c;
}
```

1.  您还可以将类分配为返回数据类型。在这个例子中，我们将返回类型分配为 PHP `DateTime`扩展的一部分的`DateTime`：

```php
function makesDateTime($year, $month, $day) : DateTime
{
  $date = new DateTime();
  $date->setDate($year, $month, $day);
  return $date;
}
```

### 注意

`makesDateTime()`函数将是标量类型提示的一个潜在候选。如果`$year`、`$month`或`$day`不是整数，在调用`setDate()`时会生成一个`Warning`。如果您使用标量类型提示，并且传递了错误的数据类型，将抛出`TypeError`。虽然生成警告或抛出`TypeError`并不重要，但至少`TypeError`会导致错误使用您的代码的开发人员警觉起来！

1.  如果一个函数有一个返回数据类型，并且您在函数代码中返回了错误的数据类型，那么在运行时会抛出`TypeError`。这个函数分配了一个`DateTime`的返回类型，但返回了一个字符串。会抛出`TypeError`，但直到运行时，当 PHP 引擎检测到不一致时才会抛出：

```php
function wrongDateTime($year, $month, $day) : DateTime
{
  return date($year . '-' . $month . '-' . $day);
}
```

### 注意

如果返回数据类型类不是内置的 PHP 类之一（即 SPL 的一部分），则需要确保已自动加载或包含该类。

## 它是如何工作的...

首先，将前面提到的函数放入名为`chap_03_developing_functions_return_types_library.php`的库文件中。这个文件需要包含在调用这些函数的`chap_03_developing_functions_return_types.php`脚本中：

```php
include (__DIR__ . '/chap_03_developing_functions_return_types_library.php');
```

现在，您可以调用`returnsString()`，提供一个`DateTime`实例和一个格式字符串：

```php
$date   = new DateTime();
$format = 'l, d M Y';
$now    = returnsString($date, $format);
echo $now . PHP_EOL;
var_dump($now);
```

如预期的那样，输出是一个字符串：

![它是如何工作的...](img/B05314_03_07.jpg)

现在您可以调用`convertsToString()`并提供三个整数作为参数。注意返回类型是字符串：

```php
echo "\nconvertsToString()\n";
var_dump(convertsToString(2, 3, 4));
```

![它是如何工作的...](img/B05314_03_08.jpg)

为了证明这一点，您可以将一个类分配为返回值，使用三个整数参数调用`makesDateTime()`：

```php
echo "\nmakesDateTime()\n";
$d = makesDateTime(2015, 11, 21);
var_dump($d);
```

![它是如何工作的...](img/B05314_03_09.jpg)

最后，使用三个整数参数调用`wrongDateTime()`：

```php
try {
    $e = wrongDateTime(2015, 11, 21);
    var_dump($e);
} catch (TypeError $e) {
    echo $e->getMessage();
}
```

注意，在运行时抛出了`TypeError`：

![它是如何工作的...](img/B05314_03_10.jpg)

## 还有更多...

PHP 7.1 添加了一个新的返回值类型，`void`。当您不希望从函数中返回任何值时使用。有关更多信息，请参阅[`wiki.php.net/rfc/void_return_type`](https://wiki.php.net/rfc/void_return_type)。

## 另请参阅

有关返回类型声明的更多信息，请参阅以下文章：

+   [`php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict`](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict)

+   [`wiki.php.net/rfc/return_types`](https://wiki.php.net/rfc/return_types)

有关可空类型的信息，请参阅本文：

+   [`wiki.php.net/rfc/nullable_types`](https://wiki.php.net/rfc/nullable_types)

# 使用迭代器

**迭代器**是一种特殊类型的类，允许您**遍历**一个*容器*或列表。关键词在于*遍历*。这意味着迭代器提供了浏览列表的方法，但它本身不执行遍历。

SPL 提供了丰富的通用和专门设计用于不同上下文的迭代器。例如，`ArrayIterator`被设计用于允许面向对象遍历数组。`DirectoryIterator`被设计用于文件系统扫描。

某些 SPL 迭代器被设计用于与其他迭代器一起工作，并增加价值。示例包括`FilterIterator`和`LimitIterator`。前者使您能够从父迭代器中删除不需要的值。后者提供了分页功能，您可以指定要遍历多少项以及确定从何处开始的偏移量。

最后，还有一系列*递归*迭代器，允许您重复调用父迭代器。一个例子是`RecursiveDirectoryIterator`，它从起始点扫描整个目录树，直到最后一个可能的子目录。

## 如何做...

1.  我们首先检查`ArrayIterator`类。它非常容易使用。您只需要将数组作为参数提供给构造函数。之后，您可以使用所有基于 SPL 的迭代器标准的方法，例如`current()`，`next()`等。

```php
$iterator = new ArrayIterator($array);
```

### 注意

使用`ArrayIterator`将标准 PHP 数组转换为迭代器。在某种意义上，这提供了过程式编程和面向对象编程之间的桥梁。

1.  作为迭代器的实际用途的一个例子，请查看这个例子。它接受一个迭代器并生成一系列 HTML`<ul>`和`<li>`标签：

```php
function htmlList($iterator)
{
  $output = '<ul>';
  while ($value = $iterator->current()) {
    $output .= '<li>' . $value . '</li>';
    $iterator->next();
  }
  $output .= '</ul>';
  return $output;
}
```

1.  或者，您可以简单地将`ArrayIterator`实例包装到一个简单的`foreach()`循环中：

```php
function htmlList($iterator)
{
  $output = '<ul>';
  foreach($iterator as $value) {
    $output .= '<li>' . $value . '</li>';
  }
  $output .= '</ul>';
  return $output;
}
```

1.  `CallbackFilterIterator`是一种很好的方式，可以为您可能正在使用的任何现有迭代器增加价值。它允许您包装任何现有迭代器并筛选输出。在这个例子中，我们将定义`fetchCountryName()`，它遍历生成国家名称列表的数据库查询。首先，我们从使用第一章中定义的`Application\Database\Connection`类的查询中定义一个`ArrayIterator`实例，*建立基础*：

```php
function fetchCountryName($sql, $connection)
{
  $iterator = new ArrayIterator();
  $stmt = $connection->pdo->query($sql);
  while($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    $iterator->append($row['name']);
  }
  return $iterator;
}
```

1.  接下来，我们定义一个过滤方法`nameFilterIterator()`，它接受部分国家名称作为参数，以及`ArrayIterator`实例：

```php
function nameFilterIterator($innerIterator, $name)
{
  if (!$name) return $innerIterator;
  $name = trim($name);
  $iterator = new CallbackFilterIterator($innerIterator, 
    function($current, $key, $iterator) use ($name) {
      $pattern = '/' . $name . '/i';
      return (bool) preg_match($pattern, $current);
    }
  );
  return $iterator;
}
```

1.  `LimitIterator` 为您的应用程序添加了基本的分页功能。要使用此迭代器，您只需要提供父迭代器、偏移量和限制。`LimitIterator` 将只产生从偏移量开始的整个数据集的子集。以步骤 2 中提到的相同示例为例，我们将对来自数据库查询的结果进行分页。我们可以通过简单地将`fetchCountryName()`方法生成的迭代器包装在`LimitIterator`实例中来实现这一点：

```php
$pagination = new LimitIterator(fetchCountryName(
$sql, $connection), $offset, $limit);
```

### 注意

在使用`LimitIterator`时要小心。为了实现限制，它需要将*整个*数据集保存在内存中。因此，在迭代大型数据集时，这不是一个好工具。

1.  迭代器可以*堆叠*。在这个简单的例子中，`ArrayIterator`由`FilterIterator`处理，然后由`LimitIterator`限制。首先，我们设置一个`ArrayIterator`实例：

```php
$i = new ArrayIterator($a);
```

1.  接下来，我们将`ArrayIterator`插入`FilterIterator`实例中。请注意，我们正在使用新的 PHP 7 匿名类特性。在这种情况下，匿名类扩展了`FilterIterator`并覆盖了`accept()`方法，只允许具有偶数 ASCII 代码的字母：

```php
$f = new class ($i) extends FilterIterator { 
  public function accept()
  {
    $current = $this->current();
    return !(ord($current) & 1);
  }
};
```

1.  最后，我们将`FilterIterator`实例作为参数提供给`LimitIterator`，并提供偏移量（在本例中为`2`）和限制（在本例中为`6`）：

```php
$l = new LimitIterator($f, 2, 6);
```

1.  然后，我们可以定义一个简单的函数来显示输出，并依次调用每个迭代器，以查看由`range('A', 'Z')`生成的简单数组的结果：

```php
function showElements($iterator)
{
  foreach($iterator as $item)  echo $item . ' ';
  echo PHP_EOL;
}

$a = range('A', 'Z');
$i = new ArrayIterator($a);
showElements($i);
```

1.  这是一个变体，通过在`ArrayIterator`上堆叠`FilterIterator`来产生每隔一个字母：

```php
$f = new class ($i) extends FilterIterator {
public function accept()
  {
    $current = $this->current();
    return !(ord($current) & 1);
  }
};
showElements($f);
```

1.  这里还有另一个变体，它只产生`F H J L N P`，这演示了一个消耗`FilterIterator`的`LimitIterator`，而`FilterIterator`又消耗`ArrayIterator`。这三个示例的输出如下：

```php
$l = new LimitIterator($f, 2, 6);
showElements($l);
```

![如何做...](img/B05314_03_12.jpg)

1.  回到我们的例子，它产生了一个国家名称列表，假设我们希望迭代一个由国家名称和 ISO 代码组成的多维数组，而不仅仅是国家名称。到目前为止提到的简单迭代器是不够的。相反，我们将使用所谓的**递归**迭代器。

1.  首先，我们需要定义一个方法，该方法使用先前提到的数据库连接类从数据库中提取所有列。与以前一样，我们返回一个由查询数据填充的`ArrayIterator`实例：

```php
function fetchAllAssoc($sql, $connection)
{
  $iterator = new ArrayIterator();
  $stmt = $connection->pdo->query($sql);
  while($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    $iterator->append($row);
  }
  return $iterator;
}
```

1.  乍一看，人们可能会简单地将标准的`ArrayIterator`实例包装在`RecursiveArrayIterator`中。不幸的是，这种方法只执行**浅**迭代，并且不能给我们想要的：对从数据库查询返回的多维数组的所有元素进行迭代：

```php
$iterator = fetchAllAssoc($sql, $connection);
$shallow  = new RecursiveArrayIterator($iterator);
```

1.  虽然这返回一个迭代，其中每个项表示数据库查询的一行，但在这种情况下，我们希望提供一个迭代，该迭代将遍历查询返回的所有行的所有列。为了实现这一点，我们需要通过`RecursiveIteratorIterator`来展开大规模的操作。

1.  蒙提·派森的粉丝将沉浸在这个类名的丰富讽刺之中，因为它让人回忆起*多余部门*。恰当地，这个类让我们的老朋友`RecursiveArrayIterator`类加班工作，并对数组的所有级别进行**深度**迭代：

```php
$deep     = new RecursiveIteratorIterator($shallow);
```

## 工作原理...

作为一个实际的例子，您可以开发一个测试脚本，使用迭代器实现过滤和分页。对于这个示例，您可以调用`chap_03_developing_functions_filtered_and_paginated.php`测试代码文件。

首先，按照最佳实践，将上述描述的函数放入名为`chap_03_developing_functions_iterators_library.php`的包含文件中。在测试脚本中，确保包含此文件。

数据源是一个名为`iso_country_codes`的表，其中包含 ISO2、ISO3 和国家名称。数据库连接可以在一个`config/db.config.php`文件中。您还可以包括在前一章中讨论的`Application\Database\Connection`类：

```php
define('DB_CONFIG_FILE', '/../config/db.config.php');
define('ITEMS_PER_PAGE', [5, 10, 15, 20]);
include (__DIR__ . '/chap_03_developing_functions_iterators_library.php');
include (__DIR__ . '/../Application/Database/Connection.php');
```

### 注意

在 PHP 7 中，您可以将常量定义为数组。在本例中，`ITEMS_PER_PAGE`被定义为一个数组，并用于生成 HTML`SELECT`元素。

接下来，您可以处理国家名称和每页项目数的输入参数。当前页码将从`0`开始，并且可以递增（下一页）或递减（上一页）：

```php
$name = strip_tags($_GET['name'] ?? '');
$limit  = (int) ($_GET['limit'] ?? 10);
$page   = (int) ($_GET['page']  ?? 0);
$offset = $page * $limit;
$prev   = ($page > 0) ? $page - 1 : 0;
$next   = $page + 1;
```

现在，您已经准备好启动数据库连接并运行一个简单的`SELECT`查询。这应该放在`try {} catch {}`块中。然后，您可以将要堆叠的迭代器放在`try {}`块内：

```php
try {
    $connection = new Application\Database\Connection(
      include __DIR__ . DB_CONFIG_FILE);
    $sql    = 'SELECT * FROM iso_country_codes';
    $arrayIterator    = fetchCountryName($sql, $connection);
    $filteredIterator = nameFilterIterator($arrayIterator, $name);
    $limitIterator    = pagination(
    $filteredIterator, $offset, $limit);
} catch (Throwable $e) {
    echo $e->getMessage();
}
```

现在我们准备好进行 HTML 编写。在这个简单的例子中，我们提供一个表单，让用户选择每页的项目数和国家名称：

```php
<form>
  Country Name:
  <input type="text" name="name" 
         value="<?= htmlspecialchars($name) ?>">
  Items Per Page: 
  <select name="limit">
    <?php foreach (ITEMS_PER_PAGE as $item) : ?>
      <option<?= ($item == $limit) ? ' selected' : '' ?>>
      <?= $item ?></option>
    <?php endforeach; ?>
  </select>
  <input type="submit" />
</form>
  <a href="?name=<?= $name ?>&limit=<?= $limit ?>
    &page=<?= $prev ?>">
  << PREV</a> 
  <a href="?name=<?= $name ?>&limit=<?= $limit ?>
    &page=<?= $next ?>">
  NEXT >></a>
<?= htmlList($limitIterator); ?>
```

输出将看起来像这样：

![工作原理...](img/B05314_03_13.jpg)

最后，为了测试国家数据库查找的递归迭代，您需要包括迭代器的库文件，以及`Application\Database\Connection`类：

```php
define('DB_CONFIG_FILE', '/../config/db.config.php');
include (__DIR__ . '/chap_03_developing_functions_iterators_library.php');
include (__DIR__ . '/../Application/Database/Connection.php');
```

与以前一样，您应该将数据库查询放在`try {} catch {}`块中。然后，您可以将用于测试递归迭代的代码放在`try {}`块内：

```php
try {
    $connection = new Application\Database\Connection(
    include __DIR__ . DB_CONFIG_FILE);
    $sql    = 'SELECT * FROM iso_country_codes';
    $iterator = fetchAllAssoc($sql, $connection);
    $shallow  = new RecursiveArrayIterator($iterator);
    foreach ($shallow as $item) var_dump($item);
    $deep     = new RecursiveIteratorIterator($shallow);
    foreach ($deep as $item) var_dump($item);     
} catch (Throwable $e) {
    echo $e->getMessage();
}
```

以下是您可以期望从`RecursiveArrayIterator`输出的内容：

![工作原理...](img/B05314_03_14.jpg)

使用`RecursiveIteratorIterator`后的输出如下：

![工作原理...](img/B05314_03_15.jpg)

# 使用生成器编写自己的迭代器

在前面的一系列示例中，我们演示了 PHP 7 SPL 中提供的迭代器的使用。但是，如果这个集合不能满足给定项目的需求，该怎么办？一个解决方案是开发一个函数，该函数不是构建一个然后返回的数组，而是使用`yield`关键字通过迭代逐步返回值。这样的函数被称为**生成器**。实际上，在后台，PHP 引擎将自动将您的函数转换为一个称为`Generator`的特殊内置类。

这种方法有几个优点。主要好处在于当您有一个大容器要遍历时（即解析一个大文件），可以看到。传统的方法是构建一个数组，然后返回该数组。这样做的问题是您实际上需要的内存量翻倍！此外，性能受到影响，因为只有在最终数组被返回后才能实现结果。

## 如何做...

1.  在这个例子中，我们在基于迭代器的函数库上构建了一个我们自己设计的生成器。在这种情况下，我们将复制上面关于迭代器的部分中描述的功能，其中我们堆叠了`ArrayIterator`，`FilterIterator`和`LimitIterator`。

1.  因为我们需要访问源数组、所需的过滤器、页码和每页项目数，所以我们将适当的参数包含到一个单独的`filteredResultsGenerator()`函数中。然后，我们根据页码和限制（即每页项目数）计算偏移量。接下来，我们循环遍历数组，应用过滤器，并在偏移量尚未达到时继续循环，或者在达到限制时中断：

```php
function filteredResultsGenerator(array $array, $filter, $limit = 10, $page = 0)
  {
    $max    = count($array);
    $offset = $page * $limit;
    foreach ($array as $key => $value) {
      if (!stripos($value, $filter) !== FALSE) continue;
      if (--$offset >= 0) continue;
      if (--$limit <= 0) break; 
      yield $value;
    }
  }
```

1.  您会注意到这个函数和其他函数之间的主要区别是`yield`关键字。这个关键字的作用是向 PHP 引擎发出信号，产生一个`Generator`实例并封装代码。

## 工作原理...

为了演示`filteredResultsGenerator()`函数的使用，我们将让您实现一个 Web 应用程序，该应用程序扫描一个网页并生成一个经过过滤和分页的 URL 列表，这些 URL 列表是从`HREF`属性中获取的。

首先，您需要将`filteredResultsGenerator()`函数的代码添加到先前配方中使用的库文件中，然后将先前描述的函数放入一个包含文件`chap_03_developing_functions_iterators_library.php`中。

接下来，定义一个测试脚本`chap_03_developing_functions_using_generator.php`，其中包括函数库以及定义在第一章中描述的`Application\Web\Hoover`文件，*构建基础*：

```php
include (__DIR__ . DIRECTORY_SEPARATOR . 'chap_03_developing_functions_iterators_library.php');
include (__DIR__ . '/../Application/Web/Hoover.php');
```

然后，您需要从用户那里收集关于要扫描的 URL，要用作过滤器的字符串，每页多少项以及当前页码的输入。

### 注意

**null coalesce**运算符(`??`)非常适合从 Web 获取输入。如果未定义，它不会生成任何通知。如果未从用户输入接收参数，则可以提供默认值。

```php
$url    = trim(strip_tags($_GET['url'] ?? ''));
$filter = trim(strip_tags($_GET['filter'] ?? ''));
$limit  = (int) ($_GET['limit'] ?? 10);
$page   = (int) ($_GET['page']  ?? 0);
```

### 提示

**最佳实践**

Web 安全性应始终是优先考虑的。在此示例中，您可以使用`strip_tags()`，并将数据类型强制转换为整数`(int)`来消毒用户输入。

然后，您可以定义用于分页列表中上一页和下一页链接的变量。请注意，您还可以应用*健全性检查*，以确保下一页不会超出结果集的末尾。为简洁起见，本示例中未应用此类健全性检查：

```php
$next   = $page + 1;
$prev   = $page - 1;
$base   = '?url=' . htmlspecialchars($url) 
        . '&filter=' . htmlspecialchars($filter) 
        . '&limit=' . $limit 
        . '&page=';
```

然后，我们需要创建一个`Application\Web\Hoover`实例，并从目标 URL 中获取`HREF`属性：

```php
$vac    = new Application\Web\Hoover();
$list   = $vac->getAttribute($url, 'href');
```

最后，我们定义了 HTML 输出，通过先前描述的`htmlList()`函数渲染输入表单并运行我们的生成器：

```php
<form>
<table>
<tr>
<th>URL</th>
<td>
<input type="text" name="url" 
  value="<?= htmlspecialchars($url) ?>"/>
</td>
</tr>
<tr>
<th>Filter</th>
<td>
<input type="text" name="filter" 
  value="<?= htmlspecialchars($filter) ?>"/></td>
</tr>
<tr>
<th>Limit</th>
<td><input type="text" name="limit" value="<?= $limit ?>"/></td>
</tr>
<tr>
<th>&nbsp;</th><td><input type="submit" /></td>
</tr>
<tr>
<td>&nbsp;</td>
<td>
<a href="<?= $base . $prev ?>"><-- PREV | 
<a href="<?= $base . $next ?>">NEXT --></td>
</tr>
</table>
</form>
<hr>
<?= htmlList(filteredResultsGenerator(
$list, $filter, $limit, $page)); ?>
```

这是一个输出的例子：

![工作原理...](img/B05314_03_16.jpg)
