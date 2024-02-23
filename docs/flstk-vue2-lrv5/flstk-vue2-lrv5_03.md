# 设置 Laravel 开发环境

在本书的前两章中，我们介绍了 Vue.js。您现在应该对其基本功能非常熟悉。在本章中，我们将启动一个 Laravel 开发环境，准备构建 Vuebnb 的后端。

本章涵盖的主题：

+   Laravel 简介

+   设置 Homestead 虚拟开发环境

+   配置 Homestead 以托管 Vuebnb

# Laravel

Laravel 是一个用于构建强大的 Web 应用程序的 PHP 开源 MVC 框架。Laravel 目前版本为 5.5，是最受欢迎的 PHP 框架之一，因其优雅的语法和强大的功能而备受喜爱。

Laravel 适用于创建各种基于 Web 的项目，例如以下项目：

+   具有用户认证的网站，如客户门户或社交网络

+   Web 应用程序，如图像裁剪器或监控仪表板

+   Web 服务，如 RESTful API

在本书中，我假设您对 Laravel 有基本的了解。您应该熟悉安装和设置 Laravel，并熟悉其核心功能，如路由、视图和中间件。

如果您是 Laravel 新手或认为自己可能需要温习一下，您应该花一两个小时阅读 Laravel 的优秀文档，然后再继续阅读本书：[`laravel.com/docs/5.5/`](https://laravel.com/docs/5.5/)。

# Laravel 和 Vue

Laravel 可能看起来像一个庞大的框架，因为它包括了构建几乎任何类型的 Web 应用程序的功能。然而，在幕后，Laravel 实际上是许多独立模块的集合，其中一些是作为 Laravel 项目的一部分开发的，一些来自第三方作者。Laravel 之所以伟大，部分原因在于它对这些组成模块的精心策划和无缝连接。

自 Laravel 5.3 版本以来，Vue.js 一直是 Laravel 安装中包含的默认前端框架。没有官方原因说明为什么选择 Vue 而不是其他值得选择的选项，如 React，但我猜想是因为 Vue 和 Laravel 分享相同的理念：简单和对开发者体验的重视。

无论出于什么原因，Vue 和 Laravel 都提供了一个非常强大和灵活的全栈框架，用于开发 Web 应用程序。

# 环境

我们将使用 Laravel 5.5 作为 Vuebnb 的后端。这个版本的 Laravel 需要 PHP 7、几个 PHP 扩展和以下软件：

+   Composer

+   一个 Web 服务器，如 Apache 或 Nginx

+   数据库，如 MySQL 或 MariaDB

Laravel 的完整要求列表可以在安装指南中找到：[`laravel.com/docs/5.5#installation`](https://laravel.com/docs/5.5#installation)。

我强烈建议您使用*Homestead*开发环境，而不是在计算机上手动安装 Laravel 的要求，因为 Homestead 已经预先安装了所有您需要的东西。

# Homestead

Laravel Homestead 是一个虚拟的 Web 应用程序环境，运行在 Vagrant 和 VirtualBox 上，可以在任何 Windows、Mac 或 Linux 系统上运行。

使用 Homestead 将为您节省从头开始设置开发环境的麻烦。它还将确保您拥有与我使用的相同环境，这将使您更容易跟随本书的内容。

如果您的计算机上没有安装 Homestead，请按照 Laravel 文档中的说明进行操作：[`laravel.com/docs/5.5/homestead`](https://laravel.com/docs/5.5/homestead)。使用默认配置选项。

安装了 Homestead 并使用`vagrant up`命令启动了 Vagrant 虚拟机后，您就可以继续了。

# Vuebnb

在第二章中，*原型设计 Vuebnb，您的第一个 Vue.js 项目*，我们制作了 Vuebnb 前端的原型。原型是从一个单独的 HTML 文件创建的，我们直接从浏览器加载。

现在我们将开始处理全栈 Vuebnb 项目，原型很快将成为其中的关键部分。这个主要项目将是一个完整的 Laravel 安装，带有 Web 服务器和数据库。

# 项目代码

如果您还没有这样做，您需要从 GitHub 克隆代码库到您的计算机上。在第一章的*代码库*部分中给出了说明，*Hello Vue - Vue.js 简介*。

代码库中的`vuebnb`文件夹包含我们现在想要使用的项目代码。切换到此文件夹并列出内容：

```php
$ cd vuebnb
$ ls -la
```

文件夹内容应该如下所示：

![](img/ae64ccdf-f60f-4333-b560-70d02ca66d37.png)图 3.1。vuebnb 项目文件

# 共享文件夹

`Homestead.yaml`文件的`folders`属性列出了您希望在计算机和 Homestead 环境之间共享的所有文件夹。

确保代码库与 Homestead 共享，以便我们在本章后期可以从 Homestead 的 Web 服务器上提供 Vuebnb。

`~/Homestead/Homestead.yaml`*：*

```php
folders:
  - map: /Users/anthonygore/Projects/Full-Stack-Vue.js-2-and-Laravel-5
    to: /home/vagrant/projects
```

# 终端命令

本书中的所有进一步的终端命令都将相对于项目目录给出，即*vuebnb*，除非另有说明。

然而，由于项目目录在主机计算机和 Homestead 之间共享，终端命令可以从这两个环境中的任何一个运行。

Homestead 可以避免您在主机计算机上安装任何软件。但如果您不这样做，许多终端命令可能无法正常工作，或者在主机环境中可能无法正常工作。例如，如果您的主机计算机上没有安装 PHP，您就无法从中运行 Artisan 命令：

```php
$ php artisan --version
-bash: php: command not found
```

如果您是这种情况，您需要首先通过 SSH 连接在 Homestead 环境中运行这些命令：

```php
$ cd ~/Homestead
$ vagrant ssh
```

然后，切换到操作系统中的项目目录，同样的终端命令现在将起作用：

```php
$ cd ~/projects/vuebnb
$ php artisan --version
Laravel Framework 5.5.20
```

从 Homestead 运行命令的唯一缺点是由于 SSH 连接而变慢。我将让您决定您更愿意使用哪一个。

# 环境变量

Laravel 项目需要在`.env`文件中设置某些环境变量。现在通过复制环境文件示例来创建一个：

```php
$ cp .env.example .env
```

通过运行此命令生成应用程序密钥：

```php
$ php artisan key:generate
```

我已经预设了大多数其他相关的环境变量，所以除非您已经按照我不同的方式配置了 Homestead，否则您不需要更改任何内容。

# Composer 安装

要完成安装过程，我们必须运行`composer install`来下载所有所需的软件包：

```php
$ composer install
```

# 数据库

我们将使用关系数据库来在后端应用程序中持久保存数据。Homestead 默认情况下运行 MySQL；您只需在`.env`文件中提供配置以在 Laravel 中使用它。默认配置将在没有进一步更改的情况下工作。

`.env`：

```php
DB_CONNECTION=mysql
DB_HOST=192.168.10.10
DB_PORT=3306
DB_DATABASE=vuebnb
DB_USERNAME=homestead
DB_PASSWORD=secret
```

无论您为数据库选择什么名称（即`DB_DATABASE`的值），都要确保将其添加到`Homestead.yaml`文件中的`databases`数组中。

`~/Homestead/Homestead.yaml`：

```php
databases:
  ... - vuebnb
```

# 提供项目

主要的 Vuebnb 项目现在已安装。让我们让 Web 服务器在本地开发域`vuebnb.test`上提供它。

在 Homestead 配置文件中，将`vuebnb.test`映射到项目的`public`文件夹。

`~/Homestead/Homestead.yaml`：

```php
sites:
  ... - map: vuebnb.test
    to: /home/vagrant/vuebnb/public
```

# 本地 DNS 条目

我们还需要更新计算机的主机文件，以便它理解`vuebnb.test`和 Web 服务器的 IP 之间的映射。Web 服务器位于 Homestead 框中，默认情况下 IP 为`192.168.10.10`。

要在 Mac 上配置这个，打开您的主机文件`/etc/hosts`，在文本编辑器中添加这个条目：

```php
192.168.10.10 vuebnb.test
```

在 Windows 系统上，hosts 文件通常可以在`C:\Windows\System32\Drivers\etc\hosts`找到。

# 访问项目

配置完成后，我们现在可以从`Homestead`目录中运行`vagrant provision`来完成设置：

```php
$ cd ~/Homestead
$ vagrant provision
# The next command will return you to the project directory
$ cd -
```

当配置过程完成时，我们应该能够在浏览器中输入`http://vuebnb.test`来看到我们的网站运行：

![](img/a6d139d3-da0b-4ba3-8c5d-18e0b9e26642.png)图 3.2. Laravel 欢迎视图

现在我们准备开始开发 Vuebnb！

# 总结

在这个简短的章节中，我们讨论了开发 Laravel 项目的要求。然后安装并配置了 Homestead 虚拟开发环境来托管我们的主要项目 Vuebnb。

在下一章中，我们将通过构建一个 Web 服务来为 Vuebnb 的前端提供数据，开始我们的主要项目工作。
