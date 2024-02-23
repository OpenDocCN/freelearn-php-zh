# 第十章。测试和调试您的应用程序

在本章中，我们将涵盖：

+   设置和配置 PHPUnit

+   编写和运行测试用例

+   使用 Mockery 测试控制器

+   使用 Codeception 编写验收测试

+   调试和分析您的应用程序

# 介绍

随着 Web 应用程序变得更加复杂，我们需要确保对现有代码所做的任何更改或更新不会对其他代码产生负面影响。检查这一点的一种方法是创建单元测试。Laravel 为我们提供了非常有用的方法来包含单元测试。

# 设置和配置 PHPUnit

在这个配方中，我们将看到如何安装和设置流行的 PHPUnit 测试软件包：PHPUnit。

## 准备工作

对于这个配方，我们需要一个正常安装的 Laravel 4。我们还需要从[`getcomposer.org`](http://getcomposer.org)安装 Composer 依赖工具。

## 如何做...

要完成这个配方，请按照给定的步骤进行操作：

1.  在应用程序的根目录中，将以下行添加到`composer.json`文件中：

```php
  "require-dev": {
  "phpunit/phpunit": "3.7.*"
  },
```

1.  打开命令行窗口，导航到根目录，并使用以下行在 Composer 工具上运行更新：

```php
 **php composer update**

```

1.  安装后，在命令行窗口中使用以下命令快速测试：

```php
 **vendor/bin/phpunit**

```

## 它是如何工作的...

我们的`composer.json`文件告诉 Composer 工具应安装哪些软件包。因此，我们的第一个任务是将`phpunit`软件包添加为要求。保存该文件后，我们将运行`update`命令，`phpunit`将被添加到我们的`vendor`目录。

安装后，我们可以运行命令来测试`phpunit`，并确保它已正确安装。Laravel 在`app/tests`目录中附带了一个示例测试用例，并且应该通过所有测试。

# 编写和运行测试用例

对于这个配方，如果我们已经安装并正常工作的 PHPUnit，我们可以编写一个测试用例，并使用 PHPUnit 来检查它是否有效。

## 准备工作

要运行测试用例，我们需要一个正常安装的 Laravel。我们还需要从前面的配方*设置和配置 PHPUnit*中安装 PHPUnit。

## 如何做...

要完成这个配方，请按照给定的步骤进行操作：

1.  在`app/tests`目录中，创建一个名为`MyAppTest.php`的文件，并添加以下代码：

```php
  <?php
class MyAppTest extends TestCase {

  /**
   * Testing the MyApp route
   *
   * @return void
   */
  public function testMyAppRoute()
{
  $response = $this->call('GET', 'myapp');
  $this->assertResponseOk();
  $this->assertEquals('This is my app', $response >getContent());
}
}
```

1.  在命令行窗口中运行测试，我们应该在输入以下命令时得到失败的测试：

```php
 **vendor/bin/phpunit**

```

1.  在我们的`routes.php`文件中，添加以下代码的新路由：

```php
  Route::get('myapp', function()
{
  return 'This is my app';
});
```

1.  再次运行测试以获得通过的单元测试

```php
 **vendor/bin/phpunit**

```

## 它是如何工作的...

当我们运行 PHPUnit 测试时，Laravel 将自动查找`app/tests`目录。我们首先在该目录中创建一个新文件来保存名为`MyAppTest`的测试，并扩展`TestCase`。

对于这个简单的测试，我们使用`call`方法并在`myapp`路由上执行`GET`请求。我们首先检查的是我们收到了`Ok`或 200 状态代码，然后返回的内容是字符串`This is my app`。在这一点上，当我们运行测试时，它将失败，因为我们还没有创建路由。

接下来，我们创建我们的`myapp`路由并返回字符串`This is my app`。最后，我们重新运行测试，应该得到成功的结果。

## 另请参阅

+   *设置和配置 PHPUnit*配方

# 使用 Mockery 测试控制器

有时，我们需要测试使用我们的数据库的代码。通常接受的做法是在运行单元测试时不应实际在数据库上执行实时查询。为了解决这个问题，我们可以使用 Mockery 包来伪造我们的数据。

## 准备工作

对于这个配方，我们需要已安装和正常工作的 Laravel，以及来自*设置和配置 PHPUnit*配方的 PHPUnit。

## 如何做...

要完成这个配方，请按照给定的步骤进行操作：

1.  打开我们的`composer.json`文件，并确保包含以下代码：

```php
  "require-dev": 
{
  "phpunit/phpunit": "3.7.*",
  "mockery/mockery": "dev-master"
},
```

1.  打开命令行终端，并使用以下命令运行 Composer 更新：

```php
 **php composer.phar update**

```

1.  更新后，在`app/controllers`目录中，使用以下代码创建`ShipsController.php`文件：

```php
<?php

class ShipsController extends BaseController {

  protected $ships; 
  public function __construct(Spaceship $ships) 
{
  $this->ships = $ships;
}

  public function showShipName()
{
  $ship = $this->ships->first();
  return $ship->name;
}
}
```

1.  在`routes.php`文件中，使用以下命令添加一个路由到这个控制器：

```php
 **Route::get('ship', 'ShipsController@showShipName');**

```

1.  在`app/tests`目录中，使用以下代码创建一个名为`SpaceshipTest.php`的文件：

```php
<?php

class SpaceshipTest extends TestCase {

  public function testFirstShip ()
{
  $this->call('GET', 'ship');
  $this->assertResponseOk();
}
}
```

1.  回到命令行窗口，使用以下命令运行我们的测试：

```php
 **vendor/bin/phpunit**

```

1.  此时，我们将得到一个显示以下消息的失败测试：

```php
**ReflectionException: Class Spaceship does not exist**

```

1.  由于`Spaceship`类将是我们的模型，我们将使用 Mockery 来模拟它。使用以下代码更新`SpaceshipTest`类：

```php
<?php

class SpaceshipTest extends TestCase {

  public function testFirstShip()
{
  $ship = new stdClass();
  $ship->name = 'Enterprise';

  $mock = Mockery::mock('Spaceship');
  $mock->shouldReceive('first')->once()->andReturn($ship);

  $this->app->instance('Spaceship', $mock);
  $this->call('GET', 'ship');
  $this->assertResponseOk();
}

   public function tearDown()
{
  Mockery::close();
}
}
```

1.  现在，回到命令行窗口，再次运行测试，它应该通过。

## 工作原理...

我们首先通过 Composer 安装 Mockery 软件包。这将允许我们在整个应用程序中使用它。接下来，我们创建一个控制器，其中包含一个将显示单个飞船名称的方法。在控制器的构造函数中，我们传入要使用的模型，这种情况下将命名为`Spaceship`并使用 Laravel 的 Eloquent ORM。

在`showShipName`方法中，我们将从 ORM 中获取第一条记录，然后简单地返回记录的名称。然后，我们需要创建一个指向控制器和`showShipName`方法的路由。

当我们首次创建测试时，我们只是发出一个`GET`请求，看看它是否返回一个 OK 响应。此时，由于我们还没有创建`Spaceship`模型，当我们运行测试时会显示错误。我们可以向数据库添加所需的表并创建模型，测试将通过。但是，在测试控制器时，我们不想担心数据库，应该只测试控制器代码是否有效。为此，我们现在可以使用 Mockery。

当我们在`Spaceship`类上调用`first`方法时，它将给我们一个包含所有返回字段的对象，因此我们首先创建一个通用对象，并将其分配给`$ship`控制器。然后，我们为`Spaceship`类创建我们的`mock`对象，当我们的控制器请求`first`方法时，`mock`对象将返回我们之前创建的通用对象。

接下来，我们需要告诉 Laravel，每当请求`Spaceship`实例时，它应该使用我们的`mock`对象。最后，在我们的船舶路线上调用`GET`，确保它返回一个 OK 响应。

## 另请参阅

+   *设置和配置 PHPUnit*配方

# 使用 Codeception 编写验收测试

验收测试是测试应用程序是否向浏览器输出正确信息的有用方法。使用 Codeception 等软件包，我们可以自动化这些测试。

## 准备工作

对于这个配方，我们需要安装一个 Laravel 的工作副本。

## 如何做...

要完成此配方，请按照给定的步骤进行：

1.  打开`composer.json`文件，并将以下行添加到我们的`require-dev`部分：

```php
  "codeception/codeception": "dev-master",
```

1.  打开命令行窗口，并使用以下命令更新应用程序：

```php
 **php composer.phar update**

```

1.  安装完成后，我们需要在终端中运行`bootstrap`命令，如下所示：

```php
 **vendor/bin/codecept bootstrap app**

```

1.  在`app/tests/acceptance`目录中，使用以下代码创建一个名为`AviatorCept.php`的文件：

```php
<?php

$I = new WebGuy($scenario);
$I->wantTo('Make sure all the blueprints are shown');
$I->amOnPage('/');
$I->see('All The Blueprints');
```

1.  在我们的主`routes.php`文件中，使用以下代码更新默认路由：

```php
Route::get('/', function()
{
return 'Way of the future';
});
```

1.  打开命令行窗口，并使用以下命令运行验收测试：

```php
 **vendor/bin/codecept run –c app**

```

1.  此时，我们应该看到它失败了。为了使其通过，再次更新默认路由，输入以下代码：

```php
Route::get('/', function()
{
return 'All The Blueprints';
});
```

1.  在命令行窗口中，使用以下命令再次运行测试：

```php
 **vendor/bin/codecept run –c app**

```

1.  这次应该通过。

## 工作原理...

我们首先通过 Composer 安装 Codeception 软件包。一旦下载完成，我们运行`bootstrap`命令，它将创建所有所需的文件和目录。Codeception 会自动将文件和文件夹添加到`tests`目录中；因此，为了确保它们被添加到 Laravel 的测试目录中，我们在`bootstrap`命令的末尾添加`app`目录。

接下来，在`acceptance`目录中创建一个文件来保存我们的测试。我们首先创建一个新的`WebGuy`对象，这是 Codeceptions 类来运行验收测试。接下来的一行描述了我们想要做的事情，在这种情况下是查看所有的蓝图。下一行告诉测试我们需要在哪个页面，这将对应我们的路由。对于我们的目的，我们只是检查默认路由。最后，我们告诉测试我们想在页面上`看到`什么。我们在这里放的任何文本都应该显示在页面的某个地方。

我们对默认路由的第一次尝试将显示`Way of the future`；因此，当运行 Codeception 测试时，它将失败。要运行测试，我们使用`run`命令，并确保我们使用`-c`标志，并指定`app`作为测试路径，因为我们安装了引导文件在`app/tests`目录内。

然后，我们可以更新路由以显示文本`All The Blueprints`并重新运行测试。这次，它会通过。

## 还有更多...

Codeception 是一个非常强大的测试套件，有许多不同的选项。要完全了解它的所有功能，请访问[`codeception.com/`](http://codeception.com/)。

# 调试和配置您的应用

如果我们想知道我们的应用在幕后是如何工作的，我们需要对其进行配置。这个步骤将展示如何向我们的 Laravel 应用添加一个配置文件。

## 准备工作

对于这个步骤，我们需要一个配置正确的 Laravel 工作副本和一个 MySQL 数据库。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  打开命令行窗口，并使用`artisan`命令按照以下代码创建我们的迁移：

```php
 **php artisan migrate::make create_spaceships_table –create –table="spaceships"**

```

1.  在`app/database/migrations`文件夹中，打开以日期开头并以`create_spaceships_table.php`结尾的文件，将其用于我们的数据库表

```php
<?php

  use Illuminate\Database\Schema\Blueprint;
  use Illuminate\Database\Migrations\Migration;

class CreateSpaceshipsTable extends Migration {

  /**
  * Run the migrations.
  *
  * @return void
  */
public function up()
{
  Schema::create('spaceships', function(Blueprint $table)
{
  $table->increments('id');
  $table->string('name');
  $table->string('movie');
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
  Schema::drop('spaceships');
}

}
```

1.  在`app/database/seeds`文件夹中，创建一个名为`SpaceshipSeeder.php`的文件，如下所示：

```php
<?php

class SpaceshipSeeder extends Seeder {

  /**
  * Run the database seeds.
  *
  * @return void
  */
  public function run()
{
  DB::table('spaceships')->delete();

  $ships = array(
  array(
  'name'   => 'Enterprise',
  'movie'  => 'Star Trek'
),
  array(
  'name'   => 'Millenium Falcon',
  'movie'  => 'Star Wars'
),
  array(
  'name'   => 'Serenity',
  'movie'  => 'Firefly'
),
);

  DB::table('spaceships')->insert($ships);
}
}
```

1.  在同一个目录中，打开`DatabaseSeeder.php`文件，并确保`run()`方法看起来像以下代码片段：

```php
public function run()
{
  Eloquent::unguard();
  $this->call('SpaceshipSeeder');
}
```

1.  回到命令行窗口，使用以下代码安装迁移并运行 seeder：

```php
 **php artisan migrate**
 **php artisan db:seed**

```

1.  在`app/models`目录中，创建一个名为`Spaceship.php`的文件，如下代码所示：

```php
<?php

class Spaceship extends Eloquent{

  protected $table = 'spaceships';
}
```

1.  在`app/controllers`目录中，创建一个名为`ShipsController.php`的文件

```php
<?php

class ShipsController extends BaseController {

  protected $ships; 

  public function __construct(Spaceship $ships) 
  {
  $this->ships = $ships;
}

  public function showShipName()
{
  $ships = $this->ships->all();
  Log::info('Ships loaded: ' . print_r($ships, TRUE));
  return View::make('ships')->with('ships', $ships);
}
}
```

1.  在`routes.php`文件中，注册路由如下命令所示：

```php
 **Route::get('ship', 'ShipsController@showShipName');**

```

1.  在`app/views`目录中，创建一个名为`ships.blade.php`的视图，如下代码所示：

```php
  @foreach ($ships as $s)
  {{ $s->name }} <hr>
  @endforeach
```

1.  此时，如果我们转到`http://{your-dev-url}/public/ship`，我们将看到船只列表。接下来，我们需要打开`composer.json`文件，并在`require-dev`部分中添加以下行：

```php
  "loic-sharma/profiler": "dev-master"
```

1.  然后在命令行窗口中，使用以下命令更新 Composer：

```php
 **php composer.phar update**

```

1.  在所有东西都下载完成后，在`app/config`文件夹中，打开`app.php`文件。在`providers`数组中，在代码的末尾添加以下行：

```php
  'Profiler\ProfilerServiceProvider',
```

1.  在同一个文件中，在`aliases`数组中，添加以下行：

```php
  'Profiler' => 'Profiler\Facades\Profiler',
```

1.  在这个文件的顶部，确保`debug`设置为 true，然后在浏览器中返回`http://{your-dev-url}/public/ship`。`profiler`将显示在浏览器窗口底部。

## 它是如何工作的...

我们的第一步是创建我们想要配置文件的页面。我们首先使用`artisan`命令创建一个`migrations`文件，然后添加 Schema 构建器代码来创建我们的 spaceships 表。完成后，我们可以使用 seeder 文件向表中添加一些信息。

完成后，我们现在可以运行迁移和 seeder，我们的表将被创建，并且所有信息已经被填充。

接下来，我们为我们的数据创建一个简单的模型和一个控制器。在控制器中，我们将简单地获取所有的船只，并将变量传递给我们的船只视图。我们还将在代码中间添加一个日志事件。这将允许我们以后调试代码，如果需要的话。

完成后，我们可以看到我们创建的船只列表。

然后，我们需要安装基于 Laravel 早期版本的性能分析器包。更新了我们的 Composer 文件后，我们注册性能分析器，这样我们的应用程序就知道它的存在；如果以后想要更多地使用它，我们还可以注册 Façade。

在我们的配置文件中，如果我们将`debug`设置为`TRUE`，那么性能分析器将在我们访问的每个页面上显示。我们可以通过简单地将`debug`设置为`FALSE`来禁用性能分析器。

## 还有更多...

我们还可以使用以下代码段中显示的 startTimer 和 endTimer 方法向我们的应用程序添加定时器：

```php
  Profiler::startTimer('myTime');
  {some code}
  Profiler::endTimer('myTime');
```
