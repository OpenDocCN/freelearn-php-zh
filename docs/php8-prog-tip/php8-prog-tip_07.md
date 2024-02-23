# *第五章*：发现潜在的面向对象编程向后兼容性问题

本章标志着本书第 2 部分*PHP 8 技巧*的开始。在这一部分，您将发现 PHP 8 的黑暗角落：**向后兼容性问题**存在的地方。本部分将让您了解如何在将现有应用程序迁移到 PHP 8 之前避免问题。您将学会如何查找现有代码中可能导致其在 PHP 8 升级后停止工作的问题。一旦掌握了本书这一部分介绍的主题，您将能够很好地修改现有代码，使其在 PHP 8 升级后继续正常运行。

在本章中，您将介绍与**面向对象编程（OOP）**相关的新的 PHP 8 特性。本章提供了大量清晰说明新特性和概念的简短代码示例。本章对帮助您快速利用 PHP 8 的强大功能至关重要，因为您可以将代码示例调整为自己的实践。本章的重点是在 PHP 8 迁移后，面向对象的代码可能会出现问题的情况。

本章涵盖的主题包括以下内容：

+   发现核心面向对象编程的差异

+   导航魔术方法的更改

+   控制序列化

+   理解扩展的 PHP 8 变异支持

+   处理**标准 PHP 库**（**SPL**）的更改

# 技术要求

要查看和运行本章提供的代码示例，建议的最低硬件要求如下：

+   基于 x86_64 的台式 PC 或笔记本电脑

+   1 千兆字节(GB)的可用磁盘空间

+   4GB 的 RAM

+   每秒 500 千位(Kbps)或更快的互联网连接

此外，您需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 的安装以及如何构建用于演示本书中解释的代码的 Docker 容器的更多信息，请参阅*第一章*的*技术要求*部分，*介绍新的 PHP 8 面向对象编程特性*。在本书中，我们将您为本书恢复的示例代码的目录称为`/repo`。

本章的源代码位于此处：https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices。

我们现在可以开始讨论核心面向对象编程的差异。

# 发现核心面向对象编程的差异

在 PHP 8 中，您可以以不同的方式编写面向对象的代码。在本节中，我们重点关注可能会导致潜在向后兼容性问题的三个关键领域。本节我们将讨论与进行静态方法调用、处理对象属性和 PHP 自动加载相关的常见不良实践。

阅读本节并完成示例后，您将更好地发现面向对象的不良实践，并了解 PHP 8 如何对此类用法进行限制。在本章中，您将学习良好的编码实践，这将最终使您成为更好的程序员。您还将能够解决 PHP 自动加载中的更改，这可能会导致迁移到 PHP 8 的应用程序失败。

让我们首先看看 PHP 8 如何加强静态调用。

## 在 PHP 8 中处理静态调用

令人惊讶的是，PHP 7 及以下版本允许开发人员对未声明为`static`的类方法进行静态调用。乍一看，任何未来审查您代码的开发人员立即会假设该方法已被定义为`static`。这可能会导致意外行为，因为未来的开发人员在错误的假设下开始误用您的代码。

在这个简单的例子中，我们定义了一个带有`nonStatic()`方法的`Test`类。在类定义后的程序代码中，我们输出了这个方法的返回值，然而，在这样做时，我们进行了一个静态调用：

```php
// /repo/ch05/php8_oop_diff_static.php
class Test {
    public function notStatic() {
        return __CLASS__ . PHP_EOL;
    }
}
echo Test::notStatic();
```

当我们在 PHP 7 中运行此代码时，结果如下：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_oop_diff_static.php
PHP Deprecated:  Non-static method Test::notStatic() should not be called statically in /repo/ch05/php8_oop_diff_static.php on line 11
Test
```

从输出中可以看出，PHP 7 会发出弃用通知，但允许调用！然而，在 PHP 8 中，结果是致命的`Error`，如下所示：

```php
root@php8_tips_php8 [ /repo/ch05 ]#
php php8_oop_diff_static.php
PHP Fatal error:  Uncaught Error: Non-static method Test::notStatic() cannot be called statically in /repo/ch05/php8_oop_diff_static.php:11
```

使用静态方法调用非静态方法的语法是一种不良实践，因为良好编写的代码使代码开发人员的意图变得清晰明了。如果您没有将方法定义为静态，但后来以静态方式调用它，未来负责维护您代码的开发人员可能会感到困惑，并可能对代码的原始意图做出错误的假设。最终结果将是更糟糕的代码！

在 PHP 8 中，您不能再使用静态方法调用非静态方法。现在让我们再看看另一个涉及将对象属性视为键的不良实践。

## 处理对象属性处理的变化

数组一直是 PHP 的一个核心特性，一直延续到最早的版本。另一方面，面向对象编程直到 PHP 4 才被引入。在面向对象编程的早期，数组函数经常被扩展以适应对象属性。这导致对象和数组之间的区别变得模糊，从而产生了一些不良实践。

为了保持数组处理和对象处理之间的清晰分离，PHP 8 现在限制`array_key_exists()`函数只接受数组作为参数。为了说明这一点，考虑以下示例：

1.  首先，我们定义一个带有单个属性的简单匿名类：

```php
// /repo/ch05/php8_oop_diff_array_key_exists.php
$obj = new class () { public $var = 'OK.'; };
```

1.  然后我们运行三个测试，分别使用`isset()`、`property_exists()`和`array_key_exists()`来检查`$var`的存在：

```php
// not all code is shown
$default = 'DEFAULT';
echo (isset($obj->var)) 
    ? $obj->var : $default;
echo (property_exists($obj,'var')) 
    ? $obj->var : $default;
echo (array_key_exists('var',$obj)) 
    ? $obj->var : $default;
```

当我们在 PHP 7 中运行这段代码时，所有测试都成功，如下所示：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_oop_diff_array_key_exists.php
OK.OK.OK.
```

然而，在 PHP 8 中，会发生致命的`TypeError`，因为`array_key_exists()`现在只接受数组作为参数。PHP 8 的输出如下所示：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php8_oop_diff_array_key_exists.php
OK.OK.PHP Fatal error:  Uncaught TypeError: array_key_exists(): Argument #2 ($array) must be of type array, class@anonymous given in /repo/ch05/php8_oop_diff_array_key_exists.php:10
```

最佳实践是使用`property_exists()`或`isset()`。现在我们将注意力转向 PHP 自动加载的变化。

## 使用 PHP 8 自动加载

在 PHP 8 中首次引入的基本**自动加载**类机制与 PHP 8 中的工作方式相同。主要区别在于，全局函数`__autoload()`在 PHP 7.2 中已弃用，并在 PHP 8 中已完全删除。从 PHP 7.2 开始，开发人员被鼓励使用`spl_autoload_register()`注册其自动加载逻辑，该函数自 PHP 5.1 起可用于此目的。另一个主要区别是如果无法注册自动加载程序，`spl_autoload_register()`的反应方式。

了解使用`spl_autoload_register()`时自动加载过程的工作原理对于作为开发人员的工作至关重要。不理解 PHP 如何自动定位和加载类将限制您作为开发人员的能力，并可能对您的职业道路产生不利影响。

在深入研究`spl_autoload_register()`之前，让我们先看一下`__autoload()`函数。

### 理解 __autoload()函数

`__autoload()`函数被许多开发人员用作自动加载逻辑的主要来源。这个函数的行为方式类似于*魔术方法*，这就是为什么它根据上下文自动调用。会触发自动调用`__autoload()`函数的情况包括创建新类实例时，但类定义尚未加载的时刻。此外，如果类扩展另一个类，则还会调用自动加载逻辑，以便在创建扩展它的子类之前加载超类。

使用`__autoload()`函数的优点是它非常容易定义，并且通常在网站的初始`index.php`文件中定义。缺点包括以下内容：

+   `__autoload()`是一个 PHP 过程函数；不是使用面向对象编程原则定义或控制的。例如，在为应用程序定义单元测试时，这可能会成为一个问题。

+   如果你的应用程序使用命名空间，`__autoload()`函数必须在全局命名空间中定义；否则，在定义`__autoload()`函数的命名空间之外的类将无法加载。

+   `__autoload()`函数与`spl_autoload_register()`不兼容。如果你同时使用`__autoload()`函数和`spl_autoload_register()`定义自动加载逻辑，`__autoload()`函数的逻辑将被完全忽略。

为了说明潜在的问题，我们将定义一个`OopBreakScan`类，更详细地讨论在*第十一章**，将现有的 PHP 应用迁移到 PHP 8*中：

1.  首先，我们定义并添加一个方法到`OopBreakScan`类中，该方法扫描文件内容以查找`__autoload()`函数。请注意，错误消息是在`Base`类中定义的一个类常量，只是警告存在`__autoload()`函数：

```php
namespace Migration;
class OopBreakScan extends Base {
    public static function scanMagicAutoloadFunction(
        string $contents, array &$message) : bool {
        $found  = 0;
        $found += (stripos($contents, 
            'function __autoload(') !== FALSE);
        $message[] = ($found)
                   ? Base::ERR_MAGIC_AUTOLOAD
                   : sprintf(Base::OK_PASSED,
                       __FUNCTION__);
        return (bool) $found;
    }
    // remaining methods not shown
```

这个类扩展了一个`Migration\Base`类（未显示）。这很重要，因为任何自动加载逻辑都需要找到子类和它的超类。

1.  接下来，我们定义一个调用程序，在其中定义了一个魔术`__autoload()`函数：

```php
// /repo/ch05/php7_autoload_function.php
function __autoLoad($class) {
    $fn = __DIR__ . '/../src/'
        . str_replace('\\', '/', $class)
        . '.php';
    require_once $fn;
}
```

1.  然后我们通过让调用程序扫描自身来使用这个类：

```php
use Migration\OopBreakScan;
$contents = file_get_contents(__FILE__);
$message  = [];
OopBreakScan::
    scanMagicAutoloadFunction($contents, $message);
var_dump($message);
```

以下是在 PHP 7 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php7_autoload_function.php
/repo/ch05/php7_autoload_function.php:23:
array(1) {
  [0] =>  string(96) "WARNING: the "__autoload()" function is removed in PHP 8: replace with "spl_autoload_register()""
}
```

从输出中可以看到，`Migration\OopBreakScan`类被自动加载了。我们知道这是因为`scanMagicAutoloadFunction`方法被调用了，我们有它的结果。此外，我们知道`Migration\Base`类也被自动加载了。我们知道这是因为输出中出现的错误消息是超类的常量。

然而，在 PHP 8 中运行相同的代码会产生这样的结果：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php7_autoload_function.php 
PHP Fatal error:  __autoload() is no longer supported, use spl_autoload_register() instead in /repo/ch05/php7_autoload_function.php on line 4
```

这个结果并不奇怪，因为在 PHP 8 中移除了对魔术`__autoload()`函数的支持。在 PHP 8 中，你必须使用`spl_autoload_register()`。现在我们转向`spl_autoload_register()`。

### 学习使用 spl_autoload_register()

`spl_autoload_register()`函数的主要优点是它允许你注册多个自动加载器。虽然这可能看起来有些多余，但想象一下一个噩梦般的情景，你正在使用许多不同的开源 PHP 库...它们都定义了自己的*自动加载器*！只要所有这些库都使用`spl_autoload_register()`，拥有多个自动加载器回调就不会有问题。

使用`spl_autoload_register()`注册的每个自动加载器都必须是可调用的。以下任何一种都被认为是`可调用`：

+   一个 PHP 过程函数

+   一个匿名函数

+   一个可以以静态方式调用的类方法

+   定义了`__invoke()`魔术方法的任何类实例

+   一个这样的数组：`[$instance, 'method']`

提示

*Composer*维护着自己的自动加载器，它又依赖于`spl_autoload_register()`。如果你正在使用 Composer 来管理你的开源 PHP 包，你可以简单地在应用程序代码的开头包含`/path/to/project/vendor/autoload.php`来使用 Composer 的自动加载器。要让 Composer 自动加载你的应用程序源代码文件，可以在`composer.json`文件的`autoload : psr-4`键下添加一个或多个条目。更多信息，请参见[`getcomposer.org/doc/04-schema.md#psr-4`](https://getcomposer.org/doc/04-schema.md#psr-4)。

一个相当典型的自动加载器类可能如下所示。请注意，这是我们在本书中许多 OOP 示例中使用的类：

1.  在`__construct()`方法中，我们分配了源目录。随后，我们使用上面提到的数组可调用语法调用`spl_auto_register()`：

```php
// /repo/src/Server/Autoload/Loader.php
namespace Server\Autoload;
class Loader {
    const DEFAULT_SRC = __DIR__ . '/../..';
    public $src_dir = '';
    public function __construct($src_dir = NULL) {
        $this->src_dir = $src_dir 
            ?? realpath(self::DEFAULT_SRC);
        spl_autoload_register([$this, 'autoload']);
    }
```

1.  实际的自动加载代码与我们上面`__autoload()`函数示例中显示的类似。以下是执行实际自动加载的方法：

```php
    public function autoload($class) {
        $fn = str_replace('\\', '/', $class);
        $fn = $this->src_dir . '/' . $fn . '.php';
        $fn = str_replace('//', '/', $fn);
        require_once($fn);
    }
}
```

现在你已经了解了如何使用`spl_auto_register()`函数，我们必须检查在运行 PHP 8 时可能出现的代码中断。

### PHP 8 中潜在的 spl_auto_register()代码中断

`spl_auto_register()`函数的第二个参数是一个可选的布尔值，默认为`FALSE`。如果将第二个参数设置为`TRUE`，则在 PHP 7 及以下版本中，如果自动加载程序注册失败，`spl_auto_register()`函数会抛出一个`Exception`。然而，在 PHP 8 中，如果第二个参数的数据类型不是`callable`，则无论第二个参数的值如何，都会抛出致命的`TypeError`！

下面显示的简单程序示例说明了这种危险。在这个例子中，我们使用`spl_auto_register()`函数注册一个不存在的 PHP 函数。我们将第二个参数设置为`TRUE`：

```php
// /repo/ch05/php7_spl_spl_autoload_register.php
try {
    spl_autoload_register('does_not_exist', TRUE);
    $data = ['A' => [1,2,3],'B' => [4,5,6],'C' => [7,8,9]];
    $response = new \Application\Strategy\JsonResponse($data);
    echo $response->render();
} catch (Exception $e) {
    echo "A program error has occurred\n";
}
```

如果我们在 PHP 7 中运行这个代码块，这是结果：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php7_spl_spl_autoload_register.php 
A program error has occurred
```

从输出中可以确定，抛出了一个`Exception`。`catch`块被调用，出现了消息**发生了程序错误**。然而，当我们在 PHP 8 中运行相同的程序时，会抛出一个致命的`Error`：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php7_spl_spl_autoload_register.php 
PHP Fatal error:  Uncaught TypeError: spl_autoload_register(): Argument #1 ($callback) must be a valid callback, no array or string given in /repo/ch05/php7_spl_spl_autoload_register.php:12
```

显然，`catch`块被绕过，因为它设计用于捕获`Exception`，而不是`Error`。简单的解决方法是让`catch`块捕获`Throwable`而不是`Exception`。这允许相同的代码在 PHP 7 或 PHP 8 中运行。

重写后的代码可能如下所示。输出没有显示，因为它与在 PHP 7 中运行相同的示例相同：

```php
// /repo/ch05/php8_spl_spl_autoload_register.php
try {
    spl_autoload_register('does_not_exist', TRUE);
    $data = ['A' => [1,2,3],'B' => [4,5,6],'C' => [7,8,9]];
    $response = new \Application\Strategy\JsonResponse($data);
    echo $response->render();
} catch (Throwable $e) {
    echo "A program error has occurred\n";
}
```

现在您对 PHP 8 自动加载有了更好的理解，以及如何发现和纠正潜在的自动加载向后兼容性问题。现在让我们来看看 PHP 8 中与魔术方法相关的变化。

# 导航魔术方法的变化

PHP 的**魔术方法**是预定义的钩子，它们中断了 OOP 应用程序的正常流程。每个魔术方法，如果定义了，都会改变应用程序的行为，从对象实例创建的那一刻开始，直到实例超出范围的那一刻。

重要提示

对象实例在被取消或被覆盖时会*超出范围*。当在函数或类方法中定义对象实例时，对象实例也会超出范围，并且该函数或类方法的执行结束。最终，如果没有其他原因，当 PHP 程序结束时，对象实例会超出范围。

本节将让您充分了解 PHP 8 中引入的魔术方法使用和行为的重要变化。一旦您了解了本节描述的情况，您就能够进行适当的代码修改，以防止您的应用程序代码在迁移到 PHP 8 时失败。

让我们首先看一下对象构造方法的变化。

## 处理构造函数的变化

理想情况下，**类构造函数**是一个在对象实例创建时自动调用的方法，用于执行某种对象初始化。这种初始化通常涉及使用作为参数提供给该方法的值填充对象属性。初始化还可以执行任何必要的任务，如打开文件句柄、建立数据库连接等。

在 PHP 8 中，类构造函数被调用的方式发生了一些变化。这意味着当您将应用程序迁移到 PHP 8 时，可能会出现向后兼容性问题。我们将首先检查的变化与使用与类相同名称的方法作为类构造函数的方法有关。

### 处理具有相同名称的方法和类的变化

在 PHP 4 版本中引入的第一个 PHP OOP 实现中，确定了与类相同名称的方法将承担类构造函数的角色，并且在创建新对象实例时将自动调用该方法。

鲜为人知的是，即使在 PHP 8 中，函数、方法甚至类名都是*不区分大小写*的。因此`$a = new ArrayObject();`等同于`$b = new arrayobject();`。另一方面，变量名是区分大小写的。

从 PHP 5 开始，随着一个新的更加健壮的 OOP 实现，魔术方法被引入。其中之一是`__construct()`，专门用于类构造，旨在取代旧的用法。通过 PHP 5 的剩余版本，一直到所有的 PHP 7 版本，都支持使用与类同名的方法作为构造函数。

在 PHP 8 中，删除了与类本身相同名称的类构造方法的支持。如果也定义了`__construct()`方法，你就不会有问题：`__construct()`优先作为类构造函数。如果没有`__construct()`方法，并且检测到一个与`class` `()`相同名称的方法，你就有可能失败。请记住，方法和类名都是不区分大小写的！

看一下以下的例子。它在 PHP 7 中有效，但在 PHP 8 中无效：

1.  首先，我们定义了一个`Text`类，它有一个同名的类构造方法。构造方法基于提供的文件名创建了一个`SplFileObject`实例：

```php
// /repo/ch05/php8_oop_bc_break_construct.php
class Text {
    public $fh = '';
    public const ERROR_FN = 'ERROR: file not found';
    public function text(string $fn) {
        if (!file_exists($fn))
            throw new Exception(self::ERROR_FN);
        $this->fh = new SplFileObject($fn, 'r');
    }
    public function getText() {
        return $this->fh->fpassthru();
    }
}
```

1.  然后我们添加了三行过程代码来使用这个类，提供一个包含葛底斯堡演说的文件的文件名：

```php
$fn   = __DIR__ . '/../sample_data/gettysburg.txt';
$text = new Text($fn);
echo $text->getText();
```

1.  首先在 PHP 7 中运行程序会产生一个弃用通知，然后是预期的文本。这里只显示了输出的前几行：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_bc_break_construct.php
PHP Deprecated:  Methods with the same name as their class will not be constructors in a future version of PHP; Text has a deprecated constructor in /repo/ch05/php8_bc_break_construct.php on line 4
Fourscore and seven years ago our fathers brought forth on this continent a new nation, conceived in liberty and dedicated to the proposition that all men are created equal. ... <remaining text not shown>
```

1.  然而，在 PHP 8 中运行相同的程序会抛出一个致命的`Error`，如你从这个输出中看到的：

```php
root@php8_tips_php8 [ /repo/ch05 ]# php php8_bc_break_construct.php 
PHP Fatal error:  Uncaught Error: Call to a member function fpassthru() on string in /repo/ch05/php8_bc_break_construct.php:16
```

重要的是要注意，在 PHP 8 中显示的错误并没有告诉你程序失败的真正原因。因此，非常重要的是要扫描你的 PHP 应用程序，特别是旧的应用程序，看看是否有一个与类同名的方法。因此，**最佳实践**就是简单地将与类同名的方法重命名为`__construct()`。

现在让我们看看在 PHP 8 中如何解决类构造函数中处理`Exception`和`exit`的不一致性。

### 解决类构造函数中的不一致性

PHP 8 中解决的另一个问题与类构造方法中抛出`Exception`或执行`exit()`有关。在 PHP 8 之前的版本中，如果在类构造函数中抛出`Exception`，则*不会调用*`__destruct()`方法（如果定义了）。另一方面，如果在构造函数中使用`exit()`或`die()`（这两个 PHP 函数是等效的），则会调用`__destruct()`方法。在 PHP 8 中，这种不一致性得到了解决。现在，在任何情况下，`__destruct()`方法都*不会*被调用。

你可能想知道为什么这很重要。你需要意识到这个重要的改变的原因是，你可能有逻辑存在于`__destruct()`方法中，而这个方法在你可能调用`exit()`或`die()`的情况下被调用。在 PHP 8 中，你不能再依赖这段代码，这可能导致向后兼容性的破坏。

在这个例子中，我们有两个连接类。`ConnectPdo`使用 PDO 扩展来提供查询结果，而`ConnectMysqli`使用 MySQLi 扩展：

1.  我们首先定义一个接口，指定一个查询方法。这个方法需要一个 SQL 字符串作为参数，并且期望返回一个数组作为结果：

```php
// /repo/src/Php7/Connector/ConnectInterface.php
namespace Php7\Connector;
interface ConnectInterface {
    public function query(string $sql) : array;
}
```

1.  接下来，我们定义一个基类，其中定义了一个`__destruct()`魔术方法。因为这个类实现了`ConnectInterface`但没有定义`query()`，所以它被标记为`abstract`：

```php
// /repo/src/Php7/Connector/Base.php
namespace Php7\Connector;
abstract class Base implements ConnectInterface {
    const CONN_TERMINATED = 'Connection Terminated';
    public $conn = NULL;
    public function __destruct() {
        $message = get_class($this)
                 . ':' . self::CONN_TERMINATED;
        error_log($message);
    }
}
```

1.  接下来，我们定义`ConnectPdo`类。它继承自`Base`，它的`query()`方法使用`PDO`语法来产生结果。`__construct()`方法如果创建连接时出现问题，则抛出`PDOException`：

```php
// /repo/src/Php7/Connector/ConnectPdo.php
namespace Php7\Connector;
use PDO;
class ConnectPdo extends Base {
    public function __construct(
        string $dsn, string $usr, string $pwd) {
        $this->conn = new PDO($dsn, $usr, $pwd);
    }
    public function query(string $sql) : array {
        $stmt = $this->conn->query($sql);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

1.  以类似的方式，我们定义了`ConnectMysqli`类。它继承自`Base`，它的`query()`方法使用`MySQLi`语法来产生结果。`__construct()`方法如果创建连接时出现问题，则执行`die()`：

```php
// /repo/src/Php7/Connector/ConnectMysqli.php
namespace Php7\Connector;
class ConnectMysqli extends Base {
    public function __construct(
        string $db, string $usr, string $pwd) {
        $this->conn = mysqli_connect('localhost', 
            $usr, $pwd, $db) 
            or die("Unable to Connect\n");
    }
    public function query(string $sql) : array {
        $result = mysqli_query($this->conn, $sql);
        return mysqli_fetch_all($result, MYSQLI_ASSOC);
    }
}
```

1.  最后，我们定义一个调用程序，使用先前描述的两个连接类，并为连接字符串、用户名和密码定义无效值：

```php
// /repo/ch05/php8_bc_break_destruct.php
include __DIR__ . '/../vendor/autoload.php';
use Php7\Connector\ {ConnectPdo,ConnectMysqli};
$db  = 'test';
$usr = 'fake';
$pwd = 'xyz';
$dsn = 'mysql:host=localhost;dbname=' . $db;
$sql = 'SELECT event_name, event_date FROM events';
```

1.  接下来，在调用程序中，我们调用两个类，并尝试执行查询。连接故意失败，因为我们提供了错误的用户名和密码：

```php
$ptn = "%2d : %s : %s\n";
try {
    $conn = new ConnectPdo($dsn, $usr, $pwd);
    var_dump($conn->query($sql));
} catch (Throwable $t) {
    printf($ptn, __LINE__, get_class($t), 
           $t->getMessage());
}
$conn = new ConnectMysqli($db, $usr, $pwd);
var_dump($conn->query($sql));
```

1.  正如您从上面的讨论中所了解的，PHP 7 中运行的输出显示了在创建`ConnectPdo`实例时从类构造函数抛出`PDOException`。另一方面，当`ConnectMysqli`实例失败时，将调用`die()`，并显示消息**无法连接**。您还可以在输出的最后一行看到来自`__destruct()`方法的错误日志信息。以下是该输出：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_bc_break_destruct.php 
15 : PDOException : SQLSTATE[28000] [1045] Access denied for user 'fake'@'localhost' (using password: YES)
PHP Warning:  mysqli_connect(): (HY000/1045): Access denied for user 'fake'@'localhost' (using password: YES) in /repo/src/Php7/Connector/ConnectMysqli.php on line 8
Unable to Connect
Php7\Connector\ConnectMysqli:Connection Terminated
```

1.  在 PHP 8 中，`__destruct()`方法在任何情况下都不会被调用，导致如下所示的输出。正如您在输出中所看到的，`PDOException`被捕获，然后发出`die()`命令。`__destruct()`方法没有任何输出。PHP 8 的输出如下：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php8_bc_break_destruct.php 
15 : PDOException : SQLSTATE[28000] [1045] Access denied for user 'fake'@'localhost' (using password: YES)
PHP Warning:  mysqli_connect(): (HY000/1045): Access denied for user 'fake'@'localhost' (using password: YES) in /repo/src/Php7/Connector/ConnectMysqli.php on line 8
Unable to Connect
```

现在您已经知道如何发现与`__destruct()`方法以及对`die()`或`exit()`的调用有关的潜在代码中断，让我们将注意力转向`__toString()`方法的更改。

## 处理对 __toString()的更改

当对象被用作字符串时，将调用`__toString()`魔术方法。一个经典的例子是当您简单地 echo 一个对象时。`echo`命令期望一个字符串作为参数。当提供非字符串数据时，PHP 执行类型转换将数据转换为`string`。由于对象不能直接转换为`string`，因此 PHP 引擎会查看是否定义了`__toString()`，如果定义了，则返回其值。

这个魔术方法的主要变化是引入了`Stringable`，一个全新的接口。新接口定义如下：

```php
interface Stringable {
   public function __toString(): string;
}
```

在 PHP 8 中运行的任何类，如果定义了`__toString()`魔术方法，都会静默实现`Stringable`接口。这种新行为并不会导致严重的潜在代码中断。然而，由于类现在实现了`Stringable`接口，您将不再允许修改`__toString()`方法的签名。

以下是一个简短的示例，揭示了与`Stringable`接口的新关联：

1.  在这个例子中，我们定义了一个定义了`__toString()`的`Test`类：

```php
// /repo/ch05/php8_bc_break_magic_to_string.php
class Test {
    public $fname = 'Fred';
    public $lname = 'Flintstone';
    public function __toString() : string {
        return $this->fname . ' ' . $this->lname;
    }
}
```

1.  然后我们创建类的一个实例，然后是一个`ReflectionObject`实例：

```php
$test = new Test;
$reflect = new ReflectionObject($test);
echo $reflect;
```

在 PHP 7 中运行的输出的前几行（如下所示）只是显示它是`Test`类的一个实例：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_bc_break_magic_to_string.php
Object of class [ <user> class Test ] {
  @@ /repo/ch05/php8_bc_break_magic_to_string.php 3-12
```

然而，在 PHP 8 中运行相同的代码示例，揭示了与`Stringable`接口的静默关联：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php8_bc_break_magic_to_string.php
Object of class [ <user> class Test implements Stringable ] {
  @@ /repo/ch05/php8_bc_break_magic_to_string.php 3-12
```

输出显示，即使您没有显式实现`Stringable`接口，也会在运行时创建关联，并由`ReflectionObject`实例显示。

提示

有关魔术方法的更多信息，请参阅此文档页面：[`www.php.net/manual/en/language.oop5.magic.php`](https://www.php.net/manual/en/language.oop5.magic.php)。

现在您已经了解了 PHP 8 代码涉及魔术方法可能导致代码中断的情况，让我们来看看序列化过程中的更改。

# 控制序列化

有许多时候，需要将本机 PHP 数据存储在文件中，或者存储在数据库表中。当前技术的问题在于，直接存储复杂的 PHP 数据，如对象或数组，是不可能的，除了一些例外。

克服这种限制的一种方法是将对象或数组转换为字符串。**JSON**（JavaScript 对象表示）通常因此而被选择。一旦数据被转换为字符串，它就可以轻松地存储在任何文件或数据库中。然而，使用 JSON 格式化对象存在问题。尽管 JSON 能够很好地表示对象属性，但它无法直接恢复原始对象的类和方法。

为了解决这个缺陷，PHP 语言包括两个原生函数`serialize()`和`unserialize()`，可以轻松地将对象或数组转换为字符串，并将它们恢复到原始状态。尽管听起来很棒，但与原生 PHP 序列化相关的问题有很多。

在我们能够正确讨论现有 PHP 序列化架构的问题之前，我们需要更仔细地了解原生 PHP 序列化的工作方式。

## 了解 PHP 序列化

当 PHP 对象或数组需要保存到非面向对象编程环境（如平面文件或关系数据库表）时，可以使用`serialize()`将对象或数组“扁平化”为适合存储的字符串。相反，`unserialize()`会恢复原始对象或数组。

以下是演示这个概念的一个简单示例：

1.  首先，我们定义一个具有三个属性的类：

```php
// /repo/ch05/php8_serialization.php
class Test  {
    public $name = 'Doug';
    private $key = 12345;
    protected $status = ['A','B','C'];
}
```

1.  然后我们创建一个实例，对该实例进行序列化，并显示生成的字符串：

```php
$test = new Test();
$str = serialize($test);
echo $str . "\n";
```

1.  以下是序列化对象的样子：

```php
O:4:"Test":3:{s:4:"name";s:4:"Doug";s:9:"Testkey"; i:12345;
s:9:"*status";a:3:{i:0;s:1:"A";i:1;s:1:"B";i:2;s:1:"C";}}
```

从序列化字符串中可以看出，字母`O`代表*对象*，`a`代表*数组*，`s`代表*字符串*，`i`代表*整数*。

1.  然后我们将对象反序列化为一个新变量，并使用`var_dump()`来检查这两个变量：

```php
$obj = unserialize($str);
var_dump($test, $obj);
```

1.  将`var_dump()`的输出并排放置，您可以清楚地看到恢复的对象与原始对象是相同的：

![](img/B16992_05_table_1.1.jpg)

现在让我们来看一下支持旧版 PHP 序列化的魔术方法：`__sleep()`和`__wakeup()`。

## 了解`__sleep()`魔术方法

`__sleep()`魔术方法的目的是提供一个过滤器，用于防止某些属性出现在序列化字符串中。以用户对象为例，您可能希望排除敏感属性，如国民身份证号码、信用卡号码或密码。

以下是使用`__sleep()`魔术方法来排除密码的示例：

1.  首先，我们定义一个具有三个属性的`Test`类：

```php
// /repo/ch05/php8_serialization_sleep.php
class Test  {
    public $name = 'Doug';
    protected $key = 12345;
    protected $password = '$2y$10$ux07vQNSA0ctbzZcZNA'
         . 'lxOa8hi6kchJrJZzqWcxpw/XQUjSNqacx.';
```

1.  然后我们定义一个`__sleep()`方法来排除`$password`属性：

```php
    public function __sleep() {
        return ['name','key'];
    }
}
```

1.  然后我们创建这个类的一个实例并对其进行序列化。最后一行输出序列化字符串的状态：

```php
$test = new Test();
$str = serialize($test)
echo $str . "\n";
```

1.  在输出中，您可以清楚地看到`$password`属性不存在。以下是输出：

```php
O:4:"Test":2:{s:4:"name";s:4:"Doug";s:6:"*key";i:12345;}
```

这一点很重要，因为在大多数情况下，您需要对对象进行序列化的原因是希望将其存储在某个地方，无论是在会话文件中还是在数据库中。如果文件系统或数据库随后受到损害，您就少了一个安全漏洞需要担心！

## 了解`__sleep()`方法中潜在的代码中断

`__sleep()`魔术方法涉及潜在的代码中断。在 PHP 8 之前的版本中，如果`__sleep()`返回一个包含不存在属性的数组，它们仍然会被序列化并赋予一个`NULL`值。这种方法的问题在于，当对象随后被反序列化时，会出现一个额外的属性，这不是设计时存在的属性！

在 PHP 8 中，`__sleep()`魔术方法中不存在的属性会被静默忽略。如果您的旧代码预期旧的行为并采取步骤*删除*不需要的属性，或者更糟糕的是，如果您的代码假设不需要的属性存在，最终会出现错误。这样的假设非常危险，因为它们可能导致意外的代码行为。

为了说明问题，让我们看一下以下代码示例：

1.  首先，我们定义一个`Test`类，该类定义了`__sleep()`来返回一个不存在的变量：

```php
class Test {
    public $name = 'Doug';
    public function __sleep() {
        return ['name', 'missing'];
    }
}
```

1.  接下来，我们创建一个`Test`的实例并对其进行序列化：

```php
echo "Test instance before serialization:\n";
$test = new Test();
var_dump($test);
```

1.  然后我们将字符串反序列化为一个新实例`$restored`：

```php
echo "Test instance after serialization:\n";
$stored = serialize($test);
$restored = unserialize($stored);
var_dump($restored);
```

1.  理论上，两个对象实例`$test`和`$restored`应该是相同的。然而，看一下在 PHP 7 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch05 ]#
php php8_bc_break_sleep.php
Test instance before serialization:
/repo/ch05/php8_bc_break_sleep.php:13:
class Test#1 (1) {
  public $name =>  string(4) "Doug"
}
Test instance after serialization:
PHP Notice:  serialize(): "missing" returned as member variable from __sleep() but does not exist in /repo/ch05/php8_bc_break_sleep.php on line 16
class Test#2 (2) {
  public $name =>  string(4) "Doug"
  public $missing =>  NULL
}
```

1.  从输出中可以看出，这两个对象显然*不*相同！然而，在 PHP 8 中，不存在的属性被忽略。看一下在 PHP 8 中运行相同脚本的情况：

```php
root@php8_tips_php8 [ /repo/ch05 ]# php php8_bc_break_sleep.php 
Test instance before serialization:
object(Test)#1 (1) {
  ["name"]=>  string(4) "Doug"
}
Test instance after serialization:
PHP Warning:  serialize(): "missing" returned as member variable from __sleep() but does not exist in /repo/ch05/php8_bc_break_sleep.php on line 16
object(Test)#2 (1) {
  ["name"]=>  string(4) "Doug"
}
```

您可能还注意到，在 PHP 7 中，会发出一个`Notice`，而在 PHP 8 中，相同的情况会产生一个`Warning`。在这种情况下，对潜在代码中断的预迁移检查是困难的，因为您需要确定魔术方法`__sleep()`是否被定义，以及是否在列表中包含了一个不存在的属性。

现在让我们来看看对应的方法`__wakeup()`。

## 学习 __wakeup()

`__wakeup()`魔术方法的目的主要是在反序列化对象时执行额外的初始化。例如，恢复数据库连接或恢复文件句柄。下面是一个使用`__wakeup()`魔术方法重新打开文件句柄的非常简单的例子：

1.  首先，我们定义一个在实例化时打开文件句柄的类。我们还定义一个返回文件内容的方法：

```php
// /repo/ch05/php8_serialization_wakeup.php
class Gettysburg {
    public $fn = __DIR__ . '/gettysburg.txt';
    public $obj = NULL;
    public function __construct() {
        $this->obj = new SplFileObject($this->fn, 'r');
    }
    public function getText() {
        $this->obj->rewind();
        return $this->obj->fpassthru();
    }
}
```

1.  要使用这个类，创建一个实例，并运行`getText()`。（这假设`$this->fn`引用的文件存在！）

```php
$old = new Gettysburg();
echo $old->getText();
```

1.  输出（未显示）是葛底斯堡演说。

1.  如果我们现在尝试对这个对象进行序列化，就会出现问题。下面是一个可能序列化对象的代码示例：

`$str = serialize($old);`

1.  到目前为止，在原地运行代码，这是输出：

```php
PHP Fatal error:  Uncaught Exception: Serialization of 'SplFileObject' is not allowed in /repo/ch05/php8_serialization_wakeup.php:19
```

1.  为了解决这个问题，我们返回到类中，添加一个`__sleep()`方法，防止`SplFileObject`实例被序列化：

```php
    public function __sleep() {
        return ['fn'];
    }
```

1.  如果我们重新运行代码来序列化对象，一切都很好。这是反序列化和调用`getText()`的代码：

```php
$str = serialize($old);
$new = unserialize($str);
echo $new->getText();
```

1.  然而，如果我们尝试对对象进行反序列化，就会出现另一个错误：

```php
PHP Fatal error:  Uncaught Error: Call to a member function rewind() on null in /repo/ch05/php8_serialization_wakeup.php:13
```

问题当然是，在序列化过程中文件句柄丢失了。当对象被反序列化时，`__construct()`方法没有被调用。

1.  这正是`__wakeup()`魔术方法存在的原因。为了解决错误，我们定义一个`__wakeup()`方法，调用`__construct()`方法：

```php
    public function __wakeup() {
        self::__construct();
    }
```

1.  如果我们重新运行代码，现在我们会看到葛底斯堡演说出现两次（未显示）。

现在您已经了解了 PHP 原生序列化的工作原理，也了解了`__sleep()`和`__wakeup()`魔术方法，以及潜在的代码中断。现在让我们来看一下一个旨在促进对象自定义序列化的接口。

## 介绍 Serializable 接口

为了促进对象的序列化，从 PHP 5.1 开始，语言中添加了`Serializable`接口。这个接口的想法是提供一种识别具有自我序列化能力的对象的方法。此外，该接口指定的方法旨在在对象序列化过程中提供一定程度的控制。

只要一个类实现了这个接口，开发人员就可以确保两个方法被定义：`serialize()`和`unserialize()`。这是接口定义：

```php
interface Serializable {
    public serialize () : string|null
    public unserialize (string $serialized) : void
}
```

任何实现了这个接口的类，在本地序列化或反序列化过程中，其自定义`serialize()`和`unserialize()`方法会自动调用。为了说明这种技术，考虑以下示例：

1.  首先，我们定义一个实现`Serializable`接口的类。该类定义了三个属性 - 两个是字符串类型，另一个表示日期和时间：

```php
// /repo/ch05/php8_bc_break_serializable.php
class A implements Serializable {
    private $a = 'A';
    private $b = 'B';
    private $u = NULL;
```

1.  然后我们定义一个自定义的`serialize()`方法，在序列化对象的属性之前初始化日期和时间。`unserialize()`方法将值恢复到所有属性中：

```php
    public function serialize() {
        $this->u = new DateTime();
        return serialize(get_object_vars($this));
    }
    public function unserialize($payload) {
        $vars = unserialize($payload);
        foreach ($vars as $key => $val)
            $this->$key = $val;
    }
}
```

1.  然后我们创建一个实例，并使用`var_dump()`检查其内容：

```php
$a1 = new A();
var_dump($a1);
```

1.  `var_dump()`的输出显示`u`属性尚未初始化：

```php
object(A)#1 (3) {
  ["a":"A":private]=> string(1) "A"
  ["b":"A":private]=> string(1) "B"
  ["u":"A":private]=> NULL
}
```

1.  然后我们对其进行序列化，并将其恢复到一个变量`$a2`中：

```php
$str = serialize($a1);
$a2 = unserialize($str);
var_dump($a2);
```

1.  从下面的`var_dump()`输出中，您可以看到对象已经完全恢复。此外，我们知道自定义的`serialize()`方法被调用，因为`u`属性被初始化为日期和时间值。以下是输出：

```php
object(A)#3 (3) {
  ["a":"A":private]=> string(1) "A"
  ["b":"A":private]=> string(1) "B"
  ["u":"A":private]=> object(DateTime)#4 (3) {
    ["date"]=> string(26) "2021-02-12 05:35:10.835999"
    ["timezone_type"]=> int(3)
    ["timezone"]=> string(3) "UTC"
  }
}
```

现在让我们来看一下实现`Serializable`接口的对象在序列化过程中可能出现的问题。

## 检查 PHP 可序列化接口问题

早期序列化方法存在一个整体问题。如果要序列化的类定义了一个`__wakeup()`魔术方法，它不会立即在反序列化时被调用。相反，任何定义的`__wakeup()`魔术方法首先被排队，整个对象链被反序列化，然后才执行队列中的方法。这可能导致对象的`unserialize()`方法看到的与其排队的`__wakeup()`方法看到的不一致。

这种架构缺陷可能导致处理实现`Serializable`接口的对象时出现不一致的行为和模棱两可的结果。许多开发人员认为`Serializable`接口由于在嵌套对象序列化时需要创建反向引用而严重破损。这种需要出现在**嵌套序列化调用**的情况下。

例如，当一个类定义了一个方法，该方法反过来调用 PHP 的`serialize()`函数时，可能会发生这样的嵌套调用。在 PHP 8 之前，PHP 序列化中预设了创建反向引用的顺序，可能导致一系列级联的失败。

解决方案是使用两个新的魔术方法来完全控制序列化和反序列化的顺序，接下来将进行描述。

## 控制 PHP 序列化的新魔术方法

控制序列化的新方法首先在 PHP 7.4 中引入，并在 PHP 8 中继续使用。为了利用这项新技术，您只需要实现两个魔术方法：`__serialize()`和`__unserialize()`。如果实现了，PHP 将完全将序列化的控制权交给`__serialize()`方法。同样，反序列化完全由`__unserialize()`魔术方法控制。如果定义了`__sleep()`和`__wakeup()`方法，则会被忽略。

作为一个进一步的好处，PHP 8 完全支持以下 SPL 类中的两个新的魔术方法：

+   `ArrayObject`

+   `ArrayIterator`

+   `SplDoublyLinkedList`

+   `SplObjectStorage`

最佳实践

为了完全控制序列化，实现新的`__serialize()`和`__unserialize()`魔术方法。您不再需要实现`Serializable`接口，也不需要定义`__sleep()`和`__wakeup()`。有关`Serializable`接口最终停用的更多信息，请参阅此 RFC：[`wiki.php.net/rfc/phase_out_serializable`](https://wiki.php.net/rfc/phase_out_serializable)。

作为新的 PHP 序列化用法的示例，请考虑以下代码示例：

1.  在示例中，一个`Test`类在实例化时使用一个随机密钥进行初始化：

```php
// /repo/ch05/php8_bc_break_serialization.php
class Test extends ArrayObject {
    protected $id = 12345;
    public $name = 'Doug';
    private $key = '';
    public function __construct() {
        $this->key = bin2hex(random_bytes(8));
    }
```

1.  我们添加一个`getKey()`方法来显示当前的关键值：

```php
    public function getKey() {
        return $this->key;
    }
```

1.  在序列化时，关键点被过滤出结果字符串：

```php
    public function __serialize() {
        return ['id' => $this->id, 
                'name' => $this->name];
    }
```

1.  在反序列化时，生成一个新的关键点：

```php
    public function __unserialize($data) {
        $this->id = $data['id'];
        $this->name = $data['name'];
        $this->__construct();
    }
}
```

1.  现在我们创建一个实例，并揭示关键点：

```php
$test = new Test();
echo "\nOld Key: " . $test->getKey() . "\n";
```

关键点可能会出现如下：

```php
Old Key: mXq78DhplByDWuPtzk820g==
```

1.  我们添加代码来序列化对象并显示字符串：

```php
$str = serialize($test);
echo $str . "\n";
```

这是序列化字符串可能的样子：

```php
O:4:"Test":2:{s:2:"id";i:12345;s:4:"name";s:4:"Doug";}
```

从输出中可以看到，秘密不会出现在序列化的字符串中。这很重要，因为如果序列化字符串的存储位置受到损害，可能会暴露安全漏洞，使攻击者有可能侵入您的系统。

1.  然后我们添加代码来反序列化字符串并显示关键点：

```php
$obj = unserialize($str);
echo "New Key: " . $obj->getKey() . "\n";
```

这是最后一部分输出。请注意，生成了一个新的关键点：

```php
New Key: kDgU7FGfJn5qlOKcHEbyqQ==
```

正如您所看到的，使用新的 PHP 序列化功能并不复杂。任何时间问题现在完全在您的控制之下，因为新的魔术方法是按照对象序列化和反序列化的顺序执行的。

重要说明

PHP 7.4 及以上版本*能够*理解来自旧版本 PHP 的序列化字符串，但是由 PHP 7.4 或 8.x 序列化的字符串可能无法被旧版本的 PHP 正确反序列化。

提示

有关完整讨论，请参阅有关自定义序列化的 RFC：

https://wiki.php.net/rfc/custom_object_serialization

您现在已经完全了解了 PHP 序列化和两种新的魔术方法提供的改进支持。现在是时候转变思路，看看 PHP 8 如何扩展方差支持了。

# 理解 PHP 8 扩展的方差支持

方差的概念是面向对象编程的核心。**方差**是一个涵盖各种**子类型**相互关系的总称。大约 20 年前，早期计算机科学家 Wing 和 Liskov 提出了一个重要的定理，它是面向对象编程子类型的核心，现在被称为**Liskov 替换原则**。

不需要进入精确的数学，这个原则可以被解释为：

*如果您能够在类 Y 的实例的位置替换 X 的实例，并且应用程序的行为没有任何改变，那么类 X 可以被认为是类 Y 的子类型。*

提示

首次描述并提供了 Liskov 替换原则的精确数学公式定义的实际论文可以在这里找到：*子类型的行为概念*，ACM 编程语言和系统交易，由 B. Liskov 和 J. Wing，1994 年 11 月（https://dl.acm.org/doi/10.1145/197320.197383）。

在本节中，我们将探讨 PHP 8 如何以**协变返回**和**逆变参数**的形式提供增强的方差支持。对协变和逆变的理解将增强您编写良好稳固代码的能力。如果没有这种理解，您的代码可能会产生不一致的结果，并成为许多错误的根源。

让我们首先讨论协变返回。

## 理解协变返回

PHP 中的协变支持旨在保留从最具体到最一般的类型的顺序。这在`try / catch`块的构造中经典地体现出来：

1.  在这个例子中，`PDO`实例是在`try`块内创建的。接下来的两个`catch`块首先寻找`PDOException`。接着是一个第二个`catch`块，它捕获任何实现`Throwable`的类。因为 PHP 的`Exception`和`Error`类都实现了`Throwable`，所以第二个`catch`块最终成为除了`PDOException`之外的任何错误的后备：

```php
try {
    $pdo = new PDO($dsn, $usr, $pwd, $opts);
} catch (PDOException $p) {
    error_log('Database Error: ' . $p->getMessage());
} catch (Throwable $t) {
    error_log('Unknown Error: ' . $t->getMessage());
}
```

1.  在这个例子中，如果`PDO`实例由于无效参数而失败，错误日志将包含条目**数据库错误**，后面跟着从`PDOException`中获取的消息。

1.  另一方面，如果发生了其他一般错误，错误日志将包含条目**未知错误**，后面跟着来自其他`Exception`或`Error`类的消息。

1.  然而，在这个例子中，`catch`块的顺序是颠倒的：

```php
try {
    $pdo = new PDO($dsn, $usr, $pwd, $opts);
} catch (Throwable $t) {
    error_log('Unknown Error: ' . $t->getMessage());
} catch (PDOException $p) {
    error_log('Database Error: ' . $p->getMessage());
}
```

1.  由于 PHP 协变支持的工作方式，第二个`catch`块永远不会被调用。相反，所有源自此代码块的错误日志条目将以**未知错误**开头。

现在让我们看看 PHP 协变支持如何适用于对象方法返回数据类型：

1.  首先，我们定义一个接口`FactoryIterface`，它标识一个`make()`方法。此方法接受一个`array`作为参数，并且预期返回一个`ArrayObject`类型的对象：

```php
interface FactoryInterface {
    public function make(array $arr): ArrayObject;
}
```

1.  接下来，我们定义一个`ArrTest`类，它扩展了`ArrayObject`：

```php
class ArrTest extends ArrayObject {
    const DEFAULT_TEST = 'This is a test';
}
```

1.  `ArrFactory`类实现了`FactoryInterface`并完全定义了`make()`方法。但是，请注意，此方法返回`ArrTest`数据类型而不是`ArrayObject`：

```php
class ArrFactory implements FactoryInterface {
    protected array $data;
    public function make(array $data): ArrTest {
        $this->data = $data;
        return new ArrTest($this->data);
    }
}
```

1.  在程序调用代码块中，我们创建了一个`ArrFactory`的实例，并两次运行其`make()`方法，理论上产生了两个`ArrTest`实例。然后我们使用`var_dump()`来显示所产生的两个对象的当前状态：

```php
$factory = new ArrFactory();
$obj1 = $factory->make([1,2,3]);
$obj2 = $factory->make(['A','B','C']);
var_dump($obj1, $obj2);
```

1.  在 PHP 7.1 中，由于它不支持协变返回数据类型，会抛出致命的`Error`。下面显示的输出告诉我们，方法返回类型声明与`FactoryInterface`中定义的不匹配：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php8_variance_covariant.php
PHP Fatal error:  Declaration of ArrFactory::make(array $data): ArrTest must be compatible with FactoryInterface::make(array $arr): ArrayObject in /repo/ch05/php8_variance_covariant.php on line 9
```

1.  当我们在 PHP 8 中运行相同的代码时，您会看到对返回类型提供了协变支持。执行继续进行，如下所示：

```php
root@php8_tips_php8 [ /repo/ch05 ]# 
php php8_variance_covariant.php
object(ArrTest)#2 (1) {
  ["storage":"ArrayObject":private]=>
  array(3) {
    [0]=>    int(1)
    [1]=>    int(2)
    [2]=>    int(3)
  }
}
object(ArrTest)#3 (1) {
  ["storage":"ArrayObject":private]=>
  array(3) {
    [0]=>    string(1) "A"
    [1]=>    string(1) "B"
    [2]=>    string(1) "C"
  }
}
```

`ArrTest`扩展了`ArrayObject`，是一个明显符合里氏替换原则定义的条件的子类型。正如您从最后的输出中看到的那样，PHP 8 比之前的 PHP 版本更全面地接受了真正的面向对象编程原则。最终结果是，在使用 PHP 8 时，您的代码和应用架构可以更直观和逻辑合理。

现在让我们来看看逆变参数。

## 使用逆变参数

而协变关注的是从一般到特定的子类型的顺序，**逆变**关注的是相反的顺序：从特定到一般。在 PHP 7 及更早版本中，对逆变的完全支持是不可用的。因此，在 PHP 7 中，实现接口或扩展抽象类时，参数类型提示是**不变**的。

另一方面，在 PHP 8 中，由于对逆变参数的支持，您可以在顶级超类和接口中自由地具体化。只要子类型是兼容的，您就可以修改扩展或实现类中的类型提示为更一般的类型。

这使您在定义整体架构时更自由地定义接口或抽象类。在使用您的接口或超类的开发人员在实现后代类逻辑时，PHP 8 在实现时提供了更多的灵活性。

让我们来看看 PHP 8 对逆变参数的支持是如何工作的：

1.  在这个例子中，我们首先定义了一个`IterObj`类，它扩展了内置的`ArrayIterator PHP 类`：

```php
// /repo/ch05/php8_variance_contravariant.php
class IterObj extends ArrayIterator {}
```

1.  然后我们定义一个抽象的`Base`类，规定了一个`stringify()`方法。请注意，它唯一参数的数据类型是`IterObj`：

```php
abstract class Base {
    public abstract function stringify(IterObj $it);
}
```

1.  接下来，我们定义一个`IterTest`类，它扩展了`Base`并为`stringify()`方法提供了实现。特别值得注意的是，我们覆盖了数据类型，将其更改为`iterable`：

```php
class IterTest extends Base {
    public function stringify(iterable $it) {
        return implode(',', 
            iterator_to_array($it)) . "\n";
    }
}
class IterObj extends ArrayIterator {}
```

1.  接下来的几行代码创建了`IterTest`、`IterObj`和`ArrayIterator`的实例。然后我们调用`stringify()`方法两次，分别将后两个对象作为参数提供：

```php
$test  = new IterTest();
$objIt = new IterObj([1,2,3]);
$arrIt = new ArrayIterator(['A','B','C']);
echo $test->stringify($objIt);
echo $test->stringify($arrIt);
```

1.  在 PHP 7.1 中运行此代码示例会产生预期的致命`Error`，如下所示：

```php
root@php8_tips_php7 [ /repo/ch05 ]#
php php8_variance_contravariant.php
PHP Fatal error:  Declaration of IterTest::stringify(iterable $it) must be compatible with Base::stringify(IterObj $it) in /repo/ch05/php8_variance_contravariant.php on line 11
```

因为 PHP 7.1 不支持逆变参数，它将其参数的数据类型视为不变，并简单地显示一条消息，指示子类的数据类型与父类中指定的数据类型不兼容。

1.  另一方面，PHP 8 提供了对逆变参数的支持。因此，它认识到`IterObj`，在`Base`类中指定的数据类型，是与`iterable`兼容的子类型。此外，提供的两个参数也与`iterable`兼容，允许程序执行继续进行。以下是 PHP 8 的输出：

```php
root@php8_tips_php8 [ /repo/ch05 ]# php php8_variance_contravariant.php
1,2,3
A,B,C
```

PHP 8 对协变返回和逆变参数的支持带来的主要优势是能够覆盖方法逻辑以及**方法签名**。您会发现，尽管 PHP 8 在执行良好的编码实践方面要严格得多，但增强的变异支持使您在设计继承结构时拥有更大的自由度。在某种意义上，至少在参数和返回值数据类型方面，PHP 8 是*更*不受限制的！

提示

要了解 PHP 7.4 和 PHP 8 中如何应用方差支持的完整解释，请查看这里：https://wiki.php.net/rfc/covariant-returns-and-contravariant-parameters。

现在我们将看一下 SPL 的更改以及这些更改如何影响迁移到 PHP 8 后的应用程序性能。

# 处理标准 PHP 库（SPL）更改

**SPL**是一个包含实现基本数据结构和增强面向对象功能的关键类的扩展。它首次出现在 PHP 5 中，现在默认包含在所有 PHP 安装中。涵盖整个 SPL 超出了本书的范围。相反，在本节中，我们讨论了在运行 PHP 8 时 SPL 发生了重大变化的地方。此外，我们还为您提供了有可能导致现有应用程序停止工作的 SPL 更改的提示和指导。

我们首先检查`SplFileObject`类的更改。

## 了解 SplFileObject 的更改

`SplFileObject`是一个很好的类，它将大部分独立的`f*()`函数（如`fgets()`，`fread()`，`fwrite()`等）合并到一个类中。`SplFileObject ::__construct()`方法的参数与`fopen()`函数提供的参数相同。

PHP 8 中的主要区别是，一个相对不常用的方法`fgetss()`已从`SplFileObject`类中删除。`SplFileObject::fgetss()`方法在 PHP 7 及以下版本中可用，它将`fgets()`与`strip_tags()`结合在一起。

为了举例说明，假设您已经创建了一个网站，允许用户上传文本文件。在显示文本文件内容之前，您希望删除任何标记。以下是使用`fgetss()`方法实现此目的的示例：

1.  我们首先定义一个获取文件名的代码块：

```php
// /repo/ch05/php7_spl_splfileobject.php
$fn = $_GET['fn'] ?? '';
if (!$fn || !file_exists($fn))
    exit('Unable to locate file');
```

1.  然后我们创建`SplFileObject`实例，并使用`fgetss()`方法逐行读取文件。最后，我们输出安全内容：

```php
$obj = new SplFileObject($fn, 'r');
$safe = '';
while ($line = $obj->fgetss()) $safe .= $line;
echo '<h1>Contents</h1><hr>' . $safe;
```

1.  假设要读取的文件是这个：

```php
<h1>This File is Infected</h1>
<script>alert('You Been Hacked');</script>
<img src="http://very.bad.site/hacked.php" />
```

1.  以下是在 PHP 7.1 中使用此 URL 运行的输出：

`http://localhost:7777/ch05/php7_spl_splfileobject.php? fn=includes/you_been_hacked.html`

从接下来显示的输出中可以看出，所有 HTML 标记都已被删除：

![图 5.1 - 使用 SplFileObject::fgetss()读取文件后的结果](img/Figure_5.1_B6992.jpg)

图 5.1 - 使用 SplFileObject::fgetss()读取文件后的结果

在 PHP 8 中实现相同的功能，之前显示的代码需要通过用`fgets()`替换`fgetss()`来进行修改。我们还需要在连接到`$safe`的行上使用`strip_tags()`。修改后的代码可能如下所示：

```php
// /repo/ch05/php8_spl_splfileobject.php
$fn = $_GET['fn'] ?? '';
if (!$fn || !file_exists($fn))
    exit('Unable to locate file');
$obj = new SplFileObject($fn, 'r');
$safe = '';
while ($line = $obj->fgets())
    $safe .= strip_tags($line);
echo '<h1>Contents</h1><hr>' . $safe;
```

修改后的代码的输出与*图 5.1*中显示的相同。现在我们将注意力转向另一个 SPL 类的更改：`SplHeap`。

## 检查 SplHeap 的更改

`SplHeap`是一个基础类，用于表示**二叉树**结构的数据。另外还有两个类建立在`SplHeap`基础上。`SplMinHeap`将树组织为顶部是最小值。`SplMaxHeap`则相反，将最大值放在顶部。

堆结构在数据无序到达的情况下特别有用。一旦插入堆中，项目会自动按正确的顺序放置。因此，在任何给定时刻，您可以显示堆，确保所有项目都按顺序排列，而无需运行 PHP 排序函数之一。

保持自动排序顺序的关键是定义一个抽象方法`compare()`。由于这个方法是抽象的，`SplHeap`不能直接实例化。相反，您需要扩展该类并实现`compare()`。

在 PHP 8 中，使用`SplHeap`可能会导致向后兼容的代码中断，因为`compare()`的方法签名必须完全如下：`SplHeap::compare($value1, $value2)`。

让我们现在来看一个使用`SplHeap`构建按姓氏组织的亿万富翁列表的代码示例：

1.  首先，我们定义一个包含亿万富翁数据的文件。在这个例子中，我们只是从这个来源复制并粘贴了数据：https://www.bloomberg.com/billionaires/。

1.  然后，我们定义一个`BillionaireTracker`类，从粘贴的文本中提取信息到有序对的数组中。该类的完整源代码（未在此处显示）可以在源代码存储库中找到：`/repo/src/Services/BillionaireTracker.php`。

这是该类生成的数据的样子：

```php
array(20) {
  [0] =>  array(1) {
    [177000000000] =>    string(10) "Bezos,Jeff"
  }
  [1] =>  array(1) {
    [157000000000] =>    string(9) "Musk,Elon"
  }
  [2] =>  array(1) {
    [136000000000] =>    string(10) "Gates,Bill"
  }
  ... remaining data not shown
```

正如你所看到的，数据以降序呈现，其中键表示净值。相比之下，在我们的示例程序中，我们计划按姓氏的升序产生数据。

1.  然后，我们定义一个常量，用于标识亿万富翁数据源文件，并设置一个自动加载程序：

```php
// /repo/ch05/php7_spl_splheap.php
define('SRC_FILE', __DIR__ 
    . '/../sample_data/billionaires.txt');
require_once __DIR__ 
    . '/../src/Server/Autoload/Loader.php';
$loader = new \Server\Autoload\Loader();
```

1.  接下来，我们创建一个`BillionaireTracker`类的实例，并将结果赋给`$list`：

```php
use Services\BillionaireTracker;
$tracker = new BillionaireTracker();
$list = $tracker->extract(SRC_FILE);
```

1.  现在来看最感兴趣的部分：创建堆。为了实现这一点，我们定义了一个扩展`SplHeap`的匿名类。然后，我们定义了一个`compare()`方法，执行必要的逻辑将插入的元素放在适当的位置。PHP 7 允许你改变方法签名。在这个例子中，我们以数组的形式提供参数：

```php
$heap = new class () extends SplHeap {
    public function compare(
        array $arr1, array $arr2) : int {
        $cmp1 = array_values($arr2)[0];
        $cmp2 = array_values($arr1)[0];
        return $cmp1 <=> $cmp2;
    }
};
```

你可能还注意到`$cmp1`的值是从第二个数组中赋值的，而`$cmp2`的值是从第一个数组中赋值的。这种切换的原因是因为我们希望按升序产生结果。

1.  然后，我们使用`SplHeap::insert()`将元素添加到堆中：

```php
foreach ($list as $item)
    $heap->insert($item);
```

1.  最后，我们定义了一个`BillionaireTracker::view()`方法（未显示）来遍历堆并显示结果：

```php
$patt = "%20s\t%32s\n";
$line = str_repeat('-', 56) . "\n";
echo $tracker->view($heap, $patt, $line);
```

1.  以下是我们在 PHP 7.1 中运行的小程序产生的输出：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php7_spl_splheap.php
--------------------------------------------------------
           Net Worth                                Name
--------------------------------------------------------
      84,000,000,000                       Ambani,Mukesh
     115,000,000,000                     Arnault,Bernard
      83,600,000,000                       Ballmer,Steve
      ... some lines were omitted to save space ...
      58,200,000,000                          Walton,Rob
     100,000,000,000                     Zuckerberg,Mark
--------------------------------------------------------
                                       1,795,100,000,000
--------------------------------------------------------
```

然而，当我们尝试在 PHP 8 中运行相同的程序时，会抛出错误。以下是在 PHP 8 中运行相同程序的输出：

```php
root@php8_tips_php8 [ /repo/ch05 ]# php php7_spl_splheap.php 
PHP Fatal error:  Declaration of SplHeap@anonymous::compare(array $arr1, array $arr2): int must be compatible with SplHeap::compare(mixed $value1, mixed $value2) in /repo/ch05/php7_spl_splheap.php on line 16
```

因此，为了使其正常工作，我们必须重新定义扩展`SplHeap`的匿名类。以下是代码的部分修改版本：

```php
$heap = new class () extends SplHeap {
    public function compare($arr1, $arr2) : int {
        $cmp1 = array_values($arr2)[0];
        $cmp2 = array_values($arr1)[0];
        return $cmp1 <=> $cmp2;
    }
};
```

唯一的变化在于`compare()`方法的签名。执行时，结果（未显示）是相同的。PHP 8 的完整代码可以在`/repo/ch05/php8_spl_splheap.php`中查看。

这结束了我们对`SplHeap`类的更改的讨论。请注意，相同的更改也适用于`SplMinHeap`和`SplMaxHeap`。现在让我们来看看`SplDoublyLinkedList`类中可能有重大变化的地方。

## 处理`SplDoublyLinkedList`中的更改

`SplDoublyLinkedList`类是一个迭代器，能够以**FIFO**（先进先出）或**LIFO**（后进先出）的顺序显示信息。然而，更常见的是说你可以以正向或反向顺序遍历列表。

这是任何开发者库中非常强大的一个补充。要使用`ArrayIterator`做同样的事情，例如，至少需要十几行代码！因此，PHP 开发者喜欢在需要随意在列表中导航的情况下使用这个类。

不幸的是，由于`push()`和`unshift()`方法的返回值不同，可能会出现潜在的代码中断。`push()`方法用于在列表的*末尾*添加值。另一方面，`unshift()`方法则在列表的*开头*添加值。

在 PHP 7 及以下版本中，如果成功，这些方法返回布尔值`TRUE`。如果方法失败，它返回布尔值`FALSE`。然而，在 PHP 8 中，这两种方法都不返回任何值。如果你查看当前文档中的方法签名，你会看到返回数据类型为`void`。可能会出现代码中断的情况，即在继续之前检查返回`push()`或`unshift()`的值。

让我们看一个简单的例子，用一个简单的五个值的列表填充一个双向链表，并以 FIFO 和 LIFO 顺序显示它们：

1.  首先，我们定义一个匿名类，它继承了`SplDoublyLinkedList`。我们还添加了一个`show()`方法来显示列表的内容：

```php
// /repo/ch05/php7_spl_spldoublylinkedlist.php
$double = new class() extends SplDoublyLinkedList {
    public function show(int $mode) {
        $this->setIteratorMode($mode);
        $this->rewind();
        while ($item = $this->current()) {
            echo $item . '. ';
            $this->next();
        }
    }
};
```

1.  接下来，我们定义一个样本数据的数组，并使用`push()`将值插入到链表中。请注意，我们使用`if()`语句来确定操作是否成功或失败。如果操作失败，将抛出一个`Exception`：

```php
$item = ['Person', 'Woman', 'Man', 'Camera', 'TV'];
foreach ($item as $key => $value)
    if (!$double->push($value))
        throw new Exception('ERROR');
```

这是潜在代码中断存在的代码块。在 PHP 7 及更低版本中，`push()`返回`TRUE`或`FALSE`。在 PHP 8 中，没有返回值。

1.  然后我们使用`SplDoublyLinkedList`类的常量将模式设置为 FIFO（正向），并显示列表：

```php
echo "**************** Foward ********************\n";
$forward = SplDoublyLinkedList::IT_MODE_FIFO
         | SplDoublyLinkedList::IT_MODE_KEEP;
$double->show($forward);
```

1.  接下来，我们使用`SplDoublyLinkedList`类的常量将模式设置为 LIFO（反向），并显示列表：

```php
echo "\n\n************* Reverse *****************\n";
$reverse = SplDoublyLinkedList::IT_MODE_LIFO
         | SplDoublyLinkedList::IT_MODE_KEEP;
$double->show($reverse);
```

这是在 PHP 7.1 中运行的输出：

```php
root@php8_tips_php7 [ /repo/ch05 ]# 
php php7_spl_spldoublylinkedlist.php
**************** Foward ********************
Person. Woman. Man. Camera. TV. 
**************** Reverse ********************
TV. Camera. Man. Woman. Person. 
```

1.  如果我们在 PHP 8 中运行相同的代码，这是结果：

```php
root@php8_tips_php8 [ /home/ch05 ]# 
php php7_spl_spldoublylinkedlist.php 
PHP Fatal error:  Uncaught Exception: ERROR in /home/ch05/php7_spl_spldoublylinkedlist.php:23
```

如果`push()`没有返回任何值，在`if()`语句中 PHP 会假定为`NULL`，然后被插入为布尔值`FALSE`！因此，在第一个`push()`命令之后，`if()`块会导致抛出一个`Exception`。因为`Exception`没有被捕获，会生成一个致命的`Error`。

要将这个代码块重写为在 PHP 8 中工作，您只需要删除`if()`语句，并且不抛出`Exception`。重写后的代码块（在*步骤 2*中显示）可能如下所示：

```php
$item = ['Person', 'Woman', 'Man', 'Camera', 'TV'];
foreach ($item as $key => $value)
    $double->push($value);
```

现在，如果我们执行重写后的代码，结果如下所示：

```php
root@php8_tips_php7 [ /home/ch05 ]# 
php php8_spl_spldoublylinkedlist.php 
**************** Foward ********************
Person. Woman. Man. Camera. TV. 
**************** Reverse ********************
TV. Camera. Man. Woman. Person. 
```

现在您已经了解如何使用`SplDoublyLinkedList`，并且也知道关于`push()`或`unshift()`可能出现的潜在代码中断。您还了解了在 PHP 8 中使用各种 SPL 类和函数可能出现的潜在代码中断。这就结束了本章的讨论。

# 总结

在本章中，您学到了在迁移到 PHP 8 时面向对象编程代码可能出现的问题。在第一节中，您了解到在 PHP 7 和之前的版本中允许许多不良实践，但现在在 PHP 8 中可能导致代码中断。有了这些知识，您将成为一个更好的开发人员，并能够提供高质量的代码来造福您的公司。

在下一节中，您学到了在使用魔术方法时的良好习惯。由于 PHP 8 现在强制实施了在早期版本中没有看到的一定程度的一致性，因此可能会出现代码中断。这些不一致性涉及类构造函数的使用和魔术方法使用的某些方面。接下来的部分教会了您关于 PHP 序列化以及 PHP 8 中所做的更改如何使您的代码更具弹性，并在序列化和反序列化过程中更不容易出现错误或受攻击。

在本章中，您还了解了 PHP 8 对协变返回类型和逆变参数的增强支持。了解了协变的知识，以及在 PHP 8 中支持的改进，使您在开发 PHP 8 中的类继承结构时更具创造性和灵活性。现在您知道如何编写在早期版本的 PHP 中根本不可能的代码。

最后一节涵盖了 SPL 中的许多关键类。您学到了关于如何在 PHP 8 中实现基本数据结构，比如堆和链表的重要知识。该部分的信息对帮助您避免涉及 SPL 的代码问题至关重要。

下一章将继续讨论潜在的代码中断。然而，下一章的重点是*过程式*而不是对象代码。
