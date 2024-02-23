# *第四章*：进行直接的 C 语言调用

本章介绍了**外部函数接口**（FFI）。在本章中，您将了解 FFI 的全部内容，它的作用以及如何使用它。本章的信息对于对使用直接 C 语言调用进行快速自定义原型设计感兴趣的开发人员非常重要。

在本章中，您不仅了解了将 FFI 引入 PHP 语言背后的背景，还学会了如何直接将 C 语言结构和函数合并到您的代码中。尽管——正如您将了解的那样——这并不是为了实现更快的速度，但它确实使您能够直接将任何 C 语言库合并到您的 PHP 应用程序中。这种能力为 PHP 打开了一个以前无法实现的功能世界。

本章涵盖的主题包括以下内容：

+   理解 FFI

+   学会何时使用 FFI

+   检查 FFI 类

+   在应用程序中使用 FFI

+   使用 PHP 回调函数

# 技术要求

为了检查和运行本章提供的代码示例，以下是最低推荐的硬件要求：

+   基于 X86_64 的台式 PC 或笔记本电脑

+   1 千兆字节（GB）的可用磁盘空间

+   4 GB 的随机存取内存（RAM）

+   500 千位每秒（Kbps）或更快的互联网连接

此外，您需要安装以下软件：

+   Docker

+   Docker Compose

请参考*第一章*的*技术要求*部分，了解有关 Docker 和 Docker Compose 安装的更多信息，以及如何构建用于演示本书中代码的 Docker 容器。在本书中，我们将恢复示例代码的目录称为`/repo`。

本章的源代码位于此处：

https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices

现在我们可以开始讨论 FFI 的理解。

# 理解 FFI

FFI 的主要目的是允许任何给定的编程语言能够将来自其他语言编写的外部库的代码和函数调用合并到其中。早期的一个例子是 20 世纪 80 年代微型计算机能够使用`PEEK`和`POKE`命令将汇编语言合并到否则笨拙的**通用符号指令代码**（BASIC）编程语言脚本中。与许多其他语言不同，PHP 在 PHP 7.4 之前没有这种能力，尽管自 2004 年以来一直在讨论中。

为了全面了解 PHP 8 中的 FFI，有必要偏离一下，看看为什么 FFI 在 PHP 语言中被完全采用花了这么长时间。还有必要快速了解一下 PHP 扩展，以及与 C 语言代码的工作能力。我们首先研究 PHP 和 C 语言之间的关系。

## PHP 和 C 语言之间的关系

**C 语言**是由丹尼斯·里奇在 1972 年末在贝尔实验室开发的。自那时起，尽管引入了其面向对象的表亲 C++，这种语言仍然主导着编程语言领域。PHP 本身是用 C 编写的；因此，直接加载 C 共享库并直接访问 C 函数和数据结构的能力对于 PHP 语言来说是一个非常重要的补充。

将 FFI 扩展引入 PHP 语言使 PHP 能够加载并直接使用 C 结构和 C 函数。为了能够明智地决定何时何地使用 FFI 扩展，让我们先看一下一般的 PHP 扩展。

## 理解 PHP 扩展

**PHP 扩展**，顾名思义，*扩展*了 PHP 语言。每个扩展都可以添加**面向对象编程**（**OOP**）类以及过程级函数。每个扩展都有一个独特的逻辑目的，例如，`GD`扩展处理图形图像处理，而`PDO`扩展处理数据库访问。

类比一下，考虑一家医院。在医院里，您有急诊、外科、儿科、骨科、心脏科、X 光等科室。每个科室都是独立的，有着不同的目的。这些科室共同构成了医院。同样地，PHP 就像医院，它的扩展就像各种科室。

并非所有扩展都是相同的。一些扩展，称为**核心扩展**，在安装 PHP 时始终可用。其他扩展必须手动下载、编译和启用。现在让我们来看看核心扩展。

### 访问 PHP 核心扩展

PHP 核心扩展直接包含在主 PHP 源代码存储库中，位于此处：https://github.com/php/php-src/tree/master/ext。如果您转到此网页，您将看到一个子目录列表，如下面的屏幕截图所示。每个子目录包含特定扩展的 C 语言代码：

![图 4.1-在 GitHub 上看到的 PHP 核心扩展](img/Figure_4.1_B16992.jpg)

图 4.1-在 GitHub 上看到的 PHP 核心扩展

因此，当 PHP 安装在服务器上时，所有核心扩展都会被编译和安装。现在我们来简要看看不属于核心的扩展。

### 检查非核心 PHP 扩展

不属于核心的 PHP 扩展通常由特定供应商（**Microsoft**就是一个例子）维护。非核心扩展通常被认为是可选的，并且使用不广泛。

一旦非核心扩展开始被越来越频繁地使用，它很可能最终会被迁移到核心中。这方面的例子很多。最近的一个是`JSON`扩展：它现在不仅是核心的一部分，而且在 PHP 8 中这个扩展不能再被禁用。

核心扩展也可能被移除。其中一个例子是`mcrypt`扩展。这在 PHP 7.1 中被弃用，因为该扩展依赖的基础库已经*被遗弃*了 9 年以上。在 PHP 7.2 中，它正式从核心中移除。现在我们考虑在哪里找到非核心扩展。

### 查找非核心扩展

在这一点上，您可能会问一个合乎逻辑的问题：*您从哪里获取非核心扩展？*一般来说，非核心扩展可以直接从供应商、[github.com](http://github.com)或此网站：http://pecl.php.net/获取。多年来一直有人抱怨[pecl.php.net](http://pecl.php.net)包含过时和未维护的代码。尽管这在一定程度上是真的，但同样也存在最新的、积极维护的代码在这个网站上。

例如，如果您查看 MongoDB 的 PHP 扩展，您会发现最新版本是在 2020 年 11 月底发布的。以下屏幕截图显示了此扩展的**PHP 扩展社区库**（**PECL**）网站页面：

![图 4.2-用于 PHP MongoDB 扩展的 pecl.php.net 页面](img/Figure_4.2_B16992.jpg)

图 4.2-用于 PHP MongoDB 扩展的 pecl.php.net 页面

在许多情况下，供应商更倾向于保留对扩展的完全控制。这意味着您需要去他们的网站获取 PHP 扩展。一个例子是 Microsoft SQL Server 的 PHP 扩展，可以在此**统一资源定位符**（**URL**）找到：https://docs.microsoft.com/en-us/sql/connect/php/download-drivers-php-sql-server?view=sql-server-ver15.

本小节的关键要点是，PHP 语言通过其扩展进行增强。这些扩展是用 C 语言编写的。因此，在 PHP 脚本中直接建模原型扩展的逻辑能力非常重要。现在让我们把注意力转向应该在哪里使用 FFI。

# 学习何时使用 FFI

直接将 C 库导入到 PHP 中的潜力真是令人震惊。PHP 核心开发人员中的一位实际上使用 FFI 扩展将 PHP 绑定到 C 语言**TensorFlow**机器学习平台！

提示

有关 TensorFlow 机器学习平台的信息，请访问此网页：https://www.tensorflow.org/。要了解 PHP 如何与此库绑定，请查看这里：[`github.com/dstogov/php-tensorflow`](https://github.com/dstogov/php-tensorflow)。

正如我们在本节中所展示的，FFI 扩展并不是解决所有需求的神奇解决方案。本节讨论了 FFI 扩展的主要优势和劣势，并为您提供了使用指南。我们在本节中揭穿的一个神话是，使用 FFI 扩展直接调用 C 语言来加速 PHP 8 程序执行。首先，让我们看看将 FFI 扩展纳入 PHP 中花费了这么长时间。

## 将 FFI 引入 PHP

实际上，第一个 FFI 扩展是由 PHP 核心开发人员**Wez Furlong**和**Ilia Alshanetsky**于 2004 年 1 月在 PECL 网站（https://pecl.php.net/）上为 PHP 5 引入的。然而，该项目从未通过 Alpha 阶段，并在一个月内停止了开发。

随着 PHP 在接下来的 14 年中的发展和成熟，人们开始意识到 PHP 将受益于在 PHP 脚本中快速原型化潜在扩展的能力。如果没有这种能力，PHP 有可能落后于其他语言，比如 Python 和 Ruby。

过去，由于缺乏快速原型能力，扩展开发人员被迫在能够在 PHP 脚本中测试之前编译完整的扩展并使用`pecl`安装它。在某些情况下，开发人员甚至不得不*重新编译 PHP 本身*来测试他们的新扩展！相比之下，FFI 扩展允许开发人员*直接在*PHP 脚本中放置 C 函数调用以进行即时测试。

从 PHP 7.4 开始，并持续到 PHP 8，核心开发人员 Dmitry Stogov 提出了改进版本的 FFI 扩展。在令人信服的概念验证之后（请参阅有关 PHP 绑定到 TensorFlow 机器学习平台的前面的*提示*框），这个 FFI 扩展版本被纳入了 PHP 语言中。

提示

原始的 FFI PHP 扩展可以在这里找到：http://pecl.php.net/package/ffi。有关修订后的 FFI 提案的更多信息，请参阅以下文章：https://wiki.php.net/rfc/ffi。

现在让我们来看看为什么不应该使用 FFI 来提高速度。

## 不要使用 FFI 来提高速度

因为 FFI 扩展允许 PHP 直接访问 C 语言库，人们很容易相信你的 PHP 应用程序会突然以机器语言速度运行得非常快。不幸的是，事实并非如此。FFI 扩展需要首先打开给定的 C 库，然后在执行之前解析和伪编译`FFI`实例。然后 FFI 扩展充当 C 库代码和 PHP 脚本之间的桥梁。

对一些读者来说，相对缓慢的 FFI 扩展性能不仅限于 PHP 8。其他语言在使用自己的 FFI 实现时也会遇到相同的限制效果。这里有一个基于*Ary 3 基准*的优秀性能比较，可以在这里找到：https://wiki.php.net/rfc/ffi#php_ffi_performance。

如果您查看刚刚引用的网页上显示的表格，您会发现 Python FFI 实现在 0.343 秒内完成了基准测试，而仅使用本机 Python 代码运行相同的基准测试只需 0.212 秒。

查看相同的表，PHP 7.4 FFI 扩展在 0.093 秒内运行了基准测试（比 Python 快 30 倍！），而仅使用本机 PHP 代码运行的相同基准测试在 0.040 秒内执行。

下一个逻辑问题是：*为什么你应该使用 FFI 扩展？* 这将在下一节中介绍。

## 为什么要使用 FFI 扩展？

对上一个问题的答案很简单：这个扩展主要是为了快速**PHP 扩展原型**。PHP 扩展是语言的命脉。没有扩展，PHP 只是*另一种编程语言*。

当高级开发人员首次着手进行编程项目时，他们需要确定项目的最佳语言。一个关键因素是可用的扩展数量以及这些扩展的活跃程度。通常，活跃维护的扩展数量与使用该语言的项目的长期成功潜力之间存在直接关系。

因此，如果有一种方法可以加快扩展开发的速度，那么 PHP 语言本身的长期可行性就得到了改善。FFI 扩展为 PHP 语言带来的价值在于，它能够在不必经历整个编译-链接-加载-测试周期的情况下，直接在 PHP 脚本中测试扩展原型。

FFI 扩展的另一个用例，除了快速原型设计之外，是允许 PHP 直接访问模糊或专有的 C 代码的一种方式。一个例子是编写用于控制工厂机器的自定义 C 代码。为了让 PHP 运行工厂，可以使用 FFI 扩展将 PHP 直接绑定到控制各种机器的 C 库。

最后，这个扩展的另一个用例是用它来*预加载* C 库，可能会减少内存消耗。在我们展示使用示例之前，让我们来看看`FFI`类及其方法。

# 检查 FFI 类

正如您在本章中所学到的，不是每个开发人员都需要使用 FFI 扩展。直接使用 FFI 扩展可以加深您对 PHP 语言内部的理解，这种加深的理解对您作为 PHP 开发人员的职业生涯可能会产生积极影响：很可能在将来的某个时候，您将被一家开发了自定义 PHP 扩展的公司雇佣。在这种情况下，了解如何操作 FFI 扩展可以让您为自定义 PHP 扩展开发新功能，同时帮助您解决扩展问题。

`FFI`类包括 20 个方法，分为四个广泛的类别，如下所述：

+   **创建性**：此类别中的方法创建了 FFI 扩展**应用程序编程接口**（**API**）中可用的类的实例。

+   **比较**：比较方法旨在比较 C 数据值。

+   **信息性**：这组方法为您提供有关 C 数据值的元数据，包括大小和*对齐*。

+   **基础设施**：基础设施方法用于执行后勤操作，如复制、填充和释放内存。

提示

完整的 FFI 类文档在这里：[`www.php.net/manual/en/class.ffi.php`](https://www.php.net/manual/en/class.ffi.php)。

有趣的是，所有`FFI`类方法都可以以静态方式调用。现在是时候深入了解与 FFI 相关的类的细节和用法了，首先是*创建性*方法。

## 使用 FFI 创建方法

属于*创建*类别的 FFI 方法旨在直接产生`FFI`实例或 FFI 扩展提供的类的实例。在使用 FFI 扩展提供的 C 函数时，重要的是要认识到不能直接将本地的 PHP 变量传递给函数并期望它能工作。数据必须首先被创建为`FFI`数据类型或导入到`FFI`数据类型中，然后才能将`FFI`数据类型传递给 C 函数。要创建`FFI`数据类型，请使用下面总结的函数之一，如下所示：

![表 4.1 – FFI 类创建方法总结](img/Table_4.1_B16992.jpg)

表 4.1 – FFI 类创建方法总结

`cdef()`和`scope()`方法都会产生一个直接的`FFI`实例，而其他方法会产生可以用来创建`FFI`实例的对象实例。`string()`用于从本地 C 变量中提取给定数量的字节。让我们来看看如何创建和使用`FFI\CType`实例。

### 创建和使用 FFI\CType 实例

非常重要的一点是，一旦创建了`FFI\CType`实例，*不要*简单地将一个值赋给它，就像它是一个本地的 PHP 变量一样。这样做只会由于 PHP 是弱类型语言而简单地覆盖`FFI\CType`实例。相反，要将标量值赋给`FFI\CType`实例，使用它的`cdata`属性。

下面的例子创建了一个`$arr` C 数组。然后用值填充本地 C 数组，直到达到最大大小，之后我们使用一个简单的`var_dump()`来查看它的内容。我们将按照以下步骤进行：

1.  首先，我们使用`FFI::arrayType()`来创建数组。作为参数，我们提供了一个`FFI::type()`方法和维度。然后我们使用`FFI::new()`来创建`FFI\Ctype`实例。代码如下所示：

```php
// /repo/ch04/php8_ffi_array.php
$type = FFI::arrayType(FFI::type("char"), [3, 3]);
$arr  = FFI::new($type);
```

1.  或者，我们也可以将操作合并成一个单一的语句，如下所示：

`$arr = FFI::new(FFI::type("char[3][3]"));`

1.  然后我们初始化了三个提供测试数据的变量，如下面的代码片段所示。请注意，本地的 PHP`count()`函数适用于`FFI\CData`数组类型：

```php
$pos   = 0;
$val   = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
$y_max = count($arr);
```

1.  现在我们可以用值填充它，就像用 PHP 数组一样，只是我们需要使用`cdata`属性来保留元素作为`FFI\CType`实例。代码如下所示：

```php
for ($y = 0; $y < $y_max; $y++) {
    $x_max = count($arr[$y]);
    for ($x = 0; $x < $x_max; $x++) {
        $arr[$y][$x]->cdata = $val[$pos++];
    }
}
var_dump($arr)
```

在前面的例子中，我们使用嵌套的`for()`循环来填充二维的 3 x 3 数组，用字母表的字母。如果我们现在执行一个简单的`var_dump()`，我们会得到以下结果：

```php
root@php8_tips_php8 [ /repo/ch04 ]# php 
php8_ffi_array.php 
object(FFI\CData:char[3][3])#2 (3) {
  [0]=> object(FFI\CData:char[3])#3 (3) {
    [0]=> string(1) "A"
    [1]=> string(1) "B"
    [2]=> string(1) "C"
  }
  [1]=> object(FFI\CData:char[3])#1 (3) {
    [0]=> string(1) "D"
    [1]=> string(1) "E"
    [2]=> string(1) "F"
  }
  [2]=> object(FFI\CData:char[3])#4 (3) {
    [0]=> string(1) "G"
    [1]=> string(1) "H"
    [2]=> string(1) "I"
}
```

从输出中要注意的第一件重要的事情是，索引都是整数。从输出中得到的第二个要点是，这显然不是一个本地的 PHP 数组。`var_dump()`告诉我们，每个数组元素都是一个`FFI\CData`实例。还要注意的是，C 语言字符串被视为数组。

因为数组的类型是`char`，我们可以使用`FFI::string()`来显示其中一行。下面是一个产生*ABC*响应的命令：

`echo FFI::string($arr[0], 3);`

任何尝试将`FFI\CData`实例提供给一个以数组作为参数的 PHP 函数都注定失败，即使它被定义为数组类型。在下面的代码片段中，注意如果我们将这个命令添加到前面的代码块中的输出：

`echo implode(',', $arr);`

从下面的输出中可以看到，因为数据类型不是`array`，`implode()`会发出致命错误。以下是结果输出：

```php
PHP Fatal error:  Uncaught TypeError: implode(): Argument #2 ($array) must be of type ?array, FFI\CData given in /repo/ch04/php8_ffi_array.php:25
```

现在你知道如何创建和使用`FFI\CType`实例了。现在让我们转向创建`FFI`实例。

### 创建和使用 FFI 实例

如章节介绍中所述，FFI 扩展有助于快速原型设计。因此，使用 FFI 扩展，你可以逐个开发设计用于新扩展的 C 函数，并立即在 PHP 应用程序中进行测试。

重要提示

FFI 扩展不会编译 C 代码。为了在 FFI 扩展中使用 C 函数，您必须首先使用 C 编译器将 C 代码编译成共享库。您将在本章的最后一节“在应用程序中使用 FFI”中学习如何做到这一点。

为了在 PHP 和本地 C 库函数调用之间建立桥梁，您需要创建一个`FFI`实例。FFI 扩展需要您提供一个定义了 C 函数签名和您计划使用的 C 库的 C 定义。`FFI::cdef()`和`FFI::scope()`都可以直接创建`FFI`实例。

以下示例使用`FFI::cdef()`绑定了两个本地 C 库函数。具体操作如下：

1.  第一个本地方法`srand()`用于初始化随机化序列。另一个本地 C 函数`rand()`调用序列中的下一个数字。`$key`变量保存了随机化的最终产品。`$size`表示要调用的随机数的数量。代码如下所示：

```php
// /repo/ch04/php8_ffi_cdef.php
$key  = '';
$size = 4;
```

1.  然后，我们通过调用`cdef()`并在字符串`$code`中标识本地 C 函数来创建`FFI`实例，该字符串取自`libc.so.6`本地 C 库，如下所示：

```php
$code = <<<EOT
    void srand (unsigned int seed);
    int rand (void);
EOT;
$ffi = FFI::cdef($code, 'libc.so.6');
```

1.  然后我们通过调用`srand()`来初始化随机化。然后，在循环中，我们调用`rand()`本地 C 库函数来生成一个随机数。我们使用`sprintf()`本地 PHP 函数将生成的整数转换为十六进制，然后将其附加到`$key`，并将其输出。代码如下所示：

```php
$ffi->srand(random_int(0, 999));
for ($x = 0; $x < $size; $x++)
    $key .= sprintf('%x', $ffi->rand());
echo $key
```

以下是前面代码片段的输出。请注意，生成的值可以用作随机密钥：

```php
root@php8_tips_php8 [ /repo/ch04 ]# php php8_ffi_cdef.php
23f306d51227432e7d8d921763b7eedf
```

在输出中，您会看到一串连接的随机整数转换为十六进制的字符串。请注意，每次调用脚本时，结果值都会发生变化。

提示

对于真正的随机化，最好只使用`random_int()`本地 PHP 函数。`openssl`扩展中还有出色的密钥生成函数。这里展示的示例主要是为了让您熟悉 FFI 扩展的用法。

重要提示

FFI 扩展还包括两种额外的创建方法：`FFI::load()`和`FFI::scope()`。`FFI::load()`用于在**预加载**过程中直接从 C 头文件（`*.h`）加载 C 函数定义。`FFI::scope()`使预加载的 C 函数可通过 FFI 扩展使用。有关预加载的更多信息，请查看 FFI 文档中的完整预加载示例：[`www.php.net/manual/en/ffi.examples-complete.php`](https://www.php.net/manual/en/ffi.examples-complete.php)。

现在让我们来看看用于比较本地 C 数据类型的 FFI 扩展函数。

## 使用 FFI 比较数据

请记住，使用 FFI 扩展创建 C 语言数据结构时，它存在于 PHP 应用程序之外。正如您在前面的示例中看到的（请参阅*创建和使用 FFI\CType 实例*部分），PHP 可以在一定程度上与 C 数据交互。但是，为了比较目的，最好使用`FFI::memcmp()`，因为本地 PHP 函数可能返回不一致的结果。

FFI 扩展中提供的两个比较函数在*表 4.2*中总结如下：

![表 4.2 – FFI 类比较方法总结](img/Table_4.2_B16992.jpg)

表 4.2 – FFI 类比较方法总结

`FFI::isNull()`可用于确定`FFI\CData`实例是否为`NULL`。更有趣的是`FFI::memcmp()`。虽然这个函数的操作方式与**太空船操作符**（`<=>`）相同，但它接受一个*第三个参数*，表示您希望在比较中包含多少字节。以下示例说明了这种用法：

1.  首先定义一组代表`FFI\CData`实例的四个变量，这些实例可以包含多达六个字符，并使用示例数据填充这些实例，如下所示：

```php
// /repo/ch04/php8_ffi_memcmp.php
$a = FFI::new("char[6]");
$b = FFI::new("char[6]");
$c = FFI::new("char[6]");
$d = FFI::new("char[6]");
```

1.  请记住，C 语言将字符数据视为数组，因此即使使用`cdata`属性，我们也不能直接分配字符串。因此，我们需要定义一个匿名函数，用字母填充实例。我们使用以下代码来实现这一点：

```php
$populate = function ($cdata, $start, $offset, $num) {
    for ($x = 0; $x < $num; $x++)
        $cdata[$x + $offset] = chr($x + $offset + 
                                   $start);
    return $cdata;
};
```

1.  接下来，我们使用该函数将四个`FFI\CData`实例填充为不同的字母集，如下所示：

```php
$a = $populate($a, 65, 0, 6);
$b = $populate($b, 65, 0, 3);
$b = $populate($b, 85, 3, 3);
$c = $populate($c, 71, 0, 6);
$d = $populate($d, 71, 0, 6);
```

1.  现在我们可以使用`FFI::string()`方法来显示到目前为止的内容，如下所示：

```php
$patt = "%2s : %6s\n";
printf($patt, '$a', FFI::string($a, 6));
printf($patt, '$b', FFI::string($b, 6));
printf($patt, '$c', FFI::string($c, 6));
printf($patt, '$d', FFI::string($d, 6));
```

1.  这是`printf()`语句的输出：

```php
$a : ABCDEF
$b : ABCXYZ
$c : GHIJKL
$d : GHIJKL
```

1.  从输出中可以看出，`$c`和`$d`的值是相同的。`$a`和`$b`的前三个字符相同，但最后三个字符不同。

1.  此时，如果我们尝试使用太空船操作符（`<=>`）进行比较，结果将如下：

```php
PHP Fatal error:  Uncaught FFI\Exception: Comparison of incompatible C types
```

1.  同样，尝试使用`strcmp()`，即使数据是字符类型，结果如下：

```php
PHP Warning:  strcmp() expects parameter 1 to be string, object given
```

1.  因此，我们唯一的选择是使用`FFI::memcmp()`。在这组比较中，注意第三个参数是`6`，表示 PHP 应该比较最多六个字符：

```php
$p = "%20s : %2d\n";
printf($p, 'memcmp($a, $b, 6)', FFI::memcmp($a, 
        $b, 6));
printf($p, 'memcmp($c, $a, 6)', FFI::memcmp($c, 
        $a, 6));
printf($p, 'memcmp($c, $d, 6)', FFI::memcmp($c, 
        $d, 6));
```

1.  如预期的那样，输出与在原生 PHP 字符串上使用太空船操作符的输出相同，如下所示：

```php
   memcmp($a, $b, 6) : -1
   memcmp($c, $a, 6) :  1
   memcmp($c, $d, 6) :  0
```

1.  请注意，如果将比较限制为仅三个字符，会发生什么。这是添加到代码块中的另一个`FFI::memcmp()`比较，将第三个参数设置为`3`：

```php
echo "\nUsing FFI::memcmp() but not full length\n";
printf($p, 'memcmp($a, $b, 3)', FFI::memcmp($a, 
        $b, 3));
```

1.  从这里显示的输出中可以看出，通过将`memcmp()`限制为仅三个字符，`$a`和`$b`被视为相等，因为它们都以相同的三个字符`a`、`b`和`c`开头：

```php
Using FFI::memcmp() but not full length
   memcmp($a, $b, 3) :  0
```

从这个例子中最重要的是，您需要在要比较的字符数和要比较的数据性质之间找到平衡。比较的字符数越少，整体操作速度越快。然而，如果数据的性质可能导致错误的结果，您必须增加字符数，并在性能上稍微损失。

现在让我们来看看如何从 FFI 扩展数据中收集信息。

### 从 FFI 扩展数据中提取信息

当您使用`FFI`实例和原生 C 数据结构时，原生 PHP 信息方法（如`strlen()`和`ctype_digit()`）无法提供有用的信息。因此，FFI 扩展包括三种方法，旨在生成有关 FFI 扩展数据的信息。这三种方法在*表 4.3*中总结如下：

![表 4.3 - FFI 类信息方法总结](img/Table_4.3_B16992.jpg)

表 4.3 - FFI 类信息方法总结

首先我们看看`FFI::typeof()`，然后再深入了解其他两种方法。

### 使用 FFI::typeof()确定 FFI 数据的性质

这是一个示例，说明了如何使用`FFI::typeof()`。该示例还演示了处理 FFI 数据时，原生 PHP 信息函数无法产生有用的结果。我们这样做：

1.  首先，我们定义一个`$char` C 字符串，并用字母表的前六个字母填充它，如下所示：

```php
// /repo/ch04/php8_ffi_typeof.php
$char = FFI::new("char[6]");
for ($x = 0; $x < 6; $x++)
    $char[$x] = chr(65 + $x);
```

1.  然后我们尝试使用`strlen()`来获取字符串的长度。在下面的代码片段中，请注意使用`$t::class`：这相当于`get_class($t)`。此用法仅适用于 PHP 8 及以上版本：

```php
try {
    echo 'Length of $char is ' . strlen($char);
} catch (Throwable $t) {
    echo $t::class . ':' . $t->getMessage();
}
```

1.  在 PHP 7.4 中的结果是一个`Warning`消息。然而，在 PHP 8 中，如果将除字符串以外的任何内容传递给`strlen()`，将抛出致命的`Error`消息。这是此时的 PHP 8 输出：

```php
TypeError:strlen(): Argument #1 ($str) must be of type string, FFI\CData given
```

1.  类似地，尝试使用`ctype_alnum()`，如下所示：

```php
echo '$char is ' .
    ((ctype_alnum($char)) ? 'alpha' : 'non-alpha');
```

1.  以下是在*步骤 4*中显示的`echo`命令的输出：

```php
$char is non-alpha
```

1.  显然，我们无法使用原生 PHP 函数获取有关 FFI 数据的有用信息！然而，使用`FFI::typeof()`，如下所示，会返回更好的结果：

```php
$type = FFI::typeOf($char);
var_dump($type);
```

1.  这是`var_dump()`的输出：

```php
object(FFI\CType:char[6])#3 (0) {}
```

从最终输出中可以看出，我们现在有了有用的信息！现在让我们来看看另外两种 FFI 信息方法。

### 利用 FFI::alignof()和 FFI::sizeof()

在进入展示这两种方法的实际示例之前，重要的是要理解**对齐**的确切含义。为了理解对齐，您需要对大多数计算机中内存的组织方式有基本的了解。

RAM 仍然是在程序运行周期内临时存储信息的最快方式。您计算机的**中央处理单元**（**CPU**）在程序执行时将信息从内存中移入和移出。内存以并行数组的形式组织。`alignof()`返回的对齐值将是可以一次从对齐内存数组的并行切片中获取多少字节。在旧计算机中，值为 4 是典型的。对于大多数现代微型计算机，常见的值为 8 或 16（或更大）。

现在让我们来看一个示例，说明了这两种 FFI 扩展信息方法的使用以及这些信息如何产生性能改进。我们将按照以下步骤进行：

1.  首先，我们创建一个`FFI`实例`$ffi`，在其中定义了两个标记为`Good`和`Bad`的 C 结构。请注意，在下面的代码片段中，这两个结构具有相同的属性；然而，这些属性的排列顺序不同。

```php
$struct = 'struct Bad { char c; double d; int i; }; '
        . 'struct Good { double d; int i; char c; }; 
          ';
$ffi = FFI::cdef($struct);
```

1.  然后我们从`$ffi`中提取这两个结构，如下所示：

```php
$bad = $ffi->new("struct Bad");
$good = $ffi->new("struct Good");
var_dump($bad, $good);
```

1.  `var_dump()`输出如下所示：

```php
object(FFI\CData:struct Bad)#2 (3) {
  ["c"]=> string(1) ""
  ["d"]=> float(0)
  ["i"]=> int(0)
}
object(FFI\CData:struct Good)#3 (3) {
  ["d"]=> float(0)
  ["i"]=> int(0)
  ["c"]=> string(1) ""
}
```

1.  然后我们使用这两个信息方法来报告这两个数据结构，如下所示：

```php
echo "\nBad Alignment:\t" . FFI::alignof($bad);
echo "\nBad Size:\t" . FFI::sizeof($bad);
echo "\nGood Alignment:\t" . FFI::alignof($good);
echo "\nGood Size:\t" . FFI::sizeof($good);
```

这个代码示例的最后四行输出如下所示：

```php
Bad Alignment:  8
Bad Size:       24
Good Alignment: 8
Good Size:      16
```

从输出中可以看出，`FFI::alignof()`的返回告诉我们对齐块的宽度为 8 字节。然而，您还可以看到，`Bad`结构占用的字节数比`Good`结构所需的空间大 50%。由于这两个数据结构具有完全相同的属性，任何理智的开发人员都会选择`Good`结构。

从这个例子中，您可以看到 FFI 扩展信息方法能够让我们了解如何最有效地构造我们的 C 数据。

提示

关于 C 语言中`sizeof()`和`alignof()`的区别的出色讨论，请参阅这篇文章：https://stackoverflow.com/questions/11386946/whats-the-difference-between-sizeof-and-alignof。

现在您已经了解了 FFI 扩展信息方法是什么，并且已经看到了它们的使用示例。现在让我们来看看与基础设施相关的 FFI 扩展方法。

## 使用 FFI 基础设施方法

FFI 扩展基础类别方法可以被视为支持 C 函数绑定所需的*幕后*组件。正如我们在本章中一直强调的那样，如果您希望直接从 PHP 应用程序中访问 C 数据结构，则需要 FFI 扩展。因此，如果您需要执行类似于 PHP `unset()`语句以释放内存，或者 PHP `include()`语句以包含外部程序代码，FFI 扩展基础方法提供了本地 C 数据和 PHP 之间的桥梁。

*表 4.4*，如下所示，总结了这个类别中的方法：

![表 4.4 – FFI 类基础方法](img/Table_4.4_B16992.jpg)

表 4.4 – FFI 类基础方法

首先，让我们先看看`FFI::addr()`、`free()`、`memset()`和`memcpy()`。

### 使用 FFI::addr()、free()、memset()和 memcpy()

PHP 开发人员经常通过**引用**给变量赋值。这允许一个变量的更改自动反映在另一个变量中。当传递参数给需要返回多个值的函数或方法时，引用的使用尤其有用。通过引用传递允许函数或方法返回无限数量的值。

`FFI::addr()`方法创建一个指向现有`FFI\CData`实例的 C 指针。就像 PHP 引用一样，对指针关联的数据所做的任何更改也将被更改。

在使用`FFI::addr()`方法构建示例的过程中，我们还向您介绍了`FFI::memset()`。这个函数很像`str_repeat()`PHP 函数，因为它（`FFI::memset()`）用特定值填充指定数量的字节。在这个例子中，我们使用`FFI::memset()`来用字母表的字母填充 C 字符字符串。

在本小节中，我们还将介绍`FFI::memcpy()`。这个函数用于将数据从一个`FFI\CData`实例复制到另一个实例。与`FFI::addr()`方法不同，`FFI::memcpy()`创建一个克隆，与复制数据源没有任何连接。此外，我们介绍了`FFI::free()`，这是一个用于释放使用`FFI::addr()`创建的指针的方法。

让我们看看这些 FFI 扩展方法如何使用，如下所示：

1.  首先，创建一个`FFI\CData`实例`$arr`，由六个字符的 C 字符串组成。请注意，在下面的代码片段中，使用了`FFI::memset()`，另一个基础设施方法，用**美国信息交换标准代码**（**ASCII**）码 65：字母`A`填充字符串：

```php
// /repo/ch04/php8_ffi_addr_free_memset_memcpy.php
$size = 6;
$arr  = FFI::new(FFI::type("char[$size]"));
FFI::memset($arr, 65, $size);
echo FFI::string($arr, $size);
```

1.  使用`FFI::string()`方法的`echo`结果如下所示：

```php
AAAAAA
```

1.  从输出中可以看到，出现了六个 ASCII 码 65（字母`A`）的实例。然后我们创建另一个`FFI\CData`实例`$arr2`，并使用`FFI::memcpy()`将一个实例中的六个字符复制到另一个实例中，如下所示：

```php
$arr2  = FFI::new(FFI::type("char[$size]"));
FFI::memcpy($arr2, $arr, $size);
echo FFI::string($arr2, $size);
```

1.  毫不奇怪，输出与*步骤 2*中的输出完全相同，如下所示：

```php
AAAAAA
```

1.  接下来，我们创建一个指向`$arr`的 C 指针。请注意，当指针被赋值时，它们会出现在本机 PHP `var_dump()`函数中作为数组元素。然后我们可以改变数组元素`0`的值，并使用`FFI::memset()`将其填充为字母`B`。代码如下所示：

```php
$ref = FFI::addr($arr);
FFI::memset($ref[0], 66, 6);
echo FFI::string($arr, $size);
var_dump($ref, $arr, $arr2);
```

1.  以下是*步骤 5*中剩余代码的输出：

```php
BBBBBB
object(FFI\CData:char(*)[6])#2 (1) {
  [0]=>   object(FFI\CData:char[6])#4 (6) {
    [0]=>  string(1) "B"
    [1]=>  string(1) "B"
    [2]=>  string(1) "B"
    [3]=>  string(1) "B"
    [4]=>  string(1) "B"
    [5]=>  string(1) "B"
  }
}
object(FFI\CData:char[6])#3 (6) {
  [0]=>  string(1) "B"
  [1]=>  string(1) "B"
  [2]=>  string(1) "B"
  [3]=>  string(1) "B"
  [4]=>  string(1) "B"
  [5]=>  string(1) "B"
}
object(FFI\CData:char[6])#4 (6) {
  [0]=>  string(1) "A"
  [1]=>  string(1) "A"
  [2]=>  string(1) "A"
  [3]=>  string(1) "A"
  [4]=>  string(1) "A"
  [5]=>  string(1) "A"
}
```

从输出中可以看到，我们首先有一个`BBBBBB`字符串。您可以看到指针的形式是一个 PHP 数组。原始的`FFI\CData`实例`$arr`现在已经改变为字母`B`。然而，前面的输出也清楚地显示了复制的`$arr2`不受对`$arr`或其`$ref[0]`指针所做的更改的影响。

1.  最后，为了释放使用`FFI::addr()`创建的指针，我们使用`FFI::free()`。这个方法很像本机 PHP 的`unset()`函数，但是设计用于处理 C 指针。这是我们添加到示例的最后一行代码：

`FFI::free($ref);`

现在您已经了解了如何使用 C 指针以及如何使用信息填充 C 数据，让我们看看如何使用`FFI\CData`实例进行类型转换。

### 学习关于 FFI::cast()

在 PHP 中，**类型转换**的过程经常发生。当 PHP 被要求执行涉及不同数据类型的操作时，就会使用它。下面是一个经典的例子：

```php
$a = 123;
$b = "456";
echo $a + $b;
```

在这个微不足道的例子中，`$a`被分配了`int`（整数）的数据类型，`$b`被分配了`string`的类型。`echo`语句要求 PHP 首先将`$b`强制转换为`int`，执行加法，然后将结果强制转换为`string`。

本机 PHP 还允许开发人员通过在变量或表达式前面的括号中添加所需的数据类型来强制数据类型。从前面代码片段的重写示例可能如下所示：

```php
$a = 123;
$b = "456";
echo (string) ($a + (int) $b);
```

强制类型转换使您的意图对其他使用您代码的开发人员非常清晰。它还保证了结果，因为强制类型转换对代码流的控制更大，并且不依赖于 PHP 的默认行为。

FFI 扩展具有类似的功能，即`FFI::cast()`方法。正如您在本章中看到的，FFI 扩展数据与 PHP 隔离，并且不受 PHP 类型转换的影响。为了强制数据类型，您可以使用`FFI::cast()`返回所需的并行`FFI\CData`类型。让我们看看如何在以下步骤中做到这一点：

1.  在这个例子中，我们创建了一个`int`类型的`FFI\CData`实例`$int1`。我们使用它的`cdata`属性来赋值`123`，如下所示：

```php
// /repo/ch04/php8_ffi_cast.php
// not all lines are shown
$patt = "%2d : %16s\n";
$int1 = FFI::new("int");
$int1->cdata = 123;
$bool = FFI::cast(FFI::type("bool"), $int1);
printf($patt, __LINE__, (string) $int1->cdata);
printf($patt, __LINE__, (string) $bool->cdata);
```

1.  正如您从这里显示的输出中看到的，将`123`的整数值强制转换为`bool`（布尔值），在输出中显示为`1`：

```php
 8 :                  123
 9 :                    1
```

1.  接下来，我们创建了一个`int`类型的`FFI\CData`实例`$int2`，并赋值`123`。然后我们将其强制转换为`float`，再转回`int`，如下面的代码片段所示：

```php
$int2 = FFI::new("int");
$int2->cdata = 123;
$float1 = FFI::cast(FFI::type("float"), $int2);
$int3   = FFI::cast(FFI::type("int"), $float1);
printf($patt, __LINE__, (string) $int2->cdata);
printf($patt, __LINE__, (string) $float1->cdata);
printf($patt, __LINE__, (string) $int3->cdata);
```

1.  最后三行的输出非常令人满意。我们看到我们的原始值`123`被表示为`1.7235971111195E-43`。当强制转换回`int`时，我们的原始值被恢复。以下是最后三行的输出：

```php
15 :                 123
16 : 1.7235971111195E-43
17 :                 123
```

1.  FFI 扩展与 C 语言一般一样，不允许所有类型进行转换。例如，在上一段代码中，我们尝试将类型为`float`的`FFI\CData`实例`$float2`强制转换为`char`类型，如下所示：

```php
try {
    $float2 = FFI::new("float");
    $float2->cdata = 22/7;
    $char1   = FFI::cast(FFI::type("char[20]"), 
        $float2);
    printf($patt, __LINE__, (string) $float2->cdata);
    printf($patt, __LINE__, (string) $char1->cdata);
} catch (Throwable $t) {
    echo get_class($t) . ':' . $t->getMessage();
}
```

1.  结果是灾难性的！正如您从这里显示的输出中看到的，抛出了一个`FFI\Exception`：

```php
FFI\Exception:attempt to cast to larger type
```

在本节中，我们介绍了一系列 FFI 扩展方法，这些方法创建了 FFI 扩展对象实例，比较值，收集信息，并处理所创建的 C 数据基础设施。您了解到有一些 FFI 扩展方法在本机 PHP 语言中具有相同的功能。在下一节中，我们将回顾一个实际的例子，将一个 C 函数库整合到 PHP 脚本中，使用 FFI 扩展。

# 在应用程序中使用 FFI

任何共享的 C 库（通常具有`*.so`扩展名）都可以使用 FFI 扩展包含在 PHP 应用程序中。如果您计划使用任何核心 PHP 库或在安装 PHP 扩展时生成的库，重要的是要注意您有能力修改 PHP 语言本身的行为。

在我们研究它是如何工作之前，让我们首先看看如何将外部 C 库整合到 PHP 脚本中，使用 FFI 扩展。

## 将外部 C 库整合到 PHP 脚本中

为了举例说明，我们使用了一个简单的函数，可能源自**计算机科学 101**（**CS101**）课程：著名的**冒泡排序**。这个算法在初学者的计算机科学课程中被广泛使用，因为它很容易理解。

重要提示

**冒泡排序**是一种极其低效的排序算法，长期以来一直被更快的排序算法（如**希尔排序**、**快速排序**或**归并排序**算法）所取代。虽然没有冒泡排序算法的权威参考，但您可以在这里阅读到一个很好的一般讨论：[`en.wikipedia.org/wiki/Bubble_sort`](https://en.wikipedia.org/wiki/Bubble_sort)。

在这个小节中，我们不会详细介绍算法。相反，这个小节的目的是演示如何将现有的 C 库并入到 PHP 脚本中的一个函数。我们现在向您展示原始的 C 源代码，如何将其转换为共享库，最后如何使用 FFI 将库整合到 PHP 中。我们将做以下事情：

1.  当然，第一步是将 C 代码编译为对象代码。以下是本例中使用的冒泡排序 C 代码：

```php
#include <stdio.h>
void bubble_sort(int [], int);
void bubble_sort(int list[], int n) {
    int c, d, t, p;
    for (c = 0 ; c < n - 1; c++) {
        p = 0;
        for (d = 0 ; d < n - c - 1; d++) {
            if (list[d] > list[d+1]) {
                t = list[d];
                list[d] = list[d+1];
                list[d+1] = t;
                p++;
            }
        }
        if (p == 0) break;
    }
}
```

1.  然后，我们使用 GNU C 编译器（包含在本课程使用的 Docker 镜像中）将 C 代码编译为对象代码，如下所示：

`gcc -c -Wall -Werror -fpic bubble.c`

1.  接下来，我们将对象代码合并到一个共享库中。这一步是必要的，因为 FFI 扩展只能访问共享库。我们运行以下代码来完成这一步：

`gcc -shared -o libbubble.so bubble.o`

1.  现在我们准备定义使用我们新共享库的 PHP 脚本。我们首先定义一个函数，该函数显示来自`FFI\CData`数组的输出，如下所示：

```php
// /repo/ch04/php8_ffi_using_func_from_lib.php
function show($label, $arr, $max) 
{
    $output = $label . "\n";
    for ($x = 0; $x < $max; $x++)
        $output .= $arr[$x] . ',';
    return substr($output, 0, -1) . "\n";
}
```

1.  接下来是关键部分：定义`FFI`实例。我们使用`FFI::cdef()`来完成这个任务，并提供两个参数。第一个参数是函数签名，第二个参数是我们新创建的共享库的路径。这两个参数都可以在以下代码片段中看到：

```php
$bubble = FFI::cdef(
    "void bubble_sort(int [], int);",
    "./libbubble.so");
```

1.  然后，我们创建了一个`FFI\CData`元素，作为一个包含 16 个随机整数的整数数组，使用`rand()`函数进行填充。代码如下所示：

```php
$max   = 16;
$arr_b = FFI::new('int[' . $max . ']');
for ($i = 0; $i < $max; $i++)
    $arr_b[$i]->cdata = rand(0,9999);
```

1.  最后，我们显示了排序之前数组的内容，执行了排序，并显示了排序后的内容。请注意，在以下代码片段中，我们使用`FFI`实例调用`bubble_sort()`来执行排序：

```php
echo show('Before Sort', $arr_b, $max);
$bubble->bubble_sort($arr_b, $max);
echo show('After Sort', $arr_b, $max);
```

1.  输出，正如您所期望的那样，在排序之前显示了一组随机整数。排序后，数值是有序的。以下是*步骤 7*中代码的输出：

```php
Before Sort
245,8405,8580,7586,9416,3524,8577,4713,
9591,1248,798,6656,9064,9846,2803,304
After Sort
245,304,798,1248,2803,3524,4713,6656,7586,
8405,8577,8580,9064,9416,9591,9846
```

既然您已经了解了如何使用 FFI 扩展将外部 C 库集成到 PHP 应用程序中，我们转向最后一个主题：PHP 回调。

## 使用 PHP 回调

正如我们在本节开头提到的，可以使用 FFI 扩展来整合实际 PHP 语言（或其扩展）中的共享 C 库。这种整合很重要，因为它允许您通过访问 PHP 共享 C 库中定义的 C 数据结构来读取和写入本机 PHP 数据。

然而，本小节的目的并不是向您展示如何创建 PHP 扩展。相反，在本小节中，我们向您介绍了 FFI 扩展覆盖本机 PHP 语言功能的能力。这种能力被称为**PHP 回调**。在我们深入实现细节之前，我们必须首先检查与这种能力相关的潜在危险。

### 理解 PHP 回调的潜在危险

重要的是要理解，PHP 共享库中定义的 C 函数通常被多个 PHP 函数使用。因此，如果您在 C 级别覆盖了其中一个低级函数，您可能会在 PHP 应用程序中遇到意外行为。

另一个已知问题是，覆盖本机 PHP C 函数很有可能会产生**内存泄漏**。随着时间的推移，使用这种覆盖的长时间运行的应用程序可能会失败，并且有可能导致服务器崩溃！

最后要考虑的是，PHP 回调功能并非在所有 FFI 平台上都受支持。因此，尽管代码可能在 Linux 服务器上运行，但在 Windows 服务器上可能无法运行（或可能无法以相同的方式运行）。

提示

与其使用 FFI PHP 回调来覆盖本机 PHP C 库功能，也许更容易、更快速、更安全的方法是只定义自己的 PHP 函数！

既然您已经了解了使用 PHP 回调涉及的危险，让我们来看一个示例实现。

### 实现 PHP 回调

在下面的示例中，使用回调覆盖了`zend_write`内部 PHP 共享库的 C 函数，该回调在输出末尾添加了**换行符**（**LF**）。请注意，此覆盖会影响任何依赖它的本机 PHP 函数，包括`echo`、`print`、`printf`：换句话说，任何产生直接输出的 PHP 函数。要实现 PHP 回调，请按照以下步骤进行：

1.  首先，我们使用`FFI::cdef()`定义了一个`FFI`实例。第一个参数是`zend_write`的函数签名。代码如下所示：

```php
// /repo/ch04/php8_php_callbacks.php
$zend = FFI::cdef("
    typedef int (*zend_write_func_t)(
        const char *str,size_t str_length);
    extern zend_write_func_t zend_write;
");
```

1.  然后，我们添加了代码来确认未经修改的`echo`不会在末尾添加额外的换行符。您可以在这里看到代码：

```php
echo "Original echo command does not output LF:\n";
echo 'A','B','C';
echo 'Next line';
```

1.  毫不奇怪，输出产生了`ABCNext line`。输出中没有回车或换行符，如下所示：

```php
Original echo command does not output LF:
ABCNext line
```

1.  然后，我们将指向`zend_write`的指针克隆到`$orig_zend_write`变量中。如果我们不这样做，我们将无法使用原始函数！代码如下所示：

$orig_zend_write = clone $zend->zend_write;

1.  接下来，我们以匿名函数的形式生成一个 PHP 回调，覆盖原始的`zend_write`函数。在函数中，我们调用原始的`zend_write`函数并在其输出中添加一个 LF，如下所示：

```php
$zend->zend_write = function($str, $len) {
    global $orig_zend_write;
    $ret = $orig_zend_write($str, $len);
    $orig_zend_write("\n", 1);
    return $ret;
};
```

1.  剩下的代码重新运行了前面步骤中显示的`echo`命令，如下所示：

```php
echo 'Revised echo command adds LF:';
echo 'A','B','C';
```

1.  以下输出演示了 PHP `echo` 命令现在在每个命令的末尾产生一个 LF：

```php
Revised echo command adds LF:
A
B
C
```

还要注意的是，修改 PHP 库 C 语言`zend_write`函数会影响使用这个 C 语言函数的所有 PHP 本机函数。这包括`print()`，`printf()`（及其变体）等。

这结束了我们对在 PHP 应用程序中使用 FFI 扩展的讨论。您现在知道如何将外部共享库中的本机 C 函数整合到 PHP 中。您还知道如何用 PHP 回调替换本机 PHP 核心或扩展共享库，从而有可能改变 PHP 语言本身的行为。

# 总结

在本章中，您了解了 FFI 及其历史，以及如何使用它来促进快速的 PHP 扩展原型设计。您还了解到，虽然 FFI 扩展不应该用于提高速度，但它也可以让您的 PHP 应用程序直接调用外部 C 库的本机 C 函数。这种能力的强大之处通过一个调用外部 C 库的冒泡排序函数的示例得到了展示。这种能力也可以扩展到包括机器学习、光学字符识别、通信、加密等成千上万个 C 库，*无穷无尽*。

在本章中，您将更深入地了解 PHP 本身在 C 语言级别的运行方式。您将学习如何创建并直接使用 C 语言数据结构，使您能够与 PHP 语言本身进行交互，甚至覆盖 PHP 语言本身。此外，您现在已经知道如何将任何 C 语言库的功能直接整合到 PHP 应用程序中。这种知识的另一个好处是，如果您找到一家计划开发自己的自定义 PHP 扩展或已经开发了自定义 PHP 扩展的公司，它将有助于增强您的职业前景。

下一章标志着书的新部分*PHP 8 技巧*的开始。在下一节中，您将学习升级到 PHP 8 时的向后兼容性问题。下一章具体讨论了面向对象编程方面的向后兼容性问题。
