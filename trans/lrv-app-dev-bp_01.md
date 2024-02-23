# 第一章。构建 URL 缩短网站

在整本书中，我们将使用 Laravel PHP 框架来构建不同类型的 Web 项目。

在本章中，我们将看到如何使用 Laravel 框架的基础知识构建 URL 缩短网站。涵盖的主题包括：

+   创建数据库并迁移我们的 URL 缩短器表

+   创建我们的表单

+   创建我们的链接模型

+   将数据保存到数据库

+   从数据库获取单个 URL 并重定向

# 创建数据库并迁移我们的 URL 缩短器表

迁移就像是应用程序数据库的版本控制。它们允许团队（或您自己）修改数据库模式，并提供有关当前模式状态的最新信息。要创建和迁移 URL 缩短器的数据库，请执行以下步骤：

1.  首先，我们必须创建一个数据库，并定义到 Laravel 的连接信息。为此，我们打开 `app/config` 下的 `database.php` 文件，然后填写所需的凭据。Laravel 默认支持 MySQL、SQLite、PostgreSQL 和 SQLSRV（Microsoft SQL Server）。在本教程中，我们将使用 MySQL。

1.  我们将不得不创建一个 MySQL 数据库。为此，打开您的 MySQL 控制台（或 phpMyAdmin），并写下以下查询：

```php
    **CREATE DATABASE urls**

    ```

1.  上一个命令将为我们生成一个名为 `urls` 的新的 MySQL 数据库。成功生成数据库后，我们将定义数据库凭据。要做到这一点，打开 `app/config` 下的 `database.php` 文件。在该文件中，您将看到返回多个包含数据库定义的数组。

1.  `default` 键定义要使用的数据库驱动程序，每个数据库驱动程序键都保存各自的凭据。我们只需要填写我们将要使用的凭据。在我们的情况下，我确保默认键的值是 `mysql`。要设置连接凭据，我们将填写 `mysql` 键的值，其中包括我们的数据库名称、用户名和密码。在我们的情况下，由于我们有一个名为 `urls` 的 `database`，用户名为 `root`，没有密码，因此 `database.php` 文件中的 `mysql` 连接设置如下：

```php
    'mysql' => array(
      'driver' => 'mysql',
      'host' => 'localhost',
      'database' => 'database',
      'username' => 'root',
      'password' => '',
      'charset' => 'utf8',
      'collation' => 'utf8_unicode_ci',
      'prefix' => '',
    ),
    ```

### 提示

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便直接通过电子邮件接收文件。

1.  现在，我们将使用 **Artisan CLI** 来创建迁移。Artisan 是专为 Laravel 制作的命令行界面。它提供了许多有用的命令来帮助我们开发。我们将使用以下 `migrate:make` 命令在 Artisan 上创建一个迁移：

```php
    **php artisan migrate:make create_links_table --table=links --create**

    ```

该命令有两部分：

+   第一个是 `migrate:make create_links_table`。命令的这一部分创建一个迁移文件，文件名类似于 `2013_05_01_194506_create_links_table.php`。我们将进一步检查该文件。

+   命令的第二部分是 `--table=links --create`。

+   `--table=links` 选项指向数据库名称。

+   `--create` 选项用于在我们给定 `--table=links` 选项的数据库服务器上创建表。

1.  如您所见，与 Laravel 3 不同，当您运行上一个命令时，它将同时创建迁移表和我们的迁移。您可以在 `app/database/migrations` 下访问迁移文件，其中包含以下代码：

```php
    <?php
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;
    class CreateLinksTable extends Migration {
      /**
      * Run the migrations.
      *
      * @return void
      */
      public function up()
      {
        Schema::create('links', function(Blueprint $table)
        {
          $table->increments('id');
        });
      }
      /**
      * Reverse the migrations.
      *
      * @return void
      */
      public function down()
      {
        Schema::drop('links');
      }
    }
    ```

1.  让我们检查示例迁移文件。有两个公共函数声明为 `up()` 和 `down()`。当您执行以下 `migrate` 命令时，将执行 `up()` 函数的内容：

```php
    **php artsian migrate**

    ```

此命令将执行所有迁移并在我们的情况下创建 `links` 表。

### 注意

如果在运行迁移文件时收到 `class not found` 错误，请尝试运行 `composer update` 命令。

1.  我们还可以回滚到上一个迁移，就像它从未执行过一样。我们可以使用以下命令完成：

```php
    **php artisan migrate:rollback**

    ```

1.  在某些情况下，我们可能还想回滚我们创建的所有迁移。这可以通过以下命令完成：

```php
    **php artisan migrate:reset**

    ```

1.  在开发阶段，我们可能会忘记添加/删除一些字段，甚至忘记创建一些表，我们可能希望回滚所有内容并重新迁移它们。这可以使用以下命令完成：

```php
    **php artisan migrate:refresh**

    ```

1.  现在，让我们添加我们的字段。我们创建了两个额外的字段，称为`url`和`hash`。`url`字段将保存实际的 URL，而`hash`字段中的 URL 将被重定向到`hash`字段中的 URL 的缩短版本。迁移文件的最终内容如下所示：

```php
    <?php
    use Illuminate\Database\Migrations\Migration;
    class CreateLinksTable extends Migration {
      /**
      * Run the migrations.
      *
      * @return void
      */
      public function up()
      {
        Schema::create('links', function(Blueprint $table)
        {
          $table->increments('id');
          $table->text('url');
          $table->string('hash',400);
        });
      }
      /**
      * Reverse the migrations.
      *
      * @return void
      */
      public function down()
      {
        Schema::drop('links');
      }
    }
    ```

# 创建我们的表单

现在让我们制作我们的第一个表单视图。

1.  将以下代码保存为`form.blade.php`，放在`app/views`下。文件的扩展名是`blade.php`，因为我们将受益于 Laravel 4 内置的模板引擎**Blade**。在表单中可能有一些您尚不理解的方法，但不要担心。我们将在本章中涵盖有关此表单的所有内容。

```php
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <title>URL Shortener</title>
        <link rel="stylesheet" href="/assets/css/styles.css" />
      </head>
      <body>
        <div id="container">
          <h2>Uber-Shortener</h2>
          {{Form::open(array('url'=>'/','method'=>'post'))}}

          {{Form::text('link',Input::old('link'),array('placeholder'=>'Insert your URL here and press enter!'))}}
          {{Form::close()}}
        </div>
      </body>
    </html>
    ```

1.  现在将以下代码保存为`styles.css`，放在`public/assets/css`下：

```php
    div#container{padding-top:100px;
      text-align:center;
      width:75%;
      margin:auto;
      border-radius:4px}
    div#container h2{font-family:Arial,sans-serif;
      font-size:28px;
      color:#555}
    div#container h3{font-family:Arial,sans-serif;
      font-size:28px}
    div#container h3.error{color:#a00}
    div#container h3.success{color:#0a0}
    div#container input{display:block;
      width:90%;
      float:left;
      font-size:24px;
      border-radius:5px}
    div#error,div#success{border-radius:3px;
      display:block;
      width:90%;
      padding:10px}
    div#error{background:#ff8080;
      border:1px solid red}
    div#success{background:#80ff80;
      border:1px solid #0f0}
    ```

这段代码将生成一个看起来像以下截图的表单：

![创建我们的表单](img/2111OS_01_01.jpg)

正如您所看到的，我们使用了一个 CSS 文件来整理表单，但是表单的实际部分位于`View`文件的底部，位于 ID 为 container 的`div`内部。

1.  我们使用了 Laravel 内置的`Form`类来生成一个表单，并使用了`Input`库的`old()`方法。现在让我们来看看代码：

+   `Form::open()`: 它创建一个`<form>`开放标签。在第一个提供的参数中，您可以定义表单的发送方式以及将要发送到哪里。它可以是控制器的操作，直接 URL 或命名路由。

+   `Form::text()`: 它创建一个类型为文本的`<input>`标签。第一个参数是输入的名称，第二个参数是输入的值，在第三个参数给定的数组中，您可以定义`<input>`标签的属性和其他属性。

+   `Input::old()`: 它将返回表单中的旧输入，在表单返回输入后。第一个参数是提交的旧输入的名称。在我们的情况下，如果表单在提交后返回（例如，如果表单验证失败），文本字段将填充我们的旧输入，我们可以在以后的请求中重用它。

+   `Form::close()`: 它关闭`<form>`标签。

# 创建我们的 Link 模型

为了从 Laravel 的 ORM 类`Eloquent`中受益，我们需要定义一个模型。将以下代码保存为`Link.php`，放在`app/models`下：

```php
<?php
class Link extends Eloquent {
  protected $table = 'links';
  protected $fillable = array('url','hash');
  public $timestamps = false;
}
```

Eloquent 模型非常容易理解。

+   变量`$table`用于定义模型的表名，但这并不是强制性的。即使我们不定义这个变量，它也会将模型名称的复数形式作为数据库表名。例如，如果模型名称是 post，它将默认查找 post 的表。这样，您可以为表使用任何模型名称。

+   受保护的`$fillable`变量定义了可以（批量）创建和更新的列。默认情况下，Laravel 4 会阻止使用`Eloquent`批量赋值所有列的值。例如，如果您有一个`users`表，并且您是唯一的用户/管理员，批量赋值将保护您的数据库免受其他用户的添加。

+   `$timestamps`变量检查模型是否应该默认尝试设置时间戳`created_at`和`updated_at`，分别在创建和更新查询时。由于我们不需要这些功能，我们将通过将值设置为`false`来禁用它。

我们现在需要定义这个视图，以显示我们是否可以导航到我们的虚拟主机的索引页面。您可以从`routes.php`中定义的控制器，或直接从`routes.php`中定义。由于我们的应用程序很小，直接从`routes.php`中定义它们应该就足够了。要定义这个，打开`app`文件夹下的`routes.php`文件，并添加以下代码：

```php
Route::get('/', function()	
{
  return View::make('form');
});
```

### 注意

如果您已经有一个以`Route::get('/', function()`开头的部分，您应该用先前的代码替换该部分。

Laravel 可以监听`get`、`post`、`put`和`delete`请求。由于我们的操作是一个`get`操作（因为我们将通过浏览器导航而不是发布），所以我们的路由类型将是`get`，因为我们想在根页面上显示视图，所以`Route::get()`方法的第一个参数将是`/`，我们将包装一个闭包函数作为第二个参数来定义我们想要做的事情。在我们的情况下，我们将返回`app/views`下放置的`form.blade.php`，所以我们只需输入`return View::make('form')`。这个方法从`views`文件夹返回`form.blade.php`视图。

### 注意

如果视图在子目录中，它将被称为`subfolder.form`。

# 将数据保存到数据库

现在我们需要编写一个路由来监听我们的`post`请求。为此，我们打开`app`文件夹下的`routes.php`文件，并添加以下代码：

```php
Route::post('/',function(){
  //We first define the Form validation rule(s)
  $rules = array(
    'link' => 'required|url'
  );
  //Then we run the form validation
  $validation = Validator::make(Input::all(),$rules);
  //If validation fails, we return to the main page with an error info
  if($validation->fails()) {
    return Redirect::to('/')
    ->withInput()
    ->withErrors($validation);
  } else {
    //Now let's check if we already have the link in our database. If so, we get the first result
    $link = Link::where('url','=',Input::get('link'))
    ->first();
    //If we have the URL saved in our database already, we provide that information back to view.
    if($link) {
      return Redirect::to('/')
      ->withInput()
      ->with('link',$link->hash);
      //Else we create a new unique URL
    } else {
      //First we create a new unique Hash
      do {
        $newHash = Str::random(6);
      } while(Link::where('hash','=',$newHash)->count() > 0);

      //Now we create a new database record
      Link::create(array('url' => Input::get('link'),'hash' => $newHash
      ));

      //And then we return the new shortened URL info to our action
      return Redirect::to('/')
      ->withInput()
      ->with('link',$newHash);
    }
  }
});
```

## 验证用户的输入

使用我们现在编写的`post`动作函数，我们将使用 Laravel 内置的`Validation`类验证用户的输入。这个类帮助我们防止无效的输入进入我们的数据库。

首先，我们定义一个`$rules`数组来设置每个字段的规则。在我们的情况下，我们希望链接具有有效的 URL 格式。然后我们可以使用`Validator::make()`方法运行表单验证，并将其赋值给`$validation`变量。让我们了解`Validator::make()`方法的参数：

+   `Validator::make()`方法的第一个参数接受一个输入和要验证的值的数组。在我们的情况下，整个表单只有一个名为 link 的字段，所以我们使用了`Input::all()`方法，该方法返回表单中的所有输入。

+   第二个参数接受要检查的验证规则。存储的`$validation`变量为我们提供了一些信息。例如，我们可以检查验证是否失败或通过（使用`$validation->fails()`和`$validation->passes()`）。这两种方法返回布尔结果，因此我们可以轻松地检查验证是否通过或失败。此外，`$validation`变量包含一个`messages()`方法，其中包含验证失败的信息。我们可以使用`$validation->messages()`捕获它们。

如果表单验证失败，我们将用户重定向回我们的索引页面（`return Redirect::to('/')`），该页面包含 URL 缩短器表单，并将一些闪存数据返回给表单。在 Laravel 中，我们通过向重定向的页面添加`withVariableName`对象来实现这一点。在这里使用`with`是强制的，这将告诉 Laravel 我们正在返回一些额外的东西。我们可以在重定向和制作视图时都这样做。如果我们正在制作视图并向最终用户显示一些内容，那么`withVariableName`将是变量，我们可以直接使用`$VariableName`调用它们，但如果我们正在重定向到一个带有`withVariableName`对象的页面，`VariableName`将是一个闪存会话数据，我们可以使用`Session`类（`Session::get('VariableName')`）来调用它。

在我们的示例中，为了返回错误，我们使用了一个特殊的`withErrors($validation)`方法，而不是返回`$validation->messages()`。我们也可以使用那个返回，但是`$errors`变量总是在视图上定义的，所以我们可以直接使用我们的`$validation`变量作为参数与`withErrors()`一起使用。`withInput()`方法也是一个特殊的方法，它将结果返回到表单。

```php
//If validation fails, we return to the main page with an error info
if($validation->fails()) {
  return Redirect::to('/')
  ->withInput()
  ->withErrors($validation);
}
```

如果用户在表单中忘记了一个字段，并且验证失败并显示带有错误消息的表单，使用`withInput()`方法，表单可以再次填充旧的输入。为了在 Laravel 中显示这些旧的输入，我们使用`Input`类的`old()`方法。例如，`Input::old('link')`将返回表单字段`link`的旧输入。

## 将消息返回给视图

为了将错误消息返回到表单中，我们可以将以下 HTML 代码添加到`form.blade.php`中：

```php
@if(Session::has('errors'))
<h3 class="error">{{$errors->first('link')}}</h3>
@endif
```

正如您可能已经猜到的那样，`Session::has('variableName')`返回一个布尔值，用于检查会话中是否有变量名。然后，使用 Laravel 的`Validator`类的`first('formFieldName')`方法，我们返回表单字段的第一个错误消息。在我们的情况下，我们正在显示`link`表单字段的第一个错误消息。

# 深入控制器和处理表单

在我们的示例中，验证检查部分的`else`部分在表单验证成功完成时执行，包含了链接的进一步处理。在这一部分，我们将执行以下步骤：

1.  检查链接是否已经在我们的数据库中。

1.  如果链接已经在我们的数据库中，返回缩短后的链接。

1.  如果链接不在我们的数据库中，为链接创建一个新的随机字符串（将在我们的 URL 中）。

1.  在我们的数据库中使用提供的数值创建一个新的记录。

1.  将缩短后的链接返回给用户。

现在，让我们深入了解代码。

1.  以下是我们代码的第一部分：

```php
    // Now let's check if we already have the link in our database. If so, we get the first result
    $link = Link::where('url','=',Input::get('link'))
    ->first();
    ```

首先，我们使用**Fluent Query Builder**的`where()`方法检查 URL 是否已经存在于我们的数据库中，并通过`first()`方法获取第一个结果，并将其赋给`$link`变量。您可以轻松地使用 Fluent 查询方法和 Eloquent ORM。如果这让您感到困惑，不用担心，我们将在后面的章节中进一步介绍。

1.  这是我们控制器方法代码的下一部分：

```php
    //If we have the URL saved in our database already, we provide that information back to view.
    if($link) {
      return Redirect::to('/')
      ->withInput()
      ->with('link',$link->hash);
    ```

如果我们在数据库中保存了 URL，`$link`变量将保存从数据库中获取的链接信息的对象。因此，通过简单的`if()`子句，我们可以检查是否有结果。如果有结果返回，我们可以使用`$link->columnname`来访问它。

在我们的情况下，如果查询有结果，我们将输入和链接重定向回表单。正如我们在这里使用的，`with()`方法也可以用两个参数而不是使用驼峰命名法——`withName('value')`与`with('name','value')`完全相同。因此，我们可以使用闪存数据名为链接`with('link',$link->hash)`来返回哈希码。为了显示这一点，我们可以将以下代码添加到我们的表单中：

```php
    @if(Session::has('link'))
    <h3 class="success">
      {{Html::link(Session::get('link'),'Click here for your shortened URL')}}</h3>
    @endif
    ```

`Html`类帮助我们轻松编写 HTML 代码。`link()`方法需要以下两个参数：

+   第一个参数是`link`。如果我们直接提供一个字符串（在我们的例子中是哈希字符串），该类将自动识别它并从我们的网站创建内部 URL。

+   第二个参数是包含链接的字符串。

可选的第三个参数必须是一个数组，包含属性（例如 class、ID 和 target）作为二维数组。

1.  以下是我们代码的下一部分：

```php
    //Else we create a new unique URL
    } else {
      //First we create a new unique Hash
      do {
        $newHash = Str::random(6);
      } while(Link::where('hash','=',$newHash)->count() > 0);
    ```

如果没有结果（变量的 else 子句），我们将使用`Str`类的`random()`方法创建一个六个字符长的字母数字随机字符串，并使用 PHP 自己的 do-while 语句每次检查它是否是唯一的字符串。对于真实世界的应用，您可以使用另一种方法来缩短，例如将 ID 列中的条目转换为 base_62 并将其用作哈希值。这样，URL 将更清晰，这总是一个更好的做法。

1.  这是我们代码的下一部分：

```php
    //Now we create a new database record
    Link::create(array(
      'url' => Input::get('link'),
      'hash' => $newHash
    ));
    ```

一旦我们有了唯一的哈希，我们可以使用 Laravel 的 Eloquent ORM 的`create()`方法将链接和哈希值添加到数据库中。唯一的参数应该是一个二维数组，其中数组的键保存数据库列名，数组的值保存要插入为新行的值。

在我们的情况下，`url`列必须具有来自表单的`link`字段的值。我们可以使用 Laravel 的`Input`类的`get()`方法捕获来自`post`请求的这些值。在我们的情况下，我们可以使用`Input::get('link')`捕获来自`post`请求的`link`表单字段的值（我们可以使用`$_POST['link']`的混乱代码捕获），并像之前一样将哈希值返回给视图。

1.  这是我们代码的最后部分：

```php
    //And then we return the new shortened URL info to our action return Redirect::to('/')
    ->withInput()
    ->with('link',$newHash);
    ```

现在，在输出中，我们被重定向到`oursite.dev/hashcode`。变量`$newHash`中存储了一个链接；我们需要捕获这个哈希码并查询我们的数据库，如果有记录，我们需要重定向到实际的 URL。

# 从数据库中获取单个 URL 并重定向

现在，在我们第一章的最后部分，我们需要从生成的 URL 中获取`hash`部分，如果有值，我们需要将其重定向到存储在我们数据库中的 URL。为此，请在`app`文件夹下的`routes.php`文件末尾添加以下代码：

```php
Route::get('{hash}',function($hash) {
  //First we check if the hash is from a URL from our database
  $link = Link::where('hash','=',$hash)
  ->first();
  //If found, we redirect to the URL
  if($link) {
    return Redirect::to($link->url);
    //If not found, we redirect to index page with error message
  } else {
    return Redirect::to('/')
    ->with('message','Invalid Link');
  }
})->where('hash', '[0-9a-zA-Z]{6}');
```

在前面的代码中，与其他路由定义不同，我们在名称`hash`周围添加了花括号，告诉 Laravel 它是一个参数；并且使用`where()`方法定义了名称参数的方式。第一个参数是变量的名称（在我们的情况下是`hash`），第二个参数是一个正则表达式，用于过滤参数。在我们的情况下，正则表达式过滤了一个精确的六个字符长的字母数字字符串。这样，我们可以过滤我们的 URL 并从一开始就保护它们，而且我们不必检查`url`参数是否有我们不想要的内容（例如，在 ID 列中输入字母而不是数字）。要从数据库中获取单个 URL 并重定向，我们执行以下步骤：

1.  在`Route`类中，我们首先进行搜索查询，就像在前面的部分中所做的那样，然后检查我们的数据库中是否有一个具有给定哈希的链接，并将其设置为名为`$link`的变量。

```php
    //First we check if the hash is from an URL from our database
    $link = Link::where('hash','=',$hash)
    ->first();
    ```

1.  如果有结果，我们将页面重定向到我们数据库的`url`列，该列包含用户应重定向到的链接。

```php
    //If found, we redirect to the link
    if($link) {
      return Redirect::to($link->url);
    }
    ```

1.  如果没有结果，我们将使用`$message`变量将用户重定向回我们的索引页面，该变量保存了`Invalid Link`的值。

```php
    //If not found, we redirect to index page with error message
    } else {
      return Redirect::to('/')
      ->with('message','Invalid Link');
    }
    ```

要在表单中显示`Invalid Link`消息，请在`app/views`下的`form.blade.php`文件中添加以下代码。

```php
    @if(Session::has('message'))
    <h3 class="error">{{Session::get('message')}}</h3>
    @endif
    ```

# 总结

在本章中，我们通过制作一个简单的 URL 缩短网站，介绍了 Laravel 路由、模型、artisan 命令和数据库驱动的基本用法。一旦您完成了本章，您就可以使用迁移创建数据库表，使用 Laravel 表单构建器类编写简单的表单，使用`Validation`类验证这些表单，并使用 Fluent 查询构建器或 Eloquent ORM 处理这些表单并将新数据插入表中。在下一章中，我们将介绍这些强大功能的高级用法。
