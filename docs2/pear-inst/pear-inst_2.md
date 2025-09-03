# 第二章。使用 PEAR 安装程序精通 PHP 软件管理

在本章中，我们学习如何将 PHP 软件转换为可分发的 PEAR 软件包。2005 年 9 月，PEAR 安装程序的 1.4.0 版本发布。这是一个里程碑，标志着 PEAR 从 `pear.php.net` 分发的狭小市场工具转变为一个完整的应用程序安装工具。第一次，可以分发大规模的应用程序，甚至完整的基于网络的数据库密集型应用程序也可以一步安装和配置。

PEAR 安装程序现在可以用来安装传统的基于网络的程序，如 phpMyAdmin 和非 PEAR 库，例如流行的 Smarty 模板引擎（这两个都可以通过 [`pearified.com`](http://pearified.com) PEAR 频道）安装）。PEAR 安装程序的两个主要设计目标是：

+   使应用程序开发能够在多个开发团队之间进行分发（即停止重复造轮子）

+   防止冲突的软件包相互覆盖

所有这些魔法都是通过 `package.xml` 文件格式实现的。`package.xml` 是 PEAR 安装程序的核心和灵魂，为了充分利用 PEAR 的功能，你需要了解其结构。`package.xml` 包含要安装的文件列表，供 PEAR 安装程序区分不同软件包和版本的信息，以及对人类有用的信息，例如软件包的描述和变更日志。实际上，这个文件就是 PEAR 安装程序正确安装软件所需的一切。PEAR 安装程序还使用 `package.xml` 中的信息，通过 `pear package` 命令创建可安装的存档，格式为 `.tar` 或压缩的 `.tar`（`.tgz`）。

PEAR 安装程序不仅限于安装本地文件，实际上它是设计用来通过互联网与 **PEAR 频道服务器** 进行通信的。什么是频道服务器？频道服务器为每个软件包提供可下载的版本，并通过 XML-RPC、REST 或 SOAP 提供关于这些软件包和版本的元信息的网络服务接口。频道在 第五章 中有深入讨论。现在，随着 PEAR 1.4.0+ 和 Chiara_PEAR_Server（[`pear.chiaraquartet.net/index.php?package=Chiara_PEAR_Server`](http://pear.chiaraquartet.net/index.php?package=Chiara_PEAR_Server)）等软件包的可用，设置自己的 PEAR 频道服务器并分发库和应用程序变得非常简单，这些都可以通过 `pear` 命令来实现。

### 小贴士

**命令行界面（CLI）与 Web/Gtk2 安装程序**

一些阅读此文档的人可能已经安装了 Web 前端（来自 `pear.php.net` 的 PEAR_Frontend_Web 包）或 Gtk2 前端（来自 `pear.php.net` 的 PEAR_Frontend_Gtk2 包）。如果是这样，那么你可能已经注意到，从其他渠道安装包甚至更简单，因为最终用户只需选择要安装的渠道，所有包都会列出最新版本和可用的升级。

在接下来的几章中，我们将使用命令行（CLI）前端来处理 PEAR 安装程序，因为与 Web 前端相比，CLI 前端提供了更高级的复杂性，Web 前端设计得更多是为了 PEAR 包的最终用户，而不是 PEAR 包的开发者。在撰写本章时，Gtk2 前端比 Web 安装程序更复杂，如果你运行的是 PHP 5.1.0 或更高版本，那么它值得使用。

例如，如果你的服务器是 `pear.example.com`，并且你发布了一个名为 `Foo` 的包，那么你的用户只需要输入以下内容来安装你的包：

```php
$ pear channel-discover pear.example.com
$ pear install pear.example.com/Foo 

```

然而，PEAR 安装程序最动态和最重要的功能是它处理对其他包、PHP 版本、PHP 扩展和系统架构的依赖的复杂方式。通过 `package.xml` 中的非常简单的语法，可以轻松且安全地管理极其复杂的依赖场景。

在我们开始实际创建自己的包的工作之前，了解 PEAR 安装程序设计背后的核心概念以及你需要如何调整你的软件设计以充分利用其优势是非常重要的。

# 分发库和应用程序

最重要的是要理解 PEAR 安装程序实际上是如何安装文件的。大多数 PHP 开发者将他们的应用程序作为解压即用的存档进行分发。因此，我们倾向于假设文件将在最终用户机器上的确切相对位置与它们在我们开发机器上的位置相同。然而，PEAR 的设计要灵活得多。

例如，共享 Web 主机通常会安装一个全局的 PEAR 副本，但用户也可以安装本地副本，并使用 `php.ini` 中的 `include_path` 来选择当本地包可用时使用本地包，当不可用时使用全局包。为了使这种灵活性成为可能，PEAR 按类型或文件 *角色* 对组和安装文件。

每个文件角色都有一个相应的配置条目，它定义了所有该文件角色的文件将被安装的位置。例如，PHP 文件角色有一个名为 `php_dir` 的配置变量，它定义了所有 PHP 文件将被安装到的目录，数据文件角色有一个名为 `data_dir` 的配置变量，依此类推。

这与传统“解压即用”的哲学有着根本性的区别，因为它允许用户根据自己的需求在他们的机器上以任何方式配置安装位置。这也意味着，在 PEAR 包中使用如 `dirname(__FILE__)` 这样的巧妙构造来定位已安装文件是危险的编码方式。幸运的是，还有其他更灵活、更安全的巧妙方法可以解决这个问题。

### 小贴士

**PEAR 配置**

要检索所有配置变量及其值，请使用 `config-show` 命令。与文件角色对应的配置变量通常在其名称中包含 `_dir`，例如 `doc_dir` 和 `php_dir`。

这也稍微改变了开发过程：不再是直接在开发目录中运行代码进行测试，而是首先通过其 `package.xml` 文件安装代码，并以 PEAR 的解压即用方式测试。因此，最常用的 `pear` 命令是：

```php
$ pear up -f package.xml 

```

或者

```php
$ pear upgrade --force package.xml 

```

此命令允许从（例如）PEAR 版本 1.5.0a1 升级到 PEAR 版本 1.5.0a1，当修复了错误或添加了新功能时。实际上，此命令通过允许在更改后快速替换版本内的现有文件，从而促进了快速开发。换句话说，如果我们每次在开发机器上对微小更改进行更改时都必须更改版本号，这将是非常繁琐的。`--force` 选项绕过了这个问题。

虽然表面上使用 PEAR 管理安装可能看起来更复杂，但快速调查 *有用的* 解压即用包表明，许多解压即用包实际上需要大量的手动配置，这些配置可以很容易地被 PEAR 安装程序的自动化功能所取代。简而言之，一旦习惯了它，你就会 wonder 如何在没有使用 PEAR 安装程序来管理你的包的情况下开发 PHP。

## 从安装程序的角度来看，库和应用程序之间的区别

缩写**PEAR**代表**PHP**扩展和**应用****存****储****库**，但在 PEAR 版本 1.4.0 之前，对应用程序的支持是有限的。大多数 PEAR 包是设计成可以集成到其他外部应用程序中的库。PEAR 安装程序的初始设计非常有效地支持了这种模式，但并没有提供应用程序安装和配置所需的所有定制化功能。引入 PEAR 版本 1.4.0 的新功能的主要动机之一是更好地支持应用程序的安装。

尽管库和完整应用程序在功能上存在明显的差异，但从安装程序的角度来看，库和应用程序不需要被非常不同地处理。两者都使用`package.xml`进行分发，都以相同的方式存储在注册表中，并应用相同的版本控制和依赖性规则。实际上，这是 PEAR 安装程序最大的优势之一，因为它既遵循**KISS**（**Keep It Simple, Stupid**）原则，又让应用程序设计取决于`package.xml`的使用方式。因此，为了充分利用其功能，对`package.xml`的设计有深入理解是必要的。

新特性旨在简化应用程序开发，包括：

+   可定制的文件角色

+   可定制的文件任务

+   更高级的依赖性可能性和预下载依赖性解析

+   将多个软件包打包成一个单一的 tar 包的能力

+   基于单个发布版本而非抽象包的静态依赖性

在阅读完这个列表后，如果你的头脑发晕或者你看到类似问号的斑点，不要担心——所有这些特性将在接下来的几节中简单而详尽地探讨。

在我们深入探讨新特性之前，让我们更仔细地看看一些基本原理，这些原理为最佳使用新特性提供了基础。

# 使用版本控制和依赖性帮助跟踪和消除错误

版本控制和依赖性是每个企业级分发系统必须非常支持的两大特性。有了简单的依赖性和高级版本控制功能，PEAR 安装程序使得依赖外部包比以往任何时候都更加安全和容易。

## 版本控制

PEAR 安装程序最基本的基础是版本控制的概念。版本控制应该对我们所有人都很熟悉，以“软件包版本 X.Y.Z”的形式，例如“PHP 版本 5.1.4”。基本思想是，软件的旧版本号较低。换句话说，PHP 版本 4.0.6 比 PHP 版本 4.1.1beta1 旧，而 PHP 版本 4.1.1beta1 又比 PHP 版本 4.1.1 旧。

版本控制如何帮助跟踪和消除错误？想象以下场景：

你正在处理一个 Wiki，允许用户随时从你的 FTP 站点获取源代码并自行使用。其中一位用户发现了一个错误并报告说：“它正在做一件奇怪的事情，试图删除我所有的文件。”用户无法记得他或她何时下载了源代码，因为他或她不得不从备份中恢复，并且文件修改时间已被重置。在这种情况下，唯一确定问题是否存在以及它是否仍然存在于当前源代码中的方法是从用户项目获取并逐行与当前源代码进行比较。在最坏的情况下，这可能是繁琐的，在最坏的情况下是完全不可能的。

对于一个流动的、不断变化的软件项目，在特定时间发布版本并分配版本号，使得最终用户报告错误变得更加简单。最终用户可以简单地说明“您的软件包的 1.2.3 版本做了这样奇怪的事情，试图删除我所有的文件”，然后您作为开发者可以要求用户尝试最新版本或当前的开发副本，这使得错误修复变得更加简单。

此外，还可能维护同一代码的两个分支，一个是稳定版本，另一个是不稳定版本，具有创新的新功能。有许多微妙的方法可以使用版本管理来提供有关软件发布的更多信息。例如，Linux 内核版本管理系统在 [`en.wikipedia.org/wiki/Linux_kernel#Version_Numbering`](http://en.wikipedia.org/wiki/Linux_kernel#Version_Numbering) 中有详细描述。在这种情况下，小数点之一用于表示内核的稳定性，因此可以在 1.3.0 之后发布版本 1.2.9。

PEAR 安装程序在版本管理上采取了一种更明确的方法。它不是在版本号内部提供稳定性信息，而是在 `package.xml` 中的单独字段 `<stability>` 中使用来指定代码的稳定性。这并不妨碍使用 Linux 风格的版本管理或任何其他版本管理方案，只要基本前提（1.3.0 总是比 1.2.9 新）仍然成立。

### 小贴士

**版本时间与绝对时间**

在现实世界中，PEAR 的简单版本管理规则实际上可能会有些令人困惑。PEAR 安装程序并不特别关心版本在现实世界中的发布时间，而是关注其稳定性和版本号。

这意味着，例如，2005 年 8 月 18 日发布的 PEAR 1.3.6（稳定版）实际上比 2005 年 2 月 26 日发布的 PEAR 1.4.0a1（alpha 版）更老，因为 1.3.6 小于 1.4.0。绝对时间与版本时间无关。

最终用户可以通过一个名为 `preferred_state` 的配置变量来告诉 PEAR 安装程序，为了安装，必须保证软件包的稳定性。在我们的假设例子中，如果这个变量设置为 `stable` 或 `beta`，那么将安装 PEAR 1.3.6 而不是 1.4.0a1；否则，对于比 beta 版本稳定性低的值（`alpha`、`devel` 和 `snapshot`），将安装 1.4.0a1，尽管 1.3.6 版本是在几个月后发布的，因为在版本时间上 1.4.0 总是比 1.3.6 更新。

## PEAR 打包和严格的版本验证

PEAR 软件包仓库位于 [pear.php.net](http://pear.php.net)，使用严格的版本管理协议。所有软件包版本都是 X.Y.Z 格式，例如 "1.3.0"。第一个数字（X）用于描述应用程序接口（API）版本，第二个数字（Y）用于描述功能集版本，第三个数字（Z）用于描述错误修复修订级别。以下是一些版本号及其含义的示例：

| 样例 PEAR 版本号及其含义 |   |
| --- | --- |
| 版本号 | 含义 |
| --- | --- |
| 1.0.0 | 稳定的 API 版本 1.0，包的初始稳定发布 |
| 0.4.5 | 不稳定的（开发中）API，第四个功能集设计，该功能集的第五次错误修复发布 |
| 2.4.0 | 稳定的 API 版本 2.0，第四个功能集设计，该新功能集的初始发布 |
| 1.3.4 | 稳定的 API 版本 1.0，自初始稳定发布以来的第三个新功能集，该功能集的第四次错误修复发布 |

在安装时，此信息是无关紧要的：PEAR 安装程序将安装它所提供的任何内容，包括类似于 24.25094.39.430 或 23.4.0-r1 等版本编号方案。然而，通过以下方式打包包时使用的验证方式：

```php
$ pear package 

```

与下载 PEAR 包时使用的验证方式相比要严格得多。为了帮助试图满足 PEAR 包仓库严格编码标准的开发者，PEAR 包中位于`PEAR/Validate.php`的验证例程集合会检查 PEAR 编码标准的多个重要方面，并对任何不符合标准的内容发出警告。例如，版本 1.0.0 必须是稳定的（而不是`devel, alpha`或`beta`稳定性），因此以下来自`package.xml`的片段将导致警告：

```php
<version>
<release>1.0.0</release>
<api>1.0.0</api>
</version>
<stability>
<release>beta</release>
<api>beta</api>
</stability>

```

具体来说，它看起来可能像这样：

**$ pear package**

```php
Warning: Channel validator warning: field "version" - version 1.0.0 probably should not be alpha or beta 

```

尽管这对希望遵守 PEAR 严格编码标准的人来说非常有帮助，但在某些情况下，这种警告并不有用，反而令人烦恼或分心。例如，一个私有项目可能希望使用 Linux 版本控制方案。幸运的是，有一种方法可以控制在 PEAR 安装程序的打包、下载或安装阶段处理自定义验证例程的方式。通过扩展位于`PEAR/Validate.php`中的`PEAR_Validate`类，可以使用面向对象继承创建一个特殊的验证器。要激活它，验证器必须与一个频道相关联。尽管这个过程将在第五章中详细讨论，但这里有一个简单的示例。

`pecl.php.net`上的 PHP 扩展开发者有一个更宽松的版本验证系统，接受的版本控制差异很大，从两位数的 1.0 到 maxdb 包（[`pecl.php.net/maxdb`](http://pecl.php.net/maxdb)）使用的 7.5.00.26，再到 MySQL 的 MaxDB 数据库使用的镜像版本控制。因此，[pecl.php.net](http://pecl.php.net)是[pear.php.net](http://pear.php.net)的一个独立频道，在其频道定义文件`channel.xml`（也在第五章中详细讨论）中定义了一个频道验证器：

```php
<validatepackage version="1.0">PEAR_Validator_PECL</validatepackage>

```

这个特定的频道验证器与 PEAR 安装程序一起分发，在文件`PEAR/Validator/PECL.php`中。这个文件是定制频道验证器的完美示例，也是最简单的，所以在这里，展示其全部的辉煌：

```php
<?php
/**
* Channel Validator for the pecl.php.net channel
*
* PHP 4 and PHP 5
*
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version CVS: $Id: PECL.php,v 1.3 2005/08/21 03:31:48 cellog
Exp $
* @link http://pear.php.net/package/PEAR
* @since File available since Release 1.4.0a5
*/
/**
* This is the parent class for all validators
*/
require_once 'PEAR/Validate.php';
/**
* Channel Validator for the pecl.php.net channel
* @category pear
* @package PEAR
* @author Greg Beaver <cellog@php.net>
* @copyright 1997-2005 The PHP Group
* @license http://www.php.net/license/3_0.txt PHP License 3.0
* @version Release: @package_version@
* @link http://pear.php.net/package/PEAR
* @since Class available since Release 1.4.0a5
*/
class PEAR_Validator_PECL extends PEAR_Validate
{
function validateVersion()
{
if ($this->_state == PEAR_VALIDATE_PACKAGING) {
$version = $this->_packagexml->getVersion();
$versioncomponents = explode('.', $version);
$last = array_pop($versioncomponents);
if (substr($last, 1, 2) == 'rc') {
$this->_addFailure('version', 'Release Candidate versions must have ' .
'upper-case RC, not lower-case rc');
return false;
}
}
return true;
}
function validatePackageName()
{
$ret = parent::validatePackageName();
if ($this->_packagexml->getPackageType() == 'extsrc') {
if (strtolower($this->_packagexml->getPackage()) !=
strtolower($this->_packagexml->getProvidesExtension())) {
$this->_addWarning('providesextension', 'package name "' .
$this->_packagexml->getPackage() . '" is different from extension name "' .
$this->_packagexml->getProvidesExtension());
}
}
return $ret;
}
}
?>

```

这个类只是简单地覆盖了其父类的严格版本验证，然后添加了一个 PECL 特定的检查，以查看一个扩展包是否声称分发一个与包名不同名称的扩展。此外，它还会检查版本中是否包含（小写）`rc`作为候选发布版本，因为 PHP 的`version_compare()`函数（[`www.php.net/version_compare`](http://www.php.net/version_compare)）对此处理方式与包含（大写）`RC`的版本非常不同。

### 小贴士

**为什么 pear.php.net 和 pecl.php.net 是独立的频道？**

在早期的 PEAR 版本中，`pear`命令用于安装 PEAR 包和**PHP 扩展社区库**（**PECL**）包。由于多个原因，这一变化发生在 1.4.0 版本中。首先，[pear.php.net](http://pear.php.net)的开发者正在开发用 PHP 编写的包。而[pecl.php.net](http://pecl.php.net)的开发者正在开发用 C 编写的包，这些包被编译成共享的`.dll`或`.so`文件，作为 PHP 本身的内部组件（扩展）。与 PECL 风格的包相比，PEAR 风格的包在安装和维护方面存在固有的差异。

这些差异之一是正确版本的重要性。PECL 和 PHP 扩展不能与冲突的扩展共存；一次只能运行一个。PEAR 包没有这种限制，因为多个不同版本的 PEAR 包可以共存并通过`include_path`进行交替加载，因此版本控制变得尤为重要。

此外，由于文件锁定，加载到内存中的扩展无法卸载，这使得在不使用`php.ini`的情况下升级 PHP 扩展成为不可能。

有许多其他小原因加在一起导致了需要分割，例如对包的功能的混淆（这是一个 PHP 扩展还是用 PHP 编写的脚本？），因此现在我们既有`pear`也有新的`pecl`命令来管理 PECL 包。

## 企业级依赖管理

依赖关系是软件设计中的自然演变。如果您的论坛包尝试通过使用模板引擎将业务逻辑与显示分离，那么将精力集中在开发论坛功能上要比设计一个新的模板引擎好得多。不仅现有模板引擎的作者在模板引擎上花费了更多的时间和精力，而且他们的模板引擎已经被成千上万的开发者（就像您一样）使用。所有常见问题和大多数不寻常问题都已被遇到并解决。如果在您发现新的问题时，您可以向维护者报告，甚至可以充分利用开源的力量自行修复问题并将解决方案反馈给维护者。

另一方面，这需要信任模板引擎的维护者。通过使用模板引擎，您隐含地信任维护者能够有效地管理它，修复发现的全部错误，并防止在未来的版本中引入新的错误。您还信任维护者会继续维护该包，并回应您遇到的问题。

PEAR 开发者已经存在很长时间，知道在最坏的情况下，这种信任可能是天真的或愚蠢的，因为软件仍然由有能力出错的人类维护。幸运的是，由于这种知识，`package.xml` 提供了对依赖关系“信任”的完全控制。

### 小贴士

**package.xml 中的依赖关系**

关于 `package.xml` 中依赖关系的工作方式，请参阅 *外部依赖关系* 部分

最简单的依赖关系会告诉安装程序必须使用该包，但我们不需要检查版本问题：只要安装即可。在 `package.xml 2.0` 中，这种依赖关系看起来像这样：

```php
<package>
<name>Dependency</name>
<channel>pear.php.net</channel>
</package>

```

一个稍微更严格的依赖关系可能会告诉安装程序只安装比最小版本 1.2.0 新的版本：

```php
<package>
<name>Dependency</name>
<channel>pear.php.net</channel>
<min>1.2.0</min>
</package>

```

进一步来说，可能存在一些混乱的发布。我们可以告诉安装程序忽略特定的发布版本 1.2.0 和 1.4.2，但其他所有比 1.2.0 新的版本都可以安装：

```php
<package>
<name>Dependency</name>
<channel>pear.php.net</channel>
<min>1.2.0</min>
<exclude>1.2.0</exclude>
<exclude>1.4.2</exclude>
</package>

```

最后，我们还可以告诉安装程序，我们强烈推荐版本 1.4.5，并且除非我们说可以升级，否则不要升级：

```php
<package>
<name>Dependency</name>
<channel>pear.php.net</channel>
<min>1.2.0</min>
<recommended>1.4.5</recommended>
<exclude>1.2.0</exclude>
<exclude>1.4.2</exclude>
</package>

```

在最后一种情况下，我们得到了对依赖关系的大量控制。最终用户在依赖关系的维护者与您合作并认证其与另一个标签 `<compatible>` 的兼容性，或者您发布了一个推荐新版本的包之前，不会意外地通过升级到依赖关系的新版本而破坏我们的包。

`package.xml` 中依赖项提供的不同信任级别意味着您可以安全地依赖其他软件包，但这只是 PEAR 依赖项功能的开始。依赖项解决的另一个常见问题是与最终用户的计算机、操作系统、PHP 版本或 `php.ini` 中启用的扩展的基本不兼容性。PEAR 为这些情况中的每一个都提供了依赖项标签。此外，可选功能或插件可以通过可选依赖项或依赖组实现。可能性的列表令人印象深刻，而 `package.xml` 中的语法简单且易于学习。

# 最终用户的分发和升级

从最终用户的角度来看，在使用解压即用的软件包时，面临的最复杂任务之一就是升级。在闭源世界中，软件包的新版本会破坏旧版本中曾经正常工作的某些功能，以此来强迫用户升级，这有时需要大量的工作。

在开源世界中，许多开发者通过引入令人兴奋的新功能继续遵循这一模式，这意味着您将无法再使用旧版本。实际的升级过程通常意味着用新文件覆盖当前版本，可能还需要新的配置。此外，它还带来了完全破坏实时网站的可怕前景，这促使需要某种备份系统。

通过使用 PEAR 安装程序，所有这些恐惧和危险都已成为过去。使用以下方法升级到新版本非常简单：

```php
$ pear upgrade Package 

```

将软件包降级到旧版本同样简单：

```php
$ pear upgrade --force Package-1.2.3 

```

这使得维护一个实时网站既简单又安全——这是一个罕见而美好的组合。请注意，此示例假设软件包的旧版本为 `1.2.3`。

此外，升级或安装依赖项同样简单。对于使用 `package.xml 2.0` 的软件包，总是下载并安装所需的依赖项，而对于使用 `package.xml 1.0` 的软件包，可以使用 `--onlyreqdeps`（仅必需依赖项）选项自动下载和安装依赖项，如下所示：

```php
$ pear upgrade --onlyreqdeps Package

```

可以使用 `--alldeps`（所有依赖项）选项自动安装可选依赖项：

```php
$ pear upgrade --alldeps Package 

```

此外，通过依赖组创建功能组，或者将几个相关软件包组合成一个单一依赖项，可以轻松实现，这意味着用户可以像安装 PEAR 的网络安装程序功能一样安装功能组：

```php
$ pear install PEAR#webinstaller 

```

当配置一个软件包时，通常需要执行大量工作来配置文件位置、设置数据库等。PEAR 提供了一些简单的方法来自动化配置，同时也提供了通过安装后脚本标准化任何复杂设置的方法。

如果您像我一样，想到内置在 PEAR 安装程序中的所有可能性会让您的心跳加速，充满期待。即使您不是那么极客，我也相信您会发现 PEAR 安装程序的力量会让您的编程生活变得更轻松。

# `package.xml`结构概述

`package.xml`包含了 PEAR 安装程序安装和配置 PEAR 包所需的所有信息。通过利用 XML 和 XSchema 等标准来记录包结构（[`pear.php.net/dtd/package1.xsd`](http://pear.php.net/dtd/package1.xsd)和[`pear.php.net/dtd/package2.xsd`](http://pear.php.net/dtd/package2.xsd)分别提供了`package.xml 1.0`和`package.xml 2.0`的完整定义），PEAR 打开了未来编程的可能性，否则这些可能性将不可用。例如，使用 XSchema 允许通过 XML 命名空间扩展`package.xml`，以提供自定义功能。

讨论到`package.xml`时，理解`package.xml 1.0`和`package.xml 2.0`之间的共性和差异是很重要的。`package.xml 2.0`是`package.xml 1.0`的超集。换句话说，可以将每个可能的`package.xml`版本 1.0 表示为`package.xml 2.0`，但有一大批`package.xml 2.0`不能简化为唯一的`package.xml 1.0`。

了解`package.xml 1.0`的结构最好的方法是查看 PEAR 仓库的 CVS（[cvs.php.net](http://cvs.php.net)）。每个子目录都包含一个`package.xml`文件。如果该`package.xml`文件以`<package version="1.0"...`开头，那么它就是`package.xml 1.0`。许多包已经利用了`package.xml 2.0`，使用名为`package2.xml`的文件。同时，调查 PECL 仓库[`cvs.php.net/pecl/`](http://cvs.php.net/pecl/)也很有用，看看 PHP 扩展开发者是如何使用`package.xml`的。

要探索等效的`package.xml 2.0`，有一个方便的 PEAR 命令可以将`package.xml 1.0`转换为`package.xml 2.0`。该命令的调用方式如下：

```php
$ pear convert 

```

这将解析当前目录中名为`package.xml`的文件，并以新格式输出名为`package2.xml`的文件。

此外，关于`package.xml`的完整最新文档始终可在 PEAR 手册的[`pear.php.net/manual/`](http://pear.php.net/manual/)中找到。

为了这本书的目的，我们将主要讨论`package.xml 2.0`，并且只提及 1.0 版本，以供熟悉旧格式的人了解重要的概念性变化。

这里是一个`package.xml`文件的示例。这个示例是从 PEAR 包本身取出的，展示了将在后续章节中探讨的`package.xml`的一些功能。您可以随意浏览这个示例，稍后再回来参考。现在，只需吸收基本结构，具体细节将在您阅读文本的后续部分时变得有意义。

```php
<?xml version="1.0" encoding="UTF-8"?>
<package version="2.0" 

xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0
http://pear.php.net/dtd/tasks-1.0.xsd
http://pear.php.net/dtd/package-2.0
http://pear.php.net/dtd/package-2.0.xsd">
<name>PEAR</name>
<channel>pear.php.net</channel>
<summary>PEAR Base System</summary>
<description>The PEAR package contains:
* the PEAR installer, for creating, distributing
and installing packages
* the beta-quality PEAR_Exception PHP5 error handling mechanism
* the beta-quality PEAR_ErrorStack advanced error handling class
* the PEAR_Error error handling mechanism
* the OS_Guess class for retrieving info about the OS
where PHP is running on
* the System class for quick handling of common operations
with files and directories
* the PEAR base class
</description>
<lead>
<name>Greg Beaver</name>
<user>cellog</user>
<email>cellog@php.net</email>
<active>yes</active>
</lead>
<lead>
<name>Stig Bakken</name>
<user>ssb</user>
<email>stig@php.net</email>
<active>yes</active>
</lead>
<lead>
<name>Tomas V.V.Cox</name>
<user>cox</user>
<email>cox@idecnet.com</email>
<active>yes</active>
</lead>
<lead>
<name>Pierre-Alain Joye</name>
<user>pajoye</user>
<email>pajoye@pearfr.org</email>
<active>yes</active>
</lead>
<helper>
<name>Martin Jansen</name>
<user>mj</user>
<email>mj@php.net</email>
<active>no</active>
</helper>
<date>2005-10-09</date>
<version>
<release>1.4.2</release>
<api>1.4.0</api>
</version>
<stability>
<release>stable</release>
<api>stable</api>
</stability>
<license uri="http://www.php.net/license">PHP License</license>
<notes>
This is a major milestone release for PEAR. In addition to several killer features, every single element of PEAR has a regression test, and so stability is much higher than any previous PEAR release, even with the beta label.
New features in a nutshell:
* full support for channels
* pre-download dependency validation
* new package.xml 2.0 format allows tremendous flexibility * while maintaining BC support for optional dependency groups * and limited support for sub-packaging robust dependency support
* full dependency validation on uninstall
* remote install for hosts with only ftp access - * no more problems with restricted host installation
* full support for mirroring
* support for bundling several packages into a single tarball
* support for static dependencies on a uri-based package
* support for custom file roles and installation tasks
</notes>
<contents>
<dir name="/">
<dir name="OS">
<file name="Guess.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
</dir> <!-- /OS -->
<dir name="PEAR">
<dir name="ChannelFile">
<file name="Parser.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
</dir> <!-- /PEAR/ChannelFile -->
<dir name="Command">
<file name="Auth-init.php" role="php"/>
<file name="Auth.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
<file name="Build-init.php" role="php"/>
<file name="Build.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
[snip...] 
</dir> <!-- /PEAR -->
<dir name="scripts" baseinstalldir="/">
<file name="pear.bat" role="script">
<tasks:replace from="@bin_dir@" to="bin_dir" type="pear-config"
/>
<tasks:replace from="@php_bin@" to="php_bin" type="pear-config"
/>
<tasks:replace from="@include_path@" to="php_dir" type="pear-
config" />
<tasks:windowseol/>
</file>
<file name="peardev.bat" role="script">
<tasks:replace from="@bin_dir@" to="bin_dir" type="pear-config"
/>
<tasks:replace from="@php_bin@" to="php_bin" type="pear-config"
/>
<tasks:replace from="@include_path@" to="php_dir" type="pear-config" />
<tasks:windowseol/>
</file>
<file name="pecl.bat" role="script">
<tasks:replace from="@bin_dir@" to="bin_dir" type="pear-config"
/>
<tasks:replace from="@php_bin@" to="php_bin" type="pear-config"
/>
<tasks:replace from="@include_path@" to="php_dir" type="pear-config" />
<tasks:windowseol/>
</file>
<file name="pear.sh" role="script">
<tasks:replace from="@php_bin@" to="php_bin" type="pear-config"
/>
<tasks:replace from="@php_dir@" to="php_dir" type="pear-config"
/>
<tasks:replace from="@pear_version@" to="version" type="package-info" />
<tasks:replace from="@include_path@" to="php_dir" type="pear-config" />
<tasks:unixeol/>
</file>
[snip...]
</dir> <!-- /scripts -->
<file name="package.dtd" role="data" />
<file name="PEAR.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
<file name="System.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info" />
</file>
<file name="template.spec" role="data" />
</dir> <!-- / -->
</contents>
<dependencies>
<required>
<php>
<min>4.2</min>
</php>
<pearinstaller>
<min>1.4.0a12</min>
</pearinstaller>
<package>
<name>Archive_Tar</name>
<channel>pear.php.net</channel>
<min>1.1</min>
<recommended>1.3.1</recommended>
<exclude>1.3.0</exclude>
</package>
<package>
<name>Console_Getopt</name>
<channel>pear.php.net</channel>
<min>1.2</min>
<recommended>1.2</recommended>
</package>
<package>
<name>XML_RPC</name>
<channel>pear.php.net</channel>
<min>1.4.0</min>
<recommended>1.4.1</recommended>
</package>
<package>
<name>PEAR_Frontend_Web</name>
<channel>pear.php.net</channel>
<max>0.5.0</max>
<exclude>0.5.0</exclude>
<conflicts/>
</package>
<package>
<name>PEAR_Frontend_Gtk</name>
<channel>pear.php.net</channel>
<max>0.4.0</max>
<exclude>0.4.0</exclude>
<conflicts/>
</package>
<extension>
<name>xml</name>
</extension>
<extension>
<name>pcre</name>
</extension>
</required>
<group name="remoteinstall" hint="adds the ability to install
packages to a remote ftp server">
<subpackage>
<name>PEAR_RemoteInstaller</name>
<channel>pear.php.net</channel>
<min>0.1.0</min>
<recommended>0.1.0</recommended>
</subpackage>
</group>
<group name="webinstaller" hint="PEAR's web-based installer">
<package>
<name>PEAR_Frontend_Web</name>
<channel>pear.php.net</channel>
<min>0.5.0</min>
</package>
</group>
<group name="gtkinstaller" hint="PEAR's PHP-GTK-based installer">
<package>
<name>PEAR_Frontend_Gtk</name>
<channel>pear.php.net</channel>
<min>0.4.0</min>
</package>
</group>
</dependencies>
<phprelease>
<installconditions>
<os>
<name>windows</name>
</os>
</installconditions>
<filelist>
<install as="pear.bat" name="scripts/pear.bat" />
<install as="peardev.bat" name="scripts/peardev.bat" />
<install as="pecl.bat" name="scripts/pecl.bat" />
<install as="pearcmd.php" name="scripts/pearcmd.php" />
<install as="peclcmd.php" name="scripts/peclcmd.php" />
<ignore name="scripts/peardev.sh" />
<ignore name="scripts/pear.sh" />
<ignore name="scripts/pecl.sh" />
</filelist>
</phprelease>
<phprelease>
<filelist>
<install as="pear" name="scripts/pear.sh" />
<install as="peardev" name="scripts/peardev.sh" />
<install as="pecl" name="scripts/pecl.sh" />
<install as="pearcmd.php" name="scripts/pearcmd.php" />
<install as="peclcmd.php" name="scripts/peclcmd.php" />
<ignore name="scripts/pear.bat" />
<ignore name="scripts/peardev.bat" />
<ignore name="scripts/pecl.bat" />
</filelist>
</phprelease>
<changelog>
<release>
<version>
<release>1.3.6</release>
<api>1.3.0</api>
</version>
<stability>
<release>stable</release>
<api>stable</api>
</stability>
<date>2005-08-18</date>
<license>PHP License</license>
<notes>
* Bump XML_RPC dependency to 1.4.0
* return by reference from PEAR::raiseError()
</notes>
</release>
<release>
<version>
<release>1.4.0a1</release>
<api>1.4.0</api>
</version>
<stability>
<release>alpha</release>
<api>alpha</api>
</stability>
<date>2005-02-26</date>
<license uri="http://www.php.net/license/3_0.txt">PHP
License</license>
<notes>
This is a major milestone release for PEAR. In addition to several
killer features,
every single element of PEAR has a regression test, and so
stability is much higher
than any previous PEAR release, even with the alpha label.
New features in a nutshell:
* full support for channels
* pre-download dependency validation
* new package.xml 2.0 format allows tremendous flexibility while
maintaining BC
* support for optional dependency groups and limited support for
sub-packaging
* robust dependency support
* full dependency validation on uninstall
* support for binary PECL packages
* remote install for hosts with only ftp access - * no more problems
with
restricted host installation
* full support for mirroring
* support for bundling several packages into a single tarball
* support for static dependencies on a url-based package
Specific changes from 1.3.5:
* Implement request #1789: SSL support for xml-rpc and download
* Everything above here that you just read
</notes>
</release>
[snip...] 
</changelog>
</package>

```

# package.xml 1.0 和 2.0 之间的共享标签

如果你已经对`package.xml 1.0`有基本的了解，只想查看`package.xml 2.0`中的变化，最好浏览接下来的几节内容。每个部分的差异总是在开头展示，随后深入探讨变化背后的原因。

在深入探讨`package.xml 2.0`的高级新功能之前，了解从`package.xml 1.0`继承下来的标签和属性非常重要。这两种格式都有共同的标签。大多数标签没有变化。少数标签增加了信息或稍微更改了名称，而其他标签则被完全重新设计。让我们开始吧。

## 包元数据

两个版本的`package.xml`都提供了类似的包元数据。任何包都必须包含的一些基本信息包括：

+   包名称/频道

+   维护者（作者）

+   包描述

+   包摘要（单行描述）

这些信息在版本之间通常保持不变。例如，包名称和频道是恒定的。包描述和摘要很少改变。维护者可能会更频繁地改变，这取决于社区，因此尽管这被归类为包元数据，但将其视为基于发布的信息可能是合理的。

### 包名称/频道

这两个字段位于`package.xml`的开头，是 PEAR 安装程序区分包的核心。它们就像数据库表中的主键：包/频道 = 唯一的包。请注意，`<channel>`和`<uri>`标签仅在`package.xml 2.0`中存在。

```php
<name>Packagename</name>
<channel>channel.example.com</channel> -or- <uri>http://www.example.com/Packagename-1.2.3</uri>

```

### 小贴士

**频道和 package.xml 1.0**

如你之前所学的，频道的概念是在`package.xml 2.0`中引入的。当它不是其规范的一部分时，`package.xml 1.0`是如何定义频道的呢？

PEAR 以一种简单的方式处理这个问题：使用`package.xml 1.0`打包的所有包都被安装，就好像在包名称声明之后立即插入了一个`<channel>pear.php.net</channel>`标签。请注意，[pecl.php.net](http://pecl.php.net)上的使用`package.xml 1.0`的包在升级到使用`package.xml 2.0`时允许迁移到[pecl.php.net](http://pecl.php.net)频道，但所有其他包必须从新的频道重新开始。

包名称使用`<name>`标签声明，并且必须以字母开头，否则只能包含字母、数字和下划线字符，除非频道特定的验证器允许包名称采用其他格式。频道特定的验证器在第五章中有详细说明。如果你只是创建一个包，你不需要了解任何关于频道特定验证器的内容。如果你的包满足包的要求，当你运行：

```php
$ pear package 

```

你的包将会无错误地创建。否则，打包将会失败，并且会显示一个或多个错误消息，描述失败的原因。

`<channel>`标签不能与`<uri>`标签共存。使用`<uri>`标签的包实际上是伪频道`__uri`的一部分，这个频道没有与之关联的服务器或协议。`__uri`频道实际上是一个真正的魔法频道，它只用来作为命名空间，防止基于 URI 的包与其他频道的包冲突。

例如，考虑一个`package.xml`文件开始的包：

```php
<name>Packagename</name>
<uri>http://pear.php.net/Packagename-1.2.3</uri>

```

即使包的版本号为`1.2.3`，这个包与`package.xml`以以下代码开始的包也不相同。

```php
<name>Packagename</name>
<channel>pear.php.net</channel>

```

基于 URI 的包是严格基于发布的。`<uri>`标签必须是一个绝对的真实 URI，可以用来访问该包。然而，URI 不应该包含`.tgz`或`.tar`文件扩展名，但两者都应该存在。在我们的例子中，[`pear.php.net/Packagename-1.2.3.tgz`](http://pear.php.net/Packagename-1.2.3.tgz)和[`pear.php.net/Packagename-1.2.3.tar`](http://pear.php.net/Packagename-1.2.3.tar)都应该存在，并且除了`.tgz`使用 zlib 压缩外，两者应该是相同的。

### 维护者（作者）

`package.xml`中维护者的列表应该起到与传统`AUTHORS`文件相同的作用。然而，这个作者列表不仅仅是有用的信息。《user》标签被频道服务器用来将包维护者与包的发布匹配起来，允许非频道管理员上传他们维护的软件包的发布版本。例如，如果你维护的是通过[pear.php.net](http://pear.php.net)发布的包，那么`<user>`标签必须包含你的 PEAR 用户名。对于非频道发布，这些标签的内容仅作为信息，让包的最终用户知道如何联系你。

`package.xml 1.0`中的`<role>`标签只能包含`lead`、`developer`、`contributor`或`helper`的值。这证明使用 XSchema 进行验证是不可能的，因此为了简化事情，`package.xml 2.0`已经将`<role>`标签的内容提取出来，并创建了新的标签`<lead>`、`<developer>`、`<contributor>`和`<helper>`。此外，`<maintainers>`和`<maintainer>`标签已被移除以简化解析。

对于`package.xml 1.0`：

```php
<maintainers>
<maintainer>
<user>pearserverhandle</user>
<role>lead</role>
<name>Dorkus McForkus</name>
<email>dorkus@example.com</email>
</maintainer>
<maintainer>
<user>pearserverhelper</user>
<role>helper</role>
<name>Horgie Borgie</name>
<email>borgie@example.com</email>
</maintainer>
</maintainers>

```

对于`package.xml 2.0`：

```php
<lead>
<name>Dorkus McForkus</name>
<user>pearserverhandle</user>
<email>dorkus@example.com</email>
<active>yes</active>
</lead>
<helper>
<name>Horgie Borgie</name>
<user>pearserverhelper</user>
<email>borgie@example.com</email>
<active>no</active>
</helper>

```

### 包描述和摘要

下面的示例摘要/描述对显示了如何在`package.xml 1.0`和`package.xml 2.0`中使用`<summary>`和`<description>`标签。这些标签仅用于信息，不被 PEAR 安装程序的安装部分使用。如果包通过频道提供，`list-all`等命令将显示摘要。当用户输入`info`或`remote-info`等命令以显示特定包或发布的详细信息时，将显示描述。

确保这些标签清晰、简洁且易于理解。我经常看到像`GBRL`这样的摘要，用于名为`File_GBRL`的包——如果这些缩写不是众所周知，请定义它们！

```php
<summary>Provides an interface to the BORG collective</summary>
<description>
The Net_BORG package uses PHP to interface with the neural
Implants found in the face of all BORG. Both an object-oriented
and a functional interface is available. Both will be used.
Resistance is futile.
</description>

```

## 基本发布元数据

`package.xml`的其余部分通常是特定于发布的资料，但有少数例外。特定于发布的资料比包名称等事物更容易变化。具体来说，`package.xml`中记录的特定于发布的资料区域包括：

+   包版本

+   包稳定性

+   外部依赖

+   发布说明

+   发布许可

+   更新日志

+   文件列表，或包的内容

在`package.xml 1.0`中，这些数据被包含在冗余的`<release>`标签中。在版本 2.0 中，已删除此标签。

### 包版本

包版本非常重要，因为这是安装程序用来确定不同发布版本相对年龄的主要机制。版本是连续的，这意味着版本 1.3.6 比版本 1.4.0b1 旧，即使 1.3.6 是在 1.4.0b1 之后一个月发布的。

```php
<version>1.2.3</version>

```

对于`package.xml 2.0:` 

```php
<version>
<release>1.2.3</release>
<api>1.0.0</api>
</version>

```

发布的 API 版本仅用于信息目的。然而，这可以在以下`replace`任务中使用：

```php
<file name="Foo.php" role="php">
<tasks:replace from="apiversion" to="@API-VER@" type="package-info"/>
</file>

```

这减少了在文件内部维护版本时的冗余。事实上，通过 API 方法提供 API 版本是一个非常好的主意。在我的包中，我通常提供以下代码：

```php
/**
* Get the current API version
* @return string
*/
function APIVersion()
{
Return '@API-VER@';
}

```

在打包后，包含上述代码的`Foo.php`文件将看起来像这样：

```php
/**
* Get the current API version
* @return string
*/
function APIVersion()
{
Return '1.0.0';
}

```

换句话说，所有`@API-VER@`标记的实例都将被`<api>`版本标签的内容所替换。替换文件任务用于执行这个魔法。

### 包稳定性

在`package.xml 1.0`中，使用`<state>`标签来描述代码的稳定性。在`package.xml 2.0`中，使用`<stability>`内的`<release>`标签来描述代码的稳定性。

```php
<state>beta</state>

```

对于`package.xml 2.0:` 

```php
<stability>
<release>beta</release>
<api>stable</api>
</stability>

```

PEAR 安装程序结合使用发布稳定性和最终用户的`preferred_state`配置变量来确定发布是否足够稳定以安装。如果用户希望安装`Foo`包，并且有这些发布可用：

| Version | Stability |
| --- | --- |
| 1.0.1 | stable |
| 1.1.0a1 | alpha |
| 1.0.0 | stable |
| 0.9.0 | beta |
| 0.8.0 | alpha |

安装程序将根据用户的`preferred_state`设置选择要安装的版本。

| preferred_state value | 要安装的 Foo 版本 |
| --- | --- |
| stable | 1.0.1 |
| beta | 1.0.1 |
| alpha | 1.1.0a1 |

注意，对于 `preferred_state` 为 `beta` 的情况，当可以选择较新版本 1.0.1（稳定）和较旧版本 0.9.0（beta）时，安装程序将选择最新、最稳定的版本——版本 1.0.1。对于 `preferred_state` 为 `alpha` 的情况，安装程序将选择较新但不太稳定的版本 1.1.0a1（alpha），即使版本 1.0.1 是在版本 1.1.0a1 之后发布的。

释放稳定性合法稳定性的列表按稳定性递减的顺序是：

+   `stable:` 代码应在所有情况下都能正常工作。

+   `beta:` 代码应在所有情况下都能正常工作，并且是功能完整的，但需要实际世界的测试。

+   `alpha:` 代码处于变化状态，功能可能会随时更改，稳定性不确定。

+   `devel:` 代码尚未功能完善，可能在大多数情况下无法工作。

+   `snapshot:` 这是正常发布之间的实时开发期间从源代码中获取的开发代码的当前副本。

API 稳定性服务于与 API 版本相同的信息目的。它不会被安装程序使用，但可以用来跟踪 API 变化的速率。标记为稳定的 API 实际上除了添加新功能外，不应发生变化——用户需要能够依赖稳定的 API，以便包依赖项能够正常工作。这是企业级依赖项的关键特性。

API 合法稳定性的列表略有不同，因为不允许 API 快照。API 稳定性应被视为：

+   `stable:` API 已设置，不会破坏向后兼容性。

+   `beta:` API 可能已设置，并且只有在测试中遇到设计中的严重错误时才会更改，以修复严重错误。

+   `alpha:` API 是流动的，可能会更改，破坏现有功能，以及添加新功能。

+   `devel:` API 极不稳定，可能随时发生重大变化。

### 外部依赖

PEAR 安装程序识别两种类型的依赖项：*必需* 和 *可选* 依赖项。此外，还有两类依赖项，一类是限制安装的依赖项，另一类是描述主要包使用的其他资源（包/扩展）的依赖项。限制性依赖项由 `<php>`, `<pearinstaller>`, `<arch>`, `<extension>` 和 `<os>` 依赖项定义。基于资源的依赖项由 `<package>`, `<subpackage>` 和 `<extension>` 依赖项定义。

虽然在 `package.xml 1.0` DTD 中定义了多种依赖类型，但只有三种被实现过：

+   `pkg:` 对包的依赖

+   `ext:` 对扩展的依赖

+   `php:` 对 PHP 版本的依赖

`package.xml 1.0` 中依赖项的结构相当简单：

```php
<deps>
<dep type="php" rel="ge" version="4.2.0"/>
<dep type="pkg" rel="has">PackageName</dep>
<dep type="pkg" rel="ge" version="1.0">PackageName2</dep>
<dep type="pkg" rel="ge" version="1.0"
optional="yes">PackageName3</dep>
<dep type="ext" rel="not">ExtensionName</dep>
</deps>

```

`rel` 属性的合法值是：

+   `has:` 依赖项必须存在

+   `not:` 依赖项必须不存在

+   `gt:` 与 `version` 属性结合使用时，依赖项的版本必须大于所需版本。

+   `ge:` 与 `version` 属性结合使用时，依赖项的版本必须大于或等于所需版本

+   `eq:` 与 `version` 属性结合使用时，依赖项必须具有 version == 所需版本。

+   `lt:` 与 `version` 属性结合使用时，依赖项必须具有 version < 所需版本。

+   `le:` 与 `version` 属性结合使用时，依赖项必须具有 version <= 所需版本。

+   `ne:` 与 `version` 属性结合使用时，依赖项必须具有 version != 所需版本。（仅限 PEAR 版本 1.3.6。）

### 小贴士

**版本比较是如何进行的？**

PHP 函数 `version_compare()` 用于确定基于版本的依赖项是否有效。`version_compare()` 的文档在 [`www.php.net/version_compare`](http://www.php.net/version_compare)。

只有在积累了大量经验之后，这种方法的严重设计缺陷才显现出来：

+   *使用 xmllint 等工具进行 XML 验证无法揭示无效的依赖项*。考虑以下依赖项 `<dep type="php" rel="has" version="4.3.0"/>`。这个依赖项是无效的，因为 `rel="has"` 忽略了版本属性，因此依赖项验证将不会执行——根据定义，每个 PEAR 安装都安装了 PHP。

+   *依赖项升级的信任级别无法控制*。 PEAR 包依赖于 Console_Getopt 包。在某个时刻，Console_Getopt 的维护者 *修复* 了一个突然导致 PEAR 升级后停止工作的错误。经过一番混乱之后才找到了解决方案，但这一事件凸显了 PEAR 安装程序在 1.4.0 版本之前的致命缺陷：对包的依赖本身是不安全的。无法限制对依赖项的信任。`rel="eq"` 属性没有达到预期的效果，因为这会阻止出于任何原因的安全升级，实际上冻结了开发。此外，依赖项验证中的缺陷意味着即使是升级到较新版本的包也是被禁止的。

    如果当前版本有一个 `<dep type="pkg" rel="eq" version="1.0">Deppackage</dep>` 的依赖项标签，PEAR 安装程序将查看新版本的依赖项，即 `<dep type="pkg" rel="eq" version="1.1">Deppackage</dep>`，然后查看磁盘上是否已安装 Deppackage 版本 1.0，并失败升级。即使主包和 Deppackage 都已通过，Deppackage 的升级也会失败，因为主包已安装的版本需要 Deppackage 的 1.0 版本。

    如果当前版本有一个 `<dep type="pkg" rel="eq" version="1.0">Deppackage</dep>` 的依赖项标签，PEAR 安装程序将查看新版本的依赖项，即 `<dep type="pkg" rel="eq" version="1.1">Deppackage</dep>`，然后查看磁盘上是否已安装 Deppackage 版本 1.0，并失败升级。即使主包和 Deppackage 都已通过，Deppackage 的升级也会失败，因为主包已安装的版本需要 Deppackage 的 1.0 版本。

+   *基于 PECL 扩展包的依赖项无法工作*。一个`<dep type="ext" rel="has">peclextension</dep>`标签只会检查内存中的扩展。大多数 PECL 扩展可以直接构建到 PHP 中（非共享），作为共享模块与 PHP 一起分发，或者下载并安装。如果 PECL 扩展被构建到 PHP 中，作为共享模块分发，或通过 PECL 安装，则`type="ext"`依赖项将正常工作。不幸的是，为了使用 PEAR 安装程序升级扩展，必须禁用`php.ini`，否则文件锁定将防止在当前 PHP 进程使用时覆盖扩展。较新的扩展，如 PDO，有依赖 PDO 扩展存在的驱动程序。如果禁用`php.ini`，将无法检测扩展以验证依赖项！

#### 简化 package.xml 的 XML 验证

为了使外部工具能够验证依赖项，需要重新设计所需的结构。通常，在`package.xml 2.0`中，优先使用标签而不是属性，尤其是优先使用属性值。`package.xml 1.0`中的依赖项示例可以用以下方式表示在`package.xml 2.0`中：

```php
<dependencies>
<required>
<php>
<min>4.2.0</min>
</php>
<pearinstaller>
<min>1.4.1</min>
</pearinstaller>
<package>
<name>Packagename</name>
<channel>pear.php.net</channel>
</package>
<package>
<name>PackageName2</name>
<channel>pear.php.net</channel>
<min>1.0</min>
</package>
<package>
<name>ExtensionName</name>
<channel>pecl.php.net</channel>
<providesextension>ExtensionName</providesextension>
<conflicts/>
</package>
</required>
<optional>
<package>
<name>PackageName3</name>
<channel>pear.php.net</channel>
<min>1.0</min>
<exclude>1.0</exclude>
</package>
</optional>
</dependencies>

```

注意，属性`optional="yes"`和隐含的`optional="no"`都已从`<dep>`标签中提取出来，放入`<required>`和`<optional>`标签中。此外，`type`属性已提取到`<package>`、`<php>`和之前未定义的`<pearinstaller>`标签中。最后，`rel`和`version`属性已被一组新的标签完全取代。

在`package.xml 1.0`中，为了定义单个依赖项的版本集，需要多个`<dep>`标签：

```php
<dep type="pkg" rel="ge" version="1.2.3">PackageName</dep>
<dep type="pkg" rel="ne" version="1.3.0">PackageName</dep>
<dep type="pkg" rel="lt" version="2.0.0">PackageName</dep>

```

这可以与`package.xml 2.0`的等效版本进行比较：

```php
<package>
<name>PackageName</name>
<channel>pear.php.net</channel>
<min>1.2.3</min>
<max>2.0.0</max>
<exclude>1.3.0</exclude>
<exclude>2.0.0</exclude>
</package>

```

实际上，这个变化促进了在复杂的`package.xml`文件中更简单地调试依赖项问题。上述依赖项可以简单地翻译为“来自通道[pear.php.net](http://pear.php.net)的 PackageName 包，最小版本 1.2.3，最大版本 2.0.0，排除版本 1.3.0 和 2.0.0。”不仅使用像`xmllint`这样的工具检测错误更容易，而且对依赖项的复杂版本控制的理解也更容易。

`rel/version`对已被`<min>/<max>/<exclude>`标签的组合所淘汰。这三个标签可以被视为`rel`的`ge`、`le`和`ne`的新实现。

+   `<min>1.2.0</min>` = `<dep rel="ge" version="1.2.0">`

+   `<max>1.2.0</max>` = `<dep rel="le" version="1.2.0">`

+   `<exclude>1.2.0</exclude>` = `<dep rel="ne" version="1.2.0">`

使用这三个标签，我们可以在单个依赖项中有效地简单地定义任何版本集，无需考虑数学比较运算符。

#### 管理依赖项的信任度

PEAR 安装程序非常灵活，使得升级到包的新版本变得如此简单；直到 PEAR 版本 1.4.0，它的最大优势也是它的最大弱点。轻松升级包的能力，以及使用`upgrade-all`和`download-all`等命令执行批量自动升级的能力，是安装程序的关键卖点之一。然而，这种便利性建立在对新版本质量的隐含信任之上。通过依赖具有简单`<dep type="pkg" rel="ge" version="X.Y.Z">`的包，这给了依赖包的开发者一张空白支票。如果依赖包的开发者引入了与先前版本的不兼容，无论是由于疏忽还是对您的代码缺乏同情，您的最终用户就会遇到麻烦。因此，直接在代码中捆绑依赖项已经成为分发应用程序的首选方法。然而，这却否定了 PEAR 安装程序的主要好处——在发现严重错误，如功能故障，或更糟糕的是，可能导致网站遭受外部攻击的微妙安全漏洞时，能够快速轻松地升级。这也将维护捆绑依赖项的负担直接放在了应用程序维护者身上，降低了分布式开发的效率及其所有相关好处。

`package.xml 2.0`通过引入新的依赖概念：推荐版本，简单地优雅地解决了这个困境。这个依赖可以从 PEAR 的`package.xml`中获取：

```php
<package>
<name>Console_Getopt</name>
<channel>pear.php.net</channel>
<min>1.2</min>
</package>

```

这指示安装程序可以随意升级`Console_Getopt`到任何版本的`Console_Getopt`，版本`1.2`或更高。通过更改依赖项：

```php
<package>
<name>Console_Getopt</name>
<channel>pear.php.net</channel>
<min>1.2</min>
<recommended>1.2</recommended>
</package>

```

这一切都改变了。现在，安装程序不会在发布时自动升级到 1.3.0 版本，除非传递了`--force`或`--loose`选项，或者`Console_Getopt`的发布中存在`<compatible>`标签。

`<compatible>`的语法很简单：

```php
<compatible>
<name>ParentPackage</name>
<channel>pear.example.com</channel>
<min>1.0.0</min>
<max>1.3.0</max>
<exclude>1.1.2</exclude> [optional]
</compatible>

```

在依赖项中，`<min>/<max>/<exclude>`用于定义一组包保证与之兼容的版本。与依赖项不同，`<max>`标签是必需的，以限制版本集。此外，版本集必须限制到实际存在的、经过测试的发布版本。

从理论上讲，懒惰的开发者仍然有可能实现一个与旧`rel="ge"-based`依赖管理技术一样危险的新`<compatible>`标签，但由于开发者惰性原则，这不太可能，而且对于依赖该依赖的应用的开发者来说，也容易捕捉和纠正。

### 小贴士

**开发者惰性原则**

开发者，就像电子一样，会选择阻力最小的路径来实现他们的目标。让编写糟糕的代码变得困难，让编写好的代码变得容易，开发者就会编写好的代码。

#### 可靠地依赖 PECL 包

由于两个创新，可靠地依赖 PECL 包变得可能：

1.  将 `pear` 脚本拆分为两个脚本：`pear` 和 `pecl`

1.  `<providesextension>` 标签的引入

`pear` 和 `pecl` 命令之间的主要区别可以概括为：“使用 `pear` 来管理像 [pear.php.net](http://pear.php.net) 上的那样用 PHP 编写的包，并使用 `pecl` 来管理扩展 PHP 的包，如 [pecl.php.net](http://pecl.php.net) 上的那样。”从技术上讲，`pecl` 命令禁用了 `php.ini`，并默认使用 [pecl.php.net](http://pecl.php.net) 通道，但除此之外，它与 `pear` 命令相同。禁用 `php.ini` 使得升级 `php.ini` 内部的扩展成为可能，而无需卸载它们。

`<providesextension>` 标签在依赖项中的使用如下：

```php
<package>
<name>PDO</name>
<channel>pecl.php.net</channel>
<min>1.0</min>
<providesextension>PDO</providesextension>
</package>

```

这个简单的添加指示安装程序将包对 `PDO` 的依赖视为这两个依赖的组合：

```php
<extension>
<name>PDO</name>
<min>1.0</min>
</extension>
<package>
<name>PDO</name>
<channel>pecl.php.net</channel>
<min>1.0</min>
</package>

```

安装程序将首先检查内存中是否存在 `PDO` 扩展，版本为 1.0 或更高版本，如果不存在，它将检查包注册表以查看是否安装了 [pecl.php.net/PDO](http://pecl.php.net/PDO) 版本 1.0 或更高版本。这允许，例如，使用基于 CLI 的 PEAR 安装程序安装基于 Apache 的 PEAR 安装的生产扩展，而无需要求基于 CLI 的安装程序将每个扩展加载到其 `php.ini` 中。此外，它允许像 `PDO_mysql` 这样的扩展依赖于 `PDO`，而无需在内存中加载 `PDO` 扩展，极大地简化了最终用户的使用体验。

### 发布说明

在 `package.xml 1.0` 和 `package.xml 2.0` 中，发布说明由 `<notes>` 标签定义。此标签的格式为任何文本。

```php
<notes>
Minor bugfix release
These bugs are fixed:
* Bug #1234: stupid politicians elected to office
* Bug #2345: I guess we voted for them didn't we
</notes>

```

### 发布许可

`<license>` 标签的文本不会进行验证——可以输入任何内容。

```php
<license>BSD license</license>

```

在 `package.xml 2.0` 中，有可选的 `uri` 和 `filesource` 属性，用于将许可证链接到在线版本，以及链接到包本身内的特定许可证文件。

```php
<license uri="http://www.opensource.org/licenses/bsd-license.php">BSD
license</license>

```

### 变更日志

`package.xml 1.0` 中的变更日志格式与 `<release>` 标签的格式相匹配，唯一的区别是 `<filelist>` 不允许。变更日志纯粹是供人类使用的纯信息，安装程序不会对其进行任何处理。`package.xml 2.0` 中的变更日志格式与 `package.xml 1.0` 非常相似，以下是一个示例：

```php
<release>
<version>
<release>1.3.6</release>
<api>1.3.0</api>
</version>
<stability>
<release>stable</release>
<api>stable</api>
</stability>
<date>2005-08-18</date>
<license uri="http://www.php.net/license">PHP License</license>
<notes>
* Bump XML_RPC dependency to 1.4.0
* return by reference from PEAR::raiseError()
</notes>
</release>

```

与 `package.xml` 的发布部分一样，`<version>`、`<stability>`、`<date>`、`<license>` 和 `<notes>` 标签都存在。这与 `package.xml 1.0` 相同。唯一的区别是 `<version>`、`<stability>` 和 `<license>` 标签的格式，它们与 `package.xml 2.0` 中所做的更改相匹配。

### 文件列表，或包的内容

PEAR 的主要目的，以及由此产生的 `package.xml`，是分发包含编程代码的文件。这最终由包中定义的文件列表控制，由 `<filelist>` 或 `<contents>` 定义。

对于 `package.xml 1.0:`

```php
<filelist>
<dir name="OS">
<file role="php" name="Guess.php">
<replace from="@package_version@" to="version" type="package-
info" />
</file>
</dir>
<dir name="PEAR">
<dir name="ChannelFile" baseinstalldir="Foo">
<file name="Parser.php" role="php">
<replace from="@package_version@" to="version" type="package-
info" />
</file>
</dir>
<file role="data" name="Foo.dat"/>
</dir>
<file role="doc" name="linux-Howto.doc" install-as="README"
platform="!windows"/>
<file role="doc" name="windows-Howto.doc" install-as="README"
platform="windows"/>
</filelist>

```

对于 `package.xml 2.0:` 

```php
<contents>
<dir name="/">
<dir name="OS">
<file name="Guess.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info"/>
</file>
</dir>
<dir name="PEAR">
<dir name="ChannelFile" baseinstalldir="Foo">
<file name="Parser.php" role="php">
<tasks:replace from="@package_version@" to="version"
type="package-info"/>
</file>
</dir>
<file role="data" name="Foo.dat"/>
</dir>
<file role="doc" name="linux-Howto.doc"/>
<file role="doc" name="windows-Howto.doc"/>
</dir>
</contents>

```

这个标签是 `package.xml` 文件的核心。一旦我们可靠的包通过了依赖性测试，这就是 `package.xml` 在安装过程中实际使用的部分。

`package.xml` 内的文件列表用于定义发布中的目录结构。它必须精确反映文件在开发者计算机上的相对位置。换句话说，如果 `package.xml` 在 `/home/frank/mypackage/` 目录中，并且 `package.xml` 中的一个文件位于 `/home/frank /mypackage/foo/test.php`，它必须列出为：

```php
<file role="php" name="foo/test.php"/>

```

或者，作为替代：

```php
<dir name="foo">
<file role="php" name="test.php"/>
</dir>

```

任何一种选择都会得到相同的结果。在安装时，PEAR 会自动将像第二个例子那样的所有递归目录树转换成像第一个例子那样的单个扁平分支。

# package.xml 中的新标签

`package.xml 2.0` 引入了新的标签 `<phprelease>`, `<extsrcrelease>` 和 `<extbinrelease>`，以区分 PEAR 安装器处理的包的不同类型。`package.xml 2.1` 引入了 `<zendextsrcrelease>` 和 `<zendextbinrelease>`，以便区分常规 PHP 扩展和像 `xdebug` ( [`pecl.php.net/xdebug`](http://pecl.php.net/xdebug)) 这样的 Zend 扩展。

你可能已经注意到，`package.xml 1.0` 中的主要标签被命名为 `<filelist>`，而 `package.xml 2.0` 中的主要标签被命名为 `<contents>`。这种变化是由于一个简单的功能请求而产生的。当 PEAR 1.3.3 流行时，定制安装的需求增长；越来越多的属性和信息被塞进了 `<file>` 标签。`platform` 属性告诉安装器一个文件应该只安装在一个特定的平台上，例如 UNIX 或 Windows。这在 PEAR 包中用于在 UNIX 上使用 shell 脚本安装 `pear` 命令，在 Windows 上使用 `.bat` 批处理文件。提交的功能请求是实施一个额外的 `platform="!windows"` 漏洞，告诉安装器除了 Windows 之外的所有平台上安装文件。这引入了一系列问题。随着包的复杂性增加，支持更多的系统，可能需要指定一个文件可以安装的有限系统列表，或者在不同系统上安装文件的不同名称。实施这一点将需要在 `install-as` 和 `platform` 属性之间进行复杂的映射，这是 `package.xml 1.0` 的设计者没有预料到的，并且会在单个文件标签中引入一个糟糕的混乱。想象一下遇到这个噩梦并试图调试它：

```php
<file name="Foo.scr" role="script" install-
as="windows=Foo.bat;Darwim=Fooscr;unix=Foo;"
platform="windows;!Solaris;unix"/>

```

你是否看到了安装为属性中埋藏的错别字？

有一次，在处理由这些复杂特性引起的难题时，我突然意识到像`platform`属性这样的属性实际上是实现特定文件依赖关系的笨拙方式。突然，答案变得清晰。比扩展属性的意义更好的是，将信息抽象成单独的发布标签。每个发布标签都会有一个文件列表和安装条件，这将定义在最终用户的计算机上应该使用哪一个。例如，而不是：

```php
<file role="doc" name="linux-Howto.doc" install-as="README"
platform="!windows"/>
<file role="doc" name="windows-Howto.doc" install-as="README"
platform="windows"/>

```

我们将会有：

```php
<file role="doc" name="linux-Howto.doc"/>
<file role="doc" name="windows-Howto.doc"/>
...
<phprelease>
<installconditions>
<os>windows</os>
</installconditions>
<filelist>
<install name="windows-Howto.doc" as="README"/>
<ignore name="linux-Howto.doc"/>
</filelist>
</phprelease>
<phprelease>
<filelist>
<install name="linux-Howto.doc" as="README"/>
<ignore name="windows-Howto.doc"/>
</filelist>
</phprelease>

```

然而，最明显的益处来自于那个糟糕的“Darwin”示例。这会翻译成：

```php
<file name="Foo.scr" role="script" install-
as="windows=Foo.bat;Darwim=Fooscr;unix=Foo;"
platform="windows;!Solaris;unix"/>

```

到：

```php
<file name="Foo.scr" role="script"/>
...
<phprelease>
<installconditions>
<os>windows</os>
</installconditions>
<filelist>
<install name="Foo.scr" as="Foo.bat"/>
</filelist>
</phprelease>
<phprelease>
<installconditions>
<os>Darwim</os>
</installconditions>
<filelist>
<install name="Foo.scr" as="Fooscr"/>
</filelist>
</phprelease>
<phprelease>
<installconditions>
<arch>Solaris</arch>
</installconditions>
<filelist>
<ignore name="Foo.scr"/>
</filelist>
</phprelease>
<phprelease>
<installconditions>
<os>unix</os>
</installconditions>
<filelist>
<install name="Foo.scr" as="Foo"/>
</filelist>
</phprelease>

```

在这里，简单的验证的潜力是明显的：`<os>`安装条件限制为几个已知的可能性，如`windows、unix、linux、Darwin`等。操作系统“Darwim”将简单地拒绝验证。此外，如何处理`Foo.scr`的复杂性是根据操作系统分组，而不是放入几个具有易出错的非 XML 语法的属性中。

# 文件/目录属性：name、role 和 baseinstalldir

`<file>`和`<dir>`标签都有许多可用的选项。这两个标签都需要一个`name`属性，用于定义元素在磁盘上的名称。与操作系统不同，`package.xml`不允许空目录。所有`<dir>`标签都必须包含至少一个`<file>`标签。如前所述，在`package.xml`中描述文件位置有两种方式，要么使用完整的相对路径，由 UNIX 路径分隔符`/:`分隔。

```php
<file role="php" name="foo/test.php"/>

```

或者，也可以这样：

```php
<dir name="foo">
<file role="php" name="test.php"/>
</dir>

```

所有文件都必须有一个**角色**属性。此属性告诉安装程序如何处理文件。允许的文件角色默认列表如下：

| 默认文件角色 |   |
| --- | --- |
| 角色 | 描述 |
| --- | --- |
| php | PHP 脚本文件，例如"PEAR.php" |
| 数据 | 脚本使用的数据文件（只读） |
| doc | 文档文件 |
| test | 测试脚本，单元测试文件 |
| script | 可执行脚本文件（例如 pear.bat, pear.sh） |
| ext | PHP 扩展二进制文件（例如 php_mysql.dll） |
| src | PHP 扩展源文件（例如 mysql.c） |

每个文件角色都与其关联一个配置值。例如，在我的 Windows XP 系统上，当我列出配置值时，我看到如下内容：

```php
C:\>pear config-show
CONFIGURATION (CHANNEL PEAR.PHP.NET):
=====================================
Auto-discover new Channels auto_discover <not set>
Default Channel default_channel pear.php.net
HTTP Proxy Server Address http_proxy <not set>
PEAR server [DEPRECATED] master_server pear.php.net
Default Channel Mirror preferred_mirror pear.php.net
Remote Configuration File remote_config <not set>
PEAR executables directory bin_dir C:\php4
PEAR documentation directory doc_dir C:\php4\PEAR\docs
PHP extension directory ext_dir C:\php4\extensions
PEAR directory php_dir C:\php4\PEAR
PEAR Installer cache directory cache_dir C:\DOCUME~1\GREGBE~1\LOCALS~1\
Temp\pear\cache
PEAR data directory data_dir C:\php4\PEAR\data
PHP CLI/CGI binary php_bin C:\php-4.3.8\cli\php.exe
PEAR test directory test_dir C:\php4\PEAR\tests
Cache TimeToLive cache_ttl 3600
Preferred Package State preferred_state stable
Unix file mask umask 0
Debug Log Level verbose 1
PEAR password (for password <not set> maintainers)
Signature Handling Program sig_bin c:\gnupg\gpg.exe
Signature Key Directory sig_keydir C:\php4\pearkeys
Signature Key Id sig_keyid <not set> Package
Signature Type sig_type gpg
PEAR username (for username <not set> maintainers)
User Configuration File Filename C:\php4\pear.ini
System Configuration File Filename C:\php4\pearsys.ini

```

在某些情况下，特别是对于像`HTML_QuickForm_Controller`（[`pear.php.net/package/HTML_QuickForm_Controller`](http://pear.php.net/package/HTML_QuickForm_Controller)）这样的子包，所有文件都应该安装到子目录中（在我们的例子中是 HTML/QuickForm/Controller）。为了在我们的文件中反映这个安装路径，我们需要将所有文件的前缀设置为完整路径，如下所示：

```php
<file role="php" name="HTML/QuickForm/Controller.php"/>
<file role="php" name="HTML/QuickForm/Controller/Action.php"/>

```

或者，也可以这样使用`<dir>`标签：

```php
<dir name="HTML">
<dir name="QuickForm">
<dir name="Controller">
<file role="php" name="Action.php"/>
</dir>
<file role="php" name="Controller.php"/>
</dir>
</dir>

```

此外，我们的实际开发路径也需要反映这一点。这并不符合“让懒惰的开发者一切变得简单”的开发模式，因此 PEAR 提供了`baseinstalldir`属性来简化事情。现在，我们只需要：

```php
<file role="php" baseinstalldir="HTML/QuickForm" name=" Controller.php"/>
<file role="php" baseinstalldir="HTML/QuickForm" name=" Controller/Action.php"/>

```

或者，更常见的：

```php
<dir name="/" baseinstalldir="HTML/QuickForm">
<dir name="Controller">
<file role="php" name="Action.php"/>
</dir>
<file role="php" name="Controller.php"/>
</dir>

```

### 小贴士

**开发机上的路径位置**

注意，作为一个开发者，为了使这生效，磁盘上的实际路径必须与`<dir>/<file>`标签匹配。在我们上面的例子中，文件应该位于：

`Controller/Action.php Controller.php package.xml`

而不是：

`HTML/QuickForm/Controller/Action.php HTML/QuickForm/Controller.php package.xml`

否则，PEAR 安装程序将无法找到这些文件。

一般而言，如果一个文件在`package.xml`中被列出：

```php
<file role="php" name="Path/To/Foo.php"/>

```

并且`php_dir`是`C:\php4\PEAR`，文件将被安装到`C:\php4\PEAR\Path\To\Foo.php`。这并不总是有优势，尤其是对于像脚本这样的东西。例如，PEAR 将所有脚本放在`scripts/`子目录中以方便组织。然而，这意味着如果`bin_dir`是`C:\php4`，

```php
<file role="script" name="scripts/pear.bat"/>

```

将被安装到`C:\php4\scripts\pear.bat`，这不一定在路径中。要在 PEAR 1.4.x 中更改基本安装目录，您需要在发布标签内使用`<install>`标签，此外，还可以对文件内容执行文本转换。

# 摘要

在本章中，我们通过`package.xml`的结构，了解了 PEAR 安装程序内部工作原理的基础。首先，我们探讨了 PEAR 安装程序的基本设计理念，以及 PEAR 包与传统的解压缩并运行方法的不同。我们学习了 PEAR 的配置选项，以及 PEAR 处理库与应用程序的灵活方式。

接下来，我们探讨了版本控制对控制安装包质量的重要性，以及依赖关系的重要性，以及 PEAR 安装程序如何管理库和应用程序之间的重要链接。然后，我们探讨了使用 PEAR 安装程序升级的便利性，与传统的解压缩并运行应用程序的升级方式相比。

之后，我们一头扎进了`package.xml`的结构，学习如何组织包元数据，如包名、作者、发布说明和变更日志。这伴随着对关键安装数据（如文件、依赖关系和版本控制）如何组织的考察。

下一章将探讨高级主题，特别是`package.xml 2.0`如何引入更好的应用程序支持，以及如何在您的包中利用这些新特性。
