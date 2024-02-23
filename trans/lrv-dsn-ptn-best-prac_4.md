# 第四章。MVC 中的控制器

在本章中，我们将讨论控制器是什么，它的结构是什么，它在 MVC 模式中的作用是什么，以及它在 Laravel 的扩展设计模式和结构中的使用。

本章将讨论以下主题：

+   什么是控制器？

+   控制器在 MVC 设计模式中的作用

+   控制器与 MVC 设计模式的其他组件的交互

+   Laravel 如何处理其设计模式中的控制器

# 什么是控制器？

控制器是模型-视图-控制器（MVC）设计模式的一部分，我们可以简单地描述它为应用程序的逻辑层。它理解来自另一端（用户或 API 请求）的请求，调用相应的方法，执行主要检查，处理请求的逻辑，然后将数据返回给相应的视图或将最终用户重定向到另一个路由。

# 控制器的目的

以下是控制器在 MVC 结构中的一些主要角色：

+   控制应用程序的逻辑并定义在操作时应该触发哪个事件

+   作为模型、视图和应用程序的其他组件之间的中间步骤

+   翻译来自视图和模型的动作和响应，使它们可以理解，并将它们发送到其他层

+   在应用程序的其他组件之间建立桥梁，并促进它们之间的通信

+   在执行任何操作之前，在构造方法中进行主要权限检查

这可以通过一个现实世界的例子来最好地解释。

假设我们有一个用户管理网站，在管理面板中，管理员试图删除一个用户。在遵循 SOLID 原则的设计模式中，如果管理员点击删除用户按钮，将发生以下事情：

1.  从视图中，管理员发送请求到相应的控制器以删除新闻项目。

1.  控制器理解这个请求并进行主要检查。首先，它检查请求者（在我们的例子中是管理员）是否真的是管理员，并且有权限删除这个用户。

1.  在进行主要检查后，控制器告诉模型删除用户。

1.  模型进行了一些自己的检查，要么删除了用户并告诉控制器用户已被删除，要么告诉控制器用户不可用（也许用户已经被删除）。

1.  从模型得到响应后，控制器要么告诉视图告诉管理员用户已被删除，要么重定向到另一个页面，比如 404 页面未找到。

从前面的例子中可以看出，对于所有的交互，控制器在应用程序的组件之间起着重要的沟通作用。在遵循 SOLID 原则的 MVC 模式中，没有控制器，视图无法与模型进行交互，反之亦然。虽然有一些对这种架构模式的派生，比如视图直接与模型进行交互，在一个完美的 SOLID 设计架构中，控制器应该始终是所有交互的中间元素。

控制器也可以被视为一个翻译器。它以各种方式从视图中获取输入，并将其转换为模型可以理解的请求，反之亦然。

![控制器的目的](img/Image00008.jpg)

# Laravel 中的控制器

在 Laravel 4 中，控制器是简单的 PHP 类，它们的文件名和类名以后缀`Controller`结尾（不是强制的，但强烈建议；这是开发人员之间的标准），它们扩展了`BaseController`类，并存储在`app/controllers`文件夹中。这个文件夹结构在`composer.json`文件的`classmap`键中定义，不是强制的。由于 Composer，只要你定义了控制器存储在应用程序结构中的位置，你可以把它们放在任何你喜欢的文件夹中。

以下是 Laravel 4 的一个非常简单的控制器：

```php
<?php

class UserController extends BaseController {

    public function showProfile($id)
    {
        $user = User::find($id);

        return View::make('user.profile', array('user' => $user));
    }

}
```

控制器保存了在 `routes.php` 中定义的所有动作方法，其中在 Laravel 4 中设置了所有动作（用户与之交互的每个链接）。

在 Laravel 3 中，开发人员必须使用 `GET` 或 `POST` 作为前缀来理解相应请求的类型。如果您的请求是 `GET` 请求，您的方法名称必须是 `get_profile`，如果请求是 `POST` 请求，它必须类似于 `post_profile`。感谢 Laravel 4，现在不再强制，您可以按自己的喜好命名方法。

现在出现了一个问题。我们如何访问控制器的这个方法？正如我们之前提到的，我们将使用路由来做到这一点。

## 路由

路由是在 `app/routes.php` 中定义的一组规则，告诉 Laravel 在收到传入请求时，根据请求的 URL 调用哪些闭包函数和/或控制器方法。有多种定义路由的方法。其中三种如下所述：

+   您可以使用闭包函数并直接从 `app/routes.php` 设置动作的逻辑。看一下以下代码：

```php
    Route::get('hello', function(){
       return 'Ahoy, everyone!';
    });
    ```

在这里，我们调用了 `get()` 方法，因为我们希望此路由成为 `GET` 请求。第一个参数是动作的路径，因此如果我们调用 `http://ourwebsite.com/hello`，将调用此路由动作。第二个参数可以来自各种选择。它可以是一个包含名称、过滤器和动作的数组，一个定义控制器方法的字符串，或者直接包含逻辑的闭包函数。在我们的示例中，我们放置了一个闭包函数并直接向最终用户返回了一个字符串。因此，如果用户导航到 `http://ourwebsite.com/hello`，最终用户将直接看到消息**Ahoy, everyone!**。

+   设置路由的第二种方法是将第二个参数作为字符串传递，定义它传递到哪个控制器，并调用哪个动作。看一下以下代码：

```php
    Route::get('hello', 'ProfileController@hello'); 
    ```

在这里，字符串 `ProfileController@hello` 告诉 Laravel 方法 `hello` 将从名为 `ProfileController` 的控制器中调用。我们使用字符 `@` 将它们分开。

+   第三种方法是将数组作为第二个参数，其中包含各种键和值。看一下以下代码：

```php
    Route::get('hello', array(
       'before'   => 'member',
       'as'       => 'our_hello_page'
       'uses'     => 'ProfileController@hello'
    ));
    ```

数组可以有多个参数，定义路由的名称、在调用动作之前将应用的过滤器，以及将使用哪个控制器及其方法。以下是三个不同的键：

+   `before` 键定义了在调用动作之前的过滤器，因此您可以在调用每个动作之前设置一些过滤参数。例如，如果您有一个仅限会员的区域，您不希望访客访问该资源，您可以使用路由中的 `before` 参数传递过滤器。

+   `as` 键定义了路由的名称。这非常有益。假设您需要更改应用程序的 URL 结构。传统上，如果更改路由的动作路径，您需要更改应用程序中此动作的每个 URL 或重定向。相反，如果使用名称设置链接和重定向，您只需要更改一次路径，所有链接和重定向都会神奇地修复。

+   `uses` 键的结构与我们的第二个示例完全相同。它保存了调用时控制器的名称和方法。

在所有这些示例中，路由没有获取任何参数，我们也没有传递任何参数。想象一下：我们通过路由访问了配置文件区域，但在这些示例中，我们没有设置访问特定用户的方式。我们如何为这些路由设置参数？为此，我们必须在花括号中设置参数。

运行以下代码以为路由设置参数：

```php
Route::get('users/{id}', function($id){
   return 'Hello, the user with ID of '.$id;
});
```

花括号中的参数会直接成为闭包方法中的变量名。这种方法还为我们提供了一种在控制器方法运行之前过滤这些参数的方式。使用`where()`，您可以过滤这些参数。看一下以下代码：

```php
Route::get('users/{id}', function($id){
        return 'Hello, the user with ID of '.$id;
})->where(array('id' => '[0-9]+'));
```

`where()`方法可以是一个带有键和值的数组，也可以是两个参数，其中第一个是花括号中的名称，第二个是用于过滤参数的正则表达式。

在我们的示例中，我们使用正则表达式过滤参数 ID，以匹配仅数字，因此，这样，我们将区分不同类型的数据以重载端点。

此方法还有另一个好处。如果有人尝试导航到`http://ourwebsite.com/users/xssSqlInjection`，Laravel 甚至在转到控制器方法之前就会抛出 404 错误。

如果我们遵循这种结构，我们需要为每个`GET`，`POST`，`PUT`和`DELETE`请求设置每个操作。如果您想为您的操作使用 RESTful 结构，而不是逐个设置每个路由，您可以使用`Route`类的`controller()`方法。

需要遵循一定的步骤来设置 RESTful 控制器的路由。对于用户控制器，以下是步骤：

1.  首先，您需要创建一个名为`UserController`的新控制器，并设置您的 RESTful 方法，如`index()`，`create()`，`store()`，`show($id)`，`edit($id)`，`update($id)`和`destroy($id)`。

1.  然后，您需要在`app/routes.php`中设置 RESTful 控制器，方法是运行以下命令：

```php
    Route::controller('users', 'UserController');
    ```

Laravel 提供了一种更快的方法来创建资源控制器。这些被 Laravel 称为资源控制器。遵循以下步骤来设置资源控制器的路由：

1.  首先，您需要使用 Laravel 的 PHP 客户端`artisan`创建一个具有资源方法的新控制器。看一下以下代码：

```php
    php artisan controller:make NewsController
    ```

使用此命令，将在`app/controllers`文件夹中自动生成一个名为`NewsController.php`的新文件，并在其中已经定义了所有资源方法。

1.  然后，您需要通过运行以下命令在`app/routes.php`中设置资源控制器：

```php
    Route::resource('news', 'NewsController');
    ```

在设置资源控制器时，您可以通过将第三个参数设置为此`Route`定义来设置要包含或排除的操作。

1.  要包含将在资源控制器中定义的操作（类似于白名单），您可以使用`only`键，如下所示：

```php
    Route::resource(
       'news', 
       'NewsController',
       array('only' => array('index', 'show'))
    );
    ```

1.  要排除将在资源控制器中定义的操作（类似于黑名单），您可以使用`except`键，如下所示：

```php
    Route::resource(
       'news', 
       'NewsController',
       array('except' => array('create', 'store', 'update', 'destroy'))
    );
    ```

1.  所有资源控制器操作都有定义的路由名称。在某些情况下，您可能还想覆盖操作名称，可以通过设置第三个参数`names`来实现，如下所示：

```php
    Route::resource(
       'news', 
       'NewsController',
       array('names' => array('create' => 'news.myCoolCreateAction')
    );
    ```

如果您已经阅读了上一章，您可能还记得我们在路由中有过滤器，但我们没有在 RESTful 和资源控制器中使用`before`键。要使用`before`键，我们可以执行之前遵循的步骤或在控制器中设置过滤器。这些可以在控制器的`__construct()`方法中设置（如果没有，则创建一个），如下所示：

```php
class NewsController extends BaseController {

    public function __construct()
    {

        $this->beforeFilter('csrf', array('on' => 'post'));

        $this->beforeFilter(function(){
           //my custom filter codes in closure function
        });

        $this->afterFilter('log', 
           array('only' => array('fooAction', 'barAction'))
        );
    }

}
```

正如您所看到的，我们使用`beforeFilter()`和`afterFilter()`方法设置了过滤器。这些方法可以接受闭包函数或过滤器名称作为第一个参数，以及一个可选的第二个参数，作为数组，用于定义这些过滤器的工作位置。

在这个例子中，我们首先为所有`POST`操作设置了 CSRF（跨站点请求伪造，这是一种通过伪造请求来注入恶意代码到应用程序中的方法）保护过滤器；之后，我们在闭包函数中定义了一个过滤器，并使用`afterFilter`方法记录了所有`fooAction`和`barAction`事件的状态。

以下表格是 Laravel 的资源控制器处理的操作列表：

| 动词 | 路径 | 操作 | 路由名称 |
| --- | --- | --- | --- |
| `GET` | `/resource` | Index | `resource.index` |
| `GET` | `/resource/create` | Create | `resource.create` |
| `POST` | `/resource` | Store | `resource.store` |
| `GET` | `/resource/{resource}` | Show | `resource.show` |
| `GET` | `/resource/{resource}/edit` | Edit | `resource.edit` |
| `PUT` /`PATCH` | `/resource/{resource}` | Update | `resource.update` |
| `DELETE` | `/resource/{resource}` | Destroy | `resource.destroy` |

# 在文件夹中使用控制器

在某些情况下，您可能希望将控制器分组到一个文件夹中，并以更具层次结构的方式进行组织。有两种方法可以实现这一点。

在解释这些方法之前，让我们假设我们在`app/controllers/admin`文件夹中有一个`UserController.php`文件，我们刚刚为管理相关的控制器文件创建了这个文件夹。这里出现了一个问题：我们如何让 Laravel 和控制器文件知道控制器在哪里？命名空间用于这样的需求。命名空间是封装和分组项目的简单方式。

假设你有`app/controllers/UserController.php`和`app/controllers/admin/UserController.php`文件。我们如何调用特定的文件？命名空间在这里很方便。将以下文件保存为`app/controllers/admin/UserController.php`：

```php
<?php namespace admin; //The definition of namespace

use View; //What will be used

Class UserController Extends \BaseController {

   public function index() {
      return View::make('hello');
   }

}
```

现在我们按照以下方式定义路由：

```php
Route::get('my/admin/users/index', 'Admin\UserController@index');
```

在这里，我们为这个控制器添加了一些新的内容。它们如下：

+   第一种是`namespace admin;`。这简单地定义了这个文件在一个名为`admin`的文件夹中。

+   第二个是`use View;`。如果我们的控制器在一个文件夹或一个定义的`namespace`下，除非我们导入它们，否则我们调用的所有类都将像`namespace\class`一样。如果我们没有添加这一行，`View::make()`函数将会抛出一个错误，说`Class admin\View not found`。为了更好地理解这一点，你可以把它想象成 HTML 的资源调用。假设你正在浏览`admin/users.html`，里面有一张图片，其路径定义为这种格式：`<img src= "assets/img/avatar.png" />`。正如你可以想象的那样，图片将被请求从`admin/assets/img/avatar.png`，因为它在一个名为`assets`的文件夹中。这正是同样的情况。

+   当我们从`BaseController`类继承时，我们添加了一个`\`（反斜杠）字符。这将表示它将从根目录调用。如果我们没有在我们的类中添加`use View;`并且想要使`View::make()`工作，我们应该将它修改为`\View::make()`（带有一个前导反斜杠），这样正确的类将被请求。

如果有一个全新的文件夹结构，可以通过两种方式来定义。要么将每个文件夹路径添加到`composer.json`文件中的`autoload/classmap`对象中，要么定义一个`psr-0`自动加载。假设我们有一个新的`app/myApp`文件夹，里面有一个位于`app/myApp/controllers/admin/UserController.php`的控制器。

向控制器添加一个`classmap`对象，如下所示：

```php
"autoload": {
      "classmap": [
         "app/commands",
         "app/controllers",
          "app/models",
          "app/database/migrations",
          "app/database/seeds",
          "app/tests/TestCase.php",

          "app/myApp/controllers/admin",
      ]
}
```

现在按照以下方式向代码添加`psr-0`自动加载：

```php
"autoload": {
      "classmap": [
         "app/commands",
         "app/controllers",
         "app/models",
         "app/database/migrations",
         "app/database/seeds",
         "app/tests/TestCase.php",

      ],

       "psr-0": {
         "myApp", "app/"
      }

}
```

然后从终端运行`composer dump-autoload`来重新生成自动加载类。通过这种`psr-0`自动加载，我们教会了我们的 Composer 项目递归地自动加载`app`文件夹中的`myApp`文件夹内的所有内容。另一种方法是在类名前缀中添加命名空间文件夹，并在每个文件夹之间使用`underscores (_)`。

假设我们有一个控制器，`app/controllers/foo/bar/BazController.php`。将以下内容保存在这个文件夹中：

```php
<?php

Class foo_bar_BazController extends BaseController {

   public function index() {
      return View::make('hello');
   }

}
```

现在我们按照以下方式定义路由：

```php
Route::get('foobarbaz', ' foo_bar_BazController@index');
```

然后，我们导航到`http://yourwebsite/foobarbaz`。即使没有使用命名空间或包含`use`来包含类，它也会自动工作。

# 总结

在这一章中，我们学习了 MVC 模式中控制器的角色，以及如何在 Laravel 4 中使用控制器和设置路由。我们还学习了过滤器、RESTful 和 Resourceful 控制器。

更多信息，请参考位于[`laravel.com/docs/controllers`](http://laravel.com/docs/controllers)的官方文档页面。在下一章中，我们将学习关于 Laravel 独特的设计模式，以及它如何使用 Repositories、Facades 和工厂模式。
