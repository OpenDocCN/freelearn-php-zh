# 第二章。使用表单和收集输入

在本章中，我们将涵盖：

+   创建一个简单的表单

+   收集表单输入以在另一页上显示

+   验证用户输入

+   创建一个文件上传器

+   验证文件上传

+   创建自定义错误消息

+   向表单添加“蜜罐”

+   使用 Redactor 上传图像

+   使用 Jcrop 裁剪图像

+   创建一个自动完成文本输入

+   制作一个验证码样式的垃圾邮件捕捉器

# 介绍

在本章中，我们将学习如何在 Laravel 中使用表单，以及如何完成一些典型的任务。我们将从一些简单的表单验证和文件上传开始，然后继续将一些前端工具，如 Redactor 和 jCrop，整合到 Laravel 中。

# 创建一个简单的表单

任何 Web 应用程序的最基本方面之一是表单。Laravel 提供了一种简单的方法来为我们的表单构建 HTML。

## 准备工作

要开始，我们需要一个全新的 Laravel 安装。

## 操作步骤...

要完成此示例，请按照以下步骤操作：

1.  在`app/views`文件夹中，创建一个新的`userform.php`文件。

1.  在`routes.php`中，创建一个路由来加载视图：

```php
Route::get(userform, function()
{
    return View::make('userform');
});
```

1.  在`userform.php`视图中，使用以下代码创建一个表单：

```php
<h1>User Info</h1>
<?= Form::open() ?>
<?= Form::label('username', 'Username') ?>
<?= Form::text('username') ?>
<br>
<?= Form::label('password', 'Password') ?>
<?= Form::password('password') ?>
<br>
<?= Form::label('color', 'Favorite Color') ?>
<?= Form::select('color', array('red' => 'red', 'green' =>'green', 'blue' => 'blue')) ?>
<br>
<?= Form::submit('Send it!') ?>
<?= Form::close() ?>
```

通过转到`http://{your-server}/userform`（其中`{your-server}`是您的服务器的名称）在 Web 页面中查看您的表单。

## 工作原理...

对于这个任务，我们使用 Laravel 内置的`Form`类创建了一个简单的表单。这使我们能够轻松地使用最少的代码创建表单元素，并且它符合 W3C（万维网联盟）标准。

首先，我们打开表单。Laravel 会自动创建`<form>`html，包括 action、method 和 accept-charset 参数。当没有传递选项时，默认操作是当前 URL，默认方法是`POST`，字符集取自应用配置文件。

接下来，我们创建普通文本和密码输入字段，以及它们的标签。标签中的第一个参数是文本字段的名称，第二个参数是要打印的实际文本。在表单构建器中，标签应该出现在实际表单输入之前。

表单选择需要第二个参数，即下拉框中值的数组。在本例中，我们使用`'key' => 'value'`语法创建一个数组。如果我们想创建选项组，我们只需要创建嵌套数组。

最后，我们创建了提交按钮并关闭了表单。

## 还有更多...

大多数 Laravel 的表单方法也可以包括默认值和自定义属性（类、ID 等）的参数。如果我们不想使用特定的方法，我们也可以对许多字段使用`Form::input()`。例如，我们可以使用`Form::input('submit', NULL, 'Send it!')`来创建一个提交按钮。

## 另请参阅

+   *收集表单输入以在另一页上显示*示例

# 收集表单输入以在另一页上显示

用户提交表单后，我们需要能够获取该信息并将其传递到另一页。这个示例展示了我们如何使用 Laravel 的内置方法来处理我们的 POST 数据。

## 准备工作

我们需要从*创建一个简单的表单*部分设置简单的表单。

## 操作步骤...

按照以下步骤完成此示例：

1.  创建一个路由来处理表单中的 POST 数据：

```php
Route::post('userform', function()
{
    // Process the data here
    return Redirect::to('userresults')-
        >withInput(Input::only('username', 'color'));
});

```

1.  创建一个重定向到的路由，并显示数据：

```php
Route::get('userresults', function()
{
    return 'Your username is: ' . Input::old('username')
        . '<br>Your favorite color is: '
        . Input::old('color');
});

```

## 工作原理...

在我们的简单表单中，我们将数据 POST 回相同的 URL，因此我们需要创建一个接受相同路径的`POST`路由。这是我们将对数据进行任何处理的地方，包括保存到数据库或验证输入。

在这种情况下，我们只是想将数据传递到下一页。有许多方法可以实现这一点。例如，我们可以使用`Input`类的`flashOnly()`方法：

```php
Route::post('userform', function()
{
    Input::flashOnly('username', 'color');
    return Redirect::to('userresults');
});
```

但是，我们使用了 Laravel 提供的一个快捷方式，只传递了我们要求的三个表单字段中的两个。

在下一页上，我们使用`Input::old()`来显示闪存输入。

## 另请参阅

+   *创建一个简单的表单*示例

# 验证用户输入

在大多数 Web 应用程序中，将需要某些必填的表单字段来处理表单。我们还希望确保所有电子邮件地址的格式正确，或者输入必须具有一定数量的字符。使用 Laravel 的`Validator`类，我们可以检查这些规则，并让用户知道是否有不正确的地方。

## 准备工作

对于这个食谱，我们只需要一个标准的 Laravel 安装。

## 如何做...

完成这个食谱，按照以下步骤进行：

1.  创建一个路由来保存表单：

```php
Route::get('userform', function()
{
    return View::make('userform');
});
```

1.  创建一个名为`userform.php`的视图并添加一个表单：

```php
<h1>User Info</h1>
<?php $messages =  $errors->all('<pstyle="color:red">:message</p>') ?>
<?php
foreach ($messages as $msg)
{
    echo $msg;
}
?>
<?= Form::open() ?>
<?= Form::label('email', 'Email') ?>
<?= Form::text('email', Input::old('email')) ?>
<br>
<?= Form::label('username', 'Username') ?>
<?= Form::text('username', Input::old('username')) ?>
<br>
<?= Form::label('password', 'Password') ?>
<?= Form::password('password') ?>
<br>
<?= Form::label('password_confirm', 'Retype your Password')?>
<?= Form::password('password_confirm') ?>
<br>
<?= Form::label('color', 'Favorite Color') ?>
<?= Form::select('color', array('red' => 'red', 'green' =>'green', 'blue' => 'blue'), Input::old('color')) ?>
<br>
<?= Form::submit('Send it!') ?>
<?php echo Form::close() ?>
```

1.  创建一个处理我们的`POST`数据并验证它的路由：

```php
Route::post('userform', function()
{
    $rules = array(
        'email' => 'required|email|different:username',
        'username' => 'required|min:6',
        'password' => 'required|same:password_confirm'
    );
    $validation = Validator::make(Input::all(), $rules);

    if ($validation->fails())
    {
        return Redirect::to('userform')-
            >withErrors($validation)->withInput();
    }

    return Redirect::to('userresults')->withInput();

});

```

1.  创建一个路由来处理成功的表单提交：

```php
Route::get('userresults', function()
{
    return dd(Input::old());
});
```

## 它是如何工作的...

在我们的表单页面中，我们首先检查是否有任何错误，并在找到错误时显示它们。在错误内部，我们可以为每个错误消息设置默认样式。我们还可以选择使用`$errors->get('email')`来检查并显示单个字段的错误。如果 Laravel 检测到闪存错误，`$errors`变量将自动创建。

接下来，我们创建我们的表单。在表单元素的最后一个参数中，我们获取`Input::old()`，如果验证失败，我们将使用它来存储先前的输入。这样，用户就不需要一直填写整个表单。

然后创建一个路由，表单被 POST 提交，并设置我们的验证规则。在这种情况下，我们对`email`，`username`和`password`使用必填规则，以确保这些字段中有内容输入。

`email`字段还使用`email`规则，该规则使用 PHP 的内置`FILTER_VALIDATE_EMAIL`过滤器的`filter_var`函数。`email`字段也不能与`username`字段相同。`username`字段使用大小验证来检查至少六个字符。然后`password`字段检查`password_confirm`字段的值，并确保它们相同。

然后，我们创建验证器并传入所有表单数据。如果其中任何规则不符合，我们将用户导航回表单，并返回任何验证错误消息以及原始表单输入。

如果验证通过，我们使用 Laravel 的`dd()`辅助函数转到下一个页面，该函数使用`var_dump()`在页面上显示表单值。

## 另请参阅

+   *创建一个简单的表单食谱*

# 创建一个文件上传程序

有时我们希望用户将文件上传到我们的服务器。这个食谱展示了 Laravel 如何通过 Web 表单处理文件上传。

## 准备工作

要创建一个文件上传程序，我们需要安装标准版本的 Laravel。

## 如何做...

完成这个食谱，按照以下步骤进行：

1.  在我们的`routes.php`文件中创建一个路由来保存表单：

```php
Route::get('fileform', function()
{
    return View::make('fileform');
});
```

1.  在我们的`app/views`目录中创建`fileform.php`视图：

```php
<h1>File Upload</h1>
<?= Form::open(array('files' => TRUE)) ?>
<?= Form::label('myfile', 'My File') ?>
<br>
<?= Form::file('myfile') ?>
<br>
<?= Form::submit('Send it!') ?>
<?= Form::close() ?>
```

1.  创建一个路由来上传和保存文件：

```php
Route::post('fileform', function()
{
    $file = Input::file('myfile');
    $ext = $file->guessExtension();
    if ($file->move('files', 'newfilename.' . $ext))
    {
        return 'Success';
    }
    else
    {
        return 'Error';
    }
});
```

## 它是如何工作的...

在我们的视图中，我们使用`Form::open()`并传入一个数组，其中包含`'files' => TRUE`，这会自动设置`Form`标签中的 enctype；然后我们添加一个表单字段来接受文件。在`Form::open()`中不使用任何其他参数，表单将使用默认的`POST`方法和当前 URL 的操作。`Form::file()`是我们接受文件的输入字段。

由于我们的表单正在提交到相同的 URL，我们需要创建一个路由来接受`POST`输入。`$file`变量将保存所有文件信息。

接下来，我们想要使用不同的名称保存文件，但首先我们需要获取上传文件的扩展名。因此，我们使用`guessExtension()`方法，并将其存储在一个变量中。大多数文件使用方法都可以在 Symfony 的文件库中找到。

最后，我们使用文件的`move()`方法将文件移动到其永久位置，第一个参数是我们将保存文件的目录；第二个是文件的新名称。

如果一切上传正确，我们显示`'Success'`，如果不是，我们显示`'Error'`。

## 另请参阅

+   *验证文件上传*食谱

# 验证文件上传

如果我们希望允许用户通过我们的网络表单上传文件，我们可能希望限制他们上传的文件类型。使用 Laravel 的`Validator`类，我们可以检查特定的文件类型，甚至限制上传到特定文件大小。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装和一个示例文件来测试我们的上传。

## 如何做...

按照以下步骤完成这个配方：

1.  在我们的`routes.php`文件中为表单创建一个路由：

```php
Route::get('fileform', function()
{
    return View::make('fileform');
});
```

1.  创建表单视图：

```php
<h1>File Upload</h1>
<?php $messages =  $errors->all('<p style="color:red">:message</p>') ?>
<?php
foreach ($messages as $msg)
{
    echo $msg;
}
?>
<?= Form::open(array('files' => TRUE)) ?>
<?= Form::label('myfile', 'My File (Word or Text doc)') ?>
<br>
<?= Form::file('myfile') ?>
<br>
<?= Form::submit('Send it!') ?>
<?= Form::close() ?>
```

1.  创建一个路由来验证和处理我们的文件：

```php
Route::post('fileform', function()
{
    $rules = array(
        'myfile' => 'mimes:doc,docx,pdf,txt|max:1000'
    );
    $validation = Validator::make(Input::all(), $rules);

    if ($validation->fails())
    {
return Redirect::to('fileform')->withErrors($validation)->withInput();
    }
    else
    {
        $file = Input::file('myfile');
        if ($file->move('files', $file->getClientOriginalName()))
        {
            return "Success";
        }
        else 
        {
            return "Error";
        }
    }
});
```

## 它是如何工作的...

我们首先创建一个用于保存我们的表单的路由，然后是表单的 html 视图。在视图顶部，如果我们在验证中得到任何错误，它们将在这里被输出。表单以`Form::open (array('files' => TRUE))`开始，这将为我们设置默认的操作、方法和`enctype`。

接下来，我们创建一个路由来捕获 POST 数据并验证它。我们将一个`$rules`变量设置为一个数组，首先检查特定的 MIME 类型。我们可以使用尽可能少或尽可能多的规则。然后我们确保文件小于 1000 千字节，或 1 兆字节。

如果文件无效，我们将用户导航回带有错误消息的表单。如果 Laravel 检测到闪存的错误消息，`$error`变量会自动在我们的视图中创建。如果它是有效的，我们尝试将文件保存到服务器。如果保存正确，我们将看到`"Success"`，如果不是，我们将看到`"Error"`。

## 还有更多...

文件的另一个常见验证是检查图像。为此，我们可以在我们的`$rules`数组中使用以下内容：

```php
'myfile' => 'image'
```

这将检查文件是否是`.jpg`、`.png`、`.gif`或`.bmp`文件。

## 另请参阅

+   *创建文件上传器*配方

# 创建自定义错误消息

如果验证失败，Laravel 内置了错误消息，但我们可能希望自定义这些消息，使我们的应用程序变得独特。这个配方展示了创建自定义错误消息的几种不同方法。

## 准备工作

对于这个配方，我们只需要一个标准的 Laravel 安装。

## 如何做...

要完成这个配方，请按照以下步骤：

1.  在`routes.php`中创建一个路由来保存表单：

```php
Route::get('myform', function()
{
    return View::make('myform');
});
```

1.  创建一个名为`myform.php`的视图并添加一个表单：

```php
<h1>User Info</h1>
<?php $messages =  $errors->all
    ('<p style="color:red">:message</p>') ?>
<?php
foreach ($messages as $msg) 
{
    echo $msg;
}
?>
<?= Form::open() ?>
<?= Form::label('email', 'Email') ?>
<?= Form::text('email', Input::old('email')) ?>
<br>
<?= Form::label('username', 'Username') ?>
<?= Form::text('username', Input::old('username')) ?>
<br>
<?= Form::label('password', 'Password') ?>
<?= Form::password('password') ?>
<br>
<?= Form::submit('Send it!') ?>
<?= Form::close() ?>
```

1.  创建一个路由来处理我们的 POST 数据并验证它：

```php
Route::post('myform', array('before' => 'csrf', function()
{
    $rules = array(
        'email'    => 'required|email|min:6',
        'username' => 'required|min:6',
        'password' => 'required'
    );

    $messages = array(
        'min' => 'Way too short! The :attribute must be atleast :min characters in length.',
        'username.required' => 'We really, really need aUsername.'
    );

    $validation = Validator::make(Input::all(), $rules,$messages);

    if ($validation->fails())
    {
        return Redirect::to('myform')->withErrors($validation)->withInput();
    }

    return Redirect::to('myresults')->withInput();
}));
```

1.  打开文件`app/lang/en/validation.php`，其中`en`是应用程序的默认语言。在我们的情况下，我们使用的是英语。在文件底部，更新`attributes`数组如下：

```php
'attributes' => array(
    'password' => 'Super Secret Password (shhhh!)'
),
```

1.  创建一个路由来处理成功的表单提交：

```php
Route::get('myresults', function()
{
    return dd(Input::old());
});
```

## 它是如何工作的...

我们首先创建一个相当简单的表单，由于我们没有向`Form::open()`传递任何参数，它将把数据 POST 到相同的 URL。然后我们创建一个路由来接受`POST`数据并验证它。作为最佳实践，我们还在我们的`post`路由之前添加了`csrf`过滤器。这将提供一些额外的安全性，防止跨站点请求伪造。

我们在`post`路由中设置的第一个变量将保存我们的规则。下一个变量将保存我们希望在出现错误时使用的任何自定义消息。有几种不同的方法来设置消息。

自定义的第一个消息是`min`大小。在这种情况下，它将显示相同的消息，用于任何验证错误，其中有一个`min`规则。我们可以使用`:attribute`和`:min`来保存表单字段名称和错误显示时的最小大小。

我们的第二个消息仅用于特定的表单字段和特定的验证规则。我们首先放置表单字段名称，然后是一个句号，然后是规则。在这里，我们正在检查用户名是否必填，并设置错误消息。

我们的第三个消息是在验证的语言文件中设置的。在`attributes`数组中，我们可以将我们的任何表单字段名称设置为显示我们想要的任何自定义文本。此外，如果我们决定自定义整个应用程序中的特定错误消息，我们可以在该文件的顶部更改默认消息。

## 还有更多...

如果我们查看`app/lang`目录，我们会看到许多翻译已经是 Laravel 的一部分。如果我们的应用程序是本地化的，我们可以选择任何语言设置自定义验证错误消息。

## 另请参阅

+   *创建一个简单的表单*教程

# 向表单添加蜜罐

网络的一个悲哀现实是存在“垃圾邮件机器人”，它们搜索网络并寻找要提交垃圾邮件的表单。帮助应对这一问题的一种方法是使用一种称为**蜜罐**的技术。在这个教程中，我们将创建一个自定义验证来检查垃圾邮件提交。

## 准备工作

对于这个教程，我们只需要一个标准的 Laravel 安装。

## 如何做...

要完成这个教程，请按照以下步骤进行：

1.  在`routes.php`中创建一个路由来保存我们的表单：

```php
Route::get('myform', function()
{
    return View::make('myapp');
});
```

1.  在我们的`app/view`目录中创建一个名为`myform.php`的视图，并添加表单：

```php
<h1>User Info</h1>
<?php $messages =  $errors->all('<p style ="color:red">:message</p>') ?>
<?php
foreach ($messages as $msg)
{
    echo $msg;
}
?>
<?= Form::open() ?>
<?= Form::label('email', 'Email') ?>
<?= Form::text('email', Input::old('email')) ?>
<br>
<?= Form::label('username', 'Username') ?>
<?= Form::text('username', Input::old('username')) ?>
<br>
<?= Form::label('password', 'Password') ?>
<?= Form::password('password') ?>
<?= Form::text('no_email', '', array('style' =>'display:none')) ?>
<br>
<?= Form::submit('Send it!') ?>
<?= Form::close() ?>
```

1.  在我们的`routes.php`文件中创建一个路由来处理`post`数据，并对其进行验证：

```php
Route::post('myform', array('before' => 'csrf', function()
{
    $rules = array(
        'email'    => 'required|email',
        'password' => 'required',
        'no_email' => 'honey_pot'
    );
    $messages = array(
        'honey_pot' => 'Nothing should be in this field.'
    );
    $validation = Validator::make(Input::all(), $rules,$messages);

    if ($validation->fails())
    {
        return Redirect::to('myform')->withErrors($validation)->withInput();
    }

    return Redirect::to('myresults')->withInput();
}));
```

1.  在我们的`routes.php`文件中，创建一个自定义验证：

```php
Validator::extend('honey_pot', function($attribute, $value,$parameters)
{
    return $value == '';
});
```

1.  创建一个简单的路由用于成功页面：

```php
Route::get('myresults', function()
{
    return dd(Input::old());
});
```

## 它是如何工作的...

我们首先创建一个相当简单的表单；因为我们没有向`Form::open()`传递任何参数，它将把数据 POST 到相同的 URL。在表单中，我们创建一个旨在为空的字段，但使用 CSS 隐藏它。通过将其命名为带有`email`一词的内容，许多垃圾邮件机器人会误以为它是一个`email`字段并尝试填充它。

然后，我们创建一个路由来接受`post`数据并对其进行验证，并在路由之前添加一个`csrf`过滤器。我们为我们的`no_email`字段添加一个自定义验证规则，以确保该字段保持为空。我们还在`$messages`数组中为该规则创建一个错误消息。

接下来，我们实际上在`routes`文件中创建我们的自定义验证规则。这个规则将从表单字段获取值，并在值为空时返回`TRUE`。

现在，如果一个机器人试图填写整个表单，它将无法验证，因为额外的字段设计为保持为空。

## 还有更多...

创建自定义验证的另一种选择是使用规则`size: 0`，这将确保`honey_pot`字段的长度正好为`0`个字符。然而，这种方法使验证检查变得简单得多。

我们可能还希望将任何蜜罐错误重定向到另一个没有表单的页面。这样，任何自动表单提交脚本都不会继续尝试提交表单。

# 使用 Redactor 上传图片

有一些不同的 JavaScript 库可以将表单的文本区域转换为所见即所得的编辑器。Redactor 是一个较新的库，但编码非常好，并在短时间内获得了相当大的流行。在这个教程中，我们将把 Redactor 应用到我们的 Laravel 表单中，并创建路由以允许通过 Redactor 上传图片。

## 准备工作

我们需要从[`github.com/dybskiy/redactor-js/tree/master/redactor`](https://github.com/dybskiy/redactor-js/tree/master/redactor)下载 Redactor 的副本。下载`redactor.min.js`并保存到`public/js`目录。下载`redactor.css`并保存到`public/css`目录。

## 如何做...

要完成这个教程，请按照以下步骤进行：

1.  在我们的`routes.php`文件中创建一个路由来保存带有`redactor`字段的表单：

```php
Route::get('redactor', function() 
{
    return View::make('redactor');
});
```

1.  在我们的`app/views`目录中创建一个名为`redactor.php`的视图：

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Laravel and Redactor</title>
        <meta charset="utf-8">
        <link rel="stylesheet" href="css/redactor.css" />
        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        <script src="js/redactor.min.js"></script>
    </head>
    <body>
        <?= Form::open() ?>
        <?= Form::label('mytext', 'My Text') ?>
        <br>
        <?= Form::textarea('mytext', '', array('id' =>'mytext')) ?>
        <br>
        <?= Form::submit('Send it!') ?>
        <?= Form::close() ?>
        <script type="text/javascript">
            $(function() {
                $('#mytext').redactor({
                    imageUpload: 'redactorupload'
                });
            });
        </script>
    </body>
</html>
```

1.  创建一个处理图片上传的路由：

```php
Route::post('redactorupload', function()
{
    $rules = array(
        'file' => 'image|max:10000'
    );
    $validation = Validator::make(Input::all(), $rules);
    $file = Input::file('file');
    if ($validation->fails())
    {
        return FALSE;
    }
    else
    {
        if ($file->move('files', $file->
            getClientOriginalName()))
        {
            return Response::json(array('filelink' =>
               'files/' . $file->getClientOriginalName()));
        }
        else
        {
            return FALSE;
        }
    }
});

```

1.  创建另一个路由来显示我们的表单输入后。

```php
Route::post('redactor', function() 
{
    return dd(Input::all());
});
```

## 它是如何工作的...

创建完我们的表单路由后，我们创建视图来保存我们的表单 HTML。在页面的头部，我们加载 redactor CSS，jquery 库（使用 Google 的 CDN），和 redactor JavaScript 文件。

我们的表单只有一个字段，一个名为`mytext`的文本区域。在我们的脚本区域中，我们在文本区域字段上初始化 Redactor，并将`imageUpload`参数设置为一个接受图片上传的路由或控制器。我们的设置为`redactorupload`，所以我们为它创建一个接受`post`数据的路由。

在我们的`redactorupload`路由中，我们进行一些验证，如果一切正常，图像将上传到我们的图像目录。要在文本区域中显示图像，它需要一个带有文件链接的 JSON 数组作为键，图像路径作为值。为此，我们将使用 Laravel 内置的`Response::json`方法，并传入一个带有图像位置的数组。

在我们的表单页面上，如果图像验证和上传正确，Redactor 将在文本区域内显示图像。如果我们提交，我们将看到文本包括`<img>`标签和图像路径。

## 还有更多...

虽然这个示例是专门用于图像上传的，但非图像文件上传的工作方式非常类似。唯一的真正区别是文件上传路由还应该在 JSON 输出中返回文件名。

# 使用 Jcrop 裁剪图像

图像编辑和处理有时可能是我们应用程序中难以实现的事情。使用 Laravel 和 Jcrop JavaScript 库，我们可以使任务变得更简单。

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)购买的所有 Packt 图书的帐户中下载示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件发送到您的电子邮件。

## 准备工作

我们需要从[`deepliquid.com/content/Jcrop_Download.html`](http://deepliquid.com/content/Jcrop_Download.html)下载 Jcrop 库并解压缩。将文件`jquery.Jcrop.min.js`放入我们的`public/js`目录，将`jquery.Jcrop.min.css`和`Jcrop.gif`文件放入我们的`public/css`目录。我们将使用 Google CDN 版本的 jQuery。我们还需要确保在服务器上安装了 GD 库，以便进行图像处理。在我们的`public`目录中，我们需要一个图像文件夹来存储图像，并且应该对其进行可写权限设置。

## 如何做...

按照以下步骤完成此示例：

1.  让我们在我们的`routes.php`文件中创建一个路由来保存我们的表单：

```php
Route::get('imageform', function()
{
    return View::make('imageform');
});
```

1.  在`app/views`中创建用于上传图像的表单，文件名为`imageform.php`：

```php
<h1>Laravel and Jcrop</h1>
<?= Form::open(array('files' => true)) ?>
<?= Form::label('image', 'My Image') ?>
<br>
<?= Form::file('image') ?>
<br>
<?= Form::submit('Upload!') ?>
<?= Form::close() ?>
```

1.  创建一个路由来处理图像上传和验证：

```php
Route::post('imageform', function()
{
    $rules = array(
        'image' => 'required|mimes:jpeg,jpg|max:10000'
    );

    $validation = Validator::make(Input::all(), $rules);

    if ($validation->fails())
    {
        return Redirect::to('imageform')->withErrors($validation);
    }
    else
    {
        $file = Input::file('image');
        $file_name = $file->getClientOriginalName();
        if ($file->move('images', $file_name))
        {
            return Redirect::to('jcrop')->with('image',$file_name);
        }
        else
        {
            return "Error uploading file";
        }
    }
});
```

1.  为我们的 Jcrop 表单创建一个路由：

```php
Route::get('jcrop', function()
{
    return View::make('jcrop')->with('image', 'images/'. Session::get('image'));
});
```

1.  在我们的`app/views`目录中创建一个表单，我们可以在其中裁剪图像，文件名为`jcrop.php`：

```php
<html>
    <head>
        <title>Laravel and Jcrop</title>
        <meta charset="utf-8">
        <link rel="stylesheet" href="css/jquery.Jcrop.min.css" />
        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        <script src="js/jquery.Jcrop.min.js"></script>
    </head>
    <body>
        <h2>Image Cropping with Laravel and Jcrop</h2>
        <img src="<?php echo $image ?>" id="cropimage">

        <?= Form::open() ?>
        <?= Form::hidden('image', $image) ?>
        <?= Form::hidden('x', '', array('id' => 'x')) ?>
        <?= Form::hidden('y', '', array('id' => 'y')) ?>
        <?= Form::hidden('w', '', array('id' => 'w')) ?>
        <?= Form::hidden('h', '', array('id' => 'h')) ?>
        <?= Form::submit('Crop it!') ?>
        <?= Form::close() ?>

        <script type="text/javascript">
            $(function() {
                $('#cropimage').Jcrop({
                    onSelect: updateCoords
                });
            });
            function updateCoords(c) {
                $('#x').val(c.x);
                $('#y').val(c.y);
                $('#w').val(c.w);
                $('#h').val(c.h);
            };
        </script>
    </body>
</html>
```

1.  创建一个处理图像并显示图像的路由：

```php
Route::post('jcrop', function()
{
    $quality = 90;

    $src  = Input::get('image');
    $img  = imagecreatefromjpeg($src);
    $dest = ImageCreateTrueColor(Input::get('w'),
        Input::get('h'));

    imagecopyresampled($dest, $img, 0, 0, Input::get('x'),
        Input::get('y'), Input::get('w'), Input::get('h'),
        Input::get('w'), Input::get('h'));
    imagejpeg($dest, $src, $quality);

    return "<img src='" . $src . "'>";
});

```

## 它是如何工作的...

我们从基本的文件上传开始；为了简化，我们只使用`.jpg`文件。我们使用验证来检查图像类型，以及确保文件大小在 10,000 千字节以下。文件上传后，我们将路径发送到我们的 Jcrop 路由。

在 Jcrop 路由的 HTML 中，我们创建一个带有隐藏字段的表单，该字段将保存裁剪的尺寸。JavaScript 函数`updateCoords`获取裁剪尺寸并更新这些隐藏字段的值。

当我们完成裁剪时，我们提交表单，我们的路由获取 POST 数据。图像通过 GD 库进行裁剪，基于发布的尺寸。然后我们覆盖图像并显示更新和裁剪后的文件。

## 还有更多...

虽然这个示例只涵盖了裁剪 jpg 图像，添加`gif`和`png`图像也不会很困难。我们只需要通过将文件名传递给 Laravel 并使用`File::extension()`来获取文件扩展名。然后，我们可以使用适当的 PHP 函数进行`switch`或`if`语句。例如，如果扩展名是`.png`，我们将使用`imagecreatefrompng()`和`imagepng()`。更多信息可以在[`www.php.net/manual/en/ref.image.php`](http://www.php.net/manual/en/ref.image.php)找到。

# 创建自动完成文本输入

在我们的网络表单上，可能会有时候我们想要有一个自动完成文本字段。这对于填充常见搜索词或产品名称非常方便。使用 jQueryUI 自动完成库以及 Laravel，这变得非常容易。

## 准备工作

在这个食谱中，我们将使用 jQuery 和 jQueryUI 的 CDN 版本；但是，如果我们想要本地拥有它们，我们也可以下载它们并将它们放在我们的`public/js`目录中。

## 如何做...

要完成这个食谱，请按照以下步骤进行：

1.  创建一个路由来保存我们的自动完成表单：

```php
Route::get('autocomplete', function()
{
    return View::make('autocomplete');
});
```

1.  在`app/views`目录中创建一个名为`autocomplete.php`的视图，其中包含我们表单的 HTML 和 JavaScript：

```php
<!DOCTYPE html>
<html>
    <head>
        <title>Laravel Autocomplete</title>
        <meta charset="utf-8">
        <link rel="stylesheet"href="//codeorigin.jquery.com/ui/1.10.2/themes/smoothness/jquery-ui.css" />
        <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        <script src="//codeorigin.jquery.com/ui/1.10.2/jquery-ui.min.js"></script>
    </head>
    <body>
        <h2>Laravel Autocomplete</h2>

        <?= Form::open() ?>
        <?= Form::label('auto', 'Find a color: ') ?>
        <?= Form::text('auto', '', array('id' => 'auto'))?>
        <br>
        <?= Form::label('response', 'Our color key: ') ?>
        <?= Form::text('response', '', array('id' =>'response', 'disabled' => 'disabled')) ?>
        <?= Form::close() ?>

        <script type="text/javascript">
            $(function() {
                $("#auto").autocomplete({
                    source: "getdata",
                    minLength: 1,
                    select: function( event, ui ) {
                        $('#response').val(ui.item.id);
                    }
                });
            });
        </script>
    </body>
</html>
```

1.  创建一个路由，用于填充`autocomplete`字段的数据：

```php
Route::get('getdata', function()
{
    $term = Str::lower(Input::get('term'));
    $data = array(
        'R' => 'Red',
        'O' => 'Orange',
        'Y' => 'Yellow',
        'G' => 'Green',
        'B' => 'Blue',
        'I' => 'Indigo',
        'V' => 'Violet',
    );
    $return_array = array();

    foreach ($data as $k => $v) {
        if (strpos(Str::lower($v), $term) !== FALSE) {
            $return_array[] = array('value' => $v, 'id' =>$k);
        }
    }
    return Response::json($return_array);
});
```

## 它是如何工作的...

在我们的表单中，我们正在创建一个文本字段来接受用户输入，该输入将用于`autocomplete`。还有一个禁用的文本字段，我们可以用来查看所选值的 ID。如果您对特定值有一个数字的 ID，或者以非标准方式命名，这可能会很有用。在我们的示例中，我们使用颜色的第一个字母作为 ID。

当用户开始输入时，`autocomplete`会向我们添加的源发送一个`GET`请求，使用查询字符串中的单词`term`。为了处理这个，我们创建一个路由来获取输入，并将其转换为小写。对于我们的数据，我们使用一个简单的值数组，但在这一点上添加数据库查询也是相当容易的。我们的路由检查数组中的值，看看是否有任何与用户输入匹配的值，如果有，就将 ID 和值添加到我们将返回的数组中。然后，我们将数组输出为 JSON，供`autocomplete`脚本使用。

回到我们的表单页面，当用户选择一个值时，我们将 ID 添加到禁用的响应字段中。很多时候，这将是一个隐藏字段，我们可以在提交表单时传递它。

## 还有更多...

如果我们想要让我们的`getdata`路由只能从我们的自动完成表单或其他 AJAX 请求中访问，我们可以简单地将代码包装在`if (Request::ajax()) {}`中，或者创建一个拒绝任何非 AJAX 请求的过滤器。

# 制作类似 CAPTCHA 的垃圾邮件捕捉器

对抗自动填写网络表单的“机器人”的一种方法是使用 CAPTCHA 技术。这向用户显示一个带有一些随机字母的图像；用户必须在文本字段中填写这些字母。在这个食谱中，我们将创建一个 CAPTCHA 图像，并验证用户是否已正确输入。

## 准备工作

我们需要一个标准的 Laravel 安装，并确保我们的服务器上安装了 GD2 库，这样我们就可以创建一个图像。

## 如何做...

要完成这个食谱，请按照以下步骤进行：

1.  在我们的`app`目录中，创建一个名为`libraries`的目录，并在我们的`composer.json`文件中更新如下：

```php
"autoload": {
    "classmap": [
        "app/commands",
        "app/controllers",
        "app/models",
        "app/database/migrations",
        "app/database/seeds",
        "app/tests/TestCase.php",
        "app/libraries"
    ]
},
```

1.  在我们的`app/libraries`目录中，创建一个名为`Captcha.php`的文件，用于保存我们简单的`Captcha`类：

```php
<?php
class Captcha {
    public function make() 
    {
        $string = Str::random(6, 'alpha');
        Session::put('my_captcha', $string);

        $width      = 100;
        $height     = 25;
        $image      = imagecreatetruecolor($width,$height);
        $text_color = imagecolorallocate($image, 130, 130,130);
        $bg_color   = imagecolorallocate($image, 190, 190,190);

        imagefilledrectangle($image, 0, 0, $width, $height,$bg_color);        
        imagestring($image, 5, 16, 4, $string,$text_color);

        ob_start();
        imagejpeg($image);
        $jpg = ob_get_clean();
        return "data:image/jpeg;base64,". base64_encode($jpg);
    }
}
```

1.  在我们的应用程序根目录中，打开命令行界面以更新`composer`自动加载程序：

```php
**php composer.phar dump-autoload**

```

1.  在`routes.php`中创建一个路由来保存带有`captcha`的表单：

```php
Route::get('captcha', function() 
{
    $captcha = new Captcha;
    $cap = $captcha->make();
    return View::make('captcha')->with('cap', $cap);
});
```

1.  在`app/views`目录中创建我们的`captcha`视图，名称为`captcha.php`：

```php
<h1>Laravel Captcha</h1>
<?php
if (Session::get('captcha_result')) {
    echo '<h2>' . Session::get('captcha_result') . '</h2>';
}
?>
<?php echo Form::open() ?>
<?php echo Form::label('captcha', 'Type these letters:') ?>
<br>
<img src="<?php echo $cap ?>">
<br>
<?php echo Form::text('captcha') ?>
<br>
<?php echo Form::submit('Verify!') ?>
<?php echo Form::close() ?>
```

1.  创建一个路由来比较`captcha`值和用户输入：

```php
Route::post('captcha', function() 
{
    if (Session::get('my_captcha') !==Input::get('captcha')) {
        Session::flash('captcha_result', 'No Match.');
    } else {
        Session::flash('captcha_result', 'They Match!');
    }
    return Redirect::to('captcha');
});
```

## 它是如何工作的...

我们首先更新我们的`composer.json`文件，将我们的`libraries`目录添加到自动加载程序中。现在，我们可以将任何我们想要的类或库添加到该目录中，即使它们是自定义类或可能是一些旧代码。

为了保持简单，我们创建了一个简单的`Captcha`类，其中只有一个`make()`方法。在这个方法中，我们首先使用 Laravel 的`Str:random()`创建一个随机字符串，我们告诉它输出一个只包含字母的 6 个字符的字符串。然后我们将该字符串保存到会话中，以便以后用于验证。

使用字符串，我们创建了一个 100x25 像素的 jpg 图像，背景为灰色，文本为深灰色。我们不是将文件保存到服务器，而是使用输出缓冲区并将图像数据保存到一个变量中。这样，我们可以创建一个数据 URI 并发送回我们的路由。

接下来，我们需要运行 composer 的`dump-autoload`命令，这样我们的新类才能被应用程序使用。

在我们的`captcha`路由中，我们使用`Captcha`类来创建`captcha`数据 URI 并将其发送到我们的表单。对于我们的目的，表单将简单地显示图像并要求在文本字段中输入字符。

当用户提交表单时，我们将比较`Captcha`类创建的 Session 与用户输入。在这个示例中，我们只是检查这两个值是否匹配，但我们也可以创建一个自定义验证方法并将其添加到我们的规则中。然后我们设置一个会话来表示是否匹配，并将用户返回到 CAPTCHA 页面。
