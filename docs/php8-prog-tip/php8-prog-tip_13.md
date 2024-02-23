# *第十章*：提高性能

PHP 8.x 引入了许多新功能，对性能产生了积极影响。此外，许多 PHP 8 最佳实践涵盖的内容可以提高效率并降低内存使用。在本章中，您将了解如何优化您的 PHP 8 代码以实现最佳性能。

PHP 8 包括一种称为弱引用的技术。通过掌握本章最后一节讨论的这项技术，您的应用程序将使用更少的内存。通过仔细审查本章涵盖的材料并研究代码示例，您将能够编写更快，更高效的代码。这种掌握将极大地提高您作为 PHP 开发人员的地位，并带来满意的客户，同时提高您的职业潜力。

本章涵盖的主题包括以下内容：

+   使用即时（JIT）编译器

+   加速数组处理

+   实现稳定排序

+   使用弱引用来提高效率

# 技术要求

为了检查和运行本章提供的代码示例，最低推荐的硬件如下：

+   基于 x86_64 的台式 PC 或笔记本电脑

+   1 GB 的可用磁盘空间

+   4 GB 的 RAM

+   每秒 500 千比特（Kbps）或更快的互联网连接

此外，您还需要安装以下软件：

+   Docker

+   Docker Compose

有关 Docker 和 Docker Compose 安装的更多信息，请参阅*第一章*的*技术要求*部分，以及如何构建用于演示本书中解释的代码的 Docker 容器。在本书中，我们将存储本书示例代码的目录称为`/repo`。

本章的源代码位于此处：https://github.com/PacktPublishing/PHP-8-Programming-Tips-Tricks-and-Best-Practices。现在我们可以开始讨论了，看看备受期待的 JIT 编译器。

# 使用 JIT 编译器

PHP 8 引入了备受期待的 JIT 编译器。这是一个重要的步骤，对 PHP 语言的长期可行性有重要影响。尽管 PHP 已经有能力生成和缓存字节码，但在引入 JIT 编译器之前，PHP 没有直接缓存机器码的能力。

实际上，自 2011 年以来就有几次尝试为 PHP 添加 JIT 编译器功能。PHP 7 中看到的性能提升是这些早期努力的直接结果。由于它们并没有显著提高性能，因此以前的 JIT 编译器努力都没有被提议为 RFC（请求评论）。核心团队现在认为，只有使用 JIT 才能实现进一步的性能提升。作为一个附带的好处，这打开了 PHP 作为非 Web 环境语言的可能性。另一个好处是 JIT 编译器打开了使用其他语言（而不是 C）开发 PHP 扩展的可能性。

在本章中非常重要的是要仔细阅读给出的细节，因为正确使用新的 JIT 编译器有可能极大地提高 PHP 应用程序的性能。在我们深入实现细节之前，首先需要解释 PHP 在没有 JIT 编译器的情况下如何执行字节码。然后我们将向您展示 JIT 编译器的工作原理。之后，您将更好地理解各种设置以及如何对其进行微调，以产生最佳的应用程序代码性能。

让我们现在关注 PHP 在没有 JIT 编译器的情况下是如何工作的。

## 了解 PHP 在没有 JIT 的情况下如何工作

当在服务器上安装 PHP（或在 Docker 容器中），除了核心扩展之外，实际安装的主要组件是一个通常被称为**Zend 引擎**的**虚拟机**（**VM**）。这个虚拟机的运行方式与*VMware*或*Docker*等虚拟化技术大不相同。Zend 引擎更接近于**Java 虚拟机**（**JVM**），它接受*字节码*并产生*机器码*。

这引出了一个问题：*什么是字节码*和*什么是机器码*？让我们现在来看一下这个问题。

### 理解字节码和机器码

机器码，或**机器语言**，是 CPU 直接理解的一组硬件指令。每条机器码都是一条指令，会导致 CPU 执行特定的操作。这些低级操作包括在寄存器之间移动信息，在内存中移动指定字节数，加法，减法等等。

机器码通常通过使用**汇编语言**来使其在一定程度上可读。以下是一个以汇编语言呈现的机器码示例：

```php
JIT$Mandelbrot::iterate: ;
        sub $0x10, %esp
        cmp $0x1, 0x1c(%esi)
        jb .L14
        jmp .L1
.ENTRY1:
        sub $0x10, %esp
.L1:
        cmp $0x2, 0x1c(%esi)
        jb .)L15
        mov $0xec3800f0, %edi
        jmp .L2
.ENTRY2:
        sub $0x10, %esp
.L2:
        cmp $0x5, 0x48(%esi)
        jnz .L16
        vmovsd 0x40(%esi), %xmm1
        vsubsd 0xec380068, %xmm1, %xmm1
```

尽管大部分命令不容易理解，但您可以从汇编语言表示中看到指令包括比较（`cmp`），在寄存器和/或内存之间移动信息（`mov`），以及跳转到指令集中的另一个点（`jmp`）。

字节码，也称为**操作码**，是原始程序代码的大大简化的符号表示。字节码是由一个解析过程（通常称为**解释器**）产生的，该过程将可读的程序代码分解为称为**标记**的符号，以及值。值可以是程序代码中使用的任何字符串，整数，浮点数和布尔数据。

以下是基于后面显示的示例代码创建 Mandelbrot 所产生的字节码片段的一个示例：

![图 10.1 - PHP 解析过程产生的字节码片段](img/B16992_Figure_10.1.jpg)

图 10.1 - PHP 解析过程产生的字节码片段

现在让我们来看一下 PHP 程序的传统执行流程。

### 理解传统的 PHP 程序执行

在传统的 PHP 程序运行周期中，PHP 程序代码通过一个称为**解析**的操作进行评估并分解为字节码。然后将字节码传递给 Zend 引擎，Zend 引擎将字节码转换为机器码。

当 PHP 首次安装在服务器上时，安装过程会启动必要的逻辑，将 Zend 引擎定制为特定服务器的 CPU 和硬件（或虚拟 CPU 和硬件）。因此，当您编写 PHP 代码时，您并不知道最终运行代码的实际 CPU 的具体情况。正是 Zend 引擎提供了硬件特定的意识。

接下来显示的*图 10.2*说明了传统的 PHP 执行方式：

![图 10.2 - 传统的 PHP 程序执行流程](img/B16992_Figure_10.2.jpg)

图 10.2 - 传统的 PHP 程序执行流程

尽管 PHP，特别是 PHP 7，非常快，但获得额外的速度仍然很有意义。出于这个目的，大多数安装也启用了 PHP **OPcache**扩展。在继续讨论 JIT 编译器之前，让我们快速了解一下 OPcache。

### 理解 PHP OPcache 的操作

顾名思义，PHP OPcache 扩展在首次运行 PHP 程序时*缓存*了操作码（字节码）。在后续的程序运行中，字节码将从缓存中获取，消除了解析阶段。这节省了大量时间，是一个在生产环境中启用的非常理想的功能。PHP OPcache 扩展是核心扩展集的一部分；但是，默认情况下它并未启用。

在启用此扩展之前，您必须首先确认您的 PHP 版本是否使用了`--enable-opcache`配置选项进行编译。您可以通过在运行在 Web 服务器上的 PHP 代码中执行`phpinfo()`命令来检查这一点。从命令行中，输入`php -i`命令。以下是在本书使用的 Docker 容器中运行`php -i`的示例：

```php
root@php8_tips_php8 [ /repo/ch10 ]# php -i
phpinfo()
PHP Version => 8.1.0-dev
System => Linux php8_tips_php8 5.8.0-53-generic #60~20.04.1-Ubuntu SMP Thu May 6 09:52:46 UTC 2021 x86_64
Build Date => Dec 24 2020 00:11:29
Build System => Linux 9244ac997bc1 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt11-1 (2015-05-24) x86_64 GNU/Linux
Configure Command =>  './configure'  '--prefix=/usr' '--sysconfdir=/etc' '--localstatedir=/var' '--datadir=/usr/share/php' '--mandir=/usr/share/man' '--enable-fpm' '--with-fpm-user=apache' '--with-fpm-group=apache'
// not all options shown
'--with-jpeg' '--with-png' '--with-sodium=/usr' '--enable-opcache-jit' '--with-pcre-jit' '--enable-opcache'
```

从输出中可以看出，OPcache 已包含在此 PHP 安装的配置中。要启用 OPcache，请添加或取消注释以下`php.ini`文件设置：

+   `zend_extension=opcache`

+   `opcache.enable=1`

+   `opcache.enable_cli=1`

最后一个设置是可选的。它确定是否还要处理从命令行执行的 PHP 命令。一旦启用，还有许多其他`php.ini`文件设置会影响性能，但这超出了本讨论的范围。

提示

有关影响 OPcache 的 PHP `php.ini`文件设置的更多信息，请查看这里：https://www.php.net/manual/en/opcache.configuration.php。

现在让我们来看看 JIT 编译器的运行方式，以及它与 OPcache 的区别。

### 使用 JIT 编译器发现 PHP 程序执行

当前方法的问题在于，无论字节码是否被缓存，Zend 引擎仍然需要每次程序请求时将字节码转换为机器代码。JIT 编译器提供的是将字节码编译成机器代码并且*缓存机器代码*的能力。这个过程是通过一个跟踪机制来实现的，它创建请求的跟踪。跟踪允许 JIT 编译器确定哪些块的机器代码需要被优化和缓存。使用 JIT 编译器的执行流程总结在*图 10.3*中：

![图 10.3 - 带有 JIT 编译器的 PHP 执行流](img/B16992_Figure_10.3.jpg)

图 10.3 - 带有 JIT 编译器的 PHP 执行流

从图表中可以看出，包含 OPcache 的正常执行流仍然存在。主要区别在于请求可能会调用一个 trace，导致程序流立即转移到 JIT 编译器，有效地绕过了解析过程和 Zend 引擎。JIT 编译器和 Zend 引擎都可以生成准备直接执行的机器代码。

JIT 编译器并非凭空产生。PHP 核心团队选择移植了高性能和经过充分测试的**DynASM**预处理汇编器。虽然 DynASM 主要是为**Lua**编程语言使用的 JIT 编译器开发的，但其设计非常适合作为任何基于 C 的语言（如 PHP！）的 JIT 编译器的基础。

PHP JIT 实现的另一个有利方面是它不会产生任何**中间表示**（**IR**）代码。相比之下，用于使用 JIT 编译器技术运行 Python 代码的**PyPy VM**必须首先产生**图结构**中的 IR 代码，用于流分析和优化，然后才能产生实际的机器代码。PHP JIT 中的 DynASM 核心不需要这一额外步骤，因此比其他解释性编程语言可能实现的性能更高。

提示

有关 DynASM 的更多信息，请查看此网站：https://luajit.org/dynasm.html。这是关于 PHP 8 JIT 操作的出色概述：https://www.zend.com/blog/exploring-new-php-jit-compiler。您还可以在这里阅读官方的 JIT RFC：https://wiki.php.net/rfc/jit。

现在您已经了解了 JIT 编译器如何适应 PHP 程序执行周期的一般流程，是时候学习如何启用它了。

## 启用 JIT 编译器

因为 JIT 编译器的主要功能是缓存机器代码，它作为 OPcache 扩展的独立部分运行。OPcache 既可以作为启用 JIT 功能的网关，也可以从自己的分配中为 JIT 编译器分配内存。因此，为了启用 JIT 编译器，您必须首先启用 OPcache（请参阅前一节，*理解 PHP OPcache 的操作*）。

为了启用 JIT 编译器，您必须首先确认 PHP 已经使用`--enable-opcache-jit`配置选项进行编译。然后，您可以通过简单地将非零值分配给`php.ini`文件的`opcache.jit_buffer_size`指令来启用或禁用 JIT 编译器。

值可以指定为整数——在这种情况下，该值表示字节数；值为零（默认值）会禁用 JIT 编译器；或者您可以分配一个数字，后面跟着以下任何一个字母：

+   `K`：千字节

+   `M`：兆字节

+   `G`：千兆字节

您为 JIT 编译器缓冲区大小指定的值必须小于您为 OPcache 分配的内存分配，因为 JIT 缓冲区是从 OPcache 缓冲区中取出的。

以下是一个示例，将 OPcache 内存消耗设置为 256 M，JIT 缓冲区设置为 64 M。这些值可以放在`php.ini`文件的任何位置：

```php
opcache.memory_consumption=256
opcache.jit_buffer_size=64M
```

现在您已经了解了 JIT 编译器的工作原理，以及如何启用它，了解如何正确设置跟踪模式非常重要。

## 配置跟踪模式

`php.ini`设置`opcache.jit`控制 JIT 跟踪器的操作。为了方便起见，可以使用以下四个预设字符串之一：

+   `opcache.jit=disable`

完全禁用 JIT 编译器（不考虑其他设置）。

+   `opcache.jit=off`

禁用 JIT 编译器，但（在大多数情况下）您可以使用`ini_set()`在运行时启用它。

+   `opcache.jit=function`

将 JIT 编译器跟踪器设置为功能模式。此模式对应于**CPU 寄存器触发优化（CRTO）**数字 1205（下面解释）。

+   `opcache.jit=tracing`

将 JIT 编译器跟踪器设置为跟踪模式。此模式对应于 CRTO 数字 1254（下面解释）。在大多数情况下，此设置可以获得最佳性能。

+   `opcache.jit=on`

这是跟踪模式的别名。

提示

依赖运行时 JIT 激活是有风险的，并且可能产生不一致的应用程序行为。最佳实践是使用`tracing`或`function`设置。

这四个便利字符串实际上解析为一个四位数。每个数字对应 JIT 编译器跟踪器的不同方面。这四个数字不像其他`php.ini`文件设置那样是位掩码，并且按照这个顺序指定：`CRTO`。以下是每个四位数的摘要。

### C（CPU 优化标志）

第一个数字代表 CPU 优化设置。如果将此数字设置为 0，则不会进行 CPU 优化。值为 1 会启用**高级矢量扩展**（**AVX**）**指令**的生成。AVX 是针对英特尔和 AMD 微处理器的 x86 指令集架构的扩展。自 2011 年以来，AVX 已在英特尔和 AMD 处理器上得到支持。大多数服务器型处理器（如英特尔至强）都支持 AVX2。

### R（寄存器分配）

第二位数字控制 JIT 编译器如何处理**寄存器**。寄存器类似于 RAM，只是它们直接驻留在 CPU 内部。CPU 不断地在寄存器中移动信息，以执行操作（例如，加法、减法、执行逻辑 AND、OR 和 NOT 操作等）。与此设置相关的选项允许您禁用寄存器分配优化，或者在本地或全局级别允许它。

### T（JIT 触发器）

第三位数字决定 JIT 编译器何时触发。选项包括在加载脚本时首次操作 JIT 编译器或在首次执行时操作。或者，您可以指示 JIT 何时编译**热函数**。热函数是最常被调用的函数。还有一个设置，告诉 JIT 只编译带有`@jit docblock`注释的函数。

### O（优化级别）

第四位数字对应优化级别。选项包括禁用优化、最小化和选择性。您还可以指示 JIT 编译器根据单个函数、调用树或内部过程分析的结果进行优化。

提示

要完全了解四个 JIT 编译器跟踪器设置，请查看此文档参考页面：https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.jit。

现在让我们来看看 JIT 编译器的运行情况。

## 使用 JIT 编译器

在这个例子中，我们使用一个经典的基准测试程序来生成**Mandelbrot**。这是一个非常消耗计算资源的优秀测试。我们在这里使用的实现是来自 PHP 核心开发团队成员**Dmitry Stogov**的实现代码。您可以在这里查看原始实现：[`gist.github.com/dstogov/12323ad13d3240aee8f1`](https://gist.github.com/dstogov/12323ad13d3240aee8f1)：

1.  我们首先定义 Mandelbrot 参数。特别重要的是迭代次数（`MAX_LOOPS`）。较大的数字会产生更多的计算并减慢整体生产速度。我们还捕获开始时间：

```php
// /repo/ch10/php8_jit_mandelbrot.php
define('BAILOUT',   16);
define('MAX_LOOPS', 10000);
define('EDGE',      40.0);
$d1  = microtime(1);
```

1.  为了方便多次运行程序，我们添加了一个捕获命令行参数`-n`的选项。如果存在此参数，则 Mandelbrot 输出将被抑制：

```php
$time_only = (bool) ($argv[1] ?? $_GET['time'] ?? FALSE);
```

1.  然后，我们定义一个名为`iterate()`的函数，直接从 Dmitry Stogov 的 Mandelbrot 实现中提取。实际代码在此未显示，可以在前面提到的 URL 中查看。

1.  接下来，我们通过`EDGE`确定的 X/Y 坐标运行，生成 ASCII 图像：

```php
$out = '';
$f   = EDGE - 1;
for ($y = -$f; $y < $f; $y++) {
    for ($x = -$f; $x < $f; $x++) {
        $out .= (iterate($x/EDGE,$y/EDGE) == 0)
              ? '*' : ' ';
    }
    $out .= "\n";
}
```

1.  最后，我们生成输出。如果通过 Web 请求运行，则输出将包含在`<pre>`标签中。如果存在`-n`标志，则只显示经过的时间：

```php
if (!empty($_SERVER['REQUEST_URI'])) {
    $out = '<pre>' . $out . '</pre>';
}
if (!$time_only) echo $out;
$d2 = microtime(1);
$diff = $d2 - $d1;
printf("\nPHP Elapsed %0.3f\n", $diff);
```

1.  我们首先在 PHP 7 Docker 容器中使用`-n`标志运行程序三次。以下是结果。请注意，在与本书配合使用的演示 Docker 容器中，经过的时间很容易超过 10 秒：

```php
root@php8_tips_php7 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 10.320
root@php8_tips_php7 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 10.134
root@php8_tips_php7 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 11.806
```

1.  现在我们转向 PHP 8 Docker 容器。首先，我们调整`php.ini`文件以禁用 JIT 编译器。以下是设置：

```php
opcache.jit=off
opcache.jit_buffer_size=0
```

1.  以下是在使用`-n`标志的 PHP 8 中运行程序三次的结果：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 1.183
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 1.192
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 1.210
```

1.  立即可以看到切换到 PHP 8 的一个很好的理由！即使没有 JIT 编译器，PHP 8 也能在 1 秒多一点的时间内执行相同的程序：1/10 的时间量！

1.  接下来，我们修改`php.ini`文件设置，以使用 JIT 编译器`function`跟踪器模式。以下是使用的设置：

```php
opcache.jit=function
opcache.jit_buffer_size=64M
```

1.  然后我们再次使用`-n`标志运行相同的程序。以下是在使用 JIT 编译器`function`跟踪器模式的 PHP 8 中运行的结果：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.323
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.322
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.324
```

1.  哇！我们成功将处理速度提高了 3 倍。速度现在不到 1/3 秒！但是如果我们尝试推荐的 JIT 编译器`tracing`模式会发生什么呢？以下是调用该模式的设置：

```php
opcache.jit=tracing
opcache.jit_buffer_size=64M
```

1.  以下是我们上一组程序运行的结果：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.132
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.132
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
PHP Elapsed 0.131
```

正如输出所示，最后的结果真是令人震惊。我们不仅可以比没有 JIT 编译器的 PHP 8 运行相同的程序快 10 倍，而且比 PHP 7 运行*快 100 倍*！

重要提示

重要的是要注意，时间会根据您用于运行与本书相关的 Docker 容器的主机计算机而变化。您将看不到与此处显示的完全相同的时间。

现在让我们来看看 JIT 编译器调试。

## 使用 JIT 编译器进行调试

当使用 JIT 编译器时，使用**XDebug**或其他工具进行常规调试效果不佳。因此，PHP 核心团队添加了一个额外的`php.ini`文件选项`opcache.jit_debug`，它会生成额外的调试信息。在这种情况下，可用的设置采用位标志的形式，这意味着您可以使用按位运算符（如`AND`，`OR`，`XOR`等）将它们组合起来。

*表 10.1*总结了可以分配为`opcache.jit_debug`设置的值。请注意，标有**内部常量**的列不显示 PHP 预定义常量。这些值是内部 C 代码引用：

![表 10.1 - opcache.jit_debug 设置](img/Table_10.1_B16992.jpg)

表 10.1 - opcache.jit_debug 设置

例如，如果您希望为`ZEND_JIT_DEBUG_ASM`，`ZEND_JIT_DEBUG_PERF`和`ZEND_JIT_DEBUG_EXIT`启用调试，可以在`php.ini`文件中进行如下分配：

1.  首先，您需要将要设置的值相加。在这个例子中，我们将添加：

`1 + 16 + 32768`

1.  然后将总和应用于`php.ini`设置：

`opcache.jit_debug=32725`

1.  或者，使用按位`OR`表示这些值：

`opcache.jit_debug=1|16|32768`

根据调试设置，您现在可以使用诸如 Linux `perf`命令或 Intel `VTune`之类的工具来调试 JIT 编译器。

以下是在运行前一节讨论的 Mandelbrot 测试程序时的部分调试输出示例。为了说明，我们使用了`php.ini`文件设置`opcache.jit_debug=32725`：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_jit_mandelbrot.php -n
---- TRACE 1 start (loop) iterate() /repo/ch10/php8_jit_mandelbrot.php:34
---- TRACE 1 stop (loop)
---- TRACE 1 Live Ranges
#15.CV6($i): 0-0 last_use
#19.CV6($i): 0-20 hint=#15.CV6($i)
... not all output is shown
---- TRACE 1 compiled
---- TRACE 2 start (side trace 1/7) iterate()
/repo/ch10/php8_jit_mandelbrot.php:41
---- TRACE 2 stop (return)
TRACE-2$iterate$41: ; (unknown)
    mov $0x2, EG(jit_trace_num)
    mov 0x10(%r14), %rcx
    test %rcx, %rcx
    jz .L1
    mov 0xb0(%r14), %rdx
    mov %rdx, (%rcx)
    mov $0x4, 0x8(%rcx)
...  not all output is shown
```

输出显示的是用汇编语言呈现的机器代码。如果在使用 JIT 编译器时遇到程序代码问题，汇编语言转储可能会帮助您找到错误的源头。

但是，请注意，汇编语言不具有可移植性，完全面向使用的 CPU。因此，您可能需要获取该 CPU 的硬件参考手册，并查找正在使用的汇编语言代码。

现在让我们来看看影响 JIT 编译器操作的其他`php.ini`文件设置。

## 发现额外的 JIT 编译器设置

*表 10.2*提供了`php.ini`文件中尚未涵盖的所有其他`opcache.jit*`设置的摘要：

![表 10.2 - 附加的 opcache.jit* php.ini 文件设置](img/Table_10.2_B16992.jpg)

表 10.2 - 附加的 opcache.jit* php.ini 文件设置

从表中可以看出，您对 JIT 编译器的操作有很高的控制度。总体而言，这些设置代表了控制 JIT 编译器做出决策的阈值。如果正确配置这些设置，JIT 编译器可以忽略不经常使用的循环和函数调用。现在我们将离开 JIT 编译器的激动人心的世界，看看如何提高数组性能。

# 加速数组处理

数组是任何 PHP 程序的重要组成部分。实际上，处理数组是不可避免的，因为您的程序每天处理的大部分现实世界数据都以数组的形式到达。一个例子是来自 HTML 表单提交的数据。数据最终以数组的形式出现在`$_GET`或`$_POST`中。

在本节中，我们将向您介绍 SPL 中包含的一个鲜为人知的类：`SplFixedArray`类。将数据从标准数组迁移到`SplFixedArray`实例不仅可以提高性能，而且还需要更少的内存。学习如何利用本章涵盖的技术可以对当前使用大量数据的数组的任何程序代码的速度和效率产生重大影响。

## 在 PHP 8 中使用 SplFixedArray

`SplFixedArray`类是在 PHP 5.3 中引入的，它实际上是一个像数组一样操作的对象。然而，与`ArrayObject`不同，这个类要求您对数组大小设置一个硬限制，并且只允许整数索引。您可能想要使用`SplFixedArray`而不是`ArrayObject`的原因是，`SplFixedArray`占用的内存明显更少，并且性能非常好。事实上，`SplFixedArray`实际上比具有相同数据的标准数组占用*更少的内存*！

### 将 SplFixedArray 与数组和 ArrayObject 进行比较

一个简单的基准程序说明了标准数组、`ArrayObject`和`SplFixedArray`之间的差异：

1.  首先，我们定义了代码中稍后使用的一对常量：

```php
// /repo/ch10/php7_spl_fixed_arr_size.php
define('MAX_SIZE', 1000000);
define('PATTERN', "%14s : %8.8f : %12s\n");
```

1.  接下来，我们定义一个函数，该函数添加了 100 万个由 64 个字节长的字符串组成的元素：

```php
function testArr($list, $label) {
    $alpha = new InfiniteIterator(
        new ArrayIterator(range('A','Z')));
    $start_mem = memory_get_usage();
    $start_time = microtime(TRUE);
    for ($x = 0; $x < MAX_SIZE; $x++) {
        $letter = $alpha->current();
        $alpha->next();
        $list[$x] = str_repeat($letter, 64);
    }
    $mem_diff = memory_get_usage() - $start_mem;
    return [$label, (microtime(TRUE) - $start_time),
        number_format($mem_diff)];
}
```

1.  然后，我们调用该函数三次，分别提供`array`、`ArrayObject`和`SplFixedArray`作为参数：

```php
printf("%14s : %10s : %12s\n", '', 'Time', 'Memory');
$result = testArr([], 'Array');
vprintf(PATTERN, $result);
$result = testArr(new ArrayObject(), 'ArrayObject');
vprintf(PATTERN, $result);
$result = testArr(
    new SplFixedArray(MAX_SIZE), 'SplFixedArray');
vprintf(PATTERN, $result);
```

1.  以下是我们的 PHP 7.1 Docker 容器的结果：

```php
root@php8_tips_php7 [ /repo/ch10 ]# 
php php7_spl_fixed_arr_size.php 
               :       Time :       Memory
         Array : 1.19430900 :  129,558,888
   ArrayObject : 1.20231009 :  129,558,832
 SplFixedArray : 1.19744802 :   96,000,280
```

1.  在 PHP 8 中，所花费的时间显著减少，如下所示：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php7_spl_fixed_arr_size.php 
               :       Time :       Memory
         Array : 0.13694692 :  129,558,888
   ArrayObject : 0.11058593 :  129,558,832
 SplFixedArray : 0.09748793 :   96,000,280
```

从结果中可以看出，PHP 8 处理数组的速度比 PHP 7.1 快 10 倍。两个版本使用的内存量是相同的。无论使用哪个版本的 PHP，`SplFixedArray`使用的内存量都明显少于标准数组或`ArrayObject`。现在让我们来看看在 PHP 8 中`SplFixedArray`的使用方式发生了哪些变化。

### 在 PHP 8 中使用 SplFixedArray 的变化

您可能还记得在*第七章*中对`Traversable`接口的简要讨论，*在使用 PHP 8 扩展时避免陷阱*，*Traversable to IteratorAggregate migration*部分。在该部分提出的相同考虑也适用于`SplFixedArray`。虽然`SplFixedArray`没有实现`Traversable`，但它实现了`Iterator`，而`Iterator`又扩展了`Traversable`。

在 PHP 8 中，`SplFixedArray`不再实现`Iterator`。相反，它实现了`IteratorAggregate`。这种变化的好处是，PHP 8 中的`SplFixedArray`更快，更高效，并且在嵌套循环中使用也更安全。不利之处，也是潜在的代码中断，是如果您正在与以下任何方法一起使用`SplFixedArray`：`current()`、`key()`、`next()`、`rewind()`或`valid()`。

如果您需要访问数组导航方法，现在必须使用`SplFixedArray::getIterator()`方法来访问内部迭代器，从中可以使用所有导航方法。下面的简单代码示例说明了潜在的代码中断：

1.  我们首先从数组构建一个`SplFixedArray`实例：

```php
// /repo/ch10/php7_spl_fixed_arr_iter.php
$arr   = ['Person', 'Woman', 'Man', 'Camera', 'TV'];$fixed = SplFixedArray::fromArray($arr);
```

1.  然后，我们使用数组导航方法来遍历数组：

```php
while ($fixed->valid()) {
    echo $fixed->current() . '. ';
    $fixed->next();
}
```

在 PHP 7 中，输出是数组中的五个单词：

```php
root@php8_tips_php7 [ /repo/ch10 ]# 
php php7_spl_fixed_arr_iter.php 
Person. Woman. Man. Camera. TV.
```

在 PHP 8 中，结果却大不相同，如下所示：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php7_spl_fixed_arr_iter.php 
PHP Fatal error:  Uncaught Error: Call to undefined method SplFixedArray::valid() in /repo/ch10/php7_spl_fixed_arr_iter.php:5
```

为了使示例在 PHP 8 中工作，您只需要使用`SplFixedArray::getIterator()`方法来访问内部迭代器。代码的其余部分不需要重写。以下是为 PHP 8 重新编写的修订后的代码示例：

```php
// /repo/ch10/php8_spl_fixed_arr_iter.php
$arr   = ['Person', 'Woman', 'Man', 'Camera', 'TV'];
$obj   = SplFixedArray::fromArray($arr);
$fixed = $obj->getIterator();
while ($fixed->valid()) {
    echo $fixed->current() . '. ';
    $fixed->next();
}
```

现在输出的是五个单词，没有任何错误：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_spl_fixed_arr_iter.php
Person. Woman. Man. Camera. TV. 
```

现在您已经了解了如何提高数组处理性能，我们将把注意力转向数组性能的另一个方面：排序。

# 实现稳定排序

在设计数组排序逻辑时，最初的 PHP 开发人员为了速度而牺牲了稳定性。当时，这被认为是一个合理的牺牲。然而，如果在排序过程中涉及复杂对象，则需要**稳定排序**。

在本节中，我们将讨论稳定排序是什么，以及为什么它很重要。如果您可以确保数据被稳定排序，您的应用代码将产生更准确的输出，从而提高客户满意度。在我们深入了解 PHP 8 如何实现稳定排序之前，我们首先需要定义什么是稳定排序。

## 理解稳定排序

当用于排序目的的属性的值相等时，在*稳定排序*中保证了元素的原始顺序。这样的结果更接近用户的期望。让我们看一个简单的数据集，并确定什么构成了稳定排序。为了说明，让我们假设我们的数据集包括访问时间和用户名的条目：

```php
2021-06-01 11:11:11    Betty
2021-06-03 03:33:33    Betty
2021-06-01 11:11:11    Barney
2021-06-02 02:22:22    Wilma
2021-06-01 11:11:11    Wilma
2021-06-03 03:33:33    Barney
2021-06-01 11:11:11    Fred
```

如果我们希望按时间排序，您会立即注意到`2021-06-01 11:11:11`存在重复。如果我们对这个数据集执行稳定排序，预期的结果将如下所示：

```php
2021-06-01 11:11:11    Betty
2021-06-01 11:11:11    Barney
2021-06-01 11:11:11    Wilma
2021-06-01 11:11:11    Fred
2021-06-02 02:22:22    Wilma
2021-06-03 03:33:33    Betty
2021-06-03 03:33:33    Barney
```

您会注意到从排序后的数据集中，重复时间`2021-06-01 11:11:11`的条目按照它们最初的输入顺序出现。因此，我们可以说这个结果代表了一个稳定的排序。

在理想的情况下，相同的原则也应该适用于保留键/值关联的排序。稳定排序的一个额外标准是，它在性能上不应该与无序排序有任何差异。

提示

有关 PHP 8 稳定排序的更多信息，请查看官方 RFC：https://wiki.php.net/rfc/stable_sorting。

在 PHP 8 中，核心的`*sort*()`函数和`ArrayObject::*sort*()`方法已经被重写以实现稳定排序。让我们看一个代码示例，说明在 PHP 的早期版本中可能出现的问题。

## 对比稳定和非稳定排序

在这个例子中，我们希望按时间对`Access`实例的数组进行排序。每个`Access`实例有两个属性，`$name`和`$time`。样本数据集包含重复的访问时间，但用户名不同：

1.  首先，我们定义`Access`类：

```php
// /repo/src/Php8/Sort/Access.php
namespace Php8\Sort;
class Access {
    public $name, $time;
    public function __construct($name, $time) {
        $this->name = $name;
        $this->time = $time;
    }
}
```

1.  接下来，我们定义一个样本数据集，其中包含一个 CSV 文件，`/repo/sample_data/access.csv`，共有 21 行。每一行代表不同的姓名和访问时间的组合：

```php
"Fred",  "2021-06-01 11:11:11"
"Fred",  "2021-06-01 02:22:22"
"Betty", "2021-06-03 03:33:33"
"Fred",  "2021-06-11 11:11:11"
"Barney","2021-06-03 03:33:33"
"Betty", "2021-06-01 11:11:11"
"Betty", "2021-06-11 11:11:11"
"Barney","2021-06-01 11:11:11"
"Fred",  "2021-06-11 02:22:22"
"Wilma", "2021-06-01 11:11:11"
"Betty", "2021-06-13 03:33:33"
"Fred",  "2021-06-21 11:11:11"
"Betty", "2021-06-21 11:11:11"
"Barney","2021-06-13 03:33:33"
"Betty", "2021-06-23 03:33:33"
"Barney","2021-06-11 11:11:11"
"Barney","2021-06-21 11:11:11"
"Fred",  "2021-06-21 02:22:22"
"Barney","2021-06-23 03:33:33"
"Wilma", "2021-06-21 11:11:11"
"Wilma", "2021-06-11 11:11:11"
```

您会注意到，扫描样本数据时，所有具有`11:11:11`作为入口时间的日期都是重复的，但是您还会注意到，任何给定日期的原始顺序始终是用户`Fred`，`Betty`，`Barney`和`Wilma`。另外，请注意，对于时间为`03:33:33`的日期，`Betty`的条目总是在`Barney`之前。

1.  然后我们定义一个调用程序。在这个程序中，首先要做的是配置自动加载和`use` `Access`类：

```php
// /repo/ch010/php8_sort_stable_simple.php
require __DIR__ . 
'/../src/Server/Autoload/Loader.php';
$loader = new \Server\Autoload\Loader();
use Php8\Sort\Access;
```

1.  接下来，我们将样本数据加载到`$access`数组中：

```php
$access = [];
$data = new SplFileObject(__DIR__ 
    . '/../sample_data/access.csv');
while ($row = $data->fgetcsv())
    if (!empty($row) && count($row) === 2)
        $access[] = new Access($row[0], $row[1]);
```

1.  然后我们执行`usort()`。请注意，用户定义的回调函数执行每个实例的`time`属性的比较：

```php
usort($access, 
    function($a, $b) { return $a->time <=> $b->time; });
```

1.  最后，我们循环遍历新排序的数组并显示结果：

```php
foreach ($access as $entry)
    echo $entry->time . "\t" . $entry->name . "\n";
```

在 PHP 7 中，请注意虽然时间是有序的，但是姓名并不反映预期的顺序`Fred`，`Betty`，`Barney`和`Wilma`。以下是 PHP 7 的输出：

```php
root@php8_tips_php7 [ /repo/ch10 ]# 
php php8_sort_stable_simple.php 
2021-06-01 02:22:22    Fred
2021-06-01 11:11:11    Fred
2021-06-01 11:11:11    Wilma
2021-06-01 11:11:11    Betty
2021-06-01 11:11:11    Barney
2021-06-03 03:33:33    Betty
2021-06-03 03:33:33    Barney
2021-06-11 02:22:22    Fred
2021-06-11 11:11:11    Barney
2021-06-11 11:11:11    Wilma
2021-06-11 11:11:11    Betty
2021-06-11 11:11:11    Fred
2021-06-13 03:33:33    Barney
2021-06-13 03:33:33    Betty
2021-06-21 02:22:22    Fred
2021-06-21 11:11:11    Fred
2021-06-21 11:11:11    Betty
2021-06-21 11:11:11    Barney
2021-06-21 11:11:11    Wilma
2021-06-23 03:33:33    Betty
2021-06-23 03:33:33    Barney
```

从输出中可以看出，在第一组`11:11:11`日期中，最终顺序是`Fred`，`Wilma`，`Betty`和`Barney`，而原始的入口顺序是`Fred`，`Betty`，`Barney`和`Wilma`。您还会注意到，对于日期和时间`2021-06-13 03:33:33`，`Barney`在`Betty`之前，而原始的入口顺序是相反的。根据我们的定义，PHP 7 没有实现稳定排序！

现在让我们看一下在 PHP 8 中运行相同代码示例的输出。以下是 PHP 8 的输出：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_sort_stable_simple.php
2021-06-01 02:22:22    Fred
2021-06-01 11:11:11    Fred
2021-06-01 11:11:11    Betty
2021-06-01 11:11:11    Barney
2021-06-01 11:11:11    Wilma
2021-06-03 03:33:33    Betty
2021-06-03 03:33:33    Barney
2021-06-11 02:22:22    Fred
2021-06-11 11:11:11    Fred
2021-06-11 11:11:11    Betty
2021-06-11 11:11:11    Barney
2021-06-11 11:11:11    Wilma
2021-06-13 03:33:33    Betty
2021-06-13 03:33:33    Barney
2021-06-21 02:22:22    Fred
2021-06-21 11:11:11    Fred
2021-06-21 11:11:11    Betty
2021-06-21 11:11:11    Barney
2021-06-21 11:11:11    Wilma
2021-06-23 03:33:33    Betty
2021-06-23 03:33:33    Barney
```

从 PHP 8 的输出中可以看出，对于所有的`11:11:11`条目，原始的输入顺序`Fred`，`Betty`，`Barney`和`Wilma`都得到了尊重。您还会注意到，对于日期和时间`2021-06-13 03:33:33`，`Betty`始终在`Barney`之前。因此，我们可以得出结论，PHP 8 执行了稳定排序。

现在您已经看到了 PHP 7 中的问题，并且现在知道了 PHP 8 如何解决这个问题，让我们来看看稳定排序对键的影响。

## 检查稳定排序对键的影响

稳定排序的概念也影响使用`asort()`、`uasort()`或等效的`ArrayIterator`方法时的键/值对。在接下来展示的示例中，`ArrayIterator`被填充了 20 个元素，每隔一个元素是重复的。键是一个按顺序递增的十六进制数：

1.  首先，我们定义一个函数来生成随机的 3 个字母组合：

```php
// /repo/ch010/php8_sort_stable_keys.php
$randVal = function () {
    $alpha = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    return $alpha[rand(0,25)] . $alpha[rand(0,25)] 
           . $alpha[rand(0,25)];};
```

1.  接下来，我们使用示例数据加载了一个`ArrayIterator`实例。每隔一个元素是重复的。我们还记录了开始时间：

```php
$start = microtime(TRUE);
$max   = 20;
$iter  = new ArrayIterator;
for ($x = 256; $x < $max + 256; $x += 2) {
    $key = sprintf('%04X', $x);
    $iter->offsetSet($key, $randVal());
    $key = sprintf('%04X', $x + 1);
    $iter->offsetSet($key, 'AAA'); // <-- duplicate
}
```

1.  然后我们执行`ArrayIterator::asort()`并显示结果的顺序以及经过的时间：

```php
// not all code is shown
$iter->asort();
foreach ($iter as $key => $value) echo "$key\t$value\n";
echo "\nElapsed Time: " . (microtime(TRUE) - $start);
```

以下是在 PHP 7 中运行此代码示例的结果：

```php
root@php8_tips_php7 [ /repo/ch10 ]# 
php php8_sort_stable_keys.php 
0113    AAA
010D    AAA
0103    AAA
0105    AAA
0111    AAA
0107    AAA
010F    AAA
0109    AAA
0101    AAA
010B    AAA
0104    CBC
... some output omitted ...
010C    ZJW
Elapsed Time: 0.00017094612121582
```

从输出中可以看出，尽管值是有序的，但在重复值的情况下，键是以混乱的顺序出现的。相比之下，看一下在 PHP 8 中运行相同程序代码的输出：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_sort_stable_keys.php 
0101    AAA
0103    AAA
0105    AAA
0107    AAA
0109    AAA
010B    AAA
010D    AAA
010F    AAA
0111    AAA
0113    AAA
0100    BAU
... some output omitted ...
0104    QEE
Elapsed Time: 0.00010395050048828
```

输出显示，任何重复条目的键都按照它们原始的顺序出现在输出中。输出表明，PHP 8 不仅对值实现了稳定排序，而且对键也实现了稳定排序。此外，从经过的时间结果来看，PHP 8 已经成功地保持了与以前相同（或更好）的性能。现在让我们将注意力转向 PHP 8 中直接影响数组排序的另一个不同之处：处理非法排序函数。

## 处理非法排序函数

PHP 7 及更早版本允许开发人员在使用`usort()`或`uasort()`（或等效的`ArrayIterator`方法）时使用**非法函数**。您非常重要的是要意识到这种不良实践。否则，当您将代码迁移到 PHP 8 时，可能存在潜在的向后兼容性问题。

在接下来展示的示例中，创建了与“对比稳定和非稳定排序”部分中描述的示例相同的数组。*非法*排序函数返回一个布尔值，而`u*sort()`回调需要返回两个元素之间的*相对位置*。从字面上讲，用户定义的函数或回调需要在第一个操作数小于第二个操作数时返回`-1`，相等时返回`0`，第一个操作数大于第二个操作数时返回`1`。如果我们重写定义`usort()`回调的代码行，一个非法函数可能如下所示：

```php
usort($access, function($a, $b) { 
    return $a->time < $b->time; });
```

在这段代码片段中，我们没有使用太空船操作符（`<=>`），而是使用了小于符号（`<`）。在 PHP 7 及更低版本中，返回布尔返回值的回调是可以接受的，并且会产生期望的结果。但实际发生的是，PHP 解释器需要添加额外的操作来弥补缺失的操作。因此，如果回调只执行这个比较：

`op1 > op2`

PHP 解释器添加了一个额外的操作：

`op1 <= op2`

在 PHP 8 中，非法排序函数会产生一个弃用通知。以下是在 PHP 8 中运行的重写代码：

```php
root@php8_tips_php8 [ /repo/ch10 ]#
php php8_sort_illegal_func.php 
PHP Deprecated:  usort(): Returning bool from comparison function is deprecated, return an integer less than, equal to, or greater than zero in /repo/ch10/php8_sort_illegal_func.php on line 30
2021-06-01 02:22:22    Fred
2021-06-01 11:11:11    Fred
2021-06-01 11:11:11    Betty
2021-06-01 11:11:11    Barney
... not all output is shown
```

从输出中可以看出，PHP 8 允许操作继续进行，并且在使用正确的回调时结果是一致的。但是，您还可以看到发出了一个`Deprecation`通知。

提示

您也可以在 PHP 8 中使用箭头函数。之前展示的回调可以重写如下：

`usort($array, fn($a, $b) => $a <=> $b)`。

现在您对稳定排序是什么以及为什么它很重要有了更深入的了解。您还能够发现由于 PHP 8 和早期版本之间处理差异而可能出现的潜在问题。现在我们将看一下 PHP 8 中引入的其他性能改进。

# 使用弱引用来提高效率

随着 PHP 的不断发展和成熟，越来越多的开发人员开始使用 PHP 框架来促进快速应用程序开发。然而，这种做法的一个必然副产品是占用内存的对象变得越来越大和复杂。包含许多属性、其他对象或大型数组的大对象通常被称为**昂贵的对象**。

这种趋势引起的潜在内存问题的加剧是，所有 PHP 对象赋值都是自动通过引用进行的。没有引用，第三方框架的使用将变得非常麻烦。然而，当您通过引用分配一个对象时，对象必须保持在内存中，直到所有引用被销毁。只有在取消设置或覆盖对象之后，对象才会完全被销毁。

在 PHP 7.4 中，弱引用支持以解决这个问题的潜在解决方案首次引入。PHP 8 通过添加弱映射类扩展了这种新能力。在本节中，您将学习这项新技术的工作原理，以及它如何对开发有利。让我们先看看弱引用。

## 利用弱引用

**弱引用** 首次在 PHP 7.4 中引入，并在 PHP 8 中得到改进。这个类作为对象创建的包装器，允许开发人员以一种方式使用对象的引用，使得超出范围（例如 `unset()`）的对象不受垃圾回收的保护。

目前有许多 PHP 扩展驻留在 [pecl.php.net](http://pecl.php.net)，提供对弱引用的支持。大多数实现都是通过入侵 PHP 语言核心的 C 语言结构，要么重载对象处理程序，要么操纵堆栈和各种 C 指针。在大多数情况下，结果是丧失可移植性和大量的分段错误。PHP 8 的实现避免了这些问题。

如果您正在处理涉及大型对象并且程序代码可能运行很长时间的程序代码，那么掌握 PHP 8 弱引用的使用是非常重要的。在深入使用细节之前，让我们先看一下类的定义。

## 审查 `WeakReference` 类的定义

`WeakReference` 类的正式定义如下：

```php
WeakReference {
    public __construct() : void
    public static create (object $object) : WeakReference
    public get() : object|null
}
```

正如您所看到的，类的定义非常简单。该类可用于提供任何对象的包装器。这个包装器使得完全销毁一个对象变得更容易，而不必担心可能会有残留的引用导致对象仍然驻留在内存中。

提示

有关弱引用的背景和性质的更多信息，请查看这里：https://wiki.php.net/rfc/weakrefs。文档参考在这里：[`www.php.net/manual/en/class.weakreference.php`](https://www.php.net/manual/en/class.weakreference.php)。

现在让我们看一个简单的例子来帮助您理解。

## 使用弱引用

这个例子演示了如何使用弱引用。您将在这个例子中看到，当通过引用进行普通对象赋值时，即使原始对象被取消设置，它仍然保留在内存中。另一方面，如果您使用 `WeakReference` 分配对象引用，一旦原始对象被取消设置，它就会完全从内存中删除。

1.  首先，我们定义了四个对象。请注意，`$obj2` 是对 `$obj1` 的普通引用，而 `$obj4` 是对 `$obj3` 的弱引用：

```php
// /repo/ch010/php8_weak_reference.php
$obj1 = new class () { public $name = 'Fred'; };
$obj2 = $obj1;  // normal reference
$obj3 = new class () { public $name = 'Fred'; };
$obj4 = WeakReference::create($obj3); // weak ref
```

1.  然后我们显示 `$obj1` 在取消设置之前和之后的 `$obj2` 的内容。由于 `$obj1` 和 `$obj2` 之间的连接是一个普通的 PHP 引用，所以由于创建了强引用，`$obj1` 仍然保留在内存中：

```php
var_dump($obj2);
unset($obj1);
var_dump($obj2);  // $obj1 still loaded in memory
```

1.  然后我们对 `$obj3` 和 `$obj4` 做同样的操作。请注意，我们需要使用 `WeakReference::get()` 来获取关联的对象。一旦取消设置了 `$obj3`，与 `$obj3` 和 `$obj4` 相关的所有信息都将从内存中删除：

```php
var_dump($obj4->get());
unset($obj3);
var_dump($obj4->get()); // both $obj3 and $obj4 are gone
```

以下是在 PHP 8 中运行此代码示例的输出：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_weak_reference.php 
object(class@anonymous)#1 (1) {
  ["name"]=>  string(4) "Fred"
}
object(class@anonymous)#1 (1) {
  ["name"]=>  string(4) "Fred"
}
object(class@anonymous)#2 (1) {
  ["name"]=>  string(4) "Fred"
}
NULL
```

输出告诉我们一个有趣的故事！第二个 `var_dump()` 操作向我们展示了，即使 `$obj1` 已经取消设置，由于与 `$obj2` 创建的强引用，它仍然像僵尸一样存在。如果您正在处理昂贵的对象和复杂的应用程序代码，为了释放内存，您需要首先找到并销毁所有引用，然后才能释放内存！

另一方面，如果你真的需要内存，而不是直接进行对象赋值，在 PHP 中是自动引用的，可以使用`WeakReference::create()`方法创建引用。弱引用具有普通引用的所有功能。唯一的区别是，如果它引用的对象被销毁或超出范围，弱引用也会被自动销毁。

从输出中可以看出，最后一个`var_dump()`操作的结果是`NULL`。这告诉我们对象确实已经被销毁。当主对象取消引用时，它的所有弱引用也会自动消失。现在你已经了解了如何使用弱引用以及它们解决的潜在问题，是时候来看看一个新类`WeakMap`了。

## 使用 WeakMap

在 PHP 8 中，添加了一个新类`WeakMap`，它利用了弱引用支持。这个新类在功能上类似于`SplObjectStorage`。以下是官方的类定义：

```php
final WeakMap implements Countable,
    ArrayAccess, IteratorAggregate {
    public __construct ( )
    public count ( ) : int
    abstract public getIterator ( ) : Traversable
    public offsetExists ( object $object ) : bool
    public offsetGet ( object $object ) : mixed
    public offsetSet ( object $object , mixed $value ) :     void
    public offsetUnset ( object $object ) : void
}
```

就像`SplObjectStorage`一样，这个新类看起来像一个对象数组。因为它实现了`IteratorAggregate`，你可以使用`getIterator()`方法来访问内部迭代器。因此，这个新类不仅提供了传统的数组访问，还提供了面向对象的迭代器访问，两全其美！在深入了解如何使用`WeakMap`之前，你需要了解`SplObjectStorage`的典型用法。

## 使用 SplObjectStorage 实现容器类

`SplObjectStorage`类的一个潜在用途是将其用作**依赖注入**（**DI**）容器的基础（也称为**服务定位器**或**控制反转**容器）。DI 容器类旨在创建和保存对象实例，以便轻松检索。

在这个例子中，我们使用一个包含从`Laminas\Filter\*`类中提取的昂贵对象数组的容器类。然后我们使用容器来清理样本数据，之后我们取消过滤器数组：

1.  首先，我们基于`SplObjectStorage`定义一个容器类。（稍后，在下一节中，我们将开发另一个执行相同功能并基于`WeakMap`的容器类。）这是`UsesSplObjectStorage`类。在`__construct()`方法中，我们将配置的过滤器附加到`SplObjectStorage`实例：

```php
// /repo/src/Php7/Container/UsesSplObjectStorage.php
namespace Php7\Container;
use SplObjectStorage;
class UsesSplObjectStorage {
    public $container;
    public $default;
    public function __construct(array $config = []) {
        $this->container = new SplObjectStorage();
        if ($config) foreach ($config as $obj)
            $this->container->attach(
                $obj, get_class($obj));
        $this->default = new class () {
            public function filter($value) { 
                return $value; }};
    }
```

1.  然后，我们定义一个`get()`方法，遍历`SplObjectStorage`容器并返回找到的过滤器。如果找不到，则返回一个简单地将数据直接传递的默认类：

```php
    public function get(string $key) {
        foreach ($this->container as $idx => $obj)
            if ($obj instanceof $key) return $obj;
        return $this->default;    
    }
}
```

请注意，当使用`foreach()`循环来迭代`SplObjectStorage`实例时，我们返回*值*（`$obj`），而不是键。另一方面，如果我们使用`WeakMap`实例，我们需要返回*键*而不是值！

然后，我们定义一个调用程序，使用我们新创建的`UsesSplObjectStorage`类来包含过滤器集：

1.  首先，我们定义自动加载并使用适当的类：

```php
// /repo/ch010/php7_weak_map_problem.php
require __DIR__ . '/../src/Server/Autoload/Loader.php';
loader = new \Server\Autoload\Loader();
use Laminas\Filter\ {StringTrim, StripNewlines,
    StripTags, ToInt, Whitelist, UriNormalize};
use Php7\Container\UsesSplObjectStorage;
```

1.  接下来，我们定义一个样本数据数组：

```php
$data = [
    'name'    => '<script>bad JavaScript</script>name',
    'status'  => 'should only contain digits 9999',
    'gender'  => 'FMZ only allowed M, F or X',
    'space'   => "  leading/trailing whitespace or\n",
    'url'     => 'unlikelysource.com/about',
];
```

1.  然后，我们分配了对所有字段（`$required`）和对某些字段特定的过滤器（`$added`）：

```php
$required = [StringTrim::class, 
             StripNewlines::class, StripTags::class];
$added = ['status'  => ToInt::class,
          'gender'  => Whitelist::class,
          'url'     => UriNormalize::class ];
```

1.  之后，我们创建一个过滤器实例数组，用于填充我们的服务容器`UseSplObjectStorage`。请记住，每个过滤器类都带有很大的开销，可以被认为是一个*昂贵*的对象：

```php
$filters = [
    new StringTrim(),
    new StripNewlines(),
    new StripTags(),
    new ToInt(),
    new Whitelist(['list' => ['M','F','X']]),
    new UriNormalize(['enforcedScheme' => 'https']),
];
$container = new UsesSplObjectStorage($filters);
```

1.  现在我们使用我们的容器类循环遍历数据文件，以检索过滤器实例。`filter()`方法会产生特定于该过滤器的经过清理的值：

```php
foreach ($data as $key => &$value) {
    foreach ($required as $class) {
        $value = $container->get($class)->filter($value);
    }
    if (isset($added[$key])) {
        $value = $container->get($added[$key])
                            ->filter($value);
    }
}
var_dump($data);
```

1.  最后，我们获取内存统计信息，以便比较`SplObjectStorage`和`WeakMap`的使用情况。我们还取消了`$filters`，理论上应该释放大量内存。我们运行`gc_collect_cycles()`来强制 PHP 垃圾回收过程，将释放的内存重新放入池中。

```php
$mem = memory_get_usage();
unset($filters);
gc_collect_cycles();
$end = memory_get_usage();
echo "\nMemory Before Unset: $mem\n";
echo "Memory After  Unset: $end\n";
echo 'Difference         : ' . ($end - $mem) . "\n";
echo 'Peak Memory Usage : ' . memory_get_peak_usage();
```

这是在 PHP 8 中运行的调用程序的结果：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php7_weak_map_problem.php 
array(5) {
  ["name"]=>  string(18) "bad JavaScriptname"
  ["status"]=>  int(0)
  ["gender"]=>  NULL
  ["space"]=>  string(30) "leading/trailing whitespace or"
  ["url"]=>  &string(32) "https://unlikelysource.com/about"
}
Memory Before Unset: 518936
Memory After  Unset: 518672
Difference          :    264
Peak Memory Usage  : 780168
```

从输出中可以看出，我们的容器类完美地工作，让我们可以访问存储的任何过滤器类。另一个有趣的地方是，在执行`unset($filters)`命令后释放的内存是`264`字节：并不多！

现在你已经了解了`SplObjectStorage`类的典型用法。现在让我们来看看`SplObjectStorage`类可能存在的问题，以及`WeakMap`是如何解决的。

## 了解 WeakMap 相对于 SplObjectStorage 的优势

`SplObjectStorage`的主要问题是，当分配的对象被取消分配或者超出范围时，它仍然保留在内存中。原因是当对象附加到`SplObjectStorage`实例时，是通过引用进行的。

如果你只处理少量对象，可能不会遇到严重的问题。如果你使用`SplObjectStorage`并为存储分配大量昂贵的对象，这可能最终会导致长时间运行的程序内存泄漏。另一方面，如果你使用`WeakMap`实例进行存储，垃圾回收可以移除对象，从而释放内存。当你开始将`WeakMap`实例整合到你的常规编程实践中时，你会得到更高效的代码，占用更少的内存。

提示

有关`WeakMap`的更多信息，请查看原始 RFC：https://wiki.php.net/rfc/weak_maps。还请查看文档：https://www.php.net/weakMap。

现在让我们重新编写前一节的示例(`/repo/ch010/php7_weak_map_problem.php`)，但这次使用`WeakMap`：

1.  如前面的代码示例所述，我们定义了一个名为`UsesWeakMap`的容器类，其中包含我们昂贵的过滤器类。这个类和前一节中显示的类的主要区别在于`UsesWeakMap`使用`WeakMap`而不是`SplObjectStorage`进行存储。以下是类设置和`__construct()`方法：

```php
// /repo/src/Php7/Container/UsesWeakMap.php
namespace Php8\Container;
use WeakMap;
class UsesWeakMap {
    public $container;
    public $default;
    public function __construct(array $config = []) {
        $this->container = new WeakMap();
        if ($config)
            foreach ($config as $obj)
                $this->container->offsetSet(
                    $obj, get_class($obj));
        $this->default = new class () {
            public function filter($value) { 
                return $value; }};
    }
```

1.  两个类之间的另一个区别是`WeakMap`实现了`IteratorAggregate`。然而，这仍然允许我们在`get()`方法中使用简单的`foreach()`循环：

```php
    public function get(string $key) {
        foreach ($this->container as $idx => $obj)
            if ($idx instanceof $key) return $idx;
        return $this->default;
    }
}
```

请注意，当使用`foreach()`循环来迭代`WeakMap`实例时，我们返回的是*键*(`$idx`)，而不是值！

1.  然后，我们定义一个调用程序，调用自动加载程序并使用适当的过滤器类。这个调用程序和上一节的程序最大的区别在于我们使用基于`WeakMap`的新容器类：

```php
// /repo/ch010/php8_weak_map_problem.php
require __DIR__ . '/../src/Server/Autoload/Loader.php';
$loader = new \Server\Autoload\Loader();
use Laminas\Filter\ {StringTrim, StripNewlines,
    StripTags, ToInt, Whitelist, UriNormalize};
use Php8\Container\UsesWeakMap;
```

1.  与前一个示例一样，我们定义了一个样本数据数组并分配过滤器。这段代码没有显示，因为它与前一个示例的*步骤 2*和*3*相同。

1.  然后，我们在一个数组中创建过滤器实例，该数组作为参数传递给我们的新容器类。我们使用过滤器数组作为参数来创建容器类实例：

```php
$filters = [
    new StringTrim(),
    new StripNewlines(),
    new StripTags(),
    new ToInt(),
    new Whitelist(['list' => ['M','F','X']]),
    new UriNormalize(['enforcedScheme' => 'https']),
];
$container = new UsesWeakMap($filters);
```

1.  最后，就像前一个示例中的*步骤 6*一样，我们循环遍历数据并应用容器类中的过滤器。我们还收集并显示内存统计信息。

这是在 PHP 8 中运行的输出，使用`WeakMap`进行修订的程序：

```php
root@php8_tips_php8 [ /repo/ch10 ]# 
php php8_weak_map_problem.php 
array(5) {
  ["name"]=>  string(18) "bad JavaScriptname"
  ["status"]=>  int(0)
  ["gender"]=>  NULL
  ["space"]=>  string(30) "leading/trailing whitespace or"
  ["url"]=>  &string(32) "https://unlikelysource.com/about"
}
Memory Before Unset: 518712
Memory After  Unset: 517912
Difference          :    800
Peak Memory Usage  : 779944
```

正如你所期望的，总体内存使用略低。然而，最大的区别在于取消分配`$filters`后的内存差异。在前一个示例中，差异是`264`字节。而在这个示例中，使用`WeakMap`产生了`800`字节的差异。这意味着使用`WeakMap`有可能释放的内存量是使用`SplObjectStorage`的三倍以上！

这结束了我们对弱引用和弱映射的讨论。现在你可以编写更高效、占用更少内存的代码了。存储的对象越大，节省的内存就越多。

# 总结

在本章中，您不仅了解了新的 JIT 编译器的工作原理，还了解了传统的 PHP 解释-编译-执行循环。使用 PHP 8 并启用 JIT 编译器有可能将您的 PHP 应用程序加速三倍以上。

在下一节中，您将了解什么是稳定排序，以及 PHP 8 如何实现这一重要技术。通过掌握稳定排序，您的代码将以一种理性的方式产生数据，从而带来更大的客户满意度。

接下来的部分介绍了一种可以通过利用`SplFixedArray`类大大提高性能并减少内存消耗的技术。之后，您还了解了 PHP 8 对弱引用的支持以及新的`WeakMap`类。使用本章涵盖的技术将使您的应用程序执行速度更快，运行更高效，并且使用更少的内存。

在下一章中，您将学习如何成功迁移到 PHP 8。
