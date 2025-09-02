# 附录 A. 工具箱

Laravel 提供了一些实用工具，可以帮助你执行特定任务，例如发送电子邮件、排队函数和文件操作。它自带了大量内部使用的实用工具；好消息是，你也可以在应用程序中使用它们。本章将介绍最有用的实用工具，这样你就不会重写框架中已经存在的函数！

本章的结构部分基于*Jesse O'Brien*的速查表，可在[`cheats.jesse-obrien.ca/`](http://cheats.jesse-obrien.ca/)找到。示例基于 Laravel 的测试以及其官方文档和 API。

# 数组辅助函数

对于任何处理数据的 Web 应用程序来说，数组都是基础。PHP 已经提供了近 80 个函数来执行数组上的各种操作，而 Laravel 通过一些受 Python 和 Ruby 中某些函数启发的实用函数来补充它们。

### 注意

Laravel 的几个类，包括 Eloquent 集合，实现了 PHP 的`ArrayAccess`接口。这意味着你可以在代码中使用它们像普通数组一样，例如，在`foreach`循环中迭代项目或使用这里描述的数组函数。

大多数函数支持点符号来引用嵌套值，这与 JavaScript 对象类似。例如，你不必编写`$arr['foo']['bar']['baz']`，而是可以使用`array_get`辅助函数并编写`array_get($arr, 'foo.bar.baz');`。

在以下使用示例中，我们将使用三个虚拟数组，并假设每个示例都会重置它们：

```php
$associative = [
  'foo' => 1,
  'bar' => 2,
];
$multidimensional = [
  'foo' => [
      'bar' => 123,
  ],
];
$list_key_values = [
  ['foo' => 'bar'], 
  ['foo' => 'baz'],
];
```

## 数组辅助函数的使用示例

我们现在将看看如何使用 Laravel 的数组辅助函数提取和操作这些数组的值：

+   要在键不存在时使用回退值检索值，我们使用`array_get`函数，如下所示：

    ```php
    array_get($multidimensional, 'foo.bar', 'default');
    // Returns 123
    ```

    如果你引用的数组键可能存在或不存在（即在请求数据数组中），这将很有用。如果键不存在，则将返回默认值。

+   要使用点符号从数组中删除值，我们使用`array_forget`函数，如下所示：

    ```php
    array_forget($multidimensional, 'foo.bar');
    // $multidimensional == ['foo' => []];
    ```

+   要从数组中删除值并返回它，我们使用`array_pull`函数，如下所示：

    ```php
    array_pull($multidimensional, 'foo.bar');
    // Returns 123 and removes the value from the array
    ```

+   要使用点符号设置嵌套值，我们使用`array_set`函数，如下所示：

    ```php
    array_set($multidimensional, 'foo.baz', '456');
    // $multidimensional == ['foo' => ['bar' => 123, 'baz' => '456']];
    ```

+   要将多维关联数组展平，我们使用`array_dot`函数，如下所示：

    ```php
    array_dot($multidimensional);
    // Returns ['foo.bar' => 123];
    array_dot($list_key_values);
    // Returns ['0.foo' => 'bar', '1.foo' => 'baz'];
    ```

+   要从数组中返回所有键及其值（除了指定的那些），我们使用`array_except`函数，如下所示：

    ```php
    array_except($associative, ['foo']);
    // Returns ['bar' => 2];
    ```

+   要只从数组中提取一些键，我们使用`array_only`函数，如下所示：

    ```php
    array_only($associative, ['bar']);
    // Returns ['bar' => 2];
    ```

+   要返回包含所有嵌套值（键被丢弃）的展平数组，我们使用`array_fetch`函数，如下所示：

    ```php
    array_fetch($list_key_values, 'foo');
    // Returns ['bar', 'baz'];
    ```

+   要遍历数组并返回闭包返回 true 的第一个值，我们使用`array_first`函数如下：

    ```php
    array_first($associative, function($key, $value) {
       return $key == 'foo';
    });
    // Returns 1
    ```

+   要生成一个只包含在多维数组中找到的值的单维数组，我们使用`array_flatten`函数如下：

    ```php
    array_flatten($multidimensional);
    // Returns [123]
    ```

+   要从键值对列表中提取值数组，我们使用`array_pluck`函数如下：

    ```php
    array_pluck($list_key_values, 'foo');
    // Returns ['bar', 'baz'];
    ```

+   要获取数组的第一个或最后一个项目（这也适用于函数返回的值），我们使用`head`和`last`函数如下：

    ```php
    head($array); // Aliases to reset($array)
    last($array); // Aliases to end($array)
    ```

# 字符串和文本操作

字符串操作函数位于`Illuminate\Support`命名空间中，并且可以在`Str`对象上调用。

大多数函数也有更短的`snake_case`别名。例如，`Str::endsWith()`方法与全局`ends_with()`函数相同。我们可以在应用程序中自由选择我们喜欢的任何一个。

## 布尔函数

以下函数返回`true`或`false`值：

+   `is`方法检查一个值是否与一个模式匹配。星号可以用作通配符，如下所示：

    ```php
    Str::is('projects/*', 'projects/php/'); // Returns true
    ```

+   如以下代码所示，`contains`方法检查一个字符串是否包含给定的子字符串：

    ```php
    Str::contains('Getting Started With Laravel', 'Python');
    // returns false
    ```

+   `startsWith`和`endsWith`方法，如以下代码所示，检查一个字符串是否以一个或多个子字符串开始或结束：

    ```php
    Str::startsWith('.gitignore', '.git'); // Returns true
    Str::endsWith('index.php', ['html', 'php']); // Returns true
    ```

如前所述的示例所示，这些方法对于验证文件名和类似数据非常方便。

## 变换函数

在某些情况下，您需要在向用户显示或将其用于 URL 之前转换字符串。Laravel 提供了以下辅助函数来实现这一点：

+   此函数生成一个对 URL 友好的字符串：

    ```php
    Str::slug('A/B testing is fun!');
    // Returns "ab-testing-is-fun"
    ```

+   此函数生成一个每个单词都大写的标题：

    ```php
    Str::title('getting started with laravel');
    // Returns 'Getting Started With Laravel'
    ```

+   此函数将给定字符的实例添加到字符串的开头：

    ```php
    Str::finish('/one/trailing/slash', '/');
    Str::finish('/one/trailing/slash/', '/');
    // Both will return '/one/trailing/slash/'
    ```

+   此函数限制字符串中的**字符**数量：

    ```php
    Str::limit($value, $limit = 100, $end = '...')
    ```

+   此函数限制字符串中的**单词**数量：

    ```php
    Str::words($value, $words = 100, $end = '...')
    ```

## 逆变换函数

以下函数帮助您找出单词的复数或单数形式，即使它是不规则的形式：

+   此函数找出单词的复数形式：

    ```php
    Str::plural('cat');
    // Returns 'cats'
    Str::plural('fish');
    // Returns 'fish'
    Str::plural('monkey');
    // Returns 'monkeys'

    ```

+   此函数找出单词的单数形式：

    ```php
    Str::singular('elves');
    // Returns 'elf'
    ```

# 文件处理

Laravel 5 包含了优秀的**Flysystem**项目，用于与应用程序文件系统以及流行的基于云的存储解决方案（如**Amazon Simple Storage Service**（**Amazon S3**）和**Rackspace**）交互。文件系统在`config/filesystems.php`文件中配置为*磁盘*。然后您可以使用一致的 API 来管理文件，无论它们位于本地还是外部云存储中。

直接在`Storage`外观上调用方法将在默认磁盘上调用这些方法，如下所示：

```php
Storage::exists('foo.txt');
```

如果您配置了多个磁盘，您也可以显式指定要执行操作的磁盘，如下所示：

```php
Storage::disk('local')->exists('foo.txt');
```

您可以按照如下方式读取和写入文件：

```php
Storage::put('foo.txt', $contents);
$contents = Storage::get('foo.txt');
```

您也可以按照如下方式添加或附加数据：

```php
Storage::prepend('foo.txt', 'Text to prepend.');
Storage::append('foo.txt', 'Text to append.');
```

你可以使用命名恰当的方法来复制和移动文件，如下所示：

```php
Storage::copy($source, $destination);
Storage::move($source, $destination);
```

你还可以通过提供要删除的文件数组来删除文件，无论是单个文件还是一次删除多个文件，如下所示：

```php
Storage::delete('foo.txt');
Storage::delete(['foo.txt', 'bar.txt']);
```

此外，还有一些其他有用的方法，允许你获取有关文件的有用信息，如下所示：

```php
Storage::size('foo.txt');
Storage::lastModified('foo.txt');
```

除了处理文件外，你还可以处理目录。要列出特定目录中的所有文件，请使用以下代码：

```php
Storage::files('path/to/directory');
```

前面的代码将只列出当前目录中的文件。如果你想要递归地列出所有文件（即当前目录及其子目录中的文件），那么你可以使用`allFiles`方法，如下所示：

```php
Storage::allFiles('path/to/directory');
```

你可以按照以下方式创建目录：

```php
Storage::makeDirectory('path/to/directory');
```

你也可以按照以下方式删除目录：

```php
Storage::deleteDirectory('path/to/directory');
```

## 文件上传

在 Laravel 5 中处理文件上传很容易。第一步是创建一个在提交时发送文件的表单：

```php
{!! Form::open(['files' => true) !!}
```

这将设置`enctype`属性为`multipart/form-data`。然后你需要一个 HTML 的`file`输入：

```php
{!! Form::file('avatar') !!}
```

在提交时，你可以在控制器操作中按照以下方式从`Request`对象访问文件：

```php
public function store(Request $request){$file = $request->file('avatar');}
```

从这里，你通常会将文件移动到你的选择目录：

```php
public function store(Request $request)
{
  $file = $request->file('avatar');
  $file->move(storage_path('uploads/avatars'));
}
```

在前面的示例中，`$file` 是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的一个实例，它提供了一系列方便的方法来与上传的文件交互。

你可以按照以下方式获取文件的完整路径：

```php
$path = $request->file('avatar')->getRealPath();
```

你可以按照以下方式获取用户上传的文件名：

```php
$name = $request->file('avatar')->getClientOriginalName();
```

你也可以按照以下方式检索原始文件的扩展名：

```php
$ext = $request->file('avatar')->getClientOriginalExtension();
```

# 发送电子邮件

Laravel 的`Mail`类扩展了流行的 Swift Mailer 包，这使得发送电子邮件变得轻而易举。电子邮件模板的加载方式与视图相同，这意味着你可以使用 Blade 语法并将数据注入到模板中：

+   要将一些数据注入到位于 `resources/views/email/view.blade.php` 内的模板中，我们使用以下函数：

    ```php
    Mail::send('email.view', $data, function($message) {});
    ```

+   要发送 HTML 和纯文本版本，我们使用以下函数：

    ```php
    Mail::send(array('html.view', 'text.view'), $data, $callback);
    ```

+   要延迟邮件 5 分钟（这需要队列），我们使用以下函数：

    ```php
    Mail::later(5, 'email.view', $data, function($message) {});
    ```

在接收消息对象的 `$callback` 闭包内部，我们可以调用以下方法来修改要发送的消息：

+   `$message->subject('Welcome to the Jungle');`

+   `$message->from('email@example.com', 'Mr. Example');`

+   `$message->to('email@example.com', 'Mr. Example');`

一些不太常见的方法包括：

+   `$message->sender('email@example.com', 'Mr. Example');`

+   `$message->returnPath('email@example.com');`

+   `$message->cc('email@example.com', 'Mr. Example');`

+   `$message->bcc('email@example.com', 'Mr. Example');`

+   `$message->replyTo('email@example.com', 'Mr. Example');`

+   `$message->priority(2);`

要附加或嵌入文件，你可以使用以下方法：

+   `$message->attach('path/to/attachment.txt');`

+   `$message->embed('path/to/attachment.jpg');`

如果你已经在内存中有了数据，并且不想创建额外的文件，你可以使用`attachData`或`embedData`方法，如下所示：

+   `$message->attachData($data, 'attachment.txt');`

+   `$message->embedData($data, 'attachment.jpg');`

嵌入通常使用图像文件完成，你可以在消息体内部直接使用`embed`或`embedData`方法，如下面的代码片段所示：

```php
<p>Product Screenshot:</p>
<p>{!! $message->embed('screenshot.jpg') !!}</p>
```

# 使用 Carbon 简化日期和时间处理

Laravel 捆绑 Carbon ([`github.com/briannesbitt/Carbon`](https://github.com/briannesbitt/Carbon))，它通过添加更多富有表现力的方法扩展并增强了 PHP 的本地`DateTime`对象。Laravel 主要使用它来为 Eloquent 对象的日期和时间属性（`created_at`、`updated_at`和`deleted_at`）提供更多富有表现力的方法。然而，由于库已经存在，不利用它在应用程序的其他代码部分中就显得有些可惜了。

## 实例化 Carbon 对象

Carbon 对象旨在像正常的`DateTime`对象一样进行实例化。然而，它们确实支持一些更多富有表现力的方法：

+   Carbon 对象可以使用默认构造函数进行实例化，该构造函数将使用当前日期和时间，如下所示：

    +   `$now = new Carbon();`

+   它们可以使用给定时区的当前日期和时间进行实例化，如下所示：

    +   `$jetzt = new Carbon('Europe/Berlin');`

+   它们可以使用以下富有表现力的方法进行实例化：

    +   `$yesterday = Carbon::yesterday();`

    +   `$demain = Carbon::tomorrow('Europe/Paris');`

+   它们可以使用精确参数进行实例化，如下所示：

    +   `Carbon::createFromDate($year, $month, $day, $tz);`

    +   `Carbon::createFromTime($hour, $minute, $second, $tz);`

    +   `Carbon::create($year, $month, $day, $hour, $minute, $second, $tz);`

## 输出用户友好的时间戳

我们可以使用`diffForHumans()`方法生成人类可读的相对时间戳，例如*5 分钟前*、*上周*或*一年后*，如下所示：

```php
$post = App\Post::find(123);
echo $post->created_at->diffForHumans(); 
```

## 布尔方法

碳也提供了一些简单且富有表现力的方法，这些方法在你的控制器和视图中会很有用：

+   `$date->isWeekday();`

+   `$date->isWeekend();`

+   `$date->isYesterday();`

+   `$date->isToday();`

+   `$date->isTomorrow();`

+   `$date->isFuture();`

+   `$date->isPast();`

+   `$date->isLeapYear();`

## Carbon for Eloquent DateTime properties

要能够在数据库中存储为`DATE`或`DATETIME`类型的属性上调用 Carbon 的方法，你需要在模型中列出它们在`$dates`属性中：

```php
class Post extends Model {
  // ...
  protected $dates = [
    'published_at',
    'deleted_at',
  ];
}

```

你不需要包含`created_at`或`updated_at`，因为这些属性会自动被视为日期。

# 不要再等待队列了

队列允许你延迟函数的执行而不阻塞脚本。它们可以用来运行各种函数，从给大量用户发邮件到生成 PDF 报告。

Laravel 5 与以下队列驱动程序兼容：

+   使用`pda/pheanstalk`包的 Beanstalkd

+   使用`aws/aws-sdk-php`包的 Amazon SQS

+   IronMQ，使用`iron-io/iron_mq`包

每个队列系统都有其优点。Beanstalkd 可以安装在自己的服务器上；Amazon SQS 可能更经济实惠且维护成本更低，IronMQ 也是如此，它也是基于云的。后者还允许您设置*推送队列*，如果您无法在服务器上运行后台作业，这会非常好。

## 创建一个命令并将其推送到队列中

作业以命令的形式出现。命令可以是自我处理的，也可以不是。在后一种情况下，相应的处理器类将取命令类中的数据并对其采取行动。

命令类位于`app/Commands`目录中，命令处理器类可以在`app/Handlers/Commands`目录中找到。可以使用 Artisan 命令生成命令及其处理器的类，如下所示：

```php
$ php artisan make:command CommandName --handler --queued
```

`--handler`选项告诉 Artisan 创建一个处理器类（省略此选项将仅创建一个自我处理的命令类），而`--queued`选项指定应将其添加到队列中，而不是同步处理。

然后，您可以使用`Queue`外观将命令添加到队列中：

```php
Queue::push(new SendConfirmationEmail($order));
```

或者，您可以使用**命令总线**来调度命令。命令总线默认在控制器中使用`DispatchesCommands`特质设置。这意味着在您的控制器操作中，您可以使用`dispatch`方法：

```php
public function purchase(Product $product)
{
  // Create order
  $this->dispatch(new SendConfirmationEmail($order));
}
```

命令是包含执行动作所需数据的简单类——处理器随后在稍后的阶段使用命令提供的数据执行实际处理。一个例子是在下单后发送确认电子邮件。此命令看起来如下所示：

```php
<?php namespace App\Commands;

use App\Order;
use Illuminate\Contracts\Queue\ShouldBeQueued; 
use Illuminate\Queue\InteractsBeQueued;
use Illuminate\Queue\SerializesModels; 

class SendConfirmationEmail extends Command implements ShouldBeQueued {

  use InteractsWithQueue, SerializesModels;

  public $order;

  public function __construct(Order $order) {
    $this->order = $order;
  }
}
```

当队列执行处理器时，将执行实际发送电子邮件的操作，将订单传递给电子邮件模板以显示客户购买详情，如下所示：

```php
<?php namespace App\Handlers\Commands;

use App\Commands\SendConfirmationEmail;
use Illuminate\Contracts\Mail\Mailer;
use Illuminate\Queue\InteractsWithQueue;

class SendConfirmationEmailHandler {

  public function __construct(Mailer $mail) {
    $this->mail = $mail;
  }

  public function handle(SendConfirmationEmail $command) {
    $order = $command->order;
    $data = compact('order');
    $this->mail->send('emails.order.confirmation', $data, function($message) use ($order) {
      $message->subject('Your order confirmation');
      $message->to(
        $order->customer->email,
        $order->customer->name
      );
    });
  }
}
```

由于命令处理器是通过服务容器解析的，因此我们可以进行类型提示依赖项。在前面的例子中，我们需要邮件服务，因此我们类型提示合约以获取实现。然后我们可以使用邮件服务，通过从命令类接收到的订单数据向客户发送电子邮件。

### 注意

从 Laravel 5.1 开始，`app/Commands`目录将被重命名为`app/Jobs`，以表明它主要用于队列作业。

## 监听队列并执行作业

以下是用作监听队列和执行作业的函数：

+   我们可以如下监听默认队列：

    ```php
    $ php artisan queue:listen
    ```

+   我们可以指定要监听的连接，如下所示：

    ```php
    $ php artisan queue:listen connection
    ```

+   我们可以按优先级顺序指定多个连接，如下所示：

    ```php
    $ php artisan queue:listen important,not-so-important
    ```

`queue:listen`命令必须在后台运行，以便处理从队列中到达的作业。为了确保它永久运行，您必须使用进程控制系统，如**forever** ([`github.com/nodejitsu/forever`](https://github.com/nodejitsu/forever))或**supervisor** ([`supervisord.org/`](http://supervisord.org/))。

## 当作业失败时接收通知

要在作业失败时接收通知，我们使用以下函数和命令：

+   以下事件监听器用于查找失败的作业：

    ```php
    Queue::failing(function($job, $data) {
      // Send email notification
    });
    ```

+   任何失败的作业都可以存储在数据库表中，并使用以下命令查看：

    ```php
    $ php artisan queue:failed-table // Create the table 
    $ php artisan queue:failed // View the failed jobs
    ```

## 没有后台进程的队列

推送队列不需要后台进程，但它们只与`iron.io`驱动程序一起工作。推送队列在接收到作业时将调用应用程序中的端点，而不是由持续运行的工人进程处理的队列。如果您无法定义在应用程序服务器上运行的过程（例如在共享主机套餐中），这将非常有用。在`iron.io`上注册账户并将凭据添加到`app/config/queue.php`后，您可以通过定义一个接收所有传入作业的`POST`路由来使用它们。此路由调用`Queue::marshal()`，这是负责触发正确作业处理程序的方法：

```php
Route::post('queue/receive', function() {
  return Queue::marshal();
});
```

此路由需要使用`queue:subscribe`命令注册为订阅者：

```php
$ php artisan queue:subscribe queue_name http://yourapp.example.com/queue/receive
```

一旦 URL 在[`iron.io/`](http://iron.io/)上订阅，任何使用`Queue::push()`创建的新作业将通过`POST`请求从 Iron 发送回您的应用程序。

# 接下来去哪里？

以下是可以访问的资源站点列表，您可以访问它们以了解 Laravel 的最新更改：

+   在 Twitter 上关注[`twitter.com/laravelphp`](http://twitter.com/laravelphp)以获取定期更新

+   [Laravel 完整文档](http://laravel.com/docs)以获取更多信息

+   [Laravel API 浏览器](http://laravel.com/api)以查看可浏览的 API

+   [laracasts 教程网站](http://laracasts.com)
