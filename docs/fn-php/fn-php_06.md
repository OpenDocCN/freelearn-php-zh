# 第六章。现实生活中的单子

在上一章中，我们涵盖了关于各种抽象的许多理论基础，引导我们到单子的概念。现在是时候应用这些知识，通过介绍一些单子的实例，这些实例将在您日常编码中证明有用。

每个部分都将以解决给定单子的问题的介绍开始，然后是一些用法示例，以便您可以获得一些实践。正如本介绍末尾所解释的那样，书中不会呈现实现本身，而是集中于用法。

正如您将看到的，一旦理论问题解决了，大多数实现对您来说将会显得非常自然。此外，其实用性不仅限于函数式编程的范围。本章中学到的大部分内容都可以应用于任何开发环境。

将要介绍的大多数单子都与副作用的管理有关，或者说一旦它们明确包含在单子中就是影响。在进行函数式编程时，副作用是不受欢迎的。一旦包含，我们就可以控制它们，使它们仅仅成为我们程序的影响。

单子主要用于两个原因。第一个是它们非常适合执行流程控制，正如在上一章中已经解释的那样。第二个是它们的结构允许您轻松地封装效果并保护代码的其余部分免受杂质的影响。

然而，让我们记住，这只是单子的一个可能用途。您可以用这个概念做更多的事情。但是让我们不要过于急躁；我们将在途中发现这一点。

在本章中，我们将涵盖以下主题：

+   单子辅助方法

+   Maybe 和 Either 单子

+   List 单子

+   Writer 单子

+   Reader 单子

+   State 单子

+   IO 单子

为了专注于使用单子，并且由于实现通常不是最重要的部分，我们将使用**PHP Functional**库提供的单子。显然，重要的实现细节将在书中突出显示。您可以使用`composer`调用在您的项目中安装它。

```php
**composer require widmogrod/php-functional**

```

重要的是要注意，`php-functional`库的作者在方法命名和一些实现细节方面做出了其他选择：

+   `apply`方法简单地是`ap`

+   `unit`和`return`关键字在类中被`of`替换

+   继承树有点不同，例如，有`Pointed`和`Chain`接口

+   该库使用特征来共享代码

+   一些辅助函数是在类外实现的，需要单独导入

# 单子辅助方法

在上一章中，我们谈到了`flatten`方法以及它如何用于压缩相同单子实例的多个嵌套级别。这个函数经常被提及，因为它可以用于以另一种方式重写单子。然而，还有其他有用的辅助函数。

## filterM 方法

过滤是函数式编程中的一个关键概念，但是如果我们的过滤函数返回的是一个单子而不是一个简单的布尔值呢？这就是`filterM`方法的用途。该方法不是期望返回一个简单的布尔值的谓词，而是使用任何可以转换为布尔值并且还将结果集合包装在相同单子中的谓词：

```php
<?php 

use function Functional\head; 
use function Functional\tail; 

use Monad\Writer; 

function filterM(callable $f, $collection) 
{ 
    $monad = $f(head($collection)); 

    $_filterM = function($collection) use($monad, $f, &$_filterM){ 
        if(count($collection) == 0) { 
            return $monad->of([]); 
        } 

        $x = head($collection); 
        $xs = tail($collection); 

        return $f($x)->bind(function($bool) use($x, $xs, $monad, $_filterM) { 
            return $_filterM($xs)->bind(function(array $acc) use($bool, $x, $monad) { 
                if($bool) { 
                    array_unshift($acc, $x); 
                } 

                return $monad->of($acc); 
            }); 
        }); 
    }; 
    return $_filterM($collection); 
} 
```

实现有点难以理解，所以我会尝试解释发生了什么：

1.  首先，我们需要了解我们正在处理的单子的信息，因此我们提取我们的集合的第一个元素，并通过应用回调函数从中获取单子。

1.  然后我们声明一个围绕单子和谓词的闭包。

1.  闭包首先测试集合是否为空。如果是这种情况，我们将返回一个包含空数组的单子实例。否则，我们将在集合的第一个元素上运行谓词。

1.  我们将一个包含当前值的闭包绑定到包含布尔值的结果单子上。

1.  第二个闭包递归地遍历整个数组，如果需要的话。

1.  一旦我们到达最后一个元素，我们就会绑定一个新的闭包，它将使用布尔值将值添加到累加器中，或者不添加。

这并不容易，但由于它主要是内部管道工作，再加上 PHP 缺乏语法糖，理解一切并不是必要的。为了比较，这里是使用 Haskell 模式匹配和*do notation*功能实现的相同代码：

```php
filterM :: (Monad m) => (a -> m Bool) -> [a] -> m [a] 
filterM _ []     = return [] 
filterM f (x:xs) = do 
    bool <- f x 
    acc  <- filterM p xs 
    return (if bool then x:acc else acc) 
```

正如您所看到的，这样更容易阅读。我认为任何人都能理解发生了什么。不幸的是，在 PHP 中，我们必须创建嵌套的内部函数才能实现相同的结果。然而，这并不是真正的问题，因为最终的函数非常容易使用。然而，一些功能模式的内部工作有时在 PHP 中可能有点令人不快，并且它们本身并不完全功能。

随着我们发现一些单子，例子将随之而来。这个辅助函数的实现在`php-functional`库中可用。

## foldM 方法

`foldM`方法是`fold`方法的单子版本。它接受一个返回单子的函数，然后产生一个也是单子的值。然而，累加器和集合都是简单的值：

```php
<?php 

function foldM(callable $f, $initial, $collection) 
{ 
    $monad = $f($initial, head($collection)); 

    $_foldM = function($acc, $collection) use($monad, $f, &$_foldM){ 
        if(count($collection) == 0) { 
            return $monad->of($acc); 
        } 

        $x = head($collection); 
        $xs = tail($collection); 

        return $f($acc, $x)->bind(function($result) use($acc,$xs,$_foldM) { 
            return $_foldM($result, $xs); 
        }); 
    }; 

    return $_foldM($initial, $collection); 
} 
```

该实现比`filterM`方法的实现要小一点，因为我们只需要递归；不需要从布尔值到值的转换。同样，我们将在本章的后续部分展示一些例子，并且`php-funcational`库中也有实现。

## 结束语

存在多个其他函数可以增强为与单子值一起使用。例如，您可以使用`zipWithM`方法，它使用返回单子的合并函数合并两个集合。`php-functional`库有一个`mcompose`的实现，它允许您组合返回相同单子实例的函数。

当您在使用单子时发现某种重复模式时，不要犹豫将其因式分解为辅助函数。它可能经常会派上用场。

# Maybe 和 Either 单子

您应该已经非常清楚我们已经多次讨论过的 Maybe 和 Either 类型。我们首先定义了它们，然后我们了解到它们实际上是一个函子的完美例子。

我们现在将更进一步，将它们定义为单子，这样我们将能够在更多情况下使用它们。

## 动机

`Maybe`单子代表了一种计算序列随时可能停止返回有意义值的想法，使用我们在前一章中定义的`Nothing`类。当转换链相互依赖并且某些步骤可能无法返回值时，它特别有用。它允许我们避免通常伴随这种情况的可怕的`null`检查。

`Either`单子大部分具有相同的动机。微小的区别在于步骤通常要么抛出异常，要么返回错误，而不是空值。操作失败意味着我们需要存储由`Left`值表示的错误消息，而不是`Nothing`值。

## 实现

Maybe 和 Either 类型的代码可以在`php-functional`库中找到。实现非常简单-与我们自己先前的实现的主要区别是缺少`isJust`和`isNothing`等方法，并且实例是使用辅助函数构造而不是静态工厂。

重要的是要注意，`php-functional`库中实现的 Either 单子不幸地没有自己处理捕获异常。您要么应用的函数，要么绑定到它的函数必须自行正确处理异常。您还可以使用`tryCatch`辅助函数来为您执行此操作。

## 例子

为了更好地理解`Maybe`单子的工作原理，让我们看一些例子。`php-functional`库使用辅助函数而不是类上的静态方法来创建新实例。它们位于`Widmogrod\Monad\Maybe`命名空间中。

另一个非常有用的辅助函数是`maybe`方法，它是一个带有以下签名的柯里化函数-`maybe($default, callable $fn, Maybe $maybe)`命名空间。当调用时，它将首先尝试从`$maybe`变量中提取值，并默认为`$default`变量。然后将其作为参数传递给`$fn`变量：

```php
<?php 

use Widmogrod\Monad\Maybe as m; 
use Widmogrod\Functional as f; 

$just = m\just(10); 
$nothing = m\nothing(); 

$just = m\maybeNull(10); 
$nothing = m\maybeNull(null); 

echo maybe('Hello.', 'strtoupper', m\maybe('Hi!')); 
// HI! echo maybe('Hello.', 'strtoupper', m\nothing()); 
// HELLO. 
```

既然辅助函数已经完成，我们将演示如何将`Maybe`单子与`foldM`方法结合使用：

```php
<?php 

$divide = function($acc, $i) { 
    return $i == 0 ? nothing() : just($acc / $i); 
}; 

var_dump(f\foldM($divide, 100, [2, 5, 2])->extract()); 
// int(5) 

var_dump(f\foldM($divide, 100, [2, 0, 2])->extract()); 
// NULL 
```

使用传统函数和`array_reduce`方法来实现这一点，结果大多会非常相似，但它很好地演示了`foldM`方法的工作原理。由于折叠函数绑定到每次迭代的当前单子值，一旦我们有一个空值，接下来的步骤将继续返回空值，直到结束。同样的函数也可以用来返回其他类型的单子，以便还包含有关失败的信息。

我们之前已经看到单子类型如何用于在可能存在或不存在的值上链接多个函数。然而，如果我们需要使用这个值来获取另一个可能为空的值，我们将有嵌套的`Maybe`实例：

```php
<?php 

function getUser($username): Maybe { 
  return $username == 'john.doe' ? just('John Doe') : nothing(); 
} 

var_dump(just('john.doe')->map('getUser')); 
// object(Monad\Maybe\Just)#7 (1) { 
//     ["value":protected]=> object(Monad\Maybe\Just)#6 (1) { 
//         ["value":protected]=> string(8) "John Doe" 
//     } 
// } 

var_dump(just('jane.doe')->map('getUser')); 
// object(Monad\Maybe\Just)#8 (1) { 
//     ["value":protected]=> object(Monad\Maybe\Nothing)#6 (0) { } 
// } 
```

在这种情况下，您可以使用`flatten`方法，或者简单地使用`bind`方法而不是`map`方法：

```php
<?php 

var_dump(just('john.doe')->bind('getUser')); 
// object(Monad\Maybe\Just)#6 (1) { 
//     ["value":protected]=> string(8) "John Doe" 
// } 

var_dump(just('jane.doe')->bind('getUser')); 
// object(Monad\Maybe\Nothing)#8 (0) { } 
```

我同意`Maybe`单子的例子有点令人失望，因为大多数用法已经在之前的单子中描述过了，因此创建`Maybe`单子本身并不会增加功能，它只会允许我们使用其他期望单子的模式；功能与以前一样。

`Either`单子也可以做类似的情况；这就是为什么这里不会有新的例子。只需确保查看辅助函数，而不是在想要使用单子时重写管道。

# 列表单子

列表或集合单子代表了所有以集合作为参数并返回零个、一个或多个值的函数的范畴。该函数应用于输入列表中的所有可能值，并将结果连接起来生成一个新的集合。

重要的是要理解列表单子实际上并不代表一个简单的值列表，而是代表单子的所有不同可能值的列表。这个想法通常被描述为*非确定性*。正如我们在`CollectionApplicative`函数中看到的，当您将一组函数应用于一组值时，这可能会导致有趣的结果。我们将尝试在例子中扩展这个主题以澄清这一点。

## 动机

列表单子体现了这样一个观念，即在完整计算结束之前，您无法知道最佳结果。它允许我们探索所有可能的解决方案，直到我们有最终的解决方案。

## 实现

单子是在`php-functional`库中以`Collection`方法的名称实现的。这是以一种非常直接的方式完成的。然而，与我们自己以前的实现相比，有两种新方法可用：

+   `reduce`方法将对单子内存储的值执行折叠操作。

+   `traverse`方法将一个返回应用程序的函数映射到单子内存储的所有值。然后将应用程序应用于当前累加器。

## 例子

让我们从一些困难的事情开始，使用我们之前发现的`filterM`方法。我们将创建一个被称为集合的`powerset`。`powerset`集合是给定集合的所有可能子集，或者，如果您愿意，是其成员的所有可能组合：

```php
<?php 

use Monad\Collection; 
use Functional as f; 

$powerset = filterM(function($x) { 
    return Collection::of([true, false]); 
}, [1, 2, 3]); 

print_r($powerset->extract()); 
// Array ( 
//     [0] => Array ( [0] => 1 [1] => 2 [2] => 3 ) 
//     [1] => Array ( [0] => 1 [1] => 2 ) 
//     [2] => Array ( [0] => 1 [1] => 3 ) 
//     [3] => Array ( [0] => 1 ) 
//     [4] => Array ( [0] => 2 [1] => 3 ) 
//     [5] => Array ( [0] => 2 ) 
//     [6] => Array ( [0] => 3 ) 
//     [7] => Array ( ) // ) 
```

### 注意

由于构造函数没有在实际数组内包装另一个数组，所以这目前无法使用 Collection/filterM 的实际实现。请参阅[`github.com/widmogrod/php-functional/issues/31`](https://github.com/widmogrod/php-functional/issues/31)。

这里发生了什么？这似乎是某种黑魔法。事实上，这很容易解释。将函数绑定到集合会导致该函数应用于其所有成员。在这种特殊情况下，我们的过滤函数返回一个包含`true`和`false`值的集合。这意味着`filterM`方法的内部闭包负责用值替换布尔值被运行两次，然后结果被附加到先前创建的所有集合上。让我们看看第一步以使事情更清晰：

1.  过滤首先应用于值`1`，创建两个集合`[]`和`[1]`。

1.  现在过滤器应用于值`2`，创建两个新集合（`[]`和`[2]`），需要附加到我们之前创建的集合上，创建四个集合`[]`，`[1]`，`[2]`，`[1, 2]`。

1.  每一步都会创建两个集合，这些集合将附加到先前创建的集合上，使得集合的数量呈指数级增长。

还不清楚吗？让我们看另一个例子。这一次，试着将集合想象成一棵树，其中每个初始值都是一个分支。当你绑定一个函数时，它被应用于每个分支，如果结果是另一个集合，它就会创建新的分支：

```php
<?php 
use Monad\Collection; 
use Functional as f; 

$a = Collection::of([1, 2, 3])->bind(function($x) { 
    return [$x, -$x]; 
}); 
print_r($a->extract()); 
// Array ( 
//     [0] => 1 
//     [1] => -1 
//     [2] => 2 
//     [3] => -2 
//     [4] => 3 
//     [5] => -3 
// ) 

$b = $a->bind(function($y) { 
    return $y > 0 ? [$y * 2, $y / 2] : $y; 
}); 
print_r($b->extract()); 
// Array ( 
//     [0] => 2 
//     [1] => 0.5 
//     [2] => -1 
//     [3] => 4 
//     [4] => 1 
//     [5] => -2 
//     [6] => 6 
//     [7] => 1.5 
//     [8] => -3 
// ) 
```

为了让事情对你更加复杂一些，第二个函数根据给定的值返回可变数量的元素。让我们将其可视化为一棵树：

![Examples](img/image_06_001.jpg)

### 骑士可以去哪里？

现在我们对`Collection`单子的工作原理有了很好的理解，让我们来解决一个更困难的挑战。给定国际象棋棋盘上的起始位置，我们想知道骑士棋子在三步内可以到达的所有可能有效位置。

我希望你花一点时间想象一下你会如何实现它。一旦你完成了，让我们尝试使用我们的单子。我们首先需要一种方法来编码我们的骑士位置。一个简单的类就足够了。此外，国际象棋棋盘有八列和八行，所以让我们添加一个检查位置是否有效的方法：

```php
<?php 

class ChessPosition { 
    public $col; 
    public $row; 

    public function __construct($c, $r) 
    { 
        $this->col = $c; 
        $this->row = $r; 
    } 

    public function isValid(): bool 
    { 
        return ($this->col > 0 && $this->col < 9) && 
               ($this->row > 0 && $this->row < 9); 
    } 
} 

function chess_pos($c, $r) { return new ChessPosition($c, $r); } 
```

现在我们需要一个函数，给定一个起始位置，返回骑士的所有有效移动：

```php
<?php 

function moveKnight(ChessPosition $pos): Collection 
{ 
    return Collection::of(f\filter(f\invoke('isValid'), Collection::of([ 
        chess_pos($pos->col + 2, $pos->row - 1), 
        chess_pos($pos->col + 2, $pos->row + 1), 
        chess_pos($pos->col - 2, $pos->row - 1), 
        chess_pos($pos->col - 2, $pos->row + 1), 
        chess_pos($pos->col + 1, $pos->row - 2), 
        chess_pos($pos->col + 1, $pos->row + 2), 
        chess_pos($pos->col - 1, $pos->row - 2), 
        chess_pos($pos->col - 1, $pos->row + 2), 
    ]))); 
} 

print_r(moveKnight(chess_pos(8,1))->extract()); 
// Array ( 
//     [0] => ChessPosition Object ( [row] => 2 [col] => 6 ) 
//     [1] => ChessPosition Object ( [row] => 3 [col] => 7 ) 
// ) 
```

很好，看起来工作得很好。现在我们只需要连续三次绑定这个函数。而且，在此过程中，我们还将创建一个函数，检查骑士是否可以在三步内到达给定位置：

```php
<?php 

function moveKnight3($start): array 
{ 
    return Collection::of($start) 
        ->bind('moveKnight') 
        ->bind('moveKnight') 
        ->bind('moveKnight') 
        ->extract(); 
} 

function canReach($start, $end): bool 
{ 
    return in_array($end, moveKnight3($start)); 
} 

var_dump(canReach(chess_pos(6, 2), chess_pos(6, 1))); 
// bool(true) 

var_dump(canReach(chess_pos(6, 2), chess_pos(7, 3))); 
// bool(false) 
```

唯一剩下的事情就是在真正的国际象棋棋盘上检查我们的函数是否正确工作。我不知道你是如何以命令式的方式做到这一点的，但是我的解决方案比我们现在得到的解决方案要不那么优雅。

如果你想再玩一会儿，你可以尝试参数化移动的次数，或者为其他棋子实现这个功能。正如你将看到的，这只需要进行最小的更改。

# 写作单子

如果你记得的话，纯函数不能有任何副作用，这意味着你不能在其中放置调试语句，例如。如果你像我一样，`var_dump`方法是你的调试工具，那么你只能违反纯度规则或使用其他调试技术。由于函数的所有输出必须通过其返回值，脑海中首先浮现的一个想法是返回一个值元组-原始返回值和你需要的任何调试语句。

然而，这个解决方案非常复杂。想象一下，你有一个函数，它可以将一个数值减半，返回减半后的值和接收到的输入，用于调试目的。现在，如果你想将这个函数与自身组合，创建一个新的函数，返回被四除的值，你还需要修改输入，以便它们可以接受你的新返回格式。这一过程一直持续下去，直到你相应地修改了所有的函数。这也会对柯里化造成一些问题，因为现在你有了一个多余的参数，如果你不关心调试语句，这个参数实际上并不实用。

你正在寻找的解决方案是`Writer`monad。遗憾的是，在撰写本文时，`php-functional`库中还没有实现。

## 动机

Writer monad 用于封装函数的主要返回值旁边的某种相关语句。这个语句可以是任何东西。它经常用于存储生成的调试输出或跟踪信息。手动这样做是繁琐的，可能会导致复杂的管理代码。

`Writer` monad 提供了一种干净的方式来管理这种副输出，并允许你在返回简单值的函数旁边插入返回这种信息的函数。在计算序列结束时，附加值可以被丢弃、显示或根据操作模式进行任何处理。

## 实现

由于 monad 需要连接输出值，任何 monoid 的实例都可以被用作这样。为了简化基于字符串的日志记录，任何字符串也可以被直接管理。显然，使用一个具有缓慢操作的 monoid 将导致性能成本。

`php-functional`库包括一个`StringMonoid`类的实现，每个字符串都将被提升到这个类中。然而，`runWriter`方法将始终返回一个`StringMonoid`类，因此对于使用它的人来说并不奇怪。除此之外，这个实现非常简单直接。

## 示例

正如我们刚才看到的，`Writer`非常适合日志记录。结合`filter`方法，这可以用来理解在过滤函数中发生了什么，而无需倾向于转储值：

```php
<?php 

$data = [1, 10, 15, 20, 25]; 
$filter = function($i) { 
    if ($i % 2 == 1) { 
        return new Writer(false, "Reject odd number $i.\n"); 
    } else if($i > 15) { 
      return new Writer(false, "Reject $i because it is bigger than 15\n"); 
    } 

    return new Writer(true); 
}; 

list($result, $log) = filterM($filter, $data)->runWriter(); 

var_dump($result); 
// array(1) { 
//   [0]=> int(10) 
// } 

echo $log->get(); 
// Reject odd number 1\. // Reject odd number 15\. // Reject 20 because it is bigger than 15 
// Reject odd number 25\. 
```

正如我们所看到的，`Writer` monad 允许我们准确了解为什么某些数字被过滤掉。在这样一个简单的例子中，这可能看起来不像什么，但条件并不总是那么容易理解。

你也可以使用`Writer`来添加更传统的调试信息：

```php
<?php 

function some_complex_function(int $input) 
{ 
    $msg = new StringMonoid('received: '.print_r($input,  true).'.'); 

    if($input > 10) { 
        $w = new Writer($input / 2, $msg->concat(new  StringMonoid("Halved the value. "))); 
    } else { 
        $w = new Writer($input, $msg); 
    } 

    if($input > 20) 
    { 
        return $w->bind('some_complex_function'); 
    } 

    return $w; 
} 

list($value, $log) = (new Writer(15))->bind('some_complex_function')->runWriter(); 
echo $log->get(); 
// received: 15\. Halved the value. list($value, $log) = some_complex_function(27)->runWriter(); 
echo $log->get(); // received: 27\. Halved the value. received: 13\. Halved the value. list($value, $log) = some_complex_function(50)->runWriter(); 
echo $log->get(); 
// received: 50\. Halved the value. received: 25\. Halved the value. received: 12\. Halved the value. 
```

这个 monad 非常适合跟踪有用的信息。此外，它经常避免在你的函数和库代码中留下一些不需要的`var_dump`或`echo`方法。一旦调试完成，留下这些消息，它们可能对其他人有用，然后移除`runWriter`方法返回的`$log`值的使用。

显然，你也可以使用`Writer`monad 来跟踪任何类型的信息。一个很好的用法是通过`Writer`实例始终返回执行时间，将性能分析直接嵌入到你的函数中。

如果你需要存储多种类型的数据，`Writer` monad 不仅限于字符串值，任何 monoid 都可以。例如，你可以声明一个包含执行时间、堆栈跟踪和调试消息的特定 monoid，并将其与你的 Writer 一起使用。这样，你的每个函数都能向调用它们的人传递有用的信息。

我们可以说，始终具有这种信息会减慢程序的运行速度。这可能是正确的，但我想在大多数应用程序中，这种优化是不需要的。

# Reader monad

碰巧你有一堆函数，它们都应该接受相同的参数，或者给定值列表的一个子集。例如，你有一个配置文件，你的应用程序的各个部分需要访问其中存储的值。一个解决方案是有一种全局对象或单例来存储这些信息，但正如我们已经讨论过的，这会导致一些问题。在现代 PHP 框架中更常见的方法是使用一个叫做**依赖注入**（**DI**）的概念。Reader 单子允许你以纯函数的方式做到这一点。

## 动机

提供一种共享公共环境的方式，例如配置信息或类实例，跨多个函数进行。这个环境对于计算序列是只读的。然而，它可以被修改或扩展，用于当前步骤的任何子计算。

## 实施

`Reader`类执行函数评估是懒惰的，因为当函数绑定时环境的内容还不知道。这意味着所有函数都被包裹在单子内部的闭包中，当调用`runReader`方法时才运行。除此之外，在`php-functional`库中可用的实现非常直接。

## 例子

使用`Reader`单子与我们到目前为止所见到的有些不同。绑定的函数将接收计算中前一步的值，并且必须返回一个持有接收环境的函数的新 reader。如果你只想处理当前值，使用`map`函数会更容易，因为它不需要返回一个`Reader`实例。然而，你将不会收到上下文：

```php
<?php 
function hello() 
{ 
    return Reader::of(function($name) { 
        return "Hello $name!"; 
    }); 
} 

function ask($content) 
{ 
    return Reader::of(function($name) use($content) { 
        return $content. ($name == 'World' ? '' : ' How are you ?'); 
    }); 
} 

$r = hello() 
      ->bind('ask') 
      ->map('strtoupper'); 

echo $r->runReader('World'); 
// HELLO WORLD! echo $r->runReader('Gilles'); 
// HELLO GILLES! HOW ARE YOU ? 
```

这个不太有趣的例子只是提出了你可以做什么的基础知识。下一个例子将展示如何使用这个单子进行 DI。

### 注意

如果你使用过现代的 Web 框架，你可能已经知道什么是依赖注入，或者 DI。否则，这里是一个真正快速的解释，我可能会因此被烧死。DI 是一种模式，用于避免使用单例或全局可用的实例。相反，你声明你的依赖项作为函数或构造函数参数，一个**依赖注入容器**（**DIC**）负责为你提供它们。

通常，这涉及让 DIC 实例化所有对象，而不是使用`new`关键字，但方法因框架而异。

我们如何使用`Reader`单子来做到这一点？很简单。我们需要创建一个容器来保存所有的服务，然后我们将使用我们的 reader 来传递这些服务。

举例来说，假设我们有一个用于连接数据库的`EntityManager`，以及一个发送电子邮件的服务。另外，为了保持简单，我们不会进行任何封装，而是使用简单的函数而不是类：

```php
<?php 

class DIC 
{ 
    public $userEntityManager; 
    public $emailService; 
} 

function getUser(string $username) 
{ 
    return Reader::of(function(DIC $dic) use($username) { 
        return $dic->userEntityManager->getUser($username); 
    }); 
} 

function getUserEmail($username) 
{ 
    return getUser($username)->map(function($user) { 
        return $user->email; 

    }); 
} 

function sendEmail($title, $content, $email) 
{ 
    return Reader::of(function(DIC $dic) use($title, $content, $email) { 
        return $dic->emailService->send($title, $content, $email); 
    }); 
} 
```

现在我们想要编写一个在用户在我们的应用程序上注册后被调用的控制器。我们需要给他们发送一封电子邮件并显示某种确认。现在，让我们假设用户已经保存在数据库中，并且我们的理论框架提供了`POST`方法值作为参数的使用：

```php
<?php 

function controller(array $post) 
{ 
    return Reader::of(function(DIC $dic) use($post) { 
        getUserEmail($post['username']) 
            ->bind(f\curry('sendEmail', ['Welcome', '...'])) 
            ->runReader($dic); 

        return "<h1>Welcome !</h1>"; 
    }); 
} 
```

好的，我们已经准备好进行快速测试。我们将创建一些面向服务的类，以查看管道是否正常工作：

```php
<?php 

$dic = new DIC(); 
$dic->userEntityManager = new class() { 
    public function getUser() { 
      return new class() { 
          public $email = 'john.doe@email.com'; 
      }; 
    } 
}; 

$dic->emailService = new class() { 
    public function send($title, $content, $email) { 
        echo "Sending '$title' to $email"; 
    } 
}; 

$content = controller(['username' => 'john.doe'])->runReader($dic); 
// Sending 'Welcome' to john.doe@email.com 

echo $content; 
// <h1>Welcome !</h1> 
```

显然，我们还没有一个可用的框架，但我认为这很好地展示了`Reader`单子在 DI 方面提供的可能性。

关于需要执行的 IO 操作，以将新创建的用户存储到数据库中并发送邮件，我们将看到如何使用稍后将介绍的 IO 单子来实现。

# 状态单子

State 单子是读取器单子的一种泛化，因为每个步骤在调用下一步之前都可以修改当前状态。由于引用透明语言不能具有共享的全局状态，技巧是将状态封装在单子内部，并将其显式地传递给序列的每个部分。

## 动机

它提供了一个干净且易于使用的过程，可以在序列中的多个步骤之间传递共享状态。这显然可以手动完成，但这个过程容易出错，并且导致代码可读性较差。单子隐藏了复杂性，因此您可以简单地编写以状态作为输入并返回新状态的函数。

## 实现

`php-functional`库中提供的实现与我们刚刚讨论的`Reader`单子几乎相同，只有一个关键区别-每个绑定函数都可以更新状态。这导致了与绑定到单子的函数不同的函数-它们需要返回一个包含值作为第一个元素和新状态作为第二个元素的数组，而不是返回一个值。

## 示例

正如我们已经讨论过的，函数不可能返回当前时间或某种随机值。`state`单子可以通过提供一种干净的方式来传递`state`变量来帮助我们做到这一点，就像我们之前使用`Reader`环境一样：

```php
function randomInt() 
{ 
    return s\state(function($state) { 
        mt_srand($state); 
        return [mt_rand(), mt_rand()]; 
    }); 
} 

echo s\evalState(randomInt(), 12345); 
// 162946439 
```

`state`单子的另一个用途是实现缓存系统：

```php
<?php 

function getUser($id, $current = []) 
{ 
    return f\curryN(2, function($id, $current) { 
        return s\state(function($cache) use ($id, $current) { 
            if(! isset($cache[$id])) { 
                $cache[$id] = "user #$id"; 
            } 

            return [f\append($current, $cache[$id]), $cache]; 
        }); 
    })(...func_get_args()); 
} 

list($users, $cache) = s\runState( 
  getUser(1, []) 
    ->bind(getUser(2)) 
    ->bind(getUser(1)) 
    ->bind(getUser(3)), 
  [] 
); 

print_r($users); 
// Array ( 
//     [0] => user #1 
//     [1] => user #2 
//     [2] => user #1 
//     [3] => user #3 
// ) 

print_r($cache); 
// Array ( 
//     [1] => user #1 
//     [2] => user #2 
//     [3] => user #3 
// ) 
```

正如我们所看到的，用户列表中包含`user 1`两次，但缓存只有一次。这是一个非常基本的缓存机制，但它可能会派上用场。

`state`单子还有许多其他用途，但老实说，如果没有像 do 表示法这样的语法糖，我不太确定它是否适合 PHP 编程。如果您感兴趣，我相信您会在网上找到许多其他资源，但我们将在这里停止示例。

# IO 单子

输入和输出是副作用的精髓。当您从外部源获取函数输出时，没有办法保证纯度，因为这些输出会随着输入无关地发生变化。并且一旦您输出了某些东西，无论是屏幕、文件还是其他任何地方，您都改变了与函数输出无关的外部状态。

函数社区中的一些人认为，例如，日志输出或调试语句不一定应被视为副作用，因为通常它们对于运行应用程序的结果没有影响。最终用户并不在乎是否将某些内容写入日志文件，只要它能得到想要的结果并且操作可以随意重复。说实话，我对这个问题的看法还没有完全形成，而且老实说，我并不在乎，因为写入单子让我们以巧妙的方式处理日志记录和调试语句。

然而，有时您需要从外部获取信息，通常，如果您的应用程序值得做任何事情，您需要在某个地方显示或写入最终结果。

我们可以想象在开始任何计算之前获取所有值，并使用某种巧妙的数据结构传递它们。这对于一些较简单的应用程序可能有效，但是一旦您需要根据一些计算出的值执行数据库访问，现实开始显现，您会意识到这在长期内根本行不通。

IO 单子提出的技巧是按照我们刚刚提出的方式进行，但是相反。您首先描述程序所需的所有计算步骤。您将它们封装在 IO 单子的实例中，当一切都清晰地定义为引用透明的函数调用时，您启动最终执行所有所需 IO 操作的程序，并调用每个描述的步骤。

这样，您的应用程序只由纯函数组成，您可以轻松测试和理解。与输入和输出相关的所有操作都是在最后执行的，复杂性被隐藏在 IO 单子内部。为了强制执行这一点，IO 单子被称为单向单子，意味着无法从中获取任何值。您只有两个选择：

+   将计算或操作绑定到单子，以便稍后执行它们

+   运行这些计算以获得应用程序的最终结果

我想如果您从未见过像这样创建的应用程序，可能会感到非常困惑。这些例子将尝试给您第一印象，以及我们将在第十一章，“设计一个功能应用程序”中深入探讨这个主题。

## 动机

IO 单子通过将所有 IO 操作限制在单子内部，解决了输入和输出破坏引用透明度和函数纯度的问题。应用程序所需的所有计算步骤首先以功能方式描述。完成这一点后，我们接受最终步骤无法无副作用，并运行存储在单子内部的所有序列。

## 实施

`php-functional`库提供的实现非常简单，因为没有真正的微妙之处。只需要一个小技巧，即在调用`run`方法时进行计算，而不是在函数绑定时进行计算。

此外，该库还提供了`Widmogrod\Monad\IO`命名空间下的辅助函数，以帮助您使用单子。您可以轻松地从命令行读取用户输入，在屏幕上打印文本，并读取和写入文件和环境变量。

## 例子

我们将利用`mcompose`方法来组合多个`IO`操作：

```php
<?php 

use Widmogrod\Functional as f; 
use Widmogrod\Monad\IO; 
use Widmogrod\Monad\Identity; 

$readFromInput = f\mcompose(IO\putStrLn, IO\getLine, IO\putStrLn); 
$readFromInput(Monad\Identity::of('Enter something and press  <enter>'))->run(); 
// Enter something and press <enter> 
// Hi! // Hi! 
```

因此，我们首先创建一个使用`putStrLn`显示单子当前内容的函数，要求一些输入，并将结果显示回来。

如果要保持引用透明度，IO 单子需要包装整个应用程序的计算。这是因为您的输入需要通过它来检索，任何输出也必须通过单子完成。这意味着我们可以展示很多例子，而实际上并没有真正抓住其使用的真正本质。这就是为什么我们将在这里停下来，等到第十一章，“设计一个功能应用程序”，看看如何实现它。

# 总结

在本章中，我们已经看过多个单子及其实现。我希望这些例子清楚地说明了如何使用它们以及它们的好处是什么：

+   当计算可能返回空时，可以使用 Maybe 单子

+   Either 单子可用于计算可能出错的情况

+   List 单子可用于计算有多个可能结果的情况

+   当需要在返回值旁边传递一些辅助信息时，可以使用 Writer 单子

+   Reader 单子可用于在多个计算之间共享一个公共环境

+   State 单子是 Reader 单子的升级版本，其中环境可以在每次计算之间更新

+   IO 单子可用于以引用透明的方式执行 IO 操作

然而，还有其他多个计算可以使用单子简化。在编写代码时，我鼓励您退后一步，看看结构是否符合单子模式。如果是这样，您可能应该使用我们的`Monad`类来实现它，以从迄今为止学到的知识中受益。

另外，这些各种单子可以组合使用，实现复杂的转换和计算。我们将在第十章*PHP 框架和 FP*中讨论这个话题，其中我们将讨论单子变换器，以及第十一章*设计一个函数式应用*。

在书的这一部分，你可能会对一些函数式技术印象深刻，但我想我们到目前为止看到的大部分东西都有点尴尬，函数式编程可能看起来很繁琐。这种感觉对于两个主要原因来说是完全正常的。

首先，这种尴尬往往是由于某种缺失的抽象或待发现的技术所致。如果这是一本关于 Haskell 的书，你会学到所有这些内容，并且你会有一些其他书来查找它们。然而，这本书是关于 PHP 的；我们将在后面的章节中学习一些更多的概念，但之后，你将大部分时间都是靠自己，就像一个先驱一样。

我只能鼓励你在遇到这些情况时坚持下去，寻找代码中的模式和共性因素。一步一步，你将打造一个强大的工具箱，事情会变得更容易。

其次，这一切对你来说可能都是新的。转换编程范式真的很难，可能会让人感到沮丧。但不要害怕，随着时间、练习和经验的积累，你会变得更加自信，收获也会开始超过成本。学习曲线越陡峭，回报就越大。

在下一章中，我们将发现一些新的函数式概念和模式，这将使我们能够充分利用我们到目前为止学到的各种技术。
