# 第三章. 关联

在上一章中，我们学习了如何使用 Doctrine 注释向实体类添加映射信息。我们使用了 Doctrine 命令行工具提供的代码和数据库模式生成器，并创建了一个使用`EntityManager`类来创建、更新、删除和显示博客文章的极简主义博客软件。

在第三章中，我们将学习如何通过以下主题处理实体之间的关联：

+   开始使用 Doctrine 关联

+   理解注释系统中的@ManyToOne 和@OneToMany 注释

+   理解标签的@ManyToMany 注释

# 开始使用 Doctrine 关联

我们将使用注释指定 Doctrine 关联，以及其他映射信息（还支持其他方法，如 XML 和 YAML 配置文件。请参阅第二章，*实体和映射信息*）。Doctrine 支持以下关联类型：

+   **一对一**：一个实体与一个实体相关联

+   **多对一**：多个实体与一个实体相关联（仅适用于双向关联，始终是一对多关联的反向方）

+   **一对多**：一个实体与多个实体相关联

+   **多对多**：多个实体与多个实体相关联

关联可以是单向的或双向的。单向关联只有一个拥有方，而双向关联既有拥有方又有反向方。换句话说，它们可以解释如下：

+   单向关联只能以一种方式使用：相关实体可以从主实体中检索。例如，用户有关联地址。地址可以从用户中检索，但用户无法从地址中检索。

+   双向关联可以以两种方式使用：相关实体可以从主实体中检索，主实体可以从相关实体中检索。例如，用户有关联订单。订单可以从用户中检索，用户也可以从订单中检索。

Doctrine 只管理关联的拥有方。这意味着您始终需要设置拥有方；否则，如果您只设置关联的反向方，它将不会由`EntityManager`类持久化。

有一种简单的方法来识别双向关联的方向。拥有方必须具有`inversedBy`属性，而反向方必须具有`mappedBy`属性。这些属性指的是相关的实体类。

默认情况下，一对一和多对一关联在 SQL 级别上使用存储相关 ID 和外键的列进行持久化。多对多关联始终使用关联表。

Doctrine 会自动生成列和表的名称（如果适用）。可以使用`@JoinColumn`注释更改名称，并使用`@JoinTable`注释强制使用关联表。

# 理解注释系统中的@ManyToOne 和@OneToMany 注释

让我们从评论开始。我们博客的访问者应该能够对我们的帖子做出反应。我们必须创建一个新的`Comment` Doctrine 实体类型，存储读者的评论。`Comment`实体将与一个`Post`实体相关联。一个帖子可以有多条评论，一条评论与一个帖子相关联。

以下 E-R 图表示将使用映射信息生成的 MySQL 模式：

![理解注释系统中的@ManyToOne 和@OneToMany 注释](img/4104_03_01.jpg)

## 创建评论实体类（拥有方）

`Comment`实体具有以下四个属性：

+   `id`：这是评论的唯一标识符

+   `body`：这代表评论的文本

+   `publicationDate`：这是评论发布的日期

+   `post_id`：这代表与评论相关的帖子

这是`Comment`实体的第一个代码片段，包含有注释的属性。它必须放在`Comment.php`文件中，位于`src/Blog/Entity/`位置。

```php
<?php

namespace Blog\Entity;

use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Column;
use Doctrine\ORM\Mapping\ManyToOne;

/**
 * Comment entity
 *
 * @Entity
 */
class Comment
{
    /**
     * @var int
     *
     * @Id
     * @GeneratedValue
     * @Column(type="integer")
     */
    protected $id;
    /**
     * @var string
     *
     * @Column(type="text")
     */
    protected $body;
    /**
     * @var \DateTime
     *
     * @Column(type="datetime")
     */
    protected $publicationDate;
    /**
     * @var Post
     *
     * @ManyToOne(targetEntity="Post", inversedBy="comments")
     */
    protected $post;
}
```

这个实体类类似于第二章中创建的`Post`实体类，*实体和映射信息*。我们使用`@ManyToOne`注释在`Comment`和`Post`实体之间创建多对一关联。使用`targetEntity`属性指定相关实体类是必需的。

为了能够直接从`Post`实体中检索评论，这个关联必须是双向的。`inversedBy`属性将此关联标记为双向，并指示`Post`实体类拥有这个关联的反向端的属性。在这里，这是`Post`的`$comments`属性。

### 注意

对于每个具有`private`或`protected`属性的实体类，`Comment`类必须公开 getter 和 setter 来访问它们。我们将在本章后面为我们应用程序的每个实体类生成 getter 和 setter。

## 为`Post`实体类添加反向端

现在，我们需要修改`Post`实体类以添加这个关联的反向端。需要执行以下步骤：

1.  打开`src/Blog/Entity/`位置的`Post.php`文件，并从前一个代码片段中添加 use 语句：

```php
  use Doctrine\ORM\Mapping\OneToMany;
  use Doctrine\Common\Collections\ArrayCollection;
```

1.  按照以下代码片段中所示添加`$comments`属性：

```php
    /**
     * @var Comment[]
     *
     * @OneToMany(targetEntity="Comment", mappedBy="post")
     */
    protected $comments;
```

1.  将其初始化代码添加到构造函数中，如下一个代码片段所示：

```php
    /**
     * Initializes collections
     */
    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }
```

1.  使用 Doctrine 命令行工具提供的实体生成器为我们刚刚添加到`Comment`和`Post`类的属性创建 getter 和 setter：

```php
**php vendor/bin/doctrine.php orm:generate:entities src/**

```

1.  在生成的`addComment()`方法中，添加以下代码片段中的突出显示行以自动设置关联的拥有端：

```php
    public function addComment(\Blog\Entity\Comment$comments)
    {
        $this->comments[] = $comments;
        $comments->setPost($this);

        return $this;
    }
```

`$comments`属性保存与`Post`实体相关联的评论集合。我们使用`@OneToMany`注释将此属性标记为关联的反向端，之前在`Comment`的`$post`属性中定义。我们已经解释了`targetEntity`属性。`mappedBy`属性是关联的反向端的`inversedBy`属性的等价物。它指示相关实体类的属性拥有关联的另一端。

为了让 Doctrine 正确管理元素的集合，必须使用 Doctrine Common 组件提供的特殊类。`Post`实体的`$comments`属性在构造函数中初始化为`Doctrine\Common\Collections\ArrayCollection`的实例。`ArrayCollection`实现了`Doctrine\Common\Collections\Collection`接口。这将使 Doctrine 能够填充和管理集合。

Doctrine `Collection`类实现了`Countable`、`IteratorAggregate`和`ArrayAccess`接口（这些接口在 PHP 或 SPL 中预定义）。因此，Doctrine 集合可以像标准的 PHP 数组一样使用，并且可以在 foreach 循环中透明地迭代。

### 注意

有关预定义接口和标准 PHP 库（SPL）提供的接口的更多信息，请参阅以下 PHP 手册：

[`php.net/manual/en/reserved.interfaces.php`](http://php.net/manual/en/reserved.interfaces.php)和[`php.net/manual/en/spl.interfaces.php`](http://php.net/manual/en/spl.interfaces.php)

Doctrine 命令行工具生成的`addComment()`和`removeComment()`方法演示了如何使用 Doctrine `Collection`类的方法来添加和删除项目。

### 注意

可用方法的完整列表在 Doctrine 网站上有文档，如下所示：

[`docs.doctrine-project.org/en/latest/reference/working-with-associations.html`](http://docs.doctrine-project.org/en/latest/reference/working-with-associations.html)

另一个重要的事情，正如已经解释的那样，Doctrine 只管理关联的拥有方。这就是为什么我们在`addComment()`方法中调用`Comment`实体的`setPost()`方法。这允许从反向方面进行关联的持久化。

### 注意

这仅在实体的更改跟踪策略是延迟隐式时才有效（这是默认情况）。延迟隐式策略是最方便的使用方式，但可能对性能产生负面影响。

再次参考 Doctrine 文档，了解更多可以使用的不同更改跟踪策略：

[`docs.doctrine-project.org/en/latest/reference/change-tracking-policies.html`](http://docs.doctrine-project.org/en/latest/reference/change-tracking-policies.html)

接下来，我们将更新我们的 UI 以添加评论功能。首先必须更新数据库模式。

## 更新数据库模式

与其他注释一样，Doctrine 能够自动创建在 SQL 层存储关联所需的列和外键。再次运行`orm:schema-tool:update`命令，与命令行工具捆绑在一起，如下所示：

```php
**php vendor/bin/doctrine.php orm:schema-tool:update --force**

```

Doctrine 将自动检测对映的更改，并相应地更新 SQL 模式。可以添加`--force`标志来有效执行查询。

### 注意

`orm:schema-tool:update`命令不应该在生产中使用。它可能会永久删除数据（例如，当删除列时）。相反，应该使用 Doctrine Migrations 库来正确处理复杂的迁移。即使这个库还没有被认为是稳定的，它非常方便。我们可以在以下网站找到这个库：

[`docs.doctrine-project.org/projects/doctrine-migrations/en/latest/reference/introduction.html`](http://docs.doctrine-project.org/projects/doctrine-migrations/en/latest/reference/introduction.html)

## 为评论添加装置

至于帖子，我们将为评论创建一些装置。在`src/Blog/DataFixtures/`位置创建一个名为`LoadCommentData.php`的新文件。下一个代码片段用于此目的：

```php
<?php

namespace Blog\DataFixtures;

use Blog\Entity\Comment;
use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use Doctrine\Common\DataFixtures\Doctrine;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;

/**
 * Comment fixtures
 */
class LoadCommentData implements FixtureInterface,DependentFixtureInterface
{
    /**
     * Number of comments to add by post
     */
    const NUMBER_OF_COMMENTS_BY_POST = 5;

    /**
     * {@inheritDoc}
     */
    public function load(ObjectManager $manager)
    {
        $posts = $manager->getRepository('Blog\Entity\Post')->findAll();

        foreach ($posts as $post) {
            for ($i = 1; $i <= self::NUMBER_OF_COMMENTS_BY_POST;$i++) {
                $comment = new Comment();
                $comment
                    ->setBody(<<<EOTLorem ipsum dolor sit amet, consectetur adipiscing elit.EOT
                    )
                    ->setPublicationDate(new \DateTime(sprintf('-%ddays', self::NUMBER_OF_COMMENTS_BY_POST - $i)))
                    ->setPost($post)
                ;

                $manager->persist($comment);
            }
        }

        $manager->flush();
    }

    /**
     * {@inheritDoc}
     */
    public function getDependencies()
    {
        return ['Blog\DataFixtures\LoadPostData'];
    }
}
```

我们使用`EntityManager`类来检索`Post`实体存储库，然后我们使用这个存储库来检索所有的帖子。我们为每个帖子添加了五条评论。这个数据装置类实现了`Doctrine\Common\DataFixtures\DependentFixtureInterface`接口（`getDependencies()`方法）。它告诉数据加载器首先加载`LoadPostData`，因为这个数据装置类依赖于它。

## 列出和创建评论

是时候更新 UI 了。在`web/`位置创建一个名为`view-post.php`的文件。这个页面显示一个帖子和它的所有评论，还有一个添加新评论的表单，并处理评论的创建。

用于检索帖子和处理评论创建的代码如下：

```php
<?php

/**
 * View a blog post
 */

use Blog\Entity\Comment;

require_once __DIR__ . '/../src/bootstrap.php';
/** @var \Blog\Entity\Post $post The post to edit */
$post = $entityManager->find('Blog\Entity\Post', $_GET['id']);

if (!$post) {
    throw new \Exception('Post not found');
}

// Add a comment
if ('POST' === $_SERVER['REQUEST_METHOD']) {
    $comment = new Comment();
    $comment
        ->setBody($_POST['body'])
        ->setPublicationDate(new \DateTime())
        ->setPost($post)
    ;

    $entityManager->persist($comment);
    $entityManager->flush();

    header(sprintf('Location: view-post.php?id=%d', $post->getId()));
    exit;
}
?>
```

正如你所看到的，使用 Doctrine 管理简单的关联是很容易的。设置关系就像调用一个带有实体链接的 setter 一样简单。使用 getter 可以访问相关实体。用于显示帖子的详细信息、相关评论和发布新评论的表单的代码（将其放在同一个文件的底部）如下：

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title><?=htmlspecialchars($post->getTitle())?> - My blog</title>
</head>
<body>

<article>
    <h1>
        <?=htmlspecialchars($post->getTitle())?>
    </h1>
    Date of publication: <?=$post->getPublicationDate()->format('Y-m-d H:i:s')?>
    <p>
        <?=nl2br(htmlspecialchars($post->getBody()))?>
    </p>
    <?php if (count($post->getComments())): ?>
        <h2>Comments</h2>

        <?php foreach ($post->getComments() as $comment): ?>
            <article>
                <?=$comment->getPublicationDate()->format('Y-m-dH:i:s')?>

                <p><?=htmlspecialchars($comment->getBody())?></p>

                <a href="delete-comment.php?id=<?=$comment->getId()?>">Delete this comment</a>
            </article>
        <?php endforeach ?>
    <?php endif ?>

    <form method="POST">
        <h2>Post a comment</h2>

        <label>
            Comment
            <textarea name="body"></textarea>
        </label><br>

        <input type="submit">
    </form>
</article>

<a href="index.php">Back to the index</a>
```

默认情况下，Doctrine 会延迟加载关联的实体。这意味着，在我们的例子中，当调用`getComments()`时，Doctrine 首先向 DBMS 发送一个查询来检索帖子，然后再发送另一个查询来检索关联的评论。好处是，如果不调用`getComments()`方法，检索关联评论的查询将永远不会执行。但是当关联评论总是被获取时，这是一个无用的开销。

### 注意

为了使延迟加载功能起作用，Doctrine 在内部将实体包装成代理类。代理类负责在请求时获取尚未从数据库加载的关联实体的数据。关于这方面的一些细节可以在以下网址找到：

[`docs.doctrine-project.org/en/latest/reference/working-with-objects.html#entity-object-graph-traversal`](http://docs.doctrine-project.org/en/latest/reference/working-with-objects.html#entity-object-graph-traversal)

我们可以通过在关联注释上设置`fetch`属性来更改这种行为。该属性可以采用以下值：

+   `EAGER`：通常在第一个查询中使用 SQL 连接获取相关实体。

+   `懒加载`：相关实体只有在使用另一个 SQL 查询请求时才会被获取。这是默认值。

+   `EXTRA_LAZY`：这允许在不加载整个集合到内存中的情况下执行一些操作，例如计数。要了解更多信息，请参阅以下教程：

[`docs.doctrine-project.org/en/latest/tutorials/extra-lazy-associations.html`](http://docs.doctrine-project.org/en/latest/tutorials/extra-lazy-associations.html)

另一种急切加载相关实体的方法是使用 Doctrine 查询构建器来自定义生成的请求。我们将在第四章中展示查询构建器的强大功能。

通过在`view-post.php`页面中删除评论，我们创建了一个允许删除评论的链接。要使此功能正常工作，需要在`web/`位置的`delete-comment.php`文件中放入以下代码：

```php
<?php

/**
 * Deletes a comment
 */

require_once __DIR__ . '/../src/bootstrap.php';
/** @var Comment $comment The comment to delete */
$comment = $entityManager->find('Blog\Entity\Comment', $_GET['id']);

if (!$comment) {
    throw new \Exception('Comment not found');
}

// Delete the entity and flush
$entityManager->remove($comment);
$entityManager->flush();

// Redirect to the blog post
header(sprintf('Location: view-post.php?id=%d', $comment->getPost()->getId()));
exit;
```

这个文件与第一章中在`web/`位置创建的`delete-post.php`文件非常相似，*开始使用 Doctrine 2*。它通过`EntityManager`类检索存储库，使用它检索要删除的评论，调用`remove()`，并使用`flush()`将更改持久化到 DBMS。

## 更新索引

更新`web/`位置的`index.php`文件，创建一个链接到新的详细帖子视图，如下所示：

```php
        <h1>
            <?=htmlspecialchars($post->getTitle())?>
        </h1>
```

为了使我们的评论功能准备就绪，请使用以下代码替换前面的代码：

```php
        <h1>
            <a href="view-post.php?id=<?=$post->getId()?>">
                <?=htmlspecialchars($post->getTitle())?>
            </a>
        </h1>
```

# 理解标签的@ManyToMany 注释

标签按主题对帖子进行分组。一个标签包含多个帖子，一个帖子有多个标签。这是一个多对多双向关联。Doctrine 在 SQL 级别上透明地管理存储多对多关系所需的关联表。将生成的 MySQL 模式显示在以下截图中：

![理解标签的@ManyToMany 注释](img/4104_03_02.jpg)

## 创建`Tag`实体类（反向端）

`Tag`实体类只有两个属性：

+   `name`：这是标签的名称，它是唯一的，并且是实体的标识符

+   `posts`：这是与此标签关联的帖子集合

以下是创建`Tag`实体类的步骤：

1.  在`src/Blog/Entity/`位置创建一个`Tag.php`文件，其中包含使用以下代码片段的实体类：

```php
<?php

namespace Blog\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Column;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\ManyToMany;

/**
 * Tag entity
 *
 * @Entity
 */
class Tag
{
    /**
     * @var string
     *
     * @Id
     * @Column(type="string")
     */
    protected $name;
    /**
     * @var Post[]
     *
     * @ManyToMany(targetEntity="Post", mappedBy="tags")
     */
    protected $posts;

    /**
     * Initializes collection
     */
    public function __construct()
    {
        $this->posts = new ArrayCollection();
    }

    /**
     * String representation
     *
     * @return string
     */
    public function __toString()
    {
        return $this->getName();
    }
}
```

1.  使用以下命令生成 getter 和 setter：

```php
**php vendor/bin/doctrine.php orm:generate:entities src/**

```

1.  在`addPost()`方法中的`$this->posts[] = $posts;`之后添加以下代码行以设置关联的拥有端：

```php
$posts->addTag($this);
```

`$name`属性是`Tag`实体的标识符。与`Post`和`Comment`实体不同，它的值不是由 DBMS 自动生成的；它是标签的名称。这就是为什么这里不使用`@GeneratedValue`注释。标签的名称必须是唯一的，并且必须由应用程序设置。

`@ManyToMany`注释用于标记关联。`targetEntity`和`mappedBy`属性的含义与`@OneToMany`注释相同。`@ManyToMany`注释接受`mappedBy`属性作为反向端，`inversedBy`作为拥有端。这个关联的拥有端在`Post`实体上。与任何 Doctrine 集合一样，`$posts`属性在构造函数中被初始化。我们还创建一个`__toString()`方法，返回标签的名称，以便能够将`Tag`的实例转换为字符串。

### 注意

`__toString()`魔术方法允许我们将对象转换为字符串。有关更多详细信息，我们可以参考以下链接：

[`www.php.net/manual/en/language.oop5.magic.php#object.tostring`](http://www.php.net/manual/en/language.oop5.magic.php#object.tostring)

## 更新 Post 实体类（拥有方）

修改`src/Blog/Entity/`位置的`Post.php`文件，以添加关联的拥有方使用以下步骤：

1.  添加以下`use`语句：

```php
use Doctrine\ORM\Mapping\ManyToMany;
use Doctrine\ORM\Mapping\JoinTable;
use Doctrine\ORM\Mapping\JoinColumn;
```

1.  使用以下代码片段添加`mapped`属性：

```php
    /**
     * @var Tag[]
     *
     * @ManyToMany(targetEntity="Tag", inversedBy="posts",fetch="EAGER", cascade={"persist"}, orphanRemoval=true)
     * @JoinTable(
     *      inverseJoinColumns={@JoinColumn(name="tag_name",referencedColumnName="name")}
     * )
     */
    protected $tags;
```

1.  按照以下代码片段中所示的方式在构造函数中初始化属性：

```php
    public function __construct()
    {
        // …
        $this->tags = new ArrayCollection();
    }
```

1.  要生成 getter 和 setter，可以使用以下命令：

```php
**php vendor/bin/doctrine.php orm:generate:entities src/**

```

这里介绍了`@ManyToMany`注释的两个新属性，即`cascade`和`orphanRemoval`。

默认情况下，当设置主实体时，关联实体不会自动设置为托管状态。这必须通过对每个关联实体的`EntityManager`类的`persist()`方法进行手动调用来完成。如果`cascade`属性与`persist`一起使用，相关实体将在持久化主实体时自动持久化。

在这里，当持久化`Post`实体时，相关标签将一起持久化。`cascade`属性可以采用其他值，其中最有用的是`remove`。使用`remove`时，当删除主实体时，相关实体将被删除。

**对象关系映射器（ORM）**在内存中处理`CASCADE`操作。它们不等同于 SQL 的`DELETE CASCADE`操作，并且可能使用大量内存。应该谨慎使用以保持应用程序的性能。

可以通过`@JoinColumn`注释的`onDelete`属性添加 SQL`DELETE CASCADE`操作。

将`orphanRemoval`属性设置为`true`，Doctrine 将自动删除不再与主实体关联的实体。如果从`Post`实体的`$tags`集合中删除`Tag`实体，并且这个`Post`实体是唯一与`Tag`实体关联的实体，那么`Tag`实体将被永久删除。

`fetch`属性已在本章中进行了解释。使用`EAGER`值，它告诉 Doctrine 在检索帖子时自动使用`JOIN`查询检索相关标签。在我们的应用程序环境中，这是有用的，因为`Post`实体的标签在每次显示帖子时都会显示。

由于`Tag`的标识符没有标记`@GeneratedValue`注释，Doctrine 将无法猜测它。`@JoinTable`和`@JoinColumn`注释在这里用于覆盖默认行为。

我们使用`@JoinColumn`为关联方（反向方）设置自定义`JOIN`列，通过`@JoinTable`的`inverseJoinColumns`属性。`@JoinColumn`的`referencedColumnName`属性告诉 Doctrine 在 SQL 级联表中查找`Tag`的标识符的`$name`属性（默认情况下是`$id`）。`name`属性将`Tag`的标识符的列名称设置为`tag_name`（默认情况下是`tag_id`）。

## 再次更新模式

现在是时候再次更新 SQL 模式以匹配我们的更改。我们在命令行上使用以下命令：

```php
**php vendor/bin/doctrine.php orm:schema-tool:update --force**

```

## 创建标签固定装置

在`src/Blog/DataFixtures/`创建一个`LoadTagData.php`文件，其中包含使用以下代码片段的标签固定装置：

```php
<?php

namespace Blog\DataFixtures;

use Blog\Entity\Tag;
use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use Doctrine\Common\DataFixtures\Doctrine;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;

/**
 * Tag fixtures
 */
class LoadTagData implements FixtureInterface,DependentFixtureInterface
{
    /**
     * Number of comments to add by post
     */
    const NUMBER_OF_TAGS = 5;
    /**
     * {@inheritDoc}
     */
    public function load(ObjectManager $manager)
    {
        $tags = [];
        for ($i = 1; $i <= self::NUMBER_OF_TAGS; $i++) {
            $tag = new Tag();
            $tag->setName(sprintf("tag%d", $i));

            $tags[] = $tag;
        }

        $posts = $manager->getRepository('Blog\Entity\Post')->findAll();

        $tagsToAdd = 1;
        foreach ($posts as $post) {
            for ($j = 0; $j < $tagsToAdd; $j++) {
                $post->addTag($tags[$j]);
            }

            $tagsToAdd = $tagsToAdd % 5 + 1;
        }

        $manager->flush();
    }

    /**
     * {@inheritDoc}
     */
    public function getDependencies()
    {
        return ['Blog\DataFixtures\LoadPostData'];
    }
}
```

由于`persist`属性，我们可以在不手动持久化的情况下向帖子添加标签。

固定装置后，我们必须更新 UI。

## 管理帖子的标签

编辑`web/`位置的`edit-post.php`文件，并按以下步骤添加代码来管理标签：

1.  在文件顶部添加以下`use`语句：

```php
use Blog\Entity\Tag;
```

1.  找到以下代码片段：

```php
    $post
        ->setTitle($_POST['title'])
        ->setBody($_POST['body'])
    ;
```

1.  在`to extract`和管理提交的标签后添加此代码：

```php
    $newTags = [];
    foreach (explode(',', $_POST['tags']) as $tagName) {
        $trimmedTagName = trim($tagName);
        $tag = $entityManager->find('Blog\Entity\Tag',$trimmedTagName);
        if (!$tag) {
            $tag = new Tag();
            $tag->setName($trimmedTagName);
        }

        $newTags[] = $tag;
    }

    // Removes unused tags
    foreach (array_diff($post->getTags()->toArray(),$newTags) as $tag) {
        $post->removeTag($tag);
    }

    // Adds new tags
    foreach (array_diff($newTags, $post->getTags()->toArray()) as $tag) {
        $post->addTag($tag);
    }
```

1.  找到以下代码片段：

```php
    <label>
        Body
        <textarea name="body" cols="20" rows="10"required><?=isset ($post) ? htmlspecialchars($post-
        >getBody()) : ''?></textarea>
    </label><br>
```

1.  在`to display`后添加以下表单部件以更新标签：

```php
    <label>
        Tags
        <input type="text" name="tags" value="<?=isset($post) ? htmlspecialchars(implode(', ', $post->getTags()->toArray())) : ''?>" required>
    </label><br>
```

从提交的字符串中提取每个标签名称。从存储库中检索相应的`Tag`实体，如果找不到则创建。

由于它的`toArray()`方法，`Post`对象的`tag`集合被转换为标准的 PHP 数组。

标准的`array_diff()`函数用于识别已删除和已添加的`Tag`对象。`array_diff()`的参数必须是可以转换为字符串的对象数组。这里可以使用，因为我们的`Tag`类实现了`__toString()`魔术方法。

已删除的标签通过`Post::removeTag()`函数移除，新标签通过`Post::addTag()`添加。

感谢在`Post`实体类中定义的`CASCADE`属性，我们不需要针对每个新标签单独持久化。

在模板中，标签列表按照“tagname1，tagname2，tagname3”的模式转换为字符串。

# 总结

在本章中，我们学习了如何管理 Doctrine ORM 支持的所有类型的关联。我们学习了单向和双向关联以及拥有方和反向方的概念。我们还运用了我们在之前章节学到的知识，特别是`EntityManager`类、装置加载器和生成器。

在下一章中，我们将学习如何使用 DQL 和 Query Builder 创建复杂的查询。

由于它们，我们将创建按标签分组的帖子列表。我们还将研究聚合函数。
