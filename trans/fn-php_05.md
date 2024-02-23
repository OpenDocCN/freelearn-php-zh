# 第五章。函子、应用函子和单子

上一章介绍了第一个真正的函数式技术，比如函数组合和柯里化。在本章中，我们将再次深入介绍更多的理论概念，介绍单子的概念。由于我们有很多内容要涵盖，实际应用将不会很多。然而，第六章*真实生活中的单子*将使用我们在这里学到的一切来解决真实问题。

你可能已经听说过**单子**这个术语。通常，它与非函数式程序员的恐惧感联系在一起。单子通常被描述为难以理解，尽管有无数关于这个主题的教程。事实上，它们很难理解，写这些教程的人经常忘记了他们正确理解这个想法花了多少时间。这是一个常见的教学陷阱，可能在这篇文章中更好地描述了[`byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/`](https://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/)。

你可能不会一次性理解所有内容。单子是一个非常抽象的概念，即使在本章结束时，这个主题对你来说似乎很清楚，你可能在以后遇到一些东西，它会使你对单子的真正理解感到困惑。

我会尽力清楚地解释事情，但如果你觉得我的解释不够，我在本章末尾的*进一步阅读*部分添加了关于这个主题的其他材料的参考。在本章中，我们将涵盖以下主题：

+   函子及相关法则

+   应用函子及相关法则

+   幺半群及相关法则

+   单子及相关法则

将会有很多理论内容，只有概念的实现。在第六章*真实生活中的单子*之前，不要期望有很多例子。

# 函子

在直接讲述单子之前，让我们从头开始。为了理解单子是什么，我们需要介绍一些相关概念。第一个是函子。

为了让事情变得复杂一些，术语**函子**在命令式编程中用来描述函数对象，这是完全不同的东西。在 PHP 中，一个具有`__invoke`方法的对象，就像我们在第一章中看到的那样，*函数作为一等公民*，就是这样一个函数对象。

然而，在函数式编程中，函子是从范畴论的数学领域中借用并改编的概念。细节对我们的目的并不那么重要；它足以说，函子是一种模式，允许我们将函数映射到一个或多个值所包含的上下文中。此外，为了使定义尽可能完整，我们的函子必须遵守一些法则，我们将在稍后描述和验证。

我们已经多次在集合上使用了 map，这使它们成为了事实上的函子。但是如果你记得的话，我们也以相同的方式命名了我们的方法来将一个函数应用于 Maybe 中包含的值。原因是函子可以被看作是具有一种方法来将函数应用于包含的值的容器。

在某种意义上，任何实现以下接口的类都可以被称为`函子`：

```php
<?php 

interface Functor 
{ 
    public function map(callable $f): Functor; 
} 
```

然而，这样描述有点简化了。一个简单的 PHP 数组也是一个函子（因为存在`array_map`函数），只要你使用`functional-php`库和它的 map 函数，任何实现`Traversable`接口的东西也是一个函子。

为什么对于一个如此简单的想法要大惊小怪？因为，尽管这个想法本身很简单，它使我们能够以不同的方式思考正在发生的事情，并可能有助于理解和重构代码。

此外，`map`函数可以做的远不止盲目地应用给定的`callable`类型，就像数组一样。如果你记得我们的`Maybe`类型实现，在值为`Nothing`的情况下，`map`函数只是简单地保持返回`Nothing`值，以便更简单地管理空值。

我们还可以想象在我们的函子中有更复杂的数据结构，比如树，其中给`map`函数的函数应用于所有节点。

函子允许我们共享一个共同的接口，我们的`map`方法或函数，对各种数据类型执行类似的操作，同时隐藏实现的复杂性。就像函数式编程一样，认知负担减少了，因为你不需要为相同的操作有多个名称。例如，"apply"、"perform"和"walk"等函数和方法名称通常用来描述相同的事情。

## 恒等函数

我们最终关注的是与这个概念相关的两个函子定律。但在介绍它们之前，我们需要稍微偏离一下，讨论一下恒等函数，通常是`id`。这是一个非常简单的函数，只是简单地返回它的参数：

```php
<?php 

function id($value) 
{ 
    return $value; 
} 
```

为什么有人需要一个做得这么少的函数？首先，我们以后会需要它来证明本章中介绍的各种抽象的定律。但现实世界中也存在应用。

例如，当你对数字进行折叠运算，比如求和，你会使用初始值`0`。`id`函数在对函数进行折叠时起着相同的作用。事实上，`functional-php`库中的 compose 函数是使用`id`函数实现的。

另一个用途可能是来自另一个库的某个函数，它执行你感兴趣的操作，但也在结果数据上调用回调。如果回调是必需的，但你不想对数据做其他任何操作，只需传递`id`，你将得到未经改变的数据。

让我们使用我们的新函数来声明我们的`compose`函数的一个属性，对于任何只接受一个参数的函数`f`：

```php
compose(id, f) == compose(f, id) 
```

这基本上是说，如果你先应用参数`id`然后是`f`，你会得到与先应用`f`然后是`id`完全相同的结果。到这一点，这对你来说应该是显而易见的。如果不是，我鼓励你重新阅读上一章，直到你清楚地理解为什么会这样。

## 函子定律

现在我们已经涵盖了我们的恒等函数，让我们回到我们的定律。它们有两个重要原因：

+   它们给了我们一组约束条件，以确保我们的函子的有效性。

+   它们允许我们进行经过验证的重构

话不多说，它们在这里：

1.  *map(id) == id*

1.  *compose(map(f), map(g)) == map(compose(f, g))*

第一定律规定，将`id`函数映射到包含的值上，与直接在函子本身上调用`id`函数完全相同。当这个定律成立时，这保证了我们的 map 函数只将给定的函数应用于数据，而不进行任何其他类型的处理。

第二定律规定，首先在我们的值上映射`f`函数，然后是`g`函数，与首先将`f`和`g`组合在一起，然后映射结果函数完全相同。知道这一点，我们可以进行各种优化。例如，我们可以将它们组合在一起，只进行一次循环，而不是对我们的数据进行三种不同方法的三次循环。

我可以想象现在对你来说并不是一切都很清楚，所以不要浪费时间试图进一步解释它们，让我们验证它们是否适用于`array_map`方法。这可能会帮助你理解它的要点；以下代码期望之前定义的`id`函数在作用域内：

```php
<?php 

$data = [1, 2, 3, 4]; 

var_dump(array_map('id', $data) === id($data)); 
// bool(true) 

function add2($a) 
{ 
    return $a + 2; 
} 

function times10($a) 
{ 
    return $a * 10; 
} 

function composed($a) { 
    return add2(times10($a)); 
} 

var_dump( 
array_map('add2', array_map('times10', $data)) === array_map('composed', $data) 
); 
// bool(true) 
```

组合是手动执行的；在我看来，在这里使用柯里化只会使事情变得更加复杂。

正如我们所看到的，`array_map`方法符合这两个定律，这是一个好迹象，因为这意味着没有隐藏的数据处理在背后进行，我们可以避免在数组上循环两次或更多次，当只需要一次时。

让我们尝试一下我们之前定义的`Maybe`类型：

```php
<?php 

$just = Maybe::just(10); 
$nothing = Maybe::nothing(); 

var_dump($just->map('id') == id($just)); 
// bool(true) 

var_dump($nothing->map('id') === id($nothing)); 
// bool(true) 
```

我们不得不切换到非严格相等的方式来处理`$just`情况，因为否则我们会得到一个错误的结果，因为 PHP 比较对象实例而不是它们的值。`Maybe`类型将结果值包装在一个新对象中，PHP 只在非严格相等的情况下执行内部值比较；上面定义的`add2`、`times10`和`composed`函数预期在范围内。

```php
<?php 

var_dump($just->map('times10')->map('add2') == $just->map('composed')); 
// bool(true) 

var_dump($nothing->map('times10')->map('add2') === $nothing->map('composed')); 
// bool(true) 
```

很好，我们的`Maybe`类型实现是一个有效的函数器。

## 身份函数器

正如我们在关于身份函数的部分讨论的那样，还存在一个身份函数器。它充当一个非常简单的函数器，除了保存值之外不对值进行任何操作：

```php
<?php 

class IdentityFunctor implements Functor 
{ 
    private $value; 

    public function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public function map(callable $f): Functor 
    { 
        return new static($f($this->value)); 
    } 

    public function get() 
    { 
        return $this->value; 
    } 
} 
```

与身份函数一样，这种函数器的用途并不立即明显。然而，思想是一样的-当您有一个函数以函数器作为参数时，您可以使用它，但不想修改您的实际值。

这应该在本书的后续章节中变得更加清晰。与此同时，我们将使用身份函数器来解释一些更高级的概念。

## 结束语

让我再次重申，函数器是一个非常简单的抽象概念，但也是一个非常强大的概念。我们只看到了其中两个，但有无数的数据结构可以非常容易地转换为函数器。

任何允许您将给定函数映射到上下文中保存的一个或多个值的函数或类都可以被视为函数器。身份函数器或数组是这种上下文的简单示例；其他示例包括我们之前讨论过的`Maybe`和`Either`类型，或者任何具有`map`方法的类，该方法允许您将函数应用于包含的值。

我无法鼓励您足够尝试实现这种映射模式，并验证无论您创建一个新的类或数据结构，这两个定律是否成立。这将使您更容易理解您的代码可以执行什么，并且您将能够使用组合进行优化，并保证您的重构是正确的。

# 应用函数器

让我们拿一个我们的身份函数器的实例，保存一些整数和一个柯里化版本的`add`函数：

```php
<?php 

$add = curry(function(int $a, int $b) { return $a + $b; }); 

$id = new IdentityFunctor(5); 
```

现在，当我们尝试在我们的函数器上映射`$add`参数时会发生什么？考虑以下代码：

```php
<?php 

$hum = $id->map($add); 

echo get_class($hum->get()); 
// Closure 
```

你可能已经猜到了，我们的函数器现在包含一个闭包，代表一个部分应用的`add`参数，其值为`5`作为第一个参数。您可以使用`get`方法检索函数并使用它，但实际上并不是很有用。

另一种可能性是映射另一个函数，以我们的函数作为参数，并对其进行操作：

```php
<?php 

$result = $hum->map(function(callable $f) { 
    return $f(10); 
}); 
echo $result->get(); 
// 15 
```

但我想我们都会同意，这并不是执行这样的操作的一种非常有效的方式。更好的方法是能够简单地将值`10`或者另一个函数器传递给`$hum`并获得相同的结果。

进入应用函数器。顾名思义，这个想法是应用函数器。更准确地说，是将函数器应用于其他函数器。在我们的情况下，我们可以将包含函数的函数器`$hum`应用于另一个包含值`10`的函数器，并获得我们想要的值`15`。

让我们创建一个扩展版本的`IdentityFunctor`类来测试我们的想法：

```php
<?php 

class IdentityFunctorExtended extends IdentityFunctor 
{ 
    public function apply(IdentityFunctorExtended $f) 
    { 
        return $f->map($this->get()); 
    } 
} 

$applicative = (new IdentityFunctorExtended(5))->map($add); 
$ten = new IdentityFunctorExtended(10); 
echo $applicative->apply($ten)->get(); 
// 15 
```

甚至可以创建一个只包含函数的`Applicative`类，并在之后应用这些值：

```php
<?php 

$five = new IdentityFunctorExtended(5); 
$ten = new IdentityFunctorExtended(10); 
$applicative = new IdentityFunctorExtended($add); 

echo $applicative->apply($five)->apply($ten)->get(); 
// 15 
```

## 应用抽象

现在我们能够使用我们的`IdentifyFunctor`类作为柯里化函数的持有者。如果我们能够将这个想法抽象出来，并在`Functor`类的基础上创建一些东西会怎样？

```php
<?php 

abstract class Applicative implements Functor 
{ 
    public abstract static function pure($value): Applicative; 
    public abstract function apply(Applicative $f): Applicative; 
    public function map(callable $f): Functor 
    { 
        return $this->pure($f)->apply($this); 
    } 
} 
```

正如你所看到的，我们创建了一个新的抽象类而不是一个接口。原因是因为我们可以使用`pure`和`apply`方法来实现`map`函数，所以强制每个想要创建`Applicative`类的人都要实现它是没有意义的。

`pure`函数之所以被称为如此，是因为`Applicative`类中存储的任何东西都被认为是纯的，因为没有办法直接修改它。这个术语来自 Haskell 实现。其他实现有时使用名称*unit*。pure 用于从任何`callable`创建一个新的 applicative。

`apply`函数将存储的函数应用于给定的参数。参数必须是相同类型的，以便实现知道如何访问内部值。遗憾的是，PHP 类型系统不允许我们强制执行这个规则，我们必须默认为`Applicative`。

我们对 map 的定义也有同样的问题，必须将返回类型保持为`Functor`。我们需要这样做，因为 PHP 类型引擎不支持一种称为**返回类型协变**的特性。如果支持的话，我们可以指定一个更专门的类型（即子类型）作为返回值。

`map`函数是使用上述函数实现的。首先我们使用`pure`方法封装我们的`callable`，然后将这个新的 applicative 应用于实际值。没有什么特别的。

让我们测试我们的实现：

```php
<?php 

$five = IdentityApplicative::pure(5); 
$ten = IdentityApplicative::pure(10); 
$applicative = IdentityApplicative::pure($add); 

echo $applicative->apply($five)->apply($ten)->get(); 
// 15 

$hello = IdentityApplicative::pure('Hello world!'); 

echo IdentityApplicative::pure('strtoupper')->apply($hello)->get(); 
// HELLO WORLD! echo $hello->map('strtoupper')->get(); 
// HELLO WORLD! 
```

一切似乎都运行正常。我们甚至能够验证我们的 map 实现似乎是正确的。

与 functor 一样，我们可以创建最简单的`Applicative`类抽象：

```php
<?php 

class IdentityApplicative extends Applicative 
{ 
    private $value; 

    protected function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public static function pure($value): Applicative 
    { 
        return new static($value); 
    } 

    public function apply(Applicative $f): Applicative 
    { 
        return static::pure($this->get()($f->get())); 
    } 

    public function get() 
    { 
        return $this->value; 
    } 
} 
```

## Applicative 法则

applicative 的第一个重要属性是它们是*封闭的组合*，意味着 applicative 将返回相同类型的新 applicative。此外，apply 方法接受自己类型的 applicative。我们无法使用 PHP 类型系统来强制执行这一点，所以你需要小心，否则可能会在某个时候出现问题。

还需要遵守以下规则才能拥有一个正确的 applicative functor。我们将首先详细介绍它们，然后稍后验证它们对我们的`IdentityApplicative`类是否成立。

### 映射

*纯（f）->应用 == map（f）*

使用 applicative 应用函数与对其进行映射是相同的。这个法则简单地告诉我们，我们可以在以前使用 functor 的任何地方使用 applicative。切换到 applicative 不会使我们失去任何权力。

实际上，这并不是一个法则，因为它可以从以下四个法则中推导出来。但由于这并不明显，为了让事情更清晰，让我们来陈述一下。

### 身份

*纯（id）->应用（$x）== id（$x）*

应用恒等函数不会改变值。与 functor 的身份法则一样，这确保`apply`方法除了应用函数之外不会发生任何隐藏的转换。

### 同态

*纯（f）->应用（$x）==纯（f（$x））*

创建一个 applicative functor 并将其应用于一个值与首先在值上调用函数，然后在 functor 中封装它具有相同的效果。

这是一个重要的法则，因为我们深入研究 applicative 的第一个动机是使用柯里化函数而不是一元函数。这个法则确保我们可以在任何阶段创建我们的 applicative，而不需要立即封装我们的函数。

### 交换

*纯（f）->应用（$x）==纯（function（$f）{ $f（$x）; }）->应用（f）*

这个有点棘手。它声明对值应用函数与创建一个提升值的 applicative functor 并将其应用于函数是相同的。在这种情况下，提升值是围绕该值的闭包，它将在其上调用给定的函数。该法则确保纯函数除了封装给定值之外不执行任何修改。

### 组合

*纯（组合）->应用（f1）->应用（f2）->应用（$x）==纯（f1）->应用（纯（f2）->应用（$x））*

这种法律的简化版本可以用*pure(compose(f1, f2))->apply($x)*来写在左边。它简单地陈述了 functors 的组合法则，即你可以将两个函数的组合版本应用到你的值上，或者分别调用它们。这确保你可以对 functors 执行相同的优化。

### 验证法律是否成立

正如我们对 functors 所看到的，强烈建议测试你的实现是否符合所有法律。这可能是一个非常乏味的过程，特别是如果你有四个。因此，我们不要手动执行检查，让我们写一个辅助程序：

```php
<?php 

function check_applicative_laws(Applicative $f1, callable $f2, $x) 
{ 
    $identity = function($x) { return $x; }; 
    $compose = function(callable $a) { 
        return function(callable $b) use($a) { 
            return function($x) use($a, $b) { 
                return $a($b($x)); 
            }; 
        }; 
    }; 

    $pure_x = $f1->pure($x); 
    $pure_f2 = $f1->pure($f2); 

    return [ 
        'identity' => 
            $f1->pure($identity)->apply($pure_x) == 
            $pure_x, 
        'homomorphism' => 
            $f1->pure($f2)->apply($pure_x) == 
            $f1->pure($f2($x)), 
        'interchange' => 
            $f1->apply($pure_x) == 
            $f1->pure(function($f) use($x) { return $f($x); })->apply($f1), 
        'composition' => 
            $f1->pure($compose)->apply($f1)->apply($pure_f2)->apply($pure_x) == 
            $f1->apply($pure_f2->apply($pure_x)), 
        'map' => 
            $pure_f2->apply($pure_x) == 
            $pure_x->map($f2) 
    ]; 
} 
```

`identity`和`compose`函数在辅助程序中声明，因此它是完全自包含的，你可以在各种情况下使用它。此外，`functional-php`库中的`compose`函数不适用，因为它不是柯里化的，它接受可变数量的参数。

此外，为了避免有很多争论，我们使用`Applicative`类的一个实例，这样我们就可以有一个第一个函数和要检查的类型，然后是一个`callable`和一个将被提升到 applicative 并在必要时使用的值。

这种选择限制了我们可以使用的函数，因为值必须与两个函数的参数类型匹配；第一个函数还必须返回相同类型的参数。如果这对你来说太过约束，你可以决定扩展辅助程序，以接受另外两个参数，第二个 applicative 和一个提升的值，并在必要时使用它们。

让我们验证我们的`IdentityApplicative`类：

```php
<?php 

print_r(check_applicative_laws( 
IdentityApplicative::pure('strtoupper'), 
    'trim', 
    ' Hello World! ' 
)); 
// Array 
// ( 
//     [identity] => 1 
//     [homomorphism] => 1 
//     [interchange] => 1 
//     [composition] => 1 
//     [map] => 1 
// ) 
```

很好，一切似乎都很好。如果你想使用这个辅助程序，你需要选择兼容的函数，因为你可能会遇到一些缺乏清晰度的错误消息，因为我们无法确保第一个函数的返回值类型与第二个函数的第一个参数类型匹配。

由于这种自动检查可以极大地帮助，让我们迅速地为 functors 编写相同类型的函数：

```php
<?php 

function check_functor_laws(Functor $func, callable $f, callable $g) 
{ 
    $id = function($a) { return $a; }; 
    $composed = function($a) use($f, $g) { return $g($f($a)); }; 

    return [ 
        'identity' => $func->map($id) == $id($func), 
        'composition' => $func->map($f)->map($g) == $func->map($composed) 
    ]; 
} 
```

并检查我们从未测试过的`IdentityFunctor`：

```php
<?php 

print_r(check_functor_laws( 
    new IdentityFunctor(10), 
    function($a) { return $a * 10; }, 
    function($a) { return $a + 2; } 
)); 
// Array 
// ( 
//     [identity] => 1 
//     [composition] => 1 
// ) 
```

好的，一切都很好。

## 使用 applicatives

正如我们已经看到的，数组是 functors，因为它们有一个`map`函数。但是一个集合也很容易成为 applicative。让我们实现一个`CollectionApplicative`类：

```php
<?php 

class CollectionApplicative extends Applicative implements IteratorAggregate 
{ 
    private $values; 

    protected function __construct($values) 
    { 
        $this->values = $values; 
    } 

    public static function pure($values): Applicative 
    { 
        if($values instanceof Traversable) { 
            $values = iterator_to_array($values); 
        } else if(! is_array($values)) { 
            $values = [$values]; 
        } 

        return new static($values); 
    } 

    public function apply(Applicative $data): Applicative 
    { 
        return $this->pure(array_reduce($this->values, 
            function($acc, callable $function) use($data) { 
                return array_merge($acc, array_map($function, $data->values) ); 
            }, []) 
        ); 
    } 

    public function getIterator() { 
        return new ArrayIterator($this->values); 
    } 
} 
```

正如你所看到的，这一切都相当容易。为了简化我们的生活，我们只需将不是集合的任何东西包装在一个数组中，并将`Traversable`接口的实例转换为真正的数组。这段代码显然需要一些改进才能用于生产，但对于我们的小演示来说已经足够了：

```php
<?php 

print_r(iterator_to_array(CollectionApplicative::pure([ 
  function($a) { return $a * 2; }, 
  function($a) { return $a + 10; } 
])->apply(CollectionApplicative::pure([1, 2, 3])))); 
// Array 
// ( 
//     [0] => 2 
//     [1] => 4 
//     [2] => 6 
//     [3] => 11 
//     [4] => 12 
//     [5] => 13 
// ) 
```

这里发生了什么？我们的 applicative 中有一个函数列表，我们将其应用到一个数字列表。结果是一个新的列表，每个函数都应用到每个数字上。

这个小例子并不是真正有用的，但这个想法可以应用到任何事情上。想象一下，你有一种图库应用，用户可以上传一些图像。你还有各种处理你想对这些图像进行的处理：

+   限制最终图像的大小，因为用户倾向于上传过大的图像

+   为索引页面创建一个缩略图

+   为移动设备创建一个小版本

你唯一需要做的就是创建一个包含所有函数的数组，一个包含上传图像的数组，并将我们刚刚对数字做的相同模式应用到它们。然后你可以使用`functional-php`库中的 group 函数将你的图像重新分组在一起：

```php
<?php 

use function Functional\group; 

function limit_size($image) { return $image; } 
function thumbnail($image) { return $image.'_tn'; } 
function mobile($image) { return $image.'_small'; } 

$images = CollectionApplicative::pure(['one', 'two', 'three']); 

$process = CollectionApplicative::pure([ 
  'limit_size', 'thumbnail', 'mobile' 
]); 

$transformed = group($process->apply($images), function($image, $index) { 
    return $index % 3; 
}); 
```

我们使用转换后的数组中的索引来将图像重新分组。每三个图像是限制的，每四个是缩略图，最后是移动版本。结果如下所示：

```php
<?php 

print_r($transformed); 
// Array 
// ( 
//     [0] => Array 
//         ( 
//             [0] => one 
//             [3] =>one_tn 
//             [6] =>one_small 
//         ) 
// 
//     [1] => Array 
//         ( 
//             [1] => two 
//             [4] =>two_tn 
//             [7] =>two_small 
//         ) 
// 
//     [2] => Array 
//         ( 
//             [2] => three 
//             [5] =>three_tn 
//             [8] =>three_small 
//         ) 
// 
//) 
```

在这个阶段，你可能会渴望更多，但你需要耐心。让我们先完成本章的理论，我们很快就会在下一章看到更有力的例子。

# 单子

现在我们对应用函子有了一定的了解，在谈论单子之前，我们需要在这个谜题中增加最后一块，即单子。再次，这个概念来自范畴论的数学领域。

**单子**是任何类型和该类型上的二元操作的组合，具有关联的身份元素。例如，以下是一些组合，您可能从未预料到它们是单子：

+   整数和加法操作，其身份是 0，因为*$i + 0 == $i*

+   整数和乘法操作，其身份是 1，因为*$i * 1 == $i*

+   数组和合并操作，其身份是空数组，因为*array_merge($a, []) == $a*

+   字符串和连接操作，其身份是空字符串，因为*$s . '' == $s*

在本章的其余部分，让我们称我们的操作为*op*，身份元素为*id*。`op`调用来自操作或操作员，并在多种语言的`Monoid`实现中使用。Haskell 使用术语**mempty**和**mappend**以避免与其他函数名称冲突。有时使用零代替*id*或身份。

单子还必须遵守一定数量的法则，确切地说是两个。

## 身份法则

*$a op id == id op $a == $a*

第一个法则确保了身份可以在操作符的两侧使用。身份元素只有在作为操作符的右手或左手侧应用时才能起作用。例如，对矩阵的操作就是这种情况。在这种情况下，我们谈论左和右身份元素。在`Monoid`的情况下，我们需要一个双侧身份，或者简单地说是身份。

对于大多数身份法则，验证`Monoid`实现可以确保我们正确应用操作符而没有其他副作用。

## 结合律

*($a op $b) op $c == $a op ($b op $c)*

这项法律保证了我们可以按任何顺序重新组合我们对操作员的呼叫，只要其他一些操作没有交错。这很重要，因为它允许我们推理可能的优化，并确保结果是相同的。

知道一系列操作是可结合的；您还可以将序列分成多个部分，将计算分布到多个线程、核心或计算机上，当所有中间结果出现时，将它们之间的操作应用以获得最终结果。

## 验证法则

让我们验证一下我们之前谈到的单子的法则。首先是整数加法：

```php
<?php 

$a = 10; $b = 20; $c = 30; 

var_dump($a + 0 === $a); 
// bool(true) 
var_dump(0 + $a === $a); 
// bool(true) 
var_dump(($a + $b) + $c === $a + ($b + $c)); 
// bool(true) 
```

然后，整数乘法：

```php
<?php 

var_dump($a * 1 === $a); 
// bool(true) 
var_dump(1 * $a === $a); 
// bool(true) 
var_dump(($a * $b) * $c === $a * ($b * $c)); 
// bool(true) 
```

然后数组合并如下：

```php
<?php 

$v1 = [1, 2, 3]; $v2 = [5]; $v3 = [10]; 

var_dump(array_merge($v1, []) === $v1); 
// bool(true) 
var_dump(array_merge([], $v1) === $v1); 
// bool(true) 
var_dump( 
array_merge(array_merge($v1, $v2), $v3) === 
array_merge($v1, array_merge($v2, $v3)) 
); 
// bool(true) 
```

最后，字符串连接：

```php
<?php 

$s1 = "Hello"; $s2 = " World"; $s3 = "!"; 

var_dump($s1 . '' === $s1); 
// bool(true) 
var_dump('' . $s1 === $s1); 
// bool(true) 
var_dump(($s1 . $s2) . $s3 == $s1 . ($s2 . $s3)); 
// bool(true) 
```

很好，我们所有的单子都遵守这两个法则。

减法或除法呢？它们也是单子吗？很明显，0 是减法的身份，1 是除法的身份，但结合性呢？

考虑以下检查减法或除法的结合性：

```php
<?php

var_dump(($a - $b) - $c === $a - ($b - $c));
// bool(false)
var_dump(($a / $b) / $c === $a / ($b / $c));
// bool(false) 
```

我们清楚地看到，减法和除法都不是可结合的。在处理这种抽象时，始终重要的是使用法则来测试我们的假设。否则，重构或调用某个期望`Monoid`的函数可能会出现严重问题。显然，对于函子和应用函子也是如此。

## 单子有什么用？

老实说，单子本身并不是真正有用的，特别是在 PHP 中。最终，在一种语言中，您可以声明新的操作符或重新定义现有的操作符，您可以确保它们的结合性和存在单子。但即使如此，也没有真正的优势。

另外，如果语言可以自动分配使用`Monoid`的运算，那将是加快漫长计算的一个很好的方法。但我不知道任何语言，即使是学术语言，目前都能做到这一点。一些语言执行操作重新排序以提高效率，但仅此而已。显然，PHP 不能做任何这些，因为幺半群的概念不在核心中。

那么为什么要费心呢？因为幺半群可以与高阶函数和一些我们将在后面发现的构造一起使用，以充分利用它们的法律。此外，由于 PHP 不允许我们像 Haskell 那样使用现有的运算符作为函数，例如，我们之前不得不定义`add`之类的函数。相反，我们可以定义一个`Monoid`类。它将具有与我们的简单函数相同的效用，并添加一些很好的属性。

冒昧地说，明确声明一个操作是幺半群可以减轻认知负担。使用幺半群时，您可以确保操作是可结合的，并且遵守双边单位。

## 一个幺半群的实现

PHP 不支持泛型，因此我们无法正式地编码我们的`Monoid`的类型信息。您将不得不选择一个不言自明的名称或者清楚地记录这是什么类型。

另外，由于我们希望我们的实现能够替换诸如`add`之类的函数，我们需要在我们的类上添加一些额外的方法来允许这种用法。让我们看看我们能做些什么：

```php
<?php 

abstract class Monoid 
{ 
    public abstract static function id(); 
    public abstract static function op($a, $b); 

    public static function concat(array $values) 
    { 
        $class = get_called_class(); 
        return array_reduce($values, [$class, 'op'], [$class, 'id']()); 
    } 

    public function __invoke(...$args) 
    { 
        switch(count($args)) { 
            case 0: throw new RuntimeException("Except at least 1 parameter"); 
            case 1: 
                return function($b) use($args) { 
                    return static::op($args[0], $b); 
                }; 
            default: 
                return static::concat($args); 
        } 
    } 
} 
```

显然，我们的`id`和`op`函数声明为抽象，因为它们将是我们每个幺半群的特定部分。

拥有`Monoid`的一个主要优势是可以轻松地折叠具有`Monoid`类类型的值的集合。这就是为什么我们创建`concat`方法作为一个辅助方法来做到这一点。

最后，我们有一个`__invoke`函数，以便我们的`Monoid`可以像普通函数一样使用。该函数以一种特定的方式进行柯里化。如果您在第一次调用时传递了多个参数，`concat`方法将被用于立即返回结果。否则，只有一个参数，您将得到一个等待第二个参数的新函数。

既然我们在这里，让我们编写一个检查法律的函数：

```php
<?php 

function check_monoid_laws(Monoid $m, $a, $b, $c) 
{ 
    return [ 
        'left identity' => $m->op($m->id(), $a) == $a, 
        'right identity' => $m->op($a, $m->id()) == $a, 
        'associativity' => 
            $m->op($m->op($a, $b), $c) == 
            $m->op($a, $m->op($b, $c)) 
    ]; 
} 
```

## 我们的第一个幺半群

让我们为我们之前看到的情况创建幺半群，并演示我们如何使用它们：

```php
<?php 

class IntSum extends Monoid 
{ 
    public static function id() { return 0; } 
    public static function op($a, $b) { return $a + $b; } 
} 

class IntProduct extends Monoid 
{ 
    public static function id() { return 1; } 
    public static function op($a, $b) { return $a * $b; } 
} 

class StringConcat extends Monoid 
{ 
    public static function id() { return ''; } 
    public static function op($a, $b) { return $a.$b; } 
} 

class ArrayMerge extends Monoid 
{ 
    public static function id() { return []; } 
    public static function op($a, $b) { return array_merge($a, $b); } 
} 
```

让我们验证它们的法律：

```php
<?php 

print_r(check_monoid_laws(new IntSum(), 5, 10, 20)); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 

print_r(check_monoid_laws(new IntProduct(), 5, 10, 20)); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 

print_r(check_monoid_laws(new StringConcat(), "Hello ", "World", "!")); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 

print_r(check_monoid_laws(new ArrayMerge(), [1, 2, 3], [4, 5], [10])); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 
```

举个例子，让我们尝试创建一个减法的幺半群并检查法律：

```php
<?php 

class IntSubtraction extends Monoid 
{ 
    public static function id() { return 0; } 
    public static function op($a, $b) { return $a - $b; } 
} 

print_r(check_monoid_laws(new IntSubtraction(), 5, 10, 20)); 
// Array 
// ( 
//     [left identity] => 
//     [right identity] => 1 
//     [associativity] => 
// ) 
```

如预期的那样，结合律失败了。我们还有一个左单位的问题，因为 *0 - $a == -$a*。所以让我们不要忘记测试我们的幺半群，以确保它们是正确的。

关于布尔类型，可以创建两个有趣的幺半群：

```php
<?php 

class Any extends Monoid 
{ 
    public static function id() { return false; } 
    public static function op($a, $b) { return $a || $b; } 
} 

class All extends Monoid 
{ 
    public static function id() { return true; } 
    public static function op($a, $b) { return $a && $b; } 
} 

print_r(check_monoid_laws(new Any(), true, false, true)); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 

print_r(check_monoid_laws(new All(), true, false, true)); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 
```

这两个幺半群使我们能够验证是否至少满足一个条件或所有条件。这些是`functional-php`库中`every`和`some`函数的幺半群版本。这两个幺半群与求和和乘积的作用相同，因为 PHP 不允许我们将布尔运算符用作函数：

```php
<?php 

echo Any::concat([true, false, true, false]) ? 'true' : 'false'; 
// true 

echo All::concat([true, false, true, false]) ? 'true' : 'false'; 
// false 
```

当您需要以编程方式创建一系列条件时，它们可能会很有用。只需将它们提供给`Monoid`，而不是迭代所有条件来生成结果。您还可以编写一个*none*幺半群作为练习，以查看您是否理解了这个概念。

## 使用幺半群

使用我们的新幺半群最明显的方法之一是折叠一组值：

```php
<?php 

$numbers = [1, 23, 45, 187, 12]; 
echo IntSum::concat($numbers); 
// 268 

$words = ['Hello ', ', ', 'my ', 'name is John.']; 
echo StringConcat::concat($words); 
// Hello , my name is John. $arrays = [[1, 2, 3], ['one', 'two', 'three'], [true, false]]; 
print_r(ArrayMerge::concat($arrays)); 
// [1, 2, 3, 'one', 'two', 'three', true, false] 
```

这个属性非常有趣，以至于大多数函数式编程语言都实现了`Foldable`类型的想法。这样的类型需要有一个关联的幺半群。借助我们刚刚看到的属性，该类型可以很容易地折叠。然而，将这个想法移植到 PHP 是困难的，因为我们将缺少改进使用`concat`方法所需的语法糖。

您还可以将它们用作`callable`类型，并将它们传递给高阶函数：

```php
<?php 

use function Functional\compose; 

$add = new IntSum(); 
$times = new IntProduct(); 

$composed = compose($add(5), $times(2)); 
echo $composed(2); 
// 14 
```

显然，这不仅限于 compose 函数。您可以重写本书中使用`add`函数的所有先前示例，并使用我们的新`Monoid`代替。

随着我们在本书中的进展，我们将看到更多与我们尚未发现的功能技术相关联的单子的使用方式。

# 单子

我们开始学习函子，它是一组可以映射的值。然后我们介绍了应用函子的概念，它允许我们将这些值放入特定的上下文并对它们应用函数，同时保留上下文。我们还快速讨论了幺半群及其属性。

有了所有这些先前的知识，我们终于准备好开始单子的概念了。正如 James Iry 在*编程语言简史*中幽默地指出的那样：

> *单子是自函子范畴中的幺半群，有什么问题吗？*

这句虚构的引语归功于 Philip Wadler，他是 Haskell 规范的最初参与者之一，也是单子使用的倡导者，可以在[`james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html`](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)找到其上下文。

如果没有一些范畴论的知识，很难清楚地解释这句话到底是什么意思，特别是因为它是虚构的，故意模糊以至于有趣。可以说，单子类似于幺半群，因为它们大致共享相同的法则集。此外，它们直接与函子和应用相关联。

单子，就像函子一样，充当某种值的容器。此外，就像应用程序一样，您可以将函数应用于封装的值。这三种模式都是将一些数据放入上下文的一种方式。但是，两者之间有一些区别：

+   应用封装了一个函数。单子和函子封装了一个值。

+   应用程序使用返回非提升值的函数。单子使用返回相同类型的单子的函数。

由于函数也是有效值，这并不意味着两者不兼容，只是意味着我们需要为我们的单子类型定义一个新的 API。但是，我们可以自由地扩展 Applicative，因为它在单子上下文中包含完全有效的方法：

```php
<?php 

abstract class Monad extends Applicative 
{ 
    public static function return($value): Monad 
    { 
        return static::pure($value); 
    } 

    public abstract function bind(callable $f): Monad; 
} 
```

我们的实现非常简单。我们将 pure 与 Haskell 中的 return 别名，这样人们就不会迷失。请注意，它与您习惯的 return 关键字无关；它只是将值放入单子的上下文中。我们还定义了一个新的绑定函数，它以`callable`类型作为参数。

由于我们不知道内部值将如何存储，并且由于 PHP 类型系统的限制，我们无法实现`apply`或`bind`函数，尽管它们应该是非常相似的：

+   `apply`方法接受一个包装在`Applicative`类中的值，并将存储的函数应用于它

+   `bind`方法接受一个函数并将其应用于存储的值

两者之间的区别在于`bind`方法需要直接返回值，而`apply`方法首先再次使用`pure`或`return`函数包装值。

正如您可能已经了解的那样，使用不同语言的人倾向于以稍有不同的方式命名事物。这就是为什么有时您会看到`bind`方法被称为**chain**或**flatMap**，这取决于您正在查看的实现。

## 单子定律

你现在知道了；单子必须遵守一些法则才能被称为单子。这些法则与幺半群的法则相同-单位元和结合律。因此，单子的所有有用属性也适用于单子。

然而，正如你将看到的，我们将描述的法则似乎与我们之前为单子看到的幂等性和结合性法则没有任何共同之处。这与我们定义的`bind`和`return`函数的方式有关。使用一种叫做**Kleisli**组合操作符，我们可以转换这些法则，使它们看起来有点像我们之前看到的那些。然而，这有点复杂，对我们的目的毫无用处。如果你想了解更多，我可以引导你到[`wiki.haskell.org/Monad_laws`](https://wiki.haskell.org/Monad_laws)。

### 左单位元

*return(x)->bind(f) == f(x)*

这个法则规定，如果你取一个值，将其包装在单子的上下文中，并将其绑定到*f*，结果必须与直接在值上调用函数的结果相同。它确保`bind`方法对函数和值除了应用之外没有任何副作用。

这只有在`bind`方法不像`apply`方法那样再次将函数的返回值包装在单子内时才成立。这是函数的工作。

### 右单位元

*m->bind(return) == m*

这个法则规定，如果你将返回的值绑定到一个单子，你将得到你的单子。它确保`return`除了将值放入单子的上下文之外没有其他影响。

### 结合性

*m->bind(f)->bind(g) == m->bind((function($x) { f($x)->bind(g); })*

这些法则规定，你可以先将单子内的值绑定到*f*，然后再绑定到*g*，或者你可以将其绑定到第一个函数与第二个函数的组合。我们需要一个中间函数来模拟这一点，就像我们在 applicatives 的交换法则中需要一个中间函数一样。

这个法则允许我们获得与之前的结合性和组合性法则相同的好处。这种形式有点奇怪，因为单子保存的是值，而不是函数或操作符。

### 验证我们的单子

让我们写一个函数来检查单子的有效性：

```php
<?php 

function check_monad_laws($x, Monad $m, callable $f, callable $g) 
{ 
    return [ 
        'left identity' => $m->return($x)->bind($f) == $f($x), 
        'right identity' => $m->bind([$m, 'return']) == $m, 
        'associativity' => 
            $m->bind($f)->bind($g) ==             $m->bind(function($x) use($f, $g) { return $f($x)->bind($g); }), 
    ]; 
} 
```

我们还需要一个单位单子：

```php
class IdentityMonad extends Monad 
{ 
    private $value; 

    private function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public static function pure($value): Applicative 
    { 
        return new static($value); 
    } 

    public function get() 
    { 
        return $this->value; 
    } 

    public function bind(callable $f): Monad 
    { 
        return $f($this->get()); 
    } 

    public function apply(Applicative $a): Applicative 
    { 
        return static::pure($this->get()($a->get())); 
    } 
} 
```

最后我们可以验证一切是否成立：

```php
<?php 

print_r(check_monad_laws( 
    10, 
IdentityMonad::return(20), 
    function(int $a) { return IdentityMonad::return($a + 10); }, 
    function(int $a) { return IdentityMonad::return($a * 2); } 
)); 
// Array 
// ( 
//     [left identity] => 1 
//     [right identity] => 1 
//     [associativity] => 1 
// ) 
```

## 为什么要使用单子？

第一个原因是实际的。当你使用 applicative 应用一个函数时，结果会自动放入 applicative 的上下文中。这意味着，如果你有一个返回 applicative 的函数，并应用它，结果将是一个 applicative 内部的 applicative。任何看过电影《盗梦空间》的人都知道，把东西放在东西里面并不总是一个好主意。

单子是一种避免这种不必要嵌套的方式。`bind`函数将封装返回值的任务委托给函数，这意味着你只会有一层深度。

单子也是一种执行流程控制的方式。正如我们所见，函数式程序员倾向于避免使用循环或任何其他类型的控制流，比如使你的代码更难以理解的`if`条件。单子是一种强大的方式，以一种非常表达性的方式来顺序转换，同时保持你的代码整洁。

像 Haskell 这样的语言还有特定的语法糖来处理单子，比如`do`符号，这使得你的代码更容易阅读。一些人尝试在 PHP 中实现这样的东西，但在我看来并没有取得太大的成功。

然而，要真正理解单子抽象的力量，你必须看一些具体的实现，就像我们将在下一章中所做的那样。它们将允许我们以纯函数的方式执行*IO*操作，将日志消息从一个函数传递到另一个函数，甚至使用纯函数计算随机数。

## 关于单子的另一种看法

我们决定实现我们的`Monad`类，将`apply`和`bind`方法都留为抽象的。我们别无选择，因为值在`Monad`类内部的存储方式将只在`child`类中决定。

然而，正如我们已经说过的，`bind`方法有时在 Scala 中被称为 flatMap。顾名思义，这只是 map 和一个叫做`flatten`的函数的组合。

你明白我要说什么了吗？还记得嵌套应用的问题吗？我们可以添加一个`flatten`函数，或者像 Haskell 称呼的那样，将它作为`Monad`类的方法，而不是将`bind`作为一个抽象方法，我们可以使用`map`和我们的新方法来实现它。

我们仍然需要实现两种方法，但是两者不再做大致相同的工作，调用一个带有值的函数，一个将继续执行，另一个将负责解除`Monad`实例的嵌套。

因此，这样的函数对外部世界的用途有限，我决定使用所提供的实现。使用`flatten`函数进行实现是一个不错的练习，您可以尝试解决以更好地理解单子的工作原理。

## 一个快速的单子示例

想象一下，我们需要使用`read_file`函数读取文件的内容，然后使用`post`函数将其发送到**webservice**。我们将创建两个版本的上传函数来实现这一点：

+   第一个版本将使用传统函数，在出现错误的情况下返回布尔值`false`。

+   功能版本将假定返回`Either`单子实例的柯里化函数。我们将在下一章中进一步描述这个单子；让我们假设它的工作原理与我们之前看到的`Either`类型相同。

在成功的情况下，必须调用给定的回调函数，并返回`post`方法返回的状态码：

```php
<?php 

function upload(string $path, callable $f) { 
    $content = read_file(filename); 
    if($content === false) { 
        return false; 
    } 

    $status = post('/uploads', $content); 
    if($status === false) { 
        return $false; 
    } 

    return $f($status); 
} 
```

现在是功能版本，如下所示：

```php
<?php 

function upload_fp(string $path, callable $f) { 
    return Either::pure($path) 
      ->bind('read_file') 
      ->bind(post('/uploads')) 
      ->bind($f); 
} 
```

我不知道你更喜欢哪一个，但我的选择很明确。使用`Either`而不是`Maybe`的选择也不是无辜的。这意味着在出现错误的情况下，功能版本还可以返回详细的错误消息，而不仅仅是`false`。

# 进一步阅读

如果在完成本章后，您仍然感到有些迷茫，因为这是一个如此重要的话题，不要犹豫阅读以下文章或您自己找到的其他文章：

+   PHP 中关于单子的简要介绍，还有一个相关的库，网址是[`blog.ircmaxell.com/2013/07/taking-monads-to-oop-php.html`](http://blog.ircmaxell.com/2013/07/taking-monads-to-oop-php.html)。

+   Scala 的一个很好的介绍，任何写过一些 Java 的人都应该能理解，网址是[`medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534`](https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534)。

+   一个更数学化的视频，网址是[`channel9.msdn.com/Shows/Going+Deep/Brian-Beckman-Dont-fear-the-Monads`](https://channel9.msdn.com/Shows/Going+Deep/Brian-Beckman-Dont-fear-the-Monads)。

+   一个关于单子的幽默 JavaScript 教程。你可能会喜欢也可能会讨厌这种风格。如果你精通 JavaScript，我只能建议你阅读整本书：[`drboolean.gitbooks.io/mostly-adequate-guide/content/ch9.html`](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch9.html)。

+   一个关于单子的非常完整，尽管有些困难的介绍。需要一些基本的 Haskell 知识才能理解[`wiki.haskell.org/All_About_Monads`](https://wiki.haskell.org/All_About_Monads)中的解释。

# 总结

这一章肯定是一个艰深的话题，但不要害怕，这是最后一个。从现在开始，我们将处理更多实际的主题和真实的应用。第六章，“真实的单子”将介绍我们刚刚学到的抽象的一些有用用途。

抽象，如函子，应用和单子，是函数世界的设计模式。它们是高级抽象，可以在许多不同的地方找到，您需要一些时间才能辨别它们。但是，一旦您对它们有了感觉，您可能会意识到它们无处不在，并且这将极大地帮助您思考如何操纵数据。

我们抽象的法则确实很普遍。在编写代码时，您可能已经在不知不觉中假设了它们。能够识别我们学到的模式将使您在重构或编写算法时更加自信，因为您的直觉总是会得到事实的支持。

如果您想玩玩本章的概念，我只能建议您开始使用我们在第三章中介绍的`functional-php`库进行实验。它包含许多定义各种代数结构的接口，这是数学家给予函子、单子等的花哨名称。一些方法名称可能不完全与我们使用的名称相同，但您应该能够理解它们背后的思想。由于库的名称有点难以找到，这里再次提供链接，[`github.com/widmogrod/php-functional`](https://github.com/widmogrod/php-functional)。
