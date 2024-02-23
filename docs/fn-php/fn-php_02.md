# 第二章：纯函数、引用透明度和不可变性

阅读有关函数式编程的附录的人会发现，它围绕纯函数展开，换句话说，只使用其输入来产生结果的函数。

确定一个函数是否是纯的似乎很容易。只需检查是否调用了任何全局状态，对吗？遗憾的是，事情并不那么简单。函数产生副作用的方式也有多种。有些很容易发现，而其他一些则更难。

本章不会涵盖使用函数式编程的好处。如果您对好处感兴趣，我建议您阅读附录，其中深入讨论了这个主题。然而，我们将讨论不可变性和引用透明性所提供的优势，因为它们相当具体，并且在附录中只是粗略地提到。

在本章中，我们将涵盖以下主题：

+   隐藏的输入和输出

+   函数纯度

+   不可变性

+   引用透明度

# 两组输入和输出

让我们从一个简单的函数开始：

```php
<?php 

function add(int $a, int $b): int 
{ 
    return $a + $b; 
} 
```

这个函数的输入和输出很容易发现。我们有两个参数和一个返回值。毫无疑问，这个函数是纯的。

参数和返回值是函数可能具有的第一组输入和输出。但还有第二组，通常更难发现。看看以下两个函数：

```php
<?php 

function nextMessage(): string 
{ 
    return array_pop($_SESSION['message']); 
} 

// A simple score updating method for a game 
function updateScore(Player $player, int $points) 
{ 
    $score = $player->getScore(); 
    $player->setScore($score + $points); 
} 
```

第一个函数没有明显的输入。然而，很明显我们从`$_SESSION`变量中获取一些数据来创建输出值，所以我们有一个隐藏的输入。我们还对会话产生了隐藏的副作用，因为`array_pop`方法从消息列表中删除了我们刚刚得到的消息。

第二种方法没有明显的输出。但是，更新玩家的得分显然是一个副作用。此外，我们从玩家那里得到的`$score`可能被视为函数的第二个输入。

在这样简单的代码示例中，隐藏的输入和输出很容易发现。然而，随着时间的推移，尤其是在面向对象的代码库中，情况很快变得更加困难。不要误解，任何隐藏的东西，即使以最明显的方式隐藏，都可能产生后果，比如：

+   增加认知负担。现在你必须考虑`Session`或`Player`类中发生了什么。

+   相同的输入参数可能导致测试结果不同，因为软件的某些其他状态已经改变，导致难以理解的行为。

+   函数签名或 API 并不清楚您可以从函数中期望什么，这使得有必要阅读它们的代码或文档。

这两个看起来简单的函数的问题在于它们需要读取和更新程序的现有状态。本章的主题还不是向您展示如何更好地编写它们，我们将在第六章《真实生活中的单子》中讨论这个问题。

对于习惯于依赖注入的读者来说，第一个函数使用了静态调用，可以通过注入 Session 变量的实例来避免。这样做将解决隐藏输入的问题，但我们修改`$_SESSION`变量的事实仍然是副作用。

本章的其余部分将尝试教会您如何发现不纯的函数以及它们对函数式编程和代码质量的重要性。

在本书的其余部分，我们将使用术语“副作用”来表示隐藏的输入，以及“副作用”来表示隐藏的输出。这种二分法并不总是被使用，但我认为它有助于更准确地描述我们讨论的代码中的隐藏依赖或隐藏输出。

尽管是一个更广泛的概念，可用的功能性文献可能会使用术语“自由变量”来指代副作用。维基百科关于这个主题的文章陈述如下：

> *在计算机编程中，自由变量是指在函数中使用的不是局部变量也不是该函数的参数的变量。在这个上下文中，非局部变量通常是一个同义词。*

根据这个定义，使用`use`关键字传递给 PHP 闭包的变量可以被称为自由变量；这就是为什么我更喜欢使用副作用这个术语来清楚地区分这两者。

# 纯函数

假设你有一个函数签名`getCurrentTvProgram (Channel $channel)`。在没有函数纯度的指示的情况下，你无法知道这样一个函数背后可能隐藏的复杂性。

你可能会得到实际播放在给定频道的节目。但你不知道函数是否检查了你是否已登录系统。也许有一些用于分析目的的数据库更新。也许函数会因为日志文件处于只读状态而返回异常。你无法确定，所有这些都是副作用或者副作用。

考虑到所有与这些隐藏依赖关系相关的复杂性，你面临三个选择：

+   深入文档或代码，了解所有正在发生的事情

+   让依赖关系显而易见

+   什么都不做，祈求最好的结果

最后一个选项在短期内显然更好，但你可能会受到严重的打击。第一个选项可能看起来更好，但你的同事在应用程序的其他地方也需要使用这个函数，他们需要像你一样遵循相同的路径吗？

第二个选项可能是最困难的，因为一开始需要付出巨大的努力，因为我们根本不习惯这样做。但一旦你完成了，好处就会开始积累。并且随着经验的增加，这将变得更容易。

## 封装呢？

封装是为了隐藏实现细节。纯度是为了让隐藏的依赖关系变得明显。这两者都是有用的，是良好的实践，它们并不冲突。如果你足够小心，你可以同时实现这两者，这通常是函数式程序员所追求的。他们喜欢简洁、简单的解决方案。

简单来说，这就是解释：

+   封装是为了隐藏内部实现

+   避免副作用是为了让外部输入变得明显

+   避免副作用是为了让外部变化变得明显

## 发现副作用的原因

让我们回到我们的`getCurrentTvProgram`函数。接下来的实现并不纯净，你能发现原因吗？

为了帮助你一点，我会告诉你到目前为止我们所学到的关于纯函数的东西意味着当使用相同的参数调用时，它们总是返回相同的结果：

```php
<?php 

function getCurrentTvProgram(Channel $channel ): string 

{ 
    // let's assume that getProgramAt is a pure method. return $channel->getProgramAt(time()); 
} 
```

明白了吗？我们的嫌疑对象是对`time()`方法的调用。因为如果你在不同的时间调用该函数，你会得到不同的结果。让我们来修复这个问题：

```php
<?php 

functiongetTvProgram(Channel $channel, int $when): string 
{ 
    return $channel->getProgramAt($when); 
} 
```

我们的函数不仅是纯净的，这本身就是一个成就，我们还获得了两个好处：

+   现在我们可以根据名称更改隐含的意思来获取任何时间的节目

+   现在可以测试该函数，而无需使用某种魔术技巧来改变当前时间

让我们快速看一些其他副作用的例子。在阅读之前，尝试自己找出问题：

```php
<?php 

$counter = 0; 

function increment() 
{ 
    global $counter; 
    return ++$counter; 
} 

function increment2() 
{ 
    static $counter = 0;
    return ++$counter; 
} 

function get_administrators(EntityManager $em) 
{ 
    // Let's assume $em is a Doctrine EntityManager allowing 
    // to perform DB queries 
    return $em->createQueryBuilder() 
              ->select('u') 
              ->from('User', 'u') 
              ->where('u.admin = 1') 
              ->getQuery()->getArrayResult(); 
} 

function get_roles(User $u) 
{ 
    return array_merge($u->getRoles(), $u->getGroup()->getRoles()); 
} 
```

使用`global`关键字很明显地表明第一个函数使用了全局范围的某个变量，因此使函数不纯。从这个例子中可以得出的关键点是 PHP 作用域规则对我们有利。任何你能发现这个关键字的函数很可能是不纯的。

第二个示例中的静态关键字是一个很好的指示，表明我们可能会尝试在函数调用之间存储状态。在这个例子中，它是一个在每次运行时递增的计数器。该函数显然是不纯的。然而，与`global`变量相反，使用`static`关键字可能只是一种在调用之间缓存数据的方式，因此在得出结论之前，您将不得不检查为什么使用它。

第三个函数毫无疑问是不纯的，因为进行了一些数据库访问。如果您只允许使用纯函数，您可能会问自己如何从数据库或用户那里获取数据。如果您想编写纯函数式代码，第六章将更深入地探讨这个主题。如果您无法或不愿意完全使用函数式编程，我建议您尽可能将不纯的调用分组，然后尝试从那里仅调用纯函数，以限制产生副作用的地方。

关于第四个函数，仅仅通过查看它，您无法确定它是否是纯的。您将不得不查看被调用的方法的代码。在大多数情况下，您将遇到这种情况，一个函数调用其他函数和方法，您也将不得不阅读以确定纯度。

## 发现副作用

通常，发现副作用比发现副因更容易。每当您更改一个将对外部产生可见影响的值，或者在这样做时调用另一个函数，您都会产生副作用。

如果我们回到之前定义的两个`increment`函数，您对它们有什么看法？考虑以下代码：

```php
<?php 

$counter = 0; 

function increment() 
{ 
    global $counter; 
    return ++$counter; 
} 

function increment2() 
{ 
    static $counter = 0; 
    return ++$counter; 
} 
```

第一个函数显然对全局变量产生了副作用。但第二个版本呢？变量本身无法从外部访问，所以我们能认为该函数是没有副作用的吗？答案是否定的。因为更改意味着对函数的后续调用将返回另一个值，这也属于副作用。

让我们看一些函数，看看您是否能发现副作用：

```php
<?php 

function set_administrator(EntityManager $em, User $u) 
{ 
    $em->createQueryBuilder() 
       ->update('models\User', 'u') 
       ->set('u.admin', 1) 
       ->where('u.id = ?1') 
       ->setParameter(1, $u->id) 
       ->getQuery()->execute(); 
} 

function log_message($message) 
{ 
    echo $message."\n"; 
} 

function updatePlayers(Player $winner, Player $loser, int $score) 
{ 
    $winner->updateScore($score); 
    $loser->updateScore(-$score); 
} 
```

第一个函数显然有副作用，因为我们更新了数据库中的值。

第二个方法向屏幕打印了一些内容。通常这被认为是一个副作用，因为该函数对其范围之外的东西产生了影响，也就是我们的情况下，屏幕。

最后，最后一个函数可能会产生副作用。这是一个很好的、基于方法名称的猜测。但在我们看到方法的代码以验证之前，我们不能确定。当发现副作用时，通常需要深入挖掘，而不仅仅是一个函数，以确定它是否会产生副作用。

## 对象方法呢？

在一个纯粹的函数式语言中，一旦需要更改对象、数组或任何类型的集合中的值，实际上会返回一个具有新值的副本。这意味着任何方法，例如`updateScore`方法，都不会修改对象的内部属性，而是会返回一个具有新分数的新实例。

这可能看起来一点也不实用，鉴于 PHP 本身提供的可能性，我同意。然而，我们将看到一些真正有助于管理这一点的函数式技术。

另一个选择是决定实例在更改后不是相同的值。在某种程度上，这已经是 PHP 的情况。考虑以下示例：

```php
<?php 
class Test 
{ 
    private $value; 
    public function __construct($v) 
    { 
        $this->set($v); 
    } 

    public function set($v) { 
        $this->value = $v; 
    } 
} 

function compare($a, $b) 
{ 
    echo ($a == $b ? 'identical' : 'different')."\n"; 
} 

$a = new Test(2); 
$b = new Test(2); 

compare($a, $b); 
// identical 

$b->set(10); 
compare($a, $b); 
// different 

$c = clone $a; 
$c->set(5); 
compare($a, $c); 
```

在进行简单的对象相等比较时，PHP 考虑的是内部值而不是实例本身来进行比较。重要的是要注意，一旦使用严格比较（例如使用`===`运算符），PHP 会验证两个变量是否持有相同的实例，在所有三种情况下返回`'different'`字符串。

然而，这与引用透明的概念是不兼容的，我们将在本章后面讨论。

## 结束语

正如我们在前面的例子中所尝试展示的，尝试确定一个函数是否是纯函数可能在开始时会有些棘手。但是当你开始对此有所感觉时，你会变得更快更舒适。

检查函数是否是纯函数的最佳方法是验证以下内容：

+   使用全局关键字是一个明显的暴露

+   检查是否使用了任何不是函数本身参数的值

+   验证你的函数调用的所有函数也都是纯函数

+   任何对外部存储的访问都是不纯的（数据库和文件）

+   特别注意那些返回值依赖于外部状态（`time`，`random`）的函数

现在你知道如何发现那些不纯的函数了，你可能想知道如何使它们成为纯函数。遗憾的是，对于这个请求并没有简单的答案。接下来的章节将尝试提供避免不纯性的配方和模式。

# 不可变性

我们说一个变量是不可变的，如果它一旦被赋值，就不能改变其内容。在函数纯度之后，这是函数式编程中第二重要的事情。

在一些学术语言中，比如**Haskell**，你根本无法声明变量。一切都必须是函数。由于所有这些函数也都是纯函数，这意味着你可以免费获得不可变性。其中一些语言提供了一些类似变量声明的语法糖，以避免总是声明函数可能带来的繁琐。

大多数函数式语言只允许声明不可变变量或具有相同目的的构造。这意味着你有一种存储数值的方式，但是在初始赋值后无法更改数值。也有一些语言让你为每个变量选择你想要的方式。例如，在 Scala 中，你可以使用`var`关键字声明传统的可变变量，使用`val`关键字声明不可变变量。

然而，大多数语言，比如 PHP，对变量没有不可变性的概念。

## 为什么不可变性很重要？

首先，它有助于减少认知负担。在算法中记住所有涉及的变量已经相当困难了。没有不可变性，你还需要记住所有值的变化。对于人类大脑来说，将一个值与特定标签（即变量名）关联起来要容易得多。如果你能确信数值不会改变，推理发生的事情就会容易得多。

另外，如果你有一些全局状态是无法摆脱的，只要它是不可变的，你可以在你附近的一张纸上记录数值并保留以供参考。无论执行过程中发生了什么，所写的内容始终是程序的当前状态，这意味着你不必启动调试器或回显变量来确保数值没有改变。

想象一下，你将一个对象传递给一个函数。你不知道这个函数是否是纯函数，也就是说对象的属性可能会被改变。这会让你感到担忧，分散你的思绪。你必须问自己内部状态是否改变，这降低了你推理代码的能力。如果你的对象是不可变的，你可以百分之百地确信它和以前一样，加快你对发生的事情的理解。

你还可以获得与线程安全和并行化相关的优势。如果你的所有状态都是不可变的，那么确保你的程序能够在多个核心或多台计算机上同时运行就会更容易得多。大多数并发问题发生在某个线程在没有正确与其他线程同步的情况下修改了一个值。这导致它们之间的不一致，通常会导致计算错误。如果你的变量是不可变的，只要所有线程都收到了相同的数据，这种情况发生的可能性就会小得多。然而，由于 PHP 主要用于非线程场景，这并不是真正有用的。

## 数据共享

不可变性的另一个好处是，当语言本身强制执行时，编译器可以执行一种称为**数据共享**的优化。由于 PHP 目前还不支持这一点，我只会简单介绍一下。

数据共享是共享一个公共内存位置，用于包含相同数据的多个变量。这允许更小的内存占用，并且几乎没有成本地将数据从一个变量复制到另一个变量。

例如，想象一下以下代码片段：

```php
<?php 

//let's assume we have some big array of data 
$array= ['one', 'two', 'three', '...']; 

$filtered = array_filter($array, function($i) { /* [...] */ }); 
$beginning = array_slice($array, 0, 10); 
$final = array_map(function($i) { /* [...] */ }, $array); 
```

在 PHP 中，每个新变量都将是数据的一个新副本。这意味着我们有一个内存和时间成本，当我们的数组越大时，这可能会成为一个问题。

使用巧妙的技术，函数式语言可能只在内存中存储一次数据，然后使用另一种方式描述每个变量包含的数据部分。这仍然需要一些计算，但对于大型结构，你将节省大量内存和时间。

这样的优化也可以在非不可变语言中实现。但通常不这样做，因为你必须跟踪每个变量的每次写访问，以确保数据的一致性。编译器的隐含复杂性被认为超过了这种方法的好处。

然而，在 PHP 中，时间和内存开销并不足以避免使用不可变性。PHP 有一个相当不错的垃圾收集器，这意味着当对象不再使用时，内存会被清理得相当有效。而且我们通常使用相对较小的数据结构，这意味着几乎相同数据的创建速度相当快。

## 使用常量

你可以使用常量和类常量来实现某种不可变性，但它们只适用于标量值。目前，你无法将它们用于对象或更复杂的数据结构。由于这是唯一可用的选项，让我们还是来看一下吧。

你可以声明包含任何标量值的全局可用常量。从 PHP 5.6 开始，当使用`const`关键字时，你还可以在常量中存储标量值的数组，并且自 PHP 7 开始，使用定义语法也可以。

常量名称必须以字母或下划线开头，不能以数字开头。通常，常量都是大写的，这样它们就很容易被发现。以下划线开头也是不鼓励的，因为它可能与 PHP 核心定义的任何常量发生冲突：

```php
<?php 

define('FOO', 'something'); 
const BAR=42; 

//this only works since PHP 5.6 
const BAZ = ['one', 'two', 'three']; 

// the 'define' syntax for array work since PHP 7 
define('BAZ7', ['one', 'two', 'three']); 

// names starting and ending with underscores are discouraged 
define('__FOO__', 'possible clash'); 
```

你可以使用函数的结果来填充常量。但这只在使用定义的语法时才可能。如果你使用`const`关键字，你必须直接使用标量值：

```php
<?php 

define('UPPERCASE', strtoupper('Hello World !')); 
```

如果你尝试访问一个不存在的常量，PHP 将假定你实际上是在尝试将该值用作字符串：

```php
<?php 

echo UPPERCASE; 
//display 'HELLO WORLD !' echo I_DONT_EXISTS; 
//PHPNotice:  Use of undefined constant 

I_DONT_EXISTS
//- assumed'I_DONT_EXISTS' 
//display 'I_DONT_EXISTS'anyway 
```

这可能会非常误导，因为假定的字符串将计算为`true`，如果你期望你的常量保存一个`false`值，这可能会破坏你的代码。

如果你想避免这种陷阱，你可以使用 defined 或 constant 函数。遗憾的是，这将增加很多冗余性：

```php
<?php 

echo constant('UPPERCASE'); 
// display 'HELLO WORLD !' echo defined('UPPERCASE') ? 'true' : 'false'; 
// display 'true' 

echo constant('I_DONT_EXISTS'); 
// PHP Warning:  constant(): Couldn't find constant I_DONT_EXISTS 
// display nothings as 'constant' returns 'null' in this case 

echo defined('I_DONT_EXISTS') ? 'true' : 'false'; 
// display 'false' 
```

PHP 还允许在类内部声明常量：

```php
<?php 

class A 
{ 
    const FOO='some value'; 

    public static function bar() 
    { 
        echo self::FOO; 
    } 
} 

echo A::FOO; 
// display 'some value' 

echo constant('A::FOO'); 
// display 'some value' 

echo defined('A::FOO') ? 'true' : 'false'; 
// display 'true' 

A::bar(); 
// display 'some value' 
```

遗憾的是，当这样做时，你只能直接使用标量值；无法使用函数的返回值，就像`define`关键字一样：

```php
<?php 

class A 
{ 
    const FOO=uppercase('Hello World !'); 
} 

// This will generate an error when parsing the file : 
// PHP Fatal error:  Constant expression contains invalid operations 
```

然而，从 PHP 5.6 开始，你可以使用任何标量表达式或先前声明的常量与`const`关键字一起使用：

```php
<?php 

const FOO=6; 

class B 
{ 
    const BAR=FOO*7; 
    const BAZ="The answer is ": self::BAR; 
} 
```

除了它们的不可变性之外，常量和变量之间还有另一个基本区别。通常的作用域规则不适用。只要常量被声明，你就可以在代码中的任何地方使用它：

```php
<?php 

const FOO='foo'; 
$bar='bar'; 

function test() 
{ 
    // here FOO is accessible 
    echo FOO; 

    // however, if you want to access $bar, you have to use 
    // the 'global' keyword. global $bar; 
    echo $bar; 
}
```

在撰写本文时，PHP 7.1 仍处于测试阶段。发布计划于 2016 年秋末。这个新版本将引入类常量可见性修饰符：

```php
<?php 

class A 
{ 
    public const FOO='public const'; 
    protected const BAR='protected const'; 
    private const BAZ='private const'; 
} 

// public constants are accessible as always 
echo A::FOO; 

// this will however generate an error 
echo A::BAZ; 
// PHP Fatal error: Uncaught Error: Cannot access private const A::BAR 
```

最后警告一句。尽管它们是不可变的，但常量是全局的，这使它们成为你的应用程序的状态。任何使用常量的函数实际上都是不纯的，因此你应该谨慎使用它们。

## RFC 正在进行中。

正如我们刚才看到的，常量在不可变性方面充其量只是一个木腿。它们非常适合存储诸如我们希望每页显示的项目数量之类的简单信息。但是一旦您想要有一些复杂的数据结构，您将会陷入困境。

幸运的是，PHP 核心团队的成员们都很清楚不可变性的重要性，目前正在进行一项 RFC 的工作，以在语言级别包含它（[`wiki.php.net/rfc/immutability`](https://wiki.php.net/rfc/immutability)）。

对于不了解新 PHP 功能涉及的流程的人来说，**请求评论**（**RFC**）是核心团队成员提出向 PHP 添加新内容的建议。该建议首先经过草案阶段，在此阶段编写并进行了一些示例实现。之后，进行讨论阶段，其他人可以提供建议和建议。最后，进行投票决定是否将该功能包含在下一个 PHP 版本中。

在撰写本文时，*不可变类和属性* RFC 仍处于草案阶段。对此既没有真正的赞成意见，也没有反对意见。只有时间会告诉我们它是否被接受。

## 值对象

来自[`en.wikipedia.org/wiki/Value_object`](https://en.wikipedia.org/wiki/Value_object)：

> *在计算机科学中，值对象是表示简单实体的小对象，其相等性不是基于标识的：即两个值对象在具有相同值时是相等的，不一定是相同的对象。*
> 
> *[...]*
> 
> *值对象应该是不可变的：这是两个相等的值对象的隐式契约所要求的，应该保持相等。值对象不可变也是有用的，因为客户端代码不能在实例化后将值对象置于无效状态或引入错误行为。*

由于在 PHP 中无法获得真正的不可变性，通常通过在类上具有私有属性和没有 setter 来实现。因此，当开发人员想要修改值时，强制他们创建一个新对象。该类还可以提供实用方法来简化新对象的创建。让我们看一个简短的例子：

```php
<?php 

class Message 
{ 
    private $message; 
    private $status; 

    public function __construct(string $message, string $status) 
    { 
        $this->status = $status; 
        $this->message = $message; 
    } 

    public function getMessage() 
    { 
        return $this->message; 
    } 

    public function getStatus() 
    { 
        return $this->status; 
    } 

    public function equals($m) 
    { 
        return $m->status === $this->status && 
               $m->message === $this->message; 
    } 

    public function withStatus($status): Message 
    { 
        $new = clone $this; 
        $new->status = $status; 
        return $new; 
    } 
} 
```

这种模式可以用于创建从数据使用者的角度来看是不可变的数据实体。但是，您必须特别小心，以确保类上的所有方法都不会破坏不可变性；否则，您所有的努力都将是徒劳的。

除了不可变性之外，使用值对象还有其他好处。您可以在对象内部添加一些业务或领域逻辑，从而将所有相关内容保持在同一位置。此外，如果您使用它们而不是数组，您可以：

+   将它们用作类型提示，而不仅仅是数组

+   避免由于拼写错误的数组键而导致任何可能的错误

+   强制存在或格式化某些项目

+   提供格式化值以适应不同上下文的方法

值对象的常见用途是存储和操作与货币相关的数据。您可以查看[`money.rtfd.org`](http://money.rtfd.org)，这是一个很好的如何有效使用它们的示例。

另一个对于真正重要的代码片段使用值对象的例子是**PSR-7: "HTTP 消息接口"**。这个标准引入并规范了一种框架和应用程序以可互操作的方式管理 HTTP 请求和响应的方法。所有主要的框架都有核心支持或可用的插件。我邀请您阅读他们为什么应该在 PHP 生态系统的如此重要的部分使用不可变性的完整理由：[`www.php-fig.org/psr/psr-7/meta/#why-value-objects`](http://www.php-fig.org/psr/psr-7/meta/#why-value-objects)。

从本质上讲，将 HTTP 消息建模为值对象可以确保消息状态的完整性，并且可以避免双向依赖的需要，这往往会导致不同步或导致调试或性能问题。

总的来说，值对象是在 PHP 中获得某种不可变性的好方法。您不会获得所有的好处，特别是与性能相关的好处，但大部分认知负担都被移除了。进一步探讨这个话题超出了本书的范围；如果您想了解更多，可以访问专门的网站：[`www.phpvalueobjects.info/`](http://www.phpvalueobjects.info/)。

## 不可变集合的库

如果您想进一步走向不可变性之路，至少有两个库提供不可变集合：**Laravel 集合**和**immutable.php**。

这两个库都协调了与数组相关的 PHP 函数的参数顺序的差异，比如`array_map`和`array_filter`。它们还提供了与大多数 PHP 函数相反的工作任何类型的`Iterable`或`Traversable`的可能性；这些函数通常需要一个真正的数组。

本章将只是快速介绍这些库。示例用法将在第三章中给出，*PHP 中的功能基础*，以便它们可以与允许执行相同任务的其他库一起显示。此外，我们还没有详细介绍诸如映射或折叠等技术，因此示例可能不够清晰。

### Laravel 集合

Laravel 框架包含一个名为`Collection`的类，用于取代 PHP 数组。这个类在内部使用一个简单的数组，但可以使用 collect 辅助函数从任何集合类型的变量创建。然后，它提供了许多非常有用的方法来处理数据，主要以一种功能性的方式。这也是 Laravel 的一个核心部分，因为**Eloquent**，ORM，将数据库实体作为`Collection`实例返回。

如果您不使用 Laravel，但仍希望从这个优秀的库中受益，您可以使用[`github.com/tightenco/collect`](https://github.com/tightenco/collect)，这只是从 Laravel 支持包的其余部分中分离出来的 Collection 部分，以保持小巧。您也可以参考 Laravel 集合的官方文档([`laravel.com/docs/5.3/collections`](https://laravel.com/docs/5.3/collections))。

### Immutable.php

这个库定义了`ImmArray`类，它实现了一个类似数组的不可变集合。

`ImmArray`类是`SplFixedArray`类的包装器，用于修复其 API 的一些缺陷，提供了通常希望在集合上执行的性能操作的方法。在幕后使用`SplFixedArray`类的优势在于其实现是用 C 编写的，性能非常高且内存效率高。您可以参考 GitHub 存储库以获取有关 Immutable.php 的更多信息：[`github.com/jkoudys/immutable.php`](https://github.com/jkoudys/immutable.php)。

# 引用透明度

如果您的代码库中的所有表达式都可以在任何时候用其输出替换而不改变程序的行为，则该表达式被称为引用透明。为了做到这一点，您的所有函数都必须是纯函数，所有变量都必须是不可变的。

我们从引用透明性中获得了什么？再一次，它有助于减少认知负担。让我们想象一下我们有以下函数和数据：

```php
<?php 

// The Player implementation is voluntarily simple for brevity. // Obviously you would use immutable.php in a real project. class Player 
{ 
    public $hp; 
    public $x; 
    public $y; 

    public function __construct(int $x, int $y, int $hp) { 
        $this->x = $x; 
        $this->y = $y; 
        $this->hp = $hp; 
    } 
} 

function isCloseEnough(Player $one, Player $two): boolean 
{ 
    return abs($one->x - $two->x) < 2 && 
           abs($one->y - $two->y) < 2; 
} 

function loseHitpoint(Player $p): Player 
{ 
    return new Player($p->x, $p->y, $p->hp - 1); 
} 

function hit(Player $p, Player $target): Player 
{ 
    return isCloseEnough($p, $target) ? loseHitpoint($target) : 
        $target; 
} 
```

现在让我们模拟两个人之间的一场非常简单的争吵：

```php
<?php 

$john=newPlayer(8, 8, 10); 
$ted =newPlayer(7, 9, 10); 

$ted=hit($john, $ted); 
```

上面定义的所有函数都是纯函数，由于我们没有可变的数据结构，它们也是引用透明的。现在，为了更好地理解我们的代码片段，我们可以使用一种称为**等式推理**的技术。这个想法非常简单，你只需要用*等于替换等于*来推理代码。在某种程度上，这就像手动评估代码。

让我们首先将我们的`isCloseEnough`函数内联。这样做，我们的 hit 函数可以被转换为如下形式：

```php
<?php 

return abs($p->x - $target->x) < 2 && abs($p->y - $target->y) < 2 ? loseHitpoint($target) : 
    $target; 
```

我们的数据是不可变的，现在我们可以简单地使用以下值：

```php
<?php 

return abs(8 - 7) < 2 && abs(8 - 8) < 2 ? loseHitpoint($target) : 
    $target; 
```

让我们做一些数学：

```php
<?php 

return 1<2 && 0<2 ? loseHitpoint($target) : 
    $target; 
```

条件显然评估为 true，所以我们只保留右分支：

```php
<?php 

return loseHitpoint($target); 
```

让我们继续进行剩余的函数调用：

```php
<?php 

return newPlayer($target->x, $target->y, $target->hp-1); 
```

再次替换值：

```php
<?php 

return newPlayer(8, 7, 10-1); 
```

最后，我们的初始函数调用变成了：

```php
<?php 

$ted = newPlayer(8, 7, 9); 
```

通过使用可以用其结果值替换引用透明表达式的事实，我们能够将一个相对冗长的代码片段减少到一个简单的对象创建。

这种能力应用于重构或理解代码非常有用。如果你在理解一些代码时遇到困难，并且你知道其中的一部分是纯的，你可以在尝试理解它时简单地用结果替换它。这可能会帮助你找到问题的核心。

## 非严格性或惰性评估

引用透明性的一个巨大好处是编译器或解析器可以惰性地评估值的可能性。例如，Haskell 允许你通过数学函数定义无限列表。语言的惰性特性确保列表的值只在需要值时才计算。

在术语表中，我们将非严格语言定义为评估发生惰性的语言。事实上，惰性和非严格性之间有一些细微差别。如果你对细节感兴趣，你可以访问[`wiki.haskell.org/Lazy_vs._non-strict`](https://wiki.haskell.org/Lazy_vs._non-strict)并阅读相关内容。在本书的目的上，我们将这些术语互换使用。

你可能会问自己这有什么用。让我们简单地看一下用例。

### 性能

通过使用惰性评估，你确保只有需要的值才会被有效计算。让我们看一个简短而天真的例子来说明这个好处：

```php
<?php 

function wait(int $value): int 
{ 
    // let's imagine this is a function taking a while 
    // to compute a value 
    sleep(10); 
    return $value; 
} 

function do_something(bool $a, int $b, int $c): int 
{ 
    if($a) { 
        return $b; 
    } else { 
        return $c; 
    } 
} 

do_something(true, sleep(10), sleep(8)); 
```

由于 PHP 在函数参数上不执行惰性评估，当调用`do_something`时，你首先必须等待两次 10 秒，然后才能开始执行函数。如果 PHP 是一种非严格语言，只有我们需要的值才会被计算，从而将所需的时间减少了一半。情况甚至更好，因为返回值甚至没有保存在一个新变量中，可能根本不需要执行函数。

PHP 有一种情况下执行一种惰性评估：布尔运算符短路。当你有一系列布尔操作时，只要 PHP 能够确定结果，它就会停止执行：

```php
<?php 

// 'wait' will never get called as those operators are short- circuited 

$a= (false && sleep(10));   
$b = (true  || sleep(10)); 
$c = (false and sleep(10)); 
$d = (true  or  sleep(10)); 
```

我们可以重写我们之前的例子以利用这一点。但正如你在下面的例子中看到的，这是以可读性为代价的。此外，我们的例子真的很简单，不是你在现实生活应用代码中会遇到的东西。想象一下为具有多个可能分支的复杂函数做同样的事情？这在下面的片段中显示：

```php
<?php 

($a && sleep(10)) || sleep(8); 
```

前面的代码还有两个更大的问题：

+   如果由于任何原因，第一次调用 sleep 返回 false 值，第二次调用也将被执行

+   你的方法的返回值将自动转换为布尔值

### 代码可读性

当你的变量和函数评估是惰性的时，你可以花更少的时间考虑声明的最佳顺序，甚至你计算的数据是否会被使用。相反，你可以专注于编写可读的代码。想象一个博客应用程序有很多帖子、标签、类别，并按年份存档。你是想为每个页面编写自定义查询，还是使用惰性评估，如下所示：

```php
<?php 

// let's imagine $blogs is a lazily evaluated collection 
// containing all the blog posts of your application order by date 
$posts = [ /* ... */ ]; 

// last 10 posts for the homepage 
return $posts->reverse()->take(10); 

// posts with tag 'functional php' 
return $posts->filter(function($b) { 
    return $b->tags->contains('functional-php'); 
})->all(); 

// title of the first post from 2014 in the category 'life' 
return $posts->filter(function($b) { 
    return $b->year == 2014; 
})->filter(function($b) { 
    return $b->category == 'life'; 
})->pluck('title')->first(); 
```

清楚地说，如果我们将所有帖子加载到`$posts`中，这段代码可能会工作得很好，但性能会非常糟糕。然而，如果我们有惰性评估和足够强大的 ORM，数据库查询可以延迟到最后一刻。那时，我们将确切地知道我们需要的数据，SQL 将自动为这个确切的页面定制，使我们拥有易于阅读的代码和出色的性能。

据我所知，这个想法纯粹是假设的。我目前并不知道有任何 ORM 足够强大，即使在最功能强大的语言中，也无法达到这种程度的懒惰。但如果可以的话，那不是很好吗？

如果你对示例中使用的语法感到困惑，那是受到了我们之前讨论的 Laravel 的 Collection 的 API 的启发。

### 无限列表或流

惰性求值允许你创建无限列表。在 Haskell 中，要获取所有正整数的列表，你可以简单地使用`[1..]`。然后，如果你想要前十个数字，你可以取`10 [1..]`。我承认这个例子并不是很令人兴奋，但更复杂的例子更难理解。

PHP 自版本 5.5 起支持生成器。你可以通过使用它们来实现类似无限列表的东西。例如，我们所有正整数的列表如下：

```php
<?php 

function integers() 
{ 
    $i=0; 
    while(true) yield $i++; 
} 
```

然而，懒惰无限列表和我们的生成器之间至少有一个显著的区别。你可以对 Haskell 版本执行任何你通常对集合执行的操作-例如计算其长度和对其进行排序。而我们的生成器是一个`Iterator`，如果你尝试在其上使用`iterator_to_array`，你的 PHP 进程很可能会一直挂起，直到内存耗尽。

你问我如何计算无限列表的长度或对其进行排序？实际上很简单；Haskell 只会计算列表值，直到它有足够的值来执行计算。比如我们在 PHP 中有条件`count($list) < 10`，即使你有一个无限列表，Haskell 会在达到 10 时停止计数，因为它在那时就会有一个比较的答案。

## 代码优化

看一下下面的代码，然后尝试决定哪个更快：

```php
<?php 

$array= [1, 2, 3, 4, 5, 6 /* ... */]; 

// version 1 
for($i = 0; $i < count($array); ++$i) { 
    // do something with the array values 
} 

// version 2 
$length = count($array); 
for($i = 0; $i < $length; ++$i) { 
    // do something with the array values 
} 
```

版本 2 应该快得多。因为你只计算数组的长度一次，而在版本 1 中，PHP 必须在每次验证 for 循环的条件时计算长度。这个例子很简单，但有些情况下这样的模式更难发现。如果你有引用透明性，这并不重要。编译器可以自行执行这种优化。任何引用透明的计算都可以在不改变程序结果的情况下移动。这是可能的，因为我们保证每个函数的执行不依赖于全局状态。因此，移动计算以实现更好的性能是可能的，而不改变结果。

另一个可能的改进是执行常见子表达式消除或 CSE。编译器不仅可以更自由地移动代码的一部分，还可以将一些共享公共计算的操作转换为使用中间值。想象一下以下代码：

```php
<?php 

$a= $foo * $bar + $u; 
$b = $foo * $bar * $v; 
```

如果计算`$foo * $bar`的成本很高，编译器可以决定通过使用中间值来转换它：

```php
<?php 

$tmp= $foo * $bar; 
$a = $tmp + $u; 
$b = $tmp * $v; 
```

再次强调，这只是一个很简单的例子。这种优化可以在整个代码库的范围内进行。

## 记忆化

记忆化是一种技术，它可以缓存给定参数集的函数的结果，这样你就不必在下一次调用时再次执行它。我们将在第八章*性能效率*中详细讨论这个问题。现在，让我只说一下，如果你的语言只具有引用透明表达式，它可以在需要时自动执行记忆化。

这意味着它可以根据调用的频率和其他各种参数来决定是否值得自动记忆函数，而无需开发人员的干预或提示。

# PHP 中的一切？

如果 PHP 开发人员只能从其中的一小部分优势中受益，那么为什么要费心纯函数、不可变性，最终是引用透明呢？

首先，就像不可变性的 RFC 一样，事情正在朝着正确的方向发展。这意味着，最终，PHP 引擎将开始纳入一些先进的编译器技术。当这发生时，如果你的代码库已经使用了这些函数式技术，你将获得巨大的性能提升。

其次，在我看来，所有这些的主要好处是减少认知负担。当然，要适应这种新的编程风格需要一些时间。但一旦你练习了一下，你很快就会发现你的代码更容易阅读和理解。其结果是你的应用程序将包含更少的错误。

最后，如果你愿意使用一些外部库，或者如果你能够应对语法并不总是很完善的情况，你现在就可以从其他改进中受益了。显然，我们无法改变 PHP 的核心以添加我们之前谈到的编译器优化，但在接下来的章节中，我们将看到一些引用透明性的好处是如何被模拟的。

# 总结

这一章包含了很多理论。希望你不会介意太多。这是必要的，以奠定我们共同词汇的基础，并解释为什么这些概念很重要。你现在很清楚纯度和不可变性是什么，也学会了一些识别不纯函数的技巧。我们还讨论了这两个属性如何导致了所谓的引用透明性以及好处是什么。

我们也了解到，遗憾的是，PHP 并不原生支持大部分的好处。然而，关键的收获是使用函数式方法减少了理解代码的认知负担，从而使其更容易阅读。最终的好处是现在你的代码将更容易维护和重构，你可以快速找到并修复错误。通常，纯函数也更容易测试，这也会导致更少的错误。

现在我们已经很好地讨论了理论基础，接下来的章节将专注于帮助我们在软件中实现纯度和不可变性的技术。
