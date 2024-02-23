# 第三章。构建一个图片分享网站

通过这一章，我们将创建一个照片分享网站。首先，我们将创建一个图像表。然后我们将介绍调整大小和分享图像的方法。

本章涵盖以下主题：

+   创建数据库并迁移图像表

+   创建一个照片模型

+   设置自定义配置值

+   安装第三方库

+   创建一个安全的文件上传表单

+   验证和处理表单

+   使用用户界面显示图像

+   列出图像

+   从数据库和服务器中删除图像

# 创建数据库并迁移图像表

成功安装 Laravel 4 并从`app/config/database.php`中定义数据库凭据后，创建一个名为`images`的数据库。为此，您可以从托管提供商的面板上创建一个新的数据库，或者如果您是服务器管理员，您可以简单地运行以下 SQL 命令：

```php
**CREATE DATABASE images**

```

成功为应用程序创建数据库后，我们需要创建一个`photos`表并将其安装到数据库中。为此，打开您的终端，导航到项目文件夹，并运行以下命令：

```php
php artisan migrate:make create_photos_table --table=photos –create
```

这个命令将为我们生成一个新的 MySQL 数据库迁移，用于创建一个名为 photos 的表。

现在我们需要定义数据库表中应该有哪些部分。对于我们的示例，我认为`id 列`，`图像标题`，`图像文件名`和`时间戳`应该足够了。因此，打开刚刚用前面的命令创建的迁移文件，并按照以下代码更改其内容：

```php
<?php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePhotosTable extends Migration {

  /**
  * Run the migrations.
  * @return void
  */
  public function up()
    {
    Schema::create('photos', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('title',400)->default('');//the column that holds the image's name
      $table->string('image',400)->default('');//the column that holds the image's filename
      $table->timestamps();
    });
  }

  /**
  * Reverse the migrations.
  * @return void
  */
  public function down()
  {
    Schema::drop('photos');
  }
}
```

保存文件后，运行以下命令执行迁移：

```php
**php artsian migrate**

```

如果没有发生错误，您已经准备好进行项目的下一步了。

# 创建一个照片模型

如您所知，对于 Laravel 上的任何与数据库操作相关的事情，使用模型是最佳实践。我们将利用**Eloquent ORM**。

将以下代码保存为`app/models/`目录中的`images.php`：

```php
<?php
class Photo extends Eloquent {

  //the variable that sets the table name
  protected $table = 'photos';

  //the variable that sets which columns can be edited
  protected $fillable = array('title','image');

  //The variable which enables or disables the Laravel'stimestamps option. Default is true. We're leaving this hereanyways
  public $timestamps = true;
}
```

我们使用`protected $table`变量设置了表名。表的哪些列的内容可以被更新/插入将由`protected $fillable`变量决定。最后，模型是否可以添加/更新时间戳将由`public $timestamps`变量的值决定。只需设置这个模型（即使什么都不设置），我们就可以轻松地使用 Eloquent ORM 的所有优势。

我们的模型已经准备好了，现在我们可以继续下一步，开始创建我们的控制器以及上传表单。但在此之前，我们缺少一件简单的事情。图像应该上传到哪里？缩略图的最大宽度和高度是多少？要设置这些配置值（将其视为原始 PHP 的常量），我们应该创建一个新的配置文件。

# 设置自定义配置值

使用 Laravel，设置配置值非常容易。所有`config`值都在一个数组中，并且将被定义为`key=>value`对。

现在让我们创建一个新的配置文件。将此文件保存为`app/config`中的`image.php`：

```php
<?php

/**
 * app/config/image.php
*/

return array(

  //the folder that will hold original uploaded images
  'upload_folder' => 'uploads',

  //the folder that will hold thumbnails
  'thumb_folder' => 'uploads/thumbs',

  //width of the resized thumbnail
  'thumb_width' => 320,

  //height of the resized thumbnail
  'thumb_height' => 240

);
```

您可以根据自己的喜好设置任何其他设置。这取决于您的想象力。您可以使用 Laravel 内置的`Config`库的`get()`方法调用设置。示例用法如下所示：

```php
Config::get('filename.key')
```

在参数之间有一个点（`。`），它将字符串分成两部分。第一部分是`Config`的文件名，不包括扩展名，第二部分是配置值的键名。在我们的示例中，如果我们想要确定上传文件夹的名称，我们应该按照以下代码所示进行编写：

```php
Config::get('image.upload_folder')
```

前面的代码将返回任何值。在我们的示例中，它将返回`public`/`uploads`。

还有一件事：我们为我们的应用程序定义了一些文件夹名称，但我们没有创建它们。对于某些服务器配置，文件夹可能会在第一次尝试上传文件时自动创建，但如果您不创建它们，很可能会导致服务器配置错误。在`public`文件夹中创建以下文件夹，并使其可写：

+   `uploads/`

+   `uploads/thumbs`

现在我们应该为我们的图片站点制作一个上传表单。

# 安装第三方库

我们应该为我们的图片站点制作一个上传表单，然后为其创建一个控制器。但在这之前，我们将安装一个用于图像处理的第三方库，因为我们将从中受益。Laravel 4 使用**Composer**，因此安装包、更新包甚至更新 Laravel 都非常容易。对于我们的项目，我们将使用一个名为`Intervention`的库。必须按照以下步骤来安装该库：

1.  首先，确保您通过在终端中运行`php composer.phar self-update`来拥有最新的`composer.phar`文件。

1.  然后打开`composer.json`，并在`require`部分添加一个新值。我们库的值是`intervention/image: "dev-master"`。

目前，我们的`composer.json`文件的`require`部分如下所示：

```php
    "require": {
      "laravel/framework": "4.0.*",
      "intervention/image": "dev-master"
    }
    ```

您可以在[www.packagist.org](http://www.packagist.org)上找到更多 Composer 包。

1.  设置完值后，打开您的终端，导航到项目的`root`文件夹，并输入以下命令：

```php
    **php composer.phar update**

    ```

这个命令将检查`composer.json`并更新所有依赖项（包括 Laravel 本身），如果添加了新的要求，它将下载并安装它们。

1.  成功下载库后，我们现在将激活它。为此，我们参考`Intervention`类的网站。现在打开你的`app/config/app.php`，并将以下值添加到`providers`键中：

```php
    Intervention\Image\ImageServiceProvider
    ```

1.  现在，我们需要设置一个别名，以便我们可以轻松调用该类。为此，在同一文件的别名键中添加以下值：

```php
    'Image' => 'Intervention\Image\Facades\Image',
    ```

1.  该类有一个相当容易理解的注释。要调整图像大小，运行以下代码就足够了：

```php
    Image::make(Input::file('photo')->getRealPath())->resize(300, 200)->save('foo.jpg');
    ```

### 注意

有关`Intervention`类的更多信息，请访问以下网址：

[`intervention.olivervogel.net`](http://intervention.olivervogel.net)

现在，所有关于视图和表单处理的准备工作都已经完成；我们可以继续进行项目的下一步。

# 创建一个安全的文件上传表单

现在我们应该为我们的图片站点制作一个上传表单。我们必须制作一个视图文件，它将通过控制器加载。

1.  首先，打开`app/routes.php`，删除以 Laravel 开头的`Route::get()`行，并添加以下行：

```php
    //This is for the get event of the index page
    Route::get('/',array('as'=>'index_page','uses'=>'ImageController@getIndex'));
    //This is for the post event of the index.page
    Route::post('/',array('as'=>'index_page_post','before' =>'csrf', 'uses'=>'ImageController@postIndex'));
    ```

键`'as'`定义了路由的名称（类似于快捷方式）。因此，如果您为路由创建链接，即使路由的 URL 发生变化，您的应用链接也不会断开。`before`键定义了在动作开始之前将使用哪些过滤器。您可以定义自己的过滤器，或者使用内置的过滤器。我们设置了`csrf`，因此在动作开始之前将进行**CSRF**（跨站点请求伪造）检查。这样，您可以防止攻击者向您的应用程序注入未经授权的请求。您可以使用分隔符与多个过滤器；例如，`filter1|filter2`。

### 注意

您还可以直接从控制器定义 CSRF 保护。

1.  现在，让我们为控制器创建我们的第一个方法。添加一个新文件，其中包含以下代码，并将其命名为`ImageController.php`，放在`app/controllers/`中：

```php
    <?php

    class ImageController extends BaseController {

      public function getIndex()
      {
        //Let's load the form view
        return View::make('tpl.index');
      }

    }
    ```

我们的控制器是 RESTful 的；这就是为什么我们的方法 index 被命名为`getIndex()`。在这个方法中，我们只是加载一个视图。

1.  现在让我们使用以下代码为视图创建一个主页面。将此文件保存为`frontend_master.blade.php`，放在`app/views/`中：

```php
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" 
    "http://www.w3.org/TR/html4/loose.dtd">

    <html lang="en">
      <head>
      <meta http-equiv="content-type"content="text/html; charset=utf-8">
      <title>Laravel Image Sharing</title>
      {{HTML::style('css/styles.css')}}
      </head>

      <body>
        {{--Your title of the image (and yeah, blade enginehas its own commenting, cool, isn't it?)--}}
        <h2>Your Awesome Image Sharing Website</h2>

        {{--If there is an error flashdata in session(from form validation), we show the first one--}}
        @if(Session::has('errors'))
          <h3 class="error">{{$errors->first()}}</h3>
        @endif

        {{--If there is an error flashdata in session whichis set manually, we will show it--}}
        @if(Session::has('error'))
          <h3 class="error">{{Session::get('error')}}</h3>
        @endif

        {{--If we have a success message to show, we printit--}}
        @if(Session::has('success'))
          <h3 class="error">{{Session::get('success')}}</h3>
        @endif

        {{--We yield (get the contents of) the section named'content' from the view files--}}
        @yield('content')

      </body>
    </html>
    ```

要添加一个`CSS`文件（我们将在下一步中创建），我们使用`HTML`类的`style()`方法。我们的主页面产生一个名为`content`的部分，它将用`view files`部分填充。

1.  现在，让我们使用以下代码创建我们的`view file`部分。将此文件保存为`index.blade.php`，放在`app/views/tpl/`目录中：

```php
    @extends('frontend_master')

    @section('content')
      {{Form::open(array('url' => '/', 'files' => true))}}
      {{Form::text('title','',array('placeholder'=>'Please insert your title here'))}}
      {{Form::file('image')}}
      {{Form::submit('save!',array('name'=>'send'))}}
      {{Form::close()}}
    @stop
    ```

在上述代码的第一行中，我们告诉 Blade 引擎，我们将使用`frontend_master.blade.php`作为布局。这是使用 Laravel 4 中的`@extends()`方法完成的。

### 注意

如果您来自 Laravel 3，`@layout`已更名为`@extends`。

借助 Laravel 的`Form`类，我们生成了一个带有`title`字段和`upload`字段的上传表单。与 Laravel 3 不同，要创建一个新的上传表单，我们不再使用`Form::open_for_files()`。它已与`open()`方法合并，该方法接受一个字符串或一个数组，如果要传递多个参数，则可以使用数组。我们将传递动作 URL 以及告诉它这是一个上传表单，因此我们传递了两个参数。`url`键用于定义表单将被提交的位置。`files`参数是布尔值，如果设置为`true`，它将使表单成为上传表单，因此我们可以处理文件。

为了保护表单并防止不必要的表单提交尝试，我们需要在表单中添加一个 CSRF 密钥`hidden`。多亏了 Laravel 的`Form`类，它会在表单打开标签后自动生成。您可以通过查看生成的表单的源代码来检查它。

自动生成的隐藏 CSRF 表单元素如下所示：

```php
    <input name="_token" type="hidden" value="SnRocsQQlOnqEDH45ewP2GLxPFUy5eH4RyLzeKm3">
    ```

1.  现在让我们稍微整理一下表单。这与我们的项目没有直接关系，只是为了外观。将`styles.css`文件保存在`public/css/`（我们在主页面上定义的路径）中：

```php
    /*Body adjustment*/
    body{width:60%; margin:auto; background:#dedede}
    /*The title*/
    h2{font-size:40px; text-align:center; font-family:Tahoma,Arial,sans-serif}
    /*Sub title (success and error messages)*/
    h3{font-size:25px; border-radius:4px; font-family:Tahoma,Arial,sans-serif; text-align:center;width:100%}
    h3.error{border:3px solid #d00; background-color:#f66; color:#d00 }
    h3.success{border:3px solid #0d0; background-color:#0f0; color:#0d0}p{font-size:25px; font-weight: bold; color: black;font-family: Tahoma,Arial,sans-serif}ul{float:left;width:100%;list-style:none}li{float:left;margin-right:10px}
    /*For the input files of the form*/
    input{float:left; width:100%; border-radius:13px;font-size:20px; height:30px; border:10px 0 10px 0;margin-bottom:20px}
    ```

我们通过将其宽度设置为 60％，使其居中对齐，并给它一个灰色的背景来样式化主体。我们还使用`success`和`error`类以及`forms`格式化了`h2`和`h3`消息。

在样式化之后，表单将如下截图所示：

![创建安全的文件上传表单](img/2111OS_03_01.jpg)

现在我们的表单已经准备好了，我们准备进入项目的下一步。

# 验证和处理表单

在本节中，我们将验证提交的表单，并确保必填字段存在，并且提交的文件是一张图片。然后我们将上传图片到我们的服务器，处理图片，创建缩略图，并将图片信息保存到数据库中，如下所示：

1.  首先，我们需要定义表单验证规则。我们更喜欢将这些值添加到相关模型中，这样规则就可以重复使用，这可以防止代码变得臃肿。为此，请在`app/models/`目录中的`photo.php`文件中添加以下代码（在本章前面生成的模型中）类定义的最后一个右花括号（`}`）之前：

```php
    //rules of the image upload form
    public static $upload_rules = array(
      'title'=> 'required|min:3',
      'image'=> 'required|image'
    );
    ```

我们将变量设置为`public`，这样它就可以在模型文件之外使用，并将其设置为静态，这样我们就可以直接访问变量。

我们希望`title`和`image`都是必填的，而`title`应至少包含三个字符。此外，我们希望检查`image`列的 MIME 类型，并确保它是一张图片。

### 注意

Laravel 的 MIME 类型检查需要安装`Fileinfo`扩展。因此，请确保在您的 PHP 配置中启用它。

1.  现在我们需要控制器的`post`方法来处理表单。在`app/controllers/`中的`ImageController.php`文件中添加此方法，放在最后一个右花括号（`}`）之前：

```php
    public function postIndex()
    {

      //Let's validate the form first with the rules which areset at the model
      $validation = Validator::make(Input::all(),Photo::$upload_rules);

      //If the validation fails, we redirect the user to theindex page, with the error messages 
      if($validation->fails()) {
        return Redirect::to('/')->withInput()->withErrors($validation);
      }
      else {

        //If the validation passes, we upload the image to thedatabase and process it
        $image = Input::file('image');

        //This is the original uploaded client name of theimage
        $filename = $image->getClientOriginalName();
        //Because Symfony API does not provide filename//without extension, we will be using raw PHP here
        $filename = pathinfo($filename, PATHINFO_FILENAME);

        //We should salt and make an url-friendly version of//the filename
        //(In ideal application, you should check the filename//to be unique)
        $fullname = Str::slug(Str::random(8).$filename).'.'.$image->getClientOriginalExtension();

        //We upload the image first to the upload folder, thenget make a thumbnail from the uploaded image
        $upload = $image->move(Config::get( 'image.upload_folder'),$fullname);

        //Our model that we've created is named Photo, thislibrary has an alias named Image, don't mix them two!
        //These parameters are related to the image processingclass that we've included, not really related toLaravel
        Image::make(Config::get( 'image.upload_folder').'/'.$fullname)->resize(Config::get( 'image.thumb_width'),null, true)->save(Config::get( 'image.thumb_folder').'/'.$fullname);

        //If the file is now uploaded, we show an error messageto the user, else we add a new column to the databaseand show the success message
        if($upload) {

          //image is now uploaded, we first need to add columnto the database
          $insert_id = DB::table('photos')->insertGetId(
            array(
              'title' => Input::get('title'),
              'image' => $fullname
            )
          );

          //Now we redirect to the image's permalink
          return Redirect::to(URL::to('snatch/'.$insert_id))->with('success','Your image is uploadedsuccessfully!');
        } else {
          //image cannot be uploaded
          return Redirect::to('/')->withInput()->with('error','Sorry, the image could not beuploaded, please try again later');
        }
      }
    }
    ```

让我们逐行查看代码。

1.  首先，我们进行了表单验证，并从我们通过`Photo::$upload_rules`生成的模型中调用了我们的验证规则。

1.  然后，我们对文件名进行了加盐处理（添加额外的随机字符以增强安全性），并使文件名适合 URL。首先，我们使用 getClientOriginalName()方法获取上传的文件名，然后使用 getClientOriginalExtension()方法获取扩展名。我们使用 STR 类的 random()方法获得一个八个字符长的随机字符串对文件名进行了加盐处理。最后，我们使用 Laravel 的内置 slug()方法使文件名适合 URL。

1.  在所有变量准备就绪后，我们首先使用 move()方法将文件上传到服务器，该方法需要两个参数。第一个参数是文件将要传输到的路径，第二个参数是上传文件的文件名。

1.  上传后，我们为上传的图像创建了一个静态缩略图。为此，我们利用了之前实现的图像处理类 Intervention。

1.  最后，如果一切顺利，我们将标题和图像文件名添加到数据库，并使用 Fluent Query Builder 的 insertGetId()方法获取 ID，该方法首先插入行，然后返回列的 insert_id。我们还可以通过将 create()方法设置为变量并获取 id_column 名称，如$create->id，使用 Eloquent ORM 创建行。

1.  在一切都正常并且我们获得了`insert_id`之后，我们将用户重定向到一个新页面，该页面将显示缩略图、完整图像链接和一个论坛缩略图**BBCode**，我们将在接下来的部分中生成。

# 使用用户界面显示图像

现在，我们需要从控制器创建一个新的视图和方法来显示上传的图像的信息。可以按以下方式完成：

1.  首先，我们需要为控制器定义一个`GET`路由。为此，打开`app`文件夹中的`routes.php`文件，并添加以下代码：

```php
    //This is to show the image's permalink on our website
    Route::get('snatch/{id}',
      array('as'=>'get_image_information',
      'uses'=>'ImageController@getSnatch'))
      ->where('id', '[0-9]+');
    ```

我们在路由上定义了一个`id`变量，并使用正则表达式的`where()`方法首先进行了过滤。因此，我们不需要担心过滤 ID 字段，无论它是自然数还是其他。

1.  现在，让我们创建我们的控制器方法。在`app/controllers/`中的`ImageController.php`中最后一个右花括号(`}`)之前添加以下代码：

```php
    public function getSnatch($id) {
      //Let's try to find the image from database first
      $image = Photo::find($id);
      //If found, we load the view and pass the image info asparameter, else we redirect to main page with errormessage
      if($image) {
        return View::make('tpl.permalink')->with('image',$image);
      } else {
        return Redirect::to('/')->with('error','Image not found');
      }
    }
    ```

首先，我们使用 Eloquent ORM 的`find()`方法查找图像。如果它返回 false，那意味着找到了一行。因此，我们可以简单地使用一个简单的`if`子句来检查是否有结果。如果有结果，我们将使用`with()`方法将找到的图像信息作为名为`$image`的变量加载到我们的视图中。如果没有找到值，我们将返回到索引页面并显示错误消息。

1.  现在让我们创建包含以下代码的模板文件。将此文件保存为`permalink.blade.php`，放在`app/views/tpl/`中：

```php
    @extends('frontend_master')
    @section('content')
    <table cellpadding="0" cellspacing="0" border="0"width="100percent">
      <tr>
        <td width="450" valign="top">
          <p>Title: {{$image->title}}</p>
        {{HTML::image(Config::get('image.thumb_folder').'/'.$image->image)}}
        </td>
          <td valign="top">
          <p>Direct Image URL</p>
          <input onclick="this.select()" type="text"width="100percent" value="{{URL::to(Config::get('image.upload_folder').'/'$image->image)}}" />

          <p>Thumbnail Forum BBCode</p>
          <input onclick="this.select()" type="text"width="100percent" value="[url={{URL::to('snatch/'$image->id)}}][img]{{URL::to(Config::get('image.thumb_folder')'/'.$image->image)}}[/img][/url]" />

          <p>Thumbnail HTML Code</p>
          <input onclick="this.select()" type="text"width="100percent"value="{{HTML::entities(HTML::link(URL::to('snatch/'.$image->id),HTML::image(Config::get('image.thumb_folder').'/'$image->image)))}}" />
        </td>
      </tr>
    </table>
    @stop
    ```

现在，您应该对此模板中使用的大多数方法都很熟悉了。还有一个名为`entities()`的新方法，属于`HTML`类，实际上是原始 PHP 的`htmlentities()`，但带有一些预检查，并且是 Laravel 的方式。

此外，因为我们将`$image`变量返回到视图中（这是我们使用 Eloquent 直接获得的数据库行对象），我们可以在视图中直接使用`$image->columnName`。

这将产生一个视图，如下图所示：

![使用用户界面显示图像](img/2111OS_03_02.jpg)

1.  我们为项目添加了永久链接功能，但是如果我们想要显示所有图像怎么办？为此，我们需要在系统中添加一个`'all pages'`部分。

# 列出图像

在本节中，我们将在系统中创建一个`'all images'`部分，该部分将具有页面导航（分页）系统。如下所示，需要遵循一些步骤：

1.  首先，我们需要从我们的`route.php`文件中定义其 URL。为此，打开`app/routes.php`并添加以下行：

```php
    //This route is to show all images.
    Route::get('all',array('as'=>'all_images','uses'=>'ImageController@getAll'));
    ```

1.  现在，我们需要一个名为`getAll()`的方法（因为它将是一个 RESTful 控制器，所以在开头有一个`get`方法）来获取值并加载视图。为此，请打开`app/controllers/ImageController.php`，并在最后一个右花括号（}）之前添加以下代码：

```php
    public function getAll(){

      //Let's first take all images with a pagination feature
      $all_images = DB::table('photos')->orderBy('id','desc')->paginate(6);

      //Then let's load the view with found data and pass thevariable to the view
      return View::make('tpl.all_images')->with('images',$all_images);
    }
    ```

首先，我们使用`paginate()`方法从数据库中获取了所有图像，这将使我们能够轻松获取分页链接。之后，我们加载了用户的视图，并显示了带有分页的图像数据。

1.  要正确查看这个，我们需要一个视图文件。将以下代码保存在名为`all_image.blade.php`的文件中，放在`app/views/tpl/`目录中：

```php
    @extends('frontend_master')

    @section('content')

    @if(count($images))
      <ul>

        @foreach($images as $each)
          <li>
            <a href="{{URL::to('snatch/'$each->id)}}">{{HTML::image(Config::get('image.thumb_folder')'/'.$each->image)}}</a>
          </li>
        @endforeach
      </ul> 
      <p>{{$images->links()}}</p>
    @else
      {{--If no images are found on the database, we will showa no image found error message--}}
      <p>No images uploaded yet, {{HTML::link('/','care to upload one?')}}</p>
    @endif
    @stop
    ```

我们首先用我们的内容部分扩展了`frontend_master.blade.php`文件。至于内容部分，我们首先检查是否返回了任何行。如果是，那么我们将它们全部循环在列表项标签（`<li>`）中，并附上它们的永久链接。`paginate`类提供的`links()`方法将为我们创建分页。

### 注意

您可以从`app/config/view.php`切换分页模板。

如果没有返回行，那意味着还没有图像，因此我们会显示一个警告消息，并附上一个指向新上传页面的链接（在我们的情况下是首页）。

如果有人上传了不允许或不安全的工作图像，怎么办？您肯定不希望它们出现在您的网站上，对吧？因此，您的网站应该有一个图像删除功能。

# 从数据库和服务器中删除图像

我们希望在我们的脚本中有一个删除功能，使用该功能我们将从数据库和上传的文件夹中删除图像。使用 Laravel，这个过程非常简单。

1.  首先，我们需要为该操作创建一个新路由。为此，请打开`app/routes.php`，并添加以下行：

```php
    //This route is to delete the image with given ID
    Route::get('delete/{id}', array
    ('as'=>'delete_image','uses'=>
    'ImageController@getDelete'))
    ->where('id', '[0-9]+');
    ```

1.  现在，我们需要在`ImageController`中定义控制器方法`getDelete($id)`。为此，请打开`app/controllers/ImageController.php`，并在最后一个右花括号（`}`）之前添加以下代码：

```php
    public function getDelete($id) {
      //Let's first find the image
      $image = Photo::find($id);

      //If there's an image, we will continue to the deletingprocess
      if($image) {
        //First, let's delete the images from FTP
        File::delete(Config::get('image.upload_folder').'/'$image->image);
        File::delete(Config::get('image.thumb_folder').'/'$image->image);

        //Now let's delete the value from database
        $image->delete();

        //Let's return to the main page with a success message
        return Redirect::to('/')->with('success','Image deleted successfully');

      } else {
        //Image not found, so we will redirect to the indexpage with an error message flash data.
        return Redirect::to('/')->with('error','No image with given ID found');
      }
    }
    ```

让我们理解这段代码：

1.  首先，我们查看我们的数据库，如果我们已经有了给定 ID 的图像，则使用 Eloquent ORM 的`find()`方法将其存储在名为`$image`的变量中。

1.  如果`$image`的值不为 false，则数据库中有与图像匹配的图像。然后，我们使用 File 类的`delete()`方法删除文件。或者，您也可以使用原始 PHP 的 unlink()方法。

1.  在文件从文件服务器中删除后，我们将从数据库中删除图像的信息行。为此，我们使用了 Eloquent ORM 的`delete()`方法。

1.  如果一切顺利，我们应该重定向回主页，并显示成功消息，说明图像已成功删除。

### 注意

在实际应用中，您应该为此类操作创建一个后端界面。

# 总结

在本章中，我们使用 Laravel 的内置功能创建了一个简单的图像分享网站。我们学会了如何验证我们的表单，如何处理文件并检查它们的 MIME 类型，并设置自定义配置值。我们还学习了使用 Fluent 和 Eloquent ORM 的数据库方法。此外，对于图像处理，我们使用 Composer 从[packagist.org](http://packagist.org)安装了第三方库，并学会了如何更新它们。我们还使用页面导航功能列出了图像，并学会了如何从服务器中删除文件。在下一章中，我们将构建一个带有身份验证和仅限会员区域的个人博客网站，并将博客文章分配给作者。
