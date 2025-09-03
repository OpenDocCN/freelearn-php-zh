# 前言

Zend Framework 2 是知名的 Zend Framework 的最新更新。这个版本大大简化了使用即插即用组件以最小开发努力构建复杂 Web 应用的过程。Zend Framework 2 还提供了一个高度健壮和可扩展的框架，用于开发 Web 应用。

本书将指导你通过使用 ZF2 开发强大 Web 应用的过程。它涵盖了 Zend Framework 应用开发的各个方面，从安装和配置开始；任务设计得易于读者理解和使用，以便他们能够轻松地构建自己的应用。

本书从 Zend Framework 的基本安装和配置开始。随着你完成练习，你将彻底熟悉 ZF2。通过本书，你将了解使用 Zend Framework 2 构建稳固的 MVC Web 应用的基本概念。详细的分步指导将使你能够构建诸如群聊、文件和媒体共享服务、搜索以及简单的商店等功能。你还将使用各种外部模块来实现原生不可用的功能。

到本书结束时，你将熟练掌握使用 Zend Framework 2 构建复杂且功能丰富的 Web 应用。

# 本书涵盖的内容

第一章，*开始使用 Zend Framework 2.0*，介绍了开发环境的配置。在本章中，我们将设置 PHP 应用服务器，安装 MySQL，并创建一个开发数据库，该数据库将在后续章节中用于我们的 Zend Framework 学习练习。

第二章，*构建您的第一个 Zend Framework 应用*，解释了创建 Zend Framework 2 项目的步骤；我们将通过创建模块、控制器和视图来回顾构建 ZF2 MVC 应用的一些关键方面。我们将在 Zend Framework 中创建自己的自定义模块，该模块将在本书的后续章节中得到进一步扩展。

第三章，*创建通信应用*，介绍了 Zend\Form。在本章中，我们将创建我们的第一个注册表单，并使用 Zend Framework 组件设置注册用户的登录和认证。

第四章，*数据管理和文档共享*，涵盖了 Zend Framework 的一些数据和管理文件的概念。在本章中，我们将学习 Zend Framework 的各个方面，包括 ServiceManager、TableGateway 模式、处理上传和文件共享。

第五章，*聊天和电子邮件*，涵盖了在您的应用程序中使用 JavaScript 的内容。本章以一个简单的群聊实现为例，解释了在您的应用程序中使用 JavaScript 的用法；您还将了解到如何使用 Zend\Mail 和 ZF2 事件管理器发送电子邮件。

第六章，*媒体共享*，解释了使用 Zend Framework 管理和共享图片和视频。在本章中，我们将使用各种外部 Zend Framework 2 模块来处理图片和视频。

第七章，*使用 Lucene 进行搜索*，介绍了使用 Zend Framework 的 Lucene 搜索实现。本章首先解释了用户关于安装 ZendSearch\Lucene 模块，然后我们涵盖了实现数据库记录和文档文件搜索的细节。

第八章，*创建一个简单的商店*，介绍了电子商务。在本章中，我们将构建一个简单的在线商店，以展示购物车开发过程中涉及的过程。在本章中，我们将使用 PayPal Express Checkout 作为我们的支付处理器。

第九章，*HTML5 支持*，介绍了 Zend Framework 2 中的 HTML5 支持。与之前版本相比，ZF2 提供了对各种 HTML5 功能的全面支持；本章涵盖了 ZF2 HTML5 支持的两个主要方面——新的输入类型和多个文件上传。

第十章，*构建移动应用程序*，介绍了在 Zend Framework 2 和 Zend Studio 10 的帮助下开发原生移动应用程序。在本章中，我们将学习使用 Zend Framework 构建云连接移动应用程序的基础知识；我们还将了解设置 Zend PHP 开发者云环境。

# 你需要这本书的内容

您需要一个能够运行 Zend Server CE 和 MySQL 的系统。书中涉及的任务性能所需的前提软件在第一章，*开始使用 Zend Framework 2.0* 中进行了介绍。

# 这本书面向的对象

如果您是一位新接触 Zend Framework 的 PHP 开发者，但希望快速上手该产品，这本书就是为您准备的。预期您具备 PHP 面向对象编程的基本知识。

# 惯例

在这本书中，你将发现几个经常出现的标题。

为了清楚地说明如何完成一个程序或任务，我们使用：

# 行动时间 - 标题

1.  行动 1

1.  行动 2

1.  行动 3

指令通常需要一些额外的解释，以便它们有意义，因此它们后面跟着：

## *刚才发生了什么？*

这个标题解释了您刚刚完成的任务或指令的工作原理。

您还会在书中找到一些其他的学习辅助工具，包括：

## 快速问答——标题

这些是旨在帮助您测试自己理解的简短多项选择题。

## 尝试一下英雄——标题

这些实践挑战为您提供了对所学知识的实验想法。

您还会发现多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示：“`TableGateway`类扩展了`AbstractTableGateway`，它实现了`TableGatewayInterface`。”

代码块设置如下：

```php
    // Add Document to index
    $indexDoc = new Lucene\Document();
    $indexDoc->addField($label);
    $indexDoc->addField($owner);
    $indexDoc->addField($fileUploadId);
    $index->addDocument($indexDoc);
  }
  // Commit Index
  $index->commit();
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```php
    // Add Document to index
    $indexDoc = new Lucene\Document();
    $indexDoc->addField($label);
    $indexDoc->addField($owner);
    $indexDoc->addField($fileUploadId);
 $index->addDocument($indexDoc);
  }
  // Commit Index
  $index->commit();
```

任何命令行输入或输出都应如下编写：

```php
$ sudo apt-get install php5-cli 
$ sudo apt-get install git
$ curl -s https://getcomposer.org/installer | php

```

**新** **术语** 和 **重要** **词汇** 以粗体显示。您在屏幕上看到的，例如在菜单或对话框中的文字，将以如下方式显示：“在**选择** **目标** **位置**屏幕上，点击**下一步**以接受默认目标。”

### 注意

警告或重要注意事项以如下框中显示。

### 小贴士

技巧和窍门看起来如下。

# 读者反馈

读者反馈始终受到欢迎。告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题领域有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)购买的 Packt 图书的账户下载所有示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分现有的勘误列表中。

## 盗版

在互联网上对版权材料的盗版是所有媒体中持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送链接到疑似盗版材料至`<copyright@packtpub.com>`与我们联系。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
