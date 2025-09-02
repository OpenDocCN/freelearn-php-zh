# 集成边界上下文

每个企业级应用程序通常由公司运营的几个区域组成。例如，*计费*、*库存*、*运输管理*、*目录*等区域是常见的例子。管理所有这些关注点的最简单方法可能倾向于采用**单体系统**。但是，你可能想知道，这必须是这样吗？如果通过将这个大单体应用程序拆分成更小、更独立的块来减少在这些不同区域工作的团队之间的摩擦，会怎么样呢？在本章中，我们将探讨如何做到这一点，因此请准备好了解关于**战略设计**的见解和启发式方法。

处理分布式系统

处理分布式系统是**困难的**。将系统分解成独立的自主部分有其好处，但它也增加了复杂性。例如，分布式系统的协调和同步不是微不足道的，因此应该仔细考虑。正如马丁·福勒在[PoEAA](https://www.martinfowler.com/books/eaa.html)一书中所说，分布式系统的第一定律始终是：**不要分布式**。

# 通过数据存储进行集成

集成应用程序不同部分的最常用技术之一始终是共享相同的数据存储库和相同的代码库。这通常被称为单体应用，并且通常最终会形成一个单一的数据存储库，它托管着与应用程序中所有相关关注点相关的数据。

考虑一个电子商务应用程序。共享的数据存储库将包含所有关注点（例如：关系型数据库中的表）周围的相关内容，如目录、计费、库存等。这种方法本身并没有问题——例如，在小型线性应用程序中，复杂性不是太高。然而，在复杂的领域内，可能会出现一些问题。如果你在多个涉及多个应用程序关注点的表中共享数据，事务将对性能产生重大影响。

另一个可能出现的非技术问题与通用语言有关。边界上下文分离的主要优势是每个边界上下文都有**一个单一的通用语言**。通过这样做，模型将被分离到它们自己的上下文中。在同一个上下文中混合所有模型可能会导致歧义和混淆。

回到电子商务系统，假设我们想要引入 T 恤的概念。在目录上下文中，T 恤将是一个具有如*颜色*、*尺寸*、*材质*和可能的一些花哨的*图片*等属性的*产品*。然而，在*库存*系统中，我们并不真正关心这些事情。在这里，*产品*有不同的含义，我们关注的是不同的属性，如*重量*、*仓库中的位置*或*尺寸*。将这两个上下文混合在一起会混淆概念并复杂化设计。在领域驱动设计的术语中，以这种方式混合概念被称为共享内核。

共享内核

指定一些领域模型子集，让团队达成共识并共享。当然，这包括与该模型部分相关的代码子集或数据库设计子集。这些明确共享的内容具有特殊地位，并且未经与其他团队协商不应更改。频繁地集成功能系统，但频率略低于团队内部持续集成的步伐。在这些集成中，运行两个团队的测试。埃里克·埃文斯 - [领域驱动设计：软件核心的复杂性处理](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)

我们不建议使用共享内核，因为多个团队在开发过程中可能会发生冲突，这不仅会导致维护问题，还可能成为摩擦的焦点。然而，如果您选择使用共享内核，应在所有相关方之间事先达成一致。从概念上讲，这种方法还有其他问题，比如人们将其视为一个可以放置不属于其他任何地方的*东西*的袋子，并且这种东西会无限增长。处理单体不断增长的复杂性的更好方法是将它拆分成不同的自主部分，例如通过 REST、RPC 或消息系统进行通信。这需要绘制清晰的边界，每个上下文可能最终都会拥有自己的基础设施——数据存储、服务器、消息中间件等等——甚至自己的团队。

如您所想，这可能会导致一定程度的重叠，但这是我们为了减少复杂性而愿意做出的权衡。在领域驱动设计中，我们称这些独立部分为**边界上下文**。

# 集成关系

## 客户 - 供应商

当两个边界上下文之间存在单向集成时，其中一个充当提供者（**上游**），另一个充当客户端（**下游**），我们将得到**客户 - 供应商开发团队**。

在两个团队之间建立清晰的客户/供应商关系。在规划会议中，让下游团队扮演上游团队的客户角色。协商并预算下游团队的需求任务，以便每个人都能理解承诺和进度。共同开发将验证预期接口的自动化验收测试。将这些测试添加到上游团队的测试套件中，作为其持续集成的一部分。这种测试将使上游团队在没有下游副作用恐惧的情况下进行更改。埃里克·埃文斯 - [领域驱动设计：软件核心的复杂性处理](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)。

客户-供应商开发团队是有界上下文集成最常见的方式，并且当团队紧密合作时通常代表双赢的局面。

# 分道扬镳

继续以电子商务为例，考虑向一个旧的遗留零售商财务系统报告收入。这种集成可能非常昂贵，以至于不值得投入精力去实施。在领域驱动设计的战略术语中，这被称为**分道扬镳**。

集成总是昂贵的。有时好处很小。因此，宣布一个有界上下文与其它上下文没有任何联系，允许开发者在这样一个小范围内找到简单、专业的解决方案。埃里克·埃文斯 - *领域驱动设计:* [软件核心的复杂性处理](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)。

# 顺从者

再次考虑电子商务示例和与第三方物流服务的集成。这两个领域在模型、团队和基础设施方面都存在差异。负责维护第三方物流服务的团队不会参与你的产品规划或为电子商务系统提供任何解决方案。这些团队之间没有紧密的关系。我们可以选择接受并**顺从**他们的领域模型。在战略设计中，我们称之为**顺从集成**。

通过盲目遵循上游团队的模型来消除有界上下文之间翻译的复杂性。尽管这可能会限制下游设计师的风格，并且可能不会产生理想的应用模型，但选择一致性可以极大地简化集成。此外，你将与供应商团队共享一个**无处不在的语言**。供应商处于主导地位，因此让他们的沟通变得容易是好事。利他主义可能足以让他们与你分享信息。埃里克·埃文斯 - *领域驱动设计:* [软件核心的复杂性处理](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)。

# 实施有界上下文集成

为了简化事情，我们将假设有界上下文有一个客户-供应商关系。

# 现代 RPC

在现代 RPC 中，我们通过 RESTful 资源来指代 RPC。一个有界上下文揭示了一个清晰的接口，用于与外界交互。它暴露了可以通过 HTTP 动词进行操作的资源。我们可以这样说，有界上下文提供了一套服务和操作。从战略的角度来看，这被称为**开放主机服务**。

开放主机服务

定义一个协议，该协议将您的子系统作为一组服务提供访问。开放协议，以便所有需要与您集成的人都可以使用它。增强和扩展协议以处理新的集成需求，除非单个团队有特殊需求。然后，使用一次性翻译器来增强该特殊情况的协议，以便共享协议可以保持简单和一致。埃里克·埃文斯 - *领域驱动设计：* [在软件核心处理复杂性](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)*.*

让我们探索这本书的 GitHub 组织提供的[最后愿望应用程序](https://github.com/dddinphp/last-wishes)中的示例。

该应用程序是一个旨在让人们在他们去世之前保存他们的遗嘱的 Web 平台。有两个上下文：一个负责处理遗嘱的有界上下文（Will Bounded Context），另一个负责给系统用户评分的[游戏化上下文](https://github.com/dddinphp/last-wishes-gamify)。在遗嘱上下文中，用户可能会有与他们在游戏化上下文中获得的积分相关的徽章。这意味着我们需要将这两个上下文集成在一起，以便显示用户在遗嘱上下文中的徽章。

游戏化上下文是一个完整的事件驱动应用程序，由一个定制的事件源引擎提供支持。它是一个全栈 Symfony 应用程序，使用[FOSRestBundle](http://symfony.com/doc/current/bundles/FOSRestBundle/index.html)、[BazingaHateoasBundle](https://github.com/willdurand/BazingaHateoasBundle)、[JMSSerializerBundle](https://github.com/schmittjoh/JMSSerializerBundle)、[NelmioApiDocBundle](https://github.com/schmittjoh/JMSSerializerBundle)和[OngrElasticsearchBundle](https://github.com/schmittjoh/JMSSerializerBundle)来提供 3 级及以上 REST API（通常称为 REST 的荣耀），根据[理查森成熟度*模型](https://martinfowler.com/articles/richardsonMaturityModel.html)。在这个上下文中触发的事件都会投影到一个 Elasticsearch 服务器上，以产生视图所需的数据。我们将通过类似`http://gamification.context.host/api/users/{id}`的端点公开特定用户的积分数量。

我们还将从 Elasticsearch 获取用户投影，并将其序列化为与客户端先前协商的格式：

```php
namespace AppBundle\Controller;

use FOS\RestBundle\Controller\Annotations as Rest;
use FOS\RestBundle\Controller\FOSRestController;
use Nelmio\ApiDocBundle\Annotation\ApiDoc;

class UsersController extends FOSRestController
{
    /**
     * @ApiDoc(
     *     resource = true,
     *     description = "Finds a user given a user ID",
     *     statusCodes = {
     *         200 = "Returned when the user have been found",
     *         404 = "Returned when the user could not be found"
     *     }
     * )
     *
     * @Rest\View(
     *     statusCode = 200
     * )
     */
    public function getUserAction($id)
    {
        $repo = $this->get('es.manager.default.user');
        $user = $repo->find($id);

        if (!$user) {
            throw $this->createNotFoundException(
                sprintf(
                    'A user with an ID of %s does not exist',
                    $id
                )
            );
        }
        return $user;
    }
}

```

正如我们在第二章中解释的，*架构风格*的读取被视为基础设施关注点，因此不需要将它们包裹在命令/命令处理流程中。

用户的结果 JSON+HAL 表示形式如下：

```php
{
    "id": "c3c587c6-610a-42df",
    "points": 0,
    "_links": {
        "self": {
            "href":
            "http://gamification.ctx/api/users/c3c587c6-610a-42df"
        }
    }
}

```

现在，我们处于整合两个上下文的好位置。我们只需要在 Will 上下文中编写客户端来消费我们刚刚创建的端点。我们应该混合这两个领域模型吗？直接消化游戏化上下文将意味着将 Will 上下文调整为游戏化上下文，从而导致**同质化**的集成。然而，分离这些关注点似乎值得付出努力。我们需要一个层来保证领域模型在 Will 上下文中的完整性和一致性，并且我们需要将*分数*（游戏化）转换为*徽章*（Will）。在领域驱动设计中，这种转换机制被称为**反腐败层**。

反腐败层

创建一个隔离层，为客户端提供其自身领域模型的功能。该层通过其现有接口与其他系统通信，对其他系统几乎没有或不需要进行修改。内部，该层在两个模型之间按需进行双向转换。埃里克·埃文斯 - *领域驱动设计*：[在软件核心解决复杂性。](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)

那么，反腐败层看起来是什么样子？大多数时候，服务将与适配器和外观的组合进行交互。服务封装并隐藏了这些转换背后的低级复杂性。外观有助于隐藏和封装从游戏化模型获取数据所需的访问细节。适配器在模型之间进行转换，通常使用专门的翻译器。

让我们看看如何在 Will 模型中定义一个用户服务，该服务将负责检索给定用户获得的徽章：

```php
namespace Lw\Domain\Model\User;

interface UserService
{
    public function badgesFrom(UserId $id);
}

```

现在，让我们看看基础设施方面的实现。我们将使用适配器来完成转换过程：

```php
namespace Lw\Infrastructure\Service;

use Lw\Domain\Model\User\UserId;
use Lw\Domain\Model\User\UserService;

class TranslatingUserService implements UserService
{
    private $userAdapter;

    public function __construct(UserAdapter $userAdapter)
    {
        $this->userAdapter = $userAdapter;
    }

    public function badgesFrom(UserId $id)
    {
        return $this->userAdapter->toBadges($id);
    }
}

```

下面是`UserAdapter`的 HTTP 实现：

```php
namespace Lw\Infrastructure\Service;

use GuzzleHttp\Client;

class HttpUserAdapter implements UserAdapter
{
    private $client;

    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function toBadges( $id)
    {
        $response = $this->client->get(
            sprintf('/users/%s', $id),
            [
                'allow_redirects' => true,
                'headers' => [
                    'Accept' => 'application/hal+json'
                ]
            ]
        );

        $badges = [];
        if (200 === $response->getStatusCode()) {
            $badges = 
                (new UserTranslator())
                    ->toBadgesFromRepresentation(
                        json_decode(
                            $response->getBody(),
                            true
                        )
                    );
        }
        return $badges;
    }
}

```

如您所见，适配器也充当了**游戏化上下文的外观**。我们这样做是因为在游戏化侧获取用户资源相当直接。适配器使用`UserTranslator`来执行转换：

```php
namespace Lw\Infrastructure\Service;

use Lw\Infrastructure\Domain\Model\User\FirstWillMadeBadge;
use Symfony\Component\PropertyAccess\PropertyAccess;

class UserTranslator
{
    public function toBadgesFromRepresentation($representation)
    {
        $accessor = PropertyAccess::createPropertyAccessor();
        $points = $accessor->getValue($representation, 'points');
        $badges = [];
        if ($points > 3) {
            $badges[] = new FirstWillMadeBadge();
        }
        return $badges;
    }
}

```

翻译器专门负责将来自游戏化上下文的分数转换为徽章。

我们已经展示了如何集成两个边界上下文，其中相应的团队共享**客户-供应商**关系。游戏化上下文通过一个由 RESTful 协议实现的**开放主机服务**公开集成。另一方面，Will 上下文通过一个**反腐败层**消费服务，该层负责将一个领域模型转换为另一个领域模型，确保 Will 上下文的完整性。

# 消息队列

RESTful 资源并不是实现边界上下文之间集成的唯一方式。正如我们将看到的，消息中间件使得不同上下文之间的解耦集成成为可能。

在继续使用 Last Wishes 应用程序的过程中，我们刚刚实现了两支团队之间的单向关系，以管理各自上下文中的积分和徽章。然而，我们故意将一个重要的功能排除在范围之外：**每次用户许愿时都给予奖励**。

我们可以考虑采用另一种具有拉取策略的开放主机服务。Will Context 将会定期拉取 Gamification Context 以同步徽章（例如：通过像 Cron 这样的调度器）。这种解决方案将影响用户体验，并且会浪费大量不必要的资源。

更好的方法是使用**消息中间件**。使用这种解决方案，上下文可以将消息推送到中间件（通常是消息队列）。感兴趣的各方将能够订阅、检查和按需以解耦的方式消费信息。为了做到这一点，我们需要一种**专业、共享和通用的通信语言**，以便所有各方都能理解传输的信息。这就是所谓的**发布语言**。

发布语言

使用一个经过良好文档化的共享语言，该语言可以表达必要的领域信息，作为通用的通信媒介，并在必要时将其翻译成和从该语言中翻译出来。 埃里克·埃文斯 - *领域驱动设计：* [软件核心的复杂性处理](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)。

在思考这些消息的格式并更仔细地审视我们的领域模型时，我们意识到我们已经有我们所需要的东西：第六章，*领域事件*。没有必要定义一种新的在边界上下文之间进行通信的方式。相反，我们可以直接使用领域事件来定义上下文之间的通用语言。*领域专家关心刚刚发生的事情*的定义与我们正在寻找的完美契合：一种正式的发布语言。

在我们的示例中，我们可以使用 RabbitMQ 作为消息中间件。这可能是最可靠和最健壮的消息 [AMQP](https://www.amqp.org/) 协议之一。我们还将整合广泛使用的 PHP 库 [php-amqplib](https://github.com/php-amqplib/php-amqplib) 和 [RabbitMQBundle](https://github.com/php-amqplib/RabbitMqBundle)。

让我们从 Will Context 开始，因为它是当用户注册或许愿时触发事件的那个。正如我们在 第六章 中看到的，*领域事件*，**将领域事件存储到持久机制中是一个好主意**，所以我们假设这就是所做的工作。我们需要一个消息发布者来从事件存储中检索并发布存储的领域事件到消息中间件。我们已经在 第六章 的 *领域事件* 中完成了与 RabbitMQ 的集成，所以我们只需要在 Gamification Context 中实现代码。我们将监听 Will Context 触发的事件。由于我们使用的是 Symfony 框架，我们利用一个名为 RabbitMQBundle 的 Symfony 包。

我们为 *用户注册* 和 *愿望已创建* 事件定义了两个消息消费者：

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;

use Lw\Gamification\Command\SignupCommand;
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;

class PhpAmqpLibLastWillUserRegisteredConsumer
    implements ConsumerInterface
{
    private $commandBus;

    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');

        if('Lw\Domain\Model\User\UserRegistered' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);

            $this->commandBus->handle(
                new SignupCommand($eventBody->user_id->id)
            );
            return true;
        }
        return false;
    }
}

```

注意，在这种情况下，我们只处理类型为 `Lw\Domain\Model\User\UserRegistered` 的消息：

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;

use Lw\Gamification\Command\RewardUserCommand;
use Lw\Gamification\Domain\Model\AggregateDoesNotExist; 
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;

class PhpAmqpLibLastWillWishWasMadeConsumer implements ConsumerInterface
{
    private $commandBus;

    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');

        if ('Lw\Domain\Model\Wish\WishWasMade' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);

            try {
                $points = 5;
                $this->commandBus->handle(
                    new RewardUserCommand(
                        $eventBody->user_id->id,
                        $points
                    )
                );
            } catch (AggregateDoesNotExist $e) {
                // Noop
            }

            return true;
        }

        return false;
    }
}

```

再次强调，我们只对跟踪 `Lw\Domain\Model\Wish\WishWasMade` 事件感兴趣。

在这两种情况下，我们使用命令总线，这在应用章节中已经讨论过。然而，我们可以将其总结为一个解耦命令和接收者的高速公路。命令的 **何时** 和 **如何** 执行与 **谁** 触发它是独立的。

Gamification Context 使用 [Tactician](http://tactician.thephpleague.com/)（以及 [TacticianBundle](https://github.com/thephpleague/tactician-bundle)），这是一个简单的命令总线，可以扩展和适应您的系统。因此，我们现在几乎准备好从 Will Context 消费事件了。

我们还需要做的一件事是在 Symfony 的 `config.yml` 文件中定义 RabbitMQBundle 配置：

```php
services:
    last_will_user_registered_consumer:
        class:
            AppBundle\Infrastructure\Messaging\
                PhpAmqpLib\PhpAmqpLibLastWillUserRegisteredConsumer
        arguments:
            - @tactician.commandbus

    last_will_wish_was_made_consumer:
        class:
            AppBundle\Infrastructure\Messaging\
                PhpAmqpLib\PhpAmqpLibLastWillWishWasMadeConsumer
        arguments:
            - @tactician.commandbus

old_sound_rabbit_mq:
    connections:
         default:
              host: " %rabbitmq_host%"
              port: " %rabbitmq_port%"
              user: " %rabbitmq_user%"
              password: " %rabbitmq_password%"
              vhost: " %rabbitmq_vhost%"
              lazy: true

    consumers:
        last_will_user_registered:
            connection: default
            callback: last_will_user_registered_consumer

            exchange_options:
                name: last-will
                type: fanout

            queue_options:
                name: last-will

        last_will_wish_was_made:
            connection: default
            callback: last_will_wish_was_made_consumer

            exchange_options:
                name: last-will
                type: fanout

            queue_options:
                name: last-wil

```

最方便的 RabbitMQ 配置可能是 [[发布/订阅](https://www.rabbitmq.com/tutorials/tutorial-three-php.html)] 模式。Will Context 发布的所有消息都将被发送到所有连接的消费者。这在 RabbitMQ 交换配置中被称为 **fanout**。

交换由一个负责将消息发送到相应队列的代理组成：

```php
> php app/console rabbitmq:consumer --messages=1000 last_will_user_registered
> php app/console rabbitmq:consumer --messages=1000 last_will_wish_was_made

```

使用这两个命令，Symfony 将执行两个消费者，它们将开始监听领域事件。我们指定了消费消息的限制为 1,000 条，因为 PHP 不是执行长时间运行进程的最佳平台。使用类似 [Supervisor](http://supervisord.org/) 的工具定期监控和重启进程可能也是一个好主意。

# 总结

尽管我们只看到了其中的一小部分，但战略设计是领域驱动设计的核心和灵魂。它是开发更好、更语义化模型的一个基本部分。我们建议使用消息中间件来集成边界上下文，因为这自然会引导出更简单、解耦和事件驱动的架构。
