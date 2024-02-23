# 第四章：构建个人博客

在本章中，我们将使用 Laravel 编写一个简单的个人博客。我们还将介绍 Laravel 内置的身份验证、分页机制和命名路由。我们将详细介绍一些快速开发方法，这些方法是 Laravel 自带的，比如创建路由 URL。本章将涵盖以下主题：

+   创建和迁移帖子数据库

+   创建一个帖子模型

+   创建和迁移作者数据库

+   创建一个仅限会员的区域

+   保存博客帖子

+   将博客帖子分配给用户

+   列出文章

+   对内容进行分页

# 创建和迁移帖子数据库

我们假设你已经在`app/config/database.php`文件中定义了数据库凭据。对于这个应用程序，我们需要一个数据库。你可以简单地创建并运行以下 SQL 命令，或者基本上你可以使用你的数据库管理界面，比如 phpMyAdmin：

```php
**CREATE DATABASE laravel_blog**

```

成功创建应用程序的数据库后，首先我们需要创建一个帖子表并将其安装在数据库中。要做到这一点，打开你的终端，导航到你的项目文件夹，并运行这个命令：

```php
**php artisan migrate:make create_posts_table --table=posts --create**

```

这个命令将在`app/database/migrations`下生成一个迁移文件，用于在我们的`laravel_blog`数据库中生成一个名为`posts`的新 MySQL 表。

为了定义我们的表列和规范，我们需要编辑这个文件。编辑迁移文件后，它应该看起来像这样：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePostsTable extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
    Schema::create('posts', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('title');
      $table->text('content');
      $table->integer('author_id');
      $table->timestamps();
    });
  }

  /**
   * Reverse the migrations.
   *
   * @return void
   */
  public function down()
  {
    Schema::drop('posts');
  }
}
```

保存文件后，我们需要使用一个简单的`artisan`命令来执行迁移：

```php
**php artisian migrate**

```

如果没有错误发生，请检查`laravel_blog`数据库中的`posts`表和列。

# 创建一个帖子模型

如你所知，对于 Laravel 上的任何与数据库操作相关的事情，使用模型是最佳实践。我们将受益于 Eloquent ORM。

将这段代码保存在一个名为`Posts.php`的文件中，放在`app/models/`下：

```php
<?php
class Post extends Eloquent {

protected $table = 'posts';

protected $fillable = array('title','content','author_id');

public $timestamps = true;

public function Author(){

      return $this->belongsTo('User','author_id');
}

}
```

我们已经使用受保护的`$table`变量设置了数据库表名。我们还使用了`$fillable`变量设置可编辑的列，并使用了`$timestamps`变量设置时间戳，就像我们在之前的章节中已经看到并使用过的那样。在模型中定义的变量足以使用 Laravel 的 Eloquent ORM。我们将在本章的*将博客帖子分配给用户*部分介绍公共的`Author()`函数。

我们的帖子模型已经准备好了。现在我们需要一个作者模型和数据库来将博客帖子分配给作者。让我们研究一下 Laravel 内置的身份验证机制。

# 创建和迁移作者数据库

与大多数 PHP 框架相反，Laravel 有一个基本的身份验证类。身份验证类在快速开发应用程序方面非常有帮助。首先，我们需要一个应用程序的秘钥。应用程序的秘钥对于我们应用程序的安全非常重要，因为所有数据都是使用这个秘钥进行哈希加盐的。`artisan`命令可以用一个单一的命令行为我们生成这个秘钥：

```php
**php artisian key:generate**

```

如果没有错误发生，你将看到一条消息，告诉你秘钥已经成功生成。在生成秘钥后，如果你在打开 Laravel 应用程序时遇到问题，只需清除浏览器缓存，然后重试。接下来，我们应该编辑身份验证类的配置文件。为了使用 Laravel 内置的身份验证类，我们需要编辑位于`app/config/auth.php`的配置文件。该文件包含了身份验证设施的几个选项。如果你需要更改表名等，你可以在这个文件下进行更改。默认情况下，Laravel 自带`User`模型。你可以看到位于`app/models/`下的`User.php`文件。在 Laravel 4 中，我们需要定义`Users`模型中哪些字段是可填充的。让我们编辑位于`app/models/`下的`User.php`并添加"fillable"数组：

```php
<?php

use Illuminate\Auth\UserInterface;
use Illuminate\Auth\Reminders\RemindableInterface;

class User extends Eloquent implements UserInterface, RemindableInterface {

  /**
   * The database table used by the model.
   *
   * @var string
   */
  protected $table = 'users';

  /**
   * The attributes excluded from the model's JSON form.
   *
   * @var array
   */
  protected $hidden = array('password');

  //Add to the "fillable" array
   protected $fillable = array('email', 'password', 'name');

  /**
   * Get the unique identifier for the user.
   *
   * @return mixed
   */
  public function getAuthIdentifier()
  {
    return $this->getKey();
  }

  /**
   * Get the password for the user.
   *
   * @return string
   */
  public function getAuthPassword()
  {
    return $this->password;
  }

  /**
   * Get the e-mail address where password reminders are sent.
   *
   * @return string
   */
  public function getReminderEmail()
  {
    return $this->email;
  }

}
```

基本上，我们需要为我们的作者有三列。这些是： 

+   `email`：这一列存储作者的电子邮件

+   `password`：这一列存储作者的密码

+   `name`：这一列存储作者的名字和姓氏

现在我们需要几个迁移文件来创建`users`表并向我们的数据库添加作者。要创建一个迁移文件，可以给出以下命令：

```php
**php artisan migrate:make create_users_table --table=users --create**

```

打开最近创建的迁移文件，位于`app/database/migrations/`。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('users', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('email');
      $table->string('password');
      $table->string('name');
      $table->timestamps();
    });
  }
```

编辑迁移文件后，运行`migrate`命令：

```php
**php artisian migrate**

```

如你所知，该命令创建了`users`表及其列。如果没有错误发生，请检查`laravel_blog`数据库中的`users`表和列。

现在我们需要创建一个新的迁移文件，向数据库中添加一些作者。我们可以通过运行以下命令来实现：

```php
**php artisan migrate:make add_some_users**

```

打开迁移文件并编辑`up()`函数如下：

```php
  public function up()
  {
    User::create(array(
            'email' => 'your@email.com',
            'password' => Hash::make('password'),
            'name' => 'John Doe'
        ));
  }
```

我们在`up()`函数中使用了一个新的类，名为`Hash`。Laravel 有一个基于安全**Bcrypt**的哈希制造/检查类。Bcrypt 是一种被接受的、安全的哈希方法，用于重要数据，如密码。

我们在本章开头使用 artisan 工具创建应用程序密钥的类用于加盐。因此，要应用迁移，我们需要使用以下 artisan 命令进行迁移：

```php
**php artisian migrate**

```

现在，检查`users`表是否有记录。如果你检查`password`列，你会看到记录存储如下：

```php
**$2y$08$ayylAhkVNCnkfj2rITbQr.L5pd2AIfpeccdnW6.BGbA.1VtJ6Sdqy**

```

安全地存储用户的密码和关键数据非常重要。不要忘记，如果你更改应用程序密钥，所有现有的哈希记录将无法使用，因为`Hash`类在验证和存储给定数据时使用应用程序密钥作为盐键。

# 创建一个仅会员可访问的区域

我们的博客系统是基于会员的。因此，我们需要一些区域只能会员访问，以便添加新的博客文章。我们有两种不同的方法来实现这一点。第一种是路由过滤器方法，我们将在接下来的章节中详细介绍。第二种是基于模板的授权检查。这种方法是更有效地理解`Auth`类与**Blade 模板系统**的使用方式。

通过`Auth`类，我们可以通过一行代码来检查访问者的授权状态：

```php
Auth::check();
```

基于`Auth`类的`check()`函数总是返回`true`或`false`。这意味着我们可以在我们的代码中轻松地在`if/else`语句中使用该函数。如你从之前的章节所知，使用 blade 模板系统，我们能够在模板文件中使用这种类型的 PHP 语句。

在创建模板文件之前，我们需要编写我们的路由。我们的应用程序需要四个路由。它们是：

+   创建一个登录路由来处理登录请求

+   创建一个处理新文章请求的新文章路由

+   一个用于显示新文章表单和登录表单的管理路由

+   一个用于列出文章的索引路由

命名路由是 Laravel 框架的另一个令人惊叹的特性，用于快速开发。命名路由允许在生成重定向或 URL 时更舒适地引用路由。你可以按以下方式为路由指定名称：

```php
Route::get('all/posts', array('as' => 'posts', function()
{
    //
}));
```

你也可以为控制器指定路由名称：

```php
Route::get('all/posts', array('as' => 'allposts', , 'uses' => 'PostController@showPosts'));
```

由于命名路由，我们可以轻松地为我们的应用程序创建 URL：

```php
$url = URL::route('allposts');
```

我们也可以使用命名路由进行重定向：

```php
$redirect = Redirect::route('allposts');
```

打开路由配置文件，位于`app/routes.php`，并添加以下代码：

```php
Route::get('/', array('as' => 'index', 'uses' => 'PostsController@getIndex'));
Route::get('/admin', array('as' => 'admin_area', 'uses' => 'PostsController@getAdmin'));
Route::post('/add', array('as' => 'add_new_post', 'uses' => 'PostsController@postAdd'));
Route::post('/login', array('as' => 'login', 'uses' => 'UsersController@postLogin'));
Route::get('/logout', array('as' => 'logout', 'uses' => 'UsersController@getLogout'));
```

现在我们需要编写应用程序的控制器端和模板的代码。首先，我们可以从我们的管理区域开始编码。让我们在`app/views/`下创建一个名为`addpost.blade.php`的文件。我们的管理模板应该如下所示：

```php
<html>
<head>
<title>Welcome to Your Blog</title>
<link rel="stylesheet" type="text/css" href="/assets/css/style.css">
<!--[if lt IE 9]><script src="//html5shim.googlecode.com/svn/trunk/html5.js"></script><![endif]-->
</head>
<body>
@if(Auth::check())
<section class="container">
<div class="content">
<h1>Welcome to Admin Area, {{Auth::user()->name}} ! - <b>{{link_to_route('logout','Logout')}}</b></h1>
<form name="add_post" method="POST" action="{{URL::route('add_new_post')}}">
<p><input type="text" name="title" placeholder="Post Title" value=""/></p>
<p><textarea name="content" placeholder="Post Content"></textarea></p>
<p><input type="submit" name="submit" /></p>
</div>
</section>
@else
<section class="container">
<div class="login">
<h1>Please Login</h1>
<form name="login" method="POST" action="{{URL::route('login')}}">
<p><input type="text" name="email" value="" placeholder="Email"></p>
<p><input type="password" name="password" value="" placeholder="Password"></p>
<p class="submit"><input type="submit" name="commit" value="Login"></p>
</form>
</div>
</section>
@endif
</body>
</html>
```

正如你在代码中所看到的，我们在模板中使用`if`/`else`语句来检查用户的登录凭据。我们从本节的开头就已经知道，我们使用`Auth::check()`函数来检查用户的登录状态。此外，我们还使用了一种新的方法来获取当前登录用户的名称：

```php
Auth::user()->name;
```

我们可以使用`user`方法获取关于当前用户的任何信息：

```php
Auth::user()->id; 
Auth::user()->email;
```

模板代码首先检查访问者的登录状态。如果访问者已登录，则模板显示一个新文章表单；否则显示一个登录表单。

现在我们需要编写博客应用程序的控制器端。让我们从我们的用户控制器开始。在 `app/controller/` 下创建一个名为 `UsersContoller.php` 的文件。控制器的最终代码应如下所示：

```php
<?php

class UsersController extends BaseController{

  public function postLogin()
  {
    Auth::attempt(array('email' => Input::get('email'),'password' => Input::get('password')));
  return Redirect::route('add_new_post');

  }

  public function getLogout()
  {
    Auth::logout();
    return Redirect::route('index');
  }
}
```

控制器有两个函数：第一个是 `postLogin()` 函数。该函数基本上检查用户登录的表单数据，然后将访问者重定向到 `add_new_post` 路由以显示新文章表单。第二个函数处理注销请求，并重定向到 `index` 路由。

# 保存博客文章

现在我们需要为我们的博客文章创建一个控制器。因此，在 `app/controller/` 下创建一个名为 `PostsContoller.php` 的文件。控制器的最终代码应如下所示：

```php
<?php
class PostsController extends BaseController{

  public function getIndex()
  {

  $posts = Post::with('Author')-> orderBy('id', 'DESC')->get();
  return View::make('index')->with('posts',$posts);

  }
  public function getAdmin()
  {
  return View::make('addpost');
  }
  public function postAdd()
  {
  Post::create(array(
              'title' => Input::get('title'),
              'content' => Input::get('content'),
              'author_id' => Auth::user()->id
   ));
  return Redirect::route('index');
  }
}
```

## 将博客文章分配给用户

`postAdd()` 函数处理数据库上的新博客文章创建请求。正如您所看到的，我们可以使用先前提到的方法获取作者的 ID：

```php
Auth::user()->id
```

使用这种方法，我们可以为当前用户分配一个博客文章。正如您将看到的，我们在查询中有一个新方法：

```php
Post::with('Author')->
```

如果您记得，我们在我们的 `Posts` 模型中定义了一个公共的 `Author()` 函数：

```php
public function Author(){

      return $this->belongsTo('User','author_id');
}
```

`belongsTo()` 方法是一个 `Eloquent` 函数，用于创建表之间的关系。基本上，该函数需要一个必需的变量和一个可选的变量。第一个变量（必需）定义了目标 `Model`。第二个可选变量用于定义当前模型表的源列。如果不定义可选变量，`Eloquent` 类会搜索 `targetModelName_id` 列。在 `posts` 表中，我们将作者的 ID 存储在 `author_id` 列中，而不是在名为 `user_id` 的列中。因此，我们需要在函数中定义第二个可选变量。使用这种方法，我们可以将博客文章及其所有作者的信息传递到模板文件中。您可以将该方法视为某种 SQL 连接方法。

当我们想在查询中使用这些关系函数时，我们可以轻松地调用它们如下所示：

```php
Books::with('Categories')->with('Author')->get();
```

使用较少的变量管理模板文件很容易。现在我们只需要一个变量来传递模板文件，其中包含所有必要的数据。因此，我们需要第二个模板文件来列出我们的博客文章。这个模板将在我们博客的前端工作。

# 列出文章

在本章的前几节中，我们已经学会了在 blade 模板文件中使用 PHP `if/else` 语句。Laravel 将数据作为数组传递到模板文件中。因此，我们需要使用 `foreach` 循环将数据解析到模板文件中。我们还可以在模板文件中使用 `foreach` 循环。因此，在 `app/views/` 下创建一个名为 `index.blade.php` 的文件。代码应如下所示：

```php
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>My Awesome Blog</title>
<link rel="stylesheet" href="/assets/blog/css/styles.css" type="text/css" media="screen" />
<link rel="stylesheet" type="text/css" href="/assets/blog/css/print.css" media="print" />
<!--[if IE]><script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script><![endif]-->
</head>
<body>
<div id="wrapper">
<header>
<h1><a href="/">My Awesome Blog</a></h1>
<p>Welcome to my awesome blog</p>
</header>
<section id="main">
<section id="content">
@foreach($posts as $post)
<article>
<h2>{{$post->title}}</h2>
<p>{{$post->content}}</p>
<p><small>Posted by <b>{{$post->Author->name}}</b> at <b>{{$post->created_at}}</b></small></p>
</article>

@endforeach          
</section>
</aside>
</section>
<footer>
<section id="footer-area">
<section id="footer-outer-block">
<aside class="footer-segment">
<h4>My Awesome Blog</h4>
</aside>
</section>
</section>
</footer>
</div>
</body>
</html>
```

让我们来看看代码。我们在模板文件中使用了 `foreach` 循环来解析所有博客文章数据。此外，我们还在 `foreach` 循环中看到了组合作者数据的使用。正如您可能记得的那样，我们在模型端使用 `belongsTo()` 方法获取作者信息。整个关系数据解析都在一个名为关系函数名称的数组中完成。例如，如果我们有一个名为 `Categories()` 的第二个关系函数，那么在控制器端查询将如下所示：

```php
$books = Books::with('Author')-> with('Categories')->orderBy('id', 'DESC')->get();
```

`foreach` 循环如下所示：

```php
@foreach($books as $book)

<article>
<h2>{{$book->title}}</h2>
<p>Author: <b>{{$book->Author->name}}</b></p>
<p>Category: <b>{{$book->Category->name}}</b></p>
</article>

@endforeach
```

# 对内容进行分页

`Eloquent` 的 `get()` 方法在控制器端的 `Eloquent` 查询中使用，从数据库中获取所有数据。通常，我们需要对内容进行分页，以便用户友好的前端或减少页面加载和优化。`Eloquent` 类有一个快速执行此操作的有用方法，称为 `paginate()`。该方法获取分页数据并在模板中生成分页链接，只需一行代码。打开 `app/controllers/PostsController.php` 文件，并将查询更改为如下所示：

```php
$posts = Post::with('Author')->orderBy('id', 'DESC')->paginate(5);
```

`paginate()` 方法使用给定的数字值对数据进行分页。因此，博客文章将每页分页为 `5` 篇博客文章。我们还需要更改我们的模板以显示分页链接。打开 `app/views/index.blade.php`，并在 `foreach` 循环之后添加以下代码：

```php
{{$posts->links()}}
```

模板中具有 ID 为 "main" 的部分应如下所示：

```php
<section id="main">
<section id="content">
@foreach($posts as $post)

<article>
<h2>{{$post->title}}</h2>
<p>{{$post->content}}</p>
<p><small>Posted by <b>{{$post->Author->name}}</b> at <b>{{$post->created_at}}</b></small></p>
</article>
@endforeach

</section>
{{$posts->links()}}
</section>
```

`links()` 函数将自动生成分页链接，如果有足够的数据进行分页。否则，该函数不显示任何内容。

# 摘要

在本章中，我们使用 Laravel 的内置函数和 Eloquent 数据库驱动程序创建了一个简单的博客。我们学习了如何对数据进行分页以及 Eloquent 的基本数据关系机制。同时，我们也介绍了 Laravel 的内置身份验证机制。在接下来的章节中，我们将学习如何处理更复杂的表格和关联数据。
