# 第四章：实体

我们已经讨论了首先将域中的所有内容建模为值对象的益处。但在对域进行建模时，可能会遇到一些情况，你会发现普遍语言中的某些概念需要一条身份线索。

# 引言

需要身份的对象的清晰例子包括：

+   一个**人**。一个人总是有一个身份，并且他们的名字或身份证在时间上始终相同。

+   在电子商务系统中，一个**订单**。在这种情况下，每个新创建的订单都有自己的身份，并且随着时间的推移保持不变。

这些概念具有随时间持久存在的身份。无论概念中的数据如何变化，它们的身份保持不变。这就是它们是实体而不是值对象的原因。在 PHP 实现方面，它们将是普通的旧类。例如，考虑以下关于人的情况：

```php
namespace Ddd\Identity\Domain\Model;

class Person
{
    private $identificationNumber;
    private $firstName;
    private $lastName;

    public function __construct(
        $anIdentificationNumber, $aFirstName, $aLastName
    ) {
        $this->identificationNumber = $anIdentificationNumber;
        $this->firstName = $aFirstName;
        $this->lastName  = $aLastName;
    }

    public function identificationNumber()
    {
        return $this->identificationNumber;
    }

    public function firstName()
    {
        return $this->firstName;
    }

    public function lastName()
    {
        return $this->lastName;
    }
 }

```

或者，考虑以下关于订单的情况：

```php
namespace Ddd\Billing\Domain\Model\Order;

class Order
{
    private $id;
    private $amount;
    private $firstName;
    private $lastName;

    public function __construct(
        $anId, Amount $amount, $aFirstName, $aLastName
    ) {
        $this->id = $anId;
        $this->amount = $amount;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }

    public function id()
    {
        return $this->id;
    }

    public function firstName()
    {
        return $this->firstName;
    }

    public function lastName()
    {
        return $this->lastName;
    }
}

```

# 对象与原始类型

大多数情况下，实体的身份以原始类型表示——通常是字符串或整数。但使用值对象来表示它有更多优势：

+   值对象是不可变的，因此不能被修改。

+   值对象是具有自定义行为的复杂类型，这是原始类型所不具备的。以**等价操作**为例。使用值对象，等价操作可以建模并封装在自己的类中，使概念从隐式变为显式。

让我们看看`OrderId`、`Order`身份（它已演变为值对象）的一个可能的实现：

```php
namespace Ddd\Billing\Domain\Model;

class OrderId
{
    private $id;

    public function __construct($anId)
    {
        $this->id = $anId;
    }

    public function id()
    {
        return $this->id;
    }

    public function equalsTo(OrderId $anOrderId)
    {
        return $anOrderId->id === $this->id;
    }
}

```

对于实现`OrderId`，你可以考虑不同的实现方式。上面显示的例子相当简单。正如第三章中解释的，*值对象*，你可以将`__constructor`方法设为私有，并使用静态工厂方法来创建新实例。与你的团队讨论，进行实验，并达成一致。因为实体身份并不复杂，我们的建议是，你在这里不必过于担心。

回到`Order`，是时候更新对`OrderId`的引用了：

```php
 class Order
 {
     private $id;
     private $amount;
     private $firstName;
     private $lastName;

     public function __construct(
         OrderId $anOrderId, Amount $amount, $aFirstName, $aLastName
     ) {
         $this->id = $anOrderId;
         $this->amount = $amount;
         $this->firstName = $aFirstName;
         $this->lastName = $aLastName;
     }

     public function id()
     {
         return $this->id;
     }

     public function firstName()
     {
         return $this->firstName;
     }

     public function lastName()
     {
         return $this->lastName;
     }

     public function amount()
     {
         return $this->amount;
     }
}

```

我们的实体使用值对象建模了身份。让我们考虑创建`OrderId`的不同方法。

# 身份操作

如前所述，实体的身份定义了它。因此，处理它就是实体的重要方面之一。通常有四种方式来定义实体的身份：持久化机制提供身份，客户端提供身份，应用程序本身提供身份，或者另一个边界上下文提供身份。

# 持久化机制生成身份

通常，生成标识符的最简单方法是将它委托给持久化机制，因为绝大多数持久化机制都支持某种类型的标识符生成——比如 MySQL 的`AUTO_INCREMENT`属性或 Postgres 和 Oracle 序列。虽然这很简单，但有一个主要的缺点：我们只有在持久化实体之后才能获得实体的标识符。因此，在某种程度上，如果我们采用由持久化机制生成的标识符，我们将把标识符操作与底层持久化存储耦合起来：

```php
CREATE TABLE `orders` (
    `id` int(11) NOT NULL auto_increment,
    `amount` decimal (10,5) NOT NULL,
    `first_name` varchar(100) NOT NULL,
    `last_name` varchar(100) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

然后我们可能会考虑以下代码：

```php
namespace Ddd\Identity\Domain\Model;

class Person
{
    private $identificationNumber;
    private $firstName;
    private $lastName;

    public function __construct(
        $anIdentificationNumber, $aFirstName, $aLastName
    ) {
        $this->identificationNumber = $anIdentificationNumber;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    } 

    public function identificationNumber()
    {
        return $this->identificationNumber;
    }

    public function firstName()
    {
        return $this->firstName;
    }

    public function lastName()
    {
        return $this->lastName;
    }
}

```

如果你曾经尝试构建自己的 ORM，你肯定已经经历过这种情况。创建一个新的 Person 对象的方法是什么？如果数据库将要生成标识符，我们是否需要在构造函数中传递它？何时何地会有魔法更新 Person 对象的标识符？如果我们最终没有持久化实体，会发生什么？

# 代理标识符

有时在使用 ORM 将实体映射到持久化存储时，会施加一些约束——例如，如果使用`IDENTITY`生成策略，Doctrine 要求一个整数字段。这可能会与需要另一种类型标识符的领域模型冲突。

处理这种情况的最简单方法是通过使用[层超类型](http://martinfowler.com/eaaCatalog/layerSupertype.html)，将用于持久化存储创建的标识符字段放置在其中：

```php
namespace Ddd\Common\Domain\Model;

abstract class IdentifiableDomainObject
{
    private $id;

    protected function id()
    {
        return $this->id;
    }

    protected function setId($anId)
    {
        $this->id = $anId;
    }
}

namespace Acme\Billing\Domain;

use Acme\Common\Domain\IdentifiableDomainObject;

class Order extends IdentifiableDomainObject
{
    private $orderId;

    public function orderId()
    {
        if (null === $this->orderId) {
           $this->orderId = new OrderId($this->id());
        }

        return $this->orderId;
    }
 }

```

# Active Record 与富领域模型的数据映射器

每个项目都会面临选择使用哪种 ORM 的决定。PHP 有很多好的 ORM：Doctrine、Propel、Eloquent、Paris 等等。

其中大多数是[Active Record](http://www.martinfowler.com/eaaCatalog/activeRecord.html)实现。Active Record 实现对于 CRUD 应用程序来说很好，但它并不是富领域模型的理想解决方案，以下是一些原因：

+   Active Record 模式假设实体与数据库表之间存在一对一的关系。因此，它将数据库的设计与对象系统的设计耦合在一起**。**在一个富领域模型中，有时实体是使用可能来自不同数据源的信息构建的。

+   集合和继承等高级功能实现起来很棘手。

+   大多数实现都强制通过继承使用某种类型的构造，这会强加几个约定。这可能导致通过将领域模型与 ORM 耦合，将持久性泄漏到领域模型中。我们看到的唯一不强制从基类继承的 Active Record 实现是来自[Castle Project](http://www.castleproject.org/)的[Castle ActiveRecord](http://docs.castleproject.org/Active%20Record.MainPage.ashx)，这是一个.NET 框架。虽然这导致在生成的实体中持久性和领域关注点之间有一定的分离，但它并没有将低级持久性细节与高级领域设计解耦。

如前一章所述，目前 PHP 最好的 ORM 是 [Doctrine](http://doctrine-project.org)，它实现了 [数据映射模式](http://www.martinfowler.com/eaaCatalog/dataMapper.html)。数据映射将持久性关注点与域关注点解耦，导致持久性无实体。这使得该工具成为想要构建丰富域模型的人的最佳选择。

# 客户端提供身份

有时，在处理某些域时，身份会自然出现，客户端消费域模型。这可能是理想的情况，因为身份可以轻松建模。让我们看看图书销售市场：

```php
namespace Ddd\Catalog\Domain\Model\Book;

class ISBN
{
    private $isbn;

    private function __construct($anIsbn)
    {
        $this->setIsbn($anIsbn);
    }

    private function setIsbn($anIsbn)
    {
        $this->assertIsbnIsValid($anIsbn, 'The ISBN is invalid.');

        $this->isbn = $anIsbn;
    }

    public static function create($anIsbn)
    {
        return new static($anIsbn);
    }

    private function assertIsbnIsValid($anIsbn, $string)
    {
        // ... Validates an ISBN code
    }
}

```

根据 [维基百科](https://en.wikipedia.org/wiki/International_Standard_Book_Number)：**国际标准书号**（ISBN）是一个独特的商业书号。ISBN 被分配给每本书的每个版本和变体（除了再版）。例如，同一本书的电子书、平装版和精装版将各自有不同的 ISBN。如果是在 2007 年 1 月 1 日或之后分配的，ISBN 将有 13 位数字，如果是在 2007 年之前分配的，ISBN 将有 10 位数字。分配 ISBN 的方法基于国家，并且各国之间有所不同，通常取决于一个国家内出版业的规模。

ISBN 的好处在于它已经在域中定义了，它是一个有效的标识符，因为它具有唯一性，并且可以轻松验证。这是一个客户端提供的身份的很好例子：

```php
class Book
{
   private $isbn;
   private $title;

   public function __construct(ISBN $anIsbn, $aTitle)
   {
       $this->isbn  = $anIsbn;
       $this->title = $aTitle;
   }
} 

```

现在，关键在于使用它：

```php
 $book = new Book(
     ISBN::create('...'),
     'Domain-Driven Design in PHP'
 ); 

```

练习

考虑其他在域中构建身份的域，并对其进行建模。

# 应用程序生成身份

如果客户端不能一般性地提供身份，处理身份操作的首选方式是让应用程序生成身份，通常是通过 UUID。如果你没有前述章节中所示的场景，这是我们推荐的方法。

根据 [维基百科](https://en.wikipedia.org/wiki/Universally_unique_identifier)：

UUID 的目的是使分布式系统能够在不进行重大中央协调的情况下唯一标识信息。在此上下文中，"唯一"一词应理解为"实际上唯一"而不是"保证唯一"。由于标识符具有有限的大小，两个不同的项目可能共享相同的标识符。这是一种哈希冲突的形式。标识符的大小和生成过程需要选择，以便在实际中使这种情况尽可能不可能。任何人都可以创建一个 UUID 并用它来标识某物，有合理的信心认为相同的标识符不会无意中被其他人用来标识其他事物。因此，带有 UUID 标记的信息可以在以后合并到一个数据库中，而无需解决标识符（ID）冲突。

PHP 中有几个库可以生成 UUID，它们可以在 Packagist 上找到：[`packagist.org/search/?q=uuid`](https://packagist.org/search/?q=uuid)。最佳推荐是 Ben Ramsey 在以下链接中开发的版本：[`github.com/ramsey/uuid`](https://github.com/ramsey/uuid)，因为它在 GitHub 上有大量的关注者，在 Packagist 上有数百万的安装量。

创建标识的最佳位置是在仓库中（我们将在第十章 Chapter 10，*仓库*中进一步探讨这个问题）：

```php
namespace Ddd\Billing\Domain\Model\Order;

interface OrderRepository
{
    public function nextIdentity();
    public function add(Order $anOrder);
    public function remove(Order $anOrder);
}

```

在使用 Doctrine 时，我们需要创建一个实现该接口的自定义仓库。它基本上会创建新的标识并使用`EntityManager`来持久化和删除实体。一个小变化是将`nextIdentity`实现放入将成为抽象类的接口中：

```php
namespace Ddd\Billing\Infrastructure\Domain\Model\Order;

use Ddd\Billing\Domain\Model\Order\Order;
use Ddd\Billing\Domain\Model\Order\OrderId;
use Ddd\Billing\Domain\Model\Order\OrderRepository;

use Doctrine\ORM\EntityRepository;

class DoctrineOrderRepository
    extends EntityRepository
    implements OrderRepository
{
    public function nextIdentity()
    {
        return OrderId::create();
    }

    public function add(Order $anOrder)
    {
        $this->getEntityManager()->persist($anOrder);
    }

    public function remove(Order $anOrder)
    {
       $this->getEntityManager()->remove($anOrder);
    }
}

```

让我们快速回顾一下`OrderId`值对象的最终实现：

```php
namespace Ddd\Billing\Domain\Model\Order;

use Ramsey\Uuid\Uuid;

class OrderId
{
    private $id;

    private function __construct($anId = null)
    {
        $this->id = $id ? :Uuid::uuid4()->toString();
    }

    public static function create($anId = null )
    {
        return new static($anId);
    }
}

```

关于这种方法的主要担忧，正如你将在以下章节中看到的，是持久化包含值对象的实体有多容易。然而，根据 ORM 的不同，映射实体内部的嵌入式值对象可能会很棘手。

# 其他边界上下文生成标识

这可能是最复杂的标识生成策略，因为它迫使本地实体不仅依赖于本地边界上下文事件，还依赖于外部边界上下文事件。因此，在维护方面，成本会很高。

另一个边界上下文提供了一个接口，用于从本地实体中选择标识。它可以采用一些公开的属性作为自己的属性。

当需要在边界上下文的实体之间进行同步时，通常可以通过在每个需要通知原始实体发生更改的边界上下文中使用事件驱动架构来实现。

# 持久化实体

目前，正如本章前面所讨论的，将实体状态保存到持久存储的最佳工具是 Doctrine ORM。Doctrine 有几种指定实体元数据的方式：通过实体代码中的注解、通过 XML、通过 YAML 或通过纯 PHP。在本章中，我们将深入讨论为什么在映射实体时使用注解不是最佳选择。

# 设置 Doctrine

首先，我们需要通过 Composer 要求 Doctrine。在项目的根目录中，必须执行以下命令：

```php
 > php composer.phar require "doctrine/orm=².5"

```

然后，这些行将允许你设置 Doctrine：

```php
require_once '/path/to/vendor/autoload.php';

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ['/path/to/entity-files'];
$isDevMode = false;

// the connection configuration
$dbParams = [
    'driver'   => 'pdo_mysql',
    'user'     => 'the_database_username',
    'password' => 'the_database_password',
    'dbname'   => 'the_database_name',
];

$config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

```

# 实体的映射

默认情况下，Doctrine 的文档使用注解来展示代码示例。因此，我们开始使用注解编写代码示例，并讨论为什么在可能的情况下应避免使用注解。

为了做到这一点，我们将回顾本章前面讨论过的`Order`类。

# 使用注解代码映射实体

当 Doctrine 发布时，通过在代码示例中使用注解来展示如何映射对象是一种吸引人的方式。

什么是注解？

注释是一种特殊的元数据形式。在 PHP 中，它位于源代码注释下。例如，*PHPDocumentor* 使用注释来构建 API 信息，而 `PHPUnit` 使用一些注释来指定数据提供者或提供关于代码抛出异常的期望：

`class SumTest extends PHPUnit_Framework_TestCase {`

`    /** @dataProvider aMethodName */`

`    public function testAddition() {`

`    //...`

`    }`

`}`

为了将 `Order` 实体映射到持久化存储，应该修改 `Order` 的源代码以添加 Doctrine 注释：

```php
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Column;

/** @Entity */
class Order
{
    /** @Id @GeneratedValue(strategy="AUTO") */
    private $id;

    /** @Column(type="decimal", precision="10", scale="5") */
    private $amount;

    /** @Column(type="string") */
    private $firstName;

    /** @Column(type="string") */
    private $lastName;

    public function __construct(
        Amount $anAmount,
        $aFirstName,
        $aLastName
    ) {
        $this->amount = $anAmount;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }

    public function id()
    {
        return $this->id;
    }

    public function firstName()
    {
        return $this->firstName;
    }

    public function lastName()
    {
        return $this->lastName;
    }

    public function amount()
    {
        return $this->amount;
    }
}

```

然后，要将实体持久化到持久化存储，只需执行以下操作即可：

```php
$order = new Order(
    new Amount(15, Currency::EUR()),
    'AFirstName',
    'ALastName'
);
$entityManager->persist($order);
$entityManager->flush();

```

初看，这段代码看起来很简单，这可以是一个指定映射信息的简单方法。但它是有代价的。最终代码有什么奇怪的地方？

首先，领域关注点与基础设施关注点混合在一起。订单是一个领域概念，而表、列等则是基础设施关注点。

因此，这个实体与源代码中指定的注释映射信息紧密耦合。如果实体需要使用另一个实体管理器和不同的映射元数据来持久化，这将不可能实现。

注释往往会引起副作用和紧密耦合，因此最好不用它们。

那么指定映射信息最好的方法是什么？最好的方法是允许你将映射信息与实体本身分离。这可以通过使用 XML 映射、YAML 映射或 PHP 映射来实现。在这本书中，我们将介绍 XML 映射。

# 使用 XML 映射映射实体

要使用 XML 映射映射 `Order` 实体，需要稍微修改 Doctrine 的设置代码：

```php
require_once '/path/to/vendor/autoload.php';

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ['/path/to/mapping-files'];
$isDevMode = false;

// the connection configuration
$dbParams = [
    'driver'   => 'pdo_mysql',
    'user'     => 'the_database_username',
    'password' => 'the_database_password',
    'dbname'   => 'the_database_name',
];

$config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

```

映射文件应该创建在 Doctrine 将要搜索映射文件的路劲上，并且映射文件应该以完全限定的类名命名，将反斜杠 `\` 替换为点。考虑以下示例：

```php
Acme\Billing\Domain\Model\Order

```

上述示例中的映射文件将命名为如下：

```php
Acme.Billing.Domain.Model.Order.dcm.xml

```

此外，所有映射文件都使用一个专门为指定映射信息而创建的特殊 XML 架构，这很方便：

```php
https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd

```

# 映射实体标识符

我们的标识符，`OrderId`，是一个值对象。正如前一章所看到的，使用 Doctrine、嵌入对象和自定义类型映射值对象有不同的方法。当值对象用作标识符时，最佳选项是自定义类型。

*Doctrine 2.5* 中一个有趣的新特性是，现在可以使用对象作为实体的标识符，只要它们实现了 `__toString()` 魔法方法。因此，我们可以将 `__toString` 添加到我们的标识符值对象中，并在映射中使用它们：

```php
namespace Ddd\Billing\Domain\Model\Order;

use Ramsey\Uuid\Uuid;

class OrderId
{
    // ...

    public function __toString()
    {
        return $this->id;
    }
}

```

检查 Doctrine 自定义类型的实现。它们继承自`GuidType`，因此它们的内部表示将是 UUID。我们需要指定数据库的本地转换。然后在我们使用它们之前，我们需要注册我们的自定义类型。如果你需要这些步骤的帮助，[自定义映射类型](http://doctrine-orm.readthedocs.io/projects/doctrine-orm/en/latest/cookbook/custom-mapping-types.html)是一个很好的参考。

```php
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Doctrine\DBAL\Types\GuidType;

class DoctrineOrderId extends GuidType
{
    public function getName()
    {
        return 'OrderId';
    }

    public function convertToDatabaseValue(
        $value, AbstractPlatform $platform
    ) {
        return $value->id();
    }

    public function convertToPHPValue(
        $value, AbstractPlatform $platform
    ) {
        return new OrderId($value);
    }
}

```

最后，我们将设置自定义类型的注册。同样，我们必须更新我们的引导过程：

```php
require_once '/path/to/vendor/autoload.php';

// ...

\Doctrine\DBAL\Types\Type::addType(
     'OrderId',
     'Ddd\Billing\Infrastructure\Domain\Model\DoctrineOrderId'
);

$config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

```

# 最终映射文件

经过所有这些变化，我们终于准备就绪，现在让我们看一下最终的映射文件。最有趣的细节是检查`OrderId`的 ID 是如何与我们的自定义类型进行映射的：

```php
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity
        name="Ddd\Billing\Domain\Model\Order"
        table="orders">

        <id name="id" column="id" type="OrderId" />

        <field
            name="amount"
            type="decimal"
            nullable="false"
            scale="10"
            precision="5"
        />
        <field
            name="firstName"
            type="string"
            nullable="false"
        />
        <field
            name="lastName"
            type="string"
            nullable="false"
        />
    </entity>
</doctrine-mapping>

```

# 测试实体

测试实体相对简单，因为它们是具有从它们所代表的领域概念派生出的操作的普通 PHP 类。测试的重点应该是实体所保护的那些不变性，因为实体的行为很可能会围绕这些不变性进行建模。

例如，为了简化，假设需要一个博客的领域模型。一个可能的模型可以是这个：

```php
class Post
{
    private $title;
    private $content;
    private $status;
    private $createdAt;
    private $publishedAt;

    public function __construct($aContent, $title)
    {
        $this->setContent($aContent);
        $this->setTitle($title);

        $this->unpublish();
        $this->createdAt(new DateTimeImmutable());
    }

    private function setContent($aContent)
    {
        $this->assertNotEmpty($aContent);

        $this->content = $aContent;
    }

    private function setTitle($aPostTitle)
    {
        $this->assertNotEmpty($aPostTitle);

        $this->title = $aPostTitle;
    }

    private function setStatus(Status $aPostStatus)
    {
        $this->assertIsAValidPostStatus($aPostStatus);

        $this->status = $aPostStatus;
    }

    private function createdAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);

        $this->createdAt = $aDate;
    }

    private function publishedAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);

        $this->publishedAt = $aDate;
    }

    public function publish()
    {
        $this->setStatus(Status::published());
        $this->publishedAt(new DateTimeImmutable());
    }

    public function unpublish()
    {
        $this->setStatus(Status::draft());
        $this->publishedAt = null ;
    }

    public function isPublished()
    {
        return $this->status->equalsTo(Status::published());
    }

    public function publicationDate()
    {
        return $this->publishedAt;
    }
}

class Status
{
    const PUBLISHED = 10;
    const DRAFT = 20;

    private $status;

    public static function published()
    {
        return new self(self::PUBLISHED);
    }

    public static function draft()
    {
        return new self(self::DRAFT);
    }

    private function __construct($aStatus)
    {
        $this->status = $aStatus;
    }

    public function equalsTo(self $aStatus)
    {
        return $this->status === $aStatus->status;
    }
}

```

为了测试这个领域模型，我们必须确保测试覆盖了所有的`Post`不变性：

```php
class PostTest extends PHPUnit_Framework_TestCase
{
     /** @test */
     public function aNewPostIsNotPublishedByDefault()
     {
          $aPost = new Post(
              'A Post Content',
              'A Post Title'
          );

          $this->assertFalse(
              $aPost->isPublished()
          );

          $this->assertNull(
              $aPost->publicationDate()
          );
      }

    /** @test */
    public function aPostCanBePublishedWithAPublicationDate()
    {
        $aPost = new Post(
            'A Post Content',
            'A Post Title'
        );

        $aPost->publish();

        $this->assertTrue(
            $aPost->isPublished()
        );

        $this->assertInstanceOf(
            'DateTimeImmutable',
            $aPost->publicationDate()
        );
    }
}

```

# 日期时间

由于`DateTimes`在实体中广泛使用，我们认为指出针对具有日期类型字段的实体进行单元测试的具体方法很重要。考虑一下，如果一篇`Post`是在过去 15 天内创建的，那么它就是新的：

```php
class Post
{
    const NEW_TIME_INTERVAL_DAYS = 15;

    // ...
    private $createdAt;

    public function __construct($aContent, $title)
    {
        // ...
        $this->createdAt(new DateTimeImmutable());
    }

    // ...

    public function isNew()
    {
        return
            (new DateTimeImmutable())
                 ->diff($this->createdAt)
                 ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```

`isNew()`方法需要比较两个`DateTimes`；这是创建`Post`时的日期与今天的日期之间的比较。我们计算差异并检查它是否小于指定的天数。我们如何单元测试`isNew()`方法？正如我们在实现中展示的那样，在测试套件中重现特定的流程是困难的。我们有什么选择？

# 将所有日期作为参数传递

一个可能的选择是在需要时将所有日期作为参数传递：

```php
class Post
{
    // ...

    public function __construct($aContent, $title, $createdAt = null)
    {
        // ...
        $this->createdAt($createdAt ?: new DateTimeImmutable());
    }

    // ...

    public function isNew($today = null)
    {
        return
            ($today ? :new DateTimeImmutable())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```

这对于单元测试来说是最简单的方法。只需传递不同的日期对来测试所有可能的场景，实现 100%的覆盖率。然而，如果你考虑创建和请求`isNew()`方法结果的客户端代码，事情看起来就不那么好了。由于总是传递今天的`DateTime`，生成的代码可能有点奇怪：

```php
$aPost = new Post(
    'Hello world!',
    'Hi',
    new DateTimeImmutable()
);

$aPost->isNew(
    new DateTimeImmutable()
);

```

# 测试类

另一个替代方案是使用测试类模式。这个想法是扩展`Post`类，创建一个新的类，我们可以操作它来强制特定的场景。这个新类将仅用于单元测试目的。坏消息是我们必须稍微修改原始的`Post`类，提取一些方法，并将一些字段和方法从`private`改为`protected`。一些开发者可能担心仅仅因为测试原因而增加类属性的可见性。然而，我们认为在大多数情况下，这是值得的：

```php
class Post
{
    protected $createdAt;

    public function isNew()
    {
        return
            ($this->today())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }

    protected function today()
    {
        return new DateTimeImmutable();
    }

    protected function createdAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);

        $this->createdAt = $aDate;
    }
}

```

如您所见，我们已经将获取今天日期的逻辑提取到`today()`方法中。这样，通过应用模板方法模式，我们可以从派生类中改变其行为。类似的情况也发生在`createdAt`方法和字段上。现在它们是受保护的，因此可以在派生类中使用和重写：

```php
class PostTestClass extends Post
{
    private $today;

    protected function today()
    {
       return $this->today;
    }

    public function setToday($today)
    {
       $this->today = $today;
    }
}

```

通过这些更改，我们现在可以通过测试`PostTestClass`来测试我们的原始`Post`类：

```php
class PostTest extends PHPUnit_Framework_TestCase
{
    // ...

    /** @test */
    public function aPostIsNewIfIts15DaysOrLess()
    {
        $aPost = new PostTestClass(
            'A Post Content' ,
            'A Post Title'
        );

        $format = 'Y-m-d';
        $dateString = '2016-01-01';
        $createdAt = DateTimeImmutable::createFromFormat(
            $format,
            $dateString
        );

        $aPost->createdAt($createdAt);
        $aPost->setToday(
            $createdAt->add(
                new DateInterval('P15D')
            )
        );

        $this->assertTrue(
            $aPost->isNew()
        );

        $aPost->setToday(
            $createdAt->add(
               new DateInterval('P16D')
            )
        );

        $this->assertFalse(
            $aPost->isNew()
        );
    }
}

```

最后一个小细节：使用这种方法，我们无法在`Post`类上实现 100%的覆盖率，因为`today()`方法永远不会被执行。然而，它可以通过其他测试来覆盖。

# 外部模拟

另一个选项是使用新类和一些静态方法包装对`DateTimeImmutable`构造函数或命名构造函数的调用。这样做，我们可以静态地改变这些方法的结果，使其根据特定的测试场景表现出不同的行为：

```php
class Post
{
    // ...
    private $createdAt;

    public function __construct($aContent, $title)
    {
        // ...
        $this->createdAt(MyCustomDateTimeBuilder::today());
    }

    // ...

    public function isNew()
    {
        return
            (MyCustomDateTimeBuilder::today())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```

为了获取今天的`DateTime`，我们现在使用对`MyCustomDateTimeBuilder::today()`的静态调用。这个类还有一些设置方法，可以在后续调用中模拟返回结果：

```php
class PostTest extends PHPUnit_Framework_TestCase
{
    // ...

    /** @test */
    public function aPostIsNewIfIts15DaysOrLess()
    {
        $createdAt = DateTimeImmutable::createFromFormat(
            'Y-m-d',
            '2016-01-01'
        );

        MyCustomDateTimeBuilder::setReturnDates(
            [
                $createdAt,
                $createdAt->add(
                    new DateInterval('P15D')
                ),
                $createdAt->add(
                    new DateInterval('P16D')
                )
            ] 
        );

        $aPost = new Post(
            'A Post Content' ,
            'A Post Title'
        );

        $this->assertTrue(
            $aPost->isNew()
        );

        $this->assertFalse(
            $aPost->isNew()
        );
    } 
}

```

这种方法的主要问题是与对象静态耦合。根据您的使用情况，创建一个灵活的模拟对象也会变得很棘手。

# 反射

您还可以使用反射技术来构建一个新的`Post`类，并自定义日期。考虑使用[Mimic](https://github.com/keyvanakbary/mimic)，这是一个简单的用于对象原型设计、数据注入和数据展示的功能性库：

```php
namespace Domain;

use mimic as m;

class ComputerScientist {
    private $name;
    private $surname;

    public function __construct($name, $surname) 
    {
        $this->name = $name;
        $this->surname = $surname;
    }

    public function rocks() 
    {
        return $this->name . ' ' . $this->surname . ' rocks!';
    }
}

assert(m\prototype('Domain\ComputerScientist')
    instanceof Domain\ComputerScientist);

m\hydrate('Domain\ComputerScientist', [
    'name'   =>'John' ,
    'surname'=>'McCarthy'
])->rocks(); //John McCarthy rocks!

assert(m\expose(
    new Domain\ComputerScientist('Grace', 'Hopper')) ==
    [
        'name'    => 'Grace' ,
        'surname' => 'Hopper'
    ]
)

```

分享和讨论

与您的同事讨论如何正确地对具有固定`DateTimes`的实体进行单元测试，并提出额外的替代方案。

如果您想了解更多关于测试模式和方法的资料，请查看 Gerard Meszaros 所著的书籍《xUnit Test Patterns: Refactoring Test Code》。

# 验证

验证是我们领域模型中一个非常重要的过程。它不仅检查属性的正确性，还检查整个对象及其组成的正确性。为了保持模型的有效状态，需要不同级别的验证。仅仅因为一个对象由有效的属性组成（基于每个属性），并不意味着该对象（作为一个整体）是有效的。反之亦然：有效的对象不一定等于有效的组合。

# 属性验证

有些人将验证理解为服务验证给定对象状态的过程。在这种情况下，验证符合[设计-by-合同](http://en.wikipedia.org/wiki/Design_by_contract)方法，该方法包括前置条件、后置条件和不变性。保护单个属性的一种方式是使用第三章，*值对象*。为了使我们的设计更灵活，我们只关注断言必须满足的领域前置条件。在这里，我们将使用守卫作为验证前置条件的一种简单方式：

```php
class Username
{
    const MIN_LENGTH = 5;
    const MAX_LENGTH = 10;
    const FORMAT = '/^[a-zA-Z0-9_]+$/';

    private $username;

    public function __construct($username)
    {
        $this->setUsername($username);
    }

    private setUsername($username)
    {
        $this->assertNotEmpty($username);
        $this->assertNotTooShort($username);
        $this->assertNotTooLong($username);
        $this->assertValidFormat($username);
        $this->username = $username;
    }

    private function assertNotEmpty($username)
    {
        if (empty($username)) {
            throw new InvalidArgumentException('Empty username');
        }  
    }

    private function assertNotTooShort($username)
    {
        if (strlen($username) < self::MIN_LENGTH) {
            throw new InvalidArgumentException(sprintf(
                'Username must be %d characters or more',
                self::MIN_LENGTH
            ));
        }
    }

    private function assertNotTooLong($username)
    {
        if (strlen( $username) > self::MAX_LENGTH) {
            throw new InvalidArgumentException(sprintf(
                'Username must be %d characters or less',
                self::MAX_LENGTH
            ));
        }
    }

    private function assertValidFormat($username)
    {
        if (preg_match(self:: FORMAT, $username) !== 1) {
            throw new InvalidArgumentException(
                'Invalid username format'
            );
        }
    }
}

```

如上例所示，为了构建一个用户名值对象，必须满足四个前提条件。它：

+   不能为空

+   必须至少 5 个字符

+   必须少于 10 个字符

+   必须遵循字母数字字符或下划线的格式

如果所有前提条件都满足，属性将被设置，对象将被成功构建。否则，将抛出`InvalidArgumentException`异常，执行将被终止，并且客户端将显示错误。

一些开发者可能会认为这种验证是防御性编程。然而，我们并没有检查输入是否为字符串或 nulls 是否不被允许。我们无法避免人们错误地使用我们的代码，但我们可以控制我们的领域状态的正确性。如第三章中的*值对象*所示，验证还可以帮助我们提高安全性。

[防御性编程](https://en.wikipedia.org/wiki/Defensive_programming)并不是一件坏事。一般来说，当开发将要作为第三方在其他项目中使用的组件或库时，这样做是有意义的。然而，当开发自己的边界上下文时，那些额外的偏执检查（nulls、基本类型、类型提示等）可以通过依赖单元测试套件的覆盖率来避免，从而提高开发速度。

# 整个对象验证

有时候，一个由有效属性组成的对象整体仍然可能被认为是无效的。可能会诱使人们将这种验证添加到对象本身，但通常这并不是一个好的做法。高级别的验证变化节奏与对象逻辑本身不同。此外，将责任分离是良好的实践。

验证通知客户端任何已发现的错误，或者收集结果以供稍后审查，因为有时我们不想在出现问题的第一个迹象时就停止执行。

一个`抽象`且可重用的`Validator`可能如下所示：

```php
abstract class Validator
{
    private $validationHandler;

    public function __construct(ValidationHandler $validationHandler)
    {
        $this->validationHandler = $validationHandler;
    }

    protected function handleError($error)
    {
        $this->validationHandler->handleError($error);
    }

    abstract public function validate();
}

```

具体来说，我们想要验证一个由有效的国家、城市和邮编值对象组成的整个`Location`对象。然而，这些个别值在验证时可能处于无效状态。也许城市不是国家的一部分，或者邮编可能不符合城市格式：

```php
class Location
{
    private $country;
    private $city;
    private $postcode;

    public function __construct(
        Country $country, City $city, Postcode $postcode
    ) {
        $this->country = $country;
        $this->city = $city;
        $this->postcode = $postcode;
    }

    public function country()
    {
        return $this->country;
    }

    public function city()
    {
        return $this->city;
    }

    public function postcode()
    {
        return $this->postcode;
    }
}

```

验证器检查`Location`对象的整体状态，分析属性之间关系的意义：

```php
class LocationValidator extends Validator
{
    private $location;

    public function __construct(
        Location $location, ValidationHandler $validationHandler
    ) {
        parent:: __construct($validationHandler);
        $this->location = $location;
    }

    public function validate()
    {
        if (!$this->location->country()->hasCity(
            $this->location->city()
        )) {
            $this->handleError('City not found');
        }

        if (!$this->location->city()->isPostcodeValid(
            $this->location->postcode()
        )) {
            $this->handleError('Invalid postcode');
        }
    }
}

```

一旦所有属性都已设置，我们就能验证实体，这通常是在某个描述的过程之后。表面上看起来，位置似乎是自我验证的。然而，事实并非如此。`Location`类将这种验证委托给一个具体的验证器实例，将这两个清晰的责任分开：

```php
class Location
{
    // ...

    public function validate(ValidationHandler $validationHandler)
    {
     $validator = new LocationValidator($this, $validationHandler);
     $validator->validate();
    }
}

```

# 解耦验证消息

通过对我们现有实现的一些小改动，我们能够将验证消息与验证器解耦：

```php
class LocationValidationHandler implements ValidationHandler
{
    public function handleCityNotFoundInCountry();

    public function handleInvalidPostcodeForCity();
}

class LocationValidator
{
    private $location;
    private $validationHandler;

    public function __construct(
        Location $location,
        LocationValidationHandler $validationHandler
    ) {
        $this->location = $location;
        $this->validationHandler = $validationHandler;
    }

    public function validate()
    {
        if (!$this->location->country()->hasCity(
            $this->location->city()
        )) {
            $this->validationHandler->handleCityNotFoundInCountry();
        } 

        if (! $this->location->city()->isPostcodeValid(
            $this->location->postcode()
        )) {
            $this->validationHandler->handleInvalidPostcodeForCity();
        }
    }
}

```

我们还需要将验证方法的签名更改为以下形式：

```php
class Location
{
   // ...

    public function validate(
        LocationValidationHandler $validationHandler
    ) {
        $validator = new LocationValidator($this, $validationHandler);
        $validator->validate();
    }
}

```

# 验证对象组合

验证对象组合可能很复杂。因此，实现这一目标的最佳方式是通过领域服务。然后，该服务与存储库通信，以检索有效的聚合。由于可能创建的复杂对象图，聚合可能处于中间状态，需要先验证其他聚合。我们可以使用领域事件来通知系统的其他部分，特定元素已被验证。

# 实体和领域事件

我们将在未来的章节中探讨第六章，*领域事件*；然而，重要的是要强调，对实体执行的操作可以触发领域事件。这种方法用于将领域变化传达给应用程序的其他部分，或者甚至传达给其他应用程序，正如你将在第十二章，*集成边界上下文*中看到的：

```php
class Post
{
   // ...

    public function publish()
    {
        $this->setStatus(
            Status::published()
        );

        $this->publishedAt(new DateTimeImmutable());

        DomainEventPublisher::instance()->publish(
            new PostPublished($this->id)
        );
    }

    public function unpublish()
    {
        $this->setStatus(
            Status::draft()
        );

        $this-> publishedAt = null;

        DomainEventPublisher::instance()->publish(
            new PostUnpublished($this->id)
        );
    }

    // ...
}

```

当我们的实体创建新实例时，甚至可以触发领域事件：

```php
class User
{
    // ...

    public function __construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);

        DomainEventPublisher::instance()->publish(
            new UserRegistered($this->userId)
        );
    }
}

```

# 总结

领域中的一些概念需要身份——也就是说，它们内部状态的变化不会改变它们自己的唯一身份。我们已经看到如何将身份建模为值对象，除了操作身份本身的逻辑外，还能带来不可变等好处。我们还展示了提供身份的几种方法，以下是一些要点：

+   持久化机制：易于实现，但在持久化对象之前，你将无法获得身份，这会延迟并复杂化事件传播。

+   代理 ID：一些 ORM 要求在实体上添加额外字段，以将身份与持久化机制映射。

+   由客户端提供：有时身份符合领域概念，你可以在领域内部对其进行建模。

+   由应用程序生成：你可以使用库来生成 ID。

+   由边界上下文生成：这可能是最复杂的策略。其他边界上下文提供了一个生成身份的接口。

我们已经看到并讨论了 Doctrine 作为持久化机制，我们研究了使用 Active Record 模式的缺点，最后，我们检查了实体验证的不同级别：

+   属性验证：通过前置条件、后置条件和不变性检查对象状态中的具体内容。

+   整个对象验证：寻找对象作为一个整体的一致性。将验证提取到外部服务是一种良好的实践。

+   对象组合：可以通过领域服务验证复杂对象组合。将此传达给应用程序其余部分的好方法是使用领域事件。
