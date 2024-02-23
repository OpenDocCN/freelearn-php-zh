# 第五章。构建新闻聚合网站

在本章中，我们将创建一个新闻聚合网站。我们将解析多个源，对它们进行分类，为我们的网站激活/停用它们，并使用 PHP 的 SimpleXML 扩展在我们的网站上显示它们。本章将涵盖以下主题：

+   创建数据库并迁移 feeds 表

+   创建 feeds 模型

+   创建我们的表单

+   验证和处理表单

+   扩展核心类

+   读取和解析外部源

# 创建数据库并迁移 feeds 表

成功安装 Laravel 4 并从`app/config/database.php`定义数据库凭据后，创建一个名为`feeds`的数据库。

创建数据库后，打开终端，进入项目文件夹，并运行此命令：

```php
**php artisan migrate:make create_feeds_table --table=feeds --create**

```

这个命令将为我们生成一个名为`feeds`的新数据库迁移。现在导航到`app/database/migrations`，打开刚刚由前面的命令创建的迁移文件，并将其内容更改如下：

```php
<?php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFeedsTable extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
    Schema::create('feeds', function(Blueprint $table)
    {
      $table->increments('id');
      $table->enum('active', array('0', '1'));
      $table->string('title,100)->default('');
      $table->enum('category', array('News', 'Sports','Technology'));
      $table->string('feed',1000)->default('');
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
    Schema::drop('feeds');
  }

}
```

我们有一个`title`列用于在网站上显示标题，这更加用户友好。此外，我们设置了一个名为`active`的键，因为我们想要启用/禁用源；我们使用 Laravel 4 提供的新`enum()`方法进行设置。我们还设置了一个`category`列，也使用`enum()`方法进行分组源的设置。

保存文件后，运行以下命令执行迁移：

```php
**php artisan migrate**

```

如果没有错误发生，您已经准备好进行项目的下一步了。

# 创建 feeds 模型

如您所知，对于 Laravel 上的任何与数据库操作相关的事情，使用模型是最佳实践。我们将受益于 Eloquent ORM。

将此文件保存为`feeds.php`，放在`app/models/`下：

```php
<?php
Class Feeds Extends Eloquent{
    protected $table = 'feeds';
    protected $fillable = array('feed', 'title', 'active','category');
}
```

我们设置表名和可填充列的值。现在我们的模型已经准备好，我们可以继续下一步，开始创建我们的控制器和表单。

# 创建我们的表单

现在我们应该创建一个表单来保存记录到数据库并指定其属性。

1.  首先，打开终端并输入以下命令：

```php
**php artisan controller:make FeedsController**

```

这个命令将为您在`app/controllers`文件夹中生成一个`FeedsController.php`文件，并带有一些空白方法。

### 注意

由`artisan`命令自动填充的控制器中的默认方法不是 RESTful 的。

1.  现在，打开`app/routes.php`并添加以下行：

```php
**//We defined a RESTful controller and all its via route directly**
**Route::controller('feeds', 'FeedsController');**

```

我们可以使用一行代码定义控制器上声明的所有操作，而不是逐个定义所有操作。如果您的方法名称可以直接用作 get 或 post 操作，使用`controller()`方法可以节省大量时间。第一个参数设置控制器的 URI，第二个参数定义`controllers`文件夹中将被访问和定义的类。

### 注意

以这种方式设置的控制器自动是 RESTful 的。

1.  现在，让我们创建表单的方法。将以下代码添加到您的控制器文件中：

```php
  //The method to show the form to add a new feed
  public function getCreate() {
    //We load a view directly and return it to be served
    return View::make('create_feed');
      }
```

这里的过程非常简单；我们将方法命名为`getCreate()`，因为我们希望我们的`create`方法是 RESTful 的。我们只是加载了一个视图文件，我们将在下一步直接生成它。

1.  现在让我们创建我们的视图文件。将此文件保存为`create_feed.blade.php`，放在`app/views/`下：

```php
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Save a new ATOM Feed to Database</title>
</head>
<body>
  <h1>Save a new ATOM Feed to Database</h1>
  @if(Session::has('message'))
    <h2>{{Session::get('message')}}</h2>
  @endif
    {{Form::open(array('url' => 'feeds/create', 'method' => 'post'))}}
    <h3>Feed Category</h3>
  {{Form::select('category',array('News'=>'News','Sports'=>'Sports','Technology'=>'Technology'),Input::old('category'))}}
  <h3>Title</h3>
    {{Form::text('title',Input::old('title'))}}
    <h3>Feed URL</h3>
    {{Form::text('feed',Input::old('feed'))}}

    <h3>Show on Site?</h3>
{{Form::select('active',array('1'=>'Yes','2'=>'No'),Input::old('active'))}}
    {{Form::submit('Save!',array('style'=>'margin:20px 100% 0 0'))}}
    {{Form::close()}}
</body>
</html>
```

上述代码将生成一个简单的表单，如下所示：

![创建我们的表单](img/2111OS_05_01.jpg)

# 验证和处理表单

在本节中，我们将验证提交的表单，并确保字段有效且必填字段已填写。然后我们将数据保存到数据库中。

1.  首先，我们需要定义表单验证规则。我们更喜欢将验证规则添加到相关模型中，这样规则就可以重复使用，这可以防止代码变得臃肿。为此，在本章前面生成的`feeds.php`中的`app/models/`（我们生成的模型）中，类定义的最后一个`}`之前添加以下代码：

```php
//Validation rules
public static $form_rules = array(
  'feed'    => 'required|url|active_url',
  'title'  => 'required'
  'active'  => 'required|between:0,1',
  'category'  => 'required| in:News,Sports,Technology'
);
```

我们将变量设置为`public`，这样它可以在模型文件之外使用，并将其设置为`static`，这样我们可以直接访问这个变量。

我们希望 feed 是一个 URL，并且我们希望使用`active_url`验证规则来检查它是否是一个活动的 URL，这取决于 PHP 的`chkdnsrr()`方法。

我们的 active 字段只能获得两个值，`1`或`0`。由于我们将其设置为整数，我们可以使用 Laravel 的表单验证规则`between`来检查数字是否在`1`和`0`之间。

我们的 category 字段也具有`enum`类型，其值应该只是`News`、`Sports`或`Technology`。要使用 Laravel 检查确切的值，你可以使用验证规则`in`。

### 注意

并非所有的服务器配置都支持`chkdnsrr()`方法，所以确保它在你这边已安装，否则你可能只依赖于验证 URL 是否格式正确。

1.  现在我们需要一个控制器的 post 方法来处理表单。在最后一个`}`之前，将以下方法添加到`app/controllers/FeedsController.php`中：

```php
//Processing the form
public function postCreate(){

//Let's first run the validation with all provided input
  $validation = Validator::make(Input::all(),Feeds::$form_rules);
  //If the validation passes, we add the values to the database and return to the form 
  if($validation->passes()) {
    //We try to insert a new row with Eloquent
    $create = Feeds::create(array(
      'feed'    => Input::get('feed'),
      'title'  => Input::get('title'),
      'active'  => Input::get('active'),
      'category'  => Input::get('category')
    ));

    //We return to the form with success or error message due to state of the 
    if($create) {
      return Redirect::to('feeds/create')
        ->with('message','The feed added to the database successfully!');
    } else {
      return Redirect::to('feeds/create')
        ->withInput()
        ->with('message','The feed could not be added, please try again later!');
    }
  } else {
    //If the validation does not pass, we return to the form with first error message as flash data
    return Redirect::to('feeds/create')
        ->withInput()
        ->with('message',$validation->errors()->first());

  }
}
```

让我们逐一深入代码。首先，我们进行了表单验证，并从我们通过`Feeds::$form_rules`生成的模型中调用了我们的验证规则。

之后，我们创建了一个`if()`语句，并用它将代码分成两部分。如果表单验证失败，我们将使用`withInput()`特殊方法返回到表单，并使用`with()`方法添加一个 flash 数据消息字段。

如果表单验证通过，我们尝试使用 Eloquent 的`create()`方法向数据库添加新列，并根据`create`方法返回的结果返回到表单，显示成功或错误消息。

现在，我们需要为索引页面创建一个新的视图，它将显示所有 feed 的最后五个条目。但在此之前，我们需要一个函数来解析 Atom feeds。为此，我们将扩展 Laravel 的内置`Str`类。

# 扩展核心类

Laravel 有许多内置的方法，使我们的生活更轻松。但是，就像所有捆绑包一样，捆绑包本身可能不会满足任何用户，因为它是被引入的。因此，你可能希望使用自己的方法以及捆绑的方法。你总是可以创建新的类，但是如果你想要实现的一半已经内置了呢？例如，你想添加一个表单元素，但已经有一个`Form`类捆绑了。在这种情况下，你可能希望扩展当前的类，而不是创建新的类来保持代码整洁。

在这一部分，我们将使用名为`parse_atom()`的方法来扩展`Str`类，我们将编写这个方法。

1.  首先，你必须找到类文件所在的位置。我们将扩展`Str`类，它位于`vendor/laravel/framework/src/Illuminate/Support`下。请注意，你也可以在`app/config/app.php`的 aliases 键中找到这个类。

1.  现在在`app/folder`下创建一个名为`lib`的新文件夹。这个文件夹将保存我们的类扩展。因为`Str`类被分组到`Support`文件夹下，建议你也在`lib`下创建一个名为`Support`的新文件夹。

1.  现在在`app/lib/Support`下创建一个名为`Str.php`的新文件，你刚刚创建的：

```php
<?php namespace app\lib\Support;
class Str extends \Illuminate\Support\Str {
    //Our shiny extended codes will come here
  }
```

我们给它命名空间，这样我们就可以轻松地访问它。你可以直接使用`Str::trim()`，而不是像`\app\lib\Support\Str::trim()`那样使用它（你可以）。其余的代码解释了如何扩展库。我们提供了从`Illuminate`路径开始的类名，以直接访问`Str`类。

1.  现在打开位于`app/config/`下的`app.php`文件；注释掉以下行：

```php
'Str'             => 'Illuminate\Support\Str',
```

1.  现在，添加以下行：

```php
'Str'             => 'app\lib\Support\Str',
```

这样，我们用我们的类替换了自动加载的`Str`类，而我们的类已经扩展了原始类。

1.  现在为了在 autoruns 上进行标识，打开你的`composer.json`文件，并将这些行添加到 autoload 的`classmap`对象中：

```php
"app/lib",
"app/lib/Support"
```

1.  最后，在终端中运行以下命令：

```php
**php composer.phar dump-autoload**

```

这将寻找依赖项并重新编译常见类。如果一切顺利，你现在将拥有一个扩展的`Str`类。

### 注意

文件夹和类名在 Windows 服务器上也是区分大小写的。

# 读取和解析外部反馈

我们在服务器上已经对反馈的 URL 和标题进行了分类。现在我们要做的就是解析它们并展示给最终用户。这需要遵循一些步骤：

1.  首先，我们需要一个方法来解析外部 Atom 反馈。打开位于`app/lib/Support/`下的`Str.php`文件，并将此方法添加到类中：

```php
public static function parse_feed($url) {
    //First, we get our well-formatted external feed
    $feed = simplexml_load_file($url);
    //if cannot be found, or a parse/syntax error occurs, we return a blank array
    if(!count($feed)) {
      return array();
    } else {
      //If found, we return the newest five <item>s in the <channel>
      $out = array();
      $items = $feed->channel->item;
      for($i=0;$i<5;$i++) {
        $out[] = $items[$i];
      }
      //and we return the output
      return $out;
    }
  }
```

首先，我们使用 SimpleXML 的内置方法`simplexml_load_file()`在方法中加载 XML 反馈。如果没有找到结果或者反馈包含错误，我们就返回一个空数组。在 SimpleXML 中，所有对象及其子对象都与 XML 标签完全一样。所以如果有一个`<channel>`标签，就会有一个名为`channel`的对象，如果在`<channel>`内有`<item>`标签，那么在每个`channel`对象下面就会有一个名为`item`的对象。所以如果你想访问通道内的第一项，你可以这样访问：`$xml->channel->item[0]`。

1.  现在我们需要一个视图来显示内容。首先打开`app`下的`routes.php`，并删除默认存在的`get`路由：

```php
Route::get('/', array('as'=>'index', 'uses' =>'FeedsController@getIndex'));
```

1.  现在打开`FeedsController.php`，位于`app/controller/`下，并粘贴以下代码：

```php
public function getIndex(){
  //First we get all the records that are active category by category:
    $news_raw   = Feeds::whereActive(1)->whereCategory('News')->get();
    $sports_raw  = Feeds::whereActive(1)->whereCategory('Sports')->get();
    $technology_raw = Feeds::whereActive(1)->whereCategory('Technology')->get();

  //Now we load our view file and send variables to the view
  return View::make('index')
    ->with('news',$news_raw)
    ->with('sports',$sports_raw)
    ->with('technology',$technology_raw);
  }
```

在控制器中，我们逐个获取反馈的 URL，然后加载一个视图，并将它们逐个设置为每个类别的单独变量。

1.  现在我们需要循环每个反馈类别并显示其内容。将以下代码保存在名为`index.blade.php`的文件中，放在`app/views/`下：

```php
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Your awesome news aggregation site</title>
    <style type="text/css">
    body { font-family: Tahoma, Arial, sans-serif; }
    h1, h2, h3, strong { color: #666; }
    blockquote{ background: #bbb; border-radius: 3px; }
    li { border: 2px solid #ccc; border-radius: 5px; list-style-type: none; margin-bottom: 10px }
    a { color: #1B9BE0; }
    </style>
</head>
<body>
   <h1>Your awesome news aggregation site</h1>
   <h2>Latest News</h2>
    @if(count($news))
        {{--We loop all news feed items --}}
        @foreach($news as $each)
            <h3>News from {{$each->title}}:</h3>
            <ul>
            {{-- for each feed item, we get and parse its feed elements --}}
            <?php $feeds = Str::parse_feed($each->feed); ?>
            @if(count($feeds))
                {{-- In a loop, we show all feed elements one by one --}}
                @foreach($feeds as $eachfeed)
                    <li>
                        <strong>{{$eachfeed->title}}</strong><br />
                        <blockquote>{{Str::limit(strip_tags($eachfeed->description),250)}}</blockquote>
                        <strong>Date: {{$eachfeed->pubDate}}</strong><br />
                        <strong>Source: {{HTML::link($eachfeed->link,Str::limit($eachfeed->link,35))}}</strong>

                    </li>
                @endforeach
            @else
                <li>No news found for {{$each->title}}.</li>
            @endif
            </ul>
        @endforeach
    @else
        <p>No News found :(</p>
    @endif

    <hr />

    <h2>Latest Sports News</h2>
    @if(count($sports))
        {{--We loop all news feed items --}}
        @foreach($sports as $each)
            <h3>Sports News from {{$each->title}}:</h3>
            <ul>
            {{-- for each feed item, we get and parse its feed elements --}}
            <?php $feeds = Str::parse_feed($each->feed); ?>
            @if(count($feeds))
                {{-- In a loop, we show all feed elements one by one --}}
                @foreach($feeds as $eachfeed)
                    <li>
                        <strong>{{$eachfeed->title}}</strong><br />
                        <blockquote>{{Str::limit(strip_tags($eachfeed->description),250)}}</blockquote>
                        <strong>Date: {{$eachfeed->pubDate}}</strong><br />
                        <strong>Source: {{HTML::link($eachfeed->link,Str::limit($eachfeed->link,35))}}</strong>
                    </li>
                @endforeach
            @else
                <li>No Sports News found for {{$each->title}}.</li>
            @endif
            </ul>
        @endforeach
    @else
        <p>No Sports News found :(</p>
    @endif

    <hr />

    <h2>Latest Technology News</h2>
    @if(count($technology))
       {{--We loop all news feed items --}}
        @foreach($technology as $each)
            <h3>Technology News from {{$each->title}}:</h3>
            <ul>
            {{-- for each feed item, we get and parse its feed elements --}}
            <?php $feeds = Str::parse_feed($each->feed); ?>
            @if(count($feeds))
                {{-- In a loop, we show all feed elements one by one --}}
                @foreach($feeds as $eachfeed)
                    <li>
                        <strong>{{$eachfeed->title}}</strong><br />
                        <blockquote>{{Str::limit(strip_tags($eachfeed->description),250)}}</blockquote>
                        <strong>Date: {{$eachfeed->pubDate}}</strong><br />
                        <strong>Source: {{HTML::link($eachfeed->link,Str::limit($eachfeed->link,35))}}</strong>
                    </li>
                @endforeach
            @else
                <li>No Technology News found for {{$each->title}}.</li>
            @endif
            </ul>
        @endforeach
    @else
        <p>No Technology News found :(</p>
    @endif

</body>
</html>
```

1.  我们为每个类别写了相同的代码三次。此外，在`head`标签之间进行了一些样式处理，以便页面对最终用户看起来更漂亮。

我们用`<hr>`标签分隔了每个类别的部分。所有三个部分的工作机制都相同，除了源变量和分组。

我们首先检查每个类别是否存在记录（来自数据库的结果，因为我们可能还没有添加任何新闻源）。如果有结果，就使用 Blade 模板引擎的`@foreach()`方法循环遍历每条记录。

对于每条记录，我们首先显示反馈的友好名称（我们在保存时定义的），并使用我们刚刚创建的`parse_feed()`方法解析反馈。

在解析每个反馈后，我们查看是否找到了任何记录；如果找到了，我们再次循环它们。为了保持我们反馈阅读器的整洁，我们使用 PHP 的`strip_tags()`函数去除了所有 HTML 标签，并使用 Laravel 的`Str`类的`limit()`方法将它们限制在最多 250 个字符。

各个反馈项也有自己的标题、日期和源链接，所以我们也在反馈上显示了它们。为了防止链接破坏我们的界面，我们将文本限制在 35 个字符之间写在锚标签之间。

在所有编辑完成后，你应该得到如下输出：

![读取和解析外部反馈](img/2111OS_05_02.jpg)

# 摘要

在本章中，我们使用 Laravel 的内置函数和 PHP 的`SimpleXML`类创建了一个简单的反馈阅读器。我们学会了如何扩展核心库，编写自己的方法，并在生产中使用它们。我们还学会了在查询数据库时如何过滤结果以及如何创建记录。最后，我们学会了如何处理字符串，限制它们，并清理它们。在下一章中，我们将创建一个照片库系统。我们将确保上传的文件是照片。我们还将把照片分组到相册中，并使用 Laravel 的内置关联方法关联相册和照片。
