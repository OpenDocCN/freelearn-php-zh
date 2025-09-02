# 值对象

通过使用 `self` 关键字，我们不需要... 值对象是领域驱动设计的基本构建块，它们用于在代码中建模你的通用语言的概念。值对象不仅仅是领域中的一个事物——它衡量、量化或描述某物。值对象可以看作是小型、简单的对象——例如货币或日期范围——它们的等价性不是基于身份，而是基于所包含的内容。

例如，可以使用值对象来模拟产品价格。在这种情况下，它不是代表一个事物，而是一个值，它允许我们衡量一个产品值多少钱。这些对象的内存占用非常小（通过它们的组成部分计算得出）并且几乎没有开销。因此，在表示相同值时，新实例的创建比引用重用更受欢迎。然后根据两个实例的字段的可比性来检查等价性。

# 定义

Ward Cunningham [定义](http://c2.com/cgi/wiki?ValueObject) 值对象为：

对某物的衡量或描述。值对象的例子包括像数字、日期、货币和字符串这样的东西。通常，它们是使用相当广泛的小对象。它们的身份基于它们的状态而不是它们的对象身份。这样，你可以有多个相同概念上的值对象副本。每张 5 美元纸币都有其自己的身份（多亏了它的序列号），但现金经济依赖于每张 5 美元纸币都有与其他 5 美元纸币相同的值。

Martin Fowler [定义](http://martinfowler.com/bliki/ValueObject.html) 值对象为：

一个小的对象，如货币或日期范围对象。它们的关键属性是它们遵循值语义而不是引用语义。你通常可以通过它们的等价性不是基于身份来判断它们，而是如果所有字段都相等，则两个值对象是相等的。尽管所有字段都相等，但如果子集是唯一的，则不需要比较所有字段——例如，货币对象的货币代码就足以测试等价性。一个一般的启发式方法是值对象应该是完全不可变的。如果你想更改一个值对象，你应该用一个新的对象替换它，而不允许更新值对象本身的值——可更新的值对象会导致别名问题。

值对象的例子包括数字、文本字符串、日期、时间、一个人的全名（由名、中名、姓和头衔组成）、货币、颜色、电话号码和邮政地址。

练习

尝试在你的当前领域中定位更多潜在的值对象示例。

# 值对象与实体

考虑以下来自 [维基百科](http://en.wikipedia.org/wiki/Domain-driven_design#Building_blocks_of_DDD) 的例子，以更好地理解值对象与实体之间的区别：

+   **值对象**：当人们交换美元纸币时，他们通常不会区分每一张独特的纸币；他们只关心美元纸币的面值。在这种情况下，美元纸币是值对象。然而，联邦储备银行可能关心每一张独特的纸币；在这种情况下，每一张纸币都是一个实体。

+   **实体**：大多数航空公司会在每架航班上唯一区分每个座位。在这种情况下，每个座位都是一个实体。然而，西南航空、易捷航空和瑞安航空不会区分每个座位；所有座位都是相同的。在这种情况下，座位实际上是一个值对象。

练习

考虑地址的概念（街道、号码、邮编等）。在什么情况下可以将地址建模为实体而不是值对象？与同伴讨论你的发现。

# 货币与货币示例

`Currency`和`Money`值对象可能是解释值对象最常用的例子，归功于[货币模式](http://martinfowler.com/eaaCatalog/money.html)。此设计模式提供了一种避免浮点数舍入问题的解决方案，从而允许进行确定性计算。

在现实世界中，货币以与米和码相同的方式描述货币单位。每种货币都使用一个三个字母的大写 ISO 代码表示：

```php
class Currency 
{ 
    private $isoCode;

    public function __construct($anIsoCode)
    {
        $this->setIsoCode($anIsoCode);
    }

    private function setIsoCode($anIsoCode)
    {
        if (!preg_match('/^[A-Z]{3}$/', $anIsoCode)) {
           throw new InvalidArgumentException();
        }

        $this->isoCode = $anIsoCode;
    }

    public function isoCode()
    {
        return $this->isoCode;
    }
}

```

值对象的主要目标之一也是面向对象设计的圣杯：封装。通过遵循此模式，你将最终拥有一个专门的位置来放置给定概念的所有验证、比较逻辑和行为。

货币的额外验证

在前面的代码示例中，我们可以使用 AAA 货币 ISO 代码构建一个货币。这完全不正确。编写一个更具体的规则来检查 ISO 代码是否有效。有效的货币 ISO 代码的完整列表可以在[这里](http://www.xe.com/iso4217.php)找到。如果你需要帮助，请查看[Money](https://github.com/moneyphp/money)包管理器库。

货币用于衡量特定货币的数量。它使用金额和货币进行建模。在货币模式的情况下，金额使用货币最不重要的分数的整数表示实现——例如，在美元或欧元的情况下，是分。

作为额外奖励，你可能还会注意到我们正在使用[自我封装](http://martinfowler.com/bliki/SelfEncapsulation.html)来设置 ISO 代码，这集中了值对象本身的变化：

```php
class Money 
{ 
    private $amount; 
    private $currency;

    public function __construct($anAmount, Currency $aCurrency)
    {
        $this->setAmount($anAmount);
        $this->setCurrency($aCurrency);
    }

    private function setAmount($anAmount)
    {
        $this->amount = (int) $anAmount;
    }

    private function setCurrency(Currency $aCurrency)
    {
        $this->currency = $aCurrency;
    }

    public function amount()
    {
        return $this->amount;
    }

    public function currency()
    {
        return $this->currency;
    }
}

```

现在你已经了解了值对象的形式定义，让我们更深入地了解它们提供的强大功能。

# 特征

在将通用语言概念建模到代码中时，你应该始终优先考虑值对象而不是实体。值对象更容易创建、测试、使用和维护。

考虑到这一点，你可以确定所讨论的概念是否可以建模为值对象，如果：

+   它测量、量化或描述领域中的事物

+   它可以保持不变

+   它通过将相关属性作为整体单元组合来模拟一个概念整体

+   它可以通过值相等与其他对象进行比较

+   当测量或描述发生变化时，它可以完全替换

+   它为协作者提供无副作用的操作

# 测量、量化或描述

如前所述，值对象不应仅被视为领域中的“事物”。作为一个值，它测量、量化或描述领域中的概念。

在我们的示例中，`Currency`对象描述了它是哪种类型的货币。`Money`对象测量或量化特定货币的单位。

# 不可变性

这是理解最重要的方面之一。对象值在其生命周期内不应能够被更改。正因为这种不可变性，值对象易于推理和测试，并且没有不希望或不预期的副作用。因此，值对象应通过其构造函数创建。为了构建一个，你通常通过这个构造函数传递所需的原始类型或其他值对象。

值对象始终处于有效状态；这就是为什么我们通过单个原子步骤创建它们。带有多个设置器和获取器的空构造函数将创建责任转移到客户端，导致[贫血领域模型](http://www.martinfowler.com/bliki/AnemicDomainModel.html)，这被认为是一种反模式。

还应指出，不建议在值对象中保留实体的引用。实体是可变的，持有它们的引用可能导致值对象中出现不希望或不预期的副作用。

在支持[方法重载](http://en.wikipedia.org/wiki/Function_overloading)的语言中，例如 Java，你可以创建具有相同名称的多个构造函数。这些构造函数都提供了不同的选项来构建相同类型的对象。在 PHP 中，我们能够通过[工厂方法模式](http://en.wikipedia.org/wiki/Factory_method_pattern)提供类似的能力，这些特定的工厂方法也被称为语义构造函数。`fromMoney`的主要目标是提供比普通构造函数更多的上下文意义。更激进的方案建议将`__construct`方法设为私有，并使用语义构造函数来构建每个实例。

在我们的`Money`对象中，我们可以添加一些有用的工厂方法，如下所示：

```php
class Money 
{ 
    // ...
    public static function fromMoney(Money $aMoney)
    {
        return new self(
            $aMoney->amount(),
            $aMoney->currency()
        );
    }

    public static function ofCurrency(Currency $aCurrency)
    {
        return new self(0, $aCurrency);
    }
}

```

通过使用`self`关键字，我们不会将代码与类名耦合。因此，对类名或命名空间的更改不会影响这些工厂方法。这个小的实现细节有助于在以后重构代码时。

静态与自身

当一个值对象从另一个值对象继承时，使用静态而非自身可能会导致不希望的问题。

由于这种不可变性，我们必须考虑如何在有状态上下文中处理常见的可变操作。如果我们需要状态变化，我们现在必须返回一个新的带有这种变化的值对象表示。如果我们想增加例如 `Money` 值对象的数量，我们必须返回一个新的带有所需修改的 `Money` 实例。

幸运的是，遵守这个规则相对简单，如下面的示例所示：

```php
class Money 
{ 
   // ...
    public function increaseAmountBy($anAmount)
    {
        return new self(
            $this->amount() + $anAmount,
            $this->currency()
        );
    }
}

```

`increaseAmountBy` 返回的 `Money` 对象与接收方法调用的 `Money` 客户端对象不同。这可以在下面的示例比较检查中观察到：

```php
$aMoney = new Money(100, new Currency('USD')); 
$otherMoney = $aMoney->increaseAmountBy(100);

var_dump($aMoney === otherMoney); // bool(false)

$aMoney = $aMoney->increaseAmountBy(100); 
var_dump($aMoney === $otherMoney); // bool(false)

```

# 概念整体

那为什么不直接实现以下示例，从而完全避免需要新的值对象类呢？

```php
class Product 
{ 
    private id; 
    private name;
    /**
     * @var int
     */
    private $amount;
    /**
     * @var string
     */
    private $currency;

    // ...
}

```

这种方法有一些明显的缺陷，比如说，如果你想验证 ISO。让产品负责货币的 ISO 验证（从而违反了单一职责原则）实际上并没有太多意义。如果你想在域的其他部分重用相关的逻辑（以遵守 DRY 原则），这一点尤为突出。

考虑到这些因素，这个用例是抽象成值对象的完美候选者。使用这种抽象不仅给你提供了将相关属性分组在一起的机会，而且还允许你创建更高阶的概念和更具体的通用语言。

练习

与同行讨论一下，是否可以将电子邮件视为值对象。使用的上下文是否重要？

# 值等价

如本章开头所讨论的，如果两个值对象测量的、计量的或描述的内容相同，则这两个值对象是相等的。

例如，想象有两个 `Money` 对象代表 1 美元。我们能认为它们相等吗？在*现实世界*中，两张 1 美元的纸币价值相同吗？当然，它们是相同的。将我们的注意力转回代码，所讨论的值对象指的是 `Money` 的不同实例。然而，它们都代表相同的值，这使得它们相等。

关于 PHP，使用 `==` 操作符比较两个值对象是很常见的。查看[PHP 文档](http://php.net/manual/en/language.oop5.object-comparison.php)中对操作符的定义，可以突出一个有趣的行为：

当使用比较操作符 `==` 时，对象变量以简单的方式进行比较，即：如果两个对象实例具有相同的属性和值，并且是同一类的实例，则这两个对象实例是相等的。

这种行为与我们的值对象正式定义一致。然而，由于存在精确的类匹配谓词，在处理子类型值对象时你应该小心。

考虑到这一点，甚至更严格的 `===` 操作符不幸地并没有帮上忙：

当使用身份操作符`===`时，如果对象变量引用的是同一类的同一实例，则它们是相同的。

以下示例应有助于确认这些细微的差异：

```php
$a = new Currency('USD'); 
$b = new Currency('USD');

var_dump($a == $b); // bool(true) 
var_dump($a === $b); // bool(false)

$c = new Currency('EUR');

var_dump($a == $c); // bool(false)
var_dump($a === $c); // bool(false)

```

一种解决方案是在每个值对象中实现一个传统的`equals`方法。此方法负责检查其复合属性的类型和相等性。使用 PHP 内置的类型提示，抽象数据类型比较很容易实现。如果需要，您还可以使用`get_class()`函数来帮助进行可比较性检查。

然而，该语言无法解析在您的领域概念中真正意味着什么，这意味着提供答案的责任在于您。为了比较`Currency`对象，我们只需确认它们关联的 ISO 代码相同。在这种情况下，`===`操作符做得相当不错：

```php
class Currency 
{ 
    // ...
    public function equals(Currency $currency)
    {
        return $currency->isoCode() === $this->isoCode();
    }
}

```

由于`Money`对象使用`Currency`对象，`equals`方法需要执行这种可比较性检查，同时比较金额：

```php
class Money 
{ 
    // ...
    public function equals(Money $money)
    {
        return
            $money->currency()->equals($this->currency()) &&
            $money->amount() === $this->amount();
    }
}

```

# 可替换性

考虑一个包含用于量化其价格的`Money`值对象的`Product`实体。此外，考虑两个具有相同价格的`Product`实体——例如 100 美元。这种场景可以使用两个单独的`Money`对象或两个指向单个值对象的引用来建模。

共享相同的值对象可能存在风险；如果其中一个被修改，两个都会反映这种变化。这种行为可以被认为是一个意外的副作用。例如，如果卡洛斯于 2 月 20 日被雇佣，并且我们知道克里斯蒂安也是在同一天被雇佣的，我们可能会将克里斯蒂安的雇佣日期设置为与卡洛斯相同的实例。如果卡洛斯随后将他的雇佣日期月份改为五月，克里斯蒂安的雇佣日期也会改变。无论这是否正确，这并不是人们所期望的。

由于本例中指出的问题，当持有值对象的引用时，建议整个替换对象而不是修改其值：

```php
$this−>price = new Money(100, new Currency('USD')); 
//...
$this->price = $this->price->increaseAmountBy(200);

```

这种行为类似于 PHP 中基本类型（如字符串）的工作方式。考虑函数`strtolower`。它返回一个新的字符串而不是修改原始字符串。没有使用引用；相反，返回了一个新值。

# 无副作用行为

如果我们想在我们的`Money`类中包含一些额外的行为——比如一个`add`方法——那么检查输入是否符合任何先决条件并保持任何不变性似乎是自然的。在我们的情况下，我们只想添加相同货币的金额：

```php
class Money 
{ 
    // ...
    public function add(Money $money)
    {
        if ($money->currency() !== $this->currency()) {
            throw new InvalidArgumentException();
        }

        $this->amount += $money->amount();
    }
}

```

如果两种货币不匹配，将引发异常。否则，金额将被相加。然而，这段代码有一些不希望出现的陷阱。现在想象一下，在我们的代码中有一个神秘的`otherMethod`方法调用：

```php
class Banking 
{
    public function doSomething() 
    { 
        $aMoney = new Money(100, new Currency('USD')); 

        $this->otherMethod($aMoney);//mysterious call
        // ...
    }
}

```

一切都很顺利，直到，由于某种原因，当我们返回或完成 `otherMethod` 时，我们开始看到意外的结果。突然，`$aMoney` 不再包含 100 美元。发生了什么？如果 `otherMethod` 内部使用我们之前定义的 `add` 方法，会发生什么？你可能没有意识到 `add` 方法会改变 `Money` 实例的状态。这就是我们所说的副作用。你必须避免产生副作用。你不应该修改你的参数。如果你这样做，使用你的对象的开发者可能会遇到奇怪的行为。他们会抱怨，而且他们是对的。

那么，我们如何解决这个问题？简单——通过确保值对象保持不可变，我们避免了这种意外问题。一个简单的解决方案可能是对于每个可能可变操作，都返回一个新的实例，这是 `add` 方法所做的：

```php
class Money 
{ 
    // ...
    public function add(Money $money)
    {
        if (!$money->currency()->equals($this->currency())) {
            throw new \InvalidArgumentException();
        }

        return new self(
            $money->amount() + $this->amount(),
            $this->currency()
        );
    }
}

```

通过这个简单的更改，不可变性得到了保证。每次两个 `Money` 实例相加时，都会返回一个新的结果实例。其他类可以执行任何数量的更改，而不会影响原始副本。没有副作用的代码易于理解，易于测试，且错误率较低。

# 基本类型

考虑以下代码片段：

```php
$a = 10; 
$b = 10; 
var_dump($a == $b); 
// bool(true) 
var_dump($a === $b); 
// bool(true) 
$a = 20; 
var_dump($a); 
// integer(20) 
$a = $a + 30; 
var_dump($a); 
// integer(50); 

```

虽然 `$a` 和 `$b` 是存储在不同内存位置的不同的变量，但在比较时，它们是相同的。它们持有相同的值，因此我们认为是相等的。你可以随时将 `$a` 的值从 `10` 改为 `20`，使新值为 `20` 并消除 `10`。你可以随意替换整数值，无需考虑之前的值，因为你没有修改它；你只是在替换它。如果你对这些变量应用任何操作——例如加法（即 `$a + $b`）——你将得到另一个新值，该值可以分配给另一个变量或之前定义的一个。当你将 `$a` 传递给另一个函数时，除非明确通过引用传递，否则你传递的是一个值。无论 `$a` 在该函数内部是否被修改，都无关紧要，因为在你当前的代码中，你仍然会有原始副本。值对象的行为就像基本类型。

# 测试值对象

值对象与普通对象以相同的方式进行测试。然而，必须测试不可变性和无副作用的行为。一种解决方案是在执行任何修改之前创建你正在测试的值对象的副本。使用实现的相等性检查断言两者相等。执行你想要测试的操作，并断言结果。最后，断言原始对象和副本仍然相等。

让我们将这个应用到实践中，并测试 `Money` 类中 add 方法的无副作用实现：

```php
class MoneyTest extends *Framework*TestCase 
{ 
    /** 
     * @test 
     */ 
    public function copiedMoneyShouldRepresentSameValue()
    { 
        $aMoney = new Money(100, new Currency('USD'));

        $copiedMoney = Money::fromMoney($aMoney);

        $this->assertTrue($aMoney->equals($copiedMoney));
    }

    /**
     * @test
     */
    public function originalMoneyShouldNotBeModifiedOnAddition()
    {
        $aMoney = new Money(100, new Currency('USD'));

        $aMoney->add(new Money(20, new Currency('USD')));

        $this->assertEquals(100, $aMoney->amount());
    }

    /**
     * @test
     */
    public function moniesShouldBeAdded()
    {
        $aMoney = new Money(100, new Currency('USD'));

        $newMoney = $aMoney->add(new Money(20, new Currency('USD')));

        $this->assertEquals(120, $newMoney->amount());
    }

    // ...
}

```

# 持久化值对象

值对象自身不进行持久化；它们通常在聚合（Aggregate）内部进行持久化。尽管在某些情况下可以选择将其作为完整记录进行持久化，但值对象不应作为完整记录进行持久化。相反，最好使用嵌入式值（Embedded Value）或序列化 LOB 模式。这两种模式都可以在持久化对象时使用开源 ORM，如 Doctrine，或定制 ORM。由于值对象较小，嵌入式值通常是最佳选择，因为它提供了一种通过值对象具有的任何属性查询实体的简单方法。然而，如果您不重视通过这些字段进行查询，序列化策略可以非常容易实现。

考虑以下具有字符串 id、`name` 和 `price`（`Money` 值对象）属性的 `Product` 实体。我们故意决定简化这个例子，将 id 设计为字符串而不是值对象：

```php
 class Product
 {
     private $productId;
     private $name;
     private $price;

     public function __construct(
         $aProductId,
         $aName,
         Money $aPrice
     ) {
         $this->setProductId($aProductId);
         $this->setName($aName);
         $this->setPrice($aPrice);
     }

     // ...
 }

```

假设您有一个 第十章，用于持久化 `Product` 实体的 *仓库*，创建和持久化一个新的 `Product` 的实现可能看起来像这样：

```php
$product = new Product(
    $productRepository->nextIdentity(), 
    'Domain-Driven Design in PHP', 
    new Money(999, new Currency('USD')) 
);
$productRepository−>persist(product);

```

现在我们来看看可以用来持久化包含值对象的 `Product` 实体的临时性 ORM 和 Doctrine 实现。我们将突出嵌入式值和序列化 LOB 模式的应用，以及持久化单个值对象和它们的集合之间的差异。

为什么选择 Doctrine？

[Doctrine](http://www.doctrine-project.org/projects/orm.html) 是一个优秀的 ORM。它解决了 PHP 应用面临的大部分需求。它拥有一个庞大的社区。通过正确调优的设置，它可以与定制 ORM（不失去可维护性）的性能相同，甚至更好。我们建议在处理实体和业务逻辑时，大多数情况下使用 Doctrine。这将为您节省大量时间和精力。

# 持久化单个值对象

对于持久化单个值对象，有许多不同的选项。这些选项从使用序列化 LOB 或嵌入式值作为映射策略，到使用临时性 ORM 或开源替代品，如 Doctrine。我们认为临时性 ORM 是公司可能为了在数据库中持久化实体而开发的定制 ORM。在我们的场景中，临时性 ORM 代码将使用 [DBAL](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/) 库来实现。根据 [官方文档](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/introduction.html)，**Doctrine 数据库抽象** & **访问层**（**DBAL**）在类似 PDO 的 API 周围提供了一个轻量级且薄的运行时层，以及许多额外的横向功能，如通过面向对象的 API 进行数据库模式自省和操作。

# 使用临时性 ORM 的嵌入式值

如果我们处理的是使用嵌入式值模式的 Ad Hoc ORM，我们需要在实体表中为值对象中的每个属性创建一个字段。在这种情况下，当持久化`Product`实体时需要两个额外的列——一个用于值对象的金额，另一个用于其货币 ISO 代码：

```php
CREATE TABLE `products` (
    id INT NOT NULL,
    name VARCHAR( 255) NOT NULL,
    price_amount INT NOT NULL,
    price_currency VARCHAR( 3) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

为了在数据库中持久化实体，我们的第十章，*存储库*必须映射实体和`Money`值对象中的每个字段。

如果您使用基于 DBAL 的 Ad hoc ORM 存储库——让我们称它为`DbalProductRepository`——您必须注意创建`INSERT`语句，绑定参数，并执行该语句：

```php
class DbalProductRepository
    extends DbalRepository
    implements ProductRepository
{
     public function add(Product $aProduct)
     {
         $sql = 'INSERT INTO products VALUES (?, ?, ?, ?)' ;
         $stmt = $this->connection()->prepare($sql);
         $stmt->bindValue(1, $aProduct->id());
         $stmt->bindValue(2, $aProduct->name());
         $stmt->bindValue(3, $aProduct->price()->amount());
         $stmt->bindValue(4, $aProduct
             ->price()->currency()->isoCode());
         $stmt->execute();

       // ...
     }
 }

```

在执行此代码片段以创建`Products`实体并将其持久化到数据库后，每个列都填充了所需的信息：

```php
mysql> select * from products \G
*************************** 1\. row ***************************
id: 1
name: Domain-Driven Design in PHP
price_amount: 999
price_currency: USD
1 row in set (0.00 sec)

```

如您所见，您可以通过 Ad hoc 方式映射您的值对象和查询参数，以持久化您的值对象。然而，事情并不像看起来那么简单。让我们尝试获取带有其关联的`Money`值对象的持久化产品。一个常见的方法是执行一个`SELECT`语句并返回一个新的实体：

```php
class DbalProductRepository
    extends DbalRepository
    implements ProductRepository
{
    public function productOfId($anId)
    {
        $sql = 'SELECT * FROM products WHERE id = ?';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1, $anId);
        $res = $stmt->execute();
        // ...

        return new Product(
            $row['id'],
            $row['name'],
            new Money(
                $row['price_amount'],
                new Currency($row['price_currency'])
            )
        );
    }
}

```

这种方法有一些好处。首先，您可以轻松地逐步阅读持久化和随后的创建过程。其次，您可以根据值对象的任何属性执行查询。最后，持久化实体的空间需求正好是所需的——不多也不少。

然而，使用 Ad hoc ORM 方法有其缺点。如第六章中所述，*领域事件*，实体（以聚合形式）应该在构造函数中触发事件，如果您的领域对聚合的创建感兴趣。如果您使用新操作符，您将根据从数据库中获取聚合的次数触发事件。

这是 Doctrine 使用内部代理和`serialize`以及`unserialize`方法来在不使用构造函数的情况下，以特定状态重新构成一个对象及其属性的原因之一。一个实体在其生命周期中应该只使用新操作符创建一次：

构造函数

构造函数不需要为对象中的每个属性包含一个参数。考虑一下博客文章。构造函数可能需要一个 id 和一个标题；然而，在内部它也可以设置其状态属性为草稿。当发布文章时，应该调用发布方法以相应地更改其状态并设置发布日期。

如果您的意图仍然是推出您自己的 ORM，请准备好解决一些基本问题，例如事件、不同的构造函数、值对象、延迟加载关系等。这就是为什么我们建议为领域驱动设计应用程序尝试使用 Doctrine。

此外，在这个例子中，您需要创建一个继承自`Product`实体并能够从数据库中重新构成实体的`DbalProduct`实体，而不使用新操作符，而是使用静态工厂方法。

# Doctrine >= 2.5.*中的嵌入式值（Embeddables）

目前最新的稳定 Doctrine 版本是*版本 2.5*，它提供了映射值对象的支持，从而消除了在*Doctrine 2.4*中自己执行此操作的需求。自 2015 年 12 月以来，Doctrine 还支持嵌套嵌入式对象。虽然支持率不是 100%，但已经足够高，可以尝试一下。如果它不适合您的场景，请查看下一节。对于官方文档，请查看 Doctrine [嵌入式对象参考](http://doctrine-orm.readthedocs.org/en/latest/tutorials/embeddables.html)。如果正确实现，这绝对是我们最推荐的选择。这将是一个最简单、最优雅的解决方案，同时也为您的*DQL*查询提供了搜索支持。

由于`Product`、`Money`和`Currency`类已经展示过，剩下要展示的只是 Doctrine 映射文件：

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping 

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Product"
        table="product">
        <id
            name="id"
            column="id"
            type="string"
            length="255">
            <generator strategy="NONE">
            </generator>
        </id>

        <field
            name="name"
            type="string"
            length="255"
        />

        <embedded
            name="price"
            class="Ddd\Domain\Model\Money"
        />
    </entity>
</doctrine-mapping>

```

在产品映射中，我们定义`price`为一个实例变量，它将保存一个`Money`实例。同时，`Money`被设计为一个金额和一个`Currency`实例：

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <embeddable
        name="Ddd\Domain\Model\Money">

        <field
            name="amount"
            type="integer"
        />
        <embedded
            name="currency"
            class="Ddd\Domain\Model\Currency"
        />
    </embeddable>
</doctrine-mapping>

```

最后，是时候展示我们的`Currency`值对象的 Doctrine 映射了：

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <embeddable
        name="Ddd\Domain\Model\Currency">

        <field
            name="iso"
            type="string"
            length="3"
        />
    </embeddable>
</doctrine-mapping>

```

如您所见，上述代码具有一个标准的可嵌入定义，只需一个字符串字段来存储 ISO 代码。这种方法是使用可嵌入对象的最简单方式，并且效果更佳。默认情况下，Doctrine 通过在对象名称前加前缀来命名您的列。您可以通过更改 XML 表示法中的列前缀属性来更改此行为以满足您的需求。

# Doctrine <= 2.4.*中的嵌入式值

如果您仍然停留在*Doctrine 2.4*，您可能会想知道使用*Doctrine < 2.5*的嵌入式值的可接受解决方案是什么。现在我们需要在`Product`实体中代理所有值对象属性，这意味着创建新的人工属性来保存值对象的信息。有了这个，我们可以使用 Doctrine 映射所有这些新属性。让我们看看这对`Product`实体有什么影响：

```php
class Product 
{ 
    private $productId; 
    private $name; 
    private $price;
    private $surrogateCurrencyIsoCode;
    private $surrogateAmount;

    public function __construct($aProductId, $aName, Money $aPrice)
    {
        $this->setProductId($aProductId);
        $this->setName($aName);
        $this->setPrice($aPrice);
    }

    private function setPrice(Money $aMoney)
    {
        $this->price = $aMoney;
        $this->surrogateAmount = $aMoney->amount();
        $this->surrogateCurrencyIsoCode =
            $aMoney->currency()->isoCode();
    }

    private function price()
    {
        if (null === $this->price) {
            $this->price = new Money(
                $this->surrogateAmount,
                new Currency($this->surrogateCurrency)
            );
        }
        return $this->price;
    }

    // ...
}

```

如您所见，有两个新属性：一个用于金额，另一个用于货币的 ISO 代码。我们还更新了`setPrice`方法，以便在设置时保持属性一致性。在此基础上，我们还更新了价格获取器，以便返回由新字段构建的`Money`值对象。让我们看看相应的 XML Doctrine 映射应该如何更改：

```php
 <?xml version="1.0" encoding="utf-8"?>
 <doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Product"
        table="product">

        <id
            name="id"
            column="id"
            type="string"
            length="255" >
            <generator strategy="NONE">
            </generator>
        </id>

       <field
           name="name"
           type="string"
           length="255"
       />

       <field
           name="surrogateAmount"
           type="integer"
           column="price_amount"
       />
       <field
           name="surrogateCurrencyIsoCode"
           type="string"
           column="price_currency"
       />
    </entity>
</doctrine-mapping>

```

代理属性

这两个新字段并不严格属于域，因为它们不引用基础设施细节。相反，它们是由于 Doctrine 缺乏可嵌入支持而成为必要的。有其他方法可以将这两个属性推到纯域之外；然而，这种方法更简单、更容易，并且作为权衡，是可以接受的。本书中还有另一种代理属性的使用；你可以在第四章的“实体”部分的*代理身份*子部分中找到它。

如果我们想要将这两个属性推到域之外，可以通过使用[抽象工厂](http://en.wikipedia.org/wiki/Abstract_factory_pattern)来实现。首先，我们需要在我们的基础设施文件夹中创建一个新的实体，`DoctrineProduct`。这个实体将扩展自`Product`实体。所有代理字段都将放置在新类中，并且像价格或`setPrice`这样的方法应该被重新实现。我们将映射 Doctrine 使用新的`DoctrineProduct`而不是`Product`实体。

现在我们能够从数据库中检索实体，但创建一个新的`Product`呢？在某个时候，我们需要调用新的`Product`，但由于我们需要处理`DoctrineProduct`，并且我们不希望我们的应用程序服务了解基础设施细节，因此我们需要使用工厂来创建`Product`实体。所以，在每次使用`new`创建实体的情况下，你将调用`ProductFactory`上的`createProduct`。

为了避免将代理属性放置在原始实体中，可能需要许多额外的类。因此，我们建议将所有值对象代理到同一个实体中，尽管这确实会导致一个不那么纯粹的解决方案。

# 序列化 LOB 和 Ad Hoc ORM

如果向值对象属性添加搜索功能并不重要，可以考虑另一种模式：序列化 LOB。这种模式通过将整个值对象序列化为一个字符串格式，该格式可以轻松持久化和检索。与嵌入式替代方案相比，这种解决方案的最显著区别在于，在后者中，持久化足迹需求减少到单个列：

```php
CREATE TABLE ` products` (
    id INT NOT NULL,
    name VARCHAR( 255) NOT NULL,
    price TEXT NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

为了使用这种方法持久化`Product`实体，需要对`DbalProductRepository`进行更改。在持久化`final`实体之前，必须将`Money`值对象序列化为字符串：

```php
class DbalProductRepository extends DbalRepository implements ProductRepository
{ 
    public function add(Product $aProduct) 
    { 
        $sql = 'INSERT INTO products VALUES (?, ?, ?)'; 
        $stmt = this->connection()->prepare(sql); 
        $stmt->bindValue(1, aProduct−>id()); 
        $stmt->bindValue(2, aProduct−>name()); 
        $stmt->bindValue(3, $this−>serialize($aProduct->price())); 

        // ...
    }

    private function serialize($object)
    {
        return serialize($object);
    }
}

```

让我们看看我们的产品现在在数据库中的表示。表列`price`是一个`TEXT`类型的列，其中包含表示 9.99 美元的`Money`对象的序列化：

```php
mysql > select * from products \G
*************************** 1.row***************************
id   : 1
name : Domain-Driven Design in PHP
price : O:22:"Ddd\Domain\Model\Money":2:{s:30:"Ddd\Domain\Model\\
Money amount";i :
999;s:32:"Ddd\Domain\Model\Money currency";O : 25:"Ddd\Domain\Model\\
Currency":1:{\
s:34:" Ddd\Domain\Model\Currency isoCode";s:3:"USD";}}1 row in set(\ 0.00 sec)

```

这种方法可以完成任务。然而，由于在代码中重构类时可能出现问题，因此不推荐使用。你能想象如果我们决定重命名我们的`Money`类会出现什么问题吗？你能想象当我们把`Money`类从一个命名空间移动到另一个命名空间时，我们的数据库表示需要做出哪些更改吗？另一个权衡，如前所述，是缺乏查询能力。无论你是否使用 Doctrine，使用序列化策略时，编写一个查询以获取比 200 美元便宜的产品几乎是不可能的。

查询问题只能通过使用嵌入式值来解决。然而，可以通过用于处理序列化过程的专用库来解决序列化重构问题。

# 使用 JMS Serializer 改进序列化

当处理类和命名空间重构时，序列化/反序列化原生的 PHP 策略存在问题。一个替代方案是使用你自己的序列化机制——例如，将数量、一个字符分隔符（如`|`）和货币 ISO 代码连接起来。然而，还有一个更受欢迎的方法：使用开源序列化库，如[JMS Serializer](http://jmsyst.com/libs/serializer)。让我们看看在序列化`Money`对象时应用它的一个例子：

```php
$myMoney = new Money(999, new Currency('USD'));

$serializer = JMS\Serializer\SerializerBuilder::create()->build(); 
$jsonData = $serializer−>serialize(myMoney, 'json');

```

为了反序列化对象，过程很简单：

```php
$serializer = JMS\Serializer\SerializerBuilder::create()->build(); 
// ... 
$myMoney = $serializer−>deserialize(jsonData, 'Ddd', 'json');

```

通过这个例子，你可以重构你的`Money`类，而无需更新你的数据库。JMS Serializer 可以在许多不同的场景中使用——例如，当与 REST API 一起工作时。一个重要特性是能够指定在序列化过程中应该省略对象哪些属性——例如，密码。

查阅[映射参考](http://jmsyst.com/libs/serializer/master/reference/xml_reference)和[食谱](http://jmsyst.com/libs/serializer/master/cookbook)以获取更多信息。JMS Serializer 在任何领域驱动设计项目中都是必不可少的。

# 使用 Doctrine 序列化 LOB

在 Doctrine 中，有不同方式序列化对象以便最终持久化。

# Doctrine 对象映射类型

Doctrine 支持序列化 LOB 模式。有许多预定义的映射类型你可以使用，以便将实体属性与数据库列或表匹配。其中一种映射是对象类型，它使用`serialize()`和`unserialize()`将 SQL CLOB 映射到 PHP 对象。

根据[Doctrine DBAL 2 文档](http://doctrine-orm.readthedocs.io/projects/doctrine-dbal/en/latest/reference/types.html#object)，`object`类型：

根据 PHP 序列化映射和转换对象数据。如果你需要存储你对象数据的精确表示，你应该考虑使用此类型，因为它使用序列化来在数据库中以字符串形式表示你对象的精确副本。从数据库检索的值始终通过反序列化转换为 PHP 的对象类型，如果没有数据则转换为 null。

由于无法在数据库中本地存储 PHP 对象表示，此类型将始终映射到数据库供应商的文本类型。此外，此类型需要一个 SQL 列注释提示，以便可以从数据库中反向工程。如果供应商不支持列注释，Doctrine 无法正确映射此类型，并将回退到文本类型。

由于 PostgreSQL 内置的文本类型不支持 NULL 字节，对象类型将导致反序列化错误。解决这个问题的一个方法是手动将 PHP 对象 `serialize()/unserialize()` 和 `base64_encode()/base64_decode()` 存储到文本字段中。

让我们通过使用对象类型来查看为产品实体可能的 XML 映射：

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Product"
        table="products">

        <id
            name="id"
            column="id"
            type="string"
            length="255">
            <generator strategy="NONE">
            </generator>
        </id>
        <field
            name="name"
            type="string"
            length="255"
        />
        <field
            name="price"
            type="object"
        />
    </entity>
</doctrine-mapping>

```

关键新增是 `type="object"`，它告诉 Doctrine 我们将使用对象映射。让我们看看我们如何使用 Doctrine 创建和持久化 `Product` 实体：

```php
// ... 
$em−>persist($product); 
$em−>flush($product);

```

让我们检查一下，如果我们现在从数据库中检索 `Product` 实体，它是否以预期的状态返回：

```php
// ...
$repository = $em->getRepository('Ddd\\Domain\\Model\\Product');
$item = $repository->find(1);
var_dump($item);

/*
class Ddd\Domain\Model\Product#177 (3) {
    private $productId => int(1)
    private $name => string(41) "Domain-Driven Design in PHP"
    private $money => class Ddd\Domain\Model\Money#174 (2) {
        private $amount => string(3) "100"
        private $currency => class Ddd\Domain\Model\Currency#175 (1){
            private $isoCode => string(3) "USD"
        }
    }
}
* /

```

最后但同样重要的是，[Doctrine DBAL 2 文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#doctrine-mapping-types) 指出：

对象类型是通过引用比较的，而不是通过值。当引用改变时，Doctrine 会更新这个值，因此表现得像这些对象是不可变值对象。

这种方法与临时性 ORM 所遇到的重构问题相同。对象映射类型内部使用 `serialize/unserialize`。那么，我们是否可以使用自己的序列化方法呢？

# Doctrine 自定义类型

另一个选项是使用 Doctrine 自定义类型来处理值对象的持久化。自定义类型向 Doctrine 添加了一个新的映射类型——它描述了实体字段与数据库表示之间的自定义转换，以便持久化前者。

如 [Doctrine DBAL 2 文档](http://doctrine-orm.readthedocs.io/projects/doctrine-dbal/en/latest/reference/types.html#custom-mapping-types) 所解释：

仅重新定义数据库类型如何映射到所有现有的 Doctrine 类型并不十分有用。你可以通过扩展 `Doctrine\DBAL\Types\Type` 来定义自己的 Doctrine 映射类型。你需要实现 4 个不同的方法才能使其工作。

使用对象类型，序列化步骤包括诸如类等信息，这使得安全重构我们的代码变得相当困难。

让我们尝试改进这个解决方案。考虑一个自定义序列化过程，它可以解决这个问题。

其中一种方法是将 `Money` 值对象以 `amount|isoCode` 格式编码为字符串持久化到数据库中：

```php
use Ddd\Domain\Model\Currency;
use Ddd\Domain\Model\Money;
use Doctrine\DBAL\Types\TextType;
use Doctrine\DBAL\Platforms\AbstractPlatform;

class MoneyType extends TextType 
{ 
    const MONEY = 'money';

    public function convertToPHPValue(
        $value,
        AbstractPlatform $platform
    ) {
        $value = parent::convertToPHPValue($value, $platform);
        $value = explode('|', $value);
        return new Money(
            $value[0],
            new Currency($value[1])
        );
    }

    public function convertToDatabaseValue(
        $value,
        AbstractPlatform $platform
    ) {
        return implode(
           '|',
           [
               $value->amount(),
               $value->currency()->isoCode()
           ]
        );
    }

    public function getName()
    {
        return self::MONEY;
    }
}

```

使用 Doctrine，你需要注册所有自定义类型。通常使用一个 `EntityManagerFactory` 来集中创建这个 `EntityManager`。

或者，你也可以通过引导应用程序来执行这一步：

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools\Setup;

class EntityManagerFactory 
{ 
    public function build() 
    { 
        Type::addType( 
            'money',
            'Ddd\Infrastructure\Persistence\Doctrine\Type\MoneyType' 
        );

        return EntityManager::create(
            [
                'driver' => 'pdo_mysql',
                'user' => 'root',
                'password' => '',
                'dbname' => 'ddd',
            ],
            Setup::createXMLMetadataConfiguration(
                [__DIR__.'/config'],
                true
            )
        );
    }
}

```

现在，我们需要在映射中指定我们想要使用我们的自定义类型：

```php
<?xml version = "1.0" encoding = "utf-8"?>
<doctrine-mapping>
    <entity
        name = "Product"
        table = "product">

        <!-- ... -->
        <field
            name = "price"
            type = "money"
        />
    </entity>
</doctrine-mapping>

```

为什么使用 XML 映射？

多亏了 XML 映射文件头部的 XSD 架构验证，许多 **集成开发环境（IDEs**） 设置提供了对映射定义中所有元素和属性的自动完成功能。然而，在其他部分的书里，我们使用 YAML 来展示不同的语法。

让我们检查数据库，看看使用这种方法是如何持久化价格的：

```php
mysql> select * from products \G
*************************** 1\. row***************************
id: 1
name: Domain-Driven Design in PHP
price: 999|USD
1 row in set (0.00 sec)

```

与之前的方法相比，这种方法在未来的重构方面有所改进。然而，由于列的格式，搜索能力仍然有限。使用 Doctrine 自定义类型，你可以稍微改善这种情况，但这仍然不是构建你的 DQL 查询的最佳选项。有关更多信息，请参阅 [Doctrine 自定义映射类型](http://doctrine-orm.readthedocs.org/en/latest/cookbook/custom-mapping-types.html)。

讨论时间

思考并与同伴讨论如何使用 JMS `序列化` 和 `反序列化` 值对象来创建 Doctrine 自定义类型。

# 持久化值对象集合

假设我们现在想向我们的 `Product` 实体添加一个要持久化的价格集合。这些价格可能代表产品在其生命周期中承担的不同价格，或者不同货币中的产品价格。这可以命名为 `HistoricalPrice`，如下所示：

```php
class HistoricalProduct extends Product 
{ 
    /** 
     * @var Money[] 
     */ 
    protected $prices;

    public function __construct(
        $aProductId, 
        $aName, 
        Money $aPrice,
        array $somePrices
    ){
        parent::__construct($aProductId, $aName, $aPrice);
        $this->setPrices($somePrices);
    }

    private function setPrices(array $somePrices)
    {
        $this->prices = $somePrices;
    }

    public function prices()
    {
        return $this->prices;
    }
}

```

`HistoricalProduct` 从 `Product` 继承，因此继承了相同的行为，以及价格集合功能。

如前几节所述，如果你不关心查询能力，序列化是一个可行的方案。然而，如果我们知道要持久化的价格数量，嵌入式值是一个可能的选择。但如果我们想持久化一个不确定数量的历史价格会发生什么呢？

# 集合序列化到单个列

将值对象集合序列化到单个列中可能是最简单的解决方案。在关于持久化单个值对象的章节中解释的所有内容都适用于这种情况。使用 Doctrine，你可以使用对象或自定义类型——但需要考虑一些额外的因素：值对象应该体积小，但如果你想持久化大量集合，请确保考虑数据库引擎可以处理的最大列长度和最大行宽度。

练习

提出使用 `Doctrine` 对象类型和 `Doctrine 自定义` 类型实现策略，以不同价格持久化一个产品。

# 由连接表支持的集合

如果你想通过其相关值对象（Value Objects）持久化和查询实体（Entity），你可以选择将值对象作为实体持久化。在领域（Domain）方面，这些对象仍然是值对象，但我们需要给它们一个 id，并将它们与拥有者（owner）建立一个一对一/一对多（one-to-many/one-to-one）的关系，即一个真正的实体。总结来说，你的对象关系映射（ORM）将值对象集合作为实体处理，但在你的领域（Domain）中，它们仍然被视为值对象。

连接表策略背后的主要思想是创建一个连接所有者实体（owner Entity）及其值对象的表。让我们看看数据库表示：

```php
CREATE TABLE ` historical_products` (
    `id` char( 36) COLLATE utf8mb4_unicode_ci NOT NULL,
    `name` varchar( 255) COLLATE utf8mb4_unicode_ci NOT NULL,
    `price_amount` int( 11 ) NOT NULL,
    `price_currency` char( 3) COLLATE utf8mb4_unicode_ci NOT NULL,
     PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

`historical_products`表将与产品表看起来相同。记住，`HistoricalProduct`扩展了`Product`实体，以便更容易地展示如何处理持久化集合。现在需要一个新表来持久化`Product`实体可以处理的所有的不同`Money`值对象：

```php
CREATE TABLE `prices`(
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `amount` int(11) NOT NULL,
    `currency` char(3) COLLATE utf8mb4_unicode_ci NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

最后，需要一个关联产品和价格的表：

```php
CREATE TABLE `products_prices` (
    `product_id` char( 36) COLLATE utf8mb4_unicode_ci NOT NULL,
    `price_id` int( 11 ) NOT NULL,
    PRIMARY KEY (`product_id`, `price_id`),
    UNIQUE KEY `UNIQ_62F8E673D614C7E7` (`price_id`),
    KEY `IDX_62F8E6734584665A` (`product_id`),
    CONSTRAINT `FK_62F8E6734584665A` FOREIGN KEY (`product_id`)
        REFERENCES `historical_products` (`id`),
    CONSTRAINT `FK_62F8E673D614C7E7` FOREIGN KEY (`price_id`)
        REFERENCES `prices`(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

# 由连接表支持的集合与 Doctrine

Doctrine 要求所有数据库实体都有一个唯一标识符。因为我们想持久化`Money`值对象，所以我们需要添加一个人工标识符，这样 Doctrine 才能处理其持久化。有两种选择：在`Money`值对象中包含代理标识符，或者将其放在扩展类中。

第一个选项的问题是新身份标识（new identity）仅由于数据库持久化层（Database persistence layer）的需要。这个身份标识不是领域的一部分。

第二个选项的问题是需要进行大量的修改以避免所谓的边界泄露（boundary leak）。由于创建`Money`值对象类的新实例会违反反转原则（Inversion Principle），因此不建议从任何领域对象（Domain Object）中创建`Money`值对象类的新实例。解决方案是再次创建一个`Money`工厂（Factory），该工厂需要传递给应用程序服务（Application Services）和任何其他领域对象。

在这个场景中，我们建议使用第一个选项。让我们回顾一下在`Money`值对象中实现它的所需更改：

```php
class Money 
{ 
    private $amount; 
    private $currency;
    private $surrogateId;
    private $surrogateCurrencyIsoCode;

    public function __construct($amount, Currency $currency)
    {
        $this->setAmount($amount);
        $this->setCurrency($currency);
    }

    private function setAmount($amount)
    {
        $this->amount = $amount;
    }

    private function setCurrency(Currency $currency)
    {
        $this->currency = $currency;
        $this->surrogateCurrencyIsoCode =
            $currency->isoCode();  
    }

    public function currency()
    {
       if (null === $this->currency) {
           $this->currency = new Currency(
               $this->surrogateCurrencyIsoCode
           );
       }
       return $this->currency;
    } 

    public function amount()
    {
        return $this->amount;
    }

    public function equals(Money $aMoney)
    {
        return
            $this->amount() === $aMoney->amount() &&
            $this->currency()->equals($this->currency());
    }
}

```

如上所示，已添加了两个新属性。第一个属性，`surrogateId`，在我们的领域（Domain）中不被使用，但它对于持久化基础设施（persistence Infrastructure）将此值对象（Value Object）作为实体（Entity）保存在我们的数据库中是必需的。第二个属性，`surrogateCurrencyIsoCode`，持有货币的 ISO 代码。使用这些新属性，将我们的值对象与 Doctrine 映射起来变得非常容易。

`Money`映射相当直接：

```php
<?xml version = "1.0" encoding = "utf-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Ddd\Domain\Model\Money"
        table="prices">

        <id
            name="surrogateId"
            type="integer"
            column="id">
            <generator
                strategy="AUTO">
            </generator>

        </id>
        <field
            name="amount"
            type="integer"
            column="amount"
        />
        <field
            name="surrogateCurrencyIsoCode"
            type="string"
            column="currency"
        />
    </entity>
</doctrine-mapping>

```

使用 Doctrine，`HistoricalProduct`实体将具有以下映射：

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping 

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Ddd\Domain\Model\HistoricalProduct" 
        table="historical_products"
        repository-class="
            Ddd\Infrastructure\Domain\Model\DoctrineHistoricalProductRepository
        ">
        <many-to-many
            field="prices"
            target-entity="Ddd\Domain\Model\Money">

            <cascade>
                <cascade-all/>
            </cascade>

            <join-table
                name="products_prices">
                <join-columns>
                    <join-column
                        name="product_id"
                        referenced-column-name="id"
                    />
                </join-columns>
                <inverse-join-columns>
                    <join-column
                        name="price_id"
                        referenced-column-name="id"
                        unique="true"
                    />
                </inverse-join-columns>
            </join-table>
        </many-to-many>
    </entity>
</doctrine-mapping>

```

# 由连接表支持的集合与临时对象关系映射（ORM）

使用临时对象关系映射（Ad hoc ORM），其中需要级联`INSERTS`和`JOIN`查询，也可以做到同样的事情。重要的是要小心处理值对象的移除，以避免留下孤立的`Money`值对象。

练习

为`DbalHistoricalRepository`想出一个处理持久化方法（persist method）的解决方案。

# 由数据库实体支持的集合

数据库实体与连接表是相同的解决方案，增加了仅由所有者实体管理的值对象。在当前场景中，考虑到`Money`值对象仅由`HistoricalProduct`实体使用；连接表将过于复杂。因此，可以使用一对一的数据库关系达到相同的结果。

练习

如果使用数据库实体方法，请考虑`HistoricalProduct`和`Money`之间的映射需求。

# NoSQL

那么，关于像*Redis*、*MongoDB*或*CouchDB*这样的 NoSQL 机制呢？不幸的是，你无法避开这些问题。为了使用*Redis*持久化聚合，你需要在设置值之前将其序列化为字符串。如果你使用 PHP 的`serialize`/`unserialize`方法，你将再次面临命名空间或类名重构问题。如果你选择以自定义方式（JSON、自定义字符串等）进行序列化，你将需要在*Redis*检索期间再次重建值对象。

# PostgreSQL JSONB 和 MySQL JSON 类型

如果我们的数据库引擎不仅允许我们使用序列化 LOB 策略，而且还可以根据其值进行搜索，我们将拥有两种方法的最佳之处。好消息是：现在你可以这样做。从*PostgreSQL 版本 9.4*开始，已添加了对[JSONB](http://www.postgresql.org/docs/9.4/static/functions-json.html)的支持。值对象可以作为 JSON 序列化持久化，并在该 JSON 序列化中进行查询。

MySQL 也做了同样的事情。从*MySQL 5.7.8*开始，MySQL 支持一种原生 JSON 数据类型，它允许高效访问**JSON**（**JavaScript 对象表示法**）文档中的数据。根据[MySQL 5.7 参考手册](https://dev.mysql.com/doc/refman/5.7/en/json.html)，JSON 数据类型与在字符串列中存储 JSON 格式字符串相比具有以下优势：

+   自动验证存储在 JSON 列中的 JSON 文档。无效的文档将产生错误。

+   优化存储格式。存储在 JSON 列中的 JSON 文档被转换为一种内部格式，允许快速读取文档元素。当服务器稍后必须读取以这种二进制格式存储的 JSON 值时，无需从文本表示中解析该值。这种二进制格式被结构化，以便服务器可以直接通过键或数组索引查找子对象或嵌套值，而无需读取文档中它们之前或之后的所有值。

如果关系数据库添加了对具有高性能和所有**原子性**、**一致性**、**隔离性**、**持久性**（**ACID**）哲学优势的文档和嵌套文档搜索的支持，它可以在许多项目中节省大量的复杂性。

# 安全性

使用值对象来建模你的领域概念的一个有趣细节是其安全优势。考虑一个在销售机票的上下文中的应用。如果你处理的是国际航空运输协会机场代码，也称为[IATA 代码](https://en.wikipedia.org/wiki/International_Air_Transport_Association_airport_code)，你可以选择使用字符串或使用值对象来建模这个概念。如果你选择使用字符串，想想你会在哪些地方检查这个字符串是否是有效的 IATA 代码。你忘记在某个重要地方的概率有多大？另一方面，想想尝试实例化新的`IATA("BCN'; DROP TABLE users;--")`。如果你在构造函数中集中了*守卫*，然后将 IATA 值对象传递到你的模型中，避免 SQL 注入或类似的攻击就会变得更容易。

如果你想了解更多关于领域驱动设计安全方面的信息，你可以关注[Dan Bergh Johnsson](https://twitter.com/danbjson)或阅读他的[博客](http://dearjunior.blogspot.com.es/search/label/domain%20driven%20security)。

# 总结

在你的领域中用值对象建模那些测量、量化或描述的概念是非常推荐的。正如所示，值对象易于创建、维护和测试。为了在领域驱动设计应用中处理持久化，使用 ORM 是必须的。然而，为了使用 Doctrine 持久化值对象，首选的方式是使用内嵌对象。如果你卡在*版本 2.4*，有两种选择：直接将值对象字段添加到你的实体中并映射它们（不那么优雅，但更容易），或者扩展你的实体（更加优雅，但更复杂）。
