# 第九章：构建 RESTful API - 电影和演员数据库

设计和开发成功的 RESTful API 通常非常困难。设计和编写成功的 RESTful API 有许多方面；例如，保护和限制 API。在本章中，我们将专注于使用 Laravel 编写一个简单的电影和演员 API 的 REST 的基础知识。我们将在一个基本的认证系统后面创建一些 JSON 端点，并学习一些 Laravel 4 的技巧。本章将涵盖以下主题：

+   创建和迁移用户数据库

+   配置用户模型

+   添加样本用户

+   创建和迁移电影数据库

+   创建一个电影模型

+   添加样本电影

+   创建和迁移演员数据库

+   创建一个演员模型

+   将演员分配给电影

+   理解认证机制

+   查询 API

# 创建和迁移用户数据库

我们假设您已经在`app/config/`目录下的`database.php`文件中定义了数据库凭据。对于这个应用程序，我们需要一个数据库。您可以通过简单运行以下 SQL 命令来创建一个新的数据库，或者您可以使用您的数据库管理界面，比如 phpMyAdmin：

```php
**CREATE DATABASE laravel_api**

```

成功创建应用程序的数据库后，首先我们需要为应用程序生成一个应用程序密钥。正如您从之前的章节中了解的那样，这对于我们应用程序的安全和认证类是必要的。要做到这一点，首先打开您的终端，导航到您的项目文件夹，并运行以下命令：

```php
**php artisian key:generate**

```

如果没有错误发生，我们应该编辑认证类的配置文件。为了使用 Laravel 内置的认证类，我们需要编辑配置文件`auth.php`，该文件位于`app/config/`。该文件包含了认证设施的几个选项。如果您需要更改表名等，您可以在`auth.php`文件中进行更改。默认情况下，Laravel 带有一个用户模型；您可以看到位于`app/models/`的`User.php`文件。使用 Laravel 4，我们需要定义哪些字段可以在我们的`User`模型中填充。让我们编辑`app/models/User.php`并添加"fillable"数组：

```php
<?php

use Illuminate\Auth\UserInterface;
use Illuminate\Auth\Reminders\RemindableInterface;

class User extends Eloquent implements UserInterface, RemindableInterface {

  /**
   * The database table used by the model.
   *
   * @var string
   */
  protected $table = 'users';

  /**
   * The attributes excluded from the model's JSON form.
   *
   * @var array
   */
  protected $hidden = array('password');

  // Specify which attributes should be mass-assignable
   protected $fillable = array('email', 'password');

  /**
   * Get the unique identifier for the user.
   *
   * @return mixed
   */
  public function getAuthIdentifier()
  {
    return $this->getKey();
  }

  /**
   * Get the password for the user.
   *
   * @return string
   */
  public function getAuthPassword()
  {
    return $this->password;
  }

  /**
   * Get the e-mail address where password reminders are sent.
   *
   * @return string
   */
  public function getReminderEmail()
  {
    return $this->email;
  }

}
```

基本上，我们的 RESTful API 用户需要两列，它们是：

+   `email`：此列存储作者的电子邮件 ID

+   `password`：此列用于存储作者的密码

现在我们需要几个迁移文件来创建`users`表并向我们的数据库添加作者。要创建一个迁移文件，可以使用以下命令：

```php
**php artisan migrate:make create_users_table --table=users --create**

```

打开最近创建的位于`app/database/migrations/`的迁移文件。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('users', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('email');
      $table->string('password');
      $table->timestamps();
    });
  }
```

编辑迁移文件后，请运行`migrate`命令：

```php
**php artisian migrate**

```

如您所知，该命令将创建`users`表及其列。如果没有错误发生，请检查`laravel_api`数据库中的`users`表和列。

## 添加样本用户

现在我们需要为向数据库添加一些 API 用户创建一个新的迁移文件：

```php
**php artisan migrate:make add_some_users**

```

打开迁移文件并编辑`up()`函数如下：

```php
public function up()
  {
    User::create(array(
            'email' => 'john@gmail.com',
            'password' => Hash::make('johnspassword'),
          ));
    User::create(array(
            'email' => 'andrea@gmail.com',
            'password' => Hash::make('andreaspassword'),
          ));
  }
```

现在我们有了两个 API 用户用于我们的应用程序。这些用户将可以查询我们的 RESTful API。

# 创建和迁移电影数据库

对于一个简单的电影和演员应用程序，基本上我们需要两个用于存储数据的表。其中一个是`movies`表。该表将包含电影的名称和发行年份。

我们需要一个迁移文件来创建我们的`movies`表和其列。我们将再次使用`artisan`工具。打开您的终端，导航到您的项目文件夹，并运行以下命令：

```php
php artisan migrate:make create_movies_table --table=movies --create
```

打开最近创建的位于`app/database/migrations/`的迁移文件。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('movies', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('name');
      $table->integer('release_year');
      $table->timestamps();
    });
  }
```

编辑迁移文件后，运行`migrate`命令：

```php
**php artisian migrate**

```

# 创建一个电影模型

正如您所知，对于 Laravel 上的任何与数据库操作相关的事情，使用模型是最佳实践。我们将利用**Eloquent ORM**的好处。

将以下代码保存在`app/models/`下的`Movie.php`文件中：

```php
<?php
class Movie extends Eloquent {

protected $table = 'movies';

protected $fillable = array('name','release_year');

public function Actors(){

      return $this-> belongsToMany('Actor' , 'pivot_table');
}

}
```

我们使用受保护的`$table`变量设置了数据库表名。此外，我们使用了可编辑列的`$fillable`变量，以及时间戳的`$timestamps`变量，正如我们在前几章中已经看到和使用的那样。在模型中定义的变量足以使用 Laravel 的 Eloquent ORM。我们将在本章的*将演员分配给电影*部分中介绍公共`Actor()`函数。

我们的电影模型已经准备好了：现在我们需要一个演员模型及其相应的表。

## 添加示例电影

现在我们需要为向数据库添加一些电影创建一个新的迁移文件。实际上，您也可以使用数据库种子程序来种子数据库。在这里，我们将使用迁移文件来种子数据库。您可以在以下位置查看种子程序：

[`laravel.com/docs/migrations#database-seeding`](http://laravel.com/docs/migrations#database-seeding)

运行以下`migrate`命令：

```php
**php artisan migrate:make add_some_movies**

```

打开迁移文件并编辑`up()`函数如下：

```php
  public function up()
  {
    Movie::create(array(
            'name' => 'Annie Hall',
        'release_year' => '1977'
          ));

    Movie::create(array(
            'name' => ' Manhattan ',
        'release_year' => '1978'
          ));

Movie::create(array(
            'name' => 'The Shining',
        'release_year' => '1980'
          ));
  }
```

# 创建和迁移演员数据库

我们需要创建一个包含电影演员姓名的`actors`表。我们需要一个迁移文件来创建我们的`movies`表和列。我们将使用`artisan`工具再次进行操作。让我们打开终端，导航到我们的项目文件夹，并运行以下命令：

```php
php artisan migrate:make create_actors_table --table=actors –create
```

打开最近创建并位于`app/database/migrations/`中的迁移文件。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('actors', function(Blueprint $table)
    {
      $table->increments('id');
      $table->string('name');
      $table->timestamps();
    });
  }
```

编辑迁移文件后，运行以下`migrate`命令：

```php
php artisian migrate
```

# 创建演员模型

要创建演员模型，请将以下代码保存为`Movies.php`，并放在`app/models/`下：

```php
<?php
class Actor extends Eloquent {

protected $table = 'actors';

protected $fillable = array('name');

public function Movies(){

      return $this-> belongsToMany('Movies', 'pivot_table');
}

}
```

# 将演员分配给电影

正如您所知，我们在演员和电影模型之间使用了`belongsToMany`关系。这是因为一个演员可能出演了许多电影。一部电影也可能有许多演员。

正如您在本章的前几节中所看到的，我们使用了一个名为`pivot_table`的中间表。我们也可以使用`artisan`工具创建中间表。让我们创建它：

```php
php artisan migrate:make create_pivot_table --table=pivot_table --create
```

打开最近创建并位于`app/database/migrations/`中的迁移文件。我们需要编辑`up()`函数如下：

```php
  public function up()
  {
    Schema::create('pivot_table', function(Blueprint $table)
    {
      $table->increments('id');
      $table->integer('movies_id');
      $table->integer('actors_id');
      $table->timestamps();
    });  
}
```

编辑迁移文件后，运行`migrate`命令：

```php
php artisian migrate
```

现在我们需要为向数据库添加一些演员创建一个新的迁移文件：

```php
php artisan migrate:make add_some_actors
```

打开迁移文件并编辑`up()`函数如下：

```php
  public function up()
  {
    $woody = Actor::create(array(
            'name' => 'Woody Allen'
          ));

    $woody->Movies()->attach(array('1','2'));

    $diane = Actor::create(array(
            'name' => 'Diane Keaton'
          ));

$diane->Movies()->attach(array('1','2'));

$jack = Actor::create(array(
            'name' => 'Jack Nicholson'
        ));

$jack->Movies()->attach(3);

}
```

让我们获取迁移文件。当我们将`users`附加到`movies`时，我们必须使用以下显示的电影 ID：

```php
$voody = Actor::create(array(
            'name' => 'Woody Allen'
          ));

$voody->Movies()->attach(array('1','2'));
```

这意味着*Woody Allen*在两部电影中扮演了角色，这两部电影的 ID 分别为`1`和`2`。此外，*Diane Keaton*也在这两部电影中扮演了角色。但是*Jack Nicholson*在*The Shining*中扮演了角色，电影的 ID 为`3`。正如我们在第八章中已经详细阐述的那样，我们的关系类型是**Eloquent belongsToMany**关系。

# 理解认证机制

与许多其他 API 一样，我们的 API 系统是基于认证的。正如您可能还记得前几章所述，Laravel 带有认证机制。在本节中，我们将使用 Laravel 的基于模式的路由过滤功能来保护和限制我们的 API。首先，我们需要编辑我们应用程序的`auth.basic`过滤器。

打开位于`app/filters.php`的路由过滤器配置文件，并编辑`auth.basic`过滤器如下：

```php
Route::filter('auth.basic', function()
{
  return Auth::basic('email');
});
```

API 用户应该在其请求中发送他们的电子邮件 ID 和密码到我们的应用程序。由于请求，我们编辑了过滤器。API 请求将如下所示：

```php
**curl -i –user andrea@gmail.com:andreaspassword localhost/api/getactorinfo/Woody%20Allen**

```

现在，我们需要在我们的路由上应用一个过滤器。打开位于`app/routes.php`的路由过滤器配置文件，并添加以下代码：

```php
Route::when('*', 'auth.basic');
```

这段代码表明我们的应用程序需要对其上的每个请求进行身份验证。现在我们需要编写我们的路由。将以下行添加到`app/routes.php`：

```php
Route::get('api/getactorinfo/{actorname}', array('uses' => 'ActorController@getActorInfo'));
Route::get('api/getmovieinfo/{moviename}', array('uses' => 'MovieController@getMovieInfo'));
Route::put('api/addactor/{actorname}', array('uses' => 'ActorController@putActor'));
Route::put('api/addmovie/{moviename}/{movieyear}', array('uses' => 'MovieController@putMovie'));
Route::delete('api/deleteactor/{id}', array('uses' => 'ActorController@deleteActor'));
Route::delete('api/deletemovie/{id}', array('uses' => 'MovieController@deleteMovie'));
```

# 查询 API

我们需要两个控制器文件来处理我们的 RESTful 路由函数。让我们在`app/controllers/`下创建两个控制器文件。文件应命名为`MovieController.php`和`ActorController.php`。

## 从 API 获取电影/演员信息

首先，我们需要`getActorInfo()`和`getMovieInfo()`函数，以从数据库中获取演员和电影信息。打开位于`app/controllers/`的`ActorController.php`文件，并写入以下代码：

```php
<?php

class ActorController extends BaseController {
public function getActorInfo($actorname){

$actor = Actor::where('name', 'like', '%'.$actorname.'%')->first();
if($actor){

$actorInfo = array('error'=>false,'Actor Name'=>$actor->name,'Actor ID'=>$actor->id);
$actormovies = json_decode($actor->Movies);
foreach ($actormovies as $movie) {
$movielist[] = array("Movie Name"=>$movie->name, "Release Year"=>$movie->release_year);
}
$movielist =array('Movies'=>$movielist);
return Response::json(array_merge($actorInfo,$movielist));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We could not find any actor in database like :'.$actorname
));
}
}
}
```

接下来，打开位于`app/controllers/`的`MovieController.php`文件，并写入以下代码：

```php
<?php

class MovieController extends BaseController {

public function getMovieInfo($moviename){
$movie = Movie::where('name', 'like', '%'.$moviename.'%')->first();
if($movie){

$movieInfo = array('error'=>false,'Movie Name'=>$movie->name,'Release Year'=>$movie->release_year,'Movie ID'=>$movie->id);
$movieactors = json_decode($movie->Actors);
foreach ($movieactors as $actor) {
$actorlist[] = array("Actor"=>$actor->name);
}
$actorlist =array('Actors'=>$actorlist);
return Response::json(array_merge($movieInfo,$actorlist));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We could not find any movie in database like :'.$moviename
));
}
}
}
```

`getActorInfo()`和`getMovieInfo()`函数基本上是在数据库中搜索具有给定文本的电影/演员名称。如果找到这样的电影或演员，它将以 JSON 格式返回。因此，要从 API 中获取演员信息，我们的用户可以进行如下请求：

```php
**curl -i –-user andrea@gmail.com:andreaspassword localhost/api/getactorinfo/Woody**

```

演员信息请求的响应将如下所示：

```php
{
   "error":false,
   "Actor Name":"Woody Allen",
   "Actor ID":1,
   "Movies":[
      {
         "Movie Name":"AnnieHall",
         "Release Year":1977
      },
      {
         "Movie Name":"Manhattan",
         "Release Year":1978
      }
   ]
}
```

任何电影的请求将类似于这样：

```php
**curl -i --user andrea@gmail.com:andreaspassword localhost/api/getmovieinfo/Manhattan**

```

电影信息请求的响应将如下所示：

```php
{
   "error":false,
   "Movie Name":"Manhattan",
   "Release Year":1978,
   "Movie ID":2,
   "Actors":[
      {
         "Actor":"Woody Allen"
      },
      {
         "Actor":"Diane Keaton"
      }
   ]
}
```

如果任何用户从数据库中不存在的 API 请求电影信息，响应将如下所示：

```php
{
   "error":true,
   "description":"We could not find any movie in database like :Terminator"
}
```

对于数据库中不存在的演员，也将有类似的响应：

```php
{
   "error":true,
   "description":"We could not find any actor in database like :Al Pacino"
}
```

## 将新电影/演员发送到 API 的数据库

我们需要`putActor()`和`putMovie()`函数，以允许用户向我们的数据库添加新的演员/电影。

打开位于`app/controllers/`的`ActorController.php`文件，并添加以下函数：

```php
public function putActor($actorname)
{

$actor = Actor::where('name', '=', $actorname)->first();
if(!$actor){

$the_actor = Actor::create(array('name'=>$actorname));

return Response::json(array(
'error'=>false,
'description'=>'The actor successfully saved. The ID number of Actor is : '.$the_actor->id
));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We have already in database : '.$actorname.'. The ID number of Actor is : '.$actor->id
));
}
}
```

现在打开位于`app/controllers/`的`MovieController.php`文件，并添加以下函数：

```php
public function putMovie($moviename,$movieyear)
{

$movie = Movie::where('name', '=', $moviename)->first();
if(!$movie){

$the_movie = Movie::create(array('name'=>$moviename,'release_year'=>$movieyear));

return Response::json(array(
'error'=>false,
'description'=>'The movie successfully saved. The ID number of Movie is : '.$the_movie->id
));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We have already in database : '.$moviename.'. The ID number of Movie is : '.$movie->id
));
}
}
```

`putActor()`和`putMovie()`函数基本上是在数据库中搜索具有给定文本的电影/演员名称。如果找到电影或演员，函数将以 JSON 格式返回其 ID，否则将创建新的演员/电影并以新记录 ID 响应。因此，要在 API 数据库中创建新的演员，我们的用户可以进行如下请求：

```php
**curl –i –X PUT –-user andrea@gmail.com:andreaspassword localhost/api/addactor/Al%20Pacino**

```

电影信息请求的响应将如下所示：

```php
{
   "error":false,
   "description":"The actor successfully saved. The ID number of Actor is : 4"
}
```

如果任何 API 用户尝试添加已存在的演员，API 将如下响应：

```php
{
   "error":true,
   "description":"We have already in database : Al Pacino. The ID number of Actor is : 4"
}
```

此外，API 数据库中创建新电影的响应应该如下所示：

```php
curl -i –X PUT –-user andrea@gmail.com:andreaspassword localhost/api/addmovie/The%20Terminator/1984
```

请求的响应将如下所示：

```php
{
   "error":false,
   "description":"The movie successfully saved. The ID number of Movie is : 4"
}
```

如果任何 API 用户尝试添加已存在的演员，API 将如下响应：

```php
{
   "error":true,
   "description":"We have already in database : The Terminator. The ID number of Movie is : 4"
}
```

## 从 API 中删除电影/演员

现在我们需要`deleteActor()`和`deleteMovie()`函数，以允许用户向我们的数据库添加新的演员/电影。

打开`app/controllers/`下的`ActorController.php`文件，并添加以下函数：

```php
public function deleteActor($id)
{

$actor = Actor::find($id);
if($actor){

$actor->delete();

return Response::json(array(
'error'=>false,
'description'=>'The actor successfully deleted : '.$actor->name
));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We could not find any actor in database with ID number :'.$id
));
}
}
```

添加函数后，位于`app/controllers/`的`ActorController.php`中的内容应如下所示：

```php
<?php
class ActorController extends BaseController
{
    public function getActorInfo($actorname)
    {
        $actor = Actor::where('name', 'like', '%' . $actorname . '%')->first();
        if ($actor)
        {
            $actorInfo   = array(
                'error' => false,
                'Actor Name' => $actor->name,
                'Actor ID' => $actor->id
            );
            $actormovies = json_decode($actor->Movies);
            foreach ($actormovies as $movie)
            {
                $movielist[] = array(
                    "Movie Name" => $movie->name,
                    "Release Year" => $movie->release_year
                );
            }
            $movielist = array(
                'Movies' => $movielist
            );
            return Response::json(array_merge($actorInfo, $movielist));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We could not find any actor in database like :' . $actorname
            ));
        }
    }
    public function putActor($actorname)
    {
        $actor = Actor::where('name', '=', $actorname)->first();
        if (!$actor)
        {
            $the_actor = Actor::create(array(
                'name' => $actorname
            ));
            return Response::json(array(
                'error' => false,
                'description' => 'The actor successfully saved. The ID number of Actor is : ' . $the_actor->id
            ));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We have already in database : ' . $actorname . '. The ID number of Actor is : ' . $actor->id
            ));
        }
    }
    public function deleteActor($id)
    {
        $actor = Actor::find($id);
        if ($actor)
        {
            $actor->delete();
            return Response::json(array(
                'error' => false,
                'description' => 'The actor successfully deleted : ' . $actor->name
            ));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We could not find any actor in database with ID number :' . $id
            ));
        }
    }
}
```

现在我们需要`MovieController`的类似函数。打开位于`app/controllers/`的`MovieController.php`文件，并添加以下函数：

```php
public function deleteMovie($id)
{

$movie = Movie::find($id);
if($movie){

$movie->delete();

return Response::json(array(
'error'=>false,
'description'=>'The movie successfully deleted : '.$movie->name
));

}
else{

return Response::json(array(
'error'=>true,
'description'=>'We could not find any movie in database with ID number :'.$id
));
}
}
```

添加函数后，位于`app/controllers/`的`ActorController.php`中的内容应如下所示：

```php
<?php
class  extends BaseController
{
    public function getMovieInfo($moviename)
    {
        $movie = Movie::where('name', 'like', '%' . $moviename . '%')->first();
        if ($movie)
        {
            $movieInfo   = array(
                'error' => false,
                'Movie Name' => $movie->name,
                'Release Year' => $movie->release_year,
                'Movie ID' => $movie->id
            );
            $movieactors = json_decode($movie->Actors);
            foreach ($movieactors as $actor)
            {
                $actorlist[] = array(
                    "Actor" => $actor->name
                );
            }
            $actorlist = array(
                'Actors' => $actorlist
            );
            return Response::json(array_merge($movieInfo, $actorlist));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We could not find any movie in database like :' . $moviename
            ));
        }
    }
    public function putMovie($moviename, $movieyear)
    {
        $movie = Movie::where('name', '=', $moviename)->first();
        if (!$movie)
        {
            $the_movie = Movie::create(array(
                'name' => $moviename,
                'release_year' => $movieyear
            ));
            return Response::json(array(
                'error' => false,
                'description' => 'The movie successfully saved. The ID number of Movie is : ' . $the_movie->id
            ));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We have already in database : ' . $moviename . '. The ID number of Movie is : ' . $movie->id
            ));
        }
    }
    public function deleteMovie($id)
    {
        $movie = Movie::find($id);
        if ($movie)
        {
            $movie->delete();
            return Response::json(array(
                'error' => false,
                'description' => 'The movie successfully deleted : ' . $movie->name
            ));
        }
        else
        {
            return Response::json(array(
                'error' => true,
                'description' => 'We could not find any movie in database with ID number :' . $id
            ));
        }
    }
}
```

`deleteActor()`和`deleteMovie()`函数基本上是在数据库中搜索具有给定 ID 的电影/演员。如果有电影或演员，API 将删除演员/电影并以 JSON 格式返回状态。因此，要从 API 中删除演员，我们的用户可以进行如下请求：

```php
**curl –I –X DELETE –-user andrea@gmail.com:andreaspassword localhost/api/deleteactor/4**

```

请求的响应将如下所示：

```php
{
   "error":false,
   "description":"The actor successfully deleted : Al Pacino"
}
```

此外，从 API 数据库中删除电影的响应应如下所示：

```php
**curl –I –X DELETE –-user andrea@gmail.com:andreaspassword localhost/api/deletemovie/4**

```

请求的响应将如下所示：

```php
{
   "error":false,
   "description":"The movie successfully deleted : The Terminator"
}
```

如果任何 API 用户尝试从 API 数据库中删除不存在的电影/演员，API 将如下响应：

```php
{
   "error":true,
   "description":"We could not find any movie in database with ID number :17"
}
```

删除不存在的演员，响应将如下所示：

```php
{
   "error":true,
   "description":"We could not find any actor in database with ID number :58"
}
```

# 摘要

在本章中，我们专注于使用 Laravel 编写简单的电影和演员 API 的 REST 基础知识。我们在基本身份验证系统后面创建了一些 JSON 端点，并在本章中学习了一些 Laravel 4 的技巧，比如类似模式的路由过滤。正如你所看到的，使用 Laravel 开发和保护 RESTful 应用程序非常容易。在下一章中，我们将在编写简单的电子商务应用程序时涵盖更有效的 Laravel 方法。
