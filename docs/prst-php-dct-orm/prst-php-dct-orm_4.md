# 第四章：构建查询

在上一章中，我们为我们的博客软件添加了评论和标记支持。虽然它运行良好，但一些功能可以得到增强。

在本章中，我们将利用 Doctrine 的一些非常重要的部分：**Doctrine 查询语言**（**DQL**）、实体存储库和查询构建器。我们将在本章中涵盖以下方面：

+   优化评论功能

+   创建一个页面来通过标签过滤帖子

+   在主页上显示帖子的评论数量

# 理解 DQL

DQL 是 Doctrine 查询语言的缩写。它是一种特定领域的语言，非常类似于 SQL，但不是 SQL。DQL 不是用于查询数据库表和行，而是设计用于查询对象模型的实体和映射属性。

DQL 受 Hibernate 的查询语言 HQL 的启发和类似，Hibernate 是 Java 的流行 ORM 的查询语言。有关更多详细信息，您可以访问此网站：[`www.hibernate.org/`](http://www.hibernate.org/)。

### 注

在以下网站了解更多关于特定领域语言的信息：

[`en.wikipedia.org/wiki/Domain-specific_language`](http://en.wikipedia.org/wiki/Domain-specific_language)

为了更好地理解其含义，让我们运行我们的第一个 DQL 查询。

Doctrine 命令行工具就像瑞士军刀一样实用。它们包括一个名为`orm:run-dql`的命令，用于运行 DQL 查询并显示结果。使用它来检索带有`1`作为标识符的帖子的`title`和所有评论：

```php
php vendor/bin/doctrine.php orm:run-dql "SELECT p.title, c.bodyFROM Blog\Entity\Post p JOIN p.comments c WHERE p.id=1"
```

它看起来像一个 SQL 查询，但绝对不是 SQL 查询。检查`FROM`和`JOIN`子句；它们包含以下方面：

+   在`FROM`子句中使用完全限定的实体类名作为查询的根

+   所有与所选“帖子”实体相关联的“评论”实体都被连接在一起，这要归功于“帖子”实体类中的“评论”属性在`JOIN`子句中的存在

正如您所看到的，可以以面向对象的方式请求与主实体相关联的实体的数据。持有关联的属性（在拥有方或反向方）可以在`JOIN`子句中使用。

尽管存在一些限制（特别是在子查询领域），我们将看到如何绕过第五章中的限制，*进一步*，DQL 是一种强大而灵活的语言，用于检索对象图。在内部，Doctrine 解析 DQL 查询，通过**数据库抽象层（DBAL）**生成和执行它们对应的 SQL 查询，并使用结果填充数据结构。

### 注

到目前为止，我们只使用 Doctrine 来检索 PHP 对象。Doctrine 能够填充其他类型的数据结构，特别是数组和基本类型。还可以编写自定义水合器来填充任何数据结构。

如果您仔细观察上一次调用`orm:run-dql`的返回，您会发现它是一个数组，而不是一个对象图，已经被填充。

与本书中涵盖的所有主题一样，有关内置水合模式和自定义水合器的更多信息可在以下网站的 Doctrine 文档中找到：

[`docs.doctrine-project.org/en/latest/reference/dql-doctrine-query-language.html#hydration-modes`](http://docs.doctrine-project.org/en/latest/reference/dql-doctrine-query-language.html#hydration-modes)

# 使用实体存储库

实体存储库是负责访问和管理实体的类。就像实体与数据库行相关联一样，实体存储库与数据库表相关联。

我们已经在前几章中使用了 Doctrine 提供的默认实体存储库来检索实体。所有的 DQL 查询都应该写在与它们检索的实体类型相关的实体存储库中。它将 ORM 隐藏在应用程序的其他组件中，并使其更容易重用、重构和优化查询。

### 注

Doctrine 实体存储库是表数据网关设计模式的一种实现。有关更多详细信息，请访问以下网站：

[`martinfowler.com/eaaCatalog/tableDataGateway.html`](http://martinfowler.com/eaaCatalog/tableDataGateway.html)

为每个实体提供的基本存储库提供了管理实体的有用方法，如下所示：

+   `find($id)`: 返回具有`$id`作为标识符的实体，或`null`

### 注意

它在实体管理器的`find()`方法内部使用。我们在前面的章节中多次使用了这个快捷方式。

+   `findAll()`: 检索包含此存储库中所有实体的数组

+   `findBy(['property1' => 'value', 'property2' => 1], ['property3' => 'DESC', 'property4' => 'ASC'])`: 检索包含第一个参数中传递的所有条件匹配的实体的数组，并按照第二个参数排序

+   `findOneBy(['property1' => 'value', 'property2' => 1])`: 类似于`findBy()`，但只检索第一个实体，如果没有实体与条件匹配，则返回`null`

### 注意

实体存储库还提供了快捷方法，允许单个属性过滤实体。它们遵循这种模式：`findBy*()`和`findOneBy*()`。

例如，调用`findByTitle('My title')`等同于调用`findBy(['title' => 'My title'])`。

此功能使用了神奇的`__call()` PHP 方法。有关更多详细信息，请访问以下网站：

[`php.net/manual/en/language.oop5.overloading.php#object.call`](http://php.net/manual/en/language.oop5.overloading.php#object.call)

如第三章，“关联”中所述，这些快捷方法不会连接相关实体，除非我们在实体类的关联注释中添加了`fetch="EAGER"`属性。如果（且仅当）通过方法调用请求了相关实体（或实体集合），则会发出另一个 SQL 查询。

在我们的博客应用中，我们希望在详细的帖子视图中显示评论，但不需要从帖子列表中获取它们。通过`fetch`属性的急切加载不适合列表，而延迟加载会减慢详细视图的速度。

解决这个问题的方法是创建一个具有额外方法来执行我们自己的查询的自定义存储库。我们将编写一个在详细视图中整理评论的自定义方法。

## 创建自定义实体存储库

自定义实体存储库是扩展了 Doctrine 提供的基本实体存储库类的类。它们旨在接收运行 DQL 查询的自定义方法。

像往常一样，我们将使用映射信息告诉 Doctrine 使用自定义存储库类。这是`@Entity`注释的`repositoryClass`属性的作用。

请执行以下步骤创建自定义实体存储库：

1.  重新打开`Post.php`文件，位于`src/Blog/Entity/`位置，并在现有的`@Entity`注释中添加一个`repositoryClass`属性，就像下面的代码行一样：

```php
@Entity(repositoryClass="PostRepository")
```

1.  Doctrine 命令行工具还提供了实体存储库生成器。输入以下命令来使用它：

```php
**php vendor/bin/doctrine.php orm:generate:repositories src/**

```

1.  打开这个新的空自定义存储库，我们刚刚在`src/Blog/Entity/`位置生成的`PostRepository.php`文件。添加以下方法来检索帖子和评论：

```php
   /**
     * Finds a post with its comments
     *
     * @param  int  $id
     * @return Post
     */
    public function findWithComments($id)
    {
        return $this
            ->createQueryBuilder('p')
            ->addSelect('c')
            ->leftJoin('p.comments', 'c')
            ->where('p.id = :id')
            ->orderBy('c.publicationDate', 'ASC')
            ->setParameter('id', $id)
            ->getQuery()
            ->getOneOrNullResult()
        ;
    }
```

我们的自定义存储库扩展了 Doctrine 提供的默认实体存储库。前面章节中描述的标准方法仍然可用。

# 开始使用查询构建器

`QueryBuilder`是一个旨在通过 PHP API 和流畅的接口帮助构建 DQL 查询的对象（要了解更多关于流畅接口的信息，请参见第二章，“实体和映射信息”）。它允许我们通过`getDql()`方法检索生成的 DQL 查询（用于调试）或直接使用`Query`对象（由 Doctrine 提供）。

### 注意

为了提高性能，`QueryBuilder`缓存生成的 DQL 查询并管理内部状态。

DQL 查询的完整 API 和状态在以下网站上有文档：

[`docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html`](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html)

我们将对我们在`PostRepository`类中创建的`findWithComments()`方法进行深入解释。

首先，使用从基本实体存储库继承的`createQueryBuilder()`方法创建一个`QueryBuilder`实例。`QueryBuilder`实例以字符串作为参数。此字符串将用作主实体类的别名。默认情况下，选择主实体类的所有字段，并且除了`SELECT`和`FROM`之外没有其他子句。

`leftJoin()`调用创建一个`JOIN`子句，用于检索与帖子相关联的评论。它的第一个参数是要加入的属性，第二个是别名；这些将在查询中用于加入的实体类（这里，字母`c`将用作`Comment`类的别名）。

### 注意

除非使用 SQL `JOIN`子句，否则 DQL 查询将自动获取与主实体相关联的实体。不需要像`ON`或`USING`这样的关键字。Doctrine 会自动知道是使用连接表还是外键列。

`addSelect()`调用将注释数据附加到`SELECT`子句。实体类的别名用于检索所有字段（这类似于 SQL 中的`*`运算符）。与本章的第一个 DQL 查询一样，可以使用表示法`alias.propertyName`检索特定字段。

你猜对了，对`where()`方法的调用设置了查询的`WHERE`部分。

在幕后，Doctrine 使用准备好的 SQL 语句。它们比标准 SQL 查询更有效。

`id`参数将由`setParameter()`调用设置的值填充。

再次感谢准备好的语句和这个`setParameter()`方法，自动避免了 SQL 注入攻击。

### 注意

SQL 注入攻击是使用未转义的用户输入执行恶意 SQL 查询的一种方法。让我们来看一个检查用户是否具有特定角色的糟糕 DQL 查询的例子：

```php
$query = $entityManager->createQuery('SELECT ur FROMUserRole ur WHERE ur.username = "' . $username . '" ANDur.role = "' . $role . '"');
$hasRole = count($query->getResult());
```

这个 DQL 查询将由 Doctrine 翻译成 SQL。如果有人输入以下用户名：

`" OR "a"="a`

字符串中包含的 SQL 代码将被注入，查询将始终返回一些结果。攻击者现在已经获得了对私人区域的访问权限。

正确的方法应该是使用以下代码：

```php
$query = $entityManager->createQuery("SELECT ur FROMUserRole WHERE username = :username and role = :role");
$query->setParameters([
    'username' => $username,
    'role' => $role
]);
$hasRole = count($query->getResult());
```

由于准备好的语句，用户名中包含的特殊字符（如引号）并不危险，这段代码将按预期工作。

`orderBy()`调用生成一个`ORDER BY`子句，按评论的发布日期对结果进行排序，先是较早的。

### 提示

大多数 SQL 指令在 DQL 中也有一个面向对象的等价物。最常见的连接类型可以使用 DQL 进行；它们通常具有相同的名称。

`getQuery()`调用告诉查询构建器生成 DQL 查询（如果需要，它将尽可能从缓存中获取查询），实例化 Doctrine `Query`对象，并用生成的 DQL 查询填充它。

这个生成的 DQL 查询将如下所示：

```php
SELECT p, c FROM Blog\Entity\Post p LEFT JOIN p.comments c WHEREp.id = :id ORDER BY c.publicationDate ASC
```

`Query`对象还公开了另一个用于调试的有用方法：`getSql()`。顾名思义，`getSql()`返回与 DQL 查询对应的 SQL 查询，Doctrine 将在 DBMS 上运行。对于我们的 DQL 查询，底层 SQL 查询如下：

```php
SELECT p0_.id AS id0, p0_.title AS title1, p0_.body AS body2,p0_.publicationDate AS publicationDate3, c1_.id AS id4, c1_.bodyAS body5, c1_.publicationDate AS publicationDate6, c1_.post_id ASpost_id7 FROM Post p0_ LEFT JOIN Comment c1_ ON p0_.id =c1_.post_id WHERE p0_.id = ? ORDER BY c1_.publicationDate ASC
```

`getOneOrNullResult()`方法执行它，检索第一个结果，并将其作为`Post`实体实例返回（如果找不到结果，则此方法返回`null`）。

### 注意

与`QueryBuilder`对象一样，`Query`对象管理内部状态，仅在必要时生成底层 SQL 查询。

在使用 Doctrine 时，性能是需要非常小心的。当设置为生产模式时（参见第一章，*使用 Doctrine 2 入门*），ORM 能够缓存生成的查询（DQL 通过`QueryBuilder`对象，SQL 通过`Query`对象）和查询的结果。

ORM 必须配置为使用以下网站显示的其中一个快速支持系统（APC，Memcache，XCache 或 Redis）：

[`docs.doctrine-project.org/en/latest/reference/caching.html`](http://docs.doctrine-project.org/en/latest/reference/caching.html)

我们仍然需要更新视图层来处理我们的新的`findWithComments()`方法。

打开`web/`位置的`view-post.php`文件，在那里你会找到以下代码片段：

```php
$post = $entityManager->getRepository('Blog\Entity\Post')->find($_GET['id']);
```

用以下代码片段替换前面的代码行：

```php
$post = $entityManager->getRepository('Blog\Entity\Post')->findWithComments($_GET['id']);
```

# 按标签过滤

为了发现更高级的 QueryBuilder 和 DQL 的用法，我们将创建一个具有一个或多个标签的帖子列表。

标签过滤对于搜索引擎优化很有用，并且允许读者轻松找到他们感兴趣的内容。我们将构建一个能够列出具有多个共同标签的帖子的系统；例如，所有标记有 Doctrine 和 Symfony 的帖子。

要使用标签过滤我们的帖子，请执行以下步骤：

1.  在我们的自定义`PostRepository`类（`src/Blog/Entity/PostRepository.php`）中添加另一个方法，使用以下代码：

```php
    /**
     * Finds posts having tags
     *
     * @param string[] $tagNames
     * @return Post[]
     */
    public function findHavingTags(array $tagNames)
    {
        return $queryBuilder = $this
            ->createQueryBuilder('p')
                  ->addSelect('t')
            ->join('p.tags', 't')
            ->where('t.name IN (:tagNames)')
            ->groupBy('p.id')
            ->having('COUNT(t.name) >= :numberOfTags')
            ->setParameter('tagNames', $tagNames)
            ->setParameter('numberOfTags',count($tagNames))
            ->getQuery()
            ->getResult()
        ;
    }
```

这个方法有点复杂。它以标签名称数组的参数形式接受参数，并返回具有所有这些标签的帖子数组。

查询值得一些解释，如下所示：

+   主实体类（由继承的`createQueryBuilder()`方法自动设置）是`Post`，其别名是字母`p`。

+   我们通过`JOIN`子句连接相关标签；`Tag`类由`t`别名。

+   由于调用了`where()`，我们只检索通过参数传递的标签之一标记的帖子。我们使用 Doctrine 的一个很棒的功能，允许我们直接使用数组作为查询参数。

+   `where()`的结果通过调用`groupBy()`按`id`分组。

+   我们在`HAVING`子句中使用聚合函数`COUNT()`来过滤由`$tagNames`数组的一些标签标记的帖子，但不是所有的。

1.  编辑`web/`中的`index.php`文件以使用我们的新方法。在这里，你会找到以下代码：

```php
/** @var $posts \Blog\Entity\Post[] Retrieve the list ofall blog posts */
$posts = $entityManager->getRepository('Blog\Entity\Post')->findAll();
```

并用下一个代码片段替换前面的代码：

```php
$repository = $entityManager->getRepository('Blog\Entity\Post');
/** @var $posts \Blog\Entity\Post[] Retrieve the list ofall blog posts */
$posts = isset($_GET['tags']) ? $repository->findHavingTags($_GET['tags']) : $repository->findAll();
```

现在，当 URL 中存在名为`tags`的`GET`参数时，它将用于过滤帖子。更好的是，如果传入了多个逗号分隔的标签，只会显示具有所有这些标签的帖子。

1.  在您喜欢的浏览器中键入`http://localhost:8000/index.php?tags=tag4,tag5`。由于我们在上一章中创建的固定装置，应该列出帖子 5 和 10。

1.  在同一个文件中，找到以下代码：

```php
        <p>
            <?=nl2br(htmlspecialchars($post->getBody()))?>
        </p>
```

并按以下方式添加标签列表：

```php
        <ul>
        <?php foreach ($post->getTags() as $tag): ?>
            <li>
                <a href="index.php?tags=<?=urlencode($tag)?>"><?=htmlspecialchars($tag)?></a>
            </li>
        <?php endforeach ?>
        </ul>
```

显示带有指向标签页面的链接的智能标签列表。您可以复制此代码，然后将其粘贴到`web/`位置的`view-post.php`文件中；或者更好的是，*不要重复自己*：创建一个小的辅助函数来显示标签。

# 计数评论

我们仍然需要进行一些外观上的改变。评论很多的帖子吸引了很多读者。如果每篇帖子的评论数量可以直接从列表页面获得会更好。Doctrine 可以将包含对`aggregate`函数调用的结果的数组作为第一行，并将实体作为第二行。

添加以下方法，用于检索具有相关评论的帖子，到`PostRepository`类：

```php
    /**
     * Finds posts with comment count
     *
     * @return array
     */
    public function findWithCommentCount()
    {
        return $this
            ->createQueryBuilder('p')
            ->leftJoin('p.comments', 'c')
            ->addSelect('COUNT(c.id)')
            ->groupBy('p.id')
            ->getQuery()
            ->getResult()
        ;
    }
```

由于`GROUP BY`子句和调用`addSelect()`，此方法将返回一个二维数组，而不是`Post`实体的数组。返回的数组中包含两个值，如下所示：

+   我们的`Post`实体在第一个索引处

+   DQL 的`COUNT()`函数的结果（评论数量）在第二个索引处

在`web/`位置的`index.php`文件中，找到以下代码：

```php
    $posts = $repository->findHavingTags(explode(',',$_GET['tags']));
} else {
    $posts = $repository->findAll();
}
```

并用以下代码替换前面的代码以使用我们的新方法：

```php
    $results = $repository->findHavingTags(explode(',',$_GET['tags']));
} else {
    $results = $repository->findWithCommentCount();
} 
```

为了匹配`findWithCommentCount()`返回的新结构，找到以下代码：

```php
<?php foreach ($posts as $post): ?>
```

并用下一个代码片段替换前面的代码：

```php
<?php
    foreach ($results as $result):
        $post = $result[0];
        $commentCount = $result[1];
?>
```

### 注意

如前所述，在处理这种情况时使用自定义水合器是一个更好的做法。

您还应该查看以下网站上显示的自定义 AST Walker：

[`docs.doctrine-project.org/en/latest/cookbook/dql-custom-walkers.html`](http://docs.doctrine-project.org/en/latest/cookbook/dql-custom-walkers.html)

找到以下代码片段：

```php
<?php if (empty($posts)): ?>
```

并用下一个代码片段替换前面的代码：

```php
<?php if (empty($results)): ?>
```

是时候显示评论数量了。在标签列表后插入以下代码：

```php
        <?php if ($commentCount == 0): ?>
            Be the first to comment this post.
        <?php elseif ($commentCount == 1): ?>
            One comment
        <?php else: ?>
            <?= $commentCount ?> comments
        <?php endif ?>
```

由于`web/`位置的`index.php`文件还使用`findHavingTags()`方法来显示标记文章的列表，我们也需要更新这个方法。使用以下代码完成：

```php
            // …
            ->addSelect('t')
            ->addSelect('COUNT(c.id)')
            ->leftJoin('p.comments', 'c')
            // …
```

# 总结

在本章中，我们学习了 DQL，它与 SQL 的区别，以及它的查询构建器。我们还学习了实体存储库的概念以及如何创建自定义存储库。

即使从这些主题和 Doctrine 中还有很多东西可以学习，我们的知识应该足够开始使用 Doctrine 作为持久系统开发完整和复杂的应用程序。

在第五章，“进一步”，这本书的最后一章，我们将进一步讨论一些更高级的主题，包括如何处理继承，如何进行本地 SQL 查询以及事件系统的基础知识。
