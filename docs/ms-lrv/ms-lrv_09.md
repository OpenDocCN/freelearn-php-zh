# 第九章。扩展 Laravel

任何编程语言中构建的框架的特点是使用各种组件。正如我们在前几章中看到的，框架为软件开发人员提供了许多不同的预构建工具，以完成诸如身份验证、数据库交互和 RESTful API 创建等任务。

然而，就框架而言，可扩展性问题总是信息技术领域任何经理最担心的问题。与使用现有代码的任何库一样，总会有一定程度的开销，一定程度的膨胀，总会有比实际需要的更多的东西。

# 可扩展性问题

框架无法轻松扩展的原因有很多。让我们来看一下问题的简要列表：

+   一个问题是不必要的代码和与实际构建的应用程序无直接关系的包。例如，并非每个项目都需要身份验证，数据库驱动程序也不一定是 MySQL。框架核心的包必须监控兼容性问题。

+   设计模式、观点和学习曲线经常阻碍新团队成员快速熟悉。随着项目的扩大，日常开发需求也需要增长，软件开发团队必须不断招募那些对框架已经有一定了解或至少了解其基本概念的成员。

+   框架安全问题需要持续监控框架社区的网站或存储库，以收集有关所需的紧急安全更新的信息。甚至底层的 Web 服务器和操作系统本身也需要监控。在撰写本文时，Laravel 5.1 即将发布，它将需要 PHP 5.5，因为 PHP 5.4 将在 2015 年晚些时候宣布终止生命周期。

+   诸如 Eloquent 之类的 ORM 总是会增加一些开销，因为代码首先需要从 Eloquent 转换为流畅的查询构建器，然后再转换为 PDO 代码。显然，使用面向对象的方法来查询数据库是明智的选择，但它是有成本的。

# 走向企业

尽管可能会遇到一些障碍，Laravel 在未来的企业中仍将是一个强大的选择。PHP 7 将会非常快，而 Zend Framework 3 等框架已经宣布了他们在 PHP 7 优化方面的路线图。此外，通过使用**FastCGI 进程管理器**（**FPM**）、NGINX Web 服务器，并允许 PHP 的缓存机制正常工作，应用程序的可扩展性将继续在企业空间中得到更多的认可，因为它的复兴持续进行，新的开发人员也在为其核心做出贡献。

在本章中，您将学习如何让 Laravel 在企业环境中表现更好，其中可扩展性问题至关重要。首先，将讨论路由器缓存。然后，您将了解许多工具、技术，甚至是正在开发的以可扩展性为重点的新微框架。具体来说，我们将讨论从 Laravel 派生的官方微框架**Lumen**。最后，您将学习如何通过一种称为*读*和*写*的技术有效地使用数据库。

在代码库的大小方面，与 Zend 或 Symfony 相比，Laravel 的代码库是最小的之一，尽管它确实使用了一些 Symfony 组件。如前几章所述，不同的包被移除以减轻占用空间，这是从 Symfony 的基于组件的思想中得到的启示。例如，默认情况下不再包括 HTML、SSH 和注释包。

# 路由缓存

路由缓存有助于加快速度。在 Laravel 5 中，引入了一种缓存机制来加快执行速度。

这里显示了一个示例`routes.php`：

```php
Route::post('reserve-room', 'ReservationController@store');

Route::controllers([
  'auth' => 'Auth\AuthController',
  'password' => 'Auth\PasswordController',
]);
Route::post('/bookRoom','ReservationsController@reserve', ['middleware' => 'auth', 'domain'=>'booking.hotelwebsite.com']);

Route::resource('rooms', 'RoomsController');

Route::group(['middleware' => ['auth','whitelist']], function()
{

  Route::resource('accommodations', 'AccommodationsController');
  Route::resource('accommodations.amenities', 'AccommodationsAmenitiesController');
  Route::resource('accommodations.rooms', 'AccommodationsRoomsController');
  Route::resource('accommodations.locations', 'AccommodationsLocationsController');
  Route::resource('amenities', 'AmenitiesController');
  Route::resource('locations', 'LocationsController');
});
```

通过运行以下命令，Laravel 将缓存路由：

```php
**$ php artisan route:cache**

```

然后，将它们放入以下目录中：

```php
**/vendor/routes.php**

```

这是结果文件的一小部分：

```php
<?php

/*

| Load The Cached Routes
|--------------------------------------------------------------------------
|
| Here we will decode and unserialize the RouteCollection instance that
| holds all of the route information for an application. This allows
| us to instantaneously load the entire route map into the router.
|
*/

app('router')->setRoutes(
  unserialize(base64_decode('TzozNDoiSWxsdW1pbmF0ZVxSb3V0aW5nXF JvdXRlQ29sbGVjdGlvbiI6NDp7czo5OiIAKgByb3V0ZXMiO2E6Njp7czozOiJH RVQiO2E6NTA6e3M6MToiLyI7TzoyNDoiSWxsdW1pbmF0ZVxSb3V0aW5nXFJvdX RlIjo3OntzOjY6IgAqAHVyaSI7czoxOiIvIjtzOjEwOiIAKgBtZXRob2RzIjth OjI6e2k6MDtzOjM6IkdFVCI7aToxO3M6NDoiSEVBRCI7fX
...
Db250cm9sbGVyc1xBbWVuaXRpZXNDb250cm9sbGVyQHVwZGF0ZSI7cjoxNDQx O3M6NTQ6Ik15Q29tcGFueVxIyb2xsZXJzXEhvdGVsQ29udHJvbGxlckBkZXN0c m95IjtyOjE2MzI7fX0='))
);
```

如 DocBlock 所述，路由被编码为 base64，然后进行序列化：

```php
unserialize(base64_decode( … ));
```

这执行一些预编译。如果我们对文件的内容进行 base64 解码，我们将获得序列化的数据。以下代码是文件的一部分：

```php
O:34:"Illuminate\Routing\RouteCollection":4:{s:9:"*routes"; a:6:{s:3:"GET";a:50:{s:1:"/";O:24:"Illuminate\Routing\Route": 7:{s:6:"*uri";s:1:"/";s:10:"*methods";a:2:{i:0;s:3:"GET";i:1; s:4:"HEAD";}s:9:"*action";a:5:{s:4:"uses";s:50:"MyCompany \Http\Controllers\WelcomeController@index";s:10:"controller"; s:50:"MyCompany\Http\Controllers\WelcomeController@index"; s:9:"namespace";s:26:"MyCompany\Http\Controllers";s:6:"prefix"; N;s:5:"where";a:0:{}}s:11:"*defaults";a:0:{}s:9:"*wheres"; a:0:{}s:13:"*parameters";N;s:17:"*parameterNames";N; }s:4:"home";O:24:"Illumin…

"MyCompany\Http\Controllers\HotelController@destroy";r:1632;}}
```

如果`/vendor/routes.php`文件存在，则使用它，而不是位于`/app/Http/routes.php`的`routes.php`文件。如果在某个时候不再希望使用路由缓存文件，则使用以下`artisan`命令：

```php
**$ php artisan route:clear**

```

这个命令将删除缓存的`routes`文件，Laravel 将重新开始使用`/app/Http/routes.php`文件。

### 提示

需要注意的是，如果在`routes.php`文件中使用了任何闭包，缓存将失败。以下是路由中闭包的一个示例：

```php
Route::get('room/{$id}', function(){
  return Room::find($id);
});
```

出于任何原因，在`routes.php`文件中使用闭包是不可取的。为了能够使用路由缓存，将闭包中使用的代码移到控制器中。

## Illuminate 路由

所有这些工作都加快了请求生命周期中的一个重要部分，即路由。在 Laravel 中，路由类位于`illuminate/routing`命名空间中：

```php
<?php namespace Illuminate\Routing;
use Closure;
use LogicException;
use ReflectionFunction;
use Illuminate\Http\Request;
use Illuminate\Container\Container;
use Illuminate\Routing\Matching\UriValidator;
use Illuminate\Routing\Matching\HostValidator;
use Illuminate\Routing\Matching\MethodValidator;
use Illuminate\Routing\Matching\SchemeValidator;
use Symfony\Component\Routing\Route as SymfonyRoute;
use Illuminate\Http\Exception\HttpResponseException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException; 
```

检查`use`操作符，可以清楚地看出路由机制由许多类组成。最重要的一行是：

```php
use Symfony\Component\Routing\Route as SymfonyRoute;
```

Laravel 使用 Symfony 的路由类。然而，Nikita Popov 编写了一个新的路由软件包。`FastRoute`是一个快速的请求路由器，比其他路由软件包更快，并解决了现有路由软件包的一些问题。这个组件是 Lumen 微框架的主要优势之一。

## Lumen

从苏打营销的角度来看，Lumen 可以被认为是 Laravel *Light*或 Laravel *Zero*。除了使用`FastRoute`路由软件包外，许多软件包已从 Lumen 中删除，使其变得最小化并减少其占用空间。

## Laravel 和 Lumen 之间的比较

在下表中列出了 Laravel 和 Lumen 中的软件包，并进行了比较。运行以下命令时，将安装这些软件包：

```php
$ composer update –-no-dev
```

前面的命令是在开发完成并且应用程序准备好部署到服务器时使用的。在这个阶段，诸如 PHPUnit 和 PHPSpec 之类的工具显然被排除在外。

软件包名称对齐，以说明这些软件包在 Laravel 和 Lumen 中的位置：

| Laravel 软件包 | Lumen 软件包 |
| --- | --- |
| - | `nikic/fast-route` |
| `illuminate/cache` | - |
| `illuminate/config` | `illuminate/config` |
| `illuminate/console` | `illuminate/console` |
| `illuminate/container` | `illuminate/container` |
| `illuminate/contracts` | `illuminate/contracts` |
| `illuminate/cookie` | `illuminate/cookie` |
| `illuminate/database` | `illuminate/database` |
| `illuminate/encryption` | `illuminate/encryption` |
| `illuminate/events` | `illuminate/events` |
| `illuminate/exception` | - |
| `illuminate/filesystem` | `illuminate/filesystem` |
| `illuminate/foundation` | - |
| `illuminate/hashing` | `illuminate/hashing` |
| `illuminate/http` | `illuminate/http` |
| `illuminate/log` | - |
| `illuminate/mail` | - |
| `illuminate/pagination` | `illuminate/pagination` |
| `illuminate/pipeline` | - |
| `illuminate/queue` | `illuminate/queue` |
| `illuminate/redis` | - |
| `illuminate/routing` | - |
| `illuminate/session` | `illuminate/session` |
| `illuminate/support` | `illuminate/support` |
| `illuminate/translation` | `illuminate/translation` |
| `illuminate/validation` | `illuminate/validation` |
| `illuminate/view` | `illuminate/view` |
| `jeremeamia/superclosure` | - |
| `league/flysystem` | - |
| `monolog/monolog` | `monolog/monolog` |
| `mtdowling/cron-expression` | `mtdowling/cron-expression` |
| `nesbot/carbon` | - |
| `psy/psysh` | - |
| `swiftmailer/swiftmailer` | - |
| `symfony/console` | - |
| `symfony/css-selector` | - |
| `symfony/debug` | - |
| `symfony/dom-crawler` | - |
| `symfony/finder` | - |
| `symfony/http-foundation` | `symfony/http-foundation` |
| `symfony/http-kernel` | `symfony/http-kernel` |
| `symfony/process` | - |
| `symfony/routing` | - |
| `symfony/security-core` | `symfony/security-core` |
| `symfony/var-dumper` | `symfony/var-dumper` |
| `vlucas/phpdotenv` | - |
| `classpreloader/classpreloader` | - |
| `danielstjules/stringy` | - |
| `doctrine/inflector` | - |
| `ext-mbstring` | - |
| `ext-mcrypt` | - |

在撰写本文时，使用非开发配置在 Laravel 5.0 中安装了 51 个包（显示在左列）。将此包数量与 Lumen 中安装的包数量进行比较（显示在右列）-只有 24 个。

前述的`nikic/fast-route`包是 Lumen 拥有而 Laravel 没有的唯一包。`symfony/routing`包是 Laravel 中的补充包。

## 精简应用程序开发

我们将使用一个示例，一个简单的面向公众的 RESTful API。这个 RESTful API 以 JSON 格式向任何用户显示一系列住宿的名称和地址，通过`GET`：

+   如果不需要使用密码，则不需要`ext/mcrypt`。

+   如果不需要进行日期计算，则不需要`nesbot/carbon`。由于没有 HTML 界面，因此不需要涉及测试应用程序的 HTML 的以下库，`symfony/css-selector`和`symfony/dom-crawler`。

+   如果不需要向用户发送电子邮件，则不需要`illuminate/mail`或`swiftmailer/swiftmailer`。

+   如果不需要与文件系统进行特殊交互，则不需要`league/flysystem`。

+   如果不是从命令行运行的命令，则不需要`symfony/console`。

+   如果不需要 Redis，则可以不使用`illuminate/redis`。

+   如果不需要不同环境的特定配置值，则不需要`vlucas/phpdotenv`。

### 提示

`vlucas/phpdotenv`包是`composer.json`文件中的一个建议包。

很明显，删除某些包的决定是经过慎重考虑的，以便根据最简单的应用程序需要简化 Lumen。

## 读/写

Laravel 还有另一个帮助其在企业中提高性能的机制：读/写。这与数据库性能有关，但功能如此易于设置，以至于任何应用程序都可以利用其有用性。

关于 MySQL，原始的 MyISAM 数据库引擎在插入、更新和删除期间需要锁定整个表。这在修改数据的大型操作期间造成了严重瓶颈，而选择查询等待访问这些表。随着 InnoDB 的引入，`UPDATE`、`INSERT`和`DELETE` SQL 语句只需要在行级别上锁定。这对性能产生了巨大影响，因为选择可以从表的各个部分读取，而其他操作正在进行。

MariaDB，一个 MySQL 分支，声称比传统的 MySQL 性能更快。将数据库引擎替换为 TokuDB 将提供更高的性能，特别是在大数据环境中。

加速数据库性能的另一种机制是使用主/从配置。在下图中，所有操作都在单个表上执行。插入和更新将锁定单行，选择语句将按分配执行。

![读/写](img/B04559_09_01.jpg)

传统数据库表操作

## 主表

主/从配置使用允许`SELECT`、`UPDATE`和`DELETE`语句的主表。这些语句修改表或向其写入。也可能有多个主表。每个主表都保持持续同步：对任何表所做的更改需要通知主表。

## 从表

从数据库表是主数据库表的从属。它依赖于主数据库表进行更改。SQL 客户端只能从中执行读操作（`SELECT`）。可能还有多个从数据库依赖于一个或多个主数据库表。主数据库表将其所有更改通知给所有从数据库。以下图表显示了主/从设置的基本架构：

![从数据库表](img/B04559_09_02.jpg)

主从（读/写设置）

这种持续的同步会给数据库结构增加一些开销；然而，它提供了重要的优势：

由于从数据库表只能执行`SELECT`语句，而主数据库表可以执行`INSERT`、`UPDATE`和`DELETE`语句，因此从数据库表可以自由接受许多`SELECT`语句，而无需等待涉及相同行的任何操作完成。

一个例子是货币汇率或股票价格表。这个表将实时不断地更新最新值，甚至可能每秒更新多次。显然，一个允许许多用户访问这些信息的网站可能会有成千上万的访问者。此外，用于显示这些数据的网页可能会为每个用户不断发出多个请求。

当有`UPDATE`语句需要同时访问相同数据时，执行许多`SELECT`语句会稍微慢一些。

通过使用主/从配置，`SELECT`语句将仅在从数据库表上执行。这个表只以极其优化的方式接收已更改的数据。

在纯 PHP 中使用诸如`mysqli`之类的库，可以配置两个数据库连接：

```php
$master=mysqli_connect('127.0.0.1:3306','dbuser','dbpassword','mydatabase');
$slave=mysqli_connect('127.0.0.1:3307','dbuser','dbpassword','mydatabase');
```

在这个简化的例子中，从数据库设置在同一台服务器上。在实际应用中，它很可能会设置在另一台服务器上，以利用独立的硬件。

然后，所有涉及*写*语句的 SQL 语句将在从数据库上执行，*读*将在主数据库上执行。

这将增加一些编程工作量，因为每个 SQL 语句都需要传入不同的连接：

```php
$result= mysqli_real_query($master,"UPDATE exchanges set rate='1.345' where exchange_id=2");
$result= mysqli_query($slave,"SELECT rate from exchanges where exchange_id=2");
```

在上面的代码示例中，应该记住哪些 SQL 语句应该用于主数据库，哪些 SQL 语句应该用于从数据库。

## 配置读/写

如前所述，用 Eloquent 编写的代码会转换为流畅的查询构建器代码。然后，该代码将转换为 PDO，这是各种数据库驱动程序的标准封装。

Laravel 通过其读/写配置提供了管理主/从配置的能力。这使程序员能够编写 Eloquent 和流畅的查询构建器代码，而不必担心查询是在主数据库表还是从数据库表上执行。此外，一个最初没有主/从配置的软件项目，后来需要扩展到主/从设置，只需要改变数据库配置的一个方面。数据库配置文件位于`config/database.php`。

作为`connections`数组的一个元素，将创建一个带有键`mysql`的条目，其配置如下：

```php
'connections' =>
'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
```

```php
     'password'  => 'slave-Passw0rd', 
    ],
    'write' => [
        'host' => '196.168.1.2',
```

```php
    'username'  => 'dbhostusername'    
    ],
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'dbusername',
    'password'  => 's0methingSecure',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
],
```

读和写分别代表从和主。由于参数级联，如果用户名、密码和数据库名称相同，则只需要列出主机名的 IP 地址。但是，任何值都可以被覆盖。在这个例子中，读取的密码与主数据库不同，写入的用户名与从数据库不同。

# 创建主/从数据库配置

要设置主/从数据库，请从命令行执行以下步骤。

1.  第一步是确定 MySQL 服务器绑定到哪个地址。为此，请找到包含 bind-address 参数的 MySQL 配置文件的行：

```php
    **bind-address            = 127.0.0.1**

    ```

此 IP 地址将设置为主服务器使用的 IP 地址。

1.  接下来，取消注释包含`server-id`的 MySQL 配置文件中的行，该文件很可能位于`/etc/my.cn`或`/etc/mysql/mysql.conf.d/mysqld.cnf`。

1.  Unix 的`sed`命令可以轻松执行此操作：

```php
    **$ sed -i s/#server-id/server-id/g  /etc/mysql/my.cnf**

    ```

### 提示

`/etc/mysql/my.cnf`字符串需要替换为正确的文件名。

1.  取消注释包含`server-id`的 MySQL 配置文件中的行：

```php
    **$ sed -i s/#log_bin/log_bin/g  /etc/mysql/my.cnf**

    ```

### 提示

同样，`/etc/mysql/my.cnf`字符串需要替换为正确的文件名。

1.  现在，需要重新启动 MySQL。您可以使用以下命令执行此操作：

```php
    **$ sudo service mysql restart**

    ```

1.  以下占位符应替换为实际值：

```php
    **MYSQLUSER**
    **MYSQLPASSWORD**
    **MASTERDATABASE**
    **MASTERDATABASEUSER**
    **MASTERDATABASEPASSWORD**
    **SLAVEDATABASE**
    **SLAVEDATABASEUSER**
    **SLAVEDATABASEPASSWORD**

    ```

## 设置主服务器

设置主服务器的步骤如下：

1.  授予从数据库用户权限：

```php
    **$ echo  "GRANT REPLICATION SLAVE ON *.* TO 'DATABASEUSER'@'%' IDENTIFIED BY 'DATABASESLAVEPASSWORD';" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

1.  接下来，必须使用以下命令刷新权限：

```php
    **$ echo  "FLUSH PRIVILEGES;" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

1.  接下来，使用以下命令切换到主数据库：

```php
    **$ echo  "USE MASTERDATABASE;" | mysql -u MYSQLUSER -p"DATABASEPASSWORD"** 

    ```

1.  接下来，使用以下命令刷新表：

```php
    **$ echo  "FLUSH TABLES WITH READ LOCK;" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

1.  使用以下命令显示主数据库状态：

```php
    **$ echo  "SHOW MASTER STATUS;" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

注意输出中的位置和文件名：

```php
    POSITION
    FILENAME
    ```

1.  使用以下命令转储主数据库：

```php
    **$ mysqldump -u root -p"MYSQLPASSWORD"  --opt "MASTERDATABASE" > dumpfile.sql**

    ```

1.  使用以下命令解锁表：

```php
    **$ echo  "UNLOCK TABLES;" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

## 设置从服务器

设置从服务器的步骤如下：

1.  在从服务器上，使用以下命令创建从数据库：

```php
    **$ echo  "CREATE DATABASE SLAVEDATABASE;" | mysql -u MYSQLUSER -p"MYSQLPASSWORD"** 

    ```

1.  使用以下命令导入从主数据库创建的转储文件：

```php
    **$ mysql -u MYSQLUSER -p"MYSQLPASSWORD"  "MASTERDATABASE" < dumpfile.sql**

    ```

1.  现在，MySQL 配置文件使用 server-id 2：

```php
    server-id            = 2
    ```

1.  在 MySQL 配置文件中，应取消注释两行，如下所示：

```php
    **#log_bin			= /var/log/mysql/mysql-bin.log**
    **expire_logs_days	= 10**
    **max_binlog_size   = 100M**
    **#binlog_do_db		= include_database_name**

    ```

1.  您将得到以下结果：

```php
    log_bin			= /var/log/mysql/mysql-bin.log
    expire_logs_days	= 10
    max_binlog_size    = 100M
    binlog_do_db		= include_database_name
    ```

1.  此外，需要在`binglog_do_db`下面添加以下行：

```php
    relay-log                = /var/log/mysql/mysql-relay-bin.log
    ```

1.  现在，需要使用以下命令重新启动 MySQL：

```php
    **$ sudo service mysql restart**

    ```

1.  最后，设置主密码。主日志文件和位置将设置为步骤 5 中记录的文件名和位置。运行以下命令：

```php
    MASTER_PASSWORD='password', MASTER_LOG_FILE='FILENAME', MASTER_LOG_POS= POSITION;
    ```

# 总结

在本章中，您学会了如何通过路由缓存加快路由速度。您还学会了如何完全用 Lumen 替换 Laravel，这是完全源自 Laravel 的微框架。最后，我们讨论了 Laravel 如何使用读写配置充分利用主从配置。

Symfony 2.7 于 2015 年 5 月发布。这是一个长期支持版本。该版本将得到 36 个月的支持。在那之后不久，Taylor Otwell 决定创建 Laravel 的第一个 LTS 版本。这是 Laravel 牢固地定位在企业空间的第一个迹象。与 Symfony 和 Zend 的情况不同，Laravel 背后还没有正式的公司。然而，有一个庞大的社区包和服务生态系统，比如由 Jeffrey Way 运营的 Laracasts，他与 Taylor 密切合作提供官方培训视频。

此外，Taylor Otwell 还运行一个名为 Envoyer 的服务，该服务消除了 Laravel 部署的所有初始障碍，并为 Laravel 以及其他类型的现代 PHP 项目提供*零停机*部署。

随着 Laravel 5.1 LTS 的到来，Laravel 将会发生许多新的令人兴奋的事情。使用许多社区包的决定使 Taylor 和他的社区能够专注于框架的最重要方面，而无需重新发明轮子并维护许多冗余的包。此外，Laravel Collective 维护了已被弃用的包，即使最终从 Laravel 中删除的包也将继续得到多年的支持。

除了方便的服务，比如 Envoyer，下一章还将介绍一个最近出现的优秀自动化工具：Elixir。
