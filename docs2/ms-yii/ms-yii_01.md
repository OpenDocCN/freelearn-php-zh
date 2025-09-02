# 第一章。Composer、配置、类和路径别名

在深入了解 Yii Framework 2 之前，我们需要看看它是如何安装的，如何配置的，以及框架的核心构建块是什么。在本章中，我们将介绍如何通过名为 **Composer** 的包管理工具安装框架本身和预构建的应用程序。我们还将涵盖 Yii Framework 2 和我们的 Web 服务器的一些常见配置，包括使我们的应用程序了解它们正在运行的环境，并相应地对此环境做出反应。

### 注意

引用 Yii Framework 2 最常见的方式是 *Yii Framework 2*、*YF2* 和 *Yii2*。本书中我们将交替使用这些术语。

# Composer

安装 Yii2 有几种不同的方式，从从源代码控制（通常是从 GitHub 的 [`github.com/yiisoft/yii2`](https://github.com/yiisoft/yii2)）下载框架到使用包管理器（如 Composer）。在现代 Web 应用程序中，Composer 是安装 Yii2 的首选方法，因为它使我们能够以自动化的方式安装、更新和管理我们应用程序的所有依赖项和扩展。此外，使用 Composer，我们可以确保 Yii Framework 2 保持最新状态，包括最新的安全和错误修复。Composer 可以通过遵循 [`getcomposer.org`](https://getcomposer.org) 上的说明进行安装。通常，这个过程如下所示：

```php
curl -sS https://getcomposer.org/installer | php

```

或者，如果你系统上没有 cURL，可以通过 PHP 本身进行安装：

```php
php -r "readfile('https://getcomposer.org/installer');" | php

```

安装完成后，我们应该将 Composer 移动到一个更集中的目录，这样我们就可以从系统上的任何目录调用它。与基于每个项目的安装相比，从集中式目录安装 Composer 有几个优点：

+   它可以从任何项目中进行调用。当处理多个项目时，我们可以确保每次和每个项目都使用相同的依赖管理器。

+   在集中式目录中，Composer 只需要更新一次，而不是在我们正在工作的每个项目中都更新。

+   依赖管理器很少被认为是应该推送到你的 DCVS 仓库的代码。将 `composer.phar` 文件保留在仓库之外可以减少你需要提交和推送的代码量，并确保你的源代码与包管理器代码保持隔离。

+   通过从集中式目录安装 Composer，我们可以确保 Composer 总是可用的，这样每次克隆依赖于 Composer 的项目时，我们都可以节省一个步骤。

将 Composer 移动到 `/usr/local/bin` 是一个不错的选择，如下面的示例所示：

```php
mv composer.phar /usr/local/bin/composer
chmod a+x /usr/local/bin/composer

```

### 小贴士

在本书中，当我们引用命令行参数时，将使用 Unix 风格的命令。因此，一些命令可能在 Windows 上无法工作。如果你决定设置 Windows 环境，你可能需要使用`Composer-Setup.exe`（可在[`getcomposer.org/Composer-Setup.exe`](https://getcomposer.org/Composer-Setup.exe)找到）来为你的系统配置 Composer。如果你在系统上运行 Composer 时遇到任何问题，请确保查看在[`getcomposer.org/doc/`](https://getcomposer.org/doc/)提供的 Composer 文档。

或者，如果你已经在你的系统上安装了 Composer，确保通过运行以下命令将其更新到最新版本：

```php
composer self-update

```

### 提示

本书中所使用的命令基于假设你有足够的权限来运行它们。在类 Unix 系统中，你可能需要在一些命令前加上`sudo`以便以高权限集执行命令。如果你在 Windows 上运行这些命令，你应该确保你在具有提升权限的命令提示符中运行列出的命令。确保在使用`sudo`和使用提升的命令提示符时遵循最佳实践，以确保你的系统保持安全。

一旦安装了 Composer，我们还需要安装一个名为**The Composer Asset Plugin**的全局插件（可在[`github.com/francoispluchino/composer-asset-plugin`](https://github.com/francoispluchino/composer-asset-plugin)找到）。此插件使 Composer 能够为我们管理资产文件，而无需安装额外的软件（这些程序是 Bower，由 Twitter 创建的资产依赖管理器，以及 Node Package Manager，或 NPM，它是一个 JavaScript 依赖管理器）。

```php
composer global require "fxp/composer-asset-plugin:1.0.0"

```

### 提示

由于 GitHub API 的速率限制，在安装过程中，Composer 可能会要求你输入你的 GitHub 凭据。输入凭据后，Composer 将从 GitHub 请求一个专用的 API 密钥，该密钥可用于未来的安装。确保查看[`getcomposer.org/doc/`](https://getcomposer.org/doc/)上的 Composer 文档以获取更多信息。

安装 Composer 后，我们现在可以实例化我们的应用程序。如果我们想安装现有的 Yii2 包，我们可以简单地运行以下命令：

```php
composer create-project --prefer-dist <package/name> <foldername>

```

以 Yii2 基本应用程序为例，此命令将看起来像这样：

```php
composer create-project --prefer-dist yiisoft/yii2-app-basic basic

```

运行命令后，你应该看到类似以下内容的输出：

```php
Installing yiisoft/yii2-app-basic (2.0.6)
 - Installing yiisoft/yii2-app-basic (2.0.6)
 Downloading: 100%
Created project in basic
Loading composer repositories with package information
Installing dependencies (including require-dev)
 - Installing yiisoft/yii2-composer (2.0.3)
 - Installing ezyang/htmlpurifier (v4.6.0)
 - Installing bower-asset/jquery (2.1.4)
 - Installing bower-asset/yii2-pjax (v2.0.4)
 - Installing bower-asset/punycode (v1.3.2)
 - Installing bower-asset/jquery.inputmask (3.1.63)
 - Installing cebe/markdown (1.1.0)
 - Installing yiisoft/yii2 (2.0.6)
 - Installing swiftmailer/swiftmailer (v5.4.1)
 - Installing yiisoft/yii2-swiftmailer (2.0.4)
 - Installing yiisoft/yii2-codeception (2.0.4)
 - Installing bower-asset/bootstrap (v3.3.5)
 - Installing yiisoft/yii2-bootstrap (2.0.5)
 - Installing yiisoft/yii2-debug (2.0.5)
 - Installing bower-asset/typeahead.js (v0.10.5)
 - Installing phpspec/php-diff (v1.0.2)
 - Installing yiisoft/yii2-gii (2.0.4)
 - Installing fzaninotto/faker (v1.5.0)
 - Installing yiisoft/yii2-faker (2.0.3)
Writing lock file
Generating autoload files
> yii\composer\Installer::postCreateProject
chmod('runtime', 0777)...done.
chmod('web/assets', 0777)...done.
chmod('yii', 0755)...done.

```

### 注意

你的输出可能会因系统上缓存的数据和子包的版本而略有不同。

此命令将安装 Yii2 基础应用程序到名为 basic 的文件夹中。在创建新的 Yii2 项目时，你通常会使用 create-project 命令来克隆 "yii2-app-basic"，然后从那里开始开发你的应用程序，因为基础应用程序已经预先填充了几乎所有你需要开始新项目的东西。然而，你也可以从头开始创建一个 Yii2 项目，虽然这更复杂，但它让你对你的应用程序结构有更多的控制。

让我们看看当我们运行 `create-project` 命令时创建的 `composer.json` 文件：

```php
{
    "name": "yiisoft/yii2-app-basic",
    "description": "Yii 2 Basic Application Template",
    "keywords": ["yii2", "framework", "basic", "application template"],
    "homepage": "http://www.yiiframework.com/",
    "type": "project",
    "license": "BSD-3-Clause",
    "support": {
        "issues": "https://github.com/yiisoft/yii2/issues?state=open",
        "forum": "http://www.yiiframework.com/forum/",
        "wiki": "http://www.yiiframework.com/wiki/",
        "irc": "irc://irc.freenode.net/yii",
        "source": "https://github.com/yiisoft/yii2"
    },
    "minimum-stability": "stable",
    "require": {
        "php": ">=5.4.0",
        "yiisoft/yii2": "*",
        "yiisoft/yii2-bootstrap": "*",
        "yiisoft/yii2-swiftmailer": "*"
    },
    "require-dev": {
        "yiisoft/yii2-codeception": "*",
        "yiisoft/yii2-debug": "*",
        "yiisoft/yii2-gii": "*",
        "yiisoft/yii2-faker": "*"
    },
    "config": {
        "process-timeout": 1800
    },
    "scripts": {
        "post-create-project-cmd": [
            "yii\\composer\\Installer::postCreateProject"
        ]
    },
    "extra": {
        "yii\\composer\\Installer::postCreateProject": {
            "setPermission": [
                {
                    "runtime": "0777",
                    "web/assets": "0777",
                    "yii": "0755"
                }
            ],
            "generateCookieValidationKey": [
                "config/web.php"
            ]
        },
        "asset-installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```

虽然这些项目中的大多数（如名称、描述、许可证和 require 块）相当直观，但这里也有一些特定于 Yii2 的项目需要注意。我们首先想要查看的部分是 `"scripts"` 部分：

```php
"scripts": {
    "post-create-project-cmd": [
        "yii\\composer\\Installer::postCreateProject"
    ]
}
```

此脚本告诉 Composer 当运行 `create-project` 命令时，应该运行 `postCreateProject` 静态函数。查看框架源代码，我们看到此文件在 `yii2-composer` 包中被引用（参考 [`github.com/yiisoft/yii2-composer/blob/master/Installer.php#L232`](https://github.com/yiisoft/yii2-composer/blob/master/Installer.php#L232)）。然后此命令运行几个项目创建后的操作，包括设置本地磁盘权限、生成唯一的 cookie 验证密钥以及为 composer-asset-plugin 设置一些资产安装器路径。

接下来，我们有 `"extra"` 块：

```php
"extra": {
    "yii\\composer\\Installer::postCreateProject": {
        "setPermission": [
            {
                "runtime": "0777",
                "web/assets": "0777",
                "yii": "0755"
            }
        ],
        "generateCookieValidationKey": [
            "config/web.php"
        ]
    },
    "asset-installer-paths": {
        "npm-asset-library": "vendor/npm",
        "bower-asset-library": "vendor/bower"
    }
}
```

此部分告诉 Composer 在运行 `postCreateProject` 命令时使用这些选项。这些预配置选项为我们创建应用程序提供了一个良好的起点。

# 配置

现在我们已经安装了基本应用程序，让我们看看 Yii2 自动为我们生成的几个基本配置和引导文件。

## 需求检查器

从 `yii2-app-basic` 创建的项目现在自带一个名为 `requirements.php` 的内置需求脚本。此脚本检查几个不同的值，以确保 Yii2 可以在我们的应用服务器上运行。在运行我们的应用程序之前，让我们先运行需求检查器：

```php
php requirements.php

```

你将得到类似以下内容的输出：

```php
Yii Application Requirement Checker
This script checks if your server configuration meets the requirements for running Yii application.
It checks if the server is running the right version of PHP,  if appropriate PHP extensions have been loaded, and if php.ini file settings are correct.
Check conclusion:
-----------------
PHP version: OK
[... more checks here ...]
-----------------------------------------
Errors: 0   Warnings: 6   Total checks: 21

```

通常情况下，只要错误计数设置为 `0`，我们就可以继续前进。如果需求检查器发现错误，它将在 `Check conclusion` 部分报告错误，以便你进行纠正。

### 小贴士

作为你的部署过程的一部分，建议你的部署工具运行需求检查器。这有助于确保你的应用服务器满足 Yii2 的所有要求，并且你的应用程序不会被部署到不支持它的服务器或环境中。

## 入口脚本

与其前身一样，Yii Framework 2 附带两个独立的入口脚本：一个用于网页应用程序，另一个用于控制台应用程序。

### 网页入口脚本

在 Yii2 中，Web 应用的入口脚本已从根（`/`）文件夹移动到`web/`文件夹。在 Yii1 中，我们的 PHP 文件存储在`protected/`目录中。通过将我们的入口脚本移动到`web/`目录，Yii2 通过减少我们需要运行应用程序的 Web 服务器配置来增加了我们应用程序的安全性。此外，所有公共资产（JavaScript 和 CSS）文件现在完全与我们的源代码目录隔离。如果我们打开`web/index.php`，我们的入口脚本现在看起来如下所示：

```php
<?php

// comment out the following two lines when deployed to production
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

require(__DIR__ . '/../vendor/autoload.php');
require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/../config/web.php');

(new yii\web\Application($config))->run();
```

### 提示

**下载示例代码**

本书最新和最完整的源代码副本维护在 Packt Publishing 网站上，[`www.packtpub.com`](http://www.packtpub.com)，以及 GitHub 上的[`github.com/masteringyii`](https://github.com/masteringyii)，适用于每一章的相关内容。

虽然适用于基本应用，但默认的入口脚本需要我们在迁移到不同环境时手动注释和更改代码。由于在非开发环境中更改代码不符合最佳实践，我们应该更改此代码块，这样我们就不必触摸我们的代码来将其移动到不同的环境。

我们将首先创建一个新的应用程序范围常量`APPLICATION_ENV`。该变量将由我们的 Web 服务器或控制台环境定义，并允许我们根据我们正在工作的环境动态加载不同的配置文件：

1.  在`web/index.php`中的`<?php`标签之后，添加以下代码块：

    ```php
    // Define our application_env variable as provided by nginx/apache/console
    if (!defined('APPLICATION_ENV'))
    {
        if (getenv('APPLICATION_ENV') != false)
            define('APPLICATION_ENV', getenv('APPLICATION_ENV'));
        else 
           define('APPLICATION_ENV', 'prod');
    }
    ```

    我们的应用程序现在知道如何从环境变量中读取`APPLCATTION_ENV`变量，该变量将通过我们的命令行或我们的 Web 服务器配置传递。默认情况下，如果没有设置环境，`APPLICATION_ENV`变量将被设置为 prod。

    接下来，我们希望加载一个包含多个环境常量的单独环境文件，我们将使用这些常量来动态更改我们的应用程序在不同环境中的运行方式：

    ```php
    $env = require(__DIR__ . '/../config/env.php');
    ```

    接下来，我们将配置 Yii 以根据我们的应用程序设置`YII_DEBUG`和`YII_ENV`变量：

    ```php
    defined('YII_DEBUG') or define('YII_DEBUG', $env['debug']);
    defined('YII_ENV') or define('YII_ENV', APPLICATION_ENV);
    ```

1.  然后，按照`web/`下的我们的`index.php`文件的其余部分进行操作：

    ```php
    require(__DIR__ . '/../vendor/autoload.php');
    require(__DIR__ . '/../vendor/yiisoft/yii2/Yii.php');
    (new yii\web\Application($config))->run();
    ```

通过这些更改，我们的 Web 应用程序现在已配置为能够了解其环境并加载适当的配置文件。

### 注意

不要担心；在本章的后面部分，我们将介绍如何为我们的 Web 服务器（Apache 或 NGINX）和命令行定义`APPLICATION_ENV`变量。

## 配置文件

在 Yii2 中，配置文件仍然分为控制台和 Web 特定的配置。由于这两个文件之间有许多共同点（例如我们的数据库和环境配置），我们将存储常见元素在其自己的文件中，并在我们的 Web 和控制台配置中包含这些文件。这将帮助我们遵循 DRY 标准，并减少我们应用程序中的重复代码。

### 注意

软件开发中的**DRY**（**不要重复自己**）原则指出，我们应该避免在应用程序的多个地方出现相同的代码块。通过保持应用程序 DRY，我们可以确保应用程序的性能，并减少应用程序中的错误。通过将数据库和参数的配置移动到它们自己的文件中，我们可以在 Web 和控制台配置文件中重用相同的代码。

### 网络和控制台配置文件

Yii2 支持两种不同类型的配置文件：一个用于 Web 应用程序，另一个用于控制台应用程序。在 Yii2 中，我们的 Web 配置文件存储在`config/web.php`中，我们的控制台配置文件存储在`config/console.php`中。如果你熟悉 Yii1，你会看到这两个文件的基本结构并没有发生太大的变化。

### 数据库配置

我们接下来要查看的下一个文件是我们的数据库配置文件，存储在`config/db.php`中。此文件包含我们的 Web 和控制台应用程序连接到数据库所需的所有信息。

在我们的基本应用程序中，此文件看起来如下：

```php
<?php

return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=localhost;dbname=yii2basic',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
];
```

然而，对于一个了解其环境的程序，我们应该用以下配置替换此文件，以使用我们之前定义的`APPLICATION_ENV`变量：

```php
<?php return require __DIR__ . '/env/' . APPLICATION_ENV . '/db.php';
```

### 小贴士

目前，我们只是在设置一些基本配置。我们将在下一节中介绍如何设置我们的目录。

通过这个更改，我们的应用程序现在知道它需要查看位于`config/env/<APPLICATION_ENV>/`下的一个名为`db.php`的文件，以获取该文件的正确配置环境。

### 参数配置

类似于我们的数据库配置文件，Yii 还允许我们使用一个参数文件，我们可以将应用程序的所有非组件参数存储在这个文件中。此文件位于`config/params.php`。由于基本应用程序没有使此文件了解其环境，我们将对其进行更改，如下所示：

```php
<?php return require __DIR__ . '/env/' . APPLICATION_ENV . '/params.php';
```

### 环境配置

最后，我们有之前在处理入口脚本时定义的环境配置。我们将此文件存储在`config/env.php`中，并且它应该编写如下：

```php
<?php return require __DIR__ . '/env/' . APPLICATION_ENV . '/env.php';
```

大多数现代应用程序都有几个不同的环境，这取决于它们的需求。通常，我们会将它们分为四个不同的环境：

+   我们通常遇到的第一种环境被称为**DEV**。这个环境是我们所有本地开发发生的地方。通常，开发者可以完全控制这个环境，并根据需要对其进行更改，以构建他们的应用程序。

+   我们通常遇到的第二种环境是一个名为**TEST**的测试环境。通常，我们会将应用程序部署到这个环境中，以确保我们的代码在类似生产的环境中能够正常工作；然而，当我们使用这个环境时，我们通常仍然可以访问高日志级别和调试信息。

+   我们通常拥有的第三个环境被称为 **UAT**，即用户验收测试环境。这是一个独立的环境，我们将将其提供给我们的客户或业务利益相关者，以便他们可以测试应用程序，以验证它是否按他们期望的方式工作。

+   最后，在我们的典型设置中，我们会有一个 **PROD** 或生产环境。这是我们的代码最终部署的地方，也是所有用户最终与我们的应用程序交互的地方。

如前几节所述，我们已经将所有环境配置文件指向了 `config/env/<env>` 文件夹。由于我们的本地环境将被命名为 `DEV`，我们将首先创建它：

1.  我们将首先从命令行创建我们的 `DEV` 环境文件夹：

    ```php
    mkdir –p config/env/dev

    ```

1.  接下来，我们将在 `config/env/dev/` 下的 `db.php` 中创建我们的 `dev` 数据库配置文件。目前，我们将坚持使用基本的 SQLite 数据库：

    ```php
    <?php return [
        'dsn' => 'sqlite:/' . __DIR__ . '/../../../runtime/db.sqlite',
          'class' => 'yii\db\Connection',
        'charset' => 'utf8'
    ];
    ```

1.  接下来，我们将在 `config/env/dev` 下的 `env.php` 中创建我们的环境配置文件。如果您还记得本章前面的内容，这就是我们存储 `debug` 标志的地方，因此这个文件将如下所示：

    ```php
    <?php return [ 
        'debug' => true
    ];
    ```

1.  最后，我们将在 `config/env/dev/` 下创建我们的 `params.php` 文件。到目前为止，这个文件将简单地返回一个空数组：

    ```php
    <?php return [];
    ```

现在，为了简单起见，让我们将此配置复制到我们的其他环境中。从命令行，我们可以这样做：

```php
cp –R config/env/dev config/env/test
cp –R config/env/dev config/env/uat
cp –R config/env/dev config/env/prod

```

## 设置我们的应用程序环境

现在我们已经告诉 Yii 它需要使用哪些文件和配置来为每个环境，我们需要告诉它使用哪个环境。为此，我们将在我们的 Web 服务器配置中设置自定义变量，这将把这个选项传递给 Yii。

## 设置 NGINX 的网络环境

在我们的控制台应用程序正确配置后，我们现在需要配置我们的 Web 服务器以将 `APPLICATION_ENV` 变量传递给我们的应用程序。在典型的 NGINX 配置中，我们有一个如下所示的位置块：

```php
location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        fastcgi_pass   127.0.0.1:9000;
        #fastcgi_pass unix:/var/run/php5-fpm.sock;
        try_files $uri =404;
    }
```

要将 `APPLICATION_ENV` 变量传递给我们的应用程序，我们只需定义一个新的 `fastcgi_param`，如下所示：

```php
fastcgi_param   APPLICATION_ENV "dev";
```

在进行此更改后，只需重新启动 NGINX。

## 设置 Apache 的网络环境

我们也可以轻松地配置 Apache 将 `APPLICATION_ENV` 变量传递给我们的应用程序。使用 Apache，我们通常有一个如下所示的 VirtualHost 块：

```php
# Set document root to be "basic/web"
DocumentRoot "path/to/basic/web"

<Directory "path/to/basic/web">
    # use mod_rewrite for pretty URL support
    RewriteEngine on
    # If a directory or a file exists, use the request directly
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    # Otherwise forward the request to index.php
    RewriteRule . index.php

    # ...other settings...
</Directory>
```

要将 `APPLICATION_ENV` 变量传递给我们的应用程序，我们只需使用如下所示的 `SetEnv` 命令，这个命令可以放置在我们的 VirtualHost 块的任何位置：

```php
SetEnv   APPLICATION_ENV dev
```

在进行此更改后，只需重新启动 Apache 并导航到您的应用程序。

在最基本的层面上，我们的应用程序与我们第一次运行 composer `create-project` 命令时并没有做任何不同的事情。尽管没有做任何不同的事情，但我们的应用程序现在比之前的变化要强大得多，也更加灵活。在本书的后面部分，我们将探讨这些特定的变化如何使我们的应用程序的自动化部署变得无缝且简单。

# 组件和对象

Yii2 中几乎所有内容都扩展自两个基类：`Component` 类和 `Object` 类。

## 组件

在 Yii2 中，`Component` 类取代了 Yii1 中的 `CComponent` 类。在 Yii1 中，组件作为服务定位器，托管一组特定的应用程序组件，为请求处理提供不同的服务。在 Yii2 中，每个组件都可以使用以下语法进行访问：

```php
Yii::$app->componentID
```

例如，可以使用以下方式访问数据库组件：

```php
Yii::$app->db
```

可以使用以下方式访问缓存组件：

```php
Yii::$app->cache
```

Yii2 通过我们之前提到的应用程序配置自动在运行时按名称注册每个组件。

为了提高 Yii2 应用程序的性能，组件是延迟加载的，或者只有在第一次访问时才实例化。这意味着如果缓存组件在你的应用程序代码中从未使用过，缓存组件将永远不会被加载。然而，有时这并不理想，因此为了强制加载一个组件，你可以通过将其添加到配置文件中的 `bootstrap` 配置选项来引导它。例如，如果我们想引导日志组件，我们可以这样做：

```php
<?php return [
    'bootstrap' => [
        'log'
    ],
    […]
]
```

`bootstrap` 选项的行为类似于 Yii1 中的预加载选项——任何你想要或需要在引导时实例化的组件，如果它在配置文件的 `bootstrap` 部分中，将会被加载。

### 注意

关于服务定位器和组件的更多信息，请确保阅读位于 [`www.yiiframework.com/doc-2.0/guide-concept-service-locator.html`](http://www.yiiframework.com/doc-2.0/guide-concept-service-locator.html) 和 [`www.yiiframework.com/doc-2.0/guide-structure-application-components.html`](http://www.yiiframework.com/doc-2.0/guide-structure-application-components.html) 的 *Definitive Guide to Yii* 指南。

## 对象

在 Yii2 中，几乎所有不扩展自 `Component` 类的类都扩展自 `Object` 类。`Object` 类是实现了属性特征的基类。在 Yii2 中，属性特征允许你访问有关对象的大量信息，例如 `__get` 和 `__set` 魔法方法，以及其他实用函数，如 `hasProperty()`、`canGetProperty()` 和 `canSetProperty()`。结合这些功能，使得 Yii2 中的对象非常强大。

### 小贴士

`object` 类非常强大，许多 Yii 类都从它扩展。尽管如此，自己使用魔法方法 `__get` 和 `__set` 并不是最佳实践，因为它比原生 PHP 方法慢，并且与你的 IDE 自动完成工具和文档工具的集成不佳。

# 路径别名

在 Yii2 中，路径别名用于表示文件路径或 URL 路径，这样我们就不需要在我们的应用程序中直接硬编码路径或 URL。在 Yii2 中，别名始终以 `@` 符号开头，这样 Yii 就知道如何将其与文件路径或 URL 区分开来。

别名可以通过多种方式定义。定义新别名的最基本方法是调用 `\Yii::setAlias()`：

```php
\Yii::setAlias('@path', '/path/to/example');
\Yii::setAlias('@example, 'https://www.example.com');
```

也可以通过在应用程序配置文件中设置别名选项来定义别名，如下所示：

```php
return [
    // ...
    'aliases' => [
        '@path => '/path/to/example,
        '@example' => 'https://www.example.com',
    ],
];
```

此外，可以使用 `\Yii::getAlias()` 轻松检索别名：

```php
\Yii::getAlias('@path') // returns /path/to/example
\Yii::getAlias('@example') // returns https://www.example.com
```

Yii 中的几个地方是别名感知的，并且会接受别名作为输入。例如，`yii\caching\FileCache` 接受文件别名作为 `$cachePath` 参数的别名：

```php
$cache = new FileCache([
    'cachePath' => '@runtime/cache',
]);
```

### 注意

关于路径别名的更多信息，请查看 Yii 文档，网址为 [`www.yiiframework.com/doc-2.0/guide-concept-aliases.html`](http://www.yiiframework.com/doc-2.0/guide-concept-aliases.html)。

# 摘要

在本章中，我们介绍了如何通过 composer 创建新的 Yii2 应用程序。我们还介绍了 Yii2 伴随的基本配置文件，以及如何配置我们的 Web 应用程序以加载特定环境的配置文件。最后，我们还涵盖了组件、对象和路径别名，这些都是掌握 Yii 的基础。

在下一章中，我们将涵盖你需要知道的一切，以便成为控制台命令和应用程序的大师。
