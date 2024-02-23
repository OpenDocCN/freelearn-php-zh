# 第一章：使用 phpspec 正确设计

自 2011 年 Laravel 谦虚的开始以来，发生了许多事情。Taylor Otwell，一名.NET 程序员，寻求使用 PHP 来进行一项副业项目，因为他被告知托管 PHP 便宜且无处不在。最初作为 CodeIgniter 的扩展开始，最终成为自己的代码。将代码库从 CodeIgniter 的 PHP 5.2 的限制中释放出来，可以使用 PHP 5.3 提供的所有新功能，如命名空间和闭包。版本 1 和 3 之间的时间跨度仅为一年。版本 3 后，事情发生得非常迅速。在其爆炸式的流行之后，即版本 4 发布时，它迅速开始从其他流行框架（如 CodeIgniter、Zend、Symfony、Yii 和 CakePHP）那里夺取市场份额，最终占据了领先地位。除了其表达性语法、出色的文档和充满激情的创始人外，还有大型社区的主要支柱 IRC 和 Slack 聊天室、Laravel 播客和 Laracasts 教学视频网站。此外，新创建的商业支持，如提供*100%正常运行时间*的 Envoyer，也意味着 Laravel 也受到了企业的欢迎。随着 Laravel 4.2 的发布，最低要求的 PHP 版本提高到了 5.4，以利用现代 PHP 特性，如*traits*。

使用 Laravel 的特性以及新的语法，比如[]数组快捷方式，使编码变得轻松。Laravel 的表达性语法，再加上这些现代 PHP 特性，使它成为任何希望构建强大应用的开发者的绝佳选择。

![使用 phpspec 正确设计](img/B04559_01_01.jpg)

Laravel 在 Google 趋势报告中的成功崛起

# 一个新时代

2014 年底，Laravel 历史上第二个最重要的时刻发生了。原定的 4.3 版本改变了许多 Laravel 的核心原则，社区决定将其成为 5.0 版本。

Laravel 5 的到来带来了许多在构建软件时使用它的方式的变化。从诸如 CodeIgniter 等框架继承的内置 MVC 架构已被放弃，以更具动态性、模块化甚至大胆的框架不可知性为代价。许多组件已尽可能解耦。Laravel 历史上最重要的部分将是 Laravel 5.1 版本的到来，它将有**长期支持**（**LTS**）。因此，Laravel 在企业中的地位将更加稳固。此外，最低的 PHP 要求将更改为 5.5 版本。因此，对于任何新项目，建议使用 PHP 5.5，甚至 PHP 5.6，因为升级到 PHP 7 版本将更加容易。

## 一个更精简的应用程序

`/app`目录变得更加精简，只留下了应用程序中最基本的部分。诸如`config`、`database`、`storage`和`tests`等目录已经从`app`目录中移出，因为它们是辅助应用程序本身的。最重要的是，测试工具的集成已经大大成熟。

## PSR

由于**框架互操作性组**（**PHP-FIG**）的努力，**PHP 标准推荐**（**PSR**）的开发者，框架代码的阅读、编写和格式化变得更加容易。它甚至允许开发者更容易地在多个框架中工作。Laravel 是 FIG 的一部分，并继续将其建议纳入框架中。例如，Laravel 5.1 将采用 PSR-2 标准。有关 PHP FIG 和 PSR 的更多信息，请访问 PHP-FIG 网站[`www.php-fig.org`](http://www.php-fig.org)。

# 安装和配置 Laravel

安装 Laravel 的最新更新说明始终可以在 Laravel 网站[`laravel.com`](http://laravel.com)找到。要在开发环境中开始使用 Laravel，当前的最佳实践建议使用以下方法：

+   Vagrant：这提供了一种方便的方式来管理虚拟机，如 Virtualbox。

+   PuPHPet：这是一个可以用来创建各种类型虚拟机的优秀工具。有关 PuPHPet 的更多信息，请访问[`puphpet.com`](https://puphpet.com)。

+   Phansible：这是 PuPHPet 的另一种选择。有关 Phansible 的信息，请访问[`phansible.com`](http://phansible.com)。

+   Homestead：这是由 Laravel 社区维护的，是专门为 Laravel 创建的虚拟机，使用的是 NGINX 而不是 Apache。有关 Homestead 的更多信息，请访问[`github.com/laravel/homestead`](https://github.com/laravel/homestead)。

## 安装

基本过程涉及下载和安装 Composer，然后将 Laravel 添加为依赖项。一个重要的细节是，存储目录，它位于`/app`目录的平行位置，需要以可写的方式设置，以便允许 Laravel 5 执行诸如写日志文件之类的操作。还很重要的是确保使用`$ php artisan key:generate`生成一个用于哈希的 32 字符密钥，因为自 PHP 5.6 发布以来，Mcrypt 对其要求更为严格。对于 Laravel 5.1，OpenSSL 将取代 Mcrypt。

## 配置

在 Laravel 4 中，环境是以服务器或开发机器的主机名配置的，这相当牵强。相反，Laravel 5 使用一个`.env`文件来设置各种环境。该文件包含在`.gitignore`中。因此，每台机器都应该从源代码控制之外的源接收其配置。

因此，例如，可以使用以下代码来设置本地开发：

```php
APP_ENV=local
APP_DEBUG=true
APP_KEY=SomeRandomString
DB_HOST=localhost
DB_DATABASE=example
DB_USERNAME=DBUser
DB_PASSWORD=DBPass
CACHE_DRIVER=file
SESSION_DRIVER=file
```

## 命名空间

Laravel 的一个很好的新功能是，它允许您将最高级别的命名空间设置为诸如`MyCompany`之类的内容，通过`app:name`命令。这个命令实际上会将`/app`目录中所有相关文件的命名空间从 App 更改为`MyCompany`，例如。然后，这个命名空间存在于`/app`目录中。这将命名空间化到几乎每个文件中，而在之前的 4.x 版本中，这是可选的。

# 正确的 TDD

测试驱动开发的文化并不新鲜。相反，甚至在肯特·贝克（Kent Beck）在 1990 年代编写 SUnit 之前就已经存在。源自 SUnit 的 xUNIT 系列单元测试框架已经发展成为为 PHP 提供测试解决方案。

## PHPUnit

PHP 端口的 PHP 测试软件名为 PHPUnit。然而，在 PHP 语言中进行测试驱动开发是一个相当新的概念。例如，在他的书《*The Grumpy Programmer's Guide To Building Testable PHP Applications*》中，*Chris Hartjes*在 2012 年底出版，写道“我开始研究围绕 CodeIgniter 的测试文化。它比新生儿还弱。”

自 Laravel 3 版本以来，测试一直是 Laravel 框架的一部分，使用 PHPUnit 单元测试工具，因此 Laravel 包含`phpunit.xml`文件是在努力鼓励开发人员接受测试驱动开发的努力中迈出的重要一步。

# phpspec

另一个测试工具 RSpec 在 2007 年出现在 Ruby 社区，并对测试驱动开发进行了改进。它具有**行为驱动开发**（**BDD**）。phpspec 工具将 RSpec 的 BDD 移植到 PHP 中，正在迅速增长。它的共同创始人 Marcello Duarte 多次表示“BDD 是正确的 TDD”。因此，BDD 只是对 TDD 的*改进*或演变。Laravel 5 现在巧妙地将 phpspec 包含为一种突出*按规范设计*行为驱动开发范式的方式。

由于在构建 Laravel 5 应用程序的基本步骤是指定要创建的实体，因此在安装和配置 Laravel 5 后，开发人员可以立即通过运行 phpspec 作为设计工具开始设计。

# 实体创建

让我们创建一个示例 Web 应用程序。如果客户要求我们为旅游结构构建预订系统，那么系统可能包含诸如住宿（例如酒店和早餐客栈）、房间、价格和预订等实体。

简化的数据库架构如下所示：

![实体创建](img/B04559_01_02.jpg)

# MyCompany 数据库架构

数据库架构有以下假设：

+   一个住宿有很多房间

+   预订仅适用于单个用户

+   预订可能包括多个房间

+   预订有一个开始日期和一个结束日期

+   价格从开始日期到结束日期对一个房间有效

+   一个房间有很多设施

+   预订的开始日期必须在结束日期之前

+   预订不能超过十五天

+   预订不能包括超过四个房间

# 使用 phpspec 进行设计

现在，让我们开始使用 phpspec 作为设计工具来构建我们的实体。

如果顶级命名空间是`MyCompany`，那么使用 phpspec，只需输入以下命令：

```php
**# phpspec describe MyCompany/AccommodationRepository**

```

在输入上述命令后，将创建`spec/AccommodationSpecRepository.php`：

```php
<?php

namespace spec\MyCompany;

use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class AccommodationRepositorySpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType('MyCompany\AccommodationRepository');
    }
<?php

namespace MyCompany;

class AccommodationRepository
{
}
```

### 提示

应将 phpspec 的路径添加到`.bashrc`或`.bash_profile`文件中，以便可以直接运行 phpspec。

然后，输入以下命令：

```php
**# phpspec run**

```

在输入上述命令后，开发人员将显示如下：

```php
**class MyCompany\AcccommodationRepository does not exist.**
**Do you want me to create 'MyCompany\AccommodationRepository' for you? [Y/n]**

```

输入*Y*后，将创建`AccommodationRepository.php`类，如下所示：

```php
<?php

namespace MyCompany;

class AccommodationRepository
{}
```

### 提示

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载示例代码文件，用于您购买的所有 Packt Publishing 图书。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

phpspec 的美妙之处在于其简单性和加速类的创建，这些类与规范一起。

![使用 phpspec 进行设计](img/B04559_01_03.jpg)

使用 phpspec 描述和创建类的基本步骤

# 使用 phpspec 进行规范说明

phpspec 的核心在于允许我们指定实体的行为并同时对其进行测试。通过简单地指定客户给出的业务规则，我们可以轻松为每个业务规则创建测试。然而，phpspec 的真正力量在于它如何使用表达自然语言的语法。让我们来看看之前给我们关于预订的业务规则：

+   预订的开始日期必顶在结束日期之前

+   预订不能超过十五天

+   预订不能包括超过四个房间

运行以下命令：

```php
**# phpspec describe**
 **MyCompany/Accommodation/ReservationValidator**

```

phpspec 将为上述命令产生以下输出：

```php
<?php

namespace spec\MyCompany\Accommodation;

use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class ReservationSpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType('MyCompany\Accommodation\Reservation');
    }
}
```

然后，使用以下命令运行 phpspec：

```php
**# phpspec run**

```

phpspec 将像往常一样以以下输出做出响应：

```php
**Do you want me to create** 
 **'MyCompany\Accommodation\ReservationValidator' for you?**

```

然后，phpspec 将创建`ReservationValidator`类，如下所示：

```php
<?php namespace MyCompany\Accommodation;

 class ReservationValidator {
 }
```

让我们创建一个`validate()`函数，它将采用以下参数： 

+   确定预订开始的开始日期字符串

+   确定预订结束的结束日期字符串

+   要添加到预订的`room`对象数组

以下是创建`validate()`函数的代码片段：

```php
<?php
namespace MyCompany\Accommodation;

use Carbon\Carbon;

class ReservationValidator
{

    public function validate($start_date, $end_date, $rooms)
    {
    }
}
```

我们将包括`Carbon`类，这将帮助我们处理日期。对于第一个业务规则，即预订的开始日期必须在结束日期之前，我们现在可以在`ReservationValidatorSpec`类中创建我们的第一个规范方法，如下所示：

```php
function its_start_date_must_come_before_the_end_date ($start_date,$end_date,$room)
{
    $rooms = [$room];
    $start_date = '2015-06-03';
    $end_date = '2015-06-03';
    $this->shouldThrow('\InvalidArgumentException')->duringValidate( $start_date, $end_date, $rooms);
}
```

在前面的函数中，phpspec 以`it`或`its`开始规范。phpspec 使用蛇形命名法以提高可读性，而`start_date_must_be_less_than_the_end_date`则是规范的精确副本。这不是很棒吗？

当传入`$start_date`，`$end_date`和`room`时，它们会自动被模拟。不需要其他任何东西。我们将创建一个有效的`$rooms`数组。然而，我们将设置`$start_date`和`$end_date`，使它们具有相同的值，以导致测试失败。表达式语法如前面的代码所示。`shouldThrow`出现在`during`之前，然后采用方法名`Validate`。

我们已经给了 phpspec 自动为我们创建`validate()`方法所需的东西。我们将指定`$this`，即`ReservationValidator`类，将抛出`InvalidArgumentException`。运行以下命令：

```php
**# phpspec run**

```

再次，phpspec 问我们以下问题：

```php
 **Do you want me to create 'MyCompany\Accommodation\Reservation::validate()'** 
 **for you?**

```

只需在提示处简单地输入*Y*，方法就会在`ReservationValidator`类中创建。就是这么简单。当再次运行 phpspec 时，它会因为方法尚未抛出异常而失败。所以现在需要编写代码。在函数内部，我们将从格式为"2015-06-02"的字符串创建两个`Carbon`对象，以便能够利用 Carbon 强大的日期比较功能。在这种情况下，我们将使用`$date1->diffInDays($date2);`方法来测试`$end`和`$start`之间的差异是否小于一。如果是这样，我们将抛出`InvalidArgumentException`并显示用户友好的消息。现在，当我们重新运行 phpspec 时，测试将通过：

```php
$end = Carbon::createFromFormat('Y-m-d', $end_date);
$start = Carbon::createFromFormat('Y-m-d', $start_date);

        if ($end->diffInDays($start)<1) {
            throw new \InvalidArgumentException('Requires end date to be greater than start date.');
        }
```

## 红，绿，重构

测试驱动开发的规则要求*红*，*绿*，*重构*，这意味着一旦测试通过（绿色），我们应该尝试重构或简化方法内的代码，而不改变功能。

看一下`if`测试：

```php
if ( $end->diffInDays($start) < 1 ) {
```

前面的代码不太可读。我们可以以以下方式重构它：

```php
if (!$end->diffInDays($start)>0)
```

然而，即使前面的代码也不太易读，我们还在代码中直接使用整数。

将`0`移入一个常量中。为了提高可读性，我们将其更改为预订所需的最少天数，如下所示：

```php
 const MINIMUM_STAY_LENGTH = 1;
```

让我们将比较提取到一个方法中，如下所示：

```php
    /**
     * @param $end
     * @param $start
     * @return bool
     */
    private function endDateIsGreaterThanStartDate($end, $start)
    {
        return $end->diffInDays($start) >= MINIMUM_STAY_LENGTH;
    }
```

我们现在可以这样写`if`语句：

```php
if (!$this->endDateIsGreaterThanStartDate($end, $start))
```

前面的陈述更加表达和可读。

现在，对于下一个规则，即预订不能超过十五天，我们需要以以下方式创建方法：

```php
function it_cannot_be_made_for_more_than_fifteen_days(User $user, $start_date, $end_date, Room $room)
{
        $start_date = '2015-06-01';
        $end_date = '2015-07-30';
        $rooms = [$room];
        $this->shouldThrow('\InvalidArgumentException')
        ->duringCreateNew( $user,$start_date,$end_date,$rooms);
}
```

在这里，我们设置`$end_date`，使其被分配一个比`$start_date`晚一个月以上的日期，以导致方法抛出`InvalidArgumentException`。再次执行`phpspec`命令后，测试将失败。让我们修改现有方法来检查日期范围。我们将向方法添加以下代码：

```php
  if ($end->diffInDays($start)>15) {
       throw new \InvalidArgumentException('Cannot reserve a room
       for more than fifteen (15) days.');
  }
```

再次，phpspec 愉快地成功运行所有测试。重构后，我们将再次提取`if`条件并创建常量，如下所示：

```php
   const MAXIMUM_STAY_LENGTH = 15;
   /**
     * @param $end
     * @param $start
     * @return bool
     */
    private function daysAreGreaterThanMaximumAllowed($end, $start)
    {
        return $end->diffInDays($start) > self::MAXIMUM_STAY_LENGTH;
    }

   if ($this->daysAreGreaterThanMaximumAllowed($end, $start)) {
            throw new \InvalidArgumentException ('Cannot reserve a room for more than fifteen (15) days.');
   }
```

## 整理一下

我们可以把事情留在这里，但是让我们清理一下，因为我们有测试。由于`endDateIsGreaterThanStartDate($end, $start)`和`daysAreGreaterThanMaximumAllowed($end, $start)`函数分别检查最小和最大允许的停留时间，我们可以从另一个方法中调用它们。

我们将`endDateIsGreaterThanStartDate()`重构为`daysAreLessThanMinimumAllowed($end, $start)`，然后创建另一个方法来检查最小和最大停留长度，如下所示：

```php
private function daysAreWithinAcceptableRange($end, $start)
    {
        if ($this->daysAreLessThanMinimumAllowed($end, $start)
            || $this->daysAreGreaterThanMaximumAllowed($end, $start)) {
           return false;
        } else {
           return true;
        }
    }
```

这样我们只剩下一个函数，而不是两个，在`createNew`函数中，如下所示：

```php
if (!$this->daysAreWithinAcceptableRange($end, $start)) {
            throw new \InvalidArgumentException('Requires a stay length from '
                . self::MINIMUM_STAY_LENGTH . ' to '. self::MAXIMUM_STAY_LENGTH . ' days.');
        }
```

对于第三条规则，即预订不能包含超过四个房间，流程是一样的。创建规范，如下：

```php
it_cannot_contain_than_four_rooms
```

这里的改变将在参数中。这次，我们将模拟五个房间，以便测试失败，如下所示：

```php
function it_cannot_contain_than_four_rooms(User $user, $start_date, $end_date, Room $room1, Room $room2, Room $room3, Room $room4, Room $room5)
```

五个房间对象将被加载到`$rooms`数组中，测试将会失败，如下所示：

```php
$rooms = [$room1, $room2, $room3, $room4, $room5];
    $this->shouldThrow('\InvalidArgumentException')->duringCreateNew($user,$start_date,$end_date,$rooms);
    }
```

在添加代码以检查数组大小后，最终类将如下所示：

```php
<?php

namespace MyCompany\Accommodation;

use Carbon\Carbon;
class ReservationValidator
{

    const MINIMUM_STAY_LENGTH = 1;
    const MAXIMUM_STAY_LENGTH = 15;
    const MAXIMUM_ROOMS = 4;

    /**
     * @param $start_date
     * @param $end_date
     * @param $rooms
     * @return $this
     */
    public function validate($start_date, $end_date, $rooms)
    {
        $end = Carbon::createFromFormat('Y-m-d', $end_date);
        $start = Carbon::createFromFormat('Y-m-d', $start_date);

        if (!$this->daysAreWithinAcceptableRange($end, $start)) {
            throw new \InvalidArgumentException('Requires a stay length from '
                . self::MINIMUM_STAY_LENGTH . ' to '. self::MAXIMUM_STAY_LENGTH . ' days.');
        }
        if (!is_array($rooms)) {
            throw new \InvalidArgumentException('Requires last parameter rooms to be an array.');
        }
        if ($this->tooManyRooms($rooms)) {
            throw new \InvalidArgumentException('Cannot reserve more than '. self::MAXIMUM_ROOMS .' rooms.');
        }

        return $this;

    }

    /**
     * @param $end
     * @param $start
     * @return bool
     */
    private function daysAreLessThanMinimumAllowed($end, $start)
    {
        return $end->diffInDays($start) < self::MINIMUM_STAY_LENGTH;
    }

    /**
     * @param $end
     * @param $start
     * @return bool
     */
    private function daysAreGreaterThanMaximumAllowed($end, $start)
    {
        return $end->diffInDays($start) > self::MAXIMUM_STAY_LENGTH;
    }

    /**
     * @param $end
     * @param $start
     * @return bool
     */
    private function daysAreWithinAcceptableRange($end, $start)
    {
        if ($this->daysAreLessThanMinimumAllowed($end, $start)
            || $this->daysAreGreaterThanMaximumAllowed($end, $start)) {
            return false;
        } else {
            return true;
        }
    }

    /**
     * @param $rooms
     * @return bool
     */
    private function tooManyRooms($rooms)
    {
        return count($rooms) > self::MAXIMUM_ROOMS;
    }

    public function rooms(){
        return $this->belongsToMany('MyCompany\Accommodation\Room')->withTimestamps();
    }

}
```

这种方法非常干净。只有两个`if`语句——第一个用于验证日期范围是否有效，另一个用于验证房间数量是否在有效范围内。常量很容易访问，并且可以根据业务需求进行更改。显然，将 phpspec 添加到开发工作流程中，将之前需要两个步骤——使用 PHPUnit 编写断言，然后编写代码——合并在一起。现在，我们将离开 phpspec，转而使用 Artisan，开发人员对此很熟悉，因为它是 Laravel 先前版本的一个特性。

# 控制器

接下来，我们将创建一些示例控制器。在撰写本书时，我们需要同时使用 Artisan 和 phpspec。让我们为`room`实体创建一个控制器，如下所示：

```php
$ php artisan make:controller RoomController

<?php namespace MyCompany\Http\Controllers;

use MyCompany\Http\Requests;
use MyCompany\Http\Controllers\Controller;

use Illuminate\Http\Request;
class RoomController extends Controller {

        /**
        * Display a listing of the resource.
        *
        * @return Response
        */
        public function index()
        {}

        /**
        * Show the form for creating a new resource.
        *
        * @return Response
        */
        public function create()
        {}

        /**
        * Store a newly created resource in storage.
        *
        * @return Response
        */
        public function store()
        {}
….

}
```

### 注意

请注意，这将在`app/Http/Controllers`目录中创建，这是 Laravel 5 的新位置。新的 HTTP 目录包含控制器、中间件和请求目录，将与 HTTP 请求或实际请求相关的文件分组在一起。此外，此目录配置是可选的，路由可以调用任何自动加载的位置，通常通过命名空间 PSR-4 结构。

## 命令总线

Laravel 5 采用了命令总线模式，创建的命令存储在`app/Commands`目录中。而在 Laravel 4 中，命令被认为是命令行工具，而在 Laravel 5 中，命令被认为是一个类，其方法可以在应用程序内部使用，从而实现代码的优秀重用。这里的命令概念是需要完成的任务，或者在我们的例子中，是为用户预订的房间。总线的范式然后使用新的`DispatchesCommands`特性传输命令，该特性用于基本控制器类中。Artisan 创建的每个控制器都扩展了这个类到一个处理程序方法，实际工作在其中执行。

为了使用 Laravel 的命令总线设计模式，我们现在将使用 Artisan 创建一些命令。我们将在未来的章节中详细介绍命令，但首先，我们将输入以下命令：

```php
**$ php artisan make:commandReserveRoomCommand --handler**

```

输入此命令将创建一个用于预订房间的命令，可以从代码的任何位置调用，将业务逻辑与控制器和模型隔离，并允许以异步模式执行命令。

```php
<?php namespace MyCompany\Commands;

use MyCompany\Commands\Command;

class ReserveRoomCommand extends Command {

    /**
    * Create a new command instance.
    *
    * @return void
    */
    public function __construct()
    {
        //
    }

}
```

填写完命令的细节后，该类现在看起来是这样的：

```php
<?php namespace MyCompany\Commands;

use MyCompany\Commands\Command;
use MyCompany\User;

class ReserveRoomCommand extends Command {

    public $user;
    public $rooms;
    public $start_date;
    public $end_date;

    /**
    * Create a new command instance.
    *
    * @return void
    */
    public function __construct(User $user, $start_date, $end_date, $rooms)
    {
        $this->rooms = $rooms;
        $this->user = $user;
        $this->start_date = $start_date;
        $this->end_date = $end_date;
    }

}
```

`--handler`参数创建了一个额外的类`ReserveRoomCommandHandler`，其中包含一个构造函数和一个 handle 方法，该方法注入了`ReserveRoomCommand`。此文件将存在于`app/Handlers/Commands`目录中。如果未使用`--handler`标志，则`ReserveRoomCommand`类将包含自己的`handler`方法，并且不会创建单独的处理程序类：

```php
<?php namespace MyCompany\Handlers\Commands;

use MyCompany\Commands\ReserveRoomCommand;

use Illuminate\Queue\InteractsWithQueue;

class ReserveRoomCommandHandler {

    /**
    * Create the command handler.
    *
    * @return void
    */
    public function __construct()
    {
        //
    }

    /**
    * Handle the command.
    *
    * @paramReserveRoomCommand  $command
    * @return void
    */
    public function handle(ReserveRoomCommand $command)
    {
        //
    }

}
```

我们将填写处理预订验证的 handle 方法，如下所示：

```php
public function handle(ReserveRoomCommand $command)
    {
        $reservation = new \MyCompany\Accommodation\ReservationValidator();
        $reservation->validate(
        $command->start_date, $command->end_date, $command->rooms);
    } 
```

# 总结

phpspec 为软件的业务逻辑方面添加了成熟、健壮、测试驱动和示例驱动的规范方法。再加上模型、控制器、命令、事件和事件处理程序的轻松创建，使得 Laravel 成为 PHP 框架竞争中的佼佼者。此外，它还采用了许多行业最佳程序员使用的最佳实践。

在本章中，我们学习了如何使用 phpspec 轻松地从命令行设计类及其相应的测试。这种工作流程，加上 Artisan，使得设置 Laravel 5 应用程序的基本结构变得非常容易。

在下一章中，我们将介绍数据库迁移、其背后的机制以及创建用于测试的种子的方法。
