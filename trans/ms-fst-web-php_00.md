# 前言

更快 Web 可以定义为在所有 Web 技术领域中发展的一系列特质，以加快客户端和服务器之间的任何交易。它还包括可以影响用户对速度感知的 UI 设计原则。因此，理解更快 Web 涉及理解性能、效率和感知性能的概念，并发现构成今天互联网的大部分新基础 Web 技术。

# 本书适合对象

任何希望更好地理解更快 Web 的 Web 开发人员、系统管理员或 Web 爱好者。基本的*Docker*容器技术知识是一个加分项。

# 本书涵盖内容

第一章，*更快 Web-入门*，通过试图更好地理解其正式方面来定义更快 Web，并着手了解如何衡量性能，确定网站或 Web 应用是否属于更快 Web。

第二章，*持续性能分析和监控*，旨在帮助读者学习如何安装和配置性能分析和监控工具，以帮助他们在持续集成（CI）和持续部署（CD）环境中轻松优化 PHP 代码。

第三章，*利用 PHP 7 数据结构和函数的性能*，帮助读者学习如何通过大部分关键优化来利用 PHP 7 的性能提升。它还帮助他们探索更好地理解数据结构和数据类型，以及使用简化的函数如何帮助 PHP 应用程序在其关键执行路径上的全局性能。此外，它介绍了在我们的 PHP 代码中最好避免使用低效结构（如大多数动态结构），以及在优化 PHP 代码时一些功能技术如何立即帮助。

第四章，*异步 PHP 展望未来*，概述了如何通过学习生成器和异步非阻塞代码、使用*POSIX Threads*（`pthreads`）库进行多线程以及使用`ReactPHP`库进行多任务处理来应对输入和输出（I/O）的低延迟。

第五章，*测量和优化数据库性能*，展示了如何测量数据库性能，从简单的测量技术到高级的基准测试工具。

第六章，*高效查询现代 SQL 数据库*，解释了如何使用现代 SQL 技术来优化复杂的 SQL 查询。

[第七章](https://cdp.packtpub.com/mastering_the_faster_web_with_php__mysql__javascript/wp-admin/post.php?post=379&action=edit#post_292)，*JavaScript 和危险驱动开发*，涵盖了 JavaScript 的一些优点和缺点，特别是与代码效率和整体性能有关的部分，以及开发人员应该如何编写安全、可靠和高效的 JavaScript 代码，主要是通过避免“危险驱动开发”。

第八章，*函数式 JavaScript*，介绍了 JavaScript 如何越来越成为一种函数式语言，以及这种编程范式将成为未来性能的一个向量，通过快速查看将帮助改进 JavaScript 应用程序性能的即将推出的语言特性。

第九章，*提升 Web 服务器性能*，介绍了 HTTP/2 协议的相关内容，以及 SPDY 项目是如何实现的，PHP-FPM 和 OPcache 如何帮助提升 PHP 脚本的性能，如何通过设置 Varnish Cache 服务器来使用 ESI 技术，如何使用客户端缓存以及其他更快 Web 工具如何帮助提升 Web 服务器的整体性能。

第十章，*超越性能*，展示了当一切似乎已经完全优化时，通过更好地理解 UI 设计背后的原则，我们仍然可以超越性能。

# 为了充分利用本书

为了运行本书中包含的源代码，我们建议您首先在计算机上安装 Docker（[`docs.docker.com/engine/installation/`](https://docs.docker.com/engine/installation/)）。*Docker*是一个软件容器平台，允许您在隔离和复杂的 chroot-like 环境中轻松连接到计算机的设备。与虚拟机不同，容器不会捆绑完整的操作系统，而只会捆绑运行某些软件所需的二进制文件。您可以在 Windows、Mac 或 Linux 上安装*Docker*。但是需要注意的是，在 macOS 上运行*Docker*时，一些功能，如全功能网络，仍然不可用（[`docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds`](https://docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds)）。

本书中我们将使用的主要*Docker*镜像是*Linux for PHP* 8.1（[`linuxforphp.net/`](https://linuxforphp.net/)），其中包含 PHP 7.1.16 的非线程安全版本和*MariaDB*（*MySQL*）10.2.8（asclinux/linuxforphp-8.1:7.1.16-nts）。要启动主容器，请输入以下命令：

```php
# docker run --rm -it \
> -v ${PWD}/:/srv/fasterweb \
> -p 8181:80 \
> asclinux/linuxforphp-8.1:7.1.16-nts \
> /bin/bash
```

如果您喜欢在优化代码的同时使用多线程技术，可以运行*Linux for PHP*的线程安全版本（asclinux/linuxforphp-8.1:7.0.29-zts）。

此外，您应该`docker commit`任何对容器所做的更改，并创建容器的新镜像，以便以后可以`docker run`。如果您不熟悉 Docker 命令行及其`run`命令，请查看文档[`docs.docker.com/engine/reference/run/`](https://docs.docker.com/engine/reference/run/)。

最后，每当您启动原始的 Linux for PHP 镜像并希望开始使用本书中包含的大多数代码示例时，必须在 Linux for PHP 容器内运行以下三个命令：

```php
# /etc/init.d/mysql start
# /etc/init.d/php-fpm start
# /etc/init.d/httpd start
```

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)注册并直接将文件发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  请在[www.packtpub.com](http://www.packtpub.com/support)登录或注册

1.  选择“支持”选项卡

1.  点击“代码下载和勘误”

1.  在搜索框中输入书名并按照屏幕上的说明进行操作

文件下载后，请确保使用最新版本的解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-the-Faster-Web-with-PHP-MySQL-and-JavaScript`](https://github.com/PacktPublishing/Mastering-the-Faster-Web-with-PHP-MySQL-and-JavaScript)。如果代码有更新，将在现有的 GitHub 存储库中更新。

本书中提供的所有代码示例都可以在代码存储库中的以章节编号命名的文件夹中找到。因此，预计您在每章开始时更改工作目录，以便运行其中给出的代码示例。因此，对于第一章，您预计在容器的 CLI 上输入以下命令：

```php
# mv /srv/www /srv/www.OLD
# ln -s /srv/fasterweb/chapter_1 /srv/www
```

接下来的章节，您预计输入以下命令：

```php
# rm /srv/www
# ln -s /srv/fasterweb/chapter_2 /srv/www
```

接下来的章节也是如此。

我们还有其他代码包来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“在可能的情况下，开发人员应始终优先使用 `const` 而不是 `let` 或 `var`。”

代码块设置如下：

```php
function myJS()
{
    function add(n1, n2)
    {
        let number1 = Number(n1);
        let number2 = Number(n2);

        return number1 + number2;
    }

}
```

任何命令行输入或输出都以以下方式编写：

```php
# php parallel-download.php 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“如果您向页面的末尾滚动，现在应该看到一个 xdebug 部分。”

警告或重要提示会以这种方式出现。技巧和窍门会以这种方式出现。
