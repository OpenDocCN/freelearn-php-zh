# 第九章：有效使用安全和会话

在本章中，我们将涵盖：

+   加密和解密数据

+   哈希密码和其他数据

+   在表单中使用 CSRF 令牌和过滤器

+   在表单中使用高级验证

+   构建购物车

+   使用 Redis 保存会话

+   使用基本会话和 cookies

+   创建安全的 API 服务器

# 介绍

安全是我们在构建 Web 应用程序时需要考虑的最重要的事情之一，特别是如果我们处理敏感的用户信息。Laravel 为我们提供了许多方法来保护我们的应用程序安全。

在本章中，我们将看看掩盖敏感数据的各种方法，如何保护我们的表单免受跨站点攻击，以及如何保护 API。我们还将看到如何使用会话来构建购物车，并使用 Redis 存储会话数据。

# 加密和解密数据

在编写处理敏感数据的应用程序时，我们经常希望加密我们存储在数据库中的任何数据。Laravel 为我们提供了解决这个问题的解决方案。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装，以及一个正确设置和配置的 MySQL 数据库。

## 如何做…

这是我们将使用以下步骤完成该配方的方法：

1.  在`app/config`目录中，打开`app.php`文件，并确保`key`为空

```php
  'key' => '',
```

1.  在命令行中，转到应用程序的根目录，并使用以下命令生成一个新的密钥：

```php
  php artisan key:generate
```

1.  使用以下命令在数据库中创建一个表来保存我们的敏感信息：

```php
CREATE TABLE accounts(
  id int(11) unsigned NOT NULL AUTO_INCREMENT,
    business varchar(255) DEFAULT NULL,
    total_revenue varchar(255) DEFAULT NULL,
    projected_revenue varchar(255) DEFAULT NULL,
    PRIMARY KEY (id)) 
    ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

1.  在我们的`app/models`目录中，通过输入以下代码创建一个名为`Account.php`的文件：

```php
<?php

class Account extends Eloquent {
  protected $table = 'accounts';
  public $timestamps = false;
  public function setBusinessAttribute($business) {$this->attributes['business'] = Crypt::encrypt($business);
}

public function setTotalrevenueAttribute($total_revenue)
  {$this->attributes['total_revenue'] = Crypt::encrypt($total_revenue);
}

  public functionsetProjectedrevenueAttribute($projected_revenue)
{
  $this->attributes['projected_revenue'] = Crypt::encrypt($projected_revenue);
}

public function getBusinessAttribute()
{
  return Crypt::decrypt($this->attributes['business'])
}

public function getTotalrevenueAttribute()
{
  return number_format(Crypt::decrypt($this>attributes['total_revenue'])) ;
}

public function getProjectedrevenueAttribute()
{
  return number_format(Crypt::decrypt($this>attributes['projected_revenue']));
}
}
```

1.  在我们的`routes.php`文件中，通过添加以下代码创建查看和提交信息的路由：

```php
Route::get('accounts', function()
{
  $accounts = Account::all();
  return View::make('accounts')->with('accounts', $accounts);
});

Route::post('accounts', function()
{
  $account = new Account();
  $account->business = Input::get('business');
  $account->total_revenue = Input::get('total_revenue');
  $account->projected_revenue = Input::get('projected_revenue');
  $account->save();
  return Redirect::to('accounts');
});
```

1.  在我们的`views`目录中，创建一个名为`accounts.php`的文件

```php
  <form action="accounts" method="post">
  <label for="business">Business:</label><br>
  <input name="business"><br><br>
  <label for="total_revenue">Total Revenue ($):</label><br>
  <input name="total_revenue"><br><br>
  <label for="projected_revenue">Projected Revenue($):</label><br>
  <input name="projected_revenue"><br><br>
  <input type="submit">
  </form>
  <hr>
  <?php if ($accounts): ?>
  <table border="1">
  <thead>
  <tr>
  <th>Business</th>
  <th>Total Revenue</th>
  <th>Projected Revenue</th>
  </tr>
  </thead>
  <tbody>
  <?php foreach ($accounts as $account): ?>
  <tr>
  <td><?= $account->business ?></td>
  <td>$<?= $account->total_revenue ?></td>
  <td>$<?= $account->projected_revenue ?></td>
  </tr>
  <?php endforeach; ?>
  </tbody>
  </table>
  <?php endif; ?>
```

## 工作原理…

我们首先移除 Laravel 默认的密钥。然后，我们使用`artisan`命令为我们生成一个新的密钥，并且它会自动保存在正确的文件中。`artisan`命令创建了一个相当强大的密钥，所以我们不必担心自己想出一个密钥。

在为应用程序创建密钥之后，请确保不要更改它，因为如果您已经使用了一些加密，那么更改密钥将会破坏您的应用程序。

然后我们设置一个数据库表，用来保存我们的敏感数据。在这个例子中，我们将存储企业名称以及一些财务数据。

我们的下一步是设置我们的模型，使用`Eloquent`模型。为了让事情变得更容易一些，我们将在模型中使用 getter 和 setter，这样每当在我们的`Account`模型中设置一个值时，它都会自动使用 Laravel 的`Crypt::encrypt`类进行加密。此外，为了从数据库中获取信息，我们的模型将自动为我们解密它。

接下来，我们创建了一些路由。第一个路由将显示一个表单来添加信息，并显示数据库中已经保存的任何内容。下一个路由只是获取表单输入，并将其保存到我们的账户表中的新行中。添加信息后，我们将被重定向回账户列表和表单页面，并且新数据将显示在页面底部。

然而，如果我们查看数据库本身，我们存储的信息是不可读的文本。这样，如果有人成功入侵我们的数据库，他们也得不到太多信息。

# 哈希密码和其他数据

当我们将用户的密码存储在数据库中时，对密码进行哈希处理是常见的做法。这有助于防止任何未经授权访问数据库的人看到用户的密码。然而，我们可能还希望隐藏用户的电子邮件地址或其他信息，以便没有人能够访问它们。我们可以使用 Laravel 的**Hash**来轻松实现这一点。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装，以及一个正确设置和配置的 MySQL 数据库。

## 如何做…

以下是此配方的步骤…

1.  使用以下命令设置数据库表：

```php
CREATE TABLE register (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  username varchar(255) DEFAULT NULL,
  email char(60) DEFAULT NULL,
  password char(60) DEFAULT NULL,
  PRIMARY KEY (id)
  ) ENGINE=InnoDB AUTO_INCREMENT=1

```

1.  在`views`目录中，使用以下代码创建一个名为`register.php`的文件：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Register</title>
  </head>
  <body>
  <p>
  <h3>Register</h3>
  <form method="post" action="register">
  <label>User Name</label>
  <input name="username"><br>
  <label>Email</label>
  <input name="email"><br>
  <label>Password</label>
  <input name="password"><br>
  <input type="submit">
  </form>
  </p>
  <p style="border-top:1px solid #555">
  <h3>Login</h3>
  <form method="post" action="login">
  <label>User Name</label>
  <input name="username"><br>
  <label>Email</label>
  <input name="email"><br>
  <label>Password</label>
  <input name="password"><br>
  <input type="submit">
  </form>
  </p>
  <hr>
  <table border='1'>
  <?php if ($users): ?>
  <tr>
  <th>User Name</th>
  <th>Email</th>
  <th>Password</th>
  </tr>
  <?php foreach ($users as $user): ?>
  <tr>
  <td><?= $user->username ?></td>
  <td><?= $user->email ?></td>
  <td><?= $user->password ?></td>
  </tr>
  <?php endforeach; ?>
  <?php endif; ?>
  </table>
  </body>
  </html>
```

1.  在我们的`routes.php`文件中，通过添加以下代码创建我们的路由：

```php
Route::get('register', function()
{
  $users = DB::table('register')->get();
  return View::make('register')->with('users', $users);
});

Route::post('register', function()
{
  $data = array(
    'username' => Input::get('username'),
    'email' => Hash::make(Input::get('email')),
    'password' => Hash::make(Input::get('password')));

  DB::table('register')->insert($data);

  return Redirect::to('register');
});

Route::post('login', function()
{
  $user = DB::table('register')->where('username', '=',
    Input::get('username'))->first();
  if (!is_null($user) and Hash::check(Input::get('email'),
    $user->email) and Hash::check(Input::get('password'),
    $user->password)) {
    echo "Log in successful";
  } else {
  echo "Not able to login";
}
});

```

## 它是如何工作的...

要开始这个示例，我们首先设置一个基本的用户表，用于保存用户名、电子邮件地址和密码。在这个示例中，用户名是唯一需要以常规文本形式存在的内容。

在我们的视图中，我们将创建两个表单——一个用于注册，一个用于登录。为了显示来自数据库的原始数据，我们还将显示所有用户的列表，以及他们的电子邮件和密码在表中的样子。

当我们提交注册表单时，信息将被发布到我们的注册路由并放入一个数组中。对于电子邮件和密码，我们使用 Laravel 的`Hash::make()`函数进行哈希处理。然后，我们将数组插入到我们的注册表中，并重定向回表单和列表页面。

重定向后，我们将看到新添加的行，我们的电子邮件和密码已经被哈希处理，并且是一个无法识别的字符串。有趣的是，通过哈希处理的方式，我们可以使用完全相同的数据添加两行，哈希值将完全不同。

接下来，我们可以尝试使用用户名、电子邮件和密码登录。该路由将从与用户名对应的表中抓取一行，然后对输入值和数据库结果运行 Laravel 的`Hash::check()`函数。如果通过，它将返回`TRUE`，我们可以继续进行应用程序。

## 还有更多...

要在生产环境中使用此示例，我们需要对输入进行一些验证。我们可能还希望利用**Eloquent ORM**来使哈希处理变得更容易一些。

如果我们不需要隐藏用户的电子邮件，我们也可以使用 Laravel 内置的`Auth::attempt()`方法。关于此方法的更多信息可以在 Laravel 网站上找到：[`laravel.com/docs/security#authenticating-users`](http://laravel.com/docs/security#authenticating-users)

# 在表单中使用 CSRF 令牌和过滤器

网络表单以黑客试图访问网站或用户信息而臭名昭著。为了使我们的表单更安全，我们可以使用内置在 Laravel 中的**跨站请求伪造**（**CSRF**）策略。这将阻止来自用户会话外部的表单提交。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装。

## 如何做...

以下是完成此示例的步骤：

1.  在`routes.php`文件中，通过以下代码创建用于保存和处理表单的路由：

```php
Route::get('cross-site', function()
{
  return View::make('cross-site');
});

Route::post('cross-site', array('before' => 'csrf',function()
{
  echo 'Token: ' . Session::token() . '<br>';
  dd(Input::all());
}));
```

1.  在`filters.php`文件中，确保`csrf`令牌的`filter`存在，如下所示：

```php
Route::filter('csrf', function()
{
  if (Session::token() != Input::get('_token'))
{
  throw new Illuminate\Session\TokenMismatchException;
}
});
```

1.  在我们的`views`目录中，创建一个名为`cross-site.php`的文件，并按以下代码添加两个测试表单：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>CSRF Login</title>
  </head>
  <body>
  <p>
  <h3>CSRF Login</h3>
  <?= Form::open(array('url' => 'cross-site', 'method' =>'post')) ?>
  <?= Form::token() ?>
  <?= Form::label('email', 'Email') ?>
  <?= Form::text('email') ?>
  <?= Form::label('password', 'Password') ?>
  <?= Form::password('password') ?>
  <?= Form::submit('Submit') ?>
  <?= Form::close() ?>
  </p>
  <hr>
  <p>
  <h3>CSRF Fake Login</h3>
  <?= Form::open(array('url' => 'cross-site', 'method' =>'post')) ?>
  <?= Form::hidden('_token', 'smurftacular') ?>
  <?= Form::label('email', 'Email') ?>
  <?= Form::text('email') ?>
  <?= Form::label('password', 'Password') ?>
  <?= Form::password('password') ?>
  <?= Form::submit('Submit') ?>
  <?= Form::close() ?>
  </p>
  </body>
  </html>
```

1.  在浏览器中，转到`http://{your-server}/cross-site`（其中`{your-server}`是我们正在使用的服务器的名称），然后提交每个表单以查看结果。

## 它是如何工作的...

我们的第一步是为我们的 CSRF 表单创建路由。在表单中，我们只需要添加`Form::token()`函数；这将插入一个隐藏字段，名称为`_token`，值为用户会话 ID。对于提交表单的路由，我们在路由中添加`csrf`前过滤器。如果请求被确定为伪造，页面将返回服务器错误。

我们的下一个表单是一个示例，展示了如果请求试图被伪造会发生什么。对于这个表单，我们手动添加隐藏字段并添加一些随机值，而不是使用`Form::token()`函数。然后当我们提交表单时，页面将显示一个失败消息，并显示`TokenMismatchException`错误。

## 还有更多...

当您使用`Form::open()`函数时，Laravel 还会自动生成一个`csrf`令牌，因此您不需要手动添加它。

# 在表单中使用高级验证

有时我们需要验证表单中不属于框架的内容。这个配方将向您展示如何构建自定义验证规则并应用它。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装。

## 如何做...

以下是完成这个配方的步骤：

1.  在`views`目录中，创建一个名为`valid.php`的文件，使用以下代码来保存我们的表单：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Custom Validation</title>
  </head>
  <body>
  <p>
  <?php if ($errors): ?>
  <?php echo $errors->first('email') ?>
  <?php echo $errors->first('captain') ?>
  <?php endif; ?>
  </p>
  <p>
  <h3>Custom Validation</h3>
  <?= Form::open(array('url' => 'valid', 'method' => 'post'))?>
  <?= Form::label('email', 'Email') ?>
  <?= Form::text('email') ?><br><br>
  <?= Form::label('captain', 'Your favorite captains (choosethree)') ?><br>
  <?= 'Pike: ' . Form::checkbox('captain[]', 'Pike') ?><br>
  <?= 'Kirk: ' . Form::checkbox('captain[]', 'Kirk') ?><br>
  <?= 'Picard: ' . Form::checkbox('captain[]', 'Picard')?><br>
  <?= 'Sisko: ' . Form::checkbox('captain[]', 'Sisko') ?><br>
  <?= 'Janeway: ' . Form::checkbox('captain[]', 'Janeway')?><br>
  <?= 'Archer: ' . Form::checkbox('captain[]', 'Archer')?><br>
  <?= 'Crunch: ' . Form::checkbox('captain[]', 'Crunch')?><br>
  <?= Form::submit('Submit') ?>
  <?= Form::close() ?>
  </p>
  </body>
  </html>
```

1.  在`routes.php`文件中，使用以下代码创建我们的路由：

```php
Route::get('valid', function()
{
  return View::make('valid');
});
Route::post('valid', function()
{
  $rules = array('email' => 'required|email','captain' => 'required|check_three');
  $messages = array('check_three' => 'Thou shalt choose three captains. Nomore. No less. Three shalt be the number thou shaltchoose, and the number of the choosing shall bethree.',);
  $validation = Validator::make(Input::all(), $rules,$messages);
  if ($validation->fails())
  {
  return Redirect::to('valid')->withErrors($validation);
}
  echo "Form is valid!";
});
```

1.  同样在`routes.php`文件中，按照以下代码创建我们的自定义验证：

```php
  Validator::extend('check_three', function($attribute,$value, $parameters)
{
  return count($value) == 3;
});
```

## 它是如何工作的...

首先，我们在视图中创建表单。我们要求一个有效的电子邮件和确切地三个复选框被选中。由于没有确切地三个复选框的 Laravel 验证方法，我们需要创建一个自定义验证。

我们的自定义验证接受输入数组并进行简单计数。如果计数达到三个，它返回`TRUE`。如果不是，则返回`FALSE`并且验证失败。

回到我们的表单处理路由，我们只需要将我们创建的自定义验证器的名称添加到我们的验证规则中。如果我们想设置自定义消息，也可以添加。

## 还有更多...

这个配方的额外验证器在`routes.php`文件中，为了简单起见。如果我们有多个自定义验证器，将它们放在自己的验证器文件中可能是一个更好的主意。为此，我们应该在`app`目录中创建一个名为`validator.php`的文件，并添加任何我们想要的代码。然后，打开`app/start`目录中的`global.php`文件，在文件的最后添加`require app_path().'/validator.php'`函数。这将自动加载所有的验证器。

# 构建一个购物车

电子商务在网络上是一个巨大的业务。大多数电子商务网站的一个重要部分是使用购物车系统。这个配方将介绍如何使用 Laravel 会话来存储销售商品并构建购物车。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装，以及一个正确设置和配置的 MySQL 数据库。

## 如何做...

要完成这个配方，按照以下给定的步骤进行操作：

1.  在我们的数据库中，使用以下 SQL 代码创建一个表并添加一些数据：

```php
CREATE TABLE items (
    id int(10) unsigned NOT NULL AUTO_INCREMENT,
    name varchar(255) DEFAULT NULL,
    description text,
    price int(11) DEFAULT NULL,
    PRIMARY KEY (id)
    ) ENGINE=InnoDB;

  INSERT INTO items VALUES ('1', 'Lamp', 'This is a Lamp.','14');
  INSERT INTO items VALUES ('2', 'Desk', 'This is a Desk.','75');
  INSERT INTO items VALUES ('3', 'Chair', 'This is a
    Chair.', '22');
  INSERT INTO items VALUES ('4', 'Sofa', 'This is a
    Sofa/Couch.', '144');
  INSERT INTO items VALUES ('5', 'TV', 'This is a
    Television.', '89');

```

1.  在`routes.php`文件中，使用以下代码为我们的购物车创建路由：

```php
Route::get('items', function() 
{
  $items = DB::table('items')->get();
  return View::make('items')->with('items', $items)>nest('cart', 'cart', array('cart_items' =>Session::get('cart')));
});

Route::get('item-detail/{id}', function($id)
{
  $item = DB::table('items')->find($id);
  return View::make('item-detail')->with('item', $item)>nest('cart', 'cart', array('cart_items' =>Session::get('cart')));
});

Route::get('add-item/{id}', function($id)
{
  $item = DB::table('items')->find($id);
  $cart = Session::get('cart');
  $cart[uniqid()] = array ('id' => $item->id, 'name' => $item >name, 'price' => $item->price);
  Session::put('cart', $cart);
  return Redirect::to('items');
});

Route::get('remove-item/{key}', function($key)
{
  $cart = Session::get('cart');
  unset($cart[$key]);
  Session::put('cart', $cart);
  return Redirect::to('items');
});

Route::get('empty-cart', function()
{
  Session::forget('cart');
  return Redirect::to('items');
});
```

1.  在`views`目录中，使用以下代码创建一个名为`items.php`的文件：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Item List</title>
  </head>
  <body>
  <div>
  <?php foreach ($items as $item): ?>
  <p>
  <a href="item-detail/<?= $item->id ?>">
  <?= $item->name ?>
  </a> --
  <a href="add-item/<?= $item->id ?>">Add to Cart</a>
  </p>
  <?php endforeach; ?>
  </div>
  <?php $cart_session = Session::get('cart') ?>
  <?php if ($cart_session): ?>
  <?= $cart ?>
  <?php endif; ?>
  </body>
  </html>
```

1.  在`views`目录中，按照给定的代码创建一个名为`item-detail.php`的文件：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Item: <?= $item->name ?></title>
  </head>
  <body>
  <div>
  <h2><?= $item->name ?></h2>
  <p>Price: <?= $item->price ?></p>
  <p>Description: <?= $item->description ?></p>
  <p>
  <a href="../add-item/<?= $item->id ?>">Add to Cart</a>
  </p>
  <p><a href="../items">Item list</a></p>
  </div>
  <? if (Session::has('cart')): ?>
  <?= $cart ?>
  <? endif; ?>
  </body>
  </html>
```

1.  在`views`目录中，创建一个名为`cart.php`的文件，使用以下代码：

```php
  <div class="cart" style="border: 1px solid #555">
  <?php if ($cart_items): ?>
  <?php $price = 0 ?>
  <ul>
  <?php foreach ($cart_items as $cart_item_key =>$cart_item_value): ?>
  <?php $price += $cart_item_value['price']?>
  <li>
  <?= $cart_item_value['name'] ?>: 
  <?= $cart_item_value['price'] ?> (<a href="remove-item/<?= $cart_item_key ?>">remove</a>)
  </li>
  <?php endforeach; ?>
  </ul>
  <p><strong>Total: </strong> <?= $price ?></p>
  <?php endif; ?>
  </div>
```

1.  现在，我们可以在浏览器中输入`http://{your-server}/items`来查看来自我们数据库的项目列表，链接到它们的详细页面，并有一个选项将它们添加到购物车。添加到购物车后，它们将显示在页面底部。

## 它是如何工作的...

要开始这个配方，我们需要设置一个将保存我们想要添加到购物车的项目的数据库表。我们还将添加一些测试项目，这样我们就有一些数据可以使用。

在我们的第一个路由中，我们获取表中所有现有的项目并显示它们。我们还嵌套了一个购物车视图，将显示我们已经添加的项目。在嵌套视图中，我们还发送我们的购物车会话，以便列表可以填充。

我们的下一个路由做了类似的事情，但它只接受一个项目并显示完整信息。

下一个路由实际上添加了项目。首先，我们根据其 ID 从数据库中获取项目。然后我们将现有的购物车会话保存到一个变量中，以便我们可以操作它。我们使用 php 的`uniqid()`函数作为键将项目添加到数组中。然后我们将`cart`数组放回`Session`并将其重定向。

如果我们想要删除一个项目，首先我们要找到获取项目的 ID 并从`cart`数组中删除它。另一种方法是只删除所有会话并重新开始。

在我们的视图中，我们还会注意到，只有在购物车中实际有东西的情况下，才允许显示`cart`列表。

## 还有更多...

这个配方可以很容易地扩展为更全面的功能。例如，如果我们多次点击同一项，可以存储每个项目的总数，而不是添加新记录。这样，我们可以在项目旁边添加一个要求数量的表单字段。

# 使用 Redis 保存会话

Redis 是一种流行的键/值数据存储，速度相当快。Laravel 包括对 Redis 的支持，并且可以轻松地与 Redis 数据交互。

## 准备工作

对于这个配方，我们需要一个正确配置和运行的 Redis 服务器。关于这方面的更多信息可以在[`redis.io/`](http://redis.io/)找到。

## 如何做...

按照以下步骤完成这个配方：

1.  在我们的`routes.php`文件中，按照以下代码创建路由：

```php
Route::get('redis-login', function()
{
  return View::make('redis-login');
});

Route::post('redis-login', function()
{
  $redis = Redis::connection();
  $redis->hset('user', 'name', Input::get('name'));
  $redis->hset('user', 'email', Input::get('email'));
  return Redirect::to('redis-view');
});

Route::get('redis-view', function()
{
  $redis = Redis::connection();
  $name = $redis->hget('user', 'name');
  $email = $redis->hget('user', 'email');
  echo 'Hello ' . $name . '. Your email is ' . $email;
});
```

1.  在`views`目录中，创建一个名为`redis-login.php`的文件，其中包含以下代码：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Redis Login</title>
  </head>
  <body>
  <p>
  <h3>Redis Login</h3>
  <?= Form::open(array('url' => 'redis-login', 'method' =>'post')) ?>
  <?= Form::label('name', 'Your Name') ?>
  <?= Form::text('name') ?>
  <?= Form::label('email', 'Email') ?>
  <?= Form::text('email') ?>
  <?= Form::submit('Submit') ?>
  <?= Form::close() ?>
  </p>
  </body>
  </html>
```

1.  现在，我们可以打开浏览器，转到`http://{your-server}/redis-login`，并填写表单。提交后，我们将显示来自 Redis 的信息。

## 工作原理...

我们的第一步是创建一个简单的表单，用于将数据输入到 Redis 中。在我们的`redis-login`路由中，我们使用一个视图，该视图将要求输入姓名和电子邮件地址，并在提交时将发布到`redis-login`路由。

发布后，我们使用`Redis::connection()`函数创建一个新的 Redis 实例，该函数将使用我们的`app/config/database.php`文件中找到的默认设置。为了将信息存储在 Redis 中，我们使用一个哈希并使用`hset()`函数设置数据。我们的 Redis 实例可以使用 Redis 接受的任何命令，因此我们可以很容易地在`set()`或`sadd()`等函数之间进行选择。

一旦数据在 Redis 中，我们重定向到一个将显示数据的路由。为此，我们只需要使用键和我们添加的字段调用`hget()`函数。

# 使用基本会话和 cookie

有时我们希望在应用程序的一个页面和另一个页面之间传递数据，而不需要将信息存储在数据库中。为了实现这一点，我们可以使用 Laravel 提供的各种`Session`和`Cookie`方法。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装。

## 如何做...

对于这个配方，按照给定的步骤：

1.  在`views`文件夹中，创建一个名为`session-one.php`的文件，其中包含以下代码：

```php
  <!DOCTYPE html>
  <html>
  <head>
  <title>Laravel Sessions and Cookies</title>
  <meta charset="utf-8">
  </head>
  <body>
  <h2>Laravel Sessions and Cookies</h2>
  <?= Form::open() ?>
  <?= Form::label('email', 'Email address: ') ?>
  <?= Form::text('email') ?>
  <br>
  <?= Form::label('name', 'Name: ') ?>
  <?= Form::text('name') ?>
  <br>
  <?= Form::label('city', 'City: ') ?>
  <?= Form::text('city') ?>
  <br>
  <?= Form::submit('Go!') ?>
  <?= Form::close() ?>
  </body>
  </html>
```

1.  在`routes.php`文件中，按照以下代码创建我们的路由：

```php
Route::get('session-one', function()
{
  return View::make('session-one');
});

Route::post('session-one', function()
{
  Session::put('email', Input::get('email'));
  Session::flash('name', Input::get('name'));
  $cookie = Cookie::make('city', Input::get('city'), 30);
  return Redirect::to('session-two')->withCookie($cookie);
});

Route::get('session-two', function()
{
  $return = 'Your email, from a Session, is 'Session::get('email') . '. <br>';
  $return .= 'You name, from flash Session, is 'Session::get('name') . '. <br>';
  $return .= 'You city, from a cookie, is ' .Cookie::get('city') . '.<br>';
  $return .= '<a href="session-three">Next page</a>';
  echo  $return;
});

Route::get('session-three', function()
{
  $return = '';

  if (Session::has('email')) {
  $return .= 'Your email, from a Session, is ' . Session::get('email') . '. <br>';
} else {
$return .= 'Email session is not set.<br>';
}

if (Session::has('name')) {
  $return .= 'Your name, from a flash Session, is ' . Session::get('name') . '. <br>';
} else {
$return .= 'Name session is not set.<br>';
}

if (Cookie::has('city')) {
  $return .= 'Your city, from a cookie, is ' . Cookie::get('city') . '. <br>';
} else {
  $return .= 'City cookie is not set.<br>';
}
  Session::forget('email');
  $return .= '<a href="session-three">Reload</a>';
  echo $return;
});
```

## 工作原理...

首先，我们创建一个简单的表单，用于提交信息到会话和 cookie。在提交值之后，我们取`email`字段并将其添加到常规会话中。`name`字段将被添加到闪存会话中，`city`将被添加到 cookie 中。此外，我们将设置 cookie 在 30 分钟后过期。一旦它们都设置好了，我们重定向到我们的第二个页面，并确保将 cookie 传递给返回值。

我们的第二个页面只是获取我们设置的值，并显示它们以验证它们是否被正确设置。此时，一旦请求完成，我们的闪存会话，即名称，应该不再可用。

当我们点击到第三个页面时，我们添加了一些检查，以确保会话和 cookie 仍然存在，使用`has()`方法。我们的`email`和`city`应该仍然显示，但`name`会话不应该。然后我们使用`forget()`方法删除`email`会话。当我们重新加载页面时，我们会注意到唯一仍然显示的是`city` cookie。

## 还有更多...

闪存数据仅在我们进行的下一个请求中可用，然后被删除。但是，如果我们想保留我们的闪存数据，我们可以使用`Session::reflash()`命令，它也会将数据发送到我们的下一个请求。如果我们有多个闪存数据，我们还可以使用`Session::keep(array('your-session-key', 'your-other-session'))`函数选择保留特定会话以供下一个请求使用。

# 创建安全的 API 服务器

在这个食谱中，我们将创建一个简单的 API 来显示来自我们数据库的一些信息。为了控制谁可以访问数据，我们允许用户创建密钥并在其 API 请求中使用该密钥。

## 准备工作

对于这个食谱，我们需要一个标准的 Laravel 安装和一个配置好的 MySQL 数据库。

## 如何做到这一点...

要完成这个食谱，我们将按照以下给定的步骤进行：

1.  在我们的数据库中，创建一个表来保存 API 密钥，如下面的代码所示：

```php
CREATE TABLE api (
id int(10) unsigned NOT NULL AUTO_INCREMENT,
 name varchar(255) DEFAULT NULL,
 api_key varchar(255) DEFAULT NULL,
 status tinyint(1) DEFAULT NULL,
 PRIMARY KEY (id)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

1.  在数据库中，创建一个表以访问一些示例数据，如下面的代码所示：

```php
CREATE TABLE shows (id int(10) unsigned NOT NULL AUTO_INCREMENT,name varchar(200) NOT NULL,year int(11) NOT NULL,created_at datetime NOT NULL,updated_at datetime NOT NULL,PRIMARY KEY (id)) ENGINE=InnoDB CHARSET=utf8;

  INSERT INTO shows VALUES ('1', 'Happy Days', '1979','2013-01-01 00:00:00', '2013-01-01 00:00:00');
  INSERT INTO shows VALUES ('2', 'Seinfeld', '1999', '2013-01-01 00:00:00', '2013-01-01 00:00:00');
  INSERT INTO shows VALUES ('3', 'Arrested Development', '2006', '2013-01-01 00:00:00', '2013-01-01 00:00:00');
  INSERT INTO shows VALUES ('4', 'Friends', '1997','2013-01-01 00:00:00', '2013-01-01 00:00:00');
```

1.  在`models`目录中，创建一个名为`Api.php`的文件

```php
  <?php

class Api extends Eloquent {

  public $table = 'api';
  public $timestamps = FALSE;
}
```

1.  在`models`目录中，创建一个名为`Show.php`的文件

```php
  <?php
class Show extends Eloquent {
}
```

1.  在`views`目录中，创建一个名为`api-key.php`的文件

```php
  <!DOCTYPE html>
  <html>
  <head>
  <title>Create an API key</title>
  <meta charset="utf-8">
  </head>
  <body>
  <h2>Create an API key</h2>
  <?php echo Form::open() ?>
  <?php echo Form::label('name', 'Your Name: ') ?>
  <?php echo Form::text('name') ?>
  <br>
  <?php echo Form::submit('Go!') ?>
  <?php echo Form::close() ?>
  </body>
  </html>
```

1.  在`routes.php`文件中，创建路由以允许`api-key`注册

```php
Route::get('api-key', function() {
  return View::make('api-key');
});

Route::post('api-key', function() {
  $api = new Api();
  $api->name = Input::get('name');
  $api->api_key = Str::random(16);
  $api->status = 1;
  $api->save();
  echo 'Your key is: ' . $api->api_key;
});
```

1.  在`routes.php`中，通过以下代码创建访问`api`的路由：

```php
Route::get('api/{api_key}/shows', function($api_key)
{
  $client = Api::where('api_key', '=', $api_key)->where('status', '=', 1)->first();
  if ($client) {
  return Show::all();
  } else {
  return Response::json('Not Authorized', 401);
  }
});
Route::get('api/{api_key}/show/{show_id}', function($api_key, $show_id)
{
  $client = Api::where('api_key', '=', $api_key)->where('status', '=', 1)->first();
  if ($client) {
  if ($show = Show::find($show_id)) {
  return $show;
  } else {
  return Response::json('No Results', 204);
  }
  } else {
  return Response::json('Not Authorized', 401);
  }
});
```

1.  要测试它，在浏览器中，转到`http://{your-server}/api-key`（其中`{your-server}`是开发服务器的名称），并填写表单。在下一页中，复制生成的密钥。然后，转到`http://{your-server}/api/{your-copied-key}/shows`，应该以`json`格式显示节目列表。

## 它是如何工作的...

我们首先设置我们的表和模型。我们的 API 表将用于检查密钥，`show`表将是我们将使用密钥访问的测试数据。

我们的下一个任务是创建一种为我们的应用程序生成密钥的方法。在这个例子中，我们只会接受一个名称值。提交后，我们创建一个随机的 16 个字符的字符串，这将是用户的密钥。然后，我们将信息保存到表中，并将密钥显示给用户。

要使用此密钥，我们创建两个路由来显示信息。第一个路由在 URL 中使用`{api_key}`通配符，并将该值传递给我们的函数。然后，我们查询该密钥的数据库，并确保状态仍然是活动的。这样，如果我们决定撤销用户的密钥，我们可以将状态设置为 false，他们将无法使用 API。如果它们不存在或状态为 false，则以 401 的 HTTP 代码响应，以显示他们未经授权。否则，我们返回 Eloquent 对象，这将允许我们以`json`格式显示记录。

我们的第二个路由将显示单个节目的记录。对于该 URL，我们使用`{api_key}`通配符作为密钥，使用`{show_id}`通配符作为节目的 ID。我们将这些传递给函数，然后像以前一样检查密钥。如果密钥有效，我们确保具有该 ID 的节目存在，并再次使用 Eloquent 对象以`json`格式仅显示具有给定 ID 的节目。

## 还有更多...

我们还可以选择使用 Laravel 过滤器，如果我们宁愿使用 API 密钥发布。为此，我们将在`filters.php`文件中创建一个新的过滤器

```php
Route::filter('api', function()
{
  if ($api_key = Input::get('api_key')) {
  $client = Api::where('api_key', '=', $api_key)->where('status', '=', 1)->first();
  if (!$client) {
  return Response::json('Not Authorized', 401);
}
  } else {
  return Response::json('Not Authorized', 401);
}
});
```

然后，对于我们的`shows`路由，我们响应一个 post 请求，并添加`before`过滤器，如下面的代码所示：

```php
Route::post('api/shows', array('before' => 'api', function()
{
  return Show::all();
}));
```
