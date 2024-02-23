# 第二章。PHP 7 中的新功能

PHP 7 引入了一些新功能，可以帮助程序员编写高性能和有效的代码。此外，一些老式的功能已经完全移除，如果使用 PHP 7 将抛出错误。现在大多数致命错误都是异常，因此 PHP 不会再显示丑陋的致命错误消息；相反，它将通过可用的详细信息进行异常处理。

在本章中，我们将涵盖以下主题：

+   类型提示

+   命名空间和组使用声明

+   匿名类

+   旧式构造函数弃用

+   太空船运算符

+   空合并运算符

+   统一变量语法

+   其他更改

# 面向对象编程特性

PHP 7 引入了一些新的面向对象编程功能，使开发人员能够编写干净而有效的代码。在本节中，我们将讨论这些功能。

## 类型提示

在 PHP 7 之前，不需要声明传递给函数或类方法的参数的数据类型。此外，也不需要提及返回数据类型。任何数据类型都可以传递给函数或方法，并从函数或方法返回。这是 PHP 中的一个巨大问题，不清楚应该传递或接收哪些数据类型。为了解决这个问题，PHP 7 引入了类型提示。目前，引入了两种类型提示：标量和返回类型提示。这些将在以下部分讨论。

类型提示是面向对象编程和过程式 PHP 中的一个特性，因为它可以用于过程式函数和对象方法。

### 标量类型提示

PHP 7 使得可以为整数、浮点数、字符串和布尔值的函数和方法使用标量类型提示。让我们看下面的例子：

```php
class Person
{
  public function age(int $age)
  {
    return $age;
    }

  public function name(string $name)
  {
    return $name;
    }

  public function isAlive(bool $alive)
  {
    return $alive;
    }

}

$person = new Person();
echo $person->name('Altaf Hussain');
echo $person->age(30);
echo $person->isAlive(TRUE);
```

在上面的代码中，我们创建了一个`Person`类。我们有三种方法，每种方法接收不同的参数，其数据类型在上面的代码中进行了定义。如果运行上面的代码，它将正常工作，因为我们将为每种方法传递所需的数据类型。

年龄可以是浮点数，例如`30.5`岁；因此，如果我们将浮点数传递给`age`方法，它仍然可以工作，如下所示：

```php
echo $person->age(30.5);
```

为什么？这是因为默认情况下，*标量类型提示是非限制性的*。这意味着我们可以将浮点数传递给期望整数的方法。

为了使其更加严格，可以将以下单行代码放在文件的顶部：

```php
declare(strict_types = 1);
```

现在，如果我们将浮点数传递给`age`函数，我们将得到一个**未捕获的类型错误**，这是一个致命错误，告诉我们`Person::age`必须是给定浮点数的整数类型。如果我们将字符串传递给不是字符串类型的方法，将生成类似的错误。考虑以下例子：

```php
echo $person->isAlive('true');
```

由于传递了字符串，上面的代码将生成致命错误。

### 返回类型提示

PHP 7 的另一个重要特性是能够为函数或方法定义返回数据类型。它的行为与标量类型提示的行为相同。让我们稍微修改我们的`Person`类以理解返回类型提示，如下所示：

```php
class Person
{
  public function age(float $age) : string
  {
    return 'Age is '.$age;
  }

  public function name(string $name) : string
  {
    return $name;
    }

  public function isAlive(bool $alive) : string
  {
    return ($alive) ? 'Yes' : 'No';
  }

}
```

类中的更改已经突出显示。使用`:数据类型`语法定义了返回类型。返回类型是否与标量类型相同并不重要。只要它们与各自的数据类型匹配即可。

现在，让我们尝试一个带有对象返回类型的例子。考虑之前的`Person`类，并向其添加一个`getAddress`方法。此外，我们将在同一个文件中添加一个新的类`Address`，如下所示：

```php
**class Address** 
**{**
 **public function getAddress()**
 **{**
 **return ['street' => 'Street 1', 'country' => 'Pak'];**
 **}**
**}**

class Person
{
  public function age(float $age) **: string**
  {
    return 'Age is '.$age;
  }

  public function name(string $name) **: string**
  {
    return $name;
  }

  public function isAlive(bool $alive) : string
  {
    return ($alive) ? 'Yes' : 'No';
  }

 **public function getAddress() : Address**
 **{**
 **return new Address();**
 **}**
}
```

添加到`Person`类和新的`Address`类的附加代码已经突出显示。现在，如果我们调用`Person`类的`getAddress`方法，它将完美地工作，不会抛出错误。然而，假设我们改变返回语句，如下所示：

```php
public function getAddress() : Address
{
  return ['street' => 'Street 1', 'country' => 'Pak'];
}
```

在这种情况下，上面的方法将抛出类似于以下内容的*未捕获*异常：

```php
Fatal error: Uncaught TypeError: Return value of Person::getAddress() must be an instance of Address, array returned
```

这是因为我们返回的是一个数组，而不是一个`Address`对象。现在，问题是：为什么使用类型提示？使用类型提示的重要优势是它将始终避免意外地传递或返回错误和意外的数据到方法或函数。

如前面的例子所示，这使得代码清晰，通过查看方法的声明，可以准确知道应该传递哪些数据类型到每个方法，以及通过查看每个方法的代码或注释，返回什么类型的数据。

## 命名空间和组使用声明

在一个非常庞大的代码库中，类被划分到命名空间中，这使得它们易于管理和使用。但是，如果一个命名空间中有太多的类，而我们需要使用其中的 10 个类，那么我们必须为所有这些类输入完整的使用语句。

### 注意

在 PHP 中，不需要根据其命名空间将类分成子文件夹，这与其他编程语言不同。命名空间只是提供类的逻辑分离。但是，我们不限于根据我们的命名空间将我们的类放在子文件夹中。

例如，我们有一个`Publishers/Packt`命名空间和类`Book`、`Ebook`、`Video`和`Presentation`。此外，我们有一个`functions.php`文件，其中包含我们的常规函数，并且在相同的`Publishers/Packt`命名空间中。另一个文件`constants.php`包含应用程序所需的常量值，并且在相同的命名空间中。每个类和`functions.php`和`constants.php`文件的代码如下：

```php
//book.php
namespace Publishers\Packt;

class Book 
{
  public function get() : string
  {
    return get_class();
  }
}
```

现在，`Ebook`类的代码如下：

```php
//ebook.php
namespace Publishers\Packt;

class Ebook 
{
  public function get() : string
  {
    return get_class();
  }
}
```

`Video`类的代码如下：

```php
//presentation.php
namespace Publishers\Packt;

class Video 
{
  public function get() : string
  {
    return get_class();
  }
}
```

同样，`presentation`类的代码如下：

```php
//presentation.php
namespace Publishers\Packt;

class Presentation 
{
  public function get() : string
  {
    return get_class();
  }
}
```

所有四个类都有相同的方法，这些方法使用 PHP 内置的`get_class()`函数返回类的名称。

现在，将以下两个函数添加到`functions.php`文件中：

```php
//functions.php

namespace Publishers\Packt;

function getBook() : string
{
  return 'PHP 7';
}
function saveBook(string $book) : string
{
  return $book.' is saved';
}
```

现在，让我们将以下代码添加到`constants.php`文件中：

```php
//constants.php

namespace Publishers/Packt;

const COUNT = 10;
const KEY = '123DGHtiop09847';
const URL = 'https://www.Packtpub.com/';
```

`functions.php`和`constants.php`中的代码是不言自明的。请注意，每个文件顶部都有一行`namespace Publishers/Packt`，这使得这些类、函数和常量属于这个命名空间。

现在，有三种方法可以使用类、函数和常量。让我们逐一考虑每一种。

看一下下面的代码：

```php
//Instantiate objects for each class in namespace

$book = new Publishers\Packt\Book();
$ebook = new Publishers\Packt\Ebook();
$video = new Publishers\Packt\Video();
$presentation = new Publishers\Packt\Presentation();

//Use functions in namespace

echo Publishers/Packt/getBook();
echo Publishers/Packt/saveBook('PHP 7 High Performance');

//Use constants

echo Publishers\Packt\COUNT;
echo Publishers\Packt\KEY;
```

在前面的代码中，我们直接使用命名空间名称创建对象或使用函数和常量。代码看起来不错，但是有点混乱。命名空间到处都是，如果我们有很多命名空间，它看起来会很丑陋，可读性也会受到影响。

### 注意

我们在之前的代码中没有包含类文件。可以使用`include`语句或 PHP 的`__autoload`函数来包含所有文件。

现在，让我们重新编写前面的代码，使其更易读，如下所示：

```php
use Publishers\Packt\Book;
use Publishers\Packt\Ebook;
use Publishers\Packt\Video;
use Publishers\Packt\Presentation;
use function Publishers\Packt\getBook;
use function Publishers\Packt\saveBook;
use const Publishers\Packt\COUNT;
use const Publishers\Packt\KEY;

$book = new Book();
$ebook = new Ebook(();
$video = new Video();
$pres = new Presentation();

echo getBook();
echo saveBook('PHP 7 High Performance');

echo COUNT; 
echo KEY;
```

在前面的代码中，我们在顶部使用了 PHP 语句来指定命名空间中特定的类、函数和常量。但是，我们仍然为每个类、函数和/或常量编写了重复的代码行。这可能导致我们在文件顶部有大量的使用语句，并且整体冗长度不好。

为了解决这个问题，PHP 7 引入了组使用声明。有三种类型的组使用声明：

+   非混合使用声明

+   混合使用声明

+   复合使用声明

### 非混合组使用声明

假设我们在一个命名空间中有不同类型的特性，如类、函数和联系人。在非混合组使用声明中，我们使用`use`语句分别声明它们。为了更好地理解它，请看下面的代码：

```php
use Publishers\Packt\{ Book, Ebook, Video, Presentation };
use function Publishers\Packt\{ getBook, saveBook };
use const Publishers\Packt\{ COUNT, KEY };
```

在一个命名空间中，我们有三种特性：类、函数和常量。因此，我们使用单独的组`use`声明语句来使用它们。现在，代码看起来更清晰、有组织、可读性更好，而且不需要太多重复输入。

### 混合组使用声明

在这个声明中，我们将所有类型合并到一个`use`语句中。看看以下代码：

```php
use Publishers\Packt\{ 
  Book,
  Ebook,
  Video,
  Presentation,
  function getBook,
  function saveBook,
  const COUNT,
  const KEY
};
```

### 复合命名空间声明

为了理解复合命名空间声明，我们将考虑以下标准。

假设我们在`Publishers\Packt\Paper`命名空间中有一个`Book`类。此外，我们在`Publishers\Packt\Electronic`命名空间中有一个`Ebook`类。`Video`和`Presentation`类位于`Publishers\Packt\Media`命名空间中。因此，为了使用这些类，我们将使用以下代码：

```php
use Publishers\Packt\Paper\Book;
use Publishers\Packt\Electronic\Ebook;
use Publishers\Packt\Media\{Video,Presentation};
```

在复合命名空间声明中，我们可以使用前面的命名空间，如下所示：

```php
use Publishers\Packt\{
  Paper\Book,
  Electronic\Ebook,
  Media\Video,
  Media\Presentation
};
```

这更加优雅和清晰，如果命名空间名称很长，它不需要额外的输入。

## 匿名类

匿名类是在声明和实例化同时进行的类。它没有名称，并且可以具有普通类的全部特性。当需要执行一次性的小任务并且不需要为此编写完整的类时，这些类非常有用。

### 注意

在创建匿名类时，它没有名称，但在 PHP 内部使用基于内存块中的地址的唯一引用来命名。例如，匿名类的内部名称可能是`class@0x4f6a8d124`。

这个类的语法与命名类的语法相同，但类的名称缺失，如下所示：

```php
new class(argument) { definition };
```

让我们看一个匿名类的基本和非常简单的例子，如下所示：

```php
$name = new class() {
  public function __construct()
  {
    echo 'Altaf Hussain';
  }
};
```

前面的代码只会显示`Altaf Hussain`。

参数也可以传递给*匿名类构造函数*，如下所示的代码：

```php
$name = new class('Altaf Hussain') {
  public function __construct(string $name)
  {
    echo $name;
  }
};
```

这将给我们与第一个示例相同的输出。

匿名类可以扩展其他类，并且具有与普通命名类相同的父子类功能。让我们看另一个例子；看看以下内容：

```php
class Packt
{
  protected $number;

  public function __construct()
  {
    echo 'I am parent constructor';
  }

  public function getNumber() : float
  {
    return $this->number;
  }
}

$number = new class(5) extends packt
{
  public function __construct(float $number)
  {
    parent::__construct();
    $this->number = $number;
  }
};

echo $number->getNumber();
```

前面的代码将显示`I am parent constructor`和`5`。可以看到，我们扩展`Packt`类的方式与我们扩展命名类的方式相同。此外，我们可以在匿名类中访问`public`和`protected`属性和方法，并且可以使用匿名类对象访问公共属性和方法。

匿名类也可以实现接口，与命名类一样。让我们首先创建一个接口。运行以下代码：

```php
interface Publishers
{
  public function __construct(string $name, string $address);
  public function getName();
  public function getAddress();
}
```

现在，让我们修改我们的`Packt`类如下。我们添加了突出显示的代码：

```php
class Packt
{
  protected $number;
  protected $name;
  protected $address;
  public function …
}
```

代码的其余部分与第一个`Packt`类相同。现在，让我们创建我们的匿名类，它将实现前面代码中创建的`Publishers`接口，并扩展新的`Packt`类，如下所示：

```php
$info = new class('Altaf Hussain', 'Islamabad, Pakistan')extends packt implements Publishers
{
  public function __construct(string $name, string $address)
  {
    $this->name = $name;
    $this->address = $address;
  }

  public function getName() : string
  {
  return $this->name;
  }

  public function getAddress() : string
  {
  return $this->address;
  }
}

echo $info->getName(). ' '.$info->getAddress();
```

前面的代码是不言自明的，并将输出`Altaf Hussain`以及地址。

可以在另一个类中使用匿名类，如下所示：

```php
class Math
{
  public $first_number = 10;
  public $second_number = 20;

  public function add() : float
  {
    return $this->first_number + $this->second_number;
  }

  public function multiply_sum()
  {
    return new class() extends Math
    {
      public function multiply(float $third_number) : float
      {
        return $this->add() * $third_number;
      }
    };
  }
}

$math = new Math();
echo $math->multiply_sum()->multiply(2);
```

前面的代码将返回`60`。这是如何发生的？`Math`类有一个`multiply_sum`方法，返回匿名类的对象。这个匿名类是从`Math`类扩展出来的，并且有一个`multiply`方法。因此，我们的`echo`语句可以分为两部分：第一部分是`$math->multiply_sum()`，它返回匿名类的对象，第二部分是`->multiply(2)`，在这里我们链接了这个对象来调用匿名类的`multiply`方法，并传入值`2`。

在前面的情况下，`Math`类可以被称为外部类，匿名类可以被称为内部类。但是，请记住，内部类不需要扩展外部类。在前面的例子中，我们扩展它只是为了确保内部类可以通过扩展外部类来访问外部类的属性和方法。

## 旧式构造函数弃用

回到 PHP 4 时，类的构造函数与类的同名方法。它仍然被使用，并且在 PHP 的 5.6 版本之前是有效的。然而，现在在 PHP 7 中，它已被弃用。让我们看一个示例，如下所示：

```php
class Packt
{
  public function packt()
  {
    echo 'I am an old style constructor';
  }
}

$packt = new Packt();
```

前面的代码将显示输出`我是一个旧式构造函数`，并附带一个弃用消息，如下所示：

```php
Deprecated: Methods with the same name as their class will not be constructors in a future version of PHP; Packt has a deprecated constructor in…
```

然而，仍然调用旧式构造函数。现在，让我们向我们的类添加 PHP `__construct`方法，如下所示：

```php
class Packt
{
  public function __construct()
  {
    echo 'I am default constructor';
  }

  public function packt()
  {
    echo 'I am just a normal class method';
  }
}

$packt = new Packt();
$packt->packt();
```

在前面的代码中，当我们实例化类的对象时，会调用普通的`__construct`构造函数。`packt()`方法不被视为普通的类方法。

### 注意

旧式构造函数已经被弃用，这意味着它们在 PHP 7 中仍然可以工作，并且会显示一个弃用的消息，但它将在即将推出的版本中被移除。最好不要使用它们。

## 可抛出接口

PHP 7 引入了一个基本接口，可以作为可以使用`throw`语句的每个对象的基础。在 PHP 中，异常和错误可能会发生。以前，异常可以被处理，但无法处理错误，因此，任何致命错误都会导致整个应用程序或应用程序的一部分停止。为了使错误（最致命的错误）也可以被捕获，PHP 7 引入了*throwable*接口，它由异常和错误都实现。

### 注意

我们创建的 PHP 类无法实现可抛出接口。如果需要，这些类必须扩展异常。

我们都知道异常，因此在这个主题中，我们只讨论可以处理丑陋的致命错误的错误。

### 错误

现在几乎所有致命错误都可以抛出错误实例，类似于异常，错误实例可以使用`try/catch`块捕获。让我们来看一个简单的例子：

```php
function iHaveError($object)
{
  return $object->iDontExist();
  {

//Call the function
iHaveError(null);
echo "I am still running";
```

如果执行前面的代码，将显示致命错误，应用程序将停止，并且最终不会执行`echo`语句。

现在，让我们将函数调用放在`try/catch`块中，如下所示：

```php
try 
{
  iHaveError(null);
} catch(Error $e)
{
  //Either display the error message or log the error message
  echo $e->getMessage();
}

echo 'I am still running';
```

现在，如果执行前面的代码，`catch`体将被执行，之后，应用程序的其余部分将继续运行。在前面的情况下，`echo`语句将被执行。

在大多数情况下，错误实例将被抛出，用于最致命的错误，但对于一些错误，将抛出错误的子实例，例如`TypeError`、`DivisionByZeroError`、`ParseError`等。

现在，让我们看一个以下示例中的`DivisionByZeroError`异常：

```php
try
{
  $a = 20;
  $division = $a / 20;
} catch(DivisionByZeroError $e) 
{
  echo $e->getMessage();
}
```

在 PHP 7 之前，前面的代码会发出有关除以零的警告。然而，现在在 PHP 7 中，它将抛出一个`DivisionByZeroError`，可以处理。

# 新运算符

PHP 7 引入了两个有趣的运算符。这些运算符可以帮助编写更少、更清晰的代码，因此最终的代码将比使用传统运算符更易读。让我们来看看它们。

## 太空船运算符（<=>）

太空船或组合比较运算符对于比较值（字符串、整数、浮点数等）、数组和对象非常有用。这个运算符只是一个包装器，执行与三个比较运算符`==`、`<`和`>`相同的任务。这个运算符也可以用于为`usort`、`uasort`和`uksort`的回调函数编写干净和少量的代码。这个运算符的工作方式如下：

+   如果左右两侧的操作数相等，则返回 0

+   如果右操作数大于左操作数，则返回-1

+   如果左操作数大于右操作数，则返回 1

让我们通过比较整数、字符串、对象和数组来看几个例子，并注意结果：

```php
$int1 = 1;
$int2 = 2;
$int3 = 1;

echo $int1 <=> $int3; //Returns 0
echo '<br>';
echo $int1 <=> $int2; //Returns -1
echo '<br>';
echo $int2 <=> $int3; //Returns 1
```

运行前面的代码，你将得到类似以下的输出：

```php
0
-1
1
```

在第一个比较中，我们比较了`$int1`和`$int3`，两者都相等，所以它将返回`0`。在第二个比较中，比较了`$int1`和`$int2`，它将返回`-1`，因为右操作数（`$int2`）大于左操作数（`$int1`）。最后，第三个比较将返回`1`，因为左操作数（`$int2`）大于右操作数（`$int3`）。

上面是一个简单的例子，我们在其中比较了整数。我们可以以相同的方式检查字符串、对象和数组，并且它们是按照标准的 PHP 方式进行比较的。

### 注意

关于`<=>`运算符的一些例子可以在[`wiki.php.net/rfc/combined-comparison-operator`](https://wiki.php.net/rfc/combined-comparison-operator)找到。这是一个 RFC 出版物，其中有关于其用法的更多有用细节。

这个运算符在对数组进行排序时更有用。看看下面的代码：

```php
Function normal_sort($a, $b) : int 
{
  if( $a == $b )
    return 0;
  if( $a < $b )
    return -1;
  return 1;
}

function space_sort($a, $b) : int
{
  return $a <=> $b;
}

$normalArray = [1,34,56,67,98,45];

//Sort the array in asc
usort($normalArray, 'normal_sort');

foreach($normalArray as $k => $v)
{
  echo $k.' => '.$v.'<br>';
}

$spaceArray = [1,34,56,67,98,45];

//Sort it by spaceship operator
usort($spaceArray, 'space_sort');

foreach($spaceArray as $key => $value)
{
  echo $key.' => '.$value.'<br>';
}
```

在前面的代码中，我们使用了两个函数来对具有相同值的两个不同数组进行排序。`$normalArray`数组通过`normal_sort`函数进行排序，`normal_sort`函数使用`if`语句来比较值。第二个数组`$spaceArray`具有与`$normalArray`相同的值，但是这个数组通过`space_sort`函数进行排序，`space_sort`函数使用了太空船运算符。两个数组排序的最终结果是相同的，但回调函数中的代码是不同的。`normal_sort`函数有`if`语句和多行代码，而`space_sort`函数只有一行代码，就是这样！`space_sort`函数的代码更清晰，不需要多个 if 语句。

## 空合并运算符(??)

我们都知道三元运算符，并且大多数时候都会使用它们。三元运算符只是*if-else*语句的单行替代。例如，考虑以下代码：

```php
$post = ($_POST['title']) ? $_POST['title'] : NULL;
```

如果`$_POST['title']`存在，则`$post`变量将被赋予它的值；否则，将被赋予`NULL`。但是，如果`$_POST`或`$_POST['title']`不存在或为 null，则 PHP 将发出*未定义的索引*的通知。为了解决这个通知，我们需要使用`isset`函数，如下所示：

```php
$post = isset($_POST['title']) ? $_POST['title'] : NULL;
```

大多数情况下，看起来都很好，但当我们需要在多个地方检查值时，特别是在使用 PHP 作为模板语言时，情况就会变得非常棘手。

在 PHP 7 中，引入了合并运算符，它很简单，如果第一个操作数（左操作数）存在且不为 null，则返回其值。否则，返回第二个操作数（右操作数）。考虑以下例子：

```php
$post = $_POST['title'] ?? NULL;
```

这个例子与前面的代码完全相似。合并运算符检查`$_POST['title']`是否存在。如果存在，运算符返回它；否则，返回`NULL`。

这个运算符的另一个很棒的特性是它可以链接起来。以下是一个例子：

```php
$title = $_POST['title'] ?? $_GET['title'] ?? 'No POST or GET';
```

根据定义，它将首先检查第一个操作数是否存在并返回它；如果不存在，它将返回第二个操作数。现在，如果第二个操作数上使用了另一个合并运算符，同样的规则将被应用，如果左操作数存在，则返回它的值。否则，将返回右操作数的值。

因此，上面的代码与以下代码相同：

```php
If(isset($_POST['title']))
  $title = $_POST['title'];
elseif(isset($_GET['title']))
  $title = $_GET['title'];
else
  $title = 'No POST or GET';
```

正如前面的例子中所示，合并运算符可以帮助编写干净、简洁和更少的代码。

# 统一变量语法

大多数情况下，我们可能会遇到这样一种情况，即方法、变量或类名存储在其他变量中。看看下面的例子：

```php
$objects['class']->name;
```

在前面的代码中，首先会解释`$objects['class']`，然后会解释属性名。如前面的例子所示，变量通常是从左到右进行评估的。

现在，考虑以下情景：

```php
$first = ['name' => 'second'];
$second = 'Howdy';

echo $$first['name'];
```

在 PHP 5.x 中，这段代码将被执行，并且输出将是`Howdy`。然而，这与从左到右的表达式评估是不一致的。这是因为`$$first`应该首先被评估，然后是索引名称，但在前面的情况下，它被评估为`${$first['name']}`。很明显，变量语法不一致，可能会造成混淆。为了避免这种不一致，PHP 7 引入了一种称为统一变量语法的新语法。如果不使用这种语法，前面的例子将引起注意，并且不会产生期望的结果。为了使其在 PHP 7 中工作，应添加大括号，如下所示：

```php
echo ${$first['name']};
```

现在，让我们举一个例子，如下所示：

```php
class Packt
{
  public $title = 'PHP 7';
  public $publisher = 'Packt Publisher';

  public function getTitle() : string
  {
    return $this->title;
  }

  public function getPublisher() : string
  {
    return $this->publisher;
  }
}

$mthods = ['title' => 'getTitle', 'publisher' => 'getPublisher'];
$object = new Packt();
echo 'Book '.$object->$methods['title']().' is published by '.$object->$methods['publisher']();
```

如果在 PHP 5.x 中执行上述代码，它将正常工作并输出我们想要的结果。但是，如果我们在 PHP 7 中执行此代码，将会产生致命错误。错误将出现在代码的最后一行，这是突出显示的。PHP 7 将首先尝试评估`$object->$method`。之后，它将尝试评估`['title']`；依此类推；这是不正确的。

为了使其在 PHP 7 中工作，应添加大括号，如下所示：

```php
echo 'Book '.$object**->{$methods['title']}**().' is published by '.$object->**{$methods['publisher']}**();
```

在进行了前面提到的更改之后，我们将得到我们想要的输出。

# 其他功能和更改

PHP 7 还引入了一些其他新功能和小的更改，比如数组常量的新语法、`switch`语句中的多个默认情况、`session_start`中的选项数组等。让我们也看看这些。

## 常量数组

从 PHP 5.6 开始，可以使用`const`关键字初始化常量数组，如下所示：

```php
const STORES = ['en', 'fr', 'ar'];
```

现在，从 PHP 7 开始，可以使用`define`函数初始化常量数组，如下所示：

```php
define('STORES', ['en', 'fr', 'ar']);
```

## 在 switch 语句中的多个默认情况

在 PHP 7 之前，允许在 switch 语句中有多个默认情况。请看下面的例子：

```php
switch(true)
{
  default: 
    echo 'I am first one';
    break;
  default: 
    echo 'I am second one';
}
```

在 PHP 7 之前，允许上述代码，但在 PHP 7 中，这将导致类似以下的致命错误：

```php
Fatal error: Switch statements may only contain one default clause in…
```

## `session_start`函数的选项数组

在 PHP 7 之前，每当我们需要启动会话时，我们只是使用`session_start()`函数。这个函数不带任何参数，并且使用`php.ini`中定义的所有设置。现在，从 PHP 7 开始，可以传递一个可选的选项数组，它将覆盖`php.ini`文件中的会话设置。

一个简单的例子如下所示：

```php
session_start([
  'cookie_lifetime' => 3600,
  'read_and_close'  => true
]);
```

如前面的例子所示，很容易覆盖会话的`php.ini`设置。

## 过滤反序列化函数

序列化和反序列化对象是常见的做法。然而，PHP 的`unserialize()`函数并不安全，因为它没有任何过滤选项，并且可以反序列化任何类型的对象。PHP 7 在这个函数中引入了过滤。默认的过滤选项是反序列化所有类或类型的对象。其基本工作如下：

```php
$result = unserialize($object, ['allowed_classes' => ['Packt', 'Books', 'Ebooks']]);
```

# 总结

在本章中，我们讨论了新的面向对象编程功能，如类型提示、匿名类、可抛出接口、命名空间的组合使用声明以及两个重要的新运算符，太空船或组合比较运算符和 null 合并运算符。此外，我们还讨论了统一的变量语法和其他一些新功能，如联系数组定义的新语法、`session_start()`函数的选项数组以及在 switch 语句中删除多个默认情况。

在下一章中，我们将讨论如何提高应用程序的性能。我们将讨论 Apache 和 NGINX 以及它们的不同设置以提高性能。

我们将讨论不同的 PHP 设置，以提高其性能。还将讨论 Google 页面速度模块、CSS/JavaScript 合并和压缩、CDN 等。
