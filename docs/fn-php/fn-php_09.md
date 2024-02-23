# 第九章：性能效率

现在我们已经涵盖了与函数式编程相关的各种技术，是时候分析它如何影响像 PHP 这样的语言的性能了，尽管每个版本都引入了越来越多的函数式特性，但 PHP 仍然在其核心是命令式的。

我们还将讨论为什么性能最终并不那么重要，以及我们如何利用记忆化和其他技术来在某些情况下缓解这个问题。

我们还将探讨两种由引用透明性启用的优化技术。第一种是记忆化，这是一种缓存类型。我们还将谈论如何在 PHP 中并行运行长时间的计算，以及您如何利用这一点。

在本章中，我们将涵盖以下主题：

+   函数式编程的性能影响

+   记忆化

+   计算的并行化

# 性能影响

由于没有核心支持柯里化和函数组合等功能，它们需要使用匿名包装函数来模拟。显然，这会带来性能成本。此外，正如我们在关于尾调用递归的部分已经讨论过的那样，在 第七章 *函数式技术和主题* 中，使用跳板也更慢。但与更传统的方法相比，你会损失多少执行时间呢？

让我们创建一些函数作为基准，并测试我们可以实现的各种速度。该函数将执行一个非常简单的任务，即将两个数字相加，以确保我们尽可能有效地测量开销：

```php
<?php 

use Functional as f; 

function add($a, $b) 
{ 
    return $a + $b; 
} 

function manualCurryAdd($a, $b = null) { 
    $func = function($b) use($a) { 
        return $a + $b; 
    }; 

    return func_num_args() > 1 ? $func($b) : $func; 
} 

$curryiedAdd = f\curry('add'); 

function add2($b) 
{ 
    return $b + 2; 
} 

function add4($b) 
{ 
    return $b + 4; 
} 

$composedAdd4 = f\compose('add2', 'add2'); 

$composerCurryedAdd = f\compose($curryiedAdd(2), $curryiedAdd(2)); 
```

我们创建了第一个函数 `add` 并对其进行了柯里化；这将是我们的第一个基准。然后我们将比较一个专门添加 `4` 到一个值的函数与两种不同的组合。第一个是两个专门函数的组合，第二个是两个 `add` 函数的柯里化版本的组合。

我们将使用以下代码来对我们的函数进行基准测试。它非常基础，但应该足以展示任何有意义的差异：

```php
<?php 

use Oefenweb\Statistics\Statistics; 

function benchmark($function, $params, $expected) 
{ 
    $iteration   = 10; 
    $computation = 2000000; 

    $times = array_map(function() use($computation, $function,  $params, $expected) { 
        $start = microtime(true); 

        array_reduce(range(0, $computation), function($expected)  use ($function, $params) { 
            if(($res = call_user_func_array($function, $params))  !== $expected) { 
                throw new RuntimeException("Faulty computation"); 
            } 

            return $expected; 
        }, $expected); 

        return microtime(true) - $start; 
    }, range(0, $iteration)); 

    echo sprintf("mean: %02.3f seconds\n",  Statistics::mean($times)); 
    echo sprintf("std:  %02.3f seconds\n",  Statistics::standardDeviation($times)); } 
```

统计方法来自于可通过 composer 获取的 `oefenweb/statistics` 包。我们还检查返回的值是否符合预期，作为额外的预防措施。我们将连续运行每个函数 200 万次，每次运行 10 次，并显示 200 万次运行的平均时间。

让我们先运行柯里化的基准测试。显示的结果是针对 PHP 7.0.12。当尝试在 PHP 5.6 中运行时，所有基准测试都会变慢，但它们在各种函数之间表现出相同的差异：

```php
<?php 

benchmark('add', [21, 33], 54); 
// mean: 0.447 seconds 
// std:  0.015 seconds 

benchmark('manualCurryAdd', [21, 33], 54); 
// mean: 1.210 seconds 
// std:  0.016 seconds 

benchmark($curryiedAdd, [21, 33], 54); 
// mean: 1.476 seconds 
// std:  0.007 seconds 
```

显然，结果将根据测试运行的系统而有所不同，但相对差异应该保持大致相同。

首先，如果我们看标准偏差，我们可以看到 10 次运行大多数情况下花费了相同的时间，这表明我们可以相信我们的数字是性能的良好指标。

我们可以看到柯里化版本明显更慢。手动柯里化效率稍微更高，但两个柯里化版本大部分情况下比简单函数版本慢三倍。在得出结论之前，让我们看看组合函数的结果：

```php
<?php 

benchmark('add4', [10], 14); 
// mean: 0.434 seconds 
// std:  0.001 seconds 

benchmark($composedAdd4, [10], 14); 
// mean: 1.362 seconds 
// std:  0.005 seconds 

benchmark($composerCurryedAdd, [10], 14); 
// mean: 3.555 seconds 
// std:  0.018 seconds 
```

标准偏差足够小，以至于我们可以认为这些数字是有效的。

关于值本身，我们可以看到组合也大约慢了三倍，而柯里化函数的组合，毫不奇怪，慢了九倍。

现在，如果我们将最坏情况的 3.55 秒与最佳情况的 0.434 秒进行比较，这意味着在使用组合和柯里化时我们有 3 秒的开销。这重要吗？看起来失去了很多时间吗？让我们试着在一个 web 应用程序的背景下想象这些数字。

## 开销重要吗？

我们对我们的方法进行了两百万次执行，用了三秒钟。我最近参与的一个项目是一个奢侈品牌的电子商务应用程序，在 26 个国家和 10 多种语言中可用，完全是从零开始编写的，没有使用任何框架，渲染一个页面需要大约 25,000 次函数调用。

即使我们承认所有这些调用都是事先柯里化的组合函数，这意味着在最坏的情况下，开销现在大约为 40 毫秒。该应用程序大约需要 180 毫秒来显示一个页面，因此我们在性能上会有 20-25%的降低。

这仍然很多，但远不及我们之前看到的三倍慢的数字。与每个函数调用相关的函数式技术的开销将随着每个函数调用的增加而线性增长。在基准测试中看起来很好，因为执行的计算是微不足道的。在现实应用中，你会有外部瓶颈，比如数据库、第三方 API 或文件系统。你还有执行复杂计算的函数，需要比简单的加法更多的时间。在这种情况下，引入的开销将成为应用程序总执行时间的一个较小部分。

这也是一个最坏的情况，我们假设一切都是组合和柯里化的。在现实世界的应用程序中，你可能会使用传统的框架，其中包含没有开销的函数和方法。你还可以识别代码中的热点路径，并手动优化它们，使用显式的柯里化和组合而不是辅助函数。也不需要对所有东西都进行柯里化；你将有只有一个参数的函数不需要它，还有一些函数使用柯里化是没有意义的。

这些数字是考虑到缓存不热的应用程序。你已经采取的任何减少页面渲染时间的机制将继续发挥作用。例如，如果你有一个 Varnish 实例在运行，你的页面可能会以相同的速度提供。

## 不要忘记

我们将一个非常小的函数与组合和柯里化进行了比较。现代 PHP 代码库将使用类来保存业务逻辑和值。让我们使用以下`add`函数的实现来模拟这一点：

```php
<?php 

class Integer { 
    private $value; 
    public function __construct($v) { $this->value = $v; } 
    public function get() { return $this->value; } 
} 

class Adder { 
    public function add(Integer $a, Integer $b) { 
        return $a->get() + $b->get(); 
    } 
} 
```

传统方法所需的时间会增加：

```php
<?php 

benchmark([new Adder, 'add'], [new Integer(21), new Integer(33)], 54); 
// mean: 0.767 seconds 
// std:  0.019 seconds 
```

只需将所有东西封装在一个类中并使用 getter，执行时间几乎翻了一番，这意味着在基准测试中，函数式方法只比传统方法慢 1.5 倍，而我们示例应用程序中的开销现在已经是 10-15%，这已经好多了。

## 我们能做些什么吗？

遗憾的是，我们实际上没有什么可以做的。我们可以通过更有效地实现`curry`和`compose`方法来节省一点时间，就像我们使用手动柯里化版本的`add`方法一样，但这不会带来太大的影响。

然而，将这两种技术作为 PHP 的核心部分实现，将带来很多好处，可能使它们与传统函数和方法持平，或者非常接近。但据我所知，目前没有计划这样做。

可能还可以创建一个 C 语言扩展程序，以更有效的方式实现这两个函数在 PHP 中的应用。然而，这将是不切实际的，因为大多数 PHP 托管公司不允许用户安装自定义扩展程序。

## 结束语

正如我们刚才看到的，使用柯里化和函数组合等技术对性能有一定影响，而这种影响很难自行减轻。在我看来，收益大于成本，但重要的是要有意识地转向函数式编程。

现在大多数 Web 应用程序都在 PHP 应用程序前面有某种缓存机制。因此，唯一的成本将是在填充此缓存时。如果你处于这种情况，我认为没有理由避免使用我们学到的技术。

# 记忆化

记忆化是一种优化技术，它会存储昂贵函数的结果，以便在任何后续具有相同参数的调用中直接返回。这是数据缓存的一个特例。

尽管它可以用于非纯函数，并具有与任何其他缓存机制相同的失效问题，但它主要用于所有函数都是纯函数的函数式语言，因此极大地简化了它的使用。

这个想法是用存储空间来换取计算时间。第一次使用给定的输入调用函数时，结果会被存储，下一次使用相同参数调用相同函数时，已经计算出的结果可以立即返回。在 PHP 中可以很容易地使用`static`关键字来实现这一点：

```php
<?php 

function long_computation($n) 
{ 
    static $cache = []; 
    $key = md5(serialize($n)); 

    if(! isset($cache[$key])) { 
        // your computation comes here, the rest is boilerplate 
        sleep(2); 
        $cache[$key] = $n; 
    } 

    return $cache[$key]; 
} 
```

显然有很多种不同的方法来做类似的事情，但这种方法足够简单，可以让你了解它是如何工作的。人们也可以想象实现一种过期机制，或者，由于我们使用内存空间而不是计算时间，一种数据结构，在这种数据结构中，值在不使用时被擦除以为新的结果腾出空间。

另一个选择是将信息存储到磁盘上，以便在同一脚本的多次运行之间保留值，例如。PHP 中至少存在一个库（[`github.com/koktut/php-memoize`](https://github.com/koktut/php-memoize)）就是这样做的。

然而，该库不能很好地处理递归调用，因为函数本身没有被修改，因此值只会保存第一次调用，而不是递归调用。库自述文件中链接的文章（[`eddmann.com/posts/implementing-and-using-memoization-in-php/`](http://eddmann.com/posts/implementing-and-using-memoization-in-php/)）更详细地讨论了这个问题并提出了解决方案。

有趣的是**Hack**有一个属性，可以自动记忆具有特定类型参数的函数的结果（[`docs.hhvm.com/hack/attributes/special#__memoize`](https://docs.hhvm.com/hack/attributes/special#__memoize)）。如果您正在使用 Hack 并希望使用注释，我建议您首先阅读*Gotchas*部分，因为它可能并不总是按照您的意愿进行操作。

### 注意

Hack 是一种在 PHP 基础上添加新功能并在 Facebook 编写的 PHP 虚拟机上运行的语言-**HipHop 虚拟机**（**HHVM**）。任何 PHP 代码都与 Hack 兼容，但 Hack 添加了一些新的语法，使代码与原始的 PHP 解释器不兼容。有关更多信息，您可以访问[`hacklang.org`](http://hacklang.org)/。

## Haskell、Scala 和记忆化

Haskell 和 Scala 都不会自动执行记忆化。这两者都没有核心功能来执行记忆化，尽管你可以找到多个提供这一功能的库。

有一种误解认为 Haskell 默认情况下会对所有函数进行记忆化，这是因为这种语言是惰性的。实际上，Haskell 试图尽可能延迟函数调用的计算，并且一旦这样做了，它就使用了引用透明属性来用计算出的值替换其他类似的调用。

然而，有多种情况下，这种替换不能自动发生，除了重新计算值之外别无选择。如果您对这个话题感兴趣，这个*Stack Overflow*问题是一个很好的起点，其中包含所有正确的关键字[`stackoverflow.com/questions/3951012/when-is-memoization-automatic-in-ghc-haskell`](http://stackoverflow.com/questions/3951012/when-is-memoization-automatic-in-ghc-haskell)。

我们将在这里结束讨论，因为这本书是关于 PHP 的。

## 结束语

这只是一个对记忆化的快速介绍，因为这种技术相当简单实现，实际上并没有太多可以说的。我只是想介绍一下，让你了解这个术语。

如果你有一些长时间运行的计算，会多次使用相同的参数，我建议你使用这种技术，因为它可以真正加快速度，并且不需要调用者做任何事情。使用起来非常透明。

但要注意，这并不是万能的。根据返回值的数据结构，它可能会很快地消耗内存。如果遇到这个问题，你可以使用一些机制来清理缓存中的旧值或者少用的值。

# 计算的并行化

拥有纯函数的另一个好处是，你可以将计算分成多个小部分，分发工作负载，并组装结果。对于任何映射、过滤和折叠操作都可以这样做。我们将看到，用于折叠的函数需要是单子的。用于映射和过滤的函数除了纯度之外没有特定的约束。

映射除了纯函数之外没有特定的约束。假设你有四个核心或计算机，你只需要按照以下步骤进行：

1.  将数组分成四部分。

1.  将部分任务发送到每个核心进行映射。

1.  合并结果。

在这种特殊情况下，它可能比在单个核心上执行要慢，因为合并操作会增加额外的开销。然而，一旦计算时间变长，你就能够利用更多的计算能力，从而节省时间。

过滤操作与映射完全相同，只是你发送的是一个谓词而不是一个函数。

只有当你拥有一个单子操作时，折叠操作才能发生，因为每个拆分都需要从空值开始，否则可能会使结果产生偏差：

1.  将数组分成四部分。

1.  将部分任务发送到每个核心进行折叠，初始值为空值。

1.  将所有结果放入一个新数组中。

1.  在新数组上执行相同的折叠操作。

如果你的集合非常大，你可以将最终的折叠再次分成多个部分。

## PHP 中的并行任务

PHP 是在计算机只有一个核心时创建的，自那时起，使用它的传统方式是为每个请求提供一个单独的线程。你可以在 web 服务器中声明多个工作进程来处理不同的请求，但一个请求通常只会使用一个线程，因此只会使用一个核心。

尽管 PHP 的线程安全版本存在，但由于前述原因，Linux 发行版通常会提供非线程安全版本。这并不意味着在 PHP 中无法并行执行任务，但这确实会使任务变得更加困难。

### pthreads 扩展

PHP 7 发布了一个新版本的**pthreads**扩展，它允许你使用新设计的面向对象 API 并行运行多个任务。这真的很棒，甚至还有一个*polyfill*，如果扩展不可用，可以按顺序执行任务。

### 注意

术语*polyfill*起源于 JavaScript 开发。它是一小段代码，用于替换用户浏览器中未实现的功能。有时也会使用另一个术语*shim*。在我们的情况下，*pthreads-polyfill*提供了一个与扩展 API 在所有点上相似的 API，但是它是按顺序运行任务的。

遗憾的是，使用这个扩展有点挑战。首先，你需要一个线程安全的 PHP 二进制文件，也称为**ZTS**二进制文件，即**Zend Thread-safe**。正如我们刚才看到的，发行版通常不提供这个版本。据我所知，目前没有官方 PHP 软件包支持 ZTS。当你尝试为你的 Linux 发行版创建自己的 ZTS 二进制文件时，通常可以在谷歌上找到相关的指导。

Windows 和 Mac OS 用户则更加方便，因为你可以在[`www.php.net`](http://www.php.net)上下载 ZTS 二进制文件，并且在使用`homebrew`软件包管理器安装 PHP 时可以启用该选项。

另一个限制是该扩展将拒绝在 CGI 环境中加载。这意味着您只能在命令行上使用它。如果您对 pthreads 扩展的维护者为什么选择设置这个限制感兴趣，我建议您阅读他写的这篇博客文章，网址为[`blog.krakjoe.ninja/2015/09/the-worth-of-advice.html`](http://blog.krakjoe.ninja/2015/09/the-worth-of-advice.html)。

现在，如果我们假设您能够拥有 PHP 的 ZTS 版本，并且只编写 CLI 应用程序，让我们看看如何使用**pthreads**扩展执行并行折叠。该扩展程序托管在 GitHub 上，网址为[`github.com/krakjoe/pthreads`](https://github.com/krakjoe/pthreads)，安装说明可以在官方 PHP 文档中找到，网址为[`docs.php.net/manual/en/book.pthreads.php`](http://docs.php.net/manual/en/book.pthreads.php)。

显然，我们可以以多种方式实现使用线程进行折叠。我们将尝试采用一种通用的方法。在某些情况下，更专门的版本可能更快，但这应该已经涵盖了整个一系列用例：

```php
<?php 

class Folder extends Thread { 
    private $collection; 
    private $callable; 
    private $initial; 

    private $results; 

    private function __construct($callable, $collection, $initial) 
    { 
        $this->callable = $callable; 
        $this->collection = $collection; 
        $this->initial = $initial; 
    } 

    public function run() 
    { 
        $this->results = array_reduce($this->collection, $this- >callable, $this->initial); 
    } 

    public static function fold($callable, array $collection,  $initial, $threads=4) 
    { 
        $chunks = array_chunk($collection, ceil(count($collection) / $threads)); 

        $threads = array_map(function($i) use ($chunks, $callable,  $initial) { 
            $t = new static($callable, $chunks[$i], $initial); 
            $t->start(); 
            return $t; 
        }, range(0, $threads - 1)); 

        $results = array_map(function(Thread $t) { 
            $t->join(); 
            return $t->results; 
        }, $threads); 

        return array_reduce($results, $callable, $initial); 
    } 
} 
```

实现非常简单；我们有一个简单的`Thread`执行每个块的减少，然后使用简单的`array_reduce`函数将它们组合在一起。我们本可以选择使用`Pool`实例来管理各个线程，但在这种简单情况下，这将使实现变得复杂。

另一种可能性是递归，直到生成的数组包含最多`$threads`个元素为止；这样，我们将利用我们手头的全部计算能力直到结束。但同样，这将使实现变得复杂。

您如何使用它？只需调用静态方法：

```php
<?php 

$add = function($a, $b) { 
    return $a + $b; 
}; 

$collection = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]; 

echo Folder::fold($add, $collection, 0); 
// 55 
```

如果您想尝试一下这个想法，一个小型库以并行方式实现了所有三个高阶函数（[`github.com/functional-php/parallel`](https://github.com/functional-php/parallel)）。您可以使用 composer 安装它：

```php
**composer require functional-php/parallel**

```

### 消息队列

PHP 中另一个并行化任务的选项是使用消息队列。消息队列提供了一种异步通信协议。您将拥有一个服务器，它将保存消息，直到一个或多个客户端检索它们。

我们可以通过让我们的应用程序向服务器发送 X 条消息来实现并行计算，每个分布式任务发送一条消息。然后，一定数量的工作线程将检索消息并执行计算，将结果作为新消息发送回应用程序。

有很多不同的消息队列实现可以使用。通常，队列本身并不是用 PHP 实现的，但它们大多数都有客户端实现可以使用。我们将使用**RabbitMQ**和**php-amqplib**客户端。

解释如何安装服务器超出了本书的范围，但您可以在互联网上找到很多教程。我们也不会解释所有关于实现的细节，只解释与我们主题相关的内容。您可以使用 composer 安装 PHP 库：

```php
**composer require php-amqplib/php-amqplib**

```

我们需要为我们的工作线程和应用程序都实现。让我们首先创建一个包含公共部分的文件，我们将其命名为`09-rabbitmq.php`：

```php
<?php 

require_once './vendor/autoload.php'; 
use PhpAmqpLib\Connection\AMQPStreamConnection; 

$connection = new AMQPStreamConnection('localhost', 5672, 'guest',  'guest'); 
$channel = $connection->channel(); 
list($queue, ,) = $channel->queue_declare($queue_name, false,  false, false, false); 

$fold_function = function($a, $b) { 
    return $a + $b; 
}; 
```

现在我们创建工作线程：

```php
<?php 
use PhpAmqpLib\Message\AMQPMessage; 

$queue_name = 'fold_queue'; 
require_once('09-rabbitmq.php'); 

function callback($r) { 
    global $fold_function; 

    $data = unserialize($r->body); 

    $result = array_reduce($data['collection'], $fold_function,  $data['initial']); 

    $msg = new AMQPMessage(serialize($result)); 

    $r->delivery_info['channel']->basic_publish($msg, '', $r- >get('reply_to')); 
    $r->delivery_info['channel']->basic_ack($r- >delivery_info['delivery_tag']); 
}; 

$channel->basic_qos(null, 1, null); 
$channel->basic_consume('fold_queue', '', false, false, false,  false, 'callback'); 

while(count($channel->callbacks)) { 
    $channel->wait(); 
} 

$channel->close(); 
$connection->close(); 
```

现在我们创建应用程序本身：

```php
<?php 
use PhpAmqpLib\Message\AMQPMessage; 

$queue_name = ''; 
require_once('09-rabbitmq.php'); 

function send($channel, $queue, $chunk, $initial) 
{ 
    $data = [ 
        'collection' => $chunk, 
        'initial' => $initial 
    ]; 
    $msg = new AMQPMessage(serialize($data), array('reply_to' =>  $queue)); 
    $channel->basic_publish($msg, '', 'fold_queue'); 
} 

class Results { 
    private $results = []; 
    private $channel; 

    public function register($channel, $queue) 
    { 
        $this->channel = $channel; 
        $channel->basic_consume($queue, '', false, false, false,  false, [$this, 'process']); 
    } 

    public function process($rep) 
    { 
        $this->results[] = unserialize($rep->body); 
    } 

    public function get($expected) 
    { 
        while(count($this->results) < $expected) { 
            $this->channel->wait(); 
        } 

        return $this->results; 
    } 
} 

$results = new Results(); 
$results->register($channel, $queue); 

$initial = 0; 

send($channel, $queue, [1, 2, 3], 0); 
send($channel, $queue, [4, 5, 6], 0); 
send($channel, $queue, [7, 8, 9], 0); 
send($channel, $queue, [10], 0); 

echo array_reduce($results->get(4), $fold_function, $initial); 
// 55 
```

显然，这是一个非常天真的实现。自 PHP 5 以来，要求这样的文件是不好的做法，而且代码非常脆弱，但它达到了演示消息队列提供的可能性的目的。

当您启动工作线程时，它会注册自己作为`fold_queue`队列的消费者。当接收到消息时，它会使用在公共部分声明的折叠函数对数据进行折叠，并将结果发送回作为回复的队列。循环确保我们等待传入的消息；根据代码，工作线程不应该自行退出。

应用程序有一个`send`函数，它在`fold_queue`队列上发送消息。`Results`类实例注册自己作为默认队列的消费者，以接收每个工作进程的结果。然后发送四条消息，并要求`Results`实例等待它们。最后，我们减少接收的数据以获得最终结果。

如果只启动一个工作进程，结果将按顺序发送；但是，如果启动多个工作进程，每个工作进程将从 RabbitMQ 服务器检索消息并处理它，从而实现并行化。

与使用线程相比，消息队列有多个好处：

+   工作进程可以在多台计算机上

+   工作进程可以在任何其他具有所选队列客户端的语言中实现

+   队列服务器提供冗余和故障转移机制

+   队列服务器可以在工作进程之间进行负载平衡

在可用时使用 pthreads 库可能会更容易一些，如果你计划只在唯一计算机的核心之间分配工作负载，但如果你想要更灵活性，消息队列是更好的选择。

### 其他选择

在 PHP 中启动并行计算的其他方法，但通常会使检索值比我们刚才看到的更加困难。

一种选择是使用`curl_multi_exec`函数异步执行多个 HTTP 请求。一般结构类似于我们在消息队列示例中使用的内容。但是，与完整消息系统的全部功能相比，可能也有限制。

你也可以使用多个相关函数之一创建其他 PHP 进程。在这种情况下，难点通常是在不丢失数据的情况下传递和检索数据，因为这样做的方式将取决于与环境相关的许多因素。如果你想这样做，`popen`、`exec`或`passthru`函数可能是你最好的选择。

如果你不想做所有的苦力活，你也可以使用`Parallel.php`库，它可以将大部分复杂性抽象化。你可以使用 composer 安装它：

```php
**composer require kzykhys/parallel**

```

文档可在 GitHub 上找到[`github.com/kzykhys/Parallel.php`](https://github.com/kzykhys/Parallel.php)。由于该库使用 Unix 套接字，大部分与数据丢失相关的问题都已经解决。但是你无法在 Windows 上使用它。

## 结束语

正如我们所看到的，使用多个线程或进程在 PHP 中可能并不是最容易的事情，特别是在网页的上下文中。然而，这是可以实现的，并且可以大大加快长时间计算的速度。

随着 PHP 7 中 pthreads 的重写，我们可以希望更多的 Linux 发行版和托管公司将开始提供 ZTS 版本。

如果是这种情况，并且并行计算开始在 PHP 中变得真实，可能可以进行一些轻量级的大数据处理，而无需求助于其他语言的外部库，比如**Hadoop 框架**。

我想用几句话来结束关于消息队列的话题。即使你不以功能方式使用它们来处理数据并获取结果，它们也是在网页请求的上下文中执行长时间操作的好方法。例如，如果你让用户上传一堆图片并且你需要处理它们，你可以将操作加入队列并立即返回给用户。排队的消息将在适当的时间被处理，你的用户不必等待。

# 总结

在本章中，我们发现当进行函数式编程时，很遗憾是需要付出代价的。由于 PHP 没有对柯里化和函数组合等功能的核心支持，因此在使用它们时会有与包装函数相关的开销。在某些情况下，这显然可能是一个问题，但缓存通常可以减轻这种成本。

我们谈到了记忆化，这是一种缓存技术，结合纯函数，可以加速对给定函数的后续调用，而无需使存储的结果失效。

最后，我们讨论了通过利用在集合上执行的任何纯操作可以在多个节点之间分发，而不必担心共享状态来并行化计算在 PHP 中的计算。

下一章将专门针对使用框架的开发人员，我们将发现如何在 PHP 世界中目前最常用的框架的背景下利用我们迄今为止学到的技术。
