# 错误处理和日志记录

有效的错误处理和日志记录是应用程序的重要部分。早期版本的 PHP 缺乏对异常的支持，只使用错误来标记有缺陷的应用程序状态。PHP 5 版本为语言带来了面向对象的特性，以及异常模型。这使 PHP 具有了像其他编程语言一样的`try...catch`块。后来，PHP 5.5 版本增加了对`finally`块的支持，无论是否抛出异常，它始终在`try...catch`块之后执行。

如今，PHP 语言将错误和异常区分为应用程序的故障状态。两者都被视为应用程序逻辑的意外情况。有许多类型的错误，比如`E_ERROR`、`E_WARNING`、`E_NOTICE`等。当谈到错误时，我们默认为`E_ERROR`类型，它往往表示应用程序的结束，这是一个意外的状态，应用程序不应该尝试捕获并继续执行。这可能是由于内存不足、IO 错误、TCP/IP 错误、空引用错误等。另一方面，异常表示应用程序可能希望捕获并继续执行的意外状态。这可能是由于在给定时间无法保存数据库中的条目，意外的电子邮件发送失败等。这有助于将异常视为错误的面向对象概念。

PHP 有自己的机制，允许与一些错误类型和异常进行交互。使用`set_error_handler`，我们可以定义自定义错误处理程序，可能记录或向用户显示适当的消息。使用`try...catch...finally`块，我们可以安全地捕获可能的异常并继续执行应用程序。我们没有自动捕获的异常会自动转换为标准错误，并中断应用程序的执行。

处理错误如果没有适当的日志记录机制，就不会真正完整。虽然 PHP 本身提供了一个有趣和有用的`error_log()`函数，但在社区库中还有更强大的日志记录解决方案，比如 Mongo。

接下来，我们将详细研究以下错误处理和日志记录领域：

+   错误处理

+   错误

+   算术错误

+   `DivisionByZeroError`

+   `AssertionError`

+   `ParseError`

+   `TypeError`

+   异常

+   日志记录

+   本机日志记录

+   使用 Monolog 进行日志记录

NASA 在 1999 年 9 月丢失了一枚价值 1.25 亿美元的火星轨道器，因为工程师未能将单位从英制转换为公制。虽然这个系统与 PHP 或致命的运行时错误无关，但它表明了一个有缺陷的软件可能在现实生活中产生多大的影响。

# 错误处理

将错误和异常作为两种不同的错误处理系统引入了一定程度的混乱。早期版本的 PHP 使得很难理解`E_ERROR`，因为它们无法被自定义错误处理程序捕获。PHP 7 版本试图通过引入`Throwable`接口来解决这种混乱，总结如下：

```php
Throwable { 
  abstract public string getMessage (void) 
  abstract public int getCode (void) 
  abstract public string getFile (void) 
  abstract public int getLine (void) 
  abstract public array getTrace (void) 
  abstract public string getTraceAsString (void) 
  abstract public Throwable getPrevious (void) 
  abstract public string __toString (void) 
}

```

`Throwable`接口现在是`Error`、`Exception`和通过`throw`语句抛出的任何其他对象的基本接口。该接口中定义的方法几乎与`Exception`的方法相同。PHP 类本身不能直接实现`Throwable`接口或扩展自`Error`；它们只能扩展`Exception`，如下例所示：

```php
<?php

  class Glitch extends \Error
  {
  }

  try {
    throw new Glitch('Glitch!');
  } 
  catch (\Exception $e) {
    echo 'Caught ' . $e->getMessage();
  }

```

前面的代码将产生以下输出：

```php
PHP Fatal error: Uncaught Glitch: Glitch! in index.php:7
Stack trace:
#0 {main}
thrown in /root/app/index.php on line 7

```

这里发生的情况是`Glitch`类试图扩展`Error`类，这是不允许的，导致了一个致命错误，我们的`try...catch`块无法捕获到：

```php
<?php

  class Flaw extends \Exception
  {
  }

  try {
    throw new Flaw('Flaw!');
  } 
  catch (\Exception $e) {
    echo 'Caught ' . $e->getMessage();
  }

```

前面的例子是 PHP `Throwable`的有效用法，而我们的自定义`Flaw`类扩展了`Exception`类。触发`catch`块，导致以下输出消息：

```php
Caught Flaw!

```

PHP 7 中的新异常层次结构如下：

```php
interface Throwable
 | Error implements Throwable
   | TypeError extends Error
   | ParseError extends Error
   | ArithmeticError extends Error
     | DivisionByZeroError extends ArithmeticError
   | AssertionError extends Error
 | Exception implements Throwable
   | ...

```

新的`Throwable`接口的明显好处是我们现在可以在单个`try...catch`块中轻松捕获`Exception`和`Error`对象，如下例所示：

```php
<?php

try {
  throw new ArithmeticError('Missing numbers!');
} 
catch (Throwable $t) {
  echo $t->getMessage();
}

```

`AssertionError`扩展了`Error`，而`Error`又实现了`Throwable`接口。上面的`catch`块的签名针对`Throwable`接口，因此抛出的`ArithmeticError`将被捕获，并显示`Missing numbers!`的输出。

虽然我们的类不能实现`Throwable`接口，但我们可以定义扩展它的接口。这样的接口只能由扩展`Exception`或`Error`的类来实现，如下例所示：

```php
<?php   interface MyThrowable extends Throwable
 {  //...
 } class MyException extends Exception implements MyThrowable
 {  //...
 } throw new MyException();

```

虽然这可能不是常见的做法，但这种方法可能对特定于包的接口有用。

# 错误

`Error`类是 PHP 7 中内部 PHP 错误的基类。现在，PHP 5.x 中几乎所有致命和可恢复的致命错误都会抛出`Error`对象的实例，从而可以通过`try...catch`块捕获。

`Error`类根据以下类概要实现了`Throwable`接口：

```php
Error implements Throwable {
   /* Properties */
   protected string $message ;
   protected int $code ;
   protected string $file ;
   protected int $line ;

   /* Methods */
   public __construct (
     [ string $message = "" 
     [, int $code = 0 
     [, Throwable $previous = NULL ]]]
    )

    final public string getMessage (void)
    final public Throwable getPrevious (void)
    final public mixed getCode (void)
    final public string getFile (void)
    final public int getLine (void)
    final public array getTrace (void)
    final public string getTraceAsString (void)
    public string __toString (void)
    final private void __clone (void)
}

```

以下示例演示了在`catch`块中使用`Error`实例：

```php
<?php

class User
{
  function hello($name)
  {
    return 'Hello ' . $name;
  }
}

// Case 1 - working
try {
  $user = new User();
  $user->greeting('John');
} 
catch (Error $e) {
  echo 'Caught: ' . $e->getMessage();
}

// Case 2 - working
try {
  $user = new User();
  $user->greeting('John');
} 
catch (Throwable $t) {
  echo 'Caught: ' . $t->getMessage();
}

```

然而，仍然有一些情况下一些错误是无法捕获的：

```php
<?php

ini_set('memory_limit', '1M');

try {
  $content = '';
  while (true) {
    $content .= 'content';
  }
} 
catch (\Error $e) {
  echo 'Caught ' . $e->getMessage();
}

```

上面的例子触发了`PHP Fatal error: Allowed memory size of 2097152 bytes exhausted...`错误。

此外，即使警告也会被忽略，如下例所示：

```php
    <?php

    error_reporting(E_ALL);
    ini_set('display_errors', 1);
    ini_set('memory_limit', '1M');

    try {
      str_pad('', PHP_INT_MAX);
    } 
    catch (Throwable $t) {
      echo 'Caught ' . $t->getMessage();
    }

```

上面的例子触发了`PHP Warning:  str_pad(): Padding length is too long...`错误。

可以说，我们应该谨慎对待捕获核心语言错误的期望，因为有些错误可能会漏掉。那些被捕获的通常是基类`Error`。然而，一些错误会抛出更具体的`Error`子类：`ArithmeticError`、`DivisionByZeroError`、`AssertionError`、`ParseError`和`TypeError`。

# ArithmeticError

`ArithmeticError`类解决了执行数学运算可能出现错误结果的情况。PHP 将其用于两种情况——通过负数进行位移或者使用`intdiv()`时被除数为`PHP_INT_MIN`，除数为`-1`。

`ArithmeticError`类没有自己的方法，它们都是从父类`Error`类继承而来，如下类概要所示：

```php
     ArithmeticError extends Error {
       final public string Error::getMessage (void)
       final public Throwable Error::getPrevious (void)
       final public mixed Error::getCode (void)
       final public string Error::getFile (void)
       final public int Error::getLine (void)
       final public array Error::getTrace (void)
       final public string Error::getTraceAsString (void)
       public string Error::__toString (void)
       final private void Error::__clone (void)
     }

```

以下示例演示了使用负数进行位移时抛出`ArithmeticError`的`try...catch`块：

```php
    <?php

    try {
      $value = 5 << -1;
    } 
    catch (ArithmeticError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

结果输出如下：

```php
 Caught: Bit shift by negative number 

```

以下示例演示了使用`intdiv()`调用时抛出`ArithmeticError`的`try...catch`块，被除数为`PHP_INT_MIN`，除数为`-1`：

```php
    <?php

    try {
      intdiv(PHP_INT_MIN, -1);
    } 
    catch (ArithmeticError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

结果输出如下：

```php
 Caught: Division of PHP_INT_MIN by -1 is not an integer

```

# DivisionByZeroError

在基本算术中，除以零是一个未定义的数学表达式；因此，PHP 需要一种方式来应对这种情况。当我们尝试除以零时，将抛出`DivisionByZeroError`。

`DivisionByZeroError`类没有自己的方法，它们都是从父类`ArithmeticError`继承而来，如下类概要所示：

```php
    DivisionByZeroError extends ArithmeticError {
      final public string Error::getMessage (void)
      final public Throwable Error::getPrevious (void)
      final public mixed Error::getCode (void)
      final public string Error::getFile (void)
      final public int Error::getLine (void)
      final public array Error::getTrace (void)
      final public string Error::getTraceAsString (void)
      public string Error::__toString (void)
      final private void Error::__clone (void)
    }

```

我们需要注意我们使用什么表达式进行除法。仅使用`/`运算符将被除数数字除以`0`除数数字将不会产生与使用`intdiv()`函数相同的结果。考虑以下代码片段：

```php
    <?php

    try {
      $x = 5 / 0;
    } 
    catch (DivisionByZeroError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

上面的例子不会触发`DivisionByZeroError`的 catch 块。相反，会引发以下警告。

```php
PHP Warning: Division by zero

```

使用`intdiv()`函数而不是`/`运算符将触发`catch`块，如下面的代码片段所示：

```php
    <?php

    try {
      $x = intdiv(5, 0);
    } 
    catch (DivisionByZeroError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

如果除数为`0`，`intdiv()`函数会抛出`DivisionByZeroError`异常。如果被除数是`PHP_INT_MIN`，除数是`-1`，那么会抛出`ArithmeticError`异常，如前面的部分所示。

# AssertionError

断言是作为调试功能使用的运行时检查。使用 PHP 7 的`assert()`语言结构，我们可以确认某些 PHP 表达式是真还是假。每当断言失败时，就会抛出`AssertionError`。

`AssertionError`类没有自己的方法，它们都是从父类`Error`继承而来，如下类概要所示：

```php
    AssertionError extends Error {
      final public string Error::getMessage (void)
      final public Throwable Error::getPrevious (void)
      final public mixed Error::getCode (void)
      final public string Error::getFile (void)
      final public int Error::getLine (void)
      final public array Error::getTrace (void)
      final public string Error::getTraceAsString (void)
      public string Error::__toString (void)
      final private void Error::__clone (void)
    }

```

PHP 7 提供了两个配置指令来控制`assert()`的行为--`zend.assertions`和`assert.exception`。只有当`zend.assertions = 1`和`assert.exception = 1`时，`assert()`函数才会被执行并可能抛出`AssertionError`，如下例所示：

```php
    <?php

    try {
      assert('developer' === 'programmer');
    } 
    catch (AssertionError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

假设配置指令都已设置，上述代码将输出`Caught: assert('developer' === 'programmer')`消息。如果只有`zend.assertions = 1`但`assert.exception = 0`，那么`catch`块将没有效果，并且会引发以下警告：`Warning: assert(): assert('developer' === 'programmer') failed`。

`zend.assertions`派生可能在`php.ini`文件中完全启用或禁用。

# ParseError

`eval()`语言结构使我们能够执行任意的 PHP 代码。唯一的要求是代码不能包含在开头和结尾的 PHP 标记中。除此之外，传递的代码本身必须是有效的 PHP 代码。如果传递的代码无效，那么就会抛出`ParseError`。

`ParseError`类没有自己的方法，它们都是从父类`Error`继承而来，如下类概要所示：

```php
    ParseError extends Error {
      final public string Error::getMessage (void)
      final public Throwable Error::getPrevious (void)
      final public mixed Error::getCode (void)
      final public string Error::getFile (void)
      final public int Error::getLine (void)
      final public array Error::getTrace (void)
      final public string Error::getTraceAsString (void)
      public string Error::__toString (void)
      final private void Error::__clone (void)
    }

```

以下代码片段演示了有效的`eval()`表达式：

```php
    <?php

    try {
      $now = eval("return date('D, d M Y H:i:s');");
      echo $now;
    } 
    catch (ParseError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

以下代码块演示了在评估代码中的解析错误：

```php
    <?php

    try {
      $now = eval("return date(D, d M Y H:i:s);");
      echo $now;
    } 
    catch (ParseError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

几乎与一个正常工作的例子相同，你会注意到在日期函数参数周围缺少开头和结尾的(`'`)字符。这会破坏 eval 函数，触发`ParseError` catch 块，并输出以下内容：

```php
Caught: syntax error, unexpected 'M' (T_STRING), expecting ',' or ')'

```

现在，让我们看一下以下代码片段：

```php
    <?php

    try {
      $now = date(D, d M Y H:i:s);
      echo $now;
    }
    catch (ParseError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

在这里，我们没有使用`eval()`表达式，而是故意破坏了代码。结果输出触发了解析错误，但这次不是通过对`catch`块的反应，这有点意料之中。在现代 IDE 环境中，如 PhpStorm、Netbeans 等，这种特定情况几乎不太可能发生，因为它们会自动警告我们有损坏的语法。

# TypeError

PHP 7 引入了*函数类型参数*和*函数返回类型*。这反过来意味着需要正确处理它们的误用错误。`TypeError`被引入来解决这些错误。

`TypeError`类没有自己的方法，它们都是从父类`Error`继承而来，如下类概要所示：

```php
    ParseError extends Error {
      final public string Error::getMessage (void)
      final public Throwable Error::getPrevious (void)
      final public mixed Error::getCode (void)
      final public string Error::getFile (void)
      final public int Error::getLine (void)
      final public array Error::getTrace (void)
      final public string Error::getTraceAsString (void)
      public string Error::__toString (void)
      final private void Error::__clone (void)
    } 

```

有至少三种可能的错误场景会引发`TypeError`，如下所示：

+   传递给函数的参数类型与声明的类型不匹配

+   函数返回值与声明的函数返回类型不匹配

+   传递给内置 PHP 函数的参数数量无效

以下代码演示了错误的函数参数类型：

```php
    <?php

    declare(strict_types = 1);

    function hello(string $name) {
      return "Hello $name!";
    }
    try {
      echo hello(34);
    } 
    catch (TypeError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

在这里，我们定义了`hello()`函数，它期望接收一个字符串参数。然而，函数被传递了整数值。如果我们希望`catch`块实际上捕获`TypeError`，则需要`declare(strict_types = 1);`表达式。上述例子的结果如下输出：

```php
Caught: Argument 1 passed to hello() must be of the type string, integer given, called in...

```

以下代码演示了错误的函数返回类型：

```php
    <?php

    declare(strict_types = 1);

    function hello($name): string {
      return strlen($name);
    }

    try {
      echo hello('branko');
    } 
    catch (TypeError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

在这里，定义的`hello()`函数没有定义特定的参数类型，但确实定义了函数返回类型。为了模拟错误的情况，我们将函数体改为返回整数值而不是字符串。与前面的例子一样，需要声明`strict_types = 1`来触发`TypeError`，结果如下输出：

```php
Caught: Return value of hello() must be of the type string, integer returned

```

以下代码演示了传递给内置 PHP 函数的无效参数数量：

```php
    <?php

    declare(strict_types = 1);

    try {
      echo strlen('test', 'extra');
    } 
    catch (TypeError $e) {
      echo 'Caught: ' . $e->getMessage();
    }

```

在这里，我们使用两个参数调用`strlen()`函数。虽然这个核心 PHP 函数本身是定义为只接受一个参数，但`strict_types = 1`声明将标准警告转换为`TypeError`，从而触发`catch`块。

# 未捕获的错误处理程序

虽然现在可以通过`try...catch`捕获大量的`Error`，但也有一种额外的机制来处理错误。PHP 提供了一种机制，即`set_error_handler()`函数，允许我们为所有未捕获的错误定义一个自定义处理程序函数。`set_error_handler()`函数接受两个参数，如下面的描述所示：

```php
    mixed set_error_handler ( 
      callable $error_handler 
      [, int $error_types = E_ALL | E_STRICT ] 
    )

```

`$error_handler`函数可以是作为字符串传递的处理程序函数名称，也可以是整个匿名处理程序函数，而`$error_types`是一个或多个（用`|`分隔）指定错误类型的掩码。处理程序函数本身也接受几个参数，如下面的描述所示：

```php
    bool handler ( 
      int $errno , 
      string $errstr 
      [, string $errfile 
        [, int $errline 
          [, array $errcontext ]]] 
    )

```

让我们看看以下两个例子：

```php
    <?php

    function handler($errno, $errstr, $errfile, $errline, $errcontext)

    {
      echo 'Handler: ' . $errstr;
    }

    set_error_handler('handler', E_USER_ERROR | E_USER_WARNING);

    echo 'start';
      trigger_error('Ups!', E_USER_ERROR);
    echo 'end';

```

```php
    <?php

    set_error_handler(function ($errno, $errstr, $errfile, $errline,
      $errcontext) {
      echo 'Handler: ' . $errstr;
    }, E_USER_ERROR | E_USER_WARNING);

    echo 'start';
      trigger_error('Ups!', E_USER_WARNING);
    echo 'end';

```

这些例子几乎是相同的。第一个例子使用了一个单独定义的处理程序函数，然后将其作为字符串参数传递给`set_error_handler()`。第二个例子使用了相同定义的匿名函数。这两个例子都使用`trigger_error()`函数，一个触发`E_USER_ERROR`，另一个触发`E_USER_WARNING`。执行时，两个输出都将包含`end`字符串。

虽然自定义处理程序函数使我们能够处理各种运行时错误，但有一些错误是我们无法处理的。以下错误类型无法使用用户定义的函数处理：`E_ERROR`、`E_PARSE`、`E_CORE_ERROR`、`E_CORE_WARNING`、`E_COMPILE_ERROR`、`E_COMPILE_WARNING`，以及在调用`set_error_handler()`的文件中引发的大多数`E_STRICT`。

# 触发错误

PHP 的`trigger_error()`函数提供了一种触发用户级错误/警告/通知消息的方法。它可以与内置错误处理程序一起使用，也可以与用户定义的错误处理程序一起使用，就像我们在前一节中看到的那样。

`trigger_error()`函数接受两个参数，如下面的描述所示：

```php
    bool trigger_error ( 
      string $error_msg 
      [, int $error_type = E_USER_NOTICE ] 
    )

```

`$error_msg`参数的限制为 1024 字节，而`$error_type`限制为`E_USER_ERROR`、`E_USER_WARNING`、`E_USER_NOTICE`和`E_USER_DEPRECATED`常量。

让我们看看以下例子：

```php
    <?php

    set_error_handler(function ($errno, $errstr) {
      echo 'Handler: ' . $errstr;
    });

    echo 'start';
    trigger_error('E_USER_ERROR!', E_USER_ERROR);
    trigger_error('E_USER_ERROR!', E_USER_WARNING);
    trigger_error('E_USER_ERROR!', E_USER_NOTICE);
    trigger_error('E_USER_ERROR!', E_USER_DEPRECATED);
    echo 'end';

```

在这里，我们有四个不同的`trigger_error()`函数调用，每个函数接受不同的错误类型。自定义错误处理程序对所有四个错误都起作用，我们的代码继续执行，最终输出`end`。

**错误模型**（`set_error_handler`和`trigger_error`）和**可抛出模型**（`try...catch`和`throw new ...`）之间存在某些概念上的相似之处。看起来，两者都可以捕获和触发错误。主要区别在于可抛出模型是一种更现代、面向对象的方式。也就是说，我们应该限制使用`trigger_error()`，只在绝对需要时才使用。

# 异常

异常最初是在 PHP 5 中引入的，它也带来了面向对象的模型。它们在整个时间内基本保持不变。PHP 5.5 添加了`finally`块，PHP 7 添加了使用`|`运算符以便通过单个`catch`块捕获多个异常类型的可能性，这是其中的重大变化。

`Exception`是 PHP 7 中所有用户异常的基类。与`Error`一样，`Exception`实现了`Throwable`接口，如下面的类概要所示：

```php
    Exception implements Throwable {
      /* Properties */
      protected string $message ;
      protected int $code ;
      protected string $file ;
      protected int $line ;

      /* Methods */
      public __construct (
        [ string $message = "" 
         [, int $code = 0 
          [, Throwable $previous = NULL ]]]
      )

      final public string getMessage (void)
      final public Throwable getPrevious (void)
      final public mixed getCode (void)
      final public string getFile (void)
      final public int getLine (void)
      final public array getTrace (void)
      final public string getTraceAsString (void)
      public string __toString (void)
      final private void __clone (void)
    }

```

异常仍然是面向对象错误处理的支柱。扩展、抛出和捕获异常的简单性使它们易于处理。

# 创建自定义异常处理程序

通过扩展内置的`Exception`类，PHP 让我们可以像抛出异常一样抛出任何对象。让我们看下面的例子：

```php
    <?php

    class UsernameException extends Exception {}

    class PasswordException extends Exception {}

    $username = 'john';
    $password = '';

    try {
      if (empty($username)) {
        throw new UsernameException();
      }
      if (empty($password)) {
        throw new PasswordException();
      }
      throw new Exception();
    } 
    catch (UsernameException $e) {
      echo 'Caught UsernameException.';
    } 
    catch (PasswordException $e) {
      echo 'Caught PasswordException.';
    } 
    catch (Exception $e) {
      echo 'Caught Exception.';
    } 
    finally {
      echo 'Finally.';
    }

```

在这里，我们定义了两个自定义异常，`UsernameException`和`PasswordException`。它们只是扩展了内置的`Exception`，并没有真正引入任何新的方法或功能。然后，我们定义了两个变量，`$username`和`$password`。`$password`变量被设置为空字符串。最后，我们设置了`try...catch...finally`块，其中包含三个不同的`catch`块。前两个`catch`块针对我们的自定义异常，第三个针对内置的`Exception`。由于密码为空，前面的例子将抛出`new PasswordException`，因此输出`Caught PasswordException. Finally.`字符串。

# 重新抛出异常

重新抛出异常在开发中是一种相对常见的做法。有时，我们希望捕获异常，查看一下，进行一些额外的逻辑，然后重新抛出异常，以便父`catch`块可以进一步处理它。

让我们看下面的例子：

```php
    <?php

    class FileNotExistException extends Exception {}

    class FileReadException extends Exception {}

    class FileEmptyException extends Exception {}

    $file = 'story.txt';

    try {
      try {
        $content = file_get_contents($file);
        if (!$content) {
          throw new Exception();
        }
      } 
      catch (Exception $e) {
        if (!file_exists($file)) {
          throw new FileNotExistException();
        } 
        elseif (!is_readable($file)) {
          throw new FileReadException();
        } 
        elseif (empty($content)) {
          throw new FileEmptyException();
        } 
        else {
          throw new Exception();
        }
      }
    }

    catch (FileNotExistException $e) {
      echo 'Caught FileNotExistException.';
    } 
    catch (FileReadException $e) {
      echo 'Caught FileReadException.';
    } 
    catch (FileEmptyException $e) {
      echo 'Caught FileEmptyException.';
    } 
    catch (Exception $e) {
      echo 'Caught Exception.';
    } 
    finally {
      echo 'Finally.';
    }

```

在这里，我们定义了三个简单的异常--`FileNotExistException`，`FileReadException`和`FileEmptyException`。这对应于我们在处理文件时可能遇到的三种不同的故障结果。然后，我们在`file_get_contents`函数调用周围添加了一些逻辑，尝试将其包装在`try...catch`块中。如果文件无法读取，`file_get_contents`函数的结果是布尔值`false`。知道这一点，并且知道`empty`函数调用在文件为空时结果为`false`，我们可以很容易地通过单个`if (!$content)`语句来检查文件是否正常。一旦抛出一般的`Exception`，就会有几种可能的情况。最明显的是缺少文件。令人惊讶的是，即使有`try...catch`块，如果文件丢失，PHP 也会输出以下内容：

```php
Warning: file_get_contents(story.txt): failed to open stream: No such file or directory in /index.php on line 13
Caught FileNotExistException.Finally.

```

我们可以清楚地看到，核心 PHP 语言引发了`Warning`，并触发了适当的`catch`和`finally`块。理想情况下，我们希望摆脱警告输出。一种可能的方法是使用错误控制运算符--at 符号（`@`）。它可以抑制错误和警告。这是非常危险的，应该非常小心使用。一般来说，错误和警告是触发处理的，而不是被抑制的。然而，在这种情况下，我们可能认为是合理的，因为我们将所有内容都包裹在`try...catch`块中。最后一个一般的`catch`块只是用来捕获未预料到的故障状态，这些状态不被我们的三个自定义异常所覆盖。

# 未捕获异常处理程序

PHP 提供了一种机制，即`set_exception_handler`函数，允许我们为所有未捕获的可抛出对象（包括异常）定义自定义处理程序函数。`set_exception_handler`函数接受一个可调用参数--可以是*作为字符串传递的函数名*，也可以是*整个匿名函数*。

让我们看下面的*作为字符串传递的函数名*示例：

```php
    <?php

    function throwableHandler(Throwable $t)
    {
      echo 'Throwable Handler: ' . $t->getMessage();
    }

    set_exception_handler('throwableHandler');

    echo 'start';
      throw new Exception('Ups!');
    echo 'end';

```

让我们看下面的*匿名函数*示例：

```php
    <?php

    set_exception_handler(function (Throwable $t) {
      echo 'Throwable Handler: ' . $t->getMessage();
    });

    echo 'start';
     throw new Exception('Ups!');
    echo 'end';

```

这两个代码示例做的事情是一样的，它们之间没有区别。除了第二个示例更美观外，因为不需要定义一个单独的函数，比如`throwableHandler()`，它只会在一个地方使用。这里需要注意的重要一点是，与`try...catch`块不同，对处理程序函数的调用是我们的应用程序执行的最后一件事情，这意味着在这种情况下，我们永远不会在屏幕上看到`end`字符串。

# 日志记录

日志记录是每个应用程序的重要方面。知道如何捕获错误并不一定意味着我们处理故障情况的方式是最好的。如果我们没有记录正确的细节，并将它们传递给正确的消费者，那么我们实际上并没有正确处理这种情况。

让我们考虑以下捕获和生成用户消息的示例：

```php
    try {
      //...
    } 
    catch (\Exception $e) {
      $messages[] = __('We can't add this item to your shopping cart right now.');
    }

```

让我们考虑以下示例：

```php
<?php try {
  //... } catch (\Exception $e) {
  $this->logger->critical($e);
  $messages[] = __("We can't add this item to your shopping cart right now . "); }

```

这两个示例都通过将消息存储到`$messages`变量中来响应异常，稍后将其显示给当前用户。这很好，因为应用程序不会崩溃，用户会看到发生了什么，并且应用程序被允许执行。但是，这真的很好吗？这两个示例几乎完全相同，除了一个细微的细节。第一个示例仅在错误发生时做出响应并立即做出反应。第二个示例使用`$this->logger->critical($e);`表达式来记录错误，可能是，但不一定是，记录到文件中。通过记录错误，我们使得消费者有可能稍后进行审查。消费者很可能是开发人员，他们可能会不时地查看日志文件。请注意，`$messages`数组并未直接传递给`$e`变量，而是适合用户情况的自定义消息。这是因为用户不应该看到我们可能传递给日志的详细级别。我们传递给日志的细节越多，就越容易排除应用程序的故障。通过记录整个异常实例对象，在这种情况下，我们基本上提供了开发人员需要了解的所有细节，以便尝试并防止将来的错误。

经过深思熟虑的使用，日志记录可以提供质量分析洞察，我们可以定期重复我们的代码库，并防止在初始开发过程中可能看不到的问题。除了记录错误，我们还可以轻松记录其他分析或其他重要的部分。

开源的 Elastic stack，可在[`www.elastic.co`](https://www.elastic.co)上获得，使我们能够可靠且安全地从任何来源以任何格式获取数据，并实时搜索、分析和可视化数据。Kibana 产品，可在[`www.elastic.co/products/kibana`](https://www.elastic.co/products/kibana)上获得，通过其交互式可视化为我们的数据赋予形状。

# 本地记录

PHP 具有内置的`error_log()`函数，它将错误消息发送到定义的错误处理程序；因此，为简单的记录提供了开箱即用的解决方案。

以下代码片段描述了`error_log()`函数的定义：

```php
    bool error_log ( 
       string $message 
      [, int $message_type = 0 
        [, string $destination 
          [, string $extra_headers ] ]] 
    )

```

参数定义如下：

+   `$message`：这是一个字符串类型的值，是我们想要记录的消息

+   `$message_type`：这是一个整数类型的值；它有四个可能的值，如下所示：

+   `0`：这是一个操作系统日志记录机制

+   `1`：这通过电子邮件发送到目标参数中的地址

+   `2`：这不再是一个选项

+   `3`：此消息附加到文件目的地

+   `4`：这直接发送到 SAPI 日志处理程序

+   `$destination`：这是一个字符串类型的值，仅在`$message_type = 1`时起作用，并表示电子邮件地址

+   `$extra_headers`：这是一个字符串类型的值，仅在`$message_type = 1`时起作用，并表示电子邮件头

`error_log()`函数与`php.ini`中定义的`log_errors`和`error_log`配置选项密切相关：

+   `log_errors`：这是一个布尔类型的配置选项。它告诉我们是否应该将错误消息记录到服务器错误日志或`error_log`。要记录到使用`error_log`配置选项指定的文件，请将其设置为`1`。

+   `error_log`：这是一个字符串类型的配置选项。它指定应将错误记录到的文件的名称。如果使用`syslog`，则将错误记录到系统记录器。如果未设置任何值，则将错误发送到 SAPI 错误记录器，这很可能是 Apache 中的错误日志或 CLI 中的 stderr。

以下示例演示了记录到文件中：

```php
    <?php

    ini_set('log_errors', 1);
    ini_set('error_log', dirname(__FILE__) . '/app-error.log');

    error_log('Test!');

```

`log_errors`和`error_log`选项可以在`.php`文件中定义；然而，建议在`php.ini`中这样做，否则，如果脚本有解析错误或根本无法运行，日志将不会记录任何错误。上面示例的结果输出将是一个`app-error.log`文件，位于执行脚本本身相同的目录中，内容如下：

```php
    [26-Dec-2016 08:11:32 UTC] Test!
    [26-Dec-2016 08:11:39 UTC] Test!
    [26-Dec-2016 08:11:42 UTC] Test!

```

以下示例演示了如何记录日志到电子邮件：

```php
    <?php

    ini_set('log_errors', 1);
    ini_set('error_log', dirname(__FILE__) . '/app-error.log');

    $headers = "From: john@server.loc\r\n";
    $headers .= "Subject: My PHP email logger\r\n";
    $headers .= "MIME-Version: 1.0\r\n";
    $headers .= "Content-Type: text/html; charset=ISO-8859-1\r\n";

    error_log('<html><h2>Test!</h2></html>', 1, 'john@mail.com', $headers);

```

在这里，我们首先构建原始的`$headers`字符串，然后将其传递给`error_log()`函数，以及目标电子邮件地址。这是`error_log()`函数的一个明显缺点，因为我们需要熟悉电子邮件消息头的标准。

`error_log()`函数不是二进制安全的，这意味着`$message`参数不应包含空字符，否则它将被截断。为了避开这个限制，我们可以在调用`error_log()`之前使用一个转换/转义函数，比如`base64_encode()`、`rawurlencode()`或`addslashes()`。以下 RFC 可能对处理电子邮件消息头很有用：RFC 1896、RFC 2045、RFC 2046、RFC 2047、RFC 2048、RFC 2049 和 RFC 2822。

了解`error_log()`函数后，我们可以很容易地将其封装成我们自己的自定义函数，比如`app_error_log()`，从而抽象出整个电子邮件的样板，比如地址和头部。我们还可以使我们的`app_error_log()`函数同时记录到文件和电子邮件，从而实现一个简单的、一行的日志记录表达式，比如下面的例子，可能在我们的应用程序中使用：

```php
    try {
      //...
    } 
    catch (\Exception $e) {
      app_error_log($e);
    }

```

编写这样简单的日志记录器非常容易。然而，开发中的简单通常伴随着降低模块化的成本。幸运的是，有一些第三方库在日志记录功能方面非常强大。最重要的是，它们符合某种日志记录标准，我们将在下一节中看到。

# 使用 Monolog 进行日志记录

PHP 社区为我们提供了几个日志记录库可供选择，比如 Monolog、Analog、KLogger、Log4PHP 等。选择合适的库可能是一项艰巨的任务。尤其是因为我们可能决定以后更改日志记录机制，这可能会导致我们需要改变大量的代码。这就是 PSR-3 日志记录标准的作用。选择一个符合标准的库可以更容易地进行推理。

Monolog 是最受欢迎的 PHP 日志记录库之一。它是一个免费的、MIT 许可的库，实现了 PSR-3 日志记录标准。它允许我们轻松地将日志发送到文件、套接字、收件箱、数据库和各种网络服务。

我们可以通过在项目文件夹中运行以下控制台命令轻松安装 Monolog 库作为`composer`包：

```php
composer require monolog/monolog

```

如果`composer`不是一个选择，我们可以从 GitHub 上下载 Monolog，网址为[`github.com/Seldaek/monolog`](https://github.com/Seldaek/monolog)。那些使用主要 PHP 框架，比如 Symfony 或 Laravel 的人，可以直接使用 Monolog。

符合 PSR-3 日志记录标准也意味着 Monolog 支持 RFC 5424 描述的日志级别，如下所示：

+   `DEBUG (100)`: 调试级别消息

+   `INFO (200)`: 信息消息

+   `NOTICE (250)`: 正常但重要的条件

+   `WARNING (300)`: 警告条件

+   `ERROR (400)`: 错误条件

+   `CRITICAL (500)`: 临界条件

+   `ALERT (550)`: 必须立即采取行动

+   `EMERGENCY (600)`: 系统不可用

这些常量定义在`vendor/monolog/monolog/src/Monolog/Logger.php`文件中，大部分都有一个实际的用例示例。

每个 Monolog 记录器实例的核心概念是实例本身具有一个通道（名称）和一组处理程序。我们可以实例化多个记录器，每个定义一个特定的通道（db，request，router 等）。每个通道可以组合各种处理程序。处理程序本身可以在通道之间共享。通道反映在日志中，并允许我们轻松查看或过滤记录。最后，每个处理程序还有一个格式化器。格式化器对传入的记录进行规范化和格式化，以便处理程序输出有用的信息。

以下图表展示了这个记录器-通道-格式化器的结构：

![](img/097c18e6-815f-427d-8fe3-96e011f9a995.png)

Monolog 提供了相当丰富的记录器和格式化器列表。

+   记录器：

+   记录到文件和系统日志（`StreamHandler`，`RotatingFileHandler`，`SyslogHandler`，...）

+   发送警报和电子邮件（`SwiftMailerHandler`，`SlackbotHandler`，`SendGridHandler`，...）

+   特定于日志的服务器和网络日志（`SocketHandler`，`CubeHandler`，`NewRelicHandler`，...）

+   开发中的日志记录（`FirePHPHandler`，`ChromePHPHandler`，`BrowserConsoleHandler`，...）

+   记录到数据库（`RedisHandler`，`MongoDBHandler`，`ElasticSearchHandler`，...）

+   格式化器：

+   `LineFormatter`

+   `HtmlFormatter`

+   `JsonFormatter`

+   ...

可以通过官方的 Monolog 项目页面获取完整的 Monolog 记录器和格式化器列表 [`github.com/Seldaek/monolog`](https://github.com/Seldaek/monolog)。

让我们看一个简单的例子：

```php
    <?php

    require 'vendor/autoload.php';

    use Monolog\Logger;
    use Monolog\Handler\RotatingFileHandler;
    use Monolog\Handler\BrowserConsoleHandler;

    $logger = new Logger('foggyline');

    $logger->pushHandler(new RotatingFileHandler(__DIR__ .  
      '/foggyline.log'), 7);
    $logger->pushHandler(new BrowserConsoleHandler());

    $context = [
      'user' => 'john',
      'salary' => 4500.00
    ];

    $logger->addDebug('Logging debug', $context);
    $logger->addInfo('Logging info', $context);
    $logger->addNotice('Logging notice', $context);
    $logger->addWarning('Logging warning', $context);
    $logger->addError('Logging error', $context);
    $logger->addCritical('Logging critical', $context);
    $logger->addAlert('Logging alert', $context);
    $logger->addEmergency('Logging emergency', $context);

```

在这里，我们创建了一个 `Logger` 实例，并将其命名为 `foggyline`。然后我们使用 `pushHandler` 方法推送内联实例化的两个不同处理程序的实例。

`RotatingFileHandler` 将记录日志到文件，并每天创建一个日志文件。它还会删除早于 `$maxFiles` 参数的文件，而在我们的示例中，该参数设置为 `7`。不管日志文件名是否设置为 `foggyline.log`，由 `RotatingFileHandler` 创建的实际日志文件中包含了时间戳，因此会得到一个名为 `foggyline-2016-12-26.log` 的文件。当我们考虑这一点时，这个处理程序的作用是非常显著的。除了创建新的日志条目之外，它还负责删除旧的日志。

以下是我们的 `foggyline-2016-12-26.log` 文件的输出：

```php
    [2016-12-26 12:36:46] foggyline.DEBUG: Logging debug {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.INFO: Logging info {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.NOTICE: Logging notice {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.WARNING: Logging warning {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.ERROR: Logging error {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.CRITICAL: Logging critical {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.ALERT: Logging alert {"user":"john","salary":4500} []
    [2016-12-26 12:36:46] foggyline.EMERGENCY: Logging emergency  {"user":"john","salary":4500} []

```

我们推送到堆栈的第二个处理程序 `BrowserConsoleHandler`，将日志发送到浏览器的 JavaScript 控制台，无需浏览器扩展。这适用于大多数支持控制台 API 的现代浏览器。该处理程序的输出如下截图所示：

![](img/464af28d-9107-4de6-9c85-dae25805202e.png)

通过这几行简单的代码，我们为我们的应用程序添加了相当令人印象深刻的日志功能。`RotatingFileHandler` 似乎非常适合用于生产运行应用程序的后续状态分析，而 `BrowserConsoleHandler` 可能作为加快持续开发的便捷方式。可以说，日志的作用远不止于记录错误。通过在各种日志级别记录各种信息，我们可以轻松地将 Monolog 库用作一种分析桥梁。只需将适当的处理程序推送到堆栈，然后将日志推送到各种目的地，例如 Elasticsearch 等。

# 总结

在本章中，我们详细研究了 PHP 的错误处理机制。PHP 7 通过将大部分错误处理模型包装在`Throwable`接口下，对其进行了相当大的清理。这使得可以通过`try...catch`块捕获核心错误，而在 PHP 7 之前，这些错误只能保留给`Exception`。现在，当我们遇到`Throwable`、`Error`、`Exception`、系统错误、用户错误、通知、警告等术语时，可能会有一些术语上的混淆需要消化。从高层次来说，我们可以说任何错误状态都是错误。更具体地说，现在我们有可抛出的错误，另一方面有错误。可抛出的错误包括`Error`和`Exception`的抛出和可捕获的实例，而错误基本上包括任何不可捕获为`Throwable`的东西。

处理错误状态如果没有适当的日志记录就不会真正完整。虽然内置的`error_log()`函数提供了足够的功能让我们开始，但更健壮的解决方案可以通过各种第三方库来实现。Monolog 库是最受欢迎的库之一，被用于数十个社区项目中。

在向前迈进时，我们将深入研究魔术方法及其为 PHP 语言带来的巨大力量。
