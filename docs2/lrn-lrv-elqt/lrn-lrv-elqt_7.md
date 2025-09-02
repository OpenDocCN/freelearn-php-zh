# 第七章. 无 Laravel 的 Eloquent…

我们的旅程即将结束，英雄。你从最基础的知识学到了关于 Eloquent 的所有内容，包括模型、关系和其他主题。你可能开始喜欢它，并考虑在下一个项目中实现它。

事实上，创建一个不使用单个 SQL 查询的应用程序很有吸引力。也许你也向你的老板展示了它，并说服他/她将其用于下一个生产项目。

我为你感到骄傲，英雄！

然而，有一个小问题。是的，下一个项目并不那么新。它已经存在了，尽管如此，它并没有使用 Laravel！你开始颤抖。这真是太遗憾了，因为你上周一直在学习这个新的 ORM，一个非常酷的 ORM，然后继续前进。

好吧，别抱怨了！总有解决办法！你是一名开发者！此外，解决方案并不难找到。如果你想，你可以使用 Eloquent 而不使用 Laravel。是的，真的！

实际上，Laravel 不是一个 *单体* 框架。它由几个独立的组件组成，这些组件组合在一起构建了更大的东西。然而，没有任何东西阻止你在另一个应用程序中仅使用所选的包。

一个非常酷的想法！

那么，我们将在本章中看到什么？

首先，我们将探索数据库包的结构，看看里面有什么。然后，你将学习如何为你的项目单独安装 `illuminate/database` 包，以及如何为其首次使用进行配置。

然后，你将遇到一些示例。首先，我们将查看 **Eloquent ORM**。你将学习如何定义模型并使用它们。

做完这件事后，作为额外的补充，我将向你展示如何使用 `查询构建器`（记住，`"illuminate/database"`包不仅仅是 Eloquent）。也许你也会喜欢 `Schema Builder` 类。我会涵盖它，不用担心！

由你负责！

我们将涵盖以下内容：

+   探索目录结构

+   安装和配置数据库包

+   使用 ORM

+   使用查询和模式构建器

+   摘要

# 探索目录结构

如我之前所述，要在你的应用程序中使用 Eloquent 而不使用 Laravel 的关键步骤是使用 `"illuminate/database"` 包。

那么，在我们安装它之前，让我们先稍微了解一下。

你可以在这里看到包的内容：[`github.com/illuminate/database`](https://github.com/illuminate/database)。

所以，你可能会看到以下内容：

| 文件夹 | 描述 |
| --- | --- |
| `Capsule` | **capsule 管理器**是一个基本组件。它实例化服务容器并加载一些依赖项。 |
| `Connectors` | 数据库包可以与各种数据库系统通信。例如，SQLite、MySQL 或 PostgreSQL。每种数据库类型都有自己的连接器。这是你将找到它们的文件夹。 |
| `Console` | 数据库包不仅仅是 Eloquent 加上一堆连接器。在这个特定的文件夹中，你可以找到与控制台命令相关的一切，例如 `artisan db:seed` 或 `artisan migrate`。 |
| `Eloquent` | 每一个 Eloquent 类都放置在这里。 |
| `Migrations` | 不要与 `Console` 文件夹混淆。与迁移相关的所有类都存储在这里。当你你在终端中输入 `artisan migrate` 时，你正在调用放置在这里的一个类。 |
| `Query` | 查询构建器放置在这里。 |
| `Schema` | 与 Schema Builder 相关的一切都放置在这里。 |

在主文件夹中，你还可以找到一些其他文件。但是，不用担心，你不需要知道它们是什么。

如果你打开 `composer.json` 文件，看看下面的 `"require"` 部分：

```php
"require": {
  "php": ">=5.4.0",
  "illuminate/container": "5.1.*",
  "illuminate/contracts": "5.1.*",
  "illuminate/support": "5.1.*",
  "nesbot/carbon": "~1.0"
},
```

如你所见，数据库包有一些先决条件，你无法避免。然而，容器相当小，对于 `contracts`（只是几个接口）和 `"illuminate/support"` 也是如此。

### 注意

Eloquent 使用 `Carbon` ([`github.com/briannesbitt/Carbon`](https://github.com/briannesbitt/Carbon)) 以更智能的方式处理日期。所以，如果你是第一次看到这个，感到困惑，不用担心！一切都很正常。

现在你已经知道了这个包中可以找到什么，让我们看看如何安装它并首次配置它。

# 安装和配置数据库包

让我们从设置开始。首先，我们将像往常一样使用 composer 安装包。然后，我们将配置胶囊管理器以开始。

## 安装包

安装 `"illuminate/database"` 包非常简单。

你所需要做的只是将 `"illuminate/database"` 添加到你的 `composer.json` 文件的 `"require"` 部分，如下所示：

```php
"require": {

  "illuminate/database": "5.0.*",

},
```

然后在你的终端中输入 `composer update`，等待几秒钟。

另一种方法是在项目文件夹中使用快捷方式包含它，显然是从终端开始的：

```php
composer require illuminate/database
```

无论你选择哪种方法，你都已经安装了包。

## 配置包

是时候使用胶囊管理器了！在你的项目中，你可以使用类似以下内容开始：

```php
use Illuminate\Database\Capsule\Manager as Capsule;

$capsule = new Capsule;

$capsule->addConnection([
  'driver'    => 'mysql',
  'host'      => 'localhost',
  'database'  => 'database',
  'username'  => 'root',
  'password'  => 'password',
  'charset'   => 'utf8',
  'collation' => 'utf8_unicode_ci',
  'prefix'    => '',
]);

// Set the event dispatcher used by Eloquent models... (optional)
use Illuminate\Events\Dispatcher;
use Illuminate\Container\Container;
$capsule->setEventDispatcher(new Dispatcher(new Container));
```

我使用的配置语法与 `config/database.php` 配置文件中可以找到的完全相同。唯一的区别是这次你明确地使用胶囊管理器的一个实例来做所有的事情。

在代码的第二部分，我正在设置事件分配器。如果你项目需要事件，你必须这样做。

然而，默认情况下此包不包括事件，所以你将不得不手动将 `"illuminate/events"` 依赖项添加到你的 `composer.json` 文件中。

现在，最后一步！

将此代码添加到你的设置文件中：

```php
// Make this Capsule instance available globally via static methods... (optional)
$capsule->setAsGlobal();

// Setup the Eloquent ORM... (optional; unless you've used setEventDispatcher())
$capsule->bootEloquent();
```

在胶囊管理器上调用 `setAsGlobal()` 后，你可以将其设置为全局组件，以便使用静态方法。你可能喜欢它，也可能不喜欢；选择权在你。最后一行启动 Eloquent，所以你需要它。

然而，这也是一个可选的指令。在某些情况下，你可能只需要查询构建器。

然后，就没有其他事情要做了！你的应用程序现在已经配置了数据库包（以及 Eloquent）！

# 使用 ORM

在非 Laravel 应用程序中使用 Eloquent ORM 并不是很大的变化。你所要做的就是像你习惯的那样声明你的模型。然后，你需要调用它并像你习惯的那样使用它。

这就是我所说的完美例子：

```php
use Illuminate\Database\Eloquent\Model;

class Book extends Model {

  ...

  // some attributes here…
  protected $table = 'my_books_table';

  // some scopes here...
  public function scopeNewest()
  {
    // query here...
  }

  ...

}
```

就像你在 Laravel 中做的那样，你使用的包是相同的。所以，不用担心！如果你想使用你刚刚创建的模型，那么请使用以下内容：

```php
$books = Book::newest()->take(5)->get();
```

这也适用于关系、观察者等等。一切都是一样的。

### 注意

为了精确地使用数据库包和 ORM，你会做与 Laravel 中相同的事情；请记住，以遵循 PSR-4 自动加载约定的方式设置项目结构。

# 使用查询和模式构建器

这不仅仅关于 ORM；有了数据库包，你还可以使用查询和模式构建器。让我们来探索一下！

## 查询构建器

查询构建器也非常容易使用。这次唯一的区别是，你将通过胶囊管理对象传递，就像这样：

```php
$books = Capsule::table('books')
             ->where('title', '=', "Michael Strogoff")
             ->first();
```

然而，结果仍然是相同的。

此外，如果你喜欢 Laravel 中的 DB 门面，你可以以相同的方式使用胶囊管理类：

```php
$book = Capsule::select('select title, pages_count from books where id = ?', array(12));
```

## 模式构建器

在这本书的开头，我向你展示了模式构建器。你学习了如何使用它与迁移一起，但现在，在没有 Laravel 的情况下，你没有迁移。

然而，你仍然可以使用模式构建器。就像这样：

```php
Capsule::schema()->create('books', function($table)
{ 
    $table->increments(''id'); 
    $table->string(''title'', 30); 
    $table->integer(''pages_count''); 
    $table->decimal(''price'', 5, 2);.
    $table->text(''description''); 
    $table->timestamps(); 
});
```

之前，你通常调用`Schema facade`的`create()`方法。这次有点不同：你将使用`create()`方法，将其链接到`Capsule`类的`schema()`方法。

显然，你可以以这种方式使用任何 Schema 类方法。例如，你可以调用以下内容：

```php
Capsule::schema()->table('books', function($table)
{
    $table->string('title', 50)->change();
    $table->decimal('special_price', 5, 2);
});
```

你就可以开始了！

### 注意

记住，如果你想解锁一些 Schema Builder 特定功能，你需要安装其他依赖项。

例如，你想重命名一个列？你需要`doctrine/dbal`依赖包。

# 摘要

嗯，这次，进展相当快。

我决定添加这一章，因为很多人问我如何在没有 Laravel 的情况下使用 Eloquent。主要是因为他们喜欢这个框架，但他们无法将一个已经启动的项目整体迁移。

此外，我认为了解在某种程度上你可以在引擎盖下找到什么，也是很酷的。

这始终只是关于好奇心。好奇心开辟了新的道路，你可以选择以新的、更优雅的方式解决问题。

在这几页中，我只是触及了表面。我想给你一些建议：探索代码。编写好代码的最佳方式是阅读好代码。

现在，让我们进入最后一章！
