# 第五章：与数据库和模型一起工作

在本章中，我们将涵盖以下内容：

+   配置数据库凭据

+   加载数据库对象

+   从数据库中检索数据

+   将数据写入数据库

+   使用预处理语句防范 SQL 注入

+   创建自定义模型类

+   使用活动记录从数据库读取

+   使用活动记录向数据库写入

+   使用活动记录更新数据库记录

+   使用活动记录搜索数据库

+   使用活动记录和模型类删除对象

+   使用活动记录定义关系

# 简介

网络应用程序从连接到数据库中获得了许多功能。concrete5 是一个 **LAMP** 应用程序（在 Linux、Apache、MySQL 和 PHP 上运行）。MySQL 是 concrete5 使用的数据库，它是世界上最受欢迎的开源数据库。

在本章中，我们将探讨在 concrete5 中与数据库交互的方法，包括使用原始 SQL 查询和更面向对象的 active record 方法。我们将学习如何防范 SQL 注入攻击，创建自定义模型类，并使用对象和 active record 查询数据库。

本章将使用一些基本的 SQL 查询，因此建议您对编写数据库查询有一定的了解。

## 配置数据库凭据

在我们能够读取和写入数据库之前，我们需要确保 concrete5 有正确的数据库凭据。通常这些凭据是在安装期间设置的，但如果您将 concrete5 移动到另一个服务器或更改 concrete5 用户的用户名或密码，了解如何配置数据库访问是很重要的。

在这个菜谱中，我们将假设我们从零开始指定凭据。这将使我们能够探索数据库中存储的每个值。

## 如何操作...

查看以下步骤：

1.  在您首选的代码编辑器中打开 `/config/site.php`。

1.  指定数据库服务器的名称。在许多情况下，这仅仅是 `localhost`：

    ```php
    define('DB_SERVER', 'localhost');
    ```

1.  设置 concrete5 连接到数据库时使用的用户名：

    ```php
    define('DB_USERNAME', 'user');
    ```

1.  为该用户设置密码：

    ```php
    define('DB_PASSWORD', 'password');
    ```

1.  最后，指定 concrete5 将连接到的数据库名称：

    ```php
    define('DB_DATABASE', 'concrete5');
    ```

## 它是如何工作的...

当网站被访问时，`site.php` 文件会被 concrete5 自动加载。开发者可以使用 PHP 的 `define` 函数在这里定义配置常量。`define` 函数的第一个参数是常量的名称，**不要**更改此参数。然而，第二个参数应包含特定设置的相应值。

确保您为 concrete5 选择的用户有足够的数据库访问权限，并且允许执行所有必要的查询。

此文件中配置不当的数据库设置将导致网站显示错误。

# 加载数据库对象

在 concrete5 中执行数据库查询的核心是数据库对象。数据库对象公开了几个方法，允许开发者对数据库执行查询。

## 如何操作...

看看以下步骤：

1.  我们需要一个地方来测试一些测试代码。打开`/config/site_post.php`，这将很好地服务于你的代码编辑器。

1.  编写以下代码片段以加载数据库对象：

    ```php
    $db = Loader::db();
    ```

1.  现在数据库对象已加载，看看它公开的一些方法：

    ```php
    $methods = get_class_methods($db);
    ```

1.  将方法数组输出到屏幕：

    ```php
    var_dump($methods);
    exit;
    ```

## 工作原理...

concrete5 将使用配置文件中的数据库设置（可能位于`/config/site.php`）并连接到数据库，返回一个包含与数据库交互的几个有用方法的对象。

## 更多内容...

如果你想连接到`site.php`中指定的数据库以外的数据库，你可以将新的数据库凭据作为参数传递给数据库加载器，如下所示：

```php
$db = Loader::db('host', 'user', 'password', 'database');
```

# 从数据库中检索数据

一旦加载了数据库对象，你就可以对其执行查询。在这个菜谱中，我们将选择`Users`表的内容，并输出每个用户名。

## 准备工作

在深入编写 concrete5 中的手动查询之前，请确保你理解如何使用 SQL 编写数据库查询。网上和印刷品中有成百上千的 MySQL 资源，远远超出了本书的范围！

## 如何操作...

看看以下步骤：

1.  加载数据库对象：

    ```php
    $db = Loader::db();
    ```

1.  编写查询以从`Users`表中选择：

    ```php
    $query = 'SELECT * FROM Users';
    ```

1.  执行查询：

    ```php
    $results = $db->getAll($query);
    ```

1.  遍历结果并输出每个用户的用户名：

    ```php
    foreach ($results as $user) {
       echo $user['uName'].'<br />';
    }
     exit;
    ```

## 工作原理...

这个函数按预期工作，但它减少了使用 PHP 和 MySQL 执行原始数据库查询时的繁琐。在数据库对象上调用`getAll`函数会返回一个包含所有结果的数组。每个结果也是一个数组，其键对应于行中的每一列。

## 更多内容...

如果你正在执行只选择一个列并期望一个结果的查询（例如，一个`COUNT`查询），你可以使用`getOne`方法来获取你需要的确切值，如下面的代码片段所示：

```php
$count = $db->getOne('SELECT COUNT(*) FROM Users');
```

## 相关内容

+   *将数据写入数据库* 菜谱

# 将数据写入数据库

当然，在 MySQL 数据库上可以执行的查询中，大多数不会返回任何数据。如果开发者需要向表中插入一行怎么办？为此，我们可以使用数据库对象的`execute`方法。

在这个菜谱中，我们将向名为`CustomTable`的表中插入一行新数据。

## 准备工作

我们将在这里向数据库中插入一些行，但我们不想在正常的 concrete5 数据库表中搞乱任何东西。通过在你的 MySQL 数据库上执行以下语句创建一个虚拟表来运行一些查询（此 SQL 文件也包含在此书的免费代码下载中）：

```php
CREATE TABLE `CustomTable` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `country` varchar(255) DEFAULT '',
  PRIMARY KEY (`id`)
);
```

## 如何操作...

查看以下步骤：

1.  在您的代码编辑器中打开 `/config/site_post.php`。

1.  加载数据库对象：

    ```php
    $db = Loader::db();
    ```

1.  编写 `INSERT` 查询：

    ```php
    $query = 'INSERT INTO CustomTable (name, country) VALUES ("John Doe", "US")';
    ```

1.  执行 `INSERT` 查询：

    ```php
    $db->execute($query);
    ```

1.  输出一条成功消息，以便您知道更新已成功执行：

    ```php
    echo 'done!';
    exit;
    ```

## 它是如何工作的...

`execute` 函数允许开发者运行用户有权运行的任何查询。在这种情况下，我们只是执行一个语句来向自定义表中添加新行。

## 还有更多...

`execute` 函数可以用来在数据库上运行任何任意查询。例如，如果我们想将 `CustomTable` 的内容清空（`TRUNCATE`），我们可以编写以下代码片段：

```php
$query = 'TRUNCATE CustomTable';
$db->execute($query);
```

# 使用预定义语句防止 SQL 注入

在网络应用程序中，SQL 注入攻击一直是安全问题的主要来源。**SQL 注入**可能发生在用户提供的输入在插入 SQL 命令之前没有得到适当的转义和净化。恶意用户可能会利用这个机会在数据库上运行任意命令，要么泄露敏感数据，要么执行破坏性操作，如更改或删除数据。

幸运的是，这是一个容易解决的问题。确实，一个人可以在将输入拼接成 SQL 查询之前手动净化所有用户输入，但这很繁琐且容易出错（忘记净化应用程序的一部分仍然可能使其容易受到攻击）。防止 SQL 注入的最佳方式是使用预定义语句。

预定义语句（或参数化语句）与常规 SQL 查询类似。唯一的区别是，不是直接将用户提供的参数插入到查询中，数据库将在不同的时间运行基础查询和参数部分。因此，预定义语句不会受到 SQL 注入的威胁。

当运行包含用户提供的数据的查询时，使用仅预定义语句非常重要。

concrete5 中的数据库对象支持预定义语句，在这个配方中，我们将创建一个预定义语句，允许访客通过用户名搜索 `Users` 表，并返回该用户的电子邮件地址。

## 准备就绪

此配方将假设您的 concrete5 系统有一个用户名为 `admin` 的用户。请随意调整此代码以适应您的环境。

## 如何实现...

查看以下步骤：

1.  在您的代码编辑器中打开 `/config/site_post.php`。

1.  加载数据库对象：

    ```php
    $db = Loader::db();
    ```

1.  创建我们的预定义语句：

    ```php
    $query = 'SELECT * FROM Users WHERE uName = ?';
    ```

1.  将用户在 HTML 表单中提供的值存储起来。我们将假设它等于 `admin`：

    ```php
    $username = 'admin';
    ```

1.  运行预定义查询，将语句作为第一个参数，将过滤器值作为第二个参数：

    ```php
    $results = $db->getAll($query, $db->getAll($query, $db->getAll($query, $db->getAll($query, $db->getAll($query, 
    ```

1.  遍历结果并输出每个用户的电子邮件地址（应该只有一个）：

    ```php
    foreach ($results as $user) {
       echo $user['uEmail'] . '<br />';
    }
    exit;
    ```

## 它是如何工作的...

预处理语句是数据库管理系统的一个特性，在本例中是 MySQL。在 concrete5 中，数据库对象利用了这一原生特性，使其在单行函数中使用变得简单。这是因为查询在参数应用之前就已经准备并执行了；不存在 SQL 注入的风险。除了安全优势外，预处理语句也是提高多次运行的查询性能的绝佳方式。

## 更多内容...

数据库对象支持执行带有多个参数的预处理语句。在这些情况下，只需以查询中指定的相同顺序提供一个包含每个值的数组即可。看看以下代码片段：

```php
$sql = 'INSERT INTO CustomTable (name, country) VALUES (?, ?)';
$values = array('John Doe', 'US');
$db->execute($sql, $values);
```

## 参见

+   在[`php.net/manual/en/mysqli.quickstart.prepared-statements.php`](http://php.net/manual/en/mysqli.quickstart.prepared-statements.php)了解更多关于 PHP 中预处理语句的信息

# 创建自定义模型类

在 concrete5 中编写 SQL 命令是快速轻松地与数据库交互的方式，但这并不是最佳方式。由于 concrete5 是一个**MVC**（**模型-视图-控制器**）应用程序，所有数据操作都应该真正存在于模型中。**模型**是数据库中存在的对象的表示，它将包含读取和写入该对象的方法，并使其在数据库中持久化。

在这个菜谱中，我们将创建一个名为`Books`的新表，其中将包含多个书籍记录。然后我们将创建一个`Book`模型，它将在数据库中表示书籍数据。

## 准备工作

concrete5 在数据库表命名方面遵循特定的约定。如果你在 SQL 查询浏览器（如 phpMyAdmin 或 OS X 上的 Sequel Pro）中打开 concrete5 数据库，你会看到每个表都以大写字母开头，并且表名中的后续单词也是驼峰式命名。此外，在 concrete5 中，表应该是复数形式，因为它们包含多个项目实例。模型应该是单数形式，因为它们代表数据库中的单个项目（大多数情况下）。因此，我们将创建的新表将被称为`Books`，模型将被称为`Book`。

## 如何做...

看看以下步骤：

1.  首先，创建我们将用于新`Book`模型的`Books`表。我们建议使用数据库浏览器工具，如 phpMyAdmin、MySQL Workbench、Sequel Pro（OS X）或 Heidi SQL（Windows）。查询也可以从以下操作执行：

    ```php
    CREATE TABLE `Books` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `title` varchar(255) NOT NULL DEFAULT '',
      `author` varchar(255) NOT NULL DEFAULT '',
      PRIMARY KEY (`id`)
    );
    ```

1.  为`Book`模型创建一个新文件。在`/models`文件夹中，创建一个名为`book.php`的文件。

1.  在文件顶部添加`defined`或`die`语句。这是在 concrete5 中所有 PHP 文件中必需的，以防止脚本自行执行：

    ```php
    <?php
    defined('C5_EXECUTE') or die("Access Denied.");
    ```

1.  为`Book`模型创建类定义，扩展核心`Model`类。

    ```php
    class Book extends Model {

    }
    ```

1.  添加一个创建新书籍的方法，接受标题和作者名称作为参数：

    ```php
    public function createBook($title, $author) {
      $db = Loader::db();
           $this->title = $title;
       $this->author = $author;
           $query = 'INSERT INTO Books (title, author) VALUES (?, ?)';
       $values = array($title, $author);
           $db->execute($query, $values);
       $this->id = mysql_insert_id();
           return $this->id;
    }
    ```

1.  添加另一个方法来通过 ID 加载书籍：

    ```php
    public function loadById($id) {
       $db = Loader::db();
       $query = 'SELECT * FROM Books WHERE id = ?';
           $books = $db->getAll($query, $id);
         // get the first result (there should be only one)
       $book = $books[0];
           // set each property of the result to the current object
       $this->id = $book['id'];
       $this->title = $book['title'];
       $this->author = $book['author'];
    }
    ```

1.  最后，添加一个获取书籍标题的方法：

    ```php
    public function getTitle() {
       return $this->title;
    }
    ```

1.  保存 `book.php`。现在 `Book` 模型已经创建并保存，让我们尝试使用它来写入和读取数据库。为了简化，我们将这段代码放在 `/config/site_post.php` 中。

1.  使用 `Loader` 类加载 `Book` 模型：

    ```php
    Loader::model('book');
    ```

1.  创建 `Book` 的新实例：

    ```php
    $book = new Book();
    ```

1.  使用我们之前声明的 `createBook` 函数向数据库中添加一本新书：

    ```php
    $book->createBook('concrete5 Cookbook', 'David Strack');
    ```

1.  使用我们声明的函数获取书籍的标题。

    ```php
    $title = $book->getTitle();
    ```

## 它是如何工作的...

让我们回顾一下我们刚刚编写的代码。首先，我们确保在数据库中创建了一个名为 `Books` 的表。这个表有三个列，一个 `ID` 字段（每次创建行时自动递增），一个 `title` 字段和一个 `author` 字段。

接下来，我们在 `/models/book.php` 中创建了一个名为 `Book` 的类。这个类将代表 `Books` 表中单个记录的一个实例。该类扩展了核心 `Model` 类（它是在 `/concrete/core/libraries/model.php` 中定义的一个基本实现）。然后我们添加了一个方法来向数据库中添加新书。`createBook` 方法首先加载数据库对象，然后将 `title` 和 `author` 赋值给当前的 `Book` 对象。执行 `INSERT` 查询后，该方法使用 `mysql_insert_id()` 函数设置书籍的 `ID`。最后，函数返回我们刚刚添加的行的新的 `ID`。

接下来添加了 `loadById` 函数。这个函数将接受一个 `ID` 作为参数，然后在数据库上执行一个 `SELECT` 查询以找到对应的行。注意，在这个演示中，我们从未实际验证记录是否存在。在实际应用中，你将想要确保考虑到这一点。一旦我们加载了 `Book` 对象，我们将三个属性（`ID`、`title` 和 `author`）赋值给类对象。

最后，我们添加了一个简单的获取器函数来返回书籍的标题。请注意，这个函数只有在 `Book` 类被加载后才会工作，无论是通过调用 `loadById`，还是通过使用 `createBook` 函数创建一个新的书籍。

## 还有更多...

应该很明显，通过创建处理数据对象的模型，开发者可以节省大量工作并保持代码整洁。而不是让 SQL 语句散布在代码中，所有与数据库相关的逻辑都包含在模型中。这允许快速轻松地创建和访问数据库行。

## 参见

+   *加载数据库对象* 的配方

+   *从数据库中检索数据* 的配方

+   *将数据写入数据库* 的配方

+   *使用预处理语句防止 SQL 注入* 的配方

# 使用活动记录从数据库中读取

现在我们已经创建了一个 `Model` 类，很容易看出这对想要保持其代码库干净和可维护的开发者来说可以是一个巨大的好处。然而，我们可以通过使用活动记录更进一步。现实是，在先前的食谱中，这些任务可以用不到一半的代码来完成。

如果你熟悉 Ruby on Rails 或 CakePHP 等框架，那么你可能已经对活动记录有一些经验。**活动记录** 是一种使用包含创建、更新、读取和删除功能以及映射到数据库中记录的属性的类的约定。

在这个食谱中，我们将探讨如何使用活动记录从数据库中加载模型。

## 准备工作

我们将使用前面食谱中数据库中的相同 `Books` 表。如果你需要创建此表，请在你的数据库上运行以下 SQL：

```php
CREATE TABLE `Books` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL DEFAULT '',
  `author` varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
);
```

此外，请确保表中存在 ID 为 `1` 的记录。如果你已经完成了前面的食谱，它应该已经存在。

## 如何做到这一点...

看看以下步骤：

1.  如果 ID 为 `1` 的记录不存在，请在 `/models/book.php` 中创建一个 `Book` 类。

1.  确保文件 `book.php` 的内容如下：

    ```php
    <?php
    defined('C5_EXECUTE') or die("Access Denied.");
    class Book extends Model {
       public $_table = 'Books';
       public function getTitle() {
          return $this->title;
       }
    }
    ```

1.  现在，让我们写一点代码来让这个模型变得生动起来。对于这个演示，我们只需在 `/config/site_post.php` 中编写代码，因为这只是临时的示例代码。

1.  加载模型：

    ```php
    Loader::model('book');
    ```

1.  创建 `Book` 类的新实例：

    ```php
    $book = new Book();
    ```

1.  加载 ID 为 `1` 的书籍：

    ```php
    $book->load('id = ?', '1');
    ```

1.  获取书籍的标题：

    ```php
    $title = $book->getTitle();
    ```

1.  将标题输出到屏幕：

    ```php
    echo $title;
    exit;
    ```

## 它是如何工作的...

如你所见，`Book` 模型不包含任何 SQL，但它仍然能够对数据库进行查找并加载 ID 为 `1` 的书籍。`Book` 类扩展了 `Model` 类，而 `Model` 类又扩展了 `ADOdb_Active_Record` 类。`ADOdb_Active_Record` 类实现了几个函数，以提供典型的读取、更新、创建和删除功能。

需要注意的一个重要变化是，在类体开始处，我们添加了一个名为 `$_table` 的成员变量。如果我们省略了它，一切仍然会正常工作。实际上，`ADOdb_Active_Record` 类进行了一些基本的屈折变化（将单数类名（`Book`）转换为复数数据库表名（`Books`））。然而，屈折变化基本上只擅长在类名末尾添加 `s`，这就是全部。更复杂的复数化可能会失败，所以始终指定表名是一个好主意。

`ADOdb_Active_Record` 类的 `load` 函数期望的语法与在 SQL `WHERE` 语句中找到的语法相同。编写 `load('id = ?')` 等同于编写以下预编译语句：

```php
SELECT * FROM Books WHERE id = ?
```

然后 `load` 函数将所有属性分配给对象。很明显，当在 concrete5 中执行简单的 **CRUD** 任务时，这可以消除大量的代码。

## 相关内容

+   *创建自定义模型类* 食谱

+   *使用预处理语句防止 SQL 注入* 菜谱

# 使用活动记录写入数据库

活动记录也可以用于写入数据库。在这个菜谱中，我们将使用之前菜谱中创建的 `Book` 类向数据库添加新的书籍记录。

## 准备工作

确保已创建 `Book` 模型和 `Books` 表。请参阅 *使用活动记录从数据库中读取* 菜谱，了解该类应该如何创建。

## 如何做...

看看以下步骤：

1.  我们将在 `/config/site_post.php` 中输入代码，所以请打开该文件在您的代码编辑器中。

1.  加载模型：

    ```php
    Loader::model('book');
    ```

1.  创建 `Book` 类的新实例：

    ```php
    $book = new Book();
    ```

1.  设置书籍的标题：

    ```php
    $book->title = 'The Wind in the Willows';
    ```

1.  设置书籍的作者：

    ```php
    $book->author = 'Kenneth Grahame';
    ```

1.  保存新的书籍数据：

    ```php
    $book->save();
    exit;
    ```

## 它是如何工作的...

在将 `title` 和 `author` 属性设置为 `Book` 对象后，调用 `save` 函数将自动将项目保存到数据库中，并创建新行。

## 相关内容

+   *创建自定义模型类* 菜谱

+   *使用活动记录从数据库中读取* 菜谱

# 使用活动记录更新数据库记录

更新数据库项与创建它们非常相似。在这个菜谱中，我们将加载 ID 为 `1` 的书籍并更改其标题。

## 准备工作

此菜谱使用与之前的 *使用活动记录从数据库中读取* 菜谱相同的 `Book` 模型和 `Books` 表。在开始此菜谱之前，请确保这些文件已就位。

## 如何做...

看看以下步骤：

1.  在您的代码编辑器中打开 `/config/site_post.php`。

1.  加载模型：

    ```php
    Loader::model('book');
    ```

1.  创建一个书籍类的新实例：

    ```php
    $book = new Book();
    ```

1.  通过其 ID 加载书籍：

    ```php
    $book->load('id = ?', '1');
    ```

1.  更改书籍的标题：

    ```php
    $book->title = 'New Title';
    ```

1.  将更改保存到数据库中：

    ```php
    $book->update();
    exit;
    ```

## 它是如何工作的...

一旦加载了书籍记录，对象的 `ID` 属性就会被设置。当调用 `update` 时，`Model` 类使用这个 `ID` 来更新正确的记录。如果对象的 `ID` 在调用 `update` 时不存在或未设置，将创建一个新的记录。

## 相关内容

+   *创建自定义模型类* 菜谱

+   *使用活动记录从数据库中读取* 菜谱

# 使用活动记录在数据库中搜索

通常，Web 应用程序将需要检索多个记录。在这个菜谱中，我们将继续使用存储在数据库中的 `Books` 概念，并检索数据库中所有书籍的数组。

## 准备工作

与之前的其他菜谱一样，我们使用相同的 `Book` 模型和 `Books` 表，来自之前的 *使用活动记录从数据库中读取* 菜谱。在开始此菜谱之前，请确保这些文件已就位。

## 如何做...

看看以下步骤：

1.  在您的代码编辑器中打开 `/config/site_post.php`。

1.  加载模型：

    ```php
    Loader::model('book');
    ```

1.  创建 `Book` 模型的新实例：

    ```php
    $book = new Book();
    ```

1.  查找所有书籍记录：

    ```php
    $books = $book->find('1=1');
    ```

1.  遍历数组并输出每本书的标题：

    ```php
    foreach ($books as $b) {
       echo $b->getTitle().'<br />';
    }
    exit;
    ```

## 它是如何工作的...

`Model` 类的 `find` 函数搜索数据库，并以数组的形式返回结果对象。由于 `find` 函数已经在查询中包含了 `WHERE`，因此我们需要通过一个真值条件来过滤，以找到所有记录，所以这里 `1=1` 就可以完成这个任务。

## 更多...

`find` 函数还支持使用预处理语句来搜索表。这里，我们将查找标题中包含 `'concrete5'` 的书籍：看看以下：

```php
$books = $book->find('title LIKE "%?%"', 'concrete5');
```

## 参见

+   *创建自定义模型类* 的菜谱

+   *使用活动记录从数据库中读取* 的菜谱

# 使用活动记录和模型类删除对象

使用活动记录进行 CRUD 操作的最后一个方面是删除条目。在这个菜谱中，我们将加载 ID 为 `1` 的书籍并将其从数据库中删除。

## 准备工作

我们将使用与之前 *使用活动记录从数据库中读取* 菜谱中相同的 `Book` 模型和 `Books` 表。确保在开始此菜谱之前文件已经就绪。

## 如何做...

看看以下步骤：

1.  在你的代码编辑器中打开 `/config/site_post.php`。

1.  加载模型：

    ```php
    Loader::model('book');
    ```

1.  创建模型的新实例：

    ```php
    $book = new Book();
    ```

1.  加载 ID 为 `1` 的书籍。

    ```php
    $book->load('id = ?', '1');
    ```

1.  删除那本书：

    ```php
    $book->delete();
    exit;
    ```

## 它是如何工作的...

调用 `Model` 类的 `delete` 函数将在数据库上运行 `DELETE` 查询，永久删除与该条目关联的记录。使用你喜欢的 SQL 浏览工具，你可以看到记录已经被从数据库中删除。

## 参见

+   *创建自定义模型类* 的菜谱

+   *使用活动记录从数据库中读取* 的菜谱

# 使用活动记录定义关系

关系型数据库系统（如 MySQL）的主要特性是能够通过关系将数据链接到其他数据。考虑一个存储美国州和城市的数据库。每个州可以有多个城市，但每个城市只属于一个州。这些关系，“拥有多个”和“属于”，可以使用 concrete5 的活动记录支持来定义。

在这个菜谱中，我们将链接两个表，`States` 和 `Cities`，并使用这些关系来检索相关数据。

## 准备工作

首先，我们需要为 `Cities` 和 `States` 两个表创建表。使用以下 SQL 来填充这些表（此 SQL 也可从本书的网站上下载）：

```php
CREATE TABLE `Cities` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL DEFAULT '',
  `state_id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
);
INSERT INTO `Cities` (`id`, `name`, `state_id`)
VALUES
  (1,'New York',2),
  (2,'Buffalo',2),
  (3,'Green Bay',1),
  (4,'Chicago',3),
  (5,'San Francisco',4),
  (6,'San Diego',4),
  (7,'Oakland',4);
CREATE TABLE `States` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
);
INSERT INTO `States` (`id`, `name`)
VALUES
  (1,'Wisconsin'),
  (2,'New York'),
  (3,'Illinois'),
  (4,'California');
```

## 如何做...

看看以下步骤：

1.  在 `/models/state.php` 中为州创建一个模型：

    ```php
    class State extends Model {}
    ```

1.  在 `/models/city.php` 中为 `city` 创建一个模型：

    ```php
    class City extends Model {}
    ```

1.  在 `state` 模型文件底部（类体外部），定义与 `city` 的关系：

    ```php
    Model::ClassHasMany('State', 'Cities','state_id');
    ```

1.  保存这两个类文件。

1.  现在，我们将获取属于加利福尼亚州（ID 为 `4` 的州）的城市。

1.  在你的代码编辑器中打开 `/config/site_post.php`，这样我们就可以运行一些代码。

1.  加载 `state` 模型：

    ```php
    Loader::model('state');
    ```

1.  创建 `state` 模型的新实例：

    ```php
    $state = new State();
    ```

1.  加载 ID 为 `4` 的 `state`（在我们的数据示例中是加利福尼亚州）：

    ```php
    $state->load('id = ?', '4');
    ```

1.  使用活动记录关系获取州的城市：

    ```php
    $cities = $state->Cities;
    ```

1.  遍历城市并输出每个城市的名称：

    ```php
    foreach ($cities as $city) {
       echo $city->name .'<br />';
    }
    exit;
    ```

## 它是如何工作的...

在`Model`类上调用静态函数`ClassHasMany`接受三个参数。第一个参数是我们将此关系分配给类的名称（在这种情况下，`State`）。第二个参数是包含子内容的数据库表名称。第三个参数是外键的名称，它将用于确定哪些行是父类的子行。

## 还有更多...

也可以用类似的方式声明`属于`关系。

```php
Model::ClassBelongsTo('City','State','state_id','id');
```

## 参见

+   *创建自定义模型类*的配方

+   *使用活动记录从数据库中读取*的配方
