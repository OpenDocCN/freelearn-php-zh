# 第一章：为什么“优秀的 PHP 开发者”不是一个矛盾修辞

2010 年，MailChimp 在他们的博客上发表了一篇名为“呃，你用 PHP？”的文章。在这篇博文中，他们描述了当他们向认为“优秀的 PHP 程序员”是一个矛盾修辞的开发人员解释他们选择 PHP 时的恐怖。在他们的反驳中，他们辩称他们的 PHP 不是“你爷爷的 PHP”，他们使用了一个复杂的框架。我倾向于根据 PHP 的质量来评判，不仅仅是它的功能，还有它的安全性和架构。这本书关注的是关于如何设计代码的想法。软件的设计允许开发人员以无 bug 和优雅的方式轻松扩展代码的用途。

马丁·福勒说：

> “任何傻瓜都可以写出计算机能理解的代码。优秀的程序员写出人类能理解的代码。”

这不仅仅局限于代码风格，而是开发人员如何设计和构建他们的代码。我遇到过许多开发人员，他们总是埋头于文档，复制和粘贴代码片段，直到它能够工作；拼凑代码片段直到它能够工作。此外，我经常看到软件开发过程迅速恶化，因为开发人员将他们的类与越来越长的函数紧密耦合。

软件工程师不仅仅是编写软件；他们必须知道如何设计软件。事实上，一个优秀的软件工程师在面试其他软件工程师时会询问关于代码设计本身的问题。获得一段可以执行的代码是微不足道的，询问开发人员`strtolower`或`str2lower`哪个是函数的正确名称也是无害的（记录上，是`strtolower`）。知道类和对象之间的区别并不能让你成为一个称职的开发人员；一个更好的面试问题可能是如何将子类型多态应用于真实的软件开发挑战。未能评估软件设计技能会使面试变得肤浅，并导致无法区分谁擅长它，谁不擅长。这些高级话题将在本书中讨论，通过学习这些策略，你将更好地理解在讨论软件架构时应该问什么问题。

Moxie Marlinspike 曾在推特上发表过以下言论：

> “作为一名软件开发人员，我羡慕作家、音乐家和电影制作人。与软件不同，当他们创作出一些东西时，它真的完成了，永远。”

在开发软件时，我们不能忘记我们是作者，不仅仅是为了机器的指令，而且我们还在创作一些我们希望其他人能够扩展的东西。因此，我们的代码不仅仅是针对机器的，也是针对人类的。代码不仅仅是机器的诗歌，也应该是人类的诗歌。

这当然说起来容易做起来难。在 PHP 中，这可能特别困难，因为 PHP 为开发人员提供了如何设计和构建他们的代码的自由。由于自由的本质，它可能被使用和滥用，这在 PHP 提供的自由中也是真实的。

因此，开发人员理解适当的软件设计实践以确保他们的代码保持长期可维护性变得越来越重要。事实上，另一个关键技能在于*重构*代码，改进现有代码的设计，使其更容易在长期内扩展。

技术债务，糟糕系统设计的最终后果，是我发现作为 PHP 开发人员职业生涯的一部分。这对我来说是真实的，无论是处理提供高级功能还是简单网站的系统。它通常是因为开发人员选择以各种原因实施糟糕的设计而产生的；这是在向现有代码库添加功能或在软件的初始构建过程中做出糟糕的设计决策时。重构可以帮助我们解决这些问题。

SensioLabs（Symfony 框架的创建者）有一个名为**Insight**的工具，允许开发人员计算他们自己代码中的技术债务。2011 年，他们使用这个工具对各种项目进行了技术债务评估；毫不奇怪，他们发现 WordPress 4.1 在他们评估的所有平台中名列前茅，他们声称解决项目中的技术债务需要 20.1 年的时间。

熟悉 WordPress 核心的人可能不会对此感到惊讶，但这个问题当然不仅与 WordPress 有关。在我的 PHP 职业生涯中，从处理安全关键的加密系统到处理与使命关键嵌入式系统相关的系统，处理技术债务是工作的一部分。处理技术债务对于 PHP 开发人员来说并不是一件可耻的事情，有些人甚至可能认为这是勇敢的。处理技术债务并不是一件容易的事情，特别是面对越来越苛刻的用户群、客户或项目经理；不断要求更多功能，而不了解项目所带来的技术债务。

我最近给 PHP Internals 组发送了电子邮件，询问他们是否应该考虑废弃错误抑制运算符`@`。当任何 PHP 函数前面加上@符号时，该函数将抑制其返回的错误。这可能是残酷的，特别是当该函数产生致命错误导致脚本停止执行时，使得调试变得困难。如果错误被抑制，脚本可能无法执行，而不提供开发人员原因。在某些情况下，使用这个运算符可能被描述为反模式，我们将在第四章*结构设计模式*中进行介绍。

尽管没有人反对处理错误的更好方法（`try`/`catch`，`适当的验证`）比滥用错误抑制运算符，并且废弃应该是 PHP 的最终目标，但事实是，一些函数即使已经有了成功/失败值，仍会返回不必要的警告。这意味着由于 PHP 核心中的技术债务，这个运算符在很多其他先决条件工作完成之前无法被废弃。与此同时，开发人员需要决定处理错误的最佳方法。在不解决不必要的错误报告的固有问题之前，这个运算符无法被废弃。因此，开发人员需要接受教育，了解应该用于处理错误的适当方法，而不是不断地使用`@`符号。

从根本上说，技术债务减缓了项目的开发速度，并经常导致部署的代码存在问题，因为开发人员试图在一个脆弱的项目上工作。

在开始新项目时，永远不要害怕讨论架构，因为架构会议对开发人员的合作至关重要；正如我与之合作过的 Scrum Master 在面对“会议是工作的绝佳替代品”这一批评时所说的，“会议就是工作……如果没有会议，你会做多少工作？”

在本章的其余部分，我们将涵盖以下几点：

+   编码风格-PSR 标准

+   修订面向对象编程

+   使用 Composer 设置环境

+   谁是四人帮？

# 编码风格 - PSR 标准

在编码风格方面，我想向你介绍 PHP 框架互操作组创建的 PSR 标准。具体来说，适用于编码标准的两个标准是 PSR-1（基本编码风格）和 PSR-2（编码风格指南）。除此之外，还有 PSR 标准涵盖其他领域，例如，截至今天；PSR-4 标准是该组发布的最新的自动加载标准。你可以在[`www.php-fig.org/`](http://www.php-fig.org/)找到更多关于这些标准的信息。

我坚信使用编码风格来强制执行代码库中的一致性。这确实会对整个项目中的代码可读性产生影响。特别是在开始一个项目时（很可能你正在阅读本书以找出如何正确做到这一点），因为你的编码风格决定了在这个项目上跟随你的开发人员将采用的风格。使用 PSR-1 或 PSR-2 这样的全局标准意味着开发人员可以轻松地在项目之间切换，而不必在他们的 IDE 中重新配置他们的代码风格。良好的代码风格可以使格式错误更容易发现。毋庸置疑，随着时间的推移，编码风格会发展，迄今为止，我选择遵循 PSR 标准。

我坚信这句话：*永远编写代码，就好像最终维护你的代码的人是一个知道你住在哪里的暴力精神病患者*。虽然不知道谁最初写下这句话，但普遍认为可能是约翰·伍兹或者马丁·戈尔丁。

我强烈建议在继续阅读本书之前熟悉这些标准。

# 修订面向对象编程

面向对象编程不仅仅是类和对象；它是一种基于*对象*（数据结构）的整个编程范式，其中包含数据字段和方法。理解这一点至关重要；使用类来组织一堆无关的方法并不是面向对象。

假设你已经了解了类（以及如何实例化它们），让我提醒你一些不同的要点。

## 多态

多态是一个相当长的词，表示一个相当简单的概念。本质上，多态意味着相同的*接口*与不同的基础代码一起使用。因此，多个类可以有一个绘制函数，每个函数接受相同的参数，但在基础级别上，代码实现是不同的。

在这一部分，我想特别谈谈子类型多态（也称为子类型或包含多态）。

假设我们的超类型是动物；我们的子类型可能是猫、狗和羊。

在 PHP 中，接口允许你定义一个类必须包含的一组功能，从 PHP 7 开始，你还可以使用标量类型提示来定义我们期望的返回类型。

例如，假设我们定义了以下接口：

```php
interface Animal 
{ 
  public function eat(string $food) : bool; 

  public function talk(bool $shout) : string; 
} 

```

然后我们可以在我们自己的类中实现这个接口，如下所示：

```php
class Cat implements Animal { 
} 

```

如果我们在没有定义类的情况下运行此代码，将会收到以下错误消息：

```php
Class Cat contains 2 abstract methods and must therefore be declared abstract or implement the remaining methods (Animal::eat, Animal::talk) 

```

本质上，我们需要实现我们在接口中定义的方法，所以现在让我们创建一个实现这些方法的类：

```php
class Cat implements Animal 
{ 
  public function eat(string $food): bool 
  { 
    if ($food === "tuna") { 
      return true; 
    } else { 
      return false; 
    } 
  } 

  public function talk(bool $shout): string 
  { 
    if ($shout === true) { 
      return "MEOW!"; 
    } else { 
      return "Meow."; 
    } 
  } 
} 

```

现在我们已经实现了这些方法，我们可以实例化我们想要的类并使用其中包含的函数：

```php
**$felix = new Cat();echo**
**$felix->talk(false);**

```

那么多态是如何发挥作用的呢？假设我们有另一个类代表狗：

```php
class Dog implements Animal 
{ 
  public function eat(string $food): bool 
  { 
    if (($food === "dog food") || ($food === "meat")) { 
      return true; 
    } else { 
      return false; 
    } 
  } 

  public function talk(bool $shout): string 
  { 
    if ($shout === true) { 
      return "WOOF!"; 
    } else { 
      return "Woof woof."; 
    } 
  } 
} 

```

现在假设我们在`pets`数组中有多种不同类型的动物：

```php
$pets = array( 
  'felix'     => new Cat(), 
  'oscar'     => new Dog(), 
  'snowflake' => new Cat() 
); 

```

现在我们可以逐个循环遍历所有这些宠物，以便运行`talk`函数。我们不关心宠物的类型，因为我们扩展了动物接口，所以我们得到的每个类中实现的`talk`方法都是由我们实现的。

假设我们想让所有的动物都运行`talk`方法。我们可以使用以下代码：

```php
foreach ($pets as $pet) { 
  echo $pet->talk(false); 
} 

```

不需要不必要的`switch`/`case`块来包装我们的类，我们只需使用软件设计来让事情变得更容易。

抽象类的工作方式类似，不同之处在于抽象类可以包含功能，而接口则不行。

需要注意的是，任何定义了一个或多个抽象类的类也必须被定义为抽象类。你不能有一个普通类定义抽象方法，但你可以在抽象类中有普通方法。让我们从重构我们的接口开始，将其变成一个抽象类：

```php
abstract class Animal 
{ 
  abstract public function eat(string $food) : bool; 

  abstract public function talk(bool $shout) : string; 

  public function walk(int $speed): bool { 
    if ($speed > 0) { 
      return true; 
    } else { 
      return false; 
    } 
  } 
} 

```

你可能已经注意到我还添加了一个`walk`方法作为一个普通的、非抽象的方法；这是一个标准的方法，可以被任何继承父抽象类的类使用或扩展。它们已经有了它们的实现。

请注意，无法实例化抽象类（就像无法实例化接口一样）。相反，我们必须扩展它。

所以，在我们的`Cat`类中让我们移除以下内容：

```php
class Cat implements Animal 

```

我们将用以下代码替换它：

```php
class Cat extends Animal 

```

这就是我们需要重构的全部内容，以便让类扩展`Animal`抽象类。我们必须按照我们为接口所概述的那样在类中实现抽象函数，另外我们可以使用普通函数而不需要实现它们：

```php
**$whiskers = new Cat();**
**$whiskers->walk(1);**

```

从 PHP 5.4 开始，也可以在一个系统中实例化一个类并访问它的属性。PHP.net 宣传它为：*在实例化时添加类成员访问，例如（new Foo）->bar()。*你也可以对单个属性这样做，例如，（new Cat）->legs。在我们的例子中，我们可以这样使用它：

```php
(new \IcyApril\ChapterOne\Cat())->walk(1); 

```

再来回顾一下 PHP 如何实现 OOP 的其他一些要点，类声明之前或者函数声明之前的`final`关键字意味着在它们被定义之后不能覆盖这样的类或函数。

所以，让我们尝试扩展一个我们称之为`final`的类：

```php
final class Animal 
{ 
  public function walk() 
  { 
    return "walking..."; 
  } 
} 

class Cat extends Animal 
{ 
} 

```

这将导致以下输出：

```php
Fatal error: Class Cat may not inherit from final class (Animal) 

```

同样地，让我们在函数级别做同样的事情：

```php
class Animal 
{ 
  final public function walk() 
  { 
    return "walking..."; 
  } 
} 

class Cat extends Animal 
{ 
  public function walk () { 
    return "walking with tail wagging..."; 
  } 
} 

```

这将导致以下输出：

```php
Fatal error: Cannot override final method Animal::walk() 

```

## Trait（多重继承）

**Trait**被引入到 PHP 中作为引入水平重用的机制。PHP 传统上是一种单继承语言，因为你不能将多个类继承到一个脚本中。

传统的多重继承是一个备受软件工程师鄙视的有争议的过程。

让我先给你举一个使用 Trait 的例子；让我们定义一个抽象的`Animal`类，我们想要将其扩展到另一个类中：

```php
class Animal 
{ 
  public function walk() 
  { 
    return "walking..."; 
  } 
} 

class Cat extends Animal 
{ 
  public function walk () { 
    return "walking with tail wagging..."; 
  } 
} 

```

现在假设我们有一个函数来为我们的类命名，但我们不希望它适用于所有扩展`Animal`类的类，我们希望它适用于某些类，无论它们是否继承了抽象`Animal`类的属性。

所以我们定义了我们的函数如下：

```php
function setFirstName(string $name): bool 
{ 
  $this->firstName = $name; 
  return true; 
} 

function setLastName(string $name): bool 
{ 
  $this->lastName = $name; 
  return true; 
} 

```

现在的问题是，除了使用水平重用之外，没有地方可以放置它们，除了复制和粘贴不同的代码片段或者诉诸于使用条件继承。这就是 Trait 出手的地方；让我们首先把这些方法放在一个叫做`Name`的 Trait 中。

```php
trait Name 
{ 
  function setFirstName(string $name): bool 
  { 
    $this->firstName = $name; 
    return true; 
  } 

  function setLastName(string $name): bool 
  { 
    $this->lastName = $name; 
    return true; 
  } 
} 

```

现在我们已经定义了我们的 Trait，我们可以告诉 PHP 在我们的`Cat`类中使用它：

```php
class Cat extends Animal 
{ 
  use Name; 

  public function walk() 
  { 
    return "walking with tail wagging..."; 
  } 
} 

```

注意`Name`语句的使用？这就是魔法发生的地方。现在你可以调用 Trait 中的函数而不会出现任何问题：

```php
**$whiskers = new Cat();
$whiskers->setFirstName('Paul');
echo $whiskers->firstName;**

```

把所有东西放在一起，新的代码块看起来是这样的：

```php
trait Name 
{ 
  function setFirstName(string $name): bool 
  { 
    $this->firstName = $name; 
    return true; 
  } 

  function setLastName(string $name): bool 
  { 
    $this->lastName = $name; 
    return true; 
  } 
} 

class Animal 
{ 
  public function walk() 
  { 
    return "walking..."; 
  } 
} 

class Cat extends Animal 
{ 
  use Name; 

  public function walk() 
  { 
    return "walking with tail wagging..."; 
  } 
} 

$whiskers = new Cat(); 
$whiskers->setFirstName('Paul'); 
echo $whiskers->firstName;  

```

## 标量类型提示

让我借此机会向你介绍一个名为**标量类型提示**的 PHP 7 概念；它允许你定义返回类型（是的，我知道这严格来说不属于 OOP 的范围；接受它吧）。

让我们定义一个函数，如下所示：

```php
function addNumbers (int $a, int $b): int 
{ 
  return $a + $b; 
} 

```

让我们来看看这个函数；首先，您会注意到在每个参数之前，我们定义了我们想要接收的变量类型；在这种情况下，它是 int（或整数）。接下来，您会注意到在函数定义之后有一些代码`：int`，它定义了我们的返回类型，因此我们的函数只能接收一个整数。

如果您没有提供正确类型的变量作为函数参数，或者没有从函数中返回正确类型的变量；您将收到`TypeError`异常。在严格模式下，如果启用了严格模式，并且您还提供了不正确数量的参数，PHP 也将抛出`TypeError`异常。

在 PHP 中也可以定义`strict_types`；让我解释为什么您可能想要这样做。没有`strict_types`，PHP 将尝试在非常有限的情况下自动将变量转换为定义的类型。例如，如果您传递一个仅包含数字的字符串，它将被转换为整数，然而，一个非数字的字符串将导致`TypeError`异常。一旦您启用了`strict_types`，这一切都会改变，您将不再具有这种自动转换行为。

以前的例子中，没有`strict_types`，您可以执行以下操作：

```php
**echo addNumbers(5, "5.0");**

```

在启用`strict_types`后再次尝试，您将发现 PHP 抛出`TypeError`异常。

这个配置只适用于单个文件，将其放在包含其他文件之前不会导致这个配置被继承到那些文件中。PHP 选择这条路线有多个好处；它们在实现标量类型提示的 RFC 版本 0.5.3 中非常清楚地列出，名为**PHP RFC:标量类型声明**。您可以通过访问[`www.wiki.php.net`](http://www.wiki.php.net)（维基，而不是主要的 PHP 网站）并搜索`scalar_type_hints_v5`来了解更多信息。

为了启用它，请确保将其作为 PHP 脚本中的第一个语句：

```php
declare(strict_types=1); 

```

除非您将`strict_types`定义为 PHP 脚本中的第一个语句；不允许对此定义进行其他用途。实际上，如果您尝试稍后定义它，您的脚本 PHP 将抛出致命错误。

当然，出于对这本书的愤怒的 PHP 核心狂热分子的利益，我应该提到，还有其他有效的类型可以用于类型提示。例如，PHP 5.1.0 引入了数组，PHP 5.0.0 引入了开发人员可以对其自己的类进行类型提示的功能。

让我给你一个快速的实例，说明这在实践中是如何工作的，假设我们有一个`Address`类：

```php
class Address 
{ 
  public $firstLine; 
  public $postcode; 
  public $country; 

  public function __construct(string $firstLine, string $postcode, string $country) 
  { 
    $this->firstLine = $firstLine; 
    $this->postcode = $postcode; 
    $this->country = $country; 
  } 
} 

```

然后我们可以对我们注入到`Customer`类中的`Address`类进行类型提示：

```php
class Customer 
{ 
  public $name; 
  public $address; 

  public function __construct($name, Address $address) 
  { 
    $this->name = $name; 
    $this->address = $address; 
  } 
} 

```

这就是它如何全部组合在一起：

```php
**$address = new Address('10 Downing Street', 'SW1A 2AA', 'UK');
$customer = new Customer('Davey Cameron', $address);
var_dump($customer);**

```

## 限制对私有/受保护属性的调试访问

如果您定义一个包含私有或受保护变量的类，您会注意到一个奇怪的行为，如果您`var_dump`该类的对象。您会注意到，当您将对象包装在`var_dump`中时，它会显示所有变量；无论它们是受保护的、私有的还是公共的。

PHP 将`var_dump`视为内部调试函数，这意味着所有数据都变得可见。

幸运的是，这方面有一个解决方法。PHP 5.6 引入了`__debugInfo`魔术方法。在类中以双下划线开头的函数代表魔术方法，并与特殊功能相关联。每当您尝试`var_dump`一个设置了`__debugInfo`魔术方法的对象时，`var_dump`将被该函数调用的结果覆盖。

让我向您展示这在实践中是如何工作的，让我们从定义一个类开始：

```php
class Bear { 
  private $hasPaws = true; 
} 

```

让我们实例化这个类：

```php
**$richard = new Bear();**

```

现在，如果我们尝试访问私有变量`hasPaws`，我们将收到致命错误：

```php
**echo $richard->hasPaws;**

```

前面的调用将导致抛出以下致命错误：

```php
**Fatal error: Cannot access private property Bear::$hasPaws**

```

这是预期的输出，我们不希望`private`属性在其对象外部可见。也就是说，如果我们用`var_dump`包装对象如下：

```php
var_dump($richard); 

```

然后我们会得到以下输出：

```php
object(Bear)#1 (1) { 
   ["hasPaws":"Bear":private]=> 
   bool(true) 
} 

```

正如您所看到的，我们的`private`属性被标记为`private`，但它仍然是可见的。那么我们该如何防止这种情况发生呢？

因此，让我们重新定义我们的类如下：

```php
class Bear { 
  private $hasPaws = true; 
  public function __debugInfo () { 
    return call_user_func('get_object_vars', $this); 
  } 
} 

```

现在，当我们实例化我们的类并`var_dump`生成的对象时，我们会得到以下输出：

```php
object(Bear)#1 (0) {  
} 

```

现在，整个脚本看起来是这样的，您会注意到我添加了一个额外的`public`属性，名为`growls`，我将其设置为`true`：

```php
<?php 
class Bear { 
  private $hasPaws = true; 
  public $growls = true; 

  public function __debugInfo () { 
    return call_user_func('get_object_vars', $this); 
  } 
} 
$richard = new Bear(); 
var_dump($richard); 

```

如果我们要`var_dump`这个脚本（同时使用`public`和`private`属性），我们会得到以下输出：

```php
object(Bear)#1 (1) { 
  ["growls"]=> 
  bool(true) 
} 

```

正如您所看到的，只有`public`属性是可见的。那么从这个小实验中的故事的道德是什么呢？首先，`var_dumps`暴露了对象中的私有和受保护属性，其次，这种行为是可以被覆盖的。

# 使用 Composer 设置环境

**Composer**是 PHP 的一个依赖管理器，受 Node 的 NPM 和 Bundler 的强烈启发。它现在已经成为多个 PHP 项目的重要组成部分，包括 Laravel 和 Symfony。然而，对我们来说它有用的原因是，它包含符合 PSR-0 和 PSR-4 标准的自动加载功能。您可以从[`getcomposer.org`](http://getcomposer.org)下载并安装 Composer。

### 注意

要在 Mac OS X 或 Linux 上全局安装 Composer，首先可以运行安装程序：

**curl -sS https://getcomposer.org/installer | php**

然后您可以将 Composer 移动到全局安装：

`**mv composer.phar /usr/local/bin/composer**`

如果前面的命令由于权限问题而失败，请重新运行命令，但在开头加上 `sudo`。在输入命令后，您将被要求输入密码，只需输入密码并按 *Enter*。

一旦您按照上述步骤安装了 Composer，您可以通过运行`composer`命令来运行它。

要在 Windows 上安装 Composer，最简单的方法是直接在 Composer 网站上运行安装程序；目前您可以在以下位置找到它：

[`getcomposer.org/Composer-Setup.exe`](https://getcomposer.org/Composer-Setup.exe)。

Composer 相当容易更新，只需运行此命令：

`**Composer self-update**`

Composer 通过使用名为`composer.json`的文件中的配置来工作，在这里您可以概述外部依赖项和自动加载样式。一旦 Composer 安装了此文件中列出的依赖项，它将编写一个`composer.lock`文件，其中详细说明了它安装的确切版本。在使用版本控制时，重要的是要提交此文件（以及`composer.json`文件），如果您使用 Git，则不要将其添加到您的`.gitignore`文件中。这非常重要，因为锁定文件详细说明了在版本控制系统中特定时间安装的软件包的确切版本。但是，您可以排除一个名为`vendor`的目录，稍后我会解释它的作用。

让我们首先在项目目录中创建一个名为`composer.json`的文件。这个文件是以 JSON 格式结构化的，所以让我提醒您一下 JSON 的工作原理：

+   JSON 由数据的键/值对组成，可以将其视为在文件中定义一组变量。

+   键值对用逗号分隔，例如，`"key" : "value"`

+   花括号包含对象

+   方括号包含数组

+   多个数据必须用逗号分隔，不要在数据末尾留下逗号

+   包括字符串的键和值必须用引号括起来

+   反斜杠`\`是转义键

现在我们可以将以下标记添加到`composer.json`文件中：

```php
{ 
  "autoload": { 
    "psr-4": { 
      "IcyApril\\ChapterOne": "src/" 
    } 
  } 
} 

```

所以让我解释一下这个文件的作用；它告诉 Composer 将`src/`目录中的所有内容自动加载到`IcyApril\ChapterOne`命名空间中，使用 PSR-4 标准。

那么，下一步是创建我们的`src`目录，其中包括我们想要自动加载的代码。搞定了吗？好的，现在让我们打开命令行，并进入我们放置`composer.json`文件的目录。

为了在您的项目中安装`composer.json`文件中的所有内容，只需运行`composer install`命令。对于后续更新，`composer update`命令将根据`composer.json`中定义的所有依赖项的最新版本进行更新。如果您不想这样做，还有另一种选择；运行`composer dump-autoload`命令将仅重新生成需要包含在项目中的 PSR-0/PSR-4 类的列表（例如，您添加、删除或重命名了一些类）。

现在让我来介绍一下你实际上将如何创建一个类。所以，在我们的项目中创建一个`src`目录，在该`src`目录中创建一个名为`Book`的新类。您可以通过创建一个名为`Book.php`的文件来实现这一点。在该文件中，添加类似以下内容：

```php
<?php 
namespace IcyApril\ChapterOne; 

class Book 
{ 
  public function __construct() 
  { 
    echo "Hello world!"; 
  } 
} 

```

这是一个标准类，只是我们定义了一个构造函数，当实例化类时将会输出`Hello world!`。

您可能已经注意到，我们遵循了一些命名约定；首先，PSR-1 标准声明类名必须以 StudlyCaps 形式声明。PSR-2 有一些额外的要求；举个例子：四个空格而不是制表符，命名空间或使用声明后面有一个空格，以及将括号放在新行上。如果您还没有阅读这些标准，那么花时间阅读这些标准绝对是值得的。您可能不同意每个标准，您可能对如何格式化自己的代码有主观偏好；我的建议是将这些偏好放在一边，为了更大的利益。通过利用 PSR 标准标准化的代码在共同的代码库上进行协作时具有巨大的优势。通过 PHP-FIG 组织等外部标准的好处是，您的 IDE 中已经预先构建了您的配置（例如，PHPStorm 支持 PSR-1/PSR-2）。不仅如此，当涉及到格式化参数时，您有一个具体的公正文件，概述了应该如何做事情，这对于在代码审查期间阻止宗教性的代码格式化争论非常有益。

既然我们已经创建了类，我们可以继续运行`composer dump-autoload`命令，以刷新我们的自动加载程序脚本。

所以，我们已经配置了 Composer 自动加载程序，也有了一个测试类可以操作，但下一个问题是我们如何实现这一点。所以，让我们继续实现这一点。在我们实现`composer.json`文件的同一目录中，让我们添加我们的`index.php`文件。

在您放入 PHP 开标签后的一行，我们需要引入我们的自动加载程序脚本：

```php
require_once('vendor/autoload.php'); 

```

然后我们可以实例化我们的`Book`类：

```php
new \IcyApril\ChapterOne\Book(); 

```

设置您的 Web 服务器，将文档根目录指向我们创建的文件夹，将您选择的 Web 服务器指向您的 Web 浏览器，然后您应该在屏幕上看到 Hello world！现在您可以拆开代码并进行操作。

完成的代码示例可与本书一起使用，因此您可以直接从那里打开它并进行操作，以防您需要帮助调试您的代码。

无论您的类是抽象类还是纯接口，在自动加载时我们都将它们视为类。

# 四人帮（GoF）

建筑师克里斯托弗·亚历山大提到了模式如何用于解决常见的设计问题，最初记录了这一概念。这个想法来自亚历山大；他提出设计问题可以严格记录，以及它们的解决方案。设计模式最显著地应用于解决软件设计中的架构问题。

用克里斯托弗·亚历山大的话说：

> “这种语言的元素是被称为模式的实体。每个模式描述了我们环境中反复出现的问题，然后以这样一种方式描述了解决这个问题的核心，以便您可以使用这个解决方案一百万次，而不必重复一样的方式。”

亚历山大写了一本自己的书，早于四人帮的书，名为《模式语言》。在这本书中，亚历山大创造了自己的语言，他创造了“模式语言”这个词来描述这一点；这种语言是由建筑模式的基本构建模块形成的。通过利用这些建筑模式，该书提出普通人可以将这种语言用作改善他们的社区和城镇的框架。

书中记录的一种这样的模式是“模式 12”，被称为 7000 人的社区；书中通过以下方式记录了这种模式：

> “个人在超过 5,000-10,000 人的任何社区中都没有有效的发言权。”

通过使用这样的问题及其记录的解决方案，该书最终形成了模式，这些模式旨在成为改善社区的基本构建模块。

正如我所提到的，亚历山大先于四人帮；但他的工作对于播种软件设计模式的种子至关重要。

现在，让我们直接转向被称为“四人帮”的作者。

不，我们不是指 1981 年英国工党的叛逃者，也不是指一支英国后朋克乐队；但我们谈论的是一本名为《设计模式：可复用面向对象软件的元素》的书的作者。这本书在软件开发领域具有很高的影响力，在软件工程领域广为人知。

在书的第一章中，作者们从自己的个人经验讨论了面向对象的软件开发；这包括争论软件开发人员应该为接口而不是实现编程。这最终导致代码最终利用了面向对象编程的核心功能。

人们普遍误解这本书只包含四种设计模式，这是不正确的；它涵盖了来自三个基本类别的 23 种设计模式。

让我们来看看这些类别是什么：

+   创建型

+   结构型

+   行为

所以让我们逐一解释这些。

## 创建型设计模式

创建型设计模式涉及对象本身的创建。在不使用设计模式的情况下基本实例化类可能会导致不必要的复杂性，也可能会导致重大的设计问题。

创建型设计模式的主要用途是将类的实例化与该实例的使用分开。不使用创建型设计模式可能意味着您的代码更难理解和测试。

### 依赖注入

依赖注入是一种过程，通过这种过程，您可以直接将应用程序需要的依赖项输入到对象本身中。

John Munsch 在 Stack Overflow 上留下了一个名为“五岁孩子的依赖注入”的答案，这个答案被重新发表在书籍《Mark Seeman's Dependency Injection in .NET》中：

### 提示

当你自己去冰箱里拿东西时，可能会引起问题。你可能会把门打开，你可能会拿到爸爸妈妈不想让你拿的东西。你甚至可能在找我们根本没有或者已经过期的东西。

### 提示

你应该说出你的需求，“我需要午餐时喝点什么”，然后我们会确保你坐下来吃饭时有东西喝。

在编写类时，自然会使用其他依赖项；也许是数据库模型类。因此，通过依赖注入，类不是在自身中创建其数据库模型，而是在对象外部创建它并注入它。简而言之，我们将客户端的行为与客户端的依赖项分开。

在考虑依赖注入时，让我们概述涉及的四个独立角色：

+   要注入的服务

+   依赖于被注入服务的客户端

+   确定客户端如何使用服务的接口

+   负责实例化服务并将其注入客户端的注入器

## 结构设计模式

结构设计模式相当容易解释，它们充当实体之间的连接器。它作为基本类如何组合形成更大实体的蓝图，所有结构设计模式都涉及对象之间的互连。

## 行为设计模式

行为设计模式用于解释对象之间的交互方式；它们如何在对象之间发送消息，以及如何将各种任务的步骤分配给不同的类。

结构模式描述设计的静态架构；行为模式更加灵活，描述了一个流动的过程。

## 架构模式

这不是严格意义上的*设计模式*（但四人帮在他们的书中没有涵盖架构模式）；但由于 PHP 的面向 Web 的特性，它对 PHP 开发人员非常相关。架构模式通过解决计算机系统中的各种不同约束来解决性能限制、高可用性以及业务风险的最小化。

大多数开发人员在涉及 Web 框架时会熟悉模型-视图-控制器架构，最近其他架构开始出现；例如，微服务架构通过一组独立且相互连接的 RESTful API 工作。一些人认为微服务将问题从软件开发层转移到系统架构层。与微服务相反，通常被称为单块架构，是所有代码都集中在一个应用程序中。

# 总结

在本章中，我们复习了一些 PHP 原则，包括面向对象编程原则。我们还复习了一些 PHP 语法基础。我们已经看到您可以如何在 PHP 中使用 Composer 进行依赖管理。除此之外，我们还讨论了 PSR 标准以及如何在自己的代码中实现它们，以使您的代码更易于他人阅读，并且符合其他重要标准（无论是自动加载还是 HTTP 消息传递）。最后，我们介绍了设计模式和四人帮以及设计模式背后的历史。
