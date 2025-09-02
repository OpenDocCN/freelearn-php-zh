# 前言

近年来，响应式编程获得了显著的人气。这得益于 JavaScript Web 框架，如 Angular2 或 React，同时也因为支持多种编程范式的语言（如 JavaScript、Java、Python 或 PHP）中函数式和异步编程的日益流行。

现在，响应式编程与响应式扩展（也称为 ReactiveX 或简称 Rx）紧密相关；这是利用响应式编程最流行的库。值得注意的是，RxJS 5，Rx 的 JavaScript 实现，可能是许多开发者第一次接触响应式编程。在这本书中，我们将主要关注使用 Rx 的 PHP 端口，称为 RxPHP ([`github.com/ReactiveX/RxPHP`](https://github.com/ReactiveX/RxPHP))。

异步编程不是 PHP 开发者通常要处理的内容。实际上，它有点像一片未知的领域，因为在 PHP 中关于这个主题的资源并不多。由于响应式编程与异步编程紧密相连，我们将大量使用事件循环、阻塞和非阻塞代码、子进程、线程和 IPC。

然而，我们的主要目的是学习使用 RxPHP 来掌握响应式扩展和响应式编程。本书包括 RxPHP 1 和 RxPHP 2。所有示例都是为 RxPHP 1 编写的，因为 API 几乎相同，并且在撰写本书时，RxPHP 2 仍在开发中。此外，RxPHP 1 只需要 PHP 5.6+，而 RxPHP 2 则需要 PHP 7+。尽管如此，当 RxPHP 1 和 RxPHP 2 的 API 不同时，我们会适当强调并解释。

# 本书涵盖内容

第一章, *响应式编程简介*，解释了典型编程范式如命令式、异步、函数式、并行和响应式编程的定义。我们将了解在 PHP 中使用函数式编程的先决条件，以及所有这些与响应式扩展的关系。最后，我们将介绍 RxPHP 库作为本书整个内容的首选工具。

第二章, *使用 RxPHP 进行响应式编程*，介绍了在 RxPHP 中使用的响应式编程的基本概念和常用术语。它介绍了 Observables、observers、operators、Subjects 和 disposables 作为任何 Rx 应用程序的构建块。

第三章, *使用 RxPHP 编写 Reddit 阅读器*，基于上一章的知识编写一个基于 RxPHP 的 Reddit 阅读器应用程序。这需要通过 cURL 下载数据，处理用户输入，以及比较 PHP 中与 RxPHP 相关的阻塞和非阻塞代码之间的差异。我们还将一瞥 PHP 中的事件循环的使用。

第四章, *响应式与典型事件驱动方法比较*，展示了为了在实际中应用 Rx，我们需要知道如何将 RxPHP 代码与一些基于 Rx 的现有代码相结合。因此，我们将使用随 Symfony3 框架一起提供的 Event Dispatcher 组件，并扩展其 Rx 功能。

第五章, *测试 RxPHP 代码*，涵盖了测试，这是每个开发过程中的关键部分。除了 PHPUnit，我们还将使用随 RxPHP 一起提供的特殊测试类。我们还将探讨一般性的异步代码测试以及我们需要注意的注意事项。

第六章, *PHP 流 API 和高阶 Observables*，介绍了 PHP 流 API 和事件循环。这两个概念在 RxPHP 中紧密相连，我们将了解为什么以及如何。我们将讨论在使用同一应用程序中的多个事件循环时可能遇到的问题以及 PHP 社区如何试图解决这些问题。我们还将介绍高阶 Observables 的概念，它是 Rx 的更高级功能。

第七章, *实现套接字 IPC 和 WebSocket 服务器/客户端*，展示了为了编写更复杂的异步应用程序，我们将构建一个聊天管理器、服务器和客户端作为三个独立的过程，它们通过 Unix 套接字和 WebSocket 互相通信。我们还将实际使用上一章中的高阶 Observables。

第八章, *RxPHP 和 PHP7 pthreads 扩展中的多播*，介绍了 Rx 中的多播概念以及 RxPHP 为此目的提供的所有组件。我们还将开始使用 PHP7 的 pthreads 扩展。这将使我们能够在多个线程中并行运行我们的代码。

第九章，*使用 pthreads 和 Gearman 进行多线程和分布式计算*，将上一章中关于 pthreads 的知识封装成可重用的组件，这些组件可以与 RxPHP 一起使用。我们还介绍了 Gearman 框架作为在多个进程之间分配工作的方法。最后，我们将比较使用多个线程和进程并行运行任务的优缺点。

第十章，*在 RxPHP 中使用高级操作符和技术*，将重点关注 Rx 中不太常见的原则。这些主要是针对特定任务的先进操作符，但也包括 RxPHP 组件的实现细节和它们在特定用例中的行为，这些是我们应该注意的。

附录，*在 RxJS 中重用 RxPP 技术*，通过实际示例展示了如何处理 RxPHP 或 RxJS 有用的典型用例。我们将看到异步编程在 JavaScript 环境中的应用，并将其与 PHP 进行比较。最后一章还将更详细地介绍 RxJS 5 是什么以及它与 RxPHP 的区别。

# 你需要为这本书准备的东西

本书的大部分主要先决条件是 PHP 5.6+ 解释器和任何文本编辑器。

我们将在示例中使用 Composer ([`getcomposer.org/`](https://getcomposer.org/)) 工具安装所有外部依赖项。对 Composer 和 PHPUnit 的基本知识有帮助，但并非绝对必要。

在后面的章节中，我们还将使用 pthreads PHP 扩展，它需要 PHP 7 或更高版本和 Gearman 作业服务器；这两个都应该适用于所有平台。

此外，对 Unix 环境的一些基本知识（套接字、进程、信号等）也有帮助。

# 本书面向的对象

本书旨在为至少具备平均 PHP 知识的中级开发者编写，他们想了解 PHP 中的异步和响应式编程，特别是响应式扩展。

除了 RxPHP 库之外，本书是框架无关的，因此你不需要了解任何 Web 框架。

所有关于 RxPHP 的主题通常都适用于任何 Rx 实现，因此从 RxPHP 切换到 RxJS，例如，将会非常容易。

# 约定

在本书中，你将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示："每次我们编写一个 Observable，我们都会扩展基类 `Rx\Observable`"。

代码块设置如下：

```php
Rx\Observable::just('{"value":42}')
    ->lift(function() {
        return new JSONDecodeOperator();
    })
    ->subscribe(new DebugSubject());
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```php
use Rx\Observable\IntervalObservable;
class RedditCommand extends Command {
  /** @var \Rx\Subject\Subject */
  private $subject;
  private $interval;
```

任何命令行输入或输出都应如下编写：

```php
$ sleep.php proc1 3 
proc1: 1 
proc1: 2 
proc1: 3

```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，将以如下方式显示：“PHP 必须使用**线程安全**选项编译。”

### 注意

警告或重要注意事项将以如下方式显示。

### 提示

小技巧和技巧将以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书名。如果你在某个主题领域有专业知识，并且对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助你从购买中获得最大收益。

## 下载示例代码

你可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍名称。

1.  选择你想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的地方。

1.  点击**代码下载**。

文件下载完成后，请确保使用最新版本解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

该书的代码包也托管在 GitHub 上[`github.com/PacktPublishing/PHP-Reactive-Programming`](https://github.com/PacktPublishing/PHP-Reactive-Programming)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，请访问[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。查看它们！

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分现有的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个持续存在的问题，跨越所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法副本，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过版权@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
