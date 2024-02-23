# 前言

PHP 和 CouchDB Web 开发将教您结合 CouchDB 和 PHP 的基础知识，从构思到部署创建一个完整的应用程序。本书将指导您开发一个基本的社交网络，并引导您避免与 NoSQL 数据库经常相关的一些常见问题。

# 这本书涵盖了什么

第一章，“CouchDB 简介”，快速定义了 NoSQL 和 CouchDB 的概述。

第二章，“设置您的开发环境”，为使用 PHP 和 CouchDB 开发应用程序的计算机进行设置。

第三章，“开始使用 CouchDB 和 Futon”，定义了 CouchDB 文档，并展示了如何从命令行和 Futon（CouchDB 内置的管理实用程序）中管理它们。

第四章，“启动您的应用程序”，创建一个简单的 PHP 框架来容纳您的应用程序，并将此代码发布到 GitHub。

第五章，“将您的应用程序连接到 CouchDB”，使用各种方法将您的应用程序连接到 CouchDB，并最终为您的应用程序选择合适的解决方案。

第六章，“建模用户”，在您的应用程序中创建用户，并处理使用 CouchDB 进行文档创建和身份验证。

第七章，“用户配置文件和建模帖子”，使用 Bootstrap 完善您的用户配置文件，并将内容发布到 CouchDB。

第八章，“使用设计文档进行视图和验证”，探索了 CouchDB 专门使用设计文档来提高应用程序质量。

第九章，“为您的应用程序添加花里胡哨的东西”，利用现有工具简化和改进您的应用程序。

第十章，“部署您的应用程序”，向世界展示您的应用程序，并教您如何使用各种云服务启动应用程序和数据库。

*额外章节*，“复制您的数据”，了解如何使用 CouchDB 的复制系统来扩展您的应用程序。

您可以从[`www.packtpub.com/sites/default/files/downloads/Replicating_your_Data.pdf`](http://www.packtpub.com/sites/default/files/downloads/Replicating_your_Data.pdf)下载*额外章节*。

# 您需要为本书做些什么

您需要一台安装了 Mac OSX 的现代计算机。第一章，“CouchDB 简介”，将为 Linux 和 Windows 机器提供设置说明，并且本书中编写的代码将在任何机器上运行。但是，本书中使用的大多数命令行语句和应用程序都是特定于 Mac OSX 的。

# 这本书是为谁准备的

这本书适用于初学者和中级 PHP 开发人员，他们有兴趣在项目中使用 CouchDB 开发。高级 PHP 开发人员将欣赏 PHP 架构的熟悉性，并可以轻松学习如何将 CouchDB 纳入其现有的开发经验中。

# 约定

在本书中，您会经常看到几个标题。

为了清晰地说明如何完成某个过程或任务，我们使用：

# 行动时间 - 标题

1.  行动 1

1.  行动 2

1.  行动 3

指示通常需要一些额外的解释，以便它们有意义，因此它们后面跟着：

## 刚刚发生了什么？

这个标题解释了您刚刚完成的任务或指令的工作原理。

您还会在本书中找到其他一些学习辅助工具，包括：

## 小测验 - 标题

这些是简短的多项选择题，旨在帮助您测试自己的理解。

## 尝试一下英雄 — 标题

这些设置了实际的挑战，并为您提供了尝试所学内容的想法。

你还会发现一些文本样式，用于区分不同类型的信息。以下是一些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下：“很难为 Linux 标准化`install`方法，因为有许多不同的风味和配置。”

代码块设置如下：

```php
<Directory />
Options FollowSymLinks
AllowOverride None
Order deny,allow
Allow from all
</Directory>

```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```php
<Directory />
Options FollowSymLinks
**AllowOverride All** 
Order deny,allow
Allow from all
</Directory>

```

任何命令行输入或输出都是这样写的：

```php
**sudo apt-get install php5 php5-dev libapache2-mod-php5 php5-curl php5-mcrypt** 

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，如菜单或对话框中的单词，会以这种方式出现在文本中：*“通过打开**终端**开始”*。

### 注意

警告或重要提示会以这样的框出现。

### 提示

提示和技巧会以这种方式出现。
