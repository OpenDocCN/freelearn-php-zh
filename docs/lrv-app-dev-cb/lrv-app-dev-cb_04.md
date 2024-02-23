# 第四章。存储和使用数据

在本章中，我们将涵盖：

+   使用迁移和模式创建数据表

+   使用原始 SQL 语句查询

+   使用 Fluent 查询

+   使用 Eloquent ORM 查询

+   在模型中使用自动验证

+   使用高级 Eloquent 和关系

+   创建 CRUD 系统

+   使用 Eloquent 导入 CSV

+   使用 RSS 作为数据源

+   使用属性更改表列名称

+   在 Laravel 中使用非 Eloquent ORM

# 介绍

任何 Web 应用程序的支柱之一是使用和操作数据。Laravel 提供了许多方便的方法来与数据库交互并显示它们的信息。在本章中，我们将从一些简单的数据库交互开始。然后，我们将使用其他非数据库作为我们的数据源，然后对我们的 Laravel 应用程序进行一些自定义。

# 使用迁移和模式创建数据表

使用 Laravel，我们可以轻松地使用模式和迁移创建我们的数据模型。在这个配方中，我们将看到 Laravel 如何实现这些基本功能。

## 准备工作

对于这个配方，我们需要一个标准的 Laravel 安装，以及在我们的数据库配置文件中配置的 MySQL 数据库。

## 如何做...

要完成这个配方，请按照以下步骤进行：

1.  使用`artisan`从命令提示符中安装我们的迁移表：

```php
**php artisan migrate:install**

```

1.  创建一个迁移以保存我们的模式代码来创建一个新表：

```php
**php artisan migrate:make create_shows_table**

```

1.  在我们的`app/database/migrations`目录中，找到一个名为`2012_01_01_222551_create_shows_table.php`的类似文件。添加用于创建表和添加列的模式：

```php
class CreateShowsTable extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('shows', function($table)
        {
            $table->increments('id');
            $table->string('name', 140);
            $table->integer('rating')->nullable();
            $table->timestamps();
        });
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('shows');
    }
}
```

1.  运行迁移以将表添加到数据库，使用以下命令：

```php
**php artisan migrate**

```

1.  创建另一个迁移，以便我们可以向我们的表中添加一列：

```php
**php artisan migrate:make add_actor_to_shows_table**

```

1.  在`app/database/migrations`目录中，找到一个类似于`2012_01_01_222551_add_actor_to_shows_table.php`的文件。向我们的模式中添加列：

```php
class AddActorToShowsTable extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('shows', function($table)
        {
            $table->string('actor')->nullable();
        });
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('shows', function($table)
        {
            $table->drop_column('actor');
        });
    }
}
```

1.  在命令提示符中运行迁移，以向我们的表中添加列：

```php
**php artisan migrate**

```

## 它是如何工作的...

使用 Laravel 的 Artisan 命令行工具，运行命令来创建一个迁移表。这将跟踪我们进行的任何迁移和模式更改。然后我们使用 Artisan 创建一个将保存我们`shows`表模式的迁移文件。

在`shows`模式中，我们创建一个简单的表来保存电视节目的列表以及我们对它们的评分。节目的名称设置为字符串，评分设置为整数，并且我们使用 Laravel 的默认机制来创建时间戳。当我们运行迁移时，我们的表将被创建。

如果我们决定要在表中添加另一列，我们只需使用 Artisan 创建另一个迁移文件。在这种情况下，我们将添加一个列来保存演员的姓名。我们的模式将获取我们已经创建的表，并向其添加列。当我们重新运行迁移时，数据库中的所有内容都将得到更新。

## 还有更多...

我们还可以使用 Artisan 的一些命令行开关为我们创建一些更多的样板代码。例如，要创建 shows 表，我们可以运行以下命令：

```php
**php artisan migrate:make create_shows_table –table=show –create**

```

运行该命令将生成一个包含以下代码的迁移文件：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateShowsTable extends Migration {

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('shows', function(Blueprint $table)
        {
            $table->increments('id');
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
        Schema::drop('shows');
    }

}
```

# 使用原始 SQL 语句查询

Laravel 提供了许多访问数据库的方式。如果我们有以前使用过的现有查询，或者如果我们需要一些更复杂的东西，我们可以使用原始 SQL 来访问我们的数据库。

## 准备工作

对于这个配方，我们将使用*使用迁移和模式创建数据表*配方中创建的表。

## 如何做...

要完成这个配方，请按照以下步骤进行：

1.  在命令提示符中，创建一个迁移，以便我们可以添加一些数据：

```php
**php artisan migrate:make add_data_to_shows_table**

```

1.  在我们的`app/database/migrations`目录中，找到一个类似于`2012_01_01_222551_add_data_to_shows_table.php`的文件，并使用原始 SQL 添加一些数据：

```php
class AddDataToShowsTable {

    /**
     * Make changes to the database.
     *
     * @return void
     */

public function up()
    {
        $sql = 'INSERT INTO shows (name, rating, actor)
            VALUES (?, ?, ?)';
        $data1 = array('Doctor Who', '9', 'Matt Smith');
        $data2 = array('Arrested Development', '10', 'Jason
            Bateman');
        $data3 = array('Joanie Loves Chachi', '3', 'Scott
            Baio');
        DB::insert($sql, $data1);
        DB::insert($sql, $data2);
        DB::insert($sql, $data3);
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        $sql = "DELETE FROM shows WHERE name = ?";
        DB::delete($sql, array('Doctor Who'));
        DB::delete($sql, array('Arrested Development'));
        DB::delete($sql, array('Joanie Loves Chachi'));
    }
}
```

1.  在命令提示符中运行迁移以添加数据：

```php
**php artisan migrate**

```

1.  在我们的`app/models`目录中，创建一个名为`Show.php`的文件，并添加一个获取节目的方法：

```php
class Show {
    public function allShows($order_by = FALSE,$direction = 'ASC')
    {
        $sql = 'SELECT * FROM shows';
        $sql .= $order_by ? ' ORDER BY ' . $order_by. ' ' . $direction : '';
        return DB::select($sql);
    }
}
```

1.  在我们的`routes.php`文件中，创建一个`Show`路由来显示模型中的信息：

```php
Route::get('shows', function()
{
    $shows = new Show();
    $shows_by_rating = $shows->allShows('rating', 'DESC');
    dd($shows_by_rating);
}); 
```

## 它是如何工作的...

为了在我们的`shows`表中填充一些数据，我们首先需要使用 Artisan 命令行工具创建一个迁移。在迁移文件的`up`方法中，我们创建一个简单的 SQL 插入命令，并传入三个参数。然后我们创建三个数组，数组中的值与查询中的列的顺序相同。然后我们将 SQL 语句变量和值数组传递给 Laravel 的`DB::insert()`命令。对于我们的`down`方法，我们使用了一个 SQL 删除语句，通过节目名称进行搜索。一旦我们运行迁移，我们的数据将填充到表中。

接下来，我们在前端创建一个与数据库交互的模型。我们的模型有一个方法来显示表中的所有节目，还可以使用可选参数来重新排序它们的显示方式。

我们的路由实例化 Show 模型并运行`allShows()`方法。为了显示结果，我们使用 Laravel 的`dd()`辅助函数。在这一点上，我们可以将数据传递到视图中，并循环遍历以显示。

## 另请参阅

+   *使用迁移和模式创建数据表*示例

# 使用 Fluent 进行查询

Laravel 提供了许多访问数据库的方式。如果我们选择不编写原始的 SQL 语句，我们可以使用 Fluent 查询构建器来简化操作。

## 准备工作

对于这个示例，我们将使用在*使用迁移和模式创建数据表*示例中创建的表。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在命令提示符中，创建一个迁移以便我们添加一些数据：

```php
**php artisan migrate:make add_data_to_shows_table**

```

1.  在我们的`app/database/migrations`目录中，找到一个类似于`2012_01_01_222551_add_data_to_shows_table.php`的文件，并使用 Fluent 查询构建器添加一些数据：

```php
class AddDataToShowsTable {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        $data1 = array('name' => 'Doctor Who',
            'rating' => 9, 'actor' => 'Matt Smith');
        $data2 = array('name' => 'Arrested Development',
            'rating' => 10, 'actor' => 'Jason Bateman');
        $data3 = array('name' => 'Joanie Loves Chachi',
            'rating' => 3, 'actor' => 'Scott Baio');
        DB::table('shows')->insert(array($data1, $data2,
            $data3));
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        DB::table('shows')
            ->where('name', 'Doctor Who')
            ->orWhere('name', 'Arrested Development')
            ->orWhere('name', 'Joanie Loves Chachi')
            ->delete();
    }
}
```

1.  运行迁移以添加数据：

```php
**php artisan migrate**

```

1.  在我们的`app/models`目录中，创建一个名为`Show.php`的文件，并添加一个获取节目的方法：

```php
class Show {
    public function allShows($order_by = FALSE,$direction = 'ASC')
    {
        $shows = DB::table('shows');
        return $order_by ? $shows->order_by($order_by,$direction)->get() : $shows->get();
    }
}
```

1.  在我们的`routes.php`文件中，创建一个`Show`路由来显示模型中的信息：

```php
Route::get('shows', function()
{
    $shows = new Show();
    $shows_by_rating = $shows->allShows('rating', 'DESC');
    dd($shows_by_rating);
}); 
```

## 它是如何工作的...

为了在我们的`shows`表中填充一些数据，我们首先需要使用 Artisan 命令行工具创建一个迁移。在迁移文件的`up`方法中，我们创建三个数组来保存我们的值，使用列名作为键。然后将这些数组放入一个数组中，并传递给 Fluent 的`insert`函数。`down`方法使用`where()`和`orWhere()`函数来定位记录的名称，并删除它们。一旦我们运行迁移，我们的数据将填充到表中。

接下来，我们在前端创建一个与数据库交互的模型。我们的模型有一个方法来显示表中的所有节目，还可以使用可选参数来重新排序它们的显示方式。

我们的路由实例化 Show 模型并运行`allShows()`方法。为了显示结果，我们使用 Laravel 的`dd()`辅助函数。我们也可以创建一个视图，并将数据传递到那里进行循环。

## 还有更多...

在 Laravel 的文档中可以找到更多流畅的方法[`laravel.com/docs/queries`](http://laravel.com/docs/queries)。

## 另请参阅

+   *使用迁移和模式创建数据表*示例

# 使用 Eloquent ORM 进行查询

Laravel 提供了许多与数据库交互的方式。其中最简单的一种方式是使用 Eloquent ORM。它提供了一种简单直观的方式来处理数据。

## 准备工作

对于这个示例，我们将使用在*使用迁移和模式创建数据表*示例中创建的表。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在命令提示符中，创建一个迁移以便我们添加一些数据：

```php
**php artisan migrate:make add_data_to_shows_table**

```

1.  在我们的`app/database/migrations`目录中，找到一个类似于`2012_01_01_222551_add_data_to_shows_table.php`的文件，并使用 Fluent 查询构建器添加一些数据：

```php
class AddDataToShowsTable {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        $data1 = array('name' => 'Doctor Who',
            'rating' => 9, 'actor' => 'Matt Smith');
        $data2 = array('name' => 'Arrested Development',
            'rating' => 10, 'actor' => 'Jason Bateman');
        $data3 = array('name' => 'Joanie Loves Chachi',
            'rating' => 3, 'actor' => 'Scott Baio');
        DB::table('shows')->insert(array($data1, $data2,
            $data3));
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        DB::table('shows')
            ->where('name', 'Doctor Who')
            ->orWhere('name', 'Arrested Development')
            ->orWhere('name', 'Joanie Loves Chachi')
            ->delete();
    }
}
```

1.  运行迁移以添加数据：

```php
**php artisan migrate**

```

1.  在我们的`app/models`目录中，创建一个名为`Show.php`的文件，它继承自`Eloquent`：

```php
class Show extends Eloquent{
    public function getTopShows() {
        return $this->where('rating', '>', 5)->orderBy('rating', 'DESC')->get();
    }
}
```

1.  在我们的`routes.php`文件中，创建一个 show 路由来显示模型中的信息：

```php
Route::get('shows', function()
{
    $shows = Show::all();
    echo '<h1>All Shows</h1>';
    foreach ($shows as $show)
    {
        echo $show->name . ' - ' . $show->rating . ' - '
		    . $show->actor . '<br>';
    }

    $show_object = new Show();
    $top_shows = $show_object->getTopShows();
    echo '<h1>Top Shows</h1>';
    foreach ($top_shows as $top_show)
    {
        echo $top_show->name . ' - ' . $top_show->rating
		     . ' - '. $top_show->actor . '<br>';
    }
});
```

## 它是如何工作的...

要在我们的`shows`表中填充一些数据，我们首先需要使用 Artisan 命令行工具创建一个迁移。在迁移文件的`up`方法中，我们创建了三个包含我们的值的数组，使用列名作为键。然后，这些数组被放入一个数组中，并传递给 Fluent 的`insert`函数。`down`方法使用`where()`和`orWhere()`函数通过它们的名称来定位记录，并删除它们。一旦我们运行迁移，我们的数据将填充到表中。

接下来，我们创建一个模型来在前端与数据库交互。对于这个示例，我们只需要扩展`Eloquent`，ORM 将自动处理其他所有事情。我们还添加了一个方法，它将返回所有顶级节目。

我们的路由调用了 Show ORM 对象的`all()`方法；这将把所有数据放入`$shows`变量中。然后我们通过记录进行简单的循环，并显示我们想要的字段。接下来，我们通过在 Show 模型中调用方法来获取一个经过筛选的列表，只获取评分大于 5 的记录，并按评分排序。

## 还有更多...

在这个示例中，我们在路由中显示所有数据。理想情况下，我们会将数据传递到视图中并在那里显示。

## 另请参阅

+   *使用迁移和模式创建数据表*示例

# 在模型中使用自动验证

在验证发送到数据库的数据时，理想情况下我们应该将规则和验证放在我们的模型中。在这个示例中，我们将看到一种实现这一点的方法。

## 准备工作

对于这个示例，我们需要一个配置了 MySQL 数据库的标准 Laravel 安装。我们还需要通过运行 Artisan 命令`php artisan migrate:install`来设置我们的迁移表。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在命令提示符中，创建一个简单的`users`表的迁移：

```php
**php artisan migrate:make create_users_table**

```

1.  在迁移文件中创建模式。该文件位于`app/database/migrations`目录中，名称类似于`2012_01_01_222551_create_users_table.php`：

```php
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function($table)
        {
            $table->increments('id');
            $table->string('username', 100);
            $table->string('email', 100);
            $table->timestamps();
        });
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('users');
    }
}
```

1.  运行迁移：

```php
**php artisan migrate**

```

1.  在我们的`app/models`目录中创建一个名为`User.php`的文件。如果已经有一个名为`User.php`的文件，我们可以简单地重命名它：

```php
<?php
class User extends Eloquent {

    protected $table = 'users';

    private $rules = array(
        'email' => 'required|email',
        'username' => 'required|min:6'
    );

    public function validate($input) {
        return Validator::make($input, $this->rules);
    }
}
```

1.  创建一个加载 ORM 并尝试保存一些数据的路由：

```php
$user = new User();
    $input = array();

    $input['email'] = 'racerx@example.com';
    $input['username'] = 'Short';
    $valid = $user->validate($input);
    if ($valid->passes()) {
        echo 'Everything is Valid!';
        // Save to the database
    } else {
        var_dump($valid->messages());
    }
```

## 它是如何工作的...

首先，我们为一个基本的`users`表创建一个迁移。在我们的模式中，我们设置了一个带有 ID、用户名、电子邮件 ID 和时间戳的表。然后运行迁移，在数据库中创建表。

接下来，我们设置了我们的用户模型并扩展了`Eloquent`。我们需要创建我们的规则，使用一个名为`$rules`的私有变量，其中包含我们想要检查的验证规则数组。在我们的模型中，我们创建了一个`validate`方法。这将通过 Laravel 的`Validator`运行我们的输入，使用我们刚刚设置的规则。

在我们的路由中，我们创建了一个新用户并添加了一些值。在保存之前，我们通过`validate`方法运行输入；如果失败，我们可以循环遍历验证错误消息。如果通过，我们可以将输入保存到我们的数据库中。

## 还有更多...

还有一些其他验证数据的方法。一种方法是使用一个可以为我们处理大部分验证工作的包。一个很好的包是 Ardent，可以在[`github.com/laravelbook/ardent`](https://github.com/laravelbook/ardent)找到。

# 使用高级 Eloquent 和关系

使用 Laravel 的 Eloquent ORM 的一个很棒的地方是，我们可以轻松地与具有外键和中间表的多个表进行交互。在这个示例中，我们将看到设置我们的模型并针对连接的表运行查询有多么容易。

## 准备工作

对于这个示例，我们将使用在之前的示例*使用迁移和模式创建数据表*和*在模型中使用自动验证*中创建的`shows`和`users`表。

## 如何做...

要完成这个示例，请按照以下步骤进行：

1.  在命令提示符中，创建一个新的中间表的迁移：

```php
**php artisan migrate:make create_show_user**

```

1.  打开`app/database/migrations`目录中的迁移文件，并添加模式：

```php
use Illuminate\Database\Migrations\Migration;

class CreateShowUser extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('show_user', function($table)
        {
            $table->increments('id');
            $table->integer('user_id');
            $table->integer('show_id');
            $table->timestamps();
        });
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('show_user');
    }
}
```

1.  运行迁移：

```php
**php artisan migrate**

```

1.  在`app/model`目录中创建一个`User.php`文件：

```php
class User extends Eloquent {
    public function shows()
    {
        return $this->belongsToMany ('Show');
    }
}
```

1.  在我们的`app/model`目录中创建一个`Show.php`文件：

```php
class Show extends Eloquent {
    public function users()
    {
        return $this->belongsToMany ('User');
    }
}
```

1.  在`routes.php`中创建一个路由来添加一个新用户并附加两个节目：

```php
Route::get('add-show', function()
{
    // Create a new User
    $user = new User();
    $user->username = 'John Doe';
    $user->email = 'johndoe@example.com';
    $user->save();

    // Attach two Shows
    $user->shows()->attach(1);
    $user->shows()->attach(3);

    foreach($user->shows()->get() as $show) {
        var_dump($show->name);
    }
});
```

1.  创建一个路由来获取与一个节目关联的所有用户：

```php
Route::get('view-show', function()
{
    $show = Show::find(1)->users;
    dd($show);
});
```

## 它是如何工作的...

我们的第一个任务是创建一个将我们的`users`表与我们的`shows`表连接的中间表。在我们的迁移模式中，我们需要为我们的`user_id`和`show_id`添加列。然后我们运行迁移，以在我们的数据库中设置表。

为了设置我们的模型，我们需要创建一个函数，该函数将返回我们的多对多关系。在我们的 User 模型中，我们创建了指向我们 Show 模型的关系的`shows()`函数。在 Show 模型中，我们创建了一个名为`users()`的函数，指向我们的 User 模型。有了这个设置，我们现在可以轻松地对两个表运行查询。

接下来，我们创建一个路由来添加一个新用户。一旦我们保存了用户，我们使用`attach()`方法来创建与节目的关系，并传入我们想要附加的节目的 ID。之后，如果我们查看我们的`show_user`表，我们会看到两条记录——一条是我们新用户的 ID 和节目 ID`1`，另一条是节目 ID`3`。通过在我们的路由中运行`get()`方法，我们可以循环遍历记录，并查看哪些节目名称与我们的用户关联。

我们的下一个路由将获取与一个节目关联的所有用户。在我们的情况下，我们获取 ID 为`1`的节目，然后获取所有用户。使用 Laravel 的`dd()`助手，我们可以看到我们的结果。

## 还有更多...

数据库关系可能变得非常复杂，这个教程只是初步介绍了一些操作。要了解更多关于 Laravel 的 Eloquent ORM 如何使用关系，请查看文档[`laravel.com/docs/eloquent#many-to-many`](http://laravel.com/docs/eloquent#many-to-many)。

# 创建 CRUD 系统

为了与我们的数据库交互，我们可能需要创建一个 CRUD（创建、读取、更新和删除）系统。这样，我们可以添加和修改数据，而不需要单独的数据库客户端。这个教程将使用一个 RESTful 控制器来实现我们的 CRUD 系统。

## 准备工作

对于这个教程，我们将在*在模型中使用自动验证*的教程中创建的 User 表上进行扩展。

## 如何操作...

要完成这个教程，请按照以下步骤进行：

1.  在`app/controllers`目录中，创建一个名为`UsersController.php`的文件，并添加以下代码：

```php
<?php

class UsersController extends BaseController {

    public function getIndex()
    {
        $users = User::all();
        return View::make('users.index')->with('users',$users);
    }

    public function getCreate()
    {
        return View::make('users.create');
    }

    public function postCreate()
    {
        $user = new User();
        $user->username = Input::get('username');
        $user->email = Input::get('email');
        $user->save();
        return Redirect::to('users');
    }

    public function getRecord($id)
    {
        $user = User::find($id);
        return View::make('users.record')->with('user',$user);
    }

    public function putRecord()
    {
        $user = User::find(Input::get('user_id'));
        $user->username = Input::get('username');
        $user->email = Input::get('email');
        $user->save();
        return Redirect::to('users');
    }

    public function deleteRecord()
    {
        $user = User::find(Input::get('user_id'))->delete();
        return Redirect::to('users');
    }
}
```

1.  在我们的`routes.php`文件中，添加一个指向控制器的路由：

```php
**Route::controller('users', 'UsersController');**

```

1.  在`app/views`目录中，创建一个名为`users`的新目录，在其中创建一个名为`index.php`的文件，并添加以下代码：

```php
<style>
table, th, td {
    border:1px solid #444
}
</style>
<table>
    <thead>
        <tr>
            <th>User ID</th>
            <th>User Name</th>
            <th>Email</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        <?php foreach($users as $user): ?>
            <tr>
                <td><?php echo $user->id ?></td>
                <td><?php echo $user->username ?></td>
                <td><?php echo $user->email ?></td>
                <td>
                    <a href="users/record/<?php echo $user->id ?>">Edit</a> 
                    <form action="users/record"method="post">
                        <input type="hidden" name="_method"value="DELETE">
                        <input type="hidden" name="user_id"value="<?php echo $user->id?>">
                        <input type="submit"value="Delete">
                    </form>
                </td>
            </tr>
        <?php endforeach; ?>
    </tbody>
</table>
<a href="users/create">Add New User</a>
```

1.  在`app/views/users`目录中，创建一个名为`create.php`的新文件，并添加以下表单：

```php
<form action="create" method="post">
    Username:<br>
    <input name="username"><br>
    Email<br>
    <input name="email"><br>
    <input type="submit">
</form>
```

1.  在`app/views/users`目录中，添加一个名为`record.php`的文件，并使用以下表单：

```php
<form action="" method="post">
    <input type="hidden" name="_method" value="put">
    <input type="hidden" name="user_id" value="<?php echo$user->id ?>">
    Username:<br>
    <input name="username" value="<?php echo $user->username ?>"><br>
    Email<br>
    <input name="email" value="<?php echo $user->email?>"><br>
    <input type="submit">
</form>
```

## 它是如何工作的...

在我们的控制器中，我们的方法名称可以以我们想要使用的 HTTP 动词为前缀。然后我们在我们的路由文件中添加路由，使其指向正确的位置。

我们的第一个方法将生成所有用户的列表。我们将用户传递给我们的视图，然后循环遍历它们，并在一个简单的表中显示它们。

在该表下面，我们有一个链接到我们的第二个方法来添加一个新用户。我们的`getRreate()`方法显示一个简单的表单，然后该表单被发布并保存。保存后，我们将被重定向回列表页面。

要编辑一条记录，我们创建一个`getRecord()`方法，该方法获取传递给它的记录的 ID。我们的视图是一个表单，自动填充了传入的 ID 的用户的值。由于我们正在进行更新，我们想要使用`put`动词；为了实现这一点，我们需要一个隐藏字段，名称为`_method`，值为我们想要使用的请求。当表单提交时，Laravel 将其发送到`putRecord()`方法，并更新信息。

最后，要删除一条记录，我们创建一个简单的表单，其中包含隐藏字段名为`_method`和值为`DELETE`。当提交时，Laravel 将其发送到`deleteRecord()`方法，并将用户从数据库中删除。

## 还有更多...

请注意，这是最基本的 CRUD 系统。对于一个完整的系统，我们需要在每次添加或编辑数据时添加验证和错误检查。

# 使用 Eloquent 导入 CSV

在处理数据时，我们可能会遇到许多不同的来源和文件类型。一个常见的类型是 CSV，即逗号分隔值文件。在这个示例中，我们将获取 CSV 文件的内容并将其插入到我们的数据库中。

## 准备工作

要开始，我们需要一个配置了 MySQL 数据库的标准 Laravel 安装。我们还需要通过运行 Artisan 命令`php artisan migrate:install`来创建我们的迁移表。

## 如何做...

要完成这个示例，按照以下步骤操作：

1.  在文本编辑器中创建一个名为`scifi.csv`的文件，将其保存到应用程序的`public`文件夹中。添加以下数据：

```php
Spock,Star Trek
Kirk,Star Trek
Luke,Star Wars
Lando,Star Wars
Deckard,Blade Runner
Dave,2001
```

1.  在命令提示符中创建一个迁移：

```php
**php artisan migrate:make create_scifi_table**

```

1.  打开刚刚创建的迁移文件，并添加我们的模式：

```php
use Illuminate\Database\Migrations\Migration;

class CreateScifiTable extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('scifi', function($table)
        {
            $table->increments('id');
            $table->string('character');
            $table->string('movie');
            $table->timestamps();
        });
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('scifi');
    }
}
```

1.  运行迁移以创建表：

```php
**php artisan migrate**

```

1.  在`app/models`目录中创建一个名为`Scifi.php`的模型：

```php
class Scifi extends Eloquent {
    protected $table = 'scifi';
}
```

1.  创建一个新的路由来处理我们的 CSV 并保存结果：

```php
Route::get('csv', function()
{
    if (($handle = fopen(public_path() .. '/scifi.csv','r')) !== FALSE)
    {
        while (($data = fgetcsv($handle, 1000, ',')) !==FALSE)
        {
                $scifi = new Scifi();
                $scifi->character = $data[0];
                $scifi->movie = $data[1];
                $scifi->save();
        }
        fclose($handle);
    }

    return Scifi::all();
});
```

## 它是如何工作的...

我们的第一步是创建一个简单的 CSV 文件，其中包含一些科幻角色的名称以及他们出现的电影。然后我们创建一个迁移和一个模式，将添加一个`scifi`表和我们想要保存的字段。

对于我们的模型，我们扩展`Eloquent`并添加一个名为`$table`的受保护变量，该变量设置为我们表的名称。由于我们不会为我们的表名复数化`scifi`，我们需要让 Eloquent 知道使用哪个表。

在我们的路由中，我们打开文件并使用 PHP 的内置函数`fopen()`和`fgetcsv()`循环遍历数据。在每次循环中，我们创建一个新的`Scifi`对象，然后将值设置为我们从 CSV 文件中获取的数据。循环结束后，我们关闭文件。

要查看我们的数据，我们在`Scifi`对象上调用`all()`方法并返回它以显示所有数据。

# 使用 RSS 作为数据源

许多博客和新闻网站提供其内容的 RSS 源。使用 Laravel，我们可以获取这些源并将它们显示为一个订阅阅读器，甚至将它们保存在我们自己的数据库中。

## 准备工作

对于这个示例，我们只需要一个标准的 Laravel 安装和要使用的 RSS URL。

## 如何做...

要完成这个示例，按照以下步骤操作：

1.  在我们的`routes.php`文件中创建一个新的路由来读取 RSS：

```php
Route::get('rss', function()
{
    $source = 'http://rss.cnn.com/rss/cnn_topstories.rss';

    $headers = get_headers($source);
    $response = substr($headers[0], 9, 3);
    if ($response == '404')
    {
        return 'Invalid Source';
    }

    $data = simplexml_load_string(file_get_contents($source));

    if (count($data) == 0)
    {
        return 'No Posts';
    }
        $posts = '';
        foreach($data->channel->item as $item)
        {
            $posts .= '<h1><a href="' . $item->link . '">'. $item->title . '</a></h1>';
            $posts .= '<h4>' . $item->pubDate . '</h4>';
            $posts .= '<p>' . $item->description . '</p>';
            $posts .= '<hr><hr>';
        }
        return $posts;
});
```

## 它是如何工作的...

我们创建一个路由来保存我们的 RSS 阅读器。然后我们将我们的`$source`变量设置为我们想要使用的任何 RSS 源。

为了确保我们的源仍然活跃，我们使用 PHP 函数`get_headers()`，并获取响应代码。如果代码是`404`，则 URL 不起作用。

接下来，我们从 URL 获取内容，并使用`simplexml_load_string()`函数处理源中的 XML。如果该源实际上有数据，我们可以循环遍历它并显示信息。我们也可以在循环中将其保存到我们的数据库中。

# 使用属性更改表列名称

有时我们可能会使用使用不太合乎逻辑的列名创建的数据库。在这种情况下，我们可以使用 Laravel 的 Eloquent ORM 来使用更标准化的名称与表交互，而无需进行数据库更改。

## 准备工作

对于这个示例，我们需要一个标准的 Laravel 安装，一个正确配置的 MySQL 数据库，并且通过运行命令`php artisan migrate:install`设置我们的迁移表。

## 如何做...

要完成这个示例，按照以下步骤操作：

1.  为我们的表创建一个名为`odd`的列的迁移，在命令提示符中：

```php
**php artisan migrate:make create_odd_table --table=odd --create**

```

1.  创建一个迁移以向表中添加一些数据，在命令提示符中：

```php
**php artisan migrate:make add_data_to_odd_table**

```

1.  在`app/database/migrations`文件夹中，打开`create_odd_table`迁移并添加模式：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateOddTable extends Migration {

    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('odd', function(Blueprint $table)
        {
            $table->increments('MyIDcolumn');
            $table->string('MyUsernameGoesHere');
            $table->string('ThisIsAnEmail');
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
        Schema::drop('odd');
    }
}
```

1.  在`app/database/migrations`目录中，打开`add_data_to_odd_table`文件并添加一些数据：

```php
use Illuminate\Database\Migrations\Migration;

class AddDataToOddTable extends Migration {

    /**
     * Make changes to the database.
     *
     * @return void
     */
    public function up()
    {
        $data1 = array('MyUsernameGoesHere' => 'John Doe','ThisIsAnEmail' => 'johndoe@example.com');
        $data2 = array('MyUsernameGoesHere' => 'Jane Doe','ThisIsAnEmail' => 'janedoe@example.com');
        DB::table('odd')->insert(array($data1, $data2));
    }

    /**
     * Revert the changes to the database.
     *
     * @return void
     */
    public function down()
    {
        DB::table('odd')->delete();
    }
}
```

1.  在命令提示符中，运行迁移：

```php
**php artisan migrate**

```

1.  在`app/models`目录中，创建一个名为`Odd.php`的新文件并创建 getter：

```php
class Odd extends Eloquent {
    protected $table = 'odd';

    public function getIdAttribute($value) {
        return $this->attributes['MyIDcolumn'];
    }

    public function getUsernameAttribute($value) {
        return $this->attributes['MyUsernameGoesHere'];
    }

    public function getEmailAttribute($value) {
        return $this->attributes['ThisIsAnEmail'];
    }
}
```

1.  在`routes.php`中创建一个新的路由来访问表，使用常规的列名：

```php
Route::get('odd', function()
{
    $odds = Odd::all();
    foreach($odds as $odd) 
    {
        echo $odd->MyIDcolumn . ' - ' . $odd->MyUsernameGoesHere . ' - ' . $odd->ThisIsAnEmail . '<br>';
    }
});
```

1.  创建另一个路由，使用更标准的列名：

```php
Route::get('notodd', function()
{
    $odds = Odd::all();
    foreach($odds as $odd) 
    {
        echo $odd->id . ' - ' . $odd->username . ' - '. $odd->email . '<br>';
    }
});
```

## 工作原理...

首先，我们创建两个迁移文件。一个文件将实际创建具有非标准列名的表，另一个将填充数据。

对于我们的模型，我们扩展`Eloquent`并添加一些`get`方法。在每个`get`方法内，设置我们的属性，告诉 Eloquent 我们想要使用哪个列名。现在，由于我们在模型中有`getUsernameAttribute()`方法，每当我们尝试在对象中访问用户名时，它实际上会访问我们定义的列名。

然后，我们创建一个路由，将从我们的`odd`表中提取所有记录，并循环遍历。对于我们的第一个路由，我们使用它们的真实名称访问列。在我们的第二个路由中，我们使用新的名称。如果我们访问这两个路由，我们将看到完全相同的信息。

# 在 Laravel 中使用非 Eloquent ORM

Laravel 的 Eloquent ORM 易于使用且非常高效。但是，有许多不同的 PHP ORM，我们可能决定我们更喜欢另一个 ORM。在这个步骤中，我们将安装 RedBean ORM 并将其用于我们的数据。

## 准备工作

对于这个步骤，我们将使用 RedBean ORM。您需要从[`www.redbeanphp.com/manual/installing`](http://www.redbeanphp.com/manual/installing)下载它，并解压文件。然后将文件`rb.php`移动到您的应用程序的`app/libraries`目录中。

## 如何做...

要完成这个步骤，请按照以下步骤进行：

1.  在`composer.json`文件中，使我们的自动加载器加载我们的`libraries`目录。`autoload`部分应该类似于这样：

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
    ],
}
```

1.  在命令提示符中，转储我们的自动加载器：

```php
**php composer.phar dump-autoload**

```

1.  在我们的`routes.php`文件中，我们将添加一个简单的配置：

```php
$db_setup = Config::get('database.connections.mysql');
R::setup('mysql:host=' . $db_setup['host'] . ';dbname='. $db_setup['database'], $db_setup['username'],$db_setup['password']);
```

1.  创建一个路由，将添加一些数据然后显示它：

```php
Route::get('orm', function() 
{
    $superhero = R::dispense('superheroes');
    $superhero->name = 'Spiderman';
    $superhero->city = 'New York';
    $superhero->age = 24;

    $id1 = R::store($superhero);

    $superhero = R::dispense('superheroes');
    $superhero->name = 'Superman';
    $superhero->city = 'Metropolis';
    $superhero->age = 50;

    $id2 = R::store($superhero);

    $superhero = R::dispense('superheroes');
    $superhero->name = 'Batman';
    $superhero->city = 'Gotham';
    $superhero->age = 36;

    $id3 = R::store($superhero);

    $heroes = R::batch('superheroes',array($id1, $id2,$id3));

    foreach ($heroes as $hero)
    {
        echo $hero->name . ' - ' .  $hero->city . ' - '. $hero->age . '<br>';
    }
});
```

## 工作原理...

在将 RedBean 文件添加到我们的`libraries`目录后，我们需要更新我们的 composer 文件的自动加载器，以便它将加载`rb.php`文件。

设置数据库配置可以在各个地方完成，但是对于这个步骤，我们将在我们的路由文件的顶部设置它。因此，我们可以将我们的数据库信息保存在一个地方，我们使用 Laravel 的数据库配置来设置它。

完成所有这些后，我们准备在我们的应用程序中使用 RedBean。在我们的路由中，我们正在创建三个超级英雄并将它们添加到`superheroes`表中。使用 RedBean，如果表不存在，它将自动为您创建它并添加相关列。

最后，我们得到了三条记录，可以循环遍历它们以显示信息。

## 还有更多...

RedBeans 有许多功能，可能作为替代 ORM 很有用。要查看所有功能，请访问其官方手册[`redbeanphp.com/manual/`](http://redbeanphp.com/manual/)。
