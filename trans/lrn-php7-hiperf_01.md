# 第一章。设置环境

PHP 7 终于发布了。很长一段时间以来，PHP 社区一直在谈论它，而且仍未停止。PHP 7 的主要改进是其性能。很长一段时间以来，PHP 社区在大规模应用程序中面临性能问题。甚至一些高流量的小型应用程序也面临性能问题。服务器资源增加了，但并没有太大帮助，因为最终瓶颈是 PHP 本身。使用了不同的缓存技术，如 APC，这有所帮助。然而，社区仍然需要一个能够在应用程序性能达到顶峰时提升性能的 PHP 版本。这就是 PHPNG 的用武之地。

**PHPNG**代表**PHP 下一代**。它是一个完全独立的分支，主要针对性能。有些人认为 PHPNG 是**JIT**（**即时编译**），但实际上，PHPNG 是基于经过高度优化的**Zend 引擎**重构而成的。PHPNG 被用作 PHP 7 开发的基础，根据官方 PHP 维基页面，PHPNG 分支现在已经合并到主分支中。

在开始构建应用程序之前，应该完成并配置好开发环境。在本章中，我们将讨论在不同系统上设置开发环境，如 Windows 和不同版本的 Linux。

我们将涵盖以下主题：

+   设置 Windows

+   设置 Ubuntu 或 Debian

+   设置 CentOS

+   设置 Vagrant

其他所有环境都可以跳过，我们可以设置我们将使用的环境。

# 设置 Windows

有许多可用的工具，它们在 Windows 上捆绑了 Apache、PHP 和 MySQL，提供了简单的安装，并且非常易于使用。这些工具中的大多数已经提供了对 PHP 7 与 Apache 的支持，例如 XAMPP、WAMPP 和 EasyPHP。EasyPHP 是唯一一个还提供对**NGINX**的支持，并提供了从 NGINX 切换到 Apache 或从 Apache 切换到 Nginx 的简单步骤。

### 注意

XAMPP 也适用于 Linux 和 Mac OS X。但是，WAMP 和 EasyPHP 仅适用于 Windows。这三种工具中的任何一种都可以用于本书，但我们建议使用 EasyPHP，因为它支持 NGINX，并且在本书中，我们主要使用 NGINX。

可以使用这三种工具中的任何一种，但我们需要更多地控制我们的 Web 服务器工具的每个元素，因此我们将单独安装 NGINX、PHP 7 和 MySQL，然后将它们连接在一起。

### 注意

可以从[`nginx.org/en/download.html`](http://nginx.org/en/download.html)下载 NGINX Windows 二进制文件。我们建议使用稳定版本，尽管使用主线版本也没有问题。可以从[`windows.php.net/download/`](http://windows.php.net/download/)下载 PHP Windows 二进制文件。根据您的系统下载 32 位或 64 位的*非线程安全*版本。

执行以下步骤：

1.  下载信息框中提到的 NGINX 和 PHP Windows 二进制文件。将 NGINX 复制到合适的目录。例如，我们有一个完全独立的 D 盘用于开发目的。将 NGINX 复制到这个开发驱动器或任何其他目录。现在，将 PHP 复制到 NGINX 目录或任何其他安全文件夹位置。

1.  在 PHP 目录中，将有两个`.ini`文件，`php.ini-development`和`php.ini-production`。将其中一个重命名为`php.ini`。PHP 将使用这个配置文件。

1.  按住*Shift*键并在 PHP 目录中右键单击以打开命令行窗口。命令行窗口将在相同的位置路径中打开。发出以下命令启动 PHP：

```php
    php-cgi –b 127.0.0.1:9000
    ```

`-b`选项启动 PHP 并绑定到外部**FastCGI**服务器的路径。上述命令将 PHP 绑定到回环`127.0.0.1`IP 的端口`9000`。现在，PHP 可以在这个路径上访问。

1.  要配置 NGINX，打开`nginx_folder/conf/nginx.conf`文件。首先要做的是在服务器块中添加 root 和 index，如下所示：

```php
    server {
     **root html;**
     **index index.php index.html index.htm;**

    ```

### 提示

**下载示例代码**

您可以从 http://www.packtpub.com 的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，可以访问 http://www.packtpub.com/support 并注册，以便将文件直接发送到您的电子邮件。

您可以按照以下步骤下载代码文件：

+   使用您的电子邮件地址和密码登录或注册到我们的网站。

+   将鼠标指针悬停在顶部的 SUPPORT 选项卡上。

+   单击代码下载和勘误。

+   在搜索框中输入书名。

+   选择您要下载代码文件的书。

+   从下拉菜单中选择您购买本书的地方。

+   单击代码下载。

下载文件后，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 的 WinRAR / 7-Zip

+   Mac 的 Zipeg / iZip / UnRarX

+   Linux 的 7-Zip / PeaZip

1.  现在，我们需要配置 NGINX 以在启动时使用 PHP 作为 FastCGI 的路径。在`nginx.conf`文件中，取消注释以下位置块以用于 PHP：

```php
    location ~ \.php$ {
      fastcgi_pass    127.0.0.1:9000;
      fastcgi_param    SCRIPT_FILENAME **complete_path_webroot_folder$fastcgi_script_name;**
    include    fastcgi_params;
    }
    ```

注意`fastcgi_param`选项。突出显示的`complete_path_webroot_folder`路径应该是`nginx`文件夹内 HTML 目录的绝对路径。假设您的 NGINX 放置在`D:\nginx`路径，那么`HTML`文件夹的绝对路径将是`D:\nginx\html`。但是，对于前面的`fastcgi_param`选项，`\`应该替换为`/`。

1.  现在，在 NGINX 文件夹的根目录中发出以下命令重新启动 NGINX：

```php
    **nginx –s restart**

    ```

1.  在重新启动 NGINX 后，打开浏览器，输入 Windows 服务器或机器的 IP 或主机名，我们将看到 NGINX 的欢迎消息。

1.  现在，要验证 PHP 安装并与 NGINX 一起工作，请在 webroot 中创建一个`info.php`文件，并输入以下代码：

```php
    <?php
      phpinfo();
    ?>
    ```

1.  现在，在浏览器中访问[your_ip/info.php](http://your_ip/info.php)，我们将看到一个充满 PHP 和服务器信息的页面。恭喜！我们已经成功配置了 NGINX 和 PHP，使它们完美地配合工作。

### 注意

在 Windows 和 Mac OS X 上，我们建议您使用安装有 Linux 版本的所有工具的虚拟机，以获得服务器的最佳性能。在 Linux 中管理一切很容易。有现成的 vagrant boxes 可供使用。另外，可以在[`puphpet.com`](https://puphpet.com)上制作包括 NGINX、Apache、PHP 7、Ubuntu、Debian 或 CentOS 等工具的自定义虚拟机配置，这是一个易于使用的 GUI。另一个不错的工具是 Laravel Homestead，这是一个带有很棒工具的**Vagrant** box。

# 设置 Debian 或 Ubuntu

Ubuntu 是从 Debian 派生的，因此对于 Ubuntu 和 Debian，过程是相同的。我们将使用 Debian 8 Jessie 和 Ubuntu 14.04 Server LTS。相同的过程也适用于两者的桌面版本。

首先，为 Debian 和 Ubuntu 添加存储库。

## Debian

截至我们撰写本书的时间，Debian 没有为 PHP 7 提供官方存储库。因此，对于 Debian，我们将使用`dotdeb`存储库来安装 NGINX 和 PHP 7。执行以下步骤：

1.  打开`/etc/apt/sources.list`文件，并在文件末尾添加以下两行：

```php
    deb http://packages.dotdeb.org jessie all
    deb-src http://packages.dotdeb.org jessie all
    ```

1.  现在，在终端中执行以下命令：

```php
    **wget https://www.dotdeb.org/dotdeb.gpg**
    **sudo apt-key add dotdeb.gpg**
    **sudo apt-get update**

    ```

前两个命令将向 Debian 添加`dotdeb`存储库，最后一个命令将刷新源的缓存。

## Ubuntu

截至撰写本书的时间，Ubuntu 也没有在官方存储库中提供 PHP 7，因此我们将使用第三方存储库进行 PHP 7 的安装。执行以下步骤：

1.  在终端中运行以下命令：

```php
    **sudo add-apt-repository ppa:ondrej/php**
    **sudo apt-get update**

    ```

1.  现在，存储库已添加。让我们安装 NGINX 和 PHP 7。

### 注意

其余的过程对于 Debian 和 Ubuntu 大部分是相同的，所以我们不会单独列出它们，就像我们为添加存储库部分所做的那样。

1.  要安装 NGINX，请在终端中运行以下命令（Debian 和 Ubuntu）：

```php
    **sudo apt-get install nginx**

    ```

1.  安装成功后，可以通过输入 Debian 或 Ubuntu 服务器的主机名和 IP 来验证。如果看到类似下面的屏幕截图，那么我们的安装是成功的：![Ubuntu](img/B05225_01_01.jpg)

以下是三个有用的 NGINX 命令列表：

+   `service nginx start`：这将启动 NGINX 服务器

+   `service nginx restart`：这将重新启动 NGINX 服务器

+   `service nginx stop`：这将停止 NGINX 服务器

1.  现在，是时候通过发出以下命令来安装 PHP 7 了：

```php
    **sudo apt-get install php7.0 php7.0-fpm php7.0-mysql php7.0-mcrypt php7.0-cli**

    ```

这将安装 PHP 7 以及其他提到的模块。此外，我们还为命令行目的安装了 PHP Cli。要验证 PHP 7 是否已正确安装，请在终端中发出以下命令：

```php
    **php –v**

    ```

1.  如果显示 PHP 版本以及其他一些细节，如下面的屏幕截图所示，那么 PHP 已经正确安装：![Ubuntu](img/B05225_01_02.jpg)

1.  现在，我们需要配置 NGINX 以与 PHP 7 一起工作。首先，通过在终端中使用以下命令将 NGINX 默认配置文件`/etc/nginx/sites-available/default`复制到`/etc/nginx/sites-available/www.packt.com.conf`：

```php
    **cd /etc/nginx/sites-available**
    **sudo cp default www.packt.com.conf**
    **sudo ln –s /etc/nginx /sites-available/www.packt.com.conf /etc/ nginx/sites-enabled/www.packt.com.conf**

    ```

首先，我们复制了默认配置文件，创建了另一个虚拟主机配置文件`www.packt.com.conf`，然后在 sites-enabled 文件夹中为这个虚拟主机文件创建了一个符号链接文件。

### 注意

为每个虚拟主机创建一个与域名相同的配置文件是一个很好的做法，这样可以很容易地被其他人识别。

1.  现在，打开`/etc/nginx/sites-available/www.packt.com.conf`文件，并添加或编辑高亮显示的代码，如下所示：

```php
    server {
      server_**name your_ip:80**;
      root /var/www/html;
      index **index.php** index.html index.htm;
      **location ~ \.php$ {**
     **fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;**
     **fastcgi_index index.php;**
     **include fastcgi_params;**
     **}**
    }
    ```

上述配置不是一个完整的配置文件。我们只复制了那些重要的配置选项，我们可能想要更改的选项。

在上述代码中，我们的 webroot 路径是`/var/www/html`，我们的 PHP 文件和其他应用程序文件将放在这里。在索引配置选项中，添加`index.php`，这样如果 URL 中没有提供文件，NGINX 就可以查找并解析`index.php`。

我们添加了一个用于 PHP 的位置块，其中包括一个`fastcgi_pass`选项，该选项具有指向 PHP7 FPM 套接字的路径。在这里，我们的 PHP 运行在 Unix 套接字上，比 TCP/IP 更快。

1.  在进行这些更改后，重新启动 NGINX。现在，为了测试 PHP 和 NGINX 是否正确配置，创建一个`info.php`文件放在`webroot`文件夹的根目录，并在其中放入以下代码：

```php
    <?php
      phpinfo();
     ?>
    ```

1.  现在，在浏览器中输入`server_ip/info.php`，如果看到一个 PHP 配置页面，那么恭喜！PHP 和 NGINX 都已正确配置。

### 注意

如果 PHP 和 NGINX 在同一系统上运行，那么 PHP 会监听端口`9000`的环回 IP。端口可以更改为任何其他端口。如果我们想要在 TCP/IP 端口上运行 PHP，那么在`fastcgi_pass`中，我们将输入`127.0.0.1:9000`。

现在，让我们安装**Percona Server**。Percona Server 是 MySQL 的一个分支，经过优化以获得更高的性能。我们将在第三章中更多地了解 Percona Server，*提高 PHP 7 应用程序性能*。现在，让我们通过以下步骤在 Debian/Ubuntu 上安装 Percona Server：

1.  首先，让我们通过在终端中运行以下命令将 Percona Server 仓库添加到我们的系统中：

```php
    **sudo wget https://repo.percona.com/apt/percona-release_0.1-3.$(lsb_release -sc)_all.deb**
    **sudo dpkg -i percona-release_0.1-3.$(lsb_release -sc)_all.deb**

    ```

第一个命令将从 Percona 仓库下载软件包。第二个命令将安装已下载的软件包，并在`/etc/apt/sources.list.d/percona-release.list`创建一个`percona-release.list`文件。

1.  现在，通过在终端中执行以下命令来安装 Percona Server：

```php
    **sudo apt-get update**

    ```

1.  现在，通过发出以下命令来安装 Percona Server：

```php
    **sudo apt-get install percona-server-5.5**

    ```

安装过程将开始。下载需要一段时间。

### 注意

为了本书的目的，我们将安装 Percona Server 5.5。也可以安装 Percona Server 5.6，而且不会出现任何问题。

在安装过程中，将要求输入`root`用户的密码，如下图所示：

![Ubuntu](img/B05225_01_03.jpg)

输入密码是可选的但建议的。输入密码后，在下一个屏幕上重新输入密码。安装过程将继续。

1.  安装完成后，可以使用以下命令验证 Percona Server 的安装：

```php
    **mysql –-version**

    ```

它将显示 Percona Server 的版本。如前所述，Percona Server 是 MySQL 的一个分支，因此可以使用相同的 MySQL 命令、查询和设置。

# 设置 CentOS

CentOS 是**Red Hat Enterprise Linux**（**RHEL**）的一个分支，代表**Community Enterprise Operating System**。它是服务器上广泛使用的操作系统，特别是由托管公司提供共享托管服务。

让我们首先为我们的开发环境配置 CentOS。执行以下步骤：

## 安装 NGINX

1.  首先，我们需要将 NGINX RPM 添加到我们的 CentOS 安装中，因为 CentOS 没有提供任何默认的 NGINX 仓库。在终端中输入以下命令：

```php
    **sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm**

    ```

这将向 CentOS 添加 NGINX 仓库。

1.  现在，输入以下命令以查看可供安装的 NGINX 版本：

```php
    **sudo yum --showduplicates list Nginx**

    ```

这将显示最新的稳定版本。在我们的情况下，它显示 NGINX 1.8.0 和 NGINX 1.8.1。

1.  现在，让我们使用以下命令安装 NGINX：

```php
    **sudo yum install Nginx**

    ```

这将安装 NGINX。

1.  在 CentOS 上，NGINX 在安装或重新启动后不会自动启动。因此，首先，我们将使用以下命令使 NGINX 在系统重新启动后自动启动：

```php
    **systemctl enable Nginx.service**

    ```

1.  现在，让我们通过输入以下命令来启动 NGINX：

```php
    **systemctl start Nginx.service**

    ```

1.  然后，打开浏览器，输入 CentOS 服务器的 IP 或主机名。如果您看到与我们在 Debian 章节中看到的欢迎屏幕相同的屏幕，则 NGINX 已成功安装。

要检查安装了哪个版本的 NGINX，请在终端中输入以下命令：

```php
    **Nginx –v**

    ```

在我们的服务器上，安装的 NGINX 版本是 1.8.1。

现在，我们的 Web 服务器已准备就绪。

## 安装 PHP 7

1.  下一步是安装 PHP 7 FPM 并配置 NGINX 和 PHP 7 一起工作。在撰写本书时，PHP 7 没有打包在官方的 CentOS 仓库中。因此，我们有两种选择来安装 PHP 7：要么从源代码构建，要么使用第三方仓库。从源代码构建有点困难，所以让我们选择简单的方式，使用第三方仓库。

### 注意

对于本书，我们将使用 webtatic 仓库来安装 PHP 7，因为它们为新版本提供快速更新。还有一些其他仓库，只要它能正常工作，读者可以自行选择使用任何仓库。

1.  现在，让我们通过输入以下命令向我们的 CentOS 仓库添加 webtatic 仓库：

```php
    **rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm**
    **rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm**

    ```

1.  成功添加仓库后，输入以下命令以查看可供安装的版本：

```php
    **sudo yum –showduplicates list php70w**

    ```

在我们的情况下，可以安装 PHP 7.0.3。

1.  现在，输入以下命令以安装 PHP 7 以及可能需要的一些模块：

```php
    **sudo yum install php70w php70w-common php70w-cli php70w-fpm php70w-mysql php70w-opcache php70w-mcrypt**

    ```

1.  这将安装核心 PHP 7 和一些可用于 PHP 7 的模块。如果需要其他模块，可以轻松安装；但是首先搜索以检查其是否可用。在终端中输入以下命令以查看所有可用的 PHP 7 模块：

```php
    **sudo yum search php70w-**

    ```

将显示所有可用的 PHP 7 模块的长列表。

1.  现在，假设我们要安装 PHP 7 gd 模块；输入以下命令：

```php
    **sudo yum install php70w-gd**

    ```

这将安装 gd 模块。可以使用相同的命令安装多个模块，并通过空格分隔每个模块，就像我们在最初安装 PHP 时所做的那样。

现在，要检查安装了哪个版本的 PHP，请输入以下命令：

```php
    **php –v**

    ```

在我们的情况下，安装了 PHP 7.0.3。

1.  要启动、停止和重新启动 PHP，请在终端中输入以下命令：

```php
    **sudo systemctl start php-fpm**
    **sudo systemctl restart php-fpm**
    **sudo systemctl stop php-fpm**

    ```

1.  现在，让我们配置 NGINX 以使用 PHP FPM。使用`vi`、`nano`或您选择的任何其他编辑器打开位于`/etc/Nginx/conf.d/default.conf`的默认 NGINX 虚拟主机文件。现在，请确保服务器块中设置了两个选项，如下所示：

```php
    server {
        listen  80;
        server_name  localhost;
     **root   /usr/share/nginx/html;**
    **index  index.php index.html index.htm;**

    ```

`root`选项表示我们的网站源代码文件将放置的 Web 文档根目录。Index 表示将与扩展名一起加载的默认文件。如果找到任何这些文件，默认情况下将执行它们，而不管 URL 中提到的任何文件。

1.  NGINX 中的下一个配置是用于 PHP 的位置块。以下是 PHP 的配置：

```php
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME$document_root$fastcgi_script_name;
          include fastcgi_params;
        }
    ```

上述块是最重要的配置，因为它使 NGINX 能够与 PHP 通信。行`fastcgi_pass 127.0.0.1:9000`告诉 NGINX，PHP FPM 可以在端口`9000`上的`127.0.0.1`环回 IP 上访问。其余细节与我们讨论 Debian 和 Ubuntu 的内容相同。

1.  现在，为了测试我们的安装，我们将创建一个名为`info.php`的文件，其中包含以下内容：

```php
    <?php
      phpinfo();
    ?>
    ```

保存文件后，输入`http://server_ip/info.php`或`http://hostname/info.php`，我们将得到一个包含有关 PHP 的完整信息的页面。如果您看到此页面，恭喜！PHP 与 NGINX 一起运行。

## 安装 Percona Server

1.  现在，我们将在 CentOS 上安装 Percona Server。安装过程相同，只是它有一个单独的存储库。要将 Percona Server 存储库添加到 CentOS，请在终端中执行以下命令：

```php
    **sudo yum install http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm**

    ```

存储库安装完成后，将显示一条消息，指示安装完成。

1.  现在，为了测试存储库，发出以下命令，它将列出所有可用的 Percona 软件包：

```php
    **sudo yum search percona**

    ```

1.  要安装 Percona Server 5.5，请在终端中发出以下命令：

```php
    **sudo yum install Percona-Server-server-55**

    ```

安装过程将开始。其余的过程与 Debian/Ubuntu 相同。

1.  安装完成后，将看到完成消息。

# 设置 Vagrant

Vagrant 是开发人员用于开发环境的工具。Vagrant 提供了一个简单的命令行界面，用于设置带有所有所需工具的虚拟机。Vagrant 使用称为 Vagrant Boxes 的框，可以具有 Linux 操作系统和根据此框的其他工具。Vagrant 支持 Oracle VM VirtualBox 和 VMware。为了本书的目的，我们将使用 VirtualBox，我们假设它也安装在您的机器上。

Vagrant 有几个用于 PHP 7 的框，包括 Laravel Homestead 和 Rasmus PHP7dev。因此，让我们开始配置 Windows 和 Mac OS X 上的 Rasmus PHP7dev 框。

### 注意

我们假设我们的机器上都安装了 VirutalBox 和 Vagrant。可以从[`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads)下载 VirtualBox，可以从[`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html)下载 Vagrant，适用于不同的平台。有关 Rasmus PHP7dev VagrantBox 的详细信息，请访问[`github.com/rlerdorf/php7dev`](https://github.com/rlerdorf/php7dev)。

执行以下步骤：

1.  在其中一个驱动器中创建一个目录。例如，我们在`D`驱动器中创建了一个`php7`目录。然后，通过按住*Shift*键，右键单击，然后选择**在此处打开命令窗口**，直接在此特定文件夹中打开命令行。

1.  现在，在命令窗口中输入以下命令：

```php
    **vagrant box add rasmus/php7dev**

    ```

它将开始下载 Vagrant 框，如下截图所示：

![设置 Vagrant](img/B05225_01_04.jpg)

1.  现在，当下载完成时，我们需要初始化它，以便为我们配置并将该框添加到 VirtualBox 中。在命令窗口中输入以下命令：

```php
    **vagrant init rasmus/php7dev**

    ```

这将开始将框添加到 VirtualBox 并对其进行配置。完成该过程后，将显示一条消息，如下截图所示：

![设置 Vagrant](img/B05225_01_05.jpg)

1.  现在，输入以下命令，这将完全设置 Vagrant 框并启动它：

```php
    **vagrant up**

    ```

这个过程会花一点时间。当完成后，你的框已经准备好并且可以使用了。

1.  现在，启动后的第一件事是更新所有内容。这个框使用 Ubuntu，所以在相同的`php7dev`目录中打开命令窗口，并输入以下命令：

```php
    **vagrant ssh**

    ```

它将通过 SSH 将我们连接到虚拟机。

### 注意

在 Windows 中，如果 SSH 未安装或未在`PATH`变量中配置，可以使用 PuTTY。可以从[`www.chiark.greenend.org.uk/~sgtatham/putty/download.html`](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)下载。对于 PuTTY，主机将是`127.0.0.1`，端口将是`2222`。`Vagrant`是 SSH 的用户名和密码。

1.  当我们登录到框的操作系统时，输入以下命令来更新系统：

```php
    **sudo apt-get update**
    **sudo apt-get upgrade**

    ```

这将更新核心系统、NGINX、MySQL、PHP 7 和其他安装的工具，如果有新版本的话。

1.  现在，框已经准备好用于开发目的。可以通过在浏览器窗口中输入其 IP 地址来访问框。要找到框的 IP 地址，在 SSH 连接的命令窗口中输入以下命令：

```php
    **sudo ifconfig**

    ```

这将显示一些细节。在那里找到 IPv4 的细节并取得框的 IP。

# 总结

在本章中，我们为开发目的配置了不同的环境。我们在 Windows 机器上安装了 NGINX 和 PHP 7。我们还配置了 Debian/Ubuntu 并安装了 NGINX、PHP 和 Percona Server 5.5。然后，我们配置了 CentOS 并安装了 NGINX、PHP 和 Percona Server 5.5。最后，我们讨论了如何在 Windows 机器上配置 Vagrant Box。

在下一章中，我们将学习 PHP 7 的新功能，比如类型提示、命名空间分组和声明、太空船操作符等其他功能。
