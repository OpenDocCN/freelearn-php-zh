# 第七章。重构

在本书中，我主要关注使用设计模式来解决你编写的新代码；这是至关重要的，开发人员在批评他人的代码之前，必须首先改进自己的代码。开发人员必须首先努力理解如何编写代码，然后才能有效地重构代码。

本章将主要基于 Martin Fowler 等人的《重构：改善既有代码的设计》以及 Joshua Kerievsky 的《重构到模式》。如果您对此主题感兴趣，我强烈推荐阅读这些书籍。

# 什么是重构？

重构代码的一个关键主题是解决代码内部结构的问题，同时不改变被重构程序的外部行为。在某些情况下，这可能意味着在先前没有意图或考虑的地方引入内部结构。

重构作为一个过程，在编写代码后改进代码的设计。虽然设计是软件工程过程中的关键阶段，但通常被忽视（尤其是在 PHP 中）；除此之外，长期维护代码结构需要对软件设计的持续理解。如果开发人员在不了解原始设计的情况下接手项目，他们可能会以非常粗糙的方式进行开发。

在极限编程（XP）中，使用了一个被称为“无情重构”的短语，这是不言而喻的。在 XP 中，重构被提议作为保持软件设计尽可能简单并避免不必要复杂性的机制。正如 XP 的规则中所述：“确保一切只表达一次。最终，制作一个精心打理的系统需要更少的时间。”

重构的一个关键原则是将软件设计视为一种要发现而不是事先创建的东西。在开发系统时，我们可以使用开发作为找到良好设计解决方案的机制。通过使用重构，我们能够确保系统在开发过程中保持良好，从而我们能够降低技术债务。

重构并非总是可能的，您可能偶尔会遇到无法更改的“黑盒”系统，甚至可能需要封装系统以进行重写。然而，有许多情况下，我们可以简单地重构代码以改进设计。

# 测试，测试，再测试

没有办法绕过这一点，为了重构代码，您需要一套可靠的测试。重构代码可能会减少引入错误的机会，但改变代码的设计会引入大量引入新错误的机会。

在重构过程中会出现意外的副作用，当类紧密耦合时，您可能会发现对一个函数进行微小更改会导致完全不同类中的负面副作用。

良好的重构效果需要良好的测试。这是无法绕过的。

除此之外，从更政治的角度来看，一些公司遇到了重复糟糕的重构努力所带来的不良影响，可能会不愿意重构代码；确保有良好的测试可以确保重构不会破坏功能。

在本章中，我将展示重构工作，这应该伴随着使用单元测试的测试工作，而在本书的下一章（也是最后一章）中，我将讨论行为测试（用于 BDD）。单元测试是开发人员测试给定代码单元的最佳机制；单元测试补充了代码结构，证明方法是否按照预期工作，并测试代码单元之间的交互；在这个意义上，它们是开发人员在重构工作中最好的测试形式。然而，行为测试是用来测试代码行为的，因此在演示应用程序能够成功完成给定形式的行为方面是有用的。

每个经验丰富的开发人员都会记得痛苦的调试任务；有时候会持续到深夜。让我们想想大多数开发人员日常工作的方式。他们并不总是编写代码，他们的一些时间花在设计代码上，而相当多的时间花在调试他们已经编写的代码上。拥有自我测试的代码可以迅速减轻这种负担。

测试驱动开发围绕着在编写功能之前编写测试的方法论，确实，代码应该与测试相匹配。

在测试类时，确保测试类的`public`接口；确实，PHPUnit 不允许您在普通用法下测试`private`或`protected`方法。

# 代码异味

**代码异味**本质上是一些不良实践，使您的代码变得不必要地难以理解，可以使用本章中介绍的技术来重构不良代码。代码异味通常违反了一些基本的软件设计原则，因此可能会对整体代码的设计质量产生负面影响。

Martin Fowler 通过以下方式定义了代码异味：

> *“代码异味是通常对系统中更深层次问题的表面指示。”*

在本书的开头，我们讨论了*技术债务*这个术语，在这个意义上，代码异味可以作为*技术债务*的一部分。

代码异味可能不一定构成错误，它不会阻止程序的执行，但它可以帮助在以后引入错误的过程中，并使代码重构到适当的设计变得更加困难。

让我们来看看在处理传统 PHP 项目时可能遇到的一些基本代码异味。

我们将讨论一些代码异味以及如何以相当简单的方式解决它们，但现在让我们考虑一些稍微重要的、重复出现的模式，以及如何通过应用设计模式来简化代码的维护。

在这里，我们将具体讨论重构*到*模式，有些情况下，当简化代码设计时，您可能会从模式重构*到*模式。本章的重复主题围绕代码设计如何在代码的开发生命周期中存在，它不仅仅在任意设计阶段之后被丢弃。

模式可以用来传达意图，它们可以作为开发人员之间的语言；这就是为什么了解并继续使用大量模式在软件工程师的职业生涯中至关重要。

在书籍*重构到模式*中还有更多这样的方法，我在这里挑选了对 PHP 开发人员最合适的方法。

## 长方法和重复的代码

重复的代码是非常常见的代码异味。开发人员经常会复制和粘贴代码，而不是使用适当的控制结构来进行应用程序。如果相同的控制结构出现在多个地方，将两个结构合并成一个将使您的代码受益。

如果重复的代码是相同的，你可以使用提取方法。那么什么是提取方法？实质上，**提取方法**只是将长函数中的业务逻辑提取到更小的函数中。

假设有一个`dice`类，一旦掷骰子，它将以罗马数字返回 1 到 6 之间的随机数。

`Legacy`类可以是这样的：

```php
class LegacyDice 
{ 
  public function roll(): string 
  { 
    $rand = rand(1, 6); 

    // Switch statement to convert a number between 1 and 6 to a Roman Numeral. 
    switch ($rand) { 
      case 5: 
        $randString = "V"; 
        break; 
      case 6: 
        $randString = "VI"; 
        break; 
      default: 
        $randString = str_repeat("I", $rand); 
        break; 
    } 

    return $randString; 
  } 
} 

```

让我们提取一个方法，将随机数转换为罗马数字，并将其放入一个单独的函数中：

```php
class Dice 
{ 
  /** 
   * Roll the dice. 
   * @return string 
   */ 
  public function roll(): string 
  { 
    $rand = rand(1, 6); 

    return $this->numberToRomanNumeral($rand); 
  } 

  /** 
   * Convert a number between 1 and 6 to a Roman Numeral. 
   * 
   * @param int $number 
   * 
   * @return string 
   * @throws Exception 
   */ 
  public function numberToRomanNumeral(int $number): string 
  { 
    if (($number < 1) || ($number > 6)) { 
      throw new Exception('Number out of range.'); 
    } 

    switch ($number) { 
      case 5: 
        $randString = "V"; 
        break; 
      case 6: 
        $randString = "VI"; 
        break; 
      default: 
        $randString = str_repeat("I", $number); 
        break; 
    } 

    return $randString; 
  } 
} 

```

我们对原始代码块只进行了两个更改，我们将执行罗马数字转换的函数分离出来，并将其放入一个单独的函数中。我们用函数本身的 DocBlock 替换了内联注释。

如果重复存在于多个地方（且相同），则可以使用此方法进行复制，我们只需调用一个函数，而不是在多个地方重复代码。

如果代码在不相关的类中，看看它在逻辑上适合哪里（在这两个类中的任何一个或一个单独的类中），并将其提取到那里。

在本书的前面，我们已经讨论了保持函数小的必要性。这对于确保您的代码在长期内可读性非常重要。

我经常看到开发人员在函数内部注释代码块；相反，为什么不将这些方法拆分为它们自己的函数？通过 DocBlocks 可以添加可读的文档。因此，我们在这里使用的提取方法可以更简单地使用；拆分长方法。

处理较小的方法时，解决各种业务问题要容易得多。

## 大类

大类经常违反单一职责原则。在特定时间点上，您正在处理的类是否只有一个更改的原因？一个类应该只对功能的一个部分负责，而且该类应该完全封装该责任。

通过提取不严格符合单一职责的方法将类分成多个类，这是一种简单而有效的方法，可以帮助减轻这种代码异味。

## 用多态性或策略模式替换复杂的逻辑语句和 switch 语句

通过使用多态行为，可以大大减少 switch 语句（或者说无休止的大型 if 语句）；我在本书的早期章节中已经描述了多态性，并且它提供了一种比使用 switch 语句更优雅地处理计算问题的方式。

假设您正在根据国家代码进行切换；美国或英国，而不是以这种方式切换，通过使用多态性，您可以运行相同的方法。

在不可能进行多态行为的情况下（例如，没有共同的接口的情况下），在某些情况下，通过用策略替换类型代码甚至可能会受益；实际上，您可以将多个 switch 语句合并为仅将类注入到客户端的构造函数中，该类将处理与各个类的关系。

例如；假设我们有一个 Output 接口，这个接口由包含`load`方法的各种其他类实现。这个`load`方法允许我们注入一个数组，并且我们以所请求的格式获取一些数据。这些类是该行为的极其粗糙的实现：

```php
interface Output 
{ 
  public function load(array $data); 
} 

class Serial implements Output 
{ 
  public function load(array $data) 
  { 
    return serialize($data); 
  } 
} 

class JSON implements Output 
{ 
  public function load(array $data) 
  { 
    return json_encode($data); 
  } 
} 

class XML implements Output 
{ 
  public function load(array $data) 
  { 
    return xmlrpc_encode($data); 
  } 
} 

```

### 注意

在撰写本文时，PHP 仍然认为`xmlrpc_encode`函数是实验性的，因此，我建议不要在生产中使用它。这里纯粹是为了演示目的（为了保持代码简洁）。

一个极其粗糙的实现，带有`switch`语句，可能如下所示：

```php
$client = "JSON"; 

switch ($client) { 
  case "Serial": 
    $client = new Serial(); 
    break; 
  case "JSON": 
    $client = new JSON(); 
    break; 
  case "XML": 
    $client = new XML(); 
    break; 
} 

echo $client->load(array(1, 2)); 

```

但显然，我们可以通过实现一个允许我们将`Output`类注入到`Client`中的客户端来做很多事情，并相应地允许我们接收输出。这样的类可能是这样的：

```php
class OutputClient 
{ 
  private $output; 

  public function __construct(Output $outputType) 
  { 
    $this->output = $outputType; 
  } 

  public function loadOutput(array $data) 
  { 
    return $this->output->load($data); 
  } 
} 

```

现在我们可以非常简单地使用这个客户端：

```php
**$client = new OutputClient(new JSON());
echo $client->loadOutput(array(1, 2));**

```

## 在单一控制结构后复制代码

我不会在这里重申模板设计模式的工作原理，但我想解释的是，它可以用来帮助消除重复的代码。

我在本书中展示的模板设计模式有效地将程序的结构抽象化，然后我们只是填充了特定于实现的方法。这可以帮助我们通过避免一遍又一遍地重复单个控制结构来减少代码重复。

## 长参数列表和原始类型过度使用

原始类型过度使用是指开发人员过度使用原始数据类型而不是使用对象。

PHP 支持八种原始类型；这组可以进一步细分为标量类型、复合类型和特殊类型。

标量类型是保存单个值的数据类型。如果你问自己“这个值可以在一个范围内吗？”你可以识别它们。数字可以在*X*到*Y*的范围内，布尔值可以在 false 到 true 的范围内。以下是一些标量类型的例子：

+   布尔

+   整数

+   浮点数

+   字符串

复合类型由一组标量值组成：

+   数组

+   对象

特殊类型如下：

+   资源（引用外部资源）

+   NULL

假设我们有一个简单的`Salary`计算器类，它接受员工的基本工资、佣金率和养老金率；在发送了这些数据之后，可以使用`calculate`方法输入他们的销售额来计算他们的总工资：

```php
class Salary 
{ 
  private $baseSalary; 
  private $commission = 0; 
  private $pension = 0; 

  public function __construct(float $baseSalary, float $commission, float $pension) 
  { 
    $this->baseSalary = $baseSalary; 
    $this->commission = $commission; 
    $this->pension    = $pension; 
  } 

  public function calculate(float $sales): float 
  { 
    $base       = $this->baseSalary; 
    $commission = $this->commission * $sales; 
    $deducation = $base * $this->pension; 

    return $commission + $base - $deducation; 
  } 
} 

```

注意构造函数有多长。是的，我们可以使用生成器模式来创建一个对象，然后将其注入到构造函数中，但在这种情况下，我们能够特别地将复杂的信息抽象化。在这种情况下，如果我们将员工信息移到一个单独的类中，我们可以确保更好地遵守单一职责原则。

第一步是分离类的职责，以便我们可以分离类的职责：

```php
class Employee 
{ 
  private $name; 
  private $baseSalary; 
  private $commission = 0; 
  private $pension = 0; 

  public function __construct(string $name, float $baseSalary) 
  { 
    $this->name       = $name; 
    $this->baseSalary = $baseSalary; 
  } 

  public function getBaseSalary(): float 
  { 
    return $this->baseSalary; 
  } 

  public function setCommission(float $percentage) 
  { 
    $this->commission = $percentage; 
  } 

  public function getCommission(): float 
  { 
    return $this->commission; 
  } 

  public function setPension(float $rate) 
  { 
    $this->pension = $rate; 
  } 

  public function getPension(): float 
  { 
    return $this->commission; 
  } 
} 

```

从这一点上，我们可以简化`Salary`类的构造函数，以便它只需要输入`Employee`对象，我们就能够使用该类：

```php
class Salary 
{ 
  private $employee; 

  public function __construct(Employee $employee) 
  { 
    $this->employee = $employee; 
  } 

  public function calculate(float $sales): float 
  { 
    $base       = $this->employee->getBaseSalary(); 
    $commission = $this->employee->getCommission() * $sales; 
    $deducation = $base * $this->employee->getPension(); 

    return $commission + $base - $deducation; 
  } 
} 

```

## 不当暴露

假设我们有一个`Human`类如下：

```php
class Human 
{ 
  public $name; 
  public $dateOfBirth; 
  public $height; 
  public $weight; 
} 

```

我们可以随心所欲地设置值，没有验证，也没有统一的获取信息的方式。这有什么问题吗？嗯，在面向对象编程中，封装的原则至关重要；我们隐藏数据。换句话说，我们的数据不应该在没有拥有对象知道的情况下被公开。

相反，我们用`private`数据变量替换所有`public`数据变量。除此之外，我们还添加了适当的方法来获取和设置数据：

```php
class Human 
{ 
  private $name; 
  private $dateOfBirth; 
  private $height; 
  private $weight; 

  public function __construct(string $name, double $dateOfBirth) 
  { 
    $this->name        = $name; 
    $this->dateOfBirth = $dateOfBirth; 
  } 

  public function setWeight(double $weight) 
  { 
    $this->weight = $weight; 
  } 

  public function getWeight(): double 
  { 
    return $this->weight; 
  } 

  public function setHeight(double $height) 
  { 
    $this->height = $height; 
  } 

  public function getHeight(): double 
  { 
    return $this->height; 
  } 
} 

```

确保 setter 和 getter 是合乎逻辑的，不仅仅是因为类属性存在。完成后，您需要检查应用程序，并替换任何对变量的直接访问，以便它们首先通过适当的方法。

然而，这现在暴露了另一个代码异味；特征嫉妒。

## 特征嫉妒

松散地说，**特征嫉妒**是指我们不让一个对象计算自己的属性，而是将其偏移到另一个类。

所以在前面的例子中，我们有我们自己的`Salary`计算器类，如下：

```php
class Salary 
{ 
  private $employee; 

  public function __construct(Employee $employee) 
  { 
    $this->employee = $employee; 
  } 

  public function calculate(float $sales): float 
  { 
    $base       = $this->employee->getBaseSalary(); 
    $commission = $this->employee->getCommission() * $sales; 
    $deducation = $base * $this->employee->getPension(); 

    return $commission + $base - $deducation; 
  } 
} 

```

相反，让我们看看将这个函数实现到`Employee`类本身中，结果我们也可以忽略不必要的 getter 并将属性合理地内部化：

```php
class Employee 
{ 
  private $name; 
  private $baseSalary; 
  private $commission = 0; 
  private $pension = 0; 

  public function __construct(string $name, float $baseSalary) 
  { 
    $this->name       = $name; 
    $this->baseSalary = $baseSalary; 
  } 

  public function setCommission(float $percentage) 
  { 
    $this->commission = $percentage; 
  } 

  public function setPension(float $rate) 
  { 
    $this->pension = $rate; 
  } 

  public function calculate(float $sales): float 
  { 
    $base       = $this->baseSalary; 
    $commission = $this->commission * $sales; 
    $deducation = $base * $this->pension; 

    return $commission + $base - $deducation; 
  } 
} 

```

## 不当亲密关系

这在继承中经常发生；Martin Fowler 优雅地表达如下：

> “子类总是会比父类更了解他们的父类。”

更一般地说；当一个字段在另一个类中的使用比在类本身中更多时，我们可以使用移动字段方法在新类中创建一个字段，然后将该字段的用户重定向到新类。

我们可以将这与移动方法结合起来，将一个函数放在最常使用它的类中，并从原始类中删除它，如果这不可能，我们可以简单地在新类中引用该函数。

## 深度嵌套的语句

嵌套的 if 语句很混乱且丑陋。这会导致难以理解的意大利面逻辑；而是使用内联函数调用。

从最内部的代码块开始，试图将该代码提取到自己的函数中，让它可以幸福地存在。在第一章中，我们讨论了如何通过示例实现这一点，但如果您经常进行重构，您可能希望考虑投资一种可以帮助您的工具。

这里有一个提示，对于我们中的 PHPStorm 用户：在重构菜单中有一个很好的小选项，可以自动为您执行此操作。只需高亮显示您希望提取的代码块，转到菜单栏中的重构，然后单击提取>方法。然后会弹出一个对话框，允许您配置如何进行重构：

![深度嵌套语句](img/image_07_001.jpg)

## 删除对参数的赋值

尽量避免在函数体内设置参数：

```php
class Before 
{ 
  function deductTax(float $salary, float $rate): float 
  { 
    $salary = $salary * $rate; 

    return $salary; 
  } 
} 

```

这可以通过正确设置内部参数来实现：

```php
class After 
{ 
  function deductTax(float $salary, float $rate): float 
  { 
    $netSalary = $salary * $rate; 

    return $netSalary; 
  } 
} 

```

通过这样的行为，我们能够在前进时轻松识别和提取重复的代码，此外，它还可以在以后维护这段代码时更容易地替换代码。

这是一个简单的调整，允许我们识别代码中特定的参数在做什么。

## 注释

注释并不是一种代码气味，很多情况下，注释是非常有益的。正如 Martin Fowler 所说：

> “在我们的嗅觉类比中，注释不是一种难闻的气味；事实上，它们是一种甜美的气味。”

然而，Fowler 继续演示了注释如何被用作遮盖代码气味的除臭剂。当你发现自己在函数内部注释代码块时，你可以找到一个很好的机会使用提取方法。

如果注释隐藏了一种难闻的气味，重构掉这种气味，很快你就会发现原始注释变得多余了。这并不是不需要对函数进行 DocBlock 或不必要地寻找代码注释的借口，但重要的是要记住，当您重构设计变得更简单时，特定的注释可能变得无用。

## 用构建器封装组合

正如本书前面讨论的那样，构建器设计模式可以通过我们将一长串参数转换为一个单一对象来工作，然后我们可以将其抛入另一个类的构造函数中。

例如，我们有一个名为`APIBuilder`的类，这个构建器类可以用 API 的密钥和密钥本身来实例化，但一旦它被实例化为一个对象，我们就可以简单地将整个对象传递给另一个类的构造函数。

到目前为止，一切顺利；但我们可以使用这个构建器模式来封装组合模式。我们实际上只需创建一个构建器来创建我们的项目。通过这样做，我们可以更好地控制一个类，为我们提供了一个机会来导航和修改组合家族的整个树结构。

## 用观察者替换硬编码的通知

硬编码的通知通常是两个类紧密耦合在一起，以便一个能够通知另一个。相反，通过使用`SplObserver`和`SplSubject`接口，观察者可以使用更加可插拔的方式更新主题。在观察者中实现`update`方法后，主题只需要实现`Subject`接口：

```php
SplSubject { 
   /* Methods */ 
   abstract public void attach ( SplObserver $observer ) 
   abstract public void detach ( SplObserver $observer ) 
   abstract public void notify ( void ) 
} 

```

结果的架构是一个更加可插拔的通知系统，不再紧密耦合。

## 用组合替换一个/多个区别

当我们有单独的逻辑来处理个体到组的情况时，我们可以使用组合模式来 consolide 这些情况。这是本书早些时候介绍过的一种模式；为了将其合并到这种模式中，开发人员只需要修改他们的代码，使一个类可以处理两种形式的数据。

为了实现这一点，我们必须首先确保这两个区别实现了相同的接口。

当我最初演示这个模式时，我写了关于如何使用这个模式来处理将单个歌曲和播放列表视为一个的情况。假设我们的`Music`接口纯粹是以下内容：

```php
interface Music 
{ 
  public function play(); 
} 

```

关键任务就是确保这个接口对于单个和多个区分都得到遵守。你的`Song`类和`Playlist`类都必须实现`Music`接口。这基本上是让我们能够对待它们的行为。

## 使用适配器分离版本

我不会在这本书中长篇大论地讨论适配器，因为我之前已经非常详细地介绍过它们，但我只是想让你考虑一下，它们可以用来支持不同版本的 API。

确保不要将多个 API 版本的代码放在同一个类中，而是可以将这些版本之间的差异抽象到一个适配器中。在使用这种方法时，我建议你最初尝试使用封装方法，而不是基于继承的方法，因为这样可以为未来提供更大的自由度。

# 我应该告诉我的经理什么？

重构然后添加功能往往比仅仅添加功能更快，同时也为现有代码库增加了价值。许多了解软件及其开发方式的优秀经理都会理解这一点。

当然，有些经理对软件的实际情况一无所知，他们往往只受到最后期限的驱使，可能不愿意更多地了解自己的专业领域。我在本书中之前提到过的那些可怕的开发人员就是这样。有时，*Scrum Master*也会有这种情况，因为他们可能无法理解整个软件开发生命周期。

正如 Martin Fowler 所说：

> “当然，很多人说他们追求质量，但更多的是追求进度。在这些情况下，我给出了更具争议性的建议：不要说！”

不了解技术流程的经理可能会急于基于软件能够快速生产的基础上交付；重构可能是帮助生产软件最快速的方式。它提供了一种高效而彻底的方式来快速了解项目，并允许我们平稳地注入新功能的过程。

我们将在本书的下一章讨论管理以及项目如何有效地进行管理。

# 总结

在本章中，我们讨论了一些重构代码的方法，以确保设计始终保持良好的质量。通过重构代码，我们可以更好地理解我们的代码库，并为我们添加到软件中的额外功能未来做好准备。

简化和分解你面临的问题是重构代码时可以使用的两个最基本的工具。

如果你正在使用 CI 环境，让 PHP Mess Detector（PHPMD）在该环境中运行也可以帮助你编写更好的代码。

在下一章中，我将讨论如何适当地使用设计模式，首先快速介绍在网络环境中开发 API 的方法。
