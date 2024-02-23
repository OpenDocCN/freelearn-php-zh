# 第二章。MVC 中的模型

在本章中，我们将讨论 MVC 结构中的模型是什么，它的目的是什么，它在 SOLID 设计模式中的作用是什么，Laravel 如何定义它，以及 Laravel 的模型层和 Eloquent ORM 的优势。我们还将讨论处理数据的 Laravel 类。

以下是本章将涵盖的主题列表：

+   模型的含义

+   在坚实的 MVC 设计模式中的模型角色

+   模型和模型实例

+   Laravel 如何定义模型

+   Laravel 的与数据库相关的类

# 什么是模型？

模型是模型-视图-控制器设计模式的一部分，我们可以简单地描述为处理从相应层接收的数据并将其发送回这些层的设计模式的层。这里需要注意的一点是，模型不知道数据来自何处以及如何接收数据。

简而言之，我们可以说模型实现了应用程序的业务逻辑。模型负责获取数据并将其转换为其他应用程序层可以管理的更有意义的数据，并将其发送回相应层。模型是应用程序的域层或业务层的另一个名称。

！[什么是模型？]（Image00004.jpg）

# 模型的目的

应用程序中的模型管理所有动态数据（不是硬编码的并来自数据库驱动程序的任何数据），并让应用程序的其他相关组件了解更改。例如，假设您的数据库中有一篇新闻文章。如果您手动从数据库更改它，当调用路由时 - 由于此请求 - 控制器在路由处理程序的请求后请求模型上的数据，并且控制器从模型接收更新后的数据。结果，它将此更新后的数据发送到视图，最终用户从响应中看到更改。所有这些与数据相关的交互都是模型的任务。模型处理的这个“数据”并不总是与数据库相关。在某些实现中，模型还可以用于处理一些临时会话变量。

在一般 MVC 模式中，模型的基本目的如下：

+   使用指定（数据库）驱动程序获取数据

+   验证数据

+   存储数据

+   更新数据

+   删除数据

+   创建条件关系

+   监视文件 I / O

+   与第三方网络服务交互

+   处理缓存和会话

如您所见，如果您始终遵循 MVC 模式，模型将涵盖应用程序逻辑的很大一部分。在现代框架中，有一个常见的错误是用户在学习设计模式时常犯的错误。他们通常会混淆模型与模型实例。尽管它们非常相似，但它们有不同的含义。

# 模型实例

在您的应用程序中，通常会有多个数据结构需要管理。例如，假设您正在运行一个博客。在一个简单的博客系统中，有作者、博客文章、标签和评论。假设您想要更新一篇博客文章；您如何定义要更新的数据是针对博客文章的？这就是模型实例派上用场的地方。

模型实例是大多数情况下从应用程序的模型层扩展的简单类。这些实例将应用程序的每个部分的数据逻辑分离开来。在我们的示例中，我们有四个部分要处理（用户、帖子、标签和评论）。如果我们要使用模型来处理这些，我们至少要创建四个实例（我们将在本章的* Eloquent ORM *下的*关系*部分中解释为什么至少是四个而不是确切的四个）。

！[模型实例]（Image00005.jpg）

从图中可以看出，控制器与模型实例交互以获取数据。由于模型实例是从模型本身扩展而来的，因此控制器可以自定义它或向过程添加其他层（如验证），而不是使用原始模型输出。

假设您想获取用户名为`George`的用户。如果没有从模型实例到数据库添加验证层，则`username`参数将直接传递到数据库，这可能是有害的。如果您在模型实例上添加了验证层（检查`username`参数是否为干净的字符串），即使参数是 SQL 注入代码，它也将首先通过验证层进行过滤，而不是直接传递到数据库，然后被检测为有害代码。然后，模型实例将向控制器返回一个消息或异常，指出参数无效。然后，控制器将向视图发送相应的消息，然后消息将显示给最终用户。在此过程中，应用程序甚至可以触发事件以记录此尝试。

# Laravel 中的模型

如果您还记得，我们在本章前面提到模型需要处理许多重要的工作。Laravel 4 不直接使用 MVC 模式，而是进一步扩展了该模式。例如，验证——在坚实的 MVC 模式中是模型的一部分——有它自己的类，但它并不是模型本身的一部分。数据库连接层为每个数据库驱动程序都有自己的类，但它们并没有直接打包在模型中。这为模型带来了可测试性、模块化和可扩展性。

Laravel 模型结构更专注于直接处理数据库过程，并与其他目的分开。其他目的被归类为门面。

要访问数据库，Laravel 有两个类。第一个是流畅的查询构建器，第二个是 Eloquent ORM。

## 流畅的查询构建器

流畅是 Laravel 4 的查询构建器类。流畅的查询构建器使用后端的 PHP 数据对象处理基本的数据库查询操作，并且可以与几乎任何数据库驱动程序一起使用。假设您需要将数据库驱动程序从 SQLite 更改为 MySQL；如果您使用流畅编写了查询，那么除非您使用了`DB::raw()`编写了原始查询，否则您大多数情况下不需要重写或更改代码。流畅在幕后处理这个。

让我们快速看一下 Laravel 4 的 Eloquent 模型（可以在`Vendor\Laravel\Framework\src\Illuminate\Database\Query`文件夹中找到）：

```php
<?php namespace Illuminate\Database\Query;

use Closure;
use Illuminate\Support\Collection;
use Illuminate\Database\ConnectionInterface;
use Illuminate\Database\Query\Grammars\Grammar;
use Illuminate\Database\Query\Processors\Processor;

class Builder {
    //methods and variables come here
}
```

如您所见，Eloquent 模型使用一些类，如`Database`、`ConnectionInterface`、`Collection`、`Grammar`和`Processor`。所有这些都是为了在后端标准化数据库查询，如果需要，缓存查询，并将输出作为集合对象返回。

以下是一些基本示例，展示了查询的外观：

+   要从`users`表中获取所有名称并逐个显示它们，使用以下代码：

```php
    $users = DB::table('users')->get();
    foreach ($users as $user)
    {
        var_dump($user->name);
    }
    ```

`get()`方法以集合的形式从表中获取所有记录。通过`foreach()`循环，记录被循环，然后我们使用`->name`（一个对象）访问每个名称列。如果要访问的列是电子邮件，则会像`$user->email`一样。

+   要从`users`表中获取名为`Arda`的第一个用户，使用以下代码：

```php
    $user = DB::table('users')->where('name', 'Arda')->first();
    var_dump($user->name);
    ```

`where()`方法使用给定参数过滤查询。`first()`方法直接从第一个匹配的元素中返回单个项目的集合对象。如果有两个名为`Arda`的用户，则只会捕获第一个并将其设置为`$user`变量。

+   如果要在`where`子句中使用`OR`语句，可以使用以下代码：

```php
    $user = DB::table('users')
    ->where('name', 'Arda')
    ->orWhere('name', 'Ibrahim')
    ->first();
    var_dump($user->name);
    ```

+   要在`where`子句中使用操作符，应在要过滤的列名和变量之间添加以下第三个参数：

```php
    $user = DB::table('users')->where('id', '>', '2')->get();
    foreach ($users as $user)
    {
        var_dump($user->email);
    }
    ```

+   如果你使用偏移和限制，执行以下查询：

```php
    $users = DB::table('users')->skip(10)->take(5)->get();
    ```

这在 MySQL 中产生了`SELECT` `* FROM` users `LIMIT 10,5`。`skip($integer)`方法将为查询设置一个偏移量，`take($ integer)`将限制输出为已设置为参数的自然数。

+   你还可以使用`select()`方法限制要获取的内容，并在 Fluent Query Builder 中轻松使用以下`join`语句：

```php
    DB::table('users')
       ->join('contacts', 'users.id', '=', 'contacts.user_id')
       ->join('orders', 'users.id', '=', 'orders.user_id')
       ->select('users.id', 'contacts.phone', 'orders.price');
    ```

这些方法简单地将`users`表与 contacts 连接，然后将 orders 与 users 连接，然后获取`contacts`表中的用户 ID 和电话列，以及`orders`表中的价格列。

+   你可以使用闭包函数轻松地按参数分组查询。这将使您能够轻松编写更复杂的查询，如下所示：

```php
    DB::table('users')
        ->where('name', '=', 'John')
        ->orWhere(function($query)
        {
            $query->where('votes', '>', 100)
                  ->where('title', '<>', 'Admin');
        })
        ->get();
    ```

这将产生以下 SQL 查询：

```php
    select * from users 
       where name = 'John' 
       or 
       (votes > 100 and title <> 'Admin')
    ```

+   你还可以在查询构建器中使用聚合（如`count`、`max`、`min`、`avg`和`sum`）：

```php
    $users = DB::table('users')->count();
    $price = DB::table('orders')->max('price');
    ```

+   有时，这样的构建器可能不够，或者你可能想要运行原始查询。你也可以将原始查询包装在 Fluent 中，如下所示：

```php
    $users = DB::table('users')
         ->select(
    array(
    DB::raw('count(*) as user_count'),
    'status',
    )
    )
         ->where('status', '<>', 1)
         ->groupBy('status')
         ->get();
    ```

+   要将新数据插入表中，请使用`insert()`方法：

```php
    DB::table('users')->insert(
        array('email' => 'me@ardakilicdagi.com', 'points' => 100)
    ); 
    ```

+   要从表中更新行，请使用`update()`方法：

```php
    DB::table('users')
    ->where('id', 1)
    ->update(array('votes' => 100)); 
    ```

+   要从表中删除行，请使用`delete()`方法：

```php
    DB::table('users')
    ->where('last_login', '2013-01-01 00:00:00')
    ->delete(); 
    ```

+   利用`CachingIterator`，它使用`Collection`类，Fluent Query Builder 也可以在调用`remember()`方法时缓存结果：

```php
    $user = DB::table('users')
    ->where('name', 'Arda')
    ->remember(10)
    ->first();
    ```

一旦调用了这个查询，它就会被缓存 10 分钟；如果再次调用这个查询，它将直接从缓存中获取，而不是从数据库中获取，直到 10 分钟过去。

## Eloquent ORM

Eloquent ORM 是 Laravel 中的 Active Record 实现。它简单、强大，易于处理和管理。

对于每个数据库表，你都需要一个新的 Model Instance 来从 Eloquent 中受益。

假设你有一个`posts`表，并且你想要从 Eloquent 中受益；你需要导航到`app/models`，并将此文件保存为`Post.php`（表名的单数形式）：

```php
<?php class Post extends Eloquent {}
```

就是这样！你已经准备好从 Eloquent 方法中受益了。

Laravel 允许你将任何表分配给任何 Eloquent Model Instance。这不是必需的，但以相应表的单数形式命名 Model Instances 是一个好习惯。这个名字应该是它所代表的表名的单数形式。如果你必须使用不遵循这个一般规则的名字，你可以通过在 Model Instance 内部设置受保护的`$table`变量来这样做。

```php
<?php Class Post Extends Eloquent {

   protected $table = 'my_uber_posts_table';

}
```

通过这种方式，你可以将表分配给任何所需的 Model Instance。

### 注意

不需要将实例添加到`app`中的`models`文件夹中。只要在`composer.json`中设置了`autoload`路径，你可以完全摆脱这个文件夹，并将其添加到任何你喜欢的地方。这将在编程过程中为你的架构带来灵活性。

让我们快速看一下 Laravel 4 的以下`Model`类，我们刚刚从中扩展出来的（位于`Vendor\Laravel\Framework\src\Illuminate\Database\Eloquent`文件夹中）：

```php
<?php namespace Illuminate\Database\Eloquent;

use DateTime;
use ArrayAccess;
use Carbon\Carbon;
use LogicException;
use JsonSerializable;
use Illuminate\Events\Dispatcher;
use Illuminate\Database\Eloquent\Relations\Pivot;
use Illuminate\Database\Eloquent\Relations\HasOne;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Support\Contracts\JsonableInterface;
use Illuminate\Support\Contracts\ArrayableInterface;
use Illuminate\Database\Eloquent\Relations\Relation;
use Illuminate\Database\Eloquent\Relations\MorphOne;
use Illuminate\Database\Eloquent\Relations\MorphMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Query\Builder as QueryBuilder;
use Illuminate\Database\Eloquent\Relations\MorphToMany;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;
use Illuminate\Database\ConnectionResolverInterface as Resolver;

abstract class Model implements ArrayAccess, ArrayableInterface, JsonableInterface, JsonSerializable {
   //Methods and variables come here
}
```

Eloquent 使用`Illuminate\Database\Query\Builder`，这是我们之前描述的 Fluent Query Builder，它的方法在其中定义。由于这一点，所有可以在 Fluent Query Builder 中定义的方法也可以在 Eloquent ORM 中使用。

如你所见，所有使用的类都根据其目的进行了拆分。这为架构带来了更好的**抽象**和**可重用性**。

### 关系

Eloquent ORM 除了 Fluent Query Builder 之外还有其他好处。主要好处是 Model Instance Relations，它允许 Fluent Query Builder 轻松地与其他 Model Instances 建立关系。假设你有`users`和`posts`表，并且你想要获取 ID 为`5`的用户发布的帖子。关系建立后，可以轻松地使用以下代码获取这些帖子的集合：

```php
User::find(5)->posts;
```

这难道不是更容易了吗？有三种主要的关系类型：一对一，一对多和多对多。除了这些，Laravel 4 还有 has-many-through 和 morph-to-many（多对多多态）关系：

+   **一对一关系**：当两个模型彼此只有一个元素时使用。比如你有一个`User`模型，它应该只有一个元素在你的`Phone`模型中。在这种情况下，关系将被定义如下：

```php
    //User.php model
    Class User Extends Eloquent {

       public function phone() {
          return $this->hasOne('Phone'); //Phone is the name of Model Instance here, not a table or column name
       }

    }

    //Phone.php model
    Class Phone Extends Eloquent {

       public function user() {
          return $this->hasOne('User');
       }

    }
    ```

+   **一对多关系**：当一个模型有另一个模型的多个元素时使用。比如你有一个带有分类的新闻系统。一个分类可以有多个项目。在这种情况下，关系将被定义如下：

```php
    //Category.php model
    class Category extends Eloquent {

       public function news() {
          return $this->hasMany('News'); //News is the name of Model Instance here
       }

    }

    //News.php model
    class News extends Eloquent {

       public function categories() {
          return $this->belongsTo('Category');
       }

    }
    ```

+   **多对多关系**：当两个模型彼此有多个元素时使用。比如你有`Blog`和`Tag`模型。一篇博客文章可能有多个标签，一个标签可能被分配给多篇博客文章。对于这种情况，需要使用一个中间表和多对多关系来定义关系：

```php
    //Blog.php Model
    Class Blog Extends Eloquent {

       public function tags() {
          return $this->belongsToMany('Tag', 'blog_tag'); //blog_tag is the name of the pivot table
       }

    }

    //Tag.php model
    Class Tag Extends Eloquent {

       public function blogs() {
          return $this->belongsToMany('Blog', 'blog_tag');
       }

    }
    ```

Laravel 4 为这些已知的关系添加了一些灵活性和额外的关系。它们是“has-many-through”和“多态关系”。

+   **Has-many-through 关系**：这更像是快捷方式。比如你有一个`Country`模型，`User`模型和`Post`模型。一个国家可能有多个用户，一个用户可能有多个帖子。如果你想访问特定国家的用户创建的所有帖子，你需要定义关系如下：

```php
    //Country.php Model
    Class Country Extends Eloquent {

       public function posts() {
          return $this->hasManyThrough('Post', 'User');
       }

    }
    ```

+   **多态关系**：这在 Laravel v4.1 中有。比如你有一个`News`模型，`Blog`模型和`Photo`模型。这个`Photo`模型为`News`和`Blog`都保存了图片，但是如何关联或识别特定的照片是为博客还是帖子？这可以很容易地完成。需要设置如下：

```php
    //Photo.php Model
    Class Photo Extends Eloquent {

       public function imageable() {
          return $this->morphTo(); //This method doesn't take any parameters, Eloquent will understand what will be morphed by calling this method
       }

    }

    //News.php Model
    Class News Extends Eloquent {

       public function photos() {
          return $this->morphMany('Photo', 'imageable');
       }

    }

    //Blog.php Model
    Class Blog Extends Eloquent {

       public function photos() {
          return $this->morphMany('Photo', 'imageable');
       }

    }
    ```

关键字`imageable`，用来描述图片的所有者，不是必须的；它可以是任何东西，但你需要将它设置为一个方法名，并将它作为`morphMany`关系定义的第二个参数。这有助于我们理解我们将如何访问照片的所有者。这样，我们可以轻松地从`Photo`模型中调用它，而不需要了解它的所有者是`Blog`还是`News`：

```php
    $photo = Photo::find(1);
    $owner = $photo->imageable; //This brings either a blog collection or News according to its owner.
    ```

### 注意

在这个例子中，你需要在`Photo`模型的表中添加两个额外的列。这些列是`imageable_id`和`imageable_type`。`imageable`部分是变形方法的名称，后缀的 ID 和类型是将定义它将变形为的项目的确切 ID 和类型的键。

### 批量赋值

在创建新的模型实例（插入或更新数据时），我们传递一个变量，它被设置为一个带有属性名称和值的数组。然后这些属性通过批量赋值分配给模型。如果我们盲目地将所有输入添加到批量赋值中，这将成为一个严重的安全问题。除了查询方法，Eloquent ORM 还帮助我们进行批量赋值。比如你不希望`User`模型（`Object`）中的电子邮件列被以任何方式更改（黑名单），或者你只希望`Post`模型中的标题和正文列被更改（白名单）。这可以通过在你的模型中设置受保护的`$fillable`和`$guarded`变量来完成：

```php
//User.php model
Class User Extends Eloquent {
   //Disable the mass assignment of the column email 
   protected $guarded = array('email');

}

//Blog.php model
Class User Extends Eloquent {
   //Only allow title and body columns to be mass assigned
   protected $fillable = array('title', 'body');

}
```

### 软删除

假设你有一个`posts`表，假设这个表中的数据很重要。即使从模型运行`delete`命令，你也希望保留已删除的数据在你的数据库中以防万一。在这种情况下，你可以使用 Laravel 的软删除。

软删除实际上并不从表中删除行；相反，它在数据实际删除时添加一个键。当进行软删除时，一个名为`deleted_at`的新列将填充一个时间戳。

要启用软删除，您需要首先在表中添加一个名为`deleted_at`的时间戳列（您可以通过在迁移中添加`$table->softDeletes()`来完成），然后在您的模型实例中将一个名为`$softDelete`的变量设置为`true`。

以下是软删除的示例模型实例：

```php
//Post.php model
Class Post Extends Eloquent {
   //Allow Soft Deletes
   protected $softDelete = true;

}
```

现在，当您在此模型中运行`delete()`方法时，它将不会实际删除该列，而是会向其添加一个`deleted_at`时间戳。

现在，当您运行`all()`或`get()`方法时，软删除的列将不会被列出，就好像它们实际上已被删除一样。

在这种删除后，您可能希望获取包括软删除行在内的结果。要做到这一点，请使用`withTrashed()`方法如下：

```php
$allPosts = Post::withTrashed()->get(); //These results will include both soft-deleted and non-soft-deleted posts.
```

在某些情况下，您可能只想获取软删除的行。要做到这一点，使用`onlyTrashed()`方法如下：

```php
$onlySoftDeletedPosts = Post::onlyTrashed()->get();
```

要恢复软删除的行，请使用`restore()`方法。要恢复所有软删除的帖子，请运行以下代码：

```php
$restoreAllSoftDeletedPosts = Post::onlyTrashed()->restore();
```

要彻底删除表中的软删除行，请使用`forceDelete()`方法如下：

```php
$forceDeleteSoftDeletedPosts = Post::onlyTrashed()->forceDelete();
```

从表中获取行（包括软删除）时，您可能想要检查它们是否已被软删除。通过在集合行上运行`trashed()`方法来进行此检查。此方法将返回一个布尔值。如果为 true，则表示该行已被软删除。

```php
//Let's fetch a post without the soft-delete checking:
$post = Post::withTrashed()->find(1);
//Then let's check whether it's soft deleted or now if($post->trashed()) {
return 'This post is soft-deleted'; } else {
   return 'This post is not soft-deleted';
}
```

### 急切加载

Eloquent ORM 还通过**急切加载**为 N+1 查询问题提供了一个简洁的解决方案。假设您有一个查询和循环，如下所示：

```php
$blogs = Blog::all();
foreach($blogs as $blog) {
   var_dump($blog->images());
}
```

在这种情况下，为了访问这些图片，在后端的每个循环中执行一个查询。这将极大地耗尽数据库，为了防止这种情况，我们将在查询中使用`with()`方法。这将在后端获取所有的博客和图片，将它们关联起来，并直接作为一个集合提供。参考以下代码：

```php
$blogs = Blog::with('images')->get();
foreach($blogs as $blog) {
   var_dump($blog->images);
}
```

这样，查询速度将会更快，使用的资源也会更少。

### 时间戳

Eloquent ORM 的主要优势在于将`$timestamps`设置为`true`（这是默认值）；您将有两列，第一列是`created_at`，第二列是`updated_at`。这两列将数据的创建和最后更新时间作为时间戳，并在每行的创建或更新时自动更新它们。

### 查询范围

假设您因为它是您的应用程序中常用的子句之一而多次重复一个`where`条件，并且这个条件有意义。假设您想要获取所有具有超过 100 次浏览的博客文章（我们将其称为热门帖子）。如果不使用范围，您将以以下格式获取帖子：

```php
$popularBlogPosts = Blog::where('views', '>', '100')->get();
```

然而，在这个例子中，您将一遍又一遍地在应用程序中重复这个过程。那么，为什么不将其设置为一个范围呢？您可以使用 Laravel 的查询范围功能轻松实现这一点。

将以下代码添加到您的`Blog`模型中：

```php
public function scopePopular($query) {
   return $query->where('views', '>', '100');
}
```

完成这些操作后，您可以使用以下代码轻松使用您的范围：

```php
$popularBlogPosts = Blog::popular()->get();
```

您还可以将帖子链接如下：

```php
$popularBlogPosts = Blog::recent()->popular()->get();
```

### 访问器和修改器

Eloquent ORM 的一个特性是访问器和修改器。假设您的表上有一个名为`name`的列，并且在调用这个列时，您想要传递 PHP 的`ucfirst()`方法来将其名称大写。只需将以下代码添加到模型中即可完成：

```php
public function getNameAttribute($value) {
    return ucfirst($value);
}
```

现在，让我们考虑相反的情况。每次保存或更新名称列时，您都希望将 PHP 的`strtolower()`函数传递给该列（您希望修改输入）。只需将以下代码添加到模型中即可完成：

```php
public function setNameAttribute($value) {
    return strtolower($value);
}
```

请注意，方法名称应该是驼峰式命名，即使列名是`snake_cased`。如果您的列名是`first_name`，则 getter 方法名称应该是`getFirstNameAttribute`。

### 模型事件

模型事件在 Laravel 设计模式中扮演着重要的角色。使用模型事件，您可以在事件触发后立即调用任何方法。

假设您为评论设置了缓存，并且希望在每次删除评论时刷新缓存。您如何捕获评论的删除事件并在那里执行某些操作？应用程序中是否有多个地方可以删除此类评论？是否有一种方法可以精确捕获“删除”或“已删除”事件？在这种情况下，模型事件非常有用。

模型包含以下方法：`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring` 和 `restored`。

每当第一次保存新项目时，将触发创建和已创建事件。如果您正在更新模型上的当前项目，则将触发 `updating` / `updated` 事件。无论您是创建新项目还是更新当前项目，都将触发 `saving` / `saved` 事件。

如果从 `creating`、`updating`、`saving` 或 `deleting` 事件中返回 `false`，则操作将被取消。

例如，让我们检查用户是否已创建。如果其名字是 `Hybrid`，我们将取消创建。要添加此条件，请在您的 `User` 模型中包含以下代码行：

```php
User::creating(function($user){
    if ($user->first_name == 'Hybrid') return false;
});
```

### 模型观察者

模型观察者与模型事件非常相似，但方法略有不同。它不是在模型内部定义所有事件（`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring` 和 `restored`），而是将事件的逻辑“抽象”到不同的类中，并使用 `observe()` 方法“观察”它。假设我们有一个如下所示的模型事件：

```php
User::creating(function($user){
    if ($user->first_name == 'Hybrid') return false;
});
```

为了保持抽象，最好将所有这些事件封装起来，并将它们的逻辑与模型分开。在观察者中，这些事件将如下所示：

```php
class UserObserver {

   public function creating($model){
       if ($model->first_name == 'Hybrid') return false;
    }

    public function saving($model)
    {
        //Another model event action here
    }

}
```

您可以想象，您可以将这个类放在应用程序的任何位置。您甚至可以将所有这些事件分组在一个单独的文件夹中，以获得更好的架构模式。

现在，您需要将此事件 `Observer` 类注册到模型。可以使用以下简单命令完成：

```php
 **User::observe(new UserObserver);** 

```

这种方法的主要优势是您可以在多个模型中使用观察者，并以这种方式向一个模型注册多个观察者。

## 迁移

迁移是管理数据库版本控制的简单工具。假设有一个地方需要向表中添加新列，或者回滚到以前的状态，因为您做错了事情或应用程序的链接断开了。没有迁移，这些都是繁琐的任务，但有了迁移，您的生活将变得更加轻松。

使用迁移的各种原因如下：

+   您将受益于这种版本控制系统。如果出现错误或需要回滚到以前的状态，您只需使用迁移的一个命令即可完成。

+   使用迁移进行更改将带来灵活性。编写的迁移将适用于所有支持的数据库驱动程序，因此您无需为不同的驱动程序重写数据库代码。Laravel 将在后台处理这一切。

+   它们非常容易生成。使用 Laravel `php` 客户端的迁移命令，称为 `artisan`，您可以管理应用程序的所有迁移。

以下是迁移文件的样子：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class CreateNewsTable extends Migration {

        /**
        * Run the migrations.
        */
        public function up()
        {
                //
        }

        /**
        * Reverse the migrations.
        */
        public function down()
        {
                //
        }

}
```

`up()` 方法在向前运行迁移时运行（新迁移）。`down()` 方法在向后运行迁移时运行，意味着它会反转或重置（反转并重新运行）迁移。

通过 `artisan` 命令触发这些方法后，它会运行 `up` 或 `down` 方法，与 `artisan` 命令的参数相对应，并返回消息的状态。

## 数据库种子数据填充器

假设您编写了一个博客应用程序。您需要展示它的功能，但没有示例博客文章来展示您编写的出色博客。这就是种子数据填充器派上用场的地方。

数据库 seeder 是一些简单的类，它们在指定的表中填充随机数据。seeder 类有一个简单的方法叫做`run()`来进行这种填充。以下是 seeder 的样子：

```php
<?php

class BlogTableSeeder extends Seeder {

  public function run()
  {
    DB::table('blogs')->insert(array(
      array('title' => 'My Title'),
      array('title' => 'My Second Title'),
      array('title' => 'My Third Title')
    ));
  }

}
```

当您使用`artisan`命令从终端调用这个类时，它会连接到数据库并用给定的数据填充它。尝试完成后，它会通过终端向用户返回有关填充状态的命令消息。

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，直接将文件发送到您的电子邮件。

# 读累了记得休息一会哦~

**公众号：古德猫宁李**

+   电子书搜索下载

+   书单分享

+   书友学习交流

**网站：**[沉金书屋 https://www.chenjin5.com](https://www.chenjin5.com)

+   电子书搜索下载

+   电子书打包资源分享

+   学习资源分享

# 总结

在本章中，我们已经了解了 MVC 模式中模型的作用，以及 Laravel 4 如何通过扩展其角色到各种类来“定义”模型。我们还通过示例看到了 Laravel 4 的模型组件的功能。

在下一章中，我们将学习视图的作用，以及它如何在 Laravel 4 中使用 MVC 模式与最终用户和应用程序的其他方面进行交互。
