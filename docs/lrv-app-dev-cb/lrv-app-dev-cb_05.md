# 第五章：使用控制器和路由处理 URL 和 API

在本章中，我们将涵盖：

+   创建一个基本控制器

+   使用闭包创建路由

+   创建一个 RESTful 控制器

+   使用高级路由

+   在路由上使用过滤器

+   使用路由组

+   使用路由构建 RESTful API

+   使用命名路由

+   在您的路由中使用子域

# 介绍

在本章中，我们将介绍一些使用 Laravel 路由系统的方法。路由应用程序有两种基本方法：要么在`routes.php`文件中使用闭包设置路由，要么使用控制器。我们将看到每种方法的强大之处，并展示它们如何在我们的应用程序中使用。

# 创建一个基本控制器

**模型-视图-控制器**（**MVC**）模式在 PHP 框架中非常流行。在这个示例中，我们将创建一个简单的控制器，它扩展了另一个基本控制器。

## 准备工作

首先，我们只需要一个标准的 Laravel 安装。

## 如何做...

要完成这个步骤，按照以下步骤进行：

1.  在`app/controllers`目录中，创建一个名为`UsersController.php`的文件，并输入以下代码：

```php
<?php
class  UsersController extends BaseController {

  public function actionIndex()
  {
    return "This is a User Index page";
  }

  public function actionAbout()
  {
    return "This is a User About page";
  }
}
```

1.  然后，在`routes.php`文件中添加以下行：

```php
Route::get('users', 'UsersController@actionIndex');
Route::get('users/about', 'UsersController@actionAbout');
```

1.  通过访问`http://your-server/users`和`http://your-server/users/about`来测试控制器，其中`your-server`是您的应用程序的 URL。

## 它是如何工作的...

在我们的用户控制器（以及我们创建的几乎所有其他控制器中），我们首先通过扩展基本控制器来开始。如果我们查看`BaseController.php`文件，我们只会看到一个方法，即`setupLayout()`方法，它用于我们的布局视图。如果有一些代码我们希望在站点的每个页面上运行，基本控制器也可以使用。

回到用户控制器，在那里我们为我们的首页和关于页面定义了两个方法，每个方法都以`action`为前缀。对于我们的目的，我们只是返回一个字符串，但这将是我们所有控制器逻辑的地方，并且我们将设置要显示的视图。

这样，Laravel 就能解析 URL 并确定要使用哪个控制器和方法，我们需要在`routes`文件中注册路由。现在，在我们的浏览器中，当我们访问`/users`（或`/users/index`）时，我们将被带到我们的首页，而`/users/about`将带我们到我们的关于页面。

# 使用闭包创建路由

如果我们决定不使用 MVC 模式，我们可以通过使用闭包或匿名函数来创建我们的路由。

## 准备工作

对于这个示例，我们只需要一个标准的 Laravel 安装。

## 如何做...

要完成这个步骤，按照以下步骤进行：

1.  在`app/routes.php`文件中，添加以下路由：

```php
Route::get('hello/world', function()
{
  $hello = 'Hello ';
  $world = 'World!';
  return $hello . $world;
});
```

1.  打开浏览器，通过访问`http://your-server/hello/world`来测试路由，其中`your-server`是您的应用程序的 URL。

## 它是如何工作的...

Laravel 中的路由被认为是 RESTful 的，这意味着它们响应不同的 HTTP 动词。大多数时候，当简单地查看网页时，我们使用`GET`动词，如`Route::get`。我们的第一个参数是我们用于路由的 URL，它可以是几乎任何有效的 URL 字符串。在我们的情况下，当用户转到`hello/world`时，它将使用这个路由。之后是我们的闭包，或匿名函数。

在闭包中，我们可以从我们的模型中提取任何数据，进行我们想要的任何逻辑，并调用我们想要使用的视图。在我们的示例中，我们只是设置了一些变量并返回它们连接的值。

# 创建一个 RESTful 控制器

也许有一天我们想要拥有一个 RESTful 的 Web 应用程序，比如构建一个 API。为了实现这一点，我们需要我们的路由响应各种 HTTP 请求。闭包的路由已经以这种方式设置，但在这个示例中，我们将保持 MVC 模式，并创建一个 RESTful 的控制器。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装和*创建一个基本控制器*示例中的代码。

## 如何做...

要完成这个步骤，按照以下步骤进行：

1.  在用户控制器中，用以下代码替换代码：

```php
<?php
class  UsersController extends BaseController {

  public function getIndex()
  {
    $my_form = "<form method='post'>
                      <input name='text' value='Testing'>
                      <input type='submit'>
                      </form>";
    return $my_form;

  }
  public function postIndex()
  {
    dd(Input::all());
  }

  public function getAbout()
  {
     return "This is a User About page";
  }
}
```

1.  在`routes.php`中，添加到我们的控制器的路由：

```php
Route::controller('users', 'UsersController');
```

1.  在浏览器中，转到`http://your-server/users`（其中`your-server`是您的 Web 服务器的 URL），然后单击**提交**按钮。

1.  在浏览器中，转到`http://your-server/users/about`。

## 它是如何工作的...

RESTful 和非 RESTful 控制器的两个主要区别是将方法重命名为它们响应的 HTTP 请求作为前缀，并使用`Route::controller()`注册我们的路由。

我们的`getIndex()`方法是当我们转到`/users`时的默认方法，因为大多数页面视图都是`GET`请求。在这个例子中，我们返回一个非常简单的表单，该表单将把输入提交回自身。然而，由于表单使用了`POST`请求，它将触发`postIndex()`方法，这就是表单可以被处理的地方。在我们的示例中，我们只是使用 Laravel 的`dd()`助手来显示提交的表单输入。

# 使用高级路由

在创建需要参数的路由时，我们可能需要使用更高级的功能。使用 Laravel 和正则表达式，我们可以确保我们的路由只响应特定的 URL。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装。

## 操作步骤…

要完成这个示例，请按照以下步骤操作：

1.  在我们的`routes.php`文件中，添加以下代码：

```php
Route::get('tvshow/{show?}/{year?}', function($show = null, $year = null)
{
  if (!$show && !$year)
  {
    return 'You did not pick a show.';
  }
  elseif (!$year)
  {
      return 'You picked the show <strong>' . $show . '</strong>';
  }

  return 'You picked the show <strong>' . $show .'</strong> from the year <em>' . $year . '</em>.';
})
->where('year', '\d{4}');
```

1.  打开浏览器，并通过在地址栏中输入`http://your-server/tvshow/MASH/1981`（其中`your-server`是您服务器的 URL）来测试路由。

## 它是如何工作的...

我们首先让我们的路由响应`GET`请求的`tvshow`。如果我们想要向路由传递参数，我们需要设置通配符。只要我们将相同的名称传递给函数，我们可以使用任意多个参数并且可以随意命名它们。对于这个示例，我们想要获取一个节目标题，并且为了使这个参数可选，我们在末尾添加了问号。

对于我们的第二个参数，我们需要一个`year`。在这种情况下，它必须是一个四位数。要使用正则表达式，我们需要将`where()`方法链接到我们的路由上，并使用参数的名称和表达式。在这个例子中，我们只想要数字（`\d`），并且必须有四个数字（`{4}`）。路由参数中的问号使字段变为可选。

在我们的闭包中，我们使用相同的名称设置每个通配符的变量。为了使它们可选，我们将每个变量默认设置为`null`。然后我们检查参数是否已设置，如果是，则返回适当的消息。

# 在路由上使用过滤器

Laravel 的一个强大功能是添加过滤器，可以在请求发送到我们的应用程序之前和之后运行。在这个示例中，我们将探讨这些过滤器。

## 准备工作

对于这个示例，我们只需要一个标准的 Laravel 安装。

## 操作步骤...

要完成这个示例，请按照以下步骤操作：

1.  在我们的`routes.php`文件中，添加一个只有管理员可以访问的路由，并附加过滤器：

```php
Route::get('admin-only', array('before' => 'checkAdmin', 'after' => 'logAdmin', function() 
{
  return 'Hello there, admin!';
}));
```

1.  在我们的`filters.php`文件中添加两个过滤器：

```php
Route::filter('checkAdmin', function()
{
  if ('admin' !== Session::get('user_type')) 
  {
    return 'You are not an Admin. Go Away!';
  }
});

Route::filter('logAdmin', function()
{
  Log::info('Admin logged in on ' . date('m/d/Y'));
});
```

1.  创建一个可以设置管理员会话的路由：

```php
Route::get('set-admin', function()
{
  Session::put('user_type', 'admin');
  return Redirect::to('admin-only');
});
```

1.  通过转到`http://your-server/admin-only`（其中`your-server`是您服务器的 URL）来测试路由，并注意结果。然后，转到`set-admin`并查看这些结果。

1.  转到`app/storage/logs`目录并查看日志文件。

## 它是如何工作的...

在我们的`admin-only`路由中，我们不只是添加闭包，而是添加一个包含闭包的数组作为最后一个参数。对于我们的目的，我们希望在访问路由之前检查`user_type`会话是否设置为`admin`。我们还希望在页面处理后记录每次有人访问路由，但只有在页面处理后才记录。

在我们的`before`过滤器中，我们简单地检查一个会话，如果该会话不等于`admin`，我们返回一个通知并阻止路由返回其消息。如果会话等于`admin`，则路由会正常进行。

在访问路由之后，我们创建一个访问的日志以及访问路由的日期。

在这一点上，如果我们在浏览器中去到`admin-only`，`before`过滤器会启动并显示错误消息。然后，如果我们去到我们的日志目录并查看日志，它会显示尝试的时间、日志消息的名称和响应。对于我们来说，它会显示**You are not an Admin. Go Away!**。

为了使路由可访问，我们创建另一个路由，简单地设置我们想要的会话，然后重定向回我们的`admin-only`页面。如果我们访问`set-admin`，它应该自动将我们重定向到`admin-only`并显示成功页面。此外，如果我们查看我们的日志，我们会看到我们成功尝试的行。

## 还有更多...

这是一个非常基本的身份验证方法，只是为了展示过滤器的有用性。对于更好的身份验证，使用 Laravel 内置的方法。

# 使用路由组

在创建 Web 应用程序时，我们可能会发现一些需要相同 URL 前缀或过滤器的路由。使用 Laravel 的路由组，我们可以轻松地将它们应用到多个路由。

## 准备工作

对于这个示例，我们只需要一个标准的 Laravel 安装。

## 操作方法…

要完成这个示例，请按照以下步骤进行：

1.  在我们的`app/filters.php`文件中，创建一个检查用户的过滤器：

```php
Route::filter('checkUser', function()
{
  if ('user' !== Session::get('profile'))
  {
    return 'You are not Logged In. Go Away!';
  }
});
```

1.  在`app/routes.php`文件中，创建一个可以设置我们的个人资料会话的路由：

```php
Route::get('set-profile', function()
{
  Session::set('profile', 'user');
  return Redirect::to('profile/user');
});
```

1.  在`routes.php`中，创建我们的路由组：

```php
Route::group(array('before' => 'checkUser', 'prefix' => 'profile'), function()
{
    Route::get('user', function()
    {
        return 'I am logged in! This is my user profile.';
    });
    Route::get('friends', function()
    {
      return 'This would be a list of my friends';
    });
});
```

1.  在我们的浏览器中，然后我们去到`http://path/to/our/server/profile/user`，我们会得到一个错误。如果我们然后去到`http://path/to/our/server/set-profile`，它会重定向我们并显示正确的页面。

## 它是如何工作的...

我们需要做的第一件事是创建一个过滤器。这个简单的过滤器将检查一个会话名称，`profile`，看看它是否等于`user`。如果不是，它就不会让我们继续下去。

在我们的路由中，然后创建一个将为我们设置`profile`会话然后重定向我们到路由组的路由。通常在登录后会设置会话，但这里我们只是测试以确保它有效。

最后，我们创建我们的路由组。对于这个组，我们希望在允许访问之前，组内的每个路由都要通过`checkUser`过滤器。我们还希望这些路由在它们之前有`profile/`。我们通过在调用组的闭包之前将它们添加到数组中来实现这一点。现在，我们在这个组内创建的任何路由都必须通过过滤器，并且可以使用`profile`前缀访问。

# 使用路由构建 RESTful API

现代 Web 应用程序的一个常见需求是拥有一个第三方可以运行查询的 API。由于 Laravel 是以 RESTful 模式为重点构建的，因此很容易用很少的工作来构建一个完整的 API。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装，并且正确配置了 MySQL 数据库，与我们的应用程序连接起来。

## 操作方法…

要完成这个示例，请按照以下步骤进行：

1.  打开命令行，转到 Laravel 安装的根目录，并使用以下命令为我们的表创建一个迁移：

```php
php artisan migrate:make create_shows_table
```

1.  在`app/database/migrations`目录中，找到类似`2012_12_01_222821_create_shows_table.php`的文件，并按照以下方式创建我们表的模式：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class CreateShowsTable extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
    Schema::create('shows', function($table)
    {
        $table->increments('id');
        $table->string('name');
        $table->integer('year');
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
    Schema::drop('shows');
  }
}
```

1.  回到命令行，按照以下方式运行迁移：

```php
php artisan migrate
```

1.  创建另一个迁移以添加一些测试数据：

```php
php artisan migrate:make add_shows_data
```

1.  在`app/database/migrations`文件夹中，打开`add_shows_data`文件，并添加以下查询：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class AddShowsData extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
      $shows = array(
              array(
                    'name' => 'Happy Days',
                    'year' => 1981
              ),
              array(
                    'name' => 'Seinfeld',
                    'year' => 1998
              ),
              array(
                   'name' => 'Arrested Development',
                   'year' => 2006
              )
      );
      DB::table('shows')->insert($shows);
  }

  /**
   * Reverse the migrations.
   *
   * @return void
   */
  public function down()
  {
    DB::table('shows')->delete();
  }
}
```

1.  在命令行中，按照以下方式运行迁移：

```php
php artisan migrate
```

1.  在`app/models`目录中，创建一个名为`Show.php`的文件，并添加以下代码：

```php
<?php
class Show extends Eloquent {
  protected $table = 'shows';
}
```

1.  在`routes.php`中，创建一个返回所有 show 或单个 show 的 JSON 的路由：

```php
Route::get('show/{id?}', function($id = null)
{
  if (!$id)
  {
    return Show::all();
  }
  if ($show = Show::find($id))
  {
    return $show;
  }
});
```

1.  创建一个将添加新 show 的路由如下：

```php
Route::post('show', function()
{
  $show = new Show;
  $show->name = Input::get('name');
  $show->year = Input::get('year');
  $show->save();
  return $show;
});
```

1.  创建一个将删除记录的路由：

```php
Route::delete('show/{id}', function($id)
{
  if ($show = Show::find($id))
  {
    $show->delete();
    return json_encode(array('message' => 'Record ' . $id. ' deleted.'));
  }
});
```

1.  创建一个更新记录的路由：

```php
Route::put('show/{id}', function($id)
{
  if ($show = Show::find($id))
  {
       if (Input::get('name')) {
           $show->name = Input::get('name');
    }
       if (Input::get('year')) {
           $show->year = Input::get('year');
       }

       $show->save();
       return $show;
 }
});
```

1.  创建一个路由来保存我们的添加和编辑`show form`：

```php
Route::get('show-form/{id}', function($id = null)
{
  $data = array();

  if ($id) 
  {
       if (!$show = Show::find($id))
       {
          return 'No show with that ID';
       }

       $data = array(
             'id'     => $id,
             'method' => 'PUT',
             'name'   => $show->name,
             'year'   => $show->year
        );
  } 
  else 
  {
       $data = array(
             'id'     => '',
             'method' => 'POST',
             'name'   => '',
             'year'   => ''
       );
  }
  return View::make('show-form', $data);
});
```

1.  创建一个路由来显示一个列表，以便我们可以删除一个 show：

```php
Route::get('show-delete', function()
{
  $shows = Show::all();
  return View::make('show-delete')->with('shows',$shows);
});
```

1.  在我们的`app/views`文件夹中，创建一个名为`show-form.php`的文件，并添加以下代码：

```php
<?php echo Form::open(array('url' => 'show/' . $id, 'method' => $method)) ?>
<?php echo Form::label('name', 'Show Name: ') . Form::text('name', $name) ?>
<br>
<?php echo Form::label('year', 'Show Year: ') . Form::text('year', $year) ?>
<br>
<?php echo Form::submit() ?>
<?php echo Form::close() ?>
```

1.  然后，在`app/views`中，创建一个名为`show-delete.php`的文件，并添加以下代码：

```php
<?php foreach ($shows as $show): ?>
  <?php echo Form::open(array('url' => 'show/' . $show->id, 'method' => 'DELETE')) ?>
  <?php echo Form::label('name', 'Show Name: ') . $show->name ?>
  <?php echo Form::submit('Delete') ?>
  <?php echo Form::close() ?>
<?php endforeach; ?>
```

1.  通过浏览器访问`show-form`和`show-delete`路由来测试它。

## 工作原理...

我们的第一步是使用 artisan 和 migrations 创建我们想要使用的数据表。我们创建一个 shows 表，然后添加一些测试数据。

对于我们的路由，我们将响应四种不同的 HTTP 动词，`GET`，`POST`，`PUT`和`DELETE`，但都在同一个 URL，`show`上。`GET`请求将有两个目的。首先，如果 URL 中没有传入 ID，它将显示来自数据库的整个列表。其次，如果有 ID，它将显示单个记录。通过直接返回 eloquent 对象，它将自动将我们的对象显示为 JSON。

我们的下一个路由响应`POST`请求，并将在数据库中添加一个新记录。然后显示保存的记录为 JSON。

然后，我们添加一个响应`DELETE`请求的路由。它获取`id`参数，删除记录，并显示 JSON 以确认删除成功。

最后，我们有一个响应`PUT`请求和`id`参数的路由。该路由将加载传入 ID 的记录，然后编辑值。如果更新正确，它会显示更新后的记录的 JSON。

要展示 API 的运行情况，我们需要创建一个表单来添加和更新记录。我们的`show-form`路由检查是否传入了 ID，如果是，则使用`PUT`方法创建一个表单，并将记录的值加载到字段中。如果没有设置 ID，我们将使用`POST`方法创建一个空白表单。

如果我们想要删除记录，我们的`show-delete`路由将显示一个节目列表，并在每个节目旁边显示一个删除按钮。这些按钮实际上是使用`DELETE`方法的表单的一部分。

我们还可以使用命令行中的`curl`来测试路由。例如，要获取完整列表，请使用以下代码行：

```php
curl -X GET http://path/to/our/app/show
```

要发布到 API，请使用以下代码行：

```php
curl --data "name=Night+Court&year=1984" http://path/to/our/app/show
```

## 还有更多...

请记住，这个 API 示例非常基础。要使其更好，我们需要在添加或更新记录时添加一些验证。还可以考虑添加某种身份验证，以便公众无法更改我们的表格和删除记录。

我们还可以使用 Laravel 的资源控制器来实现类似的功能。有关更多信息，请参阅文档[`laravel.com/docs/controllers#resource-controllers`](http://laravel.com/docs/controllers#resource-controllers)。

# 使用命名路由

有时我们需要更改路由的名称。在一个大型网站上，如果我们有多个链接指向错误的路由，这可能会引起很多问题。Laravel 提供了一种简单易用的方式来为我们的路由分配名称，这样我们就不必担心它们是否会更改。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装。

## 如何做...

要完成这个示例，请按照以下步骤操作：

1.  在我们的`routes.php`文件中，创建一个命名路由如下：

```php
Route::get('main-route', array('as' => 'named', function()
{
  return 'Welcome to ' . URL::current();
}));
```

1.  创建一个执行简单重定向到命名路由的路由：

```php
Route::get('redirect', function()
{
  return Redirect::route('named');
});
```

1.  创建一个显示链接到命名路由的路由：

```php
Route::get('link', function()
{
  return '<a href="' . URL::route('named') . '">Link!</a>';
});
```

1.  在浏览器中，访问`http://your-server/redirect`和`http://your-server/link`（其中`your-server`是服务器的 URL），注意它们将我们发送到`main-route`路由。

1.  现在，将`main-route`路由重命名为`new-route`：

```php
Route::get('new-route', array('as' => 'named', function()
{
  return 'Welcome to ' . URL::current();
}));
```

1.  在浏览器中，访问**redirect**和**link**路由，看看它们现在将我们发送到哪里。

## 工作原理...

有时您的路由可能需要更改；例如，如果客户有一个博客，但希望路由“posts”变成“articles”。如果我们在整个网站上都有指向“posts”路由的链接，这意味着我们需要找到每个文件并确保它们已更改。通过使用命名路由，我们可以将路由重命名为任何我们想要的名称，只要我们所有的链接都指向该名称，一切都会保持更新。

在我们的示例中，我们有路由`main-route`并将其命名为`named`。现在，如果我们想要链接或重定向到该路由，我们可以使用`route()`指向命名路由。然后，如果我们将路由更改为`new-route`并重新检查这些链接，它将自动转到更改后的路由。

# 在您的路由中使用子域

许多现代 Web 应用程序为其用户提供定制内容，包括为他们提供一个可以访问其内容的自定义子域。例如，用户的个人资料页面不是`http://example.com/users/37`，我们可能希望提供`http://username.example.com`。通过更改一些 DNS 和 Apache 设置，我们可以在 Laravel 中轻松提供相同的功能。

## 准备就绪

对于这个配方，我们需要访问我们的 DNS 设置和我们服务器的 Apache 配置。我们还需要一个正确配置的 MySQL 数据库和一个标准的 Laravel 安装。在整个配方中，我们将使用`example.com`作为域名。

## 如何做...

要完成这个配方，请按照以下步骤操作：

1.  在我们域名的 DNS 中，我们需要添加一个"A"记录，使用通配符为子域，例如`*.example.com`，然后将其指向我们服务器的 IP 地址。

1.  打开 Apache 的`httpd.conf`文件，并添加一个虚拟主机，如下所示：

```php
<VirtualHost *:80>
  ServerName example.com
  ServerAlias *.example.com
</VirtualHost>
```

1.  在命令行中，转到我们的应用程序路由并为我们的`names`表创建一个迁移：

```php
php artisan migrate:make create_names_table
```

1.  在`migrations`目录中，打开`create_names_table`文件并添加我们的模式：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class CreateNamesTable extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
       Schema::create('users', function($table)
       {
            $table->increments('id');
            $table->string('name');
            $table->string('full_name');
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
    Schema::drop('name');
  }
}
```

1.  回到命令行，创建另一个迁移以添加一些测试数据：

```php
php artisan migrate:make add_names_data
```

1.  在`migrations`目录中打开`add_names_data`文件：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class AddNamesData extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
     $names = array(
                array(
                      'name' => 'bob',
                      'full_name' => 'Bob Smith'
                      ),
                        array(
                             'name' => 'carol',
                             'full_name' => 'Carol Smith'
                           ),
                          array(
                               'name' => 'ted',
                               'full_name' => 'Ted Jones'
                           )
                    );
     DB::table('name')->insert($names);
  }

  /**
   * Reverse the migrations.
   *
   * @return void
   */
  public function down()
  {
      DB::table('name')->delete();
  }
}
```

1.  在命令行中，运行迁移如下：

```php
php artisan migrate
```

1.  创建一个路由，根据子域从`names`表中获取信息：

```php
Route::get('/', function()
{
  $url = parse_url(URL::all());
  $host = explode('.', $url['host']);
  $subdomain = $host[0];

  $name = DB::table('name')->where('name',$subdomain)->get();

  dd($name);
});
```

1.  在浏览器中，访问我们的域名，使用相关子域，例如`http://ted.example.com`。

## 它是如何工作的...

首先，我们需要更新我们的 DNS 和我们的服务器。在我们的 DNS 中，我们创建一个通配符子域，并在我们的 Apache 配置中创建一个虚拟主机。这样可以确保使用的任何子域都将转到我们的主要应用程序。

对于我们的默认路由，我们使用 PHP 的`parse_url`函数来获取域名，将其分解为数组，并仅使用第一个元素。然后，我们可以使用子域查询数据库，并为用户创建定制体验。

## 还有更多...

这个配方允许一个单一的路由来处理子域，但如果我们想要使用更多带有子域的路由，我们可以使用类似以下的路由组：

```php
Route::group(array('domain' => '{subdomain}.myapp.com'), function()
{
    Route::get('/', function($subdomain)
    {
        $name = DB::table('name')->where('name', $subdomain)->get();
     dd($name);

    });
});
```
