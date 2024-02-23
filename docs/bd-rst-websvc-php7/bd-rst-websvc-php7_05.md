# 第五章：使用 Composer 进行加载和解析，一个进化

Composer 不仅是一个包管理器，也是 PHP 中的依赖管理器。在 PHP 中，如果你想要重用一个开源组件，标准的做法是通过 Composer 使用开源包，因为 Composer 已经成为了制作包、安装包和自动加载的标准。在这里，我们讨论了一些新术语，比如包管理器、依赖管理器和自动加载。在本章中，我们将详细讨论它们是什么，以及 Composer 为它们提供了什么。

前面的段落解释了 Composer 主要的功能，但 Composer 不仅仅是这样。

在本章中，我们将看到以下内容：

+   Composer 简介

+   安装

+   使用 Composer

+   Composer 作为包和依赖管理器

+   安装包

+   Composer 的工作原理

+   Composer 命令

+   `composer.json`文件

+   `composer.lock`文件

+   Composer 作为自动加载程序

# Composer 简介

PHP 社区有点分裂，有很多不同的框架和库。由于有不同的框架可用，为一个框架编写的插件或包不能在另一个框架中使用。因此，应该有一种标准的方法来编写和安装包。这就是 Composer 的作用。Composer 是编写、分发和安装包的标准方法。Composer 受到了`node.js`生态系统中的**npm**（**Node Package Manager**）的启发。

事实上，大多数开发人员使用 Composer 来安装他们使用的不同包。这也是因为使用 Composer 安装包很方便，因为通过 Composer 安装的包也可以很容易地通过 Composer 进行自动加载。我们将在本章后面讨论自动加载。

如前所述，Composer 不仅是一个包管理器，也是一个依赖管理器。这意味着如果一个包需要某些东西，Composer 将为其安装这些依赖项，然后相应地进行自动加载。

# 安装

Composer 需要 PHP 5.3.2+才能运行。以下是你可以在不同平台上安装 Composer 的方法。

# 在 Windows 上安装

在 Windows 上，安装 Composer 非常容易，所以我们不会详细介绍。你只需要从[getcomposer.org](http://getcomposer.org)下载并执行 Composer 安装程序。这是链接：[`getcomposer.org/Composer-Setup.exe`](https://getcomposer.org/Composer-Setup.exe)。

# 在 Linux/Unix/OS X 上安装

安装 Composer 有两种方式，分别是本地安装和全局安装。你可以通过以下命令简单地安装 Composer：

```php
$ php -r "copy('https://getcomposer.org/installer', 'Composer-setup.php');"
$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"
```

前面的四个命令分别执行以下任务：

1.  下载 Composer 安装 PHP 文件

1.  通过检查 SHA-384 验证安装程序

1.  运行 Composer 安装程序安装 Composer

1.  删除已下载的 Composer 安装文件

如果你对这个 Composer 安装做了什么很好奇，那就留下第四个命令。你可以看到，安装文件是一个名为`composer-setup.php`的 PHP 文件；你可以简单地打开这个文件并阅读代码。它主要是检查几个 PHP 扩展和设置，并创建`composer.phar`文件。这个`composer.phar`将负责执行 Composer 的任务。我们将很快在本章中看一下 Composer 的功能以及它执行的操作或任务。

使用上述命令，我们已经在本地安装了 Composer。默认情况下，它会在当前目录安装 Composer，这意味着通过安装 Composer 来放置`composer.phar`文件，因为这个`composer.phar`文件执行 Composer 功能。

如果你希望在特定目录中安装 Composer（放置`composer.phar`）或将`composer.phar`名称更改为其他名称，你可以简单地使用不同的参数运行安装，比如：

```php
php composer-setup.php --install-dir=bin --filename=composer
```

还有许多其他参数，你可以在[`getcomposer.org/download/`](https://getcomposer.org/download/)的“安装程序选项”下看到。

如果您已经在本地安装了 Composer，并且文件名为`composer.phar`，您可以通过以下方式简单地通过 PHP 运行它：

```php
php composer.phar
```

这将运行 Composer。如果`composer.phar`不在之前`composer.phar`的相同目录中，则需要附加`composer.phar`文件所在目录的路径。这是因为我们已经在本地安装了 Composer。因此，让我们看看如何全局安装它。

# 全局安装

要在任何地方安装和访问 Composer，我们需要将其放在系统的`PATH`目录中添加的目录中。

为此，我们可以运行以下命令将`composer.phar`移动到一个我们可以全局访问的位置：

```php
sudo mv composer.phar /usr/local/bin/Composer
```

现在，您只需通过运行命令`composer`就可以访问 Composer，并且它将正常工作；不需要其他任何东西。所以，如果您说：

```php
composer -V
```

它将返回类似以下的内容：

```php
composer version 1.4.2 2017-05-17 08:17:52
```

您可以从任何地方运行此命令，因为我们已经全局安装了 Composer。

# Composer 的用法

Composer 是一个依赖管理器，还有其他不同的用途。Composer 用于在解决依赖关系的同时安装软件包。Composer 在自动加载方面也非常出色。Composer 还有更多的用途。在这里，我们将讨论 Composer 的不同用途。

# Composer 作为依赖管理器

Composer 是一个依赖管理器。现在，您可以以一种不需要将第三方依赖项与其一起提供的方式打包您的代码。您只需要告诉它的依赖关系。实际上，您的软件包依赖关系可能有更多的依赖关系，而这些依赖关系也可能有更多的依赖关系。因此，在制作软件包或捆绑包时解决所有这些依赖关系可能会非常繁琐。但是，由于 Composer 的存在，这并不是问题。

由于 Composer 也是一个依赖管理器，因此依赖关系不再是问题。我们只需在一个 JSON 文件中指定依赖关系，Composer 就会解决这些依赖关系。我们将很快看一下该 JSON 文件。

# 安装软件包

如果我们在名为`composer.json`的 JSON 文件中有依赖关系（我们的工作依赖或将要依赖的其他软件包），那么我们可以通过 Composer 安装它们。

PHP 中有很多好的软件包，我们日常工作相关的大部分内容都是可用的，那么谁会想要重新发明轮子并重新创建所有东西呢？因此，要启动一个项目，我们可以通过 Composer 简单地安装不同的软件包，并重用已经存在的大量代码。

现在，问题是，Composer 可以从哪里安装？它可以是互联网上的任何地方吗？还是有一些固定的地方可以安装 Composer 软件包？实际上，可以有多个来源。Composer 安装软件包的一个默认位置是 Packagist [`packagist.org/`](https://packagist.org/)。

因此，我们将从 Packagist 安装一个软件包。假设我们想要从 Packagist 安装一个 PHP 单元测试框架软件包。它在这里可用：[`packagist.org/packages/phpunit/phpunit`](https://packagist.org/packages/phpunit/phpunit)。

因此，让我们使用以下命令进行安装：

```php
composer require phpunit/phpunit
```

您将看到此软件包安装还将导致许多依赖项的安装。在这里，`require`是 Composer 命令，而`phpunit/phpunit`是此软件包在 Packagist 上注册的名称。请注意，我们刚刚讨论了`composer.json`文件，但我们不需要`composer.json`文件来安装此 PHP 单元软件包。实际上，如果我们已经有一些依赖关系，`composer.json`文件是有用的。如果我们现在只需要安装一些软件包，那么我们可以简单地使用`composer require`命令。而且这个`composer require`还会创建一个`composer.json`文件，并将其更新为`phpunit/phpunit`软件包。

这是在运行上述命令后将创建的`composer.json`文件的内容：

```php
{
    "require": {
        "phpunit/phpunit": "⁶.2"
    }
}
```

您可以在 require 对象中看到，它具有包名称的键，然后在冒号后面是 `"⁶.2"`，表示包版本。在这里，包版本以正则表达式给出，表示包版本从 6.2 开始，但这并不是实际安装的版本。安装包及其依赖项后，它们的确切版本将写入 `composer.lock` 文件中。这个文件 `composer.lock` 具有重要意义，所以我们很快将详细了解它。

运行此命令后，您将能够在运行 Composer require 命令的目录中看到另一个目录。这个目录是 `vendor` 目录。在 vendor 目录中，安装了所有的包。如果您查看它，您会发现不仅 PHP 单元存在于 vendor 目录中，而且所有的依赖项和依赖项的依赖项都安装在 vendor 目录中。

# 使用 composer.json 进行安装

除了使用 `composer require`，如果有一个 `composer.json` 文件，我们还可以通过另一个命令安装包。为此，进入另一个目录。我们可以简单地创建一个包含以下内容的 `composer.json` 文件：

```php
{
    "require": {
        "phpunit/phpunit": "⁶.2",
        "phpspec/phpspec": "³.2"

    }
}
```

因此，一旦您有一个名为 `composer.json` 的文件，并且其中包含这些内容，您可以通过运行此命令根据这些版本信息安装这两个包及其依赖项：

```php
composer install
```

这将执行与 Composer require 相同的操作。但是，如果同时存在 `composer.json` 和 `composer.lock` 文件，它将从 `composer.lock` 文件中读取信息，并安装该确切版本，忽略 `composer.json`。

如果要忽略 `composer.lock` 文件并根据 `composer.json` 文件中的信息进行安装，可以删除 `composer.lock` 文件并使用 `composer install`，或者运行：

```php
composer update
```

注意，`composer update` 命令也会更新 `composer.lock` 文件。

如果一个包或库在 Packagist 上不可用，您仍然可以通过其他来源安装该包，为此，您需要在 `composer.json` 文件中输入不同的信息。您可以在这里阅读有关其他来源的详细信息 [`getcomposer.org/doc/05-repositories.md`](https://getcomposer.org/doc/05-repositories.md)。但是，请注意，由于其便利性，Packagist 是推荐的来源。

# 详细了解 composer.json

我们看到的 `composer.json` 文件是最小的。要了解典型的 `composer.json` 文件是什么样的，这里是我最喜欢的 PHP MVC 框架 Laravel 的 `composer.json` 文件：

```php
{
    "name": "laravel/laravel",
    "description": "The Laravel Framework.",
    "keywords": ["framework", "laravel"],
    "license": "MIT",
    "type": "project",
    "require": {
        "php": ">=5.6.4",
        "laravel/framework": "5.4.*",
        "laravel/tinker": "~1.0"
    },
    "require-dev": {
        "fzaninotto/faker": "~1.4",
        "mockery/mockery": "0.9.*",
        "phpunit/phpunit": "~5.7"
    },
    "autoload": {
        "classmap": [
            "database"
        ],
        "psr-4": {
            "App\\": "app/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
    "scripts": {
        "post-root-package-install": [
            "php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "php artisan key:generate"
        ],
        "post-install-cmd": [
            "Illuminate\\Foundation\\ComposerScripts::postInstall",
            "php artisan optimize"
        ],
        "post-update-cmd": [
            "Illuminate\\Foundation\\ComposerScripts::postUpdate",
            "php artisan optimize"
        ]
    },
    "config": {
        "preferred-install": "dist",
        "sort-packages": true,
        "optimize-autoloader": true
    }
}
```

我们不会深入研究此文件的明显部分，比如名称、描述等。我们将研究复杂和更重要的属性。

# require 对象

您已经看到 `require` 具有依赖项和版本信息，这些信息由 `composer install` 命令安装。

# require-dev 对象

在 `require-dev` 中，只列出了在开发阶段需要的那些包。在 Composer install 示例中，我们使用了 `phpunit/phpunit` 的示例，但实际上，像 `phpunit` 和 `phpspec` 这样的包只在开发阶段需要，而不是在生产中需要。此外，如果有任何与调试相关的包需要，也可以包含在 `require-dev` 对象中。`composer install` 命令将安装所有在 `require-dev` 中以及 `require` 对象下的包。

但是，如果我们只想安装生产环境中需要的包，可以使用以下命令进行安装：

```php
composer install --no-dev
```

在上述示例中，`composer.json` 中的 `laravel/tinker` 和 `laravel/laravel` 在 `require` 对象中，但 `phpunit`、`mockery` 和 `faker` 是在 `require-dev` 对象中提到的包，因此它们不会被安装。

# autoload 和 autoload-dev

这个 autoload 选项是用来自动加载一个命名空间或一组类到一个目录下，或者简单地加载一个类。这是 Composer 提供的 PHP 自动加载程序的替代方案。这告诉 Composer 在自动加载类时要查找哪个目录。

自动加载属性还有两个属性，即`classmap`和`psr-4`。PSR-4 是一种描述从文件路径自动加载类的规范。您可以在[`www.php-fig.org/psr/psr-4/.`](http://www.php-fig.org/psr/psr-4/)上阅读更多信息。

在这里，PSR-4 指定了一个命名空间，以及这个命名空间应该从哪里加载。在前面的示例中，`App`命名空间应该从`app`目录获取内容。

另一个属性是`classmap`。这用于自动加载不支持 PSR-4 或 PSR-0 的库。PSR-0 是另一种自动加载的标准，但是 PSR-4 是更新的，是推荐的。PSR-0 已经被弃用。

就像`require-dev`类似于`require`一样，`autoload-dev`类似于`autoload`。

# 脚本

脚本基本上在不同事件的数组中具有脚本。脚本对象的所有这些属性都是事件，以及在特定事件上执行的值指定的脚本。不同的属性代表不同的事件，例如`post-install-cmd`表示在安装包后，它将执行`post-install-cmd`属性下的数组中的脚本。其他事件也是一样。在 Composer 文档的此 URL 中，您可以找到所有这些事件的详细信息：[`getcomposer.org/doc/articles/scripts.md#command-events`](https://getcomposer.org/doc/articles/scripts.md#command-events)。

# composer.lock

`composer.lock`的主要目的是锁定依赖项。

正如讨论的那样，`composer.lock`文件非常重要。这是因为当`composer.json`中没有指定特定的确切版本，或者通过`composer require`安装包时没有版本信息时，Composer 会安装该包，并在安装后添加关于该包安装的信息，包括确切的版本。

如果包已经在`composer.lock`中，那么很可能您也在`composer.json`中列出了该包。在这种情况下，您通常通过`composer install`安装该包，Composer 将从`composer.lock`中读取包详细信息和版本信息，并安装确切的版本，因为这就是 Composer 锁定依赖项的方式。

如果您的代码库中没有`composer.lock`文件，`composer install`或`composer require`将安装包，这些包将创建`composer.lock`文件。

如果`composer.lock`文件已经存在，那么 Composer 安装将确保安装`composer.lock`文件中写入的确切版本，并且它将忽略`composer.json`。然而，如前所述，如果您想要更新依赖项并想要在`composer.lock`文件中更新它，那么您可以运行`composer update`。这是不推荐的，因为一旦您的应用程序在特定的依赖项上运行，并且您不想更新，那么`composer.lock`文件就很有用。因此，如果您想锁定依赖项，请不要运行`composer update`命令。

如果您在团队中工作，您必须提交`composer.lock`文件，以便您的团队中的其他成员可以拥有完全相同的包和版本。因此，强烈建议提交`composer.lock`文件，这不是讨论的问题。

我们不会详细讨论`composer.lock`，因为这就是我们需要了解的大部分内容。但是，我建议您打开并阅读一次`composer.lock`。不必理解所有内容，但这会给您一些想法。

它基本上包含已安装的包信息及其依赖项的确切版本。

# Composer 作为自动加载程序

正如您所见，`composer.json`文件中有与自动加载相关的信息，因为 Composer 也负责自动加载。实际上，即使没有指定自动加载属性，Composer 也可以用于自动加载文件。

以前，我们使用`require`或`include`来单独加载每个文件。您不需要单独`require`或`include`每个文件。您只需要要求或包含一个文件，即`./vendor/autoload.php`。这个`vendor`目录是 Composer 的供应商目录，所有的包都放在那里。因此，这个`autoload.php`文件将自动加载所有内容，而不必担心按顺序包含所有文件及其依赖关系。

# 例子

假设我们有一个像这样的`composer.json`文件：

```php
{
  "name": "laravel/laravel",
  "description": "The Laravel Framework.",
  "keywords": [
    "framework",
    "laravel"
  ],
  "license": "MIT",
  "type": "project",
  "require": {
    "php": ">=5.6.4",
    "twilio/sdk": "5.9.1",
    "barryvdh/laravel-debugbar": "².3",
    "barryvdh/laravel-ide-helper": "².3",
    "cartalyst/sentinel": "2.0.*",
    "gocardless/gocardless-pro": "¹.0",
    "intervention/image": "².3",
    "laravel/framework": "5.3.*",
    "laravelcollective/html": "⁵.3.0",
    "lodge/postcode-lookup": "dev-master",
    "nahid/talk": "².0",
    "predis/predis": "¹.1",
    "pusher/pusher-php-server": "².6",
    "thujohn/twitter": "².2",
    "vinkla/pusher": "².4"
  },
  "require-dev": {
    "fzaninotto/faker": "~1.4",
    "mockery/mockery": "0.9.*",
    "phpunit/phpunit": "~5.0",
    "symfony/css-selector": "3.1.*",
    "symfony/dom-crawler": "3.1.*"
  },
  "autoload": {
    "classmap": [
      "app/Models",
      "app/Traits"
    ],
    "psr-4": {
      "App\\": "app/"
    }
  },
  "autoload-dev": {
    "classmap": [
      "tests/TestCase.php"
    ]
  }
}
```

有了那个`composer.json`文件，如果我们运行`composer install`，它将安装所有这些包，然后加载所有这些包和所有类：

```php
      "app/Models",
      "app/Traits"
```

我们只需要包含一个文件，就像这样：

```php
require __DIR__.'/vendor/autoload.php';
```

这将使所有这些包在您的代码中可用。因此，所有这些包，以及我们自己在`app/Models`和`app/Traits`中的`classes/traits`，即使我们没有单独包含所有这些包，也将可用。因此，Composer 也可以作为自动加载程序。

# 用于创建项目的 Composer

我们还可以使用 Composer 从现有包创建一个新项目。这相当于执行两个步骤：

+   克隆存储库

+   在那里运行`composer install`

这意味着它将克隆一个项目并安装它的依赖项。可以使用以下命令完成：

```php
composer create-project <package name> <path on file system> <version info>
```

如果我们想要从一个代码库开始一个项目，这是非常有用的。请注意，文件系统上的路径和版本号不是必需的，但是可选的。

# 例子

要安装一个名为 Laravel 的框架，您可以简单地运行：

```php
composer create-project laravel/laravel
```

在这里，`laravel/laravel`是包。从这可以看出，文件系统上的路径或版本在这里没有提到。这是因为这些参数是可选的。

使用这些参数，该命令将如下所示：

```php
composer create-project laravel/laravel ./exampleproject 5.3
```

# 摘要

Composer 是一种制作和使用可重用组件的标准方法。如今，已经做了很多事情，可以被重用。因此，在 PHP 中，Composer 是一种标准方法。在本章中，我们已经看到了 Composer 的工作原理，它的用途是什么，如何通过它安装包等等。然而，在本章中我们没有涉及的一件事是如何为 Composer 制作包。这是因为我们的重点是如何重用已经可用的 Composer 包。如果您想学习如何创建 Composer 包，那么就从这里开始：[`getcomposer.org/doc/02-libraries.md`](https://getcomposer.org/doc/02-libraries.md)。

如果您想了解更多关于 Composer 的信息，您可以：

1.  去阅读 Composer 文档[`getcomposer.org/doc/`](https://getcomposer.org/doc/)。

1.  打开并开始阅读重要文件。您可以打开并阅读不同的 Composer 文件，比如`composer.json`和`composer.lock`，来自不同的包。

到目前为止，我们已经看到了如何重用 Composer 组件，以避免自己编写所有内容。在下一章中，我们将开始使用这些组件或项目来使我们的 RESTful web 服务更好。
