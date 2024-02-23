# 第五章：使用表单生成器

在本章中，您将学习如何使用 Laravel 的表单生成器。表单生成器将被演示以便于构建以下元素：

+   表单（打开和关闭）

+   标签

+   输入（文本，HTML5 密码，HTML5 电子邮件等）

+   复选框

+   提交

+   锚标签（href 链接）

最后，我们将看到如何使用表单生成器为住宿预订软件表单创建月份、日期和年份选择元素的示例，以及如何创建一个宏来减少代码重复。

# 历史

Laravel 4 中的表单生成器包称为 HTML。这是用来帮助您创建 HTML 的，特别是那些还必须执行 Web 设计师职责但更喜欢使用 Laravel 门面和辅助方法的开发人员。例如，以下是 Laravel 门面`select()`方法的示例，其中语言的选项，例如英式和美式英语，在此示例中作为数组参数传递：

```php
Form::select('language', ['en-us' => 'English (US)','en-gb' => 'English (UK)']);
```

这可以作为标准 HTML 的替代方案，标准 HTML 需要更多重复的代码，如下面的代码所示：

```php
<select name="language">
    <option value="en-us">English (US)</option>
    <option value="en-gb">English (UK)</option>
</select>
```

由于框架不断发展，它们需要适应满足大多数用户的需求。此外，尽可能地，它们应该继续变得更加高效。在某些情况下，这意味着重写或重构框架的部分，添加功能，甚至*删除*它们。

尽管可能看起来奇怪，但删除功能有几个有效的原因。以下是删除包的原因列表：

+   减轻框架核心开发人员需要维护的包和功能的负担和数量。

+   减少下载和自动加载的包数量。

+   删除一个不必要的功能。

+   HTML 包已经从 Laravel 5 的核心中移除，现在是一个外部包。在这种情况下，任何之前的原因都可以被引用为移除这个包的原因。

+   HTML 有助于开发人员构建表单，如果前端开发人员也是后端或全栈开发人员，并且喜欢 Laravel 的做事方式，可以使用。然而，在其他情况下，Web 应用的 HTML 界面可以使用 JavaScript 框架或库来构建，例如 AngularJS 或 Backbone.js。在这种情况下，Laravel 表单包就不是必需的。另外，如前所述，Laravel 可以用来创建一个仅仅是 RESTful API 的应用程序。在这种情况下，将 HTML 包包含在框架核心中就不是必要的，因此仍然是辅助的。

在这种特殊情况下，某些 Laravel 包被移除以简化整体体验，并朝着更*基于组件*的方法迈进，这与 Symfony 中使用的方法类似。

# 安装 HTML 包

如果您希望在 Laravel 5 中使用 HTML 包，安装它是一个简单的过程。Laravel 社区的一群开发人员成立了一个名为 Laravel collective 的存储库，用于维护已从 Laravel 中移除的包。要安装 HTML 包，只需使用`composer`命令将包添加到应用程序中，如下所示：

```php
**$ composer require laravelcollective/html**

```

### 注意

请注意，`illuminate/HTML`包已被弃用。

这将安装 HTML 包，并且`composer.json`将显示您添加到`require`部分的包如下：

```php
"require": {
    "laravel/framework": "5.0.*",
    "laravelcollective/html": "~5.0",
  },
```

此时，包已安装。

现在，我们需要将`HTMLServiceProvider`添加到`config/app.php`文件中的提供者列表中：

```php
  'providers' => [
  ...
    'Collective\Html\HtmlServiceProvider',
  ...
  ],
```

最后，需要将`Form`和`Html`别名添加到`config/app.php`文件中，如下所示：

```php
'aliases' => [
   ...
        'Form' => 'Collective\Html\FormFacade',
        'Html' => 'Collective\Html\HtmlFacade',
   ...
  ],
```

# 使用 Laravel 构建网页

Laravel 构建 Web 内容的方法是灵活的。可以使用尽可能多或尽可能少的 Laravel 来创建 HTML。Laravel 使用`filename.blade.php`约定来说明文件应该由 blade 解析器解析，实际上将文件转换为普通的 PHP。Blade 的名称受到了.NET 的剃刀模板引擎的启发，因此对于曾经使用过它的人来说可能会很熟悉。Laravel 5 在`/resources/views/`目录中提供了一个表单的工作演示。当请求`/home`路由并且用户当前未登录时，将显示此视图。显然，这个表单并不是使用 Laravel 的表单方法创建的。

路由在`routes`文件中定义如下：

```php
Route::get('home', 'HomeController@index');
```

将讨论此路由如何使用中间件来检查如何执行用户身份验证，详见第七章，“使用中间件过滤请求”。

## 主模板

这是以下的`app`（或`master`）模板：

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Laravel</title>

    <link href="/css/app.css" rel="stylesheet">

    <!-- Fonts -->
    <link href='//fonts.googleapis.com/css?family=Roboto:400,300' rel='stylesheet' type='text/css'>

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
        <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
        <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
</head>
<body>
    <nav class="navbarnavbar-default">
        <div class="container-fluid">
            <div class="navbar-header">
                <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
                    <span class="sr-only">Toggle Navigation</span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                    <span class="icon-bar"></span>
                </button>
                <a class="navbar-brand" href="#">Laravel</a>
            </div>

            <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                <ul class="navnavbar-nav">
                    <li><a href="/">Home</a></li>
                </ul>

                <ul class="navnavbar-navnavbar-right">
                    @if (Auth::guest())
                        <li><a href="{{ route('auth.login') }}">Login</a></li>
                        <li><a href="/auth/register">Register</a></li>
                    @else
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-expanded="false">{{ Auth::user()->name }} <span class="caret"></span></a>
                            <ul class="dropdown-menu" role="menu">
                                <li><a href="/auth/logout">Logout</a></li>
                            </ul>
                        </li>
                    @endif
                </ul>
            </div>
        </div>
    </nav>

    @yield('content')

    <!-- Scripts -->
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.1/js/bootstrap.min.js"></script>
</body>
</html>
```

Laravel 5 主模板是一个具有以下特点的标准 HTML5 模板：

+   如果浏览器旧于 Internet Explorer 9：

+   使用 HTML5 Shim 来自 CDN

+   使用 Respond.js JavaScript 代码来自 CDN 以适应媒体查询和 CSS3 特性

+   使用`@if (Auth::guest())`，如果用户未经过身份验证，则显示登录表单；否则，显示注销选项。

+   Twitter bootstrap 3.x 包含在 CDN 中

+   jQuery2.x 包含在 CDN 中

+   任何扩展此模板的模板都可以覆盖内容部分

## 示例页面

以下截图显示了登录页面：

![示例页面](img/B04559_05_01.jpg)

登录页面的源代码如下：

```php
@extends('app')
@section('content')
<div class="container-fluid">
    <div class="row">
        <div class="col-md-8 col-md-offset-2">
            <div class="panel panel-default">
                <div class="panel-heading">Login</div>
                <div class="panel-body">
                    @if (count($errors) > 0)
                        <div class="alert alert-danger">
                            <strong>Whoops!</strong> There were some problems with your input.<br><br>
                            <ul>
                                @foreach ($errors->all() as $error)
                                    <li>{{ $error }}</li>
                                @endforeach
                            </ul>
                        </div>
                    @endif

                    <form class="form-horizontal" role="form" method="POST" action="/auth/login">
                        <input type="hidden" name="_token" value="{{ csrf_token() }}">

                        <div class="form-group">
                            <label class="col-md-4 control-label">E-Mail Address</label>
                            <div class="col-md-6">
                                <input type="email" class="form-control" name="email" value="{{ old('email') }}">
                            </div>
                        </div>

                        <div class="form-group">
                            <label class="col-md-4 control-label">Password</label>
                            <div class="col-md-6">
                                <input type="password" class="form-control" name="password">
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-md-6 col-md-offset-4">
                                <div class="checkbox">
                                    <label>
                                        <input type="checkbox" name="remember"> Remember Me
                                    </label>
                                </div>
                            </div>
                        </div>

                        <div class="form-group">
                            <div class="col-md-6 col-md-offset-4">
                                <button type="submit" lass="btn btn-primary" style="margin-right: 15px;">
                                    Login
                                </button>

                                <a href="/password/email">Forgot Your Password?</a>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

## 从静态 HTML 到静态方法

此登录页面以以下内容开始：

```php
@extends('app')
```

显然，它使用面向对象的范例来说明将呈现`app.blade.php`模板。以下行覆盖了内容：

```php
@section('content')
```

在这个练习中，将使用表单构建器而不是静态 HTML。

### 表单标签

我们将把静态的`form`标签转换为`FormBuilder`方法。HTML 如下：

```php
<form class="form-horizontal" role="form" method="POST" action="/auth/login">
```

我们将使用的外观方法如下：

```php
Form::open();
```

在`FormBuilder.php`类中，`$reserved`属性定义如下：

```php
protected $reserved = ['method', 'url', 'route', 'action', 'files'];
```

我们需要传递给`open()`方法的属性是 class、role、method 和 action。由于 method 和 action 是保留字，因此需要以以下方式传递参数：

| Laravel 表单外观方法数组元素 | HTML 表单标签属性 |
| --- | --- |
| 方法 | 方法 |
| url | action |
| role | role |
| class | class |

因此，方法调用如下：

```php
{!! 
  Form::open(['class'=>'form-horizontal',
  'role =>'form',
  'method'=>'POST',
  'url'=>'/auth/login']) 
!!}
```

`{!! !!}`标签用于开始和结束表单构建器方法的解析。`form`方法`POST`首先放置在 HTML 表单标签的属性列表中。

### 提示

`action`属性实际上需要是一个`url`。如果使用`action`参数，则它指的是控制器动作。在这种情况下，`url`参数会生成`form`标签的`action`属性。

其他属性将传递给数组并添加到属性列表中。生成的 HTML 将如下所示：

```php
<form method="POST" action="http://laravel.example/auth/login" accept-charset="UTF-8" class="form-horizontal" role="form">

<input name="_token" type="hidden" value="wUY2hFSEWCzKHFfhywHvFbq9TXymUDiRUFreJD4h">
```

CRSF 令牌会自动添加，因为`form`方法是`POST`。

### 文本输入字段

要转换输入字段，使用外观。输入字段的 HTML 如下：

```php
<input type="email" class="form-control" name="email" value="{{ old('email') }}">
```

使用外观转换前面的输入字段如下：

```php
{!! Form::input('email','email',old('email'),['class'=>'form-control' ]) !!}
```

同样，文本字段变为：

```php
{!! Form::input('password','password',null,['class'=>'form-control']) !!}
```

输入字段具有相同的签名。当然，这可以重构如下：

```php
<?php $inputAttributes = ['class'=>'form-control'] ?>
{!! Form::input('email','email',old('email'),$inputAttributes ) !!}
...
{!! Form::input('password','password',null,$inputAttributes ) !!}
```

### 标签标签

`label`标签如下：

```php
<label class="col-md-4 control-label">E-Mail Address</label>
<label class="col-md-4 control-label">Password</label>
```

要转换`label`标签（`E-Mail Address`和`Password`），我们首先创建一个数组来保存属性，然后将此数组传递给标签，如下所示：

```php
$labelAttributes = ['class'=>'col-md-4 control-label'];
```

以下是表单标签代码：

```php
{!! Form::label('email', 'E-Mail Address', $labelAttributes) !!}
{!! Form::label('password', 'Password', $labelAttributes) !!}
```

### 复选框

要将复选框转换为外观，我们将转换为：

```php
<input type="checkbox" name="remember"> Remember Me
```

前面的代码转换为以下代码：

```php
{!! Form::checkbox('remember','') !!} Remember Me
```

### 提示

请记住，如果字符串中没有变量或其他特殊字符（如换行符），则应该用单引号发送 PHP 参数，而生成的 HTML 将使用双引号。

### 提交按钮

最后，提交按钮将被转换如下：

```php
<button type="submit" class="btn btn-primary" style="margin-right: 15px;">
    Login
</button>
```

转换后的前一行代码如下：

```php
    {!! 
        Form::submit('Login',
        ['class'=>'btn btn-primary', 
        'style'=>'margin-right: 15px;'])
     !!}
```

### 提示

请注意，数组参数提供了一种简单的方式来提供任何所需的属性，甚至那些不在标准 HTML 表单元素列表中的属性。

### 带有链接的锚标签

为了转换链接，使用了一个辅助方法。考虑以下代码行：

```php
<a href="/password/email">Forgot Your Password?</a>
```

转换后的前一行代码如下：

```php
{!! link_to('/password/email', $title = 'Forgot Your Password?', $attributes = array(), $secure = null) !!}
```

### 注意

`link_to_route()`方法可用于链接到一个路由。有关类似的辅助函数，请访问[`laravelcollective.com/docs/5.0/html`](http://laravelcollective.com/docs/5.0/html)。

### 关闭表单

为了结束表单，我们将把传统的 HTML 表单标签`</form>`转换为 Laravel 的`{!! Form::close() !!}`表单方法。

### 结果表单

将所有内容放在一起后，页面现在看起来是这样的：

```php
@extends('app')
@section('content')
<div class="container-fluid">
  <div class="row">
    <div class="col-md-8 col-md-offset-2">
      <div class="panel panel-default">
        <div class="panel-heading">Login</div>
          <div class="panel-body">
            @if (count($errors) > 0)
                <div class="alert alert-danger">
                    <strong>Whoops!</strong> There were some problems with your input.<br><br>
                    <ul>
                        @foreach ($errors->all() as $error)
                            <li>{{ $error }}</li>
                        @endforeach
                    </ul>
                </div>
            @endif
            <?php $inputAttributes = ['class'=>'form-control'];
                $labelAttributes = ['class'=>'col-md-4 control-label']; ?>
            {!! Form::open(['class'=>'form-horizontal','role'=>'form','method'=>'POST','url'=>'/auth/login']) !!}
                <div class="form-group">
                    {!! Form::label('email', 'E-Mail Address',$labelAttributes) !!}
                    <div class="col-md-6">
                    {!! Form::input('email','email',old('email'), $inputAttributes) !!}
                    </div>
                </div>
                <div class="form-group">
                    {!! Form::label('password', 'Password',$labelAttributes) !!}
                    <div class="col-md-6">
                        {!! Form::input('password','password',null,$inputAttributes) !!}
                    </div>
                </div>
                <div class="form-group">
                    <div class="col-md-6 col-md-offset-4">
                        <div class="checkbox">
                          <label>
                             {!! Form::checkbox('remember','') !!} Remember Me
                          </label>
                        </div>
                    </div>
                </div>
                <div class="form-group">
                    <div class="col-md-6 col-md-offset-4">
                        {!! Form::submit('Login',['class'=>'btn btn-primary', 'style'=>'margin-right: 15px;']) !!}
                        {!! link_to('/password/email', $title = 'Forgot Your Password?', $attributes = array(), $secure = null); !!}
                    </div>
                </div>
            {!! Form::close() !!}
          </div>
      </div>
    </div>
  </div>
</div>
@endsection
```

# 我们的例子

如果我们想要创建一个预订住宿的表单，我们可以轻松地从我们的控制器中调用一个路由：

```php
/**
 * Show the form for creating a new resource.
 *
 * @return Response
 */
public function create()
{
    return view('auth/reserve');
}
```

现在我们需要创建一个位于`resources/views/auth/reserve.blade.php`的新视图。

在这个视图中，我们可以创建一个表单来预订住宿，用户可以选择开始日期，其中包括月份和年份的开始日期，以及结束日期，也包括月份和年份的开始日期：

![我们的例子](img/B04559_05_02.jpg)

表单将如前所述开始，通过 POST 到`reserve-room`。然后，表单标签将放置在选择输入字段旁边。最后，日期、月份和年份选择表单元素将被创建如下：

```php
{!! Form::open(['class'=>'form-horizontal',
        'role'=>'form', 
        'method'=>'POST', 
        'url'=>'reserve-room']) !!}
        {!! Form::label(null, 'Start Date',$labelAttributes) !!}

        {!! Form::selectMonth('month',date('m')) !!}
        {!! Form::selectRange('date',1,31,date('d')) !!}
        {!! Form::selectRange('year',date('Y'),date('Y')+3) !!}

        {!! Form::label(null, 'End Date',$labelAttributes) !!}

        {!! Form::selectMonth('month',date('m')) !!}
        {!! Form::selectRange('date',1,31,date('d')) !!}
        {!! Form::selectRange('year',date('Y'),date('Y')+3,date('Y')) !!}

        {!! Form::submit('Reserve',
        ['class'=>'btn btn-primary', 
        'style'=>'margin-right: 15px;']) !!}
{!! Form::close() !!}
```

## 月份选择

首先，在`selectMonth`方法中，第一个参数是输入属性的名称，而第二个属性是默认值。这里，PHP 日期方法被用来提取当前月份的数字部分——在这种情况下是三月：

```php
**{!! Form::selectMonth('month',date('m')) !!}**

```

格式化后的输出如下：

```php
<select name="month">
    <option value="1">January</option>
    <option value="2">February</option>
    <option value="3" selected="selected">March</option>
    <option value="4">April</option>
    <option value="5">May</option>
    <option value="6">June</option>
    <option value="7">July</option>
    <option value="8">August</option>
    <option value="9">September</option>
    <option value="10">October</option>
    <option value="11">November</option>
    <option value="12">December</option>
</select>
```

## 日期选择

类似的技术也适用于选择日期，但是使用`selectRange`方法，将月份中的日期范围传递给该方法。同样，PHP 日期函数被用来将当前日期作为第四个参数传递给该方法：

```php
{!! Form::selectRange('date',1,31,date('d')) !!}
```

这里是格式化后的输出：

```php
<select name="date">
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="4">4</option>
    ...
    <option value="28">28</option>
    <option value="29">29</option>
    <option value="30" selected="selected">30</option>
    <option value="31">31</option>
</select>
```

应该选择的日期是 30，因为今天是 2015 年 3 月 30 日。

### 提示

对于没有 31 天的月份，通常会使用 JavaScript 方法根据月份和/或年份修改天数。

## 年份选择

用于日期范围的相同技术也适用于年份的选择；再次使用`selectRange`方法。年份范围被传递给该方法。PHP 日期函数被用来将当前年份作为第四个参数传递给该方法：

```php
{!! Form::selectRange('year',date('Y'),date('Y')+3,date('Y')) !!}
```

这里是格式化后的输出：

```php
<select name="year">
    <option value="2015" selected="selected">2015</option>
    <option value="2016">2016</option>
    <option value="2017">2017</option>
    <option value="2018">2018</option>
</select>
```

这里，选择的当前年份是 2015 年。

## 表单宏

我们有相同的代码，用于生成我们的月份、日期和年份选择表单块两次：一次用于开始日期，一次用于结束日期。为了重构代码，我们可以应用 DRY（不要重复自己）原则并创建一个表单宏。这将允许我们避免两次调用表单元素创建方法，如下所示：

```php
<?php
Form::macro('monthDayYear',function($suffix='')
{
    echo Form::selectMonth(($suffix!=='')?'month-'.$suffix:'month',date('m'));
    echo Form::selectRange(($suffix!=='')?'date-'.$suffix:'date',1,31,date('d'));
    echo Form::selectRange(($suffix!=='')?'year-'.$suffix:'year',date('Y'),date('Y')+3,date('Y'));
}); 
?>
```

这里，月份、日期和年份生成代码被放入一个宏中，该宏位于 PHP 标签内，并且需要添加`echo`来打印结果。给这个宏方法取名为`monthDayYear`。调用我们的宏两次：每个标签后调用一次；每次通过`$suffix`变量添加不同的后缀。现在，我们的表单代码看起来是这样的：

```php
<?php
Form::macro('monthDayYear',function($suffix='')
{
    echo Form::selectMonth(($suffix!=='')?'month-'.$suffix:'month',date('m'));
    echo Form::selectRange(($suffix!=='')?'date-'.$suffix:'date',1,31,date('d'));
    echo Form::selectRange(($suffix!=='')?'year-'.$suffix:'year',date('Y'),date('Y')+3,date('Y'));
});
?>
{!! Form::open(['class'=>'form-horizontal',
                'role'=>'form',
                'method'=>'POST',
                'url'=>'/reserve-room']) !!}
    {!! Form::label(null, 'Start Date',$labelAttributes) !!}
    {!! Form::monthDayYear('-start') !!}
    {!! Form::label(null, 'End Date',$labelAttributes) !!}
    {!! Form::monthDayYear('-end') !!}
    {!! Form::submit('Reserve',['class'=>'btn btn-primary',
           'style'=>'margin-right: 15px;']) !!}
{!! Form::close() !!}
```

# 结论

在 Laravel 5 中选择包含 HTML 表单生成包可以减轻创建大量 HTML 表单的负担。这种方法允许开发人员使用方法，创建可重用的宏，并使用熟悉的 Laravel 方法来构建前端。一旦学会了基本方法，就可以很容易地复制和粘贴以前创建的表单元素，然后更改它们的元素名称和/或发送给它们的数组。

根据项目的大小，这种方法可能是正确的选择，也可能不是。对于非常小的应用程序，需要编写的代码量的差异并不明显，尽管，如`selectMonth`和`selectRange`方法所示，所需的代码量是 drastc 的。

这种技术与宏的使用结合起来，可以轻松减少复制重复的发生。此外，前端设计的一个主要问题是各种元素的类的内容可能需要在整个应用程序中进行更改。这意味着需要执行大量的查找和替换操作，需要对 HTML 进行更改，例如更改类属性。通过创建包含类等属性的数组，可以通过修改这些元素使用的数组来执行对整个表单的更改。

然而，在一个更大的项目中，表单的部分可能在整个应用程序中重复，明智地使用宏可以轻松减少需要编写的代码量。不仅如此，宏还可以将代码与多个文件中需要更改的更改隔离开来。在要选择月份、日期和年份的示例中，这在一个大型应用程序中可能会被使用多达 20 次。对所需的 HTML 块进行的任何更改可以简单地通过修改这个宏来反映在使用它的所有元素中。

最终，是否使用此包的选择将由开发人员和设计人员决定。由于想要使用替代前端设计工具的设计人员可能不喜欢也可能不熟练地使用包中的方法，因此可能不想使用它。

# 总结

在本章中，概述了 HTML Laravel composer 包的历史和安装。解释了主模板的构建，然后通过示例展示了表单组件，如各种表单输入类型。

最后，解释了在书中示例软件中使用的房间预订表单的构建，以及“不要重复自己”的表单宏创建技术。

在下一章中，我们将看一种使用注释来减少应用程序控制器创建路由所需时间的方法。
