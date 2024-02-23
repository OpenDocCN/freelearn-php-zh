# 附录 B. MVC 和框架

我们在不同章节中提到了一些框架的名称，但没有讨论它们。在今天的世界中，我们不会重新发明轮子；我们会在已经构建、测试和广泛使用的工具基础上进行构建。因此，作为最佳实践，如果没有可用的工具来满足需求，我们可以使用最适合需求的框架来构建它。

我们将涵盖以下主题：

+   MVC 设计模式

+   Laravel

+   Lumen

+   Apigility

# MVC 设计模式

**模型视图控制器**（**MVC**）是一种广泛应用于不同编程语言中的设计模式。大多数 PHP 框架使用这种设计模式。这种模式将应用程序分为三层：模型、视图和控制器。每个层都有不同的任务，并且它们都相互连接。MVC 有不同的视觉表示，但是可以在以下图表中看到一个整体和简单的表示：

![MVC 设计模式](img/B05225_AppendixB_01.jpg)

现在，让我们讨论 MVC 设计模式的每个部分。

## 模型

模型层是应用程序的支柱，处理数据逻辑。大多数情况下，认为模型负责对数据库进行 CRUD 操作，这可能是真实的，也可能不是。正如我们之前提到的，模型负责数据逻辑，这意味着数据验证操作也可以在这里执行。简单地说，模型为数据提供了一个抽象。其余的应用层不知道或不关心数据来自何处，或者如何对数据执行操作。这是模型的责任，负责处理所有数据逻辑。

在当今复杂的框架结构中，整体 MVC 结构已经改变，不仅模型处理数据操作，而且每个其他应用逻辑也由模型处理。遵循的方法是“胖模型，瘦控制器”，这意味着将所有应用逻辑放在模型中，使控制器尽可能清晰。

## 视图

视图是最终用户可见的内容。与此用户和公众相关的所有数据都显示在视图中，因此视图可以被称为模型的视觉表示。视图需要数据来显示。它向控制器请求一些特定的数据或操作。视图不知道或不想知道控制器从何处获取这些数据；它只是要求控制器获取它。控制器知道要向谁请求这些特定的数据，并与特定的模型进行通信。这意味着视图没有直接连接到模型。然而，在早期的图表中，我们直接将模型与视图连接起来。这是因为在现今的先进系统中，视图可以直接从模型获取数据。例如，Magento 控制器无法将数据发送回视图。对于数据（即直接从数据库获取数据）和/或与模型通信，视图与块和辅助类进行通信。在现代实践中，视图可以直接连接到模型。

## 控制器

控制器响应用户在视图中执行的操作，并响应视图。例如，用户填写表单并提交。在这里，控制器介入并开始对表单的提交采取行动。现在，控制器将首先检查用户是否被允许发出此请求。然后，控制器将采取适当的行动，例如与模型或任何其他操作进行通信。简单地说，控制器是视图和模型之间的中间人。正如我们之前在模型部分提到的，控制器应该是精简的。因此，大多数情况下，控制器仅用于处理请求并与模型和视图进行通信。所有类型的数据操作都在模型中执行。

MVC 设计模式的唯一工作是分离应用程序中不同部分的责任。因此，模型用于管理应用程序数据。控制器用于对用户输入进行操作，视图负责数据的视觉表示。正如我们之前提到的，MVC 分离了每个部分的责任，因此无论是从控制器还是视图访问模型都无关紧要；唯一重要的是视图和控制器不应该用于对数据执行操作，因为这是模型的责任，控制器也不应该用于查看任何类型的数据，因为这是视图的责任。

# Laravel

Laravel 是最流行的 PHP 框架之一，根据 Laravel 官方网站的说法，它是一个面向 Web 工匠的框架。Laravel 美观、强大，并且拥有大量功能，可以让开发人员编写高效和高质量的代码。Laravel 官方文档写得很好，非常容易理解。所以，让我们来玩一下 Laravel 吧。

## 安装

安装非常简单。让我们使用 Composer 来安装 Laravel。我们在附录 A 中讨论了 Composer。在终端中输入以下命令来安装并创建一个 Laravel 项目：

```php
**composer create-project --prefer-dist laravel/laravel packt**

```

如果系统上没有全局安装 Composer，将`composer.phar`放在应该安装 Laravel 的目录中，并在该目录的根目录下在终端中输入以下命令：

```php
**php composer.phar create-project --prefer-dist laravel/laravel packt**

```

现在，Laravel 将被下载，并将创建一个名为`packt`的新项目。此外，Composer 将下载并安装项目的所有依赖项。

打开浏览器，转到项目的 URL，我们将受到一个简单的页面，上面写着**Laravel 5**。

### 注意

截至撰写本书时，Laravel 5.2.29 是最新版本。但是，如果使用 Composer，则每次使用`composer update`命令时，Laravel 和所有其他组件都将自动更新。

## 功能

Laravel 提供了大量的功能，我们在这里只讨论一些。

### 路由

Laravel 提供了强大的路由。路由可以分组，并且可以为路由组定义前缀、命名空间和中间件。此外，Laravel 支持所有 HTTP 方法，包括`POST`、`GET`、`DELETE`、`PUT`、`OPTIONS`和`PATCH`。所有路由都在应用程序的`app`文件夹中的`routes.php`文件中定义。看一下以下示例：

```php
Route::group(['prefix' => 'customer', 'namespace' => 'Customer', 'middleware' => 'web'], function() {
    Route::get('/', 'CustomerController@index');
    Route::post('save', 'CustomerController@save');
    Route::delete('delete/{id}', 'CustomerController@delete');
});
```

在上面的代码片段中，我们创建了一个新的路由组。只有当 URL 有一个前缀为 customer 时才会使用这个组。例如，如果 URL 类似于`domain.com/customer`，则将使用此组。我们还使用了一个 customer 命名空间。命名空间允许我们使用标准的 PHP 命名空间并将文件分割成子文件夹。在上面的示例中，所有 customer 控制器可以放在`Controllers`目录中的 Customer 子文件夹中，并且控制器将如下创建：

```php
namespace App\Http\Controllers\Customer

use App\Http\{
Controllers\Controller,
Requests,
};
use Illuminate\Http\Request;

Class CustomerController extends Controller
{
  …
  …
}
```

因此，对路由组进行命名空间使我们能够将控制器文件放在易于管理的子文件夹中。此外，我们使用了 web 中间件。中间件提供了一种在进入应用程序之前过滤请求的方法，这使我们可以使用它来检查用户是否已登录，CSRF 保护，或者是否有任何其他需要在请求发送到应用程序之前执行的中间件操作。Laravel 带有一些中间件，包括`web`、`api`、`auth`等。

如果路由定义为`GET`，则不能向该路由发送`POST`请求。这非常方便，使我们不必担心请求方法过滤。但是，HTML 表单不支持`DELETE`、`PATCH`和`PUT`等 HTTP 方法。为此，Laravel 提供了方法欺骗，其中使用带有`name _method`和 HTTP 方法值的隐藏表单字段，以使此请求成为可能。例如，在我们的路由组中，为了使删除路由的请求成为可能，我们需要一个类似于以下的表单：

```php
<form action="/customer/delete" method="post">
  {{ method_field('DELETE') }}
  {{ csrf_field() }}
</form>
```

当提交上述表单时，它将起作用，并且将使用删除路由。此外，我们创建了一个 CSRF 隐藏字段，用于 CSRF 保护。

### 注意

Laravel 路由非常有趣，是一个大的话题。更深入的细节可以在[`laravel.com/docs/5.2/routing`](https://laravel.com/docs/5.2/routing)找到。

## Eloquent ORM

Eloquent ORM 提供了与数据库交互的活动记录。要使用 Eloquent ORM，我们只需从 Eloquent 模型扩展我们的模型。让我们看一个简单的用户模型，如下所示：

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class user extends Model
{
  //protected $table = 'customer';
  //protected $primaryKey = 'id_customer';
  …
  …
}
```

就是这样；我们现在有一个可以处理所有 CRUD 操作的模型。请注意，我们已经注释了`$table 属性`，并对`$primaryKey`做了相同的操作。这是因为 Laravel 使用类的复数名称来查找表，除非表是使用受保护的`$table 属性`定义的。在我们的情况下，Laravel 将查找表名 users 并使用它。但是，如果我们想使用名为`customers`的表，我们只需取消注释该行，如下所示：

```php
protected $table = 'customers';
```

同样，Laravel 认为表将具有列名`id`的主键。但是，如果需要另一列，我们可以覆盖默认的主键，如下所示：

```php
protected $primaryKey = 'id_customer';
```

优雅的模型也使时间戳变得容易。默认情况下，如果表具有`created_at`和`updated_at`字段，则这两个日期将自动生成并保存。如果不需要时间戳，可以禁用如下：

```php
protected $timestamps = false;
```

将数据保存到表中很容易。表列被用作模型的属性，因此，如果我们的`customer`表具有诸如`name`、`email`、`phone`等列，我们可以在路由部分提到的`customer`控制器中设置它们，如下所示：

```php
namespace App\Http\Controllers\Customer

use App\Http\{
Controllers\Controller,
Requests,
};
use Illuminate\Http\Request;
use App\Customer

Class CustomerController extends Controller
{
  public function save(Request $request)
  {
    $customer = new Customer();
    $customer->name = $request->name;
    $customer->email = $request->email;
    $customer->phone = $request->phone;

    $customer->save();

  }
}
```

在上面的示例中，我们向我们的控制器添加了`save`操作。现在，如果提交了`POST`或`GET`请求以及表单数据，Laravel 将所有表单提交的数据分配给一个 Request 对象，作为与表单字段相同名称的属性。然后，使用此请求对象，我们可以访问通过`POST`或`GET`提交的所有数据。在将所有数据分配给模型属性（与表列的名称相同）之后，我们只需调用 save 方法。现在，我们的模型没有任何保存方法，但是其父类，即 Eloquent 模型，已经定义了此方法。但是，如果需要此方法中的其他功能，我们可以在我们的`model`类中覆盖此`save`方法。

从 Eloquent 模型中获取数据也很容易。让我们尝试一个例子。向`customer`控制器添加一个新操作，如下所示：

```php
public function index()
{
  $customers = Customer::all();
}
```

我们在模型中使用了`all()`静态方法，它基本上是在 Eloquent 模型中定义的，反过来获取了我们的`customers`表中的所有数据。现在，如果我们想要通过主键获取单个客户，我们可以使用`find($id)`方法，如下所示：

```php
$customer = Customer::find(3);
```

这将获取 ID 为`3`的客户。

更新很简单，使用相同的`save()`方法，如下所示：

```php
$customer = Customer::find(3);
$customer->name = 'Altaf Hussain';

$customer->save();
```

这将更新 ID 为`3`的客户。首先，我们加载了`customer`，然后我们为其属性分配了新数据，然后调用了相同的`save()`方法。删除模型简单易行，可以按如下方式完成：

```php
$customer = Customer::find(3);
$customer->delete();
```

我们首先加载了 ID 为`3`的客户，然后调用了`delete`方法，这将删除 ID 为`3`的客户。

### 注意

Laravel 的 Eloquent 模型非常强大，并提供了许多功能。这些在文档中有很好的解释，网址为[`laravel.com/docs/5.2/eloquent`](https://laravel.com/docs/5.2/eloquent)。Laravel 数据库部分也值得阅读，网址为[`laravel.com/docs/5.2/database`](https://laravel.com/docs/5.2/database)。

## Artisan CLI

Artisan 是 Laravel 提供的命令行界面，它有一些很好的命令可以用于更快的操作。它有很多命令，可以使用以下命令查看完整列表：

```php
**php artisan list**

```

这将列出所有可用的选项和命令。

### 注意

`php artisan`命令应该在`artisan`文件所在的同一目录中运行。它被放置在项目的根目录下。

一些基本命令如下：

+   `make:controller`: 这个命令在`Controllers`文件夹中创建一个新的控制器。可以如下使用：

```php
**php artisan make:controller MyController**

```

如果需要一个有命名空间的控制器，就像之前的`Customer`命名空间一样，可以如下操作：

```php
**php artisan make:controller Customer/CustomerController**

```

这个命令将在`Customer`文件夹中创建`CustomerController`。如果`Customer`文件夹不存在，它也将创建该文件夹。

+   `make:model`: 这在`app`文件夹中创建一个新的模型。语法与`make:controller`命令相同，如下：

```php
**php artisan make:model Customer**

```

对于有命名空间的模型，可以如下使用：

```php
**php artisan make:model Customer/Customer**

```

这将在`Customer`文件夹中创建`Customer`模型，并为其使用`Customer`命名空间。

+   `make:event`: 这在`Events`文件夹中创建一个新的`event`类。可以如下使用：

```php
**php artisan make:event MyEvent**

```

+   `make:listener`: 这个命令为事件创建一个新的监听器。可以如下使用：

```php
**php artisan make:listener MyListener --event MyEvent**

```

上述命令将为我们的`MyEvent`事件创建一个新的监听器。我们必须始终使用`--event`选项提及我们需要创建监听器的事件。

+   `make:migration`: 这个命令在 database/migrations 文件夹中创建一个新的迁移。

+   `php artisan migrate`: 这将运行所有尚未执行的可用迁移。

+   `php artisan optimize`: 这个命令优化框架以获得更好的性能。

+   `php artisan down`: 这将把应用程序置于维护模式。

+   `php artisan up`: 这个命令将应用程序从维护模式中恢复。

+   `php artisan cache:clear`: 这个命令清除应用程序缓存。

+   `php artisan db:seed`: 这个命令用记录填充数据库。

+   `php artisan view:clear`: 这将清除所有已编译的视图文件。

### 注意

有关 Artisan 控制台或 Artisan CLI 的更多详细信息可以在文档中找到，网址为[`laravel.com/docs/5.2/homestead`](https://laravel.com/docs/5.2/homestead)。

## 迁移

迁移是 Laravel 中的另一个强大功能。在迁移中，我们定义数据库模式——它是创建表、删除表或在表中添加/更新列。迁移在部署中非常方便，并且作为数据库的版本控制。让我们为我们的数据库中尚不存在的 customer 表创建一个迁移。要创建一个迁移，在终端中发出以下命令：

```php
**php artisan make:migration create_custmer_table**

```

在`database/migrations`文件夹中将创建一个新文件，文件名为当前日期和唯一 ID 前缀的`create_customer_table`。类被创建为`CreateCustomerTable`。这是一个如下的类：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateCustomerTable extends Migrations
{
  //Run the migrations

  public function up()
  {
    //schemas defined here
  }

  public function down()
  {
    //Reverse migrations
  }
}
```

类将有两个公共方法：`up()`和`down()`。`up()`方法应该包含表的所有新模式。`down()`方法负责撤销已执行的迁移。现在，让我们将`customers`表模式添加到`up()`方法中，如下：

```php
public function up()
{
  Schema::create('customers', function (Blueprint $table)
  {
    $table->increments('id', 11);
    $table->string('name', 250)
    $table->string('email', 50);
    $table->string('phone', 20);
    $table->timestamps();
  });
}
public function down()
{
  Schema::drop('customers');
}
```

在`up()`方法中，我们定义了模式和表名。表的列是单独定义的，包括列大小。`increments()`方法定义了自动增量列，在我们的例子中是`id`列。接下来，我们为`name`、`email`和`phone`创建了三个字符串列。然后，我们使用了`timestamps()`方法，它创建了`created_at`和`updated_at`时间戳列。在`down()`方法中，我们只是使用了`Schema`类的`drop()`方法来删除`customers`表。现在，我们需要使用以下命令运行我们的迁移：

```php
**php artisan migrate**

```

上述命令不仅会运行我们的迁移，还会运行尚未执行的所有迁移。当执行迁移时，Laravel 会将迁移名称存储在一个名为`migrations`的表中，从中决定要执行哪些迁移以及要跳过哪些迁移。

现在，如果我们需要回滚最近执行的迁移，我们可以使用以下命令：

```php
**php artisan migrate:rollback**

```

这将回滚到最后一批迁移。要回滚应用程序的所有迁移，我们可以使用 reset 命令，如下所示：

```php
**php artisan migrate:reset**

```

这将回滚完整的应用程序迁移。

迁移使部署变得容易，因为我们不需要每次在表或数据库中创建一些新的更改时上传数据库模式。我们只需创建迁移并上传所有文件，之后我们只需执行迁移命令，所有模式将被更新。

## Blade 模板

Laravel 自带了自己的模板语言 Blade。此外，Blade 模板文件支持普通的 PHP 代码。Blade 模板文件被编译为普通的 PHP 文件，并在更改之前被缓存。Blade 还支持布局。例如，以下是我们在 Blade 中的主页面布局，放在`resources/views/layout`文件夹中，名为`master.blade.php`。看一下以下代码：

```php
<!DOCTYPE html>
<html>
  <head>
    <title>@yield('title')</title>
  </head>
  <body>
    @section('sidebar')
      Our main sidebar
      @show

      <div class="contents">
        @yield('content')
      </div>
  </body>
</html>
```

在上面的例子中，我们有一个定义`content`部分的侧边栏。此外，我们有`@yield`，它显示部分的内容。现在，如果我们想要使用这个布局，我们需要在子模板文件中扩展它。让我们在`resources/views/`文件夹中创建`customers.blade.php`文件，并将以下代码放入其中：

```php
@extend('layouts.master')
  @section('title', 'All Customers')
  @section('sidebar')
  This will be our side bar contents
  @endsection
  @section('contents')
    These will be our main contents of the page
  @endsection
```

如前面的代码所示，我们扩展了`master`布局，然后在`master`布局的每个部分放置了内容。此外，还可以在另一个模板中包含不同的模板。例如，让我们在`resources/views/includes`文件夹中有两个文件，`sidebar.blade.php`和`menu.blade.php`。然后，我们可以在任何模板中包含这些文件，如下所示：

```php
@include(includes.menu)
@include(includes.sidebar)
```

我们使用`@include`来包含一个模板。点(`.`)表示文件夹分隔。我们可以轻松地从我们的控制器或路由器向 Blade 模板或视图发送数据。我们只需将数据作为数组传递给视图，如下所示：

```php
return view('customers', ['count => 5]);
```

现在，在我们的`customers`视图文件中可以访问`count`，如下所示：

```php
Total Number of Customers: {{ count }}
```

是的，Blade 使用双花括号来输出变量。对于控制结构和循环，让我们举一个例子。让我们向`customers`视图发送数据，如下所示：

```php
return view('customers', ['customers' => $allCustomers]);
```

现在，如果我们想要显示所有`customers`数据，我们的`customers`视图文件将类似于以下内容：

```php
…
…
@if (count($customers) > 0)
{{ count($customers) }} found. <br />
@foreach ($customers as $customer)
{{ $customer->name }} {{ $customer->email }} {{ $customer->phone }} <br>
@endforeach

@else
Now customers found.
@endif;
…
…
```

所有上述语法看起来很熟悉，因为它几乎与普通的 PHP 相同。但是，要显示一个变量，我们必须使用双花括号`{{}}`。

### 注意

可以在[`laravel.com/docs/5.2/blade`](https://laravel.com/docs/5.2/blade)找到一个易于阅读的 Blade 模板文档。

## 其他特性

在上一节中，我们只讨论了一些基本功能。Laravel 还有许多其他功能，例如身份验证和授权，提供了一种简单的方式来对用户进行身份验证和授权。此外，Laravel 提供了强大的缓存系统，支持基于文件的缓存、Memcached 和 Redis 缓存。Laravel 还为这些事件提供了事件和监听器，当我们想执行特定操作时以及特定事件发生时，这是非常方便的。Laravel 支持本地化，可以使用本地化内容和多种语言。Laravel 还支持任务调度和队列，我们可以在特定时间安排一些任务运行，并在轮到它们时排队运行一些任务。

# Lumen

Lumen 是由 Laravel 提供的微框架。Lumen 主要用于创建无状态 API，并具有 Laravel 的最小功能集。此外，Lumen 与 Laravel 兼容，这意味着如果我们只是将我们的 Lumen 应用程序复制到 Laravel 中，它将正常工作。安装很简单。只需使用以下 Composer 命令创建一个 Lumen 项目，它将下载包括 Lumen 在内的所有依赖项：

```php
**composer create-project --prefer-dist laravel/lumen api**

```

上述命令将下载 Lumen，然后创建我们的 API 应用程序。完成后，将`.env.example`重命名为`.env`。还要创建一个 32 个字符长的应用程序密钥，并将其放入`.env`文件中。现在，基本应用程序已准备好使用和创建 API。

### 注意

Lumen 与 Laravel 几乎相同，但默认情况下不包括一些 Laravel 功能。更多细节可以在[`lumen.laravel.com/docs/5.2`](https://lumen.laravel.com/docs/5.2)找到。

# Apigility

Apigility 是由 Zend 在 Zend Framework 2 中构建和开发的。Apigility 提供了一个易于使用的 GUI 来创建和管理 API。它非常易于使用，并能够创建复杂的 API。让我们从使用 Composer 安装 Apigility 开始。在终端中输入以下命令：

```php
**composer create-project -sdev zfcampus/zf-apigility-skeleton packt**

```

上述命令将下载 Apigility 及其依赖项，包括 Zend Framework 2，并将设置我们的名为`packt`的项目。现在，发出以下命令以启用开发模式，以便我们可以访问 GUI：

```php
**php public/index.php development enable**

```

现在，打开 URL [yourdomain.com/packt/public](http://yourdomain.com/packt/public)，我们将看到一个漂亮的 GUI，如下面的屏幕截图所示：

![Apigility](img/B05225_AppendixB_02.jpg)

现在，让我们创建我们的第一个 API。我们将称此 API 为“`books`”，它将返回一本书的列表。单击前面图片中显示的**New API**按钮，将显示一个弹出窗口。在文本框中输入`books`作为 API 名称，然后单击`Create`按钮；新 API 将被创建。创建 API 后，我们将看到以下屏幕：

![Apigility](img/B05225_AppendixB_03.jpg)

Apigility 提供了设置 API 的其他属性的简单方法，例如版本控制和身份验证。现在，通过单击左侧边栏中的**New Service**按钮来创建一个 RPC 服务。此外，我们可以在前面的屏幕截图中的**RPC**部分单击**Create a new one**链接。我们将看到以下屏幕：

![Apigility](img/B05225_AppendixB_04.jpg)

如前面的屏幕截图所示，我们在`books`API 中创建了一个名为`get`的 RPC 服务。输入的路由 URI 是`/books/get`，将用于调用此 RPC 服务。当我们单击`Create service`按钮时，将显示 API 创建成功的消息，并且还将显示以下屏幕：

![Apigility](img/B05225_AppendixB_05.jpg)

如前面的屏幕截图所示，此服务的允许 HTTP 方法仅为**GET**。让我们保持原样，但我们可以选择全部或任何一个。此外，我们希望将**内容协商选择器**保持为`Json`，并且我们的服务将以 JSON 格式接受/接收所有内容。此外，我们可以选择不同的媒体类型和内容类型。

接下来，我们应该为我们的服务添加一些将要使用的字段。点击**字段**选项卡，我们将看到**字段**屏幕。点击**新建字段**按钮，我们将看到以下弹出窗口：

![Apigility](img/B05225_AppendixB_06.jpg)

如前面的屏幕截图所示，我们可以为字段设置所有属性，如**名称**、**描述**、是否必填等，以及一些其他设置，包括验证失败时的错误消息。在创建了两个字段**title**和**author**之后，我们将看到类似以下的屏幕：

![Apigility](img/B05225_AppendixB_07.jpg)

如前面的屏幕所示，我们也可以为每个单独的字段添加验证器和过滤器。

### 注意

由于这只是 Apigility 的入门主题，我们将不会在本书中涵盖验证器、过滤器和其他一些主题。

下一个主题是文档。当我们点击**文档**选项卡时，我们将看到以下屏幕：

![Apigility](img/B05225_AppendixB_08.jpg)

在这里，我们将记录我们的服务，添加一些描述，还可以为文档目的生成响应主体。这非常重要，因为它将使其他人更好地理解我们的 API 和服务。

现在，我们需要从某个地方获取所有的书。可以是从数据库中获取，也可以是从另一个服务或其他来源获取。然而，现在，我们只是为了测试目的，将使用一组书的数组。如果我们点击**来源**选项卡，我们会发现我们服务的代码放在`module/books/src/books/V1/Rpc/Get/GetController.php`中。Apigility 为我们的 API`books`创建了一个模块，然后根据 API 的版本（默认为 V1），将所有源代码放在这个模块的不同文件夹中。我们可以为我们的 API 添加更多版本，如 V2 和 V3。现在，如果我们打开`GetController`文件，我们会发现一些代码和一个根据我们的路由 URI 命名为`getAction`的操作。代码如下，高亮显示的是我们添加的代码：

```php
namespace books\V1\Rpc\Get;

use Zend\Mvc\Controller\AbstractActionController;
**use ZF\ContentNegotiation\ViewModel;**

class GetController extends AbstractActionController
{
  public function getAction()
  {
    **$books = [ 'success' => [**
 **[**
 **'title' => 'PHP 7 High Performance',**
 **'author' => 'Altaf Hussain'**
 **],**
 **[**
 **'title' => 'Magento 2',**
 **'author' => 'Packt Publisher'**
 **],**
 **]**
 **];**

 **return new ViewModel($books);**
  }
}
```

在上面的代码中，我们使用了`ContentNegotiation\ViewModel`，它负责以我们在服务设置中选择的格式（在我们的情况下是 JSON）响应数据。然后，我们创建了一个简单的`$books`数组，其中包含我们为服务创建的字段名，并为它们分配了值。然后，我们使用`ViewModel`对象返回它们，该对象处理响应数据转换为 JSON。

现在，让我们测试我们的 API。由于我们的服务可以接受`GET`请求，我们只需在浏览器中输入带有`books/get` URI 的 URL，就会看到 JSON 响应。最好使用 RestClient 或 Google Chrome 的 Postman 等工具来检查 API，这些工具提供了一个易于使用的界面，可以向 API 发出不同类型的请求。我们使用 Postman 进行了测试，并得到了以下截图中显示的响应：

![Apigility](img/B05225_AppendixB_09.jpg)

还要注意，我们将我们的服务设置为仅接受`GET`请求。因此，如果我们发送的请求不是`GET`，我们将收到`HTTP 状态码 405 方法不允许`的错误。

Apigility 非常强大，提供了许多功能，如 RESTFul API、HTTP 身份验证、与易于创建的数据库连接器连接的数据库服务，以及服务的表格选择。在使用 Apigility 时，我们不需要担心 API、服务结构安全性和其他事情，因为 Apigility 会为我们处理这些。我们只需要专注于 API 和服务的业务逻辑。

### 注意

Apigility 无法在本附录中完全涵盖。Apigility 有很多功能，可以在一本完整的书中进行介绍。Apigility 的官方文档网址[`apigility.org/documentation`](https://apigility.org/documentation)是一个很好的起点，可以了解更多信息。

# 摘要

在本附录中，我们讨论了 MVC 设计模式的基础知识。我们还讨论了 Laravel 框架及其一些优秀特性。我们向你介绍了基于 Laravel 的微框架 Lumen。最后，我们对 Apigility 进行了简要介绍，并创建了一个测试 API 和 Web 服务。

在 IT 领域，事物很快就会过时。总是需要学习升级的工具，寻找编程中最佳方法的新途径和技术。因此，完成本书后不应该停止学习，而是开始研究新的主题，以及本书中未完全涵盖的主题。到这一点，你将拥有知识，可以用来建立高性能应用程序的高性能环境。祝你在 PHP 编程中好运和成功！
