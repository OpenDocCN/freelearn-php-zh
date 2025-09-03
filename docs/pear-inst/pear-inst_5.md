# 第五章。向世界发布：PEAR 渠道

PEAR 版本 1.4.0 及更高版本的一个主要特性是能够公开分发自己的应用程序，以便使用 PEAR 安装程序进行安装。尽管在 PEAR 1.3.6 及更早版本中这是可行的，但难度很大，因此很少尝试。PEAR 版本 1.4.0+ 通过使用一种新的分发介质称为 **渠道** 来简化包的分发。每个 PEAR 渠道提供一组独特的包，可以使用 PEAR 安装程序轻松安装。例如，要从 `pear.chiaraquartet.net` 渠道安装一个包，只需输入：

```php
$ pear channel-discover pear.chiaraquartet.net
$ pear install chiara/Chiara_PEAR_Server 

```

在过去，这根本不可能实现。安装来自 `pear.chiaraquartet.net:` 的包需要一组非直观的按键操作。

```php
$ pear config-set master_server pear.chiaraquartet.net
$ pear install Chiara_PEAR_Server
$ pear config-set master_server pear.php.net 

```

复杂性被跨渠道依赖所加剧。`pear.chiaraquartet.net/Chiara_PEAR_Server` 包依赖于 `pear.php.net/HTML_QuickForm`，因此实际上需要按照以下顺序：

```php
$ pear install HTML_QuickForm
$ pear config-set master_server pear.chiaraquartet.net
$ pear install Chiara_PEAR_Server
$ pear config-set master_server pear.php.net 

```

在升级时，需要重复相同的流程，这不仅带来了痛苦的回忆需求（“我上次是从哪里得到这个 Chiara_PEAR_Server 的？”），还增加了出错的机会。如果 `pear.chiaraquartet.net` 恰好提供了一个名为 `LogXML` 的包，而 `pear.php.net` 也引入了一个包，那么如果你不小心输入了：

```php
$ pear upgrade LogXML 

```

如果没有必要的：

```php
$ pear config-set master_server pear.chiaraquartet.net
$ pear upgrade LogXML
$ pear config-set master_server pear.php.net 

```

你可能会无意中升级到错误的包！渠道消除了所有这些麻烦，并且以严格的安全措施来实现。

那么，如何设置你自己的渠道？本章将探讨安装 Chiara_PEAR_Server 所需的步骤，以及用于记录渠道独特特征的渠道定义文件的架构。

此外，我们还将学习如何按用户分发定制的 PEAR 应用程序，即使是付费使用应用程序。我们将发现用于为用户浏览网页提供公共入口的 **Crtx_PEAR_Channel_Frontend** 包，最后，我们将讨论安全问题。

# 分发基于 package.xml 的包

分发包有两种方式（任选其一）：

+   渠道服务器

+   静态压缩包

你将在本节中了解这两种方法。

在 PEAR 1.4.0 版本发布之前，用户需要输入：

```php
$ pear remote-list 

```

PEAR 安装程序将使用 XML-RPC 发送请求，请求 `package.listAll` 方法到 [`pear.php.net/xmlrpc.php`](http://pear.php.net/xmlrpc.php)。同时，在 `pear.php.net`，将查询所有包、发布和依赖关系的数据库以获取数据（或访问服务器端缓存），然后将其动态编码成 XML-RPC 响应，在用户端解码，并转换成一个包含所有包及其发布的 PHP 数组。然后，这些信息将被格式化为一个漂亮的包名称列表，并显示在屏幕上。

### 小贴士

**XML-RPC** 代表 **XML 远程过程调用**，是一种协议，允许程序像在本地机器上实现一样在远程服务器上调用函数。

**SOAP**（直到最近它被称为 **简单对象访问协议**，但现在只是 "SOAP"，因为开发者意识到它并不简单，而且这个名字也很容易引起混淆）是同一想法的更复杂实现。

从最终用户的角度来看，使用 PEAR 安装程序安装远程包有两种方式。第一种方式是安装一个抽象包，如下所示：

```php
$ pear install PEAR
$ pear install PEAR-stable
$ pear install PEAR-1.4.3
$ pear install channel://pear/PEAR
$ pear install channel://pear.php.net/PEAR-1.4.3 

```

这些示例中的每一个都将用户传入的信息转换为从频道服务器（在这种情况下为 `pear.php.net`）检索文件的实际、现有 URL，然后下载该包进行安装。实际上，在撰写本章时，这些示例本质上都转换为以下内容：

```php
$ pear install http://pear.php.net/get/PEAR-1.4.3.tgz 

```

安装包的第二种方式是直接指定安装的 URL，如上所述。

这两种方法对最终用户来说看起来是相同的。然而，在幕后，它们有显著的不同。通过通过像 `PEAR` 或 `PEAR-stable` 这样的抽象包下载和安装包时，可以在下载单个文件之前验证所有依赖关系，从而在安装过程中最慢的部分——下载——中节省大量时间。当通过静态 URL（[`pear.php.net/get/PEAR-1.4.3.tgz`](http://pear.php.net/get/PEAR-1.4.3.tgz)）安装时，在执行任何依赖关系验证之前，必须下载整个包，这可能会导致带宽浪费。

为了将抽象包请求转换为实际的物理 URL，需要从远程频道服务器检索少量信息。这些信息用于在下载完整包之前验证依赖关系，并根据用户的需求确定要下载的包的正确版本。

例如，以下调用首先检索按版本号和稳定性组织的所有 PEAR 版本列表：

```php
$ pear upgrade PEAR-stable 

```

假设服务器返回如下列表：

| 版本 | 稳定性 |
| --- | --- |
| 1.5.0a1 | alpha |
| 1.4.3 | stable |
| 1.4.2 | stable |
| 1.4.1 | stable |
| 1.4.0 | stable |
| 1.4.0RC1 | beta |
| 1.4.0a14 | alpha |

PEAR 安装程序将检查版本 1.5.0a1，这是可用的最新版本，并确定它不够稳定以进行安装。接下来，它将检查版本 1.4.3，并且（假设已安装的版本是 1.4.2 或更早版本）确定这是应该下载的版本。接下来，它将查询服务器并检索版本 1.4.3 的依赖关系列表，这类似于以下内容：

| 依赖类型 | 依赖名称（如果有） | 依赖版本要求 |
| --- | --- | --- |
| PHP |   | 4.2.0 或更高版本 |
| PEAR 安装程序 |   | 1.3.3 1.3.6, 1.4.0a12 或更高版本 |
| 包 | Archive_Tar | 1.3.1 或更高版本（推荐 1.3.1） |
| 包 | Console_Getopt | 1.2 或更高版本（推荐使用 1.2） |
| 包 | XML_RPC | 1.4.3 或更高版本（推荐使用 1.4.3） |
| 冲突包 | PEAR_Frontend_Web | 0.4 或更早版本 |
| 冲突包 | PEAR_Frontend_Gtk | 0.3 或更早版本 |
| 远程安装程序组 | PEAR_RemoteInstaller | 0.1.0 或更高版本 |
| 网页安装程序组 | PEAR_Frontend_Web | 0.5.0 或更高版本 |
| Gtk 安装程序组 | PEAR_Frontend_Gtk | 0.4.0 或更高版本 |

在下载 PEAR 1.4.3 以安装它之前，PEAR 安装程序将使用这些信息来确定该包是否与现有已安装的包兼容，以及 PHP 和 PEAR 安装程序的运行版本。只有当所有检查都通过时，PEAR 安装程序才会继续下载/安装。

此外，由于 PEAR 1.4.3 的 `package.xml` 版本为 2.0，当从 PEAR 1.4.0 或更高版本升级时，所需的依赖项列表也将自动下载并安装。

您可能会问自己，我如何分发我的应用程序和库，以利用 PEAR 安装程序内置的强大功能和优雅性？答案出乎意料地简单，正如自 PEAR 版本 1.4.0 发布以来频道服务器数量的小幅激增所证明的那样，例如 eZ components（http://www.ez.no），以及流行的 pearified 频道（[`www.pearified.com`](http://www.pearified.com)）。**Chiara_PEAR_Server** 包是一个完全功能的 PEAR 频道服务器，可以从 `pear.chiaraquartet.net` 频道服务器安装。

### 小贴士

最初，Chiara_PEAR_Server 的名字是 PEAR_Server。意图是在代码足够稳定时，向 `pear.php.net` 仓库提出一个名为 "PEAR_Server" 或 "PEAR_Channel_Server" 的包。然而，在此之前，存在潜在的命名冲突（通常 PEAR 保留用于源自 `pear.php.net` 的包），因此只要它从 `pear.chiaraquartet.net` 分发，包将被命名为 Chiara_PEAR_Server。

一旦 Chiara_PEAR_Server 运行起来（前提条件包括一个有效的 PEAR 安装、MySQL 服务器以及 PHP 5.0.0 或更高版本，并带有 mysql 或 mysqli 扩展），您也可以考虑安装 Davey Shafik 的公共前端 **Crtx_Channel_PEAR_Server_Frontend**，该前端可通过 `crtx.org` 频道获取。这一点将在本章后面的 *配置服务器；为最终用户提供前端* 部分中讨论。

# 通过频道服务器分发包

当通过频道服务器分发包时，PEAR 安装程序需要一些信息来确定要安装哪些包。最重要的是如何与频道服务器通信。服务器是否期望接收 XML-RPC 请求，或者支持 REST？实现了哪些 XML-RPC 函数，提供了哪些 REST 信息？是否有可用的镜像？是否有任何自定义包验证要求？

所有这些问题都通过简单的`channel.xml`结构得到解答。在安装 Chiara_PEAR_Server 之前，了解该包的基础非常重要，因为它将使快速启动和运行成为可能。

## `channel.xml`文件

一个通道为了存在，首先需要的是一个`channel.xml`文件。在 XSchema 格式中，`channel.xml`的官方定义可以在[`pear.php.net/dtd/channel-1.0.xsd`](http://pear.php.net/dtd/channel-1.0.xsd)找到。`channel.xml`文件*必须*命名为`channel.xml`，并且必须位于通道的根目录中；否则，PEAR 安装程序的自动发现机制将无法工作。例如，`pear.php.net`的通道定义文件位于[`pear.php.net/channel.xml`](http://pear.php.net/channel.xml)，而`pear.chiaraquartet.net`的通道定义文件位于[`pear.chiaraquartet.net/channel.xml`](http://pear.chiaraquartet.net/channel.xml)。

此文件允许 PEAR 安装程序快速有效地确定通道服务器提供的功能，而不会浪费任何带宽。一个`channel.xml`文件必须定义通道名称（其服务器主机名和路径）、通道目的的简要说明，以及用于检索安装目的的包信息的元数据。此外，`channel.xml`文件允许显式定义通道镜像，首次使镜像通道存储库成为可能。

这里是一个包含每个可能标签的示例`channel.xml`文件：

```php
<?xml version="1.0" encoding="ISO-8859-1"?>
<channel version="1.0" 

xsi:schemaLocation="http://pear.php.net/channel-1.0
http://pear.php.net/dtd/channel-1.0.xsd">
<name>pear.example.com</name>
<summary>Example channel</summary>
<suggestedalias>example</suggestedalias>
<validatepackage version="1.0">
Example_Validate_Package</validatepackage>
<servers>
<primary ssl="yes" port="81">
<xmlrpc path="myxmlrpc.php">
<function version="1.0">logintest</function>
<function version="1.0">package.listLatestReleases</function>
<function version="1.0">package.listAll</function>
<function version="1.0">package.info</function>
<function version="1.0">package.getDownloadURL</function>
<function version="1.1">package.getDownloadURL</function>
<function version="1.0">package.getDepDownloadURL</function>
<function version="1.1">package.getDepDownloadURL</function>
<function version="1.0">package.search</function>
<function version="1.0">channel.listAll</function>
</xmlrpc>
<soap path="soap.pl">
<function version="1.0">customSoapFunction</function>
</soap>
<rest>
<baseurl type="REST1.0">http://pear.example.com/rest/</baseurl>
<baseurl type="REST1.1">http://pear.example.com/rest/</baseurl>
</rest>
</primary>
<mirror host="poor.example.com" port="80" ssl="no">
<xmlrpc>
<function version="1.0">logintest</function>
<function version="1.0">package.listLatestReleases</function>
<function version="1.0">package.listAll</function>
<function version="1.0">package.info</function>
<function version="1.0">package.getDownloadURL</function>
<function version="1.1">package.getDownloadURL</function>
<function version="1.0">package.getDepDownloadURL</function>
<function version="1.1">package.getDepDownloadURL</function>
<function version="1.0">package.search</function>
<function version="1.0">channel.listAll</function>
</xmlrpc>
<soap path="soap.php">
<function version="1.0">customSoapFunction</function>
</soap>
<rest>
<baseurl type="REST1.0">http://poor.example.com/rest/</baseurl>
<baseurl type="REST1.1">http://poor.example.com/rest/</baseurl>
</rest>
</mirror>
</servers>
</channel>

```

快速浏览`channel.xml`文件可以揭示大量信息，这些信息以非常简单的格式呈现。通过此文件，我们告诉 PEAR 安装程序是否使用安全连接，如何访问包元数据（使用 XML-RPC、SOAP 或 REST），以及用户如何访问/使用通道（建议别名、验证包）。

### 小贴士

**PEAR 安装程序支持 SOAP 吗？**

很抱歉打破你的幻想，但 SOAP 对于 PEAR 安装程序所需的相对简单的远程通信并不是必需的；所以，SOAP 没有被实现。然而，如果将来有需要，或者通道希望宣传自定义的 SOAP 方法，`channel.xml`规范支持 SOAP。

然而，这仅应用于通知客户存在一个**WSDL（Web 服务描述语言**）文件，因为这种格式比`channel.xml`丰富得多。

一个通道可以位于主机名的根目录（**pear.example.com**）或子目录（pear.example.com/subdirectory）。请注意，`pear.example.com`与`pear.example.com/subdirectory`是不同的通道。用户将按照以下方式从`pear.example.com/subdirectory`通道安装包：

```php
$ pear install pear.example.com/subdirectory/Packagename 

```

其他包将依赖于来自`pear.example.com/subdirectory`通道的包，其中包含一个类似于下面的`package.xml`标签：

```php
<dependencies>
<required>
...
<package>
<name>Packagename</name>
<channel>pear.example.com/subdirectory</channel>
</package>
</required>
</dependencies>

```

## `channel.xml`标签摘要

通道的`<summary>`应该是对通道的单行描述，例如：“PHP 扩展和应用仓库”。

通道的`<suggestedalias>`是用户可以在命令行中使用的简称。

例如，`pear.php.net`通道的建议别名是`pear`，`pecl.php.net`通道的建议别名是`pecl`，而`pear.chiaraquartet.net`通道的建议别名是`chiara`。这些别名可以用来快速安装包，例如：

```php
$ pear install pear/DB
$ pear install chiara/Chiara_PEAR_Server 

```

别名是建议的别名，因为最终用户有通过`channel-alias`命令重新定义别名的选项：

```php
$ pear channel-alias pear.chiaraquartet.net c 

```

这将允许快速安装包：

```php
$ pear install c/Chiara_PEAR_Server 

```

### 小贴士

你不能在`package.xml`文件的依赖关系部分使用通道建议的别名。你必须使用通道的全名。

通道的验证包（由`<validatepackage>`标签控制）由安装程序用于执行针对通道的特定定制验证。默认验证（在 PEAR 包的`PEAR/Validate.php`文件中找到）在版本和包命名方面非常严格，并试图实现针对`pear.php.net-based`包的特定编码标准。这些规则比`pecl.php.net`通道实施的规则更严格，因此`pecl.php.net`包使用 PEAR 包的`PEAR/Validate/PECL.php`文件中找到的定制通道验证器进行验证。

大多数外部于`pear.php.net`的通道都希望复制`pecl.php.net`通道的`channel.xml`文件，并使用`PEAR_Validate_PECL`验证包。

自定义通道验证器必须提供一个与路径匹配的类（`PEAR/Validate/PECL.php`提供了`PEAR_Validate_PECL`类），并且包名必须与类名相同。此外，该类必须扩展`PEAR_Validate`，并实现使用`validate*()`方法（如`validateVersion()`、`validatePackage()`、`validateSummary()`等）的验证。此外，该类*必须*是来自通道本身的包分发的，除非该类已经加载到内存中。

### 小贴士

默认验证类`PEAR_Validate`和 PECL 验证类`PEAR_Validate_PECL`将始终可用，供通道作为自定义验证包使用。

要使用`PEAR_Validate_PECL`类，只需将此行添加到`channel.xml`中：

`<validatepackage version="1.0"> PEAR_Validate_PECL</validatepackage>`

`channel.xml`通道定义文件最重要的部分是`<servers>`标签。这是 PEAR 安装程序确定如何连接到通道（通过 REST 或 XML-RPC）以及是否有可用镜像的地方。

主要通道服务器（必须与通道名称相同）支持的协议在`<primary>`标签中定义。镜像通过`<mirror>`标签（在本节末尾描述）逻辑上定义。`<primary>`标签有几个可选属性：

+   **ssl** — 合法值是 *yes* 和 *no*。默认情况下，`ssl` 设置为 *no*。如果设置为 *yes*，则将通过安全套接字联系通道服务器。

+   **端口** — 合法值是任何正整数。默认情况下，`端口` 设置为 *80*，这是联系远程 Web 服务器的默认 HTTP 端口。所有来自通道的数据都通过 HTTP 传输，因此这是一个自然的选择。

    ### 小贴士

    虽然 REST 非常新，但它相对于 XML-RPC 有几个显著的优势。首先，REST 内容（如 PEAR 通道标准中实现的那样）都是静态文件。这意味着可以使用轻量级服务器如 thttpd 来为高流量站点提供服务内容。此外，像 [`www.pearified.com`](http://www.pearified.com) 和 [`www.pearadise.com`](http://www.pearadise.com) 这样的通道聚合器可以爬取您的通道并提供包的可搜索索引。

    由于遵循相同的设计原则，基于 REST 的通道镜像非常简单，可以使用一个简单的网络爬虫脚本来完成。

    此外，从 PEAR 1.4.3 版本开始，XML-RPC 支持在 PEAR 安装程序中是可选的，因此并非所有用户都会在客户端支持 XML-RPC。

PEAR 安装程序只识别少数几种协议。对于 XML-RPC，识别的函数有：

+   `logintest(1.0)` : 这个函数简单地返回 true

+   `package.listLatestReleases(1.0)` : 这个函数返回一个数组，按包名索引，包含其最新发布版本的文件大小、版本、状态和依赖项（如果有）。

+   `package.listAll(1.0)` : 这个函数返回一个包含其发布版本极端详细信息的包数组。

+   `package.info(1.0)` : 这个函数返回关于单个包的详细信息的数组。

+   `package.getDownloadURL(1.0)` : 这个函数返回一个包含关于发布版本简单信息和精确下载 URL 的数组。

+   `package.getDownloadURL(1.1)` : 与版本 1.0 类似，这个函数返回一个包含关于发布版本简单信息和精确 URL 的数组。此外，这个函数接受作为参数的当前安装的包版本以缩小搜索范围。

+   `package.getDepDownloadURL(1.0)` : 与 `package.getDownloadURL` 类似，这个函数返回关于发布版本的信息和下载该发布版本的精确 URL。然而，作为输入，它接受从 `package.xml` 解析出的依赖项。

+   `package.getDepDownloadURL (1.1)`: 与 `package.getDepDownloadURL` 类似，这个函数返回关于一个发布版本和下载该发布版本的精确 URL 的信息。它还接受作为参数的当前已安装的依赖版本。

+   `package.search(1.0)`: 与 `package.listAll` 类似，这个函数返回包含详细信息的包列表。然而，这个函数根据输入参数限制信息搜索。

+   `channel.listAll(1.0)`: 这个函数返回当前通道所知的通道的简单列表。

提供此详细信息仅用于信息目的，因为所有通道的 XML-RPC 支持都已弃用。

相反，您的通道应该支持基于 REST 的静态文件，这些文件传达了通过通道可用的类别、维护者、软件包和发布信息。在撰写本章时，PEAR 安装程序支持两种协议。第一种统称为 **REST1.0**，它由安装程序做出的路径相关假设和几个 XSchema 文件定义。

在 `channel.xml` 文件中，REST 使用 `<baseurl>` 标签声明，类似于以下内容：

```php
<baseurl type="REST1.0">http://pear.php.net/rest/</baseurl>

```

实际上，安装程序只需要这些信息来完全实现 REST。从这个信息中，安装程序能够构建任何必要的查询以确定远程信息。敏锐的读者可能已经注意到了他们与数据库工作中的一个熟悉词汇——查询——查询正是 PEAR 安装程序所做的事情；直接访问数据而不是通过像 SOAP 或 XML-RPC 这样的 API 包装器。与 XML-RPC 和基于 RPC 的 SOAP 所使用的程序性协议不同，REST 基于提供超链接数据或资源的原理，每个资源都有一个独特的 URL。 

PEAR 是一个不寻常的 REST 接口，因为它严格是只读的，但这更是使用 REST 的好理由。不仅安装程序可以抓取它想要的任何数据而不依赖于本质上有限的 API，我们还可以利用 HTTP 协议的一些更强大的功能，并在客户端实现 HTTP 缓存，节省大量带宽和时间，否则这些带宽和时间将用于下载冗余信息。

这也为客户端和服务器提供了固有的安全优势。客户端只是与静态 XML 文件工作，服务器无需从客户端接受任何输入。简而言之，REST 简直是从所有方面来看都是最佳选择。

当访问 REST1.0 时，PEAR 安装程序期望的路径结构如下：

```php
c/ [Categories] 
CategoryName1/
info.xml [information on the "CategoryName1" category] 
packages.xml [list of packages in the CategoryName1 category] 
CategoryName2/
info.xml [information on the "CategoryName2" category] 
packages.xml [list of packages in the CategoryName2 category] 
m/ [Maintainers] 
joe/
info.xml [information about maintainer "joe"] 
frank/
info.xml [information about maintainer "frank"] 
amy/
info.xml [information about maintainer "amy"]
p/ [Packages]
packages.xml [A list of all packages in this channel] 
PackageName1/
info.xml [information on "PackageName1" package] 
maintainers.xml [list of maintainers of this package] 
PackageName2/
info.xml [information on "PackageName2" package] 
maintainers.xml [list of maintainers of this package] 
PackageName3/
info.xml [information on "PackageName3" package] 
maintainers.xml [list of maintainers of this package] 
r/ [Releases]
PackageName1/ [Releases of package PackageName1] 
allreleases.xml [A brief list of all releases available]
1.0.0.xml [summary information about version 1.0.0] 
package.1.0.0.xml [the complete package.xml of this release] 
deps.1.0.0.txt [PHP-serialized dependencies of version 1.0.0] 
0.9.0.xml [summary information about version 0.9.0] 
deps.0.9.0.txt [PHP-serialized dependencies of version 0.9.0] 
package.0.9.0.xml [the complete package.xml of this release] 
...
...
latest.txt [the latest version number, in text format] 
stable.txt [the latest stable version number, in text format] 
beta.txt [the latest beta version number, in text format] 
PackageName2/ [Releases of package PackageName2] 
allreleases.xml [A brief list of all releases available] 
1.1.0.xml [summary information about version 1.1.0] 
deps.1.1.0.txt[PHP-serialized dependencies of version 1.1.0] 
package.1.1.0.xml [the complete package.xml of this release] 
1.0.4.xml [summary information about version 1.0.4] 
deps.1.0.4.txt[PHP-serialized dependencies of version 1.0.4] 
package.1.0.4.xml [the complete package.xml of this release] 
...
...
latest.txt [the latest version number, in text format] 
stable.txt [the latest stable version number, in text format] 
beta.txt [the latest beta version number, in text format] 
alpha.txt [the latest alpha version number, in text format] 
devel.txt [the latest devel version number, in text format] 

```

注意，PackageName3 没有发布版本，因此没有 REST 条目。

REST1.1 将这些文件添加到结构中：

```php
c/
categories.xml [list of all categories]
CategoryName1/
packagesinfo.xml [consolidated package/release info for the entire
category]
CategoryName2/
packagesinfo.xml [consolidated package/release info for the entire
category]
m/
allmaintainers.xml [list of all maintainers]
p/
r/

```

REST1.1 的主要目的是在不要求允许老式的目录爬行的情况下，允许对通道进行爬取，消除了所有 Web 服务器固有的潜在安全漏洞。

通道服务器镜像由 `<mirror>` 标签定义。此标签与 `<primary>` 标签相同，但它需要一个额外的属性，即 `host`。`host` 属性定义了用于联系镜像的 URL。

## 获取 Chiara_PEAR_Server

Chiara_PEAR_Server 软件包很容易获取。首先，您需要满足一些先决条件。Chiara_PEAR_Server 软件包需要：

+   PHP 5.0.0 或更高版本；建议使用 PHP 5.1.0 或更高版本

+   MySQL 数据库服务器

+   mysql 或 mysqli PHP 扩展

+   一个运行中的 Web 服务器，例如 Apache

+   PEAR 版本 1.4.3 或更高

要获取 PEAR 版本 1.4.3 或更高版本，如果您使用的是 PHP 版本 5.1.0，在 UNIX 系统上您需要做的只是：

```php
$ cd php-5.1.0
$ ./buildconf
$ ./configure
$ make cli
$ make install-pear 

```

这将自动安装和配置 PEAR。请注意，`configure` 命令接受大量选项，您可以通过 `./configure --help` 来了解它们。

注意，在 Web 服务器上安装 PHP 更为复杂，并且是安装 Chiara_PEAR_Server 软件包所必需的。如果您使用 Apache，只需将 `--with-apache` 或 `--with-apache2` 指令传递给 `configure` 即可，然后您就可以开始使用了。

在 Windows 上，对于版本 5.2.0 之前的所有 PHP 版本，您需要下载 PHP 的 `.zip` 版本，*而不是* `.msi` 版本。然后切换到您解压缩 PHP 的目录，并输入：

```php
go-pear 

```

回答提示并选择安装位置。在两种情况下，安装后，请务必升级：

```php
pear upgrade PEAR 

```

这将确保您拥有 PEAR 的最新稳定版本。

mysql 或 mysqli PHP 扩展的安装文档位于 [`www.php.net/mysql`](http://www.php.net/mysql)。

一切准备就绪后，您可以通过以下简单步骤获取 Chiara_PEAR_Server 软件包：

```php
$ pear channel-discover pear.chiaraquartet.net
$ pear up Chiara_PEAR_Server-alpha 

```

就这些了！请注意，Chiara_PEAR_Server 目前需要 `pear.php.net` 软件包 DB_DataObject 和 HTML_QuickForm，所以在尝试安装 Chiara_PEAR_Server 之前，请确保您有一个正常工作的互联网连接或者已经安装了这些软件包。

配置服务器需要运行一个安装后脚本，我们将在下一节中介绍。

## 配置服务器；为最终用户提供前端

在我们运行安装后脚本之前，了解它正常运行所需的内容是很重要的。

首先，我们需要创建一个 MySQL 用户，该用户将由安装后脚本用于初始化数据库和创建表。因此，此用户需要具有 `create` 和 `alter` 权限（使用 MySQL 内部的 `GRANT` 命令来完成此操作）。由于这将作为公共 Web 脚本使用的数据库用户，出于安全考虑，一旦数据库被正确初始化，最好移除 `create/alter` 权限。Chiara_PEAR_Server 管理后端在日常工作操作中所需的唯一权限是 `insert/delete/update`。请注意，在升级 Chiara_PEAR_Server 时，应暂时重新授予 `create/alter` 权限，以防对数据库进行修改或添加。

运行 Chiara_PEAR_Server 安装后脚本在 PEAR 中是一个简单的任务，您只需输入：

```php
$ pear run-scripts chiara/Chiara_PEAR_Server 

```

PEAR 安装程序将引导您完成几个问题。完成这些问题后，如果没有错误，通道服务器将准备就绪，可以运行。

![配置服务器；为最终用户提供前端](img/1904811191_5_01.jpg)

第一组问题将要求提供数据库连接信息，以及主通道管理员（您）的`Handle`和通道名称。您的`Handle`与`package.xml`文件中的 handle 或用户名相同，并且应该是一个单个小写单词。例如，您可以通过浏览[`pear.php.net/accounts.php`](http://pear.php.net/accounts.php)上的维护者列表来查看 handle 的选择示例。您的通道名称必须与服务器相同。例如，如果您在本地主机上设置测试服务器，您的通道名称必须是`localhost`。

![配置服务器；为最终用户获取前端](img/1904811191_5_02.jpg)

接下来的一组问题将涉及您通道的基本信息。此时，您应该已经想到了一个服务器名称。一般来说，如果通道名称与其提供的内容有直接关联，用户更容易记住。例如，如果您的通道提供财务软件，那么像`software.companyname.com`这样的通道名称可能很好，别名可以是`companysoftware`。

在此之后，将要求提供有关主通道管理员的信息。最后，将询问您的 Web 服务器的文档根信息。

这一节非常重要，因为它将用于创建用于维护包和上传发布的行政前端，以及用于支持 PEAR 安装程序所需 REST 协议的 REST 文件。通常，括号中提供的默认值应该是可接受的。然而，了解不同提示的目的是有帮助的。

+   **PEAR 配置文件位置：** 这指定了用于检索通道信息的文件位置。Chiara_PEAR_Server 通道如果不能检索其自身通道的信息，将根本无法正常工作，因此这个值必须正确。

+   **本地主机的 Web 服务器文档根路径：** 这应该是您本地文件系统上 Web 服务器基目录的完整路径。如果您的 Web 服务器在用户请求[`servername/index.php`](http://servername/index.php)时读取`/var/lib/web/htdocs/servername/index.php`，那么您的文档根是`/var/lib/web/htdocs/servername`。

+   **前端.php HTML 管理前端文件的名称：** 这是您的管理前端文件的文件名。选择一个独特的文件名将有助于防止令人烦恼或恶意的人尝试未经授权访问通道管理界面。

+   **保存发布上传的临时路径：** 这应该是一个可由 Web 服务器写入的位置，您最初将在此处保存上传的发布版本。

+   **客户端应连接到的端口（443 是 SSI，80 是常规 HTTP）：** 按照说明操作。如果您有典型的配置，80 是`http://`的正常端口，443 是`https://`的典型端口。

+   **协议客户端应使用的连接方式（http 或 https）：** 再次，选择两个选项之一。

## 添加软件包和发布软件包

完成安装脚本后，导航到管理前端文件，其名称在**前端.php HTML 管理前端文件名称**部分中指定。例如，如果您的前端文件名为`foo.php`，并且您的频道是 localhost，请导航到`http://localhost/foo.php`。您应该会看到如下内容：

![添加软件包和发布软件包](img/1904811191_5_03.jpg)

要以管理员身份登录，请输入在安装脚本中为管理员指定的句柄和密码。登录后，在上传发布版本之前，您首先需要在服务器上创建软件包，然后添加维护者。

当您登录时，您将看到与此类似的屏幕：

![添加软件包和发布软件包](img/1904811191_5_04.jpg)

配置频道的链接位于屏幕左侧。为了开始，您可能还想要为要发布的软件创建一些类别，例如“数据库”或“XML 处理”。有关类别的示例，请参阅[`pear.php.net/packages.php`](http://pear.php.net/packages.php)。

在您创建了类别后，您将想要添加维护者并创建软件包。创建软件包很简单。首先，点击**创建软件包**，您将看到如下屏幕：

![添加软件包和发布软件包](img/1904811191_5_05.jpg)

填写必要的字段（由红色星号标记），然后点击**保存更改**。下一步非常重要：在您上传发布版本之前，您需要创建维护者并将他们添加为软件包的维护者。为此，请点击屏幕左侧您刚创建的软件包旁边的**（维护者）**链接：

![添加软件包和发布软件包](img/1904811191_5_06.jpg)

选择维护者、角色以及维护者是否活跃后，点击**添加维护者**，维护者将出现在软件包维护者列表中。

### 小贴士

**谁可以上传发布版本？**

频道管理员可以为频道上的任何软件包上传发布版本，以及列在频道中作为主要维护者的软件包维护者。请注意，在`package.xml`文件中列为主要维护者但在频道管理前端未列出的人员将**无法**上传发布版本，出于安全原因。

## 安装公共频道前端

在配置并运行 Chiara_PEAR_Server 之后，你可能希望从 channel `pear.crtx.org`安装 Davey Shafik 的**Crtx_PEAR_Channel_Frontend**包。Crtx_PEAR_Channel_Frontend 包提供了一个可浏览的网站，允许开发者查看你的频道提供的包以及如何获取它们。此外，它还支持链接到错误跟踪器、在线修订控制浏览器和其他功能。Crtx_PEAR_Channel_Frontend 对于 Chiara_PEAR_Server 来说，就像[`pear.php.net`](http://pear.php.net)对于在[`pear.php.net/rest`](http://pear.php.net/rest)提供的安装数据一样。

获取 Crtx_PEAR_Channel_Frontend 可以通过以下步骤完成：

```php
$ pear channel-discover pear.crtx.org
$ pear upgrade crtx/Crtx_PEAR_Channel_Frontend 

```

一旦安装了包，就需要进行一些小的配置（这可能在未来的版本中通过安装后的脚本自动化）。首先，你需要找到`pear_frontend.css`文件，它被安装到`data_dir/Crtx_PEAR_Channel_Frontend/data/pear_frontend.css`。

`data_dir`是数据路径（在 UNIX 上通常是`/usr/local/lib/php/data`，在 Windows 上是`C:\php5\PEAR\data`或`C:\php4\PEAR\data`），由 PEAR 的`data_dir`配置变量定义。一旦找到文件，将其复制到你的频道文档根目录。

在复制`pear_frontend.css`文件后，你需要创建公共前端 PHP 文件。以下是一个示例前端：

```php
<?php
/**
* An example of Crtx_PEAR_Channel_Frontend Usage
*
* @copyright Copyright © David Shafik and Synaptic Media 2004.
*All rights reserved.
* @author Davey Shafik <davey@synapticmedia.net>
* @link http://www.synapticmedia.net Synaptic Media
* @version $Id: $
* @package
* @category Crtx
*/
/**
* Crtx_PEAR_Channel_Frontend Class
*/
require_once 'Crtx/PEAR/Channel/Frontend.php';
$frontend = new Crtx_PEAR_Channel_Frontend('localhost',
array('database' => 'mysqli://user:pass@localhost/pearserver',
'index' => 'index.php', 'admin' => 'admin_myfront.php'));
?>
<html>
<head>
<title>localhost Channel Server</title>
<link rel="stylesheet" type="text/css"
href="pear_frontend.css" />
<?php
$frontend->showLinks();
?>
</head>
<body>
<div id="top">
<h1><a href="index.php">localhost Channel Server</a></h1>
</div>
<div id="menu">
<?php
$frontend->showMenu();
?>
<div id="releases">
<?php
$frontend->showLatestReleases();
?>
</div>
</div>
<div id="content">
<?php
if (!$frontend->run()) {
$frontend->welcome();
}
?>
</div>
</body>
</html>

```

如果将此文件保存为`index.php`在`localhost`网络服务器的文档根目录中，将提供类似于以下的有吸引力的屏幕：

![安装公共频道前端](img/1904811191_5_07.jpg)

此包内置了几个很好的功能，包括 RSS 源和向维护者发送电子邮件的能力。此外，外观和感觉的定制非常简单，通过基本修改`pear_frontend.css`文件和`index.php`前端来实现。只需做很少的工作，就可以创建一个非常吸引人的前端。可能性的多样性示例包括[`pear.crtx.org`](http://pear.crtx.org)和[`pear.php-tools.net`](http://pear.php-tools.net)。

# 通过频道分发付费使用的 PHP 应用程序

关于频道的一些更常见的问题是：“我的业务能否通过频道分发付费使用的 PHP 应用程序并限制访问？”答案是响亮的肯定。

PEAR 安装程序通过使用`pear login`命令实现 HTTP 身份验证。为了分发你的非免费应用程序，最好依靠像 Apache 这样的 Web 服务器的优势。例如，通过将“获取”文件和`httpd.conf`中的`ForceType`指令如下使用：

```php
<Location /get>
ForceType application/x-httpd-php
</Location>

```

你可以使用一个名为`get`的文件，其中包含 PHP 代码来处理用户提供的登录/密码，并将他们引导到为他们的登录定制的受限制的包。实际上，普通用户可以被引导到软件的试用版本，而无需更改他们的安装过程。

注意，PEAR 安装器仅支持 HTTP Basic 认证，并且为了实现真正的安全连接，应使用 SSL（HTTPS），否则任何人都可以获取受限制的密码。以下是一个用于 `get` 文件的示例脚本：

```php
<?php
/**
* Example restricted access file
*
*
* This example requires Apache, PHP 4.3+, the mysqli extension, and
* this code to be added to httpd.conf/.htaccess:
* <pre>
* <Location "/get">
* ForceType application/x-httpd-php
* </Location>
* </pre>
*
* In addition, it assumes that a mysql database is set up with users
* who have purchased the packages, and that database connection
* info is set in php.ini
* @package download
*/
// shut up or we risk getting corrupted tgz files
error_reporting(0);
function error($message)
{
header('HTTP/1.0 404 Not Found');
echo $message;
exit;
}
/**
* Downloader class, handles authentication and actual downloading
* @package download
*/
class Download
{
var $user = false;
var $passwd = false;
/**
* A list of purchased versions that the current user may
*download
*
* @var array
*/
var $purchased = array();
/**
* MySQL database connection
*
* @var resource mysqli resource
*/
var $db;
/**
* Full path to offline location of package releases
*
* @var string
*/
var $path = '/path/to/releases/';
/**
* Hash of package names to demo versions
*
* This probably should be constructed from a database,
* but for our simple example it will be hard-coded
*
* @var array
*/
var $demo_versions = array(
'Foo' => array('1.0demo'),
'Bar' => array('1.1demo', '2.0demo'),
);
/**
* Hash of package names to full versions
*
* This probably should be constructed from a database,
* but for our simple example it will be hard-coded
*
* @var array
*/
var $full_versions = array(
'Foo' => array('1.0'),
'Bar' => array('1.1', '2.0'),
);
/**
* Connect to the database, authenticate the user,
* and grab the list
* of purchased packages for this user
*/
function Download()
{
// assume we have specified connection details in php.ini
$this->db = mysqli_connect();
if ($this->db) {
// on database connect failure,
// we can still download demos,
// so fail silently
if (isset($_SERVER['PHP_AUTH_USER']) &&
isset($_SERVER['PHP_AUTH_PASSWD'])) {
$this->user = $_SERVER['PHP_AUTH_USER'];
$this->passwd = $_SERVER['PHP_AUTH_PASSWD'];
// construct a list of purchased packages
// for this user/pass combination
if ($res = mysqli_query($this->db, '
SELECT purchased_package FROM regusers
WHERE user = "' .
mysqli_real_escape_string($this->db,
$this->user) . '", AND pass = "' .
mysqli_real_escape_string($this->db,
$this->passwd) . '"')) {
while ($row = mysqli_fetch_row($res)) {
$this->purchased[$row[0]] = true;
}
}
}
}
}
/**
* Feed the file to the user, or display an error
*
* @param string $path
*/
function downloadPackage($path)
{
// note that we assume the case is correct
// (the PEAR Installer always gets
// this correct, only manual downloads will fail)
if (!preg_match('/^([a-zA-Z0-9_]+)-(.+)\.(tar|tgz)$/',
$path, $matches)) {
error('invalid package/version: "' . $path . '"');
}
list(, $package, $version, $ext) = $matches;
// sanity check #1: does the release exist on the disk?
if (!file_exists($this->path . $package . '-' . $version .
'.' . $ext)) {
error('unknown package/release: "' . $path . '"');
}
// sanity check #2: do we know anything about this version?
if (!isset($this->demo_versions[$package]) &&
!isset($this->full_versions[$package])) {
error('unknown package: "' . $package . '"');
}
// check to see if it is a demo version, and return right
// away if so
// if you have more purchaser downloads than demos,
// put this after
// purchased check for slight speed increase
if (isset($this->demo_versions[$package]) &&
in_array($version, $this->demo_versions[$package],
true)) {
$this->_doDownload($package, $version, $ext);
}
if (isset($this->full_versions[$package]) &&
in_array($version, $this->full_versions[$package],
true)) {
if (isset($this->purchased[$package])) {
$this->_doDownload($package, $version, $ext);
}
// if we get here, the user has not purchased this
// version
error('version "' . $version . '" is restricted and
must be purchased. ' .
'Use "pear login" to set purchase key first');
}
// fall-through: this line of code should be unreachable
error('internal error, please report attempt to download "
' . $path . '" failed');
}
/**
* Do the actual downloading.
*
* @param string $package
* @param string $version
* @param string $ext this is either "tar" or "tgz"
* @access private
*/
function _doDownload($package, $version, $ext)
{
// construct local path to the downloadable object
$path = $this->path . $package . '-' . $version . '.' . $ext;
header('Last-modified: ' .
gmdate('D, d M Y H:i:s \G\M\T', filemtime($path)));
header('Content-type: application/octet-stream');
header('Content-disposition: attachment; filename="' .
$path . '"');
header('Content-length: ' . filesize($path));
readfile($path);
exit;
}
}
if (!isset($_SERVER['PATH_INFO']) || $_SERVER['PATH_INFO'] == '/') {
error('no package selected');
}
$info = explode('/', $_SERVER['PATH_INFO']);
switch (count($info)) {
case 2:
$dl = new Download;
$dl->downloadPackage($info[1]);
break;
default:
error('no package selected');
}
?>

```

上述示例展示了即使是复杂的版本验证也可以非常容易地完成。然而，这种方法并不非常高效——每个下载都会通过 PHP 解释器，这比获取静态文件慢得多。另一个选项，仅适用于 PEAR 版本 1.4.9 或更高版本，是发送重定向头（402），并允许 Apache 处理实际的文件下载。然而，与使用 `readfile()` 相比，节省的量非常小，可能不值得要求用户升级。

另一个选项是简单地通过 `.htaccess` 文件中的 HTTP Basic 认证限制对文件的访问，对于单个 tarballs 来说，这可能是可扩展的。

然后，指导用户在首次设置登录名/密码时采取以下步骤：

```php
$ pear -d "default_channel=your.channel.com" login
Logging into your.channel.com
Username: myuser
Password: mypassword 

```

只有当用户从 `your.channel.com` 请求包时，才会发送用户名/密码，利用了按频道配置的优势。

强烈建议为您的频道使用 SSL，这样用户名/密码对就不会以明文形式发送。

那就是所有必要的了！

# 通过静态 tarballs 为单个客户端安装分发包

除了通过频道分发包之外，还可以分发单个发布版本并将其发布到网络上。

### 小贴士

**安装带有依赖项的静态发布版本需要 PEAR 1.4.10**

PEAR 安装器中的一个错误阻止了静态 tarballs 的安装；使用版本 1.4.10 或更高版本以获得此问题的修复。

这基本上涉及通过 `pear package` 打包一个发布版本，然后将其上传到网站，然后可以下载或直接通过以下方式安装：

```php
$ pear install http://www.example.com/Package-1.0.0.tgz 

```

这不是什么新鲜事：最早的 PEAR 安装器支持这种语法。新的功能是能够在其他包发布中依赖这些 **静态 tarballs**。

## 谁需要这个功能？

在某些情况下，设置频道服务器并非必需。一般来说，设置频道服务器并以这种方式分发包会更好。然而，在现实世界中，一个常见的场景是一位为多个客户提供服务的 PHP 顾问，同时也为他们维护网站。尽管每个网站都是独特的，但拥有一套每个特定网站都可以使用的实用包非常有帮助。只为这些包提供频道只会引入不必要的复杂性。

通过将客户的网站安装为 PEAR 包，作为软件顾问的您就可以轻松地维护网站并以比频道允许的更严格的方式管理其内容。

## package.xml 和依赖项的差异

为了防止通道和静态 tar 包之间的名称冲突，静态 tar 包的`package.xml`文件不能使用`[<channel>](http://<channel>)`标签，而必须使用`<uri>`标签。此外，`<uri>`标签必须包含 tar 包在互联网上的实际位置。如果静态 tar 包位于[`www.example.com/tarballs/Package-1.0.0.tgz`](http://www.example.com/tarballs/Package-1.0.0.tgz)，则`package.xml`文件应该以类似以下内容开始：

```php
<?xml version="1.0" encoding="UTF-8"?>
<package packagerversion="1.4.3" version="2.0"

xsi:schemaLocation="http://pear.php.net/dtd/tasks-1.0
http://pear.php.net/dtd/tasks-1.0.xsd
http://pear.php.net/dtd/package-2.0
http://pear.php.net/dtd/package-2.0.xsd">
<name>Package</name>
<uri>http://www.example.com/tarballs/Package-1.0.0</uri>

```

非常重要的是要注意，`uri`已经去除了文件扩展名`.tgz`。这是因为当提供静态 tar 包时，预期您也会为没有启用 zlib 扩展的用户提供未压缩的`.tar`文件。

要依赖这个静态 tar 包，应使用如下依赖标签：

```php
<package>
<name>Package</name>
<uri>http://www.example.com/tarballs/Package-1.0.0</uri>
</package>

```

当使用静态 tar 包作为依赖项时，版本号没有意义，因此不允许使用任何正常的版本号标签（`<min>`, `<max>`, `<recommended>`）。然而，升级静态 tar 包是可能的。

每个静态 tar 包软件包都有一个隐含的`<channel>__uri</channel>`标签——所有静态 tar 包都是作为伪通道`__uri`安装/升级/卸载的。这个通道被当作其他通道一样对待，只是它不能通过`channel-update`命令修改，也不能通过`channel-delete`命令删除，并且它不包含服务器，因此永远不会尝试联系远程通道服务器。运行`pear channel-info __uri`的结果是：

```php
CHANNEL __URI INFORMATION:
==========================
Name and Server __uri
Summary Pseudo-channel for static packages
Validation Package Name PEAR_Validate
Validation Package default
Version
SERVER CAPABILITIES
===================
TYPE VERSION/REST TYPE FUNCTION NAME/REST BASE
No supported protocols 

```

除了展示`__uri`伪通道的品质外，它还告诉我们静态 tar 包软件包的验证与`pear.php.net`通道软件包的验证一样严格（使用`PEAR_Validate`）。如果您需要验证的灵活性，则有必要使用通道而不是将软件作为静态 tar 包分发。

然而，这也意味着在通过以下方式安装我们的静态软件包之后：

```php
$ pear install http://www.example.com/tarballs/Package-1.0.0.tgz 

```

如果有新版本发布，可以通过以下方式升级此软件包：

```php
$ pear upgrade http://www.example.com/tarballs/Package-1.0.1.tgz 

```

此外，还可以通过简单的以下方式卸载软件包：

```php
$ pear uninstall __uri/Package 

```

如果您想查看已安装的所有静态 tar 包软件包的列表，只需运行带有-c 选项的`list`命令：

```php
$ pear list -c __uri 

```

小心静态 tar 包的简单性！如果您的最终用户有可能从任何其他来源安装静态 tar 包，您必须通过通道分发您的软件包。否则，用户可能会遇到两个不同软件包之间的名称冲突，如下两个假设的安装命令所示：

```php
$ pear install http://www.example.com/tarballs/Foo-1.0.0.tgz
$ pear upgrade http://www.notexample.com/Foo-1.2.3.tgz 

```

在这种情况下，从`www.example.com/tarballs`分发的名为`Foo`的软件包与从`www.notexample.com`分发的名为`Foo`的软件包不是同一个代码库，但 PEAR 安装程序将它们都当作您输入了以下内容：

```php
$ pear install __uri/Foo
$ pear upgrade __uri/Foo 

```

在这种情况下，微妙到严重的破坏可能性立即存在，并且难以调试。不要冒险；如果有可能发生这种情况，请使用通道。 

## 发布等于上传

如前所述，使用静态 tarball 而不是通道的最大优点是，发布一个新的软件包只需上传由以下命令创建的`.tgz`和`.tar`文件：

```php
$ pear package
$ pear package -Z 

```

没有什么比这更容易了！

# 远程安装固有的安全问题

如 phpBB、MySpace.com 和 XML_RPC 中漏洞被互联网蠕虫利用的出现所证明的那样，安全漏洞可不是什么笑料，你必须意识到安装你自己没有编写的软件所涉及到的潜在风险。

幸运的是，PEAR 安装器模型提供了升级以获取所需的安全修复的便利性，并提供了内在的安全性，以确保你不会仅仅因为使用 PEAR 安装程序而成为恶意黑客的受害者。

虽然 PEAR 开发者已经采取了所有步骤来确保你的代码的安全性而不限制其有用性，但了解一些安全性的基础知识仍然非常重要，因为 PEAR 无法保护你自己，如果你选择使用或编写不安全的代码。最近有几篇关于 PHP 安全和互联网安全的一般性的优秀参考资料。如果你不熟悉诸如输出转义、输入过滤或 XSS、任意代码执行、隐蔽性安全等概念，Ilia Alshanetsky 的*php|architect's Guide to PHP Security*和 Chris Shiflett 的*Essential PHP Security*指南都是很好的起点。

### 小贴士

许多开发者错误地遵循旧的金科玉律：“己所不欲，勿施于人。”当你开发具有互联网连接组件的代码时，这种方法是致命的。如果除了你之外的其他人可以访问你编写的 PHP 应用程序，那么你需要假设他们只有最恶劣的意图。

当你设计一个功能时，要自问：“我如何利用这个功能来修改它运行的环境？我能用它执行意外的操作吗？”

如果答案是“是”或甚至“可能”，那么这个功能本质上是不安全的，必须限制它，直到答案是“否”或“只有在极端情况下，其他安全措施才能使其失效”。

## PEAR 安装程序和 Chiara_PEAR_Server 如何提供安全性？

PEAR 安装程序已经采取了许多重要步骤来提供安全性。然而，从根本上说，PEAR 安装程序最终是为了安装任意 PHP 代码——这是它的存在理由，因此，使用 PEAR 的安全性第一条必须是：

### 小贴士

*永远*不要在没有先查看在开发服务器上提供的代码之前，在一个正在运行的、生产性的网站上安装一个软件包。

### PEAR 提供的额外安全性

虽然 PEAR 安装程序所采取的大多数基本操作都有合理的预期安全性，但有一些应该避免。

这意味着，例如，在生产服务器上应该不惜一切代价避免使用 `upgrade-all` 命令。这个命令将所有现有包升级到最新版本，这本质上消除了你控制升级的能力。这个命令最好在开发服务器上使用，在升级生产服务器上的单个包之前测试包的新版本。

此外，如果你正在安装由 `php.net`（在撰写本章时为 `pear.php.net` 和 `pecl.php.net`）以外的渠道分发的包，首先下载该包，并在包上运行两个命令来了解更多信息：

```php
$ pear info Packagename-1.0.0.tgz
$ pear list-files Packagename-1.0.0.tgz 

```

第一个会告诉你包的依赖项。如果你不认识这些依赖项，那么你将需要对他们执行相同的步骤。

`list-files` 命令列出了存档中的所有文件。寻找安装到 `PEAR/` 子目录中的文件。除非包提供自定义文件角色、自定义文件任务或自定义命令，否则包通常没有理由将文件安装到这个位置，因为这个目录是由 PEAR 安装器使用的。安装到这个目录中的任何文件都可能试图恶意影响安装器的工作方式。

这条规则的明显例外是名称中包含 PEAR 的包，例如 `PEAR_PackageFileManager`。如果一个包将文件安装到似乎与声明的目的没有太多关系的位置，你应该立即感到怀疑。联系包维护者，询问为什么包需要在那里安装文件。

如果你没有得到满意的答复，立即在 `<pear-dev@lists.php.net>` 上通知 PEAR 开发者的邮件列表。任何试图提供恶意软件包的渠道都将被 PEAR 安装器列入黑名单。

更重要的是，如果你从这本书中只记住一件事，那就让它成为这件事：

### 小贴士

*永远*不要在没有查看安装后脚本的 PHP 源代码的情况下运行安装后脚本。

安装后脚本按定义是任意 PHP 代码。任何可以在 PHP 中完成的事情都可以通过安装后脚本完成。这包括像安装间谍软件、擦除你的硬盘驱动器以及其他你可能在运行 `pear run-scripts` 命令时不想做的事情。为了不查看脚本而冒整个系统风险是永远不值得的。

最终，由于恶意软件包被追踪到源头的极端容易性，以及需要明确用户安装包的要求，通过渠道分发的恶意软件包被分发的可能性极低。除非你同时警惕地从未知来源安装包，以便在造成任何重大损害之前将其捕获，否则这两个因素将不足以阻止那些作恶者。

此外，您必须升级 PEAR 安装器到版本 1.4.3 或更高版本，并在安装器的新版本发布时持续升级。新版本中一定会解决错误、安全问题和小型修复。

### 在设计 PEAR 安装器和 Chiara_PEAR_Server 时应用的具体安全原则

在设计和实现最新版本的 PEAR 安装器时，主要关注的是防止对用户环境的意外修改。通过使安装器能够利用 `pear.php.net/pecl.php.net` 之外的来源获取软件包，并添加如安装后脚本、自定义文件角色和自定义文件任务等功能，存在一定的风险。每个新功能都在对合法活动的开放性和对危险活动的限制之间取得平衡。

例如，频道是通过其服务器名称定义的。这意味着您不能自动且秘密地更改由 `pear.php.net` 分发的软件包的来源。此外，当用户运行：

```php
$ pear channel-update mychannel.example.com 

```

PEAR 安装器试图检索 [`mychannel.example.com/channel.xml`](http://mychannel.example.com/channel.xml)。一个聪明而邪恶的频道管理员实际上可以提供一个没有定义 `mychannel.example.com` 的 `channel.xml` 文件。这种恶作剧会被安装器立即检测到，并禁止。

添加频道镜像也引入了风险元素。通过在 `channel.xml` 中定义这些镜像，无法使安装器相信另一个频道是某个频道的合法镜像。

当从频道下载软件包时，会执行严格的 `package.xml` 验证。如果软件包是从 `foo.example.com` 频道下载的，而其 `package.xml` 声称来自 `pear.php.net`，PEAR 安装器将拒绝安装或升级该软件包，因为这将是明显的安全漏洞。

此外，如果请求的软件包名称与 `package.xml` 中的软件包名称不同，PEAR 安装器将拒绝安装或升级该软件包。否则，就有可能分发一个声称是 `foo.example.com/Foo` 实际上分发 `pear.php.net/PEAR` 的软件包。相同的机制防止了对恶意软件包的依赖。从频道/软件包下载的软件包必须是那个频道/软件包，仅此而已。对于静态 tarball 也是如此。一个依赖于使用静态 tarball 软件包依赖的软件包不能分发基于频道的软件包。

在 PEAR 安装器中实现的最可能危险的功能是安装后脚本和自定义文件任务。这两个功能在调用时都会自动执行任意代码。PEAR 通过使意外执行恶意代码变得极其困难来提供一层安全防护。用户必须：

+   明确安装恶意文件任务

+   明确安装使用恶意文件任务的软件包

对于安装后脚本，最终用户必须：

+   明确安装包含恶意后安装脚本的包

+   明确输入 `pear run-scripts maliciouschannel/maliciouspackage` 以运行恶意包

这些额外步骤使得 PEAR 安装程序在无意中损害系统变得非常困难，同时也为执行不寻常情况提供了极高的可见性。

重要的是要注意，直到 PEAR 版本 1.4.3，PEAR 安装程序有两个安全漏洞。两者都需要用户安装一个公开分发的恶意包。这两个漏洞都是由命令模式的不当实现引起的。

命令模式是一种基于在特殊子目录中加载文件来提供扩展性的方法。PEAR 安装程序自 1.0 版本以来一直在使用这种模式来加载当你输入时显示的实际命令：

```php
$ pear help 

```

实现这种模式的文件位于 `PEAR/Command/*.php`（包括 `PEAR/Command/Auth.php`、`PEAR/Command/Install.php` 等），在 PEAR 版本 1.4.2 及更早版本中，这些文件在用户使用 `pear` 命令时会被加载。

这最终为恶意包提供了一个在不受控制的情况下执行任意 PHP 代码的途径。PEAR 1.4.3 及更高版本通过实现描述命令的 XML 格式来修复这个问题。除非用户明确请求除 `help` 之外的命令，否则不会加载任何实际的 PHP 代码。

命令模式也用于自定义文件角色，首次在 PEAR 1.4.x 版本中引入。每次执行 `pear` 命令时，都会加载所有 `PEAR/Installer/Role/*.php` 文件，以构建自定义配置变量的列表。用于修复命令中任意代码执行漏洞的相同解决方案也被用来修复这个漏洞。

# 摘要

本章向我们展示了频道的设计目的是为了使从任何位置安装包变得容易，但在过程中损害系统的难度很大，遵循一个基本的安全原则：总是让做事最容易的方式成为最安全的方式。

频道打破了 `pear.php.net` 对 PEAR 安装程序的垄断，使其面向整个互联网。通过你的频道分发的自定义构建的包甚至可以出售并提供给特定用户，同时与公开可用的开源包和平共处。
