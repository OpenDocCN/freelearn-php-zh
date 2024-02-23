# 第一章：了解和设置我们的开发环境

在本章中，我们将介绍运行 Magento 所涉及的技术堆栈以及如何为开发设置一个合适的环境。本章将涵盖以下主题：

+   LAMP 虚拟机

+   设置和使用 VirtualBox

+   设置和使用 Vagrant

+   IDE 和版本控制系统

我们还将学习如何从头开始设置一个 LAMP 虚拟机，以及如何使用 Vagrant 和 Chef 完全自动化这个过程。

# 从头开始的 LAMP

**LAMP**（**Linux，Apache，MySQL 和 PHP**）是一种开源技术解决方案堆栈，用于构建 Web 服务器，也是运行 Magento 的当前标准。

有关更详细的要求清单，请访问[www.magentocommerce.com/system-requirements](http://www.magentocommerce.com/system-requirements)。

### 注意

尽管在撰写本书时，Nginx 在 Magento 开发人员中得到了更广泛的采用，但 Apache2 仍然是社区公认的标准。我们将专注于与它一起工作。

作为开发人员，我们面临着多个挑战和细微差别，如设置和维护我们的开发环境：

+   匹配您的开发和生产环境

+   在不同平台和团队成员之间保持一致的环境

+   设置一个需要几个小时的新环境

+   并非所有开发人员都具有自己设置 LAMP 服务器的知识或经验

我们可以通过 Oracle 的 VirtualBox（[www.virtualbox.org](http://www.virtualbox.org)）来解决前两个问题。VirtualBox 是一个强大且广受欢迎的虚拟化引擎，它将允许我们创建虚拟机（VMs）。VMs 也可以在开发人员之间和所有主要操作系统之间共享。

## 获取 VirtualBox

VirtualBox 是开源的，并且在所有平台上都受支持。可以直接从[www.virtualbox.org/wiki/Downloads](http://www.virtualbox.org/wiki/Downloads)下载。

现在，我们将继续设置一个 Linux 虚拟机。我们选择了 Ubuntu Server 12.04.2 LTS，因为它易于使用并且有广泛的支持。首先，从[www.ubuntu.com/download/server](http://www.ubuntu.com/download/server)下载 ISO 文件；64 位和 32 位版本都可以使用。

要创建一个新的 Linux 虚拟机，请执行以下步骤：

1.  启动**VirtualBox Manager**，并单击左上角的**New**按钮，如下截图所示：![获取 VirtualBox](img/3060OS_01_01_revised.jpg)

1.  一个向导对话框将弹出并引导我们完成创建一个裸虚拟机的步骤。向导将要求我们提供设置虚拟机的基本信息：

+   **VM 名称**：我们应该如何命名我们的虚拟机？让我们将其命名为`Magento_dev 01`。

+   **内存**：这是在我们的 VM 启动时将分配给客户操作系统的系统内存值；对于运行完整的 LAMP 服务器，建议使用 1GB 或更多。

+   **操作系统类型**：这是我们稍后将安装的操作系统类型；在我们的情况下，我们要选择**Linux/Ubuntu**，根据我们的选择，VirtualBox 将启用或禁用某些 VM 选项。

1.  接下来，我们需要指定一个虚拟硬盘。选择**现在创建虚拟硬盘**，如下截图所示：![获取 VirtualBox](img/3060OS_01_02.jpg)

1.  有许多硬盘选项可用，但对于大多数情况，选择**VirtualBox 磁盘映像**（**VDI**）就足够了。这将在我们的主机操作系统上创建一个单个文件。

1.  现在我们需要选择物理驱动器上的存储类型。我们提供以下两个选项：

+   **动态分配**：磁盘映像将随着客户操作系统上的文件数量和使用量的增加而自动增长

+   **固定大小**：此选项将从一开始限制虚拟磁盘的大小

1.  接下来，我们需要指定虚拟硬盘的大小。我们希望根据我们计划使用的 Magento 安装数量来调整大小。

### 注意

一般来说，我们希望每个 Magento 安装至少保留 2GB 的空间，如果我们在同一安装上运行数据库服务器，还需要另外 3GB。这并不是说所有的空间会立即或甚至根本不会被使用，但是一旦考虑到产品图片和缓存文件，Magento 安装可能会使用大量的磁盘空间。

1.  最后，我们只需要点击**创建**按钮。

### 提示

主要区别在于固定大小的硬盘将从一开始就在物理硬盘上保留空间，而动态分配的硬盘将逐渐增长，直到获得指定的大小。

新创建的框将出现在左侧导航菜单中，但在启动我们最近创建的 VM 之前，我们需要进行一些更改，如下所示：

i. 选择我们新创建的 VM，然后点击顶部的**设置**按钮。

ii. 打开**网络**菜单，选择**适配器 2**。我们将把**连接到**设置为**桥接适配器**，因为我们希望将其设置为桥接适配器到我们的主网络接口。这将允许我们远程使用 SSH 连接。

![获取 VirtualBox](img/3060OS_01_03.jpg)

iii. 转到**系统**菜单，更改启动顺序，使 CD/DVD-ROM 首先启动。

iv. 在**存储**菜单中，选择一个空的 IDE 控制器，并挂载我们之前下载的 Ubuntu ISO 镜像。

![获取 VirtualBox](img/3060OS_01_04.jpg)

## 启动我们的虚拟机

此时，我们已经成功安装和配置了我们的 VirtualBox 实例，现在我们已经准备好首次启动我们的新虚拟机。要做到这一点，只需在左侧边栏中选择 VM，然后点击顶部的**启动**按钮。

一个新窗口将弹出，显示 VM 的界面。Ubuntu 将需要几分钟来启动。

一旦 Ubuntu 完成启动，我们将看到两个菜单。第一个菜单将允许我们选择语言，第二个菜单是主菜单，提供了几个选项。在我们的情况下，我们只想继续选择**安装 Ubuntu 服务器**选项。

![启动虚拟机](img/3060OS_01_05.jpg)

现在我们应该看到 Ubuntu 安装向导，它将要求我们选择语言和键盘设置；在选择适合我们国家和语言的设置后，安装程序将继续将所有必要的软件包加载到内存中。这可能需要几分钟。

Ubuntu 将继续配置我们的主网络适配器，一旦自动配置完成，我们将被要求设置虚拟机的主机名。我们可以将主机名保留为默认设置。

![启动虚拟机](img/3060OS_01_06.jpg)

下一个屏幕将要求我们输入用户的全名；在这个例子中，让我们使用`Magento Developer`：

![启动虚拟机](img/3060OS_01_07.jpg)

接下来，我们将被要求创建用户名和密码。让我们使用`magedev`作为我们的用户名：

![启动虚拟机](img/3060OS_01_08.jpg)

让我们使用`magento2013`作为我们的密码：

![启动虚拟机](img/3060OS_01_09.jpg)

在接下来的屏幕上，我们将被要求确认我们的密码并设置正确的时区；输入正确的值后，安装向导将显示以下屏幕，询问我们的分区设置：

![启动虚拟机](img/3060OS_01_10.jpg)

在我们的情况下，我们选择**引导-使用整个磁盘并设置 LVM**；现在让我们确认我们正在分区我们的虚拟磁盘：

![启动虚拟机](img/3060OS_01_11.jpg)

我们将被要求最后一次确认我们的更改；选择**完成分区并将更改写入磁盘**，如下截图所示：

![启动虚拟机](img/3060OS_01_12.jpg)

安装向导将要求我们选择预定义的软件包进行安装；可用选项之一是**LAMP 服务器**。

虽然这非常方便，但我们不想安装预先打包在我们的 Ubuntu CD 中的 LAMP 服务器；我们将手动安装所有 LAMP 组件，以确保它们根据特定需求进行设置，并且与最新的补丁保持最新。

接下来，我们需要一个 SSH 服务器；从列表中选择**OpenSSH 服务器**并点击**继续**：

![启动我们的虚拟机](img/3060OS_01_13.jpg)

现在，Ubuntu 的安装已经完成，它将重新启动到我们新安装的虚拟盒中。

我们几乎准备好继续安装我们环境的其余部分了，但首先我们需要更新我们的软件包管理器存储库定义，登录控制台并运行以下命令：

```php
**$ sudo apt-get update**

```

**APT**代表**高级包装工具**，是大多数 Debian GNU/Linux 发行版中包含的核心库之一；`apt`大大简化了在我们的系统上安装和维护软件的过程。

一旦`apt-get`完成更新所有存储库源，我们可以继续安装我们的 LAMP 服务器的其他组件。

## 安装 Apache2

Apache 是一个 HTTP 服务器。目前，它用于托管超过 60％的网站，并且是运行 Magento 商店的公认标准。有许多在线指南和教程可供调整和优化 Apache2 以提高 Magento 性能。

安装 Apache 就像运行以下命令一样简单：

```php
**$ sudo apt-get install apache2 -y**

```

这将负责为我们安装 Apache2 和所有必需的依赖项。如果一切安装正确，我们现在可以通过打开浏览器并输入`http://192.168.36.1/`来进行测试。

Apache 默认作为服务运行，并且可以使用以下命令进行控制：

```php
**$ sudo apache2ctl stop** 
**$ sudo apache2ctl start** 
**$ sudo apache2ctl restart** 

```

现在，您应该看到 Apache 的默认网页，上面有**It Works!**的消息。

## 安装 PHP

**PHP**是一种服务器端脚本语言，代表**PHP 超文本处理器**。Magento 是基于 PHP5 和 Zend Framework 实现的，我们需要安装 PHP 和一些额外的库才能运行它。

让我们再次使用`apt-get`并运行以下命令来安装`php5`和所有必要的库：

```php
**$ sudo apt-get install php5 php5-curl php5-gd php5-imagick php5-imap php5-mcrypt php5-mysql -y**
**$ sudo apt-get install php-pear php5-memcache -y**
**$ sudo apt-get install libapache2-mod-php5 -y**

```

第一个命令安装了不仅`php5`，还安装了 Magento 连接到我们的数据库和操作图像所需的其他软件包。

第二个命令将安装 PEAR，一个 PHP 包管理器和一个 PHP memcached 适配器。

### 注意

Memcached 是一个高性能的分布式内存缓存系统；这是 Magento 的一个可选缓存系统。

第三个命令安装并设置了 Apache 的`php5`模块。

我们最终可以通过运行以下命令来测试我们的 PHP 安装是否正常工作：

```php
**$ php -v**

```

## 安装 MySQL

MySQL 是许多 Web 应用程序的流行数据库选择，Magento 也不例外。我们需要安装和设置 MySQL 作为开发堆栈的一部分，使用以下命令：

```php
**$ sudo apt-get install mysql-server mysql-client -y**

```

在安装过程中，我们将被要求输入根密码；使用`magento2013`。安装程序完成后，我们应该有一个在后台运行的`mysql`服务实例。我们可以通过尝试使用以下命令连接到`mysql`服务器来测试它：

```php
**$ sudo mysql -uroot -pmagento2013**

```

如果一切安装正确，我们应该看到以下`mysql`服务器提示：

```php
**mysql>**

```

此时，我们有一个完全功能的 LAMP 环境，不仅可以用于开发和处理 Magento 网站，还可以用于任何其他类型的 PHP 开发。

## 将所有内容放在一起

此时，我们已经有了一个基本的 LAMP 设置并正在运行。然而，为了使用 Magento，我们需要进行一些配置更改和额外的设置。

我们需要做的第一件事是创建一个位置来存储我们开发站点的文件，因此我们将运行以下命令：

```php
**$ sudo mkdir -p /srv/www/magento_dev/public_html/**
**$ sudo mkdir /srv/www/magento_dev/logs/**
**$ sudo mkdir /srv/www/magento_dev/ssl/**

```

这将为我们的第一个 Magento 站点创建必要的文件夹结构。现在我们需要通过使用 SVN 来快速获取文件的最新版本。

首先，我们需要在服务器上安装 SVN，使用以下命令：

```php
**$ sudo apt-get install subversion -y**

```

安装程序完成后，打开`magento_dev`目录并运行`svn`命令以获取最新版本的文件：

```php
**$ cd /srv/www/magento_dev** 
**$ sudo svn export --force http://svn.magentocommerce.com/source/branches/1.7 public_html/**

```

我们还需要修复新的 Magento 副本上的一些权限：

```php
**$ sudo chown -R www-data:www-data public_html/**
**$ sudo chmod -R 755 public_html/var/** 
**$ sudo chmod -R 755 public_html/media/** 
**$ sudo chmod -R 755 public_html/app/etc/**

```

接下来，我们需要为 Magento 安装创建一个新的数据库。让我们打开我们的`mysql` shell：

```php
**$ sudo mysql -uroot -pmagento2013**

```

进入`mysql` shell 后，我们可以使用`create`命令，后面应该跟着我们想要创建的实体类型（`database`，`table`）和要创建的数据库名称来创建一个新的数据库：

```php
**mysql> create database magento_dev;**

```

虽然我们可以使用 root 凭据访问我们的开发数据库，但这不是一个推荐的做法，因为这不仅可能危及单个站点，还可能危及整个数据库服务器。MySQL 帐户受权限限制。我们想要创建一组新的凭据，这些凭据只对我们的工作数据库有限的权限：

```php
**mysql> GRANT ALL PRIVILEGES ON magento_dev.* TO 'mage'@'localhost' IDENTIFIED BY 'dev2013$#';**

```

现在，我们需要正确设置 Apache2 并启用一些额外的模块；幸运的是，这个版本的 Apache 带有一组有用的命令：

+   `a2ensite`：这将在`sites-available`和`sites-enabled`文件夹之间创建符号链接，以允许 Apache 服务器读取这些文件。

+   `a2dissite`：这将删除`a2ensite`命令创建的符号链接。这将有效地禁用该站点。

+   `a2enmod`：这用于在`mods-enabled`目录和模块配置文件之间创建符号链接。

+   `a2dismod`：这将从`mods-enabled`目录中删除符号链接。此命令将阻止 Apache 加载该模块。

Magento 使用`mod_rewrite`模块来生成 URL。`mod_rewrite`使用基于规则的重写引擎来实时重写请求的 URL。

我们可以使用`a2enmod`命令启用`mod_rewrite`：

```php
**$ sudo a2enmod rewrite**

```

下一步需要我们在`sites-available`目录下创建一个新的虚拟主机文件：

```php
**$ sudo nano /etc/apache2/sites-available/magento.localhost.com**

```

`nano`命令将打开一个 shell 文本编辑器，我们可以在其中设置虚拟域的配置：

```php
<VirtualHost *:80>
  ServerAdmin magento@locahost.com
  ServerName magento.localhost.com
  DocumentRoot /srv/www/magento_dev/public_html

  <Directory /srv/www/magento_dev/public_html/>
    Options Indexes FollowSymlinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
  </Directory>
  ErrorLog /srv/www/magento_dev/logs/error.log
  LogLevel warn
</VirtualHost>
```

要保存新的虚拟主机文件，请按*Ctrl* + *O*，然后按*Ctrl* + *X*。虚拟主机文件将告诉 Apache 在哪里找到站点文件以及给予它们什么权限。为了使新的配置更改生效，我们需要启用新站点并重新启动 Apache。我们可以使用以下命令来实现：

```php
**$ sudo a2ensite magento.localhost.com**
**$ sudo apache2ctl restart**

```

我们几乎准备好安装 Magento 了。我们只需要通过以下任一方式在主机系统的主机文件中设置本地映射：

+   Windows

i. 用记事本打开`C:\system32\drivers\etc\hosts`

ii. 在文件末尾添加以下行：

```php
192.168.36.1 magento.localhost.com
```

+   Unix/Linux/OSX

i. 使用`nano`打开`/etc/hosts`：

```php
**$ sudo nano /etc/hosts**

```

ii. 在文件末尾添加以下行：

```php
192.168.36.1 magento.localhost.com
```

### 提示

如果您在对主机文件进行必要更改时遇到问题，请访问`http://www.magedevguide.com/hostfile-help`。

现在，我们可以通过在浏览器中打开`http://magento.localhost.com`来安装 Magento。最后，我们应该看到安装向导。按照向导指示的步骤进行操作，您就可以开始使用了！

![把所有东西放在一起](img/3060OS_01_14.jpg)

# 使用 Vagrant 快速上手

之前，我们使用 VM 创建了一个 Magento 安装。虽然使用 VM 为我们提供了一个可靠的环境，但为每个 Magento 分期安装设置我们的 LAMP 仍然可能非常复杂。这对于没有在 Unix/Linux 环境上工作经验的开发人员尤其如此。

如果我们能够获得运行 VM 的所有好处，但是具有完全自动化的设置过程呢？如果我们能够为我们的每个分期网站创建和配置新的 VM 实例？

这是通过使用 Vagrant 结合 Chef 实现的。我们可以创建自动化的虚拟机，而无需对 Linux 或不同的 LAMP 组件有广泛的了解。

### 注意

Vagrant 目前支持 VirtualBox 4.0.x、4.1.x 和 4.2.x。

## 安装 Vagrant

Vagrant 可以直接从[downloads.vagrantup.com](http://downloads.vagrantup.com)下载。此外，它的软件包和安装程序适用于多个平台。下载 Vagrant 后，运行安装。

一旦我们安装了 Vagrant 和 VirtualBox，启动基本 VM 就像在终端或命令提示符中输入以下行一样简单，具体取决于您使用的操作系统：

```php
**$ vagrant box add lucid32 http://files.vagrantup.com/lucid32.box**
**$ vagrant init lucid32**
**$ vagrant up**

```

这些命令将启动一个安装了 Ubuntu Linux 的新 Vagrant 盒子。从这一点开始，我们可以像平常一样开始安装我们的 LAMP。但是，为什么我们要花一个小时为每个项目配置和设置 LAMP 服务器，当我们可以使用 Chef 自动完成呢？Chef 是一个用 Ruby 编写的配置管理工具，可以集成到 Vagrant 中。

为了让刚开始使用 Magento 的开发人员更容易，我在 Github 上创建了一个名为`magento-vagrant`的 Vagrant 存储库，其中包括 Chef 所需的所有必要的食谱和配方。`magento-vagrant`存储库还包括一个新的食谱，将负责特定的 Magento 设置和配置。

为了开始使用`magento-vagrant`，您需要一个 Git 的工作副本。

如果您使用 Ubuntu，请运行以下命令：

```php
**$ sudo apt-get install git-core -y**

```

对于 Windows，我们可以使用本地工具在[`windows.github.com/`](http://windows.github.com/)下载和管理我们的存储库。

无论您使用的操作系统是什么，我们都需要在本地文件系统中检出此存储库的副本。我们将使用`C:/Users/magedev/Documents/magento-vagrant/`来下载和保存我们的存储库；在`magento-vagrant`中，我们将找到以下文件和目录：

+   食谱

+   `data_bags`

+   公共

+   `.vagrant`

+   `Vagrantfile`

`magento-vagrant`存储库包括我们开发环境的每个组件的食谱，一旦我们启动新的 Vagrant 盒子，它们将自动安装。

现在唯一剩下的事情就是设置我们的开发站点。通过使用 Vagrant 和 Chef，向我们的 Vagrant 安装添加新的 Magento 站点的过程已经变得简化。

在`data_bags`目录中，我们有一个文件用于 Vagrant 盒子中每个 Magento 安装；默认存储库中包含 Magento CE 1.7 的示例安装。

对于每个站点，我们需要创建一个包含 Chef 所需的所有设置的新 JSON 文件。让我们看一下`magento-vagrant`默认文件，可以在位置`C:/Users/magedev/Documents/magento-vagrant/data_bags/sites/default.json`找到：

```php
{
    "id": "default",
    "host": "magento.localhost.com",
    "repo": [
        "url": "svn.magentocommerce.com/source/branches/1.7",
        "revision": "HEAD"  
     ],
   "database": [
      "name": "magento_staging",
      "username": "magento",
      "password": "magento2013$"
   ]
}
```

这将自动使用 Magento 存储库中的最新文件设置 Magento 安装。

向我们的 Vagrant 盒子添加新站点只是添加一个相应站点的新 JSON 文件并重新启动 Vagrant 盒子的问题。

现在我们有一个运行中的 Magento 安装，让我们来选择一个合适的**集成开发环境**（**IDE**）。

# 选择一个 IDE

选择合适的 IDE 主要是个人开发者口味的问题。然而，选择合适的 IDE 对于 Magento 开发者来说可能是至关重要的。

IDE 的挑战主要来自 Magento 对工厂名称的广泛使用。这使得某些功能的实现，如代码完成（也称为智能感知），变得困难。目前，有两个 IDE 在其对 Magento 的本地支持方面表现出色-NetBeans 和 PhpStorm。

尽管 NetBeans 是开源的，并且已经存在很长时间，但 PhpStorm 一直占据上风，并得到了 Magento 社区的更多支持。

此外，最近发布的 Magicento 插件，专门用于扩展和集成 Magento 到 PhpStorm 中，已成为当前可用选项中最佳选择。

# 使用版本控制系统

Magento 代码库非常庞大，包括超过 7,000 个文件和近 150 万行代码。因此，使用版本控制系统不仅是一种良好的实践，也是一种必要性。

版本控制系统用于跟踪多个文件和多个开发人员之间的更改；通过使用版本控制系统，我们可以获得非常强大的工具。

在几种可用的版本控制系统中（Git、SVN、Mercurial），Git 由于其简单性和灵活性而值得特别关注。通过在 Git 托管服务 Github 上发布即将推出的 Magento 2 版本，Magento 核心开发团队已经认识到 Git 在 Magento 社区中的重要性。

### 注意

有关 Magento2 的更多信息，请访问[`github.com/magento/magento2`](https://github.com/magento/magento2)。

Github 现在包括一个特定于 Magento 的`.gitignore`文件，它将忽略 Magento 核心中的所有文件，只跟踪我们自己的代码。

也就是说，在处理 Magento 项目时，有几个版本控制概念需要牢记：

+   **分支**：这允许我们在不影响主干（稳定版本）的情况下工作新功能。

+   **合并**：这用于将代码从一个地方移动到另一个地方。通常，这是在开发分支准备好移动到生产环境时从开发分支到主干进行的。

+   **标记**：这用于创建发布的快照。

# 总结

在这第一章中，我们学习了如何设置和使用 LAMP 环境，在多个平台上设置开发环境，创建和配置 Vagrant 虚拟机，使用 Chef 配方以及使用 Magento 开发的版本控制系统。

拥有适当的环境是开始为 Magento 开发的第一步，也是我们 Magento 工具箱的一个组成部分。

现在我们已经设置好并准备好使用开发环境，是时候深入了解 Magento 的基本概念了；这些概念将为我们提供开发 Magento 所需的工具和知识。
