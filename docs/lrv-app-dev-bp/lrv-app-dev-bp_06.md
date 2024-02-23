# 第六章 创建照片库系统

在本章中，我们将使用 Laravel 编写一个简单的照片库系统。我们还将涵盖 Laravel 内置的文件验证、文件上传和**hasMany**数据库关系机制。我们将使用`validation`类来验证数据和文件。此外，我们还将涵盖用于处理文件的文件类。本章涵盖以下主题：

+   创建相册模型

+   创建图像模型

+   创建相册

+   创建照片上传表单

+   在相册之间移动照片

# 创建相册表并迁移

我们假设你已经在`app/config/`目录下的`database.php`文件中定义了数据库凭据。要构建一个照片库系统，我们需要一个包含两个表`albums`和`images`的数据库。要创建一个新数据库，只需运行以下 SQL 命令：

```php
**CREATE DATABASE laravel_photogallery**

```

成功创建应用程序的数据库后，我们首先需要创建`albums`表并将其安装到数据库中。为此，请打开终端，导航到项目文件夹，运行以下命令：

```php
**php artisan migrate:make create_albums_table --table=albums --create**

```

上述命令将在`app/database/migrations`下生成一个迁移文件，用于在我们的`laravel_photogallery`数据库中生成一个名为`posts`的新 MySQL 表。

为了定义我们的表列，我们需要编辑迁移文件。编辑后，文件应该包含以下代码：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateAlbumsTable extends Migration {

  /**
  * Run the migrations.
  *
  * @return void
  */
  public function up()
    {
      Schema::create('albums', function(Blueprint $table)
      {
        $table->increments('id')->unsigned();
        $table->string('name');
        $table->text('description');
        $table->string('cover_image');
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
    Schema::drop('albums');
  }
}
```

保存文件后，我们需要再次使用简单的 artisan 命令来执行迁移：

```php
**php artisan migrate**

```

如果没有发生错误，请检查`laravel_photogallery`数据库的`albums`表及其列。

让我们检查以下列表中的列：

+   `id`：此列用于存储相册的 ID

+   `name`：此列用于存储相册的名称

+   `description`：此列用于存储相册的描述

+   `cover_image`：此列用于存储相册的封面图像

我们已成功创建了`albums`表，现在需要编写我们的**Album**模型。

## 创建相册模型

如你所知，对于 Laravel 上的任何与数据库操作相关的事情，使用模型是最佳实践。我们将受益于使用 Eloquent ORM。

将以下代码保存为`Album.php`，放在`app/models/`目录中：

```php
<?php
class Album extends Eloquent {

  protected $table = 'albums';

  protected $fillable = array('name','description','cover_image');

  public function Photos(){

    return $this->has_many('images');
  }
}
```

我们使用`protected $table`变量设置了数据库表名；我们还使用了`protected $fillable`变量设置了可编辑的列，这是我们在之前章节中已经见过和使用过的。模型中定义的变量足以使用 Laravel 的 Eloquent ORM。我们将在本章的*将照片分配给相册*部分中介绍`public Photos()`函数。

我们的**Album**模型已准备好；现在我们需要一个**Image**模型和一个分配照片到相册的数据库。让我们创建它们。

# 使用迁移类创建图像数据库

要为图像创建我们的迁移文件，打开终端，导航到项目文件夹，运行以下命令：

```php
**php artisan migrate:make create_images_table --table=images --create**

```

如你所知，该命令将在`app/database/migrations`中生成一个迁移文件。让我们编辑迁移文件；最终代码应该如下所示：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateImagesTable extends Migration {

  /**
  * Run the migrations.
  *
  * @return void
  */
  public function up()
  {
    Schema::create('images', function(Blueprint $table)
    {
      $table->increments('id')->unsigned();
      $table->integer('album_id')->unsigned();
      $table->string('image');
      $table->string('description');
      $table->foreign('album_id')->references('id')->on('albums')->onDelete('CASCADE')->onUpdate('CASCADE');
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
    Schema::drop('images');
  }
}
```

编辑迁移文件后，运行以下迁移命令：

```php
**php artisan migrate**

```

如你所知，该命令创建了`images`表及其列。如果没有发生错误，请检查`laravel_photogallery`数据库的`users`表及其列。

让我们检查以下列表中的列：

+   `id`：此列用于存储图像的 ID

+   `album_id`：此列用于存储图像所属相册的 ID

+   `description`：此列用于存储图像的描述

+   `image`：此列用于存储图像的路径

我们需要解释一下这个迁移文件的另一件事。如你在迁移代码中所见，有一个`foreign`键。当我们需要链接两个表时，我们使用`foreign`键。我们有一个`albums`表，每个相册都会有图像。如果从数据库中删除相册，你也希望删除其所有图像。

# 创建一个 Image 模型

我们已经创建了`images`表。所以，你知道，我们需要一个模型来在 Laravel 上操作数据库表。为了创建它，将以下代码保存为 Image.php 在`app/models/`目录中：

```php
class Images extends Eloquent {

  protected $table = 'images';

  protected $fillable = array('album_id','description','image');

}
```

我们的**Image**模型已经准备好了；现在我们需要一个控制器来在我们的数据库上创建相册。让我们来创建它。

# 创建相册

正如你从本书的前几章中所了解的，Laravel 拥有一个很棒的 RESTful 控制器机制。我们将继续使用它来在开发过程中保持代码简单和简洁。在接下来的章节中，我们将介绍另一种很棒的控制器/路由方法，名为**资源控制器**。

为了列出、创建和删除相册，我们需要在我们的控制器中添加一些函数。为了创建它们，将以下代码保存为`AlbumsController.php`在`app/controllers/`目录中：

```php
<?php

class AlbumsController extends BaseController{

  public function getList()
  {
    $albums = Album::with('Photos')->get();
    return View::make('index')
    ->with('albums',$albums);
  }
  public function getAlbum($id)
  {
    $album = Album::with('Photos')->find($id);
    return View::make('album')
    ->with('album',$album);
  }
  public function getForm()
  {
    return View::make('createalbum');
  }
  public function postCreate()
  {
    $rules = array(

      'name' => 'required',
      'cover_image'=>'required|image'

    );

    $validator = Validator::make(Input::all(), $rules);
    if($validator->fails()){

      return Redirect::route('create_album_form')
      ->withErrors($validator)
      ->withInput();
    }

    $file = Input::file('cover_image');
    $random_name = str_random(8);
    $destinationPath = 'albums/';
    $extension = $file->getClientOriginalExtension();
    $filename=$random_name.'_cover.'.$extension;
    $uploadSuccess = Input::file('cover_image')
    ->move($destinationPath, $filename);
    $album = Album::create(array(
      'name' => Input::get('name'),
      'description' => Input::get('description'),
      'cover_image' => $filename,
    ));

    return Redirect::route('show_album',array('id'=>$album->id));
  }

  public function getDelete($id)
  {
    $album = Album::find($id);

    $album->delete();

    return Redirect::route('index');
  }
}
```

`postCreate()`函数首先验证表单提交的数据。我们将在下一节中介绍验证。如果数据验证成功，我们将重命名封面图像并使用新文件名上传它，因为代码会覆盖具有相同名称的文件。

`getDelete()`函数正在从数据库中删除相册以及分配的图像（存储在`images`表中）。请记住以下迁移文件代码：

```php
$table->foreign('album_id')->references('id')->on('albums')->onDelete('CASCADE')->onUpdate('CASCADE');
```

在创建我们的模板之前，我们需要定义路由。因此，打开`app`文件夹中的`routes.php`文件，并用以下代码替换它：

```php
<?php
Route::get('/', array('as' => 'index','uses' => 'AlbumsController@getList'));
Route::get('/createalbum', array('as' => 'create_album_form','uses' => 'AlbumsController@getForm'));
Route::post('/createalbum', array('as' => 'create_album','uses' => 'AlbumsController@postCreate'));
Route::get('/deletealbum/{id}', array('as' => 'delete_album','uses' => 'AlbumsController@getDelete'));
Route::get('/album/{id}', array('as' => 'show_album','uses' => 'AlbumsController@getAlbum'));
```

现在，我们需要一些模板文件来显示、创建和列出相册。首先，我们应该创建索引模板。为了创建它，将以下代码保存为`index.blade.php`在`app/views/`目录中：

```php
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Awesome Albums</title>
    <!-- Latest compiled and minified CSS -->
    <link href="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/css/bootstrap.min.css" rel="stylesheet">

    <!-- Latest compiled and minified JavaScript -->
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/js/bootstrap.min.js"></script>
    <style>
      body {
        padding-top: 50px;
      }
      .starter-template {
        padding: 40px 15px;
      text-align: center;
      }
    </style>
  </head>
  <body>
    <div class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
      <button type="button" class="navbar-toggle"data-toggle="collapse" data-target=".nav-collapse">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="/">Awesome Albums</a>
      <div class="nav-collapse collapse">
        <ul class="nav navbar-nav">
          <li><a href="{{URL::route('create_album_form')}}">Create New Album</a></li>
        </ul>
      </div><!--/.nav-collapse -->
    </div>
    </div>

      <div class="container">

        <div class="starter-template">

        <div class="row">
          @foreach($albums as $album)
            <div class="col-lg-3">
              <div class="thumbnail" style="min-height: 514px;">
                <img alt="{{$album->name}}" src="/albums/{{$album->cover_image}}">
                <div class="caption">
                  <h3>{{$album->name}}</h3>
                  <p>{{$album->description}}</p>
                  <p>{{count($album->Photos)}} image(s).</p>
                  <p>Created date:  {{ date("d F Y",strtotime($album->created_at)) }} at {{date("g:ha",strtotime($album->created_at)) }}</p>
                  <p><a href="{{URL::route('show_album', array('id'=>$album->id))}}" class="btn btn-big btn-default">Show Gallery</a></p>
                </div>
              </div>
            </div>
          @endforeach
        </div>

      </div><!-- /.container -->
    </div>

  </body>
</html>
```

## 为创建相册添加模板

正如你在以下代码中所看到的，我们更喜欢使用 Twitter 的 bootstrap **CSS**框架。这个框架允许你快速创建有用、响应式和多浏览器支持的界面。接下来，我们需要为创建相册创建一个模板。为了创建它，将以下代码保存为`createalbum.blade.php`在`app/views/`目录中：

```php
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>Create an Album</title>
    <!-- Latest compiled and minified CSS -->
    <link href="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/css/bootstrap.min.css" rel="stylesheet">

    <!-- Latest compiled and minified JavaScript -->
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/js/bootstrap.min.js"></script>
  </head>
  <body>
    <div class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <button type="button" class="navbar-toggle"data-toggle="collapse" data-target=".nav-collapse">
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span lclass="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="/">Awesome Albums</a>
        <div class="nav-collapse collapse">
          <ul class="nav navbar-nav">
            <li class="active"><ahref="{{URL::route('create_album_form')}}">CreateNew Album</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </div>
    <div class="container" style="text-align: center;">
      <div class="span4" style="display: inline-block;margin-top:100px;">

        @if($errors->has())
          <div class="alert alert-block alert-error fade in"id="error-block">
             <?php
             $messages = $errors->all('<li>:message</li>');
            ?>
            <button type="button" class="close"data-dismiss="alert">×</button>

            <h4>Warning!</h4>
            <ul>
              @foreach($messages as $message)
                {{$message}}
              @endforeach

            </ul>
          </div>
        @endif

        <form name="createnewalbum" method="POST"action="{{URL::route('create_album')}}"enctype="multipart/form-data">
          <fieldset>
            <legend>Create an Album</legend>
            <div class="form-group">
              <label for="name">Album Name</label>
              <input name="name" type="text" class="form-control"placeholder="Album Name"value="{{Input::old('name')}}">
            </div>
            <div class="form-group">
              <label for="description">Album Description</label>
              <textarea name="description" type="text"class="form-control" placeholder="Albumdescription">{{Input::old('descrption')}}</textarea>
            </div>
            <div class="form-group">
              <label for="cover_image">Select a Cover Image</label>
              {{Form::file('cover_image')}}
            </div>
            <button type="submit" class="btnbtn-default">Create!</button>
          </fieldset>
        </form>
      </div>
    </div> <!-- /container -->
  </body>
</html>
```

该模板创建了一个基本的上传表单，并显示了从控制器端传递的验证错误。我们只需要再创建一个模板文件来列出相册图像。因此，为了创建它，将以下代码保存为`album.blade.php`在`app/views/`目录中：

```php
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>{{$album->name}}</title>
    <!-- Latest compiled and minified CSS -->
    <link href="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/css/bootstrap.min.css" rel="stylesheet">

    <!-- Latest compiled and minified JavaScript -->
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/js/bootstrap.min.js"></script>
    <style>
      body {
        padding-top: 50px;
      }
      .starter-template {
        padding: 40px 15px;
        text-align: center;
      }
    </style>
  </head>
  <body>
    <div class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <button type="button" class="navbar-toggle"data-toggle="collapse" data-target=".nav-collapse">
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="/">Awesome Albums</a>
        <div class="nav-collapse collapse">
          <ul class="nav navbar-nav">
            <li><a href="{{URL::route('create_album_form')}}">Create New Album</a></li>
          </ul>
        </div><!--/.nav-collapse -->
     </div>
    </div>
    <div class="container">

      <div class="starter-template">
        <div class="media">
          <img class="media-object pull-left"alt="{{$album->name}}" src="/albums/{{$album->cover_image}}" width="350px">
          <div class="media-body">
            <h2 class="media-heading" style="font-size: 26px;">Album Name:</h2>
            <p>{{$album->name}}</p>
          <div class="media">
          <h2 class="media-heading" style="font-size: 26px;">AlbumDescription :</h2>
          <p>{{$album->description}}<p>
          <a href="{{URL::route('add_image',array('id'=>$album->id))}}"><button type="button"class="btn btn-primary btn-large">Add New Image to Album</button></a>
          <a href="{{URL::route('delete_album',array('id'=>$album->id))}}" onclick="return confirm('Are yousure?')"><button type="button"class="btn btn-danger btn-large">Delete Album</button></a>
        </div>
      </div>
    </div>
    </div>
      <div class="row">
        @foreach($album->Photos as $photo)
          <div class="col-lg-3">
            <div class="thumbnail" style="max-height: 350px;min-height: 350px;">
              <img alt="{{$album->name}}" src="/albums/{{$photo->image}}">
              <div class="caption">
                <p>{{$photo->description}}</p>
                <p><p>Created date:  {{ date("d F Y",strtotime($photo->created_at)) }} at {{ date("g:ha",strtotime($photo->created_at)) }}</p></p>
                <a href="{{URL::route('delete_image',array('id'=>$photo->id))}}" onclick="return confirm('Are you sure?')"><button type="button" class="btnbtn-danger btn-small">Delete Image </button></a>
              </div>
            </div>
          </div>
        @endforeach
      </div>
    </div>

  </body>
</html>
```

正如你可能记得的，我们在模型端使用了`hasMany()` Eloquent 方法。在控制器端，我们使用以下函数：

```php
**$albums = Album::with('Photos')->get();**

```

该代码在数组中获取了属于相册的整个图像数据。因此，我们在以下模板中使用`foreach`循环：

```php
@foreach($album->Photos as $photo)
  <div class="col-lg-3">
    <div class="thumbnail" style="max-height: 350px;min-height: 350px;">
    <img alt="{{$album->name}}" src="/albums/{{$photo->image}}">
      <div class="caption">
        <p>{{$photo->description}}</p>
        <p><p>Created date:  {{ date("d F Y",strtotime($photo->created_at)) }} at {{ date("g:ha",strtotime($photo->created_at)) }}</p></p>
        <a href="{{URL::route('delete_image',array('id'=>$photo->id))}}" onclick="return confirm('Are yousure?')"><button type="button" class="btnbtn-danger btn-small">Delete Image</button></a>
      </div>
    </div>
  </div>
@endforeach
```

# 创建一个照片上传表单

现在我们需要创建一个照片上传表单。我们将上传照片并将它们分配到相册中。让我们首先设置路由；打开`app`文件夹中的`routes.php`文件，并添加以下代码：

```php
Route::get('/addimage/{id}', array('as' => 'add_image','uses' => 'ImagesController@getForm'));
Route::post('/addimage', array('as' => 'add_image_to_album','uses' => 'ImagesController@postAdd'));
Route::get('/deleteimage/{id}', array('as' => 'delete_image','uses' => 'ImagesController@getDelete'));
```

我们需要一个照片上传表单的模板。为了创建它，将以下代码保存为`addimage.blade.php`在`app/views/`目录中：

```php
<!doctype html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>Laravel PHP Framework</title>
    <!-- Latest compiled and minified CSS -->
    <link href="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/css/bootstrap.min.css" rel="stylesheet">

    <!-- Latest compiled and minified JavaScript -->
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.0-rc1/js/bootstrap.min.js"></script>
  </head>
  <body>

    <div class="container" style="text-align: center;">
      <div class="span4" style="display: inline-block;margin-top:100px;">
        @if($errors->has())
          <div class="alert alert-block alert-error fade in"id="error-block">
            <?php
            $messages = $errors->all('<li>:message</li>');
            ?>
            <button type="button" class="close"data-dismiss="alert">×</button>

            <h4>Warning!</h4>
            <ul>
              @foreach($messages as $message)
                {{$message}}
              @endforeach

            </ul>
          </div>
        @endif
        <form name="addimagetoalbum" method="POST"action="{{URL::route('add_image_to_album')}}"enctype="multipart/form-data">
          <input type="hidden" name="album_id"value="{{$album->id}}" />
          <fieldset>
            <legend>Add an Image to {{$album->name}}</legend>
            <div class="form-group">
              <label for="description">Image Description</label>
              <textarea name="description" type="text"class="form-control" placeholder="Imagedescription"></textarea>
            </div>
            <div class="form-group">
              <label for="image">Select an Image</label>
              {{Form::file('image')}}
            </div>
            <button type="submit" class="btnbtn-default">Add Image!</button>
          </fieldset>
        </form>
      </div>
    </div> <!-- /container -->
  </body>
</html>
```

在创建模板之前，我们需要编写我们的控制器。因此，将以下代码保存为`ImageController.php`在`app/controllers/`目录中：

```php
<?php
class ImagesController extends BaseController{

  public function getForm($id)
  {
    $album = Album::find($id);
    return View::make('addimage')
    ->with('album',$album);
  }

  public function postAdd()
  {
    $rules = array(

      'album_id' => 'required|numeric|exists:albums,id',
      'image'=>'required|image'

    );

    $validator = Validator::make(Input::all(), $rules);
    if($validator->fails()){

      return Redirect::route('add_image',array('id' =>Input::get('album_id')))
      ->withErrors($validator)
      ->withInput();
    }

    $file = Input::file('image');
    $random_name = str_random(8);
    $destinationPath = 'albums/';
    $extension = $file->getClientOriginalExtension();
    $filename=$random_name.'_album_image.'.$extension;
    $uploadSuccess = Input::file('image')->move($destinationPath, $filename);
    Image::create(array(
      'description' => Input::get('description'),
      'image' => $filename,
      'album_id'=> Input::get('album_id')
    ));

    return Redirect::route('show_album',array('id'=>Input::get('album_id')));
  }
  public function getDelete($id)
  {
    $image = Image::find($id);
    $image->delete();
    return Redirect::route('show_album',array('id'=>$image->album_id));
  }
}
```

控制器有三个函数；第一个是`getForm()`函数。这个函数基本上显示了我们的照片上传表单。第二个函数验证并将数据插入数据库。我们将在下一节中解释验证和插入函数。第三个是`getDelete()`函数。这个函数基本上从数据库中删除图像记录。

## 验证照片

Laravel 拥有强大的验证库，在本书中已经多次提到。我们在控制器中验证数据如下：

```php
$rules = array(

  'album_id' => 'required|numeric|exists:albums,id',
  'image'=>'required|image'

);

$validator = Validator::make(Input::all(), $rules);
if($validator->fails()){

  return Redirect::route('add_image',array('id' =>Input::get('album_id')))
  ->withErrors($validator)
  ->withInput();
}
```

让我们来看一下代码。我们在`array`中定义了一些规则。在`rules`数组中有两个验证规则。第一个规则如下：

```php
'album_id' => 'required|numeric|exists:albums,id'
```

前面的规则意味着`album_id`字段是必需的（必须在表单中发布），必须是数值，并且必须存在于`albums`表的`id`列中，因为我们想要将图片分配给`albums`。第二条规则如下：

```php
'image'=>'required|image'
```

前面的规则意味着`image`字段是必需的（必须在表单中发布），其内容必须是图片。然后我们使用以下代码检查发布的表单数据：

```php
$validator = Validator::make(Input::all(), $rules);
```

验证函数需要两个变量。第一个是我们需要验证的数据。在这种情况下，我们使用`Input::all()`方法进行设置，这意味着我们需要验证发布的表单数据。第二个是`rules`变量。`rules`变量必须设置为一个数组，如下所示：

```php
$rules = array(

  'album_id' => 'required|numeric|exists:albums,id',
  'image'=>'required|image'

);
```

Laravel 的验证类带有许多预定义规则。您可以在[`laravel.com/docs/validation#available-validation-rules`](http://laravel.com/docs/validation#available-validation-rules)上看到所有可用验证规则的更新列表。

有时，我们需要验证特定的 MIME 类型，例如`JPEG、BMP、ORG 和 PNG`。您可以轻松地设置此类验证的验证规则，如下所示：

```php
'image' =>'required|mimes:jpeg,bmp,png'
```

然后我们使用以下代码检查验证过程：

```php
if($validator->fails()){

  return Redirect::route('add_image',array('id' =>Input::get('album_id')))
  ->withErrors($validator)
  ->withInput();
}
```

如果验证失败，我们将浏览器重定向到图片上传表单。然后，我们使用以下代码在模板文件中显示规则：

```php
@if($errors->has())
  <div class="alert alert-block alert-error fade in"id="error-block">
    <?php
    $messages = $errors->all('<li>:message</li>');
    ?>
    <button type="button" class="close"data-dismiss="alert">×</button>

    <h4>Warning!</h4>
    <ul>
      @foreach($messages as $message)
        {{$message}}
      @endforeach

    </ul>
  </div>
@endif
```

## 将照片分配给相册

`postAdd()`函数用于处理请求，在数据库中创建新的图片记录。我们使用以下先前提到的方法获取作者的 ID：

```php
Auth::user()->id
```

使用以下方法，我们将当前用户与博客文章进行关联。我们在查询中有一个新的方法，如下所示：

```php
Posts::with('Author')->…
```

我们在相册模型中定义了一个`public Photos()`函数，使用以下代码：

```php
public function Photos(){

  return $this->hasMany('images','album_id');
}
```

`hasMany()`方法是一个用于创建表之间关系的 Eloquent 函数。基本上，该函数有一个`required`变量和一个可选变量。第一个变量（`required`）用于定义目标模型。第二个可选变量用于定义当前模型表的源列。在这种情况下，我们将相册的 ID 存储在`images`表的`album_id`列中。因此，我们需要在函数中将第二个变量定义为`album_id`。如果您的 ID 不遵循约定，则第二个参数是必需的。使用这种方法，我们可以同时将相册信息和分配的图片数据传递给模板。

正如您在第四章*构建个人博客*中所记得的，我们可以在`foreach`循环中列出关系数据。让我们快速查看一下我们模板文件中的图像列表部分的代码，该文件位于`app/views/album.blade.php`中：

```php
@foreach($album->Photos as $photo)

  <div class="col-lg-3">
    <div class="thumbnail" style="max-height: 350px;min-height: 350px;">
    <img alt="{{$album->name}}" src="/albums/{{$photo->image}}">
      <div class="caption">
        <p>{{$photo->description}}</p>
        <p><p>Created date:  {{ date("d F Y",strtotime($photo->created_at)) }} at {{ date("g:ha",strtotime($photo->created_at)) }}</p></p>
        <a href="{{URL::route('delete_image',array('id'=>$photo->id))}}" onclick="return confirm('Are yousure?')"><button type="button" class="btnbtn-danger btn-small">Delete Image</button></a>
      </div>
    </div>
  </div>

@endforeach
```

# 在相册之间移动照片

在相册之间移动照片是管理相册图像的一个很好的功能。许多相册系统都具有此功能。因此，我们可以使用 Laravel 轻松编写它。我们需要一个表单和控制器函数来将此功能添加到我们的相册系统中。让我们首先编写控制器函数。打开位于`app/controllers/`中的`ImagesController.php`文件，并在其中添加以下代码：

```php
public function postMove()
{
  $rules = array(

    'new_album' => 'required|numeric|exists:albums,id',
    'photo'=>'required|numeric|exists:images,id'

  );

  $validator = Validator::make(Input::all(), $rules);
  if($validator->fails()){

    return Redirect::route('index');
  }
  $image = Image::find(Input::get('photo'));
  $image->album_id = Input::get('new_album');
  $image->save();
  return Redirect::route('show_album',array('id'=>Input::get('new_album')));
}
```

如您在前面的代码中所看到的，我们再次使用`Validation`类。让我们检查规则。第一个规则如下：

```php
'new_album' => 'required|numeric|exists:albums,id'
```

前面的规则意味着`new_album`字段是`required`（必须在表单中发布），必须是数值，并且存在于`albums`表的`id`列中。我们想要将图片分配给相册，所以图片必须存在。第二条规则如下：

```php
'photo'=>'required|numeric|exists:images,id'
```

前面的规则意味着`photo`字段是`required`（必须在表单中发布），必须是数值，并且存在于`images`表的`id`列中。

成功验证后，我们会更新`photos`字段的`album_id`列，并使用以下代码将浏览器重定向到显示新相册照片的页面：

```php
$image = Image::find(Input::get('photo'));
$image->album_id = Input::get('new_album');
$image->save();
return Redirect::route('show_album',array('id'=>Input::get('new_album')));
```

`Images`控制器的最终代码应如下所示：

```php
<?php

class ImagesController extends BaseController{

  public function getForm($id)
  {
    $album = Album::find($id);

    return View::make('addimage')
    ->with('album',$album);
  }

  public function postAdd()
  {
    $rules = array(

      'album_id' => 'required|numeric|exists:albums,id',
      'image'=>'required|image'

    );

    $validator = Validator::make(Input::all(), $rules);
    if($validator->fails()){

      return Redirect::route('add_image',array('id' =>Input::get('album_id')))
      ->withErrors($validator)
      ->withInput();
    }

    $file = Input::file('image');
    $random_name = str_random(8);
    $destinationPath = 'albums/';
    $extension = $file->getClientOriginalExtension();
    $filename=$random_name.'_album_image.'.$extension;
    $uploadSuccess = Input::file('image')->move($destinationPath, $filename);
    Image::create(array(
      'description' => Input::get('description'),
      'image' => $filename,
      'album_id'=> Input::get('album_id')
    ));

    return Redirect::route('show_album',array('id'=>Input::get('album_id')));
  }
  public function getDelete($id)
  {
    $image = Image::find($id);

    $image->delete();

    return Redirect::route('show_album',array('id'=>$image->album_id));
  }
  public function postMove()
  {
    $rules = array(
      'new_album' => 'required|numeric|exists:albums,id',
      'photo'=>'required|numeric|exists:images,id'
    );
    $validator = Validator::make(Input::all(), $rules);
    if($validator->fails()){

      return Redirect::route('index');
    }
    $image = Image::find(Input::get('photo'));
    $image->album_id = Input::get('new_album');
    $image->save();
    return Redirect::route('show_album',array('id'=>Input::get('new_album')));
  }
}
```

我们的控制器已经准备好了，所以我们需要在`app/routes.php`中设置更新后的表单路由。打开文件并添加以下代码：

```php
Route::post('/moveimage', array('as' => 'move_image', 'uses' => 'ImagesController@postMove'));
```

`app/routes.php`中的最终代码应如下所示：

```php
<?php
Route::get('/', array('as' => 'index', 'uses' =>
  'AlbumsController@getList'));
Route::get('/createalbum', array('as' => 'create_album_form',
  'uses' => 'AlbumsController@getForm'));
Route::post('/createalbum', array('as' => 'create_album',
  'uses' => 'AlbumsController@postCreate'));
Route::get('/deletealbum/{id}', array('as' => 'delete_album',
  'uses' => 'AlbumsController@getDelete'));
Route::get('/album/{id}', array('as' => 'show_album', 'uses' =>
  'AlbumsController@getAlbum'));
Route::get('/addimage/{id}', array('as' => 'add_image', 'uses' =>
  'ImagesController@getForm'));
Route::post('/addimage', array('as' => 'add_image_to_album',
  'uses' => 'ImagesController@postAdd'));
Route::get('/deleteimage/{id}', array('as' => 'delete_image',
'uses' => 'ImagesController@getDelete'));
Route::post('/moveimage', array('as' => 'move_image',
'uses' => 'ImagesController@postMove'));
```

## 创建更新表单

现在我们需要在模板文件中创建更新表单。打开位于`app/views/album.blade.php`中的模板文件，并将`foreach`循环更改如下：

```php
@foreach($album->Photos as $photo)
  <div class="col-lg-3">
    <div class="thumbnail" style="max-height: 350px;min-height: 350px;">
      <img alt="{{$album->name}}" src="/albums/{{$photo->image}}">
      <div class="caption">
        <p>{{$photo->description}}</p>
        <p>Created date:  {{ date("d F Y",strtotime($photo->created_at)) }}at {{ date("g:ha",strtotime($photo->created_at)) }}</p>
        <a href="{{URL::route('delete_image',array('id'=>$photo->id))}}" onclick="returnconfirm('Are you sure?')"><button type="button"class="btn btn-danger btn-small">Delete Image</button></a>
        <p>Move image to another Album :</p>
        <form name="movephoto" method="POST"action="{{URL::route('move_image')}}">
          <select name="new_album">
            @foreach($albums as $others)
              <option value="{{$others->id}}">{{$others->name}}</option>
            @endforeach
          </select>
          <input type="hidden" name="photo"value="{{$photo->id}}" />
          <button type="submit" class="btn btn-smallbtn-info" onclick="return confirm('Are you sure?')">Move Image</button>
        </form>
      </div>
    </div>
  </div>
@endforeach
```

# 摘要

在本章中，我们使用 Laravel 的内置函数和 Eloquent 数据库驱动创建了一个简单的相册系统。我们学会了如何验证数据，以及 Eloquent 中强大的数据关联方法 hasMany。在接下来的章节中，我们将学习如何处理更复杂的表格和关联数据以及关联类型。
