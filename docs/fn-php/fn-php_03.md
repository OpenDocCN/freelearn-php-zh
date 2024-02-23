# 第三章。PHP 中的功能基础

在第一章中介绍了 PHP 中的函数，接着是第二章中的函数式编程的理论方面，我们最终将开始编写真正的代码。我们将从 PHP 中可用的函数开始，这些函数允许我们编写功能性代码。一旦基本技术得到很好的理解，我们将转向各种库，这些库将在整本书中帮助我们。

在本章中，我们将涵盖以下主题：

+   映射、折叠、减少和压缩

+   递归

+   为什么异常会破坏引用透明度

+   使用 Maybe 和 Either 类型更好地处理错误的方法

+   PHP 中可用的功能性库

# 一般建议

在前几章中，我们描述了功能应用程序必须具有的重要属性。然而，我们从未真正讨论过如何实现它。除了我们将在以后学习的各种技术之外，还有一些简单的建议可以立即帮助您。

## 使所有输入明确

我们在上一章中大量讨论了纯度和隐藏输入，或者副作用。现在，应该很清楚，函数的所有依赖关系都应该作为参数传递。然而，这个建议还要进一步。

避免将对象或复杂数据结构传递给您的函数。尽量限制输入到必要的内容。这样做将使您的函数范围更容易理解，并且将有助于确定函数的操作方式。它还具有以下好处：

+   调用将更容易

+   测试它将需要较少的数据存根

## 避免临时变量

正如您可能已经了解的那样，状态是邪恶的，特别是全局状态。然而，局部变量是一种局部状态。一旦您开始在代码中频繁使用它们，您就慢慢地打开了潘多拉的魔盒。这在 PHP 这样的语言中尤其如此，因为所有变量都是可变的。如果值在途中发生变化会发生什么？

每次声明一个变量，你都必须记住它的值，才能理解代码的其余部分是如何工作的。这大大增加了认知负担。此外，由于 PHP 是动态类型的，一个变量可以被完全不同的数据重复使用。

使用临时变量时，总会存在某种方式修改或重复使用的风险，导致难以调试的错误。

在几乎所有情况下，使用函数比使用临时变量更好。函数允许获得相同的好处：

+   通过命名中间结果来提高可读性

+   避免重复自己

+   缓存冗长操作的结果（这需要使用备忘录，我们将在第八章中讨论，*性能效率*）

调用函数的额外成本通常是微不足道的，不会打破平衡。此外，使用函数而不是临时变量意味着您可以在其他地方重用这些函数。它们还可以使未来的重构更容易，并且可以改善关注点的分离。

正如最佳实践所期望的那样，有时使用临时变量会更容易一些。例如，如果您需要在一个短函数中存储一个返回值，以便在之后立即使用，以便保持行长度舒适，请毫不犹豫地这样做。唯一严格禁止的是使用相同的临时变量来存储各种不同的信息。

## 更小的函数

我们已经提到函数就像积木一样。通常，您希望您的积木多才多艺且坚固。如果您编写只专注于做一件事情的小函数，那么这两个属性都会得到更好的强化。

如果您的函数做得太多，很难重用。我们将在下一章中讨论如何组合函数，以及如何利用所有小型实用函数来创建具有更大影响力的新函数。

此外，阅读较小的代码片段并对其进行推理更容易。相关的影响更容易理解，通常情况下边界情况更少，使函数更容易测试。

## 参数顺序很重要

选择函数参数的顺序似乎并不重要，但实际上它很重要。高阶函数是函数式编程的核心特性；这意味着你将会传递很多函数。

这些函数可以是匿名的，这种情况下，出于可读性的考虑，你可能希望避免将函数声明作为中间参数。在 PHP 中，可选参数也受到签名末尾的限制。正如我们将看到的，一些函数构造可以接受具有默认值的函数。

我们还将在第四章*组合函数*中进一步讨论这个话题。当你将多个函数链接在一起时，每个函数的第一个参数是前一个函数的返回值。这意味着在选择哪些参数先传递时，你需要特别小心。

# 映射函数

在 PHP 中，map 或`array_map`方法是一个高阶函数，它将给定的回调应用于集合的所有元素。`return`值是按顺序排列的集合。一个简单的例子是：

```php
<?php 

function square(int $x): int 
{ 
    return $x * $x; 
} 
$squared = array_map('square', [1, 2, 3, 4]); 
// $squared contains [1, 4, 9, 16] 
```

我们创建一个计算给定整数的平方的函数，然后使用`array_map`函数来计算给定数组的所有平方值。`array_map`函数的第一个参数可以是任何形式的 callable，第二个参数必须是一个*真实的数组*。你不能传递一个迭代器或一个 Traversable 的实例。

你也可以传递多个数组。你的回调将从每个数组中接收一个值：

```php
<?php 

$numbers = [1, 2, 3, 4]; 
$english = ['one', 'two', 'three', 'four']; 
$french = ['un', 'deux', 'trois', 'quatre']; 

function translate(int $n, string $e, string $f): string 
{ 
    return "$n is $e, or $f in French."; 
} 
print_r(array_map('translate', $numbers, $english, $french)); 
```

这段代码将显示：

```php
Array 
( 
    [0] => 1 is one, or un in French. [1] => 2 is two, or deux in French. [2] => 3 is three, or trois in French. [3] => 4 is four, or quatre in French. ) 
```

最长的数组将决定结果的长度。较短的数组将用 null 值扩展，以使它们的长度相匹配。

如果你将 null 作为函数传递，PHP 将合并这些数组：

```php
<?php 

print_r(array_map(null, [1, 2], ['one', 'two'], ['un', 'deux'])); 
```

结果是：

```php
Array 
( 
    [0] => Array 
        ( 
            [0] => 1 
            [1] => one 
            [2] => un 
        ) 
    [1] => Array 
        ( 
            [0] => 2 
            [1] => two 
            [2] => deux 
        ) 
) 
```

如果你只传递一个数组，键将被保留；但如果你传递多个数组，它们将丢失：

```php
<?php 
  function add(int $a, int $b = 10): int 
  { 
      return $a + $b; 
  } 

  print_r(array_map('add', ['one' => 1, 'two' => 2])); 
  print_r(array_map('add', [1, 2], [20, 30])); 
```

结果是：

```php
Array 
( 
    [one] => 11 
    [two] => 12 
) 
Array 
( 
    [0] => 21 
    [1] => 32 
) 
```

最后要注意的是，很遗憾，无法轻松访问每个项目的键。然而，你的回调可以是一个闭包，因此你可以使用来自你上下文的任何变量。利用这一点，你可以在数组的键上进行映射，并使用闭包来检索值：

```php
$data = ['one' => 1, 'two' => 2];

array_map(function to_string($key) use($data) {
    return (str) $data[$key];
}, 
array_keys($data);
```

# 过滤函数

在 PHP 中，filter 或`array_filter`方法是一个高阶函数，它基于布尔谓词仅保留集合的某些元素。`return`值是仅包含谓词函数返回 true 的元素的集合。一个简单的例子是：

```php
<?php

function odd(int $a): bool
{
    return $a % 2 === 1;
}

$filtered = array_filter([1, 2, 3, 4, 5, 6], 'odd');
/* $filtered contains [1, 3, 5] */
```

我们首先创建一个接受值并返回布尔值的函数。这个函数将是我们的谓词。在我们的例子中，我们检查一个整数是否是奇数。与`array_map`方法一样，谓词可以是任何`callable`，集合必须是一个数组。然而，请注意参数顺序是相反的；集合首先出现。

回调是可选的；如果你不提供一个，PHP 将过滤掉所有会被评估为 false 的元素，比如空字符串和数组：

```php
<?php

$filtered = array_filter(["one", "two", "", "three", ""]); 
/* $filtered contains ["one", "two", "three"] */

$filtered = array_filter([0, 1, null, 2, [], 3, 0.0]); 
/* $filtered contains [1, 2, 3] */
```

你也可以传递第三个参数，作为一个标志，确定你想要接收键还是值，或者两者都要：

```php
<?php

$data = [];
function key_only($key) { 
    // [...] 
}

$filtered = array_filter($data, 'key_only', ARRAY_FILTER_USE_KEY);

function both($value, $key) { 
    // [...] 
}

$filtered = array_filter($data, 'both', ARRAY_FILTER_USE_BOTH);
```

# 折叠或减少函数

折叠是指使用组合函数将集合减少为返回值的过程。根据语言的不同，这个操作可能有多个名称，如 fold、reduce、accumulate、aggregate 或 compress。与与数组相关的其他函数一样，在 PHP 中的版本是`array_reduce`函数。

你可能熟悉`array_sum`函数，它计算数组中所有值的总和。实际上，这是一种折叠操作，可以很容易地使用`array_reduce`函数来实现：

```php
<?php

function sum(int $carry, int $i): int
{
    return $carry + $i;
}

$summed = array_reduce([1, 2, 3, 4], 'sum', 0);
/* $summed contains 10 */
```

像`array_filter`方法一样，首先是集合；然后传递一个回调，最后是一个可选的初始值。在我们的情况下，我们被迫传递初始值 0，因为默认的 null 对于我们的 int 类型函数签名是无效的类型。

回调函数有两个参数。第一个是基于所有先前项目的当前减少值，有时称为**carry**或**accumulator**。第二个是当前正在处理的数组元素。在第一次迭代中，carry 等于初始值。

您不一定需要使用元素本身来生成值。例如，您可以使用 fold 实现`in_array`的简单替代：

```php
<?php

function in_array2(string $needle, array $haystack): bool
{
    $search = function(bool $contains, string $item) use ($needle):bool 
    {
        return $needle == $item ? true : $contains;
    };
    return array_reduce($haystack, $search, false);
}

var_dump(in_array2('two', ['one', 'two', 'three']));
// bool(true)
```

reduce 操作从初始值 false 开始，因为我们假设数组不包含我们要找的项目。这也使我们能够很好地处理数组为空的情况。

对于每个项目，如果项目是我们正在搜索的项目，我们返回 true，这将是新的传递值。如果不匹配，我们只需返回累加器的当前值，它将是`true`（如果我们之前找到了项目）或`false`（如果我们没有找到）。

我们的实现可能比官方的慢一点，因为无论如何，我们都必须在返回结果之前遍历整个数组，而不能在遇到搜索项目时立即退出函数。

然而，我们可以实现一个 max 函数的替代方案，性能应该是相当的，因为任何实现都必须遍历所有值：

```php
<?php

function max2(array $data): int
{
    return array_reduce($data, function(int $max, int $i) : int 
    {
        return $i > $max ? $i : $max;
    }, 0);
}

echo max2([5, 10, 23, 1, 0]);
// 23
```

这个想法和之前一样，只是使用数字而不是布尔值。我们从初始值`0`开始，我们的当前最大值。如果我们遇到更大的值，我们返回它，以便传递。否则，我们继续返回我们当前的累加器，已经包含到目前为止遇到的最大值。

由于 max PHP 函数适用于数组和数字，我们可以重用它来进行减少。然而，这将带来没有意义，因为原始函数已经可以直接在数组上操作：

```php
<?php

function max3(array $data): int
{
    return array_reduce($data, 'max', 0);
}
```

只是为了明确起见，我不建议在生产中使用这些。语言中已经有更好的函数。这些只是为了教育目的，以展示折叠的各种可能性。

我完全理解，这些简短的示例可能不比`foreach`循环或其他更命令式的方法更好，来实现这两个函数。但是它们有一些优点：

+   如果您使用 PHP 7 标量类型提示，每个项目的类型都会被强制执行，使您的软件更加健壮。您可以通过将字符串放入用于`max2`方法的数组来验证这一点。

+   您可以对传递给`array_reduce`方法的函数进行单元测试，或者对`array_map`和`array_filter`函数进行测试，以确保其正确性。

+   如果您有这样的架构，您可以在多个线程或网络节点之间分发大数组的减少。这在`foreach`循环中将会更加困难。

+   正如`max3`函数所示，这种方法允许您重用现有方法，而不是编写自定义循环来操作数据。

## 使用 fold 的 map 和 filter 函数

目前，我们的`fold`只返回简单的标量值。但没有什么能阻止我们构建更复杂的数据结构。例如，我们可以使用`fold`来实现 map 和 filter 函数：

```php
<?php 

function map(array $data, callable $cb): array 
{ 
    return array_reduce($data, function(array $acc, $i) use ($cb) { 
        $acc[] = $cb($i); 
        return $acc; 
    }, []);     
} 

function filter(array $data, callable $predicate): array 
{ 
  return array_reduce($data, function(array $acc, $i)  use($predicate) { 
      if($predicate($i)) { 
          $acc[] = $i; 
      } 
      return $acc; 
  }, []); 
} 
```

再次强调，这些大部分是为了演示使用折叠返回数组是可能的。如果您不需要操作更复杂的集合，原生函数就足够了。

作为读者的练习，尝试实现`map_filter`或`filter_map`函数，如果您愿意，还可以尝试编写 head 和 tail 方法，它们分别返回数组的第一个和最后一个元素，并且通常在函数式语言中找到。

正如您所看到的，折叠是非常强大的，其背后的思想对许多函数式技术至关重要。这就是为什么我更喜欢谈论折叠而不是缩减，我觉得这有点简化，双关语。

在继续之前，请确保您了解折叠的工作原理，因为这将使其他所有事情都变得更容易。

## 左折叠和右折叠

函数式语言通常实现了两个版本的折叠，`foldl`和`foldr`。区别在于第一个从左边折叠，第二个从右边折叠。

例如，如果你有数组`[1, 2, 3, 4, 5]`，你想计算它的总和，你可以有`(((1 + 2) + 3) + 4) + 5`或`(((5 + 4) + 3) + 2) + 1`。如果有一个初始值，它将始终是计算中使用的第一个值。

如果您应用于值的操作是可交换的，左右两个变体都将产生相同的结果。可交换操作的概念来自数学，在第七章 *Functional Techniques and Topics*中有解释。

对于允许无限列表的语言，比如 Haskell，取决于列表是如何生成的，两种折叠中的一种可能能够计算一个值并停止。此外，如果语言实现了尾递归消除，一个正确的折叠起始点可能会避免堆栈溢出并允许操作完成。

由于 PHP 不执行无限列表或尾递归消除，我认为没有理由去区分。如果您感兴趣，`array_reduce`函数从左边折叠，实现一个从右边折叠的函数不应该太复杂。

## MapReduce 模型

你可能已经听说过**MapReduce**编程模型的名字。起初，它指的是 Google 开发的专有技术，但如今在各种语言中有多种实现。

尽管 MapReduce 背后的思想受到我们刚讨论的 map 和 reduce 函数的启发，但这个概念更广泛。它描述了使用并行和分布式算法在集群上处理大型数据集的整个模型。

本书中学到的每一种技术都可以帮助您实现 MapReduce 来分析数据。然而，这个话题超出了范围，所以如果您想了解更多，可以从维基百科页面开始访问[`en.wikipedia.org/wiki/MapReduce`](https://en.wikipedia.org/wiki/MapReduce)。

# 卷积或 zip

卷积，或更常见的 zip 是将所有给定数组的每个第 n 个元素组合在一起的过程。事实上，这正是我们之前通过向`array_map`函数传递 null 值所做的：

```php
<?php 

print_r(array_map(null, [1, 2], ['one', 'two'], ['un', 'deux'])); 
```

输出为：

```php
Array 
( 
    [0] => Array 
        ( 
            [0] => 1 
            [1] => one 
            [2] => un 
        ) 
    [1] => Array 
        ( 
            [0] => 2 
            [1] => two 
            [2] => deux 
        ) 
) 
```

重要的是要注意，如果数组的长度不同，PHP 将使用 null 作为填充值：

```php
<?php 

$numerals = [1, 2, 3, 4]; 
$english = ['one', 'two']; 
$french = ['un', 'deux', 'trois']; 

print_r(array_map(null, $numerals, $english, $french)); 
Array 
( 
    [0] => Array 
        ( 
            [0] => 1 
            [1] => one 
            [2] => un 
        ) 
    [1] => Array 
        ( 
            [0] => 2 
            [1] => two 
            [2] => deux 
        ) 
    [2] => Array 
        ( 
            [0] => 3 
            [1] => 
            [2] => trois 
        ) 
    [3] => Array 
        ( 
            [0] => 4 
            [1] => 
            [2] => 
        ) 
) 
```

请注意，在大多数编程语言中，包括 Haskell、Scala 和 Python，在 zip 操作中，将停止在最短的数组处，而不会填充任何值。您可以尝试在 PHP 中实现类似的功能，例如使用`array_slice`函数将所有数组减少到相同的大小，然后调用`array_merge`函数。

我们还可以通过从数组中创建多个数组来执行反向操作。这个过程有时被称为**unzip**。这里是一个天真的实现，缺少了很多检查，使其足够健壮用于生产：

```php
<?php 

function unzip(array $data): array 
{ 
    $return = []; 

    $data = array_values($data); 
    $size = count($data[0]); 

    foreach($data as $child) { 
        $child = array_values($child); 
        for($i = 0; $i < $size; ++$i) { 
            if(isset($child[$i]) && $child[$i] !== null) { 
                $return[$i][] = $child[$i]; 
            } 
        } 
    } 

    return $return; 
} 
```

你可以像这样使用它：

```php
$zipped = array_map(null, $numerals, $english, $french); 

list($numerals2, $english2, $french2) = unzip($zipped); 

var_dump($numerals == $numerals2); 
// bool(true) 
var_dump($english == $english2); 
// bool(true) 
var_dump($french == $french2); 
// bool(true) 
```

# 递归

在学术意义上，递归是将问题分解为相同问题的较小实例的想法。例如，如果您需要递归扫描一个目录，您首先扫描起始目录，然后扫描其子目录和子目录的子目录。大多数编程语言通过允许函数调用自身来支持递归。这个想法通常被描述为递归。

让我们看看如何使用递归扫描目录：

```php
<?php 

function searchDirectory($dir, $accumulator = []) { 
    foreach (scandir($dir) as $path) { 
        // Ignore hidden files, current directory and parent directory 
        if(strpos($path, '.') === 0) { 
            continue; 
        } 

        $fullPath = $dir.DIRECTORY_SEPARATOR.$path; 

        if(is_dir($fullPath)) { 
            $accumulator = searchDirectory($path, $accumulator); 
        } else { 
            $accumulator[] = $fullPath; 
        } 
    } 
    return $accumulator; 
} 
```

我们首先使用`scandir`函数获取所有文件和目录。然后，如果遇到子目录，我们再次调用该函数。否则，我们只需将文件添加到累加器中。这个函数是递归的，因为它调用自身。

您可以使用控制结构来编写这个函数，但是由于您无法预先知道文件夹层次结构的深度，代码可能会变得更加混乱和难以理解。

有些书籍和教程使用斐波那契数列，或计算阶乘作为递归示例，但公平地说，这些示例相当差，因为最好使用传统的`for`循环来实现第二个示例，并且对于第一个示例，提前计算项更好。

相反，让我们思考一个更有趣的挑战，*Hanoi Towers*。对于不了解这个游戏的人来说，传统版本的游戏包括三根杆，上面堆叠着不同大小的圆盘，最小的在顶部。在游戏开始时，所有圆盘都在最左边的杆上，目标是将它们移到最右边的杆上。游戏遵循以下规则：

+   一次只能移动一个圆盘

+   只能移动杆的顶部圆盘

+   不能将一个圆盘放在较小的圆盘上方

这个游戏的设置如下：

![递归](img/image_03_001.jpg)

如果我们想解决这个游戏，较大的圆盘必须首先放在最后一个杆上。为了做到这一点，我们需要先将所有其他圆盘移到中间的杆上。沿着这种推理方式，我们可以得出我们必须实现的三个大步骤：

1.  将所有的圆盘移动到中间，除了最大的一个。

1.  将大圆盘移到右边。

1.  将所有圆盘移动到最大的圆盘上方。

*步骤 1*和*3*是初始问题的较小版本。这些步骤中的每一个又可以被缩小为更小的版本，直到我们只有一个圆盘需要移动-递归函数的完美情况。让我们尝试实现这一点。

为了避免在我们的函数中使用与杆和圆盘相关的变量，我们将假设计算机会向某人发出移动的命令。在我们的代码中，我们还将假设最大的圆盘是编号 1，较小的圆盘编号较大：

```php
<?php 

function hanoi(int $disc, string $source, string $destination,  string $via) 
{ 
    if ($disc === 1) { 
        echo("Move a disc from the $source rod to the $destination  rod\n"); 
    } else { 
        // step 1 : move all discs but the first to the "via" rod         hanoi($disc - 1, $source, $via, $destination); 
        // step 2 : move the last disc to the destination 
        hanoi(1, $source, $destination, $via); 
        // step 3 : move the discs from the "via" rod to the  destination 
        hanoi($disc - 1, $via, $destination, $source); 
    } 
} 
```

使用`hanoi(3, 'left', 'right', 'middle')`输入进行三个圆盘的移动，我们得到以下输出：

```php
Move a disc from the left rod to the right rod 
Move a disc from the left rod to the middle rod 
Move a disc from the right rod to the middle rod 
Move a disc from the left rod to the right rod 
Move a disc from the middle rod to the left rod 
Move a disc from the middle rod to the right rod 
Move a disc from the left rod to the right rod 
```

想要以递归的方式思考而不是使用更传统的循环需要一段时间，显然递归并不是解决您尝试解决的所有问题的银弹。

一些函数式语言根本没有循环结构，强制您使用递归。PHP 不是这种情况，所以让我们使用正确的工具来解决问题。如果您能将问题视为较小类似问题的组合，通常使用递归会很容易。例如，尝试找到*Towers of Hanoi*的迭代解决方案需要仔细思考。或者您可以尝试仅使用循环来重写目录扫描函数，以说服自己。

递归有用的其他领域包括：

+   生成具有多个级别的菜单的数据结构

+   遍历 XML 文档

+   渲染一系列可能包含子组件的 CMS 组件

一个很好的经验法则是，当您的数据具有树状结构，具有根节点和子节点时，尝试使用递归。

尽管阅读起来通常更容易，但一旦您掌握了它，递归就会带来内存成本。在大多数应用程序中，您不应遇到任何困难，但我们将在第十章中进一步讨论这个话题，*PHP 框架和 FP*，并提出一些避免这些问题的方法。

## 递归和循环

一些函数式语言，如 Haskell，没有任何循环结构。这意味着迭代数据结构的唯一方法是使用递归。虽然在函数世界中不鼓励使用 for 循环，因为当您可以修改循环索引时会出现所有问题，但使用`foreach`循环等并没有真正的危险。

为了完整起见，以下是一些替换循环为递归调用的方法，如果您想尝试或需要理解用另一种没有循环结构的语言编写的代码。

替换`while`循环：

```php
<?php 

function while_iterative() 
{ 
    $result = 1; 
    while($result < 50) { 
        $result = $result * 2; 
    } 
    return $result; 
} 

function while_recursive($result = 1, $continue = true) 
{ 
    if($continue === false) { 
        return $result; 
    } 
    return while_recursive($result * 2, $result < 50); 
} 
```

或者`for`循环：

```php
<?php 

function for_iterative() 
{ 
    $result = 5; 

    for($i = 1; $i < 10; ++$i) { 
        $result = $result * $i; 
    } 

    return $result; 
} 

function for_recursive($result = 5, $i = 1) 
{ 
    if($i >= 10) { 
        return $result; 
    } 

    return for_recursive($result * $i, $i + 1); 
} 
```

如您所见，诀窍在于使用函数参数将循环的当前状态传递给下一个递归。在 while 循环的情况下，您传递条件的结果，当您模拟 for 循环时，您传递循环计数器。显然，计算的当前状态也必须始终传递。

通常，递归本身是在辅助函数中完成的，以避免在签名中使用可选参数来执行循环。为了保持全局命名空间的清洁，这个辅助函数在原始函数内声明。以下是一个示例：

```php
<?php 

function for_with_helper() 
{ 
    $helper = function($result = 5, $i = 1) use(&$helper) { 
        if($i >= 10) { 
            return $result; 
        } 

        return $helper($result * $i, $i + 1); 
    }; 

    return $helper(); 
} 
```

请注意，您需要使用`use`关键字通过引用传递包含函数的变量。这是由于我们已经讨论过的一个事实。传递给闭包的变量在声明时绑定，但当函数声明时，赋值尚未发生，变量为空。但是，如果我们通过引用传递变量，它将在赋值完成后更新，我们将能够在匿名函数内部使用它作为回调。

# 异常

错误管理是编写软件时面临的最棘手的问题之一。很难决定哪段代码应该处理错误。在低级函数中执行，您可能无法访问显示错误消息的设施或足够的上下文来决定最佳操作。在更高层次上执行可能会在数据中造成混乱或使应用程序陷入无法恢复的状态。

在面向对象编程代码库中管理错误的常规方法是使用异常。您在库或实用程序代码中抛出异常，并在准备好按照您的意愿处理它时捕获它。

抛出异常和捕获是否可以被视为副作用或副原因，甚至在学术界也是一个有争议的问题。有各种观点。我不想用修辞论证来使您感到厌烦，所以让我们坚持一些几乎每个人都同意的观点：

+   由任何**外部来源**（数据库访问，文件系统错误，不可用的外部资源，无效的用户输入等）引发的异常本质上是不纯的，因为访问这些来源已经是一个副原因。

+   由于**逻辑错误**（索引超出范围，无效类型或数据等）引发的异常通常被认为是纯的，因为它可以被视为函数的有效`return`值。但是，异常必须清楚地记录为可能的结果。

+   捕获异常会破坏引用透明性，因此任何带有 catch 块的函数都会变得不纯。

前两个语句应该很容易理解，但第三个呢？让我们从一小段代码开始演示：

```php
<?php 
function throw_exception() 
{ 
    throw new Exception('Message'); 
} 

function some_function($x) 
{ 
    $y = throw_exception(); 
    try { 
        $z = $x + $y; 
    } catch(Exception $e) { 
        $z = 42; 
    } 

    return $z; 
} 

echo some_function(42); 
// PHP Warning: Uncaught Exception: Message 
```

很容易看出，我们对`some_function`函数的调用将导致未捕获的异常，因为对`throw_exception`函数的调用位于`try ... catch`块之外。现在，如果我们应用引用透明性的原则，我们应该能够用其值替换加法中的`$y`参数。让我们试试看：

```php
<?php 

try { 
    $z = $x + throw_exception(); 
} catch(Exception $e) { 
    $z = 42; 
} 
```

现在`$z`参数的值是多少，我们的函数将返回什么？与以前不同，我们现在将返回值为`42`，显然改变了调用我们的函数的结果。通过简单地尝试应用等式推理，我们刚刚证明了捕获异常可能会破坏引用透明性。

如果你无法捕获异常，那么异常有什么用呢？不多；这就是为什么在整本书中我们都会避免使用它们。然而，你可以将它们视为副作用，然后应用我们将在第六章中看到的技术，*真实的 Monad*，来管理它们。例如，Haskell 允许抛出异常，只要它们使用 IO Monad 捕获。

另一个问题是认知负担。一旦你使用它们，你就无法确定它们何时会被捕获；它们甚至可能直接显示给最终用户。这破坏了对代码片段本身进行推理的能力，因为现在你必须考虑更高层发生的事情。

这通常是你听到诸如*仅将异常用于错误，而不是流程控制*之类的建议的原因。这样，你至少可以确定你的异常将被用来显示某种错误，而不是想知道你将应用程序置于哪种状态。

## PHP 7 和异常

即使我们大多数时候在负面情况下讨论异常，让我趁此机会介绍一下在新的 PHP 版本中关于这个主题所做的改进。

以前，某些类型的错误会生成致命错误或错误，这些错误会停止脚本的执行并显示错误消息。你可以使用`set_error_handler`异常来定义一个自定义处理程序来处理非致命错误，并最终继续执行。

PHP 7.0 引入了一个`Throwable`接口，它是异常的新父类。`Throwable`类还有一个新的子类叫做`Error`类，你可以用它来捕获以前无法处理的大多数错误。仍然有一些错误，比如解析错误，显然是无法捕获的，因为这意味着你的整个 PHP 文件在某种程度上是无效的。

让我们用一段代码来演示这一点，试图在对象上调用一个不存在的方法：

```php
<?php 
class A {} 

$a = new A(); 

$a->invalid_method(); 

// PHP Warning: Uncaught Error: Call to undefined method  A::invalid_method() 
```

如果你使用的是 PHP 5.6 或更低版本，消息将会说类似于：

```php
Fatal error: Call to undefined method A::invalid_method()
```

然而，使用 PHP 7.0，消息将是（重点是我的）：

```php
Fatal error: **Uncaught Error**: Call to undefined method A::invalid_method()
```

区别在于 PHP 通知你这是一个未捕获的错误。这意味着你现在可以使用通常的`try ... catch`语法来捕获它。你可以直接捕获`Error`类，或者如果你想更广泛地捕获任何可能的异常，你可以使用`Throwable`接口。然而，我不建议这样做，因为你将失去关于你究竟有哪种错误的信息：

```php
<?php class B {} 

$a = new B(); 

try { 
    $a->invalid_method(); 
} catch(Error $e) { 
    echo "An error occured : ".$e->getMessage(); 
} 
// An error occured : Call to undefined method B::invalid_method() 
```

对我们来说也很有趣的是，`TypeError`参数是`Error`类的子类，当使用错误类型的参数调用函数或返回类型错误时会引发它：

```php
<?php 
function add(int $a, int $b): int 
{ 
    return $a + $b; 
} 

try { 
    add(10, 'foo'); 
} catch(TypeError $e) { 
    echo "An error occured : ".$e->getMessage(); 
} 
// An error occured : Argument 2 passed to add() must be of the type integer, string given 
```

对于那些想知道为什么在新的`Error`类旁边创建了一个新的接口，主要是出于两个原因：

+   清楚地将`Exception`接口与以前的内部引擎错误分开

+   为了避免破坏现有代码捕获`Exception`接口，让开发人员选择是否也要开始捕获错误

# 异常的替代方案

正如我们刚才看到的，如果我们想保持代码的纯净，我们就不能使用异常。那么我们有哪些选择来确保我们可以向我们函数的调用者表示错误呢？我们希望我们的解决方案具有以下特点：

+   强制错误管理，以便没有错误会冒泡到最终用户

+   避免样板或复杂的代码结构

+   在我们函数的签名中宣传

+   避免任何将错误误认为是正确结果的风险

在本章的下一节中，我们将介绍一个具有所有这些好处的解决方案，让我们先看看命令式语言中是如何进行错误管理的。

为了测试各种方式，我们将尝试实现我们之前已经使用过的`max`函数：

```php
<?php 
function max2(array $data): int 
{ 
    return array_reduce($data, function(int $max, int $i) : int { 
        return $i > $max ? $i : $max; 
    }, 0); 
} 
```

因为我们选择了初始值 0，如果我们用一个空数组调用函数，我们将得到结果 0。0 真的是一个空数组的最大值吗？如果我们调用与 PHP 捆绑的版本，`max([])`方法会发生什么？

```php
**Warning: max(): Array must contain at least one element**

```

此外，还返回了 false 值。我们的版本使用值 0 作为默认值，我们可以将 false 视为错误代码。PHP 版本还会显示警告。

既然我们有一个可以改进的函数，让我们尝试一下我们可以使用的各种选项。我们将从最差的选项到最好的选项。

## 记录/显示错误消息

正如我们刚才看到的，PHP 可以显示警告消息。我们也可以选择通知或错误级别的消息。这可能是您可以做的最糟糕的事情，因为调用您的函数的人无法知道发生了错误。消息只会在日志中或在应用程序运行时显示在屏幕上。

此外，在某些情况下，错误是可以恢复的。由于您不知道发生了什么，因此在这种情况下无法做到这一点。

更糟糕的是，PHP 允许您配置显示哪个错误级别。在大多数情况下，通知只是隐藏的，所以没有人会看到应用程序中发生了错误。

公平地说，PHP 有一种方法可以在运行时捕获这些警告和通知，即使用`set_error_handler`参数声明自定义错误处理程序。但是，为了正确管理错误，您必须找到一种方法在处理程序内部确定生成错误的函数，并相应地采取行动。

如果您有多个函数使用这些类型的消息来表示错误，您很快要么会有一个非常大的错误处理程序，要么会有很多较小的错误处理程序，这使整个过程容易出错且非常繁琐。

## 错误代码

错误代码是 C 语言的遗产，它没有任何异常的概念。这个想法是一个函数总是返回一个代码来表示计算的状态，并找到其他一些方法来传递返回值。通常，代码 0 表示一切顺利，其他任何代码都是错误。

在涉及数字错误代码时，PHP 没有使用它们作为返回值的函数，据我所知。然而，该语言有很多函数在发生错误时返回`false`值，而不是预期的值。只有一个潜在值来表示失败可能会导致传递有关发生了什么的信息的困难。例如，`move_uploaded_file`的文档说明：

> *成功返回 TRUE。*
> 
> *如果文件名不是有效的上传文件，则不会发生任何操作，move_uploaded_file()将返回 False。*
> 
> *如果文件名是有效的上传文件，但由于某种原因无法移动，则不会发生任何操作，move_uploaded_file()将返回 False。此外，将发出警告。*

这意味着当您发生错误时会收到通知，但是除非阅读错误消息，否则您无法知道它属于哪个错误类别。即使这样，您也会缺少重要信息，例如为什么上传的文件无效。

如果我们想更好地模仿 PHP 的`max`函数，我们可以这样做：

```php
<?php 
function max3(array $data) 
{ 
    if(empty($data)) { 
        trigger_error('max3(): Array must contain at least one  element', E_USER_WARNING); 
        return false; 
    } 

    return array_reduce($data, function(int $max, int $i) : int { 
        return $i > $max ? $i : $max; 
    }, 0); 
} 
```

由于现在我们的函数需要在发生错误时返回 false 值，我们不得不删除返回值的类型提示，从而使我们的签名不太自我说明。

其他函数，通常是包装外部库的函数，也会在发生错误时返回`false`值，但是在形式上具有`X_errno`和`X_error`的伴随函数，它们返回有关上次执行的函数的错误的更多信息。一些示例包括`curl_exec`、`curl_errno`和`curl_error`函数。

这些辅助程序允许更精细的错误处理，但代价是您必须考虑它们。错误管理不是强制的。为了进一步证明我的观点，让我们注意一下官方文档中`curl_exec`函数的示例甚至没有设置检查返回值的最佳实践：

```php
<?php 

/* create a new cURL resource */ 
$ch = curl_init(); 

/* set URL and other appropriate options */ 
curl_setopt($ch, CURLOPT_URL, "http://www.example.com/"); 
curl_setopt($ch, CURLOPT_HEADER, 0); 

/* grab URL and pass it to the browser */ 
curl_exec($ch); 

/* close cURL resource, and free up system resources */ 
curl_close($ch); 
```

在像 PHP 这样执行松散类型转换的语言中，将`false`值用作失败的标记也会产生另一个后果。正如前面的文档中所述，如果您不执行严格的相等比较，您可能会将一个作为 false 的有效返回值误认为是错误：

> *警告：此函数可能返回布尔值 FALSE，但也可能返回一个非布尔值，该值会被视为假。请阅读布尔值部分以获取更多信息。使用===运算符来测试此函数的返回值。*

PHP 仅在出现错误时使用 false 错误代码，但不像 C 语言通常情况下会返回`true`或`0`。您不必找到一种方法将返回值传递给用户。

但是，如果您想要使用数字错误代码来实现自己的函数，以便有可能对错误进行分类，您必须找到一种方法来同时返回代码和值。通常，您可以使用以下两种选项之一：

+   使用通过引用传递的参数来保存结果；例如，`preg_match`参数就是这样做的，即使出于不同的原因。只要参数明确标识为返回值，这并不严格违背函数的纯度。

+   返回一个数组或其他可以容纳两个或更多值的数据结构。这个想法是我们将在下一节中作为我们的函数解决方案的开端。

## 默认值/空值

在认知负担方面，与错误代码相比，默认值要好一点。如果您的函数只有一组可能导致错误的输入，或者如果错误原因不重要，您可以考虑返回一个默认值，而不是通过错误代码指定错误原因。

然而，这将引发新的问题。确定一个好的默认值并不总是容易的，在某些情况下，您的默认值也将是一个有效值，这将使确定是否存在错误变得不可能。例如，如果在调用我们的`max2`函数时得到 0 作为结果，您无法知道数组是空的还是只包含值为 0 和负数。

默认值也可能取决于上下文，这种情况下，您将不得不向函数添加一个参数，以便在调用时也可以指定默认值。除了使函数签名变得更大之外，这也会破坏我们稍后将学习的一些性能优化，并且，尽管完全纯净和引用透明，但会增加认知负担。

让我们向我们的`max`函数添加一个默认值参数：

```php
<?php 

function max4(array $data, int $default = 0): int 
{ 
    return empty($data) ? $default : 
      array_reduce($data, function(int $max, int $i) : int 
      { 
          return $i > $max ? $i : $max; 
      }, 0); 
} 
```

由于我们强制默认值的类型，我们能够恢复返回值的类型提示。如果您想将任何东西作为默认值传递，您还必须删除类型提示。

为了避免讨论的一些问题，有时会使用 null 值作为默认返回值。尽管 null 并不是一个真正的值，但在某些情况下，它并不属于*错误代码*类别，因为在某些情况下它是一个完全有效的值。比如说，如果你在一个集合中搜索一个项目，如果什么都没找到，你会返回什么？

然而，使用 null 值作为可能的返回值有两个问题：

+   您不能将返回类型提示为 null，因为 null 不会被视为正确类型。此外，如果您计划将该值用作参数，它也不能被类型提示，或者必须是带有 null 值作为默认值的可选参数。这将迫使您要么删除类型提示，要么将参数设为可选的。

+   如果您的函数通常返回对象，您将不得不检查 null 值，否则您将面临托尼·霍尔所说的*十亿美元错误*，即空指针引用。或者，如 PHP 中所述，*在 null 上调用成员函数 XXX()*。

顺便说一句，Tony Hoare 是在 1965 年引入空值的人，因为它很容易实现。后来，他非常后悔这个决定，并决定这是他的十亿美元错误。如果你想了解更多原因，我邀请你观看他在[`www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare`](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)上的演讲。

## 错误处理程序

最后一种方法在 JavaScript 世界中被广泛使用，因为回调函数随处可见。这个想法是每次调用函数时传递一个错误回调。如果允许调用者传递多个回调，每种错误都可以有一个回调，那么它甚至可以更加强大。

尽管它缓解了默认值的一些问题，比如可能将有效值与默认值混淆，但你仍然需要根据上下文传递不同的回调，使得这种解决方案只能略微好一些。

这种方法对我们的函数会是什么样子呢？考虑以下实现：

```php
<?php 

function max5(array $data, callable $onError): int 
{ 
    return empty($data) ? $onError() : 
      array_reduce($data, function(int $max, int $i) : int { 
          return $i > $max ? $i : $max; 
      }, 0); 
} 

max5([], function(): int { 
    // You are free to do anything you want here. // Not really useful in such a simple case but 
    // when creating complex objects it can prove invaluable. return 42; 
}); 
```

同样，我们保留了返回类型提示，因为我们与调用者的契约是返回一个整数值。正如评论中所述，在这种特殊情况下，参数的默认值可能就足够了，但在更复杂的情况下，这种方法提供了更多的功能。

我们还可以想象将初始参数传递给回调，同时传递有关失败的信息，以便错误处理程序可以相应地采取行动。在某种程度上，这种方法有点像我们之前看到的所有东西的组合，因为它允许你：

+   指定你选择的默认返回值

+   显示或记录任何你想要的错误消息

+   如果你愿意，可以返回一个更复杂的数据结构和错误代码

# Option/Maybe 和 Either 类型

如前所述，我们的解决方案是使用一个包含所需值或在出现错误时包含其他内容的返回类型。这种数据结构称为**联合类型**。联合可以包含不同类型的值，但一次只能包含一个。

让我们从本章中将要看到的两种联合类型中最简单的开始。一如既往，命名在计算机科学中是一件困难的事情，人们提出了不同的名称来指代基本上相同的结构：

+   Haskell 称之为 Maybe 类型，**Idris**也是如此

+   Scala 称之为 Option 类型，**OCaml**、**Rust**和**ML**也是如此

+   自 Java 8 以来，Java 就有了 Optional 类型，Swift 和下一个 C++规范也是如此

就个人而言，我更喜欢称之为 Maybe，因为我认为选项是另一回事。因此，本书的其余部分将使用这个术语，除非特定的库有一个名为**Option**的类型。

Maybe 类型在某种意义上是特殊的，它可以保存特定类型的值，也可以是*nothing*的等价物，或者如果你愿意的话，是空值。在 Haskell 中，这两种可能的值被称为`Just`和`Nothing`。在 Scala 中，它是`Some`和`None`，因为`Nothing`已经被用来指定值 null 的类型等价物。

只实现了 Maybe 或 Option 类型的库存在于 PHP 中，本章后面介绍的一些库也带有这些类型。但为了正确理解它们的工作原理和功能，我们将实现自己的类型。

让我们首先重申我们的目标：

+   强制错误管理，以便没有错误会冒泡到最终用户

+   避免样板代码或复杂的代码结构

+   在我们函数的签名中进行广告

+   避免任何错误被误认为是正确的结果

如果您使用我们将在接下来创建的类型对函数返回值进行类型提示，那么您已经照顾到了我们的第三个目标。`Just`和`Nothing`值的存在确保您不会将有效结果误认为错误。为了确保我们不会在某个地方得到错误的值，我们必须确保在没有指定默认值的情况下，不能从我们的新类型中获取值。关于我们的第二个目标，我们将看到我们是否可以写出一些好东西：

```php
<?php 

abstract class Maybe 
{ 
    public static function just($value): Just 
    { 
        return new Just($value); 
    } 

    public static function nothing(): Nothing 
    { 
        return Nothing::get(); 
    } 

    abstract public function isJust(): bool; 

    abstract public function isNothing(): bool; 

    abstract public function getOrElse($default); 
} 
```

我们的类有两个静态辅助方法，用于创建我们即将到来的子类的两个实例，代表我们的两种可能状态。`Nothing`值将作为单例实现，出于性能原因；因为它永远不会持有任何值，这样做是安全的。

我们类中最重要的部分是一个抽象的`getOrElse`函数，它将强制任何想要获取值的人也传递一个默认值，如果我们没有值则返回该默认值。这样，我们可以强制在错误的情况下返回一个有效的值。显然，您可以将 null 值作为默认值传递，因为 PHP 没有强制执行其他机制，但这就像是在自己的脚上开枪：

```php
<?php 
final class Just extends Maybe 
{ 
    private $value; 

    public function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public function isJust(): bool 
    { 
        return true; 
    } 

    public function isNothing(): bool 
    { 
        return false; 
    } 

    public function getOrElse($default) 
    { 
        return $this->value; 
    } 
} 
```

我们的`Just`类非常简单；一个构造函数和一个 getter：

```php
<?php 
final class Nothing extends Maybe 
{ 
    private static $instance = null; 
    public static function get() 
    { 
        if(is_null(self::$instance)) { 
            self::$instance = new static(); 
        } 

        return self::$instance; 
    } 

    public function isJust(): bool 
    { 
        return false; 
    } 

    public function isNothing(): bool 
    { 
        return true; 
    } 

    public function getOrElse($default) 
    { 
        return $default; 
    } 
} 
```

如果您不考虑成为单例的部分，`Nothing`类甚至更简单，因为`getOrElse`函数将始终返回默认值。对于那些好奇的人，保持构造函数公开是一个有意为之的选择。如果有人想直接创建`Nothing`实例，这绝对没有任何后果，那又何必费心呢？

让我们测试一下我们的新的`Maybe`类型：

```php
<?php 

$hello = Maybe::just("Hello World !"); 
$nothing = Maybe::nothing(); 

echo $hello->getOrElse("Nothing to see..."); 
// Hello World ! var_dump($hello->isJust()); 
// bool(true) 
var_dump($hello->isNothing()); 
// bool(false) 

echo $nothing->getOrElse("Nothing to see..."); 
// Nothing to see... var_dump($nothing->isJust()); 
// bool(false) 
var_dump($nothing->isNothing()); 
// bool(true) 
```

一切似乎都运行得很顺利。尽管需要样板代码，但可以改进。在这一点上，每当您想要实例化一个新的`Maybe`类型时，您需要检查您拥有的值，并在`Some`和`Nothing`值之间进行选择。

还可能会出现这样的情况，您需要在将值传递给下一个步骤之前对其应用一些函数，但在这一点上不知道最佳的默认值是什么。由于在创建新的`Maybe`类型之前，使用一些临时默认值获取值会很麻烦，让我们也尝试解决这个方面：

```php
<?php 

abstract class Maybe 
{ 
    // [...] 

    public static function fromValue($value, $nullValue = null) 
    { 
        return $value === $nullValue ? self::nothing() : 
            self::just($value); 
    } 

    abstract public function map(callable $f): Maybe; 
} 

final class Just extends Maybe 
{ 
    // [...] 

    public function map(callable $f): Maybe 
    { 
        return new self($f($this->value)); 
    } 
} 

final class Nothing extends Maybe 
{ 
    // [...] 

    public function map(callable $f): Maybe 
    { 
        return $this; 
    } 
} 
```

为了使实用方法的命名有些连贯性，我们使用与处理集合的函数相同的名称。在某种程度上，您可以将`Maybe`类型视为一个只有一个或没有值的列表。让我们基于相同的假设添加一些其他实用方法：

```php
<?php abstract class Maybe 
{ 
    // [...] 
    abstract public function orElse(Maybe $m): Maybe; 
    abstract public function flatMap(callable $f): Maybe;
    abstract public function filter(callable $f): Maybe;
} 

final class Just extends Maybe 
{ 
    // [...] 

    public function orElse(Maybe $m): Maybe 
    { 
        return $this; 
    } 

    public function flatMap(callable $f): Maybe 
    { 
        return $f($this->value); 
    } 

    public function filter(callable $f): Maybe 
    { 
        return $f($this->value) ? $this : Maybe::nothing(); 
    } 
} 

final class Nothing extends Maybe 
{ 
    // [...] 

    public function orElse(Maybe $m): Maybe 
    { 
        return $m; 
    } 

    public function flatMap(callable $f): Maybe 
    { 
        return $this; 
    } 

    public function filter(callable $f): Maybe 
    { 
        return $this; 
    } 
  } 
```

我们已经向我们的实现添加了三个新方法：

+   `orElse`方法如果有值则返回当前值，如果是`Nothing`则返回给定值。这使我们能够轻松地从多个可能的来源获取数据。

+   `flatMap`方法将一个可调用对象应用于我们的值，但不会将其包装在 Maybe 类中。可调用对象有责任自己返回一个 Maybe 类。

+   `filter`方法将给定的断言应用于值。如果断言返回 true 值，我们保留该值；否则，我们返回`Nothing`值。

现在我们已经实现了一个可工作的`Maybe`类型，让我们看看如何使用它轻松摆脱错误和空值管理。假设我们想要在应用程序的右上角显示有关已连接用户的信息。如果没有`Maybe`类型，您可能会做以下操作：

```php
<?php 
$user = getCurrentUser(); 

$name = $user == null ? 'Guest' : $user->name; 

echo sprintf("Welcome %s", $name); 
// Welcome John 
```

在这里，我们只使用名称，因此我们可以限制自己只进行一次空值检查。如果我们需要从用户那里获取更多信息，通常的方法是使用一种有时被称为**空对象**模式的模式。在我们的情况下，我们的空对象将是`AnonymousUser`方法的一个实例：

```php
<?php 

$user = getCurrentUser(); 

if($user == null) { 
   $user = new AnonymousUser(); 
} 

echo sprintf("Welcome %s", $user->name); 
// Welcome John 
```

现在让我们尝试使用我们的`Maybe`类型做同样的事情：

```php
<?php 

$user = Maybe::fromValue(getCurrentUser()); 

$name = $user->map(function(User $u) { 
  return $u->name; 
})->getOrElse('Guest'); 

echo sprintf("Welcome %s", $name); 
// Welcome John 

echo sprintf("Welcome %s", $user->getOrElse(new AnonymousUser())->name); 
// Welcome John 
```

第一个版本可能不会好多少，因为我们不得不创建一个新的函数来提取名称。但让我们记住，在需要提取最终值之前，你可以对对象进行任意数量的处理。此外，我们稍后介绍的大多数函数库都提供了更简单地从对象中获取值的辅助方法。

你还可以轻松地调用一系列方法，直到其中一个返回一个值。比如你想显示一个仪表板，但这些可以根据每个组和每个级别重新定义。让我们比较一下我们的两种方法的表现。

首先，空值检查方法：

```php
<?php 

$dashboard = getUserDashboard(); 
if($dashboard == null) { 
    $dashboard = getGroupDashboard(); 
} 
if($dashboard == null) { 
    $dashboard = getDashboard(); 
} 
```

现在，使用`Maybe`类型：

```php
<?php 

/* We assume the dashboards method now return Maybe instances */ 
$dashboard = getUserDashboard() 
             ->orElse(getGroupDashboard()) 
             ->orElse(getDashboard()); 
```

我认为更易读的那个更容易确定！

最后，让我们演示一个小例子，说明我们如何可以在`Maybe`实例上链式调用多个调用，而无需检查我们当前是否有值。所选择的例子可能有点愚蠢，但它展示了可能的情况：

```php
<?php 

$num = Maybe::fromValue(42); 

$val = $num->map(function($n) { return $n * 2; }) 
         ->filter(function($n) { return $n < 80; }) 
         ->map(function($n) { return $n + 10; }) 
         ->orElse(Maybe::fromValue(99)) 
         ->map(function($n) { return $n / 3; }) 
         ->getOrElse(0); 
echo $val; 
// 33 
```

我们的`Maybe`类型的强大之处在于，我们从未考虑过实例是否包含值。我们只能将函数应用于它，直到最后，使用`getOrElse`方法提取最终值。

## 提升函数

我们已经看到了我们新的`Maybe`类型的强大之处。但事实是，你要么没有时间重写所有现有的函数来支持它，要么根本无法这样做，因为它们是外部第三方的。

幸运的是，你可以**提升**一个函数，创建一个新的函数，它以`Maybe`类型作为参数，将原始函数应用于其值，并返回修改后的`Maybe`类型。

为此，我们将需要一个新的辅助函数。为了保持事情相对简单，我们还将假设，如果提升函数的任何参数的值评估为`Nothing`，我们将简单地返回`Nothing`：

```php
<?php 

function lift(callable $f) 
{ 
    return function() use ($f) 
    { 
        if(array_reduce(func_get_args(), function(bool $status, Maybe $m) { 
            return $m->isNothing() ? false : $status; 
        }, true)) { 
            $args = array_map(function(Maybe $m) { 
                // it is safe to do so because the fold above  checked 
                // that all arguments are of type Some 
                return $m->getOrElse(null); 
            }, func_get_args()); 
            return Maybe::just(call_user_func_array($f, $args)); 
        } 
        return Maybe::nothing(); 
    }; 
} 
```

让我们试试：

```php
<?php 
function add(int $a, int $b) 
{ 
    return $a + $b; 
} 

$add2 = lift('add'); 

echo $add2(Maybe::just(1), Maybe::just(5))->getOrElse('nothing'); 
// 6 

echo $add2(Maybe::just(1), Maybe::nothing())- >getOrElse('nothing'); 
// nothing 
```

现在你可以提升任何函数，以便它可以接受我们的新`Maybe`类型。唯一需要考虑的是，如果你想依赖函数的任何可选参数，它将不起作用。

我们可以使用反射或其他手段来确定函数是否具有可选值，或者将一些默认值传递给提升的函数，但这只会使事情变得更加复杂，并使我们的函数变得更慢。如果你需要使用带有可选参数和`Maybe`类型的函数，你可以重写它或为它制作一个自定义包装器。

最后，提升的过程并不局限于 Maybe 类型。你可以提升任何函数以接受任何类型的容器。我们的辅助程序更好的名称可能是**liftMaybe**，或者我们可以将其添加为`Maybe`类的静态方法，以使事情更清晰。

## Either 类型

`Either`类型是我们`Maybe`类型的泛化。与其有值和无值不同，你有左值和右值。由于它也是一个联合类型，这两种可能的值中只能有一个在任何给定时间被设置。

当只有少数错误来源或错误本身并不重要时，`Maybe`类型的工作效果很好。使用`Either`类型，我们可以通过左值提供任何我们想要的错误信息。右值用于成功，因为这是一个明显的双关语。

这是`Either`类型的一个简单实现。由于代码本身相当无聊，书中只介绍了基类。你可以在 Packt 网站上访问两个子类：

```php
<?php 
abstract class Either 
{ 
    protected $value; 

    public function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public static function right($value): Right 
    { 
        return new Right($value); 
    } 

    public static function left($value): Left 
    { 
        return new Left($value); 
    } 

    abstract public function isRight(): bool; 
    abstract public function isLeft(): bool; 
    abstract public function getRight(); 
    abstract public function getLeft(); 
    abstract public function getOrElse($default); 
    abstract public function orElse(Either $e): Either; 
    abstract public function map(callable $f): Either; 
    abstract public function flatMap(callable $f): Either; 
    abstract public function filter(callable $f, $error): Either; 
} 
```

该实现提议与我们为`Maybe`类提供的 API 相同，假设右值是有效的。你应该能够在不改变逻辑的情况下，到处使用`Either`类而不是`Maybe`类。唯一的区别是检查我们处于哪种情况的方法，并将方法更改为新的`getRight`或`getLeft`方法。

也可以为我们的新类型编写提升：

```php
<?php 
function liftEither(callable $f, $error = "An error occured") 
{ 
    return function() use ($f) 
    { 
        if(array_reduce(func_get_args(), function(bool $status, Either $e) { 
            return $e->isLeft() ? false : $status; 
        }, true)) { 
            $args = array_map(function(Either $e) { 
                // it is safe to do so because the fold above  checked 
                // that all arguments are of type Some 
                return $e->getRight(null); 
            }, func_get_args()); 
            return Either::right(call_user_func_array($f, $args)); 
        } 
        return Either::left($error); 
    }; 
} 
```

然而，这个函数比自定义包装器要少一些用处，因为你无法指定一个特定于可能的错误的错误消息。

# 图书馆

现在我们已经介绍了 PHP 中已有的各种功能性技术的基础知识，是时候看看各种库了，这些库将使我们能够专注于我们的业务代码，而不是编写辅助函数和实用程序函数，就像我们使用新的`Maybe`和`Either`类型一样。

## 功能性 PHP 库

`functional-php`库可能是与 PHP 相关的功能性编程中最古老的库之一，因为它的第一个版本可以追溯到 2011 年 6 月。它与最新的 PHP 版本良好地发展，并且甚至去年切换到了 Composer 进行分发。

该代码可在 GitHub 上找到[`github.com/lstrojny/functional-php`](https://github.com/lstrojny/functional-php)。如果您习惯使用 Composer，安装应该非常容易，只需写入以下命令：

```php
**composer require lstrojny/functional-php.**

```

该库曾经在 PHP 中实现，并作为 C 扩展的一部分出于性能原因。但是，由于 PHP 核心在速度方面的最新改进以及维护两个代码库的负担，该扩展已经过时。

实现了许多辅助函数-我们现在没有足够的空间详细介绍每一个。如果您感兴趣，可以查看文档。但是，我们将快速介绍重要的函数，本书的其余部分将包含更多的示例。

此外，我们还没有讨论库相关函数涵盖的一些概念，我们将在处理这些主题时进行介绍。

### 如何使用这些函数

正如在第一章中已经讨论的那样，自 PHP 5.6 以来，您可以从命名空间导入函数。这是使用该库的最简单方法。您还可以导入整个命名空间，并在调用函数时添加前缀：

```php
<?php 
require_once __DIR__.'/vendor/autoload.php'; 

use function Functional\map; 

map(range(0, 4), function($v) { return $v * 2; }); 

use Functional as F; 

F\map(range(0, 4), function($v) { return $v * 2; }); 
```

还要注意的是，大多数函数接受数组和实现`Traversable`接口的任何内容，例如迭代器。

### 通用辅助函数

这些函数可以帮助您在各种情境下，而不仅仅是在功能性方面：

+   `true`和`false`函数检查集合中的所有元素是否严格为 True 或严格为 False。

+   `truthy`和`falsy`函数与以前相同，但比较不是严格的。

+   `const_function`函数返回一个新函数，该函数将始终返回给定值。这可以用于模拟不可变数据。

### 扩展 PHP 函数

PHP 函数倾向于仅在*真实*数组上工作。以下函数将它们的行为扩展到任何可以使用`foreach`循环进行迭代的内容。所有函数的参数顺序也保持一致：

+   `contains`方法检查给定集合中是否包含该值。第三个参数控制比较是否应该是严格的。

+   `sort`方法对集合进行排序，但返回一个新数组，而不是通过引用进行排序。您可以决定是否保留键。

+   `map`方法将`array_map`方法的行为扩展到所有集合。

+   `sum`、`maximum`和`minimum`方法在任何类型的集合上执行与它们的 PHP 对应方法相同的工作。除此之外，该库还包含 product、ratio、difference 和 average。

+   当您不传递函数时，`zip`方法执行与`array_map`方法相同的工作。但是，您也可以传递回调函数来确定如何合并各个项目。

+   `reduce_left`和`reduce_right`方法从左侧或右侧折叠集合。

### 使用谓词

在处理集合时，通常希望检查某些、全部或没有元素是否满足某个条件，并相应地采取行动。为了做到这一点，您可以使用以下函数：

+   `every`函数如果集合的所有元素都对谓词有效，则返回 true 值

+   `some`函数如果至少有一个元素对谓词有效，则返回 true 值

+   `none`函数如果没有元素对于谓词有效，则返回 true

这些函数不会修改集合。它们只是检查元素是否符合某个条件。如果需要过滤一些元素，可以使用以下辅助函数：

+   `select`或`filter`函数仅返回对于谓词有效的元素。

+   `reject`函数仅返回对于谓词无效的元素。

+   `head`函数返回对于谓词有效的第一个元素。

+   最后一个函数返回对于谓词有效的最后一个元素。

+   `drop_first`函数从集合的开头删除元素，直到给定的回调为`true`。一旦回调返回 false，停止删除元素。

+   `drop_last`函数与上一个函数相同，但是从末尾开始。

所有这些函数都返回一个新的数组，原始集合保持不变。

### 调用函数

当您想在回调中调用函数时，声明匿名函数是很麻烦的。这些辅助函数将为您提供更简单的语法：

+   `invoke`辅助函数在集合中的所有对象上调用方法，并返回具有结果的新集合

+   `invoke_first`和`invoke_last`辅助函数分别在集合的第一个和最后一个对象上调用方法

+   `invoke_if`辅助函数如果第一个参数是有效对象，则调用给定的方法。您可以传递方法参数和默认值。

+   `invoker`辅助函数返回一个新的可调用对象，它使用给定的参数调用给定的方法。

您可能还希望调用函数，直到获得一个值或达到某个阈值。该库已经为您做好了准备：

+   `retry`库调用函数，直到它停止返回异常或达到尝试次数

+   `poll`库调用函数，直到它返回真值或达到给定的超时时间

### 操作数据

之前的函数组是关于使用辅助函数调用函数；这个函数组是关于获取和操作数据，而不必每次都求助于匿名函数：

+   `pluck`函数从给定集合中的所有对象中提取属性，并返回具有这些值的新集合。

+   `pick`函数根据给定的键从数组中选择一个元素。如果元素不存在，可以提供默认值。

+   `first_index_of`和`last_index_of`函数分别返回匹配给定值的元素的第一个和最后一个索引。

+   `indexes_of`函数返回所有匹配给定值的索引。

+   `flatten`函数将嵌套集合的深度减少为单个平面集合。

有时，您也希望根据谓词或某个分组值将集合分成多个部分：

+   `partition`方法接受一组谓词-根据第一个谓词的有效性，将集合的每个项目放入给定的组中。

+   `group`方法根据每个元素的回调返回的每个不同值创建多个组

### 总结

正如您所看到的，`functional-php`库提供了许多不同的辅助函数和实用函数。现在可能不明显您如何充分利用它们，但我希望本书的剩余部分能让您一窥您可以实现的内容。

另外，不要忘记我们没有介绍所有的函数，因为其中一些需要一点理论解释。一切都在适当的时间。

## php-option 库

我们之前创建了自己版本的`Maybe`类型。这个库提出了一个更完整的实现。选择了 Scala 使用的命名，然而。源代码在 GitHub 上[`github.com/schmittjoh/php-option`](https://github.com/schmittjoh/php-option)。最简单的安装方法是使用 Composer 写入以下命令：

```php
**composer require phpoption/phpoption**

```

一个有趣的补充是`LazyOption`方法，它接受一个回调而不是一个值。只有在需要值时才会执行回调。当您使用`orElse`方法为前一个无效值提供替代时，这是特别有趣的。在这种情况下使用`LazyOption`方法，可以避免在一个值有效时进行不必要的计算。

您还可以使用各种辅助程序来帮助您仅在值有效时调用方法，例如，还提供了多种实例化可能性。该库还提供了一个 API，更类似于您习惯于集合的 API。

## Laravel 集合

如第一章所述，Laravel 提供了一个很好的库来管理集合。它声明了一个名为`Collection`的类，该类在其 ORM **Eloquent**和大多数其他依赖于集合的部分内部使用。

在内部，使用了一个简单的数组，但以一种促进数据的不可变性和功能性方法来包装它。为了实现这个目标，为开发人员提供了 60 到 70 种方法。

如果您已经在使用 Laravel，您可能已经熟悉此支持类提供的可能性。如果您正在使用其他任何框架，仍然可以从中受益，方法是从[`github.com/tightenco/collect`](https://github.com/tightenco/collect)获取提取的部分。

文档可在 Laravel 官方网站[`laravel.com/docs/collections`](https://laravel.com/docs/collections)上找到。我们不会详细描述每个方法，因为它们有很多。如果您正在使用 Laravel 并想了解其集合提供的所有可能性，可以前往[`adamwathan.me/refactoring-to-collections/`](https://adamwathan.me/refactoring-to-collections/)。

### 使用 Laravel 的集合

第一步是使用 collect 实用程序函数将数组或`Traversable`接口转换为`Collection`类的实例。然后您将可以访问类提供的各种方法。让我们快速列出到目前为止我们已经以另一种形式遇到的那些方法：

+   `map`方法将函数应用于所有元素并返回新值

+   `filter`方法使用谓词过滤集合

+   `reduce`方法使用给定的回调函数折叠集合

+   `pluck`方法从所有元素中获取给定的属性

+   `groupBy`方法使用每个元素的给定值对集合进行分区

所有这些方法都返回`Collection`类的新实例，保留原始实例的值。

完成操作后，您可以使用 all 方法将当前值作为数组获取。

## immutable-php 库

这个提出不可变数据结构的库是由于对**标准 PHP**库中的`SplFixedArray`方法的各种抱怨，主要是其难以使用的 API。在其核心，`immutable-php`库使用前面提到的数据结构，但使用一组很好的方法来包装它。

`SplFixedArray`方法是一个具有固定大小并且只允许数字索引的数组的特定实现。这些约束允许实现一个非常快速的数组结构。

您可以在 GitHub 项目页面[`github.com/jkoudys/immutable.php`](https://github.com/jkoudys/immutable.php)上查看或通过使用 Composer 编写以下命令来安装它：

```php
**composer require qaribou/immutable.php.**

```

### 使用 immutable.php

使用专用的静态助手`fromArray`或`fromItems`为`Traversable`类的任何实例创建新实例非常容易。您新创建的`ImmArray`实例可以像任何数组一样访问，使用`foreach`循环进行迭代，并使用`count`方法进行计数。但是，如果尝试设置一个值，将会收到异常。

一旦你有了不可变数组，你可以使用各种方法来应用你现在应该习惯的转换：

+   `map` 方法将函数应用于所有项目并返回新值

+   `filter` 方法创建仅包含谓词有效项目的新数组

+   `reduce` 方法使用回调折叠项目

你还有其他帮手：

+   `join` 方法连接字符串集合

+   `sort` 方法使用给定的回调返回排序后的集合

你的数据也可以很容易地以传统数组形式检索或编码为 JSON 格式。

总的来说，这个库提供的方法比 Laravel 的 Collection 更少，但性能更好，内存占用更低。

## 其他库

由于 PHP 核心缺乏很多实用函数和功能来进行适当的函数式编程，很多人开始致力于实现缺失部分的库。这就是为什么如果你开始寻找，你会发现很多这样的库。

以下是一份不完整且无序的库列表，如果之前介绍的那些不符合你的需求。

### Underscore.php 库

基于 `Underscore.js` 库的 API 存在多种用于 PHP 的端口。我个人不太喜欢 `Underscore.js` 库，因为函数参数经常顺序错误，无法有效地进行函数组合。这一点在这个视频中有很好的解释：[`www.youtube.com/watch?v=m3svKOdZijA`](https://www.youtube.com/watch?v=m3svKOdZijA)。

然而，如果你习惯使用它，这是一个各种端口的简短列表：

+   [`github.com/brianhaveri/Underscore.php`](https://github.com/brianhaveri/Underscore.php)：据我所知，这是最古老的端口。自 2012 年以来就没有任何活动，但存在很多分支来改进与新版本 PHP 的兼容性并修复错误。

+   [`github.com/wikiHow/Underscore.php`](https://github.com/wikiHow/Underscore.php)：前述库中最受维护的分支之一。

+   [`github.com/Anahkiasen/underscore-php`](https://github.com/Anahkiasen/underscore-php)：最初是其 JS 对应的一个端口。现在它包含一些不同的功能，试图尊重原始的哲学。

+   [`github.com/Im0rtality/Underscore`](https://github.com/Im0rtality/Underscore)：更近期的尝试，类似于 `Underscore.js` 库。在撰写本文时，文档缺少一些重要主题，而且该库在很多地方与 JavaScript 版本不同。

### Saber

**Saber** 严格遵循最新的 PHP 版本作为要求。它使用强类型、不可变对象和惰性求值。为了使用它的各种方法，你必须将你的值*装箱*到库提供的类中。这可能有些麻烦，但它提供了安全性并减少了错误。

它似乎受到 C# 和主要是 F# 的启发，后者是在 .NET 虚拟机上运行的函数语言，或者用其真实名称 **`CLR`** 来称呼它。你可以在 GitHub 上找到源代码和文档：[`github.com/bluesnowman/fphp-saber`](https://github.com/bluesnowman/fphp-saber)。

### Rawr

**Rawr** 不仅仅是一个函数库。它试图以更一般的方式修复 PHP 语言的缺陷。与 Saber 一样，它提供了一个新类来装箱你的标量值；然而，类型的使用更接近 Haskell。你还可以将你的匿名函数包装在一个类中，以在其周围获得更好的类型安全性。

该库还添加了更多 **Smalltalk** 风格的面向对象、单子，并允许你执行一些类似 JavaScript 的基于原型的编程。

遗憾的是，该库似乎停滞不前，文档与源代码不同步。然而，您可以在那里找到一些灵感。您可以在 GitHub 上找到代码[`github.com/haskellcamargo/rawr`](https://github.com/haskellcamargo/rawr)。

### PHP 功能性

这个库主要围绕我们将在第五章中看到的 Monad 的概念。承认的灵感来自 Haskell，该库实现了：

+   State Monad

+   IO Monad

+   集合 Monad

+   Either Monad

+   Maybe Monad

通过`Collection` Monad，该库提供了我们期望的各种方法`map`、`reduce`和`filter`方法。

由于受 Haskell 的启发，您可能会发现在开始时使用它有点困难。然而，最终它应该会更加强大。您可以在 GitHub 上找到代码[`github.com/widmogrod/php-functional`](https://github.com/widmogrod/php-functional)。

### 功能性

最初创建为一个学习游乐场，这个库已经发展成为一个相对小型但有用的东西。主要的想法是提供一个框架，以便您可以在代码中删除所有循环。

最有趣的特点是所有函数都可以部分应用而无需进行任何特殊操作。部分应用对于函数组合非常重要。我们将在第四章中发现这两个主题，*组合函数*。

该库还具有所有传统的竞争者，如映射和减少。代码和文档可在 GitHub 上找到[`github.com/sergiors/functional`](https://github.com/sergiors/functional)。

### PHP 函数式编程工具

这个库试图走与我们在前几页中介绍的`functional-php`库相同的道路。据我所知，它目前的功能略少。对于想要更小、可能更容易学习的人来说，这可能是一个有趣的库。代码在 GitHub 上[`github.com/daveross/functional-programming-utils`](https://github.com/daveross/functional-programming-utils)。

### 非标准 PHP 库

这个库并不严格是一个功能性的库。这个想法更多的是通过各种辅助和实用函数来扩展标准库，以使处理集合更加容易。

它包含一些有用的功能，例如帮助轻松验证函数参数，无论是使用已定义的约束还是自定义的约束。它还扩展了现有的 PHP 函数，以便它们可以处理任何`Traversable`接口的内容，而不仅仅是数组。

该库创建于 2014 年，但直到 2015 年底工作再次开始才几乎停滞不前。现在它可能是我们之前介绍的任何库的替代品。如果您感兴趣，请在 GitHub 上获取代码[`github.com/ihor/Nspl`](https://github.com/ihor/Nspl)。

# 总结

在这一长章节中，我们介绍了我们将在整本书中使用的所有实用构建块。希望这些例子不会显得太枯燥。有很多内容要涵盖，而页面数量有限。接下来的章节将在我们学到的基础上进行更好的示例。

您首先阅读了一些关于编程的一般建议，这对于功能代码库尤其重要。然后，我们发现了基本的功能技术，如映射、折叠、过滤和压缩，所有这些都可以直接在 PHP 中使用。

接下来是对递归的简要介绍，这是一种解决特定问题集的技术，也是避免使用循环的方法。在一本关于功能性语言的书中，这个主题可能值得一整章，但由于 PHP 有各种循环结构，它的重要性稍低一些。此外，我们将在接下来的章节中看到更多的递归示例。

我们还讨论了异常以及它们在功能代码库中引发的问题，并在讨论其他方法的利弊后，编写了 Maybe 和 Either 类型的实现，作为更好地管理错误的方法。

最后，我们介绍了一些提供功能构造和辅助功能的库，这样我们就不必自己编写了。
