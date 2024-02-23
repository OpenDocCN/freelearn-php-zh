# 第三章：*第二章*：学习 PHP 8 的功能增强

本章将带您了解在程序级别引入的**PHP 8**的重要增强和改进。使用的代码示例展示了新的 PHP 8 功能和技术，以便促进程序化编程。

掌握本章中新函数和技术的使用将帮助您编写更快、更干净的应用程序。尽管本章重点介绍命令和函数，但在开发类方法时，所有这些技术也很有用。

本章涵盖以下主题：

+   使用新的 PHP 8 操作符

+   使用箭头函数

+   理解统一变量语法

+   学习新的数组和字符串处理技术

+   使用 authorizer 保护 SQLite 数据库

# 技术要求

要检查和运行本章提供的代码示例，以下是最低推荐的硬件要求：

+   基于 x86_64 的台式机或笔记本电脑

+   1 千兆字节（GB）的可用磁盘空间

+   4 GB 的随机存取存储器（RAM）

+   每秒 500 千位（Kbps）或更快的互联网连接

+   另外，您需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 安装的更多信息，请参阅*第一章*的*技术要求*部分，介绍了如何构建用于演示本书中代码的 Docker 容器。在整个过程中，我们将参考您恢复本书示例代码的目录为`/repo`。

本章的源代码位于此处：[`github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices`](https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices)。

我们现在可以开始讨论新的 PHP 8 操作符了。

# 使用新的 PHP 8 操作符

PHP 8 引入了许多新的**操作符**。此外，PHP 8 通常引入了一种统一和一致的方式来使用这些操作符。在本节中，我们将讨论以下操作符：

+   variadics 操作符

+   Nullsafe 操作符

+   连接操作符

+   三元操作符

让我们从讨论 variadics 操作符开始。

## 使用 variadics 操作符

**variadics**操作符由三个前导点（`...`）组成，位于普通 PHP 变量（或对象属性）之前。这个操作符实际上从 PHP 5.6 版本开始就存在了。它也被称为以下内容：

+   Splat 操作符

+   散列操作符

+   扩展操作符

在我们深入研究 PHP 8 使用这个操作符的改进之前，让我们快速看一下这个操作符通常的用法。

### 未知数量的参数

variadics 操作符最常见的用途之一是在定义具有未知数量参数的函数的情况下。

在以下代码示例中，`multiVardump()`函数能够接受任意数量的变量。然后连接`var_export()`的输出并返回一个字符串：

```php
// /repo/ch02/php7_variadic_params.php
function multiVardump(...$args) {
    $output = '';
    foreach ($args as $var)
        $output .= var_export($var, TRUE);
    return $output;
}
$a = new ArrayIterator(range('A','F'));
$b = function (string $val) { return str_rot13($val); };
$c = [1,2,3];
$d = 'TEST';
echo multiVardump($a, $b, $c);
echo multiVardump($d);
```

第一次调用函数时，我们提供了三个参数。第二次调用时，我们只提供了一个参数。由于我们使用了 variadics 操作符，所以无需重写函数来适应更多或更少的参数。

提示

有一个`func_get_args()` PHP 函数，可以将所有函数参数收集到一个数组中。但是，variadics 操作符更受青睐，因为它必须在函数签名中声明，从而使程序开发人员的意图更加清晰。更多信息，请参阅[`php.net/func_get_args`](https://php.net/func_get_args)。

### 吸入剩余参数

variadics 操作符的另一个用途是**吸入**任何剩余参数。这种技术允许您将强制参数与未知数量的可选参数混合使用。

在这个例子中，`where()`函数生成一个要添加到**结构化查询语言**（**SQL**）`SELECT`语句中的`WHERE`子句。前两个参数是必需的：没有理由生成没有参数的`WHERE`子句！看一下这里的代码：

```php
// ch02/includes/php7_sql_lib.php
// other functions not shown
function where(stdClass $obj, $a, $b = '', $c = '', 
        $d = '') {
    $obj->where[] = $a;
    $obj->where[] = $b;
    $obj->where[] = $c;
    $obj->where[] = $d;
}
```

使用此函数的调用代码可能如下所示：

```php
// /repo/ch02/php7_variadics_sql.php
require_once __DIR__ . '/includes/php7_sql_lib.php';
$start = '2021-01-01';
$end   = '2021-04-01';
$select = new stdClass();
from($select, 'events');
cols($select, ['id', 'event_key', 
    'event_name', 'event_date']);
limit($select, 10);
where($select, 'event_date', '>=', "'$start'");
where($select, 'AND');
where($select, 'event_date', '<', "'$end'");
$sql = render($select);
// remaining code not shown
```

您可能已经注意到，由于参数数量有限，必须多次调用`where()`。这是可变参数运算符的一个完美应用场景！以下是重写的`where()`函数可能会看起来：

```php
// ch02/includes/php8_sql_lib.php
// other functions not shown
function where(stdClass $obj, ...$args) {
    $obj->where = (empty($obj->where))
                ? $args
                : array_merge($obj->where, $args);
}
```

因为`...$args`始终作为数组返回，为了确保对函数的任何额外调用不会丢失子句，我们需要执行一个`array_merge()`操作。以下是重写的调用程序：

```php
// /repo/ch02/php8_variadics_sql.php
require_once __DIR__ . '/includes/sql_lib2.php';
$start = '2021-01-01';
$end   = '2021-04-01';
$select = new stdClass();
from($select, 'events');
cols($select, ['id', 'event_key', 
    'event_name', 'event_date']);
limit($select, 10);
where($select, 'event_date', '>=', "'$start'", 
    'AND', 'event_date', '<', "'$end'");
$sql = render($select);
// remaining code not shown
```

生成的 SQL 语句如下所示：

```php
SELECT id,event_key,event_name,event_date 
FROM events 
WHERE event_date >= '2021-01-01' 
    AND event_date <= '2021-04-01' 
LIMIT 10
```

前面的输出显示了我们的 SQL 生成逻辑生成了一个有效的语句。

### 使用可变参数运算符作为替代

到目前为止，对于有经验的 PHP 开发人员来说，这些都不是陌生的。在 PHP 8 中的不同之处在于，可变参数运算符现在可以在可能涉及*扩展*的情况下使用。

为了正确描述可变参数运算符的使用方式的不同之处，我们需要简要回顾一下**面向对象编程**（**OOP**）。如果我们将刚才描述的`where()`函数重写为类方法，它可能会像这样：

```php
// src/Php7/Sql/Where.php
namespace Php7\Sql;
class Where {
    public $where = [];
    public function where($a, $b = '', $c = '', $d = '') {
        $this->where[] = $a;
        $this->where[] = $b;
        $this->where[] = $c;
        $this->where[] = $d;
        return $this;
    }
    // other code not shown
}
```

现在，假设我们有一个`Select`类，它扩展了`Where`，但使用可变参数运算符重新定义了方法签名。它可能如下所示：

```php
// src/Php7/Sql/Select.php
namespace Php7\Sql;
class Select extends Where {
    public function where(...$args)    {
        $this->where = (empty($obj->where))
                    ? $args
                    : array_merge($obj->where, $args);
    }
    // other code not shown
}
```

使用可变参数运算符是合理的，因为提供给`WHERE`子句的参数数量是未知的。以下是使用面向对象编程重写的调用程序：

```php
// /repo/ch02/php7_variadics_problem.php
require_once __DIR__ . '/../src/Server/Autoload/Loader.php'
$loader = new \Server\Autoload\Loader();
use Php7\Sql\Select;
$start = "'2021-01-01'";
$end   = "'2021-04-01'";
$select = new Select();
$select->from($select, 'events')
       ->cols($select, ['id', 'event_key', 
              'event_name', 'event_date'])
       ->limit($select, 10)
       ->where($select, 'event_date', '>=', "'$start'",
               'AND', 'event_date', '<=', "'$end'");
$sql = $select->render();
// other code not shown
```

然而，当您尝试在 PHP 7 下运行此示例时，会出现以下警告：

```php
Warning: Declaration of Php7\Sql\Select::where(...$args) should be compatible with Php7\Sql\Where::where($a, $b = '', $c = '', $d = '') in /repo/src/Php7/Sql/Select.php on line 5 
```

请注意，代码仍然有效；但是，PHP 7 不认为可变参数运算符是一个可行的替代方案。以下是在 PHP 8 下运行相同代码的情况（使用`/repo/ch02/php8_variadics_no_problem.php`）：

![图 2.1-可接受扩展类中的可变参数运算符](img/Figure_2.1_B16992.jpg)

图 2.1-可接受扩展类中的可变参数运算符

提示

以下是两个 PHP 文档引用，解释了 PHP 可变参数运算符背后的原因：

[`wiki.php.net/rfc/variadics`](https://wiki.php.net/rfc/variadics)

[`wiki.php.net/rfc/argument_unpacking`](https://wiki.php.net/rfc/argument_unpacking)

现在让我们来看看 nullsafe 运算符。

## 使用 nullsafe 运算符

nullsafe 运算符用于对象属性引用链。如果链中的某个属性不存在（换句话说，它被视为`NULL`），该运算符会安全地返回一个`NULL`值，而不会发出警告。

举个例子，假设我们有以下**扩展标记语言**（**XML**）文件：

```php
<?xml version='1.0' standalone='yes'?>
<produce>
      <file>/repo/ch02/includes/produce.xml</file>
    <dept>
        <fruit>
            <apple>11</apple>
            <banana>22</banana>
            <cherry>33</cherry>
        </fruit>
        <vegetable>
            <artichoke>11</artichoke>
            <beans>22</beans>
            <cabbage>33</cabbage>
        </vegetable>
    </dept>
</produce>
```

以下是一个扫描 XML 文档并显示数量的代码片段：

```php
// /repo/ch02/php7_nullsafe_xml.php
$xml = simplexml_load_file(__DIR__ . 
        '/includes/produce.xml');
$produce = [
    'fruit' => ['apple','banana','cherry','pear'],
    'vegetable' => ['artichoke','beans','cabbage','squash']
];
$pattern = "%10s : %d\n";
foreach ($produce as $type => $items) {
    echo ucfirst($type) . ":\n";
    foreach ($items as $item) {
        $qty = getQuantity($xml, $type, $item);
        printf($pattern, $item, $qty);
    }
}
```

我们还需要定义一个`getQuantity()`函数，首先检查该属性是否不为空，然后再进行下一级的操作，如下所示：

```php
function getQuantity(SimpleXMLElement $xml, 
        string $type, string $item {
    $qty = 0;
    if (!empty($xml->dept)) {
        if (!empty($xml->dept->$type)) {
            if (!empty($xml->dept->$type->$item)) {
                $qty = $xml->dept->$type->$item;
            }
        }
    }
    return $qty;
}
```

当您开始处理更深层次的嵌套级别时，需要检查属性是否存在的函数变得更加复杂。这正是 nullsafe 运算符可以发挥作用的地方。

看一下相同的程序代码，但不需要`getQuantity()`函数，如下所示：

```php
// /repo/ch02/php8_nullsafe_xml.php
$xml = simplexml_load_file(__DIR__ . 
        '/includes/produce.xml'
$produce = [
    'fruit' => ['apple','banana','cherry','pear']
    'vegetable' => ['artichoke','beans','cabbage','squash']
];
$pattern = "%10s : %d\n";
foreach ($produce as $type => $items) {
    echo ucfirst($type) . ":\n";
    foreach ($items as $item) {
        printf($pattern, $item, 
            $xml?->dept?->$type?->$item);
    }
}
```

现在让我们来看看 nullsafe 运算符的另一个用途。

### 使用 nullsafe 运算符来短路链

nullsafe 运算符在连接的操作链中也很有用，包括对对象属性的引用、数组元素方法调用和静态引用。

举个例子，这里有一个配置文件，返回一个匿名类。它定义了根据文件类型提取数据的不同方法：

```php
// ch02/includes/nullsafe_config.php
return new class() {
    const HEADERS = ['Name','Amt','Age','ISO','Company'];
    const PATTERN = "%20s | %16s | %3s | %3s | %s\n";
    public function json($fn) {
        $json = file_get_contents($fn);
        return json_decode($json, TRUE);
    }
    public function csv($fn) {
        $arr = [];
        $fh = new SplFileObject($fn, 'r');
        while ($node = $fh->fgetcsv()) $arr[] = $node;
        return $arr;            
    }
    public function txt($fn) {
        $arr = [];
        $fh = new SplFileObject($fn, 'r');
        while ($node = $fh->fgets())
            $arr[] = explode("\t", $node);
        return $arr;
    }
    // all code not shown
};
```

该类还包括一个显示数据的方法，如下面的代码片段所示：

```php
    public function display(array $data) {
        $total  = 0;
        vprintf(self::PATTERN, self::HEADERS);
        foreach ($data as $row) {
            $total += $row[1];
            $row[1] = number_format($row[1], 0);
            $row[2] = (string) $row[2];
            vprintf(self::PATTERN, $row);
        }
        echo 'Combined Wealth: ' 
            . number_format($total, 0) . "\n"
    }    
```

在调用程序中，为了安全地执行 `display()` 方法，我们需要在执行回调之前添加一个 `is_object()` 的额外安全检查，以及 `method_exists()`，如下面的代码片段所示：

```php
// /repo/ch02/php7_nullsafe_short.php
$config  = include __DIR__ . 
        '/includes/nullsafe_config.php';
$allowed = ['csv' => 'csv','json' => 'json','txt'
                  => 'txt'];
$format  = $_GET['format'] ?? 'txt';
$ext     = $allowed[$format] ?? 'txt';
$fn      = __DIR__ . '/includes/nullsafe_data.' . $ext;
if (file_exists($fn)) {
    if (is_object($config)) {
        if (method_exists($config, 'display')) {
            if (method_exists($config, $ext)) {
                $config->display($config->$ext($fn));
            }
        }
    }
}
```

与前面的例子一样，空安全运算符可以用来确认 `$config` 是否为对象。通过简单地在第一个对象引用中使用空安全运算符，如果对象或方法不存在，运算符将 *短路* 整个链并返回 `NULL`。

以下是使用 PHP 8 空安全运算符重写的代码：

```php
// /repo/ch02/php8_nullsafe_short.php
$config  = include __DIR__ . 
        '/includes/nullsafe_config.php';
$allowed = ['csv' => 'csv','json' => 'json',
                     'txt' => 'txt'];
$format  = $_GET['format'] ?? $argv[1] ?? 'txt';
$ext     = $allowed[$format] ?? 'txt';
$fn      = __DIR__ . '/includes/nullsafe_data.' . $ext;
if (file_exists($fn)) {
    $config?->display($config->$ext($fn));
}
```

如果 `$config` 返回为 `NULL`，则整个操作链将被取消，不会生成任何警告或通知，并且返回值（如果有）为 `NULL`。最终结果是我们省去了编写三个额外的 `if()` 语句！

提示

有关使用此运算符时的其他注意事项，请查看这里：[`wiki.php.net/rfc/nullsafe_operator`](https://wiki.php.net/rfc/nullsafe_operator)。

重要提示

为了将格式参数传递给示例代码文件，您需要从浏览器中运行以下代码：`http://localhost:8888/ch02/php7_nullsafe_short.php?format=json`。

接下来，我们将看看连接运算符的更改。

## 连接运算符已经被降级

尽管 **连接** 运算符的精确用法（例如，句号（.）在 PHP 8 中没有改变，但在其 **优先级顺序** 中发生了极其重要的变化。在早期版本的 PHP 中，连接运算符在优先级方面被认为与较低级别的算术运算符加号（`+`）和减号（`-`）相等。接下来，让我们看看传统优先级顺序可能出现的问题：令人费解的结果。

### 处理令人费解的结果

不幸的是，这种安排会产生意想不到的结果。以下代码片段在使用 PHP 7 时执行时呈现出令人费解的输出：

```php
// /repo/ch02/php7_ops_concat_1.php
$a = 11;
$b = 22;
echo "Sum: " . $a + $b;
```

仅仅看代码，您可能期望输出类似于 `"Sum:33"`。但事实并非如此！在 PHP 7.1 上运行时，请查看以下输出：

```php
root@php8_tips_php7 [ /repo/ch02 ]# php php7_ops_concat_1.php
PHP Warning:  A non-numeric value encountered in /repo/ch02/php7_ops_concat_1.php on line 5
PHP Stack trace:
PHP   1\. {main}() /repo/ch02/php7_ops_concat_1.php:0
Warning: A non-numeric value encountered in /repo/ch02/php7_ops_concat_1.php on line 5
Call Stack:
  0.0001     345896   1\. {main}()
22
```

此时，您可能会想，*因为代码从不说谎*，那么 `11` + `22` 的和为 `22`，正如我们在前面的输出（最后一行）中看到的那样？

答案涉及优先级顺序：从 PHP 7 开始，它始终是从左到右。因此，如果我们使用括号来使操作顺序更清晰，实际发生的情况是这样的：

`echo ("Sum: " . $a) + $b;`

`11` 被连接到 `"Sum: "`，结果为 `"Sum: 11"`。作为字符串。然后将字符串转换为整数，得到 `0` + `22` 表达式，这给我们了结果。

如果您在 PHP 8 中运行相同的代码，请注意这里的区别：

```php
root@php8_tips_php8 [ /repo/ch02 ]# php php8_ops_concat_1.php 
Sum: 33
```

正如您所看到的，算术运算符优先于连接运算符。使用括号，这实际上是 PHP 8 中代码的处理方式：

`echo "Sum: " . ($a + $b);`

提示

**最佳实践**：使用括号来避免依赖优先级顺序而产生的复杂性。有关降低连接运算符优先级背后的原因的更多信息，请查看这里：[`wiki.php.net/rfc/concatenation_precedence`](https://wiki.php.net/rfc/concatenation_precedence)。

现在我们将注意力转向三元运算符。

## 使用嵌套的三元运算符

**三元运算符** 对于 PHP 语言来说并不新鲜。然而，在 PHP 8 中，它们的解释方式有一个重大的不同。这种变化与该运算符的传统 **左关联行为** 有关。为了说明这一点，让我们看一个简单的例子，如下所示：

1.  在这个例子中，假设我们正在使用 `RecursiveDirectoryIterator` 类与 `RecursiveIteratorIterator` 类结合扫描目录结构。起始代码可能如下所示：

```php
// /repo/ch02/php7_nested_ternary.php
$path = realpath(__DIR__ . '/..');
$searchPath = '/ch';
$searchExt  = 'php';
$dirIter    = new RecursiveDirectoryIterator($path);
$itIter     = new RecursiveIteratorIterator($dirIter);
```

1.  然后我们定义一个函数，匹配包含`$searchPath`搜索路径并以`$searchExt`扩展名结尾的文件，如下所示：

```php
function find_using_if($iter, $searchPath, $searchExt) {
    $matching  = [];
    $non_match = [];
    $discard   = [];
    foreach ($iter as $name => $obj) {
        if (!$obj->isFile()) {
            $discard[] = $name;
        } elseif (!strpos($name, $searchPath)) {
            $discard[] = $name;
        } elseif ($obj->getExtension() !== $searchExt) {
            $non_match[] = $name;
        } else {
            $matching[] = $name;
        }
    }
    show($matching, $non_match);
}
```

1.  然而，一些开发人员可能会诱惑重构此函数，而不是使用`if / elseif / else`，而是使用嵌套三元运算符。以下是在前一步骤中使用的相同代码可能的样子：

```php
function find_using_tern($iter, $searchPath, 
        $searchExt){
    $matching  = [];
    $non_match = [];
    $discard   = [];
    foreach ($iter as $name => $obj) {
        $match = !$obj->isFile()
            ? $discard[] = $name
            : !strpos($name, $searchPath)
                ? $discard[] = $name
                : $obj->getExtension() !== $searchExt
                    ? $non_match[] = $name
                    : $matching[] = $name;
    }
    show($matching, $non_match);
}
```

两个函数的输出在 PHP 7 中产生相同的结果，如下截图所示：

![图 2.2 - 使用 PHP 7 进行嵌套三元输出](img/Figure_2.2_B16992.jpg)

图 2.2 - 使用 PHP 7 进行嵌套三元输出

然而，在 PHP 8 中，不再允许使用没有括号的嵌套三元操作。运行相同代码块时的输出如下：

![图 2.3 - 使用 PHP 8 进行嵌套三元输出](img/Figure_2.3_B16992.jpg)

图 2.3 - 使用 PHP 8 进行嵌套三元输出

提示

**最佳实践**：使用括号避免嵌套三元操作的问题。有关三元运算符嵌套差异的更多信息，请参阅此文章：[`wiki.php.net/rfc/ternary_associativity`](https://wiki.php.net/rfc/ternary_associativity)。

您现在对新的 nullsafe 运算符有了一个概念。您还学习了三个现有运算符——可变参数、连接和三元运算符——它们的功能略有修改。您现在可以避免升级到 PHP 8 时可能出现的潜在危险。现在让我们来看看另一个新功能，*箭头函数*。

# 使用箭头函数

**箭头函数**实际上是在 PHP 7.4 中首次引入的。然而，由于许多开发人员并不关注每个发布更新，因此在本书中包含这一出色的新功能是很重要的。

在本节中，您将了解箭头函数及其语法，以及与匿名函数相比的优缺点。

## 通用语法

箭头函数是传统匿名函数的简写语法，就像三元运算符是`if(){} else{}`的简写语法一样。箭头函数的通用语法如下：

`fn(<ARGS>) => <EXPRESSION>`

`<ARGS>`是可选的，包括任何其他用户定义的 PHP 函数中看到的内容。`<EXPRESSION>`可以包括任何标准的 PHP 表达式，如函数调用、算术运算等。

现在让我们来看看箭头函数和匿名函数之间的区别。

## 箭头函数与匿名函数

在本小节中，您将学习**箭头函数**和**匿名函数**之间的区别。为了成为一个有效的 PHP 8 开发人员，了解箭头函数何时何地可能取代匿名函数并提高代码性能是很重要的。

在进入箭头函数之前，让我们看一个简单的匿名函数。在下面的示例中，分配给`$addOld`的匿名函数产生了两个参数的和：

```php
// /repo/ch02/php8_arrow_func_1.php
$addOld = function ($a, $b) { return $a + $b; };
```

在 PHP 8 中，您可以产生完全相同的结果，如下所示：

`$addNew = fn($a, $b) => $a + $b;`

尽管代码更易读，但这一新功能有其优点和缺点，总结如下表所示：

![表 2.1 - 匿名函数与箭头函数](img/Table_2.1_B16992.jpg)

表 2.1 - 匿名函数与箭头函数

从上表中可以看出，箭头函数比匿名函数更高效。然而，缺乏间接性和不支持多行意味着您仍然需要偶尔使用匿名函数。

## 变量继承

匿名函数，就像任何标准的 PHP 函数一样，只有在将值作为参数传递、使用全局关键字或添加`use()`修饰符时，才能识别其范围外的变量。

以下是一个`DateTime`实例通过`use()`方式继承到匿名函数中的示例：

```php
// /repo/ch02/php8_arrow_func_2.php
// not all code shown
$old = function ($today) use ($format) {
    return $today->format($format);
};
```

这里使用箭头函数完全相同的东西：

`$new = fn($today) => $today->format($format);`

正如您所看到的，语法非常易读和简洁。现在让我们来看一个结合箭头函数的实际例子。

## 实际例子：使用箭头函数

回到生成难以阅读的 CAPTCHA 的想法（首次在*第一章*中介绍，*介绍新的 PHP 8 OOP 功能*），让我们看看如何结合箭头函数可能提高效率并减少所需的编码量。现在我们来看一个生成基于文本的 CAPTCHA 的脚本，如下所示：

1.  首先，我们定义一个生成由字母、数字和特殊字符随机选择组成的字符串的函数。请注意在以下代码片段中，使用了新的 PHP 8 `match`表达式结合箭头函数（高亮显示）：

```php
// /repo/ch02/php8_arrow_func_3.php
function genKey(int $size) {
    $alpha1  = range('A','Z');
    $alpha2  = range('a','z');
    $special = '!@#$%^&*()_+,./[]{}|=-';
    $len     = strlen($special) - 1;
    $numeric = range(0, 9);
    $text    = '';
    for ($x = 0; $x < $size; $x++) {
        $algo = rand(1,4);
        $func = match ($algo) {
            1 => fn() => $alpha1[array_rand($alpha1)],
            2 => fn() => $alpha2[array_rand($alpha2)]
            3 => fn() => $special[rand(0,$len)],
            4 => fn() => 
                       $numeric[array_rand($numeric)],
            default => fn() => ' '
        };
        $text .= $func();            
    }
    return $text;
}
```

1.  然后，我们定义一个`textCaptcha()`函数来生成文本 CAPTCHA。我们首先定义代表算法和颜色的两个数组。然后对它们进行*洗牌*以进一步随机化。我们还定义**超文本标记语言**（**HTML**）`<span>`元素来产生大写和小写字符，如下面的代码片段所示：

```php
function textCaptcha(string $text) {
    $algos = ['upper','lower','bold',
              'italics','large','small'];
    $color = ['#EAA8A8','#B0F6B0','#F5F596',
              '#E5E5E5','white','white'];
    $lgSpan = '<span style="font-size:32pt;">';
    $smSpan = '<span style="font-size:8pt;">';
    shuffle($algos);
    shuffle($color);
```

1.  接下来，我们定义一系列`InfiniteIterator`实例。这是一个有用的**标准 PHP 库**（**SPL**）类，允许您继续调用`next()`，而无需检查您是否已经到达迭代的末尾。这个迭代器类的作用是自动将指针移回数组的顶部，允许您无限迭代。代码可以在以下片段中看到：

```php
    $bkgTmp = new ArrayIterator($color);
    $bkgIter = new InfiniteIterator($bkgTmp);
    $algoTmp = new ArrayIterator($algos);
    $algoIter = new InfiniteIterator($algoTmp);
    $len = strlen($text);
```

1.  然后，我们逐个字符构建文本 CAPTCHA，应用适当的算法和背景颜色，如下所示：

```php
    $captcha = '';
    for ($x = 0; $x < $len; $x++) {
        $char = $text[$x];
        $bkg  = $bkgIter->current();
        $algo = $algoIter->current();
        $func = match ($algo) {
            'upper'   => fn() => strtoupper($char),
            'lower'   => fn() => strtolower($char),
            'bold'    => fn() => "<b>$char</b>",
            'italics' => fn() => "<i>$char</i>",
            'large'   => fn() => $lgSpan 
                         . $char . '</span>',
            'small'   => fn() => $smSpan 
                         . $char . '</span>',
            default   => fn() => $char
        };
        $captcha .= '<span style="background-color:' 
            . $bkg . ';">' 
            . $func() . '</span>';
        $algoIter->next();
        $bkgIter->next();
    }
    return $captcha;
}
```

再次注意混合使用`match`和`arrow`函数以实现期望的结果。

脚本的其余部分只是调用这两个函数，如下所示：

```php
$text = genKey(8);
echo "Original: $text<br />\n";
echo 'Captcha : ' . textCaptcha($text) . "\n";
```

以下是从浏览器中`/repo/ch02/php8_arrow_func_3.php`输出的样子：

![图 2.4 - 来自 php8_arrow_func_3.php 的输出](img/Figure_2.4_B16992.jpg)

图 2.4 - 来自 php8_arrow_func_3.php 的输出

提示

有关箭头函数的更多背景信息，请查看这里：[`wiki.php.net/rfc/arrow_functions_v2`](https://wiki.php.net/rfc/arrow_functions_v2)。

有关`InfiniteIterator`的信息，请查看 PHP 文档：[`www.php.net/InfiniteIterator`](https://www.php.net/InfiniteIterator)。

现在让我们来看一下*统一变量语法*。

# 理解统一变量语法

PHP 7.0 中引入的最激进的举措之一是努力规范化 PHP 语法。早期版本的 PHP 存在的问题是，在某些情况下，操作是从左到右解析的，而在其他情况下是从右到左解析的。这种不一致性是许多编程漏洞和困难的根本原因。因此，PHP 核心开发团队发起了一项名为**统一变量语法**的举措。但首先，让我们定义形成统一变量语法举措的关键要点。

## 定义统一变量语法

统一变量语法既不是协议也不是正式的语言构造。相反，它是一个指导原则，旨在确保所有操作以统一和一致的方式执行。

以下是这项举措的一些关键要点：

+   变量的顺序和引用的统一性

+   函数调用的统一性

+   解决数组解引用问题

+   提供在单个命令中混合函数调用和数组解引用的能力

提示

有关 PHP 7 统一变量语法的原始提案的更多信息，请查看这里：[`wiki.php.net/rfc/uniform_variable_syntax`](https://wiki.php.net/rfc/uniform_variable_syntax)。

现在让我们来看一下统一变量语法举措如何影响 PHP 8。

## 统一变量语法如何影响 PHP 8？

统一变量语法倡议在所有 PHP 7 的版本中都取得了极大的成功，过渡相对顺利。然而，有一些领域没有升级到这个标准。因此，提出了一个新的提案来解决这些问题。在 PHP 8 中，以下内容已经实现了统一性：

+   解引用插入字符串

+   魔术常量的不一致解引用

+   类常量解引用的一致性

+   增强了`new`和`instanceof`的表达式支持

在进入每个这些领域的示例之前，我们必须首先定义*解引用*的含义。

### 定义解引用

**解引用**是提取数组元素或对象属性的值的过程。它还指获取对象方法或函数调用的返回值的过程。这里有一个简单的例子：

```php
// /repo/ch02/php7_dereference_1.php
$alpha = range('A','Z');
echo $alpha[15] . $alpha[7] . $alpha[15];
// output: PHP
```

`$alpha`包含 26 个元素，代表字母`A`到`Z`。这个例子解引用了数组，提取了第 7 和第 15 个元素，产生了`PHP`的输出。解引用函数或方法调用简单地意味着执行函数或方法并访问结果。

### 解引用插入字符串

下一个例子有点疯狂，请仔细跟随。以下示例在 PHP 8 中有效，但在 PHP 7 或之前的版本中无效：

```php
// /repo/ch02/php8_dereference_2.php
$alpha = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
$num   = '0123456789';
$test  = [15, 7, 15, 34];
foreach ($test as $pos)
    echo "$alpha$num"[$pos];
```

在这个例子中，两个字符串`$alpha`和`$num`在`foreach()`循环内使用双引号进行插值。以下是 PHP 7 的输出：

```php
root@php8_tips_php7 [ /repo/ch02 ]# php php7_dereference_2.php 
PHP Parse error:  syntax error, unexpected '[', expecting ',' or ';' in /repo/ch02/php7_dereference_2.php on line 7
Parse error: syntax error, unexpected '[', expecting ',' or ';' in /repo/ch02/php7_dereference_2.php on line 7
```

在 PHP 8 中相同的代码产生以下输出：

```php
root@php8_tips_php8 [ /repo/ch02 ]# php php8_dereference_2.php 
PHP8
```

结论是，PHP 7 在解引用插入字符串方面不一致，而 PHP 8 展现了改进的一致性。

### 魔术常量的不一致解引用

在 PHP 7 和之前的版本中，常量可以被解引用，而魔术常量则不行。下面是一个简单的例子，它产生了当前文件的最后三个字母：

```php
// /repo/ch02/php8_dereference_3.php
define('FILENAME', __FILE__);
echo FILENAME[-3] . FILENAME[-2] . FILENAME[-1];
echo __FILE__[-3] . __FILE__[-2] . __FILE__[-1];
```

以下是 PHP 7 的结果：

```php
root@php8_tips_php7 [ /repo/ch02 ]# php php7_dereference_3.php
PHP Parse error:  syntax error, unexpected '[', expecting ',' or ';' in /repo/ch02/php7_dereference_3.php on line 7
Parse error: syntax error, unexpected '[', expecting ',' or ';' in /repo/ch02/php7_dereference_3.php on line 7
```

以下是 PHP 8 的结果：

```php
root@php8_tips_php8 [ /repo/ch02 ]# php php8_dereference_3.php
phpphp
```

再次强调的是，PHP 8 中的解引用操作是一致的（这是一件好事！）。

### 类常量解引用的一致性

当尝试解引用类常量时会出现相关问题。为了最好地说明问题，想象一下我们有三个类。第一个类`JsonResponse`以**JavaScript 对象表示法**（**JSON**）格式产生数据，如下面的代码片段所示：

```php
class JsonResponse {
    public static function render($data) {
        return json_encode($data, JSON_PRETTY_PRINT);
    }
}
```

第二个类`SerialResponse`使用内置的 PHP `serialize()`函数产生响应，如下面的代码片段所示：

```php
class SerialResponse {
    public static function render($data) {
        return serialize($data);
    }
}
```

最后，一个`Test`类能够产生任何一个响应，如下面的代码片段所示：

```php
class Test {
    const JSON = ['JsonResponse'];
    const TEXT = 'SerialResponse';
    public static function getJson($data) {
        echo self::JSON[0]::render($data);
    }
    public static function getText($data) {
        echo self::TEXT::render($data);
    }
}
```

正如你在本节的早期示例中所看到的，PHP 早期版本的结果是不一致的。调用`Test::getJson($data)`可以正常工作。然而，调用`Test::getText($data)`会产生错误：

```php
root@php8_tips_php7 [ /repo/ch02 ]# php php7_dereference_4.php PHP Parse error:  syntax error, unexpected '::' (T_PAAMAYIM_NEKUDOTAYIM), expecting ',' or ';' in /repo/ch02/php7_dereference_4.php on line 26
Parse error: syntax error, unexpected '::' (T_PAAMAYIM_NEKUDOTAYIM), expecting ',' or ';' in /repo/ch02/php7_dereference_4.php on line 26
```

在 PHP 8 下，与之前显示的类中定义的方法调用产生了一致的结果，如下所示：

```php
root@php8_tips_php8 [ /repo/ch02 ]# php php8_dereference_4.php
{
    "A": 111,
    "B": 222,
    "C": 333}
a:3:{s:1:"A";i:111;s:1:"B";i:222;s:1:"C";i:333;}
```

总之，在 PHP 8 中，类常量现在以统一的方式进行解引用，使您能够产生更清晰的代码。现在，让我们看看 PHP 8 如何允许您在更多地方使用表达式。

### 增强了`new`和`instanceof`的表达式支持

与 PHP 7 编程相关的乐趣之一是能够在几乎任何地方使用任意 PHP 表达式。在这个简单的例子中，注意在引用`$nav`数组的方括号内使用了一个`$_GET['page'] ?? 'home'`任意表达式：

```php
// /repo/ch02/php7_arbitrary_exp.php
$nav = [
    'home'     => 'home.html',
    'about'    => 'about.html',
    'services' => 'services/index.html',
    'support'  => 'support/index.html',
];
$html = __DIR__ . '/../includes/'
      . $nav[$_GET['page'] ?? 'home'];
```

在 PHP 7 和之前的版本中，如果表达式涉及`new`或`instanceof`关键字，则不可能做到这一点。正如你可能已经猜到的那样，这种不一致性已经在 PHP 8 中得到解决。现在可以实现以下操作：

```php
// /repo/ch02/php8_arbitrary_exp_new.php
// definition of the JsonRespone and SerialResponse
// classes are shown above
$allowed = [
    'json' => 'JsonResponse',
    'text' => 'SerialResponse'
];
$data = ['A' => 111, 'B' => 222, 'C' => 333];
echo (new $allowed[$_GET['type'] ?? 'json'])
        ->render($data);
```

这个代码示例展示了在数组引用内使用任意表达式，与`new`关键字一起使用。

提示

有关 PHP 8 中统一变量语法更新的更多信息，请参阅此文章：[`wiki.php.net/rfc/variable_syntax_tweaks`](https://wiki.php.net/rfc/variable_syntax_tweaks)。

现在让我们来看看 PHP 8 中可用的新的字符串和数组处理技术。

# 学习新的数组和字符串处理技术

PHP 8 中的数组和字符串处理技术有许多改进。虽然本书中没有足够的空间来涵盖每一个增强功能，但我们将在本节中检查更重要的改进。

## 使用 `array_splice()`

`array_splice()` 函数是 `substr()` 和 `str_replace()` 的混合体：它允许您用另一个数组替换一个数组的子集。然而，当您只需要用不同的内容替换数组的最后部分时，它的使用会变得麻烦。快速查看语法会让人觉得开始变得不方便——`replacement` 参数在 `length` 参数之前，如下所示：

`array_splice(&$input,$offset[,$length[,$replacement]]):array`

传统上，开发人员首先在原始数组上运行 `count()`，然后将其用作 `length` 参数，如下所示：

`array_splice($arr, 3, count($arr), $repl);`

在 PHP 8 中，第三个参数可以是 `NULL`，省去了对 `count()` 的额外调用。如果您利用 PHP 8 的**命名参数**特性，代码会变得更加简洁。下面是为 PHP 8 编写的相同代码片段：

`array_splice($arr, 3, replacement: $repl);`

这里有另一个例子清楚地展示了 PHP 7 和 PHP 8 之间的差异：

```php
// /repo/ch02/php7_array_splice.php
$arr  = ['Person', 'Camera', 'TV', 'Woman', 'Man'];
$repl = ['Female', 'Male'];
$tmp  = $arr;
$out  = array_splice($arr, 3, count($arr), $repl);
var_dump($arr);
$arr  = $tmp;
$out  = array_splice($arr, 3, NULL, $repl);
var_dump($arr);
```

如果您在 PHP 7 中运行代码，请注意最后一个 `var_dump()` 实例的结果，如下所示：

```php
repo/ch02/php7_array_splice.php:11:
array(7) {
  [0] =>  string(6) "Person"
  [1] =>  string(6) "Camera"
  [2] =>  string(2) "TV"
  [3] =>  string(6) "Female"
  [4] =>  string(4) "Male"
  [5] =>  string(5) "Woman"
  [6] =>  string(3) "Man"
}
```

在 PHP 7 中，将 `NULL` 值提供给 `array_splice()` 的第三个参数会导致两个数组简单合并，这不是期望的结果！

现在，让我们来看一下最后一个 `var_dump()` 的输出，但这次是在 PHP 8 下运行的：

```php
root@php8_tips_php8 [ /repo/ch02 ]# php php8_array_splice.php
// some output omitted
array(5) {
  [0]=>  string(6) "Person"
  [1]=>  string(6) "Camera"
  [2]=>  string(2) "TV"
  [3]=>  string(6) "Female"
  [4]=>  string(4) "Male"
}
```

如您所见，在 PHP 8 下，将第三个参数设为 `NULL` 与在运行时将数组 `count()` 作为第三个参数提供给 `array_splice()` 具有相同的功能。您还会注意到在 PHP 8 中，数组元素的总数为 `5`，而在 PHP 7 中，相同代码的运行结果为 `7`。

## 使用 `array_slice()`

`array_slice()` 函数在数组上的操作与 `substr()` 在字符串上的操作一样。PHP 早期版本的一个大问题是，在内部，PHP 引擎会顺序地遍历整个数组，直到达到所需的偏移量。如果偏移量很大，性能会直接与数组大小成正比地受到影响。

在 PHP 8 中，使用了一种不需要顺序数组迭代的不同算法。随着数组大小的增加，性能改进变得越来越明显。

1.  在这个示例中，我们首先构建了一个大约有 600 万条目的大数组：

```php
// /repo/ch02/php8_array_slice.php
ini_set('memory_limit', '1G');
$start = microtime(TRUE);
$arr   = [];
$alpha = range('A', 'Z');
$beta  = $alpha;
$loops = 10000;     // size of outer array
$iters = 500;       // total iterations
$drip  = 10;        // output every $drip times
$cols  = 4;
for ($x = 0; $x < $loops; $x++)
    foreach ($alpha as $left)
        foreach ($beta as $right)
            $arr[] = $left . $right . rand(111,999);
```

1.  接下来，我们遍历数组，取大于 `999,999` 的随机偏移量。这会迫使 `array_slice()` 艰苦工作，并显示出 PHP 7 和 8 之间的显著性能差异，如下面的代码片段所示：

```php
$max = count($arr);
for ($x = 0; $x < $iters; $x++ ) {
    $offset = rand(999999, $max);
    $slice  = array_slice($arr, $offset, 4);
    // not all display logic is shown
}
$time = (microtime(TRUE) - $start);
echo "\nElapsed Time: $time seconds\n";
```

在 PHP 7 下运行代码时的输出如下：

![图 2.5 – 使用 PHP 7 的 array_slice() 示例](img/Figure_2.5_B16992.jpg)

图 2.5 – 使用 PHP 7 的 array_slice() 示例

请注意，在 PHP 8 下运行相同代码时的显著性能差异：

![图 2.6 – 使用 PHP 8 的 array_slice() 示例](img/Figure_2.6_B16992.jpg)

图 2.6 – 使用 PHP 8 的 array_slice() 示例

重要提示

新算法只在数组不包含 `NULL` 值的情况下有效。如果数组包含 `NULL` 元素，则会触发旧算法，并进行顺序迭代。

现在让我们转向一些出色的新字符串函数。 

## 检测字符串的开头、中间和结尾

PHP 开发人员经常需要处理的一个问题是检查字符串的开头、中间或结尾是否出现一组字符。当前一组字符串函数的问题在于它们*不是设计*来处理子字符串的存在或不存在。相反，当前一组函数是设计来确定子字符串的*位置*。然后，可以以布尔方式插值来确定子字符串的存在或不存在。

这种方法的问题可以用温斯顿·丘吉尔爵士的一句著名的引语来概括： 

“高尔夫是一个旨在用极不适合这一目的的武器将一个非常小的球打入一个更小的洞的游戏。”

– *温斯顿·丘吉尔*

现在让我们来看看三个非常有用的新字符串函数，它们解决了这个问题。

### str_starts_with()

我们要检查的第一个函数是`str_starts_with()`。为了说明它的用法，考虑一个代码示例，我们要在开头找到`https`，在结尾找到`login`，如下面的代码片段所示：

```php
// /repo/ch02/php7_starts_ends_with.php
$start = 'https';
if (substr($url, 0, strlen($start)) !== $start) 
    $msg .= "URL does not start with $start\n";
// not all code is shown
```

正如我们在本节的介绍中提到的，为了确定一个字符串是否以`https`开头，我们需要调用`substr()`和`strlen()`。这两个函数都不是设计来给我们想要的答案的。而且，使用这两个函数会在我们的代码中引入低效，并导致不必要的资源利用增加。

相同的代码可以在 PHP 8 中编写如下：

```php
// /repo/ch02/php8_starts_ends_with.php
$start = 'https';
if (!str_starts_with($url, $start))
    $msg .= "URL does not start with $start\n";
// not all code is shown
```

### str_ends_with()

与`str_starts_with()`类似，PHP 8 引入了一个新函数`str_ends_with()`，用于确定字符串的结尾是否与某个值匹配。为了说明这个新函数的用处，考虑使用`strrev()`和`strpos()`的旧 PHP 代码，可能如下所示：

```php
$end = 'login';
if (strpos(strrev($url), strrev($end)) !== 0)
    $msg .= "URL does not end with $end\n";
```

在一个操作中，`$url`和`$end`都需要被反转，这个过程会随着字符串长度的增加而变得越来越昂贵。而且，正如前面提到的，`strpos()`的目的是返回子字符串的*位置*，而不是确定其存在与否。

在 PHP 8 中，可以通过以下方式实现相同的功能：

```php
if (!str_ends_with($url, $end))
    $msg .= "URL does not end with $end\n";
```

### str_contains()

在这个上下文中的最后一个函数是`str_contains()`。正如我们讨论过的，在 PHP 7 及更早版本中，除了`preg_match()`之外，没有特定的 PHP 函数告诉你一个子字符串是否存在于一个字符串中。

使用`preg_match()`的问题，正如我们一再被警告的那样，是性能下降。为了处理*正则表达式*，`preg_match()`首先需要分析模式。然后，它必须执行第二次扫描，以确定字符串的哪个部分与模式匹配。这在时间和资源利用方面是一个极其昂贵的操作。

重要提示

当我们提到一个操作在时间和资源方面是*昂贵*时，请记住，如果您的脚本只包含几十行代码和/或您在循环中没有重复操作数千次，那么使用本节中描述的新函数和技术可能不会带来显著的性能提升。

在下面的例子中，一个 PHP 脚本使用`preg_match()`来搜索*GeoNames*项目数据库中人口超过`15,000`的城市，以查找包含对`London`的引用的任何列表：

```php
// /repo/ch02/php7_str_contains.php
$start    = microtime(TRUE);
$target   = '/ London /';
$data_src = __DIR__ . '/../sample_data
                      /cities15000_min.txt';
$fileObj  = new SplFileObject($data_src, 'r');
while ($line = $fileObj->fgetcsv("\t")) {
    $tz     = $line[17] ?? '';
    if ($tz) unset($line[17]);
    $str    = implode(' ', $line);
    $city   = $line[1] ?? 'Unknown';
    $local1 = $line[10] ?? 'Unknown';
    $iso    = $line[8] ?? '??';
    if (preg_match($target, $str))
        printf("%25s : %12s : %4s\n", $city, $local1, 
                $iso);
}
echo "Elapsed Time: " . (microtime(TRUE) - $start) . "\n";
```

在 PHP 7 中运行时的输出如下：

![图 2.7 - 使用 preg_match()扫描 GeoNames 文件](img/Figure_2.7_B16992.jpg)

图 2.7 - 使用 preg_match()扫描 GeoNames 文件

在 PHP 8 中，可以通过用以下代码替换`if`语句来实现相同的输出：

```php
// /repo/ch02/php8_str_contains.php
// not all code is shown
    if (str_contains($str, $target))
        printf("%25s : %12s : %4s\n", $city, $local1, 
               $iso);
```

以下是来自 PHP 8 的输出：

![图 2.8 - 使用 str_contains()扫描 GeoNames 文件](img/Figure_2.8_B16992.jpg)

图 2.8 - 使用 str_contains()扫描 GeoNames 文件

从两个不同的输出屏幕可以看出，PHP 8 代码运行大约需要`0.14`微秒，而 PHP 7 需要`0.19`微秒。这本身并不是一个巨大的性能提升，但正如本节前面提到的，更多的数据、更长的字符串和更多的迭代会放大你所获得的任何小的性能提升。

提示

**最佳实践**：实现小的代码修改可以带来小的性能提升，最终积少成多，带来整体性能的大幅提升！

有关*GeoNames*开源项目的更多信息，请访问他们的网站：[`www.geonames.org/`](https://www.geonames.org/)。

现在你知道了如何以及在哪里使用三个新的字符串函数。你还可以编写更高效的代码，使用专门设计用于检测目标字符串开头、中间或结尾的子字符串存在与否的函数。

最后，我们以查看新的 SQLite3 授权回调结束本章。

# 使用授权回调保护 SQLite 数据库

许多 PHP 开发人员更喜欢使用**SQLite**作为他们的数据库引擎，而不是像 PostgreSQL、MySQL、Oracle 或 MongoDB 这样的独立数据库服务器。使用 SQLite 的原因有很多，但通常归结为以下几点：

+   **SQLite 是基于文件的数据库**：你不需要安装单独的数据库服务器。

+   **易于分发**：唯一的要求是目标服务器需要安装`SQLite`可执行文件。

+   **SQLite 轻量级**：由于没有不断运行的服务器，所需资源更少。

尽管如此，缺点是它的可扩展性不是很好。如果你有相当大量的数据要处理，最好安装一个更强大的数据库服务器。另一个潜在的主要缺点是 SQLite 没有安全性，下一小节将介绍。

提示

有关 SQLite 的更多信息，请访问他们的主要网页：[`sqlite.org/index.html`](https://sqlite.org/index.html)。

## 等等...没有安全性？

是的，你听对了：默认情况下，按照其设计，SQLite 没有安全性。当然，这就是许多开发人员喜欢使用它的原因：没有安全性使得它非常容易使用！

以下是一个连接到 SQLite 数据库并对`geonames`表进行简单查询的示例代码块。它返回了印度人口超过 200 万的城市列表：

```php
// /repo/ch02/php8_sqlite_query.php
define('DB_FILE', __DIR__ . '/tmp/sqlite.db');
$sqlite = new SQLite3(DB_FILE);
$sql = 'SELECT * FROM geonames '
      . 'WHERE country_code = :cc AND population > :pop';
$stmt = $sqlite->prepare($sql);
$stmt->bindValue(':cc', 'IN');
$stmt->bindValue(':pop', 2000000);
$result = $stmt->execute();
while ($row = $result->fetchArray(SQLITE3_ASSOC)) {
    printf("%20s : %2s : %16s\n", 
        $row['name'], $row['country_code'],
        number_format($row['population']));
}  // not all code is shown
```

大多数其他数据库扩展在建立连接时至少需要用户名和密码。如前面的代码片段所示，`$sqlite`实例是完全没有安全性的：没有用户名或密码。

## 什么是 SQLite 授权回调？

SQLite3 引擎现在允许你向 SQLite 数据库连接注册一个**授权回调**。当向数据库发送**预编译语句**进行编译时，将调用回调例程。以下是在`SQLite3`实例上设置授权回调的通用语法：

`$sqlite3->setAuthorizer(callable $callback);`

回调函数应该返回三个`SQLite3`类常量中的一个，每个代表一个整数值。如果回调函数返回除这三个值之外的任何值，就假定为`SQLite3::DENY`，操作将不会继续进行。下表列出了三个期望的返回值：

![表 2.2 - 有效的 SQLite 授权回调返回值](img/Table_2.2_B16992.jpg)

表 2.2 - 有效的 SQLite 授权回调返回值

现在你对回调有了一些了解，让我们看看它是如何被调用的。

## 回调函数会接收到什么？

当您执行`$sqlite->prepare($sql)`时，回调被调用。在那时，SQLite3 引擎将在回调中传递一个到五个参数。第一个参数是一个**操作代码**，确定剩余参数的性质。因此，以下可能是您最终定义的回调的适当通用函数签名：

```php
function NAME (int $actionCode, ...$params) 
{ /* callback code */ };
```

大部分情况下，操作代码与要准备的 SQL 语句相对应。以下表总结了一些更常见的操作代码：

![表 2.3 – 发送到回调的常见操作代码](img/Table_2.3_B16992.jpg)

表 2.3 – 发送到回调的常见操作代码

现在是时候看一个使用示例了。

## 授权使用示例

在下面的示例中，我们被允许从 SQLite `geonames`表中读取，但不能插入、删除或更新：

1.  我们首先在`/repo/ch02/includes/`目录中定义一个`auth_callback.php`包含文件。在`include`文件中，我们首先定义在回调中使用的常量，如下面的代码片段所示：

```php
// /repo/ch02/includes/auth_callback.php
define('DB_FILE', '/tmp/sqlite.db');
define('PATTERN', '%-8s | %4s | %-28s | %-15s');
define('DEFAULT_TABLE', 'Unknown');
define('DEFAULT_USER', 'guest');
define('ACL' , [
    'admin' => [
        'users' => [SQLite3::READ, SQLite3::SELECT,
            SQLite3::INSERT, SQLite3::UPDATE,
            SQLite3::DELETE],
        'geonames' => [SQLite3::READ, SQLite3::SELECT,
            SQLite3::INSERT, SQLite3::UPDATE, 
            SQLite3::DELETE],
    ],
    'guest' => [
        'geonames' => [SQLite3::READ, 
                       SQLite3::SELECT],
    ],
]);
```

**访问控制列表**（**ACL**）的工作方式是，主要外键是用户（例如`admin`或`guest`）；次要键是表（例如`users`或`geonames`）；值是允许该用户和表的`SQLite3`操作代码的数组。

在先前显示的示例中，`admin`用户对两个表都有所有权限，而`guest`用户只能从`geonames`表中读取。

1.  接下来，我们定义实际的授权回调函数。函数中我们需要做的第一件事是将默认返回值设置为`SQLite3::DENY`。我们还检查操作代码是否为`SQLite3::SELECT`，如果是，则简单地返回`OK`。当首次处理`SELECT`语句并且不提供有关表或列的任何信息时，将发出此操作代码。代码可以在以下片段中看到：

```php
function auth_callback(int $code, ...$args) {
    $status = SQLite3::DENY;
    $table  = DEFAULT_TABLE;
    if ($code === SQLite3::SELECT) {
        $status = SQLite3::OK;
```

1.  如果操作代码不是`SQLite3::SELECT`，我们需要首先确定涉及哪个表，然后才能决定允许还是拒绝该操作。表名作为提供给我们回调的第二个参数报告。

1.  现在是使用*variadics operator*的绝佳时机，因为我们不确定可能传递多少参数。但是，对于关注的主要操作（例如`INSERT`、`UPDATE`或`DELETE`），放入`$args`的第一个位置的是表名。否则，我们从会话中获取表名。

代码显示在以下片段中：

```php
    } else {
        if (!empty($args[0])) {
            $table = $args[0];
        } elseif (!empty($_SESSION['table'])) {
            $table = $_SESSION['table'];
        }
```

1.  同样地，我们从会话中检索用户名，如下所示：

```php
        $user  = $_SESSION['user'] ?? DEFAULT_USER;
```

1.  接下来，我们检查用户是否在 ACL 中定义，然后检查表是否为该用户分配了权限。如果给定的操作代码在与用户和表组合关联的数组中，返回`SQLite3::OK`。

代码显示在以下片段中：

```php
    if (!empty(ACL[$user])) {
        if (!empty(ACL[$user][$table])) {
            if (in_array($code, ACL[$user][$table])) {
                $status = SQLite3::OK;
            }
        }
    }
```

1.  然后我们将表名存储在会话中并返回状态代码，如下面的代码片段所示：

```php
  } // end of "if ($code === SQLite3::SELECT)"
  $_SESSION['table'] = $table;
  return $status;
} // end of function definition
```

现在我们转向调用程序。

1.  在包含定义授权回调的 PHP 文件之后，我们通过接受命令行参数、**统一资源定位符**（**URL**）参数或简单地分配`admin`来模拟获取用户名，如下面的代码片段所示：

```php
// /repo/ch02/php8_sqlite_auth_admin.php
include __DIR__ . '/includes/auth_callback.php';
// Here we simulate the user acquisition:
session_start();
$_SESSION['user'] = 
    $argv[1] ?? $_GET['usr'] ?? DEFAULT_USER;
```

1.  接下来，我们创建两个数组并使用`shuffle()`使它们的顺序随机。我们从随机数组中构建用户名、电子邮件和 ID 值，如下面的代码片段所示：

```php
$name = ['jclayton','mpaulovich','nrousseau',
         'jporter'];
$email = ['unlikelysource.com',
          'lfphpcloud.net','phptraining.net'];
shuffle($name);
shuffle($email);
$user_name = $name[0];
$user_email = $name[0] . '@' . $email[0];
$id = md5($user_email . rand(0,999999));
```

1.  然后，我们创建`SQLite3`实例并分配授权回调，如下所示：

```php
$sqlite = new SQLite3(DB_FILE);
$sqlite->setAuthorizer('auth_callback');
```

1.  现在 SQL `INSERT`语句已经定义并发送到 SQLite 进行准备。请注意，这是调用授权回调的时候。

代码显示在以下片段中：

```php
$sql = 'INSERT INTO users '
     . 'VALUES (:id, :name, :email, :pwd);';
$stmt = $sqlite->prepare($sql);
```

1.  如果授权回调拒绝操作，则语句对象为`NULL`，因此最好使用`if()`语句来测试其存在。如果是这样，我们然后继续绑定值并执行语句，如下面的代码片段所示：

```php
if ($stmt) {
    $stmt->bindValue(':id', $id);
    $stmt->bindValue(':name', $user_name);
    $stmt->bindValue(':email', $user_email);
    $stmt->bindValue(':pwd', 'password');
    $result = $stmt->execute();
```

1.  为了确认结果，我们定义了一个 SQL `SELECT`语句，以显示`users`表的内容，如下所示：

```php
    $sql = 'SELECT * FROM users';
    $result = $sqlite->query($sql);
    while ($row = $result->fetchArray(SQLITE3_ASSOC))
        printf("%-10s : %-  10s\n",
            $row['user_name'], $row['user_email']);
}
```

重要提示

这里没有显示所有代码。有关完整代码，请参考`/repo/ch02/php8_sqlite_auth_admin.php`。

如果我们运行调用程序，并将用户设置为`admin`，则结果如下：

![图 2.9 – SQLite3 授权回调：admin 用户](img/Figure_2.9_B16992.jpg)

图 2.9 – SQLite3 授权回调：admin 用户

前面截图的输出显示，由于我们以`admin`用户身份运行，并具有足够的授权权限，操作成功。当用户设置为`guest`时，输出如下：

![图 2.10 – SQLite3 授权回调：guest 用户](img/Figure_2.10_B16992.jpg)

图 2.10 – SQLite3 授权回调：guest 用户

输出显示，由于我们以权限不足的用户身份运行，尝试运行`prepare()`是不成功的。

这就结束了我们对这一期待已久的功能的讨论。您现在知道如何向一个否则不安全的数据库技术添加授权。

提示

描述添加 SQLite 授权回调的原始拉取请求：[`github.com/php/php-src/pull/4797`](https://github.com/php/php-src/pull/4797)

有关官方 SQLite 文档的授权回调：[`www.sqlite.org/c3ref/set_authorizer.html`](https://www.sqlite.org/c3ref/set_authorizer.html)

传递给回调函数的操作代码：[`www.sqlite.org/c3ref/c_alter_table.html`](https://www.sqlite.org/c3ref/c_alter_table.html)

结果代码的完整列表：[`www.sqlite.org/rescode.html`](https://www.sqlite.org/rescode.html)

`SQLite3`类的文档：[`www.php.net/sqlite3`](https://www.php.net/sqlite3)

# 总结

在本章中，您了解了 PHP 8 在过程级别引入的一些更改。您首先了解了新的 nullsafe 运算符，它允许您大大缩短可能失败的对象引用链的任何代码。您还了解了三元运算符和可变参数运算符的使用已经得到了加强和改进，以及连接运算符在优先级顺序中已经降级。本章还涵盖了箭头函数的优缺点，以及它们如何作为匿名函数的清晰简洁的替代方案。

本章的后续部分向您展示了 PHP 8 如何继续沿着在 PHP 7 中首次引入的统一变量语法的趋势发展。您了解了 PHP 8 中如何解决剩余的不一致之处，包括插值字符串和魔术常量的解引用，以及在数组和字符串处理方面的改进，这些改进承诺使您的 PHP 8 更清洁、更简洁和更高性能。

最后，在最后一节中，您了解了一个新功能，它提供了对 SQLite 授权回调的支持，允许您在使用 SQLite 作为数据库时最终提供一定程度的安全性。

在下一章中，您将了解 PHP 8 的错误处理增强功能。
