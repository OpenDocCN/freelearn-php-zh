# 第十七章：自动化测试

自动化测试是一个过程，我们依靠特殊的软件来持续运行预定义的测试，以验证我们应用程序的完整性。为此，自动化测试是一系列覆盖应用程序功能并比较触发结果与预期结果的步骤。

手动测试是确保所编写的功能按预期工作的一种很好的方法。大多数采用这种策略的人遇到的主要问题是回归。一旦某个功能被测试，他们唯一能保证没有引入回归（或错误）的方法是重新测试它。随着应用程序的增长，这变得难以处理。这就是自动化测试发挥作用的地方。

自动化测试使用具有 API 的特殊软件，该 API 允许我们自动化测试功能中涉及的步骤。这意味着我们可以依赖机器运行这些测试，次数不限，阻止我们拥有一个完全工作的应用程序的唯一因素是缺乏适当的测试覆盖率以及定义良好的测试。

有很多不同的软件可用于执行此类测试，并且通常针对特定类型的自动化测试。例如，Behat 是一个基于 PHP 的强大开源行为测试框架，允许编写与手动测试人员所做非常相似的测试脚本——通过浏览器与应用程序交互并测试其行为。还有其他测试框架在测试目标的级别上更低。例如，PHP 行业标准工具 PHPUnit 广泛用于执行单元测试。这种类型的测试专注于尽可能低级别的实际代码；通过提供不同的输入来验证类方法是否正常工作，并检查它们的输出。这种测试的强烈论点是它鼓励更好的代码架构，这可以通过编写单元测试的容易程度（部分地）来衡量。

我们还有功能测试或集成测试，它们介于上述两种示例之间。这些测试高于代码级别，将应用程序子系统纳入其中，以测试更全面的功能集，而不必考虑浏览器行为和用户交互。

同意一个经过良好测试的应用程序应结合不同的测试方法并不困难。例如，测试应用程序的各个架构单元并不能保证整个子系统正常工作，就像只测试子系统并不能保证其各个组件在所有情况下都能正常工作。同样，对于依赖于用户交互的某些子系统也是如此——这些子系统也需要测试覆盖率。

在本章中，我们将了解 Drupal 8 中自动化测试的工作原理。更具体地说，我们将逐一解释所有可用的测试方法，并以每个测试为例进行说明。到本章结束时，您将准备好编写自己的测试，并对代码足够熟悉，以便进一步探索可用的测试功能。

# Drupal 8 的测试方法

与许多其他开发方面一样，在 Drupal 8 中，自动化测试得到了极大的改进。在之前的版本中，测试框架是一个专门为测试 Drupal 应用程序定制的自定义框架——*Simpletest*。其主要测试能力集中在功能测试上，并强调用户与伪浏览器的交互。然而，它相当强大，允许测试广泛的功能。

Drupal 8 的开发也是从 *Simpletest* 开始的。然而，随着 PHPUnit 的采用，Drupal 正在远离它，并正在逐步淘汰它。为了替代它，有一系列不同类型的测试——所有这些测试都由 PHPUnit 运行——可以覆盖更多的测试方法。那么，让我们看看这些是什么。

Drupal 8 包含以下类型的测试：

+   Simpletest：由于遗留原因而存在，但不再用于创建新的测试。这将在 Drupal 9 中被移除。

+   单元测试：使用最小依赖（通常为模拟）的低级别类测试。

+   内核测试：使用启动的内核、数据库访问以及仅加载的几个模块进行的功能测试。

+   功能测试：使用启动的 Drupal 实例、一些已安装的模块以及基于 Mink 的浏览器模拟器（Goutte 驱动器）进行的功能测试。

+   功能 JavaScript：与之前的类似，使用 Selenium 驱动器进行 Mink 测试，允许测试由 JavaScript 驱动的功能。

除了 Simpletest 之外，所有这些测试套件都是建立在 PHPUnit 之上的，因此由它运行。根据测试类所在的命名空间以及目录位置，Drupal 可以发现这些测试并知道它们的类型。

在本章中，我们将随着测试我们在本书中编写的某些功能，看到所有这些示例（除了 Simpletest）。

# PHPUnit

Drupal 8 使用 PHPUnit 作为所有类型测试的测试框架。在本节中，我们将了解如何与之合作来运行测试。

在您的开发环境（或您想要运行测试的任何地方），请确保您已使用 `--dev` 标志安装了 composer 依赖项。这将包括 PHPUnit。请记住，永远不要在生产环境中这样做，因为这可能会危及您应用程序的安全性。

虽然 Drupal 有一个用于运行测试的用户界面，但 PHPUnit 与该界面集成得并不好。因此，建议我们使用命令行来运行它们。实际上，这样做非常简单。要运行整个测试套件（某种类型的），我们必须导航到 Drupal 核心文件夹：

```php
cd core  
```

然后运行以下命令：

```php
../vendor/bin/phpunit —testsuite=unit  
```

这个命令通过供应商目录回退一个文件夹，并使用安装的`phpunit`可执行文件。作为一个选项，在之前的例子中，我们指定了只想运行单元测试。省略这一点将运行所有类型的测试。然而，对于大多数其他测试，将需要一些配置，正如我们将在相应的部分中看到的那样。

如果我们想运行特定的测试，我们可以将其作为参数传递给`phpunit`命令（文件的路径）：

```php
../vendor/bin/phpunit tests/Drupal/Tests/Core/Routing/UrlGeneratorTest.php  
```

在这个例子中，我们运行了一个 Drupal 核心测试，该测试用于测试`UrlGenerator`类。

或者，我们可以运行属于同一*组*的多个测试（我们很快就会看到如何将测试添加到组中）：

```php
../vendor/bin/phpunit —group=Routing 
```

这将运行来自`Routing`组的所有测试，实际上它包含了我们之前看到的`UrlGeneratorTest`。如果我们用逗号分隔它们，我们也可以运行来自多个组的测试。

此外，为了检查可用的组，我们可以运行以下命令：

```php
../vendor/bin/phpunit —list-groups 
```

这将列出所有已注册到 PHPUnit 的组。

最后，我们还可以通过使用`—filter`参数来运行测试中的特定方法：

```php
../vendor/bin/phpunit —filter=testAliasGenerationUsingInterfaceConstants  
```

这是之前看到的同一个`UrlGeneratorTest`中的一个测试方法，并且是唯一会运行的方法。

# 注册测试

在不同的测试套件类型之间，关于我们需要做什么以便 Drupal（以及 PHPUnit）能够发现和运行它们，有一些共同之处。

首先，我们需要确定测试类应该放在哪个目录中。模式是这样的：`tests/src/[suite_type]`，其中`[suite_type]`是这个测试套件类型的名称，这个测试应该属于这个类型。它可以有以下几种：

+   单元

+   核心库

+   功能测试

+   功能 JavaScript 测试

例如，单元测试应该放在我们模块的`tests/src/Unit`文件夹中。

第二，测试类还需要遵循一个命名空间结构：

```php
namespace Drupal\Tests\[module_name]\[suite_type]  
```

这一点也很容易理解。

第三，我们需要在测试类的 PHPDoc 中包含某些元数据。每个类都必须有一行总结，描述测试类的用途。只有使用`@coversDefaultClass`属性的类可以省略总结行。此外，所有测试类都必须有`@group` PHPDoc 注释，指出它们所属的组。这就是 PHPUnit 如何只运行属于某些组的测试。

既然我们已经知道了如何注册和运行测试，让我们来看看单元测试，看看我们如何编写自己的测试。

# 单元测试

如开头简要提到的，单元测试用于测试构成代码架构的单个*单元*。在实践中，这意味着测试单个类，特别是它们包含的方法以及它们应该做什么。由于测试发生在如此低级别，它们是目前可以运行的最快的测试。

单元测试背后的逻辑相当简单：在提供输入后，测试断言方法输出是正确的。通常，它覆盖的`输入 -> 输出`场景越多，测试的代码就越稳定。例如，测试还应涵盖意外场景，以及练习测试方法中包含的所有代码（例如由`if/else`语句创建的分支）。

依赖注入的编程模式——对象应该接收它们可能需要的其他对象作为依赖项——在单元测试中变得至关重要。原因是如果类方法与全局作用域一起工作或实例化其他对象，我们就无法干净地测试它们。相反，如果它们需要依赖项，我们可以*模拟*它们，并在执行测试的上下文中传递这些依赖项。我们很快就会看到一些示例。但在我们这样做之前，让我们创建一个可以轻松使用单元测试进行测试的简单类。

一个典型的例子是一个简单的计算器类。它将接受两个数字作为构造函数的参数，并具有四个方法来对这些数字执行基本算术运算。我们将把它放入我们的*Hello World*模块中：

```php
namespace Drupal\hello_world; 

/** 
 * Class used to demonstrate a simple Unit test. 
 */ 
class Calculator { 

  private $a; 
  private $b; 

  public function __construct($a, $b) { 
    $this->a = $a; 
    $this->b = $b; 
  } 

  public function add() { 
    return $this->a + $this->b; 
  } 

  public function subtract() { 
    return $this->a - $this->b; 
  } 

  public function multiply() { 
    return $this->a * $this->b; 
  } 

  public function divide() { 
    return $this->a / $this->b; 
  } 
} 
```

这里没有这么复杂。你可以争论说计算器类不应该有任何依赖，而是应该将数字传递给实际的算术方法。然而，这在我们示例中也能很好地工作，并且稍微少一些重复。

现在，让我们创建第一个单元测试，以确保这个类表现如我们所期望。在上一个部分中，我们看到了这些测试需要放入哪个目录。所以，在我们的情况下，它将是`/tests/src/Unit`。测试类看起来像这样：

```php
namespace Drupal\Tests\hello_world\Unit; 

use Drupal\hello_world\Calculator; 
use Drupal\Tests\UnitTestCase; 

/** 
 * Tests the Calculator class methods. 
 * 
 * @group hello_world 
 */ 
class CalculatorTest extends UnitTestCase { 

  /** 
   * Tests the Calculator::add() method. 
   */ 
  public function testAdd() { 
    $calculator = new Calculator(10, 5); 
    $this->assertEquals(15, $calculator->add()); 
  } 

  /** 
   * Tests the Calculator::subtract() method. 
   */ 
  public function testSubtract() { 
    $calculator = new Calculator(10, 5); 
    $this->assertEquals(5, $calculator->subtract()); 
  } 

  /** 
   * Tests the Calculator::multiply() method. 
   */ 
  public function testMultiply() { 
    $calculator = new Calculator(10, 5); 
    $this->assertEquals(50, $calculator->multiply()); 
  } 

  /** 
   * Tests the Calculator::divide() method. 
   */ 
  public function testDivide() { 
    $calculator = new Calculator(10, 5); 
    $this->assertEquals(2, $calculator->divide()); 
  } 

}  
```

首先，你会注意到命名空间与我们在上一章中看到的模式相对应。其次，PHPDoc 包含所需的信息：摘要和`@group`标签。第三，类名以单词`Test`结尾。最后，该类扩展了`UnitTestCase`，这是我们为所有单元测试需要扩展的基类。

在 Drupal 8 中，所有类型的测试类名称都需要以单词*Test*结尾，并扩展提供该类型测试特定代码的相关基类。

然后，我们有实际的方法来测试`Calculator`类的各个方面，并且这些方法总是必须以单词`test`开头。这就是告诉 PHPUnit 它们需要被执行。这些方法是实际的独立测试本身，这意味着`CalculatorTest`类有四个测试。此外，这些测试都是独立于其他测试运行的。

由于`计算器`的算术运算非常简单，理解我们如何测试它并不困难。对于每种方法，我们使用一些数字实例化一个新的实例，然后我们*断言*算术运算的结果等于我们预期的结果。基类提供了许多不同的断言方法，我们可以在测试中使用。由于它们有很多，我们在这里不会全部涵盖。随着我们编写更多的测试，我们将看到更多。但我强烈建议您检查各种测试套件的基类，看看是否有以单词`assert`开头的方法。一个很好的方法也是使用一个在你输入方法名时自动补全的 IDE。这可以非常方便。

这样，我们就可以运行测试并查看它是否通过。通常情况下，它应该会通过，因为我们可以在脑海中做数学运算，并且我们知道它是正确的：

```php
../vendor/bin/phpunit ../modules/custom/hello_world/tests/src/Unit/CalculatorTest.php  
```

结果应该是绿色的：

```php
OK (4 tests, 4 assertions)  
```

然而，我之前提到，一个好的测试也应该考虑到意外情况和负面响应。然而，在我们的例子中，我们并没有做得很好。如果我们看看`testAdd()`，我们可以看到使用这两个数字时断言是正确的。但如果我们后来不小心将`Calculator::add()`方法改为这样：

```php
return 15;  
```

测试仍然会通过，但它实际上是一个真正的阳性吗？并不一定，因为如果我们传递不同的数字，计算结果将不再匹配。因此，我们应该使用不止一组数字来测试这些方法，以实际证明`计算器`类背后的数学是有效的。

因此，我们可以做类似这样的事情：

```php
$calculator = new Calculator(10, 5); 
$this->assertEquals(15, $calculator->add()); 
$calculator = new Calculator(10, 6); 
$this->assertEquals(16, $calculator->add());  
```

这样，我们就可以确保加法操作是正确的。在这个方法中，一个权衡是，我们有一些重复的代码，特别是如果我们必须对所有其他操作也这样做的话。

通常，在编写测试时，重复性比编写实际代码时更被接受。很多时候，你对此无能为力，因为代码看起来非常重复。然而，在我们的情况下，我们可以通过使用`setUp()`方法来实际做些事情，这个方法是在 PHPUnit 运行每个测试方法之前被调用的。它的目的是执行对类中所有测试都通用的各种准备任务。然而，不要认为它只运行一次然后被所有测试使用。实际上，它在每个单独的测试方法运行之前都会运行。

因此，我们可以做的是类似这样的事情：

```php
/** 
 * @var \Drupal\hello_world\Calculator 
 */ 
protected $calculatorOne; 

/** 
 * @var \Drupal\hello_world\Calculator 
 */ 
protected $calculatorTwo; 

/** 
 * {@inheritdoc} 
 */ 
public function setUp() { 
  parent::setUp(); 
  $this->calculatorOne = new Calculator(10, 5); 
  $this->calculatorTwo = new Calculator(10, 2); 
}  
```

我们创建两个类属性，并在`setUp()`方法中将它们分配给我们的计算器对象。需要记住的一个重要事情是始终调用此方法的父调用，因为它为环境设置做了非常重要的事情。特别是当我们转向 Kernel 和功能测试时。

现在，`testAdd()`方法可以看起来像这样：

```php
public function testAdd() { 
  $this->assertEquals(15, $this->calculatorOne->add()); 
  $this->assertEquals(12, $this->calculatorTwo->add()); 
}  
```

更加简洁，重复性更低。基于此，你可以自己推断并应用相同的更改到其他方法。

# 模拟依赖

被测试的类很少像我们的计算器类那样简单。大多数情况下，它们将具有依赖关系，而这些依赖关系反过来也有依赖关系。因此，单元测试变得稍微复杂一些。事实上，编写单元测试的容易程度已经成为被测试代码质量的一个试金石——单元测试越简单，代码质量越好。

作为编写单元测试的第二个示例，让我们进入“现实世界”，并测试我们在这本书中编写的其中一个类，即 `UserTypesAccess` 类。如果你记得从第十章 访问控制，我们创建了这个服务，用于在路由上作为访问检查器。虽然我们可以编写功能测试来验证它作为访问系统一部分的工作情况，但我们也可以编写一个单元测试来检查 `access()` 方法中的实际代码。所以让我们开始吧。

我们需要做的第一件事是创建类（同时尊重目录放置以及类命名空间）：

```php
namespace Drupal\Tests\user_types\Unit; 

use Drupal\Tests\UnitTestCase; 

/** 
 * Tests the UserTypesAccess class methods. 
 * 
 * @group user_types 
 */ 
class UserTypesAccessTest extends UnitTestCase {}  
```

到目前为止，事情看起来像我们之前的例子——我们有 PHPDoc 信息，并且正在扩展 `UnitTestCase` 类。所以让我们为 `UserTypesAccess` 类的 `access()` 方法编写一个测试。然而，如果你记得，这个方法接受两个参数（一个用户账户和一个路由对象），并且还使用了注入到类中的实体类型管理器。这就是我们复杂性的大部分所在。我们需要测试的是方法返回值取决于这些参数。基本上，如果用户账户在路由上具有某些值，它将允许或拒绝访问。

在单元测试中，依赖项通常被模拟。这意味着 PHPUnit 将创建空的外观对象，它们的行为就像我们描述的那样，我们可以将这些用作依赖项。创建简单模拟对象的方法如下：

```php
$user = $this->createMock('Drupal\user\Entity\User');  
```

`$user` 对象现在将成为 Drupal 8 `User` 实体类的模拟。当然，它不会做任何事情，但它可以用作依赖项。但是，为了使其真正有用，我们需要根据测试代码对它的使用来指定一些行为。例如，如果它调用了其 `id()` 方法，我们需要指定这种行为。我们可以通过 *预期* 来做到这一点：

```php
$user->expects($this->any()) 
  ->method('id') 
  ->will($this->returnValue(1));  
```

这告诉模拟对象，对于对它的 `id()` 方法的每次调用，它应该返回值 `1`。`expects()` 方法接受一个匹配器，它可以更加限制性。例如，我们不仅可以使用 `$this->any()`，还可以使用 `$this->once()`，这意味着模拟对象的 `id()` 方法只能被调用一次。查看基类以获取其他可用选项，以及可以传递给 `will()` 方法的选项——尽管 `$this->returnValue()` 将是最常见的一个。最后，如果 `id()` 方法接受一个参数，我们还可以使用 `with()` 方法，将预期的参数值传递给匹配器。

创建模拟对象的一种更复杂的方式是使用模拟构建器：

```php
$user = $this->getMockBuilder('Drupal\user\Entity\User') 
  ->getMock(); 
```

这将获取相同的模拟对象，但将允许在它的构建中拥有更多选项。我建议查看 PHPUnit 文档以获取更多信息，因为这是我们在这本书中关于模拟对象将要深入探讨的深度。

现在我们对模拟有了些了解，我们可以继续编写我们的测试。为此，我们需要考虑最终目标，并从所有需要模拟的方法调用回溯。提醒一下，这是我们需要测试的代码：

```php
public function access(AccountInterface $account, Route $route) { 
  $user_types = $route->getOption('_user_types'); 
  if (!$user_types) { 
    return AccessResult::forbidden(); 
  } 
  if ($account->isAnonymous()) { 
    return AccessResult::forbidden(); 
  } 
  $user = $this->entityTypeManager->getStorage('user')->load($account->id()); 
  $type = $user->get('field_user_type')->value; 
  return in_array($type, $user_types) ? AccessResult::allowed() : AccessResult::forbidden(); 
}  
```

因此，乍一看，我们需要模拟`EntityTypeManager`。我们将手动使用一些虚拟数据实例化其方法参数。然而，模拟`EntityTypeManager`将会相当复杂。对其`getStorage()`方法的调用需要返回一个`UserStorage`对象。这也需要被模拟，因为对其`load()`方法的调用需要返回一个`User`实体对象。最后，我们也需要模拟它，因为对其`get()`方法的调用预期也将返回一个值对象。

正如我提到的，我们将从我们的最终目标开始回溯。因此，我们可以从实例化我们想要传递的`AccountInterface`对象类型以及路由对象开始：

```php
/**
  * Tests the UserTypesAccess::access() method.
  */
 public function testAccess() {
   // User accounts
   $anonymous = new UserSession(['uid' => 0]);
   $registered = new UserSession(['uid' => 2]);

   // Route definitions.
   $manager_route = new Route('/test_manager', [], [], ['_user_types' => ['manager']]);
   $board_route = new Route('/test_board', [], [], ['_user_types' => ['board']]);
   $none_route = new Route('/test_board');
 }
```

以及顶部的新的`use`语句：

```php
use Drupal\Core\Session\UserSession; 
use Symfony\Component\Routing\Route;  
```

基本上，我们想要测试两种用户类型的情况：匿名用户和注册用户。在实例化`UserSession`对象（这些对象实现了`AccountInterface`接口）时，我们传递一些与它一起存储的数据。在我们的情况下，我们需要用户`uid`，因为测试代码在检查用户是否匿名时将请求它。

然后，我们创建三个路由：一个管理器应该可以访问的路由，一个董事会成员应该可以访问的路由，以及一个没有人可以访问的路由（如路由上的`_user_types`选项所示）。如果你不记得这个功能是什么，请回顾第十章，*访问控制*。

一旦完成这个步骤，接下来就是实例化我们的`UserTypesAccess`类，以便使用各种组合的账户和路由对象调用其`access()`方法：

```php
$access = new UserTypesAccess($entity_type_manager);
```

以及顶部的新的`use`语句：

```php
 use Drupal\user_types\Access\UserTypesAccess;
```

然而，我们还没有实体类型管理器，因此我们需要对其进行模拟。以下是我们需要模拟实体类型管理器以使其为测试代码工作所需的所有代码（这些代码在我们迄今为止编写的测试代码之前）：

```php
// User entity mock. 
$type = new \stdClass(); 
$type->value = 'manager'; 
$user = $this->getMockBuilder('Drupal\user\Entity\User') 
  ->disableOriginalConstructor() 
  ->getMock(); 
$user->expects($this->any()) 
  ->method('get') 
  ->will($this->returnValue($type)); 

// User storage mock 
$user_storage = $this->getMockBuilder('Drupal\user\UserStorage') 
  ->disableOriginalConstructor() 
  ->getMock(); 
$user_storage->expects($this->any()) 
  ->method('load') 
  ->will($this->returnValue($user)); 

// Entity type manager mock. 
$entity_type_manager = $this->getMockBuilder('Drupal\Core\Entity\EntityTypeManager') 
  ->disableOriginalConstructor() 
  ->getMock(); 
$entity_type_manager->expects($this->any()) 
  ->method('getStorage') 
  ->will($this->returnValue($user_storage));  
```

首先，你会注意到实体类型管理器只是在最后才被模拟。我们首先需要启动调用链，该链以用户实体对象字段值为结束。因此，第一个块模拟了用户实体对象，它期望对其`get()`方法进行任意次数的调用，并将始终返回一个具有`value`属性等于`manager`字符串的`stdClass()`对象。这样我们就模拟了实体字段系统访问器。

在使用模拟构建器创建模拟时，我们可以使用 `disableOriginalConstructor()` 方法来防止 PHPUnit 调用原始类的构造函数。这很重要，以防止需要所有各种其他依赖项，而这些依赖项实际上并不影响被测试的代码。

现在我们有了用户实体模拟，我们可以将其用作 `UserStorage` 模拟的 `load()` 方法的返回值。这反过来又是实体类型管理模拟的 `getStorage()` 方法的返回值。因此，我们编写的所有代码意味着我们已经模拟了以下链：

```php
$this->entityTypeManager->getStorage('user')->load($account->id());  
```

我们传递给 `load()` 方法的参数实际上并不重要，因为我们始终有一个具有 `manager` 用户类型的用户实体。

现在一切都已模拟，我们可以使用之前创建的 `$access` 对象，并根据对其 `access()` 方法的调用进行断言：

```php
// Access denied due to lack of route option. 
$this->assertInstanceOf('Drupal\Core\Access\AccessResultForbidden', $access->access($registered, $none_route)); 

// Access denied due to user being anonymous on any of the routes 
$this->assertInstanceOf('Drupal\Core\Access\AccessResultForbidden', $access->access($anonymous, $manager_route)); 
$this->assertInstanceOf('Drupal\Core\Access\AccessResultForbidden', $access->access($anonymous, $board_route)); 

// Access denied due to user not having proper field value 
$this->assertInstanceOf('Drupal\Core\Access\AccessResultForbidden', $access->access($registered, $board_route)); 

// Access allowed due to user having the proper field value. 
$this->assertInstanceOf('Drupal\Core\Access\AccessResultAllowed', $access->access($registered, $manager_route));  
```

返回值始终是一个实现接口的对象——要么是 `AccessResultAllowed`，要么是 `AccessResultForbidden`，因此这是我们需要断言的内容。我们正在检查四个不同的用例：

+   如果没有路由选项，则拒绝访问

+   任何路由上的匿名用户访问被拒绝

+   对于具有错误用户类型的注册用户，拒绝访问

+   对于注册用户且具有正确用户类型的情况，允许访问

因此，我们可以运行测试，并希望得到一个绿色的结果：

```php
../vendor/bin/phpunit ../modules/custom/user_types/tests/src/Unit/UserTypesAccessTest.php   
```

这就是编写单元测试的基础。在 Drupal 8 中，还有更多类型的断言，你最终会模拟很多依赖项。但不要因为一开始遇到的缓慢速度而气馁，随着经验的积累，事情会变得更快。

# 内核测试

内核测试是 Drupal 8 中我们可以拥有的直接高级测试方法，实际上是集成测试，专注于测试各种组件。它们比常规功能测试更快，因为它们不执行完整的 Drupal 安装，而是使用一个内存中的伪安装，启动速度更快。因此，它们也不处理任何浏览器交互，并且不会自动安装任何模块。

除了代码本身之外，内核测试还与数据库一起工作，并允许我们加载运行测试所需的模块。然而，与下一节中我们将看到的函数测试不同，内核测试还要求我们手动触发所需数据库模式的安装。但我们将看到在本节中涵盖的两个示例中如何做到这一点。

然而，在我们能够进行内核测试之前，我们需要确保我们有一个数据库连接，并且 PHPUnit 意识到这一点。在我们的 Drupal 安装 `core` 文件夹中，我们找到一个 `phpunit.xml.dist` 文件，我们需要将其复制并重命名为 `phpunit.xml`。这是 PHPUnit 的配置文件。通常，这个文件应该已经被 Git 忽略，因此无需担心将其提交到仓库。

在此文件中，我们找到一个名为`SIMPLETEST_DB`的环境变量，我们可以使用以下注释代码中展示的格式来指定数据库连接：

```php
mysql://username:password@localhost/databasename#table_prefix  
```

一旦设置好，PHPUnit 将能够连接到数据库，以便为内核测试以及功能测试和功能 JavaScript 测试安装 Drupal。

根据经验法则，当不涉及浏览器交互且内核测试足以完成任务时，你应该始终选择内核测试而不是功能测试。这是因为充满测试的套件可能会花费很长时间运行，所以你应该尽可能提高其性能。

# TeamCleaner 测试

现在我们已经覆盖了这些内容，是时候编写我们的第一个内核测试了。一个简单的好例子可以是对我们在第十四章中创建的`TeamCleaner`和`QueueWorker`插件进行测试，*批处理、队列和 Cron*。如果你想知道为什么不能使用超快的单元测试方法进行测试，答案是它的单一方法不返回任何内容。相反，它改变了我们需要访问以检查其是否正确发生的数据库值。

测试类自然位于我们的模块的`tests/src/Kernel`文件夹中，可以开始如下：

```php
namespace Drupal\Tests\sports\Kernel; 

use Drupal\KernelTests\KernelTestBase; 

/** 
 * Test the TeamCleaner QueueWorker plugin. 
 * 
 * @group sports 
 */ 
class TeamCleanerTest extends KernelTestBase {}  
```

命名空间与我们之前看到的保持一致，并且我们有正确的 PHPDoc 注释来注册测试。此外，这次，我们是从`KernelTestBase`扩展的。请注意这个类的实际版本，因为来自旧 Simpletest 框架的版本也称为`KernelTestBase`。所以请确保你扩展的是在*use*语句中看到的正确版本。

我们需要做的第一件事是指定在运行此测试时要加载哪些模块。在我们的例子中，这是`sports`模块，因此我们可以添加一个包含此名称的类属性：

```php
/** 
 * Modules to enable. 
 * 
 * @var array 
 */ 
protected static $modules = ['sports'];  
```

在此处指定模块列表实际上并不安装它们，只是将它们加载并添加到服务容器中。所以是的，我们可以访问模块和代码以及容器。但这也意味着这些模块定义的模式实际上并没有创建，因此我们需要手动完成。同样，对于模块附带配置也是如此。但我们可以将这些事情处理在`setUp()`方法中或在实际的测试方法本身中。我们将选择后者，因为在这种情况下，我们只有一个测试方法在类中。整个事情可以看起来像这样：

```php
/** 
 * Tests the TeamCleaner::processItem() method. 
 */ 
public function testProcessItem() { 
  $this->installSchema('sports', 'teams'); 
  $database = $this->container->get('database'); 
  $fields = ['name' => 'Team name']; 
  $id = $database->insert('teams') 
    ->fields($fields) 
    ->execute(); 

  $records = $database->query("SELECT id FROM {teams} WHERE id = :id", [':id' => $id])->fetchAll(); 
  $this->assertNotEmpty($records); 

  $worker = new TeamCleaner([], NULL, NULL, $database); 
  $data = new \stdClass(); 
  $data->id = $id; 
  $worker->processItem($data); 
  $records = $database->query("SELECT id FROM {teams} WHERE id = :id", [':id' => $id])->fetchAll(); 
  $this->assertEmpty($records); 
}  
```

以及*use*语句：

```php
use Drupal\sports\Plugin\QueueWorker\TeamCleaner;  
```

由于`TeamCleaner`插件会删除团队，因此只需安装该表就足够了。我们可以使用父`installSchema()`方法来做到这一点，我们将模块名称和要安装的表传递给它。我们实际上不处理玩家，因此我们应该避免进行不必要的操作，如创建`players`表。

然后，非常类似于我们在实际代码中这样做，我们从容器中获取 `database` 服务并向 `teams` 表添加一条记录。这将是我们将要删除的测试记录，这样我们就能记住它的 `$id`。但在测试之前，我们想确保我们的记录确实被保存了。因此，我们查询它并断言结果不为空。`assertNotEmpty()` 方法是我们在处理数组时可以使用的另一个有用的断言。

现在我们确定记录已经在数据库中，我们可以使用我们的插件来“处理”它。因此，我们实例化一个 `TeamCleaner` 对象，传递所有其所需的依赖项——最重要的是数据库服务。然后我们创建一个简单的对象，模拟 `processItem()` 方法所期望的，并调用后者，同时将前者传递给它。在这个时候，如果我们的插件正确执行了其任务，团队记录应该已经被从数据库中删除。因此，我们可以查询它，这次断言与之前相反：查询结果为空。

有了这个，我们的测试就结束了。像往常一样，我们应该实际运行它并确保它通过：

```php
../vendor/bin/phpunit ../modules/custom/sports/tests/src/Kernel/TeamCleanerTest.php  
```

这是一个非常简单的例子，展示了如何使用内核测试来测试一个组件，特别是那些与数据库集成的组件。我们也可以使用功能测试，但这会有些过度——它会运行得更慢，并且无法利用它相对于内核测试的优势，比如浏览器集成。

# CsvImporter 测试

在这个简单的例子之后，让我们再写一个测试，来展示一个更复杂的场景。我们将写一个测试来测试我们在上一章中创建的 `CsvImporter` 插件。

这个插件中包含了很多功能，与之一起工作——我们有实际的导入、插件和配置实体创建、用户界面等等。这是一个很好的例子，说明了可以从多方法测试覆盖中受益的功能。在这方面，我们从测试其基本目的开始，即产品导入，这不需要浏览器交互。这意味着我们可以使用内核测试。

与我们编写的前一个测试类似，我们可以从类开始（这次在 `products` 模块中）：

```php
namespace Drupal\Tests\products\Kernel; 

use Drupal\KernelTests\KernelTestBase; 

/** 
 * Tests the CSV Product Importer 
 * 
 * @group products 
 */ 
class CsvImporterTest extends KernelTestBase {}  
```

到目前为止，没有新的内容。

接下来，我们需要指定需要加载的模块。这里有一个更长的列表：

```php
  /** 
   * Modules to enable. 
   * 
   * @var array 
   */ 
  protected static $modules = ['system', 'csv_importer_test', 'products', 'image', 'file', 'user'];  
```

只有 `products` 模块可能对你来说很明显，但所有其他模块也都是必需的。`system`、`image`、`file` 和 `user` 模块都是处理 `CsvImporter` 插件所需的文件上传和存储过程所必需的。

并非总是那么容易确定需要哪些模块，所以这可能会涉及一些试错，至少在开始时是这样。一个典型的场景是运行测试并注意到由于缺少功能而导致的失败。将此功能追踪到模块并在列表中指定此模块通常是您获得完整模块列表的方法，尤其是当测试复杂且需要广泛的子系统及其依赖项时。

但你可能想知道`csv_importer_test`模块在那里有什么用。通常，你可能需要创建仅用于测试的模块——通常是因为它们包含一些你希望在测试中使用的配置。在我们的案例中，我们这样做是为了演示这些模块将放在哪里，并添加一个`products.csv`测试文件，我们可以在测试中使用它。

测试模块位于包含使用它们的测试的模块的`tests/modules`文件夹中。因此，在我们的案例中，我们有`csv_importer_test`及其`info.yml`文件：

```php
name: CSV Importer Test 
description: Used for testing the CSV Importer 
core: 8.x 
type: module 
package: Testing  
```

我们将要使用的提到的 CSV 文件就在它的旁边：

```php
id,name,number 
1,Car,45345 
2,Motorbike,54534  
```

现在我们已经讨论了这一点，我们可以编写测试方法：

```php
/** 
 * Tests the import of the CSV based plugin. 
 */ 
public function testImport() { 
  $this->installEntitySchema('product'); 
  $this->installEntitySchema('file'); 
  $this->installSchema('file', 'file_usage'); 
  $manager = $this->container->get('entity_type.manager'); 
  $products = $manager->getStorage('product')->loadMultiple(); 
  $this->assertEmpty($products); 

  $csv_path = drupal_get_path('module', 'csv_importer_test') . '/products.csv'; 
  $csv_contents = file_get_contents($csv_path); 
  $file = file_save_data($csv_contents, 'public://simpletest-products.csv', FileSystemInterface::EXISTS_REPLACE); 
  $config = $manager->getStorage('importer')->create([ 
    'id' => 'csv', 
    'label' => 'CSV', 
    'plugin' => 'csv', 
    'plugin_configuration' => [ 
      'file' => [$file->id()] 
    ], 
    'source' => 'Testing', 
    'bundle' => 'goods', 
    'update_existing' => true 
  ]); 
  $config->save(); 

  $plugin = $this->container->get('products.importer_manager')->createInstanceFromConfig('csv'); 
  $plugin->import(); 
  $products = $manager->getStorage('product')->loadMultiple(); 
  $this->assertCount(2, $products); 

  $products = $manager->getStorage('product')->loadByProperties(['number' => 45345]); 
  $this->assertNotEmpty($products); 
  $this->assertCount(1, $products); 
}  
```

以及顶部的`use`语句：

```php
use Drupal\Core\File\FileSystemInterface;  
```

这里的初始设置稍微复杂一些，部分原因是因为内核测试没有安装模块模式。使用父`installEntitySchema()`方法，我们可以安装产品实体和文件内容实体所需的所有必要表。然而，由于我们正在处理管理文件，我们还需要手动安装`file_usage`表。从技术上讲，它不是一个实体表。再次强调，使用试错法到达这些步骤并不丢脸。

现在我们已经设置了基础，我们进行一次合理性检查，确保数据库中没有产品实体。我们没有理由应该有任何，但确保这一点并无害处。这保证了测试的有效性，因为我们的目标将是后来断言产品的存在。

然后，我们通过使用来自`csv_importer_test`模块的`products.csv`文件创建一个管理的文件实体。`drupal_get_path()`函数是一种非常常见的检索模块或主题的相对路径的方法，无论它实际上位于何处。我们将此文件的 内容保存到测试环境的`public://`文件系统中。但请记住，一旦测试成功运行，此文件将被删除，因为 Drupal 会自行清理。

接下来，我们需要创建一个使用基于 CSV 的插件来运行导入的导入器配置实体。而不是通过 UI 进行，我们以编程方式完成。使用存储管理器，我们创建实体，就像我们在第六章“数据建模和存储”中学到的那样。一旦我们有了这个，我们就使用导入器插件管理器根据这个配置实体（我们给它分配了 ID `csv`）创建一个实例。最后，我们运行产品的导入。

现在，对于断言，我们进行双重检查。由于我们的测试 CSV 文件包含两行，我们再次加载所有产品实体并断言总数为两个。不多也不少。在这里，我们看到了另一种用于处理数组的实用断言方法：`assertCount()`。但接下来我们更加具体，尝试加载一个字段值（即`number`）等于测试 CSV 文件中预期数值的产品。并断言它确实被找到。

我们甚至可以进行更多的断言。例如，我们可以检查所有产品字段值是否已正确设置。我将让您探索如何进行此操作——要么基于这些值进行查询，要么断言字段值与其预期值之间的相等性。但重要的是不要过度，因为这会影响速度，在某些情况下，还会给测试覆盖率增加不足的价值，以补偿它。诀窍是找到正确的平衡。

最后，随着我们的测试已经就绪，我们实际上可以运行它：

```php
../vendor/bin/phpunit ../modules/custom/products/tests/src/Kernel/CsvImporterTest.php  
```

这个测试也应该通过。

# 功能测试

在上一节中，我们讨论了内核测试，并表示它们基本上是集成测试，侧重于组件而不是与浏览器的交互。在本节中，我们将上升一个层次，讨论全面的功能测试，也称为浏览器测试（从我们需要扩展的基本类名称）。 

Drupal 8 中的功能测试使用模拟浏览器（使用流行的 Mink 模拟器），允许用户点击链接、导航到页面、处理表单以及就页面上的 HTML 元素做出断言。它们不允许我们测试基于 JavaScript 的交互（有关这些内容，请参阅下一节）。

在 Drupal 7 中，功能测试是最常用的测试类型，大多数类都扩展了 Simpletest 的`WebTestBase`类。但在 Drupal 8 中，我们有`Drupal\Tests\BrowserTestBase`类，它像之前看到的那些一样与 PHPUnit 集成。基类包含大量用于断言的方法以及执行 Drupal（和 Web）相关任务的快捷方式：创建用户、实体、导航到页面、填写和提交表单、登录等。就像之前一样，每个测试（类方法）都是独立运行的，因此像内容和用户这样的东西不能在多个测试之间共享，而必须重新创建（可能使用我们之前看到的`setUp()`方法）。

浏览器测试使用具有最少数量的模块（使用*Testing*安装配置文件）执行完整的 Drupal 安装。这意味着我们可以指定安装其他模块，这些模块的架构也会被安装。此外，重要的是要理解，生成的安装与我们的当前开发站点没有任何共同之处。我们需要的所有配置，我们都需要创建。没有用户，没有内容，也没有文件。因此，这是一个全新的、并行的安装，它在单个测试的持续时间内运行，并在完成后被清理。

# 功能测试的配置

在编写我们的功能测试之前，我们需要回到我们的`phpunit.xml`文件并更改一些环境变量。除了我们之前调整的`SIMPLETEST_DB`变量外，我们还有`SIMPLETEST_BASE_URL`和`BROWSERTEST_OUTPUT_DIRECTORY`。第一个用于知道应用程序在浏览器中可以访问的位置。后者是 PHPUnit 可以保存输出数据的目录，需要是一个绝对本地路径（例如，本地`files`文件夹中的一个文件夹）：

```php
/var/www/sites/default/files/browser-output   
```

此外，确保运行测试的用户有权限写入`sites/simpletest`文件夹，因为虚拟文件系统就是在这里为每个测试创建的。最简单的方法是将文件夹的所有权更改为运行该进程的 Web 服务器用户。在 Apache 的情况下，这通常是`www-data`。

# Hello World 页面测试

我们将要编写的第一个功能测试是针对我们创建的*Hello World*页面及其背后的功能。我们将测试页面是否显示了正确的*Hello World*消息，这也取决于配置中找到的值。所以让我们创建一个类，自然是在`hello_world`模块中，在`tests/src/Functional`文件夹内：

```php
namespace Drupal\Tests\hello_world\Functional; 

use Drupal\Tests\BrowserTestBase; 

/** 
 * Basic testing of the main Hello World page. 
 * 
 * @group hello_world 
 */ 
class HelloWorldPageTest extends BrowserTestBase {}  
```

你真的可以看到与其他测试类型的连贯性。但在这个案例中，正如之前提到的，我们扩展了`BrowserTestBase`。

同样，就像之前一样，我们可以配置我们想要安装的模块数量：

```php
/** 
 * Modules to enable. 
 * 
 * @var array 
 */ 
protected static $modules = ['hello_world', 'user']; 
```

我们将需要在运行的第二个测试中使用用户模块，它将和这个模块属于同一类。但让我们先进行第一个，更简单的测试：

```php
/** 
 * Tests the main Hello World page. 
 */ 
public function testPage() { 
  $expected = $this->assertDefaultSalutation(); 
  $config = $this->config('hello_world.custom_salutation'); 
  $config->set('salutation', 'Testing salutation'); 
  $config->save(); 

  $this->drupalGet('/hello'); 
  $this->assertSession()->pageTextNotContains($expected); 
  $expected = 'Testing salutation'; 
  $this->assertSession()->pageTextContains($expected); 
}  
```

如果你记得，我们的`/hello`页面会根据一天中的时间显示问候语，除非管理员通过配置表覆盖了该消息。因此，我们从这个测试开始，断言在一个没有覆盖的新安装中，我们看到基于时间的问候语。为此，我们创建了一个单独的断言消息，因为它有点长，我们将重用它：

```php
/** 
 * Helper function to assert that the default salutation is present on the page. 
 * 
 * Returns the message so we can reuse it in multiple places. 
 */ 
private function assertDefaultSalutation() { 
  $this->drupalGet('/hello'); 
  $this->assertSession()->pageTextContains('Our first route'); 
  $time = new \DateTime(); 
  $expected = ''; 
  if ((int) $time->format('G') >= 00 && (int) $time->format('G') < 12) { 
    $expected = 'Good morning'; 
  } 

  if ((int) $time->format('G') >= 12 && (int) $time->format('G') < 18) { 
    $expected = 'Good afternoon'; 
  } 

  if ((int) $time->format('G') >= 18) { 
    $expected = 'Good evening'; 
  } 
  $expected .= ' world'; 
  $this->assertSession()->pageTextContains($expected); 
  return $expected; 
} 
```

我们在这里做的第一件事是使用`drupalGet()`方法导航到网站上的一个路径。请检查方法签名，看看你可以传递给它所有选项。我们做的第一个断言是页面包含文本*我们的第一个路由*（这是页面标题）。父`assertSession()`方法返回一个`WebAssert`实例，它包含所有 sorts of 方法，用于断言在 Mink 会话中当前页面上元素的存在。其中一种方法是通用的`pageTextContains()`，我们只需检查给定的文本是否可以在页面的任何地方找到。

虽然在许多情况下断言文本字符串的存在就足够了，但你可能想确保它确实是正确的（以避免误报）。例如，在我们的案例中，我们可以检查它是否真的是在`<h1>`标签内渲染的页面标题。我们可以这样做：

```php
$this->assertSession()->elementTextContains('css', 'h1', 'Our first route');  
```

`elementTextContains()`方法可以根据定位器（CSS 选择器或 xpath）在页面上找到元素，并断言它包含指定的文本。在我们的例子中，我们使用 CSS 选择器定位器，并尝试找到`<h1>`元素。

如果所有这些都正常，我们就继续断言实际的问候消息出现在页面上。不幸的是，我们必须重复相当多的代码，因为它是依赖于一天中的时间的。对你来说，一个好的家庭作业就是将这个逻辑提取到一个确定消息的服务中，并在这个服务和实际代码中使用这个服务。而且由于我们稍后需要这个消息，我们也返回它。

回到我们的实际测试方法，我们可以继续进行，知道消息已经在页面上正确显示。接下来，我们想要测试的是以下内容：如果存在一个具有`salutation`值的`hello_world.custom_salutation`配置对象，那么应该显示的就是这个。因此，我们程序性地创建它。接下来，我们再次导航到相同的路径（我们实际上是在重新加载页面）并检查旧消息不再显示，而新消息则显示出来。

所以如果我们实际运行这个测试：

```php
../vendor/bin/phpunit ../modules/custom/hello_world/tests/src/Functional/HelloWorldPageTest.php  
```

...该死。我们得到了一个错误：

```php
Behat\Mink\Exception\ResponseTextException: The text "Good evening world" appears in the text of this page, but it should not. 
```

就好像我们没有覆盖问候消息一样。但我们确实覆盖了。

问题在于缓存。请记住，我们作为匿名用户在这些页面上导航，并且网站上启用了缓存，就像在正常情况下一样。在第十一章“缓存”中，我记录了这个问题——`max-age`属性只对动态页面缓存（登录用户）的页面级别有效，而不是对匿名用户。

这是一个自动测试如何揭示我们在开发过程中引入的错误，而这些错误我们没有注意到的绝佳例子。我们很可能在禁用缓存和/或始终以登录用户身份访问页面时编写了我们的功能。所以这是一个很容易犯的错误。幸运的是，自动测试来救命。

这个问题的解决方案可以通过使用全出缓存关闭开关来实现。这意味着我们需要稍微修改我们的逻辑，告诉 Drupal 永远不缓存显示我们的问候语组件的页面。这是我们必须付出的代价，因为我们功能的高度动态性，而且始终是一个好的练习来评估这是否值得。

关闭开关实际上很容易使用。它是一个我们需要注入到我们的`HelloWorldSalutation`服务中的服务：

```php
/** 
 * @var \Drupal\Core\PageCache\ResponsePolicy\KillSwitch 
 */ 
protected $killSwitch; 

/** 
 * HelloWorldSalutation constructor. 
 * 
 * @param \Drupal\Core\Config\ConfigFactoryInterface $config_factory 
 * @param \Symfony\Component\EventDispatcher\EventDispatcherInterface $eventDispatcher 
 * @param \Drupal\Core\PageCache\ResponsePolicy\KillSwitch $killSwitch 
 */ 
public function __construct(ConfigFactoryInterface $config_factory, EventDispatcherInterface $eventDispatcher, KillSwitch $killSwitch) { 
  $this->configFactory = $config_factory; 
  $this->eventDispatcher = $eventDispatcher; 
  $this->killSwitch = $killSwitch; 
}  
```

以及顶部适当的 *use* 声明：

```php
use Drupal\Core\PageCache\ResponsePolicy\KillSwitch;  
```

在`getSalutation()`和`getSalutationComponent()`方法的开始处，我们只需添加这一行：

```php
$this->killSwitch->trigger();  
```

这将告诉 Drupal 的内部页面缓存永远不要缓存这个页面。但在我们再次运行测试之前，我们别忘了将`page_cache_kill_switch`服务作为依赖项添加到`hello_world.services.yml`中的`HelloWorldSalutation`服务中。现在如果我们运行这个测试，我们应该得到一个绿色结果。

# Hello World 表单测试

我们将要编写的第二个功能测试应该测试问候语覆盖表单本身。在上一个测试中，我们直接与配置 API 交互来更改配置值。现在我们将看到执行此操作的表单是否真的起作用。但由于我们可以从上一个测试中重用很多内容，并且它们非常紧密相关，我们可以将其添加到同一个类中：

```php
/** 
 * Tests that the configuration form for overriding the message works. 
 */ 
public function testForm() { 
  $expected = $this->assertDefaultSalutation(); 
  $this->drupalGet('/admin/config/salutation-configuration'); 
  $this->assertSession()->statusCodeEquals(403); 
  $account = $this->drupalCreateUser(['administer site configuration']); 
  $this->drupalLogin($account); 
  $this->drupalGet('/admin/config/salutation-configuration'); 
  $this->assertSession()->statusCodeEquals(200); 
  $this->assertSession()->pageTextContains('Salutation configuration'); 
  $this->assertSession()->elementExists('css', '#edit-salutation'); 

  $edit = [ 
    'salutation' => 'My custom salutation', 
  ]; 

  $this->drupalPostForm(NULL, $edit, 'op'); 
  $this->assertSession()->pageTextContains('The configuration options have been saved'); 
  $this->drupalGet('/hello'); 
  $this->assertSession()->pageTextNotContains($expected); 
  $this->assertSession()->pageTextContains('My custom salutation'); 
}  
```

我们以同样的方式开始这个测试，断言显示基于小时的问候消息。这也证明了每个测试都在自己的独立环境中运行，一个测试中的配置更改不会影响另一个。它们都是从一张白纸开始的。

然后我们导航到配置表单页面并断言我们没有访问权限。为此，我们使用`statusCodeEquals()`断言方法来检查响应代码。这是好的，因为我们需要用具有特定权限的用户登录。

配置表单的访问限制允许任何具有特定权限的用户。因此，我们的测试应该专注于那个权限，而不是可能间接包含这个权限的其他东西。例如，它不应该假设具有管理员角色的用户有那个权限。

因此，我们使用方便的 `drupalCreateUser()` 方法创建一个新的用户账户，其第一个参数是一个数组，包含用户应该拥有的权限。然后我们可以使用 `drupalLogin()` 方法使用生成的用户实体进行登录。在底层，这将导航到用户登录页面，提交表单，并断言一切顺利。现在我们可以回到配置表单页面，并且应该有访问权限——这也是我们断言的内容。此外，我们断言页面上有页面标题，并且有问候文本字段的 HTML 元素。我们使用 `elementExists()` 方法这样做，使用 CSS 选择器定位符，就像我们在之前的测试中所做的那样。再次提醒，查看 `WebAssert` 了解各种断言方法，这些方法可以帮助你识别页面上的内容。

现在是时候提交表单并覆盖问候消息了。我们使用 `drupalPostForm()` 来完成这项任务，其最重要的参数是一个数组，包含要填充表单元素的值，以单个表单 HTML 元素的 `name` 参数为键。在我们的例子中，我们只有一个。务必查看该方法的文档，以获取有关你可以用它做什么的所有信息。一旦表单提交，页面将重新加载，我们可以断言确认消息的存在。最后，我们可以回到 `/hello` 路径并断言旧消息不再显示，而是显示新的覆盖消息。

再次运行测试类时，应该包括这个新测试，并且所有内容都应该显示为绿色。但速度明显慢得多，因为已经完成了两个完整的 Drupal 安装。在下一节中，我们将引入 JavaScript，以便我们可以测试更动态的浏览器集成。但已经可以注意到，如果你不需要与浏览器交互，内核测试的运行速度会快得多。

# 功能性 JavaScript 测试

在 Drupal 8 中，我们可以编写的最后一种测试类型是 JavaScript 驱动的功能测试。当我们需要测试更动态的客户端功能，如 JavaScript 行为或 Ajax 交互时，功能 JavaScript 测试非常有用。

它们是常规功能测试的扩展，但使用 WebDriver。后者是一个 API，允许像 Selenium 这样的工具控制 Chrome 或 Firefox 等浏览器。Drupal 使用 Chrome 来完成这项任务，所以请确保你已经安装并配置了 Selenium 和 Chrome 驱动程序。我们在这里不会涉及这部分内容，因为它取决于你的本地环境和当前最新版本。

假设你已经运行了 Selenium，我们可以编写一些测试。但只有在我们将另一个环境变量添加到 PHPUnit 配置文件之后：

```php
<env name="MINK_DRIVER_ARGS_WEBDRIVER" value='["chrome", null, "http://localhost:4444/wd/hub"]'/> 
```

# 时间测试

如果你记得从第十二章 [JavaScript 和 Ajax API]，我们向 Hello World 问候组件添加了一个小的时间小部件，如果问候没有被覆盖，它会实时显示当前的小时。这个组件由 JavaScript 驱动，更重要的是，使用 JavaScript 添加到页面中。

此外，在前一节中，我们为 Hello World 页面编写了一个功能测试，其中我们断言了问候信息的存在。然而，实际的时间小部件永远不会出现在那里，因为在这些类型测试中使用的 Mink 驱动程序不支持 JavaScript。所以如果我们想测试这一点，我们需要编写一个功能 JavaScript 测试。

如预期的那样，这些类型的测试遵循相同的目录放置和命名空间模式。所以我们的第一个测试类可以开始如下：

```php
namespace Drupal\Tests\hello_world\FunctionalJavascript; 

use Drupal\FunctionalJavascriptTests\WebDriverTestBase; 

/** 
 * Testing the simple Javascript timer on the Hello World page. 
 * 
 * @group hello_world 
 */ 
class TimeTest extends WebDriverTestBase {}  
```

到现在，上述代码中的大部分应该已经很清晰了。然而，我们这次扩展的基类是 `WebDriverTestBase` 类，它本身是 `BrowserTestBase` 的子类。有趣的是，它实际上并没有添加很多功能，除了配置测试使用 Selenium Web Driver 和添加一些特定的 JavaScript 辅助方法。这是为了说明，功能测试和功能 JavaScript 测试之间的大部分差异是由实际的 Mink 驱动程序造成的。

注意，直到 Drupal 8.1，JavaScript 测试的默认驱动程序是 Phantom.js，因此你可能会找到对这个的过时引用。但我们仍然走在曲线的前面，使用最新的 Web Driver API 与 Selenium 和 Chrome 来运行我们的测试。

然而，一个极其方便的添加功能是截图功能。很多时候在测试前端交互时，事情并不像我们想象的那样进行，我们也不理解为什么。父 `createScreenshot()` 方法允许我们在任何给定时刻保存整个页面的截图，我们可以用来进行调试。我们只需要传入我们想要保存的文件名。所以请务必检查一下。

继续进行我们的测试，让我们添加我们想要启用的模块：

```php
/** 
 * Modules to enable. 
 * 
 * @var array 
 */ 
protected static $modules = ['hello_world']; 
```

如预期的那样，`Hello World` 模块就足够了。非常简单的测试方法可以看起来像这样：

```php
/** 
 * Tests the time component. 
 */ 
public function testTime() { 
  $this->drupalGet('/hello'); 
  $this->assertSession()->pageTextContains('The time is'); 

  $config = $this->config('hello_world.custom_salutation'); 
  $config->set('salutation', 'Testing salutation'); 
  $config->save(); 

  $this->drupalGet('/hello'); 
  $this->assertSession()->pageTextNotContains('The time is'); 
}  
```

我们正在使用与之前完全相同的断言技术，但由于启用了 JavaScript，时间小部件文本现在应该会显示出来。而且，就像之前一样，我们也测试了如果覆盖了问候方法，时间小部件不会显示。

# CsvImporter 测试

当学习关于内核测试时，我们为 `CsvImporter` 编写了一个测试，该测试侧重于在现有的导入器配置实体（我们通过编程创建）的情况下导入功能。然而，这个功能的重要角度之一是创建这个配置实体的过程，因为我们依赖于 Ajax 来动态注入与所选导入器插件相关的表单元素。所以，让我们也为此编写一个测试。

就像之前一样，测试类可以从以下内容开始：

```php
namespace Drupal\Tests\products\FunctionalJavascript; 

use Drupal\FunctionalJavascriptTests\WebDriverTestBase; 

/** 
 * Testing the creation/edit of Importer configuration entities using the CSV importer 
 * 
 * @group products 
 */ 
class ImporterFormTest extends WebDriverTestBase {}  
```

像往常一样，让我们启用一些模块：

```php
/** 
 * Modules to enable. 
 * 
 * @var array 
 */ 
protected static $modules = ['image', 'file', 'node']; 
```

你可能想知道为什么，例如，`products`模块不在那个列表中。在撰写本文时，它不起作用，因为启用它时抛出了一个与依赖项相关的错误（缺少由`image`模块定义的插件）。因此，我们也可以直接在我们的测试或`setUp()`方法中启用模块。这正是我们将要做的。

节点模块已启用，因为它定义了`access content`权限，该权限被核心`machine_name`表单元素使用。该元素用于导入实体表单，因此我们需要它才能使测试真正工作。

尽管我们只编写了一个测试方法，但为此进行的准备工作相当多，我们可能希望在其他地方重用。此外，将其与实际测试方法分离看起来也更整洁。因此，我们可以将其添加到`setUp()`方法中：

```php
/** 
 * {@inheritdoc} 
 */ 
public function setUp() { 
  parent::setUp(); 
  $this->container->get('module_installer')->install(['products', 'csv_importer_test']); 
  $csv_path = drupal_get_path('module', 'csv_importer_test') . '/products.csv'; 
  $csv_contents = file_get_contents($csv_path); 
  $this->file = file_save_data($csv_contents, 'public://simpletest-products.csv', FileSystemInterface::EXISTS_REPLACE); 
  $this->admin = $this->drupalCreateUser(['administer site configuration']); 
  $this->bundle = ProductType::create(['id' => 'goods', 'label' => 'Goods']); 
  $this->bundle->save(); 
}  
```

以及新的*使用*语句：

```php
use Drupal\products\Entity\ProductType; 
use Drupal\Core\File\FileSystemInterface;  
```

如预期的那样，我们首先安装`products`和`csv_importer_test`模块。我们使用`ModuleInstaller`服务来这样做。然后，我们像先前的测试中那样做——从`csv_importer_test`模块加载测试 CSV 文件并将其“上传”到 Drupal，创建一个新的管理文件实体。

然后，我们创建一个具有创建导入配置实体所需权限的管理员用户账户，以及一个用于产品实体的包，以便我们实际上可以创建产品。在先前的测试中，我们不需要担心包，因为我们以编程方式创建了导入配置。但现在，通过 UI，必须存在一个包才能选择它。

我们存储在类属性上的结果文件实体、管理员用户账户和 ProductType 配置实体，因此我们也应该定义这些：

```php
/** 
 * @var \Drupal\file\FileInterface 
 */ 
protected $file; 

/** 
 * @var \Drupal\Core\Session\AccountInterface 
 */ 
protected $admin; 

/** 
 * @var \Drupal\products\Entity\ProductType 
 */ 
protected $bundle; 
```

使用这个，我们就准备好编写我们的空测试方法并逐步填充它：

```php
/** 
 * Tests the importer form. 
 */ 
public function testForm() {} 
```

我们可以从基础知识开始：

```php
$this->drupalGet('/admin/structure/importer/add'); 
$assert = $this->assertSession(); 
$assert->pageTextContains('Access denied'); 
```

我们导航到创建导入配置实体的表单，并断言用户没有访问权限。这是因为默认情况下，我们是以匿名用户身份浏览的。接下来，我们需要登录并再次尝试：

```php
$this->drupalLogin($this->admin); 
$this->drupalGet('/admin/structure/importer/add'); 
$assert->pageTextContains('Add importer'); 
$assert->elementExists('css', '#edit-label'); 
$assert->elementExists('css', '#edit-plugin'); 
$assert->elementExists('css', '#edit-update-existing'); 
$assert->elementExists('css', '#edit-source'); 
$assert->elementExists('css', '#edit-bundle'); 
$assert->elementNotExists('css', 'input[name="files[plugin_configuration_plugin_file]"]'); 
```

我们使用相同的`drupalLogin()`方法并导航回表单。这次我们断言我们有标题以及各种 HTML 元素——用于创建实体的表单元素。此外，我们还断言我们没有上传 CSV 文件的元素，因为那应该只在我们选择使用 CSV 导入插件时显示。

因此，我们确实这样做了：

```php
$page = $this->getSession()->getPage(); 
$page->selectFieldOption('plugin', 'csv'); 
$this->assertSession()->assertWaitOnAjaxRequest(); 
$assert->elementExists('css', 'input[name="files[plugin_configuration_plugin_file]"]');  
```

使用`getSession()`方法，我们获取当前的 Mink 会话，从中我们可以获取代表我们正在查看的实际页面的对象。这是一个`DocumentElement`对象，可以通过各种方式遍历、检查和操作。我建议您查看`TraversableElement`类以了解所有可用方法。

其中一种方法是`selectFieldOption()`，我们可以指定 HTML 选择元素的定位器（ID、名称或标签）和一个值，它将触发选择。正如你所知，这应该会触发一个 Ajax 请求，引入我们的新表单元素。通过在`JSWebAssert`对象上使用`assertWaitOnAjaxRequest()`，我们可以等待它完成。最后，我们可以断言文件上传字段出现在页面上。

接下来，我们继续填写表单：

```php
$page->fillField('label', 'Test CSV Importer'); 
$this->assertJsCondition('jQuery(".machine-name-value").html() == "test_csv_importer"'); 
$page->checkField('update_existing'); 
$page->fillField('source', 'testing'); 
$page->fillField('bundle', $this->bundle->id()); 
$wrapper = $this->container->get('stream_wrapper_manager')->getViaUri($this->file->getFileUri()); 
$page->attachFileToField('files[plugin_configuration_plugin_file]', $wrapper->realpath()); 
$this->assertSession()->assertWaitOnAjaxRequest(); 
$page->pressButton('Save'); 
$assert->pageTextContains('Created the Test CSV Importer Importer.'); 
```

通用`fillField()`方法对于文本字段等很有用，而`checkField()`方法则预期对复选框很有用。两者的定位器都是元素的 ID、名称或标签。

我们还使用`assertJsCondition`方法让执行等待页面上的 JavaScript 发生变化。我们这样做是为了确保实体机器名字段已被当前填写。

接下来，借助我们上传的文件的流包装器，更具体地说，是其`realpath()`方法，我们使用`attachFileToField()`方法将文件附加到字段上。这触发了一个 Ajax 请求，我们再次等待其完成。最后，我们使用`pressButton()`方法点击提交按钮，并断言打印出了确认消息（表单已保存且页面已刷新）。

现在来检查操作是否真正正确完成：

```php
$config = Importer::load('test_csv_importer'); 
$this->assertInstanceOf('Drupal\products\Entity\ImporterInterface', $config); 

$fids = $config->getPluginConfiguration()['file']; 
$fid = reset($fids); 
$file = File::load($fid); 
$this->assertInstanceOf('Drupal\file\FileInterface', $file);  
```

以及新的*使用*语句：

```php
use Drupal\file\Entity\File; 
use Drupal\products\Entity\Importer;  
```

我们使用给定的 ID 加载配置实体，然后断言生成的对象是正确接口的实例。这检查我们实际上是否保存了实体。接下来，我们根据在导入器配置实体中找到的 ID 加载文件实体，并断言它本身也实现了正确的接口。这证明了文件实际上已保存且配置正确。

而不是以相同的方式程序化地检查其余字段值，我们选择导航到导入器实体的编辑表单并断言值已正确预填充：

```php
$this->drupalGet('admin/structure/importer/test_csv_importer/edit'); 
$assert->pageTextContains('Edit Test CSV Importer'); 
$assert->fieldValueEquals('label', 'Test CSV Importer'); 
$assert->fieldValueEquals('plugin', 'csv'); 
$assert->checkboxChecked('update_existing'); 
$assert->fieldValueEquals('source', 'testing'); 
$page->hasLink('products.csv'); 
$bundle_field = $this->bundle->label() . ' (' . $this->bundle->id() . ')'; 
$assert->fieldValueEquals('bundle', $bundle_field);  
```

`fieldValueEquals()`和`checkboxChecked()`方法对于检查字段值很有用。此外，我们还使用`hasLink()`方法检查页面上是否存在具有该名称的链接。这实际上是为了证明上传的文件显示正确：

![图片](img/f24f8b1c-23ea-411e-bf80-c6db74f5fcbc.png)

最后，由于捆绑字段是一个引用字段而不是简单的文本字段，我们需要构建测试框架实际看到的值，其模式如下：`标签 (ID)`。

有了这个，我们的测试就完成了，我们可以运行整个测试：

```php
../vendor/bin/phpunit ../modules/custom/products/tests/src/Kernel/CsvImporterTest.php 
```

# 摘要

在本章中，我们简要讨论了 Drupal 8 中的自动化测试。我们首先介绍了编写自动化测试为什么有用且实际上很重要，然后简要概述了几种流行的软件开发测试方法。

Drupal 8 通过集成 PHPUnit 框架，在测试方面相对于其前辈具有优势。正如我们所看到的例子，它具有相当多的方法论能力。我们有单元测试——这是测试中最低级别的形式，它专注于单个架构单元，并且是所有测试中运行速度最快的。然后我们有内核测试，这是集成测试，专注于较低级别的组件及其交互。接下来，我们有功能测试，这是较高级别的测试，专注于与浏览器的交互。最后，我们有功能 JavaScript 测试，它扩展了后者，并将 Selenium 和 Chrome 纳入其中，以便测试依赖于 JavaScript 的功能。

我们还看到，所有这些不同类型的测试都集成了 PHPUnit，因此我们可以使用这个工具运行它们。这意味着所有不同类型的测试在注册到 Drupal 时都遵循相同的“规则”，即目录放置、命名空间和 PHPDoc 信息。

自动化测试的世界非常庞大，一本书中不可能涵盖所有不同的测试方式。因此，特别是对于初学者来说，在阅读 Drupal 和 PHPUnit 代码和文档时，通往良好测试覆盖率的道路充满了试错，甚至偶尔会有挫败感。但从中，我们得到了始终稳定且不受回归影响的代码。
