# 前言

Yii 框架 2（Yii2）是流行的 Yii 框架的继任者。像它的继任者一样，Yii2 是一个开源、高性能的快速开发框架，旨在创建现代、可扩展和性能卓越的 Web 应用程序和 API。

本书旨在为没有接触过 Yii 和 Yii2 的开发者以及希望成为 Yii2 专家的 Yii 框架开发者提供指导，这本书将成为您成为 Yii 大师的指南。从初始化和配置到调试和部署，这本书将成为您掌握这个强大框架所有方面的指南。

# 本书涵盖内容

第一章，*Composer、配置、类和路径别名*，涵盖了 Yii2 应用程序的基础知识。在本章中，您将学习 Yii2 的核心约定以及如何将其配置为多环境应用程序。您还将发现如何使用 Composer，这是一个用于管理应用程序软件依赖项的依赖项管理工具。

第二章，*控制台命令和应用*，专注于如何使用内置的 Yii2 控制台命令，因为它引导您创建自己的命令。

第三章，*迁移、DAO 和查询构建*，教您如何在 Yii2 中创建迁移，以及如何使用数据库访问对象（DAO）与数据库交互，以及如何使用 Yii2 的查询构建器。

第四章，*活动记录、模型和表单*，教您如何创建和使用活动记录，以轻松地与数据库交互。此外，您还将发现如何创建模型来表示不在数据库中存储的信息，以及如何根据活动记录模型和普通模型创建 Web 表单。

第五章，*模块、小部件和助手*，涵盖了如何在我们的应用程序中集成模块。本章还将介绍如何创建和使用动态小部件，并额外介绍 Yii2 的强大助手类。

第六章，*资产管理*，专注于如何使用资产包创建和管理我们的资产，以及如何使用资产命令管理我们的资产。本章还涵盖了使用 Node Package Manage 和 Bower 等强大工具构建和生成我们的资产库的几种策略。

第七章, *验证用户身份和授权*，教你如何在 Yii2 中使用几种常见的身份验证方案（如 OAuth 身份验证、基本 HTTP 身份验证和头部身份验证）来验证用户的真实性，并展示如何授予他们访问应用程序特定部分的权利。

第八章, *路由、响应和事件*，专注于 Yii2 的路由和响应类如何在 Yii2 中工作。在本章中，我们将介绍如何处理应用程序内外部的数据，并发现如何利用 Yii2 强大的事件系统。

第九章, *RESTful API*，讨论了如何使用 Yii2 的 ActiveController 类快速轻松地通过 RESTful JSON 和 XML API 扩展你的应用程序。

第十章, *使用 Codeception 进行测试*，帮助你学习如何使用名为 Codeception 的强大测试工具为你的应用程序创建单元、功能性和验收测试。在本章中，你还将学习如何创建用于测试目的的数据固定值。

第十一章, *国际化与本地化*，介绍了如何本地化我们的应用程序并构建它们以支持多种语言。此外，你还将掌握如何使用 Yii2 控制台命令创建和管理翻译文件。

第十二章, *性能和安全*，涵盖了多种提高你的 Yii2 应用程序性能的方法以及如何使其免受现代网络应用程序攻击的保障措施。

第十三章, *调试和部署*，帮助你熟练掌握如何使用应用程序日志和 Yii2 调试工具来调试你的 Yii2 应用程序。此外，你还将发现无缝且不中断地部署你的 Yii2 应用程序的基本原则。

# 你需要这本书的内容

为了确保开发环境的统一并防止对主机操作系统的不必要的更改，强烈建议你在 Linux 虚拟机中运行所有命令。这将确保你的输出，无论是从你的网页浏览器还是从你的命令行，都与本书中展示的输出相匹配。由于自己设置这个环境可能是一项艰巨的任务，因此提供了使用 VirtualBox 和 Vagrant 的预构建虚拟机，以简化设置过程。

要开始使用这本书，你应该运行 Microsoft Windows 7、8、8.1 或 10、Apple OS X 10.9 或更高版本，或者能够运行虚拟机的 Linux 操作系统，例如 Ubuntu 14.04 LTS。此外，你还需要安装 VirtualBox 的最新版本（可在[`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads)找到）和 Vagrant（可在[`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html)找到）。

### 注意

安装这些软件依赖项后，你可能需要重新启动你的计算机以使更改生效。

安装好 VirtualBox 和 Vagrant 后，你可以在新的命令行或终端窗口中打开，为章节创建一个新的目录，然后运行以下命令来创建你的虚拟机开发环境。这些命令将下载一个预构建的虚拟机，其中包含所有启动所需的软件，并启动你的新开发环境：

```php
vagrant init charlesportwoodii/php56_trusty64
vagrant up --provider virtualbox
vagrant ssh

```

### 注意

关于这个特定 Vagrant 虚拟机的更多信息可以在[`atlas.hashicorp.com/charlesportwoodii/boxes/php56_trusty64`](https://atlas.hashicorp.com/charlesportwoodii/boxes/php56_trusty64)找到。

注意，如果你使用的是 Windows 操作系统，你可能需要像 PuTTy 这样的工具通过 SSH 连接到你的虚拟机。有关如何在 Windows 上通过 SSH 连接到你的新虚拟机的更多信息，可以在[`docs-v1.vagrantup.com/v1/docs/getting-started/ssh.html`](http://docs-v1.vagrantup.com/v1/docs/getting-started/ssh.html)找到。

一旦你的新 Vagrant 虚拟机启动，你就可以通过 SSH 访问这个虚拟机的文件，并通过打开一个新的浏览器窗口并导航到`http://localhost:8080`来访问你的`webroot`目录。默认情况下，当你打开这个网页时，你会看到`phpinfo()`的输出。

### 小贴士

根据你的操作系统安全设置，你的计算机可能会提示或阻止你访问计算机上的 8080 端口。如果你遇到问题，请确保配置你的防火墙设置，并确保计算机上的 8080 端口是开放的，并且 VirtualBox 可以从主机操作系统转发连接到虚拟操作系统。

由于 Yii2 完全兼容 PHP7，强烈建议你在 PHP7 上开发和测试你的 Web 应用程序。以下命令将允许你配置 PHP7 的 Vagrant 虚拟机：

```php
vagrant init charlesportwoodii/php7_trusty64
vagrant up --provider virtualbox
vagrant ssh

```

### 小贴士

由于这些虚拟机自动配置端口转发，建议您一次只运行一个虚拟机。有关完整命令和配置选项的列表，请参阅 Vagrant 文档：[`docs.vagrantup.com/v2`](https://docs.vagrantup.com/v2)。

# 本书面向对象

*掌握 Yii* 适合中级到高级的软件开发人员，他们希望快速掌握 Yii2。本书假设您对 PHP 5、HTML5 以及基本的软件开发实践和方法有所了解。

# 术语

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："此脚本告诉 Composer，当运行`create-project`命令时，它应该运行`postCreateProject`静态函数。"

代码块设置如下：

```php
"scripts": {
    "post-create-project-cmd": [
        "yii\\composer\\Installer::postCreateProject"
    ]
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```php
// Define our application_env variable as provided by nginx/apache
if (!defined('APPLICATION_ENV'))
{
 if (getenv('APPLICATION_ENV') != false)
 define('APPLICATION_ENV', getenv('APPLICATION_ENV'));
 else 
 define('APPLICATION_ENV', 'prod');
}

$env = require(__DIR__ . '/config/env.php');
```

任何命令行输入或输出都如下所示：

```php
$ ./yii fixture/load <FixtureName>
$ ./yii fixture/unload <FixtureName>

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："一旦我们指定了所有必要的属性，我们就可以点击**预览**按钮来预览我们的表单，然后我们可以点击**生成**按钮来生成源代码。"

### 注意

警告或重要注意事项以如下框中显示。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有一些东西可以帮助您从您的购买中获得最大收益。

## 下载示例代码

本书最新和最完整的源代码副本维护在 Packt 网站上：[`www.packtpub.com`](http://www.packtpub.com)，以及 GitHub 上的[`github.com/masteringyii`](https://github.com/masteringyii)，适用于每个适用的章节。

## 错误更正

尽管我们已经尽一切努力确保内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 侵权

在互联网上侵犯版权材料是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似侵权材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
