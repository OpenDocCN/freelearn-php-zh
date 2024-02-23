# 第三章：构建服务、命令和事件

在前两章中，我们建立了我们的住宿预订系统的基本结构。我们设计了我们的类，创建了我们的数据库模式，并学会了如何测试它们。现在我们需要将业务需求转化为代码。

在本章中，我们将涵盖以下主题：

+   命令

+   事件

+   命令处理程序

+   事件处理程序

+   排队的事件处理程序

+   排队的命令

+   控制台命令

+   命令调度程序

# 请求路由

如前所述，Laravel 5 采用了*命令总线模式*。Laravel 4 将命令视为从命令行执行的内容，而在 Laravel 5 中，命令可以在任何上下文中使用，从而实现代码的优秀重用。

以下是 Laravel 4 的 HTTP 请求流程示例：

![请求路由](img/B04559_03_01.jpg)

以下是 Laravel 5 的 HTTP 请求流程示例：

![请求路由](img/B04559_03_02.jpg)

第一张图片说明了 Laravel 4 的请求流程。通过 HTTP 的请求由路由处理，然后发送到控制器，通常情况下，我们可以与存储库或模型的目录进行交互。在 Laravel 5 中，这仍然是可能的；然而，正如第二张图片所示，我们可以看到添加额外的块、层或模块的能力使我们能够将请求的生命周期分离成单独的部分。Laravel 4 允许我们将处理请求的所有代码放在控制器内，而在 Laravel 5 中，我们可以自由地做同样的事情，尽管现在我们也能够轻松地将请求分离成各种部分。其中一些概念源自**领域驱动设计**（**DDD**）。

在控制器内，使用**数据传输对象**（**DTO**）范例实例化命令。然后，命令被发送到命令总线，在那里由处理程序类处理，该类有两个方法：`__construct()`和`handle()`。在处理程序内部，我们触发或实例化一个事件。同样，事件也由事件处理程序方法处理，该方法有两个方法：`__construct()`和`handle()`。

目录结构非常清晰，如下所示：

```php
**/app/Commands**
**/app/Events/**
**/app/Handlers/**
**/app/Handlers/Commands**
**/app/Handlers/Events**
**/app/HTTP/Controllers**

```

这相当直观；命令和事件分别在它们各自的目录中，而每个处理程序都有自己的目录。

### 注意

Laravel 5.1 已将`app/Commands`目录的名称更改为`app/Jobs`，以确保程序员不会混淆命令总线和控制台命令的概念。

## 用户故事

命令组件的想法可以很容易地从用户故事或用户为实现目标而需要的任务中得出。最简单的例子是搜索一个房间：

```php
As a hotel website user,
I want to search for a room
so that I can select from a list of results.
```

源自敏捷方法论的用户故事保证编写的代码与业务需求紧密匹配。它们通常遵循“作为…我想要…以便…”的模式。这定义了`角色`、`意图`和`利益`。它帮助我们计划如何将每个任务转换为代码。在我们的例子中，用户故事可以转化为任务。

作为酒店网站用户，我会创建以下任务列表：

1.  作为酒店网站用户，我希望搜索房间，以便我可以从结果列表中选择一个房间。

1.  作为酒店网站用户，我希望预订一个房间，以便我可以住在酒店里。

1.  作为酒店网站用户，我希望收到包含预订详情的电子邮件，以便我可以拥有预订的副本。

1.  作为酒店网站用户，我希望在等候名单上，以便我可以在有房间可用时预订一个房间。

1.  作为酒店网站用户，我希望收到房间的可用性通知，以便我可以预订房间。

## 用户故事转换为代码

搜索房间的第一个任务很可能是来自用户或外部服务的 RESTful 调用，因此这个任务会暴露给我们的控制器，从而暴露给我们的 RESTful API。

第二个任务，预订房间，是由用户或其他服务发起的类似操作。这个任务可能需要用户登录。

第三个任务可能取决于第二个任务。这个任务需要与另一个过程进行交互，向用户发送包含预订详情的确认电子邮件。我们也可以这样写：*作为酒店网站，我想发送一封带有预订详情的电子邮件，以便他或她可以拥有预订的副本*。

第四个任务，加入等待列表，可能是在发出预订房间请求后执行的命令；如果另一个用户同时预订了房间。它很可能是从应用程序本身而不是用户那里调用的，因为用户对实时住宿库存没有了解。这可以帮助我们处理竞争条件。此外，我们应该假设当网站用户决定预订哪个房间时，该房间没有锁定机制来保证可用性。我们也可以这样写：*作为酒店网站，我想将用户放在等待列表中，以便在房间可用时通知他们*。

对于第五个任务，当用户被放在等待列表上时，用户也可以在房间可用时收到通知。此操作检查房间的可用性，然后检查等待列表上的任何用户。用户故事可以重写如下：*作为酒店网站，我想通知等待列表用户房间的可用性，以便他或她可以预订房间*。如果房间变得可用，等待列表上的第一个用户将通过电子邮件收到可用性通知。这个命令将经常执行，就像是一个定时任务。幸运的是，Laravel 5 有一种新的机制，允许命令以给定的频率执行。

很明显，如果用户故事必须以使用网站作为行动者（“作为酒店网站...”）或网站用户作为行动者（“作为酒店网站用户...”）来编写，命令是有用的，并且可以从 RESTful API（用户端）或 Laravel 应用程序内部启动。

由于我们的第一个任务很可能涉及外部服务，我们将创建一个路由和一个控制器来处理请求。

## 控制器

第一步涉及创建一个路由，第二步涉及创建一个控制器。

### 搜索房间

首先，在`routes.php`文件中创建一个路由，并将其映射到`controller`方法，如下所示：

```php
Route::get('search', 'RoomController@search');
```

请求参数，如开始/结束日期和位置详情将如下所示：

```php
{
  "start_date": "2015-07-10"
  "end_date": "2015-07-17"
  "city": "London"
  "country": "England"
}
```

搜索参数将以 JSON 编码的对象形式发送。它们将发送如下：

```php
http://websiteurl.com/search?query={%22start_date%22:%222015-07-10%22,%22end_date%22:%222015-07-17%22,%22city%22:%22London%22,%22country%22:%22England%22}
```

现在，让我们在我们的`room`控制器中添加一个`search`方法，以处理以对象形式输入的 JSON 请求，如下所示：

```php
/**
* Search for a room in an accommodation
*/
public function search()
{
      json_decode(\Request::input('query'));
}
```

请求外观处理输入变量查询，然后将其 JSON 结构解码为对象。

在第四章中，*创建 RESTful API*，我们将完成`search`方法的代码，但现在，我们将简单地创建我们的 RESTful API 系统的这一部分的架构。

### 控制器转命令

对于第二个任务，预订房间，我们将创建一个命令，因为我们很可能需要后续操作，我们将通过发布者订阅者模式启用。发布者订阅者模式用于表示发送消息的*发布者*和监听这些消息的*订阅者*。

将以下路由添加到`routes.php`中：

```php
**Route::post('reserve-room', 'RoomController@store');**

```

我们将 post 映射到 room 控制器的`store`方法；这将创建预订。记住我们创建了这样的命令：

```php
**$ php artisan make:commandReserveRoomCommand -–handler**

```

我们的`ReserveRoomCommand`类如下所示：

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

我们需要将以下属性添加到构造函数中：

```php
    public $user;
    public $rooms;
    public $start_date;
    public $end_date;
```

此外，将以下赋值添加到构造函数中：

```php
        $this->rooms = $rooms;
        $this->user = $user;
        $this->start_date = $start_date;
        $this->end_date = $end_date;
```

这使我们能够传递值。

### 命令转事件

现在让我们创建一个事件。使用`artisan`创建一个事件`RoomWasReserved`，当房间被创建时触发：

```php
**$ phpartisan make:eventRoomWasReserved**

```

`RoomWasReserved`事件类看起来像以下代码片段：

```php
<?php namespace MyCompany\Events;

use MyCompany\Accommodation\Reservation;
use MyCompany\Events\Event;
use MyCompany\User;

use Illuminate\Queue\SerializesModels;

class RoomWasReserved extends Event {

    use SerializesModels;

    private $user;
    private $reservation;

    /**
    * Create a new event instance.
    *
    * @return void
    */
    public function __construct(User $user, Reservation $reservation)
    {
        $this->user = $user;
        $this->reservation = $reservation;
    }
}
```

我们将告诉它使用`MyCompany\Accommodation\Reservation`和`MyCompany\User`实体，以便我们可以将它们传递给构造函数。在构造函数内部，我们将它们分配给`event`对象内的实体。

现在，让我们从命令处理程序内部触发事件。Laravel 为您提供了一个简单的`event()`方法作为一个方便/辅助方法，它将触发一个事件。我们将实例化的预订和`user`注入`RoomWasReserved`事件如下：

```php
**event(new RoomWasReserved($user, $reservation));**

```

### `ReserveRoomCommandHandler`类

我们的`ReserveRoomCommandHandler`类现在实例化一个新的预订，使用`createNew`工厂方法来注入依赖项，最后，触发`RoomWasReserved`事件如下：

```php
<?phpnamespace MyCompany\Handlers\Commands;

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

        $reservationValidator = new \MyCompany\Accommodation\ReservationValidator();

        if ($reservationValidator->validate($command->start_date,$command->end_date,$command->rooms)) {
              $reservation = 
                $reservationRepository->create(
                ['date_start'=>$command->$command→start_date,
                'date_end'=>$command->end_date,
                'rooms'=>$command->'rooms']);
        }
    $reservation = new 
      event(new RoomWasReserved($command->user,$reservation));
    }
}
```

### 事件到处理程序

现在，我们需要创建事件处理程序。正如您所期望的那样，Artisan 提供了一个方便的方法来做到这一点，尽管语法有点不同。这一次，奇怪的是，*make*这个词没有出现在短语中：

```php
**$ php artisan handler:eventRoomReservedEmail --event=RoomWasReserved**
 **<?php namespace MyCompany\Handlers\Events;**

 **use MyCompany\Events\RoomWasReserved;**

 **use Illuminate\Queue\InteractsWithQueue;**
 **use Illuminate\Contracts\Queue\ShouldBeQueued;**

 **class RoomReservedEmail {**

 **/****
 *** Create the event handler.**
 *** @return void**
 ***/**
 **public function __construct()**
 **{**
 **}**

 **public function handle(RoomWasReserved $event)**
 **{**
 **//TODO: send email to $event->user**
 **//TODO: with details about $event->reservation;**
 **}**
 **}**

```

现在我们需要将事件连接到其监听器。我们将编辑`app/Providers/EventServiceProvider.php`文件如下：

```php
protected $listen = [
    'MyCompany\Events\RoomWasReserved' => [
      'MyCompany\Handlers\Events\RoomReservedEmail',
      ],
    ];
```

如前面的代码片段所示，我们将向`$listen`数组添加键值对。如所示，需要完整路径作为键，事件名称和处理程序数组。在这种情况下，我们只有一个处理程序。

## 排队的事件处理程序

如果我们不希望事件立即处理，而是放入队列中，我们可以在创建命令中添加`-queued`如下：

```php
**$ php artisan handler:eventRoomReservedEmail --event=RoomWasReserved --queued**

```

```php
 **<?php namespace MyCompany\Handlers\Events;**

 **use MyCompany\Events\RoomWasReserved;**

 **use Illuminate\Queue\InteractsWithQueue;**
 **use Illuminate\Contracts\Queue\ShouldBeQueued;**

 **class RoomReservedEvent implements ShouldBeQueued {**

 **use InteractsWithQueue;**

 **public function __construct()**
 **{**
 **//**
 **}**

 **use Illuminate\Contracts\Queue\ShouldBeQueued;**

```

这个接口告诉 Laravel 事件处理程序应该被排队，而不是同步执行：

```php
use Illuminate\Queue\InteractsWithQueue;
```

这个 trait 允许我们与队列交互，以便执行任务，比如删除任务。

## 等待列表命令

对于第四个任务，被放置在等待列表中，我们需要创建另一个命令，该命令将从预订控制器内部调用。再次使用 Artisan，我们可以轻松地创建命令及其相应的事件如下：

```php
**$ php artisan make:commandPlaceOnWaitingListCommand**
**$ php artisan make:eventPlacedOnWaitinglist**

```

现在，在我们的预订控制器中，我们将添加`roomAvailability`的检查，然后按以下方式分派`PlaceOnWaitinglist`命令：

```php
public function store()
    {
    …
    …
        if ($roomAvailable) {
            $this->dispatch(
              new ReserveRoomCommand( $start_date, $end_date, $rooms)
            );
        } else {
            $this->dispatch(
              new PlaceOnWaitingListCommand($start_date, $end_date, $rooms)
            );
        }
    …
```

## 排队的命令

通过在`create`命令中添加`queued`，我们可以轻松地将命令加入队列：

```php
**$ php artisan make:commandReserveRoomCommand -–handler --queued**

```

这将使用可用的任何队列系统，比如 beanstalkd，并不会立即运行命令。相反，它将被放置在队列中，并稍后运行。我们需要为`Command`类添加一个接口：

```php
**Illuminate\Contracts\Queue\ShouldBeQueued**

```

在这种情况下，`ReserveRoomCommand`类将如下所示：

```php
<?php namespace MyCompany\Commands;

use MyCompany\Commands\Command;

use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldBeQueued;

class MyCommand extends Command implements ShouldBeQueued {

	use InteractsWithQueue, SerializesModels;

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

在这里，我们可以看到`InteractsWithQueue`和`ShouldBeQueued`类已经被包含，`ReserveRoomCommand`类扩展了命令并实现了`ShouldBeQueued`类。另一个有趣的特性是`SerializesModels`。这将序列化传递的任何模型，以便稍后使用。

## 控制台命令

对于第五个任务，让我们创建一个`console`命令，这个命令将经常被执行：

```php
**$ php artisan make:consoleManageWaitinglist**

```

这将创建一个可以从 Artisan 命令行工具执行的命令。如果您使用过 Laravel 4，您可能对这种类型的命令很熟悉。这些命令存储在`Console/Commands/`目录中。

为了让 Laravel 知道这一点，我们需要将它添加到`app/Console/Kernel.php`中的`$commands`数组中：

```php
protected $commands = [
    'MyCompany\Console\Commands\Inspire',
    'MyCompany\Console\Commands\ManageWaitinglist',
    ];
```

内容如下：

```php
<?php namespace MyCompany\Console\Commands;

use Illuminate\Console\Command;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class ManageWaitinglist extends Command {

    /**
    * The console command name.
    *
    * @var string
    */
    protected $name = 'command:name';

    /**
    * The console command description.
    *
    * @var string
    */
    protected $description = 'Command description.';

    /**
    * Create a new command instance.
    *
    * @return void
    */
    public function __construct()
    {
        parent::__construct();
    }

    /**
    * Execute the console command.
    *
    * @return mixed
    */
    public function fire()
    {
        //
    }

    /**
    * Get the console command arguments.
    *
    * @return array
    */
    protected function getArguments()
    {
        return [
          ['example', InputArgument::REQUIRED, 'An example argument.'],
        ];
    }

    /**
    * Get the console command options.
    *
    * @return array
    */
    protected function getOptions()
    {
        return [
          ['example', null, InputOption::VALUE_OPTIONAL, 'An example option.', null],
        ];
    }
}
```

`$name`属性是从 Artisan 调用的名称。例如，如果我们设置如下：

```php
protected $name = 'manage:waitinglist';
```

然后，通过运行以下命令，我们可以管理等待列表：

```php
**$ php artisan manage:waitinglist**

```

`getArguments()`和`getOptions()`方法是具有相同签名的类似方法，但用途不同。

`getArguments()`方法指定了必须用于启动命令的参数数组。`getOptions()`方法用`-`指定，并且可以是`optional`、`repeated`，并且使用`VALUE_NONE`选项，它们可以简单地用作标志。

我们将在`fire()`方法中编写命令的主要代码。如果我们想要从该命令中调度一个命令，我们将在类中添加`DispatchesCommands` trait，如下所示：

```php
 **use DispatchesCommands;**

 **<?php namespace MyCompany\Console\Commands;**

 **use Illuminate\Console\Command;**
 **use Illuminate\Foundation\Bus\DispatchesCommands;**
 **use Symfony\Component\Console\Input\InputOption;**
 **use Symfony\Component\Console\Input\InputArgument;**

 **class ManageWaitinglist extends Command {**

 **use DispatchesCommands;**

 **/****
 *** The console command name.**
 *** @var string**
 ***/**
 **protected $name = 'manage:waitinglist';**

 **/****
 *** The console command description.**
 *** @var string**
 ***/**
 **protected $description = 'Manage the accommodation waiting list.';**

 **/****
 *** Create a new command instance.**
 *****
 *** @return void**
 ***/**
 **public function __construct()**
 **{**
 **parent::__construct();**
 **}**

 **/****
 *** Execute the console command.**
 *** @return mixed**
 ***/**
 **public function fire()**
 **{**
 **// TODO: write business logic to manage waiting list**
 **if ($roomIsAvailableFor($user)) {**
 **$this->dispatch(new ReserveRoomCommand());**
 **}**
 **}**

 **/****
 *** Get the console command arguments.**
 *** @return array**
 ***/**
 **protected function getArguments()**
 **{**
 **return [];**
 **}**

 **/****
 *** Get the console command options.**
 *** @return array**
 ***/**
 **protected function getOptions()**
 **{**
 **return [];**
 **}**
**}**

```

## 命令调度程序

现在，我们将安排此命令每 10 分钟运行一次。传统上，这是通过创建一个 cron 作业来执行 Laravel 控制台命令来完成的。现在，Laravel 5 提供了一个新的机制来做到这一点——命令调度程序。

新的`artisan`命令的运行方式如下：

```php
**$ php artisan schedule:run**

```

通过简单地将此命令添加到 cron 中，Laravel 将自动运行`Kernel.php`文件中的所有命令。

命令需要添加到`Schedule`函数中，如下所示：

```php
protected function schedule(Schedule $schedule)
    {
        $schedule->command('inspire')
             ->hourly();
        $schedule->command('manage:waitinglist')
            ->everyFiveMinutes();

    }
```

`inspire`命令是 Laravel 提供的一个示例命令，用于演示功能。我们将简单地添加我们的命令。这将每 5 分钟调用`manage:waitinglist`命令——比这更简单的方式都没有了。

现在我们需要修改`crontab`文件以使 Artisan 运行调度程序。

`crontab`是一个包含在特定时间运行的命令的文件。要修改此文件，请键入以下命令：

```php
**$ sudo crontab -e**

```

我们将使用`vi`或分配的编辑器来修改`cron`表。添加以下行将告诉`cron`每分钟运行调度程序：

```php
*** * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1**

```

# 总结

Laravel 在短短两年内发生了变化，从 CodeIgniter 的模型-视图-控制器范式转变为采用现代领域驱动设计的命令总线和发布者-订阅者事件监听器模式。是否使用这些模式将取决于所需的每个层之间的分离程度。当然，即使使用自处理命令也是开始创建完全独立的代码块的一种方式，这将促使代码进入一个单独的处理程序类，进一步实现关注点分离原则。通过减少控制器内的代码量，命令变得更加重要。

我们甚至还没有为每个用户故事编写与数据库交互的代码，我们只是对数据库进行了种子和测试，但结构开始变得非常设计良好；每个类都有一个非常有意义的名称，并且组织成一个有用的目录结构。

在下一章中，我们将填写有关 RESTful 控制器如何接受来自另一个系统或网站前端的输入，以及模型属性如何返回给用户以创建界面的详细信息。
