# 第二章：自动化测试-迁移和种子数据库

到目前为止，我们已经创建了一些基本模型和数据库的概要。现在，我们需要创建数据库迁移和种子。传统上，数据库“dump”文件被用作传递表结构和数据的方式，包括初始或预定义记录，如默认值；不变的列表，如城市或国家；以及用户，如“admin”。这些包含 SQL 的转储文件可以提交到源代码控制。这并不总是维护数据库完整性的最佳方式；因为每当开发人员添加记录或修改数据库时，团队中的所有开发人员都需要手动添加或删除数据、表、行、列或索引，或者删除并重新创建数据库。迁移允许数据库以代码形式存在，实际上驻留在 Laravel 项目内，并在源代码控制中进行版本控制。

迁移是从命令行运行的，也可以自动化，以在需要时自动创建数据库（如果不存在），或删除并重新创建表并填充表（如果已存在）。迁移在 Laravel 中已经存在一段时间，因此它们在 Laravel 5 中的存在并不令人惊讶。

# 使用 Laravel 的迁移功能

第一步是运行`artisan`命令：

```php
**$ php artisan migrate:install**

```

这将创建一个名为`migration`的表，其中包含两列：`migration`是 MySQL 中的 varchar 255，`batch`是整数。这个表将被 Laravel 用来跟踪已运行的迁移。换句话说，它维护了所有已执行操作的历史记录。以下是主要操作的列表：

+   `install`：如前所述，此操作安装

+   `refresh`：此操作重置并重新运行所有迁移

+   `reset`：此操作回滚所有迁移

+   `rollback`：此操作是一种“撤消”类型，只是回滚上一个操作

+   `status`：此操作生成迁移的类似表格的输出，并指出它们是否已运行

## 迁移示例

Laravel 5 在`/database/migrations`目录中包含两个迁移。

第一个迁移创建了`users`表。

第二个创建`password_resets`表，正如你可能已经猜到的，用于恢复丢失的密码。除非指定，迁移操作的是在`/config/database.php`配置文件中配置的数据库：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration {

  /**
   * Run the migrations.
   *
   * @return void
   */
  public function up()
  {
    Schema::create('users', function(Blueprint $table)
    {
      $table->smallIncrements('id')->unsigned();
      $table->string('name');
      $table->string('email')->unique();
      $table->string('password', 60);
      $table->rememberToken();
      $table->timestamps();
      $table->softDeletes();
    });
  }

  /**
   * Reverse the migrations.
   *
   * @return void
   */
  public function down()
  {
    Schema::drop('users');
  }

}
```

迁移扩展了`Migration`类并使用`Blueprint`类。

有两种方法：`up`和`down`，分别在使用`migrate`命令和`rollback`命令时使用。`Schema::create()`方法以表名作为第一个参数调用，并以函数回调作为第二个参数，接受`Blueprint`对象的实例作为参数。

## 创建表

`$table`对象有一些方法，执行任务，如创建索引，设置自增字段，指定应创建的字段类型，并将字段名称作为参数传递。

第一个命令用于创建自增字段`id`，这将是表的主键。然后，创建字符串字段，如`name`、`email`和`password`。请注意，`unique`方法链接到`email`字段的`create`语句，说明`email`字段将用作登录名/用户 ID，这是大多数现代 Web 应用程序的常见做法。`rememberToken`用于允许用户在每个会话中保持身份验证。此令牌在每次登录和注销时重置，保护用户免受潜在的恶意劫持尝试。

## Laravel 迁移魔法

Laravel 迁移还能够创建时间戳字段，用于自动存储每个模型的创建和更新信息。

## $table->timestamps();

以下代码告诉迁移自动在表中创建两列，即 `created_at` 和 `updated_at`，这是 Laravel 的 Eloquent **对象关系映射** (**ORM**) 自动使用的，以便应用程序知道对象何时创建和何时更新：

`$table->timestamps()`

在下面的示例中，字段更新如下：

```php
/*
*   created_at is set with timestamps
*/
$user = new User();
$user->email = "johndoe@acmewidgets.com";
$user->name = "John Doe";
$user->save(); // created_at is set with timestamps

/*
*   updated_at is set with timestamps
*/
$user = User::find(1); //where 1 is the $id
$user->email = "johndoe@acmeenterprise.com";
$user->save(); //updated_at is updated
```

另一个很棒的 Laravel 功能是软删除字段。这提供了一种**回收站**，允许数据在以后可选地恢复。

这个功能简单地向表中添加了另一列，以允许软删除数据。要添加到迁移中的代码如下所示：

```php
$table->softDeletes();
```

这在 `数据库, deleted_at,` 中添加了一列，它的值可以是 `null`，也可以是一个时间戳，表示记录被删除的时间。这在您的数据库应用程序中构建了一个回收站功能。

运行以下命令：

```php
**$ php artisan migrate**

```

迁移已启动并创建了表。现在出现了**迁移**表，如下截图所示：

![$table->timestamps();](img/B04559_02_01.jpg)

`users` 表的结构如下截图所示：

![$table->timestamps();](img/B04559_02_02.jpg)

要回滚迁移，请运行以下命令：

```php
**$ php artisan migrate:rollback**

```

`rollback` 命令使用迁移表来确定要回滚的操作。在这种情况下，运行后的 `migrations` 表现在是空的。

# 从模式到迁移

在开发过程中经常发生的一种情况是创建了一个模式，然后我们需要从该模式创建一个迁移。在撰写本文时，Laravel 核心中没有官方工具可以做到这一点，但有几个可用的包。

其中一个这样的包是 `migrations-generator` 包。

首先，在 `composer.json` 文件的 `require-dev` 部分中添加以下行，以在 `composer.json` 文件中要求 `migrations-generator` 依赖项：

```php
"require-dev": {
    "phpunit/phpunit": "~4.0",
    "phpspec/phpspec": "~2.1",
    "xethron/migrations-generator": "dev-feature/laravel-five-stable",
    "way/generators": "dev-feature/laravel-five-stable"
  },
```

还需要在根级别的 `composer.json` 文件中添加以下文本：

```php
"repositories": [
  {
    "type": "git",
    "url": "git@github.com:jamisonvalenta/Laravel-4-Generators.git"
  }],
```

## Composer 的 require-dev 命令

`require-dev` 命令与 `require` 相反，是 composer 的一种机制，允许只在开发阶段需要的某些包。大多数测试工具和迁移工具只会在本地开发机器、QA 机器和/或持续集成环境中使用，而不会在生产环境中使用。这种机制可以使您的生产安装不受不必要的包的影响。

## Laravel 的提供者数组

Laravel 的 `providers` 数组在 `config/app.php` 文件中列出了 Laravel 随时可用的提供者。

我们将添加 `way generator` 和 `Xethron migration` 服务提供者：

```php
'providers' => [

        /*
         * Laravel Framework Service Providers...
         */
          Illuminate\Foundation\Providers\ArtisanServiceProvider::class,
          Illuminate\Auth\AuthServiceProvider::class,
          Illuminate\Broadcasting\BroadcastServiceProvider::class,
        ...
    'Way\Generators\GeneratorsServiceProvider',
    'Xethron\MigrationsGenerator\MigrationsGeneratorServiceProvider'
]
```

## composer update 命令

`composer update` 命令是一种简单而强大的方式，确保一切都在适当的位置，并且没有错误。运行此命令后，我们现在准备运行迁移。

## 生成迁移

只需输入以下命令：

```php
**$ php artisan**

```

`artisan` 命令将显示所有可能的命令列表。`migrate:generate` 命令应该包含在有效命令列表中。如果此命令不在列表中，则说明某些配置不正确。

确认 `migrate:generate` 命令存在于列表中后，只需运行以下命令：

```php
**$ php artisan migrate:generate**

```

这将启动该过程。

在这个例子中，我们使用了 MySQL 数据库。在提示时输入 `Y`，进程将开始，输出应该显示为数据库中的每个表创建了一个迁移文件。

这是您的命令提示符在最后应该显示的样子：

```php
**Using connection: mysql**

**Generating migrations for: accommodations, amenities, amenity_room, cities, countries, currencies, locations, rates, reservation_room, reservations, rooms, states, users**
**Do you want to log these migrations in the migrations table? [Y/n] Y**
**Migration table created successfully.**
**Next Batch Number is: 1\. We recommend using Batch Number 0 so that it becomes the "first" migration [Default: 0]** 
**Setting up Tables and Index Migrations**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_accommodations_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_amenities_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_amenity_room_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_cities_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_countries_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_currencies_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_locations_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_rates_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_reservation_room_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_reservations_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_rooms_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_states_table.php**
**Created: /var/www/laravel.example/database/migrations/2015_02_07_170311_create_users_table.php**

**Finished!**

```

# 迁移解剖

考虑迁移文件中的一行的示例；我们可以看到表对象在一系列方法中使用。迁移文件的以下行设置了位置优雅属性中的状态属性在`locations`表中：

```php
$table->smallInteger('state_id')->unsigned()->index('state_id');
```

## 列出表

通常需要创建或导入通常保持不变的有限项目列表，例如城市、州、国家和类似项目。让我们称这些列表表或查找表。在这些表中，ID 通常应为正数。这些列表可能会增长，但通常不会删除或更新任何数据。`smallInteger`类型用于保持表的小型，并且表示属于有限列表的值，这些值不会自然增长。下一个方法`unsigned`表示限制将为 65535。这个值应该足以表示大多数州、省或类似类型的地理区域，酒店可能位于其中。链中的最后一个方法向数据库列添加索引。这在这样的列表表中是必不可少的，这些列表表用于`select`语句或`read`语句中。`Read`语句将在第九章*扩展 Laravel*中讨论。使用 unsigned 很重要，因为它将正限制加倍，否则将是 32767。使用索引，我们可以加快查找时间并访问表中数据的缓存版本。

## 软删除和时间戳属性

关于列表表的`softDeletes`和`timestamps`，这取决于。如果表不是很大，跟踪更新、插入或删除不会太有害；但是，如果列表包含国家，其中更改不经常发生且非常小，最好省略`softDeletes`和`timestamps`。因此，整个表可能适合内存，并且速度非常快。要省略时间戳，需要添加以下代码行：

```php
public $timestamps = false;
```

# 创建种子

要创建我们的数据库 seeder，我们将修改扩展`Seeder`的`DatabaseSeeder`类。文件的名称是`database/seeds/DatabaseSeeder.php`。文件的内容将如下所示：

```php
<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder {

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        //create a user
        $user = new \MyCompany\User();
        $user->id=1;
        $user->email = "testing@tester.com";
        $user->password = Hash::make('p@ssw0rd');
        $user->save();

        //create a country
        $country = new \MyCompany\Accommodation\Location\State;
        $country->name = "United States";
        $country->id = 236;
        $country->save();

        //create a state
        $state = new \MyCompany\Accommodation\Location\State;
        $state->name = "Pennsylvania";
        $state->id = 1;
        $state->save();

        //create a city
        $city = new \MyCompany\Accommodation\Location\City;
        $city->name = "Pittsburgh";
        $city->save();

        //create a location
        $location = new \MyCompany\Accommodation\Location;
        $location->city_id = $city->id;
        $location->state_id = $state->id;
        $location->country_id = 236;
        $location->latitude = 40.44;
        $location->longitude = 80;
        $location->code = '15212';
        $location->address_1 = "100 Main Street";
        $location->save();

        //create a new accommodation
        $accommodation = new \MyCompany\Accommodation;
        $accommodation->name = "Royal Plaza Hotel";
        $accommodation->location_id = $location;
        $accommodation->description = "A modern, 4-star hotel";
        $accommodation->save();

        //create a room
        $room1 = new \MyCompany\Accommodation\Room;
        $room1->room_number= 'A01';
        $room1->accommodation_id = $accommodation->id;
        $room1->save();

        //create another room
        $room2 = new \MyCompany\Accommodation\Room;
        $room2->room_number= 'A02';
        $room2->accommodation_id = $accommodation->id;
        $room2->save();

        //create the room array
        $rooms = [$room1,$room2];

    }

}
```

seeder 文件设置了可能的最基本的场景。对于初始测试，我们不需要将每个国家、州、城市和可能的位置都添加到数据库中；我们只需要添加必要的信息来创建各种场景。例如，要创建一个新的预订；我们将创建每个用户、国家、州、城市、位置和住宿模型的实例，然后创建两个房间，这些房间将添加到房间数组中。

让我们为预订创建一个实现非常简单的存储库接口的存储库：

```php
<?php

namespace MyCompany\Accommodation;

interface RepositoryInterface {
    public function create($attributes);
}
```

现在让我们创建`ReservationRepository`，它实现`RepositoryInterface`：

```php
<?php

namespace MyCompany\Accommodation;

class ReservationRepository implements RepositoryInterface {
    private $reservation;

    function __construct($reservation)
    {
        $this->reservation = $reservation;
    }

    public function create($attributes)
    {
        $this->reservation->create($attributes);
        return $this->reservation;
    }
}
```

现在，我们将创建所需的方法来创建预订，并填充`reservation_room`的中间表：

```php
public function create($attributes)
{

    $modelAttributes= array_except($attributes, ['rooms']);

    $reservation = $this->reservationModel->create($modelAttributes);
    if (isset($attributes['rooms']) ) {
        $reservation->rooms()->sync($attributes['rooms']);
    }
    return $reservation;
}
```

### 提示

`array_except()` Laravel 助手用于返回`attributes`数组，除了`$rooms`数组之外，该数组将用于`sync()`函数。

在这里，我们将模型的每个属性设置为方法中设置的属性。我们需要添加将建立预订和房间之间多对多关系的方法：

```php
public function rooms(){
    return $this->belongsToMany('MyCompany\Accommodation\Room')->withTimestamps();
}
```

在这种情况下，我们需要向关系添加`withTimestamps()`，以便时间戳将被更新，指示关系何时保存在`reservation_room`中。

# 使用 PHPUnit 进行数据库测试

PHPUnit 与 Laravel 5 集成良好，就像与 Laravel 4 一样，因此设置测试环境相当容易。测试的一个好方法是使用 SQLite 数据库，并将其设置为驻留在内存中，但是您需要修改`config/database.php`文件，如下所示：

```php
    'default' => 'sqlite',
       'connections' => array(
        'sqlite' => array(
            'driver'   => 'sqlite',
            'database' => ':memory:',
            'prefix'   => '',
        ),
    ),
```

然后，我们需要修改`phpunit.xml`文件以设置`DB_DRIVER`环境变量：

```php
<php>
        <env name="APP_ENV" value="testing"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="DB_DRIVER" value="sqlite"/>
</php>
```

然后，我们需要修改`config/database.php`文件中的以下行：

```php
'default' => 'mysql',
```

我们修改前面的行以匹配以下行：

```php
'default' => env('DB_DRIVER', 'mysql'),
```

现在，我们将设置 PHPUnit 在内存中的`sqlite`数据库上运行我们的迁移。

在`tests`目录中，有两个类：一个`TestCase`类，继承了`LaravelTestCase`类，和一个`ExampleTest`类，继承了`TestCase`类。

我们需要向`TestCase`添加两个方法来执行迁移，运行 seeder，然后将数据库恢复到其原始状态：

```php
<?php

class TestCase extends Illuminate\Foundation\Testing\TestCase {

    public function setUp()
    {
        parent::setUp();
        Artisan::call('migrate');
        Artisan::call('db:seed');
    }

    /**
    * Creates the application.
    *
    * @return \Illuminate\Foundation\Application
    */
    public function createApplication()
    {
        $app = require __DIR__.'/../bootstrap/app.php';
        $app->make('Illuminate\Contracts\Console\Kernel')->bootstrap();
        return $app;
    }

    public function tearDown()
    {
        Artisan::call('migrate:rollback');
    }
}
```

现在，我们将创建一个 PHPUnit 测试来验证数据是否正确保存在数据库中。我们需要将`tests/ExampleTest.php`修改为以下代码：

```php
<?php

class ExampleTest extends TestCase {

    /**
    * A basic functional test example.
    *
    * @return void
    */

public function testReserveRoomExample()
    {

        $reservationRepository = new \MyCompany\Accommodation\ReservationRepository(
            new \MyCompany\Accommodation\Reservation());
        $reservationValidator = new \MyCompany\Accommodation\ReservationValidator();
        $start_date = '2015-10-01';
        $end_date = '2015-10-10';
        $rooms = \MyCompany\Accommodation\Room::take(2)->lists('id')->toArray();
        if ($reservationValidator->validate($start_date,$end_date,$rooms)) {
            $reservation = $reservationRepository->create(['date_start'=>$start_date,'date_end'=>$end_date,'rooms'=>$rooms,'reservation_number'=>'0001']);
        }

        $this->assertInstanceOf('\MyCompany\Accommodation\Reservation',$reservation);
        $this->assertEquals('2015-10-01',$reservation->date_start);
        $this->assertEquals(2,count($reservation->rooms));
}
```

## 运行 PHPUnit

要启动 PHPUnit，只需输入以下命令：

```php
**$ phpunit**

```

测试将会运行。由于`Reservation`类的`create`方法返回一个预订，我们可以使用 PHPUnit 的`assertInstanceOf`方法来确定数据库中是否创建了预订。我们可以添加任何其他断言来确保保存的值正是我们想要的。例如，我们可以断言开始日期等于`'2015-10-01'`，`room`数组的大小等于`two`。与`testBasicExample()`方法一起，我们可以确保对`"/"`的`GET`请求返回`200`。PHPUnit 的结果将如下所示：

运行 PHPUnit

请注意，有两个点表示测试。**OK**表示没有失败，我们再次被告知有两个测试和四个断言；一个是在示例中的断言，另外三个是我们添加到`testReserveRoomExample`测试中的。如果我们测试了三个房间而不是两个，PHPUnit 将产生以下输出：

```php
**$ phpunit**
**PHPUnit 4.5.0 by Sebastian Bergmann and contributors.**

**Configuration read from /var/www/laravel.example/phpunit.xml**

**.**
**F**

**Time: 1.59 seconds, Memory: 10.75Mb**

**There was 1 failure:**

**1) ExampleTest::testReserveRoomExample**
**Failed asserting that 2 matches expected 3.**

**/var/www/laravel.example/tests/ExampleTest.php:24**

**FAILURES!** 
**Tests: 2, Assertions: 4, Failures: 1.**

```

请注意，我们有一个`F`表示失败，而不是第二个点，而不是`OK`，我们被告知有`1`个失败。然后 PHPUnit 列出了哪些测试失败，并很好地告诉我们我故意修改为不正确的行。

```php
   $this->assertEquals(3,count($reservationResult->rooms));
```

前面的行确实是不正确的：

```php
**Failed asserting that 2 matches expected 3.**

```

请记住，`2`是`($reservationResult->rooms)`的计数值。

# 使用 Behat 进行功能测试

虽然 phpspec 遵循 BDD 的规范，并且在隔离中很有用于规范和设计，但它的补充工具 Behat 用于集成和功能测试。由于 phpspec 建议对所有内容进行模拟，数据库查询实际上不会被执行，因为数据库在该方法的上下文之外。Behat 是一个在某个功能上执行行为测试的好工具。虽然 phpspec 已经包含在 Laravel 5 的依赖项中，但 Behat 将作为外部模块安装。

应该运行以下命令来安装并使 Behat 与 Laravel 5 一起工作：

```php
**$ composer require behat/behat behat/mink behat/mink-extension laracasts/behat-laravel-extension --dev**

```

运行 composer update 后，Behat 的功能将添加到 Laravel 中。接下来，应在 Laravel 项目的根目录中添加一个`behat.yaml`文件，以指定要使用哪些扩展。

接下来，运行以下命令：

```php
**$ behat --init**

```

这将创建一个`features`目录，里面有一个`bootstrap`目录。还将创建一个`FeaturesContext`类。`bootstrap`中的所有内容都将在每次运行`behat`时运行。这对于自动运行迁移和填充是有用的。

`features/bootstrap/FeaturesContext.php`文件如下：

```php
<?php

use Behat\Behat\Context\Context;
use Behat\Behat\Context\SnippetAcceptingContext;
use Behat\Gherkin\Node\PyStringNode;
use Behat\Gherkin\Node\TableNode;

/**
 * Defines application features from the specific context.
 */
class FeatureContext implements Context, SnippetAcceptingContext
{
    /**
     * Initializes context.
     *
     * Every scenario gets its own context instance.
     * You can also pass arbitrary arguments to the
     * context constructor through behat.yml.
     */
    public function __construct()
    {
    }
}
```

接下来，`FeatureContext`类需要扩展`MinkContext`类，因此类定义行需要修改如下：

```php
class FeatureContext implements Context, SnippetAcceptingContext
```

接下来，将在类中添加`prepare`和`cleanup`方法以执行迁移。我们将添加`@BeforeSuite`和`@AfterSuite`注释，告诉 Behat 在每个套件之前执行迁移和种子，并在每个套件之后回滚以将数据库恢复到其原始状态。将在文档块中使用注释将在第六章中讨论，*使用注释驯服复杂性*。我们的类现在结构如下：

```php
<?php

use Behat\Behat\Context\Context;
use Behat\Behat\Context\SnippetAcceptingContext;
use Behat\Gherkin\Node\PyStringNode;
use Behat\Gherkin\Node\TableNode;

/**
 * Defines application features from the specific context.
 */
class FeatureContext implements Context, SnippetAcceptingContext
{
    /**
     * Initializes context.
     *
     * Every scenario gets its own context instance.
     * You can also pass arbitrary arguments to the
     * context constructor through behat.yml.
     */
    public function __construct()
    {
    }
     /**
     * @BeforeSuite
     */
     public static function prepare(SuiteEvent $event)
     {
        Artisan::call('migrate');
        Artisan::call('db:seed');

     }

     /**
     * @AfterSuite 
     */
     public function cleanup(ScenarioEvent $event)
     {
        Artisan::call('migrate:rollback');
     }
}
```

现在，需要创建一个功能文件。在 room 目录中创建`reservation.feature`：

```php
Feature: Reserve Room
  In order to verify the reservation system
  As an accommodation reservation user
  I need to be able to create a reservation in the system
  Scenario: Reserve a Room
   When I create a reservation
         Then I should have one reservation
```

当运行`behat`如下时：

```php
**$ behat**

```

产生以下输出：

```php
**Feature: Reserve Room**
 **In order to verify the reservation system**
 **As an accommodation reservation user**
 **I need to be able to create a reservation in the system**

 **Scenario: List 2 files in a directory # features/reservation.feature:5**
 **When I create a reservation**
 **Then I should have one reservation**

**1 scenario (1 undefined)**
**2 steps (2 undefined)**
**0m0.10s (7.48Mb)**

**--- FeatureContext has missing steps. Define them with these snippets:**

 **/****
 *** @When I create a reservation**
 ***/**
 **public function iCreateAReservation()**
 **{**
 **throw new PendingException();**
 **}**

 **/****
 *** @Then I should have one reservation**
 ***/**
 **public function iShouldHaveOneReservation()**
 **{**
 **throw new PendingException();**
 **}**

```

Behat，就像 phpspec 一样，熟练地生成输出，向您显示需要创建的方法。请注意，此处使用驼峰命名法而不是蛇形命名法。此代码应复制到`FeatureContext`类中。请注意，默认情况下会抛出异常。

在这里，将调用 RESTful API，因此需要将 guzzle HTTP 包添加到项目中：

```php
**$ composer require guzzlehttp/guzzle**

```

接下来，向类添加一个属性来保存`guzzle`对象。我们将向 RESTful 资源控制器添加一个`POST`请求来创建预订，并期望获得 201 代码。请注意，返回代码是一个字符串，需要转换为整数。接下来，执行`get`以返回所有预订。

应该只创建一个预订，因为迁移和种子每次运行时都会运行：

```php
<?php

use Behat\Behat\Context\Context;
use Behat\Behat\Context\SnippetAcceptingContext;
use Behat\Gherkin\Node\PyStringNode;
use Behat\Gherkin\Node\TableNode;
use Behat\MinkExtension\Context\MinkContext;
use Behat\Testwork\Hook\Scope\BeforeSuiteScope;
use Behat\Testwork\Hook\Scope\AfterSuiteScope;
use GuzzleHttp\Client;

/**
 * Defines application features from the specific context.
 */
class FeatureContext extends MinkContext implements Context, SnippetAcceptingContext
{
    /**
     * Initializes context.
     *
     * Every scenario gets its own context instance.
     * You can also pass arbitrary arguments to the
     * context constructor through behat.yml.
     */
    protected $httpClient;

    public function __construct()
    {
        $this->httpClient = new Client();
    }
    /**
     * @BeforeSuite
     */
    public static function prepare(BeforeSuiteScope $scope)
    {
        Artisan::call('migrate');
        Artisan::call('db:seed');

    }

    /**
     * @When I create a reservation
     */
    public function iCreateAReservation()
    {
        $request = $this->httpClient->post('http://laravel.example/reservations',['body'=> ['start_date'=>'2015-04-01','end_date'=>'2015-04-04','rooms[]'=>'100']]);
        if ((int)$request->getStatusCode()!==201)
        {
            throw new Exception('A successfully created status code must be returned');
        }
    }

    /**
     * @Then I should have one reservation
     */
    public function iShouldHaveOneReservation()
    {
        $request = $this->httpClient->get('http://laravel.example/reservations');
        $arr = json_decode($request->getBody());
        if (count($arr)!==1)
        {
            throw new Exception('there must be exactly one reservation');
        }
    }

    /**
     * @AfterSuite
     */
    public static function cleanup(AfterSuiteScope $scope)
    {
        Artisan::call('migrate:rollback');
    }
}

    /**
     * @When I create a reservation
     */
    public function iCreateAReservation()
    {
        $request = $this->httpClient->post('http://laravel.example/reservations',['body'=> ['start_date'=>'2015-04-01','end_date'=>'2015-04-04','rooms[]'=>'100']]);
        if ((int)$request->getStatusCode()!==201)
        {
            throw new Exception('A successfully created status code must be returned');
        }
    }
```

现在，使用命令行中的 artisan 来创建`ReservationController`：

```php
**$ php artisan make:controller ReservationsController**

```

以下是预订控制器的内容：

```php
<?php namespace MyCompany\Http\Controllers;

use MyCompany\Http\Requests;
use MyCompany\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
use MyCompany\Accommodation\ReservationRepository;
use MyCompany\Accommodation\ReservationValidator;
use MyCompany\Accommodation\Reservation;

class ReservationsController extends Controller {

    /**
    * Display a listing of the resource.
    *
    * @return Response
    */
    public function index()
    {
        return Reservation::all();
    }

    /**
    * Store a newly created resource in storage.
    *
    * @return Response
    */
    public function store()
    {
        $reservationRepository = new ReservationRepository(new Reservation());
        $reservationValidator = new ReservationValidator();
        if ($reservationValidator->validate(\Input::get('start_date'),
        \Input::get('end_date'),\Input::get('rooms')))
        {
        $reservationRepository->create(['date_start'=>\Input::get('start_date'),'date_end'=>\Input::get('end_date'),'rooms'=>\Input::get('rooms')]);
        return response('', '201');
        }
    }
}
```

最后，将`ReservationController`添加到`routes.php`文件中，该文件位于`app/Http/routes.php`中：

```php
**Route::resource('reservations','ReservationController');**

```

现在，当运行`behat`时，结果如下：

```php
**Feature: Reserve Room**
 **In order to verify the reservation system**
 **As an accommodation reservation user**
 **I need to be able to create a reservation in the system**

 **Scenario: Reserve a Room**
 **When I create a reservation         # FeatureContext::iCreateAReservation()**
 **Then I should have one reservation  # FeatureContext::iShouldHaveOneReservation()**

**1 scenario (1 passed)**
**2 steps (2 passed)**

```

# 总结

配置 Laravel 以从现有模式创建迁移文件也是非全新项目的一个有用框架。通过在测试环境中运行迁移和种子，每个测试都可以从数据库的完全干净版本中受益，并且可以通过初始数据最小地验证软件的执行是否符合需要。当需要将遗留代码移植到 Laravel 时，PHPUnit 可以用于测试任何现有功能。Behat 提供了一种基于行为的替代方案，可以熟练地执行端到端测试。

我们使用 phpspec 在一个独立的环境中设计了我们的类，只专注于业务规则和客户端的请求，同时模拟诸如实际实体（如房间）之类的事物。然后，我们通过使用功能测试工具 PHPUnit 验证了实际查询是否正确执行并保存在数据库中。最后，我们使用 Behat 执行端到端测试。

在下一章中，我们将看到 RESTful API 的创建，基本的 CRUD 操作（创建，读取，更新和删除），并讨论一些最佳实践。
