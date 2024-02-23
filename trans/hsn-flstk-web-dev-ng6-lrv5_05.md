# 使用 Laravel 创建 RESTful API - 第 1 部分

在开始之前，让我们简要介绍一种称为 RESTful API 的软件开发标准。

应用程序编程接口（API）是一组用于访问基于互联网的应用程序的指令、例程和编程模式。这允许计算机或其他应用程序理解此应用程序中的指令，解释其数据，并将其用于与其他平台和软件集成，生成将由此软件或计算机执行的新指令。

通过这种方式，我们了解到 API 允许应用程序之间的互操作性。换句话说，这是客户端和服务器端之间的通信。

表述状态转移（REST）是 Web 架构的一种抽象。简而言之，REST 由原则、规则和约束组成，遵循这些原则、规则和约束可以创建一个具有明确定义接口的项目。

RESTful 服务中可用的功能可以从一组默认预定义的操作中访问或操作。这些操作使得可以使用 HTTP 协议从消息中创建（PUT）、读取（GET）、更改（POST）和删除（DELETE）资源。

尽管 Laravel 是一个 MVC 框架，但我们可以构建非常强大和可扩展的 RESTful 应用程序。

在本章中，您将学习如何使用 Laravel 框架的核心元素构建 RESTful API，例如控制器、路由和 Eloquent 对象关系映射（ORM）。主要，我们将涵盖以下主题：

+   准备应用程序并了解我们正在构建的内容

+   Eloquent ORM 关系

+   控制器和路由

# 准备应用程序并了解我们正在构建的内容

让我们使用在上一章中开始开发的应用程序来开始本节课。但是，在继续之前，我们将进行一些调整。首先，我们将把我们的代码添加到版本控制中。这样，我们就不会丢失在上一章中所做的进展。

1.  在`chapter-04`文件夹中，创建一个名为`.gitignore`的新文件，并添加以下代码：

```php
storage-db
.DS_Store
```

+   有关忽略文件的更多信息，请参阅[`help.github.com/articles/ignoring-files`](https://help.github.com/articles/ignoring-files)

+   如果您发现自己正在忽略文本编辑器或操作系统生成的临时文件，您可能希望添加一个全局忽略，而不是`git config --global core.excludesfile '~/.gitignore_global'`

+   忽略`storage`文件夹的大小

先前的代码只是将`storage-db`文件夹添加到未跟踪的文件中。

1.  让我们把更改添加到源代码控制中。在终端窗口内，输入以下命令：

```php
git init
```

最后，让我们添加我们的第一个提交。

1.  在终端内，输入以下命令：

```php
git add .
git commit -m "first commit"
```

太棒了！我们的代码已经在 Git 源代码控制下了。

# 重构应用程序文件

现在是时候更改一些文件以适应`chapter-05`了：

1.  复制`chapter-04`的所有内容，并将其粘贴到一个名为`chapter-05`的新文件夹中。

1.  打开`docker-compose.yml`文件，并用以下行替换代码：

```php
version: "3.1"
services:
 mysql:
 screenshot: mysql:5.7
 container_name: chapter-05-mysql
 working_dir: /application
 volumes:
 - .:/application
 - ./storage-db:/var/lib/mysql
 environment:
 - MYSQL_ROOT_PASSWORD=123456
 - MYSQL_DATABASE=chapter-05
 - MYSQL_USER=chapter-05
 - MYSQL_PASSWORD=123456
 ports:
 - "8083:3306"
 webserver:
 screenshot: nginx:alpine
 container_name: chapter-05-webserver
 working_dir: /application
 volumes:
 - .:/application
 - ./phpdocker/nginx/nginx.conf:
 /etc/nginx/conf.d/default.conf
 ports:
 - "8081:80"
 php-fpm:
 build: phpdocker/php-fpm
 container_name: chapter-05-php-fpm
 working_dir: /application
 volumes:
 - ./project:/application
 - ./phpdocker/php-fpm/php-ini-
 overrides.ini:/etc/php/7.2/fpm/conf.d/99-overrides.ini
```

请注意，我们更改了`MYSQL_DATABASE`和`MYSQL_USER`，还将容器名称更改为符合`chapter-05`标题。

1.  使用新的数据库信息编辑`project/.env`文件，如下所示：

```php
DB_CONNECTION=mysql
 DB_HOST=mysql
 DB_PORT=3306
 DB_DATABASE=chapter-05
 DB_USERNAME=chapter-05
 DB_PASSWORD=123456
```

1.  现在，删除`storage-db`文件夹。不用担心，我们稍后会使用`docker-compose`命令创建一个新的文件夹。

1.  现在是时候提交我们的新更改了，但这次我们将以另一种方式进行。这次，我们将使用 Git Lens VS Code 插件。

1.  打开 VS Code。在左侧边栏中，单击源代码控制的第三个图标。

1.  在左侧边栏的消息框中添加以下消息`Init chapter 05`。

1.  在 macOSX 上按 Command+Enter，或在 Windows 上按 Ctrl+Enter，然后单击 Yes。

干得好。现在，我们可以用新的文件基线开始`第五章`。

# 我们正在构建什么

现在，让我们稍微谈谈自从本书开始以来我们一直在构建的应用程序。

正如我们所看到的，到目前为止我们已经构建了很多东西，但是我们仍然不清楚我们在项目中做了什么。这是学习和练习 Web 应用程序开发的最佳方式。

很多时候，当我们第一次学习或做某事时，我们倾向于密切关注最终项目，在这一点上，我们急于完成我们开始做的事情，无法专注于建设过程和细节。

在这里，我们已经有了*40%*的项目准备就绪。然后，我们可以透露更多关于我们正在做什么的细节。

请记住，到目前为止，我们已经使用 Docker 准备了一个高度可扩展的开发环境，在我们的开发中安装了一些非常重要的工具，并学会了如何启动一个稳固的 Laravel 应用程序。

该应用将被称为 Custom Bike Garage，这是一种针对自定义摩托车文化爱好者的 Instagram/Twitter。在开发结束时，我们将拥有一个与以下线框截图非常相似的 Web 应用程序：

![](img/afbdce08-53d1-432e-bdcf-a0edffdf6e20.png)主页

前面的截图只是一个基本的应用程序主页，带有导航链接和一个呼吁行动的按钮：

![](img/53677fd1-0c17-4b1e-b9b4-538c74d82a13.png)摩托车列表页面

# 应用摘要

正如我们在前面的截图中所看到的，我们的应用程序有：

+   一个主页，我们将称之为`主页`

+   一个摩托车页面，我们将称之为`摩托车列表`页面

+   一个摩托车详细页面，我们将称之为`摩托车详情`页面

+   一个构建者页面，我们将称之为`构建者列表`页面

+   一个构建者详细页面，我们将称之为`构建者详情`页面

+   一个注册页面，我们将称之为`注册页面`

+   一个登录页面，我们将称之为`登录页面`

+   一个评分页面，用户可以在摩托车上投票

想象我们正在为一场展览会构建一个自定义摩托车应用程序。每个会议都有一个名称和客户级别。

用户可以注册，对最好的摩托车进行投票，并插入他们自己的摩托车。会议展示了一些知名摩托车制造商定制的摩托车，每辆摩托车都有许多自定义项目。

因此，为了完成应用程序的后端，我们还需要做以下工作：

+   为`Builder`、`Item`、`Garage`和`Rating`创建模型

+   为`Builder`、`Item`、`Garage`和`Rating`创建迁移文件

+   种子数据库

+   为`Bike`、`Builder`、`Item`、`Garage`和`Rating`创建控制器

+   应用模型之间的关系

+   使用资源表示关系

+   创建基于令牌的身份验证

# 创建模型和迁移文件

让我们开始使用`-m`标志创建`builders`模型和迁移文件。就像我们在本书中之前做的那样，我们可以同时创建这两个文件：

1.  打开您的终端窗口，键入以下命令：

```php
php artisan make:model Builder -m
```

1.  仍然在您的终端窗口上，键入以下命令：

```php
php artisan make:model Item -m
```

1.  仍然在您的终端窗口上，键入以下命令：

```php
php artisan make:model Garage -m
```

1.  仍然在您的终端窗口上，键入以下命令：

```php
php artisan make:model Rating -m
```

*步骤 1*到*步骤 4*将在我们的应用程序中产生以下新文件：

```php
project/app/Builder.php project/database/migrations/XXXX_XX_XX_XXXXXX_create_builders_table.php project/app/Item.php project/database/migrations/XXXX_XX_XX_XXXXXX_create_items_table.php project/app/Garage.php project/database/migrations/XXXX_XX_XX_XXXXXX_create_garages_table.php project/app/Rating.php project/database/migrations/XXXX_XX_XX_XXXXXX_create_ratings_table.php
```

注意迁移文件名之前的`XXXX_XX_XX_XXXXXX`。这是文件创建时的时间戳。

在这一点上，我们可以在 VS Code 左侧面板上看到之前的六个模型，就像以下截图中一样：

![](img/b155ca8e-c58d-4f7e-9d82-2d57457714ad.png)左侧面板

请注意，我们已经在第四章中创建了`Bike`模型，并且默认情况下，Laravel 为我们创建了`User`模型。

1.  现在，就像我们之前做的那样，让我们提交新创建的文件，并单击 VS Code 左侧面板上的`源控制图标`。

1.  在消息输入字段中键入以下文本：`添加模型和迁移文件`。

1.  在 macOSX 上按*C**ommand* *+* *Enter*，或在 Windows 上按*C**trl* *+* *Enter*，然后单击*Yes*按钮。

# 向迁移文件添加内容

现在，让我们创建我们迁移文件的内容。请记住，迁移文件是使用 Laravel 创建数据库方案的最简单和最快的方法：

1.  打开`project/database/migrations/XXXX_XX_XX_XXXXXX_create_builders_table.php`并用以下代码替换内容：

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateBuildersTable extends Migration
{
    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {
    Schema::create('builders', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->text('description');
        $table->string('location');
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
        Schema::dropIfExists('builders');
    }
}
```

1.  打开`project/database/migrations/XXXX_XX_XX_XXXXXX_create_items_table.php`并用以下代码替换内容：

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateItemsTable extends Migration
{
    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {
        Schema::create('items', function (Blueprint $table) {
        $table->increments('id');
        $table->string('type');
        $table->string('name');
        $table->text('company');
        $table->unsignedInteger('bike_id');
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
        Schema::dropIfExists('items');
    }
}
```

1.  注意 Bike 表的`$table->unsignedInteger('bike_id')`外键。在本章的后面，我们将深入研究模型关系/关联，但现在让我们专注于迁移文件。

1.  打开`project/database/migrations/XXXX_XX_XX_XXXXXX_create_garages_table.php`并用以下代码替换内容：

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateGaragesTable extends Migration
{
    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {
        Schema::create('garages', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->integer('customer_level');
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
        Schema::dropIfExists('garages');
    }
}
```

现在，我们需要另一个表，只是为了建立`Bike`和`Garage`之间的关系。我们使用`artisan`命令创建迁移文件，因为对于这种关系，我们不需要模型。这个表也被称为`pivot`表。

1.  打开您的终端窗口并输入以下命令：

```php
php artisan make:migration create_bike_garage_table
```

1.  打开`project/database/migrations/XXXX_XX_XX_XXXXXX_create_bike_garage_table.php`并用以下代码替换内容：

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateBikeGarageTable extends Migration
{
    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {
        Schema::create('bike_garage', function (Blueprint $table) {
        $table->increments('id');
        $table->integer('bike_id');
        $table->integer('garage_id');
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
        Schema::dropIfExists('bike_garage');
    }
}
```

请记住，自行车迁移文件是在上一章中创建的。

1.  打开您的终端窗口并输入以下命令：

```php
php artisan make:migration create_ratings_table
```

1.  打开`project/database/migrations/XXXX_XX_XX_XXXXXX_create_ratings_table.php`并用以下代码替换内容：

```php
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateRatingsTable extends Migration
{
    /**
    * Run the migrations.
    *
    * @return void
    */
    public function up()
    {
        Schema::create('ratings', function (Blueprint $table) {
        $table->increments('id');
        $table->unsignedInteger('user_id');
        $table->unsignedInteger('bike_id');
        $table->unsignedInteger('rating');
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
        Schema::dropIfExists('ratings');
    }
}
```

好了，现在是时候更深入地了解我们在本节中所做的事情了，所以让我们进入下一节，了解`Eloquent`是如何工作的。

# Eloquent ORM 关系

Eloquent 是 Laravel 数据库查询背后的 ORM。它是活动记录实现的抽象。

正如我们之前看到的，每个应用程序模型在我们的数据库中都有一个相应的表。有了这个，我们可以查询、插入、删除和更新记录。

Eloquent ORM 使用类的蛇形复数名称作为表名，除非另一个名称被明确指定。例如，我们的`Bike`模型类有自己的表`bikes`。

应用程序模型有以下表：

| 应用程序模型 | 数据库表 |
| --- | --- |
| `Bike.php` | bikes |
| `Builder.php` | builders |
| `Garage.php` | garages |
| `Item.php` | items |
| `Rating.php` | ratings |
| `Builder.php` | builders |
| `User.php` | users |

请注意，我们保留表约定名称，但也可以使用自定义表名。在本书的范围内，我们将保留 Laravel 生成的表名。

您可以在官方 Laravel 文档的[`laravel.com/docs/5.6/eloquent#defining-models`](https://laravel.com/docs/5.6/eloquent#defining-models)上阅读更多关于表名和模型约定的信息。

Eloquent ORM 支持模型之间的以下关系：

+   一对一

+   一对多

+   属于（反向=一对多）

+   多对多

+   有多个

+   多态关系

+   多对多多态关系

我们将详细介绍前四种关系；然而，我们无法在我们的书中详细介绍所有关系。在许多框架中，理解关系（也称为关联）是非常简单的。

您可以在[`laravel.com/docs/5.6/eloquent-relationships`](https://laravel.com/docs/5.6/eloquent-relationships)上阅读更多关于关系的信息。

# 一对一关系

让我们建立`Builder`和`Bike`之间的一对一关系。这意味着`Bike`将只有一个`Builder`。

1.  打开`project/app/Builder.php`并用以下代码替换内容：

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
/**
* @SWG\Definition(
* definition="Builder",
* required={"name", "description", "location"},
* @SWG\Property(
* property="name",
* type="string",
* description="Builder name",
* example="Jesse James"
* ),
* @SWG\Property(
* property="description",
* type="string",
* description="Famous Motorcycle builder from Texas",
* example="Austin Speed Shop"
* ),
* @SWG\Property(
* property="location",
* type="string",
* description="Texas/USA",
* example="Austin, Texas"
* ),
* )
*/
class Builder extends Model
{
    /**
    * The table associated with the model.
    *
    * @var string
    */
    protected $table = 'builders';
    /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
    protected $fillable = [
        'name',
        'description',
        'location'
    ];
    /**
    * Relationship.
    *
    * @var array
    */
    public function bike() {
        return $this->hasOne('App\Bike');
    }
}
```

请注意，我们像在上一章中那样添加了 Swagger 文档定义。`bike()`函数创建了一对一关系。您可以在关系函数上使用任何名称，但我们强烈建议您使用相同的模型名称，在我们的案例中是`Bike`模型类。

1.  现在，让我们为`Bike`模型添加相应的关系。打开`project/app/Bike.php`并在`protected fillable`函数之后立即添加以下代码：

```php
/**
* Relationship.
*
* @var string
*/
public function builder() {
     return $this->belongsTo('App\Builder');
}
```

注意，`belongsTo`关系是一对多的反向关系。

1.  打开`project/app/Item.php`并用以下代码替换内容：

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
/**
* @SWG\Definition(
* definition="Item",
* required={"type", "name", "company"},
* @SWG\Property(
* property="type",
* type="string",
* description="Item Type",
* example="Exhaust"
* ),
* @SWG\Property(
* property="name",
* type="string",
* description="Item name",
* example="2 into 1 Exhaust"
* ),
* @SWG\Property(
* property="company",
* type="string",
* description="Produced by: some company",
* example="Vance and Hines"
* )
* )
*/
class Item extends Model
{
    /**
    * The table associated with the model.
    *
    * @var string
    */
    protected $table = 'items';
    /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
    protected $fillable = [
        'type',
        'name',
        'company',
        'bike_id'
    ];
    /**
    * Relationship.
    *
    * @var string
    */
    public function bike() {
        return $this->belongsTo('App\Bike');
    }
}
```

# 一对多关系

一对多关系将应用于`Bike`和`Items`之间，这意味着一个`bike`将有许多自定义`items`。

仍然在`project/app/Bike.app`文件中，让我们在`Item`和`Bike`模型之间添加一对多关系。

在`builder()`函数之后立即添加以下代码：

```php
public function items() {
    return $this->hasMany('App\Item');
}
```

# 多对多关系

对于多对多关系，我们将通过枢轴表在许多 Garages 中有许多 Bikes。

在多对多关系中，我们需要遵守一些命名规则。

枢轴表的名称应由两个表的单数名称组成，用下划线符号分隔，并且这些名称应按字母顺序排列。

默认情况下，应该只有两个枢轴表字段和每个表的外键，在我们的情况下是`bike_id`和`garage_id`。

仍然在`project/app/Bike.app`文件中，让我们在`Bike`和`Garage`模型之间添加多对多关系。

在`items()`函数之后立即添加以下代码：

```php
public function garages() {
    return $this->belongsToMany('App\Garage');
}
```

请注意，在上面的代码中，我们正在创建`Bike`和`Garage`之间的关系，这将在第三个表中，称为枢轴表中保存与关系相关的信息，正如我们之前解释的那样。

现在，是时候在用户和评分与自行车之间添加关系了。在`garages()`函数之后立即添加以下代码：

```php
public function user() {
        return $this->belongsTo('App\User');
public function ratings() {
        return $this->hasMany('App\Rating');
}
```

在这一点上，我们将在`Bike`模型中有以下关系：

```php
/**
* Relationship.
*
* @var string
*/
public function builder() {
    return $this->belongsTo('App\Builder');
}
public function items() {
    return $this->hasMany('App\Item');
}
public function garages() {
    return $this->belongsToMany('App\Garage');
}
public function user() {
    return $this->belongsTo(User::class);
}
public function ratings() {
    return $this->hasMany(Rating::class);
}
```

现在，让我们在`project/app/Garage.app`模型中添加关系。用以下代码替换其内容：

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
/**
* @SWG\Definition(
* definition="Garage",
* required={"name", "custumer_level"},
* @SWG\Property(
* property="name",
* type="string",
* description="Jhonny Garage",
* example="Exhaust"
* ),
* @SWG\Property(
* property="customer_level",
* type="integer",
* description="Whats the garage level",
* example="10"
* )
* )
*/
class Garage extends Model
{
    /**
    * The table associated with the model.
    *
    * @var string
    */
    protected $table = 'garages';
    /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
    protected $fillable = [
        'name',
        'costumer_level'
    ];
    /
    *
    * @var string
    */
    public function bikes() {
        return $this->belongsToMany('App\Bike', 'bike_garage',
        'bike_id', 'garage_id');
    }
}
* Relationship.
```

请注意，我们使用的是`belongsToMany()`而不是`hasMany()`。`hasMany()`用于一对多关系。

现在，让我们在`project/app/User.app`模型中添加关系。用以下代码替换其内容：

```php
<?php
namespace App;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
/**
* @SWG\Definition(
* definition="User",
* required={"name", "email", "password"},
* @SWG\Property(
* property="name",
* type="string",
* description="User name",
* example="John Conor"
* ),
* @SWG\Property(
* property="email",
* type="string",
* description="Email Address",
* example="john.conor@terminator.com"
* ),
* @SWG\Property(
* property="password",
* type="string",
* description="A very secure password",
* example="123456"
* ),
* )
*/class User extends Authenticatable
{
    use Notifiable;
    /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
    protected $fillable = [
        'name', 'email', 'password',
    ];
    /**
    * The attributes that should be hidden for arrays.
    *
    * @var array
    */
    protected $hidden = [
        'password', 'remember_token',];
    * Relationship.
    ** @var string
    /
    public function bikes()
    {
        return $this->hasMany(App\Bike);
    }
}
```

打开`project/app/Rating.app`模型，并用以下代码替换其内容：

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
/**
* @SWG\Definition(
* definition="Rating",
* required={"bike_id", "user_id", "rating"},
* @SWG\Property(
* property="biker_id",
* type="integer",
* description="Bike id",
* example="1"
* ),
* @SWG\Property(
* property="user_id",
* type="integer",
* description="User id",
* example="2"
* ),
* @SWG\Property(
* property="rating",
* type="integer",
* description="Vote by rating",
* example="10"
* )
* )
*/
class Rating extends Model
{
    /**
    * The attributes that are mass assignable.
    *
    * @var array
    */
    protected $fillable = [
        'bike_id',
        'user_id',
        'rating'
    ];
    /**
    * Relationship.
    *
    * @var string
    */
    public function bike() {
        return $this->belongsTo('App\Bike');
    }
}
```

现在我们已经准备好迁移文件和应用程序模型，我们可以创建种子文件来填充我们的数据库。但在继续之前，让我们将表迁移到数据库中。在您的终端窗口中，输入以下命令：

```php
php artisan migrate
```

干得好！我们已成功迁移了所有表，现在我们的数据库已经准备就绪。

如果在尝试使用`migrate`命令时遇到问题，请使用`refresh`参数：

```php
php artisan migrate:refresh
```

# 填充我们的数据库

请记住，在上一章中，我们已经创建了 Bike 种子，所以现在我们只需要创建另外三个种子，它们将是`Builders`，`Items`和`Garage`。

1.  打开您的终端窗口，输入以下命令：

```php
php artisan make:seeder BuildersTableSeeder
```

1.  将以下代码添加到`app/database/seeds/BuildersTableSeeder.php`的`run()`公共函数中：

```php
DB::table('builders')->delete();
$json = File::get("database/data-sample/builders.json");
$data = json_decode($json);
foreach ($data as $obj) {
    Builder::create(array(
        'id' => $obj->id,
        'name' => $obj->name,
        'description' => $obj->description,
        'location' => $obj->location
    ));
}
```

1.  仍然在您的终端窗口中，输入以下命令：

```php
php artisan make:seeder ItemsTableSeeder
```

1.  将以下代码添加到`app/database/seeds/ItemsTableSeeder.php`中：

```php
DB::table('items')->delete();
$json = File::get("database/data-sample/items.json");
$data = json_decode($json);
foreach ($data as $obj) {
    Item::create(array(
        'id' => $obj->id,
        'type' => $obj->type,
        'name' => $obj->name,
        'company' => $obj->company,
        'bike_id' => $obj->bike_id
    ));
}
```

1.  在您的终端窗口中，输入以下命令：

```php
php artisan make:seeder GaragesTableSeeder
```

1.  将以下代码添加到`app/database/seeds/GaragesTableSeeder.php`的`run()`公共函数中：

```php
DB::table('garages')->delete();
$json = File::get("database/data-sample/garages.json");
$data = json_decode($json);
foreach ($data as $obj) {
    Garage::create(array(
        'id' => $obj->id,
        'name' => $obj->name,
        'customer_level' => $obj->customer_level
    ));
}
```

1.  将以下代码添加到`app/database/seeds/UsersTableSeeder.php`文件夹的公共函数`run()`中：

```php
DB::table('users')->insert([
'name' => 'Johnny Cash',
'email' => 'johnny@cash.com',
'password' => bcrypt('123456')
]);
    DB::table('users')->insert([
        'name' => 'Frank Sinatra',
        'email' => 'frank@sinatra.com',
        'password' => bcrypt('123456')
    ]);
```

请注意，我们正在使用与上一章中相同的函数来加载示例数据。现在，是时候创建 JSON 文件了。

1.  在`project/database/data-sample/`中，创建一个名为`builders.json`的新文件，并添加以下代码：

```php
[{
    "id": 1,
    "name": "Diamond Atelier",
    "description": "Diamond Atelier was founded by two fellow riders
     who grew tired of the same played-out custom bike look and feel
     they and their friends had grown accustomed to witnessing.",
    "location": "Munich, Germany"
},{
    "id": 2,
    "name": "Deus Ex Machina's",
    "description": "Established in Australia back in 2006\. And what     started on the East Coast of Australia has spread across the
     world, building an empire of cafe racers.",
    "location": "Sydney, Australia"
},{
    "id": 3,
    "name": "Rough Crafts",
    "description": "A true testament to how far the custom bike
     world has come since the introduction of motorcycles in the
     early 20th century, Taiwan-based Rough Crafts is a design
     powerhouse.",
    "location": "Taiwan"
},{
    "id": 4,
    "name": "Roldand Sands",
    "description": "Is an American motorcycle racer and designer of
    custom high-performance motorcycles.",
    "location": "California, USA"
},{
    "id": 5,
    "name": "Chopper Dave",
    "description": "An artist, a biker, a builder and an innovator     among other things, but what it comes down to is David
     “ChopperDave” Freston is a motorcycle builder and fabricator     that is passionate about motorcycles",
    "location": "California, USA"
}]
```

1.  在`project/database/data-sample/`中，创建一个名为`items.json`的新文件，并添加以下代码：

```php
[{
    "id": 1,
    "type": "Handlebars",
    "name": "Apes Hanger 16 ",
    "company": "TC Bros",
    "bike_id": 2
},{
    "id": 2,
    "type": "Seat",
    "name": "Challenger",
    "company": "Biltwell Inc",
    "bike_id": 3
},{
    "id": 3,
    "type": "Exhaust",
    "name": "Side Shots",
    "company": "Vance and Hines",
    "bike_id": 3
}]
```

1.  现在，我们需要创建一些更多的种子，这样我们的应用程序就有了所有数据库样板。在终端窗口中，输入以下命令：

```php
php artisan make:seeder BikesGaragesTableSeeder
```

1.  将以下代码添加到`app/database/seeds/BikesGaragesTableSeeder.php`的`run()`公共函数中：

```php
DB::table('bike_garage')->insert([
    'bike_id' => 1,
    'garage_id' => 2
]);
DB::table('bike_garage')->insert([
    'bike_id' => 2,
    'garage_id' => 2
]);
```

1.  请注意，在上面的代码中，我们只是使用 Eloquent 的`insert()`方法手动插入记录，而不是为此任务创建 JSON 文件。

1.  现在，打开`project/database/data-sample/bikes.json`，并用以下代码替换内容：

```php
[{
    "id": 1,
    "make": "Harley Davidson",
    "model": "XL1200 Nightster",
    "year": "2009",
    "mods": "Nobis vero sint non eius. Laboriosam sed odit hic quia     doloribus. Numquam laboriosam numquam quas quis."
    "picture": "https://placeimg.com/640/480/nature","user_id": 2,
    "builder_id": 1
}, {
    "id": 2,
    "make": "Harley Davidson",
    "model": "Blackline",
    "year": "2008",
    "mods": "Nobis vero sint non eius. Laboriosam sed odit hic quia     doloribus. Numquam laboriosam numquam quas quis.",
    "picture": "https://placeimg.com/640/480/nature",
    "user_id": 1,
    "builder_id": 2
}, {
    "id": 3,
    "make": "Harley Davidson",
    "model": "Dyna Switchback",
    "year": "2009",
    "mods": "Nobis vero sint non eius. Laboriosam sed odit hic quia     doloribus. Numquam laboriosam numquam quas quis.",
    "picture": "https://placeimg.com/640/480/nature",
    "user_id": 2,
    "builder_id": 3
}, {
    "id": 4,
    "make": "Harley Davidson",
    "model": "Dyna Super Glide",
    "year": "2009",
    "mods": "Nobis vero sint non eius. Laboriosam sed odit hic quia     doloribus. Numquam laboriosam numquam quas quis.",
    "picture": "https://placeimg.com/640/480/nature",
    "user_id": 4,
    "builder_id": 4
},{
    "id": 5,
    "make": "Harley Davidson",
    "model": "Dyna Wild Glide",
    "year": "2005",
    "mods": "Nobis vero sint non eius. Laboriosam sed odit hic quia     doloribus. Numquam laboriosam numquam quas quis.",
    "picture": "https://placeimg.com/640/480/nature",
    "user_id": 5,
    "builder_id": 5
}]
```

在上面的代码中，我们为每辆自行车记录添加了`builder_id`和`user_id`，以建立自行车与其构建者以及用户与其自行车之间的关联。请记住，我们在上一章中创建了`project/database/data-sample/bikes.json`。

请注意，我们将自行车`4`和`5`分配给`user_id`为`4`和`5`。现在不要担心这个问题，因为在本书的后面，您将明白我们为什么现在这样做。

1.  打开`project/database/seeds/databaseSeeder.php`，取消注释用户的种子。

1.  让我们使用`seed`命令来填充我们的数据库。在您的终端窗口中输入以下命令：

```php
php artisan migrate:fresh --seed
```

使用上一个命令后，我们将得到以下输出：

```php
 Seeding: UsersTableSeeder
 Seeding: BikesTableSeeder
 Seeding: BuildersTableSeeder
 Seeding: ItemsTableSeeder
 Seeding: GaragesTableSeeder
 Seeding: BikeGarageTableSeeder
```

这意味着目前一切都是正确的。

`migrate:fresh`命令将从数据库中删除所有表，然后执行`migrate`命令进行全新安装。

# 使用 Tinker 查询数据库

**Tinker**是一个命令行应用程序，允许您与您的 Laravel 应用程序进行交互，包括 Eloquent ORM、作业、事件等。要访问 Tinker 控制台，请运行`artisan tinker`命令，我们之前用来检查数据库连接的第一章中的内容，*理解 Laravel 5 的核心概念*。

1.  打开您的终端窗口，输入以下命令：

```php
php artisan tinker
```

由于我们还没有为我们的应用程序创建任何控制器或路由，所以无法使用浏览器访问 API 端点来检查我们的数据。

然而，使用 Tinker，可以与我们的数据库进行交互，并检查我们的迁移文件和数据库种子是否一切顺利。

让我们去`builders`表，确保一切都设置正确。

1.  在您的终端和 Tinker 控制台中，输入以下命令：

```php
DB::table('builders')->get();
```

您的终端上的输出将是一个非常类似于以下 JSON 结构的构建者列表：

```php
 >>> DB::table('builders')->get();
=> Illuminate\Support\Collection {#810
     all: [
       {#811
         +"id": 1,
         +"name": "Diamond Atelier",
         +"description": "Diamond Atelier was founded by two fellow
         riders who grew tired of the same played-out custom
         bike look and feel they and their friends had grown
         accustomed     to witnessing.",
         +"location": "Munich, Germany",
         +"created_at": "XXXX",
         +"updated_at": "XXXX",
       },
       ...
       }]
```

请注意，您可以省略构建者列表上的所有记录，因此在您的代码块中不要重复。

1.  在您的终端和 Tinker 控制台中，输入以下命令：

```php
DB::table('builders')->find(3);
```

在这里，我们只有一个记录，id 为`3`，在`find()`函数中，正如我们在以下输出中所看到的：

```php
>>> DB::table('builders')->find(3);
=> {#817
     +"id": 3,
     +"name": "Rough Crafts",     +"description": "A true testament      to how far the custom bike world has come since the
     introduction     of motorcycles i
     n the early 20th century, Taiwan-based Rough Crafts is a design      powerhouse.",
     +"location": "Taiwan",
     +"created_at": "XXXX",
     +"updated_at": "XXXX",
   }
```

现在，让我们看看如何从上一个命令中获得相同的结果，但这次使用`Where`子句和`Builder`模型实例。

1.  在您的终端和 Tinker 控制台中，输入以下命令：

```php
Builder::where('id', '=', 3)->get();
```

我们将得到以下输出作为查询结果：

```php
>>> Builder::where('id', '=', 3)->get();
=> Illuminate\Database\Eloquent\Collection {#825
     all: [
       App\Builder {#828
         id: 3,
         name: "Rough Crafts",
         description: "A true testament to how far the custom bike          world has come since the introduction of motorcycles
         in the early 20th century, Taiwan-based Rough Crafts is a          design powerhouse.",
         location: "Taiwan",
         created_at: "XXXX",
         updated_at: "XXXX",
       },
     ],
   }
```

但是，请等一下，您一定在问自己，自行车数据在哪里？请记住，我们在种子中将自行车归属于构建者。让我们介绍关联查询。

1.  因此，假设我们想查询所有定制自行车。在您的终端和 Tinker 控制台中，输入以下命令：

```php
Builder::with('bike')->find(3);
```

请注意，上一个命令将使用`find()`方法和`::with()`方法关联返回 id 为`3`的构建者记录。这次，我们可以看到自行车的信息，如以下输出所示：

```php
>>> Builder::with('bike')->find(3);
=> App\Builder {#811
     id: 3,
     name: "Rough Crafts",     description: "A true testament to how
     far the custom bike world has come since the introduction of
     motorcycles in t
     he early 20th century, Taiwan-based Rough Crafts is a design
     powerhouse.",
     location: "Taiwan",
     created_at: "XXXX",
     updated_at: "XXXX",
     bike: App\Bike {#831
       id: 3,
       make: "Harley Davidson",
       model: "Dyna Switchback",
       year: "2009",
       mods: "Nobis vero sint non eius. Laboriosam sed odit hic quia
       doloribus. Numquam laboriosam numquam quas quis.",
       picture: "https://placeimg.com/640/480/nature",
       builder_id: 3,
       created_at: "XXXX",
       updated_at: "XXXX",
     },
   }
```

现在，让我们看看如何提交查询以获取所有模型关联，这次使用 Builder 模型实例。

1.  在您的终端和 Tinker 控制台中，输入以下命令：

```php
Bike::with(['items', 'builder'])->find(3);
```

请注意，我们在`::with()`方法中使用了一个数组来获取`items`和`builderassociations`，正如我们在以下输出中所看到的：

```php
>>> Bike::with(['items', 'builder'])->find(3);
[!] Aliasing 'Bike' to 'App\Bike' for this Tinker session.
=> App\Bike {#836
     id: 3,     make: "Harley Davidson",
     model: "Dyna Switchback",
     year: "2009",
     mods: "Nobis vero sint non eius. Laboriosam sed odit hic quia
     doloribus. Numquam laboriosam numquam quas quis.",
     picture: "https://placeimg.com/640/480/nature",
     builder_id: 3,
     created_at: "XXXX",
     updated_at: "XXXX",
     items: Illuminate\Database\Eloquent\Collection {#837
       all: [
         App\Item {#843
           id: 2,
           type: "Seat",
           name: "Challenger",
           company: "Biltwell Inc",
           bike_id: 3,
           created_at: "XXXX",
           updated_at: "XXXX",
         },
         App\Item {#845
           id: 3,
           type: "Exhaust",
           name: "Side Shots",
           company: "Vance and Hines",
           bike_id: 3,
           created_at: "XXXX",
           updated_at: "XXXX",
         },
       ],
     },
     builder: App\Builder {#844
       id: 3,
       name: "Rough Crafts",description: "A true testament to how
       far the custom bike world has come since the introduction of
       motorcycles in the early 20th century, Taiwan-based Rough
       Crafts is a design powerhouse.",
       location: "Taiwan",
       created_at: "XXXX",
       updated_at: "XXXX",
     },
   }
```

# 创建控制器和路由

我们已经快完成了，但还有一些步骤要走，这样我们才能完成我们的 API。现在是时候创建 API 控制器和 API 路由了。

在最新版本（5.6）的 Laravel 中，我们有一个新的可用于执行此任务的命令。这就是`--api`标志。让我们看看它在实践中是如何工作的。

# 创建和更新控制器函数

1.  打开你的终端窗口，输入以下命令：

```php
php artisan make:controller API/BuilderController --api
```

请注意，`--api`标志在`BuilderController`类中为我们创建了四个方法：

+   `index()` = GET

+   `store()` = POST

+   `show($id)` = GET

+   `update(Request $request, $id)` = PUT

+   `destroy($id)` = POST

1.  打开`project/app/Http/Controllers/API/BuilderController.php`，并在控制器导入后添加`App\Builder`代码。

1.  现在，让我们为每个方法添加内容。打开`project/app/Http/Controllers/API/BuilderController.php`，并用以下代码替换内容：

```php
<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\Builder;
class BuilderController extends Controller
{
    /**
    * Display a listing of the resource.
    *
    * @return \Illuminate\Http\Response
    *
    * @SWG\Get(
    * path="/api/builders",
    * tags={"Builders"}
    * summary="List Builders",
    * @SWG\Response(
    * response=200,
    * description="Success: List all Builders",
    * @SWG\Schema(ref="#/definitions/Builder")
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function index()
    {
        $listBuilder = Builder::all();
        return $listBuilder;

    }
```

1.  现在，让我们为`store/create`方法添加代码。在`index()`函数后添加以下代码：

```php
/**
* Store a newly created resource in storage.
*
* @param \Illuminate\Http\Request $request
* @return \Illuminate\Http\Response
*
* @SWG\Post(
* path="/api/builders",
* tags={"Builders"},
* summary="Create Builder",
* @SWG\Parameter(
*          name="body",
*          in="body",
*          required=true,
*          @SWG\Schema(ref="#/definitions/Builder"),
*          description="Json format",
*      ),
* @SWG\Response(
* response=201,
* description="Success: A Newly Created Builder",
* @SWG\Schema(ref="#/definitions/Builder")
* ),
* @SWG\Response(
* response="422",
* description="Missing mandatory field"
* ),
* @SWG\Response(
* response="404",
* description="Not Found"
* ),
* @SWG\Response(
     *          response="405",
     *          description="Invalid HTTP Method
* )
* ),
*/
public function store(Request $request)
{
    $createBuilder = Builder::create($request->all());
    return $createBuilder;
}
```

现在，让我们为按`id`获取方法添加代码。在`store()`函数后添加以下代码：

```php

/**
* Display the specified resource.
*
* @param int $id* @return \Illuminate\Http\Response
*
* @SWG\Get(
* path="/api/builders/{id}",
* tags={"Builders"},
* summary="Get Builder by Id",
* @SWG\Parameter(
* name="id",
* in="path",
* required=true,
* type="integer",
* description="Display the specified Builder by id.",
*      ),
* @SWG\Response(
* response=200,
* description="Success: Return the Builder",
* @SWG\Schema(ref="#/definitions/Builder")
* ),
* @SWG\Response(
* response="404",
* description="Not Found"
* ),
* @SWG\Response(
     *          response="405",
     *          description="Invalid HTTP Method"
     * )
* )
*/
public function show($id)
{
    $showBuilderById = Builder::with('Bike')->findOrFail($id);
    return $showBuilderById;
}
```

让我们为更新方法添加代码。在`show()`函数后添加以下代码：

```php
/**
* Update the specified resource in storage.
*
* @param \Illuminate\Http\Request $request
* @param int $id
* @return \Illuminate\Http\Response
*
* @SWG\Put(
* path="/api/builders/{id}",
* tags={"Builders"},
* summary="Update Builder",
* @SWG\Parameter(
* name="id",
* in="path",
* required=true,
* type="integer",
* description="Update the specified Builder by id.",
*      ),
* @SWG\Parameter(
*          name="body",
*          in="body",
*          required=true,
*          @SWG\Schema(ref="#/definitions/Builder"),
*          description="Json format",
*      ),
* @SWG\Response(
* response=200,
* description="Success: Return the Builder updated",
* @SWG\Schema(ref="#/definitions/Builder")
* ),
* @SWG\Response(
* response="422",
* description="Missing mandatory field"
* ),
* @SWG\Response(
* response="404",
* description="Not Found"
* ),
* @SWG\Response(
     *          response="405",
     *          description="Invalid HTTP Method"
     * )
* ),
*/
public function update(Request $request, $id)
{
    $updateBuilderById = Builder::findOrFail($id);
    $updateBuilderById->update($request->all());
    return $updateBuilderById;
}
```

现在，让我们为删除方法添加代码。在`update()`函数后添加以下代码：

```php
/**
* Remove the specified resource from storage.
*
* @param int $id
* @return \Illuminate\Http\Response
*
* @SWG\Delete(
* path="/api/builders/{id}",
* tags={"Builders"},
* summary="Delete Builder",
* description="Delete the specified Builder by id",
* @SWG\Parameter(
* description="Builder id to delete",
* in="path",
* name="id",
* required=true,
* type="integer",
* format="int64"
* ),
* @SWG\Response(
* response=404,
* description="Not found"
* ),
* @SWG\Response(
     *          response="405",
     *          description="Invalid HTTP Method"
     * ),
* @SWG\Response(
* response=204,
* description="Success: successful deleted"
* ),
* )
*/
public function destroy($id)
{
    $deleteBikeById = Bike::find($id)->delete();
    return response()->json([], 204);
    }
}
```

请注意，在`index()`函数中，我们使用`all()`方法列出所有自行车，并且只在`show($id)`函数中使用关联的`::with()`方法。

我们已经将 Swagger 定义添加到了控制器中，但不要担心：稍后在本章中，我们将详细讨论这个问题。

模型关联查询列出自行车并显示自行车详细信息，是一个简单的 API 决定。正如你所看到的，我们在返回自行车列表时没有返回所有关联，只在按 id 获取自行车时返回关联。在每个请求中返回每个关联都没有意义，所以在自行车列表中，我们只显示自行车的详细信息，当我们点击详细信息时，我们将看到所有模型关联的完整信息。所以现在不要担心这个，因为在第十章 *使用 Bootstrap 4 和 NgBootstrap 创建前端视图*中，我们将看到如何做到这一点。

1.  打开你的终端窗口，输入以下命令：

```php
php artisan make:controller API/ItemController --api
```

打开`project/app/Http/Controllers/API/ItemController.php`，并在控制器导入后添加以下代码：`use App\Item;`。

1.  现在，让我们为每个方法添加内容。打开`project/app/Http/Controllers/API/ItemController.php`，并为每个方法添加以下代码：

```php
<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\Item;
class ItemController extends Controller
{
    /**
    * Display a listing of the resource.
    *
    * @return \Illuminate\Http\Response
    *
    * @SWG\Get(
    * path="/api/items",
    * tags={"Items"},
    * summary="List Items",
    * @SWG\Response(
    * response=200,
    * description="Success: List all Items",
    * @SWG\Schema(ref="#/definitions/Item")
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function index()
    {
        $listItems = Item::all();
        return $listItems;
    }
    /**
    * Store a newly created resource in storage.
    *
    * @param \Illuminate\Http\Request $request
    * @return \Illuminate\Http\Response
    *
    * @SWG\Post(
    * path="/api/items",
    * tags={"Items"},
    * summary="Create Item",
    * @SWG\Parameter(
    *           name="body",
    *           in="body",
    *           required=true,
    *           @SWG\Schema(ref="#/definitions/Item"),
    *           description="Json format",
    *       ),
    * @SWG\Response(
    * response=201,
    * description="Success: A Newly Created Item",
    * @SWG\Schema(ref="#/definitions/Item")
    * ),
    * @SWG\Response(
    * response="422",
    * description="Missing mandatory field"
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function store(Request $request)
    {
        $createItem = Item::create($request->all());
        return $createItem;
    }
    /**
    * Display the specified resource.
    *
    * @param int $id
    * @return \Illuminate\Http\Response
    *
    * @SWG\Get(
    * path="/api/items/{id}",
    * tags={"Items"},
    * summary="Get Item by Id",
    * @SWG\Parameter(
    * name="id",
    * in="path",
    * required=true,
    * type="integer",
    * description="Display the specified Item by id.",
    *       ),
    * @SWG\Response(
    * response=200,
    * description="Success: Return the Item",
    * @SWG\Schema(ref="#/definitions/Item")
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function show($id)
    {
        $showItemById = Item::with('Bike')->findOrFail($id);
        return $showItemById;
    }
    /**
    * Update the specified resource in storage.
    *
    * @param \Illuminate\Http\Request $request
    * @param int $id
    * @return \Illuminate\Http\Response
    *
    * @SWG\Put(
    * path="/api/items/{id}",
    * tags={"Items"},
    * summary="Update Item",
    * @SWG\Parameter(
    * name="id",
    * in="path",
    * required=true,
    * type="integer",
    * description="Update the specified Item by id.",
    *       ),
    * @SWG\Parameter(
    *           name="body",
    *           in="body",
    *           required=true,
    *           @SWG\Schema(ref="#/definitions/Item"),
    *           description="Json format",
    *       ),
    * @SWG\Response(
    * response=200,
    * description="Success: Return the Item updated",
    * @SWG\Schema(ref="#/definitions/Item")
    * ),
    * @SWG\Response(
    * response="422",
    * description="Missing mandatory field"
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function update(Request $request, $id)
    {
        $updateItemById = Item::findOrFail($id);
        $updateItemById->update($request->all());
        return $updateItemById;
    }
    /**
    * Remove the specified resource from storage.
    *
    * @param int $id
    * @return \Illuminate\Http\Response
    *
    * @SWG\Delete(
    * path="/api/items/{id}",
    * tags={"Items"},
    * summary="Delete Item",
    * description="Delete the specified Item by id",
    * @SWG\Parameter(
    * description="Item id to delete",
    * in="path",
    * name="id",
    * required=true,
    * type="integer",
    * format="int64"
    * ),
    * @SWG\Response(
    * response=404,
    * description="Not found"
    * ),
    * @SWG\Response(
    * response=204,
    * description="Success: successful deleted"
    * ),
    * )
    */
    public function destroy($id)
    {
        $deleteItemById = Item::findOrFail($id)->delete();
        return response()->json([], 204);
    }
}
```

1.  打开你的终端窗口，输入以下命令：

```php
php artisan make:controller API/BikeController --api
```

打开`project/app/Http/Controllers/API/BikeController.php`，并在控制器导入后添加以下代码：

```php
use App\Bike;
```

1.  现在，让我们为每个方法添加内容。打开`project/app/Http/Controllers/API/BikeController.php`，并为每个方法添加以下代码：

```php
<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\Bike;
class BikeController extends Controller
{
    /**
    * Display a listing of the resource.
    ** @return \Illuminate\Http\Response
    *
    * @SWG\Get(
    * path="/api/bikes",
    * tags={"Bikes"},
    * summary="List Bikes",
    * @SWG\Response(
    * response=200,
    * description="Success: List all Bikes",
    * @SWG\Schema(ref="#/definitions/Bike")
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function index()
    {
        $listBikes = Bike::all();
        return $listBikes;
    }
    /**
    * Store a newly created resource in storage.
    *
    * @param \Illuminate\Http\Request $request
    * @return \Illuminate\Http\Response
    *
    * @SWG\Post(
    * path="/api/bikes",
    * tags={"Bikes"},
    * summary="Create Bike",
    * @SWG\Parameter(
    *           name="body",
    *           in="body",
    *           required=true,
    *           @SWG\Schema(ref="#/definitions/Bike"),
    *           description="Json format",
    *       ),
    * @SWG\Response(
    * response=201,
    * description="Success: A Newly Created Bike",
    * @SWG\Schema(ref="#/definitions/Bike")
    * ),
    * @SWG\Response(
    * response="422",
    * description="Missing mandatory field"
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function store(Request $request)
    {
        $createBike = Bike::create($request->all());
        return $createBike;
    }
    /**
    * Display the specified resource.
    *
    * @param int $id
    * @return \Illuminate\Http\Response
    *
    * @SWG\Get(
    * path="/api/bikes/{id}",
    * tags={"Bikes"},
    * summary="Get Bike by Id",
    * @SWG\Parameter(
    * name="id",
    * in="path",
    * required=true,
    * type="integer",
    * description="Display the specified bike by id.",
    *       ),
    * @SWG\Response(
    * response=200,
    * description="Success: Return the Bike",
    * @SWG\Schema(ref="#/definitions/Bike")
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function show($id)
    {
        $showBikeById = Bike::with(['items', 'builder', 'garages'])-
        >findOrFail($id);
        return $showBikeById;
    }
    /**
    * Update the specified resource in storage.
    *
    * @param \Illuminate\Http\Request $request
    * @param int $id
    * @return \Illuminate\Http\Response
    *
    * @SWG\Put(
    * path="/api/bikes/{id}",
    * tags={"Bikes"},
    * summary="Update Bike",
    * @SWG\Parameter(
    * name="id",
    * in="path",
    * required=true,
    * type="integer",
    * description="Update the specified bike by id.",
    *       ),
    * @SWG\Parameter(
    *           name="body",
    *           in="body",
    *           required=true,
    *           @SWG\Schema(ref="#/definitions/Bike"),
    *           description="Json format",
    *       ),
    * @SWG\Response(
    * response=200,
    * description="Success: Return the Bike updated",
    * @SWG\Schema(ref="#/definitions/Bike")
    * ),
    * @SWG\Response(
    * response="422",
    * description="Missing mandatory field"
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * )
    * ),
    */
    public function update(Request $request, $id)
    {
        $updateBikeById = Bike::findOrFail($id);
        $updateBikeById->update($request->all());
        return $updateBikeById;
    }
    /**
    * Remove the specified resource from storage.
    *
    * @param int $id* @return \Illuminate\Http\Response
    *
    * @SWG\Delete(
    * path="/api/bikes/{id}",
    * tags={"Bikes"},
    * summary="Delete bike",
    * description="Delete the specified bike by id",
    * @SWG\Parameter(
    * description="Bike id to delete",
    * in="path",
    * name="id",
    * required=true,
    * type="integer",
    * format="int64"
    * ),
    * @SWG\Response(
    * response=404,
    * description="Not found"
    * ),
    * @SWG\Response(
    * response=204,
    * description="Success: successful deleted"
    * ),
    * )
    */
    public function destroy($id)
    {
        $deleteBikeById = Bike::find($id)->delete();
        return response()->json([], 204);
    }
}
```

打开你的终端窗口，输入以下命令：

```php
php artisan make:controller API/RatingController --api
```

1.  打开`project/app/Http/Controllers/API/RatingController.php`，并在控制器导入后添加以下代码：

```php
use App\Rating;
```

1.  现在，让我们为每个方法添加内容。打开`project/app/Http/Controllers/API/RatingController.php`，并为每个方法添加以下代码：

```php
<?php
namespace App\Http\Controllers\API;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\Bike;
use App\Rating;
use App\Http\Resources\RatingResource;
class RatingController extends Controller
{
    /**
    * Store a newly created resource in storage.
    *
    * @param \Illuminate\Http\Request $request
    * @return \Illuminate\Http\Response
    *
    * @SWG\Post(
    * path="/api/bikes/{bike_id}/ratings",
    * tags={"Ratings"},
    * summary="rating a Bike",
    * @SWG\Parameter(
    * in="path",
    * name="id",
    * required=true,
    * type="integer",
    * format="int64",
    *           description="Bike Id"
    *       ),
    * @SWG\Parameter(
    *           name="body",
    *           in="body",
    *           required=true,
    *           @SWG\Schema(ref="#/definitions/Rating"),
    *           description="Json format",
    *        ),
    * @SWG\Response(
    * response=201,
    * description="Success: A Newly Created Rating",
    * @SWG\Schema(ref="#/definitions/Rating")
    * ),
    * @SWG\Response(
    * response=401,
    * description="Refused: Unauthenticated"
    * ),
    * @SWG\Response(
    * response="422",
    * description="Missing mandatory field"
    * ),
    * @SWG\Response(
    * response="404",
    * description="Not Found"
    * ),
    * @SWG\Response(
    *        response="405",
    *    description="Invalid HTTP Method"
    * ),
    * security={
    *        { "api_key":{} }
    * }
    * ),
    */
    public function store(Request $request, Bike $bike)
    {
        $rating = Rating::firstOrCreate(
            [
                'user_id' => $request->user()->id,
                'bike_id' => $bike->id,
            ],
            ['rating' => $request->rating]
        );
        return new RatingResource($rating);
    }
}
```

你应该在评分控制器代码中发现一些奇怪的东西。其中，我们有一些新的错误代码，`422`，`405`，以及 Swagger 文档中的一个安全标签，还有一个叫做**rating resource**的新导入。

这可能听起来奇怪，但不要惊慌；我们将在接下来的章节中详细讨论这个问题。

# 创建 API 路由

现在是时候创建一些 API 路由并检查我们到目前为止构建的内容了。我们正在使用`apiResource`的新功能。

打开`project/routes/api.php`，并添加以下代码：

```php
Route::apiResources([
    'bikes' => 'API\BikeController',
    'builders' => 'API\BuilderController',
    'items' => 'API\ItemController',
    'bikes/{bike}/ratings' => 'API\RatingController'
]);
```

此时，我们已经为我们的 API 添加了必要的代码，所以我们需要做一些小的调整并解释更多的东西。

# 生成 Swagger UI 文档

从先前的示例中可以看出，我们已经通过 Swagger 定义向我们最近创建的控制器添加了 API 的文档。这与我们在先前的示例中使用的代码相同。让我们在 Swagger UI 上生成文档。

打开您的终端窗口，输入以下命令：

```php
php artisan l5-swagger:generate
```

根据先前的 Swagger 定义中的错误消息，您可以注意到我们有一些新的 HTTP 错误，比如`422`。

这意味着如果用户尝试输入一些数据，其中一个或多个必填字段缺失，我们的 API 必须返回一个 HTTP 错误代码。这将是`422`。因此，让我们看看如何实现一些验证并验证一些常见的 API HTTP 代码。

# 总结

我们已经完成了本章的第一部分，我们为 API 创建了一个强大且可扩展的 RESTful 基础。我们学会了如何创建控制器、路由，以及如何处理 Eloquent 关系。

我们还有很多工作要做，因为我们需要处理错误消息、资源和基于令牌的身份验证。在下一章中，我们将看到如何完成这些工作。
