# 第四章 JavaScript

到目前为止，我们已经学习了三个章节，掌握了两种语言：HTML 和 CSS。它们被用来创建网页，为其添加内容，并为其添加样式。在本章中，我们将学习第三种语言，**JavaScript**，它被用来编程你的网页，并为其添加活力。在相当长的一段时间里，JavaScript 被一些人视为二等公民。如果曾经有这种说法，那么现在肯定不再是这样了。在我们开始学习 JavaScript 之前，让我们先了解一下编程的世界。

# 编程 101

没有什么是智能计算机。计算机是一种能够执行少量指令的设备，但可以非常快地执行。这样的指令可能包括从一个地方取一个值，从另一个地方取另一个值，将这两个值相加，并将结果存储在第三个地方。**程序**是一系列这样的指令，以某种逻辑和结构化的顺序写下，并以人类可读的格式。这种格式被称为**编程语言**。一个智能的程序会将我们描述的无用设备变成计算器。

对于许多编程语言来说，存在一种特殊的程序，称为**编译器**。它将程序从人类可读的格式（通常是一个称为源代码的文本文件）翻译成计算机可以理解的格式，通常是二进制可执行文件。

你的计算机或平板电脑包含许多不同种类的程序。有一个程序来管理计算机，有程序来管理程序，还有其他程序来创建程序，有一个程序让你输入命令，这些命令反过来又是程序的名称，等等。最后一个碰巧也是一种编程语言，有时被称为**shell**或**命令解释器**。

如果你将一些命令组合到一个文本文件中，你又一次得到了一个程序：一个 shell 脚本。这是一个解释型语言而不是编译型语言的例子。

但这是**编程 101**，而不是**计算机 101**。看看以下用一种虚构的编程语言编写的源代码行：

```php
a = 1;
b = 2;
c = a + b;
printnumber (c);
```

字母`a`、`b`和`c`是变量——可以存储值的东西。这些值可以是数字、名称、完整的员工记录等等，以及包含更多变量和值的表达式的值。你可以在`=`符号的右侧找到这些值，这个符号是一个**运算符**。这个运算符将右侧的内容分配给左侧变量的值。`+`也是一个运算符。正如你可能已经猜到的，它将两个东西相加。变量必须首先声明，告诉任何或任何将要阅读这个的人你将要使用哪些变量。然后它们需要初始化，这意味着给它们一个初始值。如果你不这样做，可能会发生奇怪的事情。在这个例子中，声明和初始化都在同一行上完成。最后，我们使用了`printnumber(c)`。

这是一个只有一个参数的函数的名字，变量`c`。假设它会负责显示值：你的程序只有在你也提供了该函数的代码，或者它已经被为你编写并作为函数库的一部分与语言一起提供时才能工作。

因此，我们编写了一个四行程序来计算数字`3`。对于这样一个简单的任务，这至少多了 3 行，但我们需要一个例子。

在我们使用的变量中，没有提到变量包含什么类型的值。在一些语言中，例如 C 编程语言，当你声明变量时，你必须指定你指的是什么类型的变量。以下是一个类似的 C 程序： 

```php
int  a, b, c;
char  someletter; /* to hold a letter not a number */
char *name; /* to hold a string of letters */
float someFloat; /* to hold a floating point number */
a  = 1;
b  = 2;
c  =  a + b;
printnumber (c);
```

## 编译型和解释型语言的比较

我第一次使用 C 编程语言时，对我来说是个启示。我使用全屏编辑器输入我的代码，按了某个地方的按钮，由于没有错误，我可以立即运行我的程序，看看它是否按我预期的那样工作。在大学时，情况不同。我必须使用带有穿孔卡的机器输入我的`Fortran`程序，一次一行，然后把穿孔卡留在数据中心，用橡皮筋绑在一起。两天后，我可以和打印列表一起取回它们，那里写着：*第 37 行：缺少分号*。当然，那已经是很久以前的事了。

使用需要编译的语言有优势，尽管人们的观点可能不同。在你可以运行你的程序之前，必须经过两步过程，一个好的编译器可以彻底检查你的代码，并产生各种错误和警告信息，如`语法错误`、`未声明的变量`、`非法类型`等等，包括行号和有问题的语句。在像 JavaScript 这样的解释型语言中，以及在一定程度上可能是**PHP**，这些错误通常被默默忽略。在完美运行的程序中加入一行带有无辜错误的代码，可能会让它完全停止运行。

解释型语言能给你即时的满足感。你用你喜欢的编辑器输入它们，就像我用 iPad 上的`Textastic`一样，立即打开内置浏览器，看到结果（或者什么都没有）。你也不会遇到当你创建程序并运行它们之前，需要在系统上有一个编译器的问题。这个编译器通常是完整开发环境的一部分。

因此，我们为编程做好了准备，并学习了或重新学习了与编程相关的大量重要术语。这些是：

+   变量

+   值

+   类型

+   关键字或保留字

+   运算符

+   控制流

+   函数

+   编译器和解释器

+   库

+   表达式

+   语法

现在我们将教你们这些是什么：对于 JavaScript 语言，在本章中，对于 PHP 在下一章中。在我们这样做之前，既然我们已经理解了这些词，我们首先需要解决一个重要的话题。

## JavaScript 与 Java 不同

我已经多次提到，我多年前参加的那个关于 Web 开发的六个月课程。在课程进行到三个月时，有一个同学还在混淆 Java 和 JavaScript。现在他是一名前端开发者，所以他最终肯定已经明白了。

我们希望你在接下来的*三分钟*内看到它。Java 和 JavaScript 都是编程语言。它们共同之处在于它们名字的前四个字母，Java，这是一个国家的名字，也让很多人想到了咖啡，以及一些晚期的 Sun Microsystems。那家公司，连同 Netscape，创造了 JavaScript 这个名字，也是 Java 编程语言的创造者和推广者。因此，人们有时感到困惑并不奇怪。

### Java

Java 是一种编程语言，由 Sun Microsystems 的詹姆斯·高斯林（James Gosling）开发。它的语法很多都来自 C 编程语言，有一些部分被省略了，比如指针，这恰好是我最喜欢的 C 语言的部分。

与 C 语言相比，C 语言更像是“一次编写，到处编译”的语言，Java 是我所说的“一次编译，到处运行”的语言。Java 编译器将 Java 源代码从文本文件转换成平台无关的字节码文件，因此你可以在任何有 Java 运行时的地方运行它。你见过那些说你需要安装更新版本的 Java 运行时的消息吗？那就是 Java，不是 JavaScript。

Java 应用程序通常在服务器或桌面计算机上运行，但你也可以在 Web 开发中使用 Java。这本书中我们将首先使用 PHP（第五章;
</script>
```

如果您在浏览器中运行这段代码，屏幕上会出现一个包含*Hello, World*文本的弹出窗口。如果您按下**OK**按钮，弹出窗口将消失。如果您在不同的浏览器中尝试此操作，结果将相同，但弹出窗口的外观将完全不同。浏览器会按照它想要的方式渲染，我们对此无能为力。

我们也可以将这一行代码放入一个名为`hello.js`的文件中，放在`js`文件夹里，然后我们的程序可以说以下内容：

```php
<script type="text/javascript" src="img/hello.js">
 </script>
```

让我们简要分析一下这一行 JavaScript 代码。该行以分号（`;`）结尾。它以`alert`开头，这是 JavaScript 附带的一个函数的名称。弹出窗口的文本作为唯一的参数提供。在这个例子中，该参数是一个包含文本*Hello, World*的字符串。为了表示这是一个字符串值而不是变量，我们使用了双引号。单引号也可以。

## 变量

在 JavaScript 中，变量的名称必须以字母开头。还有一些其他字符，如`$`符号，也被允许，但我们不会使用它们。这有助于我们尽早区分 JavaScript 变量和 PHP 变量。名称也是大小写敏感的，所以`first`与`First`不同。

当然，使用非常短的名字，如`a`和`b`，是可以的，但如果我们想给变量赋予有意义的名字，一些最佳实践指南建议使用如`firstName`、`lastName`这样的名字。

### 变量声明

变量使用 `var` 关键字声明，例如：

```php
var firstName;
var lastName;
var dayOfBirth;
```

或者简单地一行显示，如下所示：

```php
var firstName, lastName, dayOfBirth;
```

当变量被声明但未初始化时，它们的值是 `undefined`，这意味着没有变量类型。要给变量赋值，我们使用 `=` 符号。因此，在适当的声明之后，你可以赋值如下：

```php
firstName = "Paul";
lastName = "Wellens";
```

当然，你也可以一步完成，如下所示：

```php
var firstName = "Paul";
var lastName = "Wellens";
```

#### 变量的值

正如我们提到的，没有变量类型。作为值，我们可以分配几乎任何东西。以下是简要概述。但请记住，我们不是参考手册，所以它并不完整。

#### 数字

最常见的数字是整数。只需使用组成数字的数字即可，所以 `myNumber = 127;` 将十进制数 `127` 赋值给变量 `myNumber`。如果添加一个前导 **零**，结果将不同。行 `myNumber =` **0127;** 将赋值八进制数 `127`，这在十进制中等于 1*64 + 2*8 + 7 = 87。

**十六进制** 数字也可以使用，通过在数字前加上 `0x`。所以 `myNumber = 0x127;` 将存储 256 + 2*16 + 7 = 295。

最后，我们可以有浮点数。始终使用点，而不是逗号，即使你在法国或其他将浮点视为浮点逗号的国家（*virgule flottante*，请原谅我的法语！）。

```php
MyFloat = 123.456;
```

#### 字符串

字符串包含零个或多个字符，这些字符被单引号 (`'`) 或双引号 (`"`) 包围。因此，可能的最短字符串是空字符串。考虑以下内容：

```php
emptyString = "";
helloWorld = "Hello, World\n";
notANumber = "345";
twoLines = "First line\nSecond line";
```

你可能在其他语言中见过 `\n`。这是一个转义序列的例子，在这里表示换行符。`"345"` 这个例子很重要。`notANumber` 变量确实包含一个字符串，而不是一个数字。

#### 将字符串转换为数字

有一些方便的函数可以用来将字符串转换为数字：`parseInt()` 和 `parseFloat()`。它们将字符串或初始子字符串转换为整数或浮点数，直到找不到更多的数字。如果字符串无法转换，函数返回 **Not a Number** （**NaN**）。以下是一些示例，其中我们列出了函数返回的内容作为注释：

```php
/*
Examples using parseInt() and parseFloat()
This is a multiline comment
*/

parseInt("10 yard line");        // returns 10
parseInt("one two three");        // returns NaN
parseFloat("3.14");            // returns 3.14
parseInt("3.14");            // returns 3
parseFloat("3,14");            // returns 3
parseInt("notanumberatall");        // returns NaN
parseInt("0xff");            // returns 255 which is FF hex

/*
parseInt() can also take a second argument specifying the base of the number to be parsed
*/
parseInt("ff",16);            // 255, same as above
parseInt("11",2);            // returns 3, 1*2 + 1
parseInt("11", 8);            // returns 9, which is 1*8 + 1
```

## 表达式和运算符

本节解释了在 JavaScript 中 **表达式** 和 **运算符** 的工作原理，重点关注我们将使用该语言做什么——编程网站。因此，不要期望对所有位运算符有深入的解释，例如。如果你不理解我刚才说的，那完全没问题。但如果你需要知道它们是什么，你可能还需要另一本书或参考资料。

表达式是 JavaScript 语句的一部分，通常是`=`运算符右侧的部分，JavaScript 解释器可以对其进行评估。结果将是表达式的值。我们已知的最简单的表达式：值和变量。在下面的例子中，`i + 7`也是一个表达式。它将评估为 8。它中间包含一个运算符`+`。它的左右两侧的部分可以被称为**操作数**。

```php
var i = 1;
var j;
j = i + 7;
```

我们现在将介绍最基础的运算符。对于那些已经了解 C、C++或 Java 的你们来说，它们看起来会非常熟悉。

### 算术运算符

这是用来计算的一组运算符。

#### 加法（+）

`+`运算符将两个操作数的值相加，也就是说，如果它们是数字，就像上一个例子中那样。如果它们不是数字，程序会挂起吗？当然不会。记住，你将使用 JavaScript 来程序性地修改网页。网页反过来又由 HTML 组成，而 HTML 又是由很多字符串组成的。所以我们的加号运算符已经成为你拥有的最有用的工具之一。看看下面的例子：

```php
var hello = 'Hello, World';
var htmlString;
htmlString = '<h1>' + hello + '</h1>';
```

所以，在 JavaScript 中，加号运算符可以用来拼接，或者，用更技术性的术语来说，`concatenate`字符串。在 PHP 中我们也会遇到同样的功能，但那里的运算符是点(`.`)。尽早习惯这些差异永远不会太早，因为你将大量使用它们。

在一个例子中，我们使用了`+`运算符的一个特殊情况。`counter++;`是人们从 C 语言等语言中熟悉的一种简写符号，JavaScript 和 PHP 都支持这种简写符号。它是一个简写符号，用来简单地将`1`加到变量的值上。所以以下两个语句是相同的：

```php
varName = varName + 1;
varName++;
```

#### 减法（-）

每个使用过计算器的人都知道减号。它用于减去数字。虽然`+`符号也可以与字符串一起使用，但在减法（以及乘法和除法）中这样做是没有意义的。还有一个类似于`++`的简写符号，看起来像"`—`"。

```php
varName = varName - 1;
varName—;
```

它们确实也是相同的。

到目前为止，我们已经使用了一个包含两个以上操作数的例子，那就是我们用来组合 HTML 字符串的那行语句。但如果我们做以下操作会发生什么呢：

```php
 var result;
 var a = 7, b = 5,  c = 2;
 result = a - b - c;
 alert (result);
```

它会是 0，还是 4？答案是 0，因为表达式是按照从左到右的顺序进行评估的，但你可以通过使用括号来消除所有疑虑。看看下面的例子：

```php
 result = (a - b) - c;   // 7-5 is 2 2-2 is 0
 result = a - (b - c);   // 5-2 is 3 7-3 is 4
```

#### 乘法（*）

`*`运算符将其两个操作数相乘。

#### 除法（/）

`/`运算符将其第一个操作数除以第二个操作数。结果始终是一个浮点数，所以 5 / 2 评估为 2.5。在某些语言中，如果两个操作数都是整数，这将是两个。

#### 模数（%）

`%`运算符非常实用。它返回第一个操作数对第二个操作数的余数。例如，5%2 评估为 1，因为 5 除以 2 的整数商是 2，余数是 1。我经常在样式中使用它，当我想要对偶数和奇数做不同的事情时。在代码中使用`if (number%2)`意味着奇数，因为如果括号内的值不是零，则为真。

#### 关系运算符

编程中的一个常见错误是使用类似`if (a = b)`的东西来比较两个东西，并发现即使`a`不等于`b`，它总是为真。这是因为许多语言，包括 JavaScript 和 PHP，单个等号是赋值运算符。要检查两个东西是否相等，你需要（至少）两个等号。使用以下运算符的运算符的表达式将评估为真或假，在 JavaScript 中相应的值不是`1`或`0`，而是实际上`true`或`false`。它们如下所示：

+   等于（`==`）：我们在这个表达式中使用它来检查两个变量是否包含相等的值，例如如果（`a == b`）

+   不等于（`!=`）：当然，这是相反的情况：如果不相等则为真，相等则为假

+   小于（`<`）

+   小于或等于（`<=`）

+   大于（`>`）

+   大于或等于（`>=`）

+   非（`!`）

这会将假转换为真，反之亦然，当它放在表达式前面时。这可以帮助使你的程序更易于阅读。

当然，当单独使用时，这样的表达式并不很有帮助。它们需要成为某个结构的一部分，我们添加控制来使我们的程序在表达式评估为假而不是真时执行不同的操作。

## 控制流

**控制流**是编程的核心。控制流语句有助于确定随后的代码中哪些部分被执行，哪些部分不执行。在任何语言中最常用的控制流语句可能是`if-else`语句。我们将引导你了解 JavaScript 中最有用的那些。

我们学习了什么是表达式。我还把一条 JavaScript 代码（以分号结尾）称为一个语句。你可以通过将大括号（`{` 和 `}`）放在它们周围来将语句组合在一起。这样一组语句被称为**语句块**。即使只有一个语句，当用于下面描述的任何一个控制结构中时，使用大括号也被认为是最佳实践，其中“语句”代表语句块。

### 如果（`if`）

`if`语句的格式是：

```php
if (expression)
{
    statements
}
```

你也可以将`if`与`else`结合起来，如下所示：

```php
if (expression) {
statements
}
else {
statements
}
```

示例：

```php
if (a < b ) {
alert ("a is smaller than b" );
}
else
{
alert ("a is not smaller than b");
}
```

### 当（`while`）

而一个`if`语句是你的基本控制语句，根据条件或其它东西来做某事，`while`语句则非常适合创建循环。循环是一系列在某个条件为真（或`while`）时重复执行的语句。确保你的条件不是永远为真非常重要，否则你最终会得到一个`无限循环`。

以下是格式：

```php
while (expression)
statement
```

接下来是一个例子：

```php
var count = 0;
var seriesA = "";
while (count < 100)
{
seriesA = seriesA + 'a'; // can also be written as seriesA += 'a';
count++;             // if you forget this one you are in trouble
}
alert(seriesA);
```

这段代码的结果将弹出一个包含字母 a 重复 100 次的字符串。

### switch

我喜欢`switch`语句。如果你需要根据超过 2 个不同的可能条件执行不同的代码，使用`switch`语句比多个`if`/`else`语句更易于阅读和维护。通常，将被评估的表达式将是一个变量，然后将有几个在变量值为`value1`、`value2`、`value3`等情况下执行的语句。

它是这样的：

```php
switch (expression)  {

case value1:
statement
break;
case value2:
statement
break;
// etc. etc.
default:
statement   // or no statement
break;
}
```

首先，评估表达式。根据表达式的值，执行属于匹配值的语句。如果没有匹配项，则执行与`default:`关联的语句。`break;`语句很重要，但不是强制的。在`switch`语句中，首先执行的语句块是属于第一个匹配的。如果省略了`break;`语句，则还会执行后续的语句，而这通常不是我们想要的。我通常首先编写一个`switch`语句的骨架，包括`case:`和`break;`，然后填充实际的代码。

现在，让我们来看一个例子，并对此进行一些有趣的探索：

```php
var plate;
plate = getPlateName();
switch (plate) {
case "first":
Who();
break;
case "second":
what();
break;
case "third":
IDontKnow();
break;
case "home":
today();
break;
default:
strike();
break;
}
```

## 函数

我们已经在我们的例子中使用了一些**函数**。函数是一段 JavaScript 代码，你可以自己编写，也可以由 JavaScript 实现预定义。函数可以传递参数，也称为**参数**。以下是我们自己编写的函数示例，用于计算作为第一个和唯一参数传递的数字的平方根：

```php
function square (x)
{
var sq;
sq =  x * x;
return  sq;
}
```

`square`是函数的名称。它由大括号之间的所有代码组成。我们称它为`function`块。一旦编写了函数，就可以反复使用。在这个例子中，函数实际上也返回一个值；在这种情况下，是参数的平方根。这是通过使用`return`语句完成的：`return sq;`。请注意，在`return`关键字之后没有括号，这与其他编程语言的情况不同。那将意味着会有一个返回的`function`！所以让我们多次使用我们的函数：

```php
var message, result;
result = square(3);
message = "the square of 3 is" + result;
alert(message);

function square (x)
{
var sq;
sq =  x * x;
return  sq;
}
alert("the square of 4 is " + square(4));
```

如你所见，我们可以在声明前后使用函数。我们首先可以将消息存储在变量中，或者直接将其传递给`alert()`函数。

### 变量的作用域

在本章介绍函数的这一部分，解释一个非常重要的概念：**变量的作用域**，以及**全局**变量与**局部**变量的概念。

如果你一直在练习，我真诚地希望你已经勤奋地使用`var`关键字声明了所有你的变量。但是，如果你在某处忘记了它呢？在这种情况下，你可能是无意中声明了一个全局变量。所以作用域，即变量可以在其中使用的 JavaScript 程序区域，是任何地方，我们称之为全局。

使用`var`关键字声明的变量具有局部作用域。局部变量仅在其声明的函数体内定义。作为函数参数传递的变量也具有局部作用域，并且仅在该函数体内定义。

如果你声明了一个与已存在的全局变量同名的地方变量，那么这个地方变量实际上会在函数内部“隐藏”全局变量。但是，如果在函数内部，你给一个变量赋值而没有先声明它，你可能会覆盖掉程序其他地方需要的值。这就是为什么在函数开始时声明所有变量很重要的原因。花几分钟时间研究下面的例子，一切都会变得清晰。

```php
// global variables
outside = "outside";
global = "global";
var same ="outside"
function   testfunction()
{
// inside function    
global = "local";    // mistakenly modified global variable

var inside = "inside";    // local variable
var local;    
var same = "inside";     

alert(inside);  // prints inside
alert (outside);// prints outside
alert(local);   // prints "" because it was not initialized
alert (same);  // prints inside
alert(global);    // prints local because we modified it

}

// running testfunction()
testfunction();    

// after running testfunction()
alert (outside);  // outside
alert(global);    // global because we had overwritten it
alert(same);      // prints outside
```

### 对象

在本章的早期，我提到变量可以包含几乎所有东西：数字、字符串，甚至是完整的员工记录。但在我们迄今为止看到的例子中，似乎没有适合如此复杂如员工记录的容器。这就是对象发挥作用的地方。在这里，我们将学习如何创建它们。

对象是命名值的集合，它们是通过一个称为**构造函数**的特殊函数创建的。JavaScript 提供了几个这样的函数。有一个通用的，`Object()`，但也有一些用于创建具有预定义结构的对象的特定函数，例如`Date()`或`String()`。语法如下：

```php
employee = new Object();
```

现在我们已经创建了一个名为 employee 的空对象，我们可以用名称和值填充它，如下所示：

```php
employee.firstName = "John";
employee.lastName = "Williams";
employee.profession = "conductor";
```

在 JavaScript 对象的上下文中，这些名称被称为**属性**。属性反过来有值。对象还可以包含用于在对象内部执行操作的函数。这些函数被称为**方法**。例如，`String()`对象包含相当多的字符串操作方法。查阅一本好的参考书或在线 JavaScript 对象参考可以减少编写代码的数量，因为所有这些都已经有了。以下是一个**String**方法的示例：

```php
hello = new String('Hello, World"); // Or simply hello = "Hello, World";
Hello = hello.toUpperCase();
alert (Hello);  // This will show HELLO, WORLD in a pop-up box
```

#### JSON

创建对象还有另一种方法，不需要构造函数。

```php
employee = {};  // Creates an empty object
employee.firstName = "John"; // same as before
// or do it all at once
employee = { firstName:"John", lastName:"Williams" };
```

右侧匹配名为**JavaScript 对象表示法**（**JSON**）的数据格式。在这本书中，我们将专门用一章来介绍 JSON，我们将说服你这是最酷的数据格式；所以关于 JSON 的更多内容将在后面介绍。

# DOM 对象、属性、方法和事件

我们已经向您介绍了 JavaScript 对象的基础知识，现在我们将深入探讨客户端 JavaScript 的核心——DOM 编程。

## 窗口对象

当你在浏览器中运行网页时，有两个对象可供你使用——**Window 对象**和**Document 对象**。

窗口对象为我们提供了访问可以告诉我们用于渲染网页的窗口状态的属性和方法。我们已经在每个示例中都使用了一个这样的方法：`alert()`。考虑以下内容：

```php
alert("Hello, world");
```

这实际上是以下内容的缩写：

```php
window.alert("Hello, world");
```

类似的有用的创建对话框的方法有`prompt`和`confirm`。前者显示一个对话框，提示访客输入；后者显示一个带有消息和**确定**和**取消**按钮的对话框。请参考下一个示例：

```php
var response = confirm ("Are you sure you want to do this?");
if (response == true) {
// if (response) has the same effect
doit();
} else {
donot();
}
```

窗口对象也有一些有趣的属性。你想知道当前窗口的宽度是多少像素？你可以通过以下方式访问：

```php
var windowWidth = window.innerWidth;
```

窗口对象最重要的属性本身也是一个对象——DOM 文档对象。

## 文档对象

**window.document**（但你不必写 window 部分）是一个属性，它给你一个对象，该对象包含浏览器加载的整个网页。

## write 和 writeln 方法

在我们的大部分示例中，我们使用了 window `alert()`方法来显示带有一些文本的弹出框。虽然这对于测试或调试很有用，但它可能会让人感到烦恼，因为你必须点击该框才能看到下一行文本。大多数教科书都会使用文档方法`write`和`writeln`。使用这些方法，你可以直接将消息文本写入你的文档。`writeln`与`write`的不同之处在于它在每个语句后添加一个新行。只需尝试以下操作：

```php
<!DOCTYPE html >
<html>
<head>
<meta http-equiv="Content-Type" content="text/html;    charset=utf-8" />
</head>
<body>
<p> The following text is generated using JavaScript </p>
<script type="text/javascript">
document.write("Hello, World");
document.write("On the same line");
</script>
</body>
</html>
```

## 节点和 DOM 遍历

我们将使用文档方法来遍历或遍历我们的文档并对其进行更改。你需要将文档想象成一个树状结构，其中包含节点。附着在这些节点上的是你的 HTML 标签及其属性和内部 HTML 文本。通过专用方法，我们可以查找文档的某些部分。一个**节点**或**节点列表**就是它们返回的内容（有时只有一个匹配的标签，有时有多个）。然后，通过更改这些节点的属性或使用其他方法，我们可以有效地更改文档的内容或布局。

查找文档树中的三个有用方法是：

+   `getElementById`：查找 id 指定的节点

+   `getElementsByClassName`：获取 class 设置为参数的节点

+   `getElementsByTagName`：获取所有 HTML 标签等于参数的节点

一个有用的属性是`innerHTML`属性，它允许你获取或设置 HTML 标签内的 HTML 文本。使用这些元素，我们可以构建以下示例：

```php
<!DOCTYPE html >
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; 
charset=utf-8" />
</head>
<body>
<h1 id="hellotext"> Hello,World</h1>
<script type="text/javascript">
var  findhello = document.getElementById('hellotext');
findhello.innerHTML = ("Hello, Beach");
</script>
</body>
</html>
```

这个简单的 HTML 文档，另一个 `你好，世界` 示例，通过 JavaScript 立即被修改，而我们看到的浏览器渲染的唯一内容是 `你好，海滩`。这并不令人兴奋。然而，如果我们让用户决定何时更改文本，触发一个事件，并让我们的代码对此事件做出反应，那么我们实际上就给我们的网页添加了动作。

## 事件

**事件** 是发生在 DOM 内部元素上的事情。它们可以由浏览器或用户触发。用户控制的可以触发的事件包括：

+   更改输入字段

+   点击按钮

+   将鼠标移至某个区域

+   点击链接

你可以将事件处理器附加到包含需要执行代码的标签上。在下一个示例中，我们通过向元素添加 `event` 属性来实现这一点。你也可以通过使用文档的 `addEventListener()` 方法达到同样的效果。

```php
<!DOCTYPE html >
<html>
<head>
<meta http-equiv="Content-Type" content="text/html;
charset=utf-8" />
<script type="text/javascript">
function goToBeach() {
var  findhello = document.getElementById('hellotext');
findhello.innerHTML = "Hello, Beach";
}
</script>
</head>
<body>
<h1 id="hellotext"> Hello,World</h1>
<input type="button" onclick="goToBeach()" value="Beach"></input>
</body>
</html>
```

两个示例之间的区别在于我们添加了一个带有 `onclick` 属性的按钮，我们将一个 JavaScript 函数的名称作为值，并将我们的 JavaScript 代码放在同名函数中。现在，直到用户点击 **海滩** 按钮，**你好，世界** 才会变成 **你好，海滩**。

# 概述

到目前为止，你可能认为，我已经到达了困难的部分，我很快就会迷失方向。好消息是！你不会迷失方向，因为你已经到达了这一章的结尾。

我们从对编程语言特性的简要概述开始。接下来，我们阐述了 JavaScript 的这些用途，同时解释了为什么我们需要这种语言以及我们将如何使用它：来编写网页。为了实现这一点，我们需要一种简单的方法来访问网页中的所有元素。

好吧，我们在上一章中已经学习了如何通过使用 **选择器** 来访问页面元素。

**jQuery**，一个流行且经过验证的 JavaScript 库，我们将主要用它来处理客户端 JavaScript。它允许你使用 CSS 风格的选择器来指定我们想要访问的元素，甚至可以为其附加事件，这样我们就不需要学习新东西，或者让这本书的大小接近《战争与和平》。jQuery 有其他几个优点，我们将在第七章第七章。jQuery 中描述。这包括一个酷炫的界面，可以在 **客户端** 和 **服务器** 之间切换。

因此，我们不会立即深入研究 jQuery。我们需要了解为什么我们还需要服务器端编程。我们将在下一章中学习这一点，同时学习这本书的第二个编程语言：**PHP**。

### 小贴士

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt Publishing 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件通过电子邮件发送给您。
