# 第四章：探索关系的世界

> *团结（和关联）使我们站立，分裂使我们倒下。*

在现实世界的背景下，一切事物都是相互关联的；例如，一辆车有一个车主，一本书有一个作者（或者可能不止一个），或者一个电子商务订单与一个或多个客户（另一个关系！）所订购的产品相关联。

实际上，一切事物都是相互关联的！

在应用开发的世界里没有区别；通常，你创建软件来解决现实世界的问题。现实世界是由相关事物构成的，所以你可能需要在你的实体之间定义许多关系。

然而，让我们明确一点：我并没有说什么新东西。只需去维基百科上搜索`实体-关系模型`。

通常，在你的学校教科书中，你可以找到三种基本的关系类型：

+   **一对一**：这是用来关联一个单一实体与另一个单一实体（例如，一个人和身份证）

+   **一对一**：这是用来定义一个实体与同一类型的其他实体之间的连接（例如，同一作者的所有书籍）

+   **多对多**：这是用来关联多个实体与许多其他实体（例如，一本书可以属于多个类别，一个类别可以包含多本书）

当然，网络开发也不例外。Eloquent 也不例外。

按照你迄今为止所看到的*约定*，Eloquent 有一个处理关系、用于定义它们的方法以及你可以采用的与它们一起工作的技术的好方法。

因此，让我们探索我们将在本章中做什么。

首先，我们将处理我们刚刚看到的基本关系类型。Eloquent 是如何处理它们的？你将发现诸如`hasMany`或`belongsTo`等强大方法的美丽。这次，没有更多的代码片段；我们将跟随我们的图书馆管理工具类的创建，定义每一个实体和每一个关系。

在基础之后，你将发现如何处理这些关系：如何查询和使用它们，以舒适和整洁的方式。此外，我们还将看到如何在数据库中插入和删除相关模型，或者更新现有模型。

有时候，处理多对多关系可能意味着存储一些特定于该关系的特定数据。Eloquent 有一个非常有用的属性名为**pivot**，你可以用它来查询所需的枢纽表。

因此，这次有很多东西要看看！然而，还没有结束！Eloquent 提供了两种其他你可以使用的*关系*：*通过 has many*和多态*多对多*关系。

好吧，别再闲聊了！我现在不会泄露任何东西。跟随这一章节，你会爱上它的。

显然，我将为每个概念展示一个现实世界的例子。来吧，英雄！

+   三位一体：一对一、一对多和多对多

+   查询相关模型

+   懒加载（以及 N + 1 问题）

+   插入和更新相关模型

+   访问 *远程* 关系

+   更强大的功能！多态关系

# 三位一体——一对一、一对多、多对多

如我之前提到的，我们将从基础知识开始。所以，我们将首先看到如何定义 Eloquent 中实体之间的关系。这真的很简单，通常你只需要为每个关系添加一行代码。

## 一对一

我们的库非常关注追踪借书的人。因此，每个新用户都必须向图书馆提供一些身份证明文件数据。

现在，每个用户都有一个唯一的身份证明文件，每个文件都是独一无二的。如果你仔细想想，这是一个完美的一对一关系。当你构建数据库时，最遵循的规则告诉你，你必须首先在第一个表中添加必要的列。在这个特定例子中，我们会向用户表添加列。

然而，有人可能会说“是的，但这是一个完全不同的实体！”

此外，我们可能需要存储每个用户身份证明文件的许多详细信息：号码、类型、到期日、城市等等。

在这种情况下，你会在一个现有的表中添加四到五列。许多人不喜欢这种解决方案，所以想象一下，你有了 `User` 和 `IdentityDocument` 模型。

这是我们的默认 `User` 模型：

```php
  <?php namespace App;

  use Illuminate\Auth\Authenticatable;
  use Illuminate\Database\Eloquent\Model;
  use Illuminate\Auth\Passwords\CanResetPassword;
  use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
  use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

  class User extends Model implements AuthenticatableContract, CanResetPasswordContract {

    use Authenticatable, CanResetPassword;

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'users';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'email', 'password'];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token'];

  }
```

以下是我们 `IdentityDocument` 模型：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class IdentityDocument extends Model {

    //

  }
```

现在，让我们看看如何定义它们之间的连接。对于一对一关系，使用的方法是 `hasOne()` 和 `belongsTo()`。`hasOne()` 方法的使用方式如下：

```php
  $this->hasOne('App\IdentityDocument');
```

此外，`belongsTo()` 方法的使用方式如下：

```php
  $this->belongsTo('App\User');
```

语法是合理的，对吧？一个用户有一个文件，一个文件属于某个用户！我知道你在想什么：你不能随意在类中放置这个方法调用。

实际上，你必须定义一个返回该方法调用的方法，如下所示：

```php
  public function identityDocument()
  {
    return $this->hasOne('App\IdentityDocument');
  }
```

同样，`belongsTo()` 必须定义为以下方式：

```php
  public function user()
  {
    return $this->belongsTo(' App\User');
  }
```

因此，最终模型的代码将如下所示：

```php
  <?php namespace App;

  use Illuminate\Auth\Authenticatable;
  use Illuminate\Database\Eloquent\Model;
  use Illuminate\Auth\Passwords\CanResetPassword;
  use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
  use Illuminate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;

  class User extends Model implements AuthenticatableContract, CanResetPasswordContract {

    use Authenticatable, CanResetPassword;

    /**
     * The database table used by the model.
     *
     * @var string
     */
    protected $table = 'users';

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'email', 'password'];

    /**
     * The attributes excluded from the model's JSON form.
     *
     * @var array
     */
    protected $hidden = ['password', 'remember_token'];

    public function identityDocument()
    {
      return $this->hasOne(' App\IdentityDocument');
    }

  }
```

`User` 类的代码如下：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class IdentityDocument extends Model {

    public function user()
    {
      return $this->belongsTo(' App\User');
    }

  }
```

哎呀！

### 究竟发生了什么？

在数据库层面，我们创建了一个包含一些数据列和 `user_id` 外键的 `identitydocuments` 表。这个外键很重要，因为它会被 Eloquent 自动用来解析关系。

如果你愿意，你可以在两个方法中指定不同的外键作为第二个参数：

```php
  $this->hasOne('App\IdentityDocument', 'another_user_external_id');
```

否则，你可以使用这个：

```php
  $this->belongsTo(' App\User', 'another_user_external_id');
```

我们指定的字段是相同的。当然，`hasOne` 方法（`User` 模型）会将其视为外键，而 `belongsTo` 方法（`IdentityDocument` 模型）会将其视为本地键。

在这两个方法中，还有一个你可以使用的第三个参数。在 `hasOne()` 中，它用于指定本地键（默认是 `id` 字段）。在 `belongsTo()` 方法中，它用于在父表上定义父键（再次，默认是 `id` 字段）。

让我们用相同的模型再举一个例子。想象一下，我们有一个名为 `IdentityDocuments` 的表，其主键名为 `documentidentifier`。此外，我们需要遵循一定的标准，我们不能使用 `user_id` 作为外部外键的名称。我们必须使用 `documentowner_id`。

没有问题。首先，你将像这样定义你的 `hasOne`：

```php
    $this->hasOne('App\IdentityDocument', 'owner_id');
```

我们不需要定义第三个参数，因为我们的 `users` 主键是 `id`。

然后，你定义 `belongsTo`：

```php
    $this->belongsTo('App\IdentityDocument', 'documentidentifier', 'owner_id');
```

现在你完成了！注意，这个概念适用于我们将要看到的其他关系方法。

## 一对多

这次比之前更容易：每本书都有一个作者，对吧？有时，一本书可能有多个作者，但让我们假设一个基本的情况。我们将要分析的第二种关系类型是一对多。每个作者可以有多本书。让我们考虑涉及到的模型：`Author` 和 `Book`。

这是 `Author` 类：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Author extends Model {

    //

  }
```

`Book` 类如下：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Book extends Model {

    //

  }
```

这些只是简单的 Eloquent 模型。现在，为了定义一个一对多关系，我们必须使用 `hasMany()` 方法。

因此，`Author` 类将看起来像这样：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Author extends Model {

    public function books()
    {
      return $this->hasMany('App\Book');
    }

  }
```

然后，你可以像之前一样使用 `belongsTo()` 方法来定义这种关系的 `inverse`：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Book extends Model {

    public function author()
    {
      return $this->belongsTo('App\Author');
    }

  }
```

所以，我们完成了！我们再次使用了 `belongsTo()`，因为 *属于* 的概念完全相同；没有差异。

在数据库结构层面，我在 `books` 表中添加了一个外部的 `author_id` 键。

所以，是的，完成了！我在之前的笔记中提到过，但我会重复一遍：记住，你可以通过在 `hasMany()` 和 `belongsTo()` 方法中指定它作为第二个参数来更改你的外键。

## 多对多

让我们想象一个多对多关系的好例子。好吧，书籍/类别的关系是完美的。事实上，想象一下 *海底两万里*，*儒勒·凡尔纳*。

这不仅是一部冒险小说，也是一部经典。所以，你需要将其归类为两个不同的类别：*经典* 和 *冒险*。我们的图书馆也可能包含另一部经典作品 *地球中心之旅*，它也是一部冒险小说。同样如此！

如你所见，这次一个多对多关系是绝对必要的。让我们来看看 Eloquent 如何处理多对多关系以及如何在模型上定义它们。

这是我们的 `Book` 模型的代码：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Book extends Model {

    public function author()
    {
      return $this->belongsTo('App\Author');
    }

  }
```

`Category` 模型的代码如下：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Category extends Model {

    //

  }
```

这次，我们没有任何 *方向* 或关系的可能 *反向*。

特别是，对于许多类别，有许多书籍。所以，在这种情况下，你需要使用的唯一方法是 `belongsToMany()`。像这样使用该方法：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Book extends Model {

    public function author()
    {
      return $this->belongsTo('App\Author');
    }

    public function categories()
    {
      return $this->belongsToMany('App\Category');
    }

  }
```

另一种方法的使用方式如下：

```php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Category extends Model {

    public function books()
    {
      return $this->belongsToMany('App\Book');
    }

  }
```

没有其他的事情！

让我们看看在数据库层面需要什么来处理这种关系。正如你很容易想象的，在这种情况下，你将不得不与一个交叉表（pivot table）一起工作。

所以，你需要在迁移文件中使用 `up()` 方法指定一个合适的额外表，如下所示：

```php
  Schema::create('book_category', function(Blueprint $table)
  {
    $this->increments('id');

    $this->integer('book_id')->unsigned();
    $this->integer('category_id')->unsigned();

    $this->text('notes');

    $this->timestamps();
  });
```

你也有一些约定要遵循。这些在这里给出：

+   表名由实体的名称组成，这些名称是单数，由下划线分隔

+   表格将包含两个以感兴趣实体命名的列（`author_id` 和 `book_id`）

### 注意

当你指定一个关系时，请记住在适当的方法调用之前使用 `return`。我知道这有点明显，但新手经常忘记这一点。

真的，真的非常重要，你必须遵循定义的约定。Laravel 和 Eloquent 可以显著改变你的工作流程时间表，但为了得到结果，你必须遵循约定。你越早这样做，感觉就会越好。

## 反向问题

在我们继续前进之前，有一些额外的事情。我们刚刚看到了如何定义一个关系及其反向。然而，如果你不定义反向，或者如果你定义了反向但没有定义 *反向的反向*，你会遇到什么后果？

实际上没有什么特别的！

最好的规则是定义你需要的关联。让我们想象这种情况：在你的软件中，你需要知道一本书的类别，但不需要知道某个类别中的所有书籍。

这是一个奇怪的情况，但这种情况确实会发生。在这种情况下，你只需在 `Book` 中定义 `categories()` 关系，无需其他。

反之，如果你的应用程序只需要从类别获取书籍列表，而不需要其他内容，你将只在 `Category` 模型中定义 `books()` 关系。仅此而已。

完成！已经涵盖了三种基本的关系类型！你不需要做更多的事情；实际上，Eloquent 会自动处理一切，所以你只需要编写代码，使用模型，提出查询。

哦，关于查询...

# 查询相关模型

现在你已经学会了如何定义你的关系，我认为你准备好学习如何查询它们了。让我们从一个非常基础的例子开始。

假设我们正在搜索特定用户的文档编号。我们将使用我们刚才看到的 `User` 和 `IdentityDocument` 实体。为了这个示例，想象你有一个名为 `identitydocuments` 的表，其中包含以下列：

+   `编号`: 这表示文档编号

+   `类型`: 这表示文档类型

+   `due_date`: 这表示文档的到期日期

+   `城市`: 这表示文档发布的城市

这是获取文档身份编号的代码，从一个 `User` 实例开始：

```php
  $user = \App\User::where('first_name', '=', 'Francesco')->where('last_name', '=', 'Malatesta')->first();

  $identityDocumentNumber = $user->identityDocument->number;
```

如果你输出 `$identityDocumentNumber`，你会读取所需的信息。不错，对吧？

嗯，这就是 Laravel 和 Eloquent 处理查询你的关系的方式。一旦你定义了它，你所要做的就是像访问一个简单的属性或方法一样访问它。

所有其他查询将由 Laravel 自动执行。实际上，请遵循以下简单说明：

```php
  $user = \App\User::where('first_name', '=', 'Francesco')->where('last_name', '=', 'Malatesta')->first();

  $identityDocumentNumber = $user->identityDocument->number;
```

你刚刚执行了这些查询：

```php
  // the user Francesco Malatesta as an ID = 1...
  select * from users where first_name = 'Francesco' AND last_name = 'Malatesta';

  select * from identitydocuments where user_id = 1
```

现在将结果放入 `$identityDocumentNumber`。对于一对一关系来说，这是显而易见的；然而，对于一对一多关系也是如此。

让我们考虑另一个例子：老牌的 Jules 将是一个完美的选择。假设我们想要获取我们拥有的所有儒勒·凡尔纳书籍的列表，代码如下：

```php
  $author = \App\Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  foreach($author->books as $book)
  {
    echo $book->title . <br/>;
  }

  // outputs:
  //
  // Journey to the Center of the Earth
  // Twenty Thousand Leagues Under the Sea
  // Around the World in Eighty Days
  // Michel Strogoff
```

### 注意

正如我之前告诉你的，你可以通过一个简单的属性或方法调用来访问你的关系。有什么区别呢？嗯，通过方法调用，你可以进行一些过滤，并且之前看到的一切都是为了得到期望的结果。实际上，你可以在关系上发起一个查询。

想象一下，我们想要获取所有标题中包含*the*的书籍。以下是代码：

```php
  $author = \App\Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  $theBooks = $author->books()->where('title', 'LIKE', '%the%')->get();

  foreach($theBooks as $book)
  {
    echo $book->title . <br/>;
  }

  // outputs:
  //
  // Journey to the Center of the Earth
  // Twenty Thousand Leagues Under the Sea
  // Around the World in Eighty Days
```

很酷，对吧？但这还没有结束，这只是触及了表面！

## 访问交叉表

在处理多对多关系时，并不仅仅是定义几个外部键。你可以选择在你的交叉表中添加额外的数据，以便存储实体之间特定连接的一些信息。

你已经知道如何创建一个交叉表，但如何访问它呢？这并不复杂：你只需要使用你关系的`pivot`属性。让我们用一个例子来说明，这个例子是我们之前创建的书籍/类别关系。

1.  首先，你必须定义你想要从表中获取哪个属性，修改你的模型中的`belongsToMany()`调用：

    ```php
      return $this->belongsToMany('App\Category')->withPivot('created_at', 'notes');
    ```

1.  然后，你的代码应该是这样的：

    ```php
      $book = App\Book::find(23);

      foreach($book->categories as $category)
      {
        echo 'Association Date: ' . $category->pivot->created_at;
        echo 'Association Notes: ' . $category->pivot->notes;
      } 
    ```

在这个小例子中，我们只是打印了所有我们将特定类别附加到`id = 23`的书籍上的日期。作为额外信息，我们还打印了一些额外的注释。这意味着在交叉表中，我们有`created_at`和`notes`字段。

作为快捷方式，你也可以使用：

```php
  return $this->belongsToMany('App\Category')->withTimestamps();
```

这用于你只想从交叉表中导入时间戳数据时。

## 查询关系

Eloquent 允许你查询一个关系。换句话说，你可以根据某个关系的存在来获取一些结果。想象一下，我们想要获取数据库中至少有一本书的所有作者。

使用 Eloquent，我们可以这样做：

```php
  $authorsWithABook = \App\Author::has('books')->get();
```

在这种情况下，你必须使用`has`方法，指定你想要检查的关系的所需方法。只有当至少找到一本相关书籍时，作者才会被添加到`$authorsWithABook`中。

如果你不喜欢这种*布尔*方法，不要担心；让我们看看如何找到数据库中至少有五本书的每个作者。

```php
  $authorsWithAtLeastFiveBooks = \App\Author::has('books', '>=', 5)->get();
```

是的，你可以指定第二个和第三个参数作为操作符和比较项，分别用于这个*计数检查*。

我知道，我知道，很酷，但还不够。好吧，那么获取所有至少有一本书在 1864 年出版的作者怎么办？

这里我们来了，这次使用`whereHas`方法：

```php
  $authorsWithABookFromThe1864 = \App\Author::whereHas('books', function($q)
  {
      $q->where('year', '=', 1864);

  })->get();
```

如你所见，你可以以一种相当优雅的方式做到这一点。指定的第一个参数是你想要查询的关系的名称。第二个参数是一个闭包，它接受一个`$q`查询参数，你可以使用它来定义条件。

### 注意

你刚才看到的关于一对一关系的相同概念也适用于多对多关系。

# 预加载（以及 N + 1 问题）

每个强大的工具都必须明智地使用。Eloquent 中的关系也不例外。实际上，使用 Eloquent 最常见的问题之一就是 *N + 1* 问题。为了解释它，我将像往常一样使用一个例子。

假设我正在展示前 100 本书的一些数据。从这些数据开始，我还想打印每本书的作者姓名。

使用我们之前学到的知识，以下是代码：

```php
  $books = \App\Book::take(100)->get();

  foreach($books as $book)
  {
    $author = $book->author;

    echo $author->first_name . ' ' . $author->last_name;
  }
```

即使语法很简单，在底层，Eloquent 正在进行 101 个查询！第一个查询是获取 100 本书列表，然后对每一本书进行查询以获取作者。这并不完全符合性能友好，对吧？

别担心，有解决方案！

## 基本预加载

预加载解决了你的问题。这次使用 `Book` 模型的 `with()` 方法，如下所示：

```php
  $books = \App\Book::with('author')->take(100)->get();

  foreach($books as $book)
  {
    $author = $book->author;

    echo $author->first_name . ' ' . $author->last_name;
  }
```

现在，执行查询的数量将急剧下降到两个，使用 `where in`！

```php
  select * from books;
  select * from authors where id in (1, 2, 3, ...);
```

如果你愿意，你还可以在你的最终结果中包含多个关系。让我们也包含每本书的分类数据！

```php
  $books = \App\Book::with('author', 'categories')->take(100)->get();

  foreach($books as $book)
  {
    $author = $book->author;

    echo 'Author: ' . $author->first_name . ' ' . $author->last_name;

    echo 'Categories:';

    foreach($book->categories as $category)
    {
      echo $category->name . ', ';
    }
  }
```

似乎还不够，你还可以包含来自 *嵌套关系* 的数据。

假设你正在获取应用中的分类列表。然后，你想要包含每个分类的书籍数据，并且对于每个相关书籍，你想要包含作者的数据。

你只需这样做：

```php
  $categories = \App\Categories::with('books.author')->get();

  foreach($categories as $category)
  {
    echo $category->name;

    foreach($category->books as $book)
    {
      echo 'Title: ' . $book->title;
      echo 'Author: ' . $book->author->first_name . ' ' . $book->author->last_name;
    }
  }
```

## 高级预加载

如果你想更好地控制预加载，你可以定义一些约束或条件。假设你正在获取一个作者列表。从这个列表中，你想要获取从最古老到最新出版的每一本书。

你可以这样操作：

```php
  $authors = \App\Author::with(['books' => function($query)
  {
    $query->orderBy('year', 'asc');

  }])->get();
```

你所要做的就是指定所需的预加载关系作为关联数组的元素，使用这种语法：

```php
  ['relationship' => function($query){

    // conditions here, using the $query object

  }]
```

## 懒预加载

有时候你会使用预加载，但不是每次都需要。有时候你需要它，有时候不需要。

如果你愿意，你可以在下一刻手动预加载一个特定的关系。

如何？可以使用 `load()` 方法来完成：

```php
  $books = Book::all();

  // some operations here...

  $books->load('author', 'categories');
```

语法与之前看到的一样；你所要做的就是指定所需的关联关系作为参数。

当然，你还可以使用以下方式定义条件：

```php
  $books->load(['categories' => function($query)
  {
      $query->orderBy('name', 'asc');

  }]);
```

同样的方式，不多也不少！

### 注意

记住，预加载是一个解决许多性能问题的绝佳方案。特别是在我最初的实验中，它通过让我达到更低的查询数量而极大地帮助了我。仅举一个例子，在 Laravel-Italia 论坛上，我展示了包含一些回复、信息和线程作者数据的线程列表，只需三个查询即可。

# 插入和更新相关模型

到目前为止，你已经学会了如何定义关系并查询它们，以便从相关模型中获取数据。然而，你也可以轻松地插入和更新相关模型。

让我们选择一个真正基础的例子来开始：想象一下我们要添加一本新书（*《米哈伊尔·斯特罗戈夫》，*儒勒·凡尔纳*）。我们必须指定以下一些细节：

```php
  $book = new Book;

  $book->title = 'Michael Strogoff';

```

然后，我们指定正确的作者 ID：

```php
$author = Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  $book = new Book;

  $book->title = 'Michael Strogoff';
  // other data...

  $book->author_id = $author->id;

  // and finally...
  $book->save();
```

## save()和 associate()方法

让我们明确一点；这确实很有效。然而，这并不是你能做的最好的方法。实际上，使用 Eloquent，你可以使用一些其他特定方法来处理相关模型。

让我们使用关系上的`save()`方法重写这个例子。同时，我们将使用关联数组作为构造函数参数来分配属性。

```php
  $author = Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  $author->books->save(new Book([
    'title' => 'Michael Strogoff',
    // other attributes...
  ]));
```

完成！这只需要几条指令。`save()`方法将自动为刚刚传入作为参数的书籍设置`author_id`键。

然而，这不仅仅是一个节省时间的技巧；如果你仔细阅读代码，你会注意到我们实际上正在创建非常易于阅读的代码。我们不是设置外部键，而是以一种更*物理*的方式处理关系，我们将书籍保存为某个作者书籍集合中的一个元素。大不相同！

你也可以使用`saveMany()`方法与对象数组做同样的事情：

```php
  $author = Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  $author->books->saveMany([
    new Book(['title' => 'Michael Strogoff']),
    new Book(['title' => 'The Mysterious Island']),
    new Book(['title' => 'Off on a Comet'])
  ]);
```

如前所述，`saveMany`方法将为数组中传递的每个`Book`实例设置外部的`author_id`键。你还可以更新现有关系，更改其与另一个模型的关系。

在这种情况下，你必须使用`associate()`方法。看看这个例子：

```php
  $wrongAuthor = Author::where('first_name', '=', 'Jules')->where('last_name', '=', 'Verne')->first();

  $wrongAuthor->books->save(new Book([
    'title' => 'The Alchemist'
  ]));

  // oops! wrong author!

  $book = Book::where('title', '=', 'The Alchemist')->first();
  $rightAuthor = Author::where('first_name', '=', 'Paulo')->where('last_name', '=', 'Coelho')->first();

  // done!
  $rightAuthor->books->associate($book);
```

发生了什么？在示例的第一部分，我将《炼金术士》分配给了儒勒·凡尔纳。在沙漠中忏悔了一年之后，我回来找到正确的作者（通过使用`$rightAuthor`变量），然后使用`associate()`方法在书籍关系上操作。

第一个传入的参数是你想要与之工作的模型实例。

### 小贴士

因此，规则相当简单：在插入新记录时使用`save()`方法，在更新现有记录时使用`associate()`方法。

记住，《炼金术士》不是由儒勒·凡尔纳写的，而是由保罗·科埃略写的！

## 那么多对多关系呢？

我们所看到的一切对于一对一或多对一的关系来说都很棒。然而，对于多对多关系呢？这次机制略有不同；不是在复杂性的意义上，而是在语法上，更多的是在语法上。然而，让我们用一个例子来说明。

+   我们有几本书：《炼金术士》和《地球中心之旅》。

+   我们还有一些类别：*科幻*、*冒险*和*经典*。

正如我在本章第一部分提到的，这是一个多对多的关系。我们如何注册《炼金术士》和《冒险》或《地球中心之旅》和《经典》之间的关系呢？

为了在单独的连接表中访问我们的数据，我们需要其他专用方法。我知道这看起来很愚蠢且无聊，但请使用你的数据库管理工具查看当你与数据交互时数据库中发生了什么。熟悉这个过程总是一个好主意。

然而，你将首先使用的是`attach()`方法：

```php
  $book = new Book();

  $book->title = "The Alchemist";
  // other attributes...

  $book->save();

  // after save() call, an id is created.

  $category = new Category();

  $category->name = "Adventure";

  $category->save();

  // and now...

  $category->books->attach($book->id);
```

如你容易看到的，`attach()`方法是你可以调用的另一个关系方法。当然，只适用于多对多关系！在这种情况下，它需要一个参数：我想与该类别关联的书籍的主 ID。

如果你正在你的连接表中存储一些额外数据，你也可以添加一个关联数组作为第二个参数：

```php
  $category = Category::where('name', '=', 'Opera')->first();

  $book = Book::where('title', '=', 'Journey to the Center of the Earth')->first();

  $category->books->attach(
    $book->id, 
    ['notes' => "Well, I'm not so sure about this..."]
  );
```

关联数组遵循`attribute_name => attribute_value`格式，就像往常一样。

就像你可以*附加*一样，你也可以*解除*。如果你想删除两个模型实例之间的多对多关系，请使用以下`detach()`方法：

```php
  $category = Category::where('name', '=', 'Opera')->first();

  $book = Book::where('title', '=', 'Journey to the Center of the Earth')->first();

  // oh, come on...
  $category->books->detach($book->id);
```

所以，你已经完成了！

此外，`attach()`和`detach()`方法都支持数组作为参数，而不是简单的整数。

```php
  $category->books->attach(
    [4, 8, 15, 17, 22, 42]
  );

  $category->books->detach([17, 22]);

  $category->books->attach([16, 23 => ['notes' => 'be careful next time...']]);
```

## `sync()`方法

这是一种处理多对多关系的真正酷的方法。然而，附加和解除东西有时可能会有些无聊。让我们尝试`sync()`方法：

```php
  $category->books->sync(
    [4, 8, 15, 17, 22, 42]
  );

  $category->books->sync(
    [4, 8, 15, 16, 23, 42]
  );
```

迷惑？让我解释一下。`sync`方法自动*同步*关系数据，接受一个 ID 数组。元素一个接一个地检查关系之前是否已创建，并在必要时设置（或取消设置）它们。让我们想象一下，连接表`book_category`是空的；这是第一条指令：

```php
  $category->books->sync(
    [4, 8, 15, 17, 22, 42]
  );
```

此指令将在所选类别和 ID 为 4、8、15、17、22 和 42 的书籍之间建立连接。然而，这里是第二个方法调用：

```php
  $category->books->sync(
    [4, 8, 15, 16, 23, 42]
  );
```

它会检查一切，并计算正差和负差。书籍 17 和 22 不再在数组中。关系将自动*解除*。相反，书籍 16 和 23 将通过*附加*方法添加，这是一个非常酷的实用方法，可以节省你大量时间！

你显然可以使用之前使用的方法向连接表中添加数据：

```php
  $category->books->sync(
    [4, 8, 15, 16, 23, 42 => ['notes' => 'We either live together.... or die alone.']]
  );
```

# 访问远程关系

另一个非常有趣的 Eloquent 特性是使用`hasManyThrough()`方法定义（然后访问）一个远程关系。

什么？我看到你有点困惑。没问题：让我们再举一个例子，这个例子与我们的实际环境略有不同。

想象一下，你正在为某个研究团队编写一个研究管理应用程序。在这个软件中，每个用户都将能够创建一个新的研究实体，然后添加一些章节到该研究中，例如：

+   一个与`Research`实体存在一对一关系的`User`实体

+   `Research`实体与`Section`实体存在一对一关系

好的。首先，对于模型，你可以写点这样的东西：

```php
  // file: app/User.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class User extends Model {

    public function researches()
    {
      return $this->hasMany('App\Research');
    }

  }

  // file: app/Research.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Research extends Model {

    public function user()
    {
      return $this->belongsTo('App\User');
    }

    public function sections()
    {
      return $this->hasMany('App\Section');
    }

  }

  // file: app/Section.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Section extends Model {

    public function research()
    {
      return $this->belongsTo('App\Research');
    }

  }
```

作为基本设置，它可以工作。现在，如果我们想访问某个用户添加的每个部分呢？可能你会使用类似这样的东西：

```php
  // getting $author with id = 1...
  $author = Author::find(1);

  // getting all $author researches...
  $researches = $author->researches;

  $allSections = [];

  // iterating to get all sections
  foreach($researches as $research)
  {
    $allSections[] = $research->sections;
  }
```

`$allSections`数组将包含用户添加的每个部分。使用 Eloquent，如果你想，你可以使用我之前提到的`hasManyThrough()`方法创建一个快捷方式。

你所要做的就是将其放入如下所示的 User 模型中：

```php
  // file: app/User.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class User extends Model {

    public function researches()
    {
      return $this->hasMany('App\Research');
    }

    public function sections()
    {
      return $this->hasManyThrough('App\Section', 'App\Research');
    }

  }
```

如果你愿意，你可以指定外部键（对于当前和*中间*实体）作为第三个和第四个参数：

```php
  return $this->hasManyThrough('App\Section', 'App\Research', 'user_id', 'research_id');
```

在某些这样的特定情况下，这是一个非常有用的快捷方式。享受它！

# 更强大的多态关系

可能你正在想 Eloquent 很酷，非常强大。

嗯，是的。然而，有时`hasMany()`或`belongsToMany()`还不够。在开发流程中的某些情况下，你将不得不处理更复杂的关系，这可能涉及两个以上的实体。

因此，作为本章的最后一部分，我将讨论多态关系。像往常一样，即使它们学习起来并不复杂，我仍会用许多详细的例子来涵盖，以便让你完全理解整个概念。

让我们从简单的多态关系开始。

## 简单的多态关系

当你有一个实体可以属于一个实体或另一个实体时，可以使用简单的多态关系。

因此，这是我们的第一个例子。想象一下你正在创建一个电子商务应用程序。你将能够上传一些照片：可以是产品、类别或博客文章。

这意味着我们首先将有四个独立的实体：

+   照片

+   产品

+   类别

+   文章

现在，让我们准备一些代码框架如下：

```php
  // file: app/Photo.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Photo extends Model {

    //

  }

  // file: app/Product.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Product extends Model {

    //

  }

  // file: app/Category.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Category extends Model {

    //

  }

  // file: app/Post.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Post extends Model {

    //

  }
```

现在，你可以使用`morphTo()`和`morphMany()`方法定义这个多态关系。

+   `morphTo()`方法由与所有其他类相关联的类使用。

+   `morphMany()`方法由`owner`类调用。

因此，让我们像这样编辑我们的模型：

```php
  // file: app/Photo.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Photo extends Model {

    public function imageable()
    {
      return $this->morphTo();
    }

  }

  // file: app/Product.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Product extends Model {

    public function photos()
    {
      return $this->morphMany('Photo', 'imageable');
    }

  }

  // file: app/Category.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Category extends Model {

    public function photos()
    {
      return $this->morphMany('Photo', 'imageable');
    }

  }

  // file: app/Post.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Post extends Model {

    public function photos()
    {
      return $this->morphMany('Photo', 'imageable');
    }

  }
```

完成！等等，那个既用作方法名称又用作字符串参数的`imageable`是什么？这是一个你可以自己选择的名字：然而，看看我使用的表结构，以了解。

```php
  products
      id - integer
      name - string

  categories
      id - integer
      name - string

  posts
      id - integer
      name - string

  photos
      id - integer
      path - string
      imageable_id - integer
      imageable_type - string
```

照片表有两个特殊字段：`imageable_id`和`imageable_type`。这是一个简单的外部键，唯一的区别是，对于这个照片表中的元素，你可以计算不同类型的*所有者*。

因此，在`imageable_id`中，你将放入所有者的 ID，在`imageable_type`中，放入所有者类名！

如果一张照片属于一个产品，你将在`imageable_type`列中看到`Product`，如果照片属于一个类别，那么就是`Category`，依此类推。

显然，处理这种关系非常简单。以下是一个例子：

```php
  // getting a sample product...
  $product = App\Product::find(3);

  foreach($product->photos as $photo)
  {
    // working with photos here...
  }
```

这适用于其他每个实体！

```php
  // getting a sample category
  $category = App\Category::find(42);

  foreach($category->photos as $photo)
  {
    // working with category photos here...
  }
```

最后，你也可以*反转*这些关系。如果你有一张照片，想知道*谁*是所有者，你所要做的就是：

```php
  $photo = App\Photo::find(23);

  // getting the owner...
  var_dump($photo->imageable);
```

无论所有者的类是什么，Eloquent 都会自动解析实例并将其返回给你。如果 *所有者* 是一篇博客文章，你将得到这篇博客文章。很简单！

## 多对多多态关系

如果可以将简单的多态关系定义为一种 *特殊* 的多对一关系，那么多对多关系在多对多多态关系中找到了等价。

正如你在前面的文本中看到的，它的工作方式与多对多关系完全一样。唯一的区别是你可以 *连接* 某个实体与更多实体。

对于我们的例子，这次，让我们回到我们的图书馆管理系统。在本章的开头，你看到了三个主要实体：

+   作者

+   书籍

+   分类

在书籍和分类之间，存在多对多关系。一本书可以属于多个分类。同样，一个分类可以包含多本书。现在，想象一下，你想要 *扩展* 这个概念到作者。

让我们以那位久经考验的朱尔斯为例。他写了冒险书籍，所以他很容易被归类为冒险作家。

多对多多态关系是处理这种情况的最佳方式。这次，你将不得不使用 `morphMany()` 和 `morphedByMany()` 方法：

```php
  // file: app/Author.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Author extends Model {

    public function categories()
    {
      return $this->morphToMany('App\Category', 'categorizable');
    }

  }

  // file: app/Book.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Book extends Model {

    public function categories()
    {
      return $this->morphToMany('App\Category', 'categorizable');
    }

  }

  // file: app/Category.php
  <?php namespace App;

  use Illuminate\Database\Eloquent\Model;

  class Category extends Model {

    public function authors()
    {
      return $this->morphedByMany('App\Author', 'categorizable');
    }

    public function books()
    {
      return $this->morphedByMany('App\Book', 'categorizable');
    }

  }
```

当然，对于每个多对多关系，你都需要一个具有类似以下结构的枢纽表：

```php
  categorizables
      tag_id - integer
      categorizable_id - integer
      categorizable_type – string
```

像往常一样，请密切关注名称和约定。`categorizable` 的第二个参数对于 `morphToMany()` 和 `morphedByMany()` 方法与你在枢纽表中指定的相同，即 `categorizable_id` 和 `categorizable_type`。

### 注意

此外，使用的表名是该术语的复数形式（`categorizable` 和 `categorizables`）。

在此设置完成后，你可以在代码中使用这种关系，如下所示：

```php
  // getting a sample author
  $author = App\Author::find(30);

  // accessing categories...
  var_dump($author->categories);

  // getting a sample book
  $book = App\Book::find(60);

  // accessing categories... in the same way!
  var_dump($book->categories);
```

在创建表时，你需要添加特定的 `_id` 和 `_type` 列，例如 `categorizable_id` 和 `categorizable_type`。

### 注意

在 Schema Builder 中，如果你想使用 `$table->morphs('categorizable')`，可以这样做。它将自动添加你需要的列，只需指定你想要的 `-able` 名称即可。

# 摘要

好吧，我认为这就足够了。实际上，这就是全部；是时候休息一下了！

你已经在 Eloquent 中学习了所有关于关系的内容，现在你可以迅速构建复杂的应用程序。你对 Eloquent 的基础知识了如指掌，所以我的建议是：花时间回顾一下所有内容，做一些测试，编写好的代码，并享受模型和关系。

准备好了，翻页，深入探索更高级的内容！
