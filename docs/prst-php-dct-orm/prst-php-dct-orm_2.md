# 第二章。实体和映射信息

在上一章中，我们了解了 Doctrine 背后的概念，学习了如何使用 Composer 进行安装，设置了 Doctrine 命令行工具，并深入了解了实体管理器。

在本章中，我们将涵盖以下主题：

+   创建我们的第一个实体类

+   使用注释将其映射到相关的数据库表和列

+   使用 Doctrine 提供的命令助手自动生成数据库模式

+   创建一些固定数据，并处理实体管理器以在 Web 用户界面中显示我们的数据

因为我们正在构建一个博客，我们的主要实体类将被称为`Post`，如下图所示：

![实体和映射信息](img/4104OS_02_01.jpg)

我们的`Post`实体类具有以下四个属性：

+   `id`：跨数据库表（和博客）的帖子的唯一标识符

+   `title`：帖子的标题

+   `body`：帖子的正文

+   `publicationDate`：帖子的发布日期

# 创建实体类

如第一章*开始使用 Doctrine 2*中所述，Doctrine 实体只是将保存在数据库中的 PHP 对象。Doctrine 注释添加在实体类属性的 PHP `DocBlock`注释中。Doctrine 使用注释将对象映射到相关的数据库表和属性到列。

### 注意

**DocBlocks**的最初目的是将技术文档直接集成到源代码中。解析 DocBlocks 的最流行的文档生成器是**phpDocumentator**，可以在此网站上找到：[`www.phpdoc.org`](http://www.phpdoc.org)。

每个实体一旦通过 Doctrine 持久化，将与数据库表的一行相关联。

在`src/Blog/Entity/`位置创建一个名为`Post.php`的新文件，其中包含以下代码的实体类：

```php
  <?php

  namespace Blog\Entity;

  use Doctrine\ORM\Mapping\Entity;
  use Doctrine\ORM\Mapping\Table;
  use Doctrine\ORM\Mapping\Index;
  use Doctrine\ORM\Mapping\Id;
  use Doctrine\ORM\Mapping\GeneratedValue;
  use Doctrine\ORM\Mapping\Column;

  /**
   * Blog Post entity
   *
   * **@Entity**
   * **@Table(indexes={**
   * **@Index(name="publication_date_idx",**    **columns="publicationDate")**
   * })
   */
  class Post
  {
    /**
     * @var int
     *
     * **@Id**
     * **@GeneratedValue**
     * **@Column(type="integer")**
     */
    protected $id;
    /**
     * @var string
     *
     * **@Column(type="string")**
     */
    protected $title;
    /**
     * @var string
     *
     * **@Column(type="text")**
     */
    protected $body;
    /**
     * @var \DateTime
     *
     * **@Column(type="datetime")**
     */
    protected $publicationDate;
  }
```

# 生成 getter 和 setter

我们在第一章*开始使用 Doctrine 2*中配置的 Doctrine 命令行工具包括一个有用的命令，用于为我们生成实体类的 getter 和 setter 方法。我们将使用它来避免编写`Post`类的 getter 和 setter 方法。

运行以下命令，生成应用程序所有实体类的 getter 和 setter：

```php
 **php vendor/bin/doctrine.php orm:generate:entities src/**

```

### 注意

如果您有多个实体，不想为所有实体生成 getter 和 setter，请使用`orm:generate:entities`命令的`filter`选项。

# 使用 Doctrine 注释进行映射

`Post`是一个简单的类，有四个属性。`$id`的 setter 实际上并没有被生成。Doctrine 在实体解析阶段直接填充`$id`实例变量。我们稍后会看到如何将 ID 生成委托给 DBMS。

Doctrine 注释从`\Doctrine\ORM\Mapping`命名空间导入，使用`use`语句。它们用于在 DocBlocks 中为类及其属性添加映射信息。DocBlocks 只是以`/**`开头的一种特殊的注释。

## 了解`@Entity`注释

`@Entity`注释用于类级别的 DocBlock 中，指定此类是实体类。

此注释的最重要属性是`repositoryClass`。它允许指定自定义实体存储库类。我们将在第四章*构建查询*中学习有关实体存储库的知识，包括如何制作自定义存储库。

## 理解`@Table`、`@Index`和`@UniqueConstraint`注释

`@Table`注释是可选的。它可用于向与实体类相关的表添加一些映射信息。

相关的数据库表名默认为实体类名。在这里，它是`Post`。可以使用注释的`name`属性进行更改。让 Doctrine 自动生成表和列名称是一个好习惯，但更改它们以匹配现有模式可能会很有用。

正如您所看到的，我们使用`@Table`注释在底层表上创建索引。为此，我们使用一个名为`indexes`的属性，其中包含索引列表。每个索引由`@Index`注释定义。每个`@Index`必须包含以下两个属性：

+   `name`：索引的名称

+   `columns`：索引列的列表

对于`Post`实体类，我们在`publicationDate`列上创建一个名为`publication_date_idx`的索引。

`@Table`注释的最后一个可选属性是`uniqueConstraints`（此处未使用）。它允许在列和列组上创建 SQL 级别的唯一约束。其语法类似于`@Index：name`来命名约束和`columns`来指定应用约束的列。

此属性仅由模式生成器使用。即使使用了`uniqueConstraints`属性，Doctrine 也不会自动检查值在整个表中是否唯一。底层的 DBMS 将会执行此操作，但可能会导致 DBMS 级别的 SQL 错误。如果我们想要强制数据的唯一性，我们应该在保存新数据之前执行检查。

## 深入了解@Column 注释

每个属性都通过`@Column`注释映射到数据库列。

映射的数据库列的名称默认为属性名称，但可以使用`name`参数进行更改。与表名一样，最好让 Doctrine 自动生成名称。

### 注意

与表名的情况一样，列名将默认为实体类属性名（如果正确遵循 PSR 样式，则为驼峰命名法）。

Doctrine 还提供了下划线命名策略（例如，与名为`MyEntity`的类相关的数据库表将是`my_entity`），并且可以编写自定义策略。

在 Doctrine 文档中了解更多信息：[`docs.doctrine-project.org/en/latest/reference/namingstrategy.html`](http://docs.doctrine-project.org/en/latest/reference/namingstrategy.html)

如果属性没有标记为`@Column`注释，Doctrine 将忽略它。

其`type`属性指示列的 Doctrine 映射类型（请参阅下一节）。这是此注释的唯一必需属性。

此注释支持一些更多的属性。与其他注释一样，支持的属性的完整列表可在 Doctrine 文档中找到。最重要的属性如下：

+   `unique`：如果为`true`，则此列的值必须在相关数据库表中是唯一的

+   `nullable`：如果为`false`，则值可以是`NULL`。默认情况下，列不能是`NULL`。

+   `length`：`string`类型的列的长度

+   `scale`：`decimal`类型的列的比例

+   `precision`：`decimal`类型的列的精度

与`@Table`一样，Doctrine 不使用`@Column`注释的属性来验证数据。这些属性仅用于映射和生成数据库模式。没有其他用途。出于安全和用户体验的原因，您必须验证用户提供的每一条数据。本书不涵盖此主题。如果您不想手动处理数据验证，请尝试来自[`symfony.com/components/Validator`](http://symfony.com/components/Validator)的 Symfony 验证器组件。

### 注意

也可以使用生命周期事件（参见第五章，“进一步”）来处理数据验证：[`docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/validation-of-entities.html`](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/validation-of-entities.html)

## 了解@Id 和@GeneratedValue 注释

`$id`属性有点特殊。这是一个映射到整数的列，但这主要是我们对象的唯一标识符。

通过`@Id`注释，此列将被用作表的主键。

默认情况下，开发人员负责确保此属性的值在整个表中是唯一的。几乎所有的 DBMS 都提供了在插入新行时自动递增标识符的机制。`@GeneratedValue`注释利用了这一点。当属性标记为`@GeneratedValue`时，Doctrine 将把标识符的生成委托给底层的 DBMS。

### 注意

其他 ID 生成策略可在[`docs.doctrine-project.org/en/latest/reference/basic-mapping.html#identifier-generation-strategies`](http://docs.doctrine-project.org/en/latest/reference/basic-mapping.html#identifier-generation-strategies)找到。

Doctrine 还支持复合主键。只需在复合主键的所有列上添加`@Id`注释。

我们将在第三章中学习另一个例子，使用唯一字符串作为标识符，*关联*。

## 使用其他注释

存在许多 Doctrine 映射注释。我们将在第三章中使用一些新的注释，*关联*，以创建实体之间的关系。

可在 Doctrine 文档中找到所有可用注释的完整列表，网址为[`docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/annotations-reference.html`](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/annotations-reference.html)。

# 了解 Doctrine 映射类型

在`@Column`注释中使用的 Doctrine 映射类型既不是 SQL 类型也不是 PHP 类型，但它们都被映射到。例如，Doctrine 的`text`类型将被转换为实体中的`string` PHP 类型，并存储在具有`CLOB`类型的数据库列中。

以下是 Doctrine 映射类型的 PHP 类型和 SQL 类型的对应表：

| Doctrine 映射类型 | PHP 类型 | SQL 类型 |
| --- | --- | --- |
| `string` | `string` | `VARCHAR` |
| `integer` | `integer` | `INT` |
| `smallint` | `integer` | `SMALLINT` |
| `bigint` | `string` | `BIGINT` |
| `boolean` | `boolean` | `BOOLEAN` |
| `decimal` | `double` | `DECIMAL` |
| `date` | `\DateTime` | `DATETIME` |
| `time` | `\DateTime` | `TIME` |
| `datetime` | `\DateTime` | `DATETIME`或`TIMESTAMP` |
| `text` | `string` | `CLOB` |
| `object` | 使用`serialize()`和`unserialize()`方法的对象 | `CLOB` |
| `array` | 使用`serialize()`和`unserialize()`方法的`array` | `CLOB` |
| `float` | `double` | `FLOAT`（双精度） |
| `simple_array` | 使用`implode()`和`explode()`的`array`，值不能包含逗号 | `CLOB` |
| `json_array` | 使用`json_encode()`和`json_decode()`方法的`object` | `CLOB` |
| `guid` | `string` | 如果 DBMS 支持`GUID`或`UUID`，则为`GUID`，否则为`VARCHAR` |
| `blob` | `resource stream`（参见[`www.php.net/manual/en/language.types.resource.php`](http://www.php.net/manual/en/language.types.resource.php)） | `BLOB` |

### 注意

请记住，我们可以创建自定义类型。要了解更多信息，请参阅：[`docs.doctrine-project.org/en/latest/cookbook/custom-mapping-types.html`](http://docs.doctrine-project.org/en/latest/cookbook/custom-mapping-types.html)

# 创建数据库模式

Doctrine 足够智能，可以生成与实体映射信息相对应的数据库模式。

### 注意

在设计相关数据库模式之前，首先设计实体是一个很好的做法。

为此，我们将再次使用第一章安装的命令行工具。在项目的根目录中键入以下命令：

```php
 **php vendor/bin/doctrine.php orm:schema-tool:create**

```

以下文本必须在终端中打印：

**注意：此操作不应在生产环境中执行。**

**创建数据库模式...**

**数据库模式创建成功！**

数据库中创建了一个名为`Post`的新表。您可以使用 SQLite 客户端来显示生成的表的结构：

```php
 **sqlite3 data/blog.db ".schema Post"**

```

它应该返回以下查询：

```php
  CREATE TABLE Post (id INTEGER NOT NULL, title VARCHAR(255) NOT NULL, body CLOB NOT NULL, publicationDate DATETIME NOT NULL, PRIMARY KEY(id));
  CREATE INDEX publication_date_idx ON Post (publicationDate);
```

以下屏幕截图是表 Post 的结构：

![创建数据库模式](img/4104OS_02_02.jpg)

Doctrine 也能够为 MySQL 和其他支持的 DBMS 生成模式。如果我们配置我们的应用程序使用 MySQL 服务器作为 DBMS，并运行相同的命令，生成的表将类似于以下屏幕截图：

![创建数据库模式](img/4104OS_02_03.jpg)

# 安装数据 fixtures

**Fixtures**是允许在每次安装后无需手动创建数据就可以测试应用程序的虚假数据。它们对自动化测试过程很有用，并且使新开发人员更容易开始在我们的项目上工作。

### 注意

任何应用程序都应该有自动化测试。我们正在构建的博客应用程序由 Behat（[`behat.org/`](http://behat.org/)）测试覆盖。它们可以在 Packt 网站提供的下载中找到。

Doctrine 有一个名为 Data Fixtures 的扩展，可以简化 fixtures 的创建。我们将安装它并使用它来创建一些虚假的博客帖子。

在项目的根目录中键入以下命令，通过 Composer 安装 Doctrine Data Fixtures：

```php
  php composer.phar require doctrine/data-fixtures:1.0.*
```

使用 Doctrine Data Fixtures 的第一步是创建一个 fixture 类。在`src/Blog/DataFixtures`目录中创建一个名为`LoadPostData.php`的文件，如下面的代码所示：

```php
  <?php

  namespace Blog\DataFixtures;

  use Blog\Entity\Post;
  use Doctrine\Common\DataFixtures\FixtureInterface;
  use Doctrine\Common\Persistence\ObjectManager;

  /**
   * Post fixtures
   */
  class LoadPostData implements **FixtureInterface**
  {
    /**
     * Number of posts to add
     */
    const NUMBER_OF_POSTS = 10;

    /**
     * {@inheritDoc}
     */
    public function **load(ObjectManager $manager)**
    {
        for ($i = 1; $i <= self::NUMBER_OF_POSTS; $i++) {
            $post = **new Post()**;
            $post
                **setTitle**(sprintf('Blog post number %d', $i))
                **setBody**(<<<EOTLorem ipsum dolor sit amet, consectetur adipiscing elit.EOT
                )
                **setPublicationDate**(**new \DateTime**(sprintf('-%d days', self::NUMBER_OF_POSTS - $i)))
            ;

            **$manager->persist($post);**
        }

        **$manager->flush();**
    }
}
```

这个`LoadPostData`类包含创建虚假数据的逻辑。它创建了十篇博客帖子，其中包括生成的标题、发布日期和正文。

`LoadPostData`类实现了`\Doctrine\Common\DataFixtures\FixtureInterface`目录中定义的`load()`方法。这个方法接受一个`EntityManager`实例的参数：

+   第一章的一些提醒，*使用 Doctrine 2 入门*：调用`EntityManager::persist()`将每个新实体的状态设置为已管理

+   在过程结束时，调用`flush()`方法将使 Doctrine 执行`INSERT`查询，有效地保存数据到数据库中

我们仍然需要为我们的 fixtures 类创建一个加载器。在项目的`bin/`目录中创建一个名为`load-fixtures.php`的文件，并使用以下代码：

```php
  <?php

  require_once __DIR__.'/../src/bootstrap.php';

  use Doctrine\Common\DataFixtures\Loader;
  use Doctrine\Common\DataFixtures\Purger\ORMPurger;
  use Doctrine\Common\DataFixtures\Executor\ORMExecutor;

 **$loader = new Loader();**
 **$loader->loadFromDirectory(__DIR__.'/../src/Blog/DataFixtures');**

 **$purger = new ORMPurger();**
 **$executor = new ORMExecutor($entityManager, $purger);**
  $executor->execute($loader->getFixtures());
```

在这个实用程序中，我们初始化我们的应用程序并按照第一章中的说明获取实体管理器，*使用 Doctrine 2 入门*。然后，我们实例化了 Doctrine Data Fixtures 提供的 fixtures 加载器，并告诉它在哪里找到我们的 fixtures 文件。

目前我们只有`LoadPostData`类，但我们将在接下来的章节中创建额外的 fixtures。

`ORMExecutor`方法被实例化并执行。它使用`ORMPurger`从数据库中删除现有数据。然后它用我们的 fixtures 填充数据库。

在我们项目的根目录中运行以下命令来加载我们的 fixtures：

```php
 **php bin/load-fixtures.php**

```

我们的 fixtures 已经插入到数据库中。请注意，每次运行此命令时，数据库中的所有数据都将被永久删除。

检查我们的数据库是否已经用以下命令填充：

```php
 **sqlite3 data/blog.db "SELECT * FROM Post;"**

```

您应该看到十行类似于以下内容的行：

**1|博客帖子编号 1|Lorem ipsum dolor sit amet，consectetur adipiscing elit。|2013-11-08 20:01:13**

**2|博客帖子编号 2|Lorem ipsum dolor sit amet，consectetur adipiscing elit。|2013-11-09 20:01:13**

# 创建一个简单的 UI

我们将创建一个简单的 UI 来处理我们的帖子。这个界面将让我们创建、检索、更新和删除博客帖子。你可能已经猜到我们将使用实体管理器来做到这一点。

为了简洁并专注于 Doctrine 部分，这个 UI 将有很多缺点。*它不应该在任何生产或公共服务器上使用*。主要问题如下：

+   **一点也不安全**：每个人都可以访问一切，因为没有认证系统，没有数据验证，也没有 CSRF 保护

+   **设计不良**：没有关注点分离，没有使用类似 MVC 的模式，没有 REST 架构，没有面向对象的代码等等。

当然，这将是…图形上极简主义的！

+   跨站点请求伪造（CSRF）：[`en.wikipedia.org/wiki/Cross-site_request_forgery`](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

+   关注点分离：[`en.wikipedia.org/wiki/Separation_of_concerns`](http://en.wikipedia.org/wiki/Separation_of_concerns)

+   模型-视图-控制器（MVC）元模式：[`en.wikipedia.org/wiki/Model-view-controller`](http://en.wikipedia.org/wiki/Model-view-controller)

+   表述状态转移（REST）：[`en.wikipedia.org/wiki/Representational_state_transfer`](http://en.wikipedia.org/wiki/Representational_state_transfer)

对于真实世界的应用程序，您应该看一下 Symfony，这是一个强大的框架，包括 Doctrine 和大量功能（已经介绍了验证组件，表单框架，模板引擎，国际化系统等）：[`symfony.com/`](http://symfony.com/)

## 列出帖子

话虽如此，在`web/index.php`文件中创建列出帖子的页面，代码如下：

```php
  <?php

  /**
   * Lists all blog posts
   */

  require_once __DIR__.'/../src/bootstrap.php';

  /** @var $posts \Blog\Entity\Post[] Retrieve the list of all blog posts */
  **$posts = $entityManager->getRepository('Blog\Entity\Post')-**    **>findAll();**
  ?>

  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8">
    <title>My blog</title>
  </head>
  <body>
  <h1>My blog</h1>

 **<?php foreach ($posts as $post): ?>**
    <article>
        <h1>
            <?=htmlspecialchars(**$post->getTitle()**)?>
        </h1>
        Date of publication: <?=**$post->getPublicationDate()->format('Y-m-d H:i:s')**?>

        <p>
            <?=nl2br(htmlspecialchars(**$post->getBody()**))?>
        </p>

        <ul>
            <li>
                <a href="edit-post.php?id=<?=**$post->getId()**?>">Edit this post</a>
            </li>
            <li>
                <a href="delete-post.php?id=<?=**$post->getId()**?>">Delete this post</a>
            </li>
        </ul>
    </article>
  <?php endforeach ?>
  <?php if (empty($posts)): ?>
    <p>
        No post, for now!
    </p>
  <?php endif ?>

  <a href="edit-post.php">
    Create a new post
  </a>
  </html>
```

这个第一个文件是博客的主页面。它列出所有帖子，并显示链接到创建、更新或删除帖子的页面。

在应用程序初始化之后，我们使用我们在第一章中编写的代码来获取`EntityManager`以配置命令行工具。

我们使用这个`EntityManager`来检索我们的`\Blog\Entity\Post`实体的存储库。目前，我们使用 Doctrine 提供的默认实体存储库。我们将在第四章中了解更多关于它们的信息，*构建查询*。这个默认存储库提供了一个`findAll()`方法，用于检索从数据库中获取的所有实体的集合。

### 注意

`Collection`接口类似于常规的 PHP 数组（带有一些增强功能）。这个类是 Doctrine Common 的一部分：[`www.doctrine-project.org/api/common/2.4/class-Doctrine.Common.Collections.Collection.html`](http://www.doctrine-project.org/api/common/2.4/class-Doctrine.Common.Collections.Collection.html)

调用它时，Doctrine 将查询数据库以查找`Post`表的所有行，并使用检索到的数据填充`\Blog\Entity\Post`对象的集合。这个集合被分配给`$posts`变量。

要浏览此页面，请在项目的根目录中运行以下命令：

```php
  php -S localhost:8000 -t web/
```

这将运行内置的 PHP Web 服务器。在您喜欢的 Web 浏览器中转到`http://localhost:8000`，您将看到我们的十个虚假帖子。

### 注意

如果不起作用，请确保您的 PHP 版本至少为 5.4。

## 创建和编辑帖子

是时候创建一个页面来添加新的博客帖子了。将其放在`web/edit-post.php`文件中，如下面的代码所示：

```php
  <?php

  /**
   * Creates or edits a blog post
   */

  use Blog\Entity\Post;

  require_once __DIR__.'/../src/bootstrap.php';

  // Retrieve the blog post if an id parameter exists
  if (isset ($_GET['id'])) {
    /** @var Post $post The post to edit */
    **$post = $entityManager->find('Blog\Entity\Post', $_GET['id']);**

    if (!$post) {
        throw new \Exception('Post not found');
    }
}

  // Create or update the blog post
  if ('POST' === $_SERVER['REQUEST_METHOD']) {
    // Create a new post if a post has not been retrieved and set its date of publication
    if (!isset ($post)) {
 **$post = new Post();**
        // Manage the entity
 **$entityManager->persist($post);**

 **$post->setPublicationDate(new \DateTime());**
    }

 **$post**
 **->setTitle($_POST['title'])**
 **->setBody($_POST['body'])**
 **;**

    // Flush changes to the database
 **$entityManager->flush();**

    // Redirect to the index
    header('Location: index.php');
    exit;
}

  /** @var string Page title */
  $pageTitle = isset ($post) ? sprintf('Edit post #%d', $post->getId()) : 'Create a new post';
  ?>

  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8">
    <title><?=$pageTitle?> - My blog</title>
  </head>
  <body>
  <h1>
    <?=$pageTitle?>
  </h1>

  <form method="POST">
    <label>
        Title
        <input type="text" name="title" value="<?=isset ($post) ? htmlspecialchars($post->getTitle()) : ''?>" maxlength="255" required>
    </label><br>

    <label>
        Body
        <textarea name="body" cols="20" rows="10" required><?=isset ($post) ? htmlspecialchars($post->getBody()) : ''?></textarea>
    </label><br>

    <input type="submit">
  </form>

  <a href="index.php">Back to the index</a>
```

这个页面有点棘手：

+   在 URL 中带有`id`参数时，它会处理具有给定 ID 的`Post`实体

### 注意

最佳实践是使用 slug 而不是标识符。它们隐藏了应用程序的内部，可以被人类记住，并且对于搜索引擎优化更好：[`en.wikipedia.org/wiki/Slug_(publishing)`](http://en.wikipedia.org/wiki/Slug_(publishing))。

+   没有`id`参数时，它会实例化一个新的`Post`实体

+   当使用`GET` HTTP 方法调用时，它会显示一个填充有当前`Post`数据的表单，以进行编辑

+   当使用`Post` HTTP 方法（当表单被提交时）时，它会创建或更新`Post`实体，然后重定向到博客的主页

如果通过 URL 提供了 ID，则实体管理器的`find()`方法用于检索存储在数据库中的具有此 ID 的实体。Doctrine 查询数据库并为我们填充实体。

如果找不到具有此 ID 的`Post`，则将`NULL`值分配给`$post`变量，而不是`\Blog\Entity\Post`的实例。为了避免进一步的错误，如果是这种情况，我们会抛出一个异常。要了解更多关于 PHP 异常的信息，请参考网站[`php.net/manual/en/language.exceptions.php`](http://php.net/manual/en/language.exceptions.php)。

然后，我们使用我们的新实体作为参数调用实体管理器的`persist()`方法。如第一章*使用 Doctrine 2 入门*中所述，对`persist()`方法的调用将实体的状态设置为受管理状态。这仅对新实体是必要的，因为通过 Doctrine 检索的实体已经具有受管理状态。

接下来，我们设置我们新创建对象的发布日期。多亏了 Doctrine 映射系统，我们只需要将`\DateTime`实例传递给`setPublicationDate()`方法，ORM 将为我们将其转换为 DBMS 所需的格式（参考类型对应表）。

我们还使用先前生成的 getter 和 setter 的流畅接口设置了`$title`和`$body`属性。

### 注意

如果您不了解流畅接口，请阅读以下文章：[`martinfowler.com/bliki/FluentInterface.html`](http://martinfowler.com/bliki/FluentInterface.html)

当调用`flush()`方法时，实体管理器告诉 Doctrine 将所有受管理的实体与数据库同步。在这种情况下，只有我们的`Post`实体是受管理的。如果它是一个新实体，将生成一个`INSERT` SQL 语句。如果它是一个现有实体，将向 DBMS 发送一个`UPDATE`语句。

默认情况下，Doctrine 在调用`EntityManager::flush()`方法时自动将所有操作包装在事务中。如果发生错误，数据库状态将恢复到刷新调用之前的状态（回滚）。

这通常是最好的选择，但如果您有特定的需求，可以停用此自动提交模式。可以在[`docs.doctrine-project.org/en/latest/reference/transactions-and-concurrency.html`](http://docs.doctrine-project.org/en/latest/reference/transactions-and-concurrency.html)中找到相关信息。

## 删除帖子

让我们在`web/delete-post.php`文件中创建一个删除帖子的页面。

```php
  <?php

  /**
   * Deletes a blog post
   */

  require_once __DIR__.'/../src/bootstrap.php';

  /** @var Post The post to delete */
 **$post = $entityManager->find('Blog\Entity\Post', $_GET['id']);**
  if (!$post) {
    throw new \Exception('Post not found');
  }

  // Delete the entity and flush
 **$entityManager->remove($post);**
 **$entityManager->flush();**

  // Redirects to the index
  header('Location: index.php');
  exit;
```

我们使用 URL 中的 ID 参数检索要删除的帖子。我们告诉 Doctrine 安排删除它，调用`EntityManager::remove()`方法。在此调用之后，实体的状态被移除。当调用`flush()`方法时，Doctrine 执行`DELETE` SQL 查询以从数据库中删除数据。

### 注意

请注意，在调用`flush()`方法并从数据库中删除后，实体仍然存在于内存中。

# 总结

现在我们有一个最小但可用的博客应用程序！多亏了 Doctrine，将数据持久化、检索和删除到数据库中从未如此简单。

我们已经学会了如何使用注释将实体类映射到数据库表和行，我们生成了数据库模式而不需要输入一行 SQL，我们创建了固定装置，并且我们使用实体管理器将数据与数据库同步。

在下一章中，我们将学习如何在实体之间映射和管理一对一、一对多/多对一和多对多的关联。
