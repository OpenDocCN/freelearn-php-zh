# 理解 Laravel 5 的核心概念

正如本章的标题所暗示的，我们将提供 Laravel 框架的概述，涵盖与使用 Web 服务架构开发 Web 应用程序相关的主要概念。更确切地说，我们将在本书中使用 RESTful 架构。

我们假设您已经对 RESTful 架构以及 Web 服务（这里我们称之为**应用程序编程接口**（**API**）端点）的工作原理有基本了解。

但是，如果您对这个概念还很陌生，不用担心。我们将帮助您入门。

Laravel 框架将是一个有用的工具，因为使用它，我们控制器中的所有数据将默认转换为 JSON 格式。

Laravel 框架是开发 Web 应用程序的强大工具，使用“约定优于配置”的范式。 Laravel 开箱即用具有构建现代 Web 应用程序所需的所有功能，使用**模型视图控制器**（**MVC**）。此外，Laravel 框架是当今最受欢迎的 PHP 框架之一，用于开发 Web 应用程序。

从现在到本书结束，我们将简称 Laravel 框架为 Laravel。

Laravel 生态系统绝对令人难以置信。诸如 Homestead、Valet、Lumen 和 Spark 之类的工具进一步丰富了使用 PHP 进行 Web 软件开发的体验。

有许多方法可以使用 Laravel 开始开发 Web 应用程序，这意味着有许多方法可以配置您的本地环境或生产服务器。本章不偏向任何特定方式；我们理解每个开发人员随着时间的推移都有自己的偏好。

无论您对工具、服务器、虚拟机、数据库等有何偏好，我们将专注于主要概念，并不假设某种方式是对还是错。本章仅用于说明主要概念和需要执行的操作。

请记住，无论您选择哪种方法（使用 Homestead、WAMP、MAMP 或 Docker），Laravel 都有一些极其必要的依赖项（或服务器要求），这对于开发 Web 应用程序非常重要。

您可以在官方 Laravel 文档中找到更多有用的信息：[`laravel.com/docs/5.6`](https://laravel.com/docs/5.6)。

在本章中，我们将涵盖以下内容：

+   搭建环境

+   Laravel 应用程序的基本架构

+   Laravel 应用程序生命周期

+   Artisan CLI

+   MVC 和路由

+   与数据库连接

# 搭建环境

请记住，无论您如何配置环境来使用 PHP 和 Laravel 开发 Web 应用程序，牢记主要的服务器要求，您将能够跟随本章的示例。

需要注意的是，某些操作系统没有安装 PHP。例如 Windows 机器，这里有一些替代方案供您创建开发环境：

+   HOMESTEAD（Laravel 文档推荐）：[`laravel.com/docs/5.6/homestead`](https://laravel.com/docs/5.6/homestead)

+   MAMP: [`www.mamp.info/en/`](https://www.mamp.info/en/)

+   XAMPP：[`www.apachefriends.org/index.html`](https://www.apachefriends.org/index.html)

+   WAMP SERVER（仅适用于 Windows 操作系统）：[`www.wampserver.com/en/`](http://www.wampserver.com/en/)

+   PHPDOCKER: [`www.docker.com/what-docker`](https://www.docker.com/what-docker)

# 安装 Composer 包管理器

Laravel 使用**Composer**，这是 PHP 的依赖管理器，与 Node.js 项目的**Node Package Manager**（NPM）、Python 的 PIP 和 Ruby 的 Bundler 非常相似。让我们看看官方文档对此的说法：

“Composer 是 PHP 中的依赖管理工具。它允许您声明项目所依赖的库，并将为您管理（安装/更新）它们。”

因此，让我们按照以下步骤安装 Composer：

转到[`getcomposer.org/download/`](https://getcomposer.org/download/)并按照您的平台的说明进行操作。

您可以在[`getcomposer.org/doc/00-intro.md`](https://getcomposer.org/doc/00-intro.md)上获取更多信息。

请注意，您可以在本地或全局安装 Composer；现在不用担心。选择对您来说最容易的方式。

所有使用 Composer 的 PHP 项目在根项目中都有一个名为`composer.json`的文件，看起来类似于以下内容：

```php
{
 "require": {
     "laravel/framework": "5.*.*",
 }
}
```

这也与 Node.js 和 Angular 应用程序上的`package.json`文件非常相似，我们将在本书后面看到。

这是关于基本命令的有用链接：[`getcomposer.org/doc/01-basic-usage.md`](https://getcomposer.org/doc/01-basic-usage.md)

# 安装 Docker

我们将在本章中使用 Docker。尽管 Laravel 的官方文档建议使用带有虚拟机和 Vagrant 的 Homestead，但我们选择使用 Docker，因为它启动快速且易于使用，我们的主要重点是 Laravel 的核心概念。

您可以在[`www.docker.com/what-docker`](https://www.docker.com/what-docker)上找到有关 Docker 的更多信息。

根据 Docker 文档的说法：

Docker 是推动容器运动的公司，也是唯一一个能够应对混合云中的每个应用程序的容器平台提供商。今天的企业面临着数字转型的压力，但受到现有应用程序和基础设施的限制，同时需要合理化日益多样化的云、数据中心和应用程序架构组合。Docker 实现了应用程序和基础设施以及开发人员和 IT 运营之间的真正独立，释放了它们的潜力，并创造了更好的协作和创新模式。

让我们按照以下步骤安装 Docker：

1.  转到[`docs.docker.com/install/`](https://docs.docker.com/install/)。

1.  选择您的平台并按照安装步骤进行操作。

1.  如果你遇到任何问题，请查看[`docs.docker.com/get-started/`](https://docs.docker.com/get-started/)上的入门链接。

由于我们正在使用 Docker 容器和镜像来启动我们的应用程序，并且不会深入探讨 Docker 在幕后的工作原理，这里是一些 Docker 命令的简短列表：

| **命令**： | **描述**： |
| --- | --- |
| `docker ps` | 显示正在运行的容器 |
| `docker ps -a` | 显示所有容器 |
| `docker start` | 启动容器 |
| `docker stop` | 停止容器 |
| `docker-compose up -d` | 在后台启动容器 |
| `docker-compose stop` | 停止`docker-compose.yml`文件上的所有容器 |
| `docker-compose start` | 启动`docker-compose.yml`文件上的所有容器 |
| `docker-compose kill` | 杀死`docker-compose.yml`文件上的所有容器 |
| `docker-compose logs` | 记录`docker-compose.yml`文件上的所有容器 |

您可以在[`docs.docker.com/engine/reference/commandline/docker/`](https://docs.docker.com/engine/reference/commandline/docker/)上查看完整的 Docker 命令列表。以及在[`docs.docker.com/compose/reference/overview/#command-options-overview-and-help`](https://docs.docker.com/compose/reference/overview/#command-options-overview-and-help)上查看 Docker-compose 命令。

# 配置 PHPDocker.io

PHPDocker.io 是一个简单的工具，它帮助我们使用 Compose 构建 PHP 应用程序的 Docker/容器概念。它非常易于理解和使用；因此，让我们看看我们需要做什么：

1.  转到[`phpdocker.io/`](https://phpdocker.io/)。

1.  单击生成器链接。

1.  填写信息，如下截图所示。

1.  单击“生成项目存档”按钮并保存文件夹：

![](img/0bfba06b-93cf-41eb-baa6-1162b0f4fe5d.png)PHPDocker 界面

数据库配置如下截图所示：

![](img/3d363633-a23b-4ed5-a8cf-d49b15f3eea7.png)数据库配置请注意，我们在前面的配置中使用了 MYSQL 数据库的最新版本，但您可以选择任何您喜欢的版本。在接下来的示例中，数据库版本将不重要。

# 设置 PHPDocker 和 Laravel

既然我们已经填写了之前的信息并为我们的机器下载了文件，让我们开始设置我们的应用程序，以便更深入地了解 Laravel 应用程序的目录结构。

执行以下步骤：

1.  打开`bash/Terminal/cmd`。

1.  在 Mac 和 Linux 上转到`Users/yourname`，或者在 Windows 上转到`C:/`。

1.  在文件夹内打开您的终端并输入以下命令：

```php
composer create-project --prefer-dist laravel/laravel chapter-01
```

在您的终端窗口底部，您将看到以下结果：

```php
Writing lock file Generating autoload files > Illuminate\Foundation\ComposerScripts::postUpdate > php artisan optimize Generating optimized class loader
php artisan key:generate
```

1.  在终端窗口中，输入：

```php
cd chapter-01 && ls
```

结果将如下所示：

![](img/460a8a5e-ccea-4297-8b42-84a7d4ce6066.png)终端窗口输出

恭喜！您有了您的第一个 Laravel 应用程序，使用了`Composer`包管理器构建。

现在，是时候将我们的应用程序与从 PHPDocker（我们的 PHP/MySQL Docker 截图）下载的文件连接起来了。要做到这一点，请按照以下步骤进行操作。

1.  获取下载的存档`hands-on-full-stack-web-development-with-angular-6-and-laravel-5.zip`，并解压缩它。

1.  复制所有文件夹内容（一个`phpdocker`文件夹和一个名为`docker-compose.yml`的文件）。

1.  打开`chapter-01`文件夹并粘贴内容。

现在，在`chapter-01`文件夹内，我们将看到以下文件：

![](img/002f2cd2-0736-445b-a096-510ba7b07fba.png)chapter-01 文件夹结构

让我们检查一下，确保一切都会顺利进行我们的配置。

1.  打开您的终端窗口并输入以下命令：

```php
docker-compose up -d
```

重要的是要记住，在这一点上，您需要在您的机器上启动和运行 Docker。如果您完全不知道如何在您的机器上运行 Docker，您可以在[`github.com/docker/labs/tree/master/beginner/`](https://github.com/docker/labs/tree/master/beginner/)找到更多信息。

1.  请注意，此命令可能需要更多时间来创建和构建所有的容器。结果将如下所示：

![](img/d7931a7a-4c40-47eb-9e03-b8d66cb23a7e.png)Docker 容器已启动

前面的截图表明我们已成功启动了所有容器：`memcached`，`webserver`（Nginx），`mysql`和`php-fpm`。

打开您的浏览器并输入`http://localhost:8081`；您应该看到 Laravel 的欢迎页面。

此时，是时候在文本编辑器中打开我们的示例项目，并检查所有的 Laravel 文件夹和文件。您可以选择您习惯使用的编辑器，或者，如果您愿意，您可以使用我们将在下一节中描述的编辑器。

# 安装 VS Code 文本编辑器

在本章和整本书中，我们将使用**Visual Studio Code**（**VS Code**），这是一个免费且高度可配置的多平台文本编辑器。它也非常适用于在 Angular 和 TypeScript 项目中使用。

按照以下步骤安装 VS Code：

1.  转到下载页面，并在[`code.visualstudio.com/Download`](https://code.visualstudio.com/Download)选择您的平台。

1.  按照您的平台的安装步骤进行操作。

VS Code 拥有一个充满活力的社区，有大量的扩展。您可以在[`marketplace.visualstudio.com/VSCode`](https://marketplace.visualstudio.com/VSCode)上研究并找到扩展。在接下来的章节中，我们将安装并使用其中一些扩展。

现在，只需从[`marketplace.visualstudio.com/items?itemName=robertohuertasm.vscode-icons`](https://marketplace.visualstudio.com/items?itemName=robertohuertasm.vscode-icons)安装 VS Code 图标。

# Laravel 应用程序的基本架构

正如之前提到的，Laravel 是用于开发现代 Web 应用程序的 MVC 框架。它是一种软件架构标准，将信息的表示与用户对其的交互分开。它采用的架构标准并不是很新；它自上世纪 70 年代中期以来就一直存在。它仍然很流行，许多框架今天仍在使用它。 

您可以在[`en.wikipedia.org/wiki/Model-view-controller`](https://en.wikipedia.org/wiki/Model-view-controller)中了解更多关于 MVC 模式的信息。

# Laravel 目录结构

现在，让我们看看如何在 Laravel 应用程序中实现这种模式：

1.  打开 VS Code 编辑器。

1.  如果这是您第一次打开 VS Code，请点击顶部菜单，然后导航到文件 | 打开。

1.  搜索`chapter-01`文件夹，并点击打开**。**

1.  在 VS Code 的左侧展开`app`文件夹。

应用程序文件如下：

![](img/f0eff070-8dd1-4a97-8c20-710ab5b5afc5.png)Laravel 根文件夹`phpdocker`文件夹和`docker-compose.yml`文件不是 Laravel 框架的一部分；我们在本章的前面手动添加了这些文件。

# MVC 流程

在一个非常基本的 MVC 工作流中，当用户与我们的应用程序交互时，将执行以下截图中的步骤。想象一个简单的关于书籍的 Web 应用程序，有一个搜索输入框。当用户输入书名并按下*Enter*时，将发生以下流程循环：

![](img/1bf73eab-92c2-4a84-978c-c5fae90f9155.png)MVC 流程

MVC 由以下文件夹和文件表示：

| **MVC 架构** | **应用程序路径** |  | **文件** |
| --- | --- | --- | --- |
| 模型 | `app/` |  | `User.php` |
| 视图 | `resources/views` |  | `welcome.blade.php` |
| 控制器 | `app/Http/Controllers` |  | `Auth/AuthController.php` `Auth/PasswordController.php` |

请注意，应用程序模型位于`app`文件夹的根目录，并且应用程序已经至少有一个文件用于 MVC 实现。

还要注意，`app`文件夹包含我们应用程序的所有核心文件。其他文件夹的名称非常直观，例如以下内容：

| 引导 | 缓存，自动加载和引导应用程序 |
| --- | --- |
| 配置 | 应用程序配置 |
| 数据库 | 工厂，迁移和种子 |
| 公共 | JavaScript，CSS，字体和图像 |
| 资源 | 视图，SASS/LESS 和本地化 |
| 存储 | 此文件夹包含分离的应用程序，框架和日志 |
| 测试 | 使用 PHPunit 进行单元测试 |
| 供应商 | Composer 依赖项 |

现在，让我们看看 Laravel 结构是如何工作的。

# Laravel 应用程序生命周期

在 Laravel 应用程序中，流程与前面的示例几乎相同，但稍微复杂一些。当用户在浏览器中触发事件时，请求到达 Web 服务器（Apache/Nginx），我们的 Web 应用程序在那里运行。因此，服务器将请求重定向到`public/index.php`，整个框架的起点。在`bootstrap`文件夹中，启动`autoloader.php`并加载由 composer 生成的所有文件，检索 Laravel 应用程序的实例。

让我们看一下以下的截图：

![](img/cb7f983b-d19f-46a1-b284-0c317a4c3ee8.png)Laravel 应用程序生命周期

该图表对于我们的第一章来说已经足够复杂了，因此我们不会详细介绍用户请求执行的所有步骤。相反，我们将继续介绍 Laravel 中的另一个非常重要的特性，即 Artisan **命令行界面（CLI）**。

您可以在官方文档的[`laravel.com/docs/5.2/lifecycle`](https://laravel.com/docs/5.2/lifecycle)中了解更多关于 Laravel 请求生命周期的信息。

# Artisan 命令行界面

现在，通过使用命令行创建 Web 应用程序是一种常见的做法；随着 Web 开发工具和技术的发展，这变得非常流行。

我们将提到 NPM 是最受欢迎的之一。但是，对于使用 Laravel 开发应用程序，我们有一个优势。当我们创建 Laravel 项目时，Artisan CLI 会自动安装。

让我们看看 Laravel 官方文档对 Artisan CLI 的说法：

Artisan 是 Laravel 附带的命令行界面的名称。它为您在开发应用程序时使用的一些有用的命令提供了帮助。

在`chapter-01`文件夹中，我们找到了 Artisan bash 文件。它负责在 CLI 上运行所有可用的命令，其中有许多命令，用于创建类、控制器、种子等等。

在对 Artisan CLI 进行了简要介绍之后，最好的事情莫过于看一些实际的例子。所以，让我们动手操作，不要忘记启动 Docker：

1.  在`chapter-01`文件夹中打开您的终端窗口，并键入以下命令：

```php
docker-compose up -d
```

1.  让我们进入`php-fpm 容器`并键入以下内容：

```php
docker-compose exec php-fpm bash
```

现在我们在终端中有所有 Artisan CLI 命令可用。

这是与我们的 Docker 容器内的 Teminal 进行交互的最简单方式。如果您正在使用其他技术来运行 Laravel 应用程序，正如本章开头所提到的，您不需要使用以下命令：

```php
docker-compose exec php-fpm bash
```

您可以在终端中键入下一步的相同命令。

1.  仍然在终端中，键入以下命令：

```php
php artisan list
```

你将看到框架版本和所有可用命令的列表：

```php
Laravel Framework version 5.2.45
Usage:
 command [options] [arguments]
Options:
 -h, --help            Display this help message
 -q, --quiet           Do not output any message
 -V, --version         Display this application version
 --ansi            Force ANSI output
 --no-ansi         Disable ANSI output
 -n, --no-interaction  Do not ask any interactive question
 --env[=ENV]       The environment the command should run under.
 -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...
```

正如您所看到的，命令列表非常长。请注意，上面的代码片段中，我们没有列出`php artisan list`命令的所有选项，但我们将在下面看到一些组合。

1.  在您的终端中，键入以下组合：

```php
php artisan -h migrate
```

输出将详细解释`migrate`命令可以做什么以及我们有哪些选项，如下面的屏幕截图所示：

![](img/81c6afbf-151a-4ae5-80f1-f7871a674726.png)输出 php artisan -h migrate

也可以看到我们对`migrate`命令有哪些选项。

1.  仍然在终端中，键入以下命令：

```php
php artisan -h make:controller
```

您将看到以下输出：

![](img/62d6573a-0b93-4cce-acd0-6e1fd7526b49.png)输出 php artisan -h make:controller

现在，让我们看看如何在 Laravel 应用程序中使用 Artisan CLI 创建 MVC。

# MVC 和路由

如前所述，我们现在将使用 Artisan CLI 分别创建模型、视图和控制器。但是，正如我们的标题所暗示的，我们将包括另一个重要项目：路由。我们已经在本章中提到过它们（在我们的 Laravel 请求生命周期图表中，以及在 MVC 本身的示例图表中）。

在本节中，我们将专注于创建文件，并在创建后检查它。

# 创建模型

让我们动手操作：

1.  在`chapter-01`文件夹中打开您的终端窗口，并键入以下命令：

```php
php artisan make:model Band
```

在命令之后，您应该看到一个绿色的成功消息，指出：模型成功创建。

1.  返回到您的代码编辑器；在`app`文件夹中，您将看到`Band.php`文件，其中包含以下代码：

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Band extends Model
{
    //
}
```

# 创建控制器

现在是使用 artisan 生成我们的控制器的时候了，让我们看看我们可以如何做到：

1.  返回到终端窗口，并键入以下命令：

```php
php artisan make:controller BandController 
```

在命令之后，您应该看到一个绿色的消息，指出：控制器成功创建。

1.  现在，在`app/Http/Controllers`中，您将看到`BandController.php`，其中包含以下内容：

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Http\Requests;
class BandController extends Controller
{
    //
}
```

作为一个良好的实践，始终使用后缀`<Somename>Controller`创建您的控制器。

# 创建视图

正如我们之前在使用`php artisan list`命令时所看到的，我们没有任何别名命令可以自动创建应用程序视图。因此，我们需要手动创建视图：

1.  返回到您的文本编辑器，并在`resources/views`文件夹中创建一个名为`band.blade.php`的新文件。

1.  将以下代码放入`band.blade.php`文件中：

```php
<div class="container">
    <div class="content">
        <div class="title">Hi i'm a view</div>
    </div>
</div>
```

# 创建路由

Laravel 中的路由负责指导来自用户请求的所有 HTTP 流量，因此路由负责 Laravel 应用程序中的整个流入，正如我们在前面的图表中看到的那样。

在本节中，我们将简要介绍 Laravel 中可用的路由类型，以及如何为我们的 MVC 组件创建一个简单的路由。

在这一点上，只需要看一下路由是如何工作的。在本书的后面，我们将深入研究应用程序路由。

因此，让我们看看在 Laravel 中可以用来处理路由的内容：

| 代码 | HTTP &#124; 方法 &#124;动词 |
| --- | --- |
| `Route::get($uri, $callback);` | 获取 |
| `Route::post($uri, $callback);` | 发布 |
| 路由::放置（$uri，$callback）; | 放置 |
| `Route::patch($uri, $callback);` | 补丁 |
| `Route::delete($uri, $callback);` | 删除 |
| `Route::options($uri, $callback);` | 选项 |

每个可用的路由都负责处理一种类型的 HTTP 请求方法。此外，我们可以在同一个路由中组合多种方法，就像下面的代码一样。现在不要太担心这个问题；我们将在本书的后面看到如何处理这种类型的路由：

```php
Route::match(['get', 'post'], '/', function () {
    //
});
```

现在，让我们创建我们的第一个路由：

1.  在文本编辑器中，打开`routes`文件夹中的`web.php`，并在`welcome view`之后添加以下代码：

```php
Route::get('/band', function () {
 return view('band');
 });
```

1.  在浏览器中打开`http://localhost:8081/band`，您将看到以下消息：

嗨，我是一个视图

不要忘记使用`docker-compose up -d`命令启动所有 Docker 容器。如果您遵循了前面的示例，您将已经拥有一切都在正常运行。

太棒了！我们已经创建了我们的第一个路由。这是一个简单的例子，但我们已经把所有东西都放在了正确的位置，并且一切都运行良好。在下一节中，我们将看看如何将模型与控制器集成并呈现视图。

# 连接到数据库

正如我们之前所看到的，控制器由路由激活，并在模型/数据库和视图之间传递信息。在前面的示例中，我们在视图中使用静态内容，但在更大的应用程序中，我们几乎总是会有来自数据库的内容，或者在控制器内生成并传递给视图的内容。

在下一个示例中，我们将看到如何做到这一点。

# 在 Docker 容器内设置数据库

现在是时候配置我们的数据库了。如果您使用 Homestead，您可能已经配置并且数据库连接正常工作。要检查，请打开终端并输入以下命令：

```php
php artisan tinker
DB::connection()->getPdo();
```

如果一切顺利，您将看到以下消息：

![](img/993a52c8-b364-46ff-9cb1-aa0283760ff7.png)数据库连接消息

然而，对于这个例子，我们正在使用 Docker，我们需要做一些配置来完成这个任务：

1.  在根项目内，打开`.env`文件并查看第 8 行（数据库连接），如下所示：

```php
 DB_CONNECTION=mysql
 DB_HOST=127.0.0.1
 DB_PORT=3306
 DB_DATABASE=homestead
 DB_USERNAME=homestead
 DB_PASSWORD=secret
```

现在，用以下行替换前面的代码：

```php
 DB_CONNECTION=mysql
 DB_HOST=mysql
 DB_PORT=3306
 DB_DATABASE=laravel-angular-book
 DB_USERNAME=laravel-angular-book
 DB_PASSWORD=123456
```

请注意，我们需要稍微更改一下以获取 Docker MySQL 容器的指示；如果您不记得在`PHPDocker.io`生成器中选择了什么，可以从容器配置中复制它。

1.  在根目录打开`docker-compose.yml`。

1.  从 MySQL 容器设置中复制环境变量：

```php
mysql:
  image: mysql:8.0
  entrypoint: ['/entrypoint.sh', '--character-set-server=utf8', '--
  collation-server=utf8_general_ci']
  container_name: larahell-mysql
  working_dir: /application
  volumes:
    - .:/application
  environment:
    - MYSQL_ROOT_PASSWORD=larahell
    - MYSQL_DATABASE=larahell-angular-book
    - MYSQL_USER=larahell-user
    - MYSQL_PASSWORD=123456
  ports:
    - "8083:3306"
```

现在是时候测试我们的连接了。

1.  在您的终端窗口中，输入以下命令：

```php
docker-compose exec php-fpm bash
```

1.  最后，让我们检查一下我们的连接；输入以下命令：

```php
php artisan tinker DB::connection()->getPdo();
```

您应该看到与上一个截图相同的消息。然后，您将拥有继续进行示例所需的一切。

# 创建迁移文件和数据库种子

**迁移**文件在一些 MVC 框架中非常常见，例如 Rails，Django 和当然，Laravel。通过这种类型的文件，我们可以使我们的数据库与我们的应用程序保持一致，因为我们无法对数据库方案进行版本控制。迁移文件帮助我们存储数据库中的每个更改，以便我们可以对这些文件进行版本控制，并保持项目的一致性。

**数据库种子**用于在数据库的表中填充一批初始记录；当我们从头开始开发 Web 应用程序时，这非常有用。初始加载的数据可以是各种各样的，从用户表到管理对象，如密码和令牌，以及我们需要的其他所有内容。

让我们看看如何在 Laravel 中为`Bands`模型创建迁移文件：

1.  打开您的终端窗口并输入以下命令：

```php
php artisan make:migration create_bands_table
```

1.  打开`database/migrations`文件夹，您将看到一个名为`<timestamp>create_bands_table.php`的文件。

1.  打开此文件，并在`public function up()`中粘贴以下代码：

```php
Schema::create('bands', function (Blueprint $table) {
   $table->increments('id');
   $table->string('name');
   $table->string('description');
   $table->timestamps();
});
```

1.  将以下代码粘贴到`public function down()`中：

```php
Schema::dropIfExists('bands');
```

1.  最终结果将是以下代码：

```php
<?php
use Illuminate\Support\Facades\Schema;
 use Illuminate\Database\Schema\Blueprint;
 use Illuminate\Database\Migrations\Migration;
class CreateBandsTable extends Migration
 {
     /**
     * Run the migrations.
     *
     * @return void
    */
     public function up()
     {
         Schema::create('bands', function (Blueprint $table) {
         $table->increments('id');
         $table->string('name');
         $table->string('description');
         $table->timestamps();
         });
     }
    /**
     * Reverse the migrations.
     *
     * @return void
     */
     public function down()
     {
         Schema::dropIfExists('bands');
     }
 }
```

1.  在`database/factories`文件夹中，打开`ModalFactory.php`文件，并在`User Factory`之后添加以下代码。请注意，我们在`factory`函数中使用了一个名为`faker`的 PHP 库，以生成一些数据：

```php
$factory->define(App\Band::class, function (Faker\Generator $faker) {
return [
 'name' => $faker->word,
 'description' => $faker->sentence
 ];
 });
```

1.  返回到您的终端窗口并创建一个数据库种子。要做到这一点，请输入以下命令：

```php
php artisan make:seeder BandsTableSeeder
```

1.  在`database/seeds`文件夹中，打开`BandsTableSeeder.php`文件，并在`public function run()`中输入以下代码：

```php
factory(App\Band::class,5)->create()->each(function ($p) {
 $p->save();
 });
```

1.  现在，在`database/seeds`文件夹中，打开`DatabaseSeeder.php`文件，并在`public function run()`中添加以下代码：

```php
$this->call(BandsTableSeeder::class);
```

您可以在[`github.com/fzaninotto/Faker`](https://github.com/fzaninotto/Faker)上阅读更多关于 Faker PHP 的信息。

在我们继续之前，我们需要对`Band`模型进行一些小的重构。

1.  在应用程序根目录中，打开`Band.php`文件并在`Band`类中添加以下代码：

```php
protected $fillable = ['name','description'];
```

1.  返回到您的终端并输入以下命令：

```php
php artisan migrate
```

在命令之后，您将在终端窗口中看到以下消息：

```php
 Migration table created successfully.
```

前面的命令只是用来填充我们的种子数据库。

1.  返回到您的终端并输入以下命令：

```php
php artisan db:seed
```

我们现在有五个项目可以在我们的数据库中使用。

让我们看看一切是否会顺利进行。

1.  在您的终端中，要退出`php-fpm 容器`，请输入以下命令：

```php
exit
```

1.  现在，在应用程序根文件夹中，在终端中输入以下命令：

```php
docker-compose exec mysql mysql -ularavel-angular-book -p123456
```

前面的命令将使您可以在`mysql Docker 容器`中访问 MySQL 控制台，几乎与我们如何访问`php-fpm 容器`相同。

1.  在终端中，输入以下命令以查看所有数据库：

```php
show databases;
```

如您所见，我们有两个表：`information_schema`和`laravel-angular-book`。

1.  让我们访问`laravel-angular-book`表；输入以下命令：

```php
use laravel-angular-book;
```

1.  现在，让我们检查我们的表，如下所示：

```php
show tables;
```

1.  现在，让我们从`bands`表中`SELECT`所有记录：

```php
SELECT * from bands;
```

我们将看到类似以下截图的内容：

![](img/6aa9f6e1-dbe6-4289-87db-c7c57e97568b.png)数据库 bands 表

1.  现在，使用以下命令退出 MySQL 控制台：

```php
exit
```

# 使用资源标志创建 CRUD 方法

让我们看看 Artisan CLI 的另一个功能，使用单个命令创建所有的**创建**、**读取**、**更新**和**删除**（CRUD）操作。

首先，在`app/Http/Controllers`文件夹中，删除`BandController.php`文件：

1.  打开您的终端窗口并输入以下命令：

```php
php artisan make:controller BandController --resource
```

这个动作将再次创建相同的文件，但现在它包括 CRUD 操作，如下面的代码所示：

```php
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
class BandController extends Controller
 {
     /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
     public function index()
     {
         //
     }
    /**
     * Show the form for creating a new resource.
     *
     * @return \Illuminate\Http\Response
     */
     public function create()
     {
         //
     }
    /**
     * Store a newly created resource in storage.
     *
     * @param \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
     public function store(Request $request)
     {
         //
     }
    /**
     * Display the specified resource.
     *
     * @param int $id
     * @return \Illuminate\Http\Response
     */
     public function show($id)
     {
         //
     }
    /**
     * Show the form for editing the specified resource.
     *
     * @param int $id
     * @return \Illuminate\Http\Response
     */
     public function edit($id)
     {
         //
     }
    /**
     * Update the specified resource in storage.
     *
     * @param \Illuminate\Http\Request $request
     * @param int $id
     * @return \Illuminate\Http\Response
     */
     public function update(Request $request, $id)
     {
         //
     }
    /**
     * Remove the specified resource from storage.
     *
     * @param int $id
     * @return \Illuminate\Http\Response
     */
     public function destroy($id)
     {
         //
     }
 }
```

在这个例子中，我们将只编写两种方法：一种用于列出所有记录，另一种用于获取特定记录。不要担心其他方法；我们将在接下来的章节中涵盖所有方法。

1.  编辑`public function index()`并添加以下代码：

```php
$bands = Band::all();
 return $bands;
```

1.  现在，编辑`public function show()`并添加以下代码：

```php
$band = Band::find($id);
 return view('bands.show', array('band' => $band));
```

1.  在`App\Http\Requests`之后添加以下行：

```php
use App\Band;
```

1.  更新`routes.php`文件，将其更改为以下代码：

```php
Route::get('/', function () {
 return view('welcome');
 });
Route::resource('bands', 'BandController');
```

1.  打开浏览器，转到`http://localhost:8081/bands`，您将看到以下内容：

```php
[{
  "id": 1,
  "name": "porro",
  "description": "Minus sapiente ut libero explicabo et voluptas harum.",
  "created_at": "2018-03-02 19:20:58",
  "updated_at": "2018-03-02 19:20:58"}
...]
```

如果你的数据与之前的代码不同，不要担心；这是由于 Faker 生成了随机数据。请注意，我们直接将 JSON 返回给浏览器，而不是将数据返回给视图。这是 Laravel 的一个非常重要的特性；它默认序列化和反序列化数据。

# 创建刀片模板引擎

现在，是时候创建另一个视图组件了。这一次，我们将使用刀片模板引擎来显示数据库中的一些记录。让我们看看官方文档对刀片的说法：

<q>刀片是 Laravel 提供的简单而强大的模板引擎。与其他流行的 PHP 模板引擎不同，刀片不限制您在视图中使用纯 PHP 代码。所有刀片视图都会被编译成纯 PHP 代码并缓存，直到被修改，这意味着刀片对您的应用基本上没有额外开销。</q>

现在，是时候看到这个行为的实际效果了：

1.  返回到代码编辑器，在`resources/views`内创建一个名为`bands`的文件夹。

1.  在`resources/views/bands`内创建一个名为`show.blade.php`的文件，并将以下代码放入其中：

```php
<h1>Band {{ $band->id }}</h1>
<ul>
<li>band: {{ $band->name }}</li>
<li>description: {{ $band->description }}</li>
</ul>
```

你可以在[`laravel.com/docs/5.2/blade`](https://laravel.com/docs/5.2/blade)了解更多关于刀片的信息。

1.  在浏览器中打开`http://localhost:8081/bands/1`。你会看到模板在运行中，结果类似以下：

![](img/88d7abed-5777-4c77-9327-8a4d51b376b6.png)模板引擎的视图

请注意，这里我们使用刀片模板引擎来显示数据库中的记录。现在，让我们创建另一个视图来渲染所有的记录。

1.  在`resources/views/bands`内创建一个名为`index.blade.php`的文件，并将以下代码放入其中：

```php
@foreach ($bands as $band)
<h1>Band id: {{ $band->id }}</h1>
<h2>Band name: {{ $band->name }}</h2>
<p>Band Description: {{ $band->description }}</p>
@endforeach
```

1.  返回到你的浏览器，访问`http://localhost:8081/bands/`，你会看到类似以下的结果：

![](img/0e5fd10e-9e6a-492e-8aee-6b1f25f78916.png)视图模板引擎

# 总结

我们终于完成了第一章，并涵盖了 Laravel 框架的许多核心概念。即使在本章讨论的简单示例中，我们也为 Laravel 的所有功能提供了相关的基础。只凭这些知识就可以创建令人难以置信的应用。但是，我们打算深入探讨一些值得单独章节的概念。在整本书中，我们将使用 RESTful API、Angular 和一些其他工具创建一个完整的应用，比如 TypeScript，我们将在下一章中讨论。
