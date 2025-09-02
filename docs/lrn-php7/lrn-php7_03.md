# 第三章. 理解 PHP 基础知识

学习一门新语言并不容易。你需要理解语言的语法，以及其语法规则，即何时以及为什么使用语言中的每个元素。幸运的是，一些语言来自同一个根源。例如，西班牙语和法语是罗曼语族，因为它们都源自拉丁语口语；这意味着这两种语言有很多规则是共享的，如果你已经知道法语，学习西班牙语就会容易得多。

编程语言相当相似。如果你已经知道另一种编程语言，那么通过这一章会非常容易。但如果这是你第一次，那么你需要从头开始理解所有那些语法规则，所以可能需要更多的时间。但别担心！我们在这里帮助你完成这项任务。

在本章中，你将学习以下内容：

+   PHP 文件

+   PHP 中的变量、字符串、数组运算符

+   PHP 在网络应用中

+   PHP 中的控制结构

+   PHP 中的函数

+   PHP 文件系统

# PHP 文件

从现在开始，我们将专注于你的 `index.php` 文件，所以你可以直接启动 Web 服务器，然后访问 `http://localhost:8080` 来查看结果。

你可能已经注意到，为了编写 PHP 代码，你必须以 `<?php` 开始文件。还有其他选项，你还可以用 `?>` 结束文件，但这些都是不必要的。重要的是要知道，一旦你用 `<?php ?>` 标签包围了 PHP 代码块，你就可以在你的 PHP 文件中混合 PHP 代码和其他内容，如 HTML、CSS 或 JavaScript。

```php
<?php
  echo 'hello world';
?>
bye world

```

```php
hello world and bye world. The reason why this happens is simple: you already know that the PHP code there prints the hello world message. What happens next is that anything outside the PHP tags will be interpreted as is. If there is an HTML code for instance, it would not be printed as is, but will be interpreted by the browser.
```

你将在第六章中学习，*适应 MVC*，为什么通常将 PHP 和 HTML 混合使用不是一个好主意。现在，假设这是不好的，让我们尽量避免。为此，你可以使用以下四个函数之一从另一个 PHP 文件中包含一个文件：

+   `include`: 每次调用时都会尝试查找并包含指定的文件。如果找不到文件，PHP 将抛出警告，但会继续执行。

+   `require`: 这将执行与 `include` 相同的操作，但如果找不到文件，PHP 将抛出错误而不是警告。

+   `include_once`: 这个函数将执行 `include` 的功能，但只有在第一次调用时才会包含文件。后续的调用将被忽略。

+   `require_once`: 这与 `require` 的功能相同，但只有在第一次调用时才会包含文件。后续的调用将被忽略。

每个函数都有自己的用途，所以说一个比另一个好是不正确的。只需仔细思考你的场景，然后做出决定。例如，让我们尝试从我们的 `index.php` 文件中包含我们的 `index.html` 文件，这样我们就不混合 PHP 和 HTML，而是两者兼得：

```php
<?php
echo 'hello world';
require 'index.html';

```

我们选择使用`require`，因为我们知道文件在那里——如果它不存在，我们就不想继续执行。此外，由于它是某些 HTML 代码，我们可能希望多次包含它，所以我们没有选择`require_once`选项。你可以尝试引入一个不存在的文件，看看浏览器会说什么。

PHP 不会考虑空行；你可以添加尽可能多的空行来使你的代码更容易阅读，而且它不会对你的应用程序产生任何影响。另一个有助于编写可读代码的元素，但 PHP 会忽略它，是注释。让我们看看它们在实际中的应用：

```php
<?php

/*
 * This is the first file loaded by the web server.
 * It prints some messages and html from other files.
 */

// let's print a message from php
echo 'hello world';

// and then include the rest of html
require 'index.html';
```

这段代码与上一个代码做的是同样的工作，但现在每个人都会很容易理解我们试图做什么。我们可以看到两种类型的注释：单行注释和多行注释。第一种类型由以`//`开头的一行组成，第二种类型在`/*`和`*/`之间包含多行。我们以星号开始每个注释行，但这完全是可选的。

# 变量

变量保存一个值以供将来引用。如果我们想改变它，这个值可以改变；这就是为什么它们被称为变量。让我们通过一个例子来看看它们。将此代码保存到你的`index.php`文件中：

```php
<?php
$a = 1;
$b = 2;
$c = $a + $b;
echo $c; // 3
```

在前面的代码片段中，我们有三个变量：`$a`的值为`1`，`$b`的值为`2`，`$c`包含`$a`和`$b`的和，因此`$c`等于 3。你的浏览器应该打印变量`$c`的值，即 3。

将值赋给变量意味着给它一个值，这就像上一个例子中所示的那样，使用等号。如果你没有给变量赋值，当 PHP 检查其内容时，我们会收到一个 PHP 的通知。通知只是一个消息，告诉我们某些事情并不完全正确，但这只是一个小问题，你可以继续执行。未分配变量的值将是 null，即没有内容。

PHP 变量以`$`符号开头，后跟变量名。一个有效的变量名以字母或下划线开头，后跟任何组合的字母、数字和/或下划线。它是区分大小写的。让我们看看一些例子：

```php
<?php
$_some_value = 'abc'; // valid
$1number = 12.3; // not valid!
$some$signs% = '&^%'; // not valid!
$go_2_home = "ok"; // valid
$go_2_Home = 'no'; // this is a different variable
$isThisCamelCase = true; // camel case
```

记住，`//`之后的所有内容都是注释，因此会被 PHP 忽略。

在这段代码中，我们可以看到像`$_some_value`和`$go_2_home`这样的变量名是有效的。`$1number`和`$some$signs%`不是有效的，因为它们以数字开头，或者它们包含无效的符号。由于名称是区分大小写的，`$go_2_home`和`$go_2_Home`是两个不同的变量。最后，我们展示了驼峰命名法，这是大多数开发者首选的选项。

## 数据类型

我们可以将不仅仅是数字赋值给变量。PHP 有八个原始类型，但到目前为止，我们将关注它的四种标量类型：

+   **布尔值**：它们只取 true 或 false 值

+   **整数**：这些是没有小数点的数值，例如，2 或 5

+   **浮点数或浮点数**：这些是带有小数点的数字，例如，2.3

+   **字符串**：这些是由单引号或双引号包围的字符的连接，例如 'this' 或 "that"

尽管 PHP 定义了这些类型，但它允许用户将不同类型的数据分配给同一个变量。查看以下代码以了解它是如何工作的：

```php
<?php
$number = 123;
var_dump($number);
$number = 'abc';
var_dump($number);
```

如果你在浏览器上查看结果，你会看到以下内容：

```php
int(123) string(3) "abc"
```

代码首先将值`123`分配给变量`$number`。由于`123`是整数，变量的类型将是整数`int`。这就是我们在使用`var_dump`打印变量内容时看到的内容。之后，我们将另一个值分配给同一个变量，这次是一个字符串。在打印新内容时，我们看到变量的类型从整数变为字符串，但 PHP 在任何时候都没有抱怨。这被称为**类型转换**。

让我们检查另一段代码：

```php
<?php
$a = "1";
$b = 2;
var_dump($a + $b); // 3
var_dump($a . $b); // 12
```

您已经知道`+`运算符返回两个数值的和。您稍后将会看到`.`运算符连接两个字符串。因此，前面的代码将一个字符串和一个整数分配给两个变量，然后尝试将它们相加和连接。

当尝试将它们相加时，PHP 知道它需要两个数值，因此它会尝试将字符串转换为整数。在这种情况下，由于字符串代表一个有效的数字，所以很容易。这就是为什么我们看到第一个结果是一个整数 3（*1 + 2*）。

在最后一行，我们正在进行字符串连接。在 `$b` 中有一个整数，所以 PHP 会首先尝试将其转换为字符串——即 "2"，然后将其与另一个字符串 "1" 连接。结果是字符串 "12"。

### 注意

**类型转换**

PHP 仅在存在需要不同变量类型的上下文时才会尝试转换变量的数据类型。但 PHP 不会改变变量本身的价值和类型。相反，它会取值并尝试转换，同时保持变量不变。

# 运算符

使用变量很方便，但如果我们不能使它们相互交互，我们就无法做太多。**运算符**是接受一些表达式（操作数）并对其执行操作以获得结果的元素。最常见的运算符例子是算术运算符，您之前已经看到了。

**表达式**几乎可以是任何具有值的元素。变量、数字或文本是表达式的例子，但你会看到它们可以变得非常复杂。运算符期望特定类型的表达式，例如，算术运算符期望整数或浮点数。但如您所知，PHP 会在可能的情况下处理给定表达式的类型转换。

让我们看看最重要的运算符组。

## 算术运算符

算术运算符非常直观，正如你所知。加法（`+`）、减法（`-`）、乘法（`*`）和除法（`/`）都按照它们的名称执行。取模运算符（`%`）给出两个操作数除法的余数。指数运算符（`**`）将第一个操作数提升为第二个操作数的幂。最后，取反运算符（`-`）对操作数取反。这是唯一只需要一个操作数的算术运算符。

让我们看看一些例子：

```php
<?php
$a = 10;
$b = 3;
var_dump($a + $b); // 13
var_dump($a - $b); // 7
var_dump($a * $b); // 30
var_dump($a / $b); // 3.333333...
var_dump($a % $b); // 1
var_dump($a ** $b); // 1000
var_dump(-$a); // -10
```

如你所见，它们很容易理解！

## 赋值运算符

你也熟悉这个，因为我们已经在我们的例子中使用过它了。赋值运算符将表达式的结果赋给一个变量。现在你知道一个表达式可以像数字一样简单，或者，例如，一系列算术运算的结果。以下示例将表达式的结果赋给一个变量：

```php
<?php
$a = 3 + 4 + 5 - 2;
var_dump($a); // 10
```

存在一系列的赋值运算符，它们作为快捷方式工作。你可以通过组合一个算术运算符和一个赋值运算符来构建它们。让我们看看一些例子：

```php
$a = 13;
$a += 14; // same as $a = $a + 14;
var_dump($a);
$a -= 2; // same as $a = $a - 2;
var_dump($a);
$a *= 4; // same as $a = $a * 4;
var_dump($a);
```

## 比较运算符

比较运算符是使用最频繁的运算符组之一。它们接受两个操作数并比较它们，通常将比较的结果作为布尔值返回，即`true`或`false`。

有四种非常直观的比较运算符：`<`（小于）、`<=`（小于等于）、`>`（大于）和`>=`（大于等于）。还有一个特殊的运算符`<=>`（飞船），它比较两个操作数并返回一个整数而不是布尔值。当比较*a*和*b*时，如果*a*小于*b*，结果将小于 0；如果*a*等于*b*，结果为 0；如果*a*大于*b*，结果大于 0。让我们看看一些例子：

```php
<?php
var_dump(2 < 3); // true
var_dump(3 < 3); // false
var_dump(3 <= 3); // true
var_dump(4 <= 3); // false
var_dump(2 > 3); // false
var_dump(3 >= 3); // true
var_dump(3 > 3); // false
var_dump(1 <=> 2); // int less than 0
var_dump(1 <=> 1); // 0
var_dump(3 <=> 2); // int greater than 0
```

存在比较运算符来评估两个表达式是否相等，但你需要小心类型转换。`==`（等于）运算符在类型转换后评估两个表达式，也就是说，它会尝试将两个表达式转换为相同的类型，然后再进行比较。相反，`===`（严格等于）运算符在无类型转换的情况下评估两个表达式，所以即使它们看起来相同，如果它们的类型不同，比较结果将返回`false`。同样的规则适用于`!=`或`<>`（不等于）和`!==`（严格不等于）：

```php
<?php
$a = 3;
$b = '3';
$c = 5;
var_dump($a == $b); // true
var_dump($a === $b); // false
var_dump($a != $b); // false
var_dump($a !== $b); // true
var_dump($a == $c); // false
var_dump($a <> $c); // true
```

你可以看到，当询问一个字符串和一个表示相同数字的整数是否相等时，它回答肯定；PHP 首先将它们都转换为相同的类型。另一方面，当询问它们是否相同类型时，它回答它们不是，因为它们的类型不同。

## 逻辑运算符

逻辑操作符对其操作数应用逻辑运算（也称为二元运算），返回布尔响应。最常用的有`!`（非），`&&`（与）和`||`（或）。`&&`只有在两个操作数都评估为`true`时才会返回`true`。`||`如果任何或两个操作数都是`true`，则返回`true`。`!`将返回操作数的否定值，即如果操作数是`false`则返回`true`，如果操作数是`true`则返回`false`。让我们看一些例子：

```php
<?php
var_dump(true && true); // true
var_dump(true && false); // false
var_dump(true || false); // true
var_dump(false || false); // false
var_dump(!false); // true
```

## 增量和减量操作符

增量和减量操作符也是像`+=`或`-=`这样的快捷方式，并且它们只作用于变量。这里有四个，需要特别注意。我们已经看到了前两个：

+   `++`：这个操作符在变量的左侧会将变量增加 1，然后返回结果。在右侧，它会返回变量的内容，然后增加 1。

+   `--`：这个操作符的作用与`++`相同，但它是减少值而不是增加值。

让我们看看一个例子：

```php
<?php
$a = 3;
$b = $a++; // $b is 3, $a is 4
var_dump($a, $b);
$b = ++$a; // $a and $b are 5
var_dump($a, $b);
```

在前面的代码中，在第一次赋值给`$b`时，我们使用了`$a++`。右侧的操作符首先会返回`$a`的值，即`3`，然后将它赋值给`$b`，然后才将`$a`增加 1。在第二次赋值中，左侧的操作符首先将`$a`增加 1，将`$a`的值变为`5`，然后将这个值赋给`$b`。

## 操作符优先级

你可以在一个表达式中添加多个操作符，使其长度满足需要，但你需要小心，因为一些操作符的优先级高于其他操作符，因此执行顺序可能不是你所期望的。以下表格显示了我们至今为止所学的操作符的优先级顺序：

| 操作符 | 类型 |
| --- | --- |
| `**` | 算术 |
| `++`, `--` | 增量/减少 |
| `!` | 逻辑 |
| `*`, `/`, `%` | 算术 |
| `+`, `-` | 算术 |
| `<`, `<=`, `>`, `>=` | 比较 |
| `==`, `!=`, `===`, `!==` | 比较 |
| `&&` | 逻辑 |
| `&#124;&#124;` | 逻辑 |
| `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `**=` | 赋值 |

前面的表格显示，表达式`3+2*3`将首先计算乘积`2*3`，然后是求和，所以结果是 9 而不是 15。如果你想以不同于自然优先级顺序的特定顺序执行操作，可以通过将操作放在括号内来强制执行。因此，`(3+2)*3`将首先执行求和，然后是乘法，这次结果是 15。

让我们通过一些例子来澄清这个相当棘手的问题：

```php
<?php
$a = 1;
$b = 3;
$c = true;
$d = false;
$e = $a + $b > 5 || $c; // true
var_dump($e);
$f = $e == true && !$d; // true
var_dump($f);
$g = ($a + $b) * 2 + 3 * 4; // 20
var_dump($g);
```

这个先前的例子可能是无穷无尽的，而且仍然不能涵盖你所能想象的所有场景，所以让我们保持简单。在第一行高亮的代码中，我们有算术、比较和逻辑操作符的组合，以及赋值操作符。因为没有括号，所以顺序是前面表格中详细说明的。优先级最高的操作符是加法，所以我们首先执行它：`$a` + `$b` 等于 4。下一个是比较操作符，所以 *4 > 5*，结果是 `false`。最后，逻辑操作符，`false` || `$c` (`$c` 是 `true`) 结果为 `true`。

第二个例子可能需要更多的解释。表中我们看到的第一种操作符是取反，所以我们解决它。`!$d` 是 `!false`，所以它是 `true`。现在表达式是 `$e` == `true` && `true`。首先我们需要解决比较 `$e` == `true`。知道 `$e` 是 `true`，比较结果为 `true`。最后的操作是逻辑结束，结果为 `true`。

尝试自己解决最后一个例子以获得一些练习。如果你认为我们没有充分覆盖操作符，不要害怕。在接下来的几节中，我们将看到很多例子。

# 处理字符串

在现实生活中处理字符串真的很简单。像 *检查这个字符串是否包含这个* 或 *告诉我这个字符出现多少次* 这样的动作很容易执行。但在编程时，字符串是字符的连接，你在搜索时不能一次看到所有的内容。相反，你必须一个一个地查看并跟踪内容。在这种情况下，那些很容易的动作不再那么容易了。

幸运的是，PHP 带有一整套预定义的函数，可以帮助你与字符串交互。你可以在 [`php.net/manual/en/ref.strings.php`](http://php.net/manual/en/ref.strings.php) 找到函数的完整列表，但我们只会介绍最常用的那些。让我们看看一些例子：

```php
<?php

$text = '   How can a clam cram in a clean cream can? ';

echo strlen($text); // 45
$text = trim($text);
echo $text; // How can a clam cram in a clean cream can?
echo strtoupper($text); // HOW CAN A CLAM CRAM IN A CLEAN CREAM CAN?
echo strtolower($text); // how can a clam cram in a clean cream can?
$text = str_replace('can', 'could', $text);
echo $text; // How could a clam cram in a clean cream could?
echo substr($text, 2, 6); // w coul
var_dump(strpos($text, 'can')); // false
var_dump(strpos($text, 'could')); // 4
```

在前面长段代码中，我们正在使用不同的函数玩字符串：

+   `strlen`: 这个函数返回字符串包含的字符数。

+   `trim`: 这个函数返回字符串，移除所有左边的空白和右边的空白。

+   `strtoupper` 和 `strtolower`: 这些函数分别返回所有字符都为大写或小写的字符串。

+   `str_replace`: 这个函数将给定的字符串的所有出现替换为替换字符串。

+   `substr`: 这个函数提取由参数指定的位置之间的字符串，第一个字符位于位置 0。

+   `strpos`: 这个函数显示给定字符串第一次出现的位置。如果字符串找不到，则返回 `false`。

此外，还有一个用于字符串的运算符（`.`），它可以连接两个字符串（或者当可能时将两个变量转换为字符串）。使用它非常简单：在下面的例子中，最后的语句将连接所有字符串和变量，形成句子*I am Hiro Nakamura!*。

```php
<?php
$firstname = 'Hiro';
$surname = 'Nakamura';
echo 'I am ' . $firstname . ' ' . $surname . '!';

```

关于字符串，还有一点需要注意，那就是它们的表示方式。到目前为止，我们一直用单引号括起来字符串，但你也可以用双引号括起来。区别在于，在单引号内，字符串的表示方式就是它本身，但在双引号内，在显示最终结果之前会应用一些规则。双引号与单引号处理不同的两个元素是：转义字符和变量扩展。

+   **转义字符**：这些是无法轻易表示的特殊字符。转义字符的例子包括换行符或制表符。为了表示它们，我们使用转义序列，它是反斜杠（`\`）后跟其他字符的连接。例如，`\n`代表换行符，`\t`代表制表符。

+   **变量扩展**：这允许你在字符串中包含变量引用，PHP 会用它们的当前值来替换它们。你还需要包括`$`符号。

看看下面的例子：

```php
<?php
$firstname = 'Hiro';
$surname = 'Nakamura';
echo "My name is $firstname $surname.\nI am a master of time and space. \"Yatta!\"";

```

上述代码将在浏览器中打印以下内容：

```php
My name is Hiro Nakamura.
I am a master of time and space. "Yatta!"
```

在这里，`\n`插入了一个新行。`\"`添加了双引号（你也需要转义它们，因为 PHP 会理解你想要结束字符串），变量`$firstname`和`$surname`被它们的值所替换。

# 数组

如果你有一些其他编程语言或数据结构的一般经验，你可能已经知道两种非常常见且有用的数据结构：**列表**和**映射**。列表是有序元素集合，而映射是带有键的元素集合。让我们看看一个例子：

```php
List: ["Harry", "Ron", "Hermione"]

Map: {
  "name": "James Potter",
  "status": "dead"
}
```

第一个元素是一个包含三个值的名字列表：`Harry`、`Ron`和`Hermione`。第二个是一个映射，它定义了两个值：`James Potter`和`dead`。这两个值中的每一个都通过一个键来标识：`name`和`status`。

在 PHP 中，我们没有列表和映射；我们有数组。数组是一种数据结构，它实现了列表和映射。

## 初始化数组

初始化数组有多种选择。你可以初始化一个空数组，或者初始化一个包含数据的数组。数组中用相同的方式写相同的数据也有不同的方法。让我们看看一些例子：

```php
<?php
$empty1 = [];
$empty2 = array();
$names1 = ['Harry', 'Ron', 'Hermione'];
$names2 = array('Harry', 'Ron', 'Hermione');
$status1 = [
    'name' => 'James Potter',
    'status' => 'dead'
];
$status2 = array(
    'name' => 'James Potter',
    'status' => 'dead'
);
```

在前面的例子中，我们定义了上一节中的列表和映射。`$names1`和`$names2`是完全相同的数组，只是使用了不同的表示法。同样，`$status1`和`$status2`也是这样。最后，`$empty1`和`$empty2`是创建空数组的两种方式。

以后你会看到列表被像映射一样处理。内部，数组`$names1`是一个映射，其键是有序的数字。在这种情况下，对`$names1`的另一种初始化，可以导致相同的数组，如下所示：

```php
$names1 = [
    0 => 'Harry',
    1 => 'Ron',
    2 => 'Hermione'
];
```

数组的键可以是任何字母数字值，如字符串或数字。数组的值可以是任何东西：字符串、数字、布尔值、其他数组等等。你可能会有以下这样的东西：

```php
<?php
$books = [
    '1984' => [
        'author' => 'George Orwell',
        'finished' => true,
        'rate' => 9.5
    ],
    'Romeo and Juliet' => [
        'author' => 'William Shakespeare',
        'finished' => false
    ]
];
```

这个数组是一个包含两个数组——映射的列表。每个映射包含不同的值，如字符串、双精度浮点数和布尔值。

## 填充数组

数组不是不可变的，也就是说，初始化后它们可以改变。你可以通过将其视为映射或列表来更改数组的内容。将其视为映射意味着你指定要覆盖的键，而将其视为列表意味着将另一个元素追加到数组的末尾：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
$status = [
    'name' => 'James Potter',
    'status' => 'dead'
];
$names[] = 'Neville';
$status['age'] = 32;
print_r($names, $status);
```

在前面的示例中，第一行高亮显示的行将名称`Neville`追加到名称列表中，因此列表将看起来像*['Harry', 'Ron', 'Hermione', 'Neville']*。第二个更改实际上向数组添加了一个新的键值对。你可以通过使用函数`print_r`来检查结果。它做的是类似`var_dump`的事情，只是没有每个值的类型和大小。

### 注意

**浏览器中的 print_r 和 var_dump**

当打印数组的内容时，看到每个键值一行很有用，但如果你检查浏览器，你会看到它在一行中显示整个数组。这是因为浏览器试图显示的是 HTML，它忽略了换行符或空白。为了检查数组的内容，就像 PHP 希望你看的那样，请检查页面的源代码——你可以在页面上右键单击来看到选项。

如果你需要从数组中删除一个元素，而不是添加或更新一个，你可以使用`unset`函数：

```php
<?php
$status = [
    'name' => 'James Potter',
    'status' => 'dead'
];
unset($status['status']);
print_r ($status);
```

新的`$status`数组只包含键名。

## 访问数组

访问数组就像指定键时更新它一样简单。为此，你需要了解列表是如何工作的。你已经知道列表在内部被视为一个具有顺序数字键的映射。第一个键总是 0；因此，具有*n*个元素的数组将具有从 0 到*n-1*的键。

你可以向给定的数组添加任何键，即使它之前只包含数字条目。问题出现在添加数字键时，后来你尝试向数组追加一个元素。你认为会发生什么？

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
$names['badguy'] = 'Voldemort';
$names[8] = 'Snape';
$names[] = 'McGonagall';
print_r($names);
```

最后一小段代码的结果如下：

```php
Array
(
    [0] => Harry
    [1] => Ron
    [2] => Hermione
    [badguy] => Voldemort
    [8] => Snape
    [9] => McGonagall
)
```

当尝试追加一个值时，PHP 将其插入到最后一个数字键之后，在这种情况下是`8`。

你可能已经自己想出来了，但你总是可以通过指定其键来打印数组的任何部分：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
print_r($names[1]); // prints 'Ron'

```

最后，尝试访问数组中不存在的键将返回 null 并抛出一个警告，因为 PHP 识别出你在代码中做了错误的事情。

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
var_dump($names[4]); // null and a PHP notice

```

## 空和 isset 函数

有两个有用的函数可以用来查询数组的内容。如果你想检查数组是否包含任何元素，你可以使用 `empty` 函数来检查它是否为空。实际上，这个函数也可以用于字符串，一个空字符串就是一个没有字符的字符串（' '）。`isset` 函数接受一个数组位置，并返回 `true` 或 `false`，取决于该位置是否存在：

```php
<?php
$string = '';
$array = [];
$names = ['Harry', 'Ron', 'Hermione'];
var_dump(empty($string)); // true
var_dump(empty($array)); // true
var_dump(empty($names)); // false
var_dump(isset($names[2])); // true
var_dump(isset($names[3])); // false
```

在前面的例子中，我们可以看到，一个没有元素的数组或一个没有字符的字符串，当被询问是否为空时，会返回 `true`，否则返回 `false`。当我们使用 `isset($names[2])` 来检查数组位置 2 是否存在时，我们得到 `true`，因为该键有一个值：`Hermione`。最后，`isset($names[3])` 评估为 `false`，因为该数组中不存在键 3。

## 在数组中搜索元素

可能，与数组一起使用最频繁的函数之一是 `in_array`。这个函数接受两个值，你想要搜索的值和数组。如果值在数组中，函数返回 `true`，否则返回 `false`。这非常有用，因为很多时候你从列表或映射中想要知道的是它是否包含一个元素，而不是知道它是否存在或其位置。

有时，`array_search` 函数更加有用。这个函数的工作方式与之前相同，只不过它返回的是一个布尔值，而不是找到的值对应的键，如果没有找到则返回 `false`。让我们看看这两个函数：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
$containsHermione = in_array('Hermione', $names);
var_dump($containsHermione); // true
$containsSnape = in_array('Snape', $names);
var_dump($containsSnape); // false
$wheresRon = array_search('Ron', $names);
var_dump($wheresRon); // 1
$wheresVoldemort = array_search('Voldemort', $names);
var_dump($wheresVoldemort); // false
```

## 排序数组

数组可以以不同的方式排序，所以很可能你需要的是与当前顺序不同的顺序。默认情况下，数组是按照元素被添加到数组的顺序排序的，但你也可以按键或值对其进行排序，包括升序和降序。此外，当按值排序数组时，你可以选择保留它们的键或者生成一个新的列表。

这些函数的完整列表可以在官方文档网站上找到，网址为 [`php.net/manual/en/array.sorting.php`](http://php.net/manual/en/array.sorting.php)，但在这里我们将展示其中最重要的几个：

| 名称 | 排序方式 | 维护键关联 | 排序顺序 |
| --- | --- | --- | --- |
| `sort` | 值 | 否 | 从低到高 |
| `rsort` | 值 | 否 | 从高到低 |
| `asort` | 值 | 是 | 从低到高 |
| `arsort` | 值 | 是 | 从高到低 |
| `ksort` | 键 | 是 | 从低到高 |
| `krsort` | 键 | 是 | 从高到低 |

这些函数始终接受一个参数，即数组，并且它们不返回任何内容。相反，它们直接对传递给它们的数组进行排序。让我们看看其中的一些函数：

```php
<?php
$properties = [
    'firstname' => 'Tom',
    'surname' => 'Riddle',
    'house' => 'Slytherin'
];
$properties1 = $properties2 = $properties3 = $properties;
sort($properties1);
var_dump($properties1);
asort($properties3);
var_dump($properties3);
ksort($properties2);
var_dump($properties2);
```

好吧，最后一个示例中有很多内容。首先，我们使用一些键值初始化一个数组，并将其分配给`$properties`。然后我们创建三个变量，它们是原始数组的副本——语法应该是直观的。我们为什么要这样做？因为我们如果对原始数组进行排序，我们就不会再有原始内容了。在这个特定的例子中，我们不想这样做，因为我们想看到不同的排序函数如何影响同一个数组。最后，我们执行了三种不同的排序，并打印出每个结果。浏览器应该显示如下：

```php
array(3) {
  [0]=>
  string(6) "Riddle"
  [1]=>
  string(9) "Slytherin"
  [2]=>
  string(3) "Tom"
}
array(3) {
  ["surname"]=>
  string(6) "Riddle"
  ["house"]=>
  string(9) "Slytherin"
  ["firstname"]=>
  string(3) "Tom"
}
array(3) {
  ["firstname"]=>
  string(3) "Tom"
  ["house"]=>
  string(9) "Slytherin"
  ["surname"]=>
  string(6) "Riddle"
}
```

第一个函数`sort`按字母顺序排序值。此外，如果您检查键，现在它们是列表中的数字，而不是原始键。相反，`asort`以相同的方式排序值，但保持键值关联。最后，`ksort`按键的字母顺序排序元素。

### 提示

**如何记住这么多函数名**

PHP 有很多函数助手，可以帮助您避免自己编写自定义函数，例如，它提供了多达 13 种不同的排序函数。您始终可以依赖官方文档。但当然，您可能希望编写不返回文档的代码。因此，这里有一些提示来记住每个排序函数的作用：

+   名称中的`a`表示**关联的**，因此将保留键值关联。

+   名称中的`r`表示**反向的**，所以顺序将是从高到低。

+   `k`表示**键**，因此排序将基于键而不是值。

## 其他数组函数

大约有 80 个与数组相关的不同函数。正如您所想象的那样，您甚至可能从未听说过其中的一些，因为它们具有非常特定的用途。完整的列表可以在[`php.net/manual/en/book.array.php`](http://php.net/manual/en/book.array.php)找到。

我们可以使用`array_keys`获取数组的键列表，以及使用`array_values`获取其值列表：

```php
<?php
$properties = [
    'firstname' => 'Tom',
    'surname' => 'Riddle',
    'house' => 'Slytherin'
];
$keys = array_keys($properties);
var_dump($keys);
$values = array_values($properties);
var_dump($values);
```

我们可以使用`count`函数来获取数组中的元素数量：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
$size = count($names);
var_dump($size); // 3
```

我们可以使用`array_merge`将两个或多个数组合并为一个：

```php
<?php
$good = ['Harry', 'Ron', 'Hermione'];
$bad = ['Dudley', 'Vernon', 'Petunia'];
$all = array_merge($good, $bad);
var_dump($all);
```

最后一个示例将打印以下数组：

```php
array(6) {
  [0]=>
  string(5) "Harry"
  [1]=>
  string(3) "Ron"
  [2]=>
  string(8) "Hermione"
  [3]=>
  string(6) "Dudley"
  [4]=>
  string(6) "Vernon"
  [5]=>
  string(7) "Petunia"
}
```

如您所见，第二个数组的键现在不同了，因为最初，这两个数组都有相同的数字键，而一个数组不能有两个相同的键的值。

# PHP 在 Web 应用程序中

尽管本章的主要目的是向您展示 PHP 的基础知识，但以参考手册的方式来做并不足够有趣，如果我们只是复制粘贴官方文档中的内容，您还不如自己去那里阅读。考虑到本书的主要目的和您的目标是用 PHP 编写网络应用程序，让我们尽快向您展示如何应用您所学的所有内容，以免您感到过于无聊。

为了做到这一点，我们现在将开始一段旅程，目标是构建一个在线书店。在最开始的时候，你可能看不到它的实用性，但这仅仅是因为我们还没有展示 PHP 能做的一切。

## 从用户获取信息

让我们先从构建一个主页开始。在这个页面上，我们要确定用户是在找书还是在路过。我们如何找到这个信息呢？目前最简单的方法是检查用户用来访问我们应用程序的 URL，并从中提取一些信息。

将此内容保存为你的`index.php`：

```php
<?php
$looking = isset($_GET['title']) || isset($_GET['author']);
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore</title>
</head>
<body>
 <p>You lookin'? <?php echo (int) $looking; ?></p>
    <p>The book you are looking for is</p>
    <ul>
        <li><b>Title</b>: <?php echo $_GET['title']; ?></li>
        <li><b>Author</b>: <?php echo $_GET['author']; ?></li>
    </ul>
</body>
</html>
```

现在访问链接，`http://localhost:8000/?author=HarperLee&title=To Kill a Mockingbird`。你会看到页面打印了你传递给 URL 的一些信息。

对于每个请求，PHP 会将来自查询字符串的所有参数存储在一个名为`$_GET`的数组中。数组的每个键都是参数的名称，其关联的值是参数的值。因此`$_GET`包含两个条目：`$_GET['author']`包含`Harper Lee`，而`$_GET['title']`的值是`To Kill a Mockingbird`。

在第一行高亮显示的代码中，我们给变量`$looking`赋了一个布尔值。如果`$_GET['title']`或`$_GET['author']`存在，该变量将为`true`，否则为`false`。紧接着，我们关闭了 PHP 标签，然后打印了一些 HTML，但正如你所看到的，我们实际上是在混合 HTML 和一些 PHP 代码。

这里还有另一条有趣的代码行，即第二行高亮显示的代码。在打印`$looking`的内容之前，我们进行了类型转换。**类型转换**意味着强制 PHP 将一种类型的值转换为另一种类型。将布尔值转换为整数意味着如果布尔值为`true`，则结果值为`1`，如果布尔值为`false`，则结果值为`0`。由于`$_GET`包含有效的键，`$looking`为`true`，因此页面显示了一个`1`。

如果我们尝试在不发送任何信息的情况下访问相同的页面，如`http://localhost:8000`，浏览器将显示**你在找一本书吗？0**。根据你的 PHP 配置设置，你会看到两条通知消息，抱怨你正在尝试访问不存在的数组键。

### 注意

**类型转换与类型转换**

我们已经知道，当 PHP 需要特定类型的变量时，它会尝试将其转换，这被称为类型转换。但 PHP 非常灵活，所以有时你必须指定你需要的类型。当使用`echo`打印内容时，PHP 会尝试将得到的所有内容转换为字符串。由于布尔值`false`的字符串形式是一个空字符串，这对我们的应用程序来说可能没有用。首先将布尔值转换为整数可以确保我们能看到一个值，即使它只是一个 0。

## HTML 表单

HTML 表单是收集用户信息最受欢迎的方式之一。它们由一系列字段组成——在 HTML 世界中称为 `input` 字段——以及一个最终的 `submit` 按钮。在 HTML 中，`form` 标签包含两个属性：`action` 指定了表单将被提交的位置，而 `method` 指定了表单将使用的 HTTP 方法（GET 或 POST）。让我们看看它是如何工作的。将以下内容保存为 `login.html` 并访问 `http://localhost:8000/login.html`。

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore - Login</title>
</head>
<body>
    <p>Enter your details to login:</p>
 <form action="authenticate.php" method="post">
        <label>Username</label>
 <input type="text" name="username" />
        <label>Password</label>
 <input type="password" name="password" />
 <input type="submit" value="Login"/>
    </form>
</body>
</html>
```

前面代码中定义的表单包含两个字段，一个用于用户名，一个用于密码。你可以看到它们通过属性 `name` 来标识。如果你尝试提交这个表单，浏览器将显示一个 **页面未找到** 消息，因为它正在尝试访问 `http://localhost:8000/authenticate.php`，而 web 服务器找不到它。那么让我们创建它：

```php
<?php
$submitted = !empty($_POST);
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore</title>
</head>
<body>
    <p>Form submitted? <?php echo (int) $submitted; ?></p>
    <p>Your login info is</p>
    <ul>
        <li><b>username</b>: <?php echo $_POST['username']; ?></li>
        <li><b>password</b>: <?php echo $_POST['password']; ?></li>
    </ul>
</body>
</html>
```

与 `$_GET` 类似，`$_POST` 是一个包含通过 POST 接收的参数的数组。在这段代码的前一部分，我们首先检查该数组是否为空——注意 `!` 操作符。之后，我们只需显示接收到的信息，就像在 `index.php` 中一样。请注意，`$_POST` 数组的键是每个输入字段的参数名称的值。

## 使用 cookies 持久化数据

当我们希望浏览器记住一些数据，比如在你的 web 应用程序中你是否已登录，你的基本信息等等，我们使用 **cookies**。Cookies 存储在客户端，并在请求时作为头部信息发送到服务器。由于 PHP 面向 web 应用程序，它允许你以非常简单的方式管理 cookies。

关于 cookies 和 PHP，你需要了解一些事情。你可以使用 `setcookie` 函数来写入 cookies，该函数接受多个参数：

+   一个有效的 cookie 名称作为字符串。

+   cookie 的值——只能是字符串或可以转换为字符串的值。此参数是可选的，如果没有设置，PHP 实际上会删除 cookie。

+   过期时间作为时间戳。如果没有设置，cookie 将在浏览器关闭时被删除。

### 注意

**时间戳**

计算机使用不同的方式来描述日期和时间，其中最常见的一种，尤其是在 Unix 系统上，就是使用时间戳。它们表示自 1970 年 1 月 1 日以来经过的秒数。例如，表示 2015 年 10 月 4 日下午 6:30 的时间戳将是 1,443,954,637，这是自该日期以来的秒数。

你可以使用 PHP 的 `time` 函数获取当前的时间戳。

与安全相关的其他参数也存在，但它们超出了本节的范围。此外，请注意，你只能在应用程序没有之前的输出之前设置 cookies，也就是说，在 HTML、`echo` 调用以及任何其他发送输出的类似函数之前。

要读取客户端发送给我们的 cookies，我们只需访问数组 `$_COOKIE`。它像其他两个数组一样工作，因此数组的键将是 cookies 的名称，数组的值将是它们的值。

非常常见的 cookies 用法是验证用户身份。根据您应用程序所需的 安全级别，有几种不同的方法可以实现。让我们尝试实现一个非常简单——尽管不安全的例子（不要用于实际 Web 应用程序）。保持 HTML 不变，更新你的`authenticate.php`文件的 PHP 部分，内容如下：

```php
<?php
setcookie('username', $_POST['username']);
$submitted = !empty($_POST);
?>
```

同样，对`index.php`中的`body`标签做同样的处理：

```php
<body>
 <p>You are <?php echo $_COOKIE['username']; ?></p>
    <p>Are you looking for a book? <?php echo (int) $lookingForBook; ?></p>
    <p>The book you are looking for is</p>
    <ul>
        <li><b>Title</b>: <?php echo $_GET['title']; ?></li>
        <li><b>Author</b>: <?php echo $_GET['author']; ?></li>
    </ul>
</body>
```

如果你再次访问`http://localhost:8000/login.html`，尝试登录，打开一个新的标签页（在同一浏览器中），并转到主页`http://localhost:8000`，你会看到浏览器仍然记得你的用户名。

## 其他超全局变量

`$_GET`、`$_POST`和`$_COOKIE`是称为**超全局变量**的特殊变量。还有其他超全局变量，如`$_SERVER`或`$_ENV`，它们会给你提供额外的信息。第一个显示了关于标题、访问的路径和其他与请求相关的信息。第二个包含了运行应用程序的机器的环境变量。你可以在[`php.net/manual/es/language.variables.superglobals.php`](http://php.net/manual/es/language.variables.superglobals.php)上看到这些数组的完整列表及其元素。

通常，使用超全局变量是有用的，因为它允许你从用户、浏览器、请求等获取信息。这对于编写需要与用户交互的 Web 应用程序来说具有无法估量的价值。但是，权力越大，责任越大，使用这些数组时你应该非常小心。这些值中的大多数都来自用户本身，这可能导致安全问题。

# 控制结构

到目前为止，我们的文件是逐行执行的。正因为如此，我们在某些场景下收到了通知，例如当数组不包含我们正在寻找的内容时。如果我们能选择执行哪些行会不是很好？控制结构来拯救我们！

**控制结构**就像一个交通分流标志。它根据一些预定义的条件来指导执行流程。有不同种类的控制结构，但我们可以将它们分为**条件**和**循环**。条件允许我们选择是否执行一个语句。循环会根据需要多次执行一个语句。让我们来看看每一个。

## 条件

条件评估一个布尔表达式，即返回值的某种东西。如果表达式为`true`，它将执行其代码块内的所有内容。代码块是一组由`{}`包围的语句。让我们看看它是如何工作的：

```php
<?php
echo "Before the conditional.";
if (4 > 3) {
 echo "Inside the conditional.";
}
if (3 > 4) {
 echo "This will not be printed.";
}
echo "After the conditional.";
```

在前面的代码片段中，我们使用了两个条件。条件是通过关键字`if`后跟括号中的布尔表达式和代码块来定义的。如果表达式为`true`，它将执行该块，否则将跳过它。

你可以通过添加关键字`else`来增强条件语句的威力。这告诉 PHP，如果前面的条件没有得到满足，则执行一些代码块。让我们看看一个例子：

```php
if (2 > 3) {
    echo "Inside the conditional.";
} else {
 echo "Inside the else.";
}
```

上述示例将在`if`的条件没有得到满足时执行`else`中的代码。

最后，你还可以添加一个`elseif`关键字，后面跟着另一个条件和一段代码，以继续向 PHP 请求更多条件。你可以在`if`之后添加任意数量的`elseif`。如果你添加了`else`，它必须是条件链中的最后一个。同时，请注意，一旦 PHP 找到一个解析为`true`的条件，它将停止评估剩余的条件。

```php
<?php
if (4 > 5) {
    echo "Not printed";
} elseif (4 > 4) {
    echo "Not printed";
} elseif (4 == 4) {
 echo "Printed.";
} elseif (4 > 2) {
    echo "Not evaluated.";
} else {
    echo "Not evaluated.";
}
if (4 == 4) {
    echo "Printed";
}
```

在最后一个例子中，第一个评估为`true`的条件是突出显示的那个。在那之后，PHP 不会评估更多的条件，直到一个新的`if`开始。

带着这些知识，让我们尝试清理我们的应用程序，只在需要时执行语句。将此代码复制到你的`index.php`文件中：

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore</title>
</head>
<body>
    <p>
<?php
if (isset($_COOKIE[username'])) {
 echo "You are " . $_COOKIE['username'];
} else {
 echo "You are not authenticated.";
}
?>
    </p>
<?php
if (isset($_GET['title']) && isset($_GET['author'])) {
?>
 <p>The book you are looking for is</p>
 <ul>
 <li><b>Title</b>: <?php echo $_GET['title']; ?></li>
 <li><b>Author</b>: <?php echo $_GET['author']; ?></li>
 </ul>
<?php
} else {
?>
 <p>You are not looking for a book?</p>
<?php
}
?>
</body>
</html>
```

在这段新代码中，我们以两种不同的方式混合了条件和 HTML 代码。第一种是打开一个 PHP 标签，并添加一个`if…else`子句，该子句将使用`echo`打印我们是否已认证。条件内没有合并 HTML，这使得它很清晰。

第二种选项——第二个突出显示的块——展示了一个更丑陋但有时必要的解决方案。当你需要打印大量的 HTML 代码时，`echo`并不那么方便，最好是关闭 PHP 标签，打印所有需要的 HTML，然后再打开标签。你甚至可以在`if`子句的代码块内这样做，就像你可以在代码中看到的那样。

### 注意

**混合 PHP 和 HTML**

如果你觉得我们最后编辑的文件看起来相当丑陋，你是对的。混合 PHP 和 HTML 会让人困惑，你应该避免这样做。在第六章，*适应 MVC*中，我们将看到如何正确地做事。

让我们再编辑一下`authenticate.php`文件，因为它正在尝试访问可能不存在的`$_POST`条目。文件的新内容如下：

```php
<?php
$submitted = isset($_POST['username']) && isset($_POST['password']);
if ($submitted) {
    setcookie('username', $_POST['username']);
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore</title>
</head>
<body>
<?php if ($submitted): ?>
    <p>Your login info is</p>
    <ul>
        <li><b>username</b>: <?php echo $_POST['username']; ?></li>
        <li><b>password</b>: <?php echo $_POST['password']; ?></li>
    </ul>
<?php else: ?>
    <p>You did not submit anything.</p>
<?php endif; ?>
</body>
</html>
```

这段代码也包含条件语句，这是我们已知的。我们正在设置一个变量来知道是否提交了登录，如果是的话，则设置 cookies。但突出显示的行展示了使用 HTML 包含条件语句的新方法。这使得在处理 HTML 代码时代码更易读，避免了使用`{}`，而是使用`:`和`endif`。两种语法都是正确的，你应该在每个情况下使用你认为更易读的一种。

## Switch…case

与`if…else`类似的另一个控制结构是`switch…case`。这个结构只评估一个表达式，并根据其值执行相应的代码块。让我们看看一个例子：

```php
<?php
switch ($title) {
    case 'Harry Potter':
        echo "Nice story, a bit too long.";
        break;
    case 'Lord of the Rings':
        echo "A classic!";
        break;
    default:
        echo "Dunno that one.";
        break;
}
```

`switch`子句接受一个表达式，在这个例子中是一个变量，然后定义了一系列的情况。当某个情况与表达式的当前值匹配时，PHP 会执行其内部的代码。一旦 PHP 找到一个`break`语句，它就会退出`switch…case`。如果没有适合表达式的任何情况，PHP 会执行默认情况（如果有的话），但这不是必须的。

你还需要知道，如果你想退出`switch…case`，break 是强制性的。如果你没有指定任何，PHP 会继续执行语句，即使它遇到了新的情况。让我们看一个类似的例子，但没有 break 语句：

```php
<?php
$title = 'Twilight';
switch ($title) {
    case 'Harry Potter':
        echo "Nice story, a bit too long.";
    case 'Twilight':
        echo 'Uh...';
    case 'Lord of the Rings':
        echo "A classic!";
    default:
        echo "Dunno that one.";
}
```

如果你在这个浏览器中测试这段代码，你会看到它打印出**嗯...经典！不知道这个**。PHP 发现第二个情况是有效的，因此执行了其内容。但是因为没有 break 语句，它会一直执行到末尾。这可能在某些情况下是期望的行为，但通常不是，所以使用时要小心！

## 循环

**循环**是允许你多次执行某些语句的控制结构，你可以根据需要多次执行。你可能会在多种不同的场景中使用它们，但最常见的一个是在与数组交互时。例如，想象你有一个包含元素的数组，但你不知道里面有什么。你想要打印出所有的元素，所以你会遍历它们。

有四种类型的循环。每种类型都有自己的使用场景，但一般来说，你可以将一种类型的循环转换为另一种类型。让我们仔细看看它们。

### While

`while`循环是最简单的循环。它会在要评估的表达式返回`false`之前执行代码块。让我们看一个例子：

```php
<?php
$i = 1;
while ($i < 4) {
    echo $i . " ";
    $i++;
}
```

在前面的例子中，我们定义了一个值为`1`的变量。然后我们有一个`while`子句，其中要评估的表达式是`$i < 4`。这个循环会执行代码块的内容，直到该表达式为`false`。正如你所看到的，在循环内部，我们每次都会将`$i`的值增加 1，所以循环会在 4 次迭代后结束。检查那个脚本的输出，你会看到"0 1 2 3"。最后打印的值是 3，所以在那时`$i`的值是 3。之后，我们将其值增加到 4，所以当`while`子句评估`$i < 4`时，结果是`false`。

### 注意

**循环和无限循环**

`while`循环最常见的一个问题就是创建无限循环。如果你没有在`while`循环内添加任何代码来更新`while`表达式中考虑的任何变量，使得它在某个时刻可以返回`false`，PHP 将永远不会退出循环！

### Do…while

`do…while`循环在某种程度上与`while`循环非常相似，因为它每次都会评估一个表达式，并且会执行代码块直到该表达式为`false`。唯一的区别在于，当这个表达式被评估时，`while`子句会在执行代码之前评估这个表达式，所以有时候，如果表达式第一次评估就为`false`，我们甚至可能不会进入循环。另一方面，`do…while`在执行其代码块之后评估这个表达式，所以即使表达式一开始就是`false`，循环至少也会执行一次。

```php
<?php
echo "with while: ";
$i = 1;
while ($i < 0) {
    echo $i . " ";
    $i++;
}
echo "with do-while: ";
$i = 1;
do {
 echo $i . " ";
 $i++;
} while ($i < 0);

```

上述代码定义了两个具有相同表达式和代码块的循环，但如果你执行它们，你会看到只有`do…while`中的代码被执行。在这两种情况下，表达式从一开始就是`false`，所以`while`甚至没有进入循环，而`do…while`进入循环一次。

### For

`for`循环是四种循环中最复杂的。它定义了一个初始化表达式、一个退出条件和迭代结束表达式。当 PHP 第一次遇到循环时，它会执行初始化表达式所定义的内容。然后，它会评估退出条件，如果它解析为`true`，它会进入循环。在执行循环内的所有内容之后，它会执行迭代结束表达式。一旦完成，它会再次评估结束条件，通过循环代码和迭代结束表达式，直到它评估为`false`。就像往常一样，一个例子会澄清它：

```php
<?php
for ($i = 1; $i < 10; $i++) {
    echo $i . " ";
}
```

初始化表达式是`$i = 1`，并且只执行第一次。退出条件是`$i < 10`，它在每次迭代的开始时被评估。迭代结束表达式是`$i++`，它在每次迭代的结束时执行。这个例子会打印从 1 到 9 的数字。`for`循环的另一种更常见的用法是与数组一起使用：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
for ($i = 0; $i < count($names); $i++) {
    echo $names[$i] . " ";
}
```

在这个例子中，我们有一个名字数组。由于它被定义为列表，它的键将是 0、1 和 2。循环将变量`$i`初始化为 0，并且它迭代直到`$i`的值不小于数组中的元素数量，即 3。在第一次迭代中，`$i`是 0，在第二次迭代中是 1，在第三次迭代中是 2。当`$i`是 3 时，它将不会进入循环，因为退出条件评估为`false`。

在每次迭代中，我们打印数组中位置`$i`的内容，因此这段代码的结果将是数组中的所有三个名字。

### Tip

**注意退出条件**

很常见的是设置一个不是我们真正需要的退出条件，尤其是在处理数组时。记住，如果数组是一个列表，它从 0 开始，所以一个有三个元素的数组将有条目 0、1 和 2。将退出条件定义为`$i <= count($array)`会在你的代码中引起错误，因为当`$i`是 3 时，它也满足退出条件，并尝试访问不存在的键 3。

### Foreach

最后，但同样重要的是，循环类型是 `foreach`。这个循环仅适用于数组，它允许您完全遍历一个数组，即使您不知道它的键。语法有两种选择，如以下示例所示：

```php
<?php
$names = ['Harry', 'Ron', 'Hermione'];
foreach ($names as $name) {
    echo $name . " ";
}
foreach ($names as $key => $name) {
    echo $key . " -> " . $name . " ";
}
```

`foreach` 循环接受一个数组——在这个例子中是 `$names`——并指定一个变量，该变量将包含数组的条目值。您可以看到我们不需要指定任何结束条件，因为 PHP 会知道何时遍历完数组。可选地，您可以指定一个变量，它包含每个迭代的键，就像第二个循环中那样。

`foreach` 循环与映射也很有用，其中键不一定是数字。PHP 遍历数组的顺序将与您在数组中插入内容的顺序相同。

让我们在我们的应用程序中使用一些循环。我们想在主页上显示可用的书籍。我们有一个书籍列表在数组中，所以我们将不得不使用 `foreach` 循环遍历它们，并从每本书中打印一些信息。将以下代码添加到 `index.php` 中的 `body` 标签：

```php
<?php endif;
    $books = [
        [
            'title' => 'To Kill A Mockingbird',
            'author' => 'Harper Lee',
            'available' => true,
            'pages' => 336,
            'isbn' => 9780061120084
        ],
        [
            'title' => '1984',
            'author' => 'George Orwell',
            'available' => true,
            'pages' => 267,
            'isbn' => 9780547249643
        ],
        [
            'title' => 'One Hundred Years Of Solitude',
            'author' => 'Gabriel Garcia Marquez',
            'available' => false,
            'pages' => 457,
            'isbn' => 9785267006323
        ],
    ];
?>
<ul>
<?php foreach ($books as $book): ?>
 <li>
 <i><?php echo $book['title']; ?></i>
 - <?php echo $book['author']; ?>
<?php if (!$book['available']): ?>
 <b>Not available</b>
<?php endif; ?>
 </li>
<?php endforeach; ?>
    </ul>
```

突出的代码显示了使用 `:` 符号的 `foreach` 循环，当与 HTML 混合使用时更好。它遍历所有的 `$books` 数组，并为每本书打印一些作为 HTML 列表的信息。请注意，我们还在循环内部有一个条件语句，这是完全正常的。当然，这个条件语句将为数组中的每个条目执行，所以你应该尽可能使循环的代码块保持简单。

# 函数

函数是一段可重用的代码块，给定输入，执行一些操作，并可选地返回一些结果。您已经知道几个预定义的函数，如 `empty`、`in_array` 或 `var_dump`。这些函数随 PHP 一起提供，因此您不必重新发明轮子，但您可以非常容易地创建自己的函数。当您确定应用程序中需要多次执行的部分或只是要封装一些功能时，您可以定义函数。

## 函数声明

声明一个函数意味着将其写下来以便以后使用。一个函数有一个名称，接受一些参数，并包含一段代码块。可选地，它可以定义要返回的值的类型。函数的名称必须遵循与变量名称相同的规则，即它必须以字母或下划线开头，并且可以包含任何字母、数字或下划线。它不能是保留字。

让我们看看一个简单的例子：

```php
function addNumbers($a, $b) {
    $sum = $a + $b;
    return $sum;
}
$result = addNumbers(2, 3);

```

前一个函数的名称是 `addNumbers`，它接受两个参数：`$a` 和 `$b`。代码块定义了一个新变量 `$sum`，它是两个参数的和，然后使用 `return` 返回其内容。为了使用此函数，您只需按其名称调用它，同时发送所有必需的参数，如突出显示的行所示。

PHP 不支持**重载函数**。重载指的是声明两个或更多具有相同名称但参数不同的函数的能力。正如你所见，你可以声明参数而不知道它们的类型，所以 PHP 无法决定使用哪个函数。

另一个需要注意的重要事项是**变量作用域**。我们在代码块内部声明了一个变量`$sum`，所以一旦函数结束，该变量将不再可访问。这意味着在函数内部声明的变量的作用域只是函数本身。此外，如果你在函数外部声明了一个变量`$sum`，它将完全不受影响，因为函数无法访问那个变量，除非我们将其作为参数传递。

## 函数参数

函数通过参数从外部获取信息。你可以定义任意数量的参数——包括 0（没有）。这些参数至少需要一个名称，以便在函数内部使用；不能有两个具有相同名称的参数。在调用函数时，你需要按照声明的顺序发送参数。

一个函数可能包含**可选参数**，也就是说，你不必为这些参数提供值。在声明函数时，你需要为这些参数提供一个默认值。因此，如果用户没有提供值，函数将使用默认值。

```php
function addNumbers($a, $b, $printResult = false) {
    $sum = $a + $b;
    if ($printResult) {
        echo 'The result is ' . $sum;
    }
    return $sum;
}

$sum1 = addNumbers(1, 2);
$sum1 = addNumbers(3, 4, false);
$sum1 = addNumbers(5, 6, true); // it will print the result
```

最后一个示例中的这个新函数接受两个必需参数和一个可选参数。可选参数的默认值是`false`，然后在函数内部正常使用。如果用户将`true`作为第三个参数提供，函数将打印求和的结果，这只有在函数被调用的第三次时才会发生。对于前两次，`$printResult`被设置为`false`。

函数接收到的参数只是用户提供的值的副本。这意味着如果你在函数内部修改这些参数，它将不会影响原始值。这个特性被称为按值传递参数。让我们看一个例子：

```php
function modify($a) {
    $a = 3;
}

$a = 2;
modify($a);
var_dump($a); // prints 2
```

我们声明一个变量`$a`，其值为`2`，然后调用`modify`方法，传递那个`$a`。`modify`方法修改了参数`$a`，将其值设置为`3`，但这不会影响`$a`的原始值，正如你可以从`var_dump`中看到的那样，它仍然是`2`。

如果你想要实际更改在调用中使用的原始变量的值，你需要按引用传递参数。要做到这一点，你需要在声明函数时在参数前添加一个和号(`&`)：

```php
function modify(&$a) {
    $a = 3;
}
```

现在，在调用`modify`函数时，`$a`将始终是`3`。

### 注意

**按值传递参数与按引用传递参数**

PHP 允许你这样做，实际上，一些 PHP 的原生函数使用引用传递参数。记得数组排序函数吗？它们没有返回排序后的数组，而是对提供的数组进行了排序。但使用引用传递参数是一种让开发者困惑的方式。通常，当某人使用一个函数时，他们期望得到一个结果，并且他们不希望提供的参数被修改。所以尽量避免这样做；人们会感激的！

## 返回语句

你可以在你的函数内部有任意多的 `return` 语句，但 PHP 会在找到第一个 `return` 语句时退出函数。这意味着如果你有两个连续的 `return` 语句，第二个将永远不会被执行。尽管如此，如果它们在条件语句内部，多个 `return` 语句仍然可能是有用的。将此函数添加到你的 `functions.php` 文件中：

```php
function loginMessage() {
    if (isset($_COOKIE['username'])) {
        return "You are " . $_COOKIE['username'];
    } else {
        return "You are not authenticated.";
    }
}
```

让我们在你的 `index.php` 文件中使用最后一个示例，通过替换高亮内容（注意，为了节省一些纸张，我将大部分完全没有更改的代码替换为 `//…`）：

```php
//...
<body>
 <p><?php echo loginMessage(); ?></p>
<?php if (isset($_GET['title']) && isset($_GET['author'])): ?>
//...
```

此外，如果你不希望函数返回任何内容，你可以省略 `return` 语句。在这种情况下，函数将在到达代码块末尾时结束。

## 类型提示和返回类型

随着 PHP 7 的发布，该语言允许开发者更具体地说明函数获取和返回的内容。你可以——始终可选地——指定函数需要的参数类型（**类型提示**），以及函数将返回的类型（**返回类型**）。让我们先看一个例子：

```php
<?php

declare(strict_types=1);

function addNumbers(int $a, int $b, bool $printSum): int {
    $sum = $a + $b;
    if ($printSum) {
        echo 'The sum is ' . $sum;
    }
    return $sum;
}

addNumbers(1, 2, true);
addNumbers(1, '2', true); // it fails when strict_types is 1
addNumbers(1, 'something', true); // it always fails
```

此前函数声明了参数需要是整数、整数和布尔值，并且结果将是整数。现在，你知道 PHP 有类型转换，所以它通常可以将一个类型的值转换为另一个类型的等效值，例如，字符串 "2" 可以用作整数 2。为了阻止 PHP 在函数的参数和结果上使用类型转换，你可以声明指令 `strict_types`，如第一行高亮所示。此指令必须在每个你想强制执行此行为的文件顶部声明。

三个调用工作如下：

+   第一次调用发送两个整数和一个布尔值，这正是函数所期望的，所以无论 `strict_types` 的值如何，它总是会工作。

+   第二次调用发送一个整数、一个字符串和一个布尔值。该字符串有一个有效的整数值，所以如果 PHP 被允许使用类型转换，调用将正常解决。但在本例中，它将因为文件顶部的声明而失败。

+   第三个调用总是会失败，因为字符串 "something" 无法转换为有效的整数。

让我们在我们的项目中尝试使用一个函数。在我们的`index.php`中，有一个`foreach`循环，它遍历书籍并打印它们。循环中的代码有点难以理解，因为它混合了 HTML 和 PHP，还有一个条件语句。让我们尝试将循环中的逻辑抽象成一个函数。首先，创建一个新的`functions.php`文件，内容如下：

```php
<?php
function printableTitle(array $book): string {
    $result = '<i>' . $book['title'] . '</i> - ' . $book['author'];
    if (!$book['available']) {
        $result .= ' <b>Not available</b>';
    }
    return $result;
}
```

这个文件将包含我们的函数。第一个函数`printableTitle`接受一个表示书籍的数组，并构建一个包含书籍在 HTML 中良好表示的字符串。代码与之前相同，只是封装在一个函数中。

现在`index.php`将需要包含`functions.php`文件，然后在循环中使用该函数。让我们看看如何：

```php
<?php require_once 'functions.php' ?>
<!DOCTYPE html>
<html lang="en">

//...

?>
    <ul>
<?php foreach ($books as $book): ?>
 <li><?php echo printableTitle($book); ?> </li>
<?php endforeach; ?>
    </ul>

//...
```

好吧，现在我们的循环看起来干净多了，对吧？此外，如果我们需要在其他地方打印书籍的标题，我们可以重用该函数而不是复制代码！

# 文件系统

如你所见，PHP 自带了许多原生函数，这些函数可以帮助你以比其他语言更简单的方式管理数组和字符串。文件系统是 PHP 试图使其尽可能简单化的另一个领域。函数列表扩展到超过 80 个不同的函数，所以我们在这里只介绍你更有可能使用的那些。

## 读取文件

在我们的代码中，我们定义了一个书籍列表。到目前为止，我们只有三本书，但你可以猜到，如果我们想使这个应用程序有用，列表将增长得多。在代码中存储信息根本不实用，所以我们必须开始考虑外部化它。

如果我们考虑将代码与数据分离，就没有必要继续使用 PHP 数组来定义书籍。使用一个不那么语言限制的系统将允许不知道 PHP 的人编辑文件的内容。为此有很多解决方案，比如 CSV 或 XML 文件，但如今，在 Web 应用程序中表示数据最常用的系统之一是 JSON。PHP 允许你使用几个函数将数组转换为 JSON，反之亦然：`json_encode`和`json_decode`。简单，对吧？

将以下内容保存到`books.json`：

```php
[
    {
        "title": "To Kill A Mockingbird",
        "author": "Harper Lee",
        "available": true,
        "pages": 336,
        "isbn": 9780061120084
    },
    {
        "title": "1984",
        "author": "George Orwell",
        "available": true,
        "pages": 267,
        "isbn": 9780547249643
    },
    {
        "title": "One Hundred Years Of Solitude",
        "author": "Gabriel Garcia Marquez",
        "available": false,
        "pages": 457,
        "isbn": 9785267006323
    }
]
```

```php
file_get_contents, and transform it to a PHP array with json_decode. Replace the array with these two lines:
```

```php
$booksJson = file_get_contents('books.json');
$books = json_decode($booksJson, true);
```

只用一个函数，我们就能将 JSON 文件中的所有内容存储在一个变量中作为字符串。使用该函数，我们将这个 JSON 字符串转换为一个数组。`json_decode`的第二个参数告诉 PHP 将其转换为数组，否则它将使用对象，我们还没有介绍过这些对象。

当在 PHP 函数中引用文件时，你需要知道是使用绝对路径还是相对路径。当使用相对路径时，PHP 会尝试在 PHP 脚本所在的同一目录中查找文件。如果找不到，PHP 会尝试在`include_path`指令中定义的其他目录中查找，但这是你想要避免的。相反，你可以使用绝对路径，这是一种确保引用不会被误解的方法。让我们看看两个例子：

```php
$booksJson = file_get_contents('/home/user/bookstore/books.json');
$booksJson = file_get_contents(__DIR__, '/books.json');
```

常量`__DIR__`包含当前 PHP 文件的目录名，如果我们将其添加到文件名前缀，我们将得到一个绝对路径。实际上，尽管你可能认为亲自写下整个路径更好，但使用`__DIR__`允许你将应用程序移动到任何其他位置，而无需在代码中做任何更改，因为其内容将始终与脚本的目录相匹配，而第一个示例中的硬编码路径将不再有效。

## 写文件

让我们在应用程序中添加一些功能。想象一下，我们想要允许用户取走他们正在寻找的书籍，但前提是它必须是可用的。如果你记得，我们通过查询字符串来识别书籍。这并不太实用，所以让我们通过在书籍列表中添加链接来帮助用户，当你点击链接时，查询字符串将包含该书籍的信息。

```php
<?php require_once 'functions.php' ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Bookstore</title>
</head>
<body>
    <p><?php echo loginMessage(); ?></p>
<?php
$booksJson = file_get_contents('books.json');
$books = json_decode($booksJson, true);
if (isset($_GET['title'])) {
 echo '<p>Looking for <b>' . $_GET['title'] . '</b></p>';
} else {
 echo '<p>You are not looking for a book?</p>';
}
?>
    <ul>
<?php foreach ($books as $book): ?>
 <li>
 <a href="?title=<?php echo $book['title']; ?>">
 <?php echo printableTitle($book); ?>
 </a>
 </li>
<?php endforeach; ?>
    </ul>
</body>
</html>
```

如果你尝试在浏览器中运行前面的代码，你会看到列表中包含链接，点击它们后，页面会刷新，查询字符串中包含新的标题。现在让我们检查书籍是否可用，如果可用，我们将更新其可用字段为`false`。在`functions.php`中添加以下函数：

```php
function bookingBook(array &$books, string $title): bool {
    foreach ($books as $key => $book) {
        if ($book['title'] == $title) {
            if ($book['available']) {
                $books[$key]['available'] = false;
                return true;
            } else {
                return false;
            }
        }
    }
    return false;
}
```

我们必须注意，当代码开始变得复杂时。这个函数接受一个书籍数组和标题，并返回一个布尔值，如果可以预订则为`true`，否则为`false`。此外，书籍数组是通过引用传递的，这意味着对该数组所做的任何更改都会影响原始数组。尽管我们之前曾劝阻这样做，但在这种情况下，这是一个合理的做法。

我们遍历整个书籍数组，每次询问当前书籍的标题是否与我们要找的标题匹配。只有当它是`true`时，我们才会检查书籍是否可用。如果可用，我们将更新可用性为`false`并返回`true`，表示我们已预订了书籍。如果书籍不可用，我们只返回`false`。

最后，请注意，`foreach`定义了`$key`和`$book`。我们这样做是因为`$book`变量是`$books`数组的副本，如果我们编辑它，原始的将不会受到影响。相反，我们要求该书籍的键也一起提供，因此当编辑数组时，我们使用`$books[$key]`而不是`$book`。

我们可以从`index.php`文件中使用此函数：

```php
//...
    echo '<p>Looking for <b>' . $_GET['title'] . '</b></p>';
 if (bookingBook($books, $_GET['title'])) {
 echo 'Booked!';
 } else {
 echo 'The book is not available...';
 }
} else {
//...
```

在浏览器中试一试。通过点击可用的书籍，你会看到**已预订**的消息。我们几乎完成了！我们只是缺少最后一步：将此信息持久化回文件系统。为了做到这一点，我们必须构建新的 JSON 内容，并将其写回`books.json`文件。当然，只有当书籍可用时才这样做。

```php
function updateBooks(array $books) {
    $booksJson = json_encode($books);
    file_put_contents(__DIR__ . '/books.json', $booksJson);
}
```

`json_encode`函数与`json_decode`相反：它接受一个数组或任何其他变量，并将其转换为 JSON。`file_put_contents`函数用于将内容写入作为第一个参数引用的文件，第二个参数是发送的内容。你知道如何使用这个函数吗？

```php
//...
if (bookingBook($books, $_GET['title'])) {
    echo 'Booked!';
 updateBooks($books);
} else {
    echo 'The book is not available...';
}
//...
```

### 注意

**文件与数据库**

将信息存储在 JSON 文件中比将其直接放在代码中要好，但这还不是最佳选择。在第五章中，*使用数据库*，你将学习如何将应用的数据存储在数据库中，这是一个更好的解决方案。

## 其他文件系统函数

如果你想要使你的应用更加健壮，你可以检查`books.json`文件是否存在，你是否有读写权限，以及之前的内容是否是有效的 JSON。你可以使用一些 PHP 函数来完成这个任务：

+   `file_exists`：此函数接受文件的路径，并返回一个布尔值：当文件存在时返回`true`，否则返回`false`。

+   `is_writable`：此函数与`file_exists`的工作方式相同，但它会检查文件是否可写。

你可以在[`uk1.php.net/manual/en/book.filesystem.php`](http://uk1.php.net/manual/en/book.filesystem.php)找到完整的函数列表。你可以找到用于移动、复制或删除文件、创建目录、设置权限和所有权等功能的函数。

# 摘要

在本章中，我们通过编写简单的示例来实践，学习了过程式 PHP 的所有基础知识。你现在知道如何使用变量和数组与控制结构和函数一起使用，如何从 HTTP 请求中获取信息，以及如何与其他事物交互，例如与文件系统交互。

在下一章，我们将学习其他最常用的范式：面向对象编程（OOP）。这使我们的应用编写更加整洁和结构良好又近了一步。
