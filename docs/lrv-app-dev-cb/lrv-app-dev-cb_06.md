# 第六章。显示您的视图

在本章中，我们将涵盖：

+   创建和使用基本视图

+   将数据传递到视图

+   将视图加载到另一个视图/嵌套视图中

+   添加资产

+   使用 Blade 创建视图

+   使用 TWIG 模板

+   利用高级 Blade 用法

+   创建内容的本地化

+   在 Laravel 中创建菜单

+   与 Bootstrap 集成

+   使用命名视图和视图组件

# 介绍

在 Model-View-Controller 设置中，我们的**视图**保存所有的 HTML 和样式，以便我们可以显示我们的数据。在 Laravel 中，我们的视图可以使用常规的 PHP 文件，也可以使用 Laravel 的 Blade 模板。Laravel 还可以扩展到允许我们使用任何我们想要包含的模板引擎。

# 创建和使用基本视图

在这个步骤中，我们将看到一些基本的**视图**功能，以及我们如何在我们的应用程序中包含视图。

## 准备工作

对于这个步骤，我们需要一个标准的 Laravel 安装。

## 如何做...

按照以下步骤完成这个步骤：

1.  在`app/views`目录中，创建一个名为`myviews`的文件夹。

1.  在新的`myviews`目录中，创建两个文件：`home.php`和`second.php`。

1.  打开`home.php`并在 HTML 中添加以下代码：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Home Page</title>
    </head>
    <body>
      <h1>Welcome to the Home page!</h1>
      <p>
        <a href="second">Go to Second Page</a>
      </p>
    </body>
</html>
```

1.  打开`second.php`文件并在 HTML 中添加以下代码：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Second Page</title>
    </head>
    <body>
      <h1>Welcome to the Second Page</h1>
      <p>
        <a href="home">Go to Home Page</a>
      </p>
    </body>
</html>
```

1.  在我们的`app/routes.php`文件中，添加将返回这些视图的路由：

```php
Route::get('home', function()
{
  return View::make('myviews.home');
});
Route::get('second', function()
{
  return View::make('myviews.second');
});
```

1.  通过转到`http://{your-server}/home`（其中`your-server`是我们的 URL）并单击链接来测试视图。

## 它是如何工作的...

Laravel 中的所有视图都保存在`app/views`目录中。我们首先创建两个文件，用于保存我们的 HTML。在这个例子中，我们正在创建静态页面，每个视图都包含自己的完整 HTML 标记。

在我们的路由文件中，然后返回`View::make()`，并传入视图的名称。由于我们的视图在视图目录的子目录中，我们使用点表示法。

# 将数据传递到视图

在我们的 Web 应用程序中，我们通常需要显示来自数据库或其他数据存储的某种数据。在 Laravel 中，我们可以轻松地将这些数据传递到我们的视图中。

## 准备工作

对于这个步骤，我们需要完成*创建和使用基本视图*步骤。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  打开`routes.php`并将我们的主页和第二个路由替换为包含以下数据：

```php
Route::get('home', function()
{
  $page_title = 'My Home Page Title';
  return View::make('myviews.home')->with('title',$page_title);
});
Route::get('second', function()
{
  $view = View::make('myviews.second');
  $view->my_name = 'John Doe';
  $view->my_city = 'Austin';
  return $view;
});
```

1.  在`view/myviews`目录中，打开`home.php`并用以下代码替换代码：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Home Page : <?= $title ?></title>
    </head>
    <body>
        <h1>Welcome to the Home page!</h1>
        <h2><?= $title ?></h2>
      <p>
        <a href="second">Go to Second Page</a>
      </p>
    </body>
</html>
```

1.  在`views/myviews`目录中，打开`second.php`文件，并用以下代码替换代码：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Second Page</title>
    </head>
    <body>
      <h1>Welcome to the Second Page</h1>
        <p> You are <?= $my_name ?>, from <?= $my_city ?>
        </p>
      <p>
        <a href="home">Go to Home Page</a>
      </p>
    </body>
</html>
```

1.  通过转到`http://{your-server}/home`（其中`your-server`是我们的 URL）来测试视图，然后单击链接。

## 它是如何工作的...

如果我们想要将数据传递到我们的视图中，Laravel 提供了各种方法来实现这一点。我们首先通过将单个变量传递给视图来更新我们的第一个路由，通过将`with()`方法链接到`View::make()`。然后，在视图文件中，我们可以通过使用我们选择的任何名称来访问变量。

在我们的下一个路由中，我们将`View::make()`分配给一个变量，然后将值分配为对象的属性。然后我们可以在视图中将这些属性作为变量访问。要显示视图，我们只需返回对象变量。

## 还有更多...

向视图添加数据的另一种方法类似于我们第二个路由中的方法；但是我们使用数组而不是对象。因此，我们的代码看起来类似于以下内容：

```php
$view = View::make('myviews.second');
$view['my_name'] = 'John Doe';
$view['my_city'] = 'Austin';
return $view;
```

# 将视图加载到另一个视图/嵌套视图中

我们的网页往往会有类似的布局和 HTML 结构。为了帮助分离重复的 HTML，我们可以在 Laravel 中使用**嵌套视图**。

## 准备工作

对于这个步骤，我们需要完成*创建和使用基本视图*步骤。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在`app/view`目录中，添加一个名为`common`的新文件夹。

1.  在`common`目录中，创建一个名为`header.php`的文件，并将以下代码添加到其中：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>My Website</title>
    </head>
    <body>
```

1.  在`common`目录中，创建一个名为`footer.php`的文件，并将以下代码添加到其中：

```php
<footer>&copy; 2013 MyCompany</footer>  
  </body>
</html>
```

1.  在`common`目录中，创建一个名为`userinfo.php`的文件，并添加以下代码：

```php
<p>You are <?= $my_name ?>, from <?= $my_city ?></p>
```

1.  在`routes.php`文件中，更新主页和第二个路由，包括以下嵌套视图：

```php
Route::get('home', function()
{
  return View::make('myviews.home')
      ->nest('header', 'common.header')
      ->nest('footer', 'common.footer');
});
Route::get('second', function()
{
  $view = View::make('myviews.second');
  $view->nest('header', 'common.header')->nest('footer','common.footer');
  $view->nest('userinfo', 'common.userinfo', array('my_name' => 'John Doe', 'my_city' => 'Austin'));
  return $view;
});
```

1.  在`views/myviews`目录中，打开`home.php`文件，并添加以下代码：

```php
<?= $header ?>
    <h1>Welcome to the Home page!</h1>
    <p>
      <a href="second">Go to Second Page</a>
    </p>
<?= $footer ?>
```

1.  在`views/myviews`目录中，打开`second.php`文件，并添加以下代码：

```php
<?= $header ?>
<h1>Welcome to the Second Page</h1>
  <?= $userinfo ?>
<p>
    <a href="home">Go to Home Page</a>
</p>
<?= $footer ?>
```

1.  通过转到`http://{your-server}/home`（其中`your-server`是我们的 URL），然后点击链接来测试视图。

## 它是如何工作的...

首先，我们需要将头部和页脚代码从我们的视图中分离出来。由于这些将在每个页面上都是相同的，我们在我们的`views`文件夹中创建一个子目录来保存我们的公共文件。第一个文件是我们的页眉，它将包含直到`<body>`标签的所有内容。第二个文件是我们的页脚，它将包含页面底部的 HTML。

我们的第三个文件是一个`userinfo`视图。如果我们有用户帐户和个人资料，我们经常希望在侧边栏或页眉中包含用户的数据。为了保持视图的一部分单独，我们创建了`userinfo`视图，并传递了一些数据给它。

对于我们的主页路由，我们将使用我们的主页视图，并嵌套头部和页脚。`nest()`方法中的第一个参数是我们将在主视图中使用的名称，第二个参数是视图的位置。在这个例子中，我们的视图在 common 子目录中，所以我们使用点表示法来引用它们。

在我们的主页视图中，为了显示嵌套视图，我们打印出了我们在路由中使用的变量名。

对于我们的第二个路由，我们嵌套了头部和页脚，但我们还想添加`userinfo`视图。为此，我们向`nest()`方法传入了第三个参数，这是我们要发送到视图的数据数组。然后，在我们的主视图中，当我们打印出`userinfo`视图时，它将自动包含这些变量。

## 另请参阅

+   *将数据传递到视图*的方法

# 添加资产

动态网站几乎需要使用 CSS 和 JavaScript。使用 Laravel 资产包提供了一种简单的方法来管理这些资产并将它们包含在我们的视图中。

## 准备工作

对于这个方法，我们需要使用*加载视图到另一个视图/嵌套视图*方法中创建的代码。

## 如何做到这一点...

要完成这个方法，按照以下步骤进行：

1.  打开`composer.json`文件，并将`asset`包添加到`require`部分，使其看起来类似于以下内容：

```php
"require": {
      "laravel/framework": "4.0.*",
      "teepluss/asset": "dev-master"
  },
```

1.  在命令行中，运行 composer update 来下载包，如下所示：

```php
**php composer.phar update**

```

1.  打开`app/config/app.php`文件，并在提供者数组的末尾添加`ServiceProvider`，如下所示：

```php
'Teepluss\Asset\AssetServiceProvider',
```

1.  在相同的文件中，在`aliases`数组中，添加包的别名，如下所示：

```php
'Asset' => 'Teepluss\Asset\Facades\Asset'
```

1.  在`app/filters.php`文件中，添加一个自定义过滤器来处理我们的资产，如下所示：

```php
Route::filter('assets', function()
{
  Asset::add('jqueryui', 'http://ajax.googleapis.com/ajax/libs/jqueryui/1.10.2/jquery-ui.min.js', 'jquery');
  Asset::add('jquery', 'http://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js');
  Asset::add('bootstrap', 'http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.min.css');
});

Update the home and second routes to use the filter
Route::get('home', array('before' => 'assets', function()
{
  return View::make('myviews.home')
      ->nest('header', 'common.header')
      ->nest('footer', 'common.footer');
}));
Route::get('second', array('before' => 'assets', function()
{
  $view = View::make('myviews.second');
  $view->nest('header', 'common.header')->nest('footer', 'common.footer');
  $view->nest('userinfo', 'common.userinfo', array('my_name' => 'John Doe', 'my_city' => 'Austin'));
  return $view;
}));
```

1.  在`views/common`目录中，打开`header.php`文件，并按照以下代码使用：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>My Website</title>
        <?= Asset::styles() ?>
    </head>
    <body>
```

1.  在`views/common`目录中，打开`footer.php`文件，并使用以下代码：

```php
<footer>&copy; 2013 MyCompany</footer> 
<?= Asset::scripts() ?>
  </body>
</html>
```

1.  通过转到`http://{your-server}/home`（其中`your-server`是我们的 URL），点击链接并查看页面源代码来测试视图，以查看包含的资产。

## 它是如何工作的...

`asset`包使向我们的 HTML 添加 CSS 和 JavaScript 文件变得非常容易。首先，我们需要在路由中“注册”每个资产。为了使事情变得更简单一些，我们将在一个在路由之前调用的过滤器中添加这些资产。这样，我们只需在一个地方编写代码，而且更改也会很容易。为了我们的目的，我们将使用来自 CDN 源的 jQuery、jQueryUI 和 bootstrap CSS。

`add()`方法的第一个参数是我们给资产的名称。第二个参数是资产的 URL；它可以是相对路径或完整的 URL。第三个可选参数是资产的依赖关系。在我们的例子中，jQueryUI 要求 jQuery 已经被加载，所以我们在第三个参数中传入了我们的 jQuery 资产的名称。

然后我们更新我们的路由以添加过滤器。如果我们在我们的过滤器中添加或删除任何资产，它将自动反映在我们的每个路由中。

由于我们使用了嵌套视图，我们只需要将资产添加到我们的页眉和页脚视图中。我们的 CSS 文件是通过`styles()`方法调用的，JavaScript 是通过`scripts()`方法调用的。Laravel 检查资产的文件扩展名，并自动将它们放在正确的位置。如果我们查看源代码，我们会注意到 Laravel 还确保在 jQueryUI 之前添加了 jQuery 脚本，因为我们将其设置为依赖项。

## 另请参阅

+   第五章中的*在路由上使用过滤器*食谱，*使用控制器和路由处理 URL 和 API*

# 使用 Blade 创建视图

PHP 有许多可用的模板库，Laravel 的 Blade 是其中之一。这个食谱将展示一个易于扩展的方法来使用 Blade 模板，并快速上手。

## 准备工作

对于这个食谱，我们需要一个标准的 Laravel 安装。

## 如何做...

要完成这个食谱，请按照以下步骤进行：

1.  在`routes.php`文件中，创建新的路由，如下所示：

```php
Route::get('blade-home', function()
{
  return View::make('blade.home');
});
Route::get('blade-second', function()
{
  return View::make('blade.second');
});
```

1.  在`views`目录中，创建一个名为`layout`的新文件夹。

1.  在`views/layout`目录中，创建一个名为`index.blade.php`的文件，并将以下代码添加到其中：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>My Site</title>
    </head>
    <body>
    <h1>
    @section('page_title')
      Welcome to 
    @show
    </h1>
    @yield('content')
    </body>
</html>
```

1.  在`views`目录中，创建一个名为`blade`的文件夹。

1.  在`views/blade`目录中，创建一个名为`home.blade.php`的文件，并将以下代码添加到其中：

```php
@extends('layout.index')

@section('page_title')
  @parent
    Our Blade Home
@endsection

@section('content')
  <p>
    Go to {{ HTML::link('blade-second', 'the Second Page.') }}
  </p>
@endsection
```

1.  在`views/blade`目录中，创建一个名为`second.blade.php`的文件，并将以下代码添加到其中：

```php
@extends('layout.index')

@section('page_title')
  @parent
    Our Second Blade Page
@endsection

@section('content')
  <p>
    Go to {{ HTML::link('blade-home', 'the Home Page.')}}
  </p>
@endsection
```

1.  通过转到`http://{your-server}/blade-home`（其中`your-server`是我们的 URL），然后单击链接，查看页面源代码，以查看包含的 Blade 布局。

## 它是如何工作的...

首先，我们创建两个简单的路由，它们将返回我们的 Blade 视图。通过使用点表示法，我们可以看到我们将把文件放在我们的`views`文件夹的`blade`子目录中。

我们的下一步是创建一个 Blade 布局视图。这将是我们页面的骨架，并将放在我们的`views`文件夹的布局子目录中，文件扩展名必须是`blade.php`。这个视图是简单的 HTML，有两个例外：`@section()`和`@yield()`区域。这些内容是我们的视图中将被替换或添加的内容。

在我们的路由视图中，我们首先声明要使用哪个 Blade 布局文件，对于我们的情况是`@extends('layout.index')`。然后我们可以添加和修改我们在布局中声明的内容部分。对于`page_title`部分，我们想要显示布局中的文本，但我们想要在末尾添加一些额外的文本。为了实现这一点，我们在内容区域的第一件事就是调用`@parent`，然后放入我们自己的内容。

在`@section('content')`中，布局中没有默认文本，所以一切都将被添加。使用 Blade，我们还可以使用`{{ }}`大括号来打印出我们需要的任何 PHP。在我们的情况下，我们使用 Laravel 的`HTML::link()`来显示一个链接。现在，当我们转到页面时，所有的内容区域都被放在了布局的正确位置。

# 使用 TWIG 模板

Laravel 的 Blade 模板可能很好，但有时我们需要另一个 PHP 模板库。一个流行的选择是 Twig。这个食谱将展示如何将 Twig 模板整合到我们的 Laravel 应用程序中。

## 准备工作

对于这个食谱，我们只需要一个标准的 Laravel 安装。

## 如何做…

按照以下步骤完成这个食谱：

1.  打开`composer.json`文件，并在`require`部分添加以下行：

```php
"rcrowe/twigbridge": "0.4.*"
```

1.  在命令行中，更新 composer 以安装包：

```php
**php composer.phar update**

```

1.  打开`app/config/app.php`文件，并在`providers`数组中添加 Twig ServiceProvider，如下所示：

```php
'TwigBridge\TwigServiceProvider'
```

1.  在命令行中，运行以下命令来创建我们的配置文件：

```php
**php artisan config:publish rcrowe/twigbridge**

```

1.  在`routes.php`中，创建一个路由如下：

```php
Route::get('twigview', function()
{
  $link = HTML::link('http://laravel.com', 'the Laravel site.');
  return View::make('twig')->with('link', $link);
**});**

```

1.  在`views`目录中，创建一个名为`twiglayout.twig`的文件，并将以下代码添加到其中：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>My Site</title>
    </head>
    <body>
    <h1>
        {% block page_title %}
            Welcome to
        {% endblock %}
    </h1>
    {% block content %}{% endblock %}
    </body>
</html>
```

1.  在`views`目录中，创建一个名为`twig.twig`的文件，并将以下代码添加到其中：

```php
{% extends "twiglayout.twig" %}

{% block page_title %}
	{{ parent() }}
	My Twig Page
{% endblock %}

{% block content %}
    <p>
		Go to {{ link|raw }}
	</p>
{% endblock %}
```

1.  通过转到`http://your-server/twigview`（其中`your-server`是我们的 URL）来测试视图，并查看页面源代码以查看包含的 twig 布局。

## 工作原理...

首先，我们将在我们的应用程序中安装`TwigBranch`包。该包还安装了`Twig`库。安装包后，我们使用`artisan`创建其配置文件，并添加其服务提供程序。

在我们的路由中，我们将使用与 Laravel 内置视图库相同的语法，并调用视图。我们还创建了一个简单的链接，将其保存到一个变量中，并将该变量传递到视图中。

接下来，我们创建我们的布局。所有 Twig 视图文件都必须具有`.twig`扩展名，因此我们的布局命名为`twiglayout.twig`。布局中包含一个标准的 HTML 框架，但我们添加了两个 Twig 内容块。`page_title`块具有一些默认内容，而`content`块为空。

对于我们路由的视图，我们首先通过扩展布局视图来开始。对于我们的`page_title`块，我们首先通过使用`{{ parent()}}`打印出默认内容，然后添加我们自己的内容。然后添加我们的内容块，并显示我们作为变量传递的链接。使用 Twig，我们不需要为我们的变量使用`$`，如果我们传入 HTML，Twig 会自动转义它。因此在我们的视图中，由于我们想要显示链接，我们需要确保添加原始参数。

现在，如果我们在浏览器中打开我们的页面，我们将看到所有内容都在正确的位置上。

# 利用高级的 Blade 用法

使用 Laravel 的 Blade 模板系统，我们可以访问一些强大的功能，使我们的开发速度更快。对于这个示例，我们将向我们的 blade 视图传递一些数据，并循环遍历它，以及一些条件语句。

## 准备工作

对于这个示例，我们将需要在*使用 Blade 创建视图*示例中创建的代码。

## 如何做...

按照以下步骤完成这个示例：

1.  打开`routes.php`文件，并按以下方式更新`blade-home`和`blade-second`路由：

```php
Route::get('blade-home', function()
{
  $movies = array(
    array('name' => 'Star Wars', 'year' => '1977', 'slug'=> 'star-wars'),
    array('name' => 'The Matrix', 'year' => '1999', 'slug' => 'matrix'),
    array('name' => 'Die Hard', 'year' => '1988', 'slug'=> 'die-hard'),
    array('name' => 'Clerks', 'year' => '1994', 'slug' => 'clerks')
  );
  return View::make('blade.home')->with('movies', $movies);
});
Route::get('blade-second/(:any)', function($slug)
{
  $movies = array(
    'star-wars' => array('name' => 'Star Wars', 'year' => '1977', 'genre' => 'Sci-Fi'),
    'matrix' => array('name' => 'The Matrix', 'year' => '1999', 'genre' => 'Sci-Fi'),
    'die-hard' => array('name' => 'Die Hard', 'year' => '1988', 'genre' => 'Action'),
    'clerks' => array('name' => 'Clerks', 'year' => '1994', 'genre' => 'Comedy')
  );
  return View::make('blade.second')->with('movie', $movies[$slug]);
});
```

1.  在`views/blade`目录中，使用以下代码更新`home.blade.php`文件：

```php
@extends('layout.index')

@section('page_title')
  @parent
    Our List of Movies
@endsection

@section('content')
  <ul>
    @foreach ($movies as $movie)
      <li>{{ HTML::link('blade-second/' . $movie['slug'],$movie['name']) }} ( {{ $movie['year'] }} )</li>
          @if ($movie['name'] == 'Die Hard')
                 <ul>
                   <li>Main character: John McClane</li>
                 </ul>
          @endif
    @endforeach
  </ul>
@endsection
```

1.  在`views/blade`目录中，使用以下代码更新`second.blade.php`文件：

```php
@extends('layout.index')

@section('page_title')
  @parent
     Our {{ $movie['name'] }} Page
@endsection

@section('content')
  @include('blade.info')
  <p>
    Go to {{ HTML::link('blade-home', 'the Home Page.') }}
  </p>
@endsection
```

1.  在`views/blade`目录中，创建一个名为`info.blade.php`的新文件，并将以下代码添加到其中：

```php
<h1>{{ $movie['name'] }}</h1>
<p>Year: {{ $movie['year'] }}</p>
<p>Genre: {{ $movie['genre'] }}</p>
```

1.  通过转到`http://{your-server}/blade-home`（其中`your-server`是我们的 URL）来测试视图，并单击链接以查看视图的工作。

## 工作原理...

对于这个示例，我们将向我们的 Blade 视图传递一些数据，循环遍历它，并添加一些条件语句。通常，我们会将其与数据库中的结果一起使用，但是为了我们的目的，我们将在我们的路由中创建一个简单的数据数组。

我们的第一个路由包含一个电影数组，其中包含它们的年份和我们可以用于 URL 的 slug。我们的第二个路由将创建一个包含 slug 作为键并接受 URL 中的 slug 的数组。然后，通过调用具有 slug 作为键的电影，我们将电影的详细信息传递到视图中。

在我们的第一个视图中，我们创建了一个`@foreach`循环，以遍历数组中的所有数据。我们还包含了一个简单的`@if`语句，用于检查特定的电影，然后打印出一些额外的信息。当我们循环遍历时，我们显示链接到第二个路由，并添加 slug。

第二个视图显示电影的名称，但是还通过在内容块中使用`@include()`包含另一个 Blade 视图。这样，所有数据也可以在包含的视图中使用；因此，对于我们的`info`视图，我们可以直接使用我们在路由中设置的相同变量。

# 创建内容的本地化

如果我们的应用程序将被不同国家或说不同语言的人使用，我们需要对内容进行本地化。Laravel 提供了一种简单的方法来实现这一点。

## 准备工作

对于这个教程，我们只需要一个标准的 Laravel 安装。

## 如何做...

对于这个教程，请按照以下步骤进行：

1.  在`app/lang`目录中，添加三个新目录（如果尚未存在）：`en`，`es`和`de`。

1.  在`en`目录中，创建一个名为`localized.php`的文件，并添加以下代码：

```php
<?php

return array(
  'greeting' => 'Good morning :name',
  'meetyou' => 'Nice to meet you!',
  'goodbye' => 'Goodbye, see you tomorrow.',
);
```

1.  在`es`目录中，创建一个名为`localized.php`的文件，并添加以下代码：

```php
<?php

return array(
  'greeting' => 'Buenos días :name',
  'meetyou' => 'Mucho gusto!',
  'goodbye' => 'Adiós, hasta mañana.',
);
```

1.  在`de`目录中，创建一个名为`localized.php`的文件，并添加以下代码：

```php
<?php

return array(
  'greeting' => 'Guten morgen :name',
  'meetyou' => 'Es freut mich!',
  'goodbye' => 'Tag. Bis bald.',
);
```

1.  在我们的`routes.php`文件中，创建四个路由如下：

```php
Route::get('choose', function()
{
  return View::make('language.choose');
});
Route::post('choose', function()
{
  Session::put('lang', Input::get('language'));
  return Redirect::to('localized');
});
Route::get('localized', function()
{
  $lang = Session::get('lang', function() { return 'en';});
  App::setLocale($lang);
  return View::make('language.localized');
});
Route::get('localized-german', function()
{
  App::setLocale('de');
  return View::make('language.localized-german');
});
```

1.  在`views`目录中，创建一个名为`language`的文件夹。

1.  在`views/language`中，创建名为`choose.php`的文件，并添加以下代码：

```php
<h2>Choose a Language:</h2>
<?= Form::open() ?>
<?= Form::select('language', array('en' => 'English', 'es' => 'Spanish')) ?>
<?= Form::submit() ?>
<?= Form::close() ?>
```

1.  在`views/language`目录中，创建一个名为`localized.php`的文件，并添加以下代码：

```php
<h2>
  <?= Lang::get('localized.greeting', array('name' => 'Lindsay Weir')) ?>
</h2>
<p>
  <?= Lang::get('localized.meetyou') ?>
</p>
<p>
  <?= Lang::get('localized.goodbye') ?>
</p>
<p>
  <?= HTML::link('localized-german', 'Page 2') ?>
</p>
```

1.  在`views/language`目录中，创建一个名为`localized-german.php`的文件，并添加以下代码：

```php
<h2>
  <?= Lang::get('localized.greeting', array('name' =>'Lindsay Weir')) ?>
</h2>
<p>
  <?= Lang::get('localized.meetyou') ?>
</p>
<p>
  <?= Lang::get('localized.goodbye') ?>
</p>
```

1.  在浏览器中，转到`http://{your-server}/choose`（其中`your-server`是我们的 URL），提交表单，并测试本地化。

## 它是如何工作的...

对于这个教程，我们首先在`app/lang`目录中设置我们的语言目录。我们将使用`en`作为英语文件，`es`作为西班牙语文件，`de`作为德语文件。在每个目录中，我们创建一个使用完全相同名称的文件，并添加一个数组，使用完全相同的键。

我们的第一个路由将是一个语言选择器页面。在此页面上，我们可以选择英语或西班牙语。当我们提交时，它将`POST`到路由，创建一个新会话，添加选择，并重定向到页面以显示所选语言的文本。

我们的本地化路由获取会话并将选择传递给`App::setLocale()`。如果没有设置会话，我们还有一个默认值为英语。

在我们的本地化视图中，我们使用`Lang::get()`打印出文本。在我们的语言文件的第一行中，我们还包含了`:name`占位符，因此当我们调用语言文件时，我们可以传递一个包含占位符名称的数组作为键。

我们的最后一个路由显示了我们如何在路由中静态设置语言默认值。

# 在 Laravel 中创建菜单

菜单是大多数网站的常见组成部分。在这个教程中，我们将使用 Laravel 的嵌套视图创建菜单，并根据我们所在的页面更改菜单项的默认“状态”。

## 准备工作

对于这个菜单，我们需要一个标准的 Laravel 安装。

## 如何做...

我们需要按照以下步骤完成这个教程：

1.  在`routes.php`文件中，创建三个路由如下：

```php
Route::get('menu-one', function()
{
  return View::make('menu-layout')
      ->nest('menu', 'menu-menu')
      ->nest('content', 'menu-one');
});
Route::get('menu-two', function()
{
  return View::make('menu-layout')
      ->nest('menu', 'menu-menu')
      ->nest('content', 'menu-two');
});
Route::get('menu-three', function()
{
  return View::make('menu-layout')
      ->nest('menu', 'menu-menu')
      ->nest('content', 'menu-three');
});
```

1.  在视图目录中，创建一个名为`menu-layout.php`的文件，并添加以下代码：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Menu Example</title>
        <style>
            #container {
              width: 1024px; 
              margin: 0 auto; 
              border-left: 2px solid #ddd;
              border-right: 2px solid #ddd;
              padding: 20px;
            }
            #menu { padding: 0 }
            #menu li {
               display: inline-block;
               border: 1px solid #ddf;
               border-radius: 6px;
               margin-right: 12px;
               padding: 4px 12px;
            }
            #menu li a {
               text-decoration: none;
               color: #069;
            }
            #menu li a:hover { text-decoration: underline}
            #menu li.active { background: #069 }
            #menu li.active a { color: #fff }
        </style>
    </head>
    <body>
      <div id="container">
          <?= $menu ?>
          <?= $content ?>
      </div>
    </body>
</html>
```

1.  在`views`目录中，创建一个名为`menu-menu.php`的文件，并添加以下代码：

```php
<ul id="menu">
  <li class="<?= Request::segment(1) == 'menu-one' ?'active' : '' ?>">
    <?= HTML::link('menu-one', 'Page One') ?>
  </li>
  <li class="<?= Request::segment(1) == 'menu-two' ? 'active' : '' ?>">
      <?= HTML::link('menu-two', 'Page Two') ?>
  </li>
  <li class="<?= Request::segment(1) == 'menu-three' ?'active' : '' ?>">
      <?= HTML::link('menu-three', 'Page Three') ?>
  </li>
</ul>
```

1.  在`views`目录中，创建三个视图文件，分别命名为`menu-one.php`，`menu-two.php`和`menu-three.php`。

1.  对于`menu-one.php`，使用以下代码：

```php
<h2>Page One</h2>
<p>
  Lorem ipsum dolor sit amet.
</p>
```

1.  对于`menu-two.php`，使用以下代码：

```php
<h2>Page Two</h2>
<p>
  Suspendisse eu porta turpis
</p>
```

1.  对于`menu-three.php`，使用以下代码：

```php
<h2>Page Three</h2>
<p>
  Nullam varius ultrices varius.
</p>
```

1.  在浏览器中，转到`http://{your-server}/menu-one`（其中`your-server`是我们的 URL），并通过菜单链接进行点击。

## 它是如何工作的...

我们首先创建三个路由来保存我们的三个页面。每个路由将使用单个布局视图，并嵌入一个特定于路由的菜单视图和内容视图。

我们的布局视图是一个基本的 HTML 骨架，带有一些页面 CSS。由于我们想要突出显示当前页面的菜单项，一个类选择器被命名为`active`，并将添加到我们的菜单列表项中。

接下来，我们创建我们的菜单视图。我们使用无序列表，其中包含到每个页面的链接。为了在当前页面项目中添加`active`类，我们使用 Laravel 的`Request::segment(1)`来获取我们所在的路由。如果它与列表项相同，我们添加`active`类，否则留空。然后我们使用 Laravel 的`HTML::link()`来添加链接到我们的页面。

其他三个视图只是非常简单的内容，有一个标题和几个单词。现在，当我们在浏览器中查看页面时，我们会看到我们所在页面的菜单项被突出显示，而其他页面没有。如果我们单击链接，那个项目将被突出显示，其他项目将不会被突出显示。

# 与 Bootstrap 集成

Bootstrap CSS 框架最近变得非常流行。这个示例将展示我们如何在 Laravel 中使用这个框架。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装。我们还需要安装`assets`包，就像*添加资产*示例中演示的那样。可选地，我们可以下载 Bootstrap 文件并将其保存在本地。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在`routes.php`文件中，创建一个新的路由，如下所示：

```php
Route::any('boot', function()
{
  Asset::add('jquery', 'http://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js');
  Asset::add('bootstrap-js', 'http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/js/bootstrap.min.js', 'jquery');
  Asset::add('bootstrap-css', 'http://netdna.bootstrapcdn.com/twitter-bootstrap/2.3.2/css/bootstrap-combined.min.css');
  $superheroes = array('Batman', 'Superman', 'Wolverine','Deadpool', 'Iron Man');
  return View::make('boot')->with('superheroes',$superheroes);
});
```

1.  在`views`目录中，创建一个名为`boot.php`的文件，并向其中添加以下代码：

```php
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>My Bootstrap Page</title>
    <?= Asset::styles() ?>
  </head>
  <body>
    <div class="container">
      <h1>Using Bootstrap with Laravel</h1>
      <ul class="nav nav-tabs">
        <li class="active"><a href="#welcome" data-toggle="tab">Welcome</a></li>
        <li><a href="#about" data-toggle="tab">About Us</a></li>
        <li><a href="#contact" data-toggle="tab">Contact</a></li>
      </ul>
        <div class="tab-content">
          <div class="tab-pane active" id="welcome">
            <h4>Welcome to our site</h4>
            <p>Here's a list of Superheroes:</p>
            <ul>
              <?php foreach($superheroes as $hero): ?>
                <li class="badge badge-info"><?= $hero ?></li>
              <?php endforeach; ?>
            </ul>
        </div>
          <div class="tab-pane" id="about">
            <h4>About Us</h4>
              <p>Cras at dui eros. Ut imperdiet pellentesque mi faucibus dapibus.Phasellus vitae lacus at massa viverra condimentum quis quis augue. Etiam pharetra erat id sem pretium egestas. Suspendisse mollis, dolor a sagittis hendrerit, urna velit commodo dui, id adipiscing magna magna ac ligula. Nunc in ligula nunc.</p>
          </div>
          <div class="tab-pane" id="contact">
            <h3>Contact Form</h3>
              <?= Form::open('boot', 'POST') ?>
                <?= Form::label('name', 'Your Name') ?>
                <?= Form::text('name') ?>
                <?= Form::label('email', 'Your Email') ?>
                <?= Form::text('email') ?>
                <br>
                <?= Form::button('Send', array('class' =>'btn btn-primary')) ?>

                <?= Form::close() ?>
          </div>
       </div>
    </div>
    <?= Asset::scripts() ?>
  </body>
</html>
```

1.  在浏览器中，转到`http://your-server/boot`（其中`your-server`是我们的 URL），并单击选项卡。

## 它是如何工作的...

对于这个示例，我们将创建一个单一的路由，并使用 Bootstrap 选项卡切换内容。为了使我们的路由响应任何请求，我们使用`Route::any()`并传入我们的闭包。要添加 CSS 和 JavaScript，我们可以使用一个过滤器，就像*添加资产*示例中的过滤器一样；然而，对于一个单一的路由，我们将把它包含在闭包中。因此，我们不必下载它们，我们将只使用 Bootstrap 和 jQuery 的 CDN 版本。

接下来，在我们的路由中，我们需要一些数据。这将是绑定数据库的好地方，但是出于我们的目的，我们将使用一个简单的数组，其中包含一些超级英雄的名字。然后将该数组传递到我们的视图中。

我们从一个 HTML 骨架开始查看，并在头部包含我们的样式，在关闭`</body>`标签之前包含脚本。在页面顶部，我们使用 Bootstrap 的导航样式和数据属性来创建我们的选项卡链接。然后在我们的正文中，我们使用三个不同的选项卡窗格，其 ID 对应于我们菜单中的`<a href>`标签。

当我们查看页面时，我们会看到第一个窗格显示，其他所有内容都被隐藏。通过单击其他选项卡，我们可以切换显示哪个选项卡窗格。

## 另请参阅

+   *添加资产*示例

# 使用命名视图和视图组件

这个示例将展示如何使用 Laravel 的命名视图和视图组件来简化一些我们路由代码。

## 准备工作

对于这个示例，我们将使用*在 Laravel 中创建菜单*示例中创建的代码。我们还需要在*添加资产*示例中安装`assets`包。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在`routes.php`文件中，添加一个名为`view`的文件，并向其中添加以下代码：

```php
View::name('menu-layout', 'layout');
```

1.  在`routes.php`中，添加一个视图组件，如下所示：

```php
View::composer('menu-layout', function($view)
{
  Asset::add('bootstrap-css', 'http://netdna.bootstrapcdn.com/twitter-bootstrap/2.2.2/css/bootstrap-combined.min.css');
    $view->nest('menu', 'menu-menu');
    $view->with('page_title', 'View Composer Title');
});
```

1.  在`routes.php`中，更新菜单路由如下：

```php
Route::get('menu-one', function()
{
  return View::of('layout')->nest('content', 'menu-one');
});
Route::get('menu-two', function()
{
  return View::of('layout')->nest('content', 'menu-two');
});
Route::get('menu-three', function()
{
  return View::of('layout')->nest('content', 'menu-three');
});
```

1.  在`views`目录中，使用以下代码更新`menu-layout.php`文件：

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title><?= $page_title ?></title>
        <?= Asset::styles() ?>
        <style>
          #container {
            width: 1024px; 
            margin: 0 auto; 
            border-left: 2px solid #ddd;
            border-right: 2px solid #ddd;
            padding: 20px;
          }
          #menu { padding: 0 }
          #menu li {
            display: inline-block;
            border: 1px solid #ddf;
            border-radius: 6px;
            margin-right: 12px;
            padding: 4px 12px;
          }
          #menu li a {
            text-decoration: none;
            color: #069;
          }
          #menu li a:hover { text-decoration: underline }
          #menu li.active { background: #069 }
          #menu li.active a { color: #fff }
        </style>
    </head>
    <body>
      <div id="container">
        <?= $menu ?>
        <?= $content ?>
      </div>
    </body>
</html>
```

1.  在浏览器中，转到`http://{your-server}/menu-one`（其中`your-server`是我们的 URL），并单击菜单链接。

## 它是如何工作的...

我们通过为我们的视图创建一个名称来开始这个示例。如果我们的视图文件名或目录结构很长或复杂，这将允许我们在我们的路由中创建一个简单的别名。这也将允许我们在将来更改我们的视图文件名；此外，如果我们在多个地方使用它，我们只需要更改一行。

接下来，我们创建一个视图组件。当您创建视图时，组件中的任何代码都将自动调用。在我们的示例中，每次创建视图时，我们都包含三个内容：包含 Bootstrap CSS 文件的资产，一个嵌套视图和一个传递给视图的变量。

对于我们的三个路由，我们将使用我们创建的名称，调用`View::of('layout')`，并将其嵌套在我们的内容中，而不是`View::make('menu-layout')`。由于我们的布局视图有一个 composer，它将自动嵌套我们的菜单，添加 CSS，并传递页面标题。

## 另请参阅

+   *在 Laravel 中创建菜单*示例
