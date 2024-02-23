# 第九章：使用 Passport 添加用户登录和 API 身份验证

在上一章中，我们允许用户保存他们喜欢的 Vuebnb 列表。但是，这个功能只在前端应用中实现，所以如果用户重新加载页面，他们的选择将会丢失。

在本章中，我们将创建一个用户登录系统，并将保存的项目持久化到数据库中，以便在页面刷新后检索。

本章涵盖的主题：

+   利用 Laravel 内置的身份验证功能设置用户登录系统

+   创建带有 CSRF 保护的登录表单

+   在存储中使用 Vuex 操作进行异步操作

+   OAuth 协议的简要介绍，用于 API 身份验证

+   使用 Laravel Passport 允许经过身份验证的 AJAX 请求

# 用户模型

为了将列表项保存到数据库中，我们首先需要一个用户模型，因为我们希望每个用户都有自己独特的列表。添加用户模型意味着我们还需要一个身份验证系统，以便用户可以登录和退出。幸运的是，Laravel 提供了一个功能齐全的用户模型和身份验证系统。

现在让我们来看看用户模型样板文件，看看需要对其进行哪些修改以适应我们的目的。

# 迁移

首先看一下数据库迁移，用户表模式已经包括 ID、名称、电子邮件和密码列。

`database/migrations/2014_10_12_000000_create_users_table.php`：

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{
  public function up()
  { Schema::create('users', function (Blueprint $table) {
      $table->increments('id');
      $table->string('name');
      $table->string('email')->unique();
      $table->string('password');
      $table->rememberToken();
      $table->timestamps();
    });
  }

  public function down()
  { Schema::dropIfExists('users');
  }
}
```

如果我们添加一个额外的列来存储保存的列表 ID，那么这个模式对我们的需求就足够了。理想情况下，我们会将它们存储在一个数组中，但是由于关系数据库没有数组列类型，我们将把它们存储为一个序列化的字符串，例如，在`text`列中`[1, 5, 10]`。

`database/migrations/2014_10_12_000000_create_users_table.php`：

```php
Schema::create('users', function (Blueprint $table) {
  ...
  $table->text('saved');
});
```

# 模型

现在让我们来看看 Laravel 提供的`User`模型类。

`app/User.php`：

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
  use Notifiable;

  protected $fillable = [
    'name', 'email', 'password',
  ];

  protected $hidden = [
    'password', 'remember_token',
  ];
}
```

默认配置是可以的，但让我们通过将其添加到`$fillable`数组中，允许`saved`属性进行批量赋值。

当我们读取或写入时，我们还将使我们的模型序列化和反序列化`saved`文本。为此，我们可以向模型添加一个`$casts`属性，并将`saved`转换为数组。

`app/User.php`：

```php
class User extends Authenticatable
{
  ...

  protected $fillable = [
    'name', 'email', 'password', 'saved'
  ];

  ...

  protected $casts = [
    'saved' => 'array'
  ];
}
```

现在我们可以将`saved`属性视为数组，即使它在数据库中存储为字符串：

```php
echo gettype($user->saved());

// array
```

# Seeder

在一个普通的带有登录系统的 Web 应用中，您会有一个注册页面，让用户创建自己的帐户。为了确保本书不会变得太长，我们将跳过该功能，而是使用数据库 seeder 生成用户帐户：

```php
$ php artisan make:seeder UsersTableSeeder
```

如果您愿意，您可以为 Vuebnb 自己实现一个注册页面。Laravel 文档在[`laravel.com/docs/5.5/authentication`](https://laravel.com/docs/5.5/authentication)中对此进行了详细介绍。

让我们至少创建一个帐户，其中包括名称、电子邮件、密码和一个保存列表的数组。请注意，我使用了`Hash`外观的`make`方法来对密码进行哈希处理，而不是将其存储为纯文本。Laravel 的默认`LoginController`在登录过程中将自动对纯文本密码进行哈希处理。

`database/seeds/UsersTableSeeder.php`：

```php
<?php

use Illuminate\Database\Seeder;
use App\User;
use Illuminate\Support\Facades\Hash;

class UsersTableSeeder extends Seeder
{
  public function run()
  { User::create([
      'name'      => 'Jane Doe',
      'email'     => 'test@gmail.com',
      'password'  => Hash::make('test'),
      'saved'     => [1,5,7,9]
    ]);
  }
}
```

要运行 seeder，我们需要从主`DatabaseSeeder`类中调用它。

`database/seeds/DatabaseSeeder.php`：

```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
  public function run()
  {
    $this->call(ListingsTableSeeder::class);
    $this->call(UsersTableSeeder::class);
  }
}
```

现在让我们重新运行我们的迁移和 seeder，以安装用户表和数据，使用以下命令：

```php
$ php artisan migrate:refresh --seed
```

为了确认我们的用户表和数据是否正确创建，我们将使用 Tinker 来查询表。您应该会得到类似以下的输出：

```php
$ php artisan tinker
 >>> DB::table('users')->get(); /* {
  "id": 1, "name": "Jane Doe", "email": "test@gmail.com", "password": "...", "remember_token": null, "created_at": "2017-10-27 02:30:31", "updated_at": "2017-10-27 02:30:31", "saved": "[1,5,7,9]"
} */
```

# 登录系统

现在我们已经创建了用户模型，我们可以实现登录系统的其余部分。同样，Laravel 将其作为一个开箱即用的功能包含在内，所以我们只需要进行少量配置。

以下是登录系统的概述：

1.  用户在登录表单中提供他们的电子邮件和密码。我们将使用 Vue 创建这个表单

1.  表单提交到`/login` POST 路由

1.  `LoginController`然后将验证用户的凭据与数据库匹配

1.  如果登录成功，用户将被重定向到主页。会话 cookie 附加到响应中，然后传递给所有外发请求以验证用户

以下是登录系统的图解表示，以便更清晰地理解：

![](img/d2939bb2-239e-4662-839b-ed986a6a665b.png)图 9.1\. 登录流程

# LoginPage 组件

我们的应用程序需要一个登录页面，所以让我们创建一个新的页面组件：

```php
$ touch resources/assets/components/LoginPage.vue
```

我们将首先定义模板标记，其中包括一个带有电子邮件和密码字段以及提交按钮的表单。表单使用 HTTP POST 方法，并发送到`/login`路径。我将表单元素包装在一个带有`.form-controller`类的`div`中，以帮助进行样式设置。

`resources/assets/components/LoginPage.vue`:

```php
<template>
  <div id="login" class="login-container">
    <form role="form" method="POST" action="/login">
      <div class="form-control">
        <input id="email" type="email" name="email" 
          placeholder="Email Address" required autofocus>
      </div>
      <div class="form-control">
        <input id="password" type="password" name="password" 
          placeholder="Password" required>
      </div>
      <div class="form-control">
        <button type="submit">Log in</button>
      </div>
    </form>
  </div>
</template>
```

我们现在还不需要任何 JavaScript 功能，所以让我们现在添加我们的 CSS 规则。

`resources/assets/components/LoginPage.vue`:

```php
<template>...</template>
<style> #login form {
    padding-top: 40px;
  }

  @media (min-width: 744px) {
    #login form {
      padding-top: 80px;
    }
  }

  #login .form-control {
    margin-bottom: 1em;
  }

  #login input[type=email],
  #login input[type=password],
  #login button,
  #login label {
    width: 100%;
    font-size: 19px !important;
    line-height: 24px;
    color: #484848;
    font-weight: 300;
  }

  #login input {
    background-color: transparent;
    padding: 11px;
    border: 1px solid #dbdbdb;
    border-radius: 2px;
    box-sizing:border-box }

  #login button {
    background-color: #4fc08d;
    color: #ffffff;
    cursor: pointer;
    border: #4fc08d;
    border-radius: 4px;
    padding-top: 12px;
    padding-bottom: 12px;
  } </style>
```

我们将在全局 CSS 文件中添加一个`login-container`类，以便该页面的页脚正确对齐。我们还将添加一个 CSS 规则，以确保文本输入在 iPhone 上正确显示。登录页面将是我们唯一需要文本输入的地方，但为了以防以后决定添加其他表单，让我们将其作为全局规则添加。

`resources/assets/css/style.css`:

```php
...

.login-container { margin: 0 auto; padding: 0 12px;
} @media (min-width: 374px) {
  .login-container { width: 350px;
  }
} input[type=text] {
  -webkit-appearance: none;
}
```

最后，让我们将这个新的页面组件添加到我们的路由器中。我们首先导入组件，然后将其添加到路由器配置中的`routes`数组中。

请注意，登录页面不需要来自服务器的任何数据，就像 Vuebnb 的其他页面一样。这意味着我们可以通过修改导航守卫中第一个`if`语句的逻辑来跳过数据获取步骤。如果路由的名称是`login`，它现在应该立即解析。

`resources/assets/js/router.js`:

```php
...

import LoginPage from '../components/LoginPage.vue';

let router = new VueRouter({
  ... routes: [
    ...
    { path: '/login', component: LoginPage, name: 'login' }
  ],
  ...
}); router.beforeEach((to, from, next) => {
  ...
  if ( to.name === 'listing'
      ? store.getters.getListing(to.params.listing)
      : store.state.listing_summaries.length > 0
    || to.name === 'login'
  ) {
    next();
  }
  ...
});

export default router;
```

# 服务器路由

现在我们在`/login`路由添加了一个登录页面，我们需要创建一个匹配的服务器端路由。我们还需要一个用于提交到相同`/login`路径的登录表单的路由。

实际上，这两个路由都是 Laravel 的默认登录系统提供的。要激活这些路由，我们只需在我们的 web 路由文件的底部添加以下行。

`routes/web.php`:

```php
... Auth::routes();
```

要查看此代码的效果，我们可以使用 Artisan 来显示应用程序中的路由列表：

```php
$ php artisan route:list 
```

输出：

![](img/a6f43ce1-9bb9-41e4-bf13-9f4f04b0ae0a.png)图 9.2\. 终端输出显示路由列表

您将看到我们手动创建的所有路由，以及一些我们没有创建的路由，例如*登录*、*注销*和*注册*。这些是 Laravel 身份验证系统使用的路由，我们刚刚激活了它们。

查看 GET/HEAD `/login`路由，您将看到它指向`LoginController`控制器。让我们来看看那个文件。

`App\Http\Controllers\Auth\LoginController.php`:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;

class LoginController extends Controller
{
  use AuthenticatesUsers;

  protected $redirectTo = '/home';

  public function __construct()
  {
    $this->middleware('guest')->except('logout');
  }
}
```

这个类使用了一个`AuthenticatesUsers`特性，定义了`showLoginForm`方法，`/login`路由处理程序引用了这个方法。让我们重写该方法，使其简单地返回我们的应用视图。由于这个视图实例不需要在头部内联任何数据（登录表单没有状态），我们将向`data`模板变量传递一个空数组。

`App\Http\Controllers\Auth\LoginController.php`:

```php
class LoginController extends Controller
{
  ...

  public function showLoginForm()
  {
    return view('app', ['data' => []]);
  }
}
```

完成后，通过将浏览器导航到`/login`，我们现在可以看到完整的登录页面：

![](img/bb73b6fd-67d9-479c-82da-240cb763fd71.png)图 9.3\. 登录页面

# CSRF 保护

CSRF（跨站请求伪造）是一种恶意利用，攻击者让用户在当前登录的服务器上执行一个不知情的操作。这个操作将改变服务器上对攻击者有利的东西，例如转账、更改攻击者知道的密码等。

例如，攻击者可能会在网页或电子邮件中隐藏一个脚本，并以某种方式引导用户访问它。当执行时，此脚本可以向`importantwebsite.com/updateEmailAndPassword`发出 POST 请求。如果用户已登录到此站点，则请求可能成功。

防止这种攻击的一种方法是在用户可能提交的任何表单中嵌入一个特殊令牌，实质上是一个随机字符串。当提交表单时，检查令牌是否与用户的会话匹配。攻击者将无法伪造此令牌，并因此受到此功能的阻碍。

在 Laravel 中，CSRF 令牌的创建和验证由默认添加到 web 路由的`VerifyCsrfToken`中间件管理：

![](img/48ea4473-071f-4ed7-bafe-d910cdeec8e5.png)图 9.4 CSRF 预防过程

要在表单中包含 CSRF 令牌，只需在`form`标记中添加`{{ csrf_field() }}`。这将生成一个包含有效 CSRF 令牌的隐藏输入字段，例如：

```php
<input type="hidden" name="_token" value="3B08L3fj...">
```

然而，在我们的情况下，这不起作用，因为我们的表单不在 Blade 视图中，而是在一个不会被 Blade 处理的单文件组件中。作为替代方案，我们可以将 CSRF 令牌添加到页面的头部，并将其分配给`window`对象。

`resources/views/app.blade.php`:

```php
<script type="text/javascript"> window.vuebnb_server_data = "{!! addslashes(json_encode($data)) !!}" window.csrf_token = "{{ csrf_token() }}" </script>
```

现在我们可以从 Vue.js 应用程序中检索到这个，并手动将其添加到登录表单中。让我们修改`LoginPage`，在表单中包含一个隐藏的`input`字段。我们现在将一些状态添加到这个组件中，其中令牌被包含为数据属性并绑定到隐藏字段中。

`resources/assets/js/components/LoginPage.vue`:

```php
<template>
  <div id="login" class="login-container">
    <form role="form" method="POST" action="/login">
      <input type="hidden" name="_token" :value="csrf_token"> ... </form>
  </div>
</template>
<script> export default {
    data() {
      return { csrf_token: window.csrf_token }
    }
  } </script>
<style>...</style>
```

如果我们现在尝试使用我们在 seeder 中创建的用户的凭据登录到我们的应用程序，我们将收到此错误页面。查看地址栏，您会看到我们所在的路由是`/home`，这不是我们应用程序中的有效路由，因此会出现`NotFoundHttpException`：

![](img/99d47c43-15ff-4837-b68a-36b6f618ced3.png)图 9.5 无效路由

# 登录后重定向

当用户登录时，Laravel 会将他们重定向到登录控制器中`$redirectTo`属性定义的页面。让我们将其从`/home`更改为`/`。

`app/Http/Auth/Controllers/LoginController.php`:

```php
class LoginController extends Controller
{
  ...

  protected $redirectTo = '/';

  ...
}
```

让我们也更新`RedirectIfAuthenticated`中间件类，以便如果已登录用户尝试查看登录页面，则将其重定向到`/`（而不是默认的`/home`值）。

`app/Http/Middleware/RedirectIfAuthenticated.php`:

```php
...

if (Auth::guard($guard)->check()) {
  return redirect('/');
}
```

完成这些步骤后，我们的登录流程现在将正常工作。

# 在工具栏中添加身份验证链接

现在让我们在工具栏中添加登录和注销链接，以便 Vuebnb 用户可以轻松访问这些功能。

登录链接只是一个指向`login`路由的`RouterLink`。

登出链接更有趣：我们捕获此链接的点击事件，并触发隐藏表单的提交。此表单向`/logout`服务器路由发送 POST 请求，将用户注销并将其重定向回主页。请注意，为使此工作，我们必须将 CSRF 令牌作为隐藏输入包含在内。

`resources/assets/components/App.vue`:

```php
<template>
  ...
  <ul class="links">
    <li>
      <router-link :to="{ name: 'saved' }"> Saved </router-link>
    </li>
    <li>
      <router-link :to="{ name: 'login' }"> Log In </router-link>
    </li>
    <li>
      <a @click="logout">Log Out</a>
      <form 
 style="display: hidden" 
 action="/logout" 
 method="POST" 
 id="logout" >
        <input type="hidden" name="_token" :value="csrf_token"/>
      </form>
    </li>
  </ul>
  ...
</template>
<script>
  ...

  export default { components: { ... },
    data() {
      return { csrf_token: window.csrf_token }
    }, methods: {
      logout() { document.getElementById('logout').submit();
      }
    }
  }
</script>
```

# 保护保存的路由

我们现在可以使用我们的登录系统来保护某些路由免受未经身份验证的用户的访问。Laravel 提供了`auth`中间件，可以应用于任何路由，并且如果访客用户尝试访问它，将会将其重定向到登录页面。让我们将其应用于我们保存的页面路由。

`routes/web.php`:

```php
Route::get('/saved', 'ListingController@get_home_web')->middleware('auth');
```

如果您从应用程序注销并尝试从浏览器的导航栏访问此路由，您会发现它会将您重定向回`/login`。

# 将身份验证状态传递给前端

我们现在有了一个完整的登录和注销 Vuebnb 的机制。然而，前端应用程序还不知道用户的身份验证状态。让我们现在解决这个问题，这样我们就可以向前端添加基于身份验证的功能。

# auth 元属性

我们将首先将身份验证状态添加到我们通过每个页面头部传递的元信息中。我们将利用`Auth`外观的`check`方法，如果用户已经验证，它将返回`true`，并将其分配给一个新的`auth`属性。

`app/Http/Controllers/ListingController.php`:

```php
...
use Illuminate\Support\Facades\Auth;

class ListingController extends Controller
{
  ...

  private function add_meta_data($collection, $request)
  {
    return $collection->merge([
      'path' => $request->getPathInfo(),
      'auth' => Auth::check()
    ]);
  }
}
```

我们还将在我们的 Vuex 存储中添加一个`auth`属性。我们将从`addData`方法中对其进行变化，正如您从上一章中记得的那样，这是我们从文档头部或 API 中检索数据的地方。由于 API 不包括元数据，我们将有条件地改变`auth`属性，以避免访问可能未定义的对象属性。

`resources/assets/js/store.js`:

```php
...

export default new Vuex.Store({ state: {
    ... auth: false
  }, mutations: {
    ...
    addData(state, { route, data }) {
      if (data.auth) { state.auth = data.auth;
      }
      if (route === 'listing') { state.listings.push(data.listing);
      } else { state.listing_summaries = data.listings;
      }
    }
  }, getters: { ... }
});
```

现在，Vuex 已经在跟踪用户的身份验证状态。一定要通过登录和注销来测试这一点，并注意 Vue Devtools 的 Vuex 选项卡中的`auth`的值：

![](img/a51dc23b-ea82-4394-b040-bc876b474174.png)图 9.6。Vue Devtools 中`auth`的值

# 响应身份验证状态

现在我们正在跟踪用户的身份验证状态，我们可以让 Vuebnb 对其做出响应。首先，让我们使用户在未登录时无法保存列表。为此，我们将修改`toggleSaved`变化器方法的行为，以便如果用户已登录，则可以保存项目，但如果没有，则通过 Vue Router 的`push`方法重定向到登录页面。

请注意，我们将不得不在文件顶部导入我们的路由模块，以便访问其功能。

`resources/assets/js/store.js`:

```php
...
import router from './router';

export default new Vuex.Store({
  ... mutations: {
    toggleSaved(state, id) {
      if (state.auth) {
        let index = state.saved.findIndex(saved => saved === id);
        if (index === -1) { state.saved.push(id);
        } else { state.saved.splice(index, 1);
        }
      } else { router.push('/login');
      }
    },
    ...    
  },
  ...
});
```

我们还将使工具栏中显示登录链接或注销链接，而不会同时显示两者。这可以通过工具栏中依赖于`$store.state.auth`值的`v-if`和`v-else`指令来实现。

除非用户已登录，否则隐藏保存页面链接也是有道理的，因此我们也要这样做。

`resources/assets/components/App.vue`:

```php
<ul class="links">
  <li v-if="$store.state.auth">
    <router-link :to="{ name: 'saved' }"> Saved </router-link>
  </li>
  <li v-if="$store.state.auth">
    <a @click="logout">Log Out</a>
    <form style="display: hidden" 
      action="/logout"  method="POST" 
      id="logout" >
      <input type="hidden" name="_token" :value="csrf_token"/>
    </form>
  </li>
  <li v-else>
    <router-link :to="{ name: 'login' }"> Log In </router-link>
  </li>
</ul> 
```

现在，工具栏的外观将取决于用户是否已登录或注销：

![](img/0c0883d4-739c-4010-a983-62e5d8b00ef4.png)图 9.8。工具栏中已登录和已注销状态的比较

# 从数据库中检索保存的项目

现在让我们开始从数据库中检索保存的项目并在前端显示它们。首先，我们将在文档头部放置的元数据中添加一个新的`saved`属性。如果用户已注销，这将是一个空数组，或者如果他们已登录，则是与该用户关联的保存列表 ID 数组。

`app/Http/Controllers/ListingController.php`:

```php
private function add_meta_data($collection, $request)
{
  return $collection->merge([
    'path' => $request->getPathInfo(), 
    'auth' => Auth::check(), 
    'saved' => Auth::check() ? Auth::user()->saved : []
  ]);
}
```

在前端，我们将把检索保存项目的逻辑放在`beforeEach`路由导航守卫中。我们将其放在这里而不是在`addData`变化中的原因是，我们不希望直接将数据分配给存储状态，而是对每个列表调用`toggleSaved`变化。您不能从另一个变化中提交变化，因此必须在存储之外完成此操作。

`resources/assets/js/router.js`:

```php
router.beforeEach((to, from, next) => {
  let serverData = JSON.parse(window.vuebnb_server_data);
  if ( ... ) { ... }
  else if ( ... ) { ... }
  else { store.commit('addData', {route: to.name, data: serverData}); serverData.saved.forEach(id => store.commit('toggleSaved', id));
    next();
  }
});
```

让我们还删除我们在上一章中添加到`saved`中的占位符列表 ID，以便存储在初始化时为空。

`resources/assets/js/store.js`:

```php
state: { saved: [], listing_summaries: [], listings: [], auth: false
}
```

完成这些操作后，我们应该发现，如果使用 Vue Devtools 检查，数据库中的保存列表现在与前端中的列表匹配：

```php
$ php artisan tinker >>> DB::table('users')->select('saved')->first();
# "saved": "[1,5,7,9]"
```

![](img/331bfe47-6056-44d8-a730-a77832088b03.png)图 9.8。Vue Devtools 的 Vuex 选项卡显示保存的列表与数据库匹配

# 持久保存列表

持久保存列表的机制如下：当在前端应用中切换列表时，我们触发一个 AJAX 请求，将 ID POST 到后端的一个路由。这个路由调用一个控制器，将更新模型。

![](img/5d82c6ee-a246-4db9-8fbf-4d2056374558.png)图 9.9。持久保存列表

现在让我们实现这个机制。

# 创建 API 路由

我们将从服务器端开始，并为前端添加一个路由来 POST listing IDS。我们需要添加`auth`中间件，以便只有经过身份验证的用户才能访问这个路由（我们将很快讨论`:api`的含义）。

`routes/api.php`:

```php
...

Route::post('/user/toggle_saved', 'UserController@toggle_saved') ->middleware('auth:api') ;
```

由于这是一个 API 路由，它的完整路径将是`/api/user/toggle_saved`。我们还没有创建这个路由调用的控制器`UserController`，所以现在让我们来做这个。

```php
$ php artisan make:controller UserController
```

在这个新的控制器中，我们将添加`toggled_saved`处理方法。由于这是一个 HTTP POST 路由，这个方法将可以访问表单数据。我们将使前端对这个路由的 AJAX 调用包含一个`id`字段，这将是我们想要切换的 listing 的 ID。要访问这个字段，我们可以使用`Input`外观，即`Input::get('id');`。

由于我们在这个路由上使用了`auth`中间件，我们可以通过使用`Auth::user()`方法来检索与请求相关联的用户模型。然后我们可以像在我们的 Vuex store 的`toggledSaved`方法中那样，要么添加要么删除用户的`saved`列表中的 ID。

一旦 ID 被切换，我们就可以使用模型的`save`方法将更新持久化到数据库中。

`app/Http/Controllers/UserController.php`:

```php
<?php

...

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Input;

class UserController extends Controller
{
  public function toggle_saved()
  {
    $id = Input::get('id');
    $user = Auth::user();
    $saved = $user->saved;
    $key = array_search($id, $saved);
    if ($key === FALSE) {
        array_push($saved, $id);
    } else {
        array_splice($saved, $key, 1);
    }
    $user->saved = $saved;
    $user->save();
    return response()->json();
  }
}
```

# Vuex actions

在第八章中，*使用 Vuex 管理应用程序状态*，我们讨论了 Flux 模式的关键原则，包括 mutations 必须是同步的，以避免使我们的应用程序数据不可预测的竞争条件。

如果您需要在一个 mutator 方法中包含异步代码，您应该创建一个*action*。Actions 类似于 mutations，但是不是直接改变状态，而是提交 mutations。例如：

```php
var store = new Vuex.Store({ state: { val: null  
  }, mutations: {
    assignVal(state, payload) { state.val = payload;
    }  
  }, actions: {
    setTimeout(() => {
      commit('assignVal', 10);
    }, 1000)
  }
}); store.dispatch('assignVal', 10);
```

通过将异步代码抽象成 actions，我们仍然可以将任何改变状态的逻辑集中在 store 中，而不会通过竞争条件来污染我们的应用程序数据。

# AJAX 请求

现在让我们使用 AJAX 来发起对`/api/user/toggle_saved`的请求当一个 listing 被保存时。我们将把这个逻辑放到一个 Vuex action 中，这样当 AJAX 调用解析时，`toggleSaved`mutation 就不会被提交。我们将在 store 中导入 Axios HTTP 库来实现这一点。

另外，让我们将认证检查从 mutation 移到 action 中，因为在发起 AJAX 调用之前进行这个检查是有意义的。

`resources/assets/js/store.js`:

```php
import axios from 'axios';

export default new Vuex.Store({
  ... mutations: {
    toggleSaved(state, id) {
      let index = state.saved.findIndex(saved => saved === id);
      if (index === -1) { state.saved.push(id);
      } else { state.saved.splice(index, 1);
      }
    },
    ...
  },
  ... actions: {
    toggleSaved({ commit, state }, id) {
      if (state.auth) { axios.post('/api/user/toggle_saved', { id }).then(
          () => commit('toggleSaved', id)
        );
      } else { router.push('/login');
      }
    }
  }
});
```

现在我们需要从我们的`ListingSave`组件中调用`toggledSaved`action，而不是 mutation。调用一个 action 的方式与 mutation 完全相同，只是术语从`commit`变为`dispatch`。

`resources/assets/components/ListingSave.vue`:

```php
toggleSaved() {
  this.$store.dispatch('toggleSaved', this.id);
}
```

前端的这个功能代码是正确的，但是如果我们测试并尝试保存一个项目，我们会从服务器得到一个*401 未认证*的错误：

![](img/41cc0829-043d-44ae-ac6c-f008896dd103.png)图 9.10. AJAX 调用导致 401 未认证错误

# API 认证

我们在`/api/user/toggle_saved`路由中添加了`auth`中间件，以保护它免受访客用户的攻击。我们还为这个中间件指定了`api`守卫，即`auth:api`。

*守卫*定义了用户如何进行认证，并在以下文件中进行配置。

`config/auth.php`:

```php
<?php

return [
  ...
  'guards' => [
    'web' => [
      'driver' => 'session',
      'provider' => 'users',
    ],
    'api' => [
      'driver' => 'token',
      'provider' => 'users',
    ],
  ],
  ...
];
```

我们的 web 路由使用*session*驱动程序，它使用会话 cookie 来维护认证状态。会话驱动程序随 Laravel 一起提供，并且可以直接使用。但是，默认情况下，API 路由使用*token*守卫。我们还没有实现这个驱动程序，因此我们的 AJAX 调用未经授权。

我们也可以在 API 路由中使用会话驱动程序，但这并不推荐，因为会话认证对于 AJAX 请求来说是不够的。相反，我们将使用`passport`守卫，它实现了 OAuth 协议。

您可能会看到`auth`用作`auth:web`的简写，因为 web 守卫是默认的。

# OAuth

OAuth 是一种授权协议，允许第三方应用程序访问服务器上用户的数据，而不暴露其密码。对受保护数据的访问是以特殊令牌的形式给予的，一旦第三方应用程序和用户向服务器确认了身份，该令牌就会被授予。OAuth 的一个典型用例是*社交登录*，例如，当您为自己的网站使用 Facebook 或 Google 登录时。

进行安全的 AJAX 请求的一个挑战是，您不能将任何凭据存储在前端源代码中，因为攻击者可以轻松找到这些凭据。OAuth 的一个简单实现，其中第三方应用实际上是您自己的前端应用，是解决这个问题的一个很好的解决方案。这是我们现在将要采取的方法，用于 Vuebnb。

虽然 OAuth 是 API 身份验证的一个很好的解决方案，但它也是一个我无法在本书中完全涵盖的深入主题。我建议您阅读这篇指南以获得更好的理解：[`www.oauth.com/`](https://www.oauth.com/)。

# Laravel Passport

Laravel Passport 是 OAuth 在 Laravel 应用程序中可以轻松设置的实现。让我们现在安装它以在 Vuebnb 中使用。

首先，使用 Composer 安装 Passport：

```php
$ composer require laravel/passport
```

Passport 包括生成存储 OAuth 令牌所需的表的新数据库迁移。让我们运行迁移：

```php
$ php artisan migrate
```

以下命令将安装生成安全令牌所需的加密密钥：

```php
$ php artisan passport:install
```

运行此命令后，将`Laravel\Passport\HasApiTokens`特性添加到用户模型。

`app/User.php`：

```php
<?php

...
 use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
  use HasApiTokens, Notifiable;

  ...
}
```

最后，在`config/auth.php`配置文件中，让我们将 API 守卫的驱动程序选项设置为`passport`。这确保`auth`中间件将使用 Passport 作为 API 路由的守卫。

`config/auth.php`：

```php
'guards' => [
  'web' => [
    'driver' => 'session',
    'provider' => 'users',
  ],

  'api' => [
    'driver' => 'passport',
    'provider' => 'users',
  ],
],
```

# 附加令牌

OAuth 要求在用户登录时将访问令牌发送到前端应用程序。Passport 包括一个中间件，可以为您处理这个问题。将`CreateFreshApiToken`中间件添加到 web 中间件组，`laravel_token` cookie 将附加到出站响应。

`app/Http/Kernel.php`：

```php
protected $middlewareGroups = [
  'web' => [
    ... \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
  ],
  ...
```

对于出站请求，我们需要在 AJAX 调用中添加一些标头。我们可以使 Axios 默认自动附加这些。`'X-Requested-With': 'XMLHttpRequest'`确保 Laravel 知道请求来自 AJAX，而`'X-CSRF-TOKEN': window.csrf_token`附加 CSRF 令牌。

`resources/assets/js/store.js`：

```php
... axios.defaults.headers.common = {
  'X-Requested-With': 'XMLHttpRequest',
  'X-CSRF-TOKEN': window.csrf_token
};

export default new Vuex.Store({
  ... });
```

完成后，我们的 API 请求现在应该得到适当的身份验证。为了测试这一点，让我们使用 Tinker 来查看我们为我们的第一个种子用户保存了哪些项目：

```php
$ php artisan tinker >>> DB::table('users')->select('saved')->first();

# "saved": "[1,5,7,9]"
```

确保您以该用户的身份登录并在浏览器中加载 Vuebnb。切换一些已保存的列表选择并重新运行上面的查询。您应该发现数据库现在正在持久保存已保存的列表 ID。

# 摘要

在本章中，我们学习了关于全栈 Vue/Laravel 应用程序中的身份验证，包括基于会话的 Web 路由身份验证，以及使用 Laravel Passport 的 API 路由的基于令牌的身份验证。

我们利用这些知识为 Vuebnb 设置了登录系统，并允许将保存的房间列表持久保存到数据库中。

在这个过程中，我们还学习了如何利用 CSRF 令牌来保护表单，以及关于 Vuex 操作用于向存储添加异步代码的知识。

在下一章，也是最后一章中，我们将学习如何通过将 Vuebnb 部署到免费的 Heroku PHP 服务器来将全栈 Vue 和 Laravel 应用程序部署到生产环境。我们还将开始从免费 CDN 提供图像和其他静态内容。
