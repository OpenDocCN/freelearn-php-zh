# 领域事件

软件事件是系统中发生的事情，其他组件可能想知道。PHP 开发者通常不习惯于使用事件，因为事件不是语言中的特性。然而，更常见的是看到新的框架和库如何接受它们，以提供新的解耦、重用和加速代码的方法。

领域事件是与领域变化相关的事件。领域事件是我们领域中发生的事情，领域专家关心的事情。

在领域驱动设计中，领域事件是基本构建块，有助于：

+   与其他边界上下文进行通信。

+   提高性能和可伸缩性，推动最终一致性。

+   作为历史检查点。

领域事件代表了异步通信的精髓。关于这个话题的更多信息，我们推荐阅读 Gregor Hohpe 和 Bobby Woolf 所著的书籍 *企业集成模式：设计、构建和部署消息解决方案*（[Designing, Building, and Deploying Messaging Solutions](http://www.amazon.com/Enterprise-Integration-Patterns-Designing-Deploying/dp/0321200683)）。

# 简介

想象一下一款 JavaScript 2D 平台游戏。屏幕上有成百上千个不同的组件在相互交互，所有这些都在同一时间进行。有一个组件显示剩余生命值，另一个显示所有得分，还有一个倒计时当前关卡剩余时间。每次你的角色跳到敌人身上，得分就会增加。当你的得分超过一定数量时，你会获得额外生命。当角色捡起钥匙时，通常门就会打开。但是，所有这些组件是如何相互交互的呢？这种场景的最佳架构是什么？

可能有两个主要选项：第一个是将每个组件与其连接的组件耦合在一起。然而，在上面的例子中，这意味着许多组件被耦合在一起，每次添加新组件都需要开发者修改代码。但你还记得**开闭原则**（**OCP**）吗？添加新组件不应该让第一个组件需要更新；这将使维护工作变得过于繁重。

第二个——也是更好的——方法是连接所有组件到一个单一的对象，该对象处理游戏中所有重要的事件。它从每个组件接收事件并将它们转发到特定的组件。例如，得分组件会对`EnemyKilled`事件感兴趣，而`LifeCaptured`事件对玩家实体和剩余生命值组件非常有用。这样，所有组件都耦合到一个管理所有通知的单个组件上。使用这种方法，添加或删除组件不会影响现有的组件。

当开发单个应用程序时，事件对于解耦组件很有帮助。当以分布式方式开发整个领域时，事件对于解耦在领域中扮演角色的每个服务或应用程序非常有用。关键点是相同的，但规模不同。

# 定义

领域事件是用于通知本地或远程边界上下文领域变化的一种特定类型的事件。

Vaughn Vernon [定义](http://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon-ebook/dp/B00BCLEBN8)领域事件为：

领域内发生的事情的一个发生实例。

Eric Evans [定义](https://domainlanguage.com/ddd/patterns/DDD_Reference_2011-01-31.pdf)领域事件为：

领域模型的一个完整部分，表示领域内发生的事情。在制作显式领域专家希望跟踪或通知的事件，或与其它模型对象状态变化相关联的事件时，忽略无关的领域活动。

Martin Fowler [定义](http://martinfowler.com/eaaDev/DomainEvent.html)领域事件为某种东西：

记录了影响领域内有趣事情的记忆。

在 Web 应用程序中，领域事件的例子有`UserRegistered`（用户注册）、`OrderPlaced`（订单已放置）、`UserRelocated`（用户迁移）和`ProductAdded`（产品添加）。

# 短篇小说

在票务销售代理机构中，内容经理决定提高 U2 演出的票价。她使用后台编辑演出。一个`ShowPriceChanged`（演出价格已更改）领域事件被发布并持久化到数据库中，与新的演出价格在同一笔交易中。

批处理过程将领域事件排队到 RabbitMQ 中。领域事件在两个队列中分发：一个用于相同的本地边界上下文，另一个用于远程的商务智能目的。

在第一个队列中，一个工作者使用事件中的 ID 检索相应的演出，并将其推送到 Elasticsearch 服务器，以便用户在搜索时可以看到新的价格。它也可以在另一个数据库表中更新新的价格。

在第二个队列中，一个工作者将信息插入到日志服务器或数据湖中，在那里可以运行报告或数据挖掘过程。

一个无法使用领域事件集成的外部应用程序可以使用本地边界上下文提供的 REST API 访问所有`ShowPriceChanged`（演出价格已更改）事件。

如您所见，领域事件对于处理最终一致性和集成不同的边界上下文非常有用。聚合创建事件并发布它们。订阅者可以存储事件，然后将它们转发给远程订阅者。

# 隐喻

我们在周二去巴布尔餐厅用餐，并使用信用卡支付。这可以建模为一个事件，事件类型为`PurchasePlaced`（购买已放置），主题是我的信用卡，发生日期为周二。如果巴布尔的系统过时且直到周五才传输交易，那么交易将在周五生效。

事情发生了。并非所有事情都很有趣，有些可能值得记录但不会引起反应。然而，最有趣的事情会引起反应。许多系统需要对有趣的事件做出反应。通常你需要知道为什么系统以这种方式做出反应。

通过将系统输入引导到领域事件流中，您可以记录系统所有输入。这有助于您组织您的处理逻辑，同时也允许您保留系统输入的审计日志。

练习

尝试在你的当前领域定位潜在的领域事件示例。

# 真实世界示例

在详细讨论领域事件之前，让我们看看一个领域事件的实际例子以及它们如何帮助我们在我们的应用程序和整个领域中。

让我们考虑一个简单的应用程序服务，它将注册新用户——例如，在电子商务环境中。应用程序服务将在另一章中解释，所以不必过于担心接口。相反，只需关注执行方法：

```php
class SignUpUserService implements ApplicationService
{
    private $userRepository;
    private $userFactory;
    private $userTransformer;

    public function __construct(
        UserRepository $userRepository,
        UserFactory $userFactory,
        UserTransformer $userTransformer
    ) {
        $this->userRepository = $userRepository;
        $this->userFactory = $userFactory;
        $this->userTransformer = $userTransformer;
    }

    /**
     * @param SignUpUserRequest $request
     * @return User
     * @throws UserAlreadyExistsException
     */
    public function execute(SignUpUserRequest $request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->userOfEmail($email );
        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $user = $this->userFactory->build(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );

        $this->userRepository->add($user);
        $this->userTransformer->write($user);
    }
}

```

如所示，*应用程序服务*部分检查用户是否已存在。如果没有，它将创建一个新的用户并将其添加到`UserRepository`。

现在考虑一个额外的要求：当新用户注册时，必须通过电子邮件通知他们。没有过多思考，首先想到的方法是更新我们的应用程序服务，包括一段执行这项工作的代码——可能是一种`EmailSender`，它会在添加方法之后运行。然而，让我们考虑另一种方法。

关于触发一个`UserRegistered`事件，以便另一个监听组件可以做出反应并发送电子邮件，有什么好处？这种新方法有一些酷炫的好处。首先，当新用户注册时，我们不需要每次都更新应用程序服务的代码来执行新操作。

其次，它更容易测试。应用程序服务保持简单，每次开发新操作时，我们只需为该操作编写测试。

在同一个电子商务项目中稍后，我们被告知要集成一个非 PHP 编写的开源游戏化平台。每次用户在我们的电子商务边界上下文中进行购买或评论产品时，他们都可以获得可以在他们的电子商务用户个人资料页面上显示的徽章，或者通过电子邮件通知。我们该如何建模这个问题？

按照第一种方法，我们会更新应用程序服务以类似于之前电子邮件确认方法的方式与新的平台集成。使用领域事件方法，我们可以为`UserRegistered`事件创建另一个监听器，该监听器将通过 REST 或 SOA 直接连接到游戏化平台。甚至更好，它可以将事件传递给像 RabbitMQ 这样的消息系统，以便游戏化边界上下文可以订阅并自动接收通知。我们的电子商务边界上下文根本不需要了解游戏化边界上下文。

# 特征

领域事件通常是**不可变的**，因为它们是过去某事的记录。除了事件的描述外，领域事件通常还包含事件发生的时间戳以及参与事件的实体标识。此外，领域事件通常还有一个单独的时间戳，表示事件被输入系统的时间，以及输入该事件的人的身份。当有用时，领域事件的标识可以基于这些属性中的一组。例如，如果同一事件的两个实例到达一个节点，它们可以被识别为相同的。

领域事件的本质是，你用它来捕获可以触发你正在开发的应用程序或领域内其他感兴趣的应用程序状态变化的事物。然后处理这些事件对象以引起系统变化，并存储起来以提供审计日志。

# 命名规范

所有事件都应该用过去时态的动词来表示，因为它们是过去已经完成的事情——例如，`CustomerRelocated`、`CargoShipped`或`InventoryLossageRecorde`d。在英语中，有一些有趣的例子，人们可能会倾向于使用名词而不是过去时态的动词；例如，对于对自然灾害感兴趣的国会议员来说，*Earthquake*或*Capsize*可以作为相关事件；我们建议避免使用这样的名称作为领域事件，并坚持使用过去时态的动词。

# 领域事件和通用语言

当我们讨论客户搬迁的副作用时，要考虑通用语言的差异。事件使概念明确，而之前，在聚合内部或多个聚合之间发生的变更被留作一个需要探索和定义的隐含概念。例如，在大多数系统中，当一个库如 Hibernate 或 Entity Framework 发生副作用时，它不会影响领域。这些事件对客户端来说是隐含且透明的。引入事件使概念明确，并成为通用语言的一部分。搬迁客户不仅仅是改变一些东西；相反，它产生了一个在语言中明确定义的`CustomerRelocatedEvent`。

# 不可变性

正如我们之前提到的，领域事件谈论的是过去，描述了已经发生的领域变化。根据定义，除非你是 Marty McFly 并且有一个德洛瑞安，否则不可能改变过去，这可能是不可能的。所以，请记住，领域事件是不可变的。

Symfony 事件分发器

一些 PHP 框架支持事件。然而，不要将这些事件与领域事件混淆；它们在特性和目标上不同。例如，Symfony 有事件分发器组件，如果你需要为状态机实现事件系统，你可以依赖它。在 Symfony 中，请求到响应的转换也由事件处理。然而，Symfony 事件是可变的，每个监听器都有能力修改、添加或更新事件中的信息。

# 建模事件

为了准确描述你的业务领域，你必须与领域专家紧密合作，并定义通用的语言。这需要使用领域事件、实体、值对象等来构建领域概念。在建模事件时，根据它们起源的边界上下文中的通用语言来命名它们及其属性。如果一个事件是执行聚合体上的命令操作的结果，那么其名称通常是从所执行的命令中派生出来的。事件名称反映事件发生的过去性质是很重要的。

让我们考虑我们的用户注册功能；领域事件需要表示它。以下代码显示了一个基础领域事件的简化接口：

```php
interface DomainEvent 
{ 
    /** 
     * @return DateTimeImmutable
     */
     public function occurredOn(); 
}

```

如你所见，所需的最小信息是一个`DateTimeImmutable`，这是为了知道事件发生的时间。

现在让我们使用以下代码来建模新的用户注册事件。如我们上面提到的，名称应该是过去时态的动词，所以`UserRegistered`可能是一个不错的选择：

```php
class UserRegistered implements DomainEvent 
{ 
    private $userId;

    public function __construct(UserId $userId)
    {
        $this->userId = $userId;
        $this->occurredOn = new \DateTimeImmutable();
    }

    public function userId()
    {
        return $this->userId;
    }

    public function occurredOn()
    {
        return $this->occurredOn;
    }
}

```

通知订阅者关于新用户创建所需的最小信息量是`UserId`。有了这些信息，任何过程、命令或应用程序服务——无论是同一个边界上下文还是不同的边界上下文——都可以对此事件做出反应。

作为经验法则

+   领域事件通常设计为不可变的

+   构造函数将初始化领域事件的完整状态。

+   领域事件将提供 getter 来访问它们的属性

+   包含执行动作的聚合体身份

+   包含与第一个相关的其他聚合体身份

+   包含导致事件的参数（如果有用）

但如果你的领域专家来自同一个边界上下文或不同的边界上下文需要更多信息怎么办？让我们看看用更多信息建模的相同领域事件——例如，电子邮件地址：

```php
class UserRegistered implements DomainEvent
{
    private $userId;
    private $userEmail ;

    public function __construct(UserId $userId, $userEmail)
    {
        $this-> userId = $userId;
        $this->userEmail = $userEmail ;
        $this->occurredOn = new DateTimeImmutable();
    }

    public function userId()
    {
        return $this->userId;
    }

    public function userEmail ()
    {
        return $this->userEmail ;
    }

    public function occurredOn()
    {
        return $this->occurredOn;
    }
}

```

如上所示，我们添加了电子邮件地址。向领域事件添加更多信息可以帮助提高性能或简化不同边界上下文之间的集成。从另一个边界上下文的角度思考可以帮助建模事件。当上游边界上下文中创建新用户时，下游的一个将不得不创建自己的用户。添加用户电子邮件地址可能在下游需要它的情况下，可能节省对上游边界上下文的同步请求。

你还记得那个游戏化示例吗？为了创建一个游戏化平台上的用户，可能被称为玩家，来自电子商务边界上下文的 UserId 可能已经足够。但如果游戏化平台需要通过电子邮件通知用户奖励信息呢？在这种情况下，电子邮件地址也是必需的。所以如果电子邮件地址包含在原始域事件中，我们就完成了。如果不是这样，游戏化边界上下文需要通过 REST 或 SOA 集成从电子商务边界上下文请求这些信息。

为什么不是整个用户实体？

想知道你是否应该在域事件中包含你边界上下文的整个用户实体吗？我们的建议是不要这样做。域事件可能用于在给定的边界上下文中内部通信消息或外部与其他边界上下文通信。换句话说，在一个 C2C 电子商务产品目录边界上下文中可能是一个“卖家”的，在产品反馈中可能是一个产品的“作者”。两者可以共享相同的 ID 或电子邮件，但“卖家”和“作者”是不同的概念，代表来自不同边界上下文的不同实体。因此，来自一个边界上下文的实体在另一个边界上下文中可能没有意义或具有完全不同的意义。

# Doctrine 事件

域事件不仅用于执行批量作业，如发送电子邮件或与其他边界上下文通信；它们对于性能和可扩展性的改进也非常有趣。让我们看看一个例子。

考虑以下场景。你有一个电子商务应用程序。你的主要持久化机制是 MySQL，但为了浏览和过滤你的目录，你使用了一个更好的方法，比如 Elasticsearch 或 Solr。在 Elasticsearch 上，你最终会得到存储在你完整数据库中的信息的一个子集。你如何保持数据同步？当内容团队从后台工具更新目录时会发生什么？

有些人时不时地重新索引整个目录。这非常昂贵且缓慢。一个更聪明的做法可能是更新已更新的产品相关的一个或多个文档。我们如何做到这一点？使用域事件。

然而，如果你一直在使用 Doctrine，这很可能不是什么新鲜事。根据[Doctrine 2 ORM 2 文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html#events)：

Doctrine 2 拥有一个轻量级的事件系统，它是 Common 包的一部分。Doctrine 使用它来分发系统事件，主要是生命周期事件。你也可以用它来处理你自己的自定义事件。

此外，它[声明](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html#lifecycle-callbacks)如下：

生命周期回调是在实体类上定义的。它们允许你在该实体类的实例经历相关生命周期事件时触发回调。每个生命周期事件都可以定义多个回调。生命周期回调最适合用于特定实体类生命周期的简单操作。

让我们看看[Doctrine 事件文档](http://doctrine-orm.readthedocs.io/projects/doctrine-orm/en/latest/reference/events.html)中的一个例子：

```php
/** @Entity @HasLifecycleCallbacks */
class User
{
    // ...

   /**
    * @Column(type="string", length=255)
    */
    public $value;

    /** @Column(name="created_at", type="string", length=255) */
    private $createdAt;

    /** @PrePersist */
    public function doStuffOnPrePersist()
    {
        $this->createdAt = date('Y-m-d H:i:s');
    }

    /** @PrePersist */
    public function doOtherStuffOnPrePersist()
    {
        $this-> value = 'changed from prePersist callback!';
    }

    /** @PostPersist */
    public function doStuffOnPostPersist()
    {
        $this->value = 'changed from postPersist callback!';
    }

    /** @PostLoad */
    public function doStuffOnPostLoad()
    {
        $this->value = 'changed from postLoad callback!';
    }

    /** @PreUpdate */
    public function doStuffOnPreUpdate()
    {
        $this->value = 'changed from preUpdate callback!';
    }
}

```

你可以在 Doctrine 实体生命周期中的每个不同的重要时刻挂钩特定的任务。例如，在`PostPersist`时，你可以生成你的实体 JSON 文档并将其放入 Elasticsearch。这样，就很容易在不同持久化机制之间保持数据的一致性。

Doctrine 事件是围绕实体周围事件带来好处的良好例子。但你可能想知道它们的问题是什么。这是因为它们与框架耦合，是同步的，并且作用于应用层面，但不是用于通信目的。所以这就是为什么尽管领域事件在实现和处理上可能更困难，但它们仍然非常有意思。

# 持久化领域事件

持久化事件总是一个好主意。有些人可能想知道为什么你不应该直接将领域事件发布到消息或日志系统中。这是因为持久化它们有一些有趣的好处：

+   你可以通过 REST 接口将你的领域事件暴露给其他边界上下文。

+   你可以在将事件和聚合更改推送到 RabbitMQ 之前，在同一个数据库事务中持久化领域事件和聚合更改。（你不想发送关于没有发生的事情的通知，就像你不想错过关于已经发生的事情的通知一样。）

+   商业智能可以使用这些数据进行分析、预测或趋势分析。

+   你可以审计实体更改。

+   对于事件源，你可以从领域事件中重新构成聚合。

# 事件存储

我们在哪里持久化领域事件？在事件存储中。事件存储是一个存在于我们的领域空间中的领域事件仓库，作为一个抽象（接口或抽象类）。其责任是追加领域事件并查询它们。一个可能的基本接口可能是以下内容：

```php
interface EventStore
{
    public function append(DomainEvent $aDomainEvent);
    public function allStoredEventsSince($anEventId);
}

```

然而，根据你的领域事件的使用情况，之前的接口可能有更多方法来查询你的事件。

在实现方面，你可以选择使用 Doctrine 仓库、DBAL 仓库或纯 PDO。由于领域事件是不可变的，使用 Doctrine 仓库会增加不必要的性能惩罚，但对于小型到中型应用，Doctrine 可能还是可以的。让我们看看使用 Doctrine 的一个可能实现：

```php
class DoctrineEventStore extends EntityRepository implements EventStore
{
    private $serializer;

    public function append(DomainEvent $aDomainEvent)
    {
        $storedEvent = new StoredEvent(
            get_class($aDomainEvent),
            $aDomainEvent->occurredOn(),
            $this->serializer()->serialize($aDomainEvent, 'json')
        );

        $this->getEntityManager()->persist($storedEvent);
     }

     public function allStoredEventsSince($anEventId)
     {
         $query = $this->createQueryBuilder('e');
         if ($anEventId) {
             $query->where('e.eventId > :eventId');
             $query->setParameters(['eventId' => $anEventId]);
         }
         $query->orderBy('e.eventId');

         return $query->getQuery()->getResult();
     }

     private function serializer()
     {
         if (null === $this->serializer) {
             /** \JMS\Serializer\Serializer\SerializerBuilder */
             $this->serializer = SerializerBuilder::create()->build();
         }

         return $this->serializer;
     }
 }

```

`StoredEvent`是映射到数据库的 Doctrine 实体。正如你可能看到的，在追加并持久化`Store`之后，没有`flush`调用。如果这个操作在 Doctrine 事务中，则不需要。所以，让我们不调用它，我们将在讨论应用程序服务时详细介绍。

现在让我们看看`StoredEvent`的实现：

```php
class StoredEvent implements DomainEvent
{
    private $eventId;
    private $eventBody;
    private $occurredOn;
    private $typeName;

    /**
     * @param string $aTypeName
     * @param \DateTimeImmutable $anOccurredOn
     * @param string $anEventBody
     */
    public function __construct(
        $aTypeName, \DateTimeImmutable $anOccurredOn, $anEventBody
    ) {
        $this->eventBody = $anEventBody;
        $this->typeName = $aTypeName;
        $this->occurredOn = $anOccurredOn;
    }

    public function eventBody()
    {
        return $this->eventBody;
    }

    public function eventId()
    {
        return $this->eventId;
    }

    public function typeName()
    {
        return $this->typeName;
    }

    public function occurredOn()
    {
        return $this->occurredOn;
    }
}

```

这里是其映射：

```php
Ddd\Domain\Event\StoredEvent:
    type: entity
    table: event
    repositoryClass:
        Ddd\Infrastructure\Application\Notification\DoctrineEventStore
    id:
        eventId:
            type: integer
            column: event_id
            generator:
            strategy: AUTO
    fields:
        eventBody:
            column: event_body
            type: text
        typeName:
            column: type_name
            type: string
            length: 255
        occurredOn:
            column: occurred_on
            type: datetime

```

为了持久化具有不同字段的领域事件，我们必须将这些字段作为序列化字符串连接起来。`typeName`标识全局领域事件。实体或值对象在边界上下文中是有意义的，但领域事件定义了边界上下文之间的通信协议。

在分布式系统中，总会发生一些事情。你将不得不处理那些未发布、在链中丢失或发布多次的领域事件。这就是为什么将领域事件持久化并带有 ID 很重要，这样就可以轻松跟踪哪些领域事件已被发布，哪些缺失。

# 从领域模型发布事件

当表示的事实发生时，应该发布领域事件。例如，当新用户注册时，应该发布一个新的`UserRegistered`事件。

按照报纸的比喻：

+   **建模**领域事件就像撰写新闻文章

+   **发布**领域事件就像在报纸上打印文章

+   **传播**领域事件就像分发报纸，让每个人都能阅读文章

发布领域事件的建议方法是使用简单的监听器-观察者模式来实现`DomainEventPublisher`。

# 从实体发布领域事件

继续以一个新用户在我们的应用程序中注册的例子，让我们看看相应的领域事件是如何发布的：

```php
class User
{
    protected $userId;
    protected $email ;
    protected $password;

    public function __construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);

        DomainEventPublisher::instance()->publish(
            new UserRegistered($this->userId)
        );
    }

    // ...
}

```

如示例所示，当创建`User`时，会发布一个新的`UserRegistered`事件。这是在实体构造函数中完成的，而不是在外部，因为采用这种方法，更容易保持我们的领域一致性；任何创建新`User`的客户端都会发布其对应的事件。另一方面，这也使得使用需要创建`User`实体而不使用其构造函数的基础设施变得更加复杂。例如，Doctrine 使用`serialize`和`unserialize`技术来重新创建一个对象，而不调用其构造函数。然而，如果你必须自己创建，这不会像在 Doctrine 中那样简单。

通常，从平面数据（如数组）构建对象称为**解冻**。让我们看看从数据库中获取新`User`的简单方法。首先，让我们通过应用[工厂方法模式](http://en.wikipedia.org/wiki/Template_method_pattern)将领域事件发布提取到自己的方法中。

根据[维基百科](https://en.wikipedia.org/wiki/Template_method_pattern)：

模板方法模式是一种行为设计模式，它在一个操作中定义了算法的程序骨架，将一些步骤推迟到子类中：

```php
class User
{
    protected $userId;
    protected $email ;
    protected $password;

    public function __construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);
        $this->publishEvent();

    }

    protected function publishEvent()
    {
        DomainEventPublisher::instance()->publish(
            new UserRegistered($this->userId)
        );
    }

    // ...
}

```

现在，让我们通过添加一个新的基础设施实体来扩展我们当前的`User`，这个实体将为我们完成工作。这里的技巧是让`publishEvent`不执行任何操作，这样域事件就不会被发布：

```php
class CustomOrmUser extends User
{
    protected function publishEvent()
    {

    }

    public static function fromRawData($data)
    {
        return new self(
            new UserId($data['user_id']),
            $data['email'],
            $data['password']
        );
    }
}

```

记住要小心使用这种方法；你可能会从持久化机制中获取无效的对象，因为域规则总是在变化。不使用父构造函数的另一种方法可能是以下：

```php
class CustomOrmUser extends User
{
    public function __construct()
    {
    }

    public static function fromRawData($data)
    {
        $user = new self();
        $user->userId = new UserId($data['user_id']);
        $user->email = $data['email'];
        $user->password = $data['password'];

        return $user;
    }
}

```

使用这种方法，不会调用父构造函数，并且用户属性必须是受保护的。其他替代方案包括反射、在构造函数中传递标志、使用像[Proxy-Manager](https://packagist.org/packages/ocramius/proxy-manager)这样的代理库，或者使用像 Doctrine 这样的 ORM。

发布域事件的另一种策略

如前一个示例所示，我们正在使用一个静态类来发布我们的域事件。其他人，作为一个替代方案，尤其是在使用[事件溯源](http://martinfowler.com/eaaDev/EventSourcing.html)时，会建议实体在字段内部保留所有已触发的事件。为了访问所有事件，聚合体中使用了一个 getter。这也是一个有效的方法。然而，有时很难跟踪哪些实体触发了事件。从不是实体的地方触发事件也可能很困难，例如：域服务。优点是，检查实体是否触发了事件要容易得多。

# 从域或应用服务发布你的域事件

你应该努力从链的更深层次发布域事件。越接近实体或值对象的内部，越好。正如我们在前一个部分中看到的，有时这并不容易，但最终结果对客户端来说更简单。我们已经看到开发者从应用服务或域服务发布域事件。这看起来更容易做，但最终会导致贫血域模型。这就像在域服务中推入业务逻辑而不是将其放入你的实体中一样。

# 域事件发布者是如何工作的

域事件发布者是一个单例类，可以从我们的边界上下文中获取，用于发布域事件。它还支持附加监听器——域事件订阅者——它们将监听任何它们感兴趣的事件。这与使用 jQuery 的 on 方法订阅事件没有太大区别：

```php
class DomainEventPublisher
{
    private $subscribers;
    private static $instance = null;

    public static function instance()
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }

        return static::$instance;
    }

    private function __construct()
    {
        $this->subscribers = [];
    }

    public function __clone()
    {
        throw new BadMethodCallException('Clone is not supported');
    }

    public function subscribe(
        DomainEventSubscriber $aDomainEventSubscriber
    ) {
        $this->subscribers[] = $aDomainEventSubscriber;
    }

    public function publish(DomainEvent $anEvent)
    {
        foreach ($this->subscribers as $aSubscriber) {
           if ($aSubscriber->isSubscribedTo($anEvent)) {
               $aSubscriber->handle($anEvent);
           }
        }
    }
}

```

`publish`方法遍历所有可能的订阅者，检查它们是否对发布的事件感兴趣。如果是这样，就会调用订阅者的`handle`方法。

`subscribe`方法添加了一个新的`DomainEventSubscriber`，它将监听特定的域事件类型：

```php
interface DomainEventSubscriber
{
    /**
     * @param DomainEvent $aDomainEvent
     */
    public function handle($aDomainEvent);

    /**
     * @param DomainEvent $aDomainEvent
     * @return bool
     */
    public function isSubscribedTo($aDomainEvent);
}

```

正如我们已经讨论过的，持久化所有领域事件是一个很好的主意。我们可以通过使用特定的订阅者轻松地将我们应用中发布的所有领域事件持久化。让我们创建一个`DomainEventSubscriber`，它将监听所有领域事件，无论其类型如何，并使用我们的`EventStore`来持久化它们：

```php
class PersistDomainEventSubscriber implements DomainEventSubscriber
{
    private $eventStore;

    public function __construct(EventStore $anEventStore)
    {
        $this->eventStore = $anEventStore;
    }

    public function handle($aDomainEvent)
    {
        $this->eventStore->append($aDomainEvent);
    }

    public function isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}

```

`$eventStore`可以是自定义的 Doctrine 存储库，如之前所见，或任何其他能够将`DomainEvents`持久化到数据库的对象。

# 设置领域事件监听器

在哪里设置`DomainEventPublisher`的订阅者最好？这取决于。对于可能会影响整个请求周期的全局订阅者，最好的地方可能是`DomainEventPublisher`的初始化本身。对于受特定应用程序服务影响的订阅者，服务实例化可能是一个更好的地方。让我们通过 Silex 看看一个例子。

在[Silex](http://silex.sensiolabs.org/)中，通过使用应用程序中间件注册将所有领域事件持久化的领域事件发布者是最简单的方法。根据[Silex 2.0 文档](http://silex.sensiolabs.org/doc/master/middlewares.html)：

一个*前置*应用程序中间件允许你在控制器执行之前调整请求。

这是订阅负责将稍后发送到 RabbitMQ 的 Event 事件持久化到数据库的监听器的正确位置：

```php
// ...
$app['em'] = $app-> share(function () {
    return (new EntityManagerFactory())->build();
});

$app['event_repository'] = $app->share(function ($app) {
    return $app['em']->getRepository(
        'Ddd\Domain\Model\Event\StoredEvent'
    );
});

$app['event_publisher'] = $app->share(function($app) {
    return DomainEventPublisher::instance();
});

$app->before(
    function(Symfony\Component\HttpFoundation\Request $request)
        use($app) {

        $app['event_publisher']->subscribe(
            new PersistDomainEventSubscriber(
                $app['event_repository']
            )
        );
    }
);

```

使用此设置，每当聚合发布领域事件时，它将被持久化到数据库中。任务完成。

练习

如果你正在使用 Symfony、Laravel 或其他 PHP 框架，找到一种方法来全局订阅特定的订阅者，以执行围绕你的领域事件的任务。

如果你想在请求即将结束时对所有领域事件执行任何操作，你可以创建一个监听器，该监听器将所有已发布的领域事件存储在内存中。如果你给这个监听器添加一个 getter 来返回所有领域事件，然后你可以决定要做什么。如果不想或不能在之前提到的同一事务中持久化事件，这可能会很有用。

# 测试领域事件

你已经知道如何发布领域事件，但如何进行单元测试以确保`UserRegistered`事件确实被触发？我们建议的最简单方法是使用一个特定的`EventListener`，它将作为一个[间谍](http://www.martinfowler.com/bliki/TestDouble.html)来记录领域事件是否被发布。让我们看看`User`实体单元测试的一个例子：

```php
use Ddd\Domain\DomainEventPublisher;
use Ddd\Domain\DomainEventSubscriber;

class UserTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldPublishUserRegisteredEvent()
    {
        $subscriber = new SpySubscriber();
        $id = DomainEventPublisher::instance()->subscribe($subscriber);

        $userId = new UserId();
        new User($userId, 'valid@email.com', 'password');
        DomainEventPublisher::instance()->unsubscribe($id);

        $this->assertUserRegisteredEventPublished($subscriber,$userId);
    }

    private function assertUserRegisteredEventPublished(
        $subscriber, $userId
    ) {
        $this->assertInstanceOf(
            'UserRegistered', $subscriber->domainEvent
        );
        $this->assertTrue(
            $subscriber->domainEvent->serId()->equals($userId)
        );
    }
}

class SpySubscriber implements DomainEventSubscriber
{
    public $domainEvent;

    public function handle($aDomainEvent)
    {
        $this->domainEvent = $aDomainEvent;
    }

    public function isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}

```

上述方法有一些替代方案。你可以为`DomainEventPublisher`使用静态 setter 或使用一些反射框架来检测调用。然而，我们认为我们分享的方法更自然。最后但并非最不重要的是，记得清理间谍订阅，以免影响单元测试的其余部分的执行。

# 将消息传播到远程边界上下文

为了将一组领域事件传达给本地或远程的边界上下文，有两种主要策略：消息传递和 REST API。第一种计划使用消息系统（如 RabbitMQ）来传输领域事件。第二种计划为访问特定边界上下文的领域事件创建 REST API。

# 消息传递

将所有领域事件持久化到数据库后，剩下的唯一事情就是将它们推送到我们喜欢的消息系统。我们更喜欢[RabbitMQ](https://www.rabbitmq.com)，但任何其他系统，如 ActiveMQ 或 ZeroMQ，也能完成这项工作。对于使用 PHP 与 RabbitMQ 集成，选项并不多，但*[`php-amqplib`](https://packagist.org/packages/php-amqplib/php-amqplib)*将完成这项工作。

首先，我们需要一个能够将持久化的领域事件发送到 RabbitMQ 的服务。你可能想查询 EventStore 中的所有事件并发送每一个，这并不是一个坏主意。然而，我们可能会多次推送同一个领域事件，一般来说，*我们需要最小化重新发布的领域事件数量*。如果重新发布的领域事件数量为 0，那就更好了。为了不重新发布领域事件，我们需要某种组件来跟踪哪些领域事件已经被推送，哪些尚未推送。最后但同样重要的是，一旦我们知道哪些领域事件需要推送，我们就将它们发送出去，并跟踪最后一条发布到我们的消息系统中的消息。让我们看看这个服务的可能实现：

```php
class NotificationService
{
    private $serializer;
    private $eventStore;
    private $publishedMessageTracker;
    private $messageProducer;

    public function __construct(
        EventStore $anEventStore,
        PublishedMessageTracker $aPublishedMessageTracker,
        MessageProducer $aMessageProducer,
        Serializer $aSerializer
    ) {
        $this->eventStore = $anEventStore;
        $this->publishedMessageTracker = $aPublishedMessageTracker;
        $this->messageProducer = $aMessageProducer;
        $this->serializer = $aSerializer;
    }

    /**
     * @return int
     */
    public function publishNotifications($exchangeName)
    {
        $publishedMessageTracker = $this->publishedMessageTracker();
        $notifications = $this->listUnpublishedNotifications(
            $publishedMessageTracker
                ->mostRecentPublishedMessageId($exchangeName)
        );

        if (!$notifications) {
            return 0;
        }

        $messageProducer = $this->messageProducer();
        $messageProducer->open($exchangeName);
        try {
            $publishedMessages = 0;
            $lastPublishedNotification = null;
            foreach ($notifications as $notification) {
                $lastPublishedNotification = $this->publish(
                    $exchangeName,
                    $notification,
                    $messageProducer
                );
                $publishedMessages++;
            }
        } catch (\Exception $e) {
            // Log your error (trigger_error, Monolog, etc.)
        }

        $this->trackMostRecentPublishedMessage(
            $publishedMessageTracker,
            $exchangeName,
            $lastPublishedNotification
        );

        $messageProducer->close($exchangeName);

        return $publishedMessages;
    }

    protected function publishedMessageTracker()
    {
        return $this->publishedMessageTracker;
    }

    /**
     * @return StoredEvent[]
     */
    private function listUnpublishedNotifications(
        $mostRecentPublishedMessageId
    ) {
        return $this
            ->eventStore()
            ->allStoredEventsSince($mostRecentPublishedMessageId);
    }

    protected function eventStore()
    {
        return $this->eventStore;
    }

    private function messageProducer()
    {
        return $this->messageProducer;
    }

    private function publish(
        $exchangeName,
        StoredEvent $notification,
        MessageProducer $messageProducer
    ) {
        $messageProducer->send(
            $exchangeName,
            $this->serializer()->serialize($notification, 'json'),
            $notification->typeName(),
            $notification->eventId(),
            $notification->occurredOn()
        );

        return $notification;
    }

    private function serializer()
    {
       return $this->serializer;
    }

    private function trackMostRecentPublishedMessage(
        PublishedMessageTracker $publishedMessageTracker,
        $exchangeName,
        $notification
    ) {
        $publishedMessageTracker->trackMostRecentPublishedMessage(
            $exchangeName, $notification
        );
    }
}

```

`NotificationService`依赖于三个接口。我们已经看到了`EventStore`，它负责追加和查询领域事件。第二个是`PublishedMessageTracker`，它负责跟踪推送的消息。第三个是`MessageProducer`，这是一个代表我们的消息系统的接口：

```php
interface PublishedMessageTracker
{
    /**
     * @param string $exchangeName
     * @return int
     */
    public function mostRecentPublishedMessageId($exchangeName);

    /**
     * @param string $exchangeName
     * @param StoredEvent $notification
     */
    public function trackMostRecentPublishedMessage(
        $exchangeName, $notification
    );
}

```

`mostRecentPublishedMessageId`方法返回最后一条`PublishedMessage`的 ID，以便过程可以从下一条开始。`trackMostRecentPublishedMessage`负责跟踪最后一条发送的消息，以便在需要时能够重新发布消息。`$exchangeName`代表我们将用于发送领域事件的通信通道。让我们看看`PublishedMessageTracker`的 Doctrine 实现：

```php
class DoctrinePublishedMessageTracker extends EntityRepository\
implements PublishedMessageTracker
{
    /**
     * @param $exchangeName
     * @return int
     */
    public function mostRecentPublishedMessageId($exchangeName)
    {
        $messageTracked = $this->findOneByExchangeName($exchangeName);
        if (!$messageTracked) {
            return null ;
        }

        return $messageTracked->mostRecentPublishedMessageId();
    }

    /**
     *@param $exchangeName
     * @param StoredEvent $notification
     */
    public function trackMostRecentPublishedMessage(
        $exchangeName, $notification
    ) {
        if(!$notification) {
            return;
        }

        $maxId = $notification->eventId();

        $publishedMessage= $this->findOneByExchangeName($exchangeName);
        if(null === $publishedMessage){
            $publishedMessage = new PublishedMessage(
                $exchangeName,
                $maxId
            );
        }

        $publishedMessage->updateMostRecentPublishedMessageId($maxId);

        $this->getEntityManager()->persist($publishedMessage);
        $this->getEntityManager()->flush($publishedMessage);
    }
}

```

这段代码相当简单。我们唯一需要考虑的边缘情况是没有任何领域事件已经被发布。

为什么需要一个交换机名称？

我们将在第十二章“集成边界上下文”中更详细地介绍这一点。然而，当系统运行时，一个新的边界上下文开始发挥作用，你可能希望将所有领域事件重新发送到新的边界上下文。因此，跟踪最后一条已发布的领域事件及其发送的通道可能会在以后派上用场。

为了跟踪已发布的领域事件，我们需要一个交换机名称和一个通知 ID。以下是一个可能的实现：

```php
class PublishedMessage
{
    private $mostRecentPublishedMessageId;
    private $trackerId;
    private $exchangeName;

    /**
     * @param string $exchangeName
     * @param int $aMostRecentPublishedMessageId
     */
    public function __construct(
        $exchangeName, $aMostRecentPublishedMessageId
    ) {
        $this->mostRecentPublishedMessageId =
            $aMostRecentPublishedMessageId;
        $this->exchangeName = $exchangeName;
    }

    public function mostRecentPublishedMessageId()
    {
        return $this->mostRecentPublishedMessageId;
    }

    public function updateMostRecentPublishedMessageId($maxId)
    {
        $this->mostRecentPublishedMessageId = $maxId;
    }

    public function trackerId()
    {
        return $this->trackerId;
    }
}

```

这里是其对应的映射：

```php
Ddd\Domain\Event\PublishedMessage:
    type: entity
    table: event_published_message_tracker
    repositoryClass:
        Ddd\Infrastructure\Application\Notification\
            DoctrinePublished\MessageTracker
    id:
        trackerId:
            column: tracker_id
            type: integer
            generator:
            strategy: AUTO
    fields:
        mostRecentPublishedMessageId:
            column: most_recent_published_message_id
            type: bigint
        exchangeName:
            type: string
            column: exchange_name

```

现在我们来看看`MessageProducer`接口是用来做什么的，以及它的实现细节：

```php
interface MessageProducer 
{ 
    public function open($exchangeName);

    /**
     * @param $exchangeName
     * @param string $notificationMessage
     * @param string $notificationType
     * @param int $notificationId
     * @param \DateTimeImmutable $notificationOccurredOn
     * @return
     */
    public function send(
        $exchangeName,
        $notificationMessage,
        $notificationType,
        $notificationId,
        \DateTimeImmutable $notificationOccurredOn
    );

    public function close($exchangeName);
}

```

很简单。打开和关闭方法用于打开和关闭与消息系统的连接。`send`方法接收一个消息体——消息名称和消息 ID——并将它们发送到我们的消息引擎，无论它是什么。因为我们选择了 RabbitMQ，我们需要实现连接和发送过程：

```php
abstract class RabbitMqMessaging
{
    protected $connection;
    protected $channel ;

    public function __construct(AMQPConnection $aConnection)
    {
        $this->connection =$aConnection;
        $this->channel = null ;
    }

    private function connect($exchangeName)
    {
        if (null !== $this->channel ) {
            return;
        }

        $channel = $this->connection->channel();
        $channel->exchange_declare(
            $exchangeName, 'fanout', false, true, false
        );
        $channel->queue_declare(
            $exchangeName, false, true, false, false
        );
        $channel->queue_bind($exchangeName, $exchangeName);

        $this->channel = $channel ;
    }

    public function open($exchangeName)
    {

    }

    protected function channel ($exchangeName)
    {
        $this->connect($exchangeName);

        return $this->channel;
    }

    public function close($exchangeName)
    {
        $this->channel->close();
        $this->connection->close();
    }
}

class RabbitMqMessageProducer
    extends RabbitMqMessaging
    implements MessageProducer
{
    /**
     * @param $exchangeName
     * @param string $notificationMessage
     * @param string $notificationType
     * @param int $notificationId
     * @param \DateTimeImmutable $notificationOccurredOn
     */
    public function send(
        $exchangeName,
        $notificationMessage,
        $notificationType,
        $notificationId,
        \DateTimeImmutable $notificationOccurredOn
    ) {
        $this->channel ($exchangeName)->basic_publish(
            new AMQPMessage(
                $notificationMessage,
                [
                  'type'=>$notificationType,
                  'timestamp'=>$notificationOccurredOn->getTimestamp(),
                  'message_id'=>$notificationId
                ]
            ),
            $exchangeName
        );
    }
}

```

现在我们已经有一个`DomainService`用于将领域事件推送到像 RabbitMQ 这样的消息系统，现在是时候执行它们了。我们需要选择一个交付机制来运行该服务。我们个人建议创建一个[Symfony 控制台命令](http://symfony.com/doc/current/components/console/introduction.html)：

```php
class PushNotificationsCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('domain:events:spread')
            ->setDescription('Notify all domain events via messaging')
            ->addArgument(
                'exchange-name',
                InputArgument::OPTIONAL,
                'Exchange name to publish events to',
                'my-bc-app'
            );
    }

    protected function execute(
        InputInterface $input, OutputInterface $output
    ) {
        $app = $this->getApplication()->getContainer();

        $numberOfNotifications =
            $app['notification_service']
                ->publishNotifications(
                    $input->getArgument('exchange-name')
                );

        $output->writeln(
            sprintf(
                '<comment>%d</comment>' .
                '<info>notification(s) sent!</info>',
                $numberOfNotifications
            )
        );
    }
}

```

按照 Silex 示例，让我们看看在[Silex Pimple 服务容器](http://silex.sensiolabs.org/doc/services.html#id1)中定义的`$app['notification_service']`的定义：

```php
 // ...
 $app['event_store']=$app->share( function ($app) {
     return $app['em']->getRepository('Ddd\Domain\Event\StoredEvent');
 });

$app['message_tracker'] = $app->share(function($app) {
    return $app['em']
        ->getRepository('Ddd\Domain\Event\Published\Message');
});

$app['message_producer'] = $app->share(function () {
    return new RabbitMqMessageProducer(
       new AMQPStreamConnection('localhost', 5672, 'guest', 'guest')
    );
});

$app['message_serializer'] = $app->share(function () {
    return SerializerBuilder::create()->build();
});

$app['notification_service'] = $app->share(function ($app) {
    return new NotificationService(
       $app['event_store'],
       $app['message_tracker'],
       $app['message_producer'],
       $app['message_serializer']
    );
});
//...

```

# 将领域服务与 REST 同步

在消息系统中已经实现了`EventStore`之后，添加一些分页功能、查询领域事件以及发布 JSON 或 XML 表示形式的 REST API 应该很容易。这有什么有趣的地方吗？嗯，使用消息传递的分布式系统必须面对许多不同的问题，例如消息没有到达、消息重复到达，或者消息以意外的顺序到达。这就是为什么提供一个 API 来发布你的领域事件，以便其他边界上下文可以请求一些缺失信息是很好的。仅作为一个例子，考虑你向`/events`端点发出一个 HTTP 请求。可能的结果如下：

```php
[
    {
        "id": 1,
        "version": 1,
        "typeName": "Lw\\Domain\\Model\\User\\UserRegistered",
        "eventBody": {
            "user_id": {
                "id": "459a4ffc-cd57-4cf0-b3a2-0f2ccbc48234"
            }
        },
        "occurredOn": {
            "date": "2016-05-26 06:06:07.000000",
            "timezone_type": 3,
            "timezone": "UTC"
        }
    },
    {
        "id": 2,
        "version": 2,
        "typeName": "Lw\\Domain\\Model\\Wish\\WishWasMade",
        "eventBody": {
            "wish_id": {
                "id": "9e90435a-395c-46b0-b4c4-d4b769cbf201"
            },
            "user_id": {
                "id": "459a4ffc-cd57-4cf0-b3a2-0f2ccbc48234"
            },
            "address": "john@example.com",
            "content": "This is my new wish!"
        },
        "occurredOn": {
            "date": "2016-05-26 06:06:27.000000",
            "timezone_type": 3,
            "timezone": "UTC"
        },
        "timeTaken": "650"
    },
    //...
]

```

如前一个示例所示，我们正在通过 JSON REST API 公开一组领域事件。在输出示例中，你可以看到每个领域事件的 JSON 表示。有一些有趣的地方。首先，`version`字段。有时你的领域事件会进化：它们将包括更多的字段，它们会改变一些现有字段的行为，或者它们会删除一些现有字段。这就是为什么在领域事件中添加一个版本字段很重要。如果其他边界上下文正在监听此类事件，它们可以使用版本字段以不同的方式解析领域事件。你可能遇到过在版本化 REST API 时遇到相同的问题。

另一点是名称。如果你想使用领域事件的`classname`，在大多数情况下可能可行。问题是当团队决定因为重构而更改类的名称时，所有监听该名称的边界上下文都会停止工作。这个问题只会在你在同一个队列中发布不同的领域事件时出现。如果你在每个不同的队列中发布每种领域事件类型，这并不是真正的问题，但如果你选择这种方法，你将面临另一组问题，例如接收无序的事件。就像在许多其他情况下一样，这里涉及到权衡。我们强烈建议你阅读*《企业集成模式：设计、构建和部署消息解决方案》* [Designing, Building, and Deploying Messaging Solutions](http://www.amazon.com/Enterprise-Integration-Patterns-Designing-Addison-Wesley-ebook/dp/B007MQLL4E)。在这本书中，你将学习到使用异步方法集成多个应用程序的不同模式。因为领域事件是在集成通道中发送的消息，所以所有消息模式也适用于它们。

练习

考虑一下为领域事件拥有 REST API 的利弊。考虑边界上下文的耦合。你也可以尝试为你的当前应用程序实现 REST API。

# 总结

我们已经看到了如何使用基接口来正确建模`DomainEvent`的技巧，我们已经看到了在哪里发布`DomainEvent`（越靠近实体越好），我们也已经看到了将那些`DomainEvents`传播到本地和远程边界上下文的策略。现在，唯一剩下的事情就是在消息系统中监听通知，读取它，并执行相应的应用程序服务或命令。我们将在第十二章*整合边界上下文*和第五章*服务*中看到如何做到这一点。Chapter 12, *Integrating Bounded Contexts* 和 Chapter 5, *Services*.
