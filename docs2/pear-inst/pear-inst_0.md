# 前言

很可能在您使用 PHP 的某个时刻看到过缩写词 **PEAR**，无论是偶然间还是安装和使用来自 `pear.php.net` 的包时。如果您已经调查过，您可能已经听说过 PEAR 提供的流行软件，例如 DB 数据库抽象包或 HTML_QuickForm 包。您可能没有意识到的是，PEAR 不仅仅是一个您可以使用的包集合。PEAR 还包含了 PHP 最灵活的安装程序，即 PEAR 安装器。

使用 PEAR 安装器，您不仅可以安装来自 `pear.php.net` 的包，还可以从其他 PEAR 通道安装包，使用您自己的 PEAR 通道分发您自己的软件项目，甚至使用 PEAR 安装器维护复杂的 Web 项目。惊讶吗？好吧，继续阅读，因为这本书揭示了 PEAR 安装器的秘密以及它将如何用 PHP 编程语言彻底改变您的日常开发！

# 本书涵盖的内容

*第一章* 介绍了您对 PEAR 安装器的了解。我们首先回顾了传统的解压并运行方法来分发 PHP 软件，并将其与 PEAR 安装器基于包的分发 PHP 软件方法进行比较。您将看到 PEAR 通道的创新之处，一瞥 PEAR 安装器如何从包中安装文件，以及它是如何知道安装位置的。最后，您将了解获取 PEAR 安装器的多种方法，甚至如何在没有提供 shell 访问的 Web 主机上远程安装 PEAR。

*第二章* 是所有 PHP 开发者必读的，因为它解释了 `package.xml` 的基本工作原理，这是 PEAR 安装器的核心。`package.xml` 用于控制 PEAR 安装器可以做的几乎所有事情。您将了解版本控制对于控制安装包质量的重要性，依赖关系的重要性，以及 PEAR 安装器如何管理库和应用程序之间的重要链接。您还将了解 `package.xml` 如何组织包元数据，如包名称、作者、发行说明和变更日志，以及如何组织关键安装数据，如文件、依赖关系和版本控制。

*第三章* 为想要利用 `package.xml` 版本 2.0 中引入的完整应用程序支持功能的开发者提供了更深入的探讨。

*第四章* 从查看 PEAR 安装器的细节转向使用 PEAR 安装器开发和维护一个复杂且快速发展的网站。

*第五章* 涵盖了 PEAR 通道。通道旨在使从任何位置安装包变得容易，但在过程中很难损害您的系统，遵循一个基本的安全原则：始终设计事情，使其最简单的方法是最安全的。

通道打开了`pear.php.net`对 PEAR 安装程序的垄断，使其面向整个互联网。通过您的通道分发的自定义包甚至可以出售并提供给特定用户，同时与公开可用的开源包和平共处。

*第六章* 教您如何嵌入 PEAR 安装程序以创建插件管理器。该章节创建了一个假博客应用程序，它提供了无缝查询设计用于分发模板的远程 PEAR 通道服务器的功能。使用 PEAR 安装程序的内部类，我们的博客 Web 应用程序智能地安装和升级模板，具有从 PEAR 安装程序期望的所有复杂性。

# 惯例

在这本书中，您将找到许多不同信息类型的文本样式。以下是一些这些样式的示例及其含义的解释。

代码有三种样式。文本中的代码单词如下所示：“接下来，您需要使用`config-create`命令为远程机器创建一个配置文件。”

代码块将如下设置：

```php
<file name="blah.php" role="php">
<tasks:replace from="@DATABASE-URL@" to="database_url"
type="pear-config" />
</file>

```

当我们希望您注意代码块中的特定部分时，相关的行或项目将被加粗：

```php
if (is_object($infoplugin)) {
$bag = new serendipity_property_bag;
$infoplugin->introspect($bag);
if ($bag->get('version') == $data['version']) {
$installable = false;
} elseif (version_compare($bag->get('version'),
$data['version'], '<')) {
$data['upgradable'] = true; 
$data['upgrade_version'] = $data['version'];
$data['version'] = $bag->get('version');

```

任何命令行输入和输出都应如下编写：

```php
$ pear -c pear.ini remote-install -o DB_DataObject 

```

**新术语**和**重要词汇**以粗体字形式引入。您在屏幕上看到的单词，例如在菜单或对话框中，在我们的文本中如下所示：“点击**下一步**按钮将您带到下一屏幕”。

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小技巧和技巧看起来是这样的。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法，您喜欢或可能不喜欢的地方。读者反馈对我们开发您真正从中受益的标题非常重要。

要发送一般反馈，只需将电子邮件发送到`<feedback@packtpub.com>`，确保在邮件主题中提及书名。

如果您需要一本书并且希望我们出版，请通过[www.packtpub.com](http://www.packtpub.com)上的**建议标题**表单或发送电子邮件至`<suggest@packtpub.com>`给我们。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大收益。

## 下载本书的示例代码

访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，并从标题列表中选择这本书来下载本书的任何示例代码或额外资源。然后，将显示可下载的文件。

可下载的文件包含如何使用它们的说明。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并有助于改进本书的后续版本。如果你发现任何勘误，请通过访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，选择你的书籍，点击**提交勘误**链接，并输入你的勘误详情来报告。一旦你的勘误得到验证，你的提交将被接受，勘误将被添加到现有勘误列表中。现有的勘误可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择你的标题来查看。

## 问答

如果你在这本书的某个方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们会尽力解决。
