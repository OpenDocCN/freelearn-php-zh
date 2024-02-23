# 第八章：使用 Ajax 和 jQuery

在本章中，我们将涵盖：

+   从另一个页面获取数据

+   设置控制器以返回 JSON 数据

+   创建一个 Ajax 搜索功能

+   使用 Ajax 创建和验证用户

+   根据复选框选择过滤数据

+   创建一个 Ajax 通讯快讯注册框

+   使用 Laravel 和 jQuery 发送电子邮件

+   使用 jQuery 和 Laravel 创建可排序的表

# 介绍

许多现代 Web 应用程序依赖于 JavaScript 来添加动态用户交互。使用 jQuery 库和 Laravel 的内置功能，我们可以在我们自己的应用程序中轻松创建这些交互。

我们将从其他页面异步接收数据，然后发送可以保存在数据库中的数据。

# 从另一个页面获取数据

在我们的应用程序中，可能会有时候我们需要从另一个页面访问一些 HTML。使用 Laravel 和 jQuery，我们可以轻松实现这一点。

## 准备工作

对于这个步骤，我们只需要一个标准的 Laravel 安装。

## 如何做...

要完成这个步骤，请按照给定的步骤进行操作：

1.  打开`routes.php`文件：

```php
Route::get('getting-data', function()
{
  return View::make('getting-data');
});

Route::get('tab1', function()
{
  if (Request::ajax()) {
  return View::make('tab1');
}
  return Response::error('404');
});

Route::get('tab2', function()
{
  if (Request::ajax()) {
  return View::make('tab2');
}
  return Response::error('404');
});
```

1.  在`views`目录中，创建一个名为`tab1.php`的文件：

```php
<h1>CHAPTER 1 - Down the Rabbit-Hole</h1>
<p>
  Alice was beginning to get very tired of sitting by her sister on the bank,and of having nothing to do: once or twice she had peeped into the book her sister was reading, but it had no pictures or conversations in it, 'and what is the use of a book,' thought Alice 'without pictures or conversation?'
</p>
<p>
  So she was considering in her own mind (as well as she could, for the hot day made her feel very sleepy and stupid), whether the pleasure of making a daisy-chain would be worth the trouble of getting up and picking the daisies, when suddenly a White Rabbit with pink eyes ran close by her.
</p>
```

1.  在`views`目录中，创建一个名为`tab2.php`的文件：

```php
<h1>Chapter 1</h1>
<p>"TOM!"</p>
<p>No answer.</p>
<p>"TOM!"</p>
<p>No answer.</p>
<p>"What's gone with that boy,  I wonder? You TOM!"</p>
<p>No answer.</p>
<p>
  The old lady pulled her spectacles down and looked over them about the room; 
  then she put them up and looked out under them. She seldom or never looked 
  through them for so small a thing as a boy; they were her state pair, 
  the pride of her heart, and were built for "style," not service—she could 
  have seen through a pair of stove-lids just as well. She looked perplexed 
  for a moment, and then said, not fiercely, but still loud enough for the 
  furniture to hear:
</p>
<p>"Well, I lay if I get hold of you I'll—"</p>
<p>
  She did not finish, for by this time she was bending down and punching 
  under the bed with the broom, and so she needed breath to punctuate 
  the punches with. She resurrected nothing but the cat.
</p>
```

1.  在`views`目录中，创建一个名为`getting-data.php`的文件：

```php
<!DOCTYPE html>
<html>
<head>
  <meta charset=utf-8 />
  <title>Getting Data</title>
  <script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
</head>
<body>
<ul>
  <li><a href="#" id="tab1" class="tabs">Alice In Wonderland</a></li>
  <li><a href="#" id="tab2" class="tabs">Tom Sawyer</a></li>
</ul>
<h1 id="title"></h1>
<div id="container"></div>
<script>
  $(function() {
  $(".tabs").on("click", function(e) {e.preventDefault();
  var tab = $(this).attr("id");
  var title = $(this).html();
  $("#container").html("loading…");
  $.get(tab, function(data) {
  $("#title").html(title);
  $("#container").html(data);
});
});
});
</script>
</body>
</html>
```

1.  在`http://{yourserver}/getting-data`页面查看页面，并单击链接以加载内容。

## 它是如何工作的...

我们首先设置我们的路由。我们的第一个路由将显示链接，当我们点击它们时，内容将加载到页面中。我们的下两个路由将保存要在主页面上显示的实际内容。为了确保这些页面不能直接访问，我们使用`Request::ajax()`方法来确保只接受 Ajax 请求。如果有人试图直接访问页面，它将把他们发送到错误页面。

我们的两个视图文件将包含一些书籍摘录。由于这将加载到另一个页面中，我们不需要完整的 HTML。然而，我们的主页面是一个完整的 HTML 页面。我们首先通过使用来自 Google 的**内容传送网络**（**CDN**）加载 jQuery。然后，我们有一个我们想要使用的书籍列表。为了使事情变得更容易一些，链接的 ID 将对应于我们创建的路由。

当有人点击链接时，脚本将使用 ID 并从具有相同名称的路由获取内容。结果将加载到我们的`container` div 中。

# 设置控制器以返回 JSON 数据

当我们使用 JavaScript 访问数据时，最简单的方法之一是使用 JSON 格式的数据。在 Laravel 中，我们可以从我们的控制器中返回 JSON，以供我们在另一个页面上的 JavaScript 使用。

## 准备工作

对于这个步骤，我们需要一个标准的 Laravel 安装。

## 如何做...

对于这个步骤，请按照给定的步骤进行操作：

1.  在`controllers`目录中，创建一个名为`BooksController.php`的文件：

```php
<?php

  class BooksController extends BaseController {

  public function getIndex()
{
  return View::make('books.index');
}

  public function getBooks()
{
  $books = array('Alice in Wonderland','Tom Sawyer','Gulliver\'s Travels','Dracula','Leaves of Grass');
  return Response::json($books);
}
}
```

1.  在`routes.php`中，注册书籍控制器

```php
Route::controller('books', 'BooksController');
```

1.  在`views`目录中，创建一个名为`books`的文件夹，在该文件夹中创建一个名为`index.php`的文件：

```php
<!DOCTYPE html>
<html>
<head>
  <meta charset=utf-8 />
  <title>Show Books</title>
  <script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
</head>
<body>
<a href="#" id="book-button">Load Books</a>
<div id="book-list"></div>
<script>
$(function() {
$('#book-button').on('click', function(e) {e.preventDefault();
$('#book-list').html('loading...');
$.get('books/books', function(data) {var book_list = '';
$.each(data, function(){book_list += this + '<br>';
})
$("#book-list").html(book_list);
$('#book-button').hide();
});
});
});
</script>
</body>
</html>
```

## 它是如何工作的...

我们首先为我们的书籍列表创建一个 RESTful 控制器，它扩展了我们的`BaseController`类。我们的控制器有两个方法：一个用于显示列表，一个用于返回格式化的列表。我们的`getBooks()`方法使用数组作为我们的数据源，并且我们使用 Laravel 的`Response::json()`方法来自动为我们执行正确的格式化。

在我们的主页面上，我们在 JavaScript 中对页面进行`get`请求，接收 JSON，并循环遍历结果。当我们循环时，我们将书籍添加到 JavaScript 变量中，然后将列表添加到我们的`book-list` div 中。

## 还有更多...

我们的列表可以来自任何数据源。我们可以添加数据库功能，甚至调用 API。当我们使用 Laravel 的 JSON 响应时，该值将以正确的格式和正确的标头进行格式化。

# 创建一个 Ajax 搜索功能

如果我们想在应用程序中搜索信息，异步执行搜索可能会很有用。这样，用户就不必转到新页面并刷新所有资产。使用 Laravel 和 JavaScript，我们可以以非常简单的方式执行这个搜索。

## 准备工作

对于这个教程，我们需要一个正常安装的 Laravel。

## 如何操作...

要完成这个教程，请按照以下步骤进行：

1.  在`controllers`目录中，创建一个名为`SearchController.php`的文件：

```php
<?php

class SearchController extends BaseController {

  public function getIndex()
{
  return View::make('search.index');
}

  public function postSearch()
{
  $return = array();
  $term = Input::get('term');

  $books = array(array('name' => 'Alice in Wonderland', 'author' => 'Lewis Carroll'),array('name' => 'Tom Sawyer', 'author' => 'Mark Twain'),array('name' => 'Gulliver\'s Travels', 'author' =>'Jonathan Swift'),array('name' => 'The Art of War', 'author' => 'Sunzi'),array('name' => 'Dracula', 'author' => 'Bram Stoker'),array('name' => 'War and Peace', 'author' =>'LeoTolstoy'),);

foreach ($books as $book) {
if (stripos($book['name'], $term) !== FALSE) $return[] =$book;
}

return Response::json($return);
}
}
```

1.  在`routes.php`文件中，注册控制器：

```php
  Route::controller('search', 'SearchController');
```

1.  在`views`目录中，创建一个名为`search`的文件夹，在该文件夹中，创建一个名为`index.php`的文件：

```php
<!DOCTYPE html>
<html>
<head>
<meta charset=utf-8 />
<title>AJAX Search</title>
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
</head>
<body>
<h1>Search</h1>
<form id="search-form">
<input name="search" id="term"> <input type="submit">
</form>
<div id="results"></div>
<script>
  $(function() {
  $("#search-form").on("submit", function(e) {e.preventDefault();
  var search_term = $("#term").val();
  var display_results = $("#results");
  display_results.html("loading...");
  var results = '';
  $.post("search/search", {term: search_term}, function(data) {if (data.length == 0) {results = 'No Results';
  } else {
  $.each(data, function() {
  results += this.name + ' by ' + this.author + '<br>';
});
}
display_results.html(results);
});
})
});
</script>
</body>
</html>
```

## 它是如何工作的...

我们首先创建一个包含两种方法的 RESTful 控制器：一个用于我们的主页面，一个用于处理搜索。在我们的主页面上，我们有一个单一的`text`字段和一个`submit`按钮。当表单提交时，我们的 JavaScript 将把表单发布到我们的搜索页面。如果有结果，它将循环遍历它们，并在我们的`results` div 中显示它们。

对于我们的`postSearch()`方法，我们使用一个数组作为我们的数据源。当进行搜索时，我们然后循环遍历数组，看看字符串是否与我们的标题匹配。如果是，该值将被添加到一个数组中，并将该数组作为 JSON 返回。

# 使用 Ajax 创建和验证用户

当用户来到我们的应用程序时，我们可能希望他们在不需要导航到另一个页面的情况下注册或登录。使用 Laravel 内的 Ajax，我们可以提交用户的表单并异步运行验证。

## 准备工作

对于这个教程，我们需要一个正常安装的 Laravel，以及一个正确配置的 MySQL 数据库。我们还需要向数据库添加一个用户表，可以使用以下代码完成：

```php
CREATE TABLE users (id int(10) unsigned NOT NULL AUTO_INCREMENT,email varchar(255) DEFAULT NULL,password char(60) DEFAULT NULL,PRIMARY KEY (id)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 如何操作...

要完成这个教程，请按照给定的步骤进行：

1.  在`controllers`目录中，创建一个名为`UsersController.php`的文件：

```php
<?php
class UsersController extends BaseController {
  public function getIndex()
  {
  return View::make('users.index');
  }

  public function postRegister()
  {
  $rules = array('email' => 'required|email','password' => 'required|min:6');

  $validation = Validator::make(Input::all(), $rules);

  if ($validation->fails())
  {
  return Response::json($validation->errors()->toArray());
}
else
{
DB::table('users')->insert(array('email' => Input::get('email'),'password' => Hash::make(Input::get('password'))));
return Response::json(array('Registration is complete!'));
}
}
}
```

1.  在`routes.php`中注册控制器：

```php
 **Route::controller('users', 'UsersController');**

```

1.  在`views`目录中，创建一个名为`users`的文件夹，在该文件夹中，创建一个名为`index.php`的文件：

```php
<!doctype html>
<html lang="en">
  <head>
  <meta charset="utf-8">
  <title>User Register</title>
  <script type="text/javascript"src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
  </head>
  <body>
  <form id="register">
  <label for="email">Your email:</label> 
  <input type="email" name="email" id="email"><br>
  <label for="password">Your password:</label> 
  <input type="password" name="password" id="password"><br>
  <input type="submit">
  </form>
  <div id="results"></div>
  <script>
  $(function(){
  $("#register").on("submit", function(e) {e.preventDefault();
  var results = '';
  $.post('users/register', {email: $("#email").val(), password:$("#password").val()}, function(data) {
  $.each(data, function(){results += this + '<br>';
});
  $("#results").html(results);
});
});
});
</script>
  </body>
</html>
```

## 它是如何工作的...

要开始这个教程，我们创建一个主页面，用于容纳用户注册表单。当表单提交时，它将发布到我们的`postRegister()`方法，并将任何结果返回到`results` div。

`postRegister()`方法首先设置我们验证的规则。在这种情况下，我们希望确保两个字段都有值，电子邮件必须有效，并且密码必须至少为 6 个字符。如果验证失败，我们将错误作为 JSON 编码的字符串发送回来，我们的主页面将显示错误。如果一切正常，我们将一切保存到数据库并返回成功消息。

## 还有更多...

如果我们不希望任何其他页面向我们的方法发布数据，我们可以添加一个`Request::ajax()`条件。这意味着只有 Ajax 调用才会被我们的方法处理。

# 根据复选框选择过滤数据

在向用户显示数据时，允许他们过滤数据可能会很方便。因此，我们不必让用户每次都点击提交并重新加载页面，我们可以使用 Ajax 来进行所有的过滤。对于这个教程，我们将制作一个书籍列表，并允许用户根据流派进行过滤。

## 准备工作

对于这个教程，我们需要一个标准的 Laravel 安装，配置为与数据库一起工作。我们需要通过运行以下 SQL 语句来设置一个要使用的表：

```php
DROP TABLE IF EXISTS books;
CREATE TABLE books (id int(10) unsigned NOT NULL AUTO_INCREMENT,name varchar(255) DEFAULT NULL,author varchar(255) DEFAULT NULL,genre varchar(255) DEFAULT NULL,PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

  INSERT INTO books VALUES ('1', 'Alice in Wonderland', 'Lewis Carroll', 'fantasy');
  INSERT INTO books VALUES ('2', 'Tom Sawyer', 'Mark Twain', 'comedy');
  INSERT INTO books VALUES ('3', 'Gulliver\'s Travels', 'Jonathan Swift', 'fantasy');
  INSERT INTO books VALUES ('4', 'The Art of War', 'Sunzi', 'philosophy');
  INSERT INTO books VALUES ('5', 'Dracula', 'Bram Stoker', 'horror');
  INSERT INTO books VALUES ('6', 'War and Peace', 'Leo Tolstoy', 'drama');
  INSERT INTO books VALUES ('7', 'Frankenstein', 'Mary Shelley', 'horror');
  INSERT INTO books VALUES ('8', 'The Importance of Being Earnest', 'Oscar Wilde', 'comedy');
  INSERT INTO books VALUES ('9', 'Peter Pan', 'J. M. Barrie', 'fantasy');
```

## 如何操作...

要完成这个教程，请按照以下步骤进行：

1.  在`controllers`目录中，创建一个名为`BooksController.php`的新文件：

```php
<?php
class BooksController extends BaseController {
  public function getIndex()
{
  return View::make('books.index');
}

  public function postBooks()
{
  if (!$genre = Input::get('genre')) {
  $books = Book::all();
  } else {
  $books = Book::whereIn('genre', $genre)->get();
}
return $books;
}
}
```

1.  在`routes.php`文件中注册`books`控制器：

```php
 **Route::controller('books', 'BooksController');**

```

1.  在`views`目录中，创建一个名为`books`的新文件夹，在该文件夹中，创建一个名为`index.php`的文件：

```php
<!doctype html>
<html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Books filter</title>
  <scriptsrc="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
  </head>
  <body>
  <form id="filter">
  Comedy: <input type="checkbox" name="genre[]" value="comedy"><br>
  Drama: <input type="checkbox" name="genre[]" value="drama"><br>
  Fantasy: <input type="checkbox" name="genre[]" value="fantasy"><br>
  Horror: <input type="checkbox" name="genre[]" value="horror"><br>
  Philosophy: <input type="checkbox" name="genre[]" value="philosophy"><br>
  </form>
  <hr>
  <h3>Results</h3>
  <div id="books"></div>
  <script>
  $(function(){
  $("input[type=checkbox]").on('click', function() {var books = '';
  $("#books").html('loading...');
  $.post('books/books', $("#filter").serialize(), function(data){$.each(data, function(){books += this.name + ' by ' + this.author + ' (' + this.genre + ')<br>';
});
$("#books").html(books);
});
});
});
</script>
</body>
</html>
```

1.  在`models`目录中，创建一个名为`Book.php`的文件：

```php
<?php
class Book extends Eloquent {
}
```

1.  在浏览器中，转到`http://{my-server}/books`，并点击一些复选框以查看结果。

## 它是如何工作的...

在我们的数据库设置好之后，我们从我们的主列表页面开始。这个页面有许多复选框，每个值对应于我们书籍表中的一个流派。当一个框被选中时，表单会异步提交到我们的`postBooks()`方法。我们使用这些结果，循环遍历它们，并在我们的`books`div 中显示它们。

我们的`postBooks()`方法首先确保实际提交了一个流派。如果没有，这意味着一切都未被选中，它将返回所有的书籍。如果有选中的内容，我们从数据库中获取与选中值匹配的所有内容。由于 Laravel 以 JSON 格式提供了原始返回的数据，我们然后返回结果，在我们的索引中，结果被正确显示。

# 创建一个 Ajax 新闻订阅框

让用户加入我们的电子邮件列表的一种方法是让他们通过我们的网站进行注册。在这个教程中，我们将使用 MailChimp 的 API 和一个模态窗口来显示一个注册表单，并通过 Ajax 调用发送它。

## 准备工作

对于这个教程，我们需要一个标准的 Laravel 安装。我们还将使用 MailChimp API 进行新闻订阅；可以在[www.mailchimp.com](http://www.mailchimp.com)创建免费帐户和 API 密钥。

## 如何做...

要完成这个教程，请按照给定的步骤进行操作：

1.  打开`composer.json`文件，并更新`require`部分以类似以下代码：

```php
  "require": {
  "laravel/framework": "4.0.*",
  "rezzza/mailchimp": "dev-master"
}
```

1.  在命令行窗口中，位于 artisan 文件的位置，使用以下命令更新 Composer：

```php
 **php composer.phar update**

```

1.  在`app/config`目录中，创建一个名为`mailchimp.php`的文件：

```php
<?php

return array('key' => '12345abcde-us1','list' => '123456789'
);
```

1.  在`views`目录中，创建一个名为`signup.php`的文件：

```php
<!doctype html>
<html lang="en">
  <head>
  <meta charset="utf-8">
  <title>Newsletter Signup</title>
  <link href="//netdna.bootstrapcdn.com/twitter-bootstrap/2.2.2/css/bootstrap-combined.min.css" rel="stylesheet">
  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.0/jquery.min.js"></script>
  <script src="//netdna.bootstrapcdn.com/twitter-bootstrap/2.2.2/js/bootstrap.min.js"></script>
  </head>
  <body>
  <p>
  <a href="#signupModal" role="button" class="btn btn-info" data-toggle="modal">Newsletter Signup</a>
  </p>
  <div id="results"></div>
  <div id="signupModal" class="modal hide fade">
  <div class="modal-header">
  <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
  <h3>Sign-up for our awesome newsletter!</h3>
  </div>
  <div class="modal-body">
  <p>
  <form id="newsletter_form">
  <label>Your First Name</label>
  <input name="fname"><br>
  <label>Last Name</label>
  <input name="lname"><br>
  <label>Email</label>
  <input name="email">
  </form>
  </p>
  </div>
  <div class="modal-footer">
  <a href="#" class="btn close" data-dismiss="modal">Close</a>
  <a href="#" class="btn btn-primary" id="newsletter_submit">Signup</a>
  </div>
  </div>
  <script>
  $(function(){
  $("#newsletter_submit").on('click', function(e){e.preventDefault();
  $("#results").html("loading...");
  $.post('signup-submit', $("#newsletter_form").serialize(), function(data){
  $('#signupModal').modal('hide');
  $("#results").html(data);
});
});
});
  </script>
  </body>
</html>
```

1.  在`routes.php`文件中，添加我们需要的路由，使用以下代码：

```php
Route::get('signup', function()
{
  return View::make('signup');
});

Route::post('signup-submit', function()
{
  $mc = new MCAPI(Config::get('mailchimp.key'));

  $response = $mc->listSubscribe('{list_id}',Input::get('email'),array('FNAME' => Input::get('fname'),'LNAME' => Input::get('lname')
)
);

if ($mc->errorCode){
return 'There was an error: ' . $mc->errorMessage;
} else {
return 'You have been subscribed!';
}
});
```

## 它是如何工作的...

我们首先通过使用 MailChimp SDK 的 composer 版本将 MailChimp 包安装到我们的应用程序中。然后我们需要创建一个配置文件来保存我们的 API 密钥和我们想要使用的列表 ID。

我们的注册页面将利用 jQuery 和 Bootstrap 进行处理和显示。由于我们只想在用户想要注册时显示表单，我们有一个单独的按钮，当点击时，将显示一个带有我们表单的模态窗口。表单将包括名字、姓氏和电子邮件地址。

当注册表单被提交时，我们序列化数据并将其发送到我们的`signup-submit`路由。一旦我们得到一个响应，我们隐藏模态窗口并在我们的页面上显示结果。

在我们的`signup-submit`路由中，我们尝试使用输入的信息订阅用户。如果我们得到一个响应，我们检查响应是否包含错误。如果有错误，我们将其显示给用户，如果没有，我们显示成功消息。

## 还有更多...

我们的`signup-submit`路由没有对表单输入进行任何验证。要包括验证，请查看*使用 Ajax 创建和验证用户*教程中的示例。

## 另请参阅

+   *使用 Ajax 创建和验证用户*教程

# 使用 Laravel 和 jQuery 发送电子邮件

当创建联系表单时，我们可以选择让用户异步发送表单。使用 Laravel 和 jQuery，我们可以在不需要用户转到不同页面的情况下提交表单。

## 准备工作

对于这个教程，我们需要一个标准的 Laravel 安装和正确配置我们的邮件客户端。我们可以在`app/config/mail.php`文件中更新我们的邮件配置。

## 如何做...

要完成这个教程，请按照给定的步骤进行操作：

1.  在`views`目录中，创建一个名为`emailform.php`的文件，如下所示：

```php
  <!doctype html>
  <html lang="en">
  <head>
  <meta charset="utf-8">
  <title></title>
  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
  </head>
  <body>
  <div id="container">
  <div id="error"></div>
  <form id="email-form">
  <label>To: </label>
  <input name="to" type="email"><br>
  <label>From: </label>
  <input name="from" type="email"><br>
  <label>Subject: </label>
  <input name="subject"><br>
  <label>Message:</label><br>
  <textarea name="message"></textarea><br>
  <input type="submit" value="Send">
  </form>
  </div>
  <script>
  $(function(){
  $("#email-form").on('submit', function(e){e.preventDefault();
  $.post('email-send', $(this).serialize(), function(data){
  if (data == 0) {
  $("#error").html('<h3>There was an error</h3>');
  } else {
  if (isNaN(data)) {
  $("#error").html('<h3>' + data + '</h3>');
  } else {
  $("#container").html('Your email has been sent!');
}
}
});
});
});
</script>
</body>
</html>
```

1.  在`views`文件夹中，创建我们的电子邮件模板视图文件，命名为`ajaxemail.php`，并使用以下代码：

```php
<!DOCTYPE html>
<html lang="en-US">
<head>
<meta charset="utf-8">
</head>
<body>
<h2>Your Message:</h2>
<div><?= $message ?></div>
</body>
</html>
```

1.  在`routes.php`文件中，根据以下代码片段创建路由：

```php
  Route::get('email-form', function()
{
  return View::make('emailform');
});
  Route::post('email-send', function()
{
  $input = Input::all();

  $rules = array('to'      => 'required|email','from'    => 'required|email','subject' => 'required','message' => 'required'
);

  $validation = Validator::make($input, $rules);

  if ($validation->fails())
{
  $return = '';
  foreach ($validation->errors()->all() as $err) {
  $return .= $err . '<br>';
}
  return $return;
}

  $send = Mail::send('ajaxemail', array('message' =>Input::get('message')), function($message)
{
  $message->to(Input::get('to'))->replyTo(Input::get('from'))->subject(Input::get('subject'));
});

  return $send;
});
```

## 它是如何工作的...

对于这个教程，我们需要正确配置我们的电子邮件客户端。我们有许多选择，包括 PHP 的`mail()`方法，sendmail 和 SMTP。我们甚至可以使用第三方电子邮件服务，如 mailgun 或 postmark。

我们的电子邮件表单是一个常规的 HTML 表单，包括四个字段：`to`和`from`电子邮件地址，`subject`行和实际的电子邮件消息。当表单被提交时，字段被序列化并发布到我们的`email-send`路由。

`email-send`路由首先验证所有发布的输入。如果有任何验证错误，它们将作为字符串返回。如果一切正常，我们将发送我们的值到`Mail::send`方法，然后发送它。

回到我们的`e-mail-form`路由 JavaScript 中，我们检查`email-send`是否返回了`FALSE`，如果是，则显示错误。如果不是，我们需要检查响应是否是一个数字。如果不是一个数字，那意味着有验证错误，我们将它们显示出来。如果是一个数字，那意味着电子邮件发送成功，所以我们显示一个成功消息。

# 使用 jQuery 和 Laravel 创建可排序的表格

在处理大量数据时，将其显示在表格视图中可能会有所帮助。为了操纵数据，例如排序或搜索，我们可以使用数据表 JavaScript 库。这样，我们就不需要每次想要更改视图时都进行数据库调用。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装和一个正确配置的 MySQL 数据库。

## 如何做...

按照给定的步骤完成这个示例：

1.  在我们的数据库中，使用以下命令创建一个新表并添加一些示例数据：

```php
DROP TABLE IF EXISTS bookprices;
CREATE TABLE bookprices (id int(10) unsigned NOT NULL AUTO_INCREMENT,price float(10,2) DEFAULT NULL,book varchar(100) DEFAULT NULL,PRIMARY KEY (id)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
  INSERT INTO bookprices VALUES ('1', '14.99', 'Alice in Wonderland');
  INSERT INTO bookprices VALUES ('2', '24.50', 'Frankenstein');
  INSERT INTO bookprices VALUES ('3', '29.80', 'War andPeace');
  INSERT INTO bookprices VALUES ('4', '11.08', 'Moby Dick');
  INSERT INTO bookprices VALUES ('5', '19.72', 'The Wizard of Oz');
  INSERT INTO bookprices VALUES ('6', '45.00', 'The Odyssey');
```

1.  在`app/models`目录中，创建一个名为`Bookprices.php`的文件，并包含以下代码片段：

```php
<?php
class Bookprices extends Eloquent {
}
```

1.  在`routes.php`文件中，按照以下代码添加我们的路由：

```php
Route::get('table', function()
{
  $bookprices = Bookprices::all();
  return View::make('table')->with('bookprices', $bookprices);
});
```

1.  在`views`目录中，创建一个名为`table.php`的文件，其中包含以下代码：

```php
<!doctype html>
<html lang="en">
  <head>
  <meta charset="utf-8">
  <title></title>
  <script src="//ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
  <script src="//ajax.aspnetcdn.com/ajax/jquery.dataTables/1.9.4/jquery.dataTables.min.js"></script>
  <link rel="stylesheet" type="text/css" href="//ajax.aspnetcdn.com/ajax/jquery.dataTables/1.9.4/css/jquery.dataTables.css">
  </head>
  <body>
  <h1>Book List</h1>
  <table>
  <thead>
  <tr>
  <th>Price</th>
  <th>Name</th>
  </tr>
  </thead>
  <tbody>
  <?php foreach ($bookprices as $book): ?>
  <tr>
  <td><?php echo $book['price'] ?></td>
  <td><?php echo $book['book'] ?></td>
  </tr>
  <?php endforeach; ?>
  </tbody>
  </table>
  <script>
  $(function(){
  $("table").dataTable();
});
  </script>
  </body>
  </html>
```

## 它是如何工作的...

要开始这个示例，我们创建一个表来保存我们的图书价格数据。然后，我们将数据插入表中。接下来，我们创建一个`Eloquent`模型，以便我们可以与数据交互。然后将该数据传递到我们的视图中。

在我们的视图中，我们加载 jQuery 和`dataTables`插件。然后，我们创建一个表来保存我们的数据，然后循环遍历数据，将每条记录放入新行中。当我们将`dataTable`插件添加到我们的表中时，它将自动为每个列添加排序。

## 还有更多...

`Datatables`是一个强大的 jQuery 插件，用于操纵表格数据。有关更多信息，请查看[`www.datatables.net`](http://www.datatables.net)上的文档。
