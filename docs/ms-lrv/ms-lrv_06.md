# 第六章：使用注解驯服复杂性

在上一章中，您学习了如何创建一个涉及从互联网接收请求、将其路由到控制器并处理的 RESTful API。在本章中，您将学习如何在 DocBlock 中使用注解，这是一种需要更少代码的路由执行方式，可以更快、更有组织地进行团队协作编程。

注解将被用于：

+   路由 HTTP 请求，如 GET、POST 和 PUT

+   将控制器转换为完全启用的 CRUDL 资源

+   监听从命令触发的事件

+   向控制器添加中间件以限制或过滤请求

注解是编程中使用的重要机制。注解是增强其他数据的元数据。由于这可能看起来有点混乱，所以我们需要首先了解元数据的含义。**元数据**是一个包含两部分的词：

+   **meta**：这是一个希腊词，意思是超越或包含。

+   **data**：这是一个拉丁词，意思是信息片段。

因此，元数据用于增强或扩展某物的含义。

# 其他编程语言中的注解

接下来，我们将讨论在计算机编程中使用的注解。我们将从 Java、C#和 PHP 中看几个例子，然后最后，看一下注解在 Laravel 中的使用。

## Java 中的注解

注解首次在 Java 版本 1.1 中提出，并在版本 1.2 中添加。以下是一个用于覆盖动物的`speak`方法的注解示例：

```php
Java 1.2
/**
 * @author      Jon Doe <jon@doe.com>
 * @version     1.6               (current version number)
 * @since       2010-03-31        (version package...)
 */
public void speak() {
}

public class Animal {
    public void speak() {
    }
} 
public class Cat extends Animal {
    @Override
    public void speak() {
        System.out.println("Meow.");
    }
 }
```

请注意，`@`符号用于向编译器发出此注解`@Override`很重要的信号。

## C#中的注解

在 C#中，注解称为属性，使用方括号而不是更常用的`@`符号：

```php
[AttributeUsageAttribute(AttributeTargets.Property|AttributeTargets.Field, AllowMultiple = false, Inherited = true)]
public sealed class AssociationAttribute : Attribute
```

## PHP 中的注解

其他 PHP 框架也使用注解。Symfony 广泛使用注解。在**Doctrine**中，这是 Symfony 的 ORM，类似于 Laravel 的 Eloquent，使用注解来定义关系。Symfony 还使用注解进行路由。**Zend Framework**（**ZF**）也使用注解。测试工具 Behat 和 PHPUnit 都使用注解。在 Behat 的以下示例中，使用注解指示应在测试套件之前执行此方法：

```php
/**
 * @BeforeSuite
 */
public static function prepare(SuiteEvent $event)
{
// prepare system for test suite
// before it runs
}
```

# DocBlock 注解

在前面的 Behat 示例中展示的注解使用示例相当有趣，因为它将注解放在了 DocBlock 内部。DocBlock 以斜杠和两个星号开头：

```php
/**
```

它包含*n*行以星号开头。

DocBlock 以单个星号和斜杠结束：

```php
 */
```

这种语法告诉解析器，除了普通注释之外，DocBlock 中还有一些有用的东西。

## Laravel 中的 DocBlock 注解

当 Laravel 5 正在开发时，最初添加了通过 DocBlock 注解支持路由和事件监听器。它的语法类似于 Symfony 和 Zend。

### Symfony

Symfony 的语法如下：

```php
/**
 * @Route("/accommodations/search")
 * @Method({"GET"})
 */

public function searchAction($id)
{
```

### Zend

Zend 的语法如下：

```php
/**
 * @Route(route="/accommodations/search")
 */

public function searchAction()
{
```

### Laravel

Laravel 的语法如下：

```php
/**
 * @Get("/hotels/search")
 */

public function search()
{
```

但是，DocBlock 注解试图解决什么类型的问题呢？

Doc-annotations 的一个用途是将它们添加到控制器中，从而将路由和中间件的控制移交给控制器。这将使控制器更具可移植性，甚至是与框架无关的，因为`routes.php`文件的作用会减少，甚至完全不存在。如下例所示，`routes.php`文件可能会变得非常庞大，这将导致复杂性甚至使文件难以管理：

```php
Route::patch('hotel/{hid}/room/{rid}','AccommodationsController@editRoom');
Route::post('hotel/{hid}/room/{rid}','AccommodationsController@reserve');
Route::get('hotel/stats,HotelController@Stats');
Route::resource('country', 'CountryController');
Route::resource(city', 'CityController');
Route::resource('state', 'StateController');
Route::resource('amenity', 'AmenitiyController');
Route::resource('country', 'CountryController');
Route::resource(city', 'CityController');
Route::resource('country', 'CountryController');
Route::resource('city', 'CityController');
Route::resource('horse', 'HorseController');
Route::resource('cow', 'CowController');
Route::resource('zebra', 'ZebraController');
Route::get('dragon/{id}', 'DragonController@show');
Route::resource('giraffe', 'GiraffeController');
Route::resource('zebrafish', 'ZebrafishController');
```

DocBlock 注解的想法是驯服这种复杂性，因为路由将被移动到控制器中。

在 Laravel 5.0 发布之前不久，由于社区的不满，该功能被移除。此外，由于一些开发人员可能不想使用这种方法，将此包从 Laravel 的核心中移出并打包是合适的。安装该包的方法类似于添加 HTML 包的方式。这个包也得到了 Laravel Collective 的支持。通过输入以下 composer 命令很容易添加注释：

```php
**$ composer require laravelcollective/annotations**

```

这将安装注释包，而`composer.json`将显示包添加到 require 部分，如下所示：

```php
"require": {
    "laravel/framework": "5.0.*",
    "laravelcollective/annotations": "~5.0",
  },
```

下一步将是创建一个名为`AnnotationsServiceProvider.php`的文件，并添加以下代码：

```php
<?php namespace App\Providers;

use Collective\Annotations\AnnotationsServiceProvider as ServiceProvider;

class AnnotationsServiceProvider extends ServiceProvider {

    /**
     * The classes to scan for event annotations.
     *
     * @var array
     */
    protected $scanEvents = [];

    /**
     * The classes to scan for route annotations.
     *
     * @var array
     */
    protected $scanRoutes = [];

    /**
     * The classes to scan for model annotations.
     *
     * @var array
     */
    protected $scanModels = [];

    /**
     * Determines if we will auto-scan in the local environment.
     *
     * @var bool
     */
    protected $scanWhenLocal = false;

    /**
     * Determines whether or not to automatically scan the controllers
     * directory (App\Http\Controllers) for routes
     *
     * @var bool
     */
    protected $scanControllers = false;

    /**
     * Determines whether or not to automatically scan all namespaced
     * classes for event, route, and model annotations.
     *
     * @var bool
     */
    protected $scanEverything = false;

}
```

接下来，`AnnotationsServiceProvider.php`文件将需要添加到`config/app.php`文件中。需要添加命名空间的类应添加到 providers 数组中，如下所示：

```php
'providers' => [
    // ...
    'App\Providers\AnnotationsServiceProvider'
  ];
```

# 使用 DocBlock 注释的资源控制器

现在，为了说明 Laravel 的 DocBlock 注释的使用，我们将检查以下步骤。

首先，我们将像往常一样创建住宿控制器：

```php
**$ php artisan make:controller AccommodationsController**

```

接下来，我们将将住宿控制器添加到注释服务提供程序要扫描的路由列表中：

```php
protected $scanRoutes = [
    'App\Http\Controllers\HomeController',
    'App\Http\Controllers\AccommodationsController'
];
```

现在，我们将向控制器添加 DocBlock 注释。在这种情况下，我们将指示解析器将此控制器用作住宿路由的资源控制器。要添加的代码如下：

```php
/**
* @Resource("/accommodations")
*/

```

由于整个控制器将被转换为资源，因此 DocBlock 注释应该在类定义之前插入。`AccommodationsController`类现在应该如下所示：

```php
<?php namespace MyCompany\Http\Controllers;

use Illuminate\Support\Facades\Response;
use MyCompany\Http\Requests;
use MyCompany\Http\Controllers\Controller;
use MyCompany\Accommodation;
use Illuminate\Http\Request;

/**
* @Resource("/accommodations")
*/
class AccommodationsController extends Controller {

    /**
     * Display a listing of the resource.
     *
     * @return Response
     */
    public function index(Accommodation $accommodation)
    {
        return $accommodation->paginate();
    }
```

### 注意

请注意，这里需要双引号：

```php
@Resource("/accommodations")
```

以下语法，使用单引号，将不正确并且不起作用：

```php
@Resource('/accommodations')
```

# 单方法路由

如果我们只想为单个方法添加一个路由，比如“搜索住宿”，那么一个注解将被添加到单个方法的上方；然而，这一次是在类的内部。为了处理 GET HTTP 请求动词，代码将如下所示：

```php
/**
 * Search for an accommodation
 * @Get("/search-accommodation")
 */
```

类将如下所示：

```php
<?php namespace MyCompany\Http\Controllers;

use Illuminate\Support\Facades\Response;
use MyCompany\Http\Requests;
use MyCompany\Http\Controllers\Controller;
use MyCompany\Accommodation;
use Illuminate\Http\Request;

class AccommodationsController extends Controller {

    /**
    * Search for an accommodation
    * @Get("/search-accommodation")
    */
    public function index(Accommodation $accommodation)
    {
        return $accommodation->paginate();
    }
```

# 扫描路由

接下来的步骤非常重要。Laravel 应用程序必须处理注释。为此，Artisan 用于扫描路由。

以下命令用于扫描路由。输出将显示`Routes scanned!`，如下所示：

```php
**$ php artisan route:scan**

**Routes scanned!**

```

此扫描的结果将在`storage/framework`目录中产生一个名为`routes.scanned.php`的文件。

以下代码将写入`storage/framework/routes.scanned.php`文件：

```php
$router->get('search-accommodation', [
  'uses' => 'MyCompany\Http\Controllers\AccommodationsController@search',
  'as' => NULL,
  'middleware' => [],
  'where' => [],
  'domain' => NULL,
]);
```

### 注意

请注意，`storage/framework/routes.scanned.php`文件不需要放入源代码控制中，因为它是生成的。

# 自动扫描

如果开发人员在构建控制器时必须执行 Artisan 路由扫描命令，那么这样做可能变得乏味。为了方便开发人员，在开发模式下，有一种方法可以让 Laravel 自动扫描`scanRoutes`数组中的控制器。

在`AnnotationsServiceProvider.php`文件中，将`scanWhenLocal`属性设置为`true`。

对于`$scanControllers`和`$scanEverything`也是如此；这两个布尔标志允许框架自动扫描`App\Http\Controllers`目录和任何有命名空间的类。

必须记住，这应该*只*在开发和开发机器上使用，因为它会给请求周期增加不必要的开销。将属性设置为`true`的示例如下所示：

```php
<?php namespace App\Providers;

use Collective\Annotations\AnnotationsServiceProvider as ServiceProvider;

class AnnotationsServiceProvider extends ServiceProvider {

    /**
     * The classes to scan for event annotations.
     *
     * @var array
     */
    protected $scanEvents = [];

    …

    /**
     * Determines if we will auto-scan in the local environment.
     *
     * @var bool
     */
    protected $scanWhenLocal = true;

    /**
     * Determines whether or not to automatically scan the controllers
     * directory (App\Http\Controllers) for routes
     *
     * @var bool
     */
    protected $scanControllers = true;

    /**
     * Determines whether or not to automatically scan all namespaced
     * classes for event, route, and model annotations.
     *
     * @var bool
     */
    protected $scanEverything = true;

}
```

启用这些选项将减慢框架的速度，但允许在开发阶段灵活性。

# 额外的注释

要将 ID 传递给路由，就像在显示单个住宿时一样，代码将如下所示：

```php
/**
* Display the specified resource.
* @Get("/accommodation/{id}")
*/
```

这个 DocBlock 注释将被放置在类内部的函数上方，这与之前的例子类似。

要将 ID 限制为一个或多个数字，可以使用`@Where`注释如下：

```php
@Where({"id": "\d+"})
```

如下所示，两个注释被合并在一起：

```php
/**
 * Display the specified resource.
 * @Get("/accommodation/{id}")
 * @Where({"id": "\d+"})
 */
```

要向示例添加中间件，限制请求仅限于经过身份验证的用户，可以使用`@Middleware`注释：

```php
/**
 * Display the specified resource.
 * @Get("/accommodation/{id}")
 * @Where({"id": "\d+"})
 * @Middleware("auth")
 */
```

## HTTP 动词

以下是可以使用注释的各种 HTTP 动词的列表，它们与 RESTful 标准相对应：

+   `@Delete`：此动词删除一个资源。

+   `@Get`：此动词显示一个资源或多个资源。

+   `@Options`：此动词显示选项列表。

+   `@Patch`：此动词修改资源的属性。

+   `@Post`：此动词创建一个新资源。

+   `@Put`：此动词修改资源。

### 其他注释

还有其他注释也可以在控制器中使用。这些注释如下：

+   `@Any`：对任何 HTTP 请求做出响应。

+   `@Controller`：为资源创建一个控制器。

+   `@Middleware`：这为资源添加中间件。

+   `@Route`：这使得路由可用。

+   `@Where`：根据特定条件限制请求。

+   `@Resource`：这使得资源可用。

# 在 Laravel 5 中使用注释

让我们回顾一下在 Laravel 中实现的路径，如下所示：

+   HTTP 请求被路由到控制器

+   命令是在控制器内部实例化的

+   事件被触发

+   事件被处理

![在 Laravel 5 中使用注释](img/B04559_06_01.jpg)

Laravel 的现代基于命令的发布-订阅路径。

使用注释，这个过程可以变得更加简单。首先，将创建一个预订控制器：

```php
$ php artisan make:controller ReservationsController
```

为了创建一个路由，允许用户创建一个新的预订，将使用 POST HTTP 动词。`@Post`注释将监听附加到`/bookRoom`网址的具有`POST`方法的请求。这将代替通常在`routes.php`文件中找到的路由：

```php
<?php namespace MyCompany\Http\Controllers;

use ...

class ReservationsController extends Controller {
/**
* @Post("/bookRoom")
*/
  public function reserve()
  {
  }
```

如果我们想要将请求限制为有效的 URL，则域参数将请求限制为特定的 URL。此外，auth 中间件要求对希望预订房间的任何请求进行身份验证：

```php
<?php namespace App\Http\Controllers;

use …
/**
* @Controller(domain="booking.hotelwebsite.com")
*/

class ReservationsController extends Controller {

/**
* @Post("/bookRoom")
* @Middleware("auth")
*/
  public function reserve()
  {
```

接下来，应该创建`ReserveRoom`命令。这个命令将在控制器内实例化：

```php
**$ php artisan make:command ReserveRoom**

```

ReserveRoom 命令的内容如下：

```php
<?php namespace MyCompany\Commands;

use MyCompany\Commands\Command;
use MyCompany\User;
use MyCompany\Accommodation\Room;
use MyCompany\Events\RoomWasReserved;

use Illuminate\Contracts\Bus\SelfHandling;

class ReserveRoomCommand extends Command implements SelfHandling {

  public function __construct()
  {
  }
  /**
   * Execute the command.
   */
  public function handle()
  {
  }
}
```

接下来，我们需要在预订控制器内部实例化`ReserveRoom`命令：

```php
<?php namespace MyCompany\Http\Controllers;

use MyCompany\Accommodation\Reservation;
use MyCompany\Commands\PlaceOnWaitingListCommand;
use MyCompany\Commands\ReserveRoomCommand;
use MyCompany\Events\RoomWasReserved;
use MyCompany\Http\Requests;
use MyCompany\Http\Controllers\Controller;
use MyCompany\User;
use MyCompany\Accommodation\Room;

use Illuminate\Http\Request;

class ReservationsController extends Controller {

/**
 * @Post("/bookRoom")
 * @Middleware("auth")
 */
  public function reserve()
  {	
    $this->dispatch(
    new ReserveRoom(\Auth::user(),$start_date,$end_date,$rooms)
    );
  }
```

现在我们将创建`RoomWasReserved`事件：

```php
**$ php artisan make:event RoomWasReserved**

```

要从`ReserveRoom`处理程序中实例化`RoomWasReserved`事件，我们可以利用`event()`辅助方法。在这个例子中，命令是自处理的，因此这样做很简单：

```php
<?php namespace App\Commands;

use App\Commands\Command;
use Illuminate\Contracts\Bus\SelfHandling;

class ReserveRoom extends Command implements SelfHandling {
    public function __construct(User $user, $start_date, $end_date, $rooms)
    {
    }
    public function handle()
    {
        $reservation = Reservation::createNew();
        event(new RoomWasReserved($reservation));
    }
}
```

由于用户需要收到房间预订电子邮件的详细信息，下一步是为`RoomWasReserved`事件创建一个电子邮件发送处理程序。为此，再次使用`artisan`来创建处理程序：

```php
**$ php artisan handler:event RoomReservedEmail –event=RoomWasReserved**

```

`RoomWasReserved`事件的`SendEmail`处理程序的方法只是构造函数和处理程序。发送电子邮件的工作将在处理程序方法内执行。`@Hears`注释被添加到其 DocBlock 中以完成这个过程：

```php
<?php namespace MyCompany\Handlers\Events;

use MyCompany\Events\RoomWasReserved;

use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldBeQueued;

class RoomReservedEmail {
  public function __construct()
  {
  }

  /**
   * Handle the event.
   * @Hears("\App\Events\RoomWasReserved")
   * @param  RoomWasReserved  $event
   */
  public function handle(RoomWasReserved $event)
  {
     //TODO: send email to $event->user
  }
}
```

只需将`RoomReservedEmail`添加到`scanEvents`数组中，以允许扫描该事件，如下所示：

```php
protected $scanEvents = [
   'App\Handlers\Events\RoomReservedEmail'
];
```

最后一步是导入。Artisan 用于扫描事件的注释并写入输出文件：

```php
**$ php artisan event:scan**

 **Events scanned!**

```

这是`storage/framework/events.scanned.php`文件的输出，显示了事件监听器：

```php
<?php $events->listen(array(0 => 'App\\Events\\RoomWasReserved',
), App\Handlers\Events\RoomReservedEmail@handle');
```

在存储目录中扫描的注释文件的最终视图如下。请注意它们是并列的：

```php
**storage/framework/events.scanned.php**
**storage/framework/routes.scanned.php**

```

### 提示

Laravel 使用`artisan`来缓存路由，但不用来扫描事件，因此以下命令会生成一个缓存文件：

```php
**$ php artisan route:cache**
**Route cache cleared!**
**Routes cached successfully!**

```

在运行`route:cache`之前必须先运行`route:scan`命令，因此按照这个顺序执行这两个命令非常重要：

```php
$ php artisan route:scan
Routes scanned!

$ php artisan route:cache
Route cache cleared!
Routes cached successfully!
```

此命令写入到：`storage/framework/routes.php`。

```php
<?php

app('router')->setRoutes(
  unserialize(base64_decode('TzozNDoiSWxsdW1pbmF0ZVxSb3V0aW5nXFd…'))
);
```

两个文件都会被创建，但只有编译后的`routes.php`文件在再次运行`php artisan route:scan`之前才会被使用。

## 优势

在 Laravel 中使用 DocBlock 注解进行路由有几个主要优势：

+   每个控制器保持独立。控制器不与单独的路由“绑定”，这使得共享控制器，并将其从一个项目移动到另一个项目变得更容易。对于只有少数控制器的简单项目来说，`routes.php`文件可能被视为不必要。

+   开发人员无需担心`routes.php`。与其他开发人员合作时，路由文件需要保持同步。通过 DocBlock 注解方法，`routes.php`文件被缓存，不放在源代码控制下；每个开发人员可以专注于自己的控制器。

+   路由注解将路由与控制器保持在一起。当控制器和路由分开时，当新程序员第一次阅读代码时，可能不会立即清楚每个控制器方法附加到哪些路由上。通过直接将路由放在函数上方的 DocBlock 中，这一点立即变得明显。

+   熟悉并习惯在 Symfony 和 Zend 等框架中使用注解的开发人员可能会发现在 Laravel 中使用注解是开发软件应用的一种非常方便的方式。此外，将 Laravel 作为首次 PHP 体验的 Java 和 C#开发人员会发现注解非常方便。

# 结论

是否在软件中使用注解的决定取决于开发人员。从 Laravel 核心中移除它的决定，以及 HTML 表单包，表明该框架变得越来越灵活，只有一组最小的包作为默认。这使得在 Laravel 5.1 发布长期支持（LTS）版本时，核心开发人员可以更加稳定和减少维护工作。

由于注解包是 Laravel Collective 的一部分，该团队将负责管理此包的支持，这保证了该功能的实用性将通过对存储库的贡献得到扩展和扩展。

此外，该包可以扩展以包括一个模板，该模板会自动创建与控制器同名的路由注解。这将在创建控制器和路由的过程中节省另一个步骤，这是软件开发过程中最重要但又单调的任务之一。

# 总结

在本章中，我们了解了注解的用法，它们在编程中的一般用法，它们在其他框架中的用法，以及它们如何被引入到 Laravel 注解 composer 包中。我们学会了如何通过使用注解来加快开发过程，以及如何自动扫描注解。在下一章中，我们将学习中间件，这是一种在路由和应用程序之间使用的机制。
