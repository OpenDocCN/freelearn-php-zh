# 第一章：使用 Drupal 8 运行起来

在本章中，我们将介绍 Drupal 8 并涵盖以下食谱：

+   安装 Drupal

+   使用带有 Drupal 的发行版

+   安装模块和主题

+   在 Drupal 8 中使用多站点

+   设置环境的工具

+   运行测试：Simpletest 和 PHPUnit

# 简介

本章将从介绍安装 Drupal 8 网站开始。我们将介绍 Drupal 的交互式安装程序。我们将介绍使用名为 Drush 的命令行工具安装 Drupal。Drupal 提供两种安装类型：标准型和最小型。在本书中，我们将使用标准安装。

一旦我们安装了我们的 Drupal 8 网站，我们将介绍扩展 Drupal 的基础知识。我们将讨论使用发行版和安装贡献项目，例如模块和主题。我们还将包括卸载模块，因为在 Drupal 8 中卸载模块的过程已经发生了变化。

本书将涉及一个与 Drupal 8 一起工作的动手示例，本章将提供有关设置本地开发环境的信息。本章还将提供如何在 Drupal 8 中设置多站点安装和运行可用测试套件的食谱。

在我们开始之前，您应该安装 Composer。Composer 是 PHP 的既定包管理工具。如果您不熟悉 Composer，它就像使用 Ruby 的 Gems、Node.js 的 npm 和前端库的 Bower 一样。请访问 Composer 文档了解如何在您的系统上全局安装 Composer：

+   Linux / Unix / macOS: [`getcomposer.org/doc/00-intro.md#installation-linux-unix-osx`](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx)

+   Windows: [`getcomposer.org/doc/00-intro.md#installation-windows`](https://getcomposer.org/doc/00-intro.md#installation-windows)

# 安装 Drupal

下载 Drupal 并安装有许多不同的方法。在本食谱中，我们将专注于从 [`www.drupal.org/`](https://www.drupal.org/) 下载 Drupal 并在基本的 Linux、Apache、MySQL 或 PHP（LAMP）服务器上设置它。

在本食谱中，我们将设置 Drupal 8 的文件并逐步介绍安装过程。

# 准备工作

在我们开始之前，您需要一个满足 Drupal 8 的新系统要求的开发生态系统：

+   Apache 2.0（或更高版本）或 Nginx 1.1（或更高版本）的 Web 服务器

+   需要 PHP 5.5.9 或更高版本，但推荐使用 PHP 5.6 或 PHP 7，因为 PHP 5.5 已经达到其生命周期的结束支持。

+   数据库需要 MySQL 5.5 或 MariaDB 5.5.20

您需要一个具有创建数据库权限的用户，或者一个已经创建的数据库，其中包含具有在该数据库中创建表权限的用户。

+   上传或移动文件到服务器的访问权限

+   虽然默认安装的 PHP 可以与 Drupal 一起工作，但它确实需要某些 PHP 扩展，例如 mbstring。请查看[`www.drupal.org/requirements/php`](https://www.drupal.org/requirements/php)以获取最新的要求信息。

Drupal 8 附带 Symfony ([`symfony.com/`](https://symfony.com/))组件。Drupal 8 中的新依赖之一，以支持 Symfony 路由系统，是 Drupal 的“清洁 URL”功能。如果服务器使用 Apache，请确保已启用`mod_rewrite`。如果服务器使用 Nginx，则必须启用`ngx_http_rewrite_module`。

我们将下载 Drupal 8 并将其文件放置在您的 Web 服务器的文档根目录中。这是`/var/www`文件夹。如果您使用了 XAMPP、WAMP 或 MAPP 等工具，请查阅适当的文档以了解您的文档根目录。

要查看 Drupal 8 的完整系统需求，请查看[`www.drupal.org/docs/8/system-requirements/`](https://www.drupal.org/docs/8/system-requirements/)。Drupal.org 文档目前正在迁移。此外，请查看[`www.drupal.org/docs/7/system-requirements/overview`](https://www.drupal.org/docs/7/system-requirements/overview)上的 Drupal 7 需求页面，该页面突出显示了 Drupal 8 的项目。

# 如何操作...

我们需要遵循以下步骤来安装 Drupal 8：

1.  首先，我们需要导航到[`www.drupal.org/download`](https://www.drupal.org/download)并下载 Drupal 8.x 的最新版本。您可以在 Drupal 8（8.3.1、8.4.0 等）的[`www.drupal.org/project/drupal`](https://www.drupal.org/project/drupal)页面上找到最新和推荐的版本。解压缩存档并将文件放置在您的文档根目录下的`drupal8`文件夹中：

![图片](img/1e2ed62e-6edf-4ee9-81ac-6656bdab7c2c.png)

1.  打开您的浏览器并访问您的 Web 服务器，例如`http://localhost/drupal8`，这将带您到 Drupal 安装向导。您将进入新的多语言选项安装屏幕。选择您的语言并点击保存并继续：

![图片](img/73e14053-ff4e-47c1-a917-7514a82ef907.png)

1.  在下一屏，选择安装配置的默认标准选项。这将为我们提供一个带有已安装最常用模块的标准配置。

1.  下一步将验证您的系统需求。如果您的系统没有任何可报告的问题，屏幕将被跳过。如果您确实有任何需求冲突，您可以解决它们并点击按钮重试。

如果您有需求问题，安装程序将报告具体问题。几乎每个需求都会链接到一个 Drupal.org 手册页面，其中包含解决方案步骤。

1.  输入 Drupal 的数据库信息。在大多数情况下，您只需要提供用户名、密码和数据库名称，并将其他设置为默认值。如果您的数据库不存在，安装程序将尝试创建数据库：

![图片](img/ccd32a82-ecf9-4a95-9fd0-9b86e3676bd8.png)

请参阅此菜谱的“更多...”部分，了解设置数据库的信息。

1.  您的 Drupal 8 站点将开始安装。当它安装完基本模块后，您将被带到站点配置屏幕。

1.  配置网站表单提供了你的 Drupal 网站的基本配置。输入你的网站名称和网站的电子邮件地址。网站电子邮件将用于发送管理通知，并作为从 Drupal 网站发出的电子邮件的原始电子邮件地址。此表单允许你设置有关网站的国家和时区的地区信息。设置时区确保时间值正确显示。

1.  填写网站维护账户信息，也称为用户 1，它在 Unix 系统中的 root 类似。网站维护账户至关重要。正如所述，它作为第一个用户存在，并赋予用户 ID 为 1。在 Drupal 中，具有用户 ID 为 1 的用户通常可以自动绕过权限检查并拥有全局访问权限。

1.  输入网站的地区信息，并检查网站是否应该检查已启用的模块和 Drupal 本身的可用更新。通过自动检查更新，你的网站将向 Drupal 报告匿名使用统计信息，并提供你的版本状态摘要。你也可以选择让网站通过电子邮件通知你新版本发布，包括安全更新。

1.  当信息满足条件时，点击保存并继续，恭喜你，你已经安装了 Drupal！下一屏将为你提供一个链接到你的已安装 Drupal 网站。

# 它是如何工作的...

Drupal 的安装过程将为所选语言提供 Drupal 安装，并根据安装配置文件（本食谱中的标准或最小）安装模块和配置。

当你访问安装程序时，它会从浏览器中读取语言代码。有了这个语言代码，它将选择一个支持的语言。如果你选择非英语安装，翻译文件将自动从 [`localize.drupal.org/`](https://localize.drupal.org/) 下载。Drupal 的早期版本不支持自动的多语言安装。关于多语言的内容将在*第八章*，*多语言和国际化*中进一步介绍。

安装配置文件指示 Drupal 安装默认的模块。贡献的安装配置文件被称为发行版；我们将在下一食谱中进一步讨论这一点。

在验证需求时，Drupal 会检查应用程序版本和 PHP 配置。例如，如果你的服务器已安装 PHP Xdebug ([`xdebug.org`](https://xdebug.org)) 扩展，则`max_nesting_level`的最小值必须是 256，否则 Drupal 将无法安装 ([`www.drupal.org/node/2393531`](https://www.drupal.org/node/2393531))。

# 还有更多...

Drupal 的安装过程很简单，但有一些值得讨论的地方。

# 创建数据库用户和数据库

如前所述，要安装 Drupal，您需要能够访问数据库服务器（或创建一个）以及现有的数据库（或创建一个）。此过程将取决于您的环境设置。

如果您与托管提供商合作，很可能有一个基于 Web 的控制面板。这应该允许您创建数据库和用户。有关此主题的更多信息，请参阅您的托管提供商的文档。

如果您在服务器上使用**phpMyAdmin** ([`www.phpmyadmin.net/`](https://www.phpmyadmin.net/))，通常由 MAMP、WAMP 和 XAMPP 安装，并且拥有 root 权限，您可以通过以下步骤创建数据库和用户：

1.  以 root 用户身份登录到 phpMyAdmin。

1.  从权限页面底部点击“添加新用户”。

1.  填写用户信息。

1.  选择为用户创建数据库并授予所有权限。

1.  您现在可以使用该用户信息将 Drupal 连接到您的数据库。

如果您没有用户界面但有命令行访问权限，您可以使用 MySQL 命令行设置数据库和用户。这些说明可以在`core/INSTALL.mysql.txt`文件中找到。从您站点的命令行执行以下操作：

1.  登录到 MySQL：

```php
$ mysql -u username -p
```

1.  创建数据库；您将使用以下命令创建`my_database`数据库：

```php
$ CREATE DATABASE my_database CHARACTER SET utf8 COLLATE utf8_general_ci;
```

1.  创建一个新用户以访问数据库：

```php
$ CREATE USER username@localhost IDENTIFIED BY 'password';
```

1.  授予新用户数据库权限，具体操作如下：

```php
$ GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES ON databasename.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

如果您使用 PostgreSQL 或 SQLite 数据库安装 Drupal，请查看相应的安装说明，即`INSTALL.pgsql.txt`或`INSTALL.sqlite.txt`。

# 数据库前缀

Drupal 与其他内容管理系统类似，允许您从数据库设置表单中为数据库表设置前缀。此前缀将放置在表名之前，以帮助使它们独特。虽然不推荐这样做，但这将允许多个安装共享一个数据库。利用表前缀可以在一定程度上通过隐藏提供安全性，因为表将不会使用它们的默认名称：

![](img/1441d718-6ef2-49c2-ab51-a26dfec36d65.png)

# 使用 Drush 下载和安装

您还可以使用 PHP 命令行工具 Drush 安装 Drupal。Drush 是由 Drupal 社区创建的命令行工具，如果您想使用它，则必须安装 Drush。Drush 在*第十三章*，*The Drupal CLI*中有介绍。

截至 Drush 9，它支持 Drupal 8.3+，本节已被弃用。使用 Drush 下载 Drupal 核心或贡献模块将抛出警告，建议使用 Composer。

`pm-download`命令将从 Drupal.org 下载包。`site-install`命令允许您指定安装配置文件和其他选项以安装 Drupal 站点。本食谱中的安装步骤可以通过 Drush 执行，如下所示：

```php
$ cd /path/to/web
$ drush pm-download drupal-8 drupal8
$ cd drupal8
$ drush site-install standard -locale=en-US --account-name=admin --account-pass=admin -account-email=demo@example.com -db-url=mysql://user:pass@localhost/database

```

我们使用 Drush 下载最新的 Drupal 8 并将其放置在名为 `drupal8` 的文件夹中。然后，`site-install` 命令指示 Drush 使用标准安装配置文件，配置维护账户，并提供数据库 URI 字符串，以便 Drupal 可以连接到其数据库。

# 使用 Composer 创建 Drupal 网站

您可以使用 Composer 下载 Drupal，这是事实上的 PHP 包管理器。社区提供的 Drupal Composer 项目模板是首选方法。

要构建您的 Drupal 8 网站，请运行以下命令：

```php
$ cd /path/to/document/root
$ composer create-project drupal-composer/drupal-project drupal8 --stability dev
```

等待命令完成--这可能需要一些时间，因为它会下载所有必需的依赖项。您可以随意拿一杯咖啡（第一次可能需要一段时间；它会预热缓存。请有信心，下次会快得多。）

完成后，您将在 `drupal8` 目录内找到一个不同的目录结构。`vendor` 目录包含所有第三方 PHP 库，而 `web` 目录包含您的 Drupal 8 网站。您需要修改您的 web 服务器，以便在 `drupal8` 目录中使用 `web` 目录作为新的 docroot。

项目及其详细信息可在 [`github.com/drupal-composer/drupal-project`](https://github.com/drupal-composer/drupal-project) 找到，以及其完整文档。

# 安全更新

如果您选择禁用更新选项，您将不得不手动检查模块升级。虽然大多数升级是为了修复错误或添加功能，但其中一些是为了安全更新。强烈建议您订阅 Drupal 安全团队的更新。这些更新可在 Twitter 上的 `@drupalsecurity` ([`twitter.com/drupalsecurity`](https://twitter.com/drupalsecurity)) 或相关订阅源上找到。

[`www.drupal.org/security`](https://www.drupal.org/security).

# 参见

+   想了解更多关于多语言的内容，请查看第 *8 章*，*多语言和国际化*

+   想了解更多关于使用命令行和 Drupal 的信息，请查看第 *13 章*，*Drupal CLI*

+   查看关于在 Drupal.org 上安装 Drupal 的手册，链接为 [`www.drupal.org/documentation/install`](https://www.drupal.org/documentation/install)

+   查看讨论 Drupal 8 和 Composer 的 Drupal.org 手册，链接为 [`www.drupal.org/docs/develop/using-composer/using-composer-with-drupal`](https://www.drupal.org/docs/develop/using-composer/using-composer-with-drupal)

+   查看有关 Drush 网站安装的更多信息，链接为 [`drushcommands.com/drush-8x/core/site-install/`](https://drushcommands.com/drush-8x/core/site-install/)

# 使用带有 Drupal 的发行版

为什么您想使用分发？分发是一个由 Drupal 核心不提供的贡献安装配置文件。分发提供具有特定已安装模块和主题以及特定配置（内容类型和块）的 Drupal 特殊版本。在 Drupal.org 上，当您下载安装配置文件时，它不仅包括配置文件及其模块，还包括 Drupal 核心的一个版本，因此得名分发。您可以在 [`www.drupal.org/project/project_distribution`](https://www.drupal.org/project/project_distribution) 找到所有 Drupal 分发的列表。

# 如何操作...

我们将遵循以下步骤下载分发，用作 Drupal 8 的定制版本：

1.  从 Drupal.org 下载分发。对于这个配方，让我们使用 Acquia 提供的 Demo Framework，网址为 [`www.drupal.org/project/df`](https://www.drupal.org/project/df)。

1.  选择 8.x 分支推荐版本。

1.  将文件夹内容提取到您的 Web 服务器文档根目录中--您会注意到其中包含 Drupal 核心文件；在 `profiles` 文件夹中，有安装配置文件的文件夹--`df`。

1.  由于当前 Drupal.org 打包限制，您需要运行一个手动步骤来安装额外的依赖项。在提取内容内的终端中运行以下命令：

```php
$ composer require "commerceguys/intl: ~0.7" "commerceguys/addressing: ~1.0" "commerceguys/zone: ~1.0" "embed/embed: ~2.2
```

1.  按照常规方式安装 Drupal，通过在浏览器中导航到您的 Drupal 网站。

1.  按照网站上的安装说明安装分发。

# 它是如何工作的...

安装配置文件通过包含贡献项目领域或自定义模块的额外模块来工作。然后配置文件将它们定义为与 Drupal 一起安装的依赖项。当您选择安装配置文件时，您是在指示 Drupal 在安装时安装一组模块。

Demo Framework 声明自己为独家安装配置文件。声明此配置文件的分发将自动选中并假定是默认安装选项。独家标志是在 Drupal 7.22 中添加的，以改善使用 Drupal 分发的体验 ([`drupal.org/node/1961012`](http://drupal.org/node/1961012))。

# 还有更多...

分发提供具有特定功能集的 Drupal 特殊版本，但有一些项目值得讨论。

# Makefiles

生成构建分发的当前标准是使用 Drush 和 makefiles。Makefiles 允许用户定义 Drupal 核心和其它项目（如主题、模块和第三方库）的特定版本，这些项目将构成 Drupal 代码库。它不是一个像 Composer 那样的依赖管理流程，而是一个构建工具。

如果您查看 Demo Framework 的 `profile` 文件夹，您将看到 `drupal-org.make` 和 `drupal-org-core.make`。这些文件被 Drupal.org 打包器解析，以编译代码库并将其打包成 `.zip` 或 `.tar.gz` 格式，就像您下载的那样。

# 使用 Drush 安装

如第一道菜谱中的 *还有更多...* 部分所述，您可以通过 Drush 命令行工具安装 Drupal 网站。您可以通过将其作为第一个参数提供来指示 Drush 使用特定的安装配置文件。

截至 Drush 9，它支持 Drupal 8.3+，本节已被弃用。使用 Drush 下载 Drupal 核心或贡献模块将抛出警告，建议使用 Composer。

以下命令将使用 Demo Framework 安装 Drupal 8 网站：

```php
 $ cd /path/to/drupal8
    $ drush pm-download df
    $ drush site-install df -db-url=mysql://user:pass@localhost/database
```

# 使用 Composer

目前，Drupal.org 不使用 Composer 打包发行版，这就是为什么在安装发行版时需要额外步骤来添加依赖项。许多发行版都提供了项目模板，以简化项目构建。

例如，以下命令将设置一个以 `docroot` 作为 Web 服务器文档根目录的 Demo Framework 网站，其中包含 Drupal 8：

```php
$ composer create-project acquia/df-project df
```

项目模板可在 Acqua 的 GitHub 上找到，链接为 [`github.com/acquia/df-project/`](https://github.com/acquia/df-project/)。

另一个发行版 Open Social 提供了自己的模板：

```php
$ composer create-project goalgorilla/social_template 
```

项目模板可在 [`github.com/goalgorilla/social_template`](https://github.com/goalgorilla/social_template) 上找到。

# 参见...

+   请参阅 *第十三章*，*Drupal CLI*，了解 makefile 的信息。

+   请参阅 Drupal 上的发行版文档 [`www.drupal.org/documentation/build/distributions`](https://www.drupal.org/documentation/build/distributions)。

+   请参阅 [`glamanate.com/blog/managing-your-drupal-project-composer`](https://glamanate.com/blog/managing-your-drupal-project-composer) 上的 *使用 Composer 管理您的 Drupal 项目*。

+   请参阅 [`glamanate.com/blog/managing-your-drupal-platform-drush`](https://glamanate.com/blog/managing-your-drupal-platform-drush) 上的 *使用 Drush 管理您的 Drupal 平台*。

# 安装模块和主题

Drupal 8 比之前的 Drupal 版本提供了更多的内置功能，让您可以以更少的努力做更多的事情。然而，Drupal 更吸引人的一个方面是扩展和定制的功能。

在本菜谱中，我们将下载并启用 Honeypot 模块 ([`www.drupal.org/project/honeypot`](https://www.drupal.org/project/honeypot))，并告诉 Drupal 使用 Bootstrap 主题 ([`www.drupal.org/project/bootstrap`](https://www.drupal.org/project/bootstrap))。Honeypot 模块为 Drupal 网站提供 Honeypot 和时间戳反垃圾邮件措施。此模块有助于保护表单免受垃圾邮件提交。Bootstrap 主题实现了 Bootstrap 前端框架，并支持使用 Bootswatch 风格来为主题 Drupal 网站。

本章的食谱将使用标准方式安装模块，即下载 Drupal.org 上可用的存档。截至 Drupal 8.2.0，通过 Composer 安装模块已成为可能，并且对于某些模块来说是必需的方法。使用 Composer 安装模块和主题的内容包含在本食谱的“更多内容...”部分，并强烈推荐。

# 准备工作

如果你之前使用过 Drupal，请注意文件夹结构已更改。模块、主题和配置文件现在放置在`root`目录下的相应文件夹中，而不是在`sites/all`下。有关开发者体验变更的更多信息，请参阅[`www.drupal.org/node/22336`](https://www.drupal.org/node/22336)。

下载示例代码：您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册自己，以便将文件直接通过电子邮件发送给您。

# 如何操作...

让我们安装模块和主题：

1.  访问[`www.drupal.org/project/honeypot`](https://www.drupal.org/project/honeypot)并下载 Honeypot 的最新 8.x 版本。

1.  解压存档并将`honeypot`文件夹放置在您的 Drupal 核心安装目录下的`modules`文件夹中：

![图片](img/94b95366-ac39-43c8-9d00-04ffe7f1beb5.png)

1.  在 Drupal 中登录并选择“扩展”选项以访问可用模块的列表。

1.  使用搜索文本框输入`Honeypot`。勾选复选框并点击“安装”。

1.  一旦启用，再次搜索它。点击模块的描述将展开行并显示配置权限和模块设置的链接：

![图片](img/652eb39e-937f-4346-bcd6-a0e75b4bc773.png)

1.  访问[`www.drupal.org/project/bootstrap`](https://www.drupal.org/project/bootstrap)并下载 Bootstrap 的最新 8.x 版本。

1.  解压存档并将`bootstrap`文件夹放置在您的 Drupal 核心安装目录下的`themes`文件夹中：

![图片](img/94680f06-9b25-4e69-9d8a-798505482308.png)

1.  在 Drupal 中，选择“外观”选项以管理您的 Drupal 主题。

1.  滚动到页面底部并点击“安装”和“Bootstrap”下的“设置为默认”以启用并设置主题为默认：

![图片](img/bfb2612f-3cad-47a2-851c-b2809a64d5d3.png)

# 它是如何工作的...

以下部分概述了安装模块或主题的流程以及 Drupal 如何发现这些扩展。

# 发现模块和主题

Drupal 扫描特定文件夹位置以识别由其目录中的`.info.yml`文件定义的模块和主题。以下为项目将被发现的顺序：

+   它们各自的核心文件夹（模块或主题）

+   当前安装的配置文件

+   根`modules`或`themes`文件夹

+   当前站点目录（默认或当前域名）

# 模块安装

通过将模块放置在根目录下的 `modules` 文件夹中，我们允许 Drupal 发现该模块并允许其安装。当模块安装时，Drupal 将通过 `module_installer` 服务将其代码注册到系统中。该服务将检查所需的依赖项，并在需要时提示启用它们。配置系统将在安装时运行模块提供的任何配置定义。如果有冲突的配置项，模块*将不会安装*。

# 主题安装

主题是通过 `theme_installer` 服务安装的，并设置主题提供的任何默认配置，同时重建主题注册表。将主题设置为默认是在 `system.theme.default` 配置中更改到主题的机器名（在菜谱中将是 `bootstrap`）。

# 更多内容...

以下部分概述了安装模块或主题的步骤，并包括一些有关安装的附加信息。

# 使用 Composer 安装模块或主题

虽然这不是安装扩展的必需方式，但这应该成为你的默认方法。为什么？因为每个模块都是你项目中的一个依赖项，而且每个模块都可能有自己的依赖项。Composer 可以为你管理依赖项，或者你可以手动管理它们。你的时间和能力可能不会像 Composer 那样增长到规模。更不用说，它还提供了一个标准方式，让 PHP 项目能够相互操作和加载类。

你可以使用以下两个命令获取 Honeypot 模块和 Bootstrap：

```php
$ cd /path/to/drupal8
$ composer require drupal/honeypot
$ composer require drupal/bootstrap

```

这里是一个贡献项目的示例，这些项目需要 Composer 进行安装，因为它们利用了整个 PHP 社区中的现有库：

+   Drupal Commerce

+   GeoIP

+   搜索 API Solr

+   实体打印

随着越来越多的模块集成了现有的 SDK 库，使用 Composer 的需求将会增加。

# 使用 Drush 安装模块

可以通过命令行使用 `drush` 下载和启用模块。复制菜谱的命令可能类似于以下内容：

```php
    $ drush pm-download honeypot
    $ drush pm-enable honeypot
```

截至 Drush 9，它支持 Drupal 8.3+，本节已弃用。使用 Drush 下载 Drupal 核心或贡献模块将抛出警告，建议使用 Composer。

它将提示你确认你的操作。如果模块有依赖项，它将询问你是否想启用这些依赖项。

Drush 仅下载来自 Drupal.org 的存档。如果模块或主题需要第三方 PHP 库依赖项，这些依赖项将不会被下载或在 Drupal 的类自动加载过程中可用。

# 卸载模块

Drupal 8 中的一个重大变化是模块禁用和卸载过程。以前，模块首先被禁用，然后禁用后卸载。这造成了一个令人困惑的过程，它会禁用其功能，但不会清理任何数据库模式更改。在 Drupal 8 中，模块不能只是被禁用，而必须被卸载。这确保了当模块被卸载时，它可以安全地从代码库中删除。

如果一个模块不是另一个模块的依赖项，或者没有正在使用的配置项（例如字段类型），则可以卸载该模块。这可能会破坏安装的完整性。

使用标准安装，在删除文章内容类型上的所有`Comment`字段之前，无法卸载`Comment`模块。这是因为`field`类型正在使用中。

# 参考信息

+   参考第*第四章*，*扩展 Drupal*，了解在启用模块时设置默认值。

+   参考第*第九章*，*配置管理 - 在 Drupal 8 中部署*。

# 在 Drupal 8 中使用多站点

Drupal 提供了从单个 Drupal 代码库实例运行多个站点的功能。这个功能被称为多站点。每个站点都有一个独立的数据库；然而，存储在`modules`、`profiles`和`themes`中的扩展可以被所有站点安装。站点文件夹也可以包含它们自己的模块和主题。当提供时，这些只能由该站点使用。

`default`文件夹是在没有匹配域名时使用的默认文件夹。

# 准备工作

如果您将要使用多站点功能，您应该了解如何设置您的 Web 服务器上的虚拟主机配置。在这个菜谱中，我们将使用 localhost 下的两个子域名，称为`dev1`和`dev2`。

# 如何操作...

我们将通过 localhost 下的两个子域名来使用 Drupal 8 的多站点：

1.  将`sites/example.sites.php`复制到`sites/sites.php`。

1.  在`sites`文件夹内创建一个`dev1.localhost`和`dev2.localhost`文件夹。

1.  将`sites/default/default.settings.php`文件复制到`dev1.localhost`和`dev2.localhost`，并在各自的文件夹中作为`settings.php`使用：

![图片](img/e4f64b42-01bd-49e0-9ba4-588514f69700.png)

1.  前往`dev1.localhost`并运行安装向导。

1.  前往`dev2.localhost`并验证您是否仍然有安装站点的选项！

# 它是如何工作的...

`sites.php`必须存在，多站点功能才能工作。默认情况下，您不需要修改其内容。`sites.php`文件提供了一种将别名映射到特定站点文件夹的方法。该文件包含了使用别名的文档。

`DrupalKernel` 类提供了 `findSitePath` 和 `getSitePath` 方法来发现站点文件夹路径。在 Drupal 的引导过程中，这被启动并读取传入的 HTTP 主机以从适当的文件夹加载正确的 `settings.php` 文件。然后加载并解析 `settings.php` 文件到 `\Drupal\Core\Site\Settings` 实例。这允许 Drupal 连接到适当的数据库。

# 还有更多...

让我们了解使用多站点的安全问题。

# 安全问题

如果你使用多站点，可能会有一些担忧。在 Drupal 站点上执行的任意 PHP 代码可能会影响共享相同代码库的其他站点。Drupal 8 标记了移除 PHP 过滤器 ([`www.drupal.org/docs/8/modules/php/overview`](https://www.drupal.org/docs/8/modules/php/overview)) 模块，该模块允许站点管理员在管理界面中使用 PHP 代码。尽管这减少了管理员通过界面轻松运行 PHP 的各种方式，但它并没有完全减轻风险。例如，PHP 过滤器模块现在是一个贡献项目，可以被安装。

# 域名别名

`sites.php` 文件提供了一种添加域名别名的方法。当你使用多站点功能并在本地开发时，这可能很有用。一个简单的例子是为每个站点提供一个 `local.alias`。

如果你将 `example.com` 和 `mycompany.com` 作为不同的站点目录，以下映射将允许 `local.example.com` 和 `local.mycompany.com` 映射到这些目录：

```php
<?php
$sites['example.com'] = 'example.com';
$sites['local.example.com'] = 'example.com';
$sites['mycompany.com'] = 'mycompany.com';
$sites['local.mycompany.com'] = 'mycompany.com';

```

# 参见...

+   参考 Drupal 的多站点文档，请访问 [`www.drupal.org/documentation/install/multi-site`](https://www.drupal.org/documentation/install/multi-site)。

# 设置环境的工具

开始使用 Drupal 的一个初步障碍是本地开发环境。本食谱将介绍如何通过 Jeff Geerling 的 DrupalVM 项目设置。DrupalVM 是通过 Vagrant 运行的 VirtualBox 虚拟机，通过 Ansible 配置和部署。它将为你设置所有服务并构建一个 Drupal 安装。

幸运的是，你只需要在你的机器上安装 VirtualBox 和 Vagrant，DrupalVM 支持 Windows、macOS X 和 Linux。

# 准备工作

要开始，你需要安装 DrupalVM 所需的两个依赖项：

+   **VirtualBox**: [`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads)

+   **Vagrant**: [`www.vagrantup.com/downloads.html`](http://www.vagrantup.com/downloads.html)

# 如何操作...

让我们按照以下步骤设置 Jeff Geerling 的 DrupalVM 项目：

1.  从 [`github.com/geerlingguy/drupal-vm/archive/master.zip`](https://github.com/geerlingguy/drupal-vm/archive/master.zip) 下载 DrupalVM 存档。

1.  解压存档并将项目放置在你选择的目录中。

1.  将 `example.drupal.make.yml` 复制到 `drupal.make.yml`。

1.  将 `default.config.yml` 复制到 `config.yml`。

1.  编辑`config.yml`并修改`local_path`设置，使其成为您放置 DrupalVM 项目的目录。这将同步到虚拟机中：

```php
vagrant_synced_folders: 
local_path: /path/to/drupalvm 
destination: /var/www 
type: nfs 
create: true 
```

1.  打开终端并导航到您放置文件的目录。

    DrupalVM 项目。

1.  输入`vagrant up`命令来告诉 Vagrant 构建虚拟机并开始配置过程。

1.  在此过程进行时，修改您的 hosts 文件以提供对开发站点的便捷访问。将`192.168.88.88 drupalvm.dev`行添加到您的 hosts 文件中。

1.  打开您的浏览器并访问[`www.drupalvm.com/`](http://www.drupalvm.com/)。

1.  使用用户名`admin`和密码`admin`登录到您的 Drupal 站点。

# 工作原理...

DrupalVM 是一个开发项目，它利用 Vagrant 工具创建一个 VirtualBox 虚拟机。Vagrant 通过项目的`Vagrantfile`进行配置。然后，Vagrant 使用 Ansible（一个开源的 IT 自动化平台）在虚拟机上安装 Apache、PHP、MySQL 和其他服务。

`config.yml`文件已经设置好，提供了一种简单的方式来自定义虚拟机和配置过程中的变量。它还使用 Drush 创建和安装 Drupal 8 站点，或者`drupal.make.yml`中指定的任何组件。此文件是一个 Drush `make`文件，默认包含 Drupal 核心的定义，并且可以修改以包含其他贡献项目。

`vagrant up`命令告诉 Vagrant 要么启动一个现有的虚拟机，要么以无头方式创建一个新的虚拟机。当 Vagrant 创建一个新的虚拟机时，它会触发配置过程。在这种情况下，Ansible 将读取`provisioning/playbook.yml`文件，并遵循每个步骤来创建最终的虚拟机。然而，需要修改的文件只有`config.yml`和`drupal.make.yml`文件。

# 更多内容...

自动化和简化本地环境的主题目前非常流行，有相当多的不同选项。如果你不习惯使用 Vagrant，还有一些其他选项可以提供服务器安装和 Drupal。

# Acquia Dev Desktop

Acquia Dev Desktop 由 Acquia 开发，可在[`docs.acquia.com/dev-desktop2`](https://docs.acquia.com/dev-desktop2)找到。它是一个 Windows 和 Mac 的自动化环境安装程序。它是一个 xAMP 堆栈（或 DAMP 堆栈）安装程序，提供包含 Apache、MySQL 和 PHP 的完整 Drupal 特定堆栈。Dev Desktop 应用程序允许您创建一个常规的 Drupal 安装或从分发版中选择。

# XAMPP + Bitnami

XAMPP - Apache + MySQL + PHP + Perl - 是一个跨平台环境安装。XAMPP 是 Apache Friends 的开源项目。XAMPP 与 Bitnami ([`bitnami.com/`](https://bitnami.com/)) 合作，为常见应用程序提供免费的集成安装，包括 Drupal 8。您可以在 [`www.apachefriends.org/download.html`](https://www.apachefriends.org/download.html) 下载 XAMPP。

# Kalabox

Kalabox 由 Kalamuna 团队开发，旨在成为 Drupal 开发的强大工作流程解决方案。Kalabox 兼容跨平台，允许您轻松地在 Windows 机器上工作。它基于命令行，并提供应用程序二进制文件供您安装。您可以在 [`www.kalamuna.com/products/kalabox/`](http://www.kalamuna.com/products/kalabox/) 上了解更多关于 Kalabox 的信息。

# 参见

+   请参阅 *第十三章*，*Drupal CLI*，了解 makefile 的信息。

+   DrupalVM 文档 [`docs.drupalvm.com/en/latest/`](http://docs.drupalvm.com/en/latest/)。

+   请参考 Drupal.org 社区文档中的本地环境设置信息，链接为 [`www.drupal.org/node/157602`](https://www.drupal.org/node/157602)。

# 运行测试 - Simpletest 和 PHPUnit

Drupal 8 随带两个测试套件。之前，Drupal 只支持 Simpletest。现在，还有 PHPUnit 测试。在官方变更记录中，PHPUnit 被添加以提供无需完整 Drupal Bootstrap 的测试，这在每次 Simpletest 测试时都会发生。您可以在 [`www.drupal.org/node/2012184`](https://www.drupal.org/node/2012184) 阅读变更记录。

目前在 Drupal 核心开发中有一个活跃的 PHPUnit 创新项目。目标是到 Drupal 9 时完全移除 Simpletest 框架。至少从 8.2 版本开始，不再编写新的 Simpletest 测试。所有当前测试正在由贡献者转换。更多关于这个项目的信息可以在这个问题中找到，[`www.drupal.org/node/2807237`](https://www.drupal.org/node/2807237)，那里正在进行协调。

我们将使用 `run-tests.sh` 测试运行器来运行测试。这是一个由 Drupal 提供的测试运行器，支持并发运行所有各种测试套件。在接下来的 *还有更多...* 部分将介绍如何直接使用 PHPUnit 运行测试。

# 准备工作

Drupal 8.1.0 引入了执行 JavaScript 浏览器测试的能力。这是使用 PhantomJS ([`phantomjs.org/`](http://phantomjs.org/)) 实现的，它使用由 Mink PHP 库 ([`mink.behat.org/`](http://mink.behat.org/)) 提供的浏览器模拟器。为了运行 FunctionalJavascript 测试套件，您必须运行 PhantomJS。

要安装 PhantomJS，请参考官方安装说明，链接为 [`phantomjs.org/download.html`](http://phantomjs.org/download.html)。

# 如何操作...

1.  首先，安装`Simpletest`模块。即使你可能只想运行 PHPUnit，这也是运行测试运行器脚本的软依赖。

1.  打开命令行终端，导航到您的 Drupal 安装目录。

1.  接下来，我们将运行测试运行器脚本。我们将传递一个`url`选项，以便功能测试可以正确运行浏览器模拟器。我们还将指定要运行的测试套件。这允许我们跳过功能 JavaScript 测试，因为 PhantomJS 在测试运行器中无法正确处理并发：

```php
$ php core/scripts/run-tests.sh **--url** http://localhost--types Simpletest,PHPUnit-Unit,PHPUnit-Kernel,PHPUnit-Functional **--concurrency 20 --all**
```

1.  运行`FunctionalJavascripts`测试需要 PhantomJS 正在运行。由于 PhantomJS 将输出打印到终端，请打开一个新标签或终端并运行以下命令：

```php
phantomjs --ssl-protocol=any --ignore-ssl-errors=true vendor/jcalderonzumba/gastonjs/src/Client/main.js 8510 1024 768 
```

1.  当 PhantomJS 运行时，我们现在可以执行功能 JavaScript 测试套件：

```php
**php core/scripts/run-tests.sh --url** http://localhost--types PHPUnit-FunctionalJavascript --concurrency 1 --all
```

1.  查看每个测试套件运行的测试输出。

# 它是如何工作的...

`run-tests.sh`脚本自 2008 年以来一直与 Drupal 一起提供，当时命名为

`run-functional-tests.php`。此命令与 Drupal 中的测试套件交互，运行所有或特定测试，并设置其他配置项。

有几个不同的测试套件，它们以特定的方式运行：

+   **Simpletest**：已弃用的测试系统，完全引导并安装 Drupal，并使用 curl 和 XPath 的自己的浏览器模拟模式。

+   **PHPUnit-Unit**：由 PHPUnit 支持的单元测试。这些测试旨在测试特定的类，而不与数据库交互。

+   **PHPUnit-Kernel**：由 PHPUnit 支持的集成级别测试。这是一个能够将模式配置安装到数据库中的测试，它以最小化引导 Drupal 进行基本集成测试。

+   **PHPUnit-Functional**：功能测试是需要在完全引导的 Drupal 中运行的测试，并通过 Mink 提供浏览器模拟。这些可以被认为是 Simpletest 测试的直接替代品，但利用第三方测试库。

+   **PHPUnit-FunctionalJavascript**：具有与 PhantomJS 交互的能力的功能测试，以测试 JavaScript，例如 AJAX 操作和特定的用户界面交互。

以下是一些有用的选项：

+   `--help`：此选项显示以下要点涵盖的内容

+   `--list`：此选项显示可以运行的可用测试组

+   `--url`：除非 Drupal 站点可通过`http://localhost:80`访问，否则此选项是必需的

+   `--sqlite`：这允许您在没有安装 Drupal 的情况下运行测试

+   `--concurrency`：这允许你定义并行运行的测试数量

# 还有更多...

我们现在将讨论更多关于运行 Drupal 测试套件的技巧和信息。

# run-tests 是一个 shell 脚本吗？

`run-tests.sh`实际上不是一个 shell 脚本。它是一个 PHP 脚本--这就是为什么你必须用 PHP 来执行它。实际上，在`core/scripts`中，每个文件都是一个 PHP 脚本文件，旨在通过命令行执行。这些脚本不是旨在通过 Web 服务器运行，这是`.sh`扩展名的原因之一。

在跨平台之间可能会有问题，这阻止了提供 shebang 行以允许将文件作为正常的 `bash` 或 `bat` 脚本执行。有关更多信息，请参阅此 Drupal.org 问题 [`www.drupal.org/node/655178`](https://www.drupal.org/node/655178)。

# 在未安装 Drupal 的情况下运行测试

在 Drupal 8 中，测试也可以从 SQLite 运行，不再需要安装数据库。这可以通过将 `sqlite` 和 `dburl` 选项传递给

`run-tests.sh` 脚本。这需要安装 PHP SQLite 扩展。

这里是一个从 DrupalCI 测试运行器改编的示例，用于 Drupal 核心测试。DrupalCI 是持续集成服务，它在 Drupal.org 上为所有提交的补丁和提交运行：

```php
php core/scripts/run-tests.sh --sqlite /tmp/.ht.sqlite --die-on-fail --dburl sqlite://tmp/.ht.sqlite --all

```

结合内置的 PHP 网络服务器进行调试，您可以在没有完整环境的情况下运行测试套件。

# 运行特定测试

到目前为止的每个示例都使用了 `all` 选项来运行所有可用的 `Simpletest`。有各种方法来运行特定的测试：

+   `--module`: 这允许您运行特定模块的所有测试

+   `--class`: 运行一个由完整命名空间路径标识的特定路径

+   `--file`: 从指定的文件运行测试

+   `--directory`: 在指定的目录内运行测试

在 Drupal 中，测试以前是分组在 `module.test` 文件中，这也是文件选项的来源。Drupal 8 使用 `PSR-4 自动加载` 方法，并要求每个文件有一个类。

# PhpStorm - Drupal 测试运行器

Drupal 8 在 Drupal 核心和贡献项目中的测试覆盖率都有所增加，这很可能是由于 PHPUnit 的采用。作为回应，作者编写了一个名为 Drupal Test Runner 的 PhpStorm 插件，该插件简化了执行 `run-tests.sh` 脚本运行器的操作。

插件的页面可以在 [`plugins.jetbrains.com/plugin/8384-drupal-test-runner`](https://plugins.jetbrains.com/plugin/8384-drupal-test-runner) 找到，其公开源代码可以在 [`github.com/mglaman/intellij-drupal-run-tests`](https://github.com/mglaman/intellij-drupal-run-tests) 找到。

# DrupalCI

随着 Drupal 8 的推出，一个新的举措是为了升级 Drupal.org 上的测试基础设施。结果是 DrupalCI。DrupalCI 是开源的，可以下载并在本地运行。DrupalCI 的项目页面是 [`www.drupal.org/project/drupalci`](https://www.drupal.org/project/drupalci)。

测试机器人使用 Docker，可以本地下载以运行测试。项目附带一个 Vagrant 文件，允许它在虚拟机或本地运行。有关测试机器人的更多信息，请访问其项目页面 [`www.drupal.org/project/drupalci_testbot`](https://www.drupal.org/project/drupalci_testbot)。

# 参见...

+   参考 PHPUnit 手册 [`phpunit.de/manual/4.8/en/writing-tests-for-phpunit.html`](https://phpunit.de/manual/4.8/en/writing-tests-for-phpunit.html)。

+   参考 Drupal PHPUnit 手册 [`drupal.org/phpunit`](https://drupal.org/phpunit)。

+   请参考[从命令行运行测试](https://www.drupal.org/node/645286)。

+   请参考《Commerce 2.x：单元、内核和功能测试，哦，我的天！》作者博客文章和教程，了解如何在 Drupal 8 中为 Drupal Commerce 运行测试，链接为[`drupalcommerce.org/blog/45322/commerce-2x-unit-kernel-and-functional-tests-oh-my`](https://drupalcommerce.org/blog/45322/commerce-2x-unit-kernel-and-functional-tests-oh-my)。
