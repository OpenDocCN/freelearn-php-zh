# PHP7，编写更好的代码

PHP7 带来了许多新功能和变化。然而，它们中没有一个是专门针对 REST 或 Web 服务的。事实上，REST 与语言结构没有直接关系。这是因为 REST 是一种架构，而语言是为了提供实现而存在的构造。那么，这是否意味着 PHP7 有一些构造或功能可以使这种实现更好？是和否。这取决于我们对实现的理解。

如果我们的意思只是获取一个请求并返回一个响应，那么不，没有这样的特定功能。但是，任何 RESTful Web 服务都与一个实体相关联，而一个实体可以有自己的逻辑。因此，为了为该实体提供 RESTful Web 服务，我们还需要编写该逻辑。为此，我们需要编写更多的 PHP 代码，而不仅仅是获取请求并返回响应。因此，为了保持代码简单和清晰，是的，PHP7 为我们提供了许多东西。

我假设你已经掌握了 PHP 的基础知识，因为了解 PHP 基础知识是本书的先决条件。因此，我们不会看 PHP5。在本章中，我们将看一下许多 PHP7 的功能和变化，这些功能和变化要么非常重要，要么我们将在我们的代码中使用。我们将直接进入这些功能。我们不会详细介绍安装或升级到 PHP7，因为互联网上有数十个教程可供参考。以下是我们将讨论的功能和变化的列表：

+   标量类型声明

+   返回类型声明

+   空合并运算符

+   太空船运算符

+   组使用语句

+   与生成器相关的新功能

+   生成器返回表达式

+   生成器委托

+   匿名类

+   `Closure::call()`函数

+   错误和异常

+   PHP7.1 功能

+   可空类型

+   对称数组解构

+   `list()`中支持键

+   多捕获异常处理

# 标量类型声明

在 PHP7 中，我们现在可以声明传递给函数的参数的类型。在以前的版本中，它们只能是用户定义的类，但现在它们也可以是标量类型。通过标量类型，我们指的是基本的原始类型，比如`int`、`string`和`float`。

以前，要验证传递给函数的参数，我们需要使用某种`if-else`。因此，我们过去会这样做：

```php
<?php
function add($num1, $num2){
    if (!is_int($num1)){
        throw new Exception("$num1 is not an integer");
    }
    if (!is_int($num2)){
        throw new Exception("$num2 is not an integer");
    }

    return ($num1+$num2);
}

echo add(2,4);  // 6
echo add(1.5,4); //Fatal error:  Uncaught Exception: 1.5 is not an integer
```

在这里，我们使用`if`来确保变量`$num1`和`$num2`的类型是`int`，否则我们会抛出异常。如果你是一个喜欢尽可能少写代码的早期 PHP 开发人员，那么你甚至可能根本不检查参数的类型。然而，如果你不检查参数类型，这可能导致运行时错误。因此，为了避免这种情况，应该检查参数类型，这就是 PHP7 所做的事情。

这是我们现在在 PHP7 中验证参数类型的方式：

```php
<?php
function add(int $num1,int $num2){
    return ($num1+$num2);
}
echo add(2,4); //6
echo add("2",4); //6
echo add("something",4); 
//Fatal error:  Uncaught TypeError: Argument 1 passed to add() must be of the type integer, string given

```

正如你现在所看到的，我们只需将`int`作为类型提示，而不需要单独验证每个参数。如果参数不是整数，它应该抛出异常。然而，你可以看到，当`2`作为字符串传递时，它并没有显示`TypeError`，而是进行了隐式转换，并将其视为`int 2`。这是因为，默认情况下，PHP 代码是在强制模式下运行的。如果启用了严格模式，写`"2"`而不是 2 将导致`TypeError`而不是隐式转换。要启用严格模式，我们需要在 PHP 代码的开头使用`declare`函数。

这是我们可以这样做的方式：

```php
<?php
declare(strict_types=1); 

function add(int $num1,int $num2){
    return ($num1+$num2);
}

echo add(2,4); //6
echo add("2",4); //Fatal error:  Uncaught TypeError: Argument 1 passed to add() must be of the type integer, string given,

echo add("something",4); // Fatal error:  Uncaught TypeError: Argument 1 passed to add() must be of the type integer, string given
```

# 返回类型声明

就像参数类型一样，还有一个返回类型；它也是可选的，但指定返回类型是一种安全的做法。

这是我们可以声明返回类型的方式：

```php
<?php
function add($num1, $num2):int{
    return ($num1+$num2);
}

echo add(2,4); //6
echo add(2.5,4); //6
```

正如你在`2.5`和`4`的情况下所看到的，它应该是`6.5`，但由于我们指定了`int`作为返回类型，它执行了隐式类型转换。为了避免这种情况，以及获得隐式转换而不是错误，我们可以简单地启用严格类型，如下所示：

```php
<?php
declare(strict_types=1);
function add($num1, $num2):int{
    return ($num1+$num2);
}

echo add(2,4); //6
echo add(2.5,4); //Fatal error:  Uncaught TypeError: Return value of add() must be of the type integer, float returned
```

# 空合并运算符

`Null`合并操作符(`??`)是一种语法糖，但非常重要。在 PHP5 中，当我们有一些可能未定义的变量时，我们使用三元运算符如下：

```php
$username = isset($_GET['username']) ? $_GET['username'] : '';
```

然而，现在在 PHP7 中，我们可以简单地写：

```php
$username = $_GET['username'] ?? '';
```

尽管这只是一种语法糖，但它可以节省时间，使代码更清晰。

# 太空船操作符

太空船操作符也是比较的快捷方式，在用户定义的排序函数中非常有用。我不会详细介绍这个，因为它在文档中已经有足够的解释：[`php.net/manual/en/migration70.new-features.php#migration70.new-features.spaceship-op`](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.spaceship-op)。

# 组合使用声明

现在可以在单个`use`语句中导入相同命名空间中的类、函数和常量。以前需要多个`use`语句。以下是一个例子，以便更好地理解它：

```php
<?php
// use statement in Pre-PHP7 code
use abc\namespace\ClassA;
use abc\namespace\ClassB;
use abc\namespace\ClassC as C;

use function abc\namespace\funcA;
use function abc\namespace\funcB;
use function abc\namespace\funcC;

use const abc\namespace\ConstA;
use const abc\namespace\ConstB;
use const abc\namespace\ConstC;

// PHP 7+ code
use abc\namespace\{ClassA, ClassB, ClassC as C};
use function abc\namespace\{funcA, funcB, funcC};
use const abc\namespace\{ConstA, ConstB, ConstC};
```

从这个例子中可以看出，组合使用语句有多么方便，这是显而易见的。使用大括号和逗号分隔的值来组合值，比如`{classA, classB, classC as C}`，这样就可以得到组合的`use`语句，而不是分别为这三个类使用`use`语句，每个类使用三次。

# 与生成器相关的功能

尽管生成器在 PHP5.5 中出现，但大多数 PHP 开发人员不使用它们，很可能也不了解生成器。因此，让我们首先讨论生成器。

# 生成器是什么？

如 PHP 手册所述：

生成器提供了一种简单的方法来实现简单的迭代器，而不需要实现实现迭代器接口的类的开销或复杂性。

好的，这里有一个更详细和易于理解的定义，来自同一来源，[php.net](http://php.net)：

生成器允许您编写使用 foreach 来迭代一组数据的代码，而无需在内存中构建数组，这可能会导致超出内存限制，或需要大量的处理时间来生成。相反，您可以编写一个生成器函数，它与普通函数相同，只是它不是一次返回，而是生成器可以 yield 多次，以便提供要迭代的值。

例如，您可以简单地编写一个返回许多不同数字或值的函数。但问题是，如果许多不同的值意味着数百万个值，那么制作并返回一个包含这些值的数组是不高效的，因为它将消耗大量内存。因此，在这种情况下，使用`generator`更有意义。

要了解，请参阅此示例：

```php
/* function to return generator */
function getValues($max){
    for($i=0; $i<$max; $i++ ){
        yield $i*2;
    }
}

// Using generator
foreach(getValues(99999) as $value){
    echo "Values: $value <br>";
}
```

正如你所看到的，代码中有一个`yield`语句。它就像 return 语句一样，但在生成器中，`yield`不会一次返回所有的值。它只会在每次 yield 执行时返回一个值，并且只有在调用`generator`函数时才会调用 yield。此外，每次 yield 执行时，它都会从上次停止的地方恢复代码执行。

现在我们了解了生成器，让我们看看与生成器相关的 PHP7 功能。

# 生成器返回表达式

正如我们之前所看到的，在调用生成器函数时，它返回的是由 yield 表达式返回的值。在 PHP7 之前，它没有`return`关键字返回一个值。但自 PHP7.0 以来，也可以使用 return 表达式。在这里，我使用了 PHP 文档中的一个例子，因为它解释得很好：

```php
<?php

$gen = (function() {
    yield "First Yield";
    yield "Second Yield";

    return "return Value";
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
```

它将输出为：

```php
First Yield
Second Yield
return Value
```

因此，它清楚地显示，在`foreach`中调用生成器函数不会返回`return`语句。相反，它只会在每次 yield 时返回。要获取`return Value`，可以使用这个语法：`$gen->getReturn()`。

# 生成器委托

函数可以互相调用，同样，生成器也可以委托给另一个生成器。以下是生成器的委托方式：

```php
<?php
function gen()
{
    yield "yield 1 from gen1";
    yield "yield 2 from gen1";
    yield from gen2();
}

function gen2()
{
    yield "yield 1 from gen2";
    yield "yield 2 from gen2";
}

foreach (gen() as $val)
{
    echo $val, PHP_EOL;
}

/* above will result in output:
yield 1 from gen1
yield 2 from gen1
yield 1 from gen2
yield 2 from gen2 
*/

```

在这里，`gen2()`是在`gen()`中调用的另一个生成器，因此在`gen()`中的第三个 yield，即`yield from gen2();`，将控制转移到`gen2()`。因此，它将开始使用`gen2()`的 yield。

请注意，`yield from`只能与数组、可遍历对象或生成器一起使用。在`yield from`中使用另一个函数（而不是生成器）将导致致命错误。

您可以将其视为在另一个函数中调用函数的方式。

# 匿名类

就像匿名函数一样，现在 PHP 中也有匿名类。请注意，如果需要对象，则很可能我们需要某种特定类型的对象，而不仅仅是随机的，例如：

```php
<?php
class App
{
    public function __construct()
    {
        //some code here
    }
}

function useApp(App $app)
{
    //use app somewhere
}

$app = new App();
useApp($app);
```

请注意，在`useApp()`函数中需要一个特定类型的对象，并且如果它不是一个类，那么这个类型`App`就无法定义。那么我们在哪里以及为什么要使用一个具有特定功能的匿名类？我们可能需要它，以防我们需要传递一个实现某个特定接口或扩展某个父类的类，但只想在一个地方使用这个类。在这种情况下，我们可以使用匿名类。

这是 PHP7 文档中给出的相同示例，这样您就可以更容易地跟进：

```php
<?php
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger()); //object(class@anonymous)#2 (0) {}
```

正如您所看到的，尽管在`$app->setLogger()`中传递了一个匿名类对象，但它也可以是一个命名类对象。因此，匿名类对象可以被命名类对象替换。但是，当我们不想再次使用同一类的对象时，最好使用匿名类对象。

# Closure::call()

将对象范围与闭包绑定是使用不同对象的闭包的有效方法。同时，它也是在不同位置为对象使用具有不同行为的不同闭包的简单方法。这是因为它在运行时绑定了对象范围与闭包，而不需要继承、组合等。

然而，以前我们没有`Closure::call()`方法；我们有类似于这样的东西：

```php
<?php
// Pre PHP 7 code
class Point{
    private $x = 1; 
    private $y = 2;
}

$getXFn = function() {return $this->x;};
$getX = $getXFn->bindTo(new Point, 'Point');//intermediate closure
echo $getX(); // will output 1
```

但现在有了`Closure::call()`，可以将相同的代码编写如下：

```php
<?php
//  PHP 7+ code
class Point{
    private $x = 1; 
    private $y = 2;
}

// PHP 7+ code
$getX = function() {return $this->x;};
echo $getX->call(new Point); // outputs 1 as doing same thing
```

这两个代码片段执行相同的操作。但是，PHP7+代码是简写的。如果需要将一些参数传递给闭包函数，可以在对象之后传递它们，如下所示：

```php
<?php
// PHP 7+ closure call with parameter and binding

class Point{
 private $x = 1; 
 private $y = 2;
}

$getX = function($margin) {return $this->x + $margin;};
echo $getX->call(new Point, 2); //outputs 3 by ($margin + $this->x)
```

# 错误和异常

在 PHP7 中，大多数错误现在被报告为错误异常。只有少数致命错误会停止脚本执行；否则，如果进行错误或异常处理，它不会停止脚本。这是因为现在`Errors`类实现了`Throwable`接口，就像`Exception`类一样，它也实现了`Throwable`。因此，现在在大多数情况下，通过异常处理可以避免致命错误。

以下是错误类的一些子类：

+   `TypeError`

+   `ParseError`

+   `ArithmeticError`

+   `DivisionByZeroError`

+   `AssertionError`

这是您可以简单捕获错误并处理它的方式：

```php
try {
    fn();
} catch(Throwable $error){
    echo $error->getMessage(); //Call to undefined function fn()
}
```

在这里，`$error->getMessage()`是一个实际返回此消息作为字符串的方法。在我们之前的示例中，消息将类似于：`Call to undefined function fn()`.

这不是您可以使用的唯一方法。以下是在`Throwable`接口中定义的方法列表；您可以在错误/异常处理期间相应地使用它们。毕竟，`Exception`和`Error`类都实现了相同的`Throwable`接口：

```php
interface Throwable
{
    public function getMessage(): string;
    public function getCode(): int;
    public function getFile(): string;
    public function getLine(): int;
    public function getTrace(): array;
    public function getTraceAsString(): string;
    public function getPrevious(): Throwable;
    public function __toString(): string;
}
```

# PHP7.1

到目前为止，我们讨论的前面的功能都是与 PHP7.0 相关的。然而，PHP7 的最新版本是 PHP7.1，因此讨论 PHP7.1 的重要功能也是值得的，至少是我们将在工作中使用的功能，或者是值得知道并在某个地方使用的功能。

要运行以下代码，您需要安装 PHP7.1，因此，您可以使用以下命令：

```php
sudo add-apt-repository ppa:ondrej/php
```

```php
sudo apt-get update
```

```php
(optional) sudo apt-get remove php7.0
```

```php
sudo apt-get install php7.1 (from comments)
```

请记住，这不是官方的升级路径。PPA 是众所周知的，并且相对安全。

# 可空类型

如果我们对参数的数据类型或函数的返回类型进行类型提示，那么重要的是应该有一种方法来传递或返回`NULL`数据类型，而不是作为参数或返回类型进行类型提示。

可能会有不同的情况需要这样做，但我们总是需要在数据类型之前放置一个`?`。假设我们想要对`string`进行类型提示；如果我们想要使其可为空，也就是允许`NULL`，我们只需将其类型提示为`?` string。

例如：

```php
<?php

function testReturn(): ?string
{
    return 'testing';
}

var_dump(testReturn());
// string(10) "testing"

function testReturn2(): ?string
{
    return null;
}

var_dump(testReturn2());
//NULL

function test(?string $name)
{
    var_dump($name);
}

test('testing');
//string(10) "testing"

test(null);
//NULL

test();
// Fatal error:  Uncaught ArgumentCountError: Too few arguments // to function test(),
```

# 对称数组解构

这不是一个重大的功能，但它是`list()`的方便缩写。因此，可以在以下示例中快速看到：

```php
<?php
$records = [
    [7, 'Haafiz'],
    [8, 'Ali'],
];

// list() style
list($firstId, $firstName) = $records[0];

// [] in PHP7.1 is having same result
[$firstId, $firstName] = $records[0];

```

# 对`list()`中的键的支持

正如您在前面的例子中所看到的，`list()`与数组一起工作，并按相同的顺序分配给变量。但是，根据 PHP7.1，`list()`现在支持键。由于`[]`是`list()`的缩写，`[]`也支持键。

以下是前述描述的一个示例：

```php
<?php
$records = [
    ["id" => 7, "name" => 'Haafiz'],
    ["id" => 8, "name" => 'Ali'],
];

// list() style
list("id" => $firstId, "name" => $firstName) = $records[0];

// [] style
["id" => $firstId, "name" => $firstName] = $records[0];
```

在这里，ID`$firstId`在前面的代码执行后将具有`7`，而`$firstName`将具有`Haafiz`，无论是使用`list()`样式还是`[]`样式。

# 多异常捕获处理

这是 PHP7.1 中一个有趣的功能。以前是可能的，但是需要多个步骤来执行。现在，不仅仅是捕获一个异常并处理它，还可以使用多异常捕获处理功能。语法可以在这里看到：

```php
<?php
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}
```

正如您在这里所看到的，有一个管道符号分隔这两个异常。因此，这个管道符号`|`分隔多个异常。在这个例子中只有两个异常，但可能会有更多。

# 更多资源

我们讨论了 PHP7 和 PHP 7.1（PHP7 的最新版本）的新功能，这些功能我们要么认为很重要要讨论，要么我们将在本书的其余部分中使用。但是，我们没有完全讨论 PHP7 的功能。您可以在[php.net](http://php.net)上找到 PHP7 功能列表：[`php.net/manual/en/migration70.new-features.php`](http://php.net/manual/en/migration70.new-features.php)。

在这里，您可以找到 PHP 7.1 的所有新功能：[`php.net/manual/en/migration71.new-features.php`](http://php.net/manual/en/migration71.new-features.php)。

# 总结

在本章中，我们讨论了重要的 PHP7 功能。此外，我们还介绍了新的 PHP7.1 功能。本章涵盖了本书其余部分将使用的基础知识。请注意，使用 PHP7 功能并不是必需的，但它可以帮助我们高效地编写简化的代码。

在下一章中，我们将开始在 PHP 中创建一个 RESTful API，就像我们在第一章中讨论的那样，*RESTful Web Services, Introduction and Motivation*，同时利用一些 PHP7 的功能。
