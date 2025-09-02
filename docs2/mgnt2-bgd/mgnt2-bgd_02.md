# 第二章 管理环境

在本章中，我们将探讨设置我们的开发和生产环境。我们的想法是拥有一个完全自动化的开发环境，可以通过单个控制台命令启动。对于生产环境，我们将关注可用的云服务之一，看看设置 Magento 进行更简单的生产项目有多容易。我们不会涵盖任何强大的环境设置，如自动扩展、缓存服务器、内容分发网络等。这些实际上是系统管理员或 DevOps 角色的工作。我们在这里的关注点是启动我们的 Magento 商店所需的最基本条件；在接下来的章节中，我们将实现以下里程碑：

+   设置开发环境

    +   VirtualBox

    +   Vagrant

    +   Vagrant 项目

        +   配置 PHP

        +   配置 MySQL

        +   配置 Apache

        +   配置 Magento 安装

+   设置生产环境

    +   **Amazon Web Services**（**AWS**）简介

    +   设置 S3 使用权限

        +   创建 IAM 用户

        +   创建 IAM 组

    +   设置 S3 以备份数据库和媒体文件

    +   自动化 EC2 设置的 Bash 脚本

        +   设置 EC2

        +   设置弹性 IP 和 DNS

# 设置开发环境

在本节中，我们将使用**VirtualBox**和**Vagrant**构建一个开发环境。

### 注意

Magento 官方要求 Apache 2.2 或 2.4，PHP 5.6.x 或 5.5.x（PHP 5.4 不受支持），以及 MySQL 5.6.x。我们在设置环境时需要记住这一点。

## VirtualBox

**VirtualBox**是一款功能强大且丰富的 x86 和 AMD64/Intel64 虚拟化软件。它是免费的，可以在大量平台上运行，并支持大量客户操作系统。如果我们日常开发中使用 Windows、Linux 或 OS X，我们可以使用 VirtualBox 启动一个虚拟机，在其中安装运行 Magento 所需的服务器软件。这意味着使用 MySQL、Apache 以及一些其他东西。

## Vagrant

**Vagrant**是一个用于虚拟化软件管理的软件包装器。我们可以用它来创建和配置开发环境。Vagrant 支持多种类型的虚拟化软件，如 VirtualBox、**VMware**、基于内核的虚拟机（**KVM**）、**Linux 容器**（**LXC**）等。它甚至支持服务器环境，如 Amazon EC2。

### 注意

在开始之前，我们需要确保已经安装了 VirtualBox 和 Vagrant。我们可以从它们的官方网站下载并按照以下说明进行安装：[`www.virtualbox.org`](https://www.virtualbox.org) 和 [`www.vagrantup.com`](https://www.vagrantup.com)。

## Vagrant 项目

我们首先在主机操作系统中某个地方手动创建一个空目录，比如说 `/Users/branko/www/B05032-Magento-Box/`。这是我们将会拉取 Magento 代码的目录。我们希望 Magento 源代码位于 Vagrant 虚拟机外部，这样我们就可以在我们的 IDE 中轻松地与之一起工作。

然后我们创建一个 Vagrant 项目目录，比如说 `/Users/branko/www/magento-box/`。

在 `magento-box` 目录中，我们运行控制台命令 `vagrant init`。这会产生如下输出：

```php
A 'Vagrantfile' has been placed in this directory. You are now ready to 'vagrant up' your first virtual environment! Please read the comments in the Vagrantfile as well as documentation on 'vagrantup.com' for more information on using Vagrant.
```

`Vagrantfile` 实际上是一个 Ruby 语言源文件。如果我们去掉注释，其原始内容看起来如下：

```php
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "base"
end
```

如果我们现在在 `magento-box` 目录下运行 `vagrant up`，这将启动 VirtualBox 的无头（无 GUI）模式并运行基础操作系统。然而，现在让我们暂缓运行该命令。

目标是创建一个更健壮的 `Vagrantfile`，它涵盖了运行 Magento 所需的所有内容，从 Apache、MySQL、PHP、PHPUnit、composer 到带有性能基准数据的完整 Magento 安装。

虽然 Vagrant 本身没有单独的配置文件，但我们将创建一个，因为我们想在其中存储配置数据，如 MySQL 用户名和密码。

让我们创建一个名为 `Vagrantfile.config.yml` 的文件，与同一目录下的 `Vagrantfile` 一起，内容如下：

```php
ip: 192.168.10.10
s3:
  access_key: "AKIAIPRNHSWEQNWHLCDQ"
  secret_key: "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"
  bucket: "foggy-project-dhj6"
synced_folder:
  host_path: "/Users/branko/www/B05032-Magento-Box/"
  guest_path: "/vagrant-B05032-Magento-Box/"
mysql:
  host: "127.0.0.1"
  username: root
  password: user123
http_basic:
  repo_magento_com:
    username: a8adc3ac98245f519ua0d2v2c8770ec8
    password: a38488dc908c6d6923754c268vc41bc4
github_oauth:
  github_com: "d79fb920d4m4c2fb9d8798b6ce3a043f0b7c2af6"
magento:
  db_name: "magento"
  admin_firstname: "John"
  admin_lastname: "Doe"
  admin_password: "admin123"
  admin_user: "admin"
  admin_email: "email@change.me"
  backend_frontname: "admin"
  language: "en_US"
  currency: "USD"
  timezone: "Europe/London"
  base_url: "http://magento.box"
  fixture: "small"
```

这里没有 Vagrant 强制的结构。这可以是任何有效的 YAML 文件。所提供的值仅是我们可以放入其中的示例。

Magento 使我们能够生成一对 32 字符的认证令牌，可用于访问 Git 仓库。这是通过使用用户名和密码登录到 Magento Connect，然后转到**我的账户** | **开发者** | **安全密钥**来完成的。然后，公钥和私钥就成为了我们访问 Magento GitHub 仓库的用户名和密码。

有一个单独的配置文件意味着我们可以将 `Vagrantfile` 提交到版本控制中，同时将 `Vagrantfile.config.yml` 排除在版本控制之外。

我们现在通过替换其内容为以下内容来编辑 `Vagrantfile`：

```php
# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

vagrantConfig = YAML.load_file 'Vagrantfile.config.yml'

Vagrant.configure(2) do |config|

  config.vm.box = "ubuntu/vivid64"

  config.vm.network "private_network", ip: vagrantConfig['ip']

  # Mount local "~/www/B05032-Magento-Box/" path into box's "/vagrant-B05032-Magento-Box/" path
  config.vm.synced_folder vagrantConfig['synced_folder']['host_path'], vagrantConfig['synced_folder']['guest_path'], owner:"vagrant", group: "www-data", mount_options:["dmode=775, fmode=664"]

  # VirtualBox specific settings
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "2048"
    vb.cpus = 2
  end

  # <provisioner here>

end
```

上述代码首先包含了 `yaml` 库，然后读取 `Vagrantfile.config.yml` 文件的内容到 `vagrantConfig` 变量中。然后我们有一个 `config` 块，在其中我们定义了虚拟机类型、固定 IP 地址、主机和客户操作系统之间的共享文件夹，以及一些 VirtualBox 特定的细节，如分配的 CPU 和内存。

我们使用的是 `ubuntu/vivid64` 虚拟机，它代表 Ubuntu 15.04（Vivid Vervet）的服务器版本。原因是这个 Ubuntu 版本为我们提供了 MySQL 5.6.x 和 PHP 5.6.x，这些是安装 Magento 的要求之一。

我们进一步有一个配置条目为我们的虚拟机分配一个固定 IP。让我们现在在我们的主机操作系统的 *hosts* 文件中添加一个条目，如下所示：

```php
192.168.10.10 magento.box
```

### 注意

我们将固定 IP 地址分配给我们的虚拟机的原因是，我们可以在主机操作系统内直接打开类似 `http://magento.box` 的 URL，然后访问客户操作系统内 Apache 提供的页面。

上一段代码的另一个重要部分是我们定义 `synced_folder` 的地方。除了源和目标路径外，这里的关键部分是 `owner`、`group` 和 `mount_options`。我们将这些设置为 vagrant 用户、`www-data` 用户组，以及目录和文件权限的 `774` 和 `664`，以便与 Magento 顺利配合。

让我们继续编辑 `Vagrantfile`，向其中添加几个配置器，一个接一个。我们通过将前一个示例中的 `# <provisioner here>` 替换为以下内容来实现：

```php
config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
config.vm.provision "shell", inline: "sudo apt-get update"
```

在这里，我们指示 Vagrant 将主机的 `.gitconfig` 文件传递到客户操作系统。这样做是为了让客户操作系统的 Git 设置继承自主机操作系统的 Git。然后我们调用 `apt-get update` 以更新客户操作系统。

### 配置 PHP

在 `Vagrantfile` 中进一步添加，我们运行多个配置器以安装 PHP、所需的 PHP 模块和 PHPUnit，如下所示：

```php
config.vm.provision "shell", inline: "sudo apt-get -y install php5 php5-dev php5-curl php5-imagick php5-gd php5-mcrypt php5-mhash php5-mysql php5-xdebug php5-intl php5-xsl"
config.vm.provision "shell", inline: "sudo php5enmod mcrypt"
config.vm.provision "shell", inline: "echo \"xdebug.max_nesting_level=200\" >> /etc/php5/apache2/php.ini"
config.vm.provision "shell", inline: "sudo apt-get -y install phpunit"
```

### 注意

这里有一点值得指出——我们将 `xdebug.max_nesting_level=200` 写入 `php.ini` 文件的行。这样做是为了排除 Magento 不会启动并抛出 **Maximum Functions Nesting Level of '100' reached...** 错误的可能性。

### 配置 MySQL

在 `Vagrantfile` 中进一步添加，我们运行配置器以安装 MySQL 服务器，如下所示：

```php
config.vm.provision "shell", inline: "sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password #{vagrantConfig['mysql']['password']}'"
config.vm.provision "shell", inline: "sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password #{vagrantConfig['mysql']['password']}'"
config.vm.provision "shell", inline: "sudo apt-get -y install mysql-server"
config.vm.provision "shell", inline: "sudo service mysql start"
config.vm.provision "shell", inline: "sudo update-rc.d mysql defaults"
```

MySQL 安装有趣的地方在于，它要求在安装过程中提供密码和密码确认。这使得它成为配置过程中的一个麻烦部分，该过程期望 shell 命令简单地执行而不需要输入。为了绕过这个问题，我们使用 `debconf-set-selections` 来存储输入参数。我们从 `Vagrantfile.config.yml` 文件中读取密码，并将其传递给 `debconf-set-selections`。

安装完成后，`update-rc.d mysql` 默认设置将 MySQL 添加到操作系统启动过程中，从而确保在重启虚拟机时 MySQL 正在运行。

### 配置 Apache

在 `Vagrantfile` 中进一步添加，我们按照以下方式运行 Apache 配置器：

```php
config.vm.provision "shell", inline: "sudo apt-get -y install apache2"
config.vm.provision "shell", inline: "sudo update-rc.d apache2 defaults"
config.vm.provision "shell", inline: "sudo service apache2 start"
config.vm.provision "shell", inline: "sudo a2enmod rewrite"
config.vm.provision "shell", inline: "sudo awk '/<Directory \\/>/,/AllowOverride None/{sub(\"None\", \"All\",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf"
config.vm.provision "shell", inline: "sudo mv /tmp/tmp.apache2.conf /etc/apache2/apache2.conf"
config.vm.provision "shell", inline: "sudo awk '/<Directory \\/var\\/www\\/>/,/AllowOverride None/{sub(\"None\", \"All\",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf"
config.vm.provision "shell", inline: "sudo mv /tmp/tmp.apache2.conf /etc/apache2/apache2.conf"
config.vm.provision "shell", inline: "sudo service apache2 stop"
```

上一段代码安装了 Apache，将其添加到启动序列中，启动它，并打开重写模块。然后我们对 Apache 配置文件进行了更新，因为我们想将 `AllowOverride None` 替换为 `AllowOverride All`，否则我们的 Magento 将无法工作。一旦完成更改，我们停止 Apache 以避免后续进程。

### 配置 Magento 安装

在 `Vagrantfile` 中进一步添加，我们现在将注意力转向 Magento 安装，我们将它分为几个步骤。首先，我们使用 Vagrant 的同步文件夹功能将主机的 `/vagrant-B05032-Magento-Box/` 文件夹链接到客户，即 `/var/www/html`：

```php
config.vm.provision "shell", inline: "sudo rm -Rf /var/www/html"
config.vm.provision "shell", inline: "sudo ln -s #{vagrantConfig['synced_folder']['guest_path']} /var/www/html"
```

然后，我们使用 `composer create-project` 命令从官方 `repo.magento.com` 源将 Magento 2 文件拉取到 `/var/www/html/` 目录：

```php
config.vm.provision "shell", inline: "curl -sS https://getcomposer.org/installer | php"
config.vm.provision "shell", inline: "mv composer.phar /usr/local/bin/composer"
config.vm.provision "shell", inline: "composer clearcache"
config.vm.provision "shell", inline: "echo '{\"http-basic\": {\"repo.magento.com\": {\"username\": \"#{vagrantConfig ['http_basic']['repo_magento_com']['username']}\",\"password\": \"#{vagrantConfig['http_basic']['repo_magento_com']['password']} \"}}, \"github-oauth\": {\"github.com\": \"#{vagrantConfig['github_oauth']['github_com']}\"}}' >> /root/.composer/auth.json"
config.vm.provision "shell", inline: "composer create-project -- repository-url=https://repo.magento.com/ magento/project- community-edition /var/www/html/"
```

然后，我们创建一个数据库，稍后将在其中安装 Magento：

```php
config.vm.provision "shell", inline: "sudo mysql -- user=#{vagrantConfig['mysql']['username']} -- password=#{vagrantConfig['mysql']['password']} -e \"CREATE DATABASE #{vagrantConfig['magento']['db_name']};\""
```

我们随后从命令行运行 Magento 安装程序：

```php
config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento setup:install --base- url=\"#{vagrantConfig['magento']['base_url']}\" --db- host=\"#{vagrantConfig['mysql']['host']}\" --db- user=\"#{vagrantConfig['mysql']['username']}\" --db- password=\"#{vagrantConfig['mysql']['password']}\" --db- name=\"#{vagrantConfig['magento']['db_name']}\" --admin- firstname=\"#{vagrantConfig['magento']['admin_firstname']}\" -- admin-lastname=\"#{vagrantConfig['magento']['admin_lastname']}\" --admin-email=\"#{vagrantConfig['magento']['admin_email']}\" -- admin-user=\"#{vagrantConfig['magento']['admin_user']}\" -- admin-password=\"#{vagrantConfig['magento']['admin_password']}\" --backend- frontname=\"#{vagrantConfig['magento']['backend_frontname']}\" - -language=\"#{vagrantConfig['magento']['language']}\" -- currency=\"#{vagrantConfig['magento']['currency']}\" -- timezone=\"#{vagrantConfig['magento']['timezone']}\""
config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento deploy:mode:set developer"
config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento cache:disable"
config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento cache:flush"
config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento setup:performance:generate-fixtures /var/www/html/setup/performance-toolkit/profiles/ce/small.xml"
```

上述代码显示我们还在安装 fixtures 数据。

在配置 `Vagrantfile.config.yml` 文件时，我们需要小心。Magento 安装对提供的数据非常敏感。我们需要确保我们为像邮件和密码这样的字段提供有效的数据，否则安装将失败，并显示类似于以下错误的错误：

```php
SQLSTATE[28000] [1045] Access denied for user 'root'@'localhost' (using password: NO)
User Name is a required field.
First Name is a required field.
Last Name is a required field.
'magento.box' is not a valid hostname for email address 'john.doe@magento.box'
'magento.box' appears to be a DNS hostname but cannot match TLD against known list
'magento.box' appears to be a local network name but local network names are not allowed
Password is required field.
Your password must be at least 7 characters.
Your password must include both numeric and alphabetic characters.
```

有了这个，我们就完成了 `Vagrantfile` 的内容。

现在，在 `Vagrantfile` 所在的同一目录下运行 `vagrant up` 命令将触发盒子创建过程。在这个过程中，之前列出的所有命令都将被执行。这个过程本身可能需要一个小时左右。

一旦 vagrant up 完成，我们可以发出另一个控制台命令，`vagrant ssh`，以登录到盒子。

同时，如果我们在我们浏览器中打开类似 `http://magento.box` 的 URL，我们应该看到 Magento 商店首页正在加载。

上述 `Vagrantfile` 简单地从官方 Magento Git 仓库中提取并从头开始安装 Magento。`Vagrantfile` 和 `Vagrantfile.config.yml` 可以进一步扩展和定制，以满足我们各自项目的需求，例如从私有 Git 仓库中提取代码、从共享驱动器中恢复数据库等。

这为我们提供了一个简单而强大的脚本过程，通过这个过程我们可以为团队中的其他开发者准备完全就绪的项目机器，以便他们能够快速启动。

# 设置生产环境

生产环境是面向客户的、专注于良好性能和可用性的环境。设置生产环境并不是我们开发者倾向于做的事情，尤其是当有关于扩展、负载均衡、高可用性等方面的稳健要求时。然而，有时我们可能需要设置一个简单的生产环境。有许多云服务提供商提供快速简单的解决方案。为了本节的目的，我们将转向 Amazon Web Services。

## 亚马逊网络服务简介

**亚马逊网络服务**（**AWS**）是一组远程计算服务，通常被称为网络服务。AWS 提供云中的按需计算资源和服务，具有 *按使用付费* 的定价。亚马逊提供了一个很好的 AWS 资源比较，说使用 AWS 资源而不是自己的资源，就像从电力公司购买电力而不是运行自己的发电机一样。

如果我们停下来思考一下，这不仅仅对系统操作角色，对我们这样的开发者来说也很有趣。我们（开发者）现在能够在几分钟内启动各种数据库、Web 应用服务器，甚至复杂的基础设施。我们可以运行这些服务几分钟、几小时或几天，然后关闭它们。同时，我们只需为实际使用付费，而不是像大多数托管服务那样支付全额的月费或年费。尽管 AWS 某些服务的整体定价可能不是最便宜的，但它确实提供了许多其他服务所不具备的商品化和易用性。商品化来自于诸如自动扩展资源这样的特性，与等效的本地基础设施相比，它通常可以提供显著的成本节约。

质量培训和认证计划是 AWS 生态系统的重要方面之一。认证适用于**解决方案架构师**、**开发者**和**系统操作管理员**，涵盖副高级和专业级别。尽管认证不是强制性的，但如果我们经常处理 AWS，我们被鼓励去参加。获得认证将证明我们在 AWS 平台上设计、部署和运行高度可用、成本效益和安全的应用的专长。

我们可以通过一个简单直观的基于 Web 的用户界面，即 AWS 管理控制台来管理我们的 AWS，该界面可在[`aws.amazon.com/console`](https://aws.amazon.com/console)找到。登录 AWS 后，我们应该能看到以下类似的屏幕：

![亚马逊网络服务的介绍](img/00002.jpeg)

上一张图片显示了 AWS 管理控制台如何将 AWS 服务视觉上分组为几个主要组，如下所示：

+   **计算**

+   **开发者工具**

+   **移动服务**

+   **存储与内容分发**

+   **管理工具**

+   **应用服务**

+   **数据库**

+   **安全与身份**

+   **网络**

+   **分析**

+   **企业应用**

作为本章的一部分，我们将探讨**计算**组下的**EC2**服务和**存储与内容分发**组下的**S3**服务。

**亚马逊弹性计算云**（**Amazon EC2**）是一种提供可伸缩计算容量的云服务。我们可以将其视为云中的虚拟计算机，我们可以在任何时间打开和关闭它，只需几分钟。我们可以同时部署一台、数百台甚至数千台这样的机器实例。这使得计算容量具有**可伸缩性**。

S3 提供安全、持久且高度可扩展的对象存储。它旨在提供 99.99%的对象持久性。该服务提供了一个网络服务接口，可以从网络上的任何地方存储和检索任何数量的数据。S3 仅按实际使用的存储收费。S3 可以单独使用，也可以与其他 AWS 服务如 EC2 一起使用。

## 设置 S3 使用的访问权限

作为我们生产环境的一部分，拥有可靠的存储空间，我们可以存档数据库和媒体文件，这是很好的。Amazon S3 是一个可能的解决方案。

为了正确设置对 S3 可扩展存储服务的访问权限，我们需要快速了解一下 AWS 的**身份和访问管理**（简称**IAM**）。IAM 是一种网络服务，帮助我们安全地控制用户对 AWS 资源的访问。我们可以使用 IAM 来控制身份验证（谁可以使用我们的 AWS 资源）和授权（他们可以使用哪些资源以及如何使用）。更具体地说，正如我们很快将看到的，我们感兴趣的是**用户**和**组**。

### 创建 IAM 用户

本节描述了如何创建 IAM 用户。IAM 用户是我们创建在 AWS 中，用于代表使用它的人或服务在与 AWS 交互时的实体。

登录 AWS 控制台。

在用户菜单下，点击如下截图所示的**安全凭证**：

![创建 IAM 用户](img/00003.jpeg)

这将打开安全仪表板页面。

点击**用户**菜单应该打开一个类似于以下屏幕的界面：

![创建 IAM 用户](img/00004.jpeg)

在**用户**菜单下，我们点击**创建新用户**，这将打开一个类似于以下页面：

![创建 IAM 用户](img/00005.jpeg)

在这里，我们填写一个或多个用户的所需用户名，例如`foggy_s3_user1`，然后点击**创建**按钮。

我们现在应该看到一个类似于以下屏幕的界面：

![创建 IAM 用户](img/00006.jpeg)

在这里，我们可以点击**下载凭证**来启动 CSV 格式文件的下载，或者手动复制粘贴我们的凭证。

### 注意

**访问密钥 ID**和**秘密访问密钥**是我们将用于访问 S3 存储的两条信息。

点击关闭链接将我们带回到**用户**菜单，显示我们刚刚创建的用户，如下截图所示：

![创建 IAM 用户](img/00007.jpeg)

### 创建 IAM 组

本节描述了如何创建 IAM 组。组是我们可以将它们作为一个单一单元管理的 IAM 用户的集合。因此，让我们开始：

1.  登录 AWS 控制台。

1.  在用户菜单下，点击如下截图所示的**安全凭证**：![创建 IAM 组](img/00008.jpeg)

1.  这将打开安全仪表板页面。点击**组**菜单应该打开一个类似于以下屏幕的界面：![创建 IAM 组](img/00009.jpeg)

1.  在**组**菜单下，我们点击**创建新组**，这将打开一个类似于以下页面：![创建 IAM 组](img/00010.jpeg)

1.  在这里，我们填写所需的组名，例如`FoggyS3Test`。

1.  我们现在应该看到一个类似于以下屏幕的界面，我们需要选择**策略类型**组，然后点击**下一步**按钮：![创建 IAM 组](img/00011.jpeg)

1.  我们选择**AmazonS3FullAccess**策略类型，并点击**下一步**按钮。现在将显示**审查**屏幕，要求我们审查提供的信息：![创建 IAM 组](img/00012.jpeg)

1.  如果提供的信息正确，我们通过点击**创建组**按钮来确认。现在，我们应该能够在**组**菜单下看到我们的组，如下面的截图所示：![创建 IAM 组](img/00013.jpeg)

1.  打开**组名**左侧的复选框，点击**组操作**下拉菜单，然后选择如下截图所示的**添加用户到组**：

1.  这将打开如下截图所示的**添加用户到组**页面：

1.  打开**用户名**左侧的复选框，并点击**添加用户**按钮。这应该会将所选用户添加到组中，并返回到**组**列表。

用户和组创建过程的结果是一个具有**访问密钥 ID**、**秘密访问密钥**和分配有**AmazonS3FullAccess**策略的用户组。我们将在演示将数据库备份到 S3 时使用这些信息。

## 设置 S3 用于数据库和媒体文件备份

S3 由存储桶组成。我们可以将存储桶视为我们 S3 账户中的第一级目录。然后，我们在该目录（存储桶）上设置权限和其他选项。在本节中，我们将创建自己的存储桶，包含两个名为`database`和`media`的空文件夹。在后续的环境设置过程中，我们将使用这些文件夹来备份我们的 MySQL 数据库和媒体文件。

我们首先登录到 AWS 管理控制台。

在**存储与内容分发**组下，我们点击**S3**。这将打开一个类似于以下截图的屏幕：

![设置 S3 用于数据库和媒体文件备份](img/00016.jpeg)

点击**创建存储桶**按钮。这将打开一个类似于以下截图的弹出窗口：

![设置 S3 用于数据库和媒体文件备份](img/00017.jpeg)

让我们提供一个独特的**存储桶名称**，最好是一个可以识别我们将要备份的`database`和`media`文件的项目的名称，然后点击**创建**按钮。为了本章节的目的，让我们假设我们选择了类似`foggy-project-dhj6`的东西。

我们的存储桶现在应该在**所有存储桶**列表中可见。如果我们点击它，将打开一个类似于以下截图的新屏幕：

![设置 S3 用于数据库和媒体文件备份](img/00018.jpeg)

在这里，我们点击**创建文件夹**按钮，并添加必要的`database`和`media`文件夹。

在根存储桶目录内，点击**属性**按钮，并填写如下截图所示的**权限**部分：

![设置 S3 用于数据库和媒体文件备份](img/00019.jpeg)

在这里，我们基本上将所有权限分配给了**认证用户**。

现在我们应该有一个 S3 存储桶，我们可以使用`s3cmd`控制台工具（我们很快会提到）将数据库和媒体备份存储到其中。

## Bash 脚本用于自动化 EC2 设置

与`Vagrantfile` shell provisioners 类似，让我们继续创建一系列 bash shell 命令，我们可以使用这些命令进行生产环境设置。

第一组命令如下：

```php
#!/bin/bash
apt-get update
apt-get -y install s3cmd

```

在这里，从`#!/bin/bash`表达式开始。这指定了我们正在执行的脚本类型。然后我们有一个系统更新和`s3cmd`工具的安装。`s3cmd`是一个免费命令行工具和客户端，用于上传、检索和管理 Amazon S3 中的数据。我们可以稍后使用它进行数据库和媒体文件的备份和恢复。

然后，我们使用以下命令安装`postfix`邮件服务器。由于 postfix 安装会在控制台触发图形界面，要求输入`mailname`和`main_mailer_type`，我们使用`sudo debconf-set-selections`绕过这些步骤。安装完成后，我们重新加载`postfix`。

```php
sudo debconf-set-selections <<< "postfix postfix/mailname string magentize.me"
sudo debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
sudo apt-get install -y postfix
sudo /etc/init.d/postfix reload

```

在 EC2 盒子上直接使用邮件服务器对于小型生产网站来说是可以的，我们预计不会有高流量或大量客户。对于更密集的生产网站，我们需要注意 Amazon，可能需要在端口`25`上设置节流，从而导致发出的电子邮件超时。在这种情况下，我们可以要求 Amazon 取消我们账户的限制，或者转向更健壮的服务，如**Amazon Simple Email Service**。

接下来，我们安装所有与 PHP 相关的组件。注意我们甚至安装了`xdebug`，尽管立即将其关闭。这可能在那些非常罕见的我们需要真正调试实时网站的时刻派上用场，然后我们可以将其关闭并尝试远程调试。我们进一步下载并设置 composer 到用户路径：

```php
apt-get -y install php5 php5-dev php5-curl php5-imagick php5-gd php5- mcrypt php5-mhash php5-mysql php5-xdebug php5-intl php5-xsl
php5enmod mcrypt
php5dismod xdebug
service php5-fpm restart
apt-get -y install phpunit
echo "Starting Composer stuff" >> /var/tmp/box-progress.txt
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

```

然后，我们继续进行 MySQL 的安装。在这里，我们也使用`debconf-set-selections`来自动化控制台部分提供安装输入参数。安装完成后，MySQL 被启动并添加到启动过程中。

```php
debconf-set-selections <<< 'mysql-server mysql-server/root_password password RrkSBi6VDg6C'
debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password RrkSBi6VDg6C'
apt-get -y install mysql-server
service mysql start
update-rc.d mysql defaults

```

除了 MySQL 之外，另一个主要组件是 Apache。我们使用以下命令来安装它。在使用 Apache 时，我们需要注意其`apache2.conf`文件。我们需要将 Magento 目录的`AllowOverride None`更改为`AllowOverride All`：

```php
apt-get -y install apache2
update-rc.d apache2 defaults
service apache2 start
a2enmod rewrite
awk '/<Directory \/>/,/AllowOverride None/{sub("None", "All",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf
mv /tmp/tmp.apache2.conf /etc/apache2/apache2.conf
awk '/<Directory \/var\/www\/>/,/AllowOverride None/{sub("None", "All",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf
mv /tmp/tmp.apache2.conf /etc/apache2/apache2.conf
service apache2 restart

```

现在我们已经安装了 MySQL 和 Apache，我们继续将源代码文件放置到位。接下来，我们从官方的 Magento Git 仓库中拉取代码。这不同于我们在设置 vagrant 时使用的`repo.magento.com`。尽管在这种情况下，Magento Git 仓库是公开的，但我们的想法是能够从私有 GitHub 仓库中拉取代码。根据我们倾向于设置的生产环境，我们可以轻松地将下一部分替换为从任何其他私有 Git 仓库中拉取。

```php
sudo rm -Rf /var/www/html/*
git clone https://github.com/magento/magento2.git /var/www/html/.
sudo composer config --global github-oauth.github.com 7d6da6bld50dub454edc27db70db78b1f8997e6
sudo composer install --working-dir="/var/www/html/"
mysql -uroot -pRrkSBi6VDg6C -e "CREATE DATABASE magento;"
PUBLIC_HOSTNAME="'wget -q -O - http://instance-data/latest/meta- data/public-hostname'"

```

### 小贴士

要从私有 git 仓库拉取代码，我们可以使用以下形式的命令，Git 克隆：`https://<user>:<OAuthToken>@github.com/<user>/<repo>.git`。

`PUBLIC_HOSTNAME` 变量存储了调用 `http://instance-data/latest/meta-data/public-hostname` URL 的 `wget` 命令的响应。这个 URL 是 AWS 的一个功能，允许我们获取当前的 EC2 实例元数据。然后我们在 Magento 安装期间使用 `PUBLIC_HOSTNAME` 变量，将其作为 `--base-url` 参数传递：

```php
php /var/www/html/bin/magento setup:install --base- url="http://$PUBLIC_HOSTNAME" --db-host="127.0.0.1" --db- user="root" --db-password="RrkSBi6VDg6C" --db-name="magento" -- admin-firstname="John" --admin-lastname="Doe" --admin- email="john.doe@change.me" --admin-user="admin" --admin- password="pass123" --backend-frontname="admin" -- language="en_US" --currency="USD" --timezone="Europe/London"
```

前面的命令包含了许多 *项目特定* 的配置值，所以我们需要确保在简单地复制粘贴之前，适当地粘贴我们自己的信息。

现在我们确保 Magento 模式设置为生产，并且缓存已开启并刷新，以便重新生成：

```php
php /var/www/html/bin/magento deploy:mode:set production
php /var/www/html/bin/magento cache:enable
php /var/www/html/bin/magento cache:flush

```

最后，我们重置 `/var/www/html` 目录的权限，以确保我们的 Magento 能够正常运行：

```php
chown -R ubuntu:www-data /var/www/html
find /var/www/html -type f -print0 | xargs -r0 chmod 640
find /var/www/html -type d -print0 | xargs -r0 chmod 750
chmod -R g+w /var/www/html/pub
chmod -R g+w /var/www/html/var
chmod -R g+w /var/www/html/app
chmod -R g+w /var/www/html/vendor

```

我们需要对前面提到的 Git 和 Magento 安装示例保持谨慎。这里的想法是展示我们如何自动从公共或私有仓库设置 Git pull。对于这个特定案例，Magento 安装部分只是一个小小的额外奖励，并不是我们会在生产机器上实际做的事情。这个脚本的整个目的就是作为启动新 AMI 图像的蓝图。所以理想情况下，一旦代码被拉取，我们通常会从一些私有存储（如 S3）恢复数据库，并将其附加到我们的安装上。这样，一旦脚本完成，就可以完成文件、数据库和媒体的完整恢复。

把这个想法放在一边，让我们回到我们的脚本，进一步添加以下命令的每日数据库备份：

```php
CRON_CMD="mysql --user=root --password=RrkSBi6VDg6C magento | gzip -9 > ~/database.sql.gz" CRON_JOB="30 2 * * * $CRON_CMD"
( crontab -l | grep -v "$CRON_CMD" ; echo "$CRON_JOB" ) | crontab -

CRON_CMD="s3cmd  --access_key="AKIAINLIM7M6WGJKMMCQ" -- secret_key="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" put ~/database.sql.gz s3://foggy-project-ghj7/database/database_'date +"%Y-%m-%d_%H-%M"'.sql.gz"
CRON_JOB="30 3 * * * $CRON_CMD"
( crontab -l | grep -v "$CRON_CMD" ; echo "$CRON_JOB" ) | crontab -

```

在这里，我们添加了一个凌晨 2:30 的 **cron 作业**，用于将数据库备份到名为 `database.sql.gz` 的家目录文件中。然后我们添加了另一个凌晨 3:30 执行的 cron 作业，将数据库备份推送到 S3 存储。

与数据库备份类似，我们可以使用以下命令集将媒体备份指令添加到我们的脚本中：

```php
CRON_CMD="tar -cvvzf ~/media.tar.gz /var/www/html/pub/media/"
CRON_JOB="30 2 * * * $CRON_CMD"
( crontab -l | grep -v "$CRON_CMD" ; echo "$CRON_JOB" ) | crontab -

CRON_CMD="s3cmd --access_key="AKIAINLIM7M6WGJKMMCQ" -- secret_key="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" put ~/media.tar.gz s3://foggy-project-ghj7/media/media_'date +"%Y-%m- %d_%H-%M"'.tar.gz"
CRON_JOB="30 3 * * * $CRON_CMD"
( crontab -l | grep -v "$CRON_CMD" ; echo "$CRON_JOB" ) | crontab -

```

前面的命令中包含了几条编码信息。我们需要确保相应地粘贴我们的访问密钥、秘密密钥和 S3 桶名称。为了简化，我们在这里没有讨论将访问令牌硬编码到 cron 作业中的安全问题。亚马逊提供了一个广泛的 *AWS 安全最佳实践* 指南，可以通过官方 AWS 网站下载。

现在我们对自动化 EC2 设置的 bash 脚本可能的样子有了些了解，让我们继续设置 EC2 实例。

### 设置 EC2

按照以下步骤完成设置：

1.  登录 AWS 控制台

1.  在 **计算** 组下，点击 **EC2**，应该会打开一个类似于以下屏幕的界面：![设置 EC2](img/00020.jpeg)

1.  点击 **启动实例** 按钮，应该会打开一个类似于以下屏幕的界面：![设置 EC2](img/00021.jpeg)

1.  点击左侧的 **社区 AMI** 选项卡，并在搜索框中输入 `Ubuntu Vivid`，如图所示：![设置 EC2](img/00022.jpeg)

    ### 小贴士

    默认情况下，Ubuntu 15.x（Vivid Vervet）服务器支持 MySQL 5.6.x 和 PHP 5.6.x，这使得它成为安装 Magento 的良好候选者。

    我们现在应该看到一个类似于以下屏幕的界面：

    ![设置 EC2](img/00023.jpeg)

1.  选择一个实例类型，然后点击**下一步：配置实例详情**按钮。我们现在应该看到一个类似于以下屏幕的界面：![设置 EC2](img/00024.jpeg)

    ### 备注

    我们不会深入到每个选项的细节。简单来说，如果我们正在处理较小的生产站点，那么我们很可能可以将大多数这些选项保留为默认值。

1.  确保将**关机行为**设置为**停止**。

1.  在仍然位于**步骤 3：配置实例详情**屏幕的情况下，向下滚动到底部的**高级详情**区域并展开它。我们应该看到一个类似于以下屏幕的界面：![设置 EC2](img/00025.jpeg)

1.  **用户数据**输入是我们将复制并粘贴上一节中描述的`auto-setup bash`脚本的地方，如下截图所示：![设置 EC2](img/00026.jpeg)

1.  一旦我们复制并粘贴了**用户数据**，点击**下一步：添加存储**按钮。这应该会显示以下截图所示的屏幕：![设置 EC2](img/00027.jpeg)

1.  在**步骤 4：添加存储**中，我们可以选择一个或多个卷来附加到我们的 EC2 实例。最好选择 SSD 类型的存储以获得更快的性能。一旦设置了卷，点击**下一步：标记实例**。我们现在应该看到一个类似于以下屏幕的界面：![设置 EC2](img/00028.jpeg)

1.  **标记实例**屏幕允许我们分配标签。标签使我们能够根据目的、所有者、环境或其他方式对 AWS 资源进行分类。一旦我们分配了一个或多个标签，我们点击**下一步：配置安全组**按钮。我们现在应该看到一个类似于以下屏幕的界面：![设置 EC2](img/00029.jpeg)

1.  **配置安全组**界面允许我们设置入站和出站流量的规则。我们希望能够访问该设备上的 SSH、HTTP、HTTPS 和 SMTP 服务。一旦我们添加了所需的规则，点击**审查和启动**按钮。这将打开一个类似于以下屏幕的界面：![设置 EC2](img/00030.jpeg)

1.  **审查实例启动**屏幕是我们可以查看到目前为止配置的盒子的总结的地方。如果需要，我们可以返回并编辑单个设置。一旦我们对总结满意，我们点击**启动**按钮。这将打开一个类似于以下弹出窗口：![设置 EC2](img/00031.jpeg)

1.  在这里，我们可以选择一个现有的安全密钥或创建一个新的密钥。密钥以 PEM 格式提供。一旦我们选择了密钥，我们点击**启动实例**按钮。

    我们现在应该看到一个类似于以下屏幕的**启动状态**界面：

    ![设置 EC2](img/00032.jpeg)

1.  点击实例名称链接应该会带我们回到**EC2 仪表板**，如下截图所示：![设置 EC2](img/00033.jpeg)

关于前面的图像，我们现在应该能够通过以下任一控制台命令连接到我们的 EC2 服务器：

```php
ssh -i /path/to/magento-box.pem ubuntu@ec2-52-29-35-49.eu-central-1.compute.amazonaws.com

ssh -i /path/to/magento-box.pem ubuntu@52.29.35.49

```

我们的 EC2 服务器执行传递给它的所有 shell 命令可能需要一些时间。我们可以方便地 SSH 到服务器，然后执行以下命令来获取当前进度的概述：

```php
sudo tail -f /var/tmp/box-progress.txt

```

通过这样，我们完成了实例启动过程。

### 设置弹性 IP 和 DNS

现在我们已经有一个 EC2 服务器，让我们继续创建所谓的弹性 IP。**弹性 IP 地址**是为动态云计算设计的静态 IP 地址。它与 AWS 账户相关联，而不是某个特定的实例。这使得它很容易从一个实例重新映射到另一个实例。

让我们继续创建一个弹性 IP，如下所示：

1.  登录到 AWS 控制台。

1.  在**计算**组下，点击**EC2**，这将带我们到**EC2 仪表板**。

1.  在**EC2 仪表板**下，在“网络和安全”分组下的左侧区域，点击弹性 IP。这将打开一个类似于以下屏幕的界面：![设置弹性 IP 和 DNS](img/00034.jpeg)

1.  点击**分配新地址**按钮，这将打开一个类似于以下弹出窗口的界面：![设置弹性 IP 和 DNS](img/00035.jpeg)

1.  点击**是，分配**按钮，这将打开另一个类似于以下弹出窗口的界面：![设置弹性 IP 和 DNS](img/00036.jpeg)

1.  现在弹性 IP 地址已创建，在表格列表中右键单击它应该会弹出如下截图所示的下拉菜单：![设置弹性 IP 和 DNS](img/00037.jpeg)

1.  点击**关联地址**链接。这将打开一个类似于以下弹出窗口的界面：![设置弹性 IP 和 DNS](img/00038.jpeg)

1.  在**关联地址**弹出窗口中，我们选择要分配弹性 IP 地址的**实例**，然后点击**关联**按钮。

到目前为止，我们的 EC2 服务器已分配了一个静态（弹性 IP）地址。现在我们可以登录到我们的域名注册商，并将 DNS 的 A 记录指向我们刚刚创建的弹性 IP。

在我们等待 DNS 更改生效之前，还有一件事需要处理。我们需要 SSH 到我们的服务器并执行以下命令集：

```php
mysql -uroot -pRrkSBi6VDg6C -e "USE magento; UPDATE core_config_data SET value = 'http://our-domain.something/' WHERE path LIKE "%web/unsecure/base_url%";"
php /var/www/html/bin/magento cache:flush

```

这将更新 Magento 的 URL，因此一旦 DNS 更改生效，我们就可以通过 Web 浏览器访问它。通过一些前期规划，我们本可以轻松地将这部分内容作为我们 EC2 实例的用户数据的一部分，只需在最初提供正确的`--base-url`参数值即可。

# 摘要

在本章中，我们主要关注了两件事：设置开发和生产环境。

作为开发环境的一部分，我们采用了如 VirtualBox 和 Vagrant 等免费软件来管理我们的环境设置。仅设置本身就归结为一个单独的`Vagrantfile`脚本，该脚本包含了安装从 Ubuntu 服务器、PHP、Apache、MySQL 甚至包括 Magento 本身的必要命令集。我们绝不应该将此脚本视为最终的，而仅仅将其视为设置我们开发环境的有效脚本。在使开发环境更接近项目特定需求上投入时间，从团队生产力的角度来看是值得的。

然后，我们转向生产环境。在这里，我们研究了 Amazon Web Services，沿途使用了 S3 和 EC2。生产环境也附带了自己的脚本安装过程，该过程设置了大多数东西。同样，这个脚本也绝不是最终的，而仅仅是一种设置事物的有效方式；它更多的是一个如何操作的基例。

在下一章中，我们将更详细地探讨一些编程概念和约定。
