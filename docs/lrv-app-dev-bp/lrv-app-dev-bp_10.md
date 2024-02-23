# 第十章：构建电子商务网站

在本章中，我们将使用 Laravel 编写一个简单的书店示例。我们还将涵盖 Laravel 的内置身份验证、命名路由和数据库种子。我们还将详细介绍一些与 Laravel 一起提供的快速开发方法，例如创建路由 URL。此外，我们将使用一种称为*belongs to many*的新关系类型进行工作。我们还将涵盖中间表。我们的电子商务应用程序将是一个简单的书店。此应用程序将具有订单、管理和购物车功能。我们将涵盖以下主题：

+   构建授权系统

+   创建和迁移作者、书籍、购物车和订单表

+   创建模板文件

+   列出书籍

+   构建购物车

+   接受订单

+   列出订单

# 构建授权系统

我们假设您已经在`app/config`中的`database.php`文件中定义了数据库凭据。要创建我们的电子商务应用程序，我们需要一个数据库。您可以创建并简单运行以下 SQL 命令，或者基本上您可以使用数据库管理界面，如 phpMyAdmin：

```php
**CREATE DATABASE laravel_store**

```

# 创建和迁移成员数据库

与大多数 PHP 框架相反，Laravel 具有基本且可定制的身份验证机制。身份验证类对于快速开发应用程序非常有帮助。首先，我们需要为我们的应用程序生成一个秘钥。正如我们在前几章中提到的，应用程序的秘钥对于应用程序的安全非常重要，因为所有数据都是使用此秘钥进行哈希加盐的。艺术家可以用一条命令为我们生成这个秘钥：

```php
**php artisan key:generate**

```

如果没有错误发生，您将看到一条消息，告诉您密钥已成功生成。生成密钥后，如果您之前访问过项目的 URL，但在打开 Laravel 应用程序时遇到问题，只需清除浏览器缓存，然后重试。接下来，我们应该编辑身份验证类的配置文件。要使用 Laravel 内置的身份验证类，我们需要编辑位于`app/config/auth.php`的配置文件。该文件包含身份验证设施的几个选项。如果您需要更改表名等内容，可以在此文件下进行更改。默认情况下，Laravel 带有一个`User`模型。您可以在`app/models/User.php`中找到该文件。使用 Laravel 4，我们需要定义`User`模型中哪些字段是可填充的。让我们编辑位于`app/models/`的`User.php`并添加`fillable`数组：

```php
<?php

use Illuminate\Auth\UserInterface;
use Illuminate\Auth\Reminders\RemindableInterface;

class User extends Eloquent implements UserInterface, RemindableInterface {

  protected $table = 'users';

  /**
   * The attributes excluded from the model's JSON form.
   *
   * @var array
   */
  protected $hidden = array('password');

  //Add to the "fillable" array
   protected $fillable = array('email', 'password', 'name', 'admin');

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

基本上，我们的成员需要四列。这些是：

+   `email`：这是用于存储成员电子邮件的列

+   `password`：这是用于存储成员密码的列

+   `name`：这是用于存储成员名字和姓氏的列

+   `admin`：这是用于标记商店管理员的列

现在我们需要几个迁移文件来创建`users`表并向我们的数据库添加成员。要创建迁移文件，请给出以下命令：

```php
**php artisan migrate:make create_users_table --table=users --create**

```

打开最近创建的位于`app/database/migrations/`的迁移文件。我们需要编辑`up()`函数，如下面的代码片段所示：

```php
  public function up()
  {
    Schema::create('users', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('email');
      $table->string('password');
      $table->string('name');
      $table->integer('admin');
      $table->timestamps();
    });
  }
```

编辑`迁移`文件后，运行`migrate`命令：

```php
**php artisan migrate**

```

现在我们需要创建一个数据库种子文件，向数据库添加一些用户。数据库种子是另一种强烈推荐的向应用程序数据库添加数据的方法。数据库种子文件位于`app/database/seeds`。让我们在`UsersTableSeeder.php`目录下创建我们的第一个种子文件。

### 注意

我们可以使用任何名称创建种子文件和种子类。但强烈建议种子文件和类名称应遵循驼峰命名法，例如`TableSeeder`。遵循全球编程标准将提高您代码的质量。

`UsersTableSeeder.php`的内容应如下所示：

```php
<?php
Class UsersTableSeeder extends Seeder {

    public function run()
    {
  DB::table('users')->delete();

User::create(array(
            'email' => 'member@email.com',
            'password' => Hash::make('password'),
            'name' => 'John Doe',
            'admin'=>0
        ));

  User::create(array(
            'email' => 'admin@store.com',
            'password' => Hash::make('adminpassword'),
            'name' => 'Jeniffer Taylor',
            'admin'=>1
        ));  

    }

}
```

要应用种子数据，首先我们需要调用种子类。让我们打开位于`app/database/seeds`的文件`DatabaseSeeder.php`并编辑文件如下代码片段所示：

```php
<?php
class DatabaseSeeder extends Seeder {

  /**
   * Run the database seeds.
   *
   * @return void
   */
  public function run()
  {
    Eloquent::unguard();

 **$this->call('UsersTableSeeder');**
 **$this->command->info('Users table seeded!');**
  }

}
```

安全地存储用户的密码和关键数据非常重要。不要忘记，如果你改变了应用程序密钥，所有现有的哈希记录将无法使用，因为`Hash`类在验证和存储给定数据时使用应用程序密钥作为盐值密钥。

# 创建和迁移作者数据库

我们需要一个`Author`模型来存储书籍作者。它将是一个非常简单的结构。让我们在`app/models`下创建`Author.php`文件并添加以下代码：

```php
<?php
Class Author extends Eloquent {

protected $table = 'authors';

protected $fillable = array('name','surname');

}
```

现在我们需要几个迁移文件来创建`authors`表并向我们的数据库中添加一些作者。要创建一个迁移文件，输入以下命令：

```php
php artisan migrate:make create_authors_table --table=authors --create
```

打开最近创建并位于`app/database/migrations/`目录下的迁移文件。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('authors', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('name');
      $table->string('surname');
      $table->timestamps();
    });
  }
```

编辑迁移文件后，运行`migrate`命令：

```php
php artisan migrate
```

如你所知，该命令会创建`authors`表及其列。如果没有错误发生，请检查`laravel_store`数据库中的`authors`表和列。

## 向数据库添加作者

现在我们需要创建一个数据库种子文件来向数据库中添加一些作者。让我们在`app/database/seeds/`下创建我们的第一个种子文件`AuthorsTableSeeder.php`。

`AuthorsTableSeeder.php`中的内容应该如下所示：

```php
<?php
Class AuthorsTableSeeder extends Seeder {

    public function run()
    {
DB::table('authors')->delete();

        Author::create(array(
            'name' => 'Lauren',
            'surname'=>'Oliver'
        ));

        Author::create(array(
            'name' => 'Stephenie',
            'surname'=>'Meyer'
        ));

        Author::create(array(
            'name' => 'Dan',
            'surname'=>'Brown'
        ));

    }

}
```

要应用种子数据，首先我们需要调用`Seeder`类。让我们打开位于`app/database/seeds/`的文件`DatabaseSeeder.php`并编辑文件如下代码片段所示：

```php
<?php
class DatabaseSeeder extends Seeder {

  /**
   * Run the database seeds.
   *
   * @return void
   */
  public function run()
  {
    Eloquent::unguard();
    $this->call('UsersTableSeeder');
    $this->command->info('Users table seeded!');
 **$this->call('AuthorsTableSeeder');**
 **$this->command->info('Authors table seeded!');**
  }

}
```

我们需要使用以下`artisan`命令向数据库中添加种子数据：

```php
**php artisan db:seed**

```

### 注意

当你想要回滚并重新运行所有迁移时，你可以使用以下命令：

```php
**php artisan migrate:refresh --seed**

```

# 创建和迁移图书数据库

我们需要一个`Book`模型来存储作者的书籍。让我们在`app/models/`下创建`Book.php`文件并添加以下代码：

```php
<?php
Class Book extends Eloquent {

protected $table = 'books';

protected $fillable = array('title','isbn','cover','price','author_id');

public function Author(){

return $this->belongsTo('Author');

}

}
```

让我们解释`author_id`列和`Author()`函数的作用。正如你从之前的章节中所了解的，`Eloquent`有几个用于不同类型数据库关系的函数。`author_id`将存储作者的 ID。`Author()`函数用于从`authors`表中获取作者的名字和姓氏。

## 向数据库添加书籍

现在我们需要创建一个数据库种子文件来向数据库中添加一些书籍。让我们在`app/database/seeds/`下创建第一个种子文件`BooksTableSeeder.php`。

`BooksTableSeeder.php`中的内容应该如下所示：

```php
<?php
Class BooksTableSeeder extends Seeder {

  public function run()
  {
  DB::table('books')->delete();

  Book::create(array(
    'title'=>'Requiem',
    'isbn'=>'9780062014535',
    'price'=>'13.40',
    'cover'=>'requiem.jpg',
    'author_id'=>1
   ));
  Book::create(array(
    'title'=>'Twilight',
    'isbn'=>'9780316015844',
    'price'=>'15.40',
    'cover'=>'twilight.jpg',
    'author_id'=>2
  ));
  Book::create(array(
    'title'=>'Deception Point',
    'isbn'=>'9780671027384',
    'price'=>'16.40',
    'cover'=>'deception.jpg',
    'author_id'=>3
  ));

  }

}
```

要应用种子数据，首先我们需要调用种子类。让我们打开位于`app/database/seeds`的文件`DatabaseSeeder.php`并编辑文件如下代码片段所示：

```php
<?php
class DatabaseSeeder extends Seeder {

  /**
   * Run the database seeds.
   *
   * @return void
   */
  public function run()
  {
    Eloquent::unguard();
    $this->call('UsersTableSeeder');
    $this->command->info('Users table seeded!');
    $this->call('AuthorsTableSeeder');
    $this->command->info('Authors table seeded!');
    **$this->call('BooksTableSeeder');**
 **$this->command->info('Books table seeded!');**
  }

}
```

现在，我们需要使用以下`artisan`命令向数据库中添加种子数据：

```php
**php artisan db:seed**

```

# 创建和迁移购物车数据库

正如你所知，所有电子商务应用程序都应该有一个购物车。在这个应用程序中，我们也会有一个购物车。我们将设计一个基于会员的购物车，这意味着我们可以存储和显示每个访问者的购物车和购物车商品。因此，我们需要一个`Cart`模型来存储购物车商品。它将是一个非常简单的结构。让我们在`app/models`下创建`Cart.php`文件并添加以下代码：

```php
<?php
Class Cart extends Eloquent {

protected $table = 'carts';

protected $fillable = array('member_id','book_id','amount','total');

public function Books(){

return $this->belongsTo('Book','book_id');

}

}
```

现在我们需要一个迁移文件来创建`carts`表。要创建一个迁移文件，输入以下命令：

```php
**php artisan migrate:make create_carts_table --table=carts --create**

```

打开最近创建并位于`app/database/migrations/`目录下的迁移文件。我们需要编辑`up()`函数如下代码片段所示：

```php
  public function up()
  {
    Schema::create('carts', function(Blueprint $table)
    {
      $table->increments('id');
      $table->integer('member_id');
      $table->integer('book_id');
      $table->integer('amount');
      $table->decimal('total', 10, 2);
      $table->timestamps();
    });
  }
```

要应用迁移，我们需要使用以下`artisan`命令进行迁移：

```php
**php artisan migrate**

```

# 创建和迁移订单数据库

为了存储会员的订单，我们需要两个表。其中之一是`orders`表，它将存储运输细节、会员 ID 和订单的总价值。另一个是`order_books`表。这个表将存储订单的书籍，并且将是我们的中间表。在这个模型中，我们将使用`belongsToMany()`关系。这是因为一个订单可以有多本书。

因此，首先我们需要一个`Order`模型来存储书籍订单。让我们在`app/models`下创建`Order.php`文件，并添加以下代码：

```php
<?php
Class Order extends Eloquent {

protected $table = 'orders';

protected $fillable = array('member_id','address','total');

public function orderItems()
    {
        return $this->belongsToMany('Book') ->withPivot('amount','total');
    }

}
```

正如您在代码中所看到的，我们使用了一个名为`withPivot()`的新选项，它与`belongsToMany()`函数一起使用。通过`withPivot()`函数，我们可以从我们的中间表中获取额外的字段。通常情况下，如果没有这个函数，关系查询只能通过相关行的`id`对象从中间表中访问。这对我们的应用程序是必要的，因为价格变动。因此，可能在任何价格变动之前完成的先前订单不受影响。

现在我们需要一个迁移文件来创建`carts`表。要创建一个`migration`文件，可以使用以下命令：

```php
**php artisan migrate:make create_orders_table --table=orders --create**

```

打开最近创建的位于`app/database/migrations`目录下的迁移文件。我们需要编辑`up()`函数如下所示：

```php
  public function up()
  {
    Schema::create('orders', function(Blueprint $table)
    {
      $table->increments('id');
      $table->integer('member_id');
      $table->text('address');
      $table->decimal('total', 10, 2);
      $table->timestamps();
    });
  }
```

要应用迁移，我们需要使用以下`artisan`命令进行迁移：

```php
**php artisan migrate**

```

让我们创建我们的`pivot`表。我们需要一个迁移文件来创建`order_books`表。要创建一个迁移文件，可以使用以下命令：

```php
**php artisan migrate:make create_order_books_table --table=order_books --create**

```

打开最近创建的位于`app/database/migrations`目录下的迁移文件。我们需要编辑`up()`函数如下所示：

```php
  public function up()
  {
    Schema::create('order_books', function(Blueprint $table)
    {
      $table->increments('id');
      $table->integer('order_id');
      $table->integer('book_id');
      $table->integer('amount');
$table->decimal('price', 10, 2);
      $table->decimal('total', 10, 2);
    });  
}
```

要应用迁移，我们需要使用以下`artisan`命令进行迁移：

```php
**php artisan migrate**

```

我们的数据库设计和模型已经完成。现在我们需要编写控制器和我们应用程序的前端页面。

# 列出书籍

首先，我们需要列出我们的产品。为此，我们需要创建一个名为`BookController`的控制器。让我们在`app/controllers/`下创建一个文件，并将其保存为`BookController.php`。控制器代码应该如下所示：

```php
<?php
class BookController extends BaseController{

  public function getIndex()
  {

    $books = Book::all();

    return View::make('book_list')->with('books',$books);

  }
}
```

该代码简单地从我们的`books`表中获取所有的书籍，并将数据传递给`book_list.blade.php`模板，使用`$books`变量。因此，我们需要在`app/controllers/`下创建一个模板文件，命名为`book_list.blade.php`。在这之前，我们需要一个用于我们模板的布局页面。使用布局文件可以很好地管理 html 代码。因此，首先我们需要在`app/controllers/`下创建一个模板文件，命名为`main_layout.blade.php`。代码应该如下所示：

```php
<!DOCTYPE html>
<html>
<head>
  <title>Awesome Book Store</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- Bootstrap -->
  <link href="//netdna.bootstrapcdn.com/twitter-bootstrap/2.3.1/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="navbar navbar-inverse nav">
    <div class="navbar-inner">
        <div class="container">
            <a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </a>
            <a class="brand" href="/">Awesome Book Store</a>
            <div class="nav-collapse collapse">
              <ul class="nav">
                  <li class="divider-vertical"></li>
                  <li><a href="/"><i class="icon-home icon-white"></i> Book List</a></li>
              </ul>
              <div class="pull-right">
                <ul class="nav pull-right">
                @if(!Auth::check())
                <ul class="nav pull-right">
                  <li class="divider-vertical"></li>
                  <li class="dropdown">
                    <a class="dropdown-toggle" href="#" data-toggle="dropdown">Sign In <strong class="caret"></strong></a>
                    <div class="dropdown-menu" style="padding: 15px; padding-bottom: 0px;">
                      <p>Please Login</a>
                        <form action="/user/login" method="post" accept-charset="UTF-8">
                          <input id="email" style="margin-bottom: 15px;" type="text" name="email" size="30" placeholder="email" />
                          <input id="password" style="margin-bottom: 15px;" type="password" name="password" size="30" />
                          <input class="btn btn-info" style="clear: left; width: 100%; height: 32px; font-size: 13px;" type="submit" name="commit" value="Sign In" />
                        </form>
                      </div>
                    </li>
                  </ul>
                @else
                <li><a href="/cart"><i class="icon-shopping-cart icon-white"></i> Your Cart</a></li>
                  <li class="dropdown"><a href="#" class="dropdown-toggle" data-toggle="dropdown">Welcome, {{Auth::user()->name}} <b class="caret"></b></a>
                        <ul class="dropdown-menu">
                            <li><a href="/user/orders"><i class="icon-envelope"></i> My Orders</a></li>
                            <li class="divider"></li>
                            <li><a href="/user/logout"><i class="icon-off"></i> Logout</a></li>
                        </ul>
                    </li>
                @endif
                </ul>
              </div>
            </div>
        </div>
    </div>
</div>

@yield('content')

  <script src="http://code.jquery.com/jquery.js"></script>
  <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0/js/bootstrap.min.js"></script>
  <script type="text/javascript">
  $(function() {
    $('.dropdown-toggle').dropdown();

    $('.dropdown input, .dropdown label').click(function(e) {
      e.stopPropagation();
    });
  });

  @if(isset($error))
    alert("{{$error}}");
  @endif

  @if(Session::has('error'))
    alert("{{Session::get('error')}}");
  @endif

  @if(Session::has('message'))
    alert("{{Session::get('message')}}");
  @endif

  </script>
</body>
</html>
```

模板文件包含一个菜单、一个登录表单和一些用于下拉式登录表单的 JavaScript 代码。我们将使用该文件作为我们应用程序的布局模板。我们需要在`app/controllers`目录下的`UserController.php`文件中编写我们的登录和注销函数。登录函数应该如下所示的代码：

```php
<?php
class UserController extends BaseController {

  public function postLogin()
  {
    $email=Input::get('email');
    $password=Input::get('password');

    if (Auth::attempt(array('email' => $email, 'password' => $password)))
    {
        return Redirect::route('index');

    }else{

      return Redirect::route('index')
        ->with('error','Please check your password & email');
    }
  }

  public function getLogout()
  {
    Auth::logout();
    return Redirect::route('index');
  }
}
```

如下所示，我们需要在我们的路由文件`routes.php`中添加路由：

```php
Route::get('/', array('as'=>'index','uses'=>'BookController@getIndex'));
Route::post('/user/login', array('uses'=>'UserController@postLogin'));
Route::get('/user/logout', array('uses'=>'UserController@getLogout'));
```

## 创建一个模板文件来列出书籍

现在我们需要一个模板文件来列出书籍。如前所述，我们需要在`app/views/`下创建一个模板文件，并将其保存为`book_list.blade.php`。该文件应该如下所示：

```php
@extends('main_layout')

@section('content')

<div class="container">
  <div class="span12">
    <div class="row">
      <ul class="thumbnails">
        @foreach($books as $book)
        <li class="span4">
          <div class="thumbnail">
            <img src="/images/{{$book->cover}}" alt="ALT NAME">
            <div class="caption">
              <h3>{{$book->title}}</h3>
              <p>Author : <b>{{$book->author->name}} {{$book->author->surname}}</b></p>
              <p>Price : <b>{{$book->price}}</b></p>
              <form action="/cart/add" name="add_to_cart" method="post" accept-charset="UTF-8">
                <input type="hidden" name="book" value="{{$book->id}}" />
                <select name="amount" style="width: 100%;">
                  <option value="1">1</option>
                  <option value="2">2</option>
                  <option value="3">3</option>
                  <option value="4">4</option>
                  <option value="5">5</option>
                </select>
              <p align="center"><button class="btn btn-info btn-block">Add to Cart</button></p>
            </form>
            </div>
          </div>
        </li>
        @endforeach
      </ul>
    </div>
  </div>
</div>

@stop
```

模板文件中有一个表单，用于将书籍添加到购物车中。现在我们需要在`app/controllers/`目录下的`CartController.php`文件中编写我们的函数。`CartController.php`的内容应该如下所示：

```php
<?php
class CartController extends BaseController {

  public function postAddToCart()
  {
    $rules=array(

      'amount'=>'required|numeric',
      'book'=>'required|numeric|exists:books,id'
    );

    $validator = Validator::make(Input::all(), $rules);

      if ($validator->fails())
      {
          return Redirect::route('index')->with('error','The book could not added to your cart!');
      }

      $member_id = Auth::user()->id;
      $book_id = Input::get('book');
      $amount = Input::get('amount');

      $book = Book::find($book_id);
      $total = $amount*$book->price;

       $count = Cart::where('book_id','=',$book_id)->where('member_id','=',$member_id)->count();

       if($count){

         return Redirect::route('index')->with('error','The book already in your cart.');
       }

      Cart::create(
        array(
        'member_id'=>$member_id,
        'book_id'=>$book_id,
        'amount'=>$amount,
        'total'=>$total
        ));

      return Redirect::route('cart');
  }

  public function getIndex(){

    $member_id = Auth::user()->id;

    $cart_books=Cart::with('Books')->where('member_id','=',$member_id)->get();

    $cart_total=Cart::with('Books')->where('member_id','=',$member_id)->sum('total');

    if(!$cart_books){

      return Redirect::route('index')->with('error','Your cart is empty');
    }

    return View::make('cart')
          ->with('cart_books', $cart_books)
          ->with('cart_total',$cart_total);
  }

  public function getDelete($id){

    $cart = Cart::find($id)->delete();

    return Redirect::route('cart');
  }

}
```

我们的控制器有三个函数。其中之一是`postAddToCart()`：

```php
public function postAddToCart()
  {
    $rules=array(

      'amount'=>'required|numeric',
      'book'=>'required|numeric|exists:books,id'
    );

    $validator = Validator::make(Input::all(), $rules);

      if ($validator->fails())
      {
          return Redirect::route('index')->with('error','The book could not added to your cart!');
      }

      $member_id = Auth::user()->id;
      $book_id = Input::get('book');
      $amount = Input::get('amount');

      $book = Book::find($book_id);
      $total = $amount*$book->price;

       $count = Cart::where('book_id','=',$book_id)->where('member_id','=',$member_id)->count();

       if($count){

         return Redirect::route('index')->with('error','The book already in your cart.');
       }

      Cart::create(
        array(
        'member_id'=>$member_id,
        'book_id'=>$book_id,
        'amount'=>$amount,
        'total'=>$total
        ));

      return Redirect::route('cart');
  }
```

该函数基本上首先验证了提交的数据。经过验证的数据检查`carts`表中是否有重复记录。如果会员的购物车中没有相同的书籍，函数将在`carts`表中创建一个新记录。`CartController`的第二个函数是`getIndex()`：

```php
  public function getIndex(){

    $member_id = Auth::user()->id;

    $cart_books=Cart::with('Books')->where('member_id','=',$member_id)->get();

    $cart_total=Cart::with('Books')->where('member_id','=',$member_id)->sum('total');

    if(!$cart_books){

      return Redirect::route('index')->with('error','Your cart is empty');
    }

    return View::make('cart')
          ->with('cart_books', $cart_books)
          ->with('cart_total',$cart_total);
  }
```

该功能获取整个购物车商品、书籍信息和购物车总额，并将数据传递给模板文件。该类的最后一个函数是`getDelete()`：

```php
  public function getDelete($id){

    $cart = Cart::find($id)->delete();

    return Redirect::route('cart');
  }
```

该功能基本上是从`carts`表中查找给定 ID 并删除记录。我们使用该功能从购物车中删除商品。现在我们需要创建一个模板文件。该文件将显示会员的所有购物车信息，并包含订单表单。将文件保存在`app/views/`下，命名为`cart.blade.php`。`cart.blade.php`的内容应该如下所示：

```php
@extends('main_layout')

@section('content')

<div class="container" style="width:60%">
  <h1>Your Cart</h1>
  <table class="table">
    <tbody>
      <tr>
        <td>
          <b>Title</b>
        </td>
        <td>
          <b>Amount</b>
        </td>
        <td>
          <b>Price</b>
        </td>
        <td>
          <b>Total</b>
        </td>
        <td>
          <b>Delete</b>
        </td>
      </tr>
      @foreach($cart_books as $cart_item)
        <tr>
          <td>{{$cart_item->Books->title}}</td>
          <td>
           {{$cart_item->amount}}
          </td>
          <td>
            {{$cart_item->Books->price}}
          </td>
          <td>
           {{$cart_item->total}}
          </td>
          <td>
            <a href="{{URL::route('delete_book_from_cart',array($cart_item->id))}}">Delete</a>
          </td>
        </tr>
      @endforeach
      <tr>
        <td>
        </td>
        <td>
        </td>
        <td>
          <b>Total</b>
        </td>
        <td>
          <b>{{$cart_total}}</b>
        </td>
        <td>
        </td>        
      </tr>
    </tbody>
  </table>
  <h1>Shipping</h1>
  <form action="/order" method="post" accept-charset="UTF-8">
    <label>Address</label>
    <textarea class="span4" name="address" rows="5"></textarea>
    <button class="btn btn-block btn-primary btn-large">Place order</button>
  </form>
</div>
@stop
```

现在我们需要编写我们的路由。控制器的功能应该只对会员可访问。因此，我们可以轻松地使用 Laravel 内置的`auth.basic`过滤器：

```php
Route::get('/cart', array('before'=>'auth.basic','as'=>'cart','uses'=>'CartController@getIndex'));
Route::post('/cart/add', array('before'=>'auth.basic','uses'=>'CartController@postAddToCart'));
Route::get('/cart/delete/{id}', array('before'=>'auth.basic','as'=>'delete_book_from_cart','uses'=>'CartController@getDelete'));
```

# 接受订单

正如您可能记得的，我们已经在位于`app/views/`的`cart.blade.php`模板文件中创建了一个订单表单。现在我们需要处理订单。让我们在`app/controllers/`下编写`OrderController.php`文件：

```php
<?php
class OrderController extends BaseController {

  public function postOrder()
  {
    $rules=array(

      'address'=>'required'
    );

  $validator = Validator::make(Input::all(), $rules);

      if ($validator->fails())
      {
          return Redirect::route('cart')->with('error','Address field is required!');
      }

      $member_id = Auth::user()->id;
      $address = Input::get('address');

       $cart_books = Cart::with('Books')->where('member_id','=',$member_id)->get();

       $cart_total=Cart::with('Books')->where('member_id','=',$member_id)->sum('total');

       if(!$cart_books){

         return Redirect::route('index')->with('error','Your cart is empty.');
       }

      $order = Order::create(
        array(
        'member_id'=>$member_id,
        'address'=>$address,
        'total'=>$cart_total
        ));

      foreach ($cart_books as $order_books) {

        $order->orderItems()->attach($order_books->book_id, array(
          'amount'=>$order_books->amount,
          'price'=>$order_books->Books->price,
          'total'=>$order_books->Books->price*$order_books->amount
          ));

      }

      Cart::where('member_id','=',$member_id)->delete();

      return Redirect::route('index')->with('message','Your order processed successfully.');
  }

  public function getIndex(){

    $member_id = Auth::user()->id;

    if(Auth::user()->admin){

      $orders=Order::all();

    }else{

      $orders=Order::with('orderItems')->where('member_id','=',$member_id)->get();
    }

    if(!$orders){

      return Redirect::route('index')->with('error','There is no order.');
    }

    return View::make('order')
          ->with('orders', $orders);
  }
}
```

控制器有两个功能。其中之一是`postOrder()`：

```php
public function postOrder()
  {
    $rules=array(

      'address'=>'required'
    );

  $validator = Validator::make(Input::all(), $rules);

      if ($validator->fails())
      {
          return Redirect::route('cart')->with('error','Address field is required!');
      }

      $member_id = Auth::user()->id;
      $address = Input::get('address');

       $cart_books = Cart::with('Books')->where('member_id','=',$member_id)->get();

       $cart_total=Cart::with('Books')->where('member_id','=',$member_id)->sum('total');

       if(!$cart_books){

         return Redirect::route('index')->with('error','Your cart is empty.');
       }

      $order = Order::create(
        array(
        'member_id'=>$member_id,
        'address'=>$address,
        'total'=>$cart_total
        ));

      foreach ($cart_books as $order_books) {

        $order->orderItems()->attach($order_books->book_id, array(
          'amount'=>$order_books->amount,
          'price'=>$order_books->Books->price,
          'total'=>$order_books->Books->price*$order_books->amount
          ));

      }

      Cart::where('member_id','=',$member_id)->delete();

      return Redirect::route('index')->with('message','Your order processed successfully.');
  }
```

该功能首先验证发布的数据。验证成功后，该功能在`orders`表上创建一个新订单。`order`表存储会员 ID、送货地址和订单总额。然后，该功能将所有购物车商品附加到与它们的数量、价格和总额相关的中间表中。通过这种方式，订单商品不会受到任何价格变化的影响。然后，该功能从会员的购物车中删除所有商品。控制器的第二个功能是`getIndex()`：

```php
public function getIndex(){

    $member_id = Auth::user()->id;

    if(Auth::user()->admin){

      $orders=Order::all();

    }else{

      $orders=Order::with('orderItems')->where('member_id','=',$member_id)->get();
    }

    if(!$orders){

      return Redirect::route('index')->with('error','There is no order.');
    }

    return View::make('order')
          ->with('orders', $orders);
  }
```

该功能通过查看当前用户的权限来查询数据库。如果当前用户具有管理员权限，则该功能获取所有订单。如果当前用户没有管理员权限，则该功能只获取用户的订单。因此，现在我们需要编写我们的路由。将以下路由代码添加到`app/routes.php`中：

```php
Route::post('/order', array('before'=>'auth.basic','uses'=>'OrderController@postOrder'));
Route::get('/user/orders', array('before'=>'auth.basic','uses'=>'OrderController@getIndex'));
```

我们的电子商务应用程序几乎完成了。现在我们需要添加一个模板文件。将文件保存在`app/views/`下，命名为`cart.blade.php`。`cart.blade.php`的内容应该如下所示：

```php
@extends('main_layout')
@section('content')
<div class="container" style="width:60%">
<h3>Your Orders</h3>
<div class="menu">
  <div class="accordion">
@foreach($orders as $order)
 <div class="accordion-group">
      <div class="accordion-heading country">
        @if(Auth::user()->admin)
        <a class="accordion-toggle" data-toggle="collapse" href="#order{{$order->id}}">Order #{{$order->id}} - {{$order->User->name}} - {{$order->created_at}}</a>
        @else
        <a class="accordion-toggle" data-toggle="collapse" href="#order{{$order->id}}">Order #{{$order->id}} - {{$order->created_at}}</a>
        @endif
      </div>
      <div id="order{{$order->id}}" class="accordion-body collapse">
        <div class="accordion-inner">
          <table class="table table-striped table-condensed">
            <thead>
              <tr>
              <th>
              Title
              </th>
              <th>
              Amount
              </th>
              <th>
              Price
              </th>
              <th>
              Total
              </th>
              </tr>
            </thead>   
            <tbody>
            @foreach($order->orderItems as $orderitem)
              <tr>
                <td>{{$orderitem->title}}</td>
                <td>{{$orderitem->pivot->amount}}</td>
                <td>{{$orderitem->pivot->price}}</td>
                <td>{{$orderitem->pivot->total}}</td>
              </tr>
            @endforeach
              <tr>
                <td></td>
                <td></td>
                <td><b>Total</b></td>
                <td><b>{{$order->total}}</b></td>
              </tr>
              <tr>
                <td><b>Shipping Address</b></td>
                <td>{{$order->address}}</td>
                <td></td>
                <td></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
@endforeach
</div>
</div>
@stop
```

模板文件包含有关订单的所有信息。该模板是如何使用中间表列的一个非常简单的示例。中间数据以数组形式呈现。因此，我们使用`foreach`循环来使用数据。您可以存储任何您不希望受数据库中任何更改影响的数据，例如价格更改。

# 总结

在本章中，我们构建了一个简单的电子商务应用程序。正如您所看到的，由于 Laravel 的模板系统和内置授权系统，您可以轻松地创建庞大的应用程序。您可以通过第三方包来改进应用程序。自 Laravel 版本 4 以来，主要的包管理器一直是 Composer。在[`packagist.org`](http://packagist.org)上有一个庞大的库，提供了用于图像处理、社交媒体 API 等的包。包的数量每天都在增加，它们默认情况下都与 Laravel 兼容。我们建议在编写任何代码之前，您先查看 Packagist 网站。在您阅读这些句子的同时，仍有许多贡献者在分享他们的代码。审查其他程序员的代码会为旧问题提供新的答案。不要忘记与他人分享您的知识，以获得更好的编程体验。

在整本书中，我们试图解释如何使用 Laravel PHP 框架构建不同类型的应用程序。因此，我们涵盖了 RESTful 控制器、路由、路由过滤器、认证类、Blade 模板引擎、数据库迁移、数据库种子、字符串和文件处理类。此外，我们还提供了一些关于如何快速开发 Laravel 的技巧。我们希望这本书能成为学习 Laravel 框架的良好资源。

本书的合著者*Halil İbrahim Yılmaz*开发了一个名为 HERKOBI 的基于 Laravel 的开源多语言 CMS。您可以使用本书章节的源代码和 CMS 的源代码。您可以在[`herkobi.org`](http://herkobi.org)和[`herkobi.com`](http://herkobi.com)上访问 CRM 和代码。

Laravel 有一个很好的社区，非常乐于助人和友好。您可以在 Laravel 论坛上提出任何问题。国际 Laravel 社区可以在[`laravel.com`](http://laravel.com)上访问。Laravel 还有一些国家的 Laravel 社区，比如土耳其的 Laravel 社区，位于[`laravel.gen.tr`](http://laravel.gen.tr)。

当您在书中或 Laravel PHP 框架中需要帮助时，可以给作者发送电子邮件。

感谢您对本书感兴趣并购买。
