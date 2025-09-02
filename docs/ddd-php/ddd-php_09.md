# 第九章：工厂

工厂是一个强大的抽象。它们帮助客户端与领域交互的细节解耦。客户端不需要知道如何构建复杂对象和聚合，因此可以使用工厂创建整个聚合，从而强制执行其不变性。

# 聚合根上的工厂方法

经典著作中定义的[工厂方法模式](https://en.wikipedia.org/wiki/Factory_method_pattern)，在《设计模式：可复用面向对象软件的基础》一书中，是一个创建型模式，它：

定义了一个创建对象的接口，但将其实例化类型的选择留给子类，创建在运行时延迟。

在聚合根中添加工厂方法隐藏了创建聚合的内部实现细节，这对于任何外部客户端都是不可见的。这也将聚合完整性的责任移回根。

在一个包含`User`实体和`Wish`实体的领域模型中，`User`充当聚合根。没有`User`就没有`Wish`。`User`实体应该管理其聚合。

将`Wish`的控制权移回`User`实体，是通过在聚合根中放置一个工厂方法来实现的：

```php
class User
{
    // ...

    public function makeWish(WishId $wishId, $email, $content)
    {
        $wish = new WishEmail(
            $wishId,
            $this->id(),
            $email,
            $content
        );

        DomainEventPublisher::instance()->publish(
            new WishMade($wishId)
        );

        return $wish;
    }
}

```

客户端不需要知道聚合根如何处理创建逻辑的内部细节：

```php
 $wish = $aUser->makeWish(
     $wishRepository->nextIdentity(),
     'user@example.com',
     'I want to be free!'
 );

```

# 强制不变性

聚合根中的工厂方法也是处理不变性的好地方。

在一个包含`Forum`和`Post`实体的领域模型中，其中`Post`是聚合根`Forum`的聚合部分，发布`Post`可能看起来像这样：

```php
class Forum
{
    // ...

    public function publishPost(PostId $postId, $content)
    {
        $post = new Post($this->id, $postId, $content);

        DomainEventPublisher::instance()->publish(
            new PostPublished($postId)
        );

        return $post;
    }
 }

```

与领域专家交谈后，我们得出结论，当`Forum`关闭时，不应发布`Post`。这是一个不变性，我们可以在创建`Post`时直接强制执行，从而防止领域状态不一致：

```php
class Forum
{
     // ...

    public function publishPost(PostId $postId, $content)
    {
        if ($this->isClosed()) {
            throw new ForumClosedException();
        }

        $post = new Post($this->id, $postId, $content);

        DomainEventPublisher::instance()->publish(
            new PostPublished($postId)
        );

        return $post;
    }
}

```

# 服务上的工厂

解耦创建逻辑也对我们服务的实现很有帮助。

# 构建规范

在我们的服务中使用规范可能是说明如何在服务中使用工厂的最佳例子。

考虑以下服务示例。给定来自外部世界的请求，我们希望根据系统中最新添加的`Posts`构建一个 feed：

```php
namespace Application\Service;

use Domain\Model\Post;
use Domain\Model\PostRepository;

class LatestPostsFeedService 
{
    private $postRepository;

    public function __construct(PostRepository $postRepository) 
    {
        $this->postRepository = $postRepository;
    }

    /**
     * @param LatestPostsFeedRequest $request
     */
    public function execute($request) 
    {
        $posts = $this->postRepository->latestPosts($request->since);

        return array_map(function(Post $post) {
            return [
                'id' => $post->id()->id(),
                'content' => $post->body()->content(),
                'created_at' => $post-> createdAt()
            ];
        }, $posts);
    }
}

```

仓库中的查找方法，如`latestPosts`，有一些限制，因为它们会无限期地向我们的仓库添加复杂性。正如我们在第十章中讨论的，*仓库*规范是一个更好的方法。

幸运的是，我们`PostRepository`中有一个不错的`query`方法，它与`Specifications`一起工作：

```php
class LatestPostsFeedService 
{
    // ...

    public function execute($request) 
    {
        $posts = $this->postRepository->query($specification);
    }
}

```

使用规范的具体实现是一个坏主意：

```php
class LatestPostsFeedService
{

    public function execute($request)
    {
        $posts = $this->postRepository->query(
            new SqlLatestPostSpecification($request->since)
        );
    }
}

```

将高级应用程序服务与低级规范实现耦合会混合层次并破坏关注点的分离。此外，这是一种将我们的服务耦合到具体基础设施实现中的非常糟糕的方式。你无法在 SQL 持久化解决方案之外使用此服务。如果我们想使用内存实现来测试我们的服务怎么办？

解决这个问题的方法是通过使用[抽象工厂模式](https://en.wikipedia.org/wiki/Abstract_factory_pattern)将规范创建与服务本身解耦。根据[OODesign.com](http://www.oodesign.com/abstract-factory-pattern.html)：

抽象工厂提供了一个创建一组相关对象的接口，而不需要明确指定它们的类。

由于我们可能有多个规范实现，我们首先需要为工厂创建一个接口：

```php
namespace Domain\Model;

interface PostSpecificationFactory
{
    public function createLatestPosts(DateTimeImmutable $since);
}

```

然后，我们需要为每个 `PostRepository` 实现创建工厂。例如，内存 `PostRepository` 实现的工厂可能看起来像这样：

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\PostSpecificationFactory;

class InMemoryPostSpecificationFactory
    implements PostSpecificationFactory
{
    public function createLatestPosts(DateTimeImmutable $since)
    {
        return new InMemoryLatestPostSpecification($since);
    }
}

```

一旦我们有一个集中化的创建逻辑位置，就很容易将其从服务中解耦：

```php
class LatestPostsFeedService
{
    private $postRepository;
    private $postSpecificationFactory;

    public function __construct(
        PostRepository $postRepository,
        PostSpecificationFactory $postSpecificationFactory
    ) {
        $this->postRepository = $postRepository;
        $this->postSpecificationFactory = $postSpecificationFactory;
    }

    public function execute($request)
    {
        $posts = $this->postRepository->query(
            $this->postSpecificationFactory->createLatestPosts(
                $request->since
            )
        );
    }
}

```

现在，通过内存 `PostRepository` 实现对服务进行单元测试相当简单：

```php
namespace Application\Service;

use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Infrastructure\Persistence\InMemory\InMemoryPostRepositor;

class LatestPostsFeedServiceTest extends PHPUnit_Framework_TestCase
{
    /**
     * @var \Infrastructure\Persistence\InMemory\InMemoryPostRepository
     */
    private $postRepository;

    /**
     * @var LatestPostsFeedService
     */
    private $latestPostsFeedService;

    public function setUp()
    {
        $this->latestPostsFeedService = new LatestPostsFeedService(
            $this->postRepository = new InMemoryPostRepository()
        );
    }

   /**
    * @test
    */
    public function shouldBuildAFeedFromLatestPosts()
    {
        $this->addPost(1, 'first', '-2 hours');
        $this->addPost(2, 'second', '-3 hours');
        $this->addPost(3, 'third', '-5 hours');

        $feed = $this->latestPostsFeedService->execute(
            new LatestPostsFeedRequest(
                 new \DateTimeImmutable('-4 hours')
            )
        );

        $this->assertFeedContains([
            ['id' => 1, 'content' => 'first'],
            ['id' => 2, 'content' => 'second']
        ], $feed);
    }

    private function addPost($id, $content, $createdAt)
    {
        $this->postRepository->add(new Post(
            new PostId($id),
            new Body($content),
            new \DateTimeImmutable($createdAt)
        ));
    }

    private function assertFeedContains($expected, $feed)
    {
        foreach ($expected as $index => $contents) {
            $this->assertArraySubset($contents, $feed[$index]);
            $this->assertNotNull($feed[$index]['created_at']);
        }
    }
}

```

# 构建聚合体

实体对持久化机制是无关的。你不希望将持久化细节耦合并污染你的实体。看看下一个应用程序服务：

```php
class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @param SignUpUserRequest $request
     */
    public function execute( $request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->userOfEmail($email);
        if (null !== $user) {
            throw new UserAlreadyExistsException();
        }

        $this->userRepository->persist(new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        ));

        return $user;
    }
}

```

想象一个像以下这样的 `User` 实体：

```php
class User
{
    private $userId;
    private $email;
    private $password;

    public function __construct(UserId $userId, $email, $password)
    {
        // ...
    }

    // ...
 }

```

假设我们想使用 Doctrine 作为我们的基础设施持久化机制。Doctrine 需要有一个作为普通字符串实例变量的 `id` 才能正常工作。在我们的实体中，`$userId` 是一个 `UserId` 值对象。仅仅因为 Doctrine 就在我们的 `User` 实体中添加一个额外的 `id` 会将我们的持久化机制与领域模型耦合。

我们在第四章中看到，通过在基础设施层中围绕我们的 `User` 实体创建一个包装器，我们可以通过代理 ID 解决这个问题：

```php
class DoctrineUser extends User
{
    private $surrogateUserId;

    public function __construct(UserId $userId, $email, $password)
    {
        parent:: __construct($userId, $email, $password);
        $this->surrogateUserId = $userId->id();
    }
}

```

由于在应用程序服务中创建 `DoctrineUser` 会使持久化层再次与领域模型耦合，我们需要通过抽象工厂将创建逻辑从服务中解耦。

我们可以通过在我们的领域创建一个接口来完成这个操作：

```php
interface UserFactory
{
    public function build(UserId $userId, $email, $password);
}

```

然后，我们将其实现在我们的基础设施层中：

```php
class DoctrineUserFactory implements UserFactory
{
    public function build(UserId $userId, $email, $password)
    {
        return new DoctrineUser($userId, $email, $password);
    }
}

```

一旦解耦，我们只需要将工厂注入到我们的应用程序服务中：

```php
class SignUpUserService
{
    private $userRepository;
    private $userFactory;

    public function __construct(
        UserRepository $userRepository,
        UserFactory $userFactory
    ) {
        $this->userRepository = $userRepository;
        $this->userFactory = $userFactory;
    }

    /**
     * @param SignUpUserRequest $request
     */
    public function execute($request)
    {
        // ...
        $user = $this->userFactory->build(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );
        $this->userRepository->persist($user);
        return $user;
    }
}

```

# 测试工厂

当你编写测试时，你会看到一个常见的模式。这是因为构建实体和复杂的聚合体可能是一个非常繁琐和重复的过程。不可避免地，复杂性和重复性将开始渗透到你的测试套件中。考虑以下实体：

```php
class Author
{
    private $username;
    private $email ;
    private $fullName;

    public function __construct(
        Username $aUsername,
        FullName $aFullName,
        Email $anEmail
    ) {
        $this->username = $aUsername;
        $this->email = $anEmail ;
        $this->fullName = $aFullName;
    }

    // ...
}

```

在你的系统中某个地方，你最终会得到一个看起来像这样的测试：

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe' ),
            new Email('john@doe.com' )
        );

        //do something with author
    }
}

```

边界内的服务共享诸如实体、聚合体和值对象等概念。想象一下，在测试中反复重复相同的构建逻辑是多么的杂乱。正如我们将看到的，将构建逻辑从测试中提取出来非常方便，并且可以防止重复。

# 对象母体

[对象母体](https://martinfowler.com/bliki/ObjectMother.html)是一个吸引人的名称，用于指代为你的测试创建固定配置的工厂。

与前面的例子类似，我们可以将重复的逻辑提取到对象母体中，以便在多个测试中复用：

```php
class AuthorObjectMother
{
    public static function createOne()
    {
        return new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe'),
            new Email('john@doe.com )
        );
    }
}

class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorObjectMother::createOne();
    }
}

```

你会注意到，随着测试和情况的增加，工厂将拥有更多的方法。

由于对象母体不够灵活，它们往往会迅速增加复杂性。幸运的是，有一个更灵活的替代方案适用于你的测试。

# 测试数据构建器

测试数据构建器只是普通的构建器，它们在测试套件中仅使用默认值，这样你就不必在特定的测试用例中指定无关的参数：

```php
class AuthorBuilder
{
    private $username;
    private $email ;
    private $fullName;

    private function __construct()
    {
        $this->username = new Username('johndoe');
        $this->email = new Email('john@doe.com');
        $this->fullName = new FullName('John', 'Doe');
    }

    public static function anAuthor()
    {
        return new self();
    }

    public function withFullName(FullName $aFullName)
    {
        $this->fullName = $aFullName;

        return $this;
    }

    public function withUsername(Username $aUsername)
    {
        $this->username = $aUsername;

        return $this;
    }

    public function withEmail(Email $anEmail)
    {
        $this->email = $anEmail ;

        return $this;
    }

    public function build()
    {
        return new Author($this->username, $this->fullName, $this->email);
    }
}

class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorBuilder::anAuthor()
            ->withEmail(new Email('other@email.com'))
            ->build();
    }
}

```

我们甚至可以将测试数据构建器组合起来构建更复杂的聚合体，例如一个`Post`：

```php
class Post
{
    private $id;
    private $author;
    private $body;
    private $createdAt;

    public function __construct(
        PostId $anId, Author $anAuthor, Body $aBody
    ) {
        $this->id = $anId;
        $this->author = $anAuthor;
        $this->body = $aBody;
        $this->createdAt = new DateTimeImmutable();
    }
}

```

让我们看看我们`Post`对应的测试数据构建器。我们可以复用`AuthorBuilder`来构建默认的`Author`：

```php
class PostBuilder
{
    private $postId;
    private $author;
    private $body;

    private function __construct()
    {
        $this->postId = new PostId();
        $this->author = AuthorBuilder::anAuthor()->build();
        $this->body = new Body('Post body');
    }

    public static function aPost()
    {
        return new self();
    }

    public function withAuthor(Author $anAuthor)
    {
        $this->author = $anAuthor;

        return $this;
    }

    public function withPostId(PostId $aPostId)
    {
        $this->postId = $aPostId;

        return $this;
    }

    public function withBody(Body $body)
    {
        $this->body = $body;

        return $this;
    }

    public function build()
    {
        return new Post($this->postId, $this->author, $this->body);
    }
}

```

这个解决方案现在足够灵活，可以覆盖任何测试用例，包括构建内部实体的可能性：

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $post = PostBuilder::aPost()
            ->withAuthor(AuthorBuilder::anAuthor()
            ->withUsername(new Username('other'))
                ->build())
            ->withBody(new Body('Another body'))
                ->build();

        //do something with the post
    }
}

```

# 总结

工厂是将构建逻辑从我们的业务逻辑中解耦的强大工具。工厂方法模式不仅通过将创建责任转移到聚合根来帮助，还可以强制执行领域不变性。在我们的服务中使用抽象工厂模式允许我们将领域逻辑与基础设施创建细节分离。一个常见的用例是规范及其相应的持久化实现。我们已经看到，工厂在我们的测试套件中也很有用。虽然我们可以将构建逻辑提取到对象母体工厂中，但测试数据构建器为我们的测试提供了更多的灵活性。
