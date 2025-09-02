# 使用 PHP 的六边形架构

以下文章由 Carlos Buenosvinos 于 2014 年 6 月发布在 php|architect 杂志上。

# 简介

随着领域驱动设计（**DDD**）的兴起，促进以领域为中心设计的架构变得越来越流行。这就是六边形架构（也称为**端口和适配器**）的情况，它似乎刚刚被 PHP 开发者重新发现。由敏捷宣言的作者之一 Alistair Cockburn 于 2005 年发明，六边形架构允许应用程序由用户、程序、自动化测试或批处理脚本同等驱动，并且可以在与最终运行时设备和数据库隔离的情况下开发和测试。这导致出现无依赖的基础设施 Web 应用程序，这些应用程序更容易测试、编写和维护。让我们看看如何使用真实的 PHP 示例来应用它。

您的公司正在构建一个名为*Idy*的头脑风暴系统。用户添加和评分想法，以便最有趣的想法可以在公司中实施。这是周一早晨，另一个冲刺即将开始，您正在与您的团队和产品负责人审查一些用户故事。**作为一个未登录的用户，我想对想法进行评分，并且作者应该通过电子邮件通知**，这是一个非常重要的用例，不是吗？

# 第一种方法

作为一名优秀的开发者，您决定分而治之用户故事，所以您将从第一部分开始，*我想对想法进行评分*。之后，您将面临*作者应该通过电子邮件通知*。这听起来像是个计划。

在业务规则方面，对想法进行评分就像在想法存储库中通过标识符找到想法一样简单，所有想法都存储在那里，添加评分，重新计算平均值，并将想法保存回存储库。如果想法不存在或存储库不可用，我们应该抛出异常，以便我们可以显示错误消息，重定向用户或执行业务要求我们做的任何事情。

为了*执行*这个*用例*，我们只需要用户的想法标识符和评分。这两个整数将来自用户请求。

您的公司网络应用程序正在处理一个 Zend Framework 1 遗留应用程序。像大多数公司一样，您的应用程序可能某些部分较新，更符合 SOLID 原则，而其他部分可能只是一团糟。然而，您知道您使用的框架并不重要，关键是编写干净的代码，使维护成为公司低成本的作业。

您试图应用您在上次会议中记得的一些敏捷原则，是的，我记得“先让它工作，再让它正确，最后让它快速”。经过一段时间的工作，您得到了类似于列表 1 的内容。

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        // Getting parameters from the request
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        // Building database connection
        $db = new Zend_Db_Adapter_Pdo_Mysql([
            'host'     => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname'   => 'idy'
        ]);

        // Finding the idea in the database
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $db->fetchRow($sql, $ideaId);
        if (!$row) {
            throw new Exception('Idea does not exist');
        }

        // Building the idea from the database
        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);

        // Add user rating
        $idea->addRating($rating);

        // Update the idea and save it to the database
        $data = [
            'votes' => $idea->getVotes(),
            'rating' => $idea->getRating()
        ];
        $where['idea_id = ?'] = $ideaId;
        $db->update('ideas', $data, $where);

        // Redirect to view idea page
        $this->redirect('/idea/' . $ideaId);
    }
}

```

我知道读者在想什么：*谁会直接从控制器访问数据？这是一个 90 年代的做法！* 好吧，好吧，你说得对。如果你已经在使用框架，那么你很可能也在使用 ORM。可能是你自己做的，或者是现有的任何一个，比如 Doctrine、Eloquent、Zend 等等。如果是这种情况，你比那些只有一些数据库连接对象但没有实际操作的人更进一步。但在孵化之前不要过早地计算你的小鸡。

对于新手来说，列表 1 的代码可以正常工作。然而，如果你仔细看看控制器，你会发现不仅仅是业务规则，你还会看到你的 Web 框架如何将请求路由到业务规则，对数据库的引用或如何连接到它。非常接近，你看到了对你的 **基础设施** 的引用。

基础设施是使你的业务规则生效的 **细节**。显然，我们需要某种方式来访问它们（API、Web、控制台应用程序等等），并且实际上我们需要一个物理位置来存储我们的想法（内存、数据库、NoSQL 等等）。然而，我们应该能够用另一个以相同方式但具有不同实现的行为的组件来交换这些组件。那么从数据库访问开始怎么样？

所有那些 `Zend_DB_Adapter` 连接（或者如果你是直接使用 MySQL 命令，那么就是直接 MySQL 命令）都在请求提升为某种封装了获取和持久化想法对象的对象。它们在恳求成为仓库。

# 仓库和持久化优势

无论业务规则还是基础设施是否有变化，我们都必须编辑相同的代码。相信我，在计算机科学中，你不想让很多人出于不同的原因触摸相同的代码。尽量让你的函数只做一件事，这样人们就不太可能对相同的代码进行干扰。你可以通过查看 **单一职责原则**（**SRP**）来了解更多关于这一点。有关此原则的更多信息：[`www.objectmentor.com/resources/articles/srp.pdf`](http://www.objectmentor.com/resources/articles/srp.pdf)

列表 1 明显是这种情况。如果我们想迁移到 Redis 或添加作者通知功能，你将不得不更新 `rateAction` 方法。影响与更新无关的 `rateAction` 方面的可能性很高。列表 1 的代码很脆弱。如果你的团队中经常听到 *如果它工作，就不要动它*，那么就缺少了 SRP。

因此，我们必须解耦我们的代码，并将处理获取和持久化想法的责任封装到另一个对象中。正如之前解释的，最好的方式是使用仓库。挑战接受！让我们看看列表 2 中的结果：

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new IdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if (!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);
        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}

class IdeaRepository
{
    private $client;

    public function __construct()
    {
        $this->client = new Zend_Db_Adapter_Pdo_Mysql([
            'host' => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname' => 'idy'
        ]);
    }

    public function find($id)
    {
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $this->client->fetchRow($sql, $id);
        if (!$row) {
            return null;
        }

        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);

        return $idea;
    }

    public function update(Idea $idea)
    {
        $data = [
            'title' => $idea->getTitle(),
            'description' => $idea->getDescription(),
            'rating' => $idea->getRating(),
            'votes' => $idea->getVotes(),
            'email' => $idea->getAuthor(),
        ];

        $where = ['idea_id = ?' => $idea->getId()];
        $this->client->update('ideas', $data, $where);
    }
}

```

结果更令人满意。`IdeaController` 的 `rateAction` 方法更易于理解。当阅读时，它讨论的是业务规则。`IdeaRepository` 是一个 **业务概念**。当与业务人员交谈时，他们理解 `IdeaRepository` 是什么：一个存放想法并获取它们的地方。

存储库*通过使用类似集合的接口来访问领域对象，在领域和数据映射层之间进行调解*，正如在马丁·福勒的模式目录中找到的那样。

如果你已经使用了一个如 Doctrine 的 ORM，你的当前存储库是从一个`EntityRepository`扩展出来的。如果你需要获取这些存储库之一，你要求 Doctrine 的`EntityManager`来完成这项工作。生成的代码几乎相同，只是在控制器动作中额外访问了`EntityManager`以获取`IdeaRepository`。

在这一点上，我们可以在六边形的景观中看到其中一条边，即*persistence*边。然而，这一边画得并不好，`IdeaRepository`是什么以及它是如何实现的之间仍然存在一些关系。

为了在*应用程序边界*和*基础设施边界*之间进行有效的分离，我们需要额外的步骤。我们需要使用某种形式的接口显式地解耦行为和实现。

# 解耦业务和持久化

你是否曾经遇到过这样的情况：当你开始与你的产品负责人、业务分析师或项目经理谈论你的数据库问题时？你能记得他们解释如何持久化和检索对象时的表情吗？他们对你所说的毫无头绪。

事实是，他们并不关心，但这没关系。如果你决定在 MySQL 服务器、Redis 或 SQLite 中存储想法，那是你的问题，不是他们的。记住，从业务角度来看，**你的基础设施是一个细节**。业务规则不会因为使用 Symfony 或 Zend Framework、MySQL 或 PostgreSQL、REST 或 SOAP 等而改变。

这就是为什么将我们的`IdeaRepository`与其实现解耦很重要。最简单的方法是使用一个合适的接口。我们如何实现这一点？让我们看看列表 3。

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new MySQLIdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if(!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);
        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}

interface IdeaRepository
{
    /**
     * @param int $id
     * @return null|Idea
     */
    public function find($id);

    /**
     * @param Idea $idea
     */
    public function update(Idea $idea);
}

class MySQLIdeaRepository implements IdeaRepository
{
    // ...
}

```

很简单，不是吗？我们已经将`IdeaRepository`的行为提取到一个接口中，将`IdeaRepository`重命名为`MySQLIdeaRepository`，并将`rateAction`更新为使用我们的`MySQLIdeaRepository`。但有什么好处呢？

我们现在可以用任何实现相同接口的存储库来替换控制器中使用的存储库。那么，让我们尝试一个不同的实现。

# 将我们的持久化迁移到 Redis

在冲刺阶段和与一些伙伴交谈之后，你意识到采用 NoSQL 策略可以提高你功能的性能。Redis 是你的好朋友之一。去做吧，并给我展示你的列表 4：

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if (!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);
        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}

interface IdeaRepository
{
    // ...
}

class RedisIdeaRepository implements IdeaRepository
{
    private $client;

    public function __construct()
    {
        $this->client = new Predis\Client();
    }

    public function find($id)
    {
        $idea = $this->client->get($this->getKey($id));
        if (!$idea) {
            return null;
        }
        return unserialize($idea);
    }

    public function update(Idea $idea)
    {
        $this->client->set(
            $this->getKey($idea->getId()),
            serialize($idea)
        );
    }

    private function getKey($id)
    {
        return 'idea:' . $id;
    }
}

```

再次简单。你已经创建了一个实现`IdeaRepository`接口的`RedisIdeaRepository`，我们决定使用 Predis 作为连接管理器。代码看起来更小、更简单、更快。但控制器呢？它保持不变，我们只是更改了使用的存储库，但这只是一行代码。

作为对读者的练习，尝试创建 SQLite、文件或使用数组的内存实现`IdeaRepository`。如果你考虑 ORM 存储库如何与领域存储库相匹配以及 ORM `@annotations`如何影响这种架构，那么你将获得额外的分数。

# 解耦业务和 Web 框架

我们已经看到从一种持久化策略切换到另一种策略是多么容易。然而，持久化并不是我们六边形架构的唯一优势。用户如何与应用程序交互呢？

您的 CTO 在路线图中已经设定，您的团队将迁移到 Symfony2，因此当您在当前的 ZF1 应用程序中开发新功能时，我们希望使即将到来的迁移更加容易。这有点棘手，请展示您的列表 5：

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    { 
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute($ideaId, $rating);

        $this->redirect('/idea/' . $ideaId);
    }
}

interface IdeaRepository
{
    // ...
}

class RateIdeaUseCase
{
    private $ideaRepository;

    public function __construct(IdeaRepository $ideaRepository)
    {
        $this->ideaRepository = $ideaRepository;
    }

    public function execute($ideaId, $rating)
    {
        try {
            $idea = $this->ideaRepository->find($ideaId);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        if (!$idea) {
            throw new IdeaDoesNotExistException();
        }

        try {
            $idea->addRating($rating);
            $this->ideaRepository->update($idea);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        } 

        return $idea;
    }
}

```

让我们回顾一下变化。我们的控制器根本没有任何业务规则。我们已经将所有逻辑推入一个名为`RateIdeaUseCase`的新对象中，该对象封装了它。这个对象也被称为控制器、交互器或应用程序服务。

魔法是由`execute`方法完成的。所有依赖项，如`RedisIdeaRepository`，都作为参数传递给构造函数。我们 UseCase 内部对`IdeaRepository`的所有引用都指向接口，而不是任何具体实现。

这真的很酷。如果你看看`RateIdeaUseCase`内部，没有任何关于 MySQL 或 Zend Framework 的提及。没有引用，没有实例，没有注解，什么都没有。就像你的基础设施不在乎一样。它只是谈论业务逻辑。

此外，我们还调整了我们抛出的异常。业务流程也有异常。`NotAvailableRepository`和`IdeaDoesNotExist`是其中两个。根据抛出的异常，我们可以在框架边界中采取不同的反应。

有时候，一个 UseCase 接收的参数数量可能太多。为了组织它们，使用**数据传输对象**（**DTO**）构建一个*UseCase 请求*来一起传递是很常见的。让我们看看如何在列表 6 中解决这个问题：

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect('/idea/' . $response->idea->getId());
    }
}

class RateIdeaRequest
{
    public $ideaId;
    public $rating;

    public function __construct($ideaId, $rating)
    {
        $this->ideaId = $ideaId;
        $this->rating = $rating;
    }
}

class RateIdeaResponse
{
    public $idea;

    public function __construct(Idea $idea)
    {
        $this->idea = $idea;
    }  
}

class RateIdeaUseCase
{
    // ...

    public function execute($request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;

        // ...

        return new RateIdeaResponse($idea);
    }
}

```

这里的主要变化是引入了两个新对象，一个请求和一个响应。它们不是强制的，也许一个 UseCase 没有请求或响应。另一个重要细节是如何构建这个请求。在这种情况下，我们是通过从 ZF 请求对象获取参数来构建它的。

好的，但等等，真正的益处是什么？从一种框架切换到另一种框架，或者通过另一种*交付机制*执行我们的 UseCase，这更容易。让我们看看这个观点。

# 使用 API 对想法进行评分

在白天，你的产品负责人来到你面前说：“顺便说一句，用户应该能够使用我们的移动应用对想法进行评分。我想我们需要更新 API，你能在这个冲刺中完成吗？”PO 又来了。“没问题！”业务对你的承诺印象深刻。

正如罗伯特·C·马丁所说：<q>网络是一种交付机制 [...] 你的系统架构应该尽可能不了解它如何交付。你应该能够以控制台应用程序、Web 应用程序或甚至是 Web 服务应用程序的形式交付它，而不会造成不必要的复杂性或对基本架构的任何更改</q>。

你当前的 API 是使用基于 Symfony2 组件的 PHP 微框架 Silex 构建的。让我们在列表 7 中看看：

```php
require_once __DIR__.'/../vendor/autoload.php';

$app = new Silex\Application();

// ... more routes

$app->get(
    '/api/rate/idea/{ideaId}/rating/{rating}',
    function ($ideaId, $rating) use ($app) {
        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        return $app->json($response->idea);
    }
);

$app->run();

```

有什么熟悉的东西吗？你能识别出你之前见过的某些代码吗？我会给你一个提示：

```php
$ideaRepository = new RedisIdeaRepository();
$useCase = new RateIdeaUseCase($ideaRepository);
$response = $useCase->execute(
    new RateIdeaRequest($ideaId, $rating)
);

```

*哇！我记得这三行代码。它们看起来和 Web 应用程序一模一样*。没错，因为用例封装了你准备请求、获取响应并相应行动所需的企业规则。

我们为用户提供了一种对想法进行评分的另一种方式；另一种*交付机制*。主要区别在于我们从哪里创建了`RateIdeaRequest`。在第一个例子中，它来自 ZF 请求，现在它来自使用路由中匹配的参数的 Silex 请求。

# 控制台应用程序 评分

有时，用例将从 Cron 作业或命令行执行。例如，批量处理或一些测试命令行以加速开发。在通过 Web 或 API 测试这个功能时，你会发现有一个命令行来做这件事会很好，这样你就不必通过浏览器了。

如果你使用 shell 脚本文件，我建议你检查 Symfony Console 组件。代码会是什么样子：

```php
namespace Idy\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class VoteIdeaCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('idea:rate')
            ->setDescription('Rate an idea')
            ->addArgument('id', InputArgument::REQUIRED)
            ->addArgument('rating', InputArgument::REQUIRED);
    }

    protected function execute(
        InputInterface $input,
        OutputInterface $output
    ) {
        $ideaId = $input->getArgument('id');
        $rating = $input->getArgument('rating');

        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $output->writeln('Done!');
      }
 }

```

再次是这三行代码。和之前一样，用例及其业务逻辑保持不变，我们只是提供了一个新的*交付机制*。恭喜你，你已经发现了*用户端*六边形边缘。

还有好多事情要做。正如你可能听说的，真正的工匠会做 TDD。我们已经开始了我们的故事，所以我们必须接受只是测试。

# 测试 评估一个想法 用例

迈克尔·费瑟斯给遗留代码下了一个定义，即*没有测试的代码*。你不想你的代码一出生就是遗留的，对吧？

为了对这一用例对象进行单元测试，你决定从最容易的部分开始，如果仓库不可用会发生什么？我们如何生成这种行为？我们在运行单元测试时停止我们的 Redis 服务器吗？不。我们需要一个具有这种行为的对象。让我们在列表 9 中使用一个*模拟*对象：

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function whenRepositoryNotAvailableAnExceptionIsThrown()
    {
        $this->setExpectedException('NotAvailableRepositoryException');
        $ideaRepository = new NotAvailableRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}

class NotAvailableRepository implements IdeaRepository
{
    public function find($id)
    {
        throw new NotAvailableException();
    }

    public function update(Idea $idea)
    {
        throw new NotAvailableException();
    }
}

```

很好。`NotAvailableRepository`具有我们需要的特性，我们可以使用它与`RateIdeaUseCase`一起，因为它实现了`IdeaRepository`接口。

接下来要测试的情况是如果想法不在仓库中会发生什么。列表 10 显示了代码：

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function whenIdeaDoesNotExistAnExceptionShouldBeThrown()
    {
        $this->setExpectedException('IdeaDoesNotExistException');
        $ideaRepository = new EmptyIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}

class EmptyIdeaRepository implements IdeaRepository
{
    public function find($id)
    {
        return null;
    }

    public function update(Idea $idea)
    {

    }
}

```

这里，我们使用相同的策略，但使用了一个`EmptyIdeaRepository`。它也实现了相同的接口，但实现总是返回 null，无论 find 方法接收哪个标识符。

为什么我们要测试这些情况？记住肯特·贝克的这句话：*测试所有可能出错的东西*。

让我们继续其他功能的剩余部分。我们需要检查一个与有一个可读但不可写的存储库相关的特殊情况。解决方案可以在列表 11 中找到：

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function whenRatingAnIdeaNewRatingShouldBeAdded()
    {
        $ideaRepository = new OneIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );

        $this->assertSame(5, $response->idea->getRating());
        $this->assertTrue($ideaRepository->updateCalled);
    }
}

class OneIdeaRepository implements IdeaRepository
{
    public $updateCalled = false;

    public function find($id)
    {
        $idea = new Idea();
        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('john@example.com');

        return $idea;
    }

    public function update(Idea $idea)
    {
        $this->updateCalled = true;
    }
}

```

好的，现在这个功能的要点仍然存在。我们有不同的测试方式，我们可以编写自己的模拟或使用模拟框架，如 Mockery 或 Prophecy。让我们选择第一个。另一个有趣的练习将是使用这些框架之一编写这个示例和前面的示例：

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function whenRatingAnIdeaNewRatingShouldBeAdded()
    {
        $ideaRepository = new OneIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );

        $this->assertSame(5, $response->idea->getRating());
        $this->assertTrue($ideaRepository->updateCalled);
    }
}

class OneIdeaRepository implements IdeaRepository
{
    public $updateCalled = false;

    public function find($id)
    {
        $idea = new Idea();
        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('john@example.com');

        return $idea;
    }

    public function update(Idea $idea)
    {
        $this->updateCalled = true;
    }
}

```

嘭！UseCase 的 100% 覆盖率。也许，下次我们可以使用 TDD 来实现，这样测试就会先进行。然而，由于这种架构中推广了解耦的方式，测试这个功能实际上非常简单。

你可能对此感到好奇：

```php
$this->updateCalled = true;

```

我们需要一个方法来确保在 UseCase 执行期间已经调用了更新方法。这个方法就做到了这一点。这个 *测试替身* 对象被称为 *间谍*，*模拟*的表亲。

何时使用模拟？作为一个一般规则，当跨越边界时使用模拟。在这种情况下，我们需要模拟，因为我们正在从领域跨越到持久性边界。

关于测试基础设施，怎么办？

# 测试基础设施

如果你想要为整个应用程序实现 100% 的覆盖率，你也必须测试你的基础设施。在这样做之前，你需要知道，这些单元测试将比业务测试更多地与你的实现相关联。这意味着，随着实现细节的变化而出现问题的概率更高。因此，这是一个你必须考虑的权衡。

所以，如果你想继续，我们需要做一些修改。我们需要进一步解耦。让我们看看列表 13 中的代码：

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $useCase = new RateIdeaUseCase(
            new RedisIdeaRepository(
                new Predis\Client()
            )
        );

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect('/idea/' . $response->idea->getId());
    }
}

class RedisIdeaRepository implements IdeaRepository
{
    private $client;

    public function __construct($client)
    {
        $this->client = $client;
    }

    // ...

    public function find($id)
    {
        $idea = $this->client->get($this->getKey($id));
        if (!$idea) {
            return null;
        }

       return $idea;
   }
}

```

如果我们想要 100% 单元测试 `RedisIdeaRepository`，我们需要能够将 `Predis\Client` 作为参数传递给存储库，而不指定类型提示，这样我们就可以传递一个模拟来强制执行必要的代码流，以覆盖所有情况。

这迫使我们更新控制器以建立 Redis 连接，将其传递给存储库，并将结果传递给 UseCase。

现在，一切都关于创建模拟、测试用例，并在断言中享受乐趣。

# 哎呀，这么多依赖项！

我必须手动创建这么多依赖项是正常的吗？不。使用具有这种功能的依赖注入组件或服务容器是常见的。再次，Symfony 来拯救我们，然而，你也可以检查 [PHP-DI 4](http://php-di.org/)。

让我们看看在将 Symfony 服务容器组件应用于我们的应用程序后，列表 14 中产生的代码：

```php
class IdeaController extends ContainerAwareController
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $useCase = $this->get('rate_idea_use_case');
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect('/idea/' . $response->idea->getId());
    }
}

```

控制器已被修改以访问容器，这就是为什么它继承自一个新的基控制器 `ContainerAwareController`，该控制器有一个 `get` 方法来检索每个包含的服务：

```php
<?xml version="1.0" ?>
<container 

    xsi:schemaLocation="
        http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <service
            id="rate_idea_use_case"
            class="RateIdeaUseCase">
            <argument type="service" id="idea_repository" />
        </service>

        <service
            id="idea_repository"
            class="RedisIdeaRepository">
            <argument type="service">
                <service class="Predis\Client" />
            </argument>
        </service>
    </services>
</container>

```

在列表 15 中，你还可以找到用于配置服务容器的 XML 文件。它真的很容易理解，但如果你需要更多信息，请查看 Symfony 服务容器组件[网站](http://symfony.com/doc/current/book/service_container.html)。

# 领域服务和通知六角边缘

我们是不是忘记了什么？*作者应该通过电子邮件通知*，是的！这是真的。让我们看看在列表 16 中我们是如何更新 UseCase 来完成这项工作的：

```php
class RateIdeaUseCase
{
    private $ideaRepository;
    private $authorNotifier;

    public function __construct(
        IdeaRepository $ideaRepository,
        AuthorNotifier $authorNotifier
    ) {
        $this->ideaRepository = $ideaRepository;
        $this->authorNotifier = $authorNotifier;
    }

    public function execute(RateIdeaRequest $request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;

        try {
            $idea = $this->ideaRepository->find($ideaId);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        if (!$idea) {
            throw new IdeaDoesNotExistException();
        }

        try {
            $idea->addRating($rating);
            $this->ideaRepository->update($idea);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        try {
            $this->authorNotifier->notify(
                $idea->getAuthor()
            );
        } catch(Exception $e) {
            throw new NotificationNotSentException();
        }

        return $idea;
    }
}

```

正如你所意识到的那样，我们添加了一个新参数来传递`AuthorNotifier`服务，该服务将发送电子邮件给作者。这是*端口*在*端口和适配器*命名中的*端口*。我们还更新了执行方法中的业务规则。

存储库不是唯一可能访问你的基础设施的对象，应该使用接口或抽象类进行解耦。领域服务也可以。当你的领域中有一个行为不是由一个实体明确拥有时，你应该创建一个领域服务。一个典型的模式是编写一个具有一些具体实现和一些其他抽象方法的抽象领域服务，这些方法将由*适配器*实现。

作为一项练习，定义`AuthorNotifier`抽象服务的实现细节。选项有 SwiftMailer 或直接使用`mail`调用。这取决于你。

# 让我们回顾一下

为了拥有一个*清晰的架构*，帮助你创建易于编写和测试的应用程序，我们可以使用六角架构。为了实现这一点，我们将用户故事的业务规则封装在 UseCase 或 Interactor 对象中。我们根据框架请求构建 UseCase 请求，实例化 UseCase 及其所有依赖项，然后执行它。我们得到响应并根据它采取相应的行动。如果我们的框架有一个依赖注入组件，你可以使用它来简化代码。

同样的 UseCase 对象可以从不同的*交付机制*中使用，以便用户可以从不同的客户端（Web、API、控制台等）访问功能。

对于测试，可以玩一些模拟所有定义的接口的模拟，这样也可以覆盖特殊案例或错误流程。享受这份出色的工作吧。

# 六角架构

在几乎所有的博客和书中，你都会找到关于软件不同区域的同心圆的插图。正如罗伯特·C·马丁在他的*Clean Architecture*帖子中解释的那样，外圈是你的基础设施所在的地方。内圈是你的实体所在的地方。使这种架构发挥作用的主要规则是**依赖规则**。这个规则说，源代码依赖只能指向内。内圈中的任何东西都不能知道外圈中的任何东西。

# 关键点

如果 100%的单元测试覆盖率对你的应用程序非常重要，或者你想要能够切换你的存储策略、Web 框架或其他任何类型的第三方代码，请使用这种方法。这种架构对于需要跟上不断变化的需求的长期应用程序特别有用。

# 接下来是什么？

如果你想要了解更多关于六边形架构以及其他相关概念，你应该回顾文章开头提供的相关网址，看看 CQRS 和事件源。此外，别忘了订阅关于领域驱动设计（DDD）的谷歌群组和 RSS，例如[`dddinphp.org`](http://dddinphp.org)，并在 Twitter 上关注像`@VaughnVernon`和`@ericevans0`这样的人。
