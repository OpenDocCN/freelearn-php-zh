# 第七章. 认证和安全

在本章中，我们将通过添加简单的认证机制和解决现有代码库中的任何安全问题来改进我们在第三章中构建的应用程序，即*你的第一个应用程序*。通过这样做，你将了解：

+   配置和使用认证服务

+   中间件及其如何应用于特定路由

+   数据验证和表单请求

+   网络应用程序中最常见的安全漏洞

+   Laravel 如何帮助你编写更安全的代码

# 认证用户

允许用户注册和登录是网络应用程序中极其常见的功能。然而，PHP 并没有规定应该如何实现，也没有提供任何辅助函数来实现它。这导致了不同的、有时是不安全的用户认证方法和限制对特定页面的访问方法的产生。在这方面，Laravel 为你提供了不同的工具来使这些功能更安全且更容易集成。它是通过其认证服务和尚未覆盖的功能——**中间件**来实现的。

## 创建用户模型

首先，我们需要定义一个模型来表示我们应用程序的用户。Laravel 已经在`config/auth.php`中为你提供了合理的默认设置，你可以在那里更改用于存储用户账户的模型或表。

它还包含一个现有的`User`模型，位于`app/User.php`中。为了这个应用程序的目的，我们将对其进行轻微简化，删除某些类变量，并添加新方法，以便它可以与`Cat`模型交互如下：

```php
namespace App;

use Illuminate\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Auth\Passwords\CanResetPassword;
use Illuminate\Contracts\Auth\Authenticatable as uthenticableContract;
use Illumunate\Contracts\Auth\CanResetPassword as CanResetPasswordContract;
use App\Cat;

class User extends Model implements AuthenticatableContract, CanResetPasswordContract {
  use Authenticable, CanResetPassword;
  public function cats() {
    return $this->hasMany('App\Cat');
  }
 public function owns(Cat $cat) {
 return $this->id == $cat->user_id;
 }
 public function canEdit(Cat $cat) {
 return $this->is_admin || $this->owns($cat);
 }
}
```

首先要注意的是，这个模型实现了`Authenticable`接口。记住，接口并不提供任何实现细节。它不过是一个**合同**，它指定了一个类在实现接口时应定义的方法名称。在这种情况下，`Authenticable`接口强制要求实现以下方法：

+   `getAuthIdentifier`

+   `getAuthPassword`

+   `getRememberToken`

+   `setRememberToken`

+   `getRememberTokenName`

如果你打开`app/User.php`文件，你可能会想知道这些方法在哪里。实际上，这些方法是由`Authenticable`特性提供的。你可以在`User`类的大括号后面看到这个特性被包含进来：

```php
use Authenticable, CanResetPassword;
```

特性允许在类中重用代码。这是为了弥补 PHP 语言的一个不足，即不允许类中的**多重继承**。因此，作为一个解决方案，你可以组合可能被放入多个类中的方法，这些类可能已经扩展了另一个基类。

在我们的`User`模型中，`cats()`方法简单地定义了与`Cat`模型的`hasMany`关系。最后两个方法将用于检查给定的`Cat`实例是否由当前`User`实例拥有或可编辑。

最后，让我们在 `User` 模型上创建一个辅助方法，以便我们可以检查我们是否有管理员权限。这个方法将被命名为 `isAdministrator`，如下所示：

```php
public function isAdministrator()
{
  return $this->getAttribute('is_admin');
}
```

如果使用 MySQL，这将返回一个由 `0` 或 `1` 组成的字符串（因为 MySQL 没有原生的布尔数据类型）。然而，我们可以将这个模型属性转换为布尔类型，以便值检查更好。在你的模型顶部添加以下代码：

```php
protected $casts = [
  'is_admin' => 'boolean',
];
```

在这个数组中，我们定义了属性以及我们实际上想要的数据类型。然后，当我们从模型中检索属性时，它将被转换为指定的数据类型。

其他可以转换为模型属性的 数据类型有如下：

+   `string`

+   `integer`

+   `real`

+   `float`

+   `double`

+   `integer`

`array` 类型可用于包含序列化 JSON 字符串的列，它将反序列化数据并将其呈现为普通的 PHP 数组。

目前我们的 `users` 表中不存在 `is_admin` 属性，所以让我们修复这个问题。

## 创建必要的数据库模式

除了 `User` 模型外，Laravel 还预包装了两个迁移文件：一个用于创建 `users` 表，另一个用于创建 `password_resets` 表。

默认情况下，用户表迁移创建了一个包含每个用户的 ID、名称、密码、记住令牌以及创建时间和更新时间的列的表。我们需要通过添加一个新列来扩展该表，指定每个用户是否是应用程序的管理员。

要做到这一点，我们可以创建另一个迁移。迁移可以用来修改现有的表，也可以用来创建全新的表。在这个例子中，我们将创建一个迁移来向 `users` 表添加一个名为 `is_admin` 的布尔列。

运行以下命令以在 `database/migrations` 中创建迁移文件：

```php
$ php artisan make:migration add_is_admin_column_to_users

```

然后将 `up` 方法更改为如下，以包含模式更改：

```php
public function up() {
  Schema::table('users', function(Blueprint $table) {
    $table->boolean('is_admin')->default(false)
      ->after('password');
  }
}

```

我们将默认值设置为 `false`，这样我们才必须显式地将任何我们希望成为管理员的用户设置为 `true`，而不是每个新用户（以及数据库表中的任何现有用户）在创建时自动获得管理员权限。

就像任何其他迁移一样，我们也必须提供 `down` 方法来回滚任何更改。由于我们已经创建了一个列，如果用户决定回滚迁移，我们需要将其删除：

```php
public function down() {
  Schema::table('users', function(Blueprint $table) {
    $table->dropColumn('is_admin');
  });
}
```

现在，我们还需要更新 `cats` 数据库表，添加一个与用户关联的列。通过遵循前面的步骤，我们创建了一个新的模式，如下所示，描述了更改：

```php
$ php artisan make:migration add_user_id_column_to_cats

```

然后我们按照以下方式完成方法：

```php
public function up() {
  Schema::table('cats', function(Blueprint $table) {
    $table->integer('user_id')->unsigned();
    $table->foreign('user_id')->references('id')->on('users')
      ->onDelete('cascade');
  });
}

```

使用前面的代码，我们修改 `cats` 表以包含一个 `user_id` 列，该列存储 `Cat` 拥有者的 `id`。在创建列之后，我们在表上创建一个 **外键约束**，以确保 `user_id` 值必须匹配 `users` 表中记录的主键。外键有助于你强制数据的一致性（例如，你将无法将 `Cat` 分配给不存在的用户）。级联删除还意味着当用户被删除时，其关联的猫记录也将被删除；否则，数据库将包含不再有任何所有者的猫！

反转此迁移的代码将简单地删除外键约束和列，然后删除 `user_id` 列：

```php
public function down() {
  Schema::table('cats', function(Blueprint $table) {
    $table->dropForeign('cats_user_id_foreign');
    $table->dropColumn('user_id');
  });
}
```

接下来，我们准备一个数据库填充器来为我们的应用程序创建两个用户，其中一个将是管理员。

```php
Use App\User;

class UsersTableSeeder extends Seeder {
  public function run() {
    User::create([
      'username' =>'admin',
      'password' => bcrypt('hunter2'),
      'is_admin' => true, 
    ]);

    User::create([
      'username' => 'scott',
      'password' => bcrypt('tiger'),
      'is_admin' => false,
    ]);
  }
}
```

一旦你将此代码保存在一个名为 `database/seeds/UsersTableSeeder.php` 的新文件中，不要忘记在主 `DatabaseSeeder` 类中调用它。

### 注意

Laravel 预期所有密码都使用 `bcrypt` 辅助函数进行散列，该函数使用 **Bcrypt** 算法创建强散列。你不应该在 *明文* 中存储密码或使用弱算法（如 `md5` 或 `sha1`）对其进行散列。

要同时运行迁移和填充数据库，请输入以下命令：

```php
$ php artisan migrate --seed

```

## 认证路由和视图

我们之前提到 PHP 没有标准的用户认证方式，但 Laravel 并非如此。Laravel 认识到最现代的 Web 应用程序将需要用户注册和登录，因此它自带控制器、路由和视图，以便从一开始就方便地完成这些操作。你可以找到主要的认证控制器在 `app/Http/Controllers/Auth/AuthController.php`。如果你打开该文件，你会看到它只包含一个构造函数，因为像 `User` 模型一样，它使用一个特质来提供功能，在这种情况下，是 `AuthenticatesAndRegistersUsers`：

```php
namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Auth\Registrar;
use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

class AuthController extends Controller {

  use AuthenticatesAndRegistersUsers;

  public function __construct(Guard $auth, Registrar $registrar) {
    $this->auth = $auth;
    $this->registrar = $registrar;

    $this->middleware('guest', ['except' => 'getLogout']);
  }
}
```

`middleware` 方法将应用访客中间件到控制器中的所有操作，除了 `getLogout()` 操作。我们将在本章的后面更深入地探讨中间件。

此控制器（以及用于处理密码重置的控制器）可以在应用程序的路由文件中找到：

```php
$router->controllers([
  'auth' => 'Auth\AuthController',
  'password' => 'Auth\PasswordController',
]);
```

Laravel 还在 `resources/views/auth` 中包含了两个视图，`login.blade.php` 和 `register.blade.php`。

让我们看看如何将 Laravel 的 `auth` 视图集成到我们的应用程序中。我们将首先修改我们的主布局（`resources/views/layouts/master.blade.php`）以显示登录链接给访客，以及显示注销链接给已登录的用户。为了检查访客是否已登录，我们使用 `Auth::check()` 方法：

```php
<div class="container">
  <div class="page-header">
    <div class="text-right">
      @if (Auth::check())
        Logged in as 
        <strong>{{ Auth::user()->username }}</strong>
        {!! link_to('auth/logout', 'Log Out') !!}
      @else
        {!! link_to('auth/login', 'Log In') !!}
      @endif
    </div>
  @yield('header')
  </div>
  @if (Session::has('message'))
    <div class="alert alert-success">
      {{ Session::get('message') }}
    </div>
  @endif

  @if (Session::has('error'))
    <div class="alert alert-warning">
      {{ Session::get('error') }}
    </div>
  @endif
  @yield('content')
</div>
```

我们可以在 `resources/views/auth/login.blade.php` 内替换登录视图，使用一个更简单的表单：

```php
@extends('layouts.master')
@section('header')<h2>Log In</h2>@stop
@section('main')
  {!! Form::open(['url' => 'auth/login') !!}
  <div class="form-group">
    {!! Form::label('username', 'Username', ['class' => 
      'control-label') !!}
    <div class="form-controls">
      {!! Form::text('username', null, ['class' => 
        'form-control']) !!}
    </div>
  </div>
  <div class="form-group">
    {!! Form::label('Password') !!}
    <div class="form-controls">
      {!! Form::password('password', ['class' => 
      'form-control']) !!}
    </div>
  </div>
  {!! Form::submit('Log in', ['class' => 'btn btn-primary') !!}
  {!! Form::close() !!}
@stop

```

我们使用 Blade 语法从 HTML 和表单辅助函数中获取原始值（`{!! $value !!}`），因为它们返回 HTML 标记，如果我们使用默认语法（`{{ $value }}`）来渲染这些，我们会在屏幕上打印 HTML 字符串。

## 中间件

如果你回顾一下`AuthController`，你会在构造方法中注意到以下行：

```php
$this->middleware('auth', ['except' => 'getLogout']);

```

中间件包括可以附加到进入您应用程序的请求的类，并用于更改这些请求的结果。中间件是 Laravel 4 中发现的**路由过滤器**的替代品。

中间件可以在定义路由时注册，或者在前面提到的控制器中注册。前面的例子将`auth`中间件附加到所有将由`AuthController`处理的请求上，除了对`getLogout`方法的请求。

中间件类可以在`app/Http/Middleware`目录中找到。在这里，你可以找到默认的`Authentication`中间件类，以及另外两个：`RedirectIfAuthenticated`和`VerifyCsrfToken`。我们可以检查`Authentication`类来了解中间件类是如何工作的：

```php
public function __construct(Guard $auth) {
  $this->auth = $auth;
}

public function handle($request, Closure $next) {
  if ($this->auth->guest()) {
    if ($request->ajax()) {
      return response('Unauthorized', 401);
    } else {
      return redirect()->guest('auth/login');
    }
  }
  return $next($request);
}
```

有两个方法：构造方法和`handle`方法。在先前的例子中，该类正在检查当前用户是否已认证（使用`Guard`类上的`guest()`方法），如果是访客，则如果请求是通过 AJAX 进行的，则返回响应，或者将用户重定向到登录表单。因为响应是在那里返回的，所以请求将不会进一步处理。

我们可以使用这种方法来检查用户是否已认证，还可以检查他们是否是管理员。我们可以使用内置的 Artisan 生成器创建一个新的中间件类，如下所示：

```php
$ php artisan make:middleware IsAdministrator

```

这将在`app/Http/Middleware/IsAdministrator.php`创建一个新文件。像`Authentication`类一样，我们需要`Guard`实现，所以添加一个构造函数，将依赖项的类型提示，以便服务容器自动注入它：

```php
public function __construct(Guard $auth) {$this->auth = $auth;}
```

我们还需要在文件顶部导入完整的命名空间，如下所示：

```php
use Illuminate\Contracts\Auth\Guard;
```

现在我们有一个`Guard`实例，并将其分配给一个类属性；我们现在可以完善`handle`方法，如下所示：

```php
public function handle($request, \Closure $next) {
  if ( ! $this->auth->user()->isAdministrator()) {
    if ($this->request->ajax()) {
      return response('Forbidden.', 403);
    } else {
      throw new AccessDeniedHttpException;
    }
  }
}
```

这次，我们从`Guard`中获取当前用户（这将产生一个`User` Eloquent 模型实例）。然后我们可以调用这个模型上的任何方法。在先前的例子中，我们调用了我们的`isAdministrator()`方法，这将返回一个布尔值，表示用户是否应该被视为管理员。如果不是——就像`Authenticated`类一样——如果请求是通过 AJAX 进行的，我们返回一个简单的字符串响应（以及适当的 HTTP 状态码）；否则，我们抛出一个`AccessDeniedHttpException`。这个异常实际上是 Symfony `HttpKernel`库的一部分，所以我们需要在文件顶部导入类的完整命名空间：

```php
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
```

创建中间件的最后一步是告诉 HTTP `Kernel` 类关于它。你可以在这个文件中找到它：`app/Http/Kernel.php`。通过打开这个文件，你会看到定义了两个属性：`$middleware` 和 `$routeMiddleware`。将类的完整命名空间添加到 `$middleware` 数组中会将中间件添加到每个请求中。我们不想这样做，因为如果我们这样做，就没有人能够访问登录页面了，因为他们在这个时候将不会被认证！相反，我们想要将一个条目添加到 `$routeMiddleware` 数组中，如下所示：

```php
protected $routeMiddleware = [
  'auth' => 'App\Core\Http\Middleware\Authenticate',
  'auth.basic' => Illuminate\Auth\Middleware\AuthenticateWithBasicAuth',
  'guest' => 'App\Core\Http\Middleware\RedirectIfAuthenticated',
  'admin' => 'App\Http\Middleware\IsAdministrator',
];
```

数组中的键是我们可以在路由和控制器中使用的内容，并且当请求指定的资源时，将应用相应的类。

下面是一个将此应用于路由的示例：

```php
Route::get('admin/dashboard', [
  'middleware' => ['auth', 'admin'],
  'uses' => '\Admin\DashboardController@index',
]);
```

如你所见，你可以指定多个中间件类来应用于单个请求。在先前的路由示例中，首先会调用 `Authenticated` 中间件类；如果一切顺利（并且用户没有被重定向到登录页面），它将被传递到 `IsAdministrator` 中间件类，该类将检查当前登录的用户是否是管理员。

## 验证用户输入

我们的应用程序仍然有一个重大缺陷——它不对用户提交的数据进行任何验证。虽然如果你用纯 PHP 做这件事，可能会得到一系列带有正则表达式的条件，但 Laravel 提供了一种更简单、更健壮的方式来实现这一点。

通过传递一个包含输入数据的数组和一个包含验证规则的数组到 `Validator::make($data, $rules)` 方法来执行验证。在我们的应用程序中，以下是我们可以编写的规则：

```php
$rules = [
  'name' => 'required|min:3', // Required, > 3 characters
  'date_of_birth' => ['required, 'date'] // Must be a date
];
```

可以通过管道符号或作为数组传递来分隔多个验证规则（前述代码中展示了这两种示例）。Laravel 提供了超过 30 种不同的验证规则，它们都记录在这里：

[`laravel.com/docs/validation#available-validation-rules`](http://laravel.com/docs/validation#available-validation-rules)

下面是如何使用表单提交的数据来检查这些规则：

```php
$validator = Validator::make($rules, Input::all());
```

然后，你可以根据 `$validator->fails()` 的输出使你的应用程序做出响应。如果这个方法调用返回 `true`，你将使用 `$validator->messages()` 获取包含所有错误信息的对象。如果你在一个控制器动作中验证数据，你可以将这个对象附加到重定向中，将用户送回表单：

```php
return redirect()
  ->back()
  ->with('errors, $validatior->messages());
```

由于每个字段可以有零个或多个验证错误，你将使用条件语句和循环以及以下方法来显示这些消息：

```php
if ($errors->has('name')) {
 foreach ($errors->get('name') as $error) {
 echo $error;
  }
}
```

你也可以使用像 Ardent 这样的工具，它扩展了 Eloquent，并允许你直接在模型中编写验证规则。你可以从以下链接下载 Ardent：

[`github.com/laravelbook/ardent`](https://github.com/laravelbook/ardent)

### 表单请求

在 Laravel 4 中，你可以自由地将验证放在任何你想放的地方。这导致开发者以无数种方式实现验证，包括在控制器中或作为验证服务。在版本 5 中，Laravel 引入了一种标准化提交数据验证的方法，即通过**表单请求**。

表单请求是封装 Laravel 标准`Request`类的类，但实现了名为`ValidatesWhenResolved`的特质。这个特质包含一个`Validator`实例，并使用你在表单请求类中定义的规则来验证请求中的数据。如果验证器通过，那么应用到的控制器动作将正常执行。如果验证器失败，则用户将被重定向到之前的 URL，并在会话中显示错误。这意味着你不需要在控制器动作中定义验证流程，甚至可以在不同场景下提交相同数据但不同动作的控制器动作中重用它们。

让我们创建一个用于保存猫的详细信息的表单请求。同样，Artisan 为我们提供了一个生成器来创建一个新的表单请求类：

```php
$ php artisan make:request SaveCatRequest

```

这将在`app/Http/Requests/SaveCatRequest.php`文件中创建文件。在文件中，你会找到两个方法：`authorize`和`rules`。

在执行验证之前，表单请求授权当前请求。实现细节取决于你。你可能希望当前用户登录或成为管理员。你可以在该方法中定义这个逻辑。默认情况下，它简单地返回`false`。这并不理想，因为它意味着*没有人*能够执行此请求。由于我们通过中间件处理用户身份验证，我们可以简单地将其更改为返回`true`。

第二个方法，`rules`，是我们提供要馈送给`Validator`实例的验证规则数组的地方。以先前的`Validator`示例为例，这可以更改为以下代码：

```php
public function rules() {
  return [
    'name' => 'required|min:3',
    'date_of_birth' => 'required|date',
  ];
}
```

规则被定义在方法中而不是简单地作为类属性，是为了允许条件验证。可能有些时候你只想验证某个字段，例如，如果另一个字段提供了值。想象一下电子商务网站上的结账表单，它会要求用户提供一个账单地址，如果与账单地址不同，则可选的送货地址。大多数在线商店都会有一个复选框，当勾选时，将显示输入送货地址的字段。如果我们为这种场景创建验证，那么它可能看起来像以下代码：

```php
public function rules() {
  $rules = [
    'billing_address' => 'required',
  ];
  if ($request->has('shipping_address_different') {
    $rules['shipping_address'] = 'required';
  }
  return $rules;
}
```

之前的示例检查是否存在带有`shipping_address_different`（复选框）字段的字段，如果存在，则附加一个验证规则来指定`shipping_address`是必需的。正如你所见，这使得表单请求中的验证非常强大。

通过指定控制器动作的参数来实例化表单请求类。在我们的保存猫的例子中，这将对我们的`CatsController`类中的`create`和`update`方法都适用：

```php
public function create(SaveCatRequest $request) {
  // method body
}

public function update(SaveCatRequest $request) {
  // method body
}
```

现在，每当请求这些动作中的任何一个时，`SaveCatRequest`类将被首先调用并检查数据是否有效。这意味着我们的控制器方法可以保持简洁，并且只处理将新数据持久化到数据库的实际操作。

# 保护你的应用程序（Securing your application）

在你将应用程序部署到充满无情机器人和恶意用户的敌对环境之前，你必须牢记一些安全考虑因素。在本节中，我们将介绍几个常见的针对 Web 应用程序的攻击向量，并了解 Laravel 是如何保护你的应用程序免受这些攻击的。由于框架不能保护你免受所有攻击，我们还将探讨要避免的常见陷阱。

## 跨站请求伪造（Cross-site request forgery）

**跨站请求伪造**（**CSRF**）攻击是通过针对具有副作用（即执行操作而不是仅显示信息）的 URL 来进行的。我们已经通过避免对具有永久效果的路线（如`DELETE/cats/1`）使用`GET`来部分缓解了 CSRF 攻击，因为这些操作无法通过简单链接访问或嵌入到`<iframe>`元素中。然而，如果攻击者能够将他的受害者引导到一个他控制的页面，他可以轻易地让受害者向目标域提交表单。如果受害者已经在目标域登录，应用程序将无法验证请求的真实性。

最有效的对策是在表单显示时发出一个令牌，然后在表单提交时检查该令牌。`Form::open`和`Form::model`都会自动插入一个隐藏的`_token`输入元素，并且中间件会应用于检查传入请求中提供的令牌是否与预期值匹配。

## 转义内容以防止跨站脚本（XSS）

**跨站脚本**（**XSS**）攻击发生在攻击者能够在其他用户查看的页面上放置客户端 JavaScript 代码时。在我们的应用程序中，假设我们的猫的名字没有被转义，如果我们将以下代码片段作为名字的值输入，那么每个访问者都会在我们的猫名字显示的每个地方看到一个警告消息：

```php
Evil Cat <script>alert('Meow!')</script>
```

虽然这是一个相当无害的脚本，但很容易插入更长的脚本或链接到外部脚本，该脚本会窃取会话或 cookie 值。为了避免这种攻击，你不应该信任任何用户提交的数据或转义任何危险字符。你应该在 Blade 模板中优先使用双大括号语法（`{{ $value }}`），并且只有在确定数据可以以原始格式安全显示时才使用`{!! $value !!}`语法。

## 避免 SQL 注入

当一个应用程序在 SQL 查询中插入任意且未经筛选的用户输入时，就存在**SQL 注入**漏洞。这种用户输入可能来自 cookie、服务器变量，或者最常见的是通过`GET`或`POST`输入值。这些攻击的目的是访问或修改通常不可用的数据，有时还会干扰应用程序的正常运行。

默认情况下，Laravel 会保护你免受此类攻击，因为查询构建器和 Eloquent 在幕后都使用**PHP 数据对象**（**PDO**）类。PDO 使用**预处理语句**，这允许你安全地传递任何参数，而无需对它们进行转义和清理。

在某些情况下，你可能想用 SQL 编写更复杂或数据库特定的查询。这可以通过使用`DB::raw`方法实现。当使用此方法时，你必须非常小心，不要创建任何像以下这样的易受攻击的查询：

```php
Route::get('sql-injection-vulnerable', function() {
  $name = "'Bobby' OR 1=1";
  return DB::select( 
    DB::raw("SELECT * FROM cats WHERE name = $name"));
});
```

为了保护此查询免受 SQL 注入攻击，你需要通过在查询中用问号替换参数来重写它，然后将值作为一个数组作为`raw`方法的第二个参数传递：

```php
Route::get('sql-injection-not-vulnerable', function() {
  $name = "'Bobby' OR 1=1";
  return DB::select(
    DB::raw("SELECT * FROM cats WHERE name = ?", [$name]));
});
```

前面的查询被称为**预处理语句**，因为我们定义了查询和预期的参数，并且任何有害的参数都会被清理，以防止它们以未预期的方式更改查询或数据库中的数据。

## 谨慎使用质量分配

在第三章，*你的第一个应用*中，我们使用了质量分配，这是一个方便的特性，允许我们根据表单输入创建一个模型，而无需逐个分配每个值。

然而，这个特性应该谨慎使用。恶意用户可能会在客户端篡改表单并添加新的输入：

```php
<input name="is_admin" value="1" />
```

然后，当表单提交时，我们尝试使用以下代码创建一个新的模型：

```php
Cat::create(Request::all())
```

由于`$fillable`数组定义了可以通过质量分配填充的字段白名单，因此此方法调用将抛出质量分配异常。

你也可以做相反的事情，使用`$guarded`属性定义一个黑名单。然而，这个选项可能具有潜在的危险性，因为你可能会忘记在向模型添加新字段时更新它。

## Cookie – 默认安全

Laravel 通过其`Cookie`类使创建、读取和过期 cookie 变得非常容易。

你还会很高兴地知道，所有 cookie 都会自动签名和加密。这意味着如果它们被篡改，Laravel 会自动丢弃它们。这也意味着你将无法使用 JavaScript 从客户端读取它们。

## 在交换敏感数据时强制使用 HTTPS

如果您通过 HTTP 提供服务，您需要记住，交换的每一比特信息，包括密码，都是以明文形式发送的。因此，同一网络上的攻击者可以拦截私人信息，例如会话变量，并以受害者身份登录。我们唯一能防止这种情况的方法是使用 HTTPS。如果您已经在您的 Web 服务器上安装了 SSL 证书，Laravel 提供了一些助手来在`http://`和`https://`之间切换并限制对某些路由的访问。例如，您可以定义一个`https`过滤器，将访客重定向到安全的路由，如下面的代码片段所示：

```php
Route::filter('https', function() { 
  if ( ! Request::secure()) 
    return Redirect::secure(URI::current()); 
});
```

# 摘要

在本章中，我们学习了如何利用 Laravel 的许多工具为网站添加认证功能、验证数据和避免常见的安全问题。现在您应该拥有创建、测试和确保 Laravel 应用程序所需的所有必要信息。

在附录中，*An Arsenal of Tools*，您将获得 Laravel 提供的许多其他有用功能的便捷参考。
