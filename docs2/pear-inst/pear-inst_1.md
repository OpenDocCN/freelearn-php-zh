# 第一章。获取 PEAR：它是什么以及如何获取它？

很可能，您在 PHP 的使用过程中某个时候见过 PEAR 的缩写，无论是偶然还是安装和使用来自`pear.php.net`的包时。如果您调查过，您可能已经听说过由 PEAR 提供的流行软件，如 DB 数据库抽象包或 HTML_QuickForm 包。您可能没有意识到，PEAR 不仅仅是可以使用的包的集合。PEAR 还包含 PHP 最通用的安装程序，即 PEAR 安装程序。使用 PEAR 安装程序，您不仅可以从`pear.php.net`安装包，还可以从其他 PEAR 频道安装包，使用您自己的 PEAR 频道分发自己的软件项目，甚至使用 PEAR 安装程序维护复杂的内部网络 Web 项目。惊讶吗？请继续阅读，这本书将揭示 PEAR 安装程序的私密秘密以及它将如何通过 PHP 编程语言彻底改变您的日常开发。

PEAR 的主要目的是支持代码重用。**PEAR**代表**PHP 扩展和应用仓库**。PEAR 提供了一个高级安装程序和一个代码仓库，位于[`pear.php.net`](http://pear.php.net)。与您可能熟悉的竞争性 PHP 代码仓库，如[`www.phpclasses.org`](http://www.phpclasses.org)或通用开发网站[`www.sourceforge.net`](http://www.sourceforge.net)不同，所有 PEAR 代码都组织成离散的可重用组件，称为包。一个**包**由一组文件和一个名为`package.xml`的描述文件组成，该文件包含有关包内容的元数据，例如包版本、任何特殊依赖项以及包描述和作者等文本信息。

虽然大多数包包含 PHP 代码，但包的内容没有特殊限制。一些包，如[`pear.php.net/HTML_AJAX`](http://pear.php.net/HTML_AJAX)，除了 PHP 文件外还提供 JavaScript 文件。在第四章中提到的示例包仅包含 MP3 音乐文件。实际上，您可以将其保存为文件的任何内容都可以在 PEAR 包中分发。

将包从惰性文件组合转换为动态软件包的软件称为**PEAR 安装程序**，它本身也是一个 PEAR 包，位于[`pear.php.net/PEAR`](http://pear.php.net/PEAR)。换句话说，PEAR 安装程序可以用来升级自己。它确实是一个非常强大的应用程序。

传统上，PHP 软件的发布采用被动安装方法，遵循以下典型步骤：

1.  下载包含应用程序所有文件的`.zip`或`.tar.gz`文件

1.  将文件解压缩到您网站文档根目录下的一个文件夹中

1.  阅读 README 和安装文件

1.  执行各种安装后任务，创建文件，检查需求

1.  测试它

1.  通常，需要在系统级别进行更改（向 `php.ini` 添加扩展，更改 `php.ini` 设置，升级 PHP 本身）

由于没有更好的名字，我们将这个 PHP 软件分发系统称为“*解压并运行*”系统。尽管实际上它对小型、单开发者、低流量网站非常有效，但它包含了一个不立即明显的隐藏成本。关于解压并运行软件安装系统的一个单一事实限制了其最终的有用性：

### 小贴士

升级解压并运行安装非常困难

在当今快节奏的开发世界中，互联网的一个弱点是安全性。相当常见的是，在软件中发现严重的安全漏洞，需要立即升级以修复。当使用完整的解压并运行软件应用程序时，升级涉及很大的风险。首先，一旦升级完成，如果软件损坏，撤销升级需要从备份中恢复或重新安装旧软件。使用 PEAR 安装程序撤销到早期包版本是一条命令，并且非常直接。

### 注意

**如果代码正在工作，为什么升级是必要的呢？**

就在撰写本章的前一个月，我们的托管提供商的网站遭到了入侵。由于一系列不幸的事件，我完全被锁在我们的服务器上，时间过长，我们因延迟收到重要邮件而丢失了业务。

这种妥协的原因是——一个无辜地安装了过时的 CVS 查看程序副本。该程序包含了一个任意的 PHP 执行漏洞，系统管理员没有升级到最新版本，因为升级查看软件非常困难。

如果这个相同的软件以 PEAR 包的形式分发，升级将只需一条命令，如下所示：

```php
$ pear upgrade PackageName 

```

那时丢失的业务永远不会成为问题。在当今世界，升级软件实际上对任何网站（无论大小）的长期成功至关重要。

使用 PEAR 安装程序而不是简单的解压并运行解决方案的优势，在项目复杂性增加时最为明显。让我们看看以下的一些优势：

+   文件冲突是不可能的

+   由不兼容的 PHP 版本/PHP 扩展/PHP 代码引起的问题都由高级依赖关系解析处理

+   由于 PEAR 版本 1.4.0 中引入的革命性新特性 PEAR 通道功能，在不同站点之间分发应用程序开发变得简单（第五章 Chapter 5 专门探讨了 PEAR 通道）

+   所有安装配置都可以以标准化和一致的方式处理所有包——一旦你学会了如何处理一个 PEAR 包；其他所有包都以相同的方式处理。

+   代码的版本控制允许清晰的故障排除，以及回滚破坏代码的更改的能力。

在使用 PEAR 安装程序之前，了解使用 PEAR 安装程序而不是解压并运行的缺点是很重要的：

+   PEAR 安装程序本身必须在开发机器上安装，最好在服务器上安装（尽管由于 1.3.3.1 节中讨论的 PEAR_RemoteInstaller 包，这不再是必需的，使用 PEAR_RemoteInstaller 可以在没有 shell 访问的服务器上进行同步）。

+   如果你在分发自己的包，你需要完全理解 `package.xml` 描述文件，并且可能需要了解 PEAR 通道以便自己设置一个。

+   依赖于相对文件位置的传统方法并不总是可行。这是由于 PEAR 配置的灵活性所致。而不是依赖于 `dirname(__FILE__)` 这样的 PEAR 特定方式，必须使用文件替换任务（在第二章中讨论），例如。

+   在 `php.ini` 之外可能还需要在 `pear.conf/pear.ini` 中进行额外的配置（大部分配置是在安装 PEAR 安装程序时处理的）。

使用 PEAR 的传统最大障碍是安装 PEAR 安装程序本身所需的努力，这也是最近改进安装程序安装过程的重点。PHP 5.1.0 或更新的版本带来的创新可能性很大，正如 PHP_Archive PEAR 包（[`pear.php.net/PHP_Archive`](http://pear.php.net/PHP_Archive)）及其兄弟 phar PECL 扩展（[`pecl.php.net/phar`](http://pecl.php.net/phar)）所证明的那样。这些包使得将应用程序分发给单个文件成为可能，这大大增强了安装程序的功能。

# PHP 的民主创新：PEAR 通道

PEAR 安装程序最重要的创新是 PEAR 通道，这是其他任何包分发机制所不支持的东西。PEAR 通道是一种简单的方法，可以轻松地从多个来源通过互联网安装包。

通过使用 PEAR 通道，应用程序可以可靠地依赖于来自多个无关来源的代码。一些更突出的通道包括：

+   `pear.php.net:` PEAR 本身是一个通道

+   `pecl.php.net:` PECL 是 PHP 扩展的 PEAR，就像 PEAR 是常规 PHP 包一样

+   `gnope.org:` PHP-GTK2 通道

+   `pear.chiaraquartet.net:` `php.net` 域之外的第一个 PEAR 通道

+   `components.ez.no:` eZ 组件 PEAR 通道

+   `pearified.com:` PEAR 打包的 Smarty、phpMyAdmin 等的来源

同样值得关注的还有通道聚合器，如：

+   [`www.pearadise.com:`](http://www.pearadise.com) 托马斯·施利特的 PEAR 通道聚合器

+   [`www.pearified.com:`](http://www.pearified.com) pearified 通道的通道聚合部分

+   [`www.upear.com:`](http://www.upear.com) 另一个聚合器

每个 PEAR 频道都分发适合广泛需求的代码。

# 什么是 PEAR？是一个代码仓库还是一个安装程序？

PEAR 容易让人混淆，因为这个名字既可以指代 PEAR 安装程序，也可以指代位于`http://pear.php.net`的 PEAR 仓库。这种混淆自然源于`pear.php.net`首页上写着**“PEAR 是一个用于可重用 PHP 组件的框架和分发系统”**，但 PEAR 安装程序的包名却是“PEAR”。实际上，PEAR 既是代码仓库也是安装程序；由于 PEAR 的大部分优势在于能够通过互联网从远程服务器上的包更新本地安装，这两个功能通过不可分割的联系紧密相连。

## PEAR 包仓库和 PEAR 频道

位于`pear.php.net`的 PEAR 包仓库是官方的 PEAR 频道，也是最早的 PEAR 频道，比 PEAR 频道的概念早五年。许多流行的包都是通过`pear.php.net`分发的，例如**HTML_QuickForm, DB_DataObject, MDB2**和**PhpDocumentor**。大多数时候，当人们抽象地提到“PEAR”时，他们谈论的通常是`pear.php.net`上分发的某个包。

`pear.php.net`上的包仓库是由 Stig Bakken 和其他一些人于 1999 年建立的，旨在帮助填补 PHP 的空白。PEAR 的第一个可用版本与大约 PHP 版本 4.0.6 同时发布。当时，PEAR 被设计为一个框架，提供了一些基础类和一个用于管理它们的安装程序。PEAR 包提供了两个基础类，`PEAR`和`System`。位于`PEAR.php`文件中的 PEAR 类被设计用来提供基本的错误处理和一些 PHP 内部`trigger_error()`错误处理之外的其他附加功能。

几个基本包被设计来处理常见任务，例如 DB 用于数据库无关的代码，**HTML_Form**用于**HTML 表单，Log**用于基本日志记录，这个列表（长话短说）一直扩展到今天，在撰写本章时，`pear.php.net`上有 374 个包可用，还有更多通过待定提案正在路上。

自从 PEAR 开始时充满希望以来，它也经历了一些困难。PHP 已经发展，并引入了几个超越 PEAR 提供功能的功能。例如，PHP 5.0.0 中引入的`Exception`类提供了与`PEAR_Error 类`类似的功能，PDO 扩展提供了与 DB 包类似的功能，等等。此外，随着 PEAR 的成长，其作为仓库的初始设计中的一些问题浮出水面，导致了小到大的危机。

在这些成长的痛苦中，产生了许多好的东西，包括用于提出新软件包的有效的 PEPr 提案系统，以及向稳定性和创新的双重推动。推动创新的大部分力量来自外部压力，这是由于来自 SolarPHP、eZ components、Phing、Propel、Agavi 和 Zend Framework 等项目的新想法的出现。然而，在 PEAR 中推动创新的显著力量之一就是 PEAR 安装程序本身，特别是新版本 1.4.0 及更高版本中的新特性。

在 PEAR 安装程序的新版本中，一个关键的新特性是 PEAR 通道。**PEAR 通道**简单来说就是一个提供类似 `pear.php.net` 这样的软件包的服务器，同时也提供了特殊的网络服务，使得 PEAR 安装程序能够与服务器通信；通道的详细内容在第四章中有介绍。

## PEAR 安装程序

大多数讨论 PEAR 的书籍都花费了大量时间在介绍如何使用 PEAR 存储库中的流行软件包（[`pear.php.net`](http://pear.php.net)）上，例如 DB（[`pear.php.net/DB`](http://pear.php.net/DB)）或 HTML_QuickForm（[`pear.php.net/HTML_QuickForm`](http://pear.php.net/HTML_QuickForm)）。本书则采取了不同的路线，专注于 PEAR 安装程序本身安装机制周围的强大功能。当你在本书中看到对 *PEAR* 的引用时，你应该知道这实际上是对 PEAR 安装程序（[`pear.php.net/PEAR`](http://pear.php.net/PEAR)）的引用，而不是对整个存储库的引用。

PEAR 安装程序由四个面向任务的抽象部分组成：

+   `package.xml` 解析器和依赖处理程序

+   文件安装处理器、配置处理器和软件包注册器

+   用户前端和命令处理器

+   远程服务器同步和下载引擎

幸运的是，作为一个最终用户，你实际上并不需要了解或关心这些内容，除了你的应用程序如何与安装程序通信以及人们如何实际使用安装程序来安装你的代码。

首先，你需要了解 PEAR 实际上是如何安装它的包的，因为这与旧的解压缩并运行哲学略有不同。PEAR 将文件分类到不同的类型，并以不同的方式安装每种类型的文件。例如，php 类型的文件将被安装到用户的配置变量`php_dir`的子目录中。所以如果用户将`php_dir`设置为`/usr/local/lib/php/pear`，那么打包在`Foo/Bar/Test.php`目录中的文件将被安装到`/usr/local/lib/php/pear/Foo/Bar/Test.php`。数据类型的文件将被安装到用户的配置变量`data_dir`的子目录中，但与 PHP 文件不同，数据文件将根据包名安装到它们自己的私有子目录中。如果用户将`data_dir`设置为`/usr/local/lib/php/data`，并且包名为“TestPackage”，那么打包在`Foo/Bar/Test.dat`目录中的文件将不会安装到`/usr/local/lib/php/data/Foo/Bar/Test.dat`，而是安装到`/usr/local/lib/php/data/TestPackage/Foo/Bar/Test.dat`！关于 PEAR 安装的其他细节，请参阅第二章。

接下来，你需要了解一些关于 PEAR 如何知道一个包中包含哪些文件，它需要什么才能工作（即“这个包仅在 PHP 4.3.0 及更高版本中工作，需要任何版本的 DB_DataObject 包，以及仅 1.2.3 版本的 Log 包，建议安装 1.4.6 版本”），或者它的依赖关系。这些信息也在第二章中有介绍，我们学习了`package.xml`文件格式。

在阅读了第二章和第三章之后，你应该对 PEAR 有足够的了解，以便管理自己包的本地安装，或者向官方`pear.php.net`仓库提出新的包建议。如果你希望将你的包分发给他人使用，无论是免费、为私人客户还是销售，你需要了解 PEAR 通道是如何工作的。设置通道服务器是一个相对简单的任务。在撰写本章时，有几种设置通道的选项，但几种潜在的竞争选项即将完成。这是一个参与 PHP 开发令人兴奋的时期！关于如何设置自己的通道，请参阅第五章。

当然，并不是每个人都公开分发他们的代码。大多数开发者都在忙于自己或团队设计自己的网站。当与版本控制系统结合使用时，PEAR 安装程序是管理完整网站的一个非常有效的工具。第四章 详细介绍了这个主题。

简而言之，PEAR 安装程序是管理高质量软件库、高质量应用程序或高质量网站的最有效的工具之一。

# 安装 PEAR 安装程序

获取 PEAR 有三种方法。第一种方法自 PHP 4.2.0 以来就可用，只需安装 PHP 并配置 `--with-pear`（在 Unix 上）或从 PHP 目录中运行 `go-pear` 即可。

### 小贴士

**我正在运行 Windows，我的 PHP 中没有 PEAR，它在哪？**

如果你正在运行 PHP 版本 5.1 或更早版本，为了获取捆绑了 PEAR 的 PHP 版本，你需要下载 `.zip` 文件而不是 Windows 安装程序（`.msi`）。PHP 5.2.0 及更高版本显著改进了 Windows 安装程序（`.msi`）的发行版，并且推荐使用它来获取 PEAR。安装程序（至少到目前为止）实际上并不太有用，因为它除非出现关键错误否则永远不会更新。因此，获取 PHP 的真正有用元素需要使用 `.zip` 文件。不必担心，PHP 在 Windows 上基本上是一个解压即用的语言。

如果我不那么外交，我可能会说些像“甚至不要考虑在 Windows 上运行 PHP 作为生产服务器”这样的话。然而，我认为重要的是要说：“在我看来，在 Windows 上运行 PHP 作为生产服务器既不值得努力也不值得花费。”尽管可以做到，但唯一应该考虑的理由是如果你的老板会在你安装 Unix 时解雇你。或者你为维护 Windows 操作系统的公司工作。在这种情况下，你会得到原谅。

第二种方法是从 [`pear.php.net/go-pear`](http://pear.php.net/go-pear) 获取 go-pear 脚本，并将其保存为 `go-pear.php`，然后运行此文件。最后，还有一些非官方的 PEAR 获取来源，最著名的是 Gnope 安装程序，它一步到位地设置了 PHP 5.1、PEAR 和 PHP-GTK2。

## PHP 捆绑的 PEAR

如上所述，如果你正在运行 Windows，安装 PEAR 相对简单。运行 `go-pear` 命令将运行安装脚本。此外，自 PHP 版本 5.2.0 以来，有一个基于 Windows 安装程序（`.msi` 文件扩展名）的优秀安装机制，它在其发行版中捆绑了 PEAR，并推荐使用。对于更早的 PHP 版本，为了获取捆绑的 PEAR，你必须下载 `.zip` 文件发行版，而不是基于 `.msi` 的 Windows 安装程序机制。

如果你正在运行 Unix，你需要使用至少以下最小配置行来设置 PEAR：

```php
$ ./configure --enable-cli --enable-tokenizer --with-pcre-regex -- enable-xml 

```

为了充分利用 PEAR，启用`zlib`扩展以访问压缩的 PEAR 包是个好主意：

```php
$ ./configure --enable-cli --enable-tokenizer --with-pcre-regex -- enable-xml --with-zlib 

```

在安装时，请确保您对 PEAR 将要安装的目录有完整的写访问权限（通常是`/usr/local/lib`，但可以通过`--prefix`选项进行配置；`/lib`附加到前缀）。当您安装 PEAR 时，它也会安装所需的包`Console_Getopt`和`Archive_Tar`。

```php
$ make install-pear
Installing PEAR Environment: /usr/local/lib/php/
[PEAR] Archive_Tar - installed : 1.3.1
[PEAR] Console_Getopt installed : 1.2
pear/PEAR can optionally use package "pear/XML_RPC" (version >= 1.4.0)
[PEAR] PEAR - installed : 1.4.9
Wrote PEAR system config file at: /usr/local/etc/pear.conf
You may want to add: /usr/local/lib/php to your php.ini include_path 

```

完成 PEAR 安装后，如安装说明的最后一行所建议的，您将想要确保 PEAR 位于`php.ini`的`include_path`设置中。确定这一点最简单的方法是首先找到`php.ini`：

```php
$ php i |grep php[.]ini
Configuration File (php.ini) Path => /usr/local/lib 

```

这个例子显示`php.ini`文件位于`/usr/local/lib/php.ini`。使用您喜欢的编辑器打开此文件，然后找到说"`include_path=`"的行，或者添加一个类似这样的新行：

```php
include_path=.:/usr/local/lib/php

```

当然，用您特定安装的推荐路径（输出最后一行）替换`/usr/local/lib/php`。

### 小贴士

**这个所谓的“include_path”是什么意思？**

哎呀——是时候再次查阅 PHP 手册了，具体来说，[`www.php.net/include`](http://www.php.net/include)。总结一下，`include`语句及其类似项`include_once, require`和`require_once`都用于动态包含外部 PHP 代码。如果您传入一个完整路径，例如`/path/to/my.php:`：

```php
include '/path/to/PEAR.php'; 

```

然后，自然地`/path/to/my.php`将*替换*掉包含语句并执行。然而，如果您传入一个相对路径，例如：

```php
include 'PEAR.php'; 

```

这将搜索`include_path`并尝试找到`PEAR.php`文件。如果`include_path`是`.:/usr/local/lib/php`，那么 PHP 将首先尝试在当前目录（`.`）中找到`PEAR.php`，如果没有找到，它将在`/usr/local/lib/php/PEAR.php`中搜索。这允许在磁盘上自定义库位置，这正是 PEAR 试图实现的目标。

如果您正在运行 Windows，`go-pear`批处理文件将提示您选择安装 PEAR 的位置，默认为版本 4 的`C:\php4`，版本 5.0 的`C:\php5`，或版本 5.1 或更高版本的当前目录。除非您有充分的理由在其他地方安装 PEAR，否则接受默认位置是个好主意，因为它可以节省一些潜在的问题。如果您从版本 5.1.0 或更高版本安装 PEAR，安装程序将询问您是安装本地还是系统安装。

```php
Are you installing a system-wide PEAR or a local copy?
(system|local) [system] : 

```

直接按*回车*安装系统范围内的 PEAR（一般来说，在安装 PEAR 时，如果您不知道它要求什么，总是选择默认值）。接下来，它将询问您安装 PEAR 副本的位置：

```php
Below is a suggested file layout for your new PEAR installation. To
change individual locations, type the number in front of the
directory. Type 'all' to change all of them or simply press Enter to
accept these locations.
1\. Installation base ($prefix) : C:\php51
2\. Binaries directory : C:\php51
3\. PHP code directory ($php_dir) : C:\php51\pear
4\. Documentation directory : C:\php51\pear\docs
5\. Data directory : C:\php51\pear\data
6\. Tests directory : C:\php51\pear\tests
7\. Name of configuration file : C:\php5\pear.ini
8\. Path to CLI php.exe : C:\php51\.
1-8, 'all' or Enter to continue: 

```

通过输入项目前面的数字并按*Enter*键来更改您希望更改的任何值。如果您想更改安装基本目录，请输入**1**并按*Enter*。如果您想修改所有值，请输入**all**并按*Enter*。一旦您满意，请按*Enter*键而不输入任何内容以继续。此时，它将安装包（应该看起来与 Unix 中 PEAR 的安装类似），最后询问是否要修改`php.ini`。如果您选择这样做，它将自动更新`php.ini`中的`include_path`设置，然后您就可以正常使用了。

## 5.1.0 及更早版本 PHP 的安装

如果您有 5.1.0 及更早版本的 PHP，并希望安装 PEAR，您不应使用捆绑的 PEAR 版本，而应通过`go-pear`脚本安装 PEAR。要检索此脚本，请使用任何网页浏览器浏览到[`pear.php.net/go-pear`](http://pear.php.net/go-pear)，并将其保存到磁盘上为`go-pear.php`，或者在 Unix 上，您有`wget`命令：

```php
$ wget -O go-pear.php http://pear.php.net/go-pear 

```

一旦您可以访问`go-pear.php`，只需运行它：

```php
$ php go-pear.php 

```

### 小贴士

**为什么只针对 PHP 5.1.0 及更早版本？**

就像有人曾经问过我，“为什么只使用`go-pear`脚本来安装 5.1.0 及更早版本的 PHP 很重要？当 PHP 6.0.0 发布带有新 PEAR 安装程序的版本时，这不会随时间改变吗？”

这种情况的原因与`go-pear`的工作方式有关。`go-pear`脚本是一个出色的设计，它可以直接从 CVS 中提取基于其发布标签的关键文件，然后根据其最新稳定版本下载包并安装它们。虽然很出色，但这是一种危险的方法，如果 CVS 中出现问题，则保证你的安装会中断。

PHP 版本 5.1.0 及更高版本捆绑的 PEAR 安装程序实际上是自包含的，根本不接触互联网，并且经过测试和保证与该版本的 PHP 兼容。一旦安装了 PEAR，获取最新版本既简单又比使用`go-pear`更安全：

```php
pear upgrade PEAR 

```

输出将与捆绑 PHP 中的`go-pear`非常相似，但将开始于：

```php
$ php go-pear.php
Welcome to go-pear!
Go-pear will install the 'pear' command and all the files needed by
it. This command is your tool for PEAR installation and maintenance.
Use 'php go-pear.php local' to install a local copy of PEAR.
Go-pear also lets you download and install the PEAR packages bundled
with PHP: DB, Net_Socket, Net_SMTP, Mail, XML_Parser, PHPUnit.
If you wish to abort, press Control-C now, or press Enter to continue: 

```

由于`go-pear`需要互联网访问，下一个问题将是代理。如果您在防火墙后面，请确保您知道是否需要使用代理：

```php
HTTP proxy (http://user:password@proxy.myhost.com:port), or Enter for none:: 

```

安装过程的下一步应该看起来相当熟悉：

```php
Below is a suggested file layout for your new PEAR installation. To change individual locations, type the number in front of the directory. Type 'all' to change all of them or simply press Enter to accept these locations.
1\. Installation prefix : C:\php51
2\. Binaries directory : $prefix
3\. PHP code directory ($php_dir) : $prefix\pear
4\. Documentation base directory : $php_dir\docs
5\. Data base directory : $php_dir\data
6\. Tests base directory : $php_dir\tests
7\. Temporary files directory :
8\. php.exe path : C:\php51\php.exe
1-8, 'all' or Enter to continue: 

```

选择位置后，`go-pear`将下载所需的代码，并安装 PEAR 包。它还会询问是否要安装一些额外的包，最后是否要修改`php.ini`。

如果您喜欢冒险，您也可以简单地尝试在网页浏览器中打开`go-pear.php`。这将安装 PEAR 的 Web 前端，这非常好，但不如**命令行安装程序**（**CLI**）活跃。

## 其他非官方来源

在我们继续之前，重要的是要注意本节中的所有内容都是非官方的，因此不会得到[pear.php.net](http://pear.php.net)那边的人的支持。你遇到的所有问题都是你下载它的网站的责任！好，不再多说了。

在 [`www.gnope.org`](http://www.gnope.org) 的 Christian Weiske 一直在努力使 PHP-GTK2 在 PHP 5.1 中工作，并使使用 PEAR 安装 PHP-GTK2 应用程序变得容易。因此，他与我们紧密合作，创建了一个 PEAR 安装程序的 GTK2 前端，现在可以通过 PEAR 在 [`pear.php.net/PEAR_Frontend_Gtk2`](http://pear.php.net/PEAR_Frontend_Gtk2) 获取。这是 PEAR 的官方 PHP-GTK2 前端。前端要求 PEAR 已经安装，才能使用它。

然而，Christian 已经将事情提升到了一个新的层次，他编写了一个 Windows 安装程序，提供了最新的稳定版 PHP 5.1、PEAR 和 PHP-GTK2，为你设置了一切。如果你是这个过程的新手，这是一个开始使用 PEAR 的好方法。当前版本是 1.0，但这是一个 alpha 版本，所以可能不会完全正常工作。

首先，你应该注意的是，如果你已经设置了 PHP，Windows 中的 PATH 环境变量可能会与现有的 PHP 安装产生一些冲突。简而言之，除非你知道自己在做什么，否则不要尝试同时安装一个常规的 PHP 复制和 gnope PHP。

安装 gnope 非常简单，只需下载安装程序并运行即可。提示与任何其他 Windows 安装一样。

# 使用 PEAR_RemoteInstaller 同步到无 Shell 访问的服务器

一旦你在开发机器上成功安装了 PEAR，你如何在实时服务器上正确安装 PEAR？对于许多开发者来说，这只是在实时服务器上重复在开发服务器上执行的同一步骤。然而，许多开发者不幸地使用了不提供 shell 但提供 PHP 的共享主机提供商。在过去，这基本上消除了成功安装 PEAR 包的任何可能性。

截至 PEAR 1.4.0，这个问题已经不再存在，多亏了 PEAR_RemoteInstaller 包（[`pear.php.net/PEAR_RemoteInstaller`](http://pear.php.net/PEAR_RemoteInstaller)）。这个包提供了同步定制 PEAR 服务器和远程服务器的特殊命令。与其他解决方案不同，实现此功能所需的唯一工具是开发机器上的 PHP 5 和对 FTP 服务器的访问。

要开始，请将 PEAR_RemoteInstaller 安装到你的开发机器上：

```php
$ pear install PEAR_RemoteInstaller 

```

### 小贴士

**发布限制**

注意，在本书编写时，PEAR_RemoteInstaller 的最新版本是 0.3.0，稳定性为 alpha。安装时，你需要追加 `alpha` 或 `-0.3.0`，例如：

```php
$ pear install PEAR_RemoteInstaller-alpha 

```

或者

```php
$ pear install PEAR_RemoteInstaller-0.3.0 

```

接下来，你需要使用 `config-create` 命令为远程机器创建一个配置文件。然而，在这之前，知道你的主机的完整路径是很重要的。确定这个路径最简单的方法是在你的网络主机上运行这个简单的脚本：

```php
<?php
echo __FILE__;
?>

```

将其保存为 `me.php`，并上传到你的网络主机。运行时，这将输出类似的内容：

```php
/home/myuser/htdocs/me.php 

```

一旦你有了这些信息，你就可以开始了

```php
$ pear config-create /home/myuser pear.ini 

```

这将显示类似的内容：

```php
CONFIGURATION (CHANNEL PEAR.PHP.NET):
=====================================
Auto-discover new Channels auto_discover <not set>
Default Channel default_channel pear.php.net
HTTP Proxy Server Address http_proxy <not set>
PEAR server [DEPRECATED] master_server <not set>
Default Channel Mirror preferred_mirror <not set>
Remote Configuration File remote_config <not set>
PEAR executables directory bin_dir /home/myuser/pear
PEAR documentation directory doc_dir /home/myuser/pear/docs
PHP extension directory ext_dir /home/myuser/pear/ext
PEAR directory php_dir /home/myuser/pear/php
PEAR Installer cache directory cache_dir /home/myuser/pear/cache
PEAR data directory data_dir /home/myuser/pear/data
PHP CLI/CGI binary php_bin C:\php5\php.exe
PEAR test directory test_dir /home/myuser/pear/tests
Cache TimeToLive cache_ttl <not set>
Preferred Package State preferred_state <not set>
Unix file mask umask <not set>
Debug Log Level verbose <not set>
PEAR password (for password <not set> maintainers)
Signature Handling Program sig_bin <not set>
Signature Key Directory sig_keydir <not set>
Signature Key Id sig_keyid <not set>
Package Signature Type sig_type <not set>
PEAR username (for username <not set> maintainers)
User Configuration File Filename C:\test\pear.ini
System Configuration File Filename #no#system#config#
Successfully created default configuration file "C:\test\pear.ini" 

```

一旦你有了 `pear.ini` 文件，将其上传到你的远程网络主机，并保存在 `/home/myuser` 目录（你的 FTP 登录的默认目录）。接下来，我们需要创建 PEAR 的本地工作副本：

```php
$ mkdir remote
$ cd remote 

```

如果你正在运行 Windows：

```php
$ pear config-create -w C:\remote\ pear.ini 

```

否则（假设你在你的开发机器上以用户 `foo` 运行）：

```php
$ pear config-create /home/foo pear.ini 

```

接下来，确定你使用的 FTP 连接类型。你需要知道访问 FTP 的用户名和密码以及主机名。在我们的例子中，我们将使用 `myuser` 和 `password` 作为用户/密码组合，以及 `yourwebsite.example.com` 作为主机。

如果你使用常规 FTP，你可能需要这样做：

```php
$ pear -c pear.ini config-set remote_config ftp://myuser:password@yourwebsite.example.com/ 

```

如果你使用 FTPS，你可能需要这样做：

```php
$ pear -c pear.ini config-set remote_config ftps://myuser:password@yourwebsite.example.com/ 

```

如果你使用 SFTP，你可能需要这样做：

```php
$ pear -c pear.ini config-set remote_config ssh2.sftp://myuser:password@yourwebsite.example.com/ 

```

为了使用 SFTP，你需要从 `pecl.php.net` ( [`pecl.php.net/ssh2`](http://pecl.php.net/ssh2)) 安装 SSH2 扩展，这是一个由 Sara Golemon 编写的优秀扩展。在 Unix 上，这很简单；SSH2 可以通过以下方式安装：

```php
$ pecl install ssh2 

```

在 Windows 上，`php_ssh2.dll` 应该与你的 PHP 5 发行版一起分发。如果没有，你可以从 [`pecl4win.php.net`](http://pecl4win.php.net) 获取一份副本。

一旦我们达到这个阶段，下一步是尝试安装一个包。让我们安装 Savant3，这是一个优秀的 PHP 5+ 模板引擎，它利用 PHP 本身的优雅性作为模板语言。

```php
$ pear -c pear.ini channel-discover savant.pearified.com
$ pear -c pear.ini remote-install savant/Savant3 

```

如果一切正常，你将看到：

**远程安装成功：savant.pearified.com/Savant3-3.0.0**

到目前为止，启动你的 FTP 浏览器，你会看到文件已上传到 `/home/myuser/pear/php/Savant3.php` 和 `/home/myuser/pear/php/Savant3` 目录。

### 小贴士

关于 alpha/beta 的错误

如果安装失败并显示有关首选状态的提示，请尝试添加 [-alpha](http://-alpha) ，例如：

```php
$ pear -c pear.ini remote-install savant/Savant3-alpha 

```

这将有效，因为它指示安装程序暂时忽略你的 `preferred_state` 配置变量，并下载最新的 alpha 或更稳定的版本。换句话说，如果首选状态为稳定，如果 Savant3 版本 3.2.0 稳定版可用，它将被安装。然而，如果版本 3.2.1 alpha 可用，一个更新的版本，它将不会安装，因为其稳定性太低。但是，通过添加 `-alpha`，安装程序被指示获取最新的版本 3.2.1 并安装该版本。

你可能会想，这与手动提取文件并手动上传有什么不同？当管理像 DB_DataObject 或 LiveUser 这样的更复杂的包时，答案就显而易见了。这两个包都有必须安装才能工作的依赖项。虽然可以手动安装并上传，但很容易忘记一个依赖项。更糟糕的是，整个过程在升级时必须重复。

使用 PEAR、Validate 和 DB 依赖项安装 DB_DataObject 非常简单：

```php
$ pear -c pear.ini remote-install o DB_DataObject 

```

同样适用于升级：

```php
$ pear -c pear.ini remote-upgrade o DB_DataObject 

```

卸载也同样简单，并且响应于 PEAR 安装程序的依赖项验证的全功能：

```php
$ pear -c pear.ini remote-uninstall DB_DataObject 

```

### 小贴士

**rsync 方法**

如果你正在为客户开发一个复杂的网站，客户有足够的预算，那么最好在具有相同软件和硬件配置的相同机器上开发。在本地设置整个网站，并使用`rsync`与远程网站同步。即使使用源控制和可安装的 PEAR 包，这也仍然工作得很好，因为开发应该与离散的包版本同步。

# 摘要

在本章中，我们学习了 PEAR 作为`pear.php.net`上的代码仓库和作为 PEAR 安装程序的双重性格。我们研究了 PHP 软件的主要分发方法——解压缩并运行，并将其与 PEAR 安装程序的基于包的分发方法进行了比较。在快速了解了令人兴奋的新创新——PEAR 通道之后，我们窥探了 PEAR 安装程序从包中安装文件的基本原理以及它知道在哪里安装它们的方法。最后，我们学习了获取 PEAR 安装程序的各种方法，甚至如何在没有提供 shell 访问权限的 Web 主机上远程安装 PEAR。
