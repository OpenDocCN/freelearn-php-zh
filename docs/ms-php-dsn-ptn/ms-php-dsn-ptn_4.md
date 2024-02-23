# 第四章：结构设计模式

结构设计模式提供了创建类结构的不同方式；例如，这可以是我们如何使用封装来从较小的对象创建更大的对象。它们存在的目的是通过允许我们识别简单的方式来实现实体之间的关系，从而简化设计。

在上一章中，我们介绍了创造模式如何用于确定如何创建对象；而结构模式可以确定类之间的结构和关系。

在简短介绍了敏捷软件架构之后，本章将涵盖以下主题：

+   装饰者模式

+   类适配器模式

+   对象适配器模式

+   享元模式

+   组合模式

+   桥接模式

+   代理模式

+   外观模式

# 敏捷软件架构

许多组织正在倾向于采用敏捷形式的项目管理。这给架构师的角色带来了新的关注；事实上，一些人认为敏捷和架构是相互冲突的。敏捷宣言的最初签署者之一 Martin Fowler 和 Robert Cecil Martin 对这一观点持有强烈反对意见。事实上，福勒明确澄清了敏捷宣言虽然对大量的事先设计（例如 Prince2 中看到的类型）持敌对态度，但并不排斥事先设计本身。

计算机科学家 Allen Holub 也持有类似观点。敏捷侧重于做对用户有用的软件，而不是仅仅对销售人员有用的软件。为了使软件长期有用，它必须是可适应、可扩展和可维护的。

福勒还对软件开发团队中的架构师有了一个愿景。他指出，不可逆转的软件很可能会在以后带来最大的麻烦，这就是架构决策必须存在的地方。此外，他声称架构师的角色应该是寻求使这些决策可逆转，从而完全减轻问题。

在许多大规模软件部署中，可能会使用“我们已经到了无法回头的地步”的说法。在“无法回头”的地步之后，将部署恢复到原始状态变得不可行。软件有自己的“无法回头”的地步，当软件变得更难重写而不是简单重建时，就会成为事实。虽然软件可能不会达到这种“无法回头”的最坏情况，但随着可维护性困难的增加，会带来商业困难。

福勒还指出，在许多情况下，软件架构师甚至不检查软件是否符合其原始设计。通过与架构师进行配对编程，以及架构师审查代码更改（即拉取请求），他们可以获得理解，以便向开发人员提供反馈，并减轻进一步的技术债务。

在本书中，您可能会注意到缺少 UML；这是因为我认为这里不需要 UML。我的意思是，我们都在用 PHP 说话，对吧？不过你可能会发现 UML 在您的团队中很有用。

架构过程通常会产生可交付物；我们称这个可交付物为“工件”。在敏捷团队中，这些工件可能以渐进式方式开发，而不是事先产品，但在敏捷环境中完全可以进行架构设计。

事实上，我认为架构使在敏捷环境中工作变得更容易。当编程到接口或抽象层时，更容易替换类；在敏捷环境中，需求可能会发生变化，这意味着可能需要替换类。软件只有对最终客户有用时才有用。敏捷可以帮助实现这一点，但为了实现敏捷，您的代码必须是适应性的。拥有出色的架构对此至关重要。

当我们编写代码时，我们应该采取防御性的编码方式。然而，对手并不是敌人，而是我们自己。破坏可靠代码的最快方式之一是编辑它以使其变得脆弱。

# 装饰器

装饰器只是在不影响同一类的其他对象行为的情况下，为单个类添加额外功能的内容。

单一责任原则，由 Robert C. Martin（我在本章开头介绍过）简单地表述为“一个类应该只有一个改变的原因”。

该原则规定每个模块或类应该有一个单一的责任，并且该责任应该完全由该类封装。类的所有服务都应该与该责任保持一致。Martin 通过以下方式总结了这一责任：

> “指定给唯一的参与者的责任，表示其对于唯一的业务任务的责任。”

通过使用装饰器设计模式，我们能够确保功能在具有独特关注领域的类之间进行划分，从而遵守单一责任原则。

让我们首先声明我们的`Book`接口。这是我们期望我们的书能够产生的内容：

```php
<?php 

interface Book 
{ 
  public function __construct(string $title, string $author, string $contents); 

  public function getTitle(): string; 

  public function getAuthor(): string; 

  public function getContents(): string; 
} 

```

然后我们可以声明我们的`EBook.php`类。这是我们将用`PrintBook`类装饰的类：

```php
<?php 

class EBook implements Book 
{ 

  public $title; 
  public $author; 
  public $contents; 

  public function __construct(string $title, string $author, string $contents) 
  { 
    $this->title = $title; 
    $this->author = $author; 
    $this->contents = $contents; 
  } 

  public function getTitle(): string 
  { 
    return $this->contents; 
  } 

  public function getAuthor(): string 
  { 
    return $this->author; 
  } 

  public function getContents(): string 
  { 
    return $this->contents; 
  } 
} 

```

现在我们可以声明我们的`PrintBook`类。这是我们用来装饰`EBook`类的内容：

```php
<?php 

class PrintBook implements Book 
{ 

  public $eBook; 

  public function __construct(string $title, string $author, string $contents) 
  { 
    $this->eBook = new EBook($title, $author, $contents); 
  } 

  public function getTitle(): string 
  { 
    return $this->eBook->getTitle(); 
  } 

  public function getAuthor(): string 
  { 
    return $this->eBook->getAuthor(); 
  } 

  public function getContents(): string 
  { 
    return $this->eBook->getContents(); 
  } 

  public function getText(): string 
  { 
    $contents = $this->eBook->getTitle() . " by " . $this->eBook->getAuthor(); 
    $contents .= "\n"; 
    $contents .= $this->eBook->getContents(); 

    return $contents; 
  } 
} 

```

现在让我们用我们的`index.php`文件来测试所有这些。

```php
<?php 

require_once('Book.php'); 
require_once('EBook.php'); 
$PHPBook = new EBook("Mastering PHP Design Patterns", "Junade Ali", "Some contents."); 

require_once('PrintBook.php'); 
$PHPBook = new PrintBook("Mastering PHP Design Patterns", "Junade Ali", "Some contents."); 
echo $PHPBook->getText(); 

```

输出如下：

```php
Some contents. by Junade Ali 
Some contents. 

```

# 适配器

适配器模式有两种类型。在可能的情况下，我更偏向于对象适配器而不是类适配器；我稍后会详细解释这一点。

适配器模式允许现有的类与其不匹配的接口一起使用。它经常用于允许现有的类与其他类一起工作，而无需修改它们的源代码。

这在使用具有各自接口的第三方库的多态设置中可能非常有用。

基本上，适配器帮助两个不兼容的接口一起工作。通过将一个类的接口转换为客户端期望的接口，否则不兼容的类可以被使得一起工作。

## 类适配器

在类适配器中，我们使用继承来创建一个适配器。一个类（适配器）可以继承另一个类（被适配者）；使用标准继承，我们能够为被适配者添加额外功能。

假设我们有一个`ATM`类，在我们的`ATM.php`文件中：

```php
<?php 

class ATM 
{ 
  private $balance; 

  public function __construct(float $balance) 
  { 
    $this->balance = $balance; 
  } 

  public function withdraw(float $amount): float 
  { 
    if ($this->reduceBalance($amount) === true) { 
      return $amount; 
    } else { 
      throw new Exception("Couldn't withdraw money."); 
    } 
  } 

  protected function reduceBalance(float $amount): bool 
  { 
    if ($amount >= $this->balance) { 
      return false; 
    } 

    $this->balance = ($this->balance - $amount); 
    return true; 
  } 

  public function getBalance(): float 
  { 
    return $this->balance; 
  } 
} 

```

让我们创建我们的`ATMWithPhoneTopUp.php`来形成我们的适配器：

```php
<?php 

class ATMWithPhoneTopUp extends ATM 
{ 
  public function getTopUp(float $amount, int $time): string 
  { 
    if ($this->reduceBalance($amount) === true) { 
      return $this->generateTopUpCode($amount, $time); 
    } else { 
      throw new Exception("Couldn't withdraw money."); 
    } 
  } 

  private function generateTopUpCode(float $amount, int $time): string 
  { 
    return $amount . $time . rand(0, 10000); 
  } 
} 

```

让我们将所有这些内容包装在一个`index.php`文件中：

```php
<?php 

require_once('ATM.php'); 

$atm = new ATM(500.00); 
$atm->withdraw(50); 
echo $atm->getBalance(); 
echo "\n"; 

require_once('ATMWithPhoneTopUp.php'); 

$adaptedATM = new ATMWithPhoneTopUp(500.00); 
echo "Top-up code: " . $adaptedATM->getTopUp(50, time()); 
echo "\n"; 
echo $adaptedATM->getBalance(); 

```

现在我们已经将初始的`ATM`类调整为生成充值码，我们现在可以利用这个新的充值功能。所有这些的输出如下：

```php
450 
Top-up code: 5014606939121598 
450 

```

请注意，如果我们想要适应多个被适配者，这在 PHP 中将会很困难。

在 PHP 中，多重继承是不可能的，除非你使用 Traits。在这种情况下，我们只能使一个类适应另一个类的接口。

我们不使用这种方法的另一个关键架构原因是，通常更倾向于优先使用组合而不是继承（正如复用组合原则所描述的）。

为了更详细地探讨这一原则，我们需要看看对象适配器。

## 对象适配器

复用组合原则规定，类应该通过它们的组合实现多态行为和代码复用。

通过应用这一原则，当类想要实现特定功能时，应该包含其他类的实例，而不是从基类或父类继承功能。

因此，四人帮提出了以下观点：

> “更偏向于‘对象组合’而不是‘类继承’。”

为什么这个原则如此重要？考虑我们上一个例子，我们在那里使用了类继承；在这种情况下，我们无法保证我们的适配器是否符合我们想要的接口。如果父类暴露了我们不想要适配器的函数会怎么样？组合给了我们更多的控制。

通过组合而不是继承，我们能够更好地支持面向对象编程中如此重要的多态行为。

假设我们有一个生成保险费的类。它根据客户希望如何支付保险费提供月度保费和年度保费。通过年度支付，客户可以节省相当于半个月的金额：

```php
<?php 

class Insurance 
{ 
  private $limit; 
  private $excess; 

  public function __construct(float $limit, float $excess) 
  { 
    if ($excess >= $limit) { 
      throw New Exception('Excess must be less than premium.'); 
    } 

    $this->limit = $limit; 
    $this->excess = $excess; 
  } 

  public function monthlyPremium(): float 
  { 
    return ($this->limit-$this->excess)/200; 
  } 

  public function annualPremium(): float 
  { 
    return $this->monthlyPremium()*11.5; 
  } 
} 

```

假设市场比较工具多态地使用诸如前面提到的类来实际上计算来自多个不同供应商的保险报价；他们使用这个接口来做到这一点：

```php
<?php 

interface MarketCompare 
{ 
  public function __construct(float $limit, float $excess); 
  public function getAnnualPremium(); 
  public function getMonthlyPremium(); 
} 

```

因此，我们可以使用这个接口来构建一个对象适配器，以确保我们的`Insurance`类，我们的保费生成器，符合市场比较工具所期望的接口：

```php
<?php 

class InsuranceMarketCompare implements MarketCompare 
{ 
  private $premium; 

  public function __construct(float $limit, float $excess) 
  { 
    $this->premium = new Insurance($limit, $excess); 
  } 

  public function getAnnualPremium(): float 
  { 
    return $this->premium->annualPremium(); 
  } 

  public function getMonthlyPremium(): float 
  { 
    return $this->premium->monthlyPremium(); 
  } 
} 

```

注意类实际上是如何实例化自己的类以适应它所尝试适配的内容。

然后适配器将这个类存储在一个`private`变量中。然后我们使用这个对象在`private`变量中代理请求。

适配器，无论是类适配器还是对象适配器，都应该充当粘合代码。我的意思是适配器不应执行任何计算或计算，它们只是在不兼容的接口之间充当代理。

将逻辑保持在我们的粘合代码之外，并将逻辑留给我们正在适应的代码是标准做法。如果在这样做时，我们遇到单一责任原则，我们需要适应另一个类。

正如我之前提到的，在类适配器中适配多个类实际上是不可能的，所以你要么必须将这样的逻辑包装在一个 Trait 中，要么我们需要使用对象适配器，比如我们正在讨论的这个。

让我们试试这个适配器。我们将通过编写以下`index.php`文件来看看我们的新类是否符合预期的接口：

```php
<?php 

require_once('Insurance.php'); 

$quote = new Insurance(10000, 250); 
echo $quote->monthlyPremium(); 
echo "\n"; 

require_once('MarketCompare.php'); 
require_once('InsuranceMarketCompare.php'); 

$quote = new InsuranceMarketCompare(10000, 250); 
echo $quote->getMonthlyPremium(); 
echo "\n"; 
echo $quote->getAnnualPremium(); 

```

输出应该看起来像这样：

```php
48.75 
48.75 
560.625 

```

与类适配器方法相比，这种方法的主要缺点是，我们必须实现公共方法，即使这些方法只是转发方法。

# FlyWeight

就像在现实生活中，不是所有的对象都容易创建，有些可能会占用过多的内存。FlyWeight 设计模式可以通过尽可能与类似对象共享尽可能多的数据来帮助我们最小化内存使用。

这种设计模式在大多数 PHP 应用程序中的使用有限，但是了解它在极端有用的情况下仍然是值得的。

假设我们有一个带有`draw`方法的`Shape`接口：

```php
<?php 

interface Shape 
{ 
  public function draw(); 
} 

```

让我们创建一个实现这个接口的`Circle`类。在实现这个过程中，我们建立了设置圆的位置和半径以及绘制它（打印出这些信息）的能力。注意颜色特征是如何在类外设置的。

这有一个非常重要的原因。在我们的例子中，颜色是与状态无关的；它是圆的固有部分。然而，圆的位置和大小是与状态相关的，因此是外部的。当需要时，外部状态信息被传递给 FlyWeight 对象；然而，固有选项与 FlyWeight 的每个过程无关。当我们讨论这个工厂是如何制作的时，这将更有意义。

这是重要的信息：

+   **外部**：状态属于对象的外部上下文，并在使用对象时输入。

+   **内在**：自然属于对象的状态，因此应该是永久的、不可变的（内部）或与上下文无关的。

考虑到这一点，让我们组合一个实现我们的`Shape`接口的实现。这是我们的`Circle`类：

```php
<?php 

class Circle implements Shape 
{ 

  private $colour; 
  private $x; 
  private $y; 
  private $radius; 

  public function __construct(string $colour) 
  { 
    $this->colour = $colour; 
  } 

  public function setX(int $x) 
  { 
    $this->x = $x; 
  } 

  public function setY(int $y) 
  { 
    $this->y = $y; 
  } 

  public function setRadius(int $radius) 
  { 
    $this->radius = $radius; 
  } 

  public function draw() 
  { 
    echo "Drawing circle which is " . $this->colour . " at [" . $this->x . ", " . $this->y . "] of radius " . $this->radius . "."; 
    echo "\n"; 
  } 
} 

```

有了这个，我们现在可以构建我们的`ShapeFactory`，它实际上实现了 FlyWeight 模式。当需要时，会实例化一个具有我们选择的颜色的对象，然后将其存储以供以后使用：

```php
<?php 

class ShapeFactory 
{ 
  private $shapeMap = array(); 

  public function getCircle(string $colour) 
  { 
    $circle = 'Circle' . '_' . $colour; 

    if (!isset($this->shapeMap[$circle])) { 
      echo "Creating a ".$colour." circle."; 
      echo "\n"; 
      $this->shapeMap[$circle] = new Circle($colour); 
    } 

    return $this->shapeMap[$circle]; 
  } 
} 

```

让我们在我们的`index.php`文件中演示这是如何工作的。

为了使这个工作，我们创建`100`个带有随机颜色的对象，放在随机位置：

```php
require_once('Shape.php'); 
require_once('Circle.php'); 
require_once('ShapeFactory.php'); 

$colours = array('red', 'blue', 'green', 'black', 'white', 'orange'); 

$factory = new ShapeFactory(); 

for ($i = 0; $i < 100; $i++) { 
  $randomColour = $colours[array_rand($colours)]; 

  $circle = $factory->getCircle($randomColour); 
  $circle->setX(rand(0, 100)); 
  $circle->setY(rand(0, 100)); 
  $circle->setRadius(100); 

  $circle->draw(); 
} 

```

现在，让我们来看一下输出。您可以看到我们画了 100 个圆，但我们只需要实例化少量圆，因为我们正在缓存相同颜色的对象以供以后使用：

```php
Creating a green circle. 
Drawing circle which is green at [29, 26] of radius 100\. 
Creating a black circle. 
Drawing circle which is black at [17, 64] of radius 100\. 
Drawing circle which is black at [81, 86] of radius 100\. 
Drawing circle which is black at [0, 73] of radius 100\. 
Creating a red circle. 
Drawing circle which is red at [10, 15] of radius 100\. 
Drawing circle which is red at [70, 79] of radius 100\. 
Drawing circle which is red at [13, 78] of radius 100\. 
Drawing circle which is green at [78, 27] of radius 100\. 
Creating a blue circle. 
Drawing circle which is blue at [38, 11] of radius 100\. 
Creating a orange circle. 
Drawing circle which is orange at [43, 57] of radius 100\. 
Drawing circle which is blue at [58, 65] of radius 100\. 
Drawing circle which is orange at [75, 67] of radius 100\. 
Drawing circle which is green at [92, 59] of radius 100\. 
Drawing circle which is blue at [53, 3] of radius 100\. 
Drawing circle which is black at [14, 33] of radius 100\. 
Creating a white circle. 
Drawing circle which is white at [84, 46] of radius 100\. 
Drawing circle which is green at [49, 61] of radius 100\. 
Drawing circle which is orange at [57, 44] of radius 100\. 
Drawing circle which is orange at [64, 33] of radius 100\. 
Drawing circle which is white at [42, 74] of radius 100\. 
Drawing circle which is green at [5, 91] of radius 100\. 
Drawing circle which is white at [87, 36] of radius 100\. 
Drawing circle which is red at [74, 94] of radius 100\. 
Drawing circle which is black at [19, 6] of radius 100\. 
Drawing circle which is orange at [70, 83] of radius 100\. 
Drawing circle which is green at [74, 64] of radius 100\. 
Drawing circle which is white at [89, 21] of radius 100\. 
Drawing circle which is red at [25, 23] of radius 100\. 
Drawing circle which is blue at [68, 96] of radius 100\. 
Drawing circle which is green at [74, 6] of radius 100\. 

```

您可能已经注意到了一些事情。我们正在存储我们正在重用的 FlyWeight 对象的缓存的方式是通过连接*Circle*_ 和颜色，例如*Circle_green*。显然，在这种情况下这是有效的，但有更好的方法；在 PHP 中，实际上可以为给定的对象获取唯一 ID。我们将在下一个模式中介绍这个。

# 组合

想象一个由单独歌曲和歌曲播放列表组成的音频系统。是的，播放列表由歌曲组成，但我们希望两者都被单独对待。两者都是音乐类型，都可以播放。

组合设计模式可以帮助我们；它允许我们忽略对象组合和单个对象之间的差异。它允许我们用相同或几乎相同的代码来处理两者。

让我们举个小例子；一首歌是我们*叶子*的例子，而播放列表是*组合*。`Music`是我们对播放列表和歌曲的抽象；因此，我们可以称之为我们的*组件*。所有这些的*客户端*是我们的`index.php`文件。

通过不区分叶节点和分支，我们的代码变得不那么复杂，因此也不那么容易出错。

让我们首先为我们的`Music`定义一个接口：

```php
<?php 

interface Music 
{ 
  public function play(); 
} 

```

现在让我们组合一些实现，首先是我们的`Song`类：

```php
<?php 

class Song implements Music 
{ 
  public $id; 
  public $name; 

  public function  __construct(string $name) 
  { 
    $this->id = uniqid(); 
    $this->name = $name; 
  } 

  public function play() 
  { 
    printf("Playing song #%s, %s.\n", $this->id, $this->name); 
  } 
} 

```

现在我们可以开始组合我们的`Playlist`类。在这个例子中，您可能注意到我使用一个名为`spl_object_hash`的函数在歌曲数组中设置键。当处理对象数组时，这个函数绝对是一个祝福。

这个函数的作用是为每个对象返回一个唯一的哈希值，只要对象没有被销毁，无论类的属性如何改变，它都保持一致。它提供了一种稳定的方式来寻址任意对象。一旦对象被销毁，哈希值就可以被重用于其他对象。

这个函数不会对对象的内容进行哈希处理；它只是显示内部句柄和句柄表指针。这意味着如果您更改对象的属性，哈希值不会改变。也就是说，它并不保证唯一性。如果一个对象被销毁，然后立即创建一个相同类的对象，您将得到相同的哈希值，因为 PHP 将在第一个类被取消引用和销毁后重用相同的内部句柄。

这将是真的，因为 PHP 可以使用内部句柄：

```php
var_dump(spl_object_hash(new stdClass()) === spl_object_hash(new stdClass())); 

```

然而，这将是错误的，因为 PHP 必须创建一个新的句柄：

```php
$object = new StdClass(); 
var_dump(spl_object_hash($object) === spl_object_hash(new stdClass())); 

```

现在让我们回到我们的`Playlist`类。让我们用它实现我们的`Music`接口；所以，这是类：

```php
<?php 

class Playlist implements Music 
{ 
  private $songs = array(); 

  public function addSong(Music $content): bool 
  { 
    $this->songs[spl_object_hash($content)] = $content; 
    return true; 
  } 

  public function removeItem(Music $content): bool 
  { 
    unset($this->songs[spl_object_hash($content)]); 
    return true; 
  } 

  public function play() 
  { 
    foreach ($this->songs as $content) { 
      $content->play(); 
    } 
  } 
} 

```

现在让我们把这一切放在我们的`index.php`文件中。我们在这里所做的是创建一些歌曲对象，其中一些我们将使用它们的`addSong`函数分配给一个播放列表。

因为播放列表的实现方式与歌曲相同，我们甚至可以使用`addSong`函数与其他播放列表一起使用（在这种情况下，最好将`addSong`函数重命名为`addMusic`）。

然后我们播放父播放列表。这将播放子播放列表，然后播放这些播放列表中的所有歌曲：

```php
<?php 

require_once('Music.php'); 
require_once('Playlist.php'); 
require_once('Song.php'); 

$songOne = new Song('Lost In Stereo'); 
$songTwo = new Song('Running From Lions'); 
$songThree = new Song('Guts'); 
$playlistOne = new Playlist(); 
$playlistTwo = new Playlist(); 
$playlistThree = new Playlist(); 
$playlistTwo->addSong($songOne); 
$playlistTwo->addSong($songTwo); 
$playlistThree->addSong($songThree); 
$playlistOne->addSong($playlistTwo); 
$playlistOne->addSong($playlistThree); 
$playlistOne->play(); 

```

当我们运行这个脚本时，我们可以看到预期的输出：

```php
Playing song #57106d5adb364, Lost In Stereo. 
Playing song #57106d5adb63a, Running From Lions. 
Playing song #57106d5adb654, Guts. 

```

# 桥接

桥接模式可能非常简单；它有效地允许我们将抽象与实现解耦，以便两者可以独立变化。

当类经常变化时，通过桥接接口和具体类，开发人员可以更轻松地变化他们的类。

让我们提出一个通用的信使接口，具有发送某种形式消息的能力，`Messenger.php`：

```php
<?php 

interface Messenger 
{ 
  public function send($body); 
} 

```

这个接口的一个具体实现是一个`InstantMessenger`应用程序，`InstantMessenger.php`：

```php
<?php 

class InstantMessenger implements Messenger 
{ 
  public function send($body) 
  { 
    echo "InstantMessenger: " . $body; 
  } 
} 

```

同样，我们可以用一个`SMS`应用程序`SMS.php`来做同样的事情：

```php
<?php 

class SMS implements Messenger 
{ 
  public function send($body) 
  { 
    echo "SMS: " . $body; 
  } 
} 

```

我们现在可以为物理设备，即发射器，创建一个接口，`Transmitter.php`：

```php
<?php 

interface Transmitter 
{ 
  public function setSender(Messenger $sender); 

  public function send($body); 
} 

```

我们可以通过使用`Device`类将实现其方法的设备与发射器解耦。`Device`类将`Transmitter`接口桥接到物理设备，`Device.php`：

```php
<?php 

abstract class Device implements Transmitter 
{ 
  protected $sender; 

  public function setSender(Messenger $sender) 
  { 
    $this->sender = $sender; 
  } 
} 

```

所以让我们组合一个具体的类来表示手机，`Phone.php`：

```php
<?php 

class Phone extends Device 
{ 
  public function send($body) 
  { 
    $body .= "\n\n Sent from a phone."; 

    return $this->sender->send($body); 
  } 
} 

```

让我们对`Tablet`做同样的事情。`Tablet.php`是：

```php
<?php 

class Tablet extends Device 
{ 
  public function send($body) 
  { 
    $body .= "\n\n Sent from a Tablet."; 

    return $this->sender->send($body); 
  } 
} 

```

最后，让我们把这一切都包装在一个`index.php`文件中：

```php
<?php 

require_once('Transmitter.php'); 
require_once('Device.php'); 
require_once('Phone.php'); 
require_once('Tablet.php'); 

require_once('Messenger.php'); 
require_once('SMS.php'); 
require_once('InstantMessenger.php'); 

$phone = new Phone(); 
$phone->setSender(new SMS()); 

$phone->send("Hello there!"); 

```

这个输出如下：

```php
SMS: Hello there! 

 Sent from a phone. 

```

# 代理模式

代理是一个仅仅是与其他东西接口的类。它可以是任何东西的接口；从网络连接、文件、内存中的大对象，或者其他太难复制的资源。

在我们的例子中，我们将简单地创建一个简单的代理，根据代理的实例化方式转发到两个对象中的一个。

访问一个简单的代理类允许客户端从一个对象中访问猫和狗的喂食器，具体取决于它是否已被实例化。

让我们首先定义一个`AnimalFeeder`的接口：

```php
<?php 

namespace IcyApril\PetShop; 

interface AnimalFeeder 
{ 
  public function __construct(string $petName); 

  public function dropFood(int $hungerLevel, bool $water = false): string; 

  public function displayFood(int $hungerLevel): string; 
} 

```

然后我们可以为猫和狗定义两个动物喂食器：

```php
<?php 

namespace IcyApril\PetShop\AnimalFeeders; 

use IcyApril\PetShop\AnimalFeeder; 

class Cat implements AnimalFeeder 
{ 
  public function __construct(string $petName) 
  { 
    $this->petName = $petName; 
  } 

  public function dropFood(int $hungerLevel, bool $water = false): string 
  { 
    return $this->selectFood($hungerLevel) . ($water ? ' with water' : ''); 
  } 

  public function displayFood(int $hungerLevel): string 
  { 
    return $this->selectFood($hungerLevel); 
  } 

  protected function selectFood(int $hungerLevel): string 
  { 
    switch ($hungerLevel) { 
      case 0: 
        return 'lamb'; 
        break; 
      case 1: 
        return 'chicken'; 
        break; 
      case 3: 
        return 'tuna'; 
        break; 
    } 
  } 
} 

```

这是我们的`AnimalFeeder`：

```php
<?php 

namespace IcyApril\PetShop\AnimalFeeders; 

class Dog 
{ 

  public function __construct(string $petName) 
  { 
    if (strlen($petName) > 10) { 
      throw new \Exception('Name too long.'); 
    } 

    $this->petName = $petName; 
  } 

  public function dropFood(int $hungerLevel, bool $water = false): string 
  { 
    return $this->selectFood($hungerLevel) . ($water ? ' with water' : ''); 
  } 

  public function displayFood(int $hungerLevel): string 
  { 
    return $this->selectFood($hungerLevel); 
  } 

  protected function selectFood(int $hungerLevel): string 
  { 
    if ($hungerLevel == 3) { 
      return "chicken and vegetables"; 
    } elseif (date('H') < 10) { 
      return "turkey and beef"; 
    } else { 
      return "chicken and rice"; 
    } 
  } 
} 

```

有了这个定义，我们现在可以创建我们的代理类，一个基本上使用构造函数来解密需要实例化的类，然后将所有函数调用重定向到这个类。为了重定向函数调用，使用`__call magic`方法。

看起来像这样：

```php
<?php 

namespace IcyApril\PetShop; 

class AnimalFeederProxy 
{ 
  protected $instance; 

  public function __construct(string $feeder, string $name) 
  { 
    $class = __NAMESPACE__ . '\\AnimalFeeders' . $feeder; 
    $this->instance = new $class($name); 
  } 

  public function __call($name, $arguments) 
  { 
    return call_user_func_array([$this->instance, $name], $arguments); 
  } 
} 

```

你可能已经注意到，我们必须在构造函数中手动创建带有命名空间的类。我们使用`__NAMESPACE__ magic`常量来找到当前命名空间，然后将其连接到类所在的特定子命名空间。请注意，我们必须使用另一个`\`来转义`\`，以便允许我们指定命名空间，而不让 PHP 将`\`解释为转义字符。

让我们构建我们的`index.php`文件，并利用代理类来构建对象：

```php
<?php 

require_once('AnimalFeeder.php'); 
require_once('AnimalFeederProxy.php'); 

require_once('AnimalFeeders/Cat.php'); 
$felix = new \IcyApril\PetShop\AnimalFeederProxy('Cat', 'Felix'); 
echo $felix->displayFood(1); 
echo "\n"; 
echo $felix->dropFood(1, true); 
echo "\n"; 

require_once('AnimalFeeders/Dog.php'); 
$brian = new \IcyApril\PetShop\AnimalFeederProxy('Dog', 'Brian'); 
echo $brian->displayFood(1); 
echo "\n"; 
echo $brian->dropFood(1, true); 

```

输出如下：

```php
chicken 
chicken with water 
turkey and beef 
turkey and beef with water 

```

那么你如何在现实中使用它呢？假设你从数据库中得到了一个包含动物类型和名称的对象的记录；你可以将这个对象传递给代理类的构造函数，并将其作为创建你的类的机制。

在实践中，当支持资源密集型对象时，这是一个很好的用例，除非客户端真正需要它们，否则你不一定想要实例化它们；对于资源密集型网络连接和其他类型的资源也是如此。

# 外观

外观（也称为*Façade*）设计模式是一件奇妙的事情；它们本质上是一个复杂系统的简单接口。外观设计模式通过提供一个单一的类来工作，这个类本身实例化其他类并提供一个简单的接口来使用这些函数。

使用这种模式时的一个警告是，由于类是在外观中实例化的，你本质上是将它所使用的类紧密耦合在一起。有些情况下你希望这样做，但也有些情况下你不希望。在你不希望这种行为的情况下，最好使用依赖注入。

我发现这在将一组糟糕的 API 封装成一个统一的 API 时非常有用。它减少了外部依赖，允许复杂性内部化；这个过程可以使你的代码更易读。

我将在一个粗糙的例子中演示这种模式，但这将使机制变得明显。

让我提议三个玩具工厂的类。

制造商（制造玩具的工厂）是一个简单的类，它根据一次制造多少个玩具来实例化：

```php
<?php 

class Manufacturer 
{ 
  private $capacity; 

  public function __construct(int $capacity) 
  { 
    $this->capacity = $capacity; 
  } 

  public function build(): string 
  { 
    return uniqid(); 
  } 
} 

```

Post 类（运输快递员）是一个简单的函数，用于从工厂发货玩具：

```php
<?php 

class Post 
{ 
  private $sender; 

  public function __construct(string $sender) 
  { 
    $this->sender = $sender; 
  } 

  public function dispatch(string $item, string $to): bool 
  { 
    if (strlen($item) !== 13) { 
      return false; 
    } 

    if (empty($to)) { 
      return false; 
    } 

    return true; 
  } 
} 

```

一个`SMS`类通知客户他们的玩具已经从工厂发货：

```php
<?php 

class SMS 
{ 
  private $from; 

  public function __construct(string $from) 
  { 
    $this->from = $from; 
  } 

  public function send(string $to, string $message): bool 
  { 
    if (empty($to)) { 
      return false; 
    } 

    if (strlen($message) === 0) { 
      return false; 
    } 

    echo $to . " received message: " . $message; 
    return true; 
  } 
} 

```

这是我们的`ToyFactory`类，它充当一个外观，将所有这些类连接在一起，并允许操作按顺序发生：

```php
<?php 

class ToyShop 
{ 
  private $courier; 
  private $manufacturer; 
  private $sms; 

  public function __construct(String $factoryAdress, String $contactNumber, int $capacity) 
  { 
    $this->courier = new Post($factoryAdress); 
    $this->sms = new SMS($contactNumber); 
    $this->manufacturer = new Manufacturer($capacity); 
  } 

  public function processOrder(string $address, $phone) 
  { 
    $item = $this->manufacturer->build(); 
    $this->courier->dispatch($item, $address); 
    $this->sms->send($phone, "Your order has been shipped."); 
  } 
} 

```

最后，我们可以将所有这些内容包装在我们的`index.php`文件中：

```php
<?php 

require_once('Manufacturer.php'); 
require_once('Post.php'); 
require_once('SMS.php'); 
require_once('ToyShop.php'); 

$childrensToyFactory = new ToyShop('1 Factory Lane, Oxfordshire', '07999999999', 5); 
$childrensToyFactory->processOrder('8 Midsummer Boulevard', '07123456789'); 

```

一旦我们运行这段代码，我们会看到来自我们的`SMS`类的消息显示出短信已发送：

![Facade](img/image_04_001.jpg)

在其他情况下，当各种类之间耦合较松时，我们可能会发现最好使用依赖注入。通过将执行各种操作的对象注入到`ToyFactory`类中，我们可以通过能够注入`ToyFactory`类可以操作的假类来使测试变得更容易。

就我个人而言，我非常相信尽可能使代码易于测试；这也是为什么我不喜欢这种方法的原因。

# 总结

本章通过引入结构设计模式扩展了我们在上一章开始学习的设计模式。

因此，我们学会了一些关键的模式来简化软件设计过程；这些模式确定了实现不同实体之间关系的简单方式：

+   我们学习了装饰器，如何包装类以向它们添加额外的行为，并且关键是，我们学会了这如何帮助我们遵守单一职责原则。

+   我们学习了类和对象适配器，以及它们之间的区别。这里的关键是为什么我们可能会选择组合而不是继承的论点。

+   我们复习了享元设计模式，它可以帮助我们以节省内存的方式执行某些过程。

+   我们学会了组合设计模式如何帮助我们将对象的组合与单个对象一样对待。

+   我们介绍了桥接设计模式，它让我们将抽象与实现解耦，使两者能够独立变化。

+   我们介绍了代理设计模式如何作为另一个类的接口，并且我们可以将其用作转发代理。

+   最后，我们学会了外观设计模式如何用于为复杂系统提供简单的接口。

在下一章中，我们将通过讨论行为模式来结束我们的设计模式部分，准备涉及架构模式。
