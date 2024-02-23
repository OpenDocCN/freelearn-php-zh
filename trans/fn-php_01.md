# 第一章。PHP 中的函数作为一等公民

函数式编程，顾名思义，围绕函数展开。为了有效地应用函数式技术，语言必须支持函数作为一等公民，也称为**第一类函数**。

这意味着函数被视为任何其他值。它们可以被创建并作为参数传递给其他函数，并且它们可以作为返回值使用。幸运的是，PHP 就是这样一种语言。本章将演示函数可以被创建和使用的各种方式。

在这一章中，我们将涵盖以下主题：

+   声明函数和方法

+   标量类型提示

+   匿名函数

+   闭包

+   将对象用作函数

+   高阶函数

+   可调用类型提示

# 开始之前

由于 PHP 7 的首次发布发生在 2015 年 12 月，因此本书中的示例将使用该版本。

然而，由于这是一个相当新的版本，每次使用新功能时，都将清楚地概述和解释。此外，由于并非每个人都能立即迁移，我们将尽可能提出在 PHP 5 上运行代码所需的更改。

撰写时可用的最新版本是 7.0.9。所有代码和示例都经过了这个版本的验证。

## 编码标准

本书中的示例将遵守**PSR**-**2**（**PHP 标准推荐 2**）及其父推荐标准 PSR-1 的编码风格，大多数所介绍的库也应该如此。对于不熟悉它们的人，以下是最重要的部分：

+   类位于命名空间中，使用首字母大写的驼峰命名法

+   方法使用驼峰命名法，首字母不大写

+   常量使用全大写字母书写

+   类和方法的大括号在新行上，其他大括号在同一行上

此外，尽管没有在 PSR-2 中定义，但做出了以下选择：

+   函数名称使用蛇形命名法

+   参数、变量和属性名称使用蛇形命名法

+   属性尽可能是私有的

## 自动加载和 Composer

示例还将假定存在一个符合 PSR-4 的自动加载器。

由于我们将使用 Composer 依赖管理器来安装所介绍的库，我们建议将其用作自动加载器。

# 函数和方法

尽管本书不是为 PHP 初学者设计的，但我们将快速介绍一些基础知识，以确保我们共享一个共同的词汇。

在 PHP 中，通常使用`function`关键字声明函数：

```php
<?php 

function my_function($parameter, $second_parameter) 
{ 
    // [...] 
} 
```

在类内声明的函数称为**方法**。它与传统函数不同，因为它可以访问对象属性，具有可见性修饰符，并且可以声明为静态的。由于我们将尽量编写尽可能纯净的代码，我们的属性将是`private`类型：

```php
<?php 

class SomeClass 
{ 
   private $some_property; 

   // a public function 
   public function some_function() 
   { 
       // [...] 
   } 

   // a protected static function 
   static protected function other_function() 
   { 
       // [...] 
   } 
} 
```

# PHP 7 标量类型提示

在 PHP 5 中，您已经可以为类、可调用函数和数组声明类型提示。PHP 7 引入了标量类型提示的概念。这意味着您现在可以声明您想要`string`、`int`、`float`或`bool`数据类型，无论是参数还是返回类型。语法与其他语言中的语法大致相似。

与类类型提示相反，您还可以在**严格**模式和**非严格**模式之间进行选择，后者是默认模式。这意味着 PHP 将尝试将值转换为所需的类型。如果没有信息丢失，转换将会悄无声息地发生，否则将引发警告。这可能导致与字符串到数字转换或 true 和 false 值相同的奇怪结果。

以下是一些此类强制转换的示例：

```php
<?php 

function add(float $a, int $b): float { 
    return $a + $b; 
} 

echo add(3.5, 1); 
// 4.5 
echo add(3, 1); 
// 4 
echo add("3.5", 1); 
// 4.5 
echo add(3.5, 1.2); // 1.2 gets casted to 1 
// 4.5 
echo add("1 week", 1); // "1 week" gets casted to 1.0 
// PHP Notice:  A non well formed numeric value encountered 
// 2 
echo add("some string", 1); 
// Uncaught TypeError Argument 1 passed to add() must be of the type float, string given 

function test_bool(bool $a): string { 
    return $a ? 'true' : 'false'; 
} 

echo test_bool(true); 
// true 
echo test_bool(false); 
// false 
echo test_bool(""); 
// false 
echo test_bool("some string"); 
// true 
echo test_bool(0); 
// false 
echo test_bool(1); 
// true 
echo test_bool([]); 
// Uncaught TypeError: Argument 1 passed to test_bool() must be of the type Boolean 
```

如果您想避免强制转换的问题，可以选择启用严格模式。这样，PHP 将在值不完全符合所需类型时引发错误。为此，必须在文件的第一行之前添加`declare(strict_types=1)`指令。它之前不能有任何内容。

PHP 允许的唯一转换是从`int`到`float`，通过添加`.0`来实现，因为绝对不会有数据丢失的风险。

以下是与之前相同的示例，但启用了严格模式：

```php
<?php 

declare(strict_types=1); 

function add(float $a, int $b): float { 
    return $a + $b; 
} 

echo add(3.5, 1); 
// 4.5 
echo add(3, 1); 
// 4 
echo add("3.5", 1); 
// Uncaught TypeError: Argument 1 passed to add() must be of the type float, string given 
echo add(3.5, 1.2); // 1.2 gets casted to 1 
// Uncaught TypeError: Argument 2 passed to add() must be of the type integer, float given 
echo add("1 week", 1); // "1 week" gets casted to 1.0 
// Uncaught TypeError: Argument 1 passed to add() must be of the type float, string given 
echo add("some string", 1); 
// Uncaught TypeError: Argument 1 passed to add() must be of the type float, string given 

function test_bool(bool $a): string { 
    return $a ? 'true' : 'false'; 
} 

echo test_bool(true); 
// true 
echo test_bool(false); 
// false 
echo test_bool(""); 
// Uncaught TypeError: Argument 1 passed to test_bool() must be of the type boolean, string given 
echo test_bool(0); 
// Uncaught TypeError: Argument 1 passed to test_bool() must be of the type boolean, integer given 
echo test_bool([]); 
// Uncaught TypeError: Argument 1 passed to test_bool() must be of the type boolean, array given 
```

尽管此处未进行演示，但返回类型也适用相同的转换规则。根据模式的不同，PHP 将愉快地执行相同的转换，并显示与参数提示相同的警告和错误。

另一个微妙之处是应用的模式是在进行函数调用的文件顶部声明的模式。这意味着当您调用在另一个文件中声明的函数时，不会考虑该文件的模式。只有当前文件顶部的指令才重要。

关于类型引发的错误，我们将在第三章中看到，*PHP 中的函数基础*，PHP 7 中的异常和错误处理发生了变化，您可以使用它来捕获这些错误。

从现在开始，只要有意义，我们的示例将使用标量类型提示，以使代码更健壮和可读。

强制类型可以被视为繁琐，并且在开始使用时可能会导致一些恼人的问题，但从长远来看，我可以向您保证，它将使您免受一些讨厌的错误。解释器可以执行的所有检查都是您无需自行测试的内容。

这也使得你的函数更容易理解和推理。查看你的代码的人不必问自己一个值可能是什么，他们确切地知道他们必须传递什么样的数据作为参数，以及他们将得到什么。结果是认知负担减轻了，你可以利用时间思考解决问题，而不是记住代码的琐碎细节。

# 匿名函数

您可能已经很熟悉我们看到的声明函数的语法。您可能不知道的是，函数不一定需要有名称。

匿名函数可以分配给变量，用作回调并具有参数。

在 PHP 文档中，匿名函数一词与*闭包*一词可以互换使用。正如我们将在下面的代码片段中看到的，匿名函数甚至是`Closure`类的一个实例，我们将讨论这一点。根据学术文献，尽管这两个概念相似，但有些不同。闭包一词的第一个用法是在 1964 年 Peter Landin 的《表达式的机械评估》中。在这篇论文中，闭包被描述为具有环境部分和控制部分。我们将在本节中声明的函数不会有任何环境，因此严格来说，它们不是闭包。

为了避免阅读其他作品时产生混淆，本书将使用*匿名函数*一词来描述没有名称的函数，就像本节中所介绍的那样：

```php
<?php 

$add = function(float $a, float $b): float { 
    return $a + $b; 
}; 
// since this is an assignment, you have to finish the statement with a semicolon 
```

前面的代码片段声明了一个匿名函数，并将其分配给一个变量，以便我们稍后可以将其重用为另一个函数的参数或直接调用它：

```php
$add(5, 10); 
$sum = array_reduce([1, 2, 3, 4, 5], $add, 0); 
```

如果您不打算重复使用，也可以直接将匿名函数声明为参数：

```php
<?php 
$uppercase = array_map(function(string $s): string { 
  return strtoupper($s); 
}, ['hello', 'world']); 
```

或者您可以像返回任何其他类型的值一样返回一个函数：

```php
<?php 

function return_new_function() 
{ 
  return function($a, $b, $c) { /* [...] */}; 
} 
```

# 闭包

正如我们之前所看到的，闭包的学术描述是指具有对外部环境的访问权限的函数。在本书中，尽管 PHP 使用后一种术语来称呼匿名函数和闭包，但我们将坚持这种语义。

您可能熟悉 JavaScript 的闭包，其中您可以简单地使用外部范围的任何变量而无需进行任何特殊操作。在 PHP 中，您需要使用`use`关键字将现有变量导入匿名函数的范围内：

```php
<?php 

$some_variable = 'value'; 

$my_closure = function() use($some_variable) 
{ 
  // [...] 
}; 
```

PHP 闭包使用早期绑定方法。这意味着闭包内的变量将具有闭包创建时变量的值。如果之后更改变量，则闭包内将看不到更改：

```php
<?php 

$s = 'orange'; 

$my_closure = function() use($s) { echo $s; }; 
$my_closure(); // display 'orange' 

$a = 'banana'; 
$my_closure(); // still display 'orange' 
```

你可以通过引用传递变量，以便变量的更改在闭包内部传播，但由于这是一本关于函数式编程的书，在这本书中我们尝试使用不可变数据结构并避免状态，因此如何做到这一点留给读者作为练习。

请注意，当你将对象传递给闭包时，对对象属性的任何修改都将在闭包内部可访问。PHP 在将对象传递给闭包时不会复制对象。

## 类内的闭包

如果你在类内声明任何匿名函数，它将自动通过通常的`$this`变量获得实例引用。为了保持词汇的一致性，该函数将自动变成一个闭包：

```php
<?php 

class ClosureInsideClass 
{ 
    public function testing() 
    { 
        return function() { 
            var_dump($this); 
        }; 
    } 
} 

$object = new ClosureInsideClass(); 
$test = $object->testing(); 

$test(); 
```

如果你想避免这种自动绑定，你可以声明一个静态匿名函数：

```php
<?php 

class ClosureInsideClass 
{ 
    public function testing() 
    { 
        return (static function() { 
            // no access to $this here, the following line 
            // will result in an error. var_dump($this); 
        }); 
    } 
}; 

$object = new ClosureInsideClass(); 
$test = $object->testing(); 

$test(); 
```

# 使用对象作为函数

有时，你可能希望将函数分成更小的部分，但这些部分不对所有人都可见。在这种情况下，你可以利用任何对象上的`__invoke`魔术方法，让你将实例作为函数使用，并将那个辅助函数隐藏为对象内部的私有方法：

```php
<?php 

class ObjectAsFunction 
{ 
    private function helper(int $a, int $b): int 
    { 
        return $a + $b; 
    } 

    public function __invoke(int $a, int $b): int 
    { 
      return $this->helper($a, $b); 
    } 
} 

$instance = new ObjectAsFunction(); 
echo $instance(5, 10); 
```

`__invoke`方法将使用你传递给实例的任何参数进行调用。如果你愿意，你也可以为你的对象添加一个构造函数，并使用它包含的任何方法和属性。只需尽量保持纯净，因为一旦使用可变属性，你的函数将变得更难理解。

# `Closure`类

所有匿名函数实际上都是`Closure`类的实例。然而，正如文档中所述（[`php.net/manual/en/class.closure.php`](http://php.net/manual/en/class.closure.php)），这个类不使用前面提到的`__invoke`方法；这是 PHP 解释器中的一个特例：

> *除了这里列出的方法，这个类还有一个`__invoke`方法。这是为了与实现调用魔术的其他类保持一致，因为这个方法不用于调用函数。*

类上的这个方法允许你更改`$this`变量在闭包内部绑定到哪个对象。你甚至可以将一个对象绑定到类外创建的闭包上。

如果你开始使用`Closure`类的特性，请记住`call`方法是在 PHP 7 中才被添加的。

# 高阶函数

PHP 函数可以将函数作为参数并返回函数作为返回值。执行任何这些操作的函数称为高阶函数。就是这么简单。

实际上，如果你阅读以下代码示例，你会很快发现我们已经创建了多个高阶函数。你还会发现，毫不奇怪，你将学到的大多数函数式技术都围绕着高阶函数。

# 什么是可调用？

`callable`是一种类型提示，可以用来强制函数的参数是可以调用的东西，比如一个函数。从 PHP 7 开始，它也可以用作返回值的类型提示：

```php
<?php 

function test_callable(callable $callback) : callable { 
    $callback(); 
    return function() { 
        // [...] 
    }; 
} 
```

然而，类型提示无法强制可调用的参数数量和类型。但能够保证有可调用的东西已经很好了。

可调用可以采用多种形式：

+   用于命名函数的字符串

+   用于类方法或静态函数的数组

+   匿名函数或闭包的变量

+   带有`__invoke`方法的对象

让我们看看如何使用所有这些可能性。让我们从按名称调用一个简单的函数开始：

```php
$callback = 'strtoupper'; 
$callback('Hello World !'); 
```

我们也可以对类内的函数做同样的操作。让我们声明一个带有一些函数的`A`类，并使用数组来调用它。

```php
class A { 
    static function hello($name) { return "Hello $name !\n"; } 
    function __invoke($name) { return self::hello($name); } 
} 

// array with class name and static method name 
$callback = ['A', 'hello']; 
$callback('World'); 
```

使用字符串只对静态方法有效，因为其他方法将需要一个对象作为它们的上下文。对于静态方法，你也可以直接使用一个简单的字符串，但这只适用于 PHP 7 及更高版本；之前的版本不支持这种语法：

```php
$callback = 'A::hello'; 
$callback('World'); 
```

您也可以轻松地在类实例上调用方法：

```php
$a = new A(); 

$callback = [$a, 'hello']; 
$callback('World'); 
```

由于我们的`A`类具有`__invoke`方法，因此我们可以直接将其用作`callable`：

```php
$callback = $a; 
$callback('World'); 
```

您还可以使用任何变量，其中分配了一个匿名函数作为`callable`：

```php
$callback = function(string s) { 
    return "Hello $s !\n"; 
} 
$callback('World'); 
```

PHP 还为您提供了两个助手来调用函数，即`call_user_func_array`和`call_user_func`。它们将可调用作为参数，并且您还可以传递参数。对于第一个助手，您传递一个包含所有参数的数组；对于第二个助手，您分别传递它们：

```php
call_user_func_array($callback, ['World']); 
```

最后要注意的一点是，如果您使用了`callable`类型提示：任何包含已声明的函数名称的字符串都被视为有效；这有时会导致一些意外的行为。

一个有些牵强的例子是一个测试套件，您可以通过传递一些字符串来检查某些函数是否只接受有效的可调用对象，并捕获结果异常。在某个时候，您引入了一个库，现在这个测试失败了，尽管两者应该是无关的。发生的情况是，所涉及的库声明了一个与您的字符串完全相同的函数名称。现在，该函数存在，不再引发异常。

# 总结

在本章中，我们发现了如何创建新的匿名函数和闭包。您现在也熟悉了传递它们的各种方式。我们还了解了新的 PHP 7 标量类型提示，这有助于使我们的程序更加健壮，以及`callable`类型提示，这样我们就可以强制参数或返回值为有效函数。

对于那些已经使用 PHP 一段时间的人来说，本章可能没有什么新鲜的内容。但现在我们有了一个共同的基础，这将帮助我们进入函数式世界。

在 PHP 函数的基础知识介绍完毕后，我们将在下一章中了解有关函数式编程的基本概念。我们将看到，您的函数必须遵守某些规则，才能真正在函数式代码库中发挥作用。
