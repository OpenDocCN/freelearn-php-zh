# 第九章：PHP 中的响应式扩展

在本章中，我们将讨论 PHP 中的响应式扩展，这是一个允许 PHP 程序员以响应式方式使用 PHP 的库，以及如何在事件中使用，也称为发布-订阅编程。我们还将讨论 PHP 中的函数式编程的概念以及如何以更简洁的方式进行编程。我们还将讨论以下主题：

+   映射

+   减少

+   延迟

+   以下是响应式扩展的使用案例：

+   日志数据分析（解析 Apache 日志）

+   排队系统（异步处理任务队列）

+   事件

响应式扩展是使用 PHP 以函数式方式编码的一种方式。它们是一组库（在 GitHub 上可用，网址为[`github.com/ReactiveX/RxPHP`](https://github.com/ReactiveX/RxPHP)），可以帮助您使用 PHP 中的可观察集合和 LINQ 风格的查询操作来组合基于事件的程序。

# 可观察对象的介绍

简而言之，您将进行事件驱动的编程，其中您将使用所谓的事件循环，并附加（连接）事件来执行您的命令。

安装只需要一个简单的 composer。

响应式 PHP 是如何工作的？在 PHP 中，除了运行代码`php -S localhost:8000`之外，没有其他方式来创建服务器。PHP 将当前目录视为公共目录的基础（在 Apache 中，通常是`/var/www`或使用**XAMPP**时为`C:/xampp/htdocs`）。顺便说一句，这仅在 PHP 5.4.0 之后才可用，也适用于 PHP 7.x。

没有可编程的方式来控制 PHP 命令行界面服务器的实际工作方式。

每次向该服务器发送请求时，PHP 服务器将负责处理它是否是有效请求，并自行处理事件。简而言之，每个请求都是一个新请求-没有涉及流或事件。

**RxPHP**通过在底层创建一个 PHP 流来创建事件循环，该流具有帮助使**响应式编程**成为可能的附加函数。该流基本上具有一个递归函数（一个不断调用自身并创建命令循环的函数）。事件循环基本上是一个编程构造，运行一个无限循环，简单地等待事件并能够对每个事件做出反应（换句话说，运行一些函数）。

# 事件循环和 ReactiveX 的介绍

熟悉事件循环的最佳方法是通过 JavaScript 世界中的一个流行库，即 jQuery。

如果您有使用 jQuery 的经验，您可以简单地创建（或链接）事件到一个简单的 DOM 选择器，然后编写代码来处理这些特定事件。例如，您可以通过将其附加到特定链接上创建一个`onClick`事件，然后编写当单击该链接时会发生什么的代码。

如果您熟悉 jQuery，控制具有 ID`someLink`的链接的代码将如下所示：

HTML：

```php
< a href="some url" id="someLink"> 

```

JavaScript：

```php
$("#someLink").on('click', function() { 
   //some code here 
}); 

```

在前面的代码片段中，每当 jQuery 找到一个 ID 为`someLink`的元素时，它将在每次单击事件上执行某些操作。

由于它在事件循环中，它将循环遍历事件循环的每个*迭代*并处理需要完成的工作。

然而，在响应式编程中有一点不同，它是函数式编程的一种形式。函数式编程是关于尽可能保持函数的纯净，不产生副作用。函数式编程的另一个方面是不可变性，但我们将在另一个部分讨论这一点。

在响应式编程中，我们基本上有**可观察对象**和**观察者**的概念。

可观察对象以数据形式发出事件，观察者订阅可观察对象以接收其事件。

使用响应式扩展进行编程的重点是能够以更*功能化*的方式进行编程。我们不再编写`while`、`for`循环，而是调用一个事件循环，它将跟踪观察者和它们的 Observable（订阅者）。以这种方式提供信息的好处是，现在可以制作基于事件或事件驱动的程序，其中您的代码将做出反应。

有了这个，你可以创建在后台永远运行的程序，只是响应式扩展。

让我们讨论一些响应式扩展的可用函数：

+   延迟

+   延迟

+   调度器

+   递归调度器

+   `map`和`flatMap`

+   减少

+   转换为数组

+   合并

+   做

+   扫描

+   压缩

## 延迟

在 RxPHP 中，`delay`函数的使用如下：

```php
<?php 
require_once __DIR__ . '/../bootstrap.php'; 

$loop = new \React\EventLoop\StreamSelectLoop(); 

$scheduler  = new \Rx\Scheduler\EventLoopScheduler($loop); 

\Rx\Observable::interval(1000, $scheduler) 
    ->doOnNext(function ($x) { 
        echo "Side effect: " . $x . "\n"; 
    }) 
    ->delay(500) 
    ->take(5) 
    ->subscribe($createStdoutObserver(), $scheduler); 

$loop->run(); 

```

在前面的代码中，我们创建了一个`EventLoopScheduler`，它将帮助我们按 1000 毫秒的间隔安排代码的执行。延迟函数被给予 500 毫秒来执行，take 函数将在最终订阅之前只花费 5 毫秒。

## 延迟

`defer`函数在执行之前等待*X*次迭代才会执行：

```php
<?php 

require_once __DIR__.'/../bootstrap.php'; 

$source = \Rx\Observable::defer(function () { 
    return \Rx\Observable::just(42); 
}); 

$subscription = $source->subscribe($stdoutObserver); 
?> 

```

在前面的代码中，我们创建了一个 Observable 对象，当调用`defer`函数时，它将返回 42。`defer`函数是一种承诺类型，返回一个 Observable，其中的代码将以异步方式执行。当 Observable 被订阅时，函数以一种方式*绑定*在一起，然后被*调用*或*触发*。

你可能会问，什么是 Observable？在 ReactiveX 中，观察者订阅 Observable。然后观察者对 Observable 发出的任何项或序列做出反应。

这意味着当您的应用程序收到一堆事件，但以异步方式处理它们时，不一定按照它们可能到达的顺序处理它们。

在前面的代码中，`stdoutObserver`是一个观察者，它将事件循环或 Observable 中的任何内容输出到`stdout`或控制台日志中。

## 调度器

调度器与三个主要组件一起工作：执行上下文，即执行给定任务的能力；执行策略是*如何*它将被排序；还有时钟或计时器或测量时间的基础系统，这是需要安排*何时*执行的。

调度器代码的使用方式如下：

```php
$loop    = \React\EventLoop\Factory::create(); 
$scheduler = new \Rx\Scheduler\EventLoopScheduler($loop); 

```

它基本上创建了一个`eventScheduler`，执行事件循环并对并发级别进行参数化。在前面的延迟中使用了 RxPHP 中的简单调度器。

## 递归调度器

这就是递归调度函数的使用方式：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

use Rx\Observable; 

class RecursiveReturnObservable extends Observable 
{ 
    private $value; 

    /** 
     * @param mixed $value Value to return. 
     */ 
    public function __construct($value) 
    { 
        $this->value = $value; 
    } 

    public function subscribe(\Rx\ObserverInterface $observer, $scheduler = null) 
    { 
        return $scheduler->scheduleRecursive(function ($reschedule) use ($observer) { 
            $observer->onNext($this->value); 
            $reschedule(); 
        }); 
    } 
} 

$loop      = React\EventLoop\Factory::create(); 
$scheduler = new Rx\Scheduler\EventLoopScheduler($loop); 

$observable = new RecursiveReturnObservable(42); 
$observable->subscribe($stdoutObserver, $scheduler); 

$observable = new RecursiveReturnObservable(21); 
$disposable = $observable->subscribe($stdoutObserver, $scheduler); 

$loop->addPeriodicTimer(0.01, function () { 
    $memory    = memory_get_usage() / 1024; 
    $formatted = number_format($memory, 3) . 'K'; 
    echo "Current memory usage: {$formatted}\n"; 
}); 

// after a second we'll dispose the 21 observable 
$loop->addTimer(1.0, function () use ($disposable) { 
    echo "Disposing 21 observable.\n"; 
    $disposable->dispose(); 
}); 

$loop->run(); 

```

前面的代码通过添加几个调度器定时器，然后递归或重复返回一个 Observable，然后订阅它。前面的代码将生成 21 个 Observables。

1 秒后发生了什么：

```php
//Next value: 21 
//Next value: 42 
//Next value: 21 
//Next value: 42 
//Next value: 21 

```

之后，它将处理 Observables 并最终打印出内存使用情况：

```php
//Disposing 21 observable. 
//Next value: 42 
//Next value: 42 
//Next value: 42 
//Next value: 42 
//Next value: 42 
//Current memory usage: 3,349.203K 

```

## 映射和扁平映射

`map`是一个简单的函数，它接受另一个函数并循环遍历一堆元素（一个数组），并对每个元素应用或调用传递给这些元素的函数。

另一方面，`flatMap`也订阅 Observable，这意味着您不再需要关心。

## 减少

`reduce`函数简单地将一个函数应用于传入的 Observables。简而言之，它接受一堆 Observables，并以顺序方式将函数应用于所有这些 Observables，将一个应用于下一个结果。

以下是如何使用`reduce`函数的示例：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

//Without a seed 
$source = \Rx\Observable::fromArray(range(1, 3)); 

$subscription = $source 
    ->reduce(function ($acc, $x) { 
        return $acc + $x; 
    }) 
    ->subscribe($createStdoutObserver()); 

```

## 转换为数组

`toArray`函数允许您操作 Observables 并从中创建数组。使用`toArray`的代码如下：

```php
<?php 

use Rx\Observer\CallbackObserver; 

require_once __DIR__ . '/../bootstrap.php'; 

$source = \Rx\Observable::fromArray([1, 2, 3, 4]); 

$observer = $createStdoutObserver(); 

$subscription = $source->toArray() 
    ->subscribe(new CallbackObserver( 
        function ($array) use ($observer) { 
            $observer->onNext(json_encode($array)); 
        }, 
        [$observer, "onError"], 
        [$observer, "onCompleted"] 
    )); 

```

在前面的代码中，我们首先基于数组`[1,2,3,4]`创建了一个 Observable。

这使我们能够使用数组的值并使用观察者订阅它们。在 ReactiveX 编程中，每个观察者只能与 Observables 一起工作。简而言之，`toArray`函数允许我们创建订阅源数组的观察者。

## 合并

`merge`函数只是一个操作符，它通过合并它们的发射将多个 Observables 合并为一个。

任何源 Observable 的`onError`通知都将立即传递给观察者。这将终止合并的 Observable：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

$loop      = React\EventLoop\Factory::create(); 
$scheduler = new Rx\Scheduler\EventLoopScheduler($loop); 

$observable       = Rx\Observable::just(42)->repeat(); 
$otherObservable  = Rx\Observable::just(21)->repeat(); 
$mergedObservable = $observable 
    ->merge($otherObservable) 
    ->take(10); 

$disposable = $mergedObservable->subscribe($stdoutObserver, $scheduler); 

$loop->run(); 

```

## do

`do`函数只是在各种 Observable 生命周期事件上注册一个操作。基本上，您将注册回调，ReactiveX 只会在 Observable 中发生某些事件时调用这些回调。这些回调将独立于正常的通知集合调用。RxPHP 设计了各种操作符来允许这样做：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

$source = \Rx\Observable::range(0, 3) 
    ->doOnEach(new \Rx\Observer\CallbackObserver( 
        function ($x) { 
            echo 'Do Next:', $x, PHP_EOL; 
        }, 
        function (Exception $err) { 
            echo 'Do Error:', $err->getMessage(), PHP_EOL; 
        }, 
        function () { 
            echo 'Do Completed', PHP_EOL; 
        } 
    )); 

$subscription = $source->subscribe($stdoutObserver); 

```

## scan

`scan`操作符对 Observable 发出的每个项目应用一个函数。它按顺序应用这个函数并发出每个连续的值：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

//With a seed 
$source = Rx\Observable::range(1, 3); 

$subscription = $source 
    ->scan(function ($acc, $x) { 
        return $acc * $x; 
    }, 1) 
    ->subscribe($createStdoutObserver()); 

```

这是一个没有种子的`scan`的例子：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

//Without a seed 
$source = Rx\Observable::range(1, 3); 

$subscription = $source 
    ->scan(function ($acc, $x) { 
        return $acc + $x; 
    }) 
    ->subscribe($createStdoutObserver()); 

```

## zip

`zip`方法返回一个 Observable，并对按顺序发出的项目的组合应用您选择的函数。这个函数的结果将成为返回的 Observable 发出的项目：

```php
<?php 

require_once __DIR__ . '/../bootstrap.php'; 

//With a result selector 
$range = \Rx\Observable::fromArray(range(0, 4)); 

$source = $range 
    ->zip([ 
        $range->skip(1), 
        $range->skip(2) 
    ], function ($s1, $s2, $s3) { 
        return $s1 . ':' . $s2 . ':' . $s3; 
    }); 

$observer = $createStdoutObserver(); 

$subscription = $source->subscribe($createStdoutObserver()); 

```

在以下示例代码中，我们使用`zip`而没有结果选择器：

```php
<?php 

use Rx\Observer\CallbackObserver; 

require_once __DIR__ . '/../bootstrap.php'; 

//Without a result selector 
$range = \Rx\Observable::fromArray(range(0, 4)); 

$source = $range 
    ->zip([ 
        $range->skip(1), 
        $range->skip(2) 
    ]); 

$observer = $createStdoutObserver(); 

$subscription = $source 
    ->subscribe(new CallbackObserver( 
        function ($array) use ($observer) { 
            $observer->onNext(json_encode($array)); 
        }, 
        [$observer, "onError"], 
        [$observer, "onCompleted"] 
    )); 

```

# 通过 Reactive 调度程序解析日志

仅仅拥有 Reactive 扩展和函数式编程技术的理论知识而不知道何时可以使用它是困难的。为了应用我们的知识，让我们看看以下的情景。

假设我们需要以异步方式读取 Apache 日志文件。

Apache 日志行看起来像这样：

```php
111.222.333.123 HOME - [01/Feb/1998:01:08:39 -0800] "GET /bannerad/ad.htm HTTP/1.0" 
200 198 "http://www.referrer.com/bannerad/ba_intro.htm""Mozilla/4.01 (Macintosh; I; PPC)" 

111.222.333.123 HOME - [01/Feb/1998:01:08:46 -0800] "GET /bannerad/ad.htm HTTP/1.0" 
200 28083 "http://www.referrer.com/bannerad/ba_intro.htm""Mozilla/4.01 (Macintosh; I; PPC)" 

```

让我们分解每行的部分。

首先，我们有 IP 地址。它在一些数字之间有三个点。其次，我们有记录服务器域的字段。

第三，我们有日期和时间。然后我们得到一个字符串，它说访问了什么，使用了什么 HTTP 协议。状态码是 200，后面是进程 ID，最后是请求者的名称，也称为引用者。

在读取 Apache 日志时，我们只想要 IP 地址、URL 和访问的日期和时间，还想知道使用了什么浏览器。

我们知道我们可以将数据分解成它们之间的空格，所以让我们将日志改为由以下方法分割的数组：

```php
<?php 
function readLogData($pathToLog) { 
$logs = []; 
$data = split('\n', read($pathToLog);) //log newlines 

foreach($data as line) { 
$logLine = split('',$line); 
  $ipAddr = $logLine[0]; 
  $time = $logLine[3]; 
$accessedUrl = $logLine[6]; 
  $referrer = $logLine[11]; 
  $logs[] = [ 
'IP' => $ipAddr, 
'Time' => $time, 
'URL' => $accessedUrl, 
'UserAgent' => $referrer 
  ]; 

} 
return $logs; 
} 

```

让我们添加一个 Observable，以便我们可以异步执行前面的函数，这意味着它将通过每小时读取日志文件来工作。

代码看起来像这样：

```php
$loop      = React\EventLoop\StreamSelectLoop; 
$scheduler = new Rx\Scheduler\EventLoopScheduler($loop); 

$intervalScheduler = \Rx\Observable::interval(3600000, $scheduler); 

//execute function to read logFile: 
$intervalScheduler::defer(function() { 
readLogData('/var/log/apache2/access.log'); 
})->subscribe($createStdoutObserver()); 

```

# ReactiveX 中的事件队列

事件队列只需确保以同步方式或先进先出方式完成的事情。让我们首先定义队列是什么。

队列基本上是一个要做的事情列表，它将一个接一个地执行，直到队列中的所有事情都完成了。

在 Laravel 中，例如，已经有了队列的概念，我们遍历队列的元素。您可以在[`laravel.com/docs/5.0/queues`](https://laravel.com/docs/5.0/queues)找到文档。

队列通常用于需要按顺序执行一些任务而不是在异步函数中执行的系统。在 PHP 中，已经有了`SplQueue`类，它使用双向链表实现了队列的主要功能。

一般来说，队列按照它们的顺序执行。在 ReactiveX 中，事情更多的是异步的。在这种情况下，我们将实现一个优先级队列，其中每个任务都有相应的优先级。

这是 ReactiveX 中一个简单的`PriorityQueue`代码：

```php
use \Rx\Scheduler\PriorityQueue; 

Var $firstItem = new ScheduledItem(null, null, null, 1, null); 

var $secondtItem = new ScheduledItem(null, null, null, 2, null); 
$queue          = new PriorityQueue(); 
$queue->enqueue($firstItem); 
$queue->enqueue($secondItem); 
//remove firstItem if not needed in queue 
$queue->remove($firstItem); 

```

在前面的代码中，我们使用了 RxPHP 的`PriorityQueue`库。我们设置了一些调度程序，并将它们排入了`PriorityQueue`中。我们为每个调度的项目分配了优先级或执行时间，分别为 1 和 2。在前面的场景中，第一个项目将首先执行，因为它具有最高的优先级并且执行时间最短（1）。最后，我们移除了`ScheduledItem`，只是为了展示在 RxPHP 库中`PriorityQueue`的可能性。

# 总结

您学会了如何使用响应式扩展库 RxPHP。响应式编程主要是使用 Observables 和 Observers，这类似于使用订阅者和发布者进行工作。

您学会了如何使用`delay`、`defer`、`map`和`flatMap`等操作符，以及如何使用调度程序。

您还学会了如何读取 Apache 日志文件并安排在每小时后进行读取，以及如何使用 RxPHP 的`PriorityQueue`类。
