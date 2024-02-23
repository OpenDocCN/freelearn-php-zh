# 第三章：验证您的应用程序

在本章中，我们将涵盖：

+   设置和配置 Auth 库

+   创建一个身份验证系统

+   在登录后检索和更新用户信息

+   限制对某些页面的访问

+   设置 OAuth 与 HybridAuth 包

+   使用 OpenID 进行登录

+   使用 Facebook 凭据登录

+   使用 Twitter 凭据登录

+   使用 LinkedIn 登录

# 介绍

许多现代网络应用程序都包括用户注册和登录的方式。为了确保我们的应用程序和用户信息的安全，我们需要确保每个用户都经过适当的身份验证。Laravel 包括一个很棒的`Auth`类，使得这个任务非常容易完成。在本章中，我们将从设置我们自己的身份验证系统开始，然后转向在我们的 Laravel 应用程序中使用第三方身份验证。

# 设置和配置 Auth 库

要使用 Laravel 的身份验证系统，我们需要确保它设置正确。在这个食谱中，我们将看到一种常见的完成设置的方式。

## 准备工作

要设置身份验证，我们只需要安装 Laravel 并运行一个 MySQL 实例。

## 如何做…

要完成这个步骤，请按照以下步骤进行：

1.  进入您的`app/config/session.php`配置文件，并确保它设置为使用`native`：

```php
**'driver' => 'native'**

```

1.  `app/config/auth.php`配置文件的默认设置应该是可以的，但确保它们设置如下：

```php
'driver' => 'eloquent',
'model' => 'User',
'table' => 'users',
```

1.  在 MySQL 中，创建一个名为`authapp`的数据库，并确保在`app/config/database.php`配置文件中设置正确。以下是我们将使用的设置：

```php
'default' => 'mysql',

'connections' => array(

    'mysql' => array(
        'driver'   => 'mysql',
        'host'     => 'localhost',
        'database' => 'authapp',
        'username' => 'root',
        'password' => '',
        'charset'  => 'utf8',
        'prefix'   => '',
    ),
),
```

1.  我们将使用迁移和 Schema 构建器以及 Artisan 命令行来设置我们的`Users`表，因此我们需要创建我们的迁移表：

```php
**php artisan migrate:install**

```

1.  为我们的`Users`表创建迁移：

```php
**php artisan migrate:make create_users_table**

```

1.  在`app/database/migrations`目录中，将会有一个新文件，文件名是日期后跟着`create_users_table.php`。在那个文件中，我们创建我们的表：

```php
<?php

use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration {

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
            $table->string('email');
            $table->string('password', 64);
            $table->string('name');
            $table->boolean('admin');
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
        Schema::drop('users');
    }

}
```

1.  在 Artisan 中运行迁移来创建我们的表，一切都应该设置好了：

```php
**php artisan migrate**

```

## 它是如何工作的…

身份验证使用会话来存储用户信息，因此我们首先需要确保我们的会话配置正确。有各种各样的方式来存储会话，包括使用数据库或 Redis，但是为了我们的目的，我们将只使用`native`驱动程序，它利用了 Symfony 的原生会话驱动程序。

在设置身份验证配置时，我们将使用 Eloquent ORM 作为我们的驱动程序，电子邮件地址作为我们的用户名，模型将是 User。Laravel 附带了一个默认的 User 模型，并且它在开箱即用时非常好用，所以我们将使用它。为了简单起见，我们将坚持使用表名的默认配置，即模型类名的复数形式，但是如果我们想要的话，我们可以自定义它。

一旦我们确保我们的数据库配置设置正确，我们就可以使用 Artisan 来创建我们的迁移。在我们的迁移中，我们将创建我们的用户表，并存储电子邮件地址、密码、姓名和一个布尔字段来存储用户是否是管理员。完成后，我们运行迁移，我们的数据库将设置好来构建我们的身份验证系统。

# 创建身份验证系统

在这个食谱中，我们将创建一个简单的身份验证系统。它可以直接使用，也可以扩展以包括更多的功能。

## 准备工作

我们将使用*设置和配置 Auth 库*食谱中创建的代码作为我们身份验证系统的基础。

## 如何做…

要完成这个步骤，请按照以下步骤进行：

1.  在我们的`routes.php`文件中创建一个路由来保存我们的注册表单：

```php
Route::get('registration', function()
{
    return View::make('registration');
});
```

1.  通过在`app/views`中创建一个名为`registration.php`的新文件来创建一个注册表单：

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Laravel Authentication - Registration</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h2>Laravel Authentication - Registration</h2>
        <?php $messages =  $errors->all('<p style="color:red">:message</p>') ?>
        <?php foreach ($messages as $msg): ?>
            <?= $msg ?>
        <?php endforeach; ?>

<?= Form::open() ?>
        <?= Form::label('email', 'Email address: ') ?>
        <?= Form::text('email', Input::old('email')) ?>
        <br>
        <?= Form::label('password', 'Password: ') ?>
        <?= Form::password('password') ?>
        <br>
        <?= Form::label('password_confirm','Retype Password: ') ?>
        <?= Form::password('password_confirm') ?>
        <br>
        <?= Form::label('name', 'Name: ') ?>
        <?= Form::text('name', Input::old('name')) ?>
        <br>
        <?= Form::label('admin', 'Admin?: ') ?>
        <?= Form::checkbox('admin','true',Input::old('admin')) ?>
        <br>
        <?= Form::submit('Register!') ?>
        <?= Form::close() ?>
    </body>
</html>
```

1.  创建一个路由来处理注册页面：

```php
Route::post('registration', array('before' => 'csrf',function()
{
    $rules = array(
        'email'    => 'required|email|unique:users',
        'password' => 'required|same:password_confirm',
        'name'     => 'required'
    );
    $validation = Validator::make(Input::all(), $rules);

    if ($validation->fails())
    {
        return Redirect::to('registration')->withErrors($validation)->withInput();
    }

    $user           = new User;
    $user->email    = Input::get('email');
    $user->password = Hash::make(Input::get('password'));
    $user->name     = Input::get('name');
    $user->admin    = Input::get('admin') ? 1 : 0;
    if ($user->save())
    {
        Auth::loginUsingId($user->id);
        return Redirect::to('profile');
    }
    return Redirect::to('registration')->withInput();
}));
```

1.  通过在`routes.php`中添加一个路由来为您的个人资料创建一个简单的页面：

```php
Route::get('profile', function()
{
    if (Auth::check())
    {
        return 'Welcome! You have been authorized!';
    }
    else
    {
        return 'Please <a href="login">Login</a>';
    }
});
```

1.  在`routes.php`中创建一个登录路由来保存登录表单：

```php
Route::get('login', function()
{
    return View::make('login');
});
```

1.  在我们的`app/views`目录中，创建一个名为`login.php`的文件：

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Laravel Authentication - Login</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h2>Laravel Authentication - Login</h2>
        <?= '<span style="color:red">' .Session::get('login_error') . '</span>' ?>

        <?= Form::open() ?>
        <?= Form::label('email', 'Email address: ') ?>
        <?= Form::text('email', Input::old('email')) ?>
        <br>
        <?= Form::label('password', 'Password: ') ?>
        <?= Form::password('password') ?>
        <br>
        <?= Form::submit('Login!') ?>
        <?= Form::close() ?>
    </body>
</html>
```

1.  在`routes.php`中创建一个路由来验证登录：

```php
Route::post('login', function()
{
    $user = array(
        'username' => Input::get('email'),
        'password' => Input::get('password')
    );

    if (Auth::attempt($user))
    {
        return Redirect::to('profile');
    }

    return Redirect::to('login')->with('login_error','Could not log in.');
});
```

1.  在`routes.php`中创建一个安全页面的路由：

```php
Route::get('secured', array('before' => 'auth', function()
{
    return 'This is a secured page!';
}));
```

## 工作原理...

首先，我们创建一个相当简单的注册系统。在我们的注册表单中，我们将要求输入电子邮件地址、密码、密码确认、姓名，以及用户是否是管理员的选项。在表单字段中，我们还添加了`Input::old()`；因此，如果表单验证不正确，我们可以重新填充字段，而无需用户重新输入所有信息。

然后我们的表单提交，添加 CSRF 过滤器，并进行一些验证。如果验证通过，我们就创建一个新的 User 模型实例，并添加表单中的字段。对于密码，我们使用`Hash::make()`来保护密码安全。由于我们的 admin 字段接受布尔值，我们检查 admin 复选框是否被选中；如果是，我们将值设置为`1`。

如果一切保存正确，我们可以通过将刚创建的用户 ID 传递给`Auth::loginUsingId()`来自动登录用户，并将他们重定向到 profile 页面。

profile 路由的第一件事是运行`Auth::check()`来查看用户是否真的已登录。如果没有，它将显示一个链接到登录页面。

登录页面是一个简单的表单，要求输入电子邮件 ID 和密码。提交后，我们将这两个值放入一个数组中，并将它们传递给`Auth::attempt()`，它将自动对我们的密码进行哈希处理，并在数据库中查找凭据。如果成功，`Auth`类将设置一个会话，并将用户重定向到 profile 页面。

如果用户尝试访问*安全*路由，系统将把他们重定向到登录页面。使用 Laravel 的`Redirect::intended()`，我们可以将他们重定向回他们最初尝试访问的页面。

## 另请参阅

+   *设置和配置 Auth 库*示例

# 在登录后检索和更新用户信息

用户登录后，我们需要获取关于他/她的信息。在这个示例中，我们将看到如何获取这些信息。

## 准备工作

我们将使用*设置和配置 Auth 库*和*创建身份验证系统*示例中创建的代码作为此示例的基础。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  使用以下代码更新 profile 路由：

```php
Route::get('profile', function()
{
    if (Auth::check())
    {
        return View::make('profile')->with('user',Auth::user());
    }
    else
    {
        return Redirect::to('login')->with('login_error','You must login first.');
    }
});
```

1.  通过在`app/views`目录中创建一个名为`profile.php`的文件来创建我们的 profile 视图：

```php
<?php echo Session::get('notify') ?  "<p style='color:
    green'>" . Session::get('notify') . "</p>" : "" ?>
<h1>Welcome <?php echo $user->name ?></h1>
<p>Your email: <?php echo $user->email ?></p>
<p>Your account was created on: <?php echo $user
    ->created_at ?></p>
<p><a href="<?= URL::to('profile-edit') ?>">Edit your
    information</a></p>
```

1.  创建一个路由来保存我们的表单以编辑信息：

```php
Route::get('profile-edit', function()
{
    if (Auth::check())
    {
        $user = Input::old() ? (object) Input::old() :Auth::user();
        return View::make('profile_edit')->with('user',$user);
    }
});
```

1.  为我们的编辑表单创建一个视图：

```php
<h2>Edit User Info</h2>
<?php $messages =  $errors->all('<p style="color:red">:message</p>') ?>
<?php foreach ($messages as $msg): ?>
    <?= $msg ?>
<?php endforeach; ?>
<?= Form::open() ?>
<?= Form::label('email', 'Email address: ') ?>
<?= Form::text('email', $user->email) ?>
<br>
<?= Form::label('password', 'Password: ') ?>
<?= Form::password('password') ?>
<br>
<?= Form::label('password_confirm', 'Retype Password: ') ?>
<?= Form::password('password_confirm') ?>
<br>
<?= Form::label('name', 'Name: ') ?>
<?= Form::text('name',  $user->name) ?>
<br>
<?= Form::submit('Update!') ?>
<?= Form::close() ?>
```

1.  创建一个处理表单的路由：

```php
Route::post('profile-edit', function()
{
    $rules = array(
        'email'    => 'required|email',
        'password' => 'same:password_confirm',
        'name'     => 'required'
    );
    $validation = Validator::make(Input::all(), $rules);

    if ($validation->fails())
    {
        return Redirect::to('profile-edit')->withErrors($validation)->withInput();
    }

    $user = User::find(Auth::user()->id);
    $user->email = Input::get('email');
    if (Input::get('password')) {
        $user->password = Hash::make(Input::get('password'));
    }
    $user->name = Input::get('name');
    if ($user->save())
    {
        return Redirect::to('profile')->with('notify','Information updated');
    }
    return Redirect::to('profile-edit')->withInput();
});
```

## 工作原理...

为了获取用户的信息并允许他/她更新信息，我们首先重新设计我们的 profile 路由。我们创建一个 profile 视图，并将`Auth::user()`传递给变量`$user`。然后，在视图文件中，我们简单地输出我们收集到的任何信息。我们还创建了一个链接到一个页面，用户可以在该页面编辑他/她的信息。

我们的 profile 编辑页面首先检查用户是否已登录。如果是，我们希望填充`$user`变量。由于如果有验证错误，我们将重新显示表单，所以我们首先检查`Input::old()`中是否有任何内容。如果没有，这可能是页面的新访问，所以我们只使用`Auth::user()`。如果正在使用`Input::old()`，我们将将其重新转换为对象，因为它通常是一个数组，并在我们的`$user`变量中使用它。

我们的编辑视图表单与注册表单非常相似，只是如果我们已登录，表单已经被填充。

当表单提交时，它会经过一些验证。如果一切有效，我们需要从数据库中获取用户，使用`User::find()`和存储在`Auth::user()`中的用户 ID。然后我们将我们的表单输入添加到用户对象中。对于密码字段，如果它为空，我们可以假设用户不想更改它。因此，我们只有在已经输入了内容时才会更新密码。

最后，我们保存用户信息并将其重定向回个人资料页面。

## 还有更多...

我们数据库中的电子邮件值可能需要是唯一的。对于本步骤，我们可能需要快速检查用户表，并确保正在更新的电子邮件地址没有在其他地方使用。

## 另请参阅

+   *创建身份验证系统*的步骤

# 限制对某些页面的访问

在本步骤中，我们将探讨如何限制对应用程序中各种页面的访问。这样，我们可以使页面只对具有正确凭据的用户可见。

## 准备工作

我们将使用*设置和配置 Auth 库*和*创建身份验证系统*的步骤中创建的代码作为本步骤的基础。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在我们的`filters.php`文件中创建一个检查已登录用户的过滤器。默认的 Laravel`auth`过滤器就可以了：

```php
Route::filter('auth', function()
{
    if (Auth::guest()) return Redirect::guest('login');
});
```

1.  在`filter.php`中创建一个用于检查用户是否为管理员的过滤器：

```php
Route::filter('auth_admin', function()
{
    if (Auth::guest()) return Redirect::guest('login');
    if (Auth::user()->admin != TRUE)
        return Redirect::to('restricted');
});
```

1.  创建一个我们限制给已登录用户的路由：

```php
Route::get('restricted', array('before' => 'auth',
    function()
{
    return 'This page is restricted to logged-in users!
        <a href="admin">Admins Click Here.</a>';
}));
```

1.  创建一个只限管理员访问的路由：

```php
Route::get('admin', array('before' => 'auth_admin',function()
{
    return 'This page is restricted to Admins only!';
}));
```

## 它是如何工作的...

过滤器是 Laravel 的一个强大部分，可以用来简化许多任务。Laravel 默认的`auth`过滤器只是简单地检查用户是否已登录，如果没有，则将其重定向到登录页面。在我们的`restricted`路由中，我们添加`auth`过滤器在函数执行之前运行。

我们的`auth_admin`过滤器用于确保用户已登录，并检查用户是否设置为`admin`。如果没有，他/她将被重定向回普通的受限页面。

# 使用 HybridAuth 包设置 OAuth

有时我们可能不想担心存储用户的密码。在这种情况下，OAuth 已经成为一个流行的选择，它允许我们基于第三方服务（如 Facebook 或 Twitter）对用户进行身份验证。本步骤将展示如何设置`HybridAuth`包以简化 OAuth。

## 准备工作

对于本步骤，我们需要一个标准的 Laravel 安装和一种访问命令行界面的方法，以便我们可以使用 Artisan 命令行实用程序。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  打开我们应用的`composer.json`文件，并将 HybridAuth 添加到`require`部分，使其看起来像这样：

```php
"require": {
    "laravel/framework": "4.0.*",
    "hybridauth/hybridauth": "dev-master"
},
```

1.  在命令行界面中，按以下方式更新 composer：

```php
**php composer.phar update**

```

1.  在`app/config`目录中，创建一个名为`oauth.php`的新文件：

```php
<?php
return array(
    "base_url"   => "http://path/to/our/app/oauth/auth",
    "providers"  => array (
        "OpenID" => array ("enabled" => true),
        "Facebook" => array (
            "enabled"  => TRUE,
            "keys"     => array ("id" => "APP_ID", "secret"=> "APP_SECRET"),
            "scope"    => "email",
        ),
        "Twitter" => array (
            "enabled" => true,
            "keys"    => array ("key" => "CONSUMER_KEY","secret" => "CONSUMER_SECRET")
        ),
        "LinkedIn" => array (
            "enabled" => true,
            "keys" => array ("key" => "APP_KEY", "secret"=> "APP_SECRET")
        )
    )
);
```

## 它是如何工作的...

我们首先要将 HybridAuth 包添加到我们的 composer 文件中。现在，当我们更新 composer 时，它将自动下载并安装该包。从那时起，我们可以在整个应用程序中使用该库。

我们的下一步是设置一个配置文件。该文件以一个 URL 开头，身份验证站点将向该 URL 发送用户。该 URL 应该路由到我们将运行 HybridAuth 并进行实际身份验证的路由或控制器。最后，我们需要添加我们要对抗进行身份验证的站点的凭据。可以在 HybridAuth 网站上找到完整的站点列表：[`hybridauth.sourceforge.net/userguide.html`](http://hybridauth.sourceforge.net/userguide.html)。

# 使用 OpenID 进行登录

如果我们不想在我们的应用程序中存储用户的密码，还有其他使用第三方的身份验证方法，比如 OAuth 和 OpenID。在本步骤中，我们将使用 OpenID 来登录我们的用户。

## 准备工作

对于本步骤，我们需要一个标准的 Laravel 安装，并完成*使用 HybridAuth 包设置 OAuth*的步骤。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在我们的`app/config`目录中，创建一个名为`openid_auth.php`的新文件：

```php
<?php
return array(
    "base_url"   => "http://path/to/our/app/openid/auth",
    "providers"  => array (
        "OpenID" => array ("enabled" => TRUE)
    )
);
```

1.  在我们的`routes.php`文件中，创建一个路由来保存我们的登录表单：

```php
Route::get('login', function()
{
    return View::make('login');
});
```

1.  在我们的`app/views`目录中，创建一个名为`login.php`的新视图：

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Laravel Open ID Login</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h1>OpenID Login</h1>
        <?= Form::open(array('url' => 'openid', 'method' =>'POST')) ?>
        <?= Form::label('openid_identity', 'OpenID') ?>
        <?= Form::text('openid_identity', Input::old('openid_identity')) ?>
        <br>
        <?= Form::submit('Log In!') ?>
        <?= Form::close() ?>
    </body>
</html>
```

1.  在`routes.php`中，创建用于运行身份验证的路由：

```php
Route::any('openid/{auth?}', function($auth = NULL)
{
    if ($auth == 'auth') {
        try {
            Hybrid_Endpoint::process();
        } catch (Exception $e) {
            return Redirect::to('openid');
        }
        return;
    }

    try {
        $oauth = new Hybrid_Auth(app_path(). '/config/openid_auth.php');
        $provider = $oauth->authenticate('OpenID',array('openid_identifier' =>Input::get('openid_identity')));
        $profile = $provider->getUserProfile();
    }
    catch(Exception $e) {
        return $e->getMessage();
    }
    echo 'Welcome ' . $profile->firstName . ' ' . $profile->lastName . '<br>';
    echo 'Your email: ' . $profile->email . '<br>';
    dd($profile);
});
```

## 它是如何工作的...

我们首先创建一个 HybridAuth 库的配置文件，设置用户在身份验证后将被重定向的 URL，并启用 OpenID。

接下来，我们创建一个路由和一个视图，用户可以在其中输入他们想要使用的 OpenID URL。一个流行的 URL 是 Google 的 URL，所以我们建议使用 URL[`www.google.com/accounts/o8/id`](https://www.google.com/accounts/o8/id)，甚至可以自动将其设置为表单中的一个值。

提交表单后，我们应该被引导到 OpenID 网站的身份验证系统，然后重定向回我们的网站。在那里，我们可以显示用户的姓名和电子邮件 ID，并显示所有发送回来的信息。

## 还有更多...

有关 OpenID 提供的更多信息，请访问[`openid.net/developers/specs/`](http://openid.net/developers/specs/)。

# 使用 Facebook 凭据登录

如果我们不想担心存储用户的信息和凭据，我们可以使用 OAuth 来与另一个服务进行身份验证。其中一个最受欢迎的是使用 Facebook 进行登录。使用 Laravel 和 HybridAuth 库，我们可以轻松地实现与 Facebook 的 OAuth 身份验证。

## 准备工作

对于这个步骤，我们需要安装 HybridAuth 包，并按照*使用 HybridAuth 包设置 OAuth*的步骤进行设置。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在[`developers.facebook.com`](https://developers.facebook.com)创建一个新的应用程序。

1.  获取 App ID 和 App Secret 密钥，在`app/config`目录中创建一个名为`fb_auth.php`的文件：

```php
<?php
return array(
    "base_url" => "http://path/to/our/app/fbauth/auth",
    "providers" => array (
        "Facebook" => array (
            "enabled"  => TRUE,
            "keys" => array ("id" => "APP_ID", "secret" =>"APP_SECRET"),
            "scope" => "email"
        )
    )
);
```

1.  在`routes.php`中创建一个用于我们的 Facebook 登录按钮的路由：

```php
Route::get('facebook', function()
{
    return "<a href='fbauth'>Login with Facebook</a>";
});
```

1.  创建一个路由来处理登录信息并显示它：

```php
Route::get('fbauth/{auth?}', function($auth = NULL)
{
    if ($auth == 'auth') {
        try {
            Hybrid_Endpoint::process();
        } catch (Exception $e) {
            return Redirect::to('fbauth');
        }
        return;
    }

    try {
        $oauth = new Hybrid_Auth(app_path(). '/config/fb_auth.php');
        $provider = $oauth->authenticate('Facebook');
        $profile = $provider->getUserProfile();
    }
    catch(Exception $e) {
        return $e->getMessage();
    }
    echo 'Welcome ' . $profile->firstName . ' '. $profile->lastName . '<br>';
    echo 'Your email: ' . $profile->email . '<br>';
    dd($profile);
});
```

## 它是如何工作的...

获取我们的 Facebook API 凭据后，我们需要创建一个包含这些凭据和回调 URL 的配置文件。我们还需要传递作用域，这是我们可能想要从用户那里获得的任何额外权限。在这种情况下，我们只是要获取他们的电子邮件 ID。

我们的 Facebook 登录页面是一个简单的链接到一个路由，我们在那里进行身份验证。然后用户将被带到 Facebook 进行登录和/或授权我们的网站，然后重定向回我们的`fbauth`路由。

在这一点上，我们只是显示返回的信息，但我们可能也想将信息保存到我们自己的数据库中。

## 还有更多...

如果我们在本地计算机上使用 MAMP 或 WAMP 进行测试，Facebook 允许我们使用 localhost 作为回调 URL。

# 使用 Twitter 凭据登录

如果我们不想担心存储用户的信息和凭据，我们可以使用 OAuth 来与另一个服务进行身份验证。一个常用的用于登录的服务是 Twitter。使用 Laravel 和 HybridAuth 库，我们可以轻松地实现与 Twitter 的 OAuth 身份验证。

## 准备工作

对于这个步骤，我们需要安装 HybridAuth 包，并按照*使用 HybridAuth 包设置 OAuth*的步骤进行设置。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在[`dev.twitter.com/apps`](https://dev.twitter.com/apps)创建一个新的应用程序。

1.  获取 Consumer Key 和 Consumer Secret，并在`app/config`目录中创建一个名为`tw_auth.php`的文件：

```php
<?php
return array(
    "base_url"   => "http://path/to/our/app/twauth/auth",
    "providers"  => array (
        "Twitter" => array (
            "enabled" => true,
            "keys"    => array ("key" => "CONSUMER_KEY",
			     "secret" => "CONSUMER_SECRET")
        )
    )
);
```

1.  在`routes.php`中创建一个用于我们的 Twitter 登录按钮的路由：

```php
Route::get('twitter', function()
{
    return "<a href='twauth'>Login with Twitter</a>";
});
```

1.  创建一个路由来处理 Twitter 信息：

```php
Route::get('twauth/{auth?}', function($auth = NULL)
{
    if ($auth == 'auth') {
        try {
            Hybrid_Endpoint::process();
        } catch (Exception $e) {
            return Redirect::to('twauth');
        }
        return;
    }

    try {
        $oauth = new Hybrid_Auth(app_path(). '/config/tw_auth.php');
        $provider = $oauth->authenticate('Twitter');
        $profile = $provider->getUserProfile();
    }
    catch(Exception $e) {
        return $e->getMessage();
    }
    echo 'Welcome ' . $profile->displayName . '<br>';
    echo 'Your image: <img src="' . $profile->photoURL. '">';
    dd($profile);
});
```

## 它是如何工作的...

获取我们的 Twitter API 凭据后，我们需要创建一个包含这些凭据和回调 URL 的配置文件。

然后我们创建一个 Twitter 登录视图，这是一个简单的链接到一个路由，我们在那里进行身份验证。然后用户将被带到 Twitter 进行登录和/或授权我们的网站，然后重定向回我们的`twauth`路由。在这里，我们获取他们的显示名称和他们的 Twitter 图标。

在这一点上，我们只是显示返回的信息，但我们可能也想将信息保存到我们自己的数据库中。

## 还有更多...

如果我们在本地计算机上使用类似 MAMP 或 WAMP 的东西进行测试，Twitter 将不允许使用 localhost 作为回调 URL，但我们可以使用`127.0.0.1`代替。

# 使用 LinkedIn 进行登录

如果我们不想担心存储用户信息和凭据，我们可以使用 OAuth 来验证另一个服务。一个常用的用于登录的服务，特别是用于商业应用程序的服务，是 LinkedIn。使用 Laravel 和`HybridAuth`库，我们可以轻松地实现与 LinkedIn 的 OAuth 验证。

## 准备工作

对于这个步骤，我们需要安装并设置 HybridAuth 包，就像在*使用 HybridAuth 包设置 OAuth*步骤中一样。

## 如何做...

要完成这个步骤，请按照以下步骤进行操作：

1.  在[`www.linkedin.com/secure/developer`](https://www.linkedin.com/secure/developer)创建一个新的应用程序。

1.  获取 API 密钥和秘密密钥，在`app/config`目录中创建一个名为`li_auth.php`的文件：

```php
<?php
return array(
    "base_url"   => "http://path/to/our/app/liauth/auth",
    "providers"  => array (
        "LinkedIn" => array (
            "enabled" => true,
            "keys"    => array ("key" => "API_KEY","secret" => "SECRET_KEY")
        )
    )
);
```

1.  在`routes.php`中创建一个用于 LinkedIn 登录按钮的路由：

```php
Route::get('linkedin', function()
{
    return "<a href='liauth'>Login with LinkedIn</a>";
});
```

1.  创建一个处理 LinkedIn 信息的路由：

```php
Route::get('liauth/{auth?}', function($auth = NULL)
{
    if ($auth == 'auth') {
        try {
            Hybrid_Endpoint::process();
        } catch (Exception $e) {
            return Redirect::to('liauth');
        }
        return;
    }

    try {
        $oauth = new Hybrid_Auth(app_path(). '/config/li_auth.php');
        $provider = $oauth->authenticate('LinkedIn');
        $profile = $provider->getUserProfile();
    }
    catch(Exception $e) {
        return $e->getMessage();
    }
    echo 'Welcome ' . $profile->firstName . ' ' . $profile->lastName . '<br>';
    echo 'Your email: ' . $profile->email . '<br>';
    echo 'Your image: <img src="' . $profile->photoURL. '">';
    dd($profile);
});
```

## 它是如何工作的...

获得我们的 LinkedIn API 凭据后，我们需要创建一个包含这些凭据和回调 URL 的配置文件。

然后我们创建一个 LinkedIn 登录视图，其中包含一个简单的链接到一个路由，我们在这个路由中进行 LinkedIn 验证。用户将被带到 LinkedIn 网站进行登录和/或授权我们的网站，然后重定向回我们的`liauth`路由。在这里，我们获取他们的名字、姓氏、电子邮件 ID 和他们的头像。

在这一点上，我们只是显示返回的信息，但我们可能也想将信息保存到我们自己的数据库中。
