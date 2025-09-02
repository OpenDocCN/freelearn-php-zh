# 第十章：仓库

为了与领域对象交互，你需要持有它的引用。实现这一目标的一种方式是通过创建。或者，你也可以遍历一个关联。在面向对象程序中，对象有链接（引用）到其他对象，这使得它们易于遍历，从而增加了我们模型的表达能力。但是，这里有一个问题：你需要一种机制来检索第一个对象，即聚合根。

仓库充当存储位置，检索的对象以持久化时的相同状态返回。在领域驱动设计中，每个聚合（Aggregate）类型通常都有一个唯一的关联仓库，用于其持久化和获取需求。然而，在需要共享聚合对象层次结构的情况下，类型可能共享一个仓库。

一旦你成功从仓库中检索到聚合，你做的任何更改都会被持久化，这样就消除了返回到仓库的需要。

# 定义

马丁·福勒将仓库（Repository）定义为：

领域和数据映射层之间的机制，充当内存中的领域对象集合。客户端对象以声明性方式构建查询规范，并将其提交给仓库以满足。对象可以被添加到仓库中，也可以从简单的对象集合中移除，仓库封装的映射代码将在幕后执行适当的操作。从概念上讲，仓库封装了在数据存储中持久化的对象集合及其上的操作，提供了对持久化层的面向对象视图。仓库还支持实现领域和数据映射层之间干净分离和单向依赖的目标。

# 仓库不是 DAO

**数据访问对象（Data Access Objects**）(**DAO**)是将领域对象持久化到数据库的常见模式。很容易将 DAO 模式与仓库混淆。显著的差异在于，仓库代表集合，而 DAO 更接近数据库，通常是更以表为中心的。通常，DAO 会包含特定领域对象的 CRUD 方法。让我们看看一个 DAO 的常见接口可能是什么样的：

```php
interface UserDAO 
{ 
    /** 
     * @param string $username 
     * @return User 
     */ 
    public function get($username);

    public function create(User $user);

    public function update(User $user);

    /**
     * @param string $username
     */
    public function delete($username);
}

```

DAO 接口可能有多个实现，这些实现可能从使用 ORM 构造到使用纯 SQL 查询。DAO 的主要问题在于它们的职责定义不明确。DAO 通常被视为数据库的网关，因此为了查询数据库，很容易通过许多特定方法来大大降低内聚性：

```php
interface BloatUserDAO 
{ 
    public function get($username);

    public function create(User $user);

    public function update(User $user);

    public function delete($username);

    public function getUserByLastName($lastName);

    public function getUserByEmail($email);

    public function updateEmailAddress($username, $email);

    public function updateLastName($username, $lastName);
}

```

正如你所见，我们添加的新方法越多，单元测试 DAO 就越困难，它与用户对象耦合得也越来越紧密。随着时间的推移，这个问题会逐渐扩大，许多其他贡献者共同使这个大泥球（Big Ball of Mud）变得更大。

# 集合导向的仓库

仓库通过实现它们的通用接口特征来模拟集合。作为一个集合，仓库不应该泄露任何持久化行为的意图，例如保存到存储的概念。

基础的持久化机制必须支持这一需求。你不需要在对象的整个生命周期中处理对象的变更。集合引用了对象的最新的变更，这意味着在每次访问时，你都会得到最新的对象状态。

仓库实现了一个具体的集合类型，即 Set。Set 是一种数据结构，具有一个不变性，即不包含重复条目。如果你尝试向 Set 中添加一个已经存在的元素，它将不会被添加。这在我们的用例中很有用，因为每个聚合体都有一个与根实体关联的唯一标识符。

例如，考虑以下领域模型：

```php
namespace Domain\Model;

class Post
{
    const EXPIRE_EDIT_TIME = 120; // seconds

    private $id;
    private $body;
    private $createdAt;

    public function __construct(PostId $anId, Body $aBody) 
    {
        $this->id = $anId;
        $this->body = $aBody;
        $this->createdAt = new \DateTimeImmutable();
    }

    public function editBody(Body $aNewBody)
    {
        if($this->editExpired()) {
            throw new RuntimeException('Edit time expired');
        }

        $this->body = $aNewBody;
    }

    private function editExpired()
    {
        $expiringTime= $this->createdAt->getTimestamp() +
            self::EXPIRE_EDIT_TIME;

        return $expiringTime < time();
    }

    public function id()
    {
        return $this->id;
    }

    public function body()
    {
       return $this->body;
    }

    public function createdAt()
    {
       return $this->createdAt;
    }
}

class Body
{
    const MIN_LENGTH = 3;
    const MAX_LENGTH = 250;

    private $content;

    public function __construct($content)
    {
        $this->setContent(trim($content));
    }

    private function setContent($content)
    {
        $this->assertNotEmpty($content);
        $this->assertFitsLength($content);

        $this->content = $content;
    }

    private function assertNotEmpty($content)
    {
        if(empty($content)) {
            throw new DomainException('Empty body');
        }
    }

    private function assertFitsLength($content)
    {
        if(strlen($content) < self::MIN_LENGTH) {
            throw new DomainException('Body is too short');
        }

        if(strlen($content) > self::MAX_LENGTH) {
            throw new DomainException('Body is too long');
        }
    }

    public function content()
    {
        return $this->content;
    }
}

class PostId
{
    private $id;

    public function __construct($id = null)
    {
        $this->id = $id ?: uniqid();
    }

    public function id()
    {
        return $this->id;
    }

    public function equals(PostId $anId)
    {
       return $this->id === $anId->id();
    }
}

```

如果我们想要持久化这个`Post`实体，可以创建一个简单的内存`Post`仓库，如下所示：

```php
class SimplePostRepository 
{   
    private $post = [];

    public add(Post $aPost)
    {
        $this->posts[(string) $aPost->id()] = $aPost;
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[(string) $anId])) {
            return $this->posts[(string) $anId];
        }

        return null;
    }
}

```

并且，正如你所期望的，它被当作一个集合来处理：

```php
$id = new PostId();
$repository = new SimplePostRepository();
$repository->add(new Post($id, 'Random content'));

// later ...
$post = $repository->postOfId($id);
$post->editBody('Updated content');

// even later ...
$post = $repository->postOfId($id);
assert('Updated content' === $post->body());

```

如你所见，从集合的角度来看，在仓库中不需要保存方法。影响对象的变化被底层持久化层正确处理。集合导向的仓库是那些不需要添加之前已持久化的聚合体的仓库。这主要发生在基于内存的仓库中，但我们也有方法在持久化导向的仓库中这样做。我们稍后会看到这一点；此外，我们将在第十一章，*应用*中更深入地探讨这一点。

设计仓库的第一步是为它定义一个类似集合的接口。该接口需要定义通常的集合方法，如下所示：

```php
interface PostRepository 
{ 
    public function add(Post $aPost);
    public function addAll(array $posts); 
    public function remove(Post $aPost); 
    public function removeAll(array $posts); 
    // ... 
}

```

对于实现这样的接口，你也可以使用一个抽象类。一般来说，当我们谈论接口时，我们指的是一般概念，而不仅仅是特定的 PHP 接口。为了保持你的设计简单，不要添加你不需要的方法；仓库接口定义及其相应的聚合体应该放在同一个模块中。

有时`remove`操作并不会在数据库中实际删除聚合体。这种策略——其中聚合体有一个状态字段被更新为*已删除*的值——被称为*软删除*。为什么这种方法有趣？它对于审计变更和性能来说可能很有趣。在这种情况下，你可以将聚合体标记为禁用或*逻辑删除*。接口可以根据需要相应地更新，通过移除删除方法或在仓库中提供禁用行为。

仓库的另一个重要方面是查找方法，如下所示：

```php
interface PostRepository 
{ 
    // ... 

    /**
     * @return Post
     */
    public function postOfId(PostId $anId);

    /**
     * @return Post[]
     */
    public function latestPosts(DateTimeImmutable $sinceADate);
}

```

正如我们在 第四章 中所建议的，*实体*，我们更喜欢应用程序生成的标识符。为聚合生成新标识符的最佳位置是其存储库。因此，为了检索 `Post` 的全局唯一 ID，一个合理的做法是在 `PostRepository` 中包含它：

```php
interface PostRepository
{ 
    // ...

    /**
     * @return PostId
     */
    public function nextIdentity();
}

```

负责构建每个 `Post` 实例的代码调用 `nextIdentity` 来获取一个唯一标识符，`PostId`：

```php
$post = newPost($postRepository->nextIdentity(), $body);

```

一些开发者倾向于将实现放置在接口定义附近，作为模块的子包。然而，因为我们希望有一个清晰的关注点分离，所以我们建议将其放置在基础设施层内部。

# 内存实现

如 Uncle Bob 在 [Screaming Architecture](http://blog.8thlight.com/uncle-bob/2011/09/30/Screaming-Architecture.html) 中所写：

一个好的软件架构允许将关于框架、数据库、Web 服务器和其他环境问题和工具的决定推迟和延迟。一个好的架构使得在项目后期才决定 Rails、Spring、Hibernate、Tomcat 或 MySql 成为可能。一个好的架构还使得改变这些决定变得容易。一个好的架构强调使用案例，并将它们与外围关注点解耦。

在应用程序的早期阶段，一个快速的内存实现可能会很有用。你可以用它来成熟系统的其他部分，允许你将数据库决策推迟到正确的时间点。内存存储库简单、快速且易于实现。

对于我们的 `Post` 存储库，一个内存中的哈希表就足以提供我们需要的所有功能：

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class InMemoryPostRepository implements PostRepository
{
    private $posts = [];

    public function add(Post $aPost)
    {
       $this->posts[$aPost->id()->id()] = $aPost;
    }

    public function remove(Post $aPost)
    {
        unset($this->posts[$aPost->id()->id()]);
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[$anId->id()])) {
            return $this->posts[$anId->id()];
        }

        return null;
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        return $this->filterPosts(
           function (Post $post) use($sinceADate) {
               return $post->createdAt() > $sinceADate;
           }
        );
    }

    private function filterPosts(callable $fn)
    {
        return array_values(array_filter($this->posts, $fn));
    }

    public function nextIdentity()
    {
         return new PostId();
    }
}

```

# Doctrine ORM

我们在过去的章节中已经多次提到了 [Doctrine](http://www.doctrine-project.org/)。Doctrine 是一套用于数据库存储和对象映射的库。它默认捆绑了流行的 [Symfony2 网络框架](http://symfony.com/)，并且，在众多特性中，它允许你通过 [数据映射模式](http://martinfowler.com/eaaCatalog/dataMapper.html)轻松地将你的应用程序与持久层解耦。

同时，ORM 位于一个强大的数据库抽象层之上，它通过一个称为 **Doctrine 查询语言**（**DQL**）的 SQL 语法来实现数据库交互，该语言受到了著名的 Java Hibernate 框架的启发。

如果我们要使用 Doctrine ORM，第一个任务是完成通过 [Composer](https://getcomposer.org/) 向我们的项目中添加依赖项：

```php
composer require doctrine/orm 

```

# 对象映射

您的领域对象与数据库之间的映射可以被视为实现细节。领域生命周期不应该意识到这些持久性细节。因此，映射信息应该作为基础设施层的一部分来定义，位于领域之外，并且作为存储库的实现。

# Doctrine 自定义映射类型

由于我们的 `Post` 实体由像 `Body` 或 `PostId` 这样的值对象组成，因此制作自定义映射类型或使用 Doctrine Embeddables 是一个好主意，正如在值对象章节中看到的那样。这将使对象映射变得容易得多：

```php
namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\Body;

class BodyType extends Type
{
    public function getSQLDeclaration(
        array $fieldDeclaration, AbstractPlatform $platform
    ) {
        return $platform->getVarcharTypeDeclarationSQL(
            $fieldDeclaration
        );
    }

    /**
     * @param string $value
     * @return Body
     */
    public function convertToPHPValue(
        $value, AbstractPlatform $platform
    ) {
        return new Body($value);
    }

    /**
     * @param Body $value
     */
    public function convertToDatabaseValue(
        $value, AbstractPlatform $platform
    ) {
        return $value->content();
    }

    public function getName()
    {
        return 'body';
    }
}

namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\PostId;

class PostIdType extends Type
{
    public function getSQLDeclaration(
        array $fieldDeclaration, AbstractPlatform $platform
    ) {
        return $platform->getGuidTypeDeclarationSQL(
            $fieldDeclaration
        );
    }

    /**
     * @param string $value
     * @return PostId
     */
    public function convertToPHPValue(
        $value, AbstractPlatform $platform
    ) {
        return new PostId($value);
    }

    /**
     * @param PostId $value
     */
    public function convertToDatabaseValue(
        $value, AbstractPlatform $platform
    ) {
       return $value->id();
    }

    public function getName()
    { 
       return 'post_id';
    }
}

```

不要忘记在 `PostId` 值对象上实现 `__toString` 魔法方法，因为 Doctrine 需要这个：

```php
class PostId 
{ 
    // ...   
    public function __toString()
    {
        return $this->id;
    }
}

```

Doctrine 提供了多种映射格式，如 YAML、XML 或注解。XML 是我们的首选选择，因为它提供了强大的 IDE 自动完成：

```php
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping

    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
    http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity name="Domain\Model\Post" table="posts">
        <id name="id" type="post_id" column="id">
            <generator strategy="NONE" />
        </id>
        <field name="body" type="body" length="250" column="body"/>
        <field name="createdAt" type="datetime" column="created_at"/>
    </entity>

</doctrine-mapping>

```

练习

将使用 Doctrine Embeddables 方法的情况下映射看起来会是什么样子写下来。如果你需要一些帮助，请查看第 值对象或实体章节。

# 实体管理器

`EntityManager` 是 ORM 功能的中心访问点。启动它很容易：

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;

Type::addType(
    'post_id',
    'Infrastructure\Persistence\Doctrine\Types\PostIdType'
);

Type::addType(
    'body',
    'Infrastructure\Persistence\Doctrine\Types\BodyType'
);

$entityManager = EntityManager::create(
    [
        'driver' => 'pdo_sqlite',
        'path'=> __DIR__ . '/db.sqlite',
    ],
    Tools\Setup::createXMLMetadataConfiguration(
         ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
         $devMode = true
    )
);

```

记得根据你的需求和设置进行配置。

# DQL 实现

在这个仓储的情况下，我们只需要 `EntityManager` 直接从数据库中检索领域对象：

```php
namespace Infrastructure\Persistence\Doctrine;

use Doctrine\ORM\EntityManager;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class DoctrinePostRepository implements PostRepository
{
    protected $em;

    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }
    public function add(Post $aPost)
    {
        $this->em->persist($aPost);
    }

    public function remove(Post $aPost)
    {
        $this->em->remove($aPost);
    }

    public function postOfId(PostId $anId)
    {
        return $this->em->find('Domain\Model\Post', $anId);
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
       return $this->em->createQueryBuilder()
           ->select('p')
           ->from('Domain\Model\Post', 'p')
           ->where('p.createdAt > :since')
           ->setParameter(':since', $sinceADate)
           ->getQuery()
           ->getResult();
    } 

    public function nextIdentity()
    {
        return new PostId();
    }
}

```

如果你检查一些 Doctrine 示例，你可能会发现运行持久化或删除后应该调用 `flush`。但是，正如我们的提案中看到的，没有调用 `flush`。刷新和处理事务被委托给应用程序服务。这就是为什么你可以使用 Doctrine，考虑到所有实体的更改将在请求结束时刷新。从性能的角度来看，一个刷新调用是最好的。

# 面向持久化的仓储

有时候，面向集合的仓储与我们的持久化机制不太匹配。如果你没有工作单元，跟踪聚合变化是一项艰巨的任务。唯一持久化这些变化的方法是显式调用 `save`。

面向持久化的仓储的接口定义类似于你定义面向集合的等效仓储：

```php
 interface PostRepository
 {
     public function nextIdentity();
     public function postOfId(PostId $anId);
     public function save(Post $aPost);
     public function saveAll(array $posts);
     public function remove(Post $aPost);
     public function removeAll(array $posts);
 }

```

在这种情况下，我们现在有保存和 `saveAll` 方法，它们提供了类似于之前添加和 `addAll` 方法的功能。然而，重要的区别在于客户端如何使用它们。在面向集合的风格中，你只需使用一次添加方法：当聚合创建时。在面向持久化的风格中，你不仅在创建新的聚合后使用 `save` 动作，而且在现有的聚合被修改时也会使用：

```php
 $post = new Post(/* ... */);
 $postRepository->save($post);

 // later ...
 $post = $postRepository->postOfId($postId);
 $post->editBody(new Body('New body!'));
 $postRepository->save($post);

```

除了这个区别之外，细节仅在于实现中。

# Redis 实现

[Redis](http://redis.io/) 是一个内存中的键值存储，可以用作缓存或存储。

根据情况，我们可以考虑将 Redis 作为我们的聚合存储。

要开始，确保你有一个 PHP 客户端来连接 Redis。我们推荐的一个好的是 [Predis](https://github.com/nrk/predis)：

```php
composer require predis/predis:~1.0
namespace Infrastructure\Persistence\Redis;

use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;
use Predis\Client;

class RedisPostRepository implements PostRepository
{
    private $client;

    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function save(Post $aPost)
    {
        $this->client->hset(
            'posts',
            (string) $aPost->id(), serialize($aPost)
        );
    }

    public function remove(Post $aPost)
    {
        $this->client->hdel('posts', (string) $aPost->id());
    }

    public function postOfId(PostId $anId)
    {
       if($data = $this->client->hget('posts', (string) $anId)) {
          return unserialize($data);
       }

       return null;
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        $latest = $this->filterPosts(
            function(Post $post) use ($sinceADate) {
                return $post->createdAt() > $sinceADate;
            }
        );

        $this->sortByCreatedAt($latest);

        return array_values($latest);
    }

    private function filterPosts(callable $fn)
    {
        return array_filter(array_map(function ($data) {
            return unserialize($data);
        },

        $this->client->hgetall('posts')), $fn);
    }

    private function sortByCreatedAt(&$posts)
    {
        usort($posts, function (Post $a, Post $b) {
            if ($a->createdAt() == $b->createdAt()) {
                return 0;
            }
            return ($a->createdAt() < $b->createdAt()) ? -1 : 1;
        });
    }

    public function nextIdentity()
    {
        return new PostId();
    }
 }

```

# SQL 实现

在一个经典示例中，我们可以通过使用纯 SQL 查询为我们的 `PostRepository` 创建一个简单的 [PDO](http://php.net/manual/en/book.pdo.php) 实现：

```php
namespace Infrastructure\Persistence\Sql;

use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class SqlPostRepository implements PostRepository
{
    const DATE_FORMAT = 'Y-m-d H:i:s';

    private $pdo;

    public function __construct(\PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function save(Post $aPost)
    {
        $sql ='INSERT INTO posts ' .
            '(id, body, created_at) VALUES ' .
            '(:id, :body, :created_at)';

        $this->execute($sql, [
            'id' => $aPost->id()->id(),
            'body' => $aPost->body()->content(),
            'created_at' => $aPost->createdAt()->format(
                self::DATE_FORMAT
            )
        ]);
    }

    private function execute($sql, array $parameters)
    {
        $st = $this->pdo->prepare($sql);

        $st->execute($parameters);

        return $st;
    }

    public function remove(Post $aPost)
    {
        $this->execute('DELETE FROM posts WHERE id = :id', [
            'id' => $aPost->id()->id()
        ]);
    } 

    public function postOfId(PostId $anId)
    {
        $st =$this->execute('SELECT * FROM posts WHERE id = :id',[
            'id' => $anId->id()
        ]);

        if($row = $st->fetch(\PDO::FETCH_ASSOC)) {
            return $this->buildPost($row);
        }

        return null;
    }

    private function buildPost($row)
    {
        return new Post(
            new PostId($row['id']),
            new Body($row['body']),
            new \DateTimeImmutable($row['created_at'])
        );
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        return $this->retrieveAll(
            'SELECT * FROM posts WHERE created_at > :since_date', [
                'since_date' => $sinceADate->format(self::DATE_FORMAT)
            ]
        );
    }

    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);

        $st->execute($parameters);

        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }

    public function nextIdentity()
    {
        return new PostId();
    }

    public function size()
    {
        return $this->pdo->query('SELECT COUNT(*) FROM posts')
            ->fetchColumn();
    }
}

```

由于我们没有任何映射配置，在同一个类中为模式提供一个初始化方法将非常有用。**共同变化的事物应该保持在一起**：

```php
class SqlPostRepository implements PostRepository
{
    // ...
    public function initSchema()
    {
        $this->pdo->exec(<<<SQL
DROP TABLE IF EXISTS posts;

CREATE TABLE posts (
    id CHAR(36) PRIMARY KEY,
    body VARCHAR (250) NOT NULL,
    created_at DATETIME NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
        SQL);
    }
}

```

# 额外行为

```php
interface PostRepository 
{  
    // ... 
    public function size(); 
}

```

实现可能看起来像这样：

```php
class DoctrinePostRepository implements PostRepository
{
    // ...

    public function size()
    {
        return $this->em->createQueryBuilder()
            ->select('count(p.id)')
            ->from('Domain\Model\Post', 'p')
            ->getQuery()
            ->getSingleScalarResult();
    }
} 

```

向存储库添加额外行为可能非常有益。一个例子是能够计算给定集合中所有项的能力。你可能会想到添加一个名为 count 的方法；然而，由于我们试图模仿集合，更好的名称应该是 size：

你还能够在存储库中放置特定的计算、计数器、读取优化查询或复杂命令（`INSERT`、`UPDATE`或`DELETE`）。然而，所有行为仍然应该遵循存储库的集合特征。你被鼓励尽可能地将逻辑移动到领域特定的无状态领域服务中，而不是简单地将这些责任添加到存储库中。

在某些情况下，你可能不需要整个聚合来简单地访问少量信息。为了解决这个问题，你可以添加存储库方法来作为快捷方式访问这些信息。你应该确保只访问可以通过聚合根导航检索到的数据。因此，你不应该允许访问聚合根的私有和内部区域，因为这会违反既定的合同协议。

对于某些用例，你可能需要非常具体的查询，这些查询是多个聚合类型的组合，每个返回特定的信息。这些查询可以运行并作为单个值对象返回。存储库返回值对象是非常常见的。

如果你发现自己正在创建许多用例最优的查找方法，你可能引入了一个常见的代码异味。这可能表明对聚合边界的判断有误。然而，如果你确信边界是正确的，那么可能是时候探索 CQRS 了。

# 查询存储库

在比较时，如果考虑到它们的查询能力，存储库与集合不同。存储库处理大量对象，这些对象在执行查询时通常不在内存中。在内存中加载域对象的全部实例并对其执行查询是不切实际的。

一个好的解决方案是传递一个标准并让存储库处理实现细节以成功执行操作。它可能将标准转换为 SQL 或 ORM 查询，或者遍历内存中的集合。然而，这并不重要，因为实现会处理它。

# 规范模式

对于标准对象的一个常见实现是规范模式。规范是一个简单的谓词，它接受一个域对象并返回一个布尔值。给定一个域对象，如果它指定了规范，则返回 true，否则返回 false：

```php
interface PostSpecification
{
    /**
     * @return boolean
     */
    public function specifies(Post $aPost);
}

```

我们只需要给我们的存储库添加一个`query`方法：

```php
interface PostRepository
{
    // ...
    public function query($specification);
}

```

# 内存实现

例如，如果我们想通过使用内存实现的规范来在我们的 `PostRepository` 中复制 `latestPosts` 查询方法，它可能看起来像这样：

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;

interface InMemoryPostSpecification
{
    /**
     * @return boolean
     */
     public function specifies(Post $aPost);
}

```

`latestPosts` 行为的内存实现可能看起来像这样：

```php
namespace Infrastructure\Persistence\InMemory;
use Domain\Model\Post;

class InMemoryLatestPostSpecification
    implements InMemoryPostSpecification
{
    private $since;

    public function __construct(\DateTimeImmutable $since)
    {
       $this->since = $since;
    }

    public function specifies(Post $aPost)
    {
      return $aPost->createdAt() > $this->since;
    }
}

```

我们存储库实现的 `query` 方法可能看起来像这样：

```php
class InMemoryPostRepository implements PostRepository
{
    // ...

    /**
     * @param InMemoryPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->filterPosts(
            function (Post $post) use($specification) {
                return $specification->specifies($post);
            }
        );
    }
}

```

从存储库中检索所有最新帖子就像创建上述实现的一个定制实例一样简单：

```php
$latestPosts = $postRepository->query(
    new InMemoryLatestPostSpecification(new \DateTimeImmutable('-24'))
);

```

# SQL 实现

标准规范对于内存实现来说效果很好。然而，由于我们不会预先在内存中加载所有领域对象，对于 SQL 实现，我们需要一个更具体的规范来处理这些情况：

```php
namespace Infrastructure\Persistence\Sql;

interface SqlPostSpecification
{
    /**
     * @return string
     */
    public function toSqlClauses();
}

```

此规范的 SQL 实现可能看起来像这样：

```php
namespace Infrastructure\Persistence\Sql;

class SqlLatestPostSpecification implements SqlPostSpecification
{
    private $since;

    public function __construct(\DateTimeImmutable $since)
    {
        $this->since = $since;
    }

    public function toSqlClauses()
    {
        return "created_at >'" .
            $this->since->format('Y-m-d H:i:s') .
            "'";
    }
}

```

下面是一个如何查询 `SQLPostRepository` 实现的例子：

```php
class SqlPostRepository implements PostRepository
{
    // ...

    /**
     * @param SqlPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->retrieveAll(
            'SELECT * FROM posts WHERE ' .
                $specification->toSqlClauses()
        );
    }

    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);

        $st->execute($parameters);

        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }
}

```

# 管理事务

领域模型不是管理事务的地方。在领域模型上应用的操作应该对持久化机制无感知。解决此问题的常见方法是在应用程序层放置一个 [外观模式](http://en.wikipedia.org/wiki/Facade_pattern)，从而将相关的用例组合在一起。当从 UI 层调用外观方法时，业务方法开始一个事务。一旦完成，外观通过提交事务结束交互。如果发生任何错误，事务将被回滚：

```php
use Doctrine\ORM\EntityManager;

class SomeApplicationServiceFacade
{
    private $em;

    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }

    public function doSomeUseCaseTask()
    {
        try {
            $this->em->getConnection()->beginTransaction();
            // Use domain model

            $this->em->getConnection()->commit();
        } catch (Exception $e) {
             $this->em->getConnection()->rollback();
             throw $e;
        }
    }
} 

```

使用外观（Facades）引入的问题是我们必须反复重复相同的样板代码。如果我们统一执行用例的方式，我们可以使用 [装饰器模式](http://en.wikipedia.org/wiki/Decorator_pattern) 将它们包装在事务中：

```php
interface ApplicationService
{
   /**
    * @param $request
    * @return mixed
    */
    public function execute(BaseRequest $request);
}

class SomeApplicationService implements ApplicationService
{
    public function execute(BaseRequest $request)
    {
       // do something
    }
}

```

我们不希望将应用程序层与具体的交易程序耦合起来，因此我们可以为它创建一个简单的接口：

```php
interface TransactionalSession
{
    /**
     * @param callable $operation
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}

```

一个可以将任何应用程序服务转换为事务性的装饰器模式实现就像这样简单：

```php
class TransactionalApplicationService implements ApplicationService
{
    private $session;
    private $service;

    public function __construct(
        ApplicationService $service,
        TransactionalSession $session
    ) {
        $this->session = $session;
        $this->service = $service;
    }

    public function execute(BaseRequest $request)
    {
        $operation = function() use($request) {
            return $this->service->execute($request);
        };

        return $this->session->executeAtomically(
            $operation->bindTo($this)
        );
    }
}

```

在此之后，我们可以选择创建一个 Doctrine 事务会话实现：

```php
class DoctrineSession implements TransactionalSession
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function executeAtomically(callable $operation)
    {
        return $this->entityManager->transactional($operation);
    }
}

```

现在我们已经拥有了在事务中执行用例所需的一切：

```php
$useCase = new TransactionalApplicationService(
    new SomeApplicationService(
        // ...
    ),
    new DoctrineSession(
       // ...
    )
);

$response = $useCase->execute();

```

# 测试存储库

为了确保存储库在生产环境中能够正常工作，我们需要测试其实现。为此，我们必须测试系统的边界，确保我们的预期是正确的。

在 Doctrine 测试的情况下，设置将稍微复杂一些：

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;
use Domain\Model\Post;

class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    private $postRepository;

    public function setUp()
    {
        $this->postRepository = $this->createPostRepository();
    }

    private function createPostRepository()
    {
        $this->addCustomTypes();
        $em = $this->initEntityManager();
        $this->initSchema($em);

        return new PrecociousDoctrinePostRepository($em);
    }

    private function addCustomTypes()
    {
        if (!Type::hasType('post_id')) {
            Type::addType(
                'post_id',
                'Infrastructure\Persistence\Doctrine\Types\PostIdType'
            );
        }

        if (!Type::hasType('body')) {
            Type::addType(
                'body',
                'Infrastructure\Persistence\Doctrine\Types\BodyType'
            );
        }
    }

    protected function initEntityManager()
    {
        return EntityManager::create(
            ['url' => 'sqlite:///:memory:'],
            Tools\Setup::createXMLMetadataConfiguration(
                ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
                $devMode = true
            )
        );
    }

    private function initSchema(EntityManager $em)
    {
        $tool = new Tools\SchemaTool($em);
        $tool->createSchema([
            $em->getClassMetadata('Domain\Model\Post')
        ]);
    }

    // ...
}

class PrecociousDoctrinePostRepository extends DoctrinePostRepository
{
    public function persist(Post $aPost)
    {
        parent::persist($aPost);

        $this->em->flush();
    }

    public function remove(Post $aPost)
    {
        parent::remove($aPost);

        $this->em->flush();
    }
}

```

一旦我们设置了此环境，我们就可以继续测试存储库的行为：

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldRemovePost()
    {
        $post = $this->persistPost('irrelevant body');

        $this->postRepository->remove($post);

        $this->assertPostExist($post->id());
    }

    private function assertPostExist($id)
    {
        $result = $this->postRepository->postOfId($id);
        $this->assertNull($result);
    }

    private function persistPost(
        $body,
        \DateTimeImmutable $createdAt = null
    ) {
        $this->postRepository->add(
            $post = new Post(
                $this->postRepository->nextIdentity(),
                new Body($body),
                $createdAt
            )
        );

        return $post;
    }
}

```

根据我们之前的断言，如果我们保存一个 `Post`，我们期望在完全相同的状态中找到它。

现在我们可以继续测试通过指定一个给定日期来查找最新帖子：

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldFetchLatestPosts()
    {
        $this->persistPost(
            'a year ago', new \DateTimeImmutable('-1 year')
        );
        $this->persistPost(
            'a month ago', new \DateTimeImmutable('-1 month')
        );
        $this->persistPost(
            'few hours ago', new \DateTimeImmutable('-3 hours')
        );
        $this->persistPost(
            'few minutes ago', new \DateTimeImmutable('-2 minutes')
        );

        $posts = $this->postRepository->latestPosts(
            new \DateTimeImmutable('-24 hours')
        );

        $this->assertCount(2, $posts);
        $this->assertEquals(
            'few hours ago', $posts[0]->body()->content()
        );
        $this->assertEquals(
            'few minutes ago', $posts[1]->body()->content()
        );
    }
}

```

# 使用内存实现测试您的服务

设置一个完全持久的仓库实现可能很复杂，并可能导致执行缓慢。你应该关心保持你的测试快速。通过整个数据库设置然后查询会极大地减慢你的速度。拥有一个内存实现可以帮助延迟持久化决策直到最后。我们可以像以前一样进行测试，但这次，我们将使用一个功能齐全、快速简单的内存实现：

```php
class MyServiceTest extends \PHPUnit_Framework_TestCase
{
    private $service;

    public function setUp()
    {
        $this->service = new MyService(
            new InMemoryPostRepository()
        );
    }
}

```

# 总结

仓库是一种充当存储位置的机制。DAO 和仓库之间的区别在于，DAO 采用数据库优先的方法，通过许多低级方法查询数据库，从而降低内聚性。根据底层持久化机制的不同，我们看到了不同的仓库方法：

+   **面向集合的仓库**倾向于更纯粹地符合领域模型，即使它们持久化实体。从客户端的角度来看，一个面向集合的仓库看起来就像一个集合（集合）。在实体更新时，不需要对实体进行显式的持久化调用，因为仓库跟踪对象的变化。我们探讨了如何使用 Doctrine 作为此类仓库的底层持久化机制。

+   **面向持久化的仓库**需要显式的持久化调用，因为它们不跟踪对象变化。我们探讨了 Redis 和纯 SQL 的实现。

在这个过程中，我们发现规范（Specification）是一种帮助我们查询数据库而不牺牲灵活性和内聚性的模式。我们还研究了如何管理事务以及如何使用简单快速的内存仓库实现来测试我们的服务。
