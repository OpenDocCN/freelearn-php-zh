# 前言

Doctrine 2 已成为 PHP 最流行的现代持久化系统。它与 Symfony2 框架的标准版一起分发，可以独立在任何 PHP 项目中使用，并与 Zend Framework 2，CodeIgniter 或 Laravel 集成得非常好。它高效，自动抽象出流行的数据库管理系统，支持 PHP 5.3 功能（包括命名空间），可以通过 Composer 安装，并且具有经过广泛测试的高质量代码库。

Doctrine 的 ORM 库允许轻松持久化和检索 PHP 对象图，而无需手动编写任何 SQL 查询。它还提供了一个强大的面向对象的类似 SQL 的查询语言称为 DQL，一个数据库模式生成工具，一个事件系统等等。

为了发现这个必不可少的库，我们将一起构建一个典型的小型博客引擎。

# 本书涵盖的内容

第一章，“开始使用 Doctrine 2”，解释了如何通过 Composer 安装 Common，DBAL 和 ORM 库，获取我们的第一个实体管理器，并在介绍了我们在整本书中构建的项目之后配置命令行工具（Doctrine 的架构和开发环境的配置）。

第二章，“实体和映射信息”，介绍了 Doctrine 实体的概念。我们将创建第一个实体，使用注释将其映射到数据库，生成数据库模式，创建数据夹具，并最终奠定博客用户界面的基础。

第三章，“关联”，解释了如何处理 PHP 对象和 ORM 之间的关联。我们将创建新实体，详细说明一对一，一对多和多对多的关联，生成底层数据库模式，创建数据夹具，并在用户界面中使用关联。

第四章，“构建查询”，创建实体存储库，并帮助理解如何使用查询构建器生成 DQL 查询和检索实体。我们还将看一下聚合函数。

第五章，“更进一步”，将介绍 Doctrine 的高级功能。我们将看到 Doctrine 管理对象继承的不同方式，玩转实体生命周期事件，并创建本机 SQL 查询。

# 本书所需的内容

要执行本书的示例，您只需要 PHP 5.4+文本编辑器或 PHP IDE 以及您喜欢的浏览器。 

# 本书适合的读者

读者应该对面向对象编程，PHP（包括 PHP 5.3 和 5.4 中引入的功能）和一般数据库概念有很好的了解。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL 和用户输入显示如下：“`NativeQuery`类允许您执行本机 SQL 查询并将其结果作为 Doctrine 实体获取。”

代码块设置如下：

```php
    /**
     * Adds comment
     *
     * @param  Comment $comment
     * @return Post
     */
    public function addComment(Comment $comment)
    {
        $this->comments[] = $comment;
        $comment->setPost($this);

        return $this;
    }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```php
    /**
     * Adds comment
     *
     * @param  Comment $comment
     * @return Post
     */
    public function addComment(Comment $comment)
    {
        $this->comments[] = $comment;
 **$comment->setPost($this);**

        return $this;
    }
```

任何命令行输入或输出都以以下方式编写：

```php
**# php bin/load-fixtures.php**

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上显示的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中：“以下文本必须在终端中打印：**注意：此操作不应在生产环境中执行**。”

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会以这样的方式出现。
