# 第四章。组合函数

在之前的章节中，我们谈了很多关于构建模块和小纯函数。 但到目前为止，我们甚至没有暗示这些如何用来构建更大的东西。 如果你不能使用构建模块，那么构建模块有什么用呢？ 答案部分地在于函数组合。

尽管这一章完成了前一章，但这种技术是任何函数式程序的一个不可或缺的重要部分，因此它值得有自己的一章。

在本章中，我们将涵盖以下主题：

+   函数组合

+   部分应用

+   柯里化

+   参数顺序的重要性

+   这些概念的现实应用

# 函数组合

正如在函数式编程中经常发生的那样，函数组合的概念是从数学中借来的。 如果您有两个函数`f`和`g`，您可以通过组合它们来创建第三个函数。 数学中的通常表示法是*(f g)(x)*，这相当于依次调用它们*f(g(x))*。

您可以使用一个包装函数非常容易地组合任何两个给定的函数与 PHP。 比如说，您想以大写字母显示标题，并且只保留安全的 HTML 字符：

```php
<?php 

function safe_title(string $s) 
{ 
    $safe = htmlspecialchars($s); 
    return strtoupper($safe); 
} 
```

您也可以完全避免临时变量：

```php
<?php 

function safe_title2(string $s) 
{ 
    return strtoupper(htmlspecialchars($s)); 
} 
```

当您只想组合几个函数时，这样做效果很好。 但是创建很多这样的包装函数可能会变得非常繁琐。 如果您能简单地使用`$safe_title = strtoupper htmlspecialchars`这行代码会怎么样呢？ 遗憾的是，PHP 中不存在这样的运算符，但我们之前介绍的`functional-php`库包含一个`compose`函数，它正是这样做的：

```php
<?php 
require_once __DIR__.'/vendor/autoload.php'; 

use function Functional\compose; 

$safe_title2 = compose('htmlspecialchars', 'strtoupper'); 
```

收益可能看起来并不重要，但让我们在更多的上下文中比较一下使用这种方法：

```php
<?php 

$titles = ['Firefly', 'Buffy the Vampire Slayer', 'Stargate Atlantis', 'Tom & Jerry', 'Dawson's Creek']; 

$titles2 = array_map(function(string $s) { 
    return strtoupper(htmlspecialchars($s)); 
}, $titles); 

$titles3 = array_map(compose('htmlspecialchars', 'strtoupper'),  $titles); 
```

就个人而言，我发现第二种方法更容易阅读和理解。 而且它变得更好了，因为您可以将两个以上的函数传递给`compose`函数：

```php
<?php 

$titles4 = array_map(compose('htmlspecialchars', 'strtoupper', 'trim'), $titles); 
```

一个可能会误导的事情是函数应用的顺序。 数学表示法`f ∘ g`首先应用`g`，然后将结果传递给`f`。 然而，`functional-php`库中的`compose`函数按照它们在`compose('first', 'second', 'third')`参数中传递的顺序应用函数。

这可能更容易理解，取决于您的个人偏好，但是当您使用另一个库时要小心，因为应用的顺序可能会被颠倒。 一定要确保您已经仔细阅读了文档。

# 部分应用

您可能想设置函数的一些参数，但将其中一些参数留到以后再分配。 例如，我们可能想创建一个返回博客文章摘录的函数。

设置这样一个值的专用术语是**绑定参数**或**绑定参数**。 过程本身称为**部分应用**，新函数被设置为部分应用。

这样做的天真方式是将函数包装在一个新函数中：

```php
<?php 
function excerpt(string $s) 
{ 
    return substr($s, 0, 5); 
} 

echo excerpt('Lorem ipsum dolor si amet.'); 
// Lorem 
```

与组合一样，总是创建新函数可能会很快变得繁琐。 但再一次，`functional-php`库为我们提供了帮助。 您可以决定从左侧、右侧或函数签名中的任何特定位置绑定参数，分别使用`partial_left`、`partial_right`或`partial_any`函数。

为什么有三个函数？ 主要是出于性能原因，因为左侧和右侧版本的性能会更快，因为参数将被一次性替换，而最后一个将使用在每次调用新函数时计算的占位符。

在上一个例子中，占位符是使用函数`...`定义的，它是省略号 Unicode 字符。 如果您的计算机没有简便的方法输入它，您也可以使用`Functional`命名空间中的`placeholder`函数，它是一个别名。

# 柯里化

**柯里化**经常被用作部分应用的同义词。 尽管这两个概念都允许我们绑定函数的一些参数，但核心思想有些不同。

柯里化的思想是将接受多个参数的函数转换为接受一个参数的函数序列。由于这可能有点难以理解，让我们尝试对`substr`函数进行柯里化。结果被称为**柯里化函数**：

```php
<?php 

function substr_curryied(string $s) 
{ 
    return function(int $start) use($s) { 
        return function(int $length) use($s, $start) { 
            return substr($s, $start, $length); 
        }; 
    }; 
} 

$f = substr_curryied('Lorem ipsum dolor sit amet.'); 
$g = $f(0); 
echo $g(5); 
// Lorem 
```

正如你所看到的，每次调用都会返回一个接受下一个参数的新函数。这说明了与部分应用的主要区别。当你调用部分应用的函数时，你会得到一个结果。但是，当你调用柯里化函数时，你会得到一个新的函数，直到传递最后一个参数。此外，你只能按顺序从左边开始绑定参数。

如果调用链看起来过长，你可以从 PHP 7 开始大大简化它。这是因为 RFC *统一变量语法*已经实现（详见[`wiki.php.net/rfc/uniform_variable_syntax`](https://wiki.php.net/rfc/uniform_variable_syntax)）：

```php
<?php 

echo substr_curryied('Lorem ipsum dolor sit amet.')(0)(5); 
// Lorem 
```

柯里化的优势可能在这样的情况下并不明显。但是，一旦你开始使用高阶函数，比如`map`或`reduce`函数，这个想法就变得非常强大了。

你可能还记得`functional-php`库中的`pluck`函数。这个想法是从对象集合中检索给定的属性。如果`pluck`函数被实现为柯里化函数，它可以以多种方式使用：

```php
<?php 

function pluck(string $property) 
{ 
    return function($o) use($property) { 
        if (is_object($o) && isset($o->{$propertyName})) { 
            return $o->{$property}; 
        } elseif ((is_array($o) || $o instanceof ArrayAccess) &&  isset($o[$property])) { 
            return $o[$property]; 
        } 

        return false; 
    }; 
} 
```

我们可以轻松地从任何类型的对象或数组中获取值：

```php
<?php 

$user = ['name' => 'Gilles', 'country' => 'Switzerland', 'member'  => true]; 
pluck('name')($user); 
```

我们可以从对象集合中提取属性，就像`functional-php`库中的版本一样：

```php
<?php 

$users = [ 
    ['name' => 'Gilles', 'country' => 'Switzerland', 'member' =>  true], 
    ['name' => 'Léon', 'country' => 'Canada', 'member' => false], 
    ['name' => 'Olive', 'country' => 'England', 'member' => true], 
]; 
pluck('country')($users); 
```

由于我们的实现在找不到内容时返回`false`，我们可以用它来过滤包含特定值的数组：

```php
<?php 

array_filter($users, pluck('member')); 
```

我们可以组合多个用例来获取所有成员的名称：

```php
<?php 

pluck('name', array_filter($users, pluck('member'))); 
```

如果没有柯里化，我们要么需要编写一个更传统的`pluck`函数的包装器，要么创建三个专门的函数。

让我们再进一步，结合多个柯里化函数。首先，我们需要创建一个包装函数，包装`array_map`和`preg_replace`函数：

```php
<?php 

function map(callable $callback) 
{ 
    return function(array $array) use($callback) { 
        return array_map($callback, $array); 
    }; 
} 

function replace($regex) 
{ 
    return function(string $replacement) use($regex) { 
        return function(string $subject) use($regex, $replacement)  
{ 
            return preg_replace($regex, $replacement, $subject); 
        }; 
    }; 
} 
```

现在我们可以使用这些来创建多个新函数，例如，一个将字符串中所有空格替换为下划线或所有元音字母替换为星号的函数：

```php
<?php function map(callable $callback) 
{ 
    return function(array $array) use($callback) { 
        return array_map($callback, $array); 
    }; 
} 

function replace($regex) 
{ 
    return function(string $replacement) use($regex) { 
        return function(string $subject) use($regex, $replacement)  
{ 
            return preg_replace($regex, $replacement, $subject); 
        }; 
    }; 
} 
```

## 在 PHP 中进行柯里化函数

我希望你现在已经相信了柯里化的力量。如果没有，我希望接下来的例子能说服你。与此同时，你可能会认为围绕现有函数编写新的实用程序函数来创建新的柯里化版本真的很麻烦，你是对的。

在 Haskell 等语言中，所有函数默认都是柯里化的。不幸的是，PHP 中并非如此，但这个过程足够简单和重复，我们可以编写一个辅助函数。

由于 PHP 中可能有可选参数的可能性，我们首先要创建一个名为`curry_n`的函数，该函数接受你想要柯里化的参数数量。这样，你也可以决定是否要对所有参数进行柯里化，还是只对其中一些进行柯里化。它也可以用于具有可变参数数量的函数：

```php
<?php 

function curry_n(int $count, callable $function): callable 
{ 
    $accumulator = function(array $arguments) use($count,  $function, &$accumulator) { 
        return function() use($count, $function, $arguments,  $accumulator) { 
            $arguments = array_merge($arguments, func_get_args()); 

            if($count <= count($arguments)) { 
                return call_user_func_array($function,  $arguments); 
            } 

            return $accumulator($arguments); 
        }; 
    }; 
    return $accumulator([]); 
} 
```

这个想法是使用一个内部辅助函数，将已传递的值作为参数，然后使用这些创建一个闭包。当调用时，闭包将根据实际值的数量决定我们是否可以调用原始函数，或者我们是否需要使用我们的辅助函数创建一个新函数。

请注意，如果你给出的参数计数高于实际计数，所有多余的参数将被传递到原始函数，但可能会被忽略。此外，给出较小的计数将导致最后一步期望更多的参数才能正确完成。

现在我们可以创建第二个函数，使用`reflection`变量来确定参数的数量：

```php
<?php 

function curry(callable $function, bool $required = true):  callable 
{ 
    if(is_string($function) && strpos($function, '::', 1) !==  false) { 
        $reflection = new \ReflectionMethod($f); 
    }  
    else if(is_array($function) && count($function) == 2)  
    { 
        $reflection = new \ReflectionMethod($function[0],  $function[1]); 
    }  
    else if(is_object($function) && method_exists($function,  '__invoke'))  
    { 
        $reflection = new \ReflectionMethod($function,  '__invoke'); 
    }  
    else  
    {         
        $reflection = new \ReflectionFunction($function); 
    } 

    $count = $required ? $reflection->getNumberOfRequiredParameters() : 
        $reflection->getNumberOfParameters(); 

    return curry_n($count, $function); 
} 
```

正如你所看到的，没有简单的方法来确定函数期望的参数数量。我们还必须添加一个参数来确定我们是否应该考虑所有参数，包括具有默认值的参数，还是只考虑必填参数。

您可能已经注意到，我们并没有创建严格只接受一个参数的函数；相反，我们使用了`func_get_args`函数来获取所有传递的参数。这使得使用柯里化函数更加自然，并且与函数式语言中所做的事情相当。我们对柯里化的定义现在更接近于*一个函数，直到接收到所有参数才返回一个新函数*。

本书其余部分的示例将假定此柯里化函数可用并准备好使用。

在撰写本文时，`functional-php`库上有一个待处理的拉取请求，以合并此函数。

# 参数顺序非常重要！

正如你可能还记得第一章所述，`array_map`和`array_filter`函数的参数顺序不同。当然，这使它们更难使用，因为你更容易出错，但这并不是唯一的问题。为了说明参数顺序的重要性，让我们创建这两个函数的柯里化版本：

```php
<?php 

$map = curry_n(2, 'array_map'); 
$filter = curry_n(2, 'array_filter'); 
```

我们在这里使用`curry_n`函数有两个不同的原因：

+   `array_map`函数接受可变数量的数组，因此我们强制将值设为 2 以确保安全

+   `array_filter`函数有一个名为`$flag`的第三个参数，其可选值是可以接受的

还记得我们新的柯里化函数的参数顺序吗？`$map`参数将首先获取回调函数，而`$filters`参数期望首先获取集合。让我们尝试创建一个新的有用函数，了解这一点：

```php
<?php 

$trim = $map('trim'); 
$hash = $map('sha1'); 

$oddNumbers = $filter([1, 3, 5, 7]); 
$vowels = $filter(['a', 'e', 'i', 'o', 'u']); 
```

我们的映射示例确实非常基础，但有一定用途，而我们的过滤示例只是静态数据。我敢打赌，你可以找到一些方法来使用`$trim`和`$hash`参数，但你需要一个奇数或元音字母的列表来进行过滤的可能性有多大呢？

本章稍早前的另一个例子可以从这里得到-还记得我们对`substr`函数的柯里化示例吗？

```php
<?php 

function substr_curryied(string $s) 
{ 
    return function(int $start) use($s) { 
        return function(int $length) use($s, $start) { 
            return substr($s, $start, $length); 
        }; 
    }; 
} 

$f = substr_curryied('Lorem ipsum dolor sit amet.'); 
$g = $f(0); 
echo $g(5); 
// Lorem 
```

我可以向你保证，如果我们首先定义开始和长度来创建，那将会更有用。例如，一个`$take5fromStart`函数；而不是拥有这些尴尬的`$substrOnLoremIpsum`参数，我们只需在示例中调用`$f`参数。

这里重要的是，你想要操作的数据，也就是你的“主题”，必须放在最后，因为这大大增加了对柯里化函数的重用，并且让你可以将它们作为其他高阶函数的参数使用。

就像上一个例子一样，假设我们想要创建一个函数，该函数获取集合中所有元素的前两个字母。我们将尝试使用一组两个函数来实现，其中参数的顺序不同。

函数的实现留作练习，因为这并不重要。

在第一个例子中，主语是第一个参数：

```php
<?php 

$map = curry(function(array $array, callable $cb) {}); 
$take = curry(function(string $string, int $count) {}); 

$firstTwo = function(array $array) { 
    return $map($array, function(string $s) { 
        return $take($s, 2); 
    }); 
} 
```

参数顺序迫使我们创建包装函数。事实上，即使函数是柯里化的，也无关紧要，因为我们无法利用这一点。

在第二个例子中，主语位于最后：

```php
<?php 

$map = curry(function(callable $cb, array $array) {}); 
$take = curry(function(int $count, string $string) {}); 

$firstTwo = $map($take(2)); 
```

事实上，精心选择的顺序也对函数组合有很大帮助，正如我们将在下一节中看到的那样。

最后，关于主题的说明，为了公平起见，使用参数顺序相反的函数版本可以使用`functional-php`库中的`partial_right`函数编写，并且您可以使用`partial_any`函数来处理参数顺序奇怪的函数。但即便如此，解决方案也不像参数顺序正确的解决方案那样简单：

```php
<?php 

use function Functional\partial_right; 

$firstTwo = partial_right($map, partial_right($take, 2)); 
```

# 使用组合来解决真正的问题

举个例子，假设你的老板走进来，希望你制作一个新报告，其中包含过去 30 天内注册的所有用户的电话号码。我们假设有以下类来表示我们的用户。显然，一个真正的类将存储和返回真实数据，但让我们只定义我们的 API：

```php
<?php 

class User { 
    public function phone(): string 
    { 
        return ''; 
    } 

    public function registration_date(): DateTime 
    { 
        return new DateTime(); 
    } 
} 

$users = [new User(), new User(), new User()]; // etc. 
```

对于没有任何函数式编程知识的人来说，你可能会写出这样的代码：

```php
<?php 

class User { 
    public function phone(): string 
    { 
        return ''; 
    }  
    public function registration_date(): DateTime 
    { 
        return new DateTime(); 
    } 
} 

$users = [new User(), new User(), new User()]; // etc. 
```

我们的函数的第一眼看告诉我们它不是纯的，因为限制是在函数内部计算的，因此后续调用可能导致不同的用户列表。我们还可以利用`map`和`filter`函数：

```php
<?php 

function getUserPhonesFromDate($limit, $users) 
{ 
    return array_map(function(User $u) { 
        return $u->phone(); 
    }, array_filter($users, function(User $u) use($limit) { 
        return $u->registration_date()->getTimestamp() > $limit; 
    })); 
} 
```

根据你的喜好，现在代码可能会更容易阅读一些，或者完全不容易，但至少我们有了一个纯函数，我们的关注点也更加分离。然而，我们可以做得更好。首先，`functional-php`库有一个函数，允许我们创建一个调用对象方法的辅助函数：

```php
<?php 

use function Functional\map; 
use function Functional\filter; 
use function Functional\partial_method; 

function getUserPhonesFromDate2($limit, $users) 
{ 
    return map( 
        filter(function(User $u) use($limit) { 
            return $u->registration_date()->getTimestamp()  >$limit; 
        }, $users), 
        partial_method('phone') 
    ); 
} 
```

这样会好一些，但如果我们接受需要创建一些新的辅助函数，我们甚至可以进一步改进解决方案。此外，这些辅助函数是我们将能够重用的新构建块：

```php
<?php 

function greater($limit) { 
    return function($a) { 
        return $a > $limit; 
    }; 
} 

function getUserPhonesFromDate3($limit, $users) 
{ 
    return map( 
        filter(compose( 
            partial_method('registration_date'), 
            partial_method('getTimestamp'), 
            greater($limit) 
          ), 
          $users), 
        partial_method('phone') 
    ); 
} 
```

如果我们有`filter`和`map`函数的柯里化版本，甚至可以创建一个只接受日期并返回一个新函数的函数，这个新函数可以进一步组合和重用：

```php
<?php 

use function Functional\partial_right; 

$filter = curry('filter'); 
$map = function($cb) { 
    return function($data) use($cb) { 
        return map($data, $cb); 
    }; 
}; 

function getPhonesFromDate($limit) 
{ 
    return function($data) use($limit) { 
        $function = compose( 
            $filter(compose( 
            partial_method('getTimestamp'), 
                partial_method('registration_date'), 
                greater($limit) 
            )), 
            $map(partial_method('phone')) 
        ); 
        return $function($data); 
    }; 
} 
```

作为一个关于拥有良好参数顺序的必要性的良好提醒，由于`functional-php`库中的`map`函数具有与 PHP 原始函数相同的签名，我们不得不手动进行柯里化。

我们的结果函数比原始的命令式函数稍长一些，但在我看来，它更容易阅读。你可以轻松地跟踪发生了什么：

1.  使用以下方式过滤数据：

1.  注册日期

1.  从中，你可以得到时间戳。

1.  检查它是否大于给定的限制。

1.  在结果上映射`phone`方法。

如果你觉得`partial_method`这个名字不太理想，并且调用`compose`函数的存在在某种程度上让人有点难以理解，我完全同意。事实上，在一个具有`compose`运算符、自动柯里化和一些语法糖来推迟对方法的调用的假设语言中，它可能看起来像这样：

```php
getFromDate($limit) = filter( 
  (->registration_date) >> 
  (->getTimestamp) >> 
  (> $limit) 
) >> map(->phone) 
```

现在我们有了我们的函数，你的老板又走进你的办公室，提出了新的要求。实际上，他只想要过去 30 天内最近的三次注册。很简单，让我们只是用一些更多的构建块来组合我们的新函数：

```php
<?php 

use function Functional\sort; 
use function Functional\compare_on; 

function take(int $count) { 
    return function($array) use($count) { 
        return array_slice($array, 0, $count); 
    }; 
}; 

function compare($a, $b) { 
    return $a == $b ? 0 : $a < $b ? -1 : 1; 
} 

function getAtMostThreeFromDate($limit) 
{ 
    return function($data) use($limit) { 
        $function = compose( 
            partial_right( 
                'sort', 
                compare_on('compare',  partial_method('registration_date')) 
            ), 
            take(3), 
            getPhonesFromDate($limit) 
        ); 
        return $function($data); 
    }; 
} 
```

为了从数组的开头获取一定数量的项目，我们需要在`array_slice`函数周围创建一个`take`函数。我们还需要一个比较值的函数，这很简单，因为`DateTime`函数重载了比较运算符。

再次，`functional-php`库对`sort`函数的参数顺序搞错了，所以我们需要部分应用而不是柯里化。而`compare_on`函数创建了一个比较器，给定一个比较函数和一个“reducer”，它在比较每个项目时被调用。在我们的情况下，我们想要比较注册日期，所以我们重用了我们的不同方法应用。

我们需要在过滤之前执行排序操作，因为我们的`getPhonesFromDate`方法只返回电话号码，正如其名称所示。我们的结果函数本身是其他函数的柯里化组合，因此可以轻松重用。

我希望这个小例子已经说服你使用小函数作为构建块并将它们组合起来解决问题的力量。如果不是这样，也许我们将在接下来的章节中看到的更高级的技术之一会说服你。

最后一点，也许你从例子中已经知道，PHP 遗憾地缺少了很多实用函数，以便让函数式编程者的生活变得更容易。而且，即使是广泛使用的`functional-php`库，也会出现一些参数顺序错误，并且缺少一些重要的代码，比如柯里化。

通过结合多个库，我们可以更好地覆盖所需的功能，但这也会增加大量无用的代码和一些不匹配的函数名称，这并不会让你的生活变得更轻松。

我可以建议的是保留一个文件，记录你在学习过程中创造的所有小技巧，很快你就会拥有自己的助手编译，真正适合你的需求和编码风格。这个建议可能违反了围绕着大型社区的可重用包的最佳实践，但在有人创建正确的库之前，它会有很大帮助。谁知道，也许你就是有足够精力创建功能 PHP 生态系统中缺失的珍珠的人。

# 总结

本章围绕函数组合展开，一旦你习惯了它，这是一个非常强大的想法。通过使用小的构建模块，你可以创建复杂的过程，同时保持短函数提供的可读性和可维护性。

我们还谈到了部分应用和柯里化的最强大概念，它们使我们能够轻松地创建现有函数的更专业化版本，并重写我们的代码以使其更易读。

我们讨论了参数顺序，这是一个经常被忽视但非常重要的话题，一旦你想使用高阶函数时就会变得重要。柯里化和正确的参数顺序的结合使我们能够减少样板代码和包装函数的需求，这个过程有时被称为 eta-reduction。

最后，通过前面提到的所有工具，我们试图演示一些你在日常编程中可能遇到的问题和难题的解决方案，以帮助你写出更好的代码。
