# 第七章：改进 RESTful Web 服务

在上一章中，我们在 Lumen 中创建了 RESTful web 服务，并且确定了一些缺失的元素或需要改进的地方。在本章中，我们将致力于改进并完成一些缺失的元素。我们将改进某些元素，以满足漏洞和代码质量的要求。

我们将在本章中涵盖以下主题，以改进我们的 RESTful web 服务：

+   Dingo，简化 RESTful API 开发：

+   安装和配置 Dingo API 软件包

+   简化路由

+   API 版本控制

+   速率限制

+   内部请求

+   身份验证和中间件

+   转换器

+   加密的需求：

+   SSL，不同的选项

+   总结

# Dingo，简化 RESTful API 开发

是的，你没听错。我没说 bingo。是 Dingo。实际上，Dingo API 是 Laravel 和 Lumen 的一个软件包，它使开发 RESTful web 服务变得更简单。它提供了许多开箱即用的功能，其中许多是我们在上一章中看到的。这些功能中的许多将使我们现有的代码更好，更容易理解和维护。您可以在[`github.com/dingo/api`](https://github.com/dingo/api)上查看 Dingo API 软件包。

让我们首先安装它，然后在使用它的同时，我们将继续查看其好处和特性。

# 安装和配置

只需通过 composer 安装它：

```php
composer require dingo/api:1.0.x@dev
```

也许您想知道这个`@dev`是什么。因此，这是 Dingo 文档中的内容：

目前，该软件包仍处于开发阶段，因此没有稳定版本。您可能需要将最低稳定性设置为 dev。

如果您仍然不确定为什么需要设置最低稳定性，那是因为每个软件包的默认最低稳定性设置为`stable`。因此，如果您依赖于`dev`软件包，则应明确指定，否则它可能不会安装，因为最低稳定性与软件包的实际稳定性状态不匹配。

安装后，您需要注册它。转到`bootstrap/app.php`并在`return $app;`之前的某个地方放入此语句：

```php
$app->register(Dingo\Api\Provider\LumenServiceProvider::class);
```

完成后，您需要在`.env`文件的末尾设置一些变量。在`.env`文件的末尾添加它们：

```php
API_PREFIX=api
API_VERSION=v1
API_DEBUG=true
API_NAME="Blog API"
API_DEFAULT_FORMAT=json
```

配置是不言自明的。现在，让我们继续。

# 简化路由

如果您查看我们放置路由的`routes/web.php`文件，您会发现我们为帖子和评论端点编写了 54 行代码。使用 Dingo API，我们可以用只有 10 行代码来替换这 54 行代码，而且它会更加简洁。所以让我们这样做。您的`routes/web.php`文件应该如下所示：

```php
<?php   /* |---------------------------------------------------------------- | Application Routes |---------------------------------------------------------------- | | Here is where you can register all of the routes for an application. | It is a breeze. Simply tell Lumen the URIs it should respond to | and give it the Closure to call when that URI is requested. | */ $api = app('Dingo\Api\Routing\Router');

$api->version('v1', function ($api) {

    $api->resource('posts', "App\Http\Controllers\PostController");
    $api->resource('comments', "App\Http\Controllers\CommentController", [
        'except' => ['store', 'index']
    ]); 
 $api->post('posts/{id}/comments', 'App\Http\Controllers\CommentController@store');

 $api->get('posts/{id}/comments', 'App\Http\Controllers\CommentController@index'); **});** $app->get('/', function () use ($app) {
  return $app->version(); });  
```

如您所见，我们刚刚在`$api`中得到了路由的对象。但是，这是 Dingo API 路由，而不是 Lumen 的默认路由。正如您所见，它具有我们想要的`resource()`方法，并且我们可以在`except`数组中提及不需要的方法。因此，总的来说，我们的路由现在变得非常简化。

要查看应用程序的确切路由，请运行以下命令：

```php
php artisan route:list
```

# API 版本控制

也许您已经注意到，在我们的路由文件中的上一个代码示例中，我们已经将 API 版本指定为`v1`。这是因为 API 版本控制很重要，而 Dingo 为我们提供了这样的功能。它有助于从不同版本提供不同的端点。您可以有另一个版本组，并且可以提供相同端点的不同内容。如果在不同版本下有相同的端点，则它将选择在您的`.env`文件中提到的版本。

但是，在 URI 中包含版本号会更好。为此，我们可以简单地使用以下前缀：

```php
$api->version('v1', ['prefix' => 'api/v1'], function ($api) {

     $api->resource('posts', "App\Http\Controllers\PostController");
     $api->resource('comments', "App\Http\Controllers\CommentController", [
 'except' => ['store', 'index']
 ]);

     $api->post('posts/{id}/comments', 'App\Http\Controllers\CommentController@store');

     $api->get('posts/{id}/comments', 'App\Http\Controllers\CommentController@index');
});
```

现在，我们的路由将在 URI 中包含版本信息。这是一种推荐的方法。因为如果有人正在使用版本 1，并且我们将在版本 2 中进行更改，那么使用版本 1 的客户端在其请求中明确指定版本号时将不受影响。因此，我们的端点 URL 将如下所示：

```php
http://localhost:8000/api/v1/posts
http://localhost:8000/api/v1/posts/1
http://localhost:8000/api/v1/posts/1/comments
```

但是，请注意，如果我们的 URI 和路由中有版本，那么最好在我们的控制器中实际应用该版本。如果没有，版本实现将非常有限。为此，我们应该为控制器设置基于版本的命名空间。在我们的控制器（`PostController`和`CommentController`）中，命名空间将更改为以下代码行：

```php
namespace App\Http\Controllers**\V1**;
```

现在，控制器目录结构也应该与命名空间匹配。因此，让我们在`Controllers`目录中创建一个名为`V1`的目录，并将我们的控制器移动到`app\Http\Controllers**\V1**`目录中。当我们有下一个版本时，我们将在`app\Http\Controllers`中创建另一个名为`V2`的目录，并在其中添加新的控制器。这也将导致一个新的命名空间`App\Http\Controllers**\V2**`。随着命名空间和目录的更改，`routes/web.php`中的控制器路径也需要相应地进行更改。

通过这个改变，你很可能会看到以下错误：

```php
Class 'App\Http\Controllers\V1\Controller' not found
```

因此，要么将`Controller.php`从 controllers 目录移动到`V1`目录，要么像这样使用完整的命名空间访问它：`\App\Http\Controllers\Controller`

```php
class PostController extends **\App\Http\Controllers\Controller**  {..
```

这取决于你如何做。

# 速率限制

速率限制也被称为节流。这意味着应该限制特定客户端在特定时间间隔内能够访问 API 端点的次数。要启用它，我们必须启用`api.throttling`中间件。您可以在所有路由或特定路由上应用节流。您只需在特定路由上应用中间件，如下所示。在我们的情况下，我们希望为所有端点启用它，所以让我们将其放在一个版本组中：

```php
$api->version('v1', ['middleware' => 'api.throttle','prefix' => 'api/v1']**,** function ($api) {   $api->resource('posts', "App\Http\Controllers\V1\PostController");
  $api->resource('comments', "App\Http\Controllers\V1\CommentController", [
  'except' => ['store', 'index']
 ]);  $api->post('posts/{id}/comments', 'App\Http\V1\Controllers\CommentController@store'); $api->get('posts/{id}/comments', 'App\Http\V1\Controllers\CommentController@index');  });
```

为了简单起见，让我们在路由中进行一些更改。我们可以在版本组中使用命名空间，而不是在每个控制器的名称中指定命名空间，如下所示：

`$api->version('v1', ['middleware' => 'api.throttle', 'prefix' => 'api/v1', **namespace => "App\HttpControllers\V1"** **]**`

现在，我们可以简单地从控制器路径中删除它。

您还可以在分钟内提及限制和时间间隔：

```php
$api->version('v1', [
  'middleware' => 'api.throttle',
 'limit' => 100,
    'expires' => 5**,**
  'prefix' => 'api/v1',
  'namespace' => 'App\Http\Controllers\V1'
  ], function ($api) {
  $api->resource('posts', "PostController");
  $api->resource('comments', "CommentController", [
  'except' => ['store', 'index']
 ]);    $api->post('posts/{id}/comments', 'CommentController@store');
  $api->get('posts/{id}/comments', 'CommentController@index');   });
```

在这里，`expires`是时间间隔，而`limit`是路由可以被访问的次数。

# 内部请求

我们大多数情况下是制作一个 API，由外部客户端作为 Web 服务访问，而不是从同一应用程序访问。但是，有时我们处于需要在同一应用程序内进行内部请求并希望以与返回给外部客户端相同的格式返回数据的情况。

假设现在您希望从 API 中的`PostController`获取评论数据，因为它返回一个响应而不是内部函数调用。我们希望在 Postman 或其他客户端访问时，获得与`/api/posts/{postId}/comments`端点返回的相同数据。在这种情况下，Dingo API 包可以帮助我们。以下是它的简单用法：

```php
use  Illuminate\Http\Request; use Illuminate\Http\Response; use Illuminate\Http\JsonResponse; use **Dingo\Api\Routing\Helpers;**   class PostController extends Controller {
 use **Helpers;**    public function __construct(\App\Post $post)
 {  $this->post = $post;
 }
.... /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */ public function show($id) {
 $comments = $this->api->get("api/posts/$id/comments"**);**    $post = $this->post->find($id); ....
 }
....
} 
```

在前面的代码片段中，加粗的语句是不同的，它有助于进行内部请求。正如您所看到的，我们已经对端点进行了`GET`请求：

```php
 $comments = $this->api->get("api/posts/$id/comments");
```

我们也可以通过使用不同的方法使其成为基于`POST`的请求，如下所示：

```php
$this->api->post("api/v1/posts/$id/comments", ['comment' => 'a nice post']);
```

# 响应

Dingo API 包为不同类型的响应提供了更多的支持。由于我们不会详细介绍 Dingo API 提供的每一件事情，您可以在其文档中查看：[`github.com/dingo/api/wiki/Responses`](https://github.com/dingo/api/wiki/Responses)。

然而，在本章的后面，我们将详细了解响应和格式。

我们将在其他方面使用 Dingo API 包，但现在让我们转向其他概念，我们将继续同时使用 Dingo API 包。

# 身份验证和中间件

我们已经多次讨论过，对于 RESTful Web 服务，会话是通过存储在客户端的身份验证令牌来维护的。因此，服务器可以查找身份验证令牌，并且可以在服务器上找到用户的会话。

有几种生成令牌的方式。在我们的情况下，我们将使用**JWT**（**JSON Web Tokens**）。如在`jwt.io`上所述：

JSON Web Tokens 是一种开放的、行业标准的 RFC 7519 方法，用于在两个方之间安全地表示声明。

我们不会详细讨论 JWT，因为 JWT 是在两个方之间传输信息的一种方式（在我们的情况下是客户端和服务器），因为 JWT 可以用于许多目的。相反，我们将使用它来进行访问/身份验证令牌以维护无状态会话。因此，我们将坚持从 JWT 中获取我们需要的内容。我们需要它来维护身份验证目的的会话，并且这也是 Dingo API 包将帮助我们解决的问题。

事实上，Dingo API 在撰写本书时支持三种身份验证提供者。默认情况下启用了 HTTP 基本身份验证。另外两种是 JWT Auth 和 OAuth 2.0。我们将使用 JWT Auth。

# JWT Auth 设置

Dingo API 用于集成 JWT 身份验证的包可以在[`github.com/tymondesigns/jwt-auth`](https://github.com/tymondesigns/jwt-auth)找到。

[﻿](https://github.com/tymondesigns/jwt-auth)

有两种设置 JWT Auth 的方式：

1.  我们可以简单地按照 JWT Auth 包的说明进行配置，并手动修复一个接一个的问题。

1.  我们可以简单地安装另一个包，帮助我们安装和设置 Dingo API 和 JWT Auth，并进行一些基本配置。

在这里，我们将看到两种方式。然而，使用手动方式可能会因不同包的不同版本和 Lumen 本身而变得模糊。因此，尽管我将展示手动方式，但我建议您使用集成包，这样您就不需要手动处理每个低级别的事情。我将向您展示手动方式，只是为了让您对底层包含的内容有一些了解。

# 手动方式

让我们按照[`github.com/tymondesigns/jwt-auth/wiki/Installation`](https://github.com/tymondesigns/jwt-auth/wiki/Installation)安装页面上提到的包进行安装。

首先，我们需要安装 JWT Auth 包：

```php
 composer require tymon/jwt-auth 1.0.0-beta.3
```

请注意，此版本适用于 Laravel 5.3。对于旧版本，您可能需要使用不同版本的 JWT Auth 包，很可能是版本 0.5。

要在`bootstrap/app.php`文件中注册服务提供者，请添加以下代码行：

```php
$app->register(Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class);
```

然后，在同一个`bootstrap/app.php`文件中添加这两个类别名：

```php
class_alias('Tymon\JWTAuth\Facades\JWTAuth', 'JWTAuth'); class_alias('Tymon\JWTAuth\Facades\JWTFactory', 'JWTFactory');
```

然后，您需要生成一个用于签署我们令牌的随机密钥。要这样做，请运行此命令：

```php
php artisan jwt:generate
```

这将生成一个随机密钥。

您可能会看到一些错误，如下所示：

`[Illuminate\Contracts\Container\BindingResolutionException] Unresolvable dependency resolving [Parameter #0 [ <required> $app ]] in class Illuminate\Cache\CacheManager`

在这种情况下，在`bootstrap/app.php`中的`$app->withEloquent();`之后添加以下行。这样就可以解决问题，您可以尝试生成一个随机密钥：

```php
$app->alias('cache', 'Illuminate\Cache\CacheManager'); $app->alias('auth', 'Illuminate\Auth\AuthManager');
```

然而，您可能想知道这个随机密钥将在哪里设置。实际上，有些包并不是为 Lumen 构建的，而是需要更像 Laravel 的结构。`tymondesigns/jwt-auth`包就是其中之一。它需要一种发布配置的方式。虽然 Lumen 没有为不同的包单独的配置文件，但我们需要它，我们可以让 Lumen 为这个包拥有这样一个`config`文件。要这样做，如果您在`app/`目录下没有`helpers.php`，那么创建它并添加以下内容：

```php
<?php if ( ! function_exists('config_path')) {
  /**
 * Get the configuration path. * * @param string $path
 * @return string
 */  function config_path($path = '')
 {  return app()->basePath() . '/config' . ($path ? '/' . $path : $path);
 } }
```

然后，在自动加载数组的`composer.json`中添加`helpers.php`：

```php
"autoload": {
  "psr-4": {
    "App\\": "app/"
  },
  "files": [
    "app/helpers.php"
  ]
},
```

运行以下命令：

```php
composer dump-autoload
```

一旦你拥有了它，运行以下命令：

```php
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"
```

在这一点上，您将收到另一个错误消息：

```php
[Symfony\Component\Console\Exception\CommandNotFoundException]
 There are no commands defined in the "vendor" namespace.
```

不用担心，这是正常的。这是因为 Lumen 没有`vendor:publish`命令。所以，我们需要安装一个小包来执行这个命令：

```php
composer require laravelista/lumen-vendor-publish
```

由于这个命令将有一个新的命令，为了使用该命令，我们需要在`app/Console/Kernel.php`的`$commands`数组中放入以下内容。

现在，尝试再次运行相同的命令，如下所示：

```php
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"
```

这一次，你会看到类似这样的东西：

```php
Copied File [/vendor/tymon/jwt-auth/src/config/config.php] To [/config/jwt.php]
Publishing complete for tag []!
```

现在，我们有`blog/config/jwt.php`，我们可以在这个文件中存储与`jwt-auth`包相关的配置。

我们需要做的第一件事是重新运行这个命令来设置随机密钥签名：

```php
php artisan jwt:generate
```

这一次，你可以在`config/jwt.php`文件的返回数组中看到这个密钥设置：

```php
'secret' => env('JWT_SECRET', 'RusJ3fiLAp4DmUNNzqGpC7IcQI8bfar7'),
```

接下来，你需要按照[`github.com/tymondesigns/jwt-auth/wiki/Configuration`](https://github.com/tymondesigns/jwt-auth/wiki/Configuration)中所示的进行配置。

然而，你也可以将`config/jwt.php`中的其他设置保持为默认。

接下来要做的事情是告诉 Dingo API 使用 JWT 作为认证方法。所以在`bootstrap/app.php`中添加这个：

```php
app('Dingo\Api\Auth\Auth')->extend('jwt', function ($app) {
 return new Dingo\Api\Auth\Provider\JWT($app['Tymon\JWTAuth\JWTAuth']);
});
```

根据 JWT Auth 文档，我们大部分配置都已经完成了，但请注意，你可能会遇到与版本相关的小问题。如果你使用的版本早于 Lumen 5.3，那么请注意，基于不同的 Laravel 版本，需要使用不同版本的 JWT Auth。对于版本 5.2，你应该使用 JWT Auth 版本 0.5。所以，如果你在低于 Laravel 5.2 的版本中仍然遇到任何错误，那么请注意，可能是因为版本差异导致的错误，所以你需要在互联网上搜索。

正如你所看到的，为了同时使用两个包来实现一些功能，我们必须花费一些时间进行配置，就像最近的几个步骤建议的那样。即使这样，由于版本差异，仍然有可能出现错误。因此，一个简单的方法是不要手动安装 Dingo API 包和 JWT Auth 包。还有另一个包，安装它将安装 Dingo API 包、Lumen 生成器、CORS（跨域资源共享）支持、JWTAuth，并使其可用，而不需要那么多的配置。现在，让我们来看看。

# 通过 Lumen JWT 认证集成包的更简单方式

更简单的方法是自己安装 Dingo API 包和 JWT Auth，只需安装[`packagist.org/packages/krisanalfa/lumen-dingo-adapter.`](https://packagist.org/packages/krisanalfa/lumen-dingo-adapter)

它将在你的基于 Lumen 的应用程序中添加 Dingo 和 JWT。只需安装这个包：

```php
composer require krisanalfa/lumen-dingo-adapter
```

然后，在`bootstrap/app.php`中，添加以下代码行：

```php
$app->register(Zeek\LumenDingoAdapter\Providers\LumenDingoAdapterServiceProvider::class);
```

现在，我们正在使用`LumenDingoAdapter`包，所以这是我们将使用的`bootstrap/app.php`文件，这样你就可以与你的文件进行比较：

```php
<?php   require_once __DIR__.'/../vendor/autoload.php';   try {
 (new Dotenv\Dotenv(__DIR__.'/../'))->load(); } catch (Dotenv\Exception\InvalidPathException $e) {
  // }   /* |-------------------------------------------------------------------------- | Create The Application |-------------------------------------------------------------------------- | | Here we will load the environment and create the application instance | that serves as the central piece of this framework. We'll use this | application as an "IoC" container and router for this framework. | */   $app = new Laravel\Lumen\Application(
  realpath(__DIR__.'/../') );     $app->withFacades();

 $app**->withEloquent();**   /* |-------------------------------------------------------------------------- | Register Container Bindings |-------------------------------------------------------------------------- | | Now we will register a few bindings in the service container. We will | register the exception handler and the console kernel. You may add | your own bindings here if you like or you can make another file. | */   $app->singleton(
 Illuminate\Contracts\Debug\ExceptionHandler::class,
 App\Exceptions\Handler::class );   $app->singleton(
 Illuminate\Contracts\Console\Kernel::class,
 App\Console\Kernel::class );   $app->register(Zeek\LumenDingoAdapter\Providers\LumenDingoAdapterServiceProvider::class);     /* |-------------------------------------------------------------------------- | Register Middleware |-------------------------------------------------------------------------- | | Next, we will register the middleware with the application. These can | be global middleware that run before and after each request into a | route or middleware that'll be assigned to some specific routes. | */   // $app->middleware([ //    App\Http\Middleware\ExampleMiddleware::class // ]);   // $app->routeMiddleware([ //     'auth' => App\Http\Middleware\Authenticate::class, // ]);   /* |-------------------------------------------------------------------------- | Register Service Providers |-------------------------------------------------------------------------- | | Here we will register all of the application's service providers which | are used to bind services into the container. Service providers are | totally optional, so you are not required to uncomment this line. | */   // $app->register(App\Providers\AppServiceProvider::class); // $app->register(App\Providers\AuthServiceProvider::class); // $app->register(App\Providers\EventServiceProvider::class);   /* |-------------------------------------------------------------------------- | Load The Application Routes |-------------------------------------------------------------------------- | | Next we will include the routes file so that they can all be added to | the application. This will provide all of the URLs the application | can respond to, as well as the controllers that may handle them. | */   $app->group(['namespace' => 'App\Http\Controllers'], function ($app) {
  require __DIR__.'/../routes/web.php'; });   return $app; 
```

如果你想知道`$app->withFacades()`到底是做什么的，那么请注意，这将在应用程序中启用门面。门面是一种设计模式，用于将复杂的事物抽象化，同时提供简化的接口进行交互。在 Lumen 中，正如 Laravel 文档所述：<q>门面为应用程序服务容器中可用的类提供了一个“静态”接口。</q>

使用门面的好处是它提供了易记的语法。我们不会经常使用门面，并且会尽量避免使用它，因为我们更倾向于使用依赖注入。然而，一些包可能会使用门面，所以为了让它们工作，我们已经启用了门面。

# 认证

现在，我们可以使用`api.auth`中间件来保护我们的端点。这个中间件检查用户认证并从 JWT 中获取用户。然而，首先要做的是让用户登录，根据用户信息创建一个令牌，并将签名令牌返回给客户端。

为了使认证工作，我们首先需要创建一个与认证相关的控制器。这个控制器不仅会根据用户登录创建令牌，还会使用户令牌过期并刷新令牌。为了做到这一点，我们可以将这个开源的`AuthController`放在`app/Http/Controllers/Auth/`目录下

[`github.com/Haafiz/REST-API-for-basic-RPG/blob/master/app/Http/Controllers/Auth/AuthController.php.`](https://github.com/Haafiz/REST-API-for-basic-RPG/blob/master/app/Http/Controllers/Auth/AuthController.php)

为了给予信用，我想告诉你，我们使用的`AuthController`版本是[`github.com/krisanalfa/lumen-jwt/blob/develop/app/Http/Controllers/Auth/AuthController.php`](https://github.com/krisanalfa/lumen-jwt/blob/develop/app/Http/Controllers/Auth/AuthController.php)的修改版本。

无论如何，如果你在阅读本书时没有看到`AuthController`在线上可用，这里是`AuthController`的内容：

```php
<?php   namespace App\Http\Controllers\Auth;   use Illuminate\Http\Request; use Illuminate\Http\Response; use Illuminate\Http\JsonResponse; use Tymon\JWTAuth\Facades\JWTAuth; use App\Http\Controllers\Controller; use Tymon\JWTAuth\Exceptions\JWTException; use Illuminate\Http\Exception\HttpResponseException;   class AuthController extends Controller {
  /**
 * Handle a login request to the application. * * @param \Illuminate\Http\Request $request
 * * @return \Illuminate\Http\Response;
 */  public function login(Request $request)
 {  try {
  $this->validateLoginRequest($request);
 } catch (HttpResponseException $e) {
  return $this->onBadRequest();
 }    try {
  // Attempt to verify the credentials and create a token for the user
  if (!$token = JWTAuth::attempt(
  $this->getCredentials($request)
 )) {  return $this->onUnauthorized();
 } } catch (JWTException $e) {
  // Something went wrong whilst attempting to encode the token
  return $this->onJwtGenerationError();
 }    // All good so return the token
  return $this->onAuthorized($token);
 }    /**
 * Validate authentication request. * * @param Request $request
 * @return void
 * @throws HttpResponseException
 */  protected function validateLoginRequest(Request $request)
 {  $this->validate(
  $request, [
  'email' => 'required|email|max:255',
  'password' => 'required',
 ] ); }    /**
 * What response should be returned on bad request. * * @return JsonResponse
 */  protected function onBadRequest()
 {  return new JsonResponse(
 [  'message' => 'invalid_credentials'
  ], Response::HTTP_BAD_REQUEST
  );
 }    /**
 * What response should be returned on invalid credentials. * * @return JsonResponse
 */  protected function onUnauthorized()
 {  return new JsonResponse(
 [  'message' => 'invalid_credentials'
  ], Response::HTTP_UNAUTHORIZED
  );
 }    /**
 * What response should be returned on error while generate JWT. * * @return JsonResponse
 */  protected function onJwtGenerationError()
 {  return new JsonResponse(
 [  'message' => 'could_not_create_token'
  ], Response::HTTP_INTERNAL_SERVER_ERROR
  );
 }    /**
 * What response should be returned on authorized. * * @return JsonResponse
 */  protected function onAuthorized($token)
 {  return new JsonResponse(
 [  'message' => 'token_generated',
  'data' => [
  'token' => $token,
 ] ] ); }    /**
 * Get the needed authorization credentials from the request. * * @param \Illuminate\Http\Request $request
 * * @return array
 */  protected function getCredentials(Request $request)
 {  return $request->only('email', 'password');
 }    /**
 * Invalidate a token. * * @return \Illuminate\Http\Response
 */  public function invalidateToken()
 {  $token = JWTAuth::parseToken();    $token->invalidate();    return new JsonResponse(['message' => 'token_invalidated']);
 }    /**
 * Refresh a token. * * @return \Illuminate\Http\Response
 */  public function refreshToken()
 {  $token = JWTAuth::parseToken();    $newToken = $token->refresh();    return new JsonResponse(
 [  'message' => 'token_refreshed',
  'data' => [
  'token' => $newToken
  ]
 ] ); }    /**
 * Get authenticated user. * * @return \Illuminate\Http\Response
 */  public function getUser()
 {  return new JsonResponse(
 [  'message' => 'authenticated_user',
  'data' => JWTAuth::parseToken()->authenticate()
 ] ); } } 
```

这个控制器有三个主要任务：

+   登录`login()`方法

+   使令牌失效

+   刷新令牌

# 登录

登录是在`login()`方法中完成的，并且它尝试使用`JWTAuth::attempt($this->getCredentials($request))`进行登录。如果凭据无效或者出现其他问题，它将返回一个错误。然而，要访问这个`login()`方法，我们需要为它添加一个路由。这是我们将在`routes/web.php`中添加的内容：

```php
$api->post(
  '/auth/login', [
  'as' => 'api.auth.login',
  'uses' => 'Auth\AuthController@login',
 ] );
```

# 使令牌失效

为了使令牌失效，换句话说，注销用户，将使用`invalidateToken()`方法。这个方法将通过一个路由来调用。我们将添加以下路由，使用删除请求方法，它将从路由文件中调用`AuthController::invalidateToken()`：

```php
$api->delete(
  '/', [
  'uses' => 'Auth/AuthController@invalidateToken',
  'as' => 'api.auth.invalidate'
  ] );
```

# 刷新令牌

当令牌根据到期时间过期时，将调用刷新令牌。为了刷新令牌，我们还需要添加以下路由：

```php
$api->patch(
  '/', [
  'uses' => 'Auth/AuthController@refreshToken',
  'as' => 'api.auth.refresh'
  ] );
```

请注意，所有这些端点都将添加在版本 v1 下。

一旦我们有了`AuthController`并且路由设置好了，用户可以使用以下端点进行登录：

```php
POST /api/v1/auth/login
Params: email, passsword
```

尝试一下，你将获得基于 JWT 的访问令牌。

**Lumen、Dingo、JWT Auth 和 CORS 样板**：

如果你在配置 Lumen 与 Dingo 和 JWT 时遇到困难，那么你可以简单地使用[`github.com/krisanalfa/lumen-jwt.`](https://github.com/Haafiz/lumen-jwt)这个存储库将为你提供使用 Dingo API 和 JWT 设置你的 Lumen 进行 API 开发的样板代码。你可以克隆它并开始使用。这只是一个 Lumen 与 JWT、Dingo API 和 CORS 支持的集成。因此，如果你要开始一个新的 RESTful web 服务项目，你可以直接使用这个样板代码开始。

在继续之前，让我们看一下我们的路由文件，确保我们在同一个页面上：

```php
<?php   /* |-------------------------------------------------------------------------- | Application Routes |-------------------------------------------------------------------------- | | Here is where you can register all of the routes for an application. | It is a breeze. Simply tell Lumen the URIs it should respond to | and give it the Closure to call when that URI is requested. | */ $api = app('Dingo\Api\Routing\Router');     $api->version('v1', [
  'middleware' => ['api.throttle'],
  'limit' => 100,
  'expires' => 5,
  'prefix' => 'api/v1',
  'namespace' => 'App\Http\Controllers\V1' ],
  function ($api) {
 $api->group(['middleware' => 'api.auth'], function ($api) {
            //Posts protected routes
            $api->resource('posts', "PostController", [
                'except' => ['show', 'index']
            ]);

            //Comments protected routes
            $api->resource('comments', "CommentController", [
                'except' => ['show', 'index']
            ]);

            $api->post('posts/{id}/comments', 'CommentController@store'**);**      // Logout user by removing token
  $api->delete(
  '/', [
  'uses' => 'Auth/AuthController@invalidateToken',
  'as' => 'api.Auth.invalidate'
  ]
 );      // Refresh token
  $api->patch(
  '/', [
  'uses' => 'Auth/AuthController@refreshToken',
  'as' => 'api.Auth.refresh'
  ]
 ); **});**  $api->get('posts', 'PostController@index');
 $api->get('posts/{id}', 'PostController@show'**);**

  $api->get('posts/{id}/comments', 'CommentController@index');
 $api->get('comments/{id}', 'CommentController@show'**);**    $api->post(
  '/auth/login', [
  'as' => 'api.Auth.login',
  'uses' => 'Auth\AuthController@login',
 ] );  });    $app->get('/', function () use ($app) {
  return $app->version(); });   
```

正如你所看到的，我们已经创建了一个路由组。路由组只是一种将相似的路由分组的方式，我们可以在其中应用相同的中间件、命名空间或前缀等，就像我们在`v1`组中所做的那样。

在这里，我们创建了另一个路由组，以便我们可以在其中添加`api.auth`中间件。还要注意的一点是，我们已经将一些帖子路由从帖子资源路由中拆分出来，以便在没有登录的情况下有一些可用的路由。我们也对评论路由做了同样的处理。

请注意，如果你不想将一些路由从资源路由中拆分出来，那么你也可以这样做。你只需在控制器中添加`api.auth`中间件，而不是在路由文件中。两种方式都是正确的；这只是一个偏好问题。我之所以这样做，是因为我发现从相同的路由文件而不是不同控制器的构造函数中知道哪些路由受保护更容易。但再次强调，这取决于你。

我们只允许已登录的用户创建、更新和删除帖子。但是，我们需要确保已登录的用户只能更新或删除自己的帖子。虽然这也可以通过创建另一个中间件来实现，但在控制器中实现会更简单。

这就是我们在 `PostController` 中的做法：

```php
<?php   namespace App\Http\Controllers\V1;   use Illuminate\Http\Request; use Illuminate\Http\Response; use Illuminate\Http\JsonResponse; use Tymon\JWTAuth\Facades\JWTAuth; use Dingo\Api\Routing\Helpers;   class PostController extends Controller {
  use Helpers;    public function __construct(\App\Post $post)
 {     $this->post = $post;   }    /**
 * Display a listing of the resource. * * @return \Illuminate\Http\Response
 */  public function index(Request $request)
 {  $posts = $this->post->paginate(20);    return $posts;
 }    /**
 * Store a newly created resource in storage. * * @param \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */  public function store(Request $request)
 {    $input = $request->all();
 $input['user_id'] = $this->user->id**;**    $validationRules = [
  'content' => 'required|min:1',
  'title' => 'required|min:1',
  'status' => 'required|in:draft,published',
  'user_id' => 'required|exists:users,id'
  ];    $validator = \Validator::make($input, $validationRules);
  if ($validator->fails()) {
  return new JsonResponse(
 [  'errors' => $validator->errors()
 ], Response::HTTP_BAD_REQUEST
  );
 }    $this->post->create($input);    return [
  'data' => $input
  ];
 }    /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function show($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    return $post;
 }    /**
 * Update the specified resource in storage. * * @param \Illuminate\Http\Request  $request
 * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function update(Request $request, $id)
 {  $input = $request->all();    $post = $this->post->find($id);    if(!$post) {
 abort(404);
 } if($this->user->id != $post->user_id){
            return new JsonResponse(
                [
                    'errors' => 'Only Post Owner can update it'
                ], Response::HTTP_FORBIDDEN
            ); **}**    $post->fill($input);
  $post->save();    return $post;
 }    /**
 * Remove the specified resource from storage. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function destroy($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 } if($this->user->id != $post->user_id){
            return new JsonResponse(
                [
                    'errors' => 'Only Post Owner can delete it'
                ], Response::HTTP_FORBIDDEN
            ); **}**    $post->delete();    return ['message' => 'deleted successfully', 'post_id' => $id];
 } } 
```

正如您所看到的，我在三个地方都突出显示了代码。在 `store()` 方法中，我们从中获取了用户 ID，并将其放入输入数组中，以便帖子的 `user_id` 基于令牌。同样，在 `update()` 和 `delete()` 中，我们使用了该用户的 ID，并放置了一个检查，以确保帖子所有者正在删除或更新帖子记录。您可能会想知道，当我们没有在任何地方定义 `$this->user` 属性时，我们是如何访问它的？实际上，我们正在使用 Helpers trait，所以 `$this->user` 来自该 trait。

注意，为了访问受保护的资源，您应该从登录端点获取令牌，并将其放在标头中，如下所示：`Authentication: bearer <token grabbed from login>`

同样，`CommentController` 将进行检查，以确保评论修改将仅限于评论所有者，并且删除将仅限于评论或帖子所有者。它将具有类似的检查和用户 ID 通过令牌的方式。因此，我将留给您实现评论控制器以进行这些检查。

# 转换器

在全栈 MVC 框架中，我们有模型视图控制器。在 Lumen 或 API 中，我们没有视图，因为我们只返回一些数据。但是，我们可能希望以与通常不同的方式显示数据。我们可能希望使用不同的格式，或者我们可能希望限制具有特定格式的对象。在所有这些情况下，我们需要一个处理格式相关任务的地方，一个可以包含不同格式相关内容的地方。我们可以在控制器中实现。但是，我们需要在不同的地方定义相同的格式。在这种情况下，我们可以在模型中添加一个方法。例如，帖子模型可以有一种特定的方式来格式化帖子对象。因此，我们可以在帖子模型中定义另一个方法。

它会很好地工作，但是如果你仔细看，它与格式有关，而不是模型。因此，我们有另一层叫做序列化或转换器。有时，我们需要嵌套对象，所以我们不希望一遍又一遍地做同样的嵌套。

Lumen 提供了一种将数据序列化为 JSON 的方法。在 Eloquent 对象中，有一个名为 `toJson()` 的方法；这个方法可以被重写以达到目的。但是，最好有一个单独的层用于格式化和序列化数据，而不是在同一个类中只有一个方法来实现。然后就是转换器；转换器只是另一层。您可以将转换器视为 API 或 web 服务的视图层。

# 理解和设置转换器

实际上，我们使用的包名为 Dingo API 包含了创建 RESTful web 服务所需的许多内容。同样，Dingo API 包还提供了对转换器的支持。

在做任何事情之前，我们需要了解转换器层由转换器组成。转换器是负责数据呈现的类。Dingo API 转换器支持转换器，对于转换器，API 依赖于另一个负责转换器功能的库。由我们决定使用哪个转换层或库。默认情况下，它使用 Fractal，一个默认的转换层。

我们不需要做任何其他与设置相关的事情。让我们开始使用转换器来处理我们的对象。但是，在那之前，让自己熟悉 Fractal。我们至少需要知道 Fractal 是什么以及它提供了什么。Fractal 的文档可以在 [`fractal.thephpleague.com/`](http://fractal.thephpleague.com/) 找到。

# 使用转换器

有两种方法可以告诉 Lumen 要使用哪个转换器类。为此，我们需要创建一个转换器类。让我们首先为我们的`Post`对象创建一个转换器，并将其命名为`PostTransformer`。首先，创建一个名为`app/Transformers`的目录，在该目录中，创建一个名为`PostTransformer`的类，内容如下：

```php
<?php   namespace App\Transformers;   use League\Fractal;   class PostTransformer extends Fractal\TransformerAbstract {   public function transform(\App\Post $post)
 {   return $post->toArray();
 } }
```

您可以在`transform()`方法中对 Post 响应进行任何想要的操作。请注意，我们在这里不是可选地重写`transform()`方法，而是提供了`transform()`的实现。您始终需要在转换器类中添加该方法。但是，如果没有从任何地方使用该类，则该类将毫无用处。因此，让我们在我们的`PostController`中使用它。让我们在`index()`方法中使用它：

```php
public function index(**\App\Transformers\PostTransformer** $postTransformer) {
  $posts = $this->post->paginate(20);   return $this->response->paginator($posts, $postTransformer**);** } 
```

正如您所看到的，我们已经将`PostTransformer`对象注入到`$this->response->paginator()`方法中。这里我们需要注意的第一件事是`$this->response->paginator()`方法和`$this->response`对象。我们现在需要知道`$this->response`对象最初是从哪里来的。我们得到它是因为我们在`PostController`中使用了`Helpers` trait。无论如何，现在让我们看看它是如何工作的。使用以下端点击中`PostController`的`index()`方法：

```php
http://localhost:8000/api/v1/posts
```

它会返回类似这样的东西：

```php
{ "data": [
 { "id": 1,
 "title": "test",
 "status": "draft",
 "content": "test post",
 "user_id": 2,
 "created_at": null,
 "updated_at": "2017-06-28 00:47:50"
 }, {  "id": 3,
  "title": "test",
  "status": "published",
  "content": "test post",
  "user_id": 2,
 "created_at": "2017-06-28 00:00:44",
  "updated_at": "2017-06-28 00:00:44"
  },
 {  "id": 4,
  "title": "test",
  "status": "published",
  "content": "test post",
  "user_id": 2,
  "created_at": "2017-06-28 03:21:36",
  "updated_at": "2017-06-28 03:21:36"
  },
 {  "id": 5,
  "title": "test post",
  "status": "draft",
  "content": "This is yet another post for testing purpose",
  "user_id": 8,
  "created_at": "2017-07-15 00:45:29",
  "updated_at": "2017-07-15 00:45:29"
  },
 {  "id": 6,
  "title": "test post",
  "status": "draft",
  "content": "This is yet another post for testing purpose",
  "user_id": 8,
  "created_at": "2017-07-15 23:53:23",
  "updated_at": "2017-07-15 23:53:23"
  } ], "meta": {
    "pagination": {
        "total": 5,
            "count": 5,
            "per_page": 20,
            "current_page": 1,
            "total_pages": 1,
            "links": []
        }
    }
}
```

如果您仔细观察，您会看到一个单独的元数据部分，其中包含与分页相关的内容。这是 Fractal 转换器本身提供的一个小功能。实际上，Fractal 可以为我们提供更多。

我们可以包含嵌套对象。例如，如果我们在`Post`中有`user_id`，并且我们希望`User`对象嵌套在同一个`Post`对象中，那么它也可以提供更简单的方法来实现。尽管转换器层就像 API 响应数据的视图层一样，但它提供的远不止于此。现在，我将向您展示当我们从`show()`和其他方法中返回`PostTransformer`后，我们的`PostController`方法将会是什么样子。有关 Fractal 的详细信息，我建议您查看 Fractal 文档，以便充分利用它，网址为[`fractal.thephpleague.com/`](http://fractal.thephpleague.com/)。

以下是我们的`PostController`方法的样子：

```php
<?php   namespace App\Http\Controllers\V1;   use Illuminate\Http\Request; use Illuminate\Http\Response; use Illuminate\Http\JsonResponse;  use **Dingo\Api\Routing\Helpers;** use App\Transformers\PostTransformer;   class PostController extends Controller {
 use **Helpers;**    public function __construct(\App\Post $post, \App\Transformers\PostTransformer $postTransformer)
 {   $this->post = $post;    $this->transformer = $postTransformer;   }    /**
 * Display a listing of the resource. * * @return \Illuminate\Http\Response
 */  public function index()
 {  $posts = $this->post->paginate(20);   return $this->response->paginator($posts, $this->transformer**);**
 }    /**
 * Store a newly created resource in storage. * * @param \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */  public function store(Request $request)
 {    $input = $request->all();
  $input['user_id'] = $this->user->id;    $validationRules = [
  'content' => 'required|min:1',
  'title' => 'required|min:1',
  'status' => 'required|in:draft,published',
  'user_id' => 'required|exists:users,id'
  ];    $validator = \Validator::make($input, $validationRules);
  if ($validator->fails()) {
  return new JsonResponse(
 [  'errors' => $validator->errors()
 ], Response::HTTP_BAD_REQUEST
  );
 }   $post = $this->post->create($input);   return $this->response->item($post, $this->transformer**);**
 }    /**
 * Display the specified resource. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function show($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }   return $this->response->item($post, $this->transformer**);**
 }    /**
 * Update the specified resource in storage. * * @param \Illuminate\Http\Request  $request
 * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function update(Request $request, $id)
 {  $input = $request->all();    $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    if($this->user->id != $post->user_id){
  return new JsonResponse(
 [  'errors' => 'Only Post Owner can update it'
  ], Response::HTTP_FORBIDDEN
  );
 }    $post->fill($input);
  $post->save();   return $this->response->item($post, $this->transformer**);**
 }    /**
 * Remove the specified resource from storage. * * @param int  $id
 * @return \Illuminate\Http\Response
 */  public function destroy($id)
 {  $post = $this->post->find($id);    if(!$post) {
 abort(404);
 }    if($this->user->id != $post->user_id){
  return new JsonResponse(
 [  'errors' => 'Only Post Owner can delete it'
  ], Response::HTTP_FORBIDDEN
  );
 }    $post->delete();    return ['message' => 'deleted successfully', 'post_id' => $id];
 } } 
```

从前面的代码片段中突出显示的行中，您可以看到我们在构造函数中添加了`PostTransformer`对象，并将其放置在`$this->transformer`中，我们在其他方法中使用了它。您还可以看到，在一个地方，我们在`index()`方法中使用了`$this->response->paginator()`方法，而在其他方法中我们使用了`$this->response->item()`。这是因为当有一个对象时，我们使用`$this->response->item()`方法，而在`index()`方法中有`paginator`对象时，我们使用`paginator`。请注意，如果您有一个集合并且没有`paginator`对象，则应该使用`$this->response->collection()`。

如前所述，Fractal 具有更多功能，这些功能在其文档中有所介绍。因此，您需要暂停一下，并在[`fractal.thephpleague.com/`](http://fractal.thephpleague.com/)上探索其文档。

# 加密

我们缺少的下一件事是加密客户端和服务器之间的通信，以便没有人可以在网络上嗅探和读取数据。为此，我们将使用**SSL**（**安全套接字层**）。由于本书不涉及加密、密码学或服务器设置，我们不会详细介绍这些概念，但重要的是我们在这里谈论加密。如果有人能够在网络上嗅探数据，那么我们的网站或网络服务就不安全。

为了保护我们的 Web 服务，我们将使用 HTTPS 而不是 HTTP。HTTPS 中的“S”代表安全。现在，问题是我们如何使它安全。你可能会说我们将使用 SSL，就像我们之前说的那样。那么 SSL 是什么？SSL 是安全套接字层，是服务器和浏览器之间安全通信的标准方式。SSL 指的是安全协议。实际上 SSL 协议有三个版本，它们对一些攻击是不安全的。所以我们实际使用的是 TLS（传输层安全）。然而，当我们提到 TLS 时，我们仍然使用 SSL 术语。如果你想使用 SSL 证书和 SSL 来使 HTTP 安全，实际上底层使用的是比原始 SSL 协议更好的 TLS。

当建立连接时，服务器会将 SSL 证书的副本与公钥一起发送给浏览器，以便浏览器也可以对与服务器之间的通信进行编码或解码。我们不会深入讨论加密细节；然而，我们需要知道如何获得 SSL 证书。

# SSL 证书，不同的选项

通常，SSL 证书是从证书提供商那里购买的。然而，你也可以从[letsencrypt.org](http://letsencrypt.org)获得免费证书。所以，如果有免费证书可用，那为什么还要购买证书呢？实际上，有时从某些机构购买证书更多的是为了保险而不是安全。如果你正在建立一个电子商务网站或者接受付款或者非常关键的金融信息等非常重要的数据，那么你需要有人在你的网站用户面前承担责任。

也许从[letsencrypt.org](http://letsencrypt.org)获得的证书与以较低价格出售的提供商的证书之间存在一些微小的差异（我不知道），但通常，购买证书更多的是为了保险而不是安全。

你将从你获得证书的地方得到安装说明。如果你选择使用[letsencrypt.org](https://letsencrypt.org/)，那么我建议你使用 certbot。请按照[`certbot.eff.org/`](https://certbot.eff.org/)上的说明进行操作。

# 总结

在本章中，我们讨论了在上一章中我们在 Lumen 中实现 RESTful Web 服务端点时所缺少的内容。我们讨论了限流（请求速率限制）以防止 DoS 或暴力破解。我们还使用了一些软件包实现了基于令牌的身份验证。请注意，我们只在这里保护了端点，我们不希望在用户未登录的情况下留下可访问的端点。如果有其他端点，你不希望公开访问，但它们不需要用户登录，那么你可以在这些端点上使用某种密钥或基本身份验证。

除此之外，我们讨论并使用了一些用于 Web 服务的视图层的转换器。然后，我们简要讨论了加密和 SSL 的重要性，然后讨论了 SSL 证书的可用选项。

在本章中，我不会给你更多资源的 URL 列表，因为我们在本章中讨论了很多不同的事情，所以我们无法深入了解每一件事的细节。要完全吸收它，你应该首先查看我们在这里讨论的每一件事的文档，然后进行实践。当你在实践中遇到问题并尝试解决它们时，你才会真正学到东西。

在下一章中，我们将讨论测试，并使用自动化测试工具为我们的端点和代码编写测试用例。
