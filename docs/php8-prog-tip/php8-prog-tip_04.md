# *第三章*：利用错误处理增强功能

如果您是 PHP 开发人员，您会注意到随着语言不断成熟，越来越多的保障措施被制定出来，最终强制执行良好的编码实践。在这方面，PHP 8 的一个关键改进是其先进的错误处理能力。在本章中，您将了解哪些`Notices`已升级为`Warnings`，哪些`Warnings`已升级为`Errors`。

本章让您对安全增强的背景和意图有了很好的理解，从而使您更好地控制代码的使用。此外，了解以前只生成`Warnings`但现在也生成`Errors`的错误条件，以采取措施防止在升级到 PHP 8 后应用程序失败，也是至关重要的。

本章涵盖以下主题：

+   理解 PHP 8 错误处理

+   处理现在是错误的警告

+   理解提升为警告的通知

+   处理`@`错误控制运算符

# 技术要求

为了检查和运行本章提供的代码示例，以下是最低推荐的硬件要求：

+   基于 x86_64 桌面 PC 或笔记本电脑

+   1 **千兆字节**（**GB**）的可用磁盘空间

+   4 GB 的**随机存取存储器**（**RAM**）

+   500 **千比特每秒**（**Kbps**）或更快的互联网连接

此外，您还需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 安装的更多信息，请参阅*第一章*的*技术要求*部分，介绍新的 PHP 8 OOP 功能，以及如何构建用于演示本书中所解释的代码的 Docker 容器。在本书中，我们将恢复本书示例代码的目录称为`/repo`。

本章的源代码位于此处：

[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices)

我们现在可以通过检查新的 PHP 8 运算符来开始我们的讨论。

# 理解 PHP 8 错误处理

从历史上看，许多 PHP 错误条件被分配了远低于其实际严重性的错误级别。这给开发人员一种错误的安全感，因为他们只看到一个`Notice`，就认为他们的代码没有问题。许多情况以前只生成`Notice`或`Warning`，而实际上它们的严重性值得更多的关注。

在本节中，我们将看看 PHP 8 中一些错误处理的增强功能，这些功能继续执行强制执行良好编码实践的总体趋势。本章的讨论将帮助您重新审视您的代码，以便更高效地进行编码，并减少未来的维护问题。

在接下来的几个小节中，我们将看看对某些可能影响您的代码的`Notice`和`Warning`错误条件的更改。让我们首先看看 PHP 8 如何处理未定义变量的更改。

## 未定义变量处理

PHP 的一个臭名昭著的特性是它如何处理**未定义的变量**。看一下这个简单的代码块。请注意，`$a`和`$b`变量没有被定义：

```php
// /repo/ch03/php8_undef_var.php
$c = $a + $b;
var_dump($c);
```

在 PHP 7 下运行，这是输出：

```php
PHP Notice:  Undefined variable: a in
/repo/ch03/php7_undef_var.php on line 3
PHP Notice:  Undefined variable: b in /repo/ch03/php7_undef_var.php on line 3
int(0)
```

从输出中可以看出，PHP 7 发出了一个`Notice`，让我们知道我们正在使用未定义的变量。如果我们使用 PHP 8 运行完全相同的代码，您可以快速看到以前的`Notice`已经提升为`Warning`，如下所示：

```php
PHP Warning:  Undefined variable $a in /repo/ch03/php8_undef_var.php on line 3
PHP Warning:  Undefined variable $b in /repo/ch03/php8_undef_var.php on line 3
int(0)
```

PHP 8 中错误级别提升背后的推理是，许多开发人员认为使用未定义变量是一种无害的做法，实际上却是非常危险的！ 你可能会问为什么？答案是，PHP 在没有明确指示的情况下，会将任何未定义的变量赋值为 `NULL`。 实际上，您的程序依赖于 PHP 的默认行为，这在将来的语言升级中可能会发生变化。

我们将在本章的接下来几节中介绍其他错误级别的提升。 但是，请注意，将 `Notices` 提升为 `Warnings` 的情况 *不会影响代码的功能*。 但是，它可能会引起更多潜在问题的注意，如果是这样，它就达到了产生更好代码的目的。 与未定义变量不同，未定义常量的错误现在已经进一步提升，您将在下一小节中看到。

## 未定义常量处理

在 PHP 8 中运行时，**未定义常量**的处理方式已经发生了变化。 但是，在这种情况下，以前是 `Warning` 的现在在 PHP 8 中是 `Error`。 看看这个看似无害的代码块：

```php
// /repo/ch03/php7_undef_const.php
echo PHP_OS . "\n";
echo UNDEFINED_CONSTANT . "\n";
echo "Program Continues ... \n";
```

第一行回显了一个标识操作系统的 `PHP_OS` **预定义常量**。 在 PHP 7 中，会生成一个 `Notice`；然而，输出的最后一行是 `Program Continues ...`，如下所示：

```php
PHP Notice:  Use of undefined constant UNDEFINED_CONSTANT - assumed 'UNDEFINED_CONSTANT' in /repo/ch03/php7_undef_const.php on line 6
Program Continues ... 
```

同样的代码现在在 PHP 8 中运行时会产生*致命错误*，如下所示：

```php
PHP Fatal error:  Uncaught Error: Undefined constant "UNDEFINED_CONSTANT" in /repo/ch03/php8_undef_const.php:6
```

因此，在 PHP 8 中，任何未在使用之前首先定义任何常量的糟糕代码都将崩溃和燃烧！一个好习惯是在应用程序代码的开头为所有变量分配默认值。 如果您计划使用常量，最好尽早定义它们，最好在一个地方。

重要提示

一个想法是在一个*包含的文件*中定义所有常量。 如果是这种情况，请确保使用这些常量的任何程序脚本已加载包含常量定义的文件。

提示

**最佳实践**：在程序代码使用之前，为所有变量分配默认值。 确保在使用之前定义任何自定义常量。 如果是这种情况，请确保使用这些常量的任何程序脚本已加载包含常量定义的文件。

## 错误级别默认值

值得注意的是，在 PHP 8 中，`php.ini` 文件 `error_reporting` 指令分配的错误级别默认值已经更新。 在 PHP 7 中，默认的 `error_reporting` 级别如下：

```php
error_reporting=E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED
```

在 PHP 8 中，新级别要简单得多，您可以在这里看到：

```php
error_reporting=E_ALL
```

值得注意的是，`php.ini` 文件设置 `display_startup_errors` 现在默认启用。 这可能会成为生产服务器的问题，因为您的网站可能会在 PHP 启动时意外地显示错误信息。

本节的关键要点是，过去，PHP 允许您通过只发出 `Notices` 或 `Warnings` 来逃脱某些不良实践。 但是，正如您在本节中所学到的，不解决 `Warning` 或 `Notice` 生成背后的问题的危险在于 PHP 在您的代表上悄悄采取的行动。 不依赖 PHP 代表您做决定会减少隐藏的逻辑错误。 遵循良好的编码实践，例如在使用之前为所有变量分配默认值，有助于避免此类错误。 现在让我们更仔细地看看在 PHP 8 中将 `Warnings` 提升为 `Errors` 的错误情况。

# 处理现在是错误的警告

在本节中，我们将研究升级的 PHP 8 错误处理，涉及对象、数组和字符串。我们还将研究过去 PHP 发出“警告”的情况，而在 PHP 8 中现在会抛出“错误”。你必须意识到本节中描述的任何潜在错误情况。原因很简单：如果你没有解决本节描述的情况，当你的服务器升级到 PHP 8 时，你的代码将会出错。

开发人员经常时间紧迫。可能有一大堆新功能或其他必须进行的更改。在其他情况下，资源已经被调走到其他项目，意味着可用于维护的开发人员更少。由于应用程序继续运行，很多开发人员经常忽略“警告”，所以他们只是关闭错误显示，希望一切顺利。

多年来，堆积如山的糟糕代码已经积累起来。不幸的是，PHP 社区现在正在付出代价，以神秘的运行时错误的形式，需要花费数小时来追踪。通过将之前只引发“警告”的某些危险做法提升为“错误”，在 PHP 8 中很快就能显现出糟糕的编码实践，因为“错误”是致命的，会导致应用程序停止运行。

让我们从对象错误处理中的错误提升开始。

重要提示

一般来说，在 PHP 8 中，当尝试*写入*数据时，“警告”会升级为“错误”。另一方面，对于相同的一般情况（例如，尝试读/写不存在对象的属性），在 PHP 8 中，当尝试*读取*数据时，“通知”会升级为“警告”。总体上的理由是，写入尝试可能导致数据的丢失或损坏，而读取尝试则不会。

## 对象错误处理中的警告提升

这里是现在被视为对象处理的“警告”现在变成了“错误”的简要总结。如果你尝试做以下操作，PHP 8 会抛出一个“错误”：

+   增加/减少非对象的属性

+   修改非对象的属性

+   给非对象的属性赋值

+   从空值创建默认对象

让我们看一个简单的例子。在下面的代码片段中，一个值被赋给了一个不存在的对象`$a`。然后对这个值进行了递增：

```php
// /repo/ch03/php8_warn_prop_nobj.php
$a->test = 0;
$a->test++;
var_dump($a);
```

这是 PHP 7 的输出：

```php
PHP Warning:  Creating default object from empty value in /repo/ch03/php8_warn_prop_nobj.php on line 4
class stdClass#1 (1) {
  public $test =>
  int(1)
}
```

正如你所看到的，在 PHP 7 中，一个`stdClass()`实例被默默创建，并发出一个“警告”，但操作是允许继续的。如果我们在 PHP 8 下运行相同的代码，注意这里输出的差异：

```php
PHP Fatal error:  Uncaught Error: Attempt to assign property "test" on null in /repo/ch03/php8_warn_prop_nobj.php:4
```

好消息是在 PHP 8 中，会抛出一个“错误”，这意味着我们可以通过实现一个`try()/catch()`块轻松地捕获它。例如，这里是之前显示的代码可能如何重写的示例：

```php
try {
    $a->test = 0;
    $a->test++;
    var_dump($a);
} catch (Error $e) {
    error_log(__FILE__ . ':' . $e->getMessage());
}
```

正如你所看到的，这三行中的任何问题现在都安全地包裹在一个`try()/catch()`块中，这意味着可以进行恢复。我们现在将注意力转向数组错误处理的增强。

## 数组处理中的警告提升

关于数组的一些不良实践，在 PHP 7 及更早版本中是允许的，现在会抛出一个“错误”。正如前一小节所讨论的，PHP 8 数组错误处理的变化旨在对我们描述的错误情况给出更有力的响应。这些增强的最终目标是推动开发人员朝着良好的编码实践方向发展。

这是数组处理中的警告提升为错误的简要列表：

+   无法将元素添加到数组中，因为下一个元素已经被占用

+   无法取消非数组变量中的偏移量

+   只有`array`和`Traversable`类型可以被解包

+   非法偏移类型

现在让我们逐一检查这个列表中的每个错误条件。

### 下一个元素已经被占用

为了说明一个可能的情况，即下一个数组元素无法被分配，因为它已经被占用，请看这个简单的代码示例：

```php
// ch03/php8_warn_array_occupied.php
$a[PHP_INT_MAX] = 'This is the end!';
$a[] = 'Off the deep end';
```

假设由于某种原因，对一个数组元素进行赋值，其数字键是可能的最大大小的整数（由`PHP_INT_MAX`预定义常量表示）。如果随后尝试给下一个元素赋值，就会出现问题！

在 PHP 7 中运行此代码块的结果如下：

```php
PHP Warning:  Cannot add element to the array as the next element is already occupied in
/repo/ch03/php8_warn_array_occupied.php on line 7
array(1) {
  [9223372036854775807] =>
  string(16) "This is the end!"
}
```

然而，在 PHP 8 中，`Warning`已经升级为`Error`，导致了这样的输出：

```php
PHP Fatal error:  Uncaught Error: Cannot add element to the
array as the next element is already occupied in
/repo/ch03/php8_warn_array_occupied.php:7
```

接下来，我们将注意力转向在非数组变量中使用偏移量的情况。

### 非数组变量中的偏移量

将非数组变量视为数组可能会产生意外结果，但某些实现了`Traversable`（例如`ArrayObject`或`ArrayIterator`）的对象类除外。一个例子是在字符串上使用类似数组的偏移量。

使用数组语法访问字符串字符在某些情况下可能很有用。一个例子是检查**统一资源定位符**（**URL**）是否以逗号或斜杠结尾。在下面的代码示例中，我们检查 URL 是否以斜杠结尾。如果是的话，我们使用`substr()`将其截断：

```php
// ch03/php8_string_access_using_array_syntax.php
$url = 'https://unlikelysource.com/';
if ($url[-1] == '/')
    $url = substr($url, 0, -1);
echo $url;
// returns: "https://unlikelysource.com"
```

在先前显示的示例中，`$url[-1]`数组语法使您可以访问字符串中的最后一个字符。

提示

您还可以使用新的 PHP 8 `str_ends_with()`函数来执行相同的操作！

然而，字符串绝对**不是**数组，也不应该被视为数组。为了避免糟糕的代码可能导致意外结果，PHP 8 中已经限制了使用数组语法引用字符串字符的滥用。

在下面的代码示例中，我们尝试在字符串上使用`unset()`：

```php
// ch03/php8_warn_array_unset.php
$alpha = 'ABCDEF';
unset($alpha[2]);
var_dump($alpha);
```

上面的代码示例实际上会在 PHP 7 和 8 中生成致命错误。同样，不要将非数组（或非`Traversable`对象）用作`foreach()`循环的参数。在接下来显示的示例中，将字符串作为`foreach()`的参数：

```php
// ch03/php8_warn_array_foreach.php
$alpha = 'ABCDEF';
foreach ($alpha as $letter) echo $letter;
echo "Continues ... \n";
```

在 PHP 7 和早期版本中，会生成一个`Warning`，但代码会继续执行。在 PHP 7.1 中运行时的输出如下：

```php
PHP Warning:  Invalid argument supplied for foreach() in /repo/ch03/php8_warn_array_foreach.php on line 6
Continues ... 
```

有趣的是，PHP 8 也允许代码继续执行，但`Warning`消息略有详细，如下所示：

```php
PHP Warning:  foreach() argument must be of type array|object, string given in /repo/ch03/php8_warn_array_foreach.php on line 6
Continues ... 
```

接下来，我们将看看过去可以使用非数组/非`Traversable`类型进行展开的情况。

### 数组展开

看到这个小节标题后，您可能会问：*什么是数组展开？* 就像解引用的概念一样，**展开**数组只是一个从数组中提取值到离散变量的术语。例如，考虑以下简单的代码：

1.  我们首先定义一个简单的函数，用于将两个数字相加，如下所示：

```php
// ch03/php8_array_unpack.php
function add($a, $b) { return $a + $b; }
```

1.  为了说明，假设数据以数字对的形式存在数组中，每个数字对都要相加：

```php
$vals = [ [18,48], [72,99], [11,37] ];
```

1.  在循环中，我们使用可变操作符（`...`）来展开对`add()`函数的调用中的数组对，如下所示：

```php
foreach ($vals as $pair) {
    echo 'The sum of ' . implode(' + ', $pair) . 
         ' is ';
    echo add(...$pair);
}
```

刚才展示的示例演示了开发人员如何使用可变操作符来强制展开。然而，许多 PHP 数组函数在内部执行展开操作。考虑以下示例：

1.  首先，我们定义一个由字母组成的数组。如果我们输出`array_pop()`的返回值，我们会看到输出的是字母`Z`，如下面的代码片段所示：

```php
// ch03/php8_warn_array_unpack.php
$alpha = range('A','Z');
echo array_pop($alpha) . "\n";
```

1.  我们可以使用`implode()`将数组展平为字符串来实现相同的结果，并使用字符串解引用来返回最后一个字母，如下面的代码片段所示：

```php
$alpha = implode('', range('A','Z'));
echo $alpha[-1];
```

1.  然而，如果我们尝试在字符串上使用`array_pop()`，就像这里所示，在 PHP 7 和早期版本中我们会得到一个`Warning`：

`echo array_pop($alpha);`

1.  在 PHP 7.1 下运行时的输出如下：

```php
ZZPHP Warning:  array_pop() expects parameter 1 to be array, string given in /repo/ch03/php8_warn_array_unpack.php on line 14
```

1.  以下是在相同的代码文件下在 PHP 8 下运行时的输出：

```php
ZZPHP Fatal error:  Uncaught TypeError: array_pop(): Argument #1 ($array) must be of type array, string given in /repo/ch03/php8_warn_array_unpack.php:14
```

正如我们已经提到的，这里是另一个例子，以前会导致`Warning`的情况现在在 PHP 8 中导致`TypeError`。然而，这两组输出也说明了，尽管你可以像操作数组一样对字符串进行解引用，但字符串不能以与数组相同的方式进行解包。

接下来，我们来检查非法偏移类型。

### 非法偏移类型

根据 PHP 文档（[`www.php.net/manual/en/language.types.array.php`](https://www.php.net/manual/en/language.types.array.php)），数组是键/值对的有序列表。数组键，也称为**索引**或**偏移**，可以是两种数据类型之一：`integer`或`string`。如果一个数组只包含`integer`键，通常被称为**数字数组**。另一方面，**关联数组**是一个术语，用于使用`string`索引。**非法偏移**是指数组键的数据类型不是`integer`或`string`的情况。

重要提示

有趣的是，以下代码片段不会生成`Warning`或`Error`：`$x = (float) 22/7; $arr[$x] = 'Value of Pi';`。在进行数组赋值之前，变量`$x`的值首先被转换为`integer`，截断任何小数部分。

举个例子，看看这段代码片段。请注意，最后一个数组元素的索引键是一个对象：

```php
// ch03/php8_warn_array_offset.php
$obj = new stdClass();
$b = ['A' => 1, 'B' => 2, $obj => 3];
var_dump($b);
```

在 PHP 7 下运行的输出产生了`Warning`的`var_dump()`输出，如下所示：

```php
PHP Warning:  Illegal offset type in /repo/ch03/php8_warn_array_offset.php on line 6
array(2) {
  'A' =>  int(1)
  'B' =>  int(2)
}
```

然而，在 PHP 8 中，`var_dump()`永远不会被执行，因为会抛出`TypeError`，如下所示：

```php
PHP Fatal error:  Uncaught TypeError: Illegal offset type in /repo/ch03/php8_warn_array_offset.php:6
```

使用`unset()`时，与非法数组偏移相同的原则存在，如下面的代码示例所示：

```php
// ch03/php8_warn_array_offset.php
$obj = new stdClass();
$b = ['A' => 1, 'B' => 2, 'C' => 3];
unset($b[$obj]);
var_dump($b);
```

在使用`empty()`或`isset()`中的非法偏移时，对数组索引键的更严格控制也可以看到，如下面的代码片段所示：

```php
// ch03/php8_warn_array_empty.php
$obj = new stdClass();
$obj->c = 'C';
$b = ['A' => 1, 'B' => 2, 'C' => 3];
$message =(empty($b[$obj])) ? 'NOT FOUND' : 'FOUND';
echo "$message\n";
```

在前面的两个代码示例中，在 PHP 7 及更早版本中，代码示例完成时会产生一个`Warning`，而在 PHP 8 中会抛出一个`Error`。除非捕获到这个`Error`，否则代码示例将无法完成。

提示

**最佳实践**：在初始化数组时，确保数组索引数据类型是`integer`或`string`。

接下来，我们来看一下字符串处理中的错误提升。

## 字符串处理中的提升警告

关于对象和数组的提升警告也适用于 PHP 8 字符串错误处理。在这一小节中，我们将检查两个字符串处理`Warning`提升为`Errors`，如下所示：

+   偏移不包含在字符串中

+   空字符串偏移

+   让我们首先检查不包含在字符串中的偏移。

### 偏移不包含在字符串中。

作为第一种情况的例子，看看下面的代码示例。在这里，我们首先将一个字符串分配给包含所有字母的字符串。然后，我们使用`strpos()`返回字母`Z`的位置，从偏移`0`开始。在下一行，我们做同样的事情；然而，偏移`27`超出了字符串的末尾：

```php
// /repo/ch03/php8_error_str_pos.php
$str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
echo $str[strpos($str, 'Z', 0)];
echo $str[strpos($str, 'Z', 27)];
```

在 PHP 7 中，如预期的那样，返回了`Z`的输出，`strpos()`产生了一个`Warning`，并且产生了一个`Notice`，说明进行了偏移转换（关于这一点，我们将在下一节中详细介绍）。以下是 PHP 7 的输出：

```php
Z
PHP Warning:  strpos(): Offset not contained in string in /repo/ch03/php8_error_str_pos.php on line 7
PHP Notice:  String offset cast occurred in /repo/ch03/php8_error_str_pos.php on line 7
```

然而，在 PHP 8 中，会抛出致命的`ValueError`，如下所示：

```php
Z
PHP Fatal error:  Uncaught ValueError: strpos(): Argument #3 ($offset) must be contained in argument #1 ($haystack) in /repo/ch03/php8_error_str_pos.php:7
```

在这种情况下我们需要传达的关键点是，以前允许这种糟糕的编码保留在一定程度上是可以接受的。然而，在进行 PHP 8 升级后，正如你可以清楚地从输出中看到的那样，你的代码将失败。现在，让我们来看一下空字符串偏移。

### 空字符串偏移错误处理

信不信由你，在 PHP 7 之前的版本中，开发人员可以通过将空值赋给目标偏移来从字符串中删除字符。例如，看看这段代码：

```php
// /repo/ch03/php8_error_str_empty.php
$str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
$str[5] = '';
echo $str . "\n";
```

这个代码示例的目的是从由`$str`表示的字符串中删除字母`F`。令人惊讶的是，在 PHP 5.6 中，你可以从这个截图中看到，尝试是完全成功的：

![图 3.1 - PHP 5.6 输出显示成功删除字符](img/Figure_3.1_B16992.jpg)

图 3.1 - PHP 5.6 输出显示成功删除字符

请注意，我们用来演示本书中的代码的虚拟环境允许访问 PHP 7.1 和 PHP 8。为了正确展示 PHP 5 的行为，我们挂载了一个 PHP 5.6 的 Docker 镜像，并对结果进行了截图。

然而，在 PHP 7 中，这种做法是被禁止的，并且会发出一个`Warning`，如下所示：

```php
PHP Warning:  Cannot assign an empty string to a string offset in /repo/ch03/php8_error_str_empty.php on line 5
ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

正如您从前面的输出中所看到的，脚本被允许执行；然而，尝试删除字母`F`是不成功的。在 PHP 8 中，正如我们所讨论的，`Warning`被提升为`Error`，整个脚本中止，如下所示：

```php
PHP Fatal error:  Uncaught Error: Cannot assign an empty string to a string offset in /repo/ch03/php8_error_str_empty.php:5
```

接下来，我们将研究在 PHP 8 中，以前的`Notices`被提升为`Warnings`的情况。

# 理解被提升为警告的通知

有许多情况被认为对 PHP 引擎在运行时的稳定性不太重要，在 PHP 7 之前的版本中被低估了。不幸的是，新的（或者可能是懒惰的！）PHP 开发人员通常会忽略`Notices`，以便匆忙将他们的代码投入生产。

多年来，PHP 标准已经大大收紧，这导致 PHP 核心团队将某些错误条件从`Notice`升级为`Warning`。任何错误报告级别都不会导致代码停止工作。然而，PHP 核心团队认为*Notice-to-Warning*的提升将使糟糕的编程实践变得更加明显。`Warnings`不太可能被忽视，最终会导致更好的代码。

以下是在早期版本的 PHP 中发出`Notice`的一些错误条件的简要列表，在 PHP 8 中，相同的条件现在会生成一个`Warning`：

+   尝试访问不存在的对象属性

+   尝试访问不存在的静态属性

+   尝试使用一个不存在的键来访问数组元素

+   错误地将资源用作数组偏移

+   模棱两可的字符串偏移转换

+   不存在或未初始化的字符串偏移

首先让我们来看一下涉及对象的`Notice`促销活动。

## 不存在的对象属性访问处理

在早期的 PHP 版本中，尝试访问不存在的属性时会发出一个`Notice`。唯一的例外是当它是一个自定义类，你在那里定义了魔术`__get()`和/或`__set()`方法。

在下面的代码示例中，我们定义了一个带有两个属性的`Test`类，其中一个被标记为`static`：

```php
// /repo/ch03/php8_warn_undef_prop.php
class Test {
    public static $stat = 'STATIC';
    public $exists = 'NORMAL';
}
$obj = new Test();
```

然后我们尝试`echo`存在和不存在的属性，如下所示：

```php
echo $obj->exists;
echo $obj->does_not_exist;
```

毫不奇怪，在 PHP 7 中，当尝试访问不存在的属性`echo`时，会返回一个`Notice`，如下所示：

```php
NORMAL
PHP Notice:  Undefined property: Test::$does_not_exist in
/repo/ch03/php8_warn_undef_prop.php on line 14
```

同样的代码文件，在 PHP 8 中，现在返回一个`Warning`，如下所示：

```php
NORMAL
PHP Warning:  Undefined property: Test::$does_not_exist in /repo/ch03/php8_warn_undef_prop.php on line 14
```

重要提示

`Test::$does_not_exist`错误消息并不意味着我们尝试了静态访问。它只是意味着`Test`类关联了一个`$does_not_exist`属性。

现在我们添加了尝试访问不存在的静态属性的代码行，如下所示：

```php
try {
    echo Test::$stat;
    echo Test::$does_not_exist;
} catch (Error $e) {
    echo __LINE__ . ':' . $e;
}
```

有趣的是，PHP 7 和 PHP 8 现在都会发出致命错误，如下所示：

```php
STATIC
22:Error: Access to undeclared static property Test::$does_not_exist in /repo/ch03/php8_warn_undef_prop.php:20
```

任何以前发出`Warning`的代码块现在发出`Error`都是值得关注的。如果可能的话，扫描你的代码，查找对静态类属性的静态引用，并确保它们被定义。否则，在 PHP 8 升级后，你的代码将失败。

现在让我们来看一下不存在的偏移处理。

## 不存在的偏移处理

如前一节所述，一般来说，在读取数据的地方，`Notices`已经被提升为`Warnings`，而在写入数据的地方，`Warnings`已经被提升为`Errors`（并且可能导致*丢失*数据）。不存在的偏移处理遵循这个逻辑。

在下面的例子中，一个数组键是从一个字符串中提取出来的。在这两种情况下，偏移量都不存在：

```php
// /repo/ch03/php8_warn_undef_array_key.php
$key  = 'ABCDEF';
$vals = ['A' => 111, 'B' => 222, 'C' => 333];
echo $vals[$key[6]];
```

在 PHP 7 中，结果是一个`Notice`，如下所示：

```php
PHP Notice:  Uninitialized string offset: 6 in /repo/ch03/php8_warn_undef_array_key.php on line 6
PHP Notice:  Undefined index:  in /repo/ch03/php8_warn_undef_array_key.php on line 6
```

在 PHP 8 中，结果是一个`Warning`，如下所示：

```php
PHP Warning:  Uninitialized string offset 6 in /repo/ch03/php8_warn_undef_array_key.php on line 6
PHP Warning:  Undefined array key "" in /repo/ch03/php8_warn_undef_array_key.php on line 6
```

这个例子进一步说明了 PHP 8 错误处理增强的一般原理：如果你的代码*写入*数据到一个不存在的偏移，以前是一个`警告`在 PHP 8 中是一个`错误`。前面的输出显示了在 PHP 8 中尝试*读取*不存在偏移的数据时，现在会发出一个`警告`。下一个要检查的`通知`提升涉及滥用资源 ID。

## 滥用资源 ID 作为数组偏移

当创建到应用程序代码外部的服务的连接时，会生成一个**资源**。这种数据类型的一个典型例子是文件句柄。在下面的代码示例中，我们打开了一个文件句柄（从而创建了`资源`）到一个`gettysburg.txt`文件：

```php
// /repo/ch03/php8_warn_resource_offset.php
$fn = __DIR__ . '/../sample_data/gettysburg.txt';
$fh = fopen($fn, 'r');
echo $fh . "\n";
```

请注意，我们在最后一行直接输出了`资源`。这显示了资源 ID 号。然而，如果我们现在尝试使用资源 ID 作为数组偏移，PHP 7 会生成一个`通知`，如下所示：

```php
Resource id #5
PHP Notice:  Resource ID#5 used as offset, casting to integer (5) in /repo/ch03/php8_warn_resource_offset.php on line 9
```

如预期的那样，PHP 8 生成了一个`警告`，如下所示：

```php
Resource id #5
PHP Warning:  Resource ID#5 used as offset, casting to integer (5) in /repo/ch03/php8_warn_resource_offset.php on line 9
```

请注意，在 PHP 8 中，许多以前生成`资源`的函数现在生成对象。这个主题在*第七章**，避免在使用 PHP 8 扩展时陷阱*中有所涉及。

提示

**最佳实践**：不要使用资源 ID 作为数组偏移！

现在我们将注意力转向与模糊的字符串偏移相关的`通知`在模糊的字符串偏移的情况下提升为`警告`。

## 模糊的字符串偏移转换

将注意力转向字符串处理，我们再次回顾使用数组语法在字符串中识别单个字符的想法。如果 PHP 必须执行内部类型转换以评估字符串偏移，但在这种类型转换中并不清楚，就可能发生**模糊的字符串偏移转换**。

在这个非常简单的例子中，我们定义了一个包含字母表中所有字母的字符串。然后我们用这些值定义了一个键的数组：`NULL`；一个布尔值，`TRUE`；和一个浮点数，`22/7`（*Pi*的近似值）。然后我们循环遍历这些键，并尝试将键用作字符串偏移，如下所示：

```php
// /repo/ch03/php8_warn_amb_offset.php
$str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
$ptr = [ NULL, TRUE, 22/7 ];
foreach ($ptr as $key) {
    var_dump($key);
    echo $str[$key];
}
```

正如你可能预料的那样，在 PHP 7 中运行的输出产生了输出`A`，`B`和`D`，以及一系列的`通知`，如下所示：

```php
NULL
PHP Notice:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
A
/repo/ch03/php8_warn_amb_offset.php:7:
bool(true)
PHP Notice:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
B
/repo/ch03/php8_warn_amb_offset.php:7:
double(3.1428571428571)
PHP Notice:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
D
```

PHP 8 始终产生相同的结果，但在这里，一个`警告`取代了`通知`：

```php
NULL
PHP Warning:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
A
bool(true)
PHP Warning:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
B
float(3.142857142857143)
PHP Warning:  String offset cast occurred in /repo/ch03/php8_warn_amb_offset.php on line 8
D
```

现在让我们来看看不存在偏移的处理。

## 未初始化或不存在的字符串偏移

这种类型的错误旨在捕获使用偏移访问字符串的情况，其中偏移超出了边界。下面是一个非常简单的代码示例，说明了这种情况：

```php
// /repo/ch03/php8_warn_un_init_offset.php
$str = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
echo $str[27];
```

在 PHP 7 中运行这段代码会产生一个`通知`。以下是 PHP 7 的输出：

```php
PHP Notice:  Uninitialized string offset: 27 in /repo/ch03/php8_warn_un_init_offset.php on line 5
```

可以预见的是，PHP 8 的输出产生了一个`警告`，如下所示：

```php
PHP Warning:  Uninitialized string offset 27 in /repo/ch03/php8_warn_un_init_offset.php on line 5
```

本节中的所有示例都证实了 PHP 8 朝着强制执行最佳编码实践的一般趋势。

提示

有关推广的“通知”和“警告”的更多信息，请查看这篇文章：[`wiki.php.net/rfc/engine_warnings`](https://wiki.php.net/rfc/engine_warnings)。

现在，我们将注意力转向（臭名昭著的）`@`警告抑制器。

# 处理@错误控制运算符

多年来，许多 PHP 开发人员一直使用`@`**错误控制运算符**来掩盖错误。当使用编写不良的 PHP 库时，这一点尤为真实。不幸的是，这种用法的净效果只会传播糟糕的代码！

许多 PHP 开发人员都在进行“一厢情愿的思考”，他们认为当他们使用`@`运算符来阻止错误显示时，问题似乎神奇地消失了！相信我，当我说这个时候：*并没有！*在这一部分，我们首先研究了`@`运算符的传统用法，之后我们研究了 PHP 8 中的`@`运算符的变化。

提示

有关传统`@`操作符的语法和用法的更多信息，请参阅此文档参考页面：[`www.php.net/manual/en/language.operators.errorcontrol.php`](https://www.php.net/manual/en/language.operators.errorcontrol.php)。

## @操作符用法

在呈现代码示例之前，再次强调非常重要的一点是我们**不**推广使用这种机制！相反，你应该在任何情况下避免使用它。如果出现错误消息，最好的解决方案是*修复错误*，而不是将其消音！

在下面的代码示例中，定义了两个函数。`bad()`函数故意触发错误。`worse()`函数包含一个文件，其中存在解析错误。请注意，当调用这些函数时，`@`符号在函数名之前，导致错误输出被抑制：

```php
// /repo/ch03/php8_at_silencer.php
function bad() {
    trigger_error(__FUNCTION__, E_USER_ERROR);
}
function worse() {
    return include __DIR__ .  '/includes/
                               causes_parse_error.php';
}
echo @bad();
echo @worse();
echo "\nLast Line\n";
```

在 PHP 7 中，根本没有输出，如下所示：

```php
root@php8_tips_php7 [ /repo/ch03 ]# php php8_at_silencer.php 
root@php8_tips_php7 [ /repo/ch03 ]# 
```

有趣的是，在 PHP 7 中程序实际上是不允许继续执行的：我们从未看到`Last Line`的输出。这是因为，尽管被掩盖了，但仍然生成了一个致命错误，导致程序失败。然而，在 PHP 8 中，致命错误没有被掩盖，如下所示：

```php
root@php8_tips_php8 [ /repo/ch03 ]# php8 php8_at_silencer.php 
PHP Fatal error:  bad in /repo/ch03/php8_at_silencer.php on line 5
```

现在让我们来看一下 PHP 8 中关于`@`操作符的另一个不同之处。

## @操作符和 error_reporting()

`error_reporting()`函数通常用于覆盖`php.ini`文件中设置的`error_reporting`指令。然而，这个函数的另一个用途是返回最新的错误代码。然而，在 PHP 8 之前的版本中存在一个奇怪的例外，即如果使用了`@`操作符，`error_reporting()`返回值为`0`。

在下面的代码示例中，我们定义了一个错误处理程序，当它被调用时报告接收到的错误编号和字符串。此外，我们还显示了`error_reporting()`返回的值：

```php
// /repo/ch03/php8_at_silencer_err_rep.php
function handler(int $errno , string $errstr) {
    $report = error_reporting();
    echo 'Error Reporting : ' . $report . "\n";
    echo 'Error Number    : ' . $errno . "\n";
    echo 'Error String    : ' . $errstr . "\n";
    if (error_reporting() == 0) {
        echo "IF statement works!\n";
    }
}
```

与以前一样，我们定义了一个`bad()`函数，故意触发错误，然后使用`@`操作符调用该函数，如下所示：

```php
function bad() {
    trigger_error('We Be Bad', E_USER_ERROR);
}
set_error_handler('handler');
echo @bad();
```

在 PHP 7 中，你会注意到`error_reporting()`返回`0`，因此导致`IF statement works!`出现在输出中，如下所示：

```php
root@root@php8_tips_php7 [ /repo/ch03 ] #
php php8_at_silencer_err_rep.php
Error Reporting : 0
Error Number    : 256
Error String    : We Be Bad
IF statement works!
```

另一方面，在 PHP 8 中运行，`error_reporting()`返回最后一个错误的值——在这种情况下是`4437`。当然，`if()`表达式失败，导致没有额外的输出。以下是在 PHP 8 中运行相同代码的结果：

```php
root@php8_tips_php8 [ /repo/ch03 ] #
php php8_at_silencer_err_rep.php
Error Reporting : 4437
Error Number    : 256
Error String    : We Be Bad
```

这结束了对 PHP 8 中`@`操作符用法的考虑。

提示

**最佳实践**：不要使用`@`错误控制操作符！`@`操作符的目的是抑制错误消息的显示，但你需要考虑为什么这个错误消息首先出现。通过使用`@`操作符，你只是避免提供问题的解决方案！

# 总结

在本章中，你了解了 PHP 8 中错误处理的重大变化概述。你还看到了可能出现错误条件的情况示例，并且现在知道如何正确地管理 PHP 8 中的错误。你现在有了一个坚实的路径，可以重构在 PHP 8 下现在产生错误的代码。如果你的代码可能导致任何前述的条件，其中以前的“警告”现在是“错误”，你就有可能使你的代码崩溃。

同样地，虽然过去描述的第二组错误条件只会产生“通知”，但现在这些相同的条件会引发“警告”。新的一组“警告”给了你一个机会来调整错误的代码，防止你的应用程序陷入严重的不稳定状态。

最后，你学会了强烈不推荐使用`@`操作符。在 PHP 8 中，这种语法将不再掩盖致命错误。在下一章中，你将学习如何在 PHP 8 中创建 C 语言结构并直接调用 C 语言函数。
