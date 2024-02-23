# 第十三章：使用 PHP 的函数式数据结构

近年来，对函数式编程语言的需求超过了面向对象编程。其中一个核心原因是函数式编程具有固有的并行性。虽然面向对象编程被广泛使用，但函数式编程在近年来也有相当大的影响。因此，像 Erlang、Elixir、Clojure、Scala 和 Haskell 这样的语言是程序员最受欢迎的函数式编程语言。PHP 不在这个列表上，因为 PHP 被认为是一种命令式和面向对象的语言。尽管 PHP 对函数式编程有很多支持，但它主要用于面向对象编程和命令式编程。FP 的核心精髓是 λ 演算，它表示一种数学逻辑和计算机科学中的形式系统，用于通过变量绑定和替换来表达计算。它不是一个框架或一个新概念。事实上，函数式编程早于所有其他编程范式。它已经存在很长时间，将来也会存在，因为世界需要更多的并发计算和更快的处理语言。在本章中，您将学习如何使用 PHP 实现函数式编程和数据结构。

# 使用 PHP 理解函数式编程

与任何面向对象编程语言不同，其中一切都通过对象表示，函数式编程开始思考一切都是函数。OOP 和 FP 不是相互排斥的。虽然 OOP 侧重于通过封装和继承实现代码的可维护性和可重用性，但与面向状态的命令式编程不同，函数式编程侧重于以值为导向的编程，将计算视为纯数学评估，并避免可变性和状态修改。在使用 OOP 时，其中一个挑战是我们创建的对象可能会带来许多额外的属性或方法，无论我们是否在特定情况下使用它。以下是函数式编程的三个关键特征：

+   不可变性

+   纯函数和引用透明度

+   头等公民函数

+   高阶函数

+   函数组合（柯里化）

不可变性的概念告诉我们，一个对象在创建后不会改变。它在整个生命周期内保持不变。这有一个很大的优势，因为我们在使用对象时不需要重新验证对象。此外，如果需要可变性，我们可以创建对象的副本或创建具有新属性的新对象。

到目前为止，在这本书中，我们看到了很多使用代码块、循环和条件的数据结构和算法的例子。一般来说，这被称为命令式编程，其中期望定义执行的每一步。例如，考虑以下代码块：

```php
$languages = ["php", "python", "java", "c", "erlang"];

foreach ($languages as $ind => $language) {

    $languages[$ind] = ucfirst($language);

}

```

前面的代码实际上将每个名称的第一个字符设置为大写。从逻辑上讲，代码是正确的，我们已经逐步呈现了它，以便我们理解发生了什么。然而，这可以使用函数式编程方法写成一行代码。

```php
$languages = array_map('ucfirst', $languages);

```

这两种方法都是做同样的事情，但一种比另一种代码块要小。后者被称为声明式编程。而命令式编程侧重于算法和步骤，声明式编程侧重于函数的输入和输出以及递归（而不是迭代）。

函数式编程的另一个重要方面是它不受任何副作用的影响。这是一个重要的特性，它确保函数不会在输入的任何地方产生任何隐式影响。函数式编程的一个常见例子是在 PHP 中对数组进行排序。通常，参数是通过引用传递的，当我们得到排序后的数组时，它实际上破坏了初始数组。这是函数中副作用的一个例子。

在跳入 PHP 的函数式编程之前，让我们探索一些函数式编程术语，我们将在接下来的部分中遇到。

# 一流函数

具有一流函数的语言允许以下行为：

+   将函数分配给变量

+   将它们作为参数传递给另一个函数

+   返回一个函数

PHP 支持所有这些行为，因此 PHP 函数是一流函数。在我们之前的例子中，`ucfirst`函数就是一流函数的一个例子。

# 高阶函数

高阶函数可以将一个或多个函数作为参数，并且还可以返回一个函数作为结果。PHP 也支持高阶函数；我们之前的例子中的`array_map`就是一个高阶函数。

# 纯函数

纯函数

# Lambda 函数

Lambda 函数或匿名函数是没有名称的函数。当作为一流函数（分配给变量）使用时，或者用于回调函数时，它们非常方便，我们可以在调用参数的位置定义函数。PHP 也支持匿名函数。

# 闭包

闭包与 Lambda 函数非常相似，但基本区别在于闭包可以访问其外部作用域变量。在 PHP 中，我们无法直接访问外部作用域变量。为了做到这一点，PHP 引入了关键字"use"，以将任何外部作用域变量传递给内部函数。

# 柯里化

柯里化是一种将接受多个参数的函数转换为一系列函数的技术，其中每个函数将只接受一个参数。换句话说，如果一个函数可以写成*f(x,y,z)*，那么它的柯里化版本将是*f(x)(y)(z)*。让我们考虑以下例子：

```php
function sum($a, $b, $c) {

    return $a + $b + $c;

}

```

在这里，我们编写了一个带有三个参数的简单函数，当用数字调用时，它将返回数字的*和*。现在，如果我们将这个函数写成柯里化形式，它将如下所示：

```php
function currySum($a) { 

    return function($b) use ($a) { 

        return function ($c) use ($a, $b) { 

            return $a + $b + $c; 

        }; 

    }; 

} 

$sum = currySum(10)(20)(30); 

echo $sum;

```

现在，如果我们将`currySum`作为柯里化函数运行，我们将得到前面例子中的结果 60。这是函数式编程非常有用的特性。

在 PHP 中以前不可能像*f(a)(b)(c)*这样调用一个函数。自 PHP 7.0 以来，统一变量语法允许立即执行可调用函数，就像我们在这个例子中看到的那样。然而，在 PHP 5.4 和更高版本中，为了做到这一点，我们必须创建临时变量来存储 Lambda 函数。

# 部分应用

部分应用或部分函数应用是一种技术，可以减少函数的参数数量，或者使用部分参数并创建另一个函数来处理剩余的参数，以产生与一次性调用所有参数时相同的输出。如果我们将我们的`sum`函数视为部分应用，它预期接受三个参数，但我们可以用两个参数调用它，然后再添加剩下的一个。以下是代码示例。在这个例子中使用的`sum`函数来自前面的部分：

```php
function partial($funcName, ...$args) { 

    return function(...$innerArgs) use ($funcName, $args) { 

        $allArgs = array_merge($args, $innerArgs); 

        return call_user_func_array($funcName, $allArgs); 

    }; 

} 

$sum = partial("sum", 10, 20); 

$sum = $sum(30); 

echo $sum;

```

有时，我们会对柯里化和部分应用感到困惑，尽管它们在方法和原则上完全不同。

正如我们所看到的，处理 PHP 中的函数式编程时需要考虑的事情很多。使用函数式编程从头开始实现数据结构将是一个更加冗长的过程。为了解决这个问题，我们将探索一个名为**Tarsana**的 PHP 优秀函数式编程库。它是开源的，并且使用 MIT 许可证。我们将探索这个库，并将其作为我们在 PHP 中实现函数式数据结构的基础。

# 开始使用 Tarsana

Tarsana 是由 Amine Ben Hammou 编写的开源库，可在 GitHub 上下载。它受到 JavaScript 的函数式编程库 Ramda JS 的启发。它没有任何依赖关系，有 100 多个预定义的函数可用于不同的目的。FP 中的函数分布在不同的模块中，有几个模块，如函数、列表、对象、字符串、数学、运算符和常见模块。Tarsana 可以从 GitHub（[`github.com/Tarsana/functional`](https://github.com/Tarsana/functional)）下载，也可以通过 composer 安装。

```php
composer require Tarsana/functional

```

一旦库被下载，我们必须通过导入`Tarsana\Functional`命名空间来使用它，就像以下代码一样：

```php
use Tarsana\Functional as F; 

```

Tarsana 的一个有趣特性是，我们可以将我们现有的任何函数转换为柯里化函数。例如，如果我们想使用我们的`sum`函数使用 Tarsana，那么它将如下所示：

```php
require __DIR__ . '/vendor/autoload.php'; 

use Tarsana\Functional as F; 

$add = F\curry(function($x, $y, $z) { 

    return $x + $y + $z; 

}); 

echo $add(1, 2, 4)."\n"; 

$addFive = $add(5); 

$addSix = $addFive(6); 

echo $addSix(2); 

```

这将分别产生 7 和 13 的输出。Tarsana 还有一个选项，可以使用`__()`函数来保留占位符。以下示例显示了在占位符中提供的条目的数组减少和数组求和：

```php
$reduce = F\curry('array_reduce'); 

$sum = $reduce(F\__(), F\plus()); 

echo $sum([1, 2, 3, 4, 5], 0);

```

Tarsana 还提供了管道功能，可以从左到右应用一系列函数。最左边的函数可以有任何 arity；其余的函数必须是一元的。管道的结果不是柯里化的。让我们考虑以下示例：

```php
$square = function($x) { return $x * $x; }; 

$addThenSquare = F\pipe(F\plus(), $square); 

echo $addThenSquare(2, 3);

```

由于我们已经探索了 Tarsana 的一些功能，我们准备开始使用 Tarsana 来开始我们的函数式数据结构。我们还将使用简单的 PHP 函数来实现这些数据结构，这样我们两方面都覆盖到了，如果我们不想使用函数式编程。让我们开始实现堆栈。

# 实现堆栈

我们已经在第四章中看到了堆栈的实现，*构建堆栈和队列*。为简单起见，我们不会再讨论整个堆栈操作。我们将直接进入使用函数式编程实现推送、弹出和顶部操作。Tarsana 有许多用于列表操作的内置函数。我们将使用它们的内置函数来实现堆栈的功能。以下是实现方式：

```php
require __DIR__ . '/vendor/autoload.php'; 

use Tarsana\Functional as F; 

$stack = []; 

$push = F\append(F\__(), F\__()); 

$top = F\last(F\__()); 

$pop = F\init(F\__()); 

$stack = $push(1, $stack); 

$stack = $push(2, $stack); 

$stack = $push(3, $stack); 

echo "Stack is ".F\toString($stack)."\n"; 

$item = $top($stack); 

$stack = $pop($stack); 

echo "Pop-ed item: ".$item."\n"; 

echo "Stack is ".F\toString($stack)."\n"; 

$stack = $push(4, $stack); 

echo "Stack is ".F\toString($stack)."\n"; 

```

在这里，我们使用 Tarsana 的 append 函数进行推送操作，在这里我们使用的是 top 操作的 last 函数，以及 pop 操作的`init`函数。以下代码的输出如下：

```php
Stack is [1, 2, 3]

Pop-ed item: 3

Stack is [1, 2]

Stack is [1, 2, 4]

```

# 实现队列

我们可以使用 Tarsana 和列表操作的内置函数来实现队列。我们还将使用这段代码来使用数组表示队列：

```php
require __DIR__ . '/vendor/autoload.php'; 

use Tarsana\Functional as F; 

$queue = []; 

$enqueue = F\append(F\__(), F\__()); 

$head = F\head(F\__()); 

$dequeue = F\tail(F\__()); 

$queue = $enqueue(1, $queue); 

$queue = $enqueue(2, $queue); 

$queue = $enqueue(3, $queue); 

echo "Queue is ".F\toString($queue)."\n"; 

$item = $head($queue); 

$queue = $dequeue($queue); 

echo "Dequeue-ed item: ".$item."\n"; 

echo "Queue is ".F\toString($queue)."\n"; 

$queue = $enqueue(4, $queue); 

echo "Queue is ".F\toString($queue)."\n"; 

```

在这里，我们使用`append`函数来执行入队操作，使用`head`和`tail`函数分别获取队列中的第一个项目和出队操作。以下是前面代码的输出：

```php
Queue is [1, 2, 3]

Dequeue-ed item: 1

Queue is [2, 3]

Queue is [2, 3, 4]

```

现在，我们将把重点转移到使用简单的 PHP 函数来实现分层数据，而不是使用类和对象。由于函数式编程在 PHP 中仍然是一个新话题，实现分层数据可能看起来具有挑战性，而且也很耗时。相反，我们将使用基本的 PHP 函数以及一些基本的函数式编程概念，如一等函数和高阶函数来转换我们的分层数据实现。因此，让我们实现一个二叉树。

# 实现树

我们将使用一个简单的递归函数来使用 PHP 数组实现二叉树的遍历。我们只是使用一个函数重写了功能，而不是一个类。以下是实现此功能的代码：

```php
function treeTraverse(array &$tree, int $index = 0,

int $level = 0, &$outputStr = "") : ?bool {

    if(isset($tree[$index])) {

        $outputStr .= str_repeat("-", $level); 

        $outputStr .= $tree[$index] . "\n";

        treeTraverse($tree, 2 * $index + 1, $level+1,$outputStr);      

        treeTraverse($tree, 2 * ($index + 1), $level+1,$outputStr);

    } else { 

        return false; 

    }

 return null; 

} 

        $nodes = []; 

        $nodes[] = "Final"; 

        $nodes[] = "Semi Final 1"; 

        $nodes[] = "Semi Final 2"; 

        $nodes[] = "Quarter Final 1"; 

        $nodes[] = "Quarter Final 2"; 

        $nodes[] = "Quarter Final 3"; 

        $nodes[] = "Quarter Final 4"; 

        $treeStr = ""; 

        treeTraverse($nodes,0,0,$treeStr); 

        echo $treeStr; 

```

如果我们看一下前面的代码，我们只是修改了遍历函数，并将其转换为一个独立的函数。这是一个纯函数，因为我们在这里没有修改实际的输入，即`$nodes`变量。我们将在每个级别构造一个字符串，并将其用于输出。现在，我们可以将大部分基于类的结构转换为基于函数的结构。

# 摘要

函数式编程对于 PHP 开发者来说相对较新，因为对其先决条件的支持是在 5.4 版本中添加的。函数式编程的出现将要求我们理解这种范式，并在需要时编写没有任何副作用的纯函数。PHP 对编写函数式编程代码有一定的支持，通过这种支持，我们也可以编写函数式数据结构和算法实现，正如我们在本书中尝试展示的那样。在不久的将来，优化和提高我们应用程序的效率时，这可能会派上用场。
