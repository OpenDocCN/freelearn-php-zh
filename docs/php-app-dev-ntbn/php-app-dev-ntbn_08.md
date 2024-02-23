# 附录 A. 在 NetBeans 7.2 中引入 Symfony2 支持

> Symfony 是一个用于开发 Web 应用程序的 PHP 框架。它在构建 PHP 中的复杂 Web 应用程序方面非常有帮助。虽然 Symfony 是设计用于从命令行工作，但 NetBeans 7.2 对 Symfony 的支持允许您在 NetBeans 图形用户界面中使用它。

本教程演示了 NetBeans IDE 7.2 对 PHP 中 Symfony 框架的内置支持。它展示了如何设置 IDE 以使用 Symfony，如何创建使用 Symfony 框架的 PHP 项目，以及有关导航项目和设置 IDE 选项的一些提示。

# 下载和集成最新的 Symfony 标准版

Symfony 标准版是启动新项目时使用的最佳发行版。它包含最常见的 bundles，并配有一个简单的配置系统。

# 创建 Symfony2 与 NetBeans 的时间

在本节中，我们将下载标准版并将存档集成到 IDE 中。所以让我们试一试。

1.  从[`symfony.com/download`](http://symfony.com/download)下载最新的 Symfony 标准 2.x.x.zip。将`.zip`存档保存到您的磁盘上；您不需要解压`.zip`文件。

1.  检查已添加到 IDE 的所有项目的 PHP 5 解释器。选择**工具 | 选项 | PHP | 通用**，并验证**PHP 5 解释器**字段中添加的解释器路径。需要添加 PHP 解释器以从 NetBeans 运行 Symfony 命令。

1.  现在，在 IDE 中提供 Symfony 标准版（`.zip`文件）的路径。选择**工具 | 选项 | PHP | Symfony2**。浏览下载的`symfony2 .zip`存档，并按**确定**保存设置。

## 刚刚发生了什么？

IDE 将每次使用添加的`symfony2`存档来提取和转储新的 Symfony 项目。下载的框架版本包含演示 Symfony 应用程序。我们可以稍后玩这些演示应用程序，以更好地掌握 Symfony 框架。

### 注意

您可以从[`symfony.com/download`](http://symfony.com/download)中选择多个下载选项。

# 创建一个新的 Symfony2 项目

由于我们已经将 Symfony2 框架安装存档与 IDE 集成，因此创建新的 Symfony2 项目与在 NetBeans 中创建新的 PHP 项目完全相同。IDE 使用安装存档并在其中创建一个带有 Symfony 框架的新 PHP 项目。

# 创建 Symfony2 项目使用 NetBeans 的时间

我们将创建一个具有 Symfony2 框架支持的新 PHP 项目。在 IDE 创建项目目录结构之后，我们将配置我们的 Symfony2 网站。所以让我们按照以下步骤进行：

1.  以通常的方式创建一个全新的 PHP 项目，在要求选择**PHP 框架**的步骤中，勾选**Symfony2 PHP Web Framework**复选框，如下截图所示：![创建 Symfony2 项目使用 NetBeans 的时间](img/5801_Appendix_A_01.jpg)

1.  一旦在**新项目创建**对话框中单击**完成**，IDE 将生成一个新的 Symfony 项目并将提取的框架转储到其中。创建的项目目录可能类似于以下内容：![创建 Symfony2 项目使用 NetBeans 的时间](img/5801_Appendix_A_02.jpg)

1.  现在，将浏览器指向`http://localhost/symfony2/web/config.php`（将`symfony2`替换为您的项目目录名称）。新的 Symfony2 项目配置页面将类似于以下截图：![创建 Symfony2 项目使用 NetBeans 的时间](img/5801_Appendix_A_03.jpg)

您应该看到来自 Symfony 的欢迎消息，可能还有一些它检测到的问题列表。在继续之前，尝试解决**建议**部分下列出的任何主要环境问题。

1.  Symfony 框架提供了一个网站配置向导。要进入向导，请访问**Configure your Symfony Application online**链接，并为应用程序配置数据库凭据。在此页面，您可以选择您的数据库驱动程序（`MySQL - PDO`），更新您的数据库信息，如主机名、数据库名称、用户名和密码，并继续下一步。

如果您已经配置了应用程序，可以选择**Bypass configuration and go to the Welcome page**链接。

1.  在下一步中，您可以为您的 Web 应用程序生成和更新全局秘密代码（随机字母数字字符串）。此秘密代码用于安全目的，如 CSRF 保护。

1.  最后一步显示了一个成功的配置消息，例如**Your distribution is configured!**实际上，这样的配置已经覆盖了`/app/config/`目录中的`parameters.ini`文件。

1.  现在，将浏览器指向`http://localhost/symfony2/web/app_dev.php/`（将`symfony2`替换为您的项目目录名称）。新的 Symfony2 项目登陆页面将类似于以下截图：![Time for action — creating a Symfony2 project using NetBeans](img/5801_Appendix_A_04.jpg)

## 刚刚发生了什么？

我们已成功创建和配置了一个新的 Symfony 项目以及演示应用程序。Symfony2 项目的基本目录结构如下所述：

+   `app/:` 这包括应用程序配置文件、日志、缓存等。

+   `src/:` 这包括项目的 PHP 代码和您的代码所在的目录。很可能里面已经有一个演示。

+   `vendor/:` 这包括第三方依赖项。

+   `web/:` 这包括 web 根目录。

### 注意

开始使用 Symfony：

[`symfony.com/get_started`](http://symfony.com/get_started)

了解 Symfony 目录结构：

[`symfony.com/doc/current/quick_tour/the_architecture.html`](http://symfony.com/doc/current/quick_tour/the_architecture.html)

# 在 NetBeans 中运行 Symfony2 控制台命令

NetBeans IDE 支持运行 Symfony2 命令。要从 IDE 中运行命令，请从项目的上下文菜单中选择**Symfony2 | Run Command...**以启动**Run Symfony2 Command**对话框。在对话框中，您可以选择所需的 Symfony 命令并添加参数。

例如：

```php
generate:bundle [--namespace="..."] [--dir="..."] [--bundle-name="..."] [--format="..."] [--structure]

```

`generate:bundle`命令帮助您生成新的 bundle。默认情况下，该命令与开发人员交互以调整生成。任何传递的选项都将用作交互的默认值（如果遵循约定，则只需要`--namespace`）：

```php
php app/console generate:bundle --namespace=Acme/BlogBundle

```

在这里，`Acme`是您的标识符或公司名称，`BlogBundle`是以`Bundle`字符串为后缀的 bundle 名称。

## 创建一个 bundle

**bundle**类似于其他软件中的插件，但更好。关键区别在于 Symfony2 中的一切都是 bundle，包括核心框架功能和为您的应用程序编写的代码。bundle 在 Symfony2 中是一等公民。这使您可以灵活地使用打包在第三方 bundle 中的预构建功能，或者分发您自己的 bundle。这使得您可以轻松地选择要在应用程序中启用的功能，并按照您想要的方式对其进行优化。

bundle 只是一个实现单个功能的目录中的一组结构化文件。您可以创建**BlogBundle**、**ForumBundle**或用于用户管理的 bundle（许多这样的 bundle 已经存在作为开源 bundle）。每个目录包含与该功能相关的所有内容，包括 PHP 文件、模板、样式表、JavaScript、测试等。功能的每个方面都存在于 bundle 中，每个功能都存在于 bundle 中。

# 采取行动 — 使用 Symfony2 控制台命令创建一个 bundle

我们将使用 IDE 的**Run Symfony2 Command**对话框使用`generate:bundle`命令创建一个新的 bundle。所以让我们试试看...

1.  在**项目**窗格中，右键单击**项目**节点，从上下文菜单中选择**Symfony2 | Run Command...**以启动**Run Symfony2 Command**对话框，如下所示：![Time for action — creating a bundle using the Symfony2 console command](img/5801_Appendix_A_05.jpg)

您将能够在**匹配任务**框中看到可用命令的列表。您可以为这些命令添加参数，并在**命令**对话框中查看完整的命令。

1.  从前面的对话框中，选择`generate:bundle`命令，然后单击**Run**，或双击列出的名称以运行命令。IDE 的图形控制台打开以提示命名空间。![Time for action — creating a bundle using the Symfony2 console command](img/5801_Appendix_A_06.jpg)

1.  输入**Bundle namespace**的值，比如`Application/FooBundle`。

1.  输入**Bundle name**的值，或按*Enter*接受默认的 bundle 名称为`ApplicationFooBundle`。

1.  在`Target`目录处按*Enter*接受默认的 bundle 路径为`/src`。

1.  您可以输入**Configuration format**的值`(yml, xml, php`，或`annotation)`为`yml`；默认值为`annotation`。

1.  输入**Yes**以生成 bundle 的整个目录结构[no]?**，以生成 bundle 的整个目录结构；默认为 no。

1.  再次输入**Yes**确认 bundle 生成。

1.  在**确认自动更新您的内核 [yes]?**和**确认自动更新路由 [yes]?**处，按*Enter*接受默认值，即 yes。这样 bundle 就可以在 Symfony 内核中注册，并且 bundle 路由文件链接到默认的路由配置文件。

1.  现在，正如你所看到的，在`/src`目录内创建了一个新的 bundle；`bundle`目录结构看起来类似于以下内容：![Time for action — creating a bundle using the Symfony2 console command](img/5801_Appendix_A_07.jpg)

请注意，默认的控制器、路由文件、模板等与 bundle 同时创建。

1.  现在，要测试您的 bundle，请将浏览器指向`http://localhost/symfony2/web/app_dev.php/hello/tonu`，您可能会看到类似**Hello Tonu!**的输出。

1.  在`/src/Application/FooBundle/Resources/config/routing.yml`处查看 bundle 路由文件，您将看到 URL 与默认控制器的索引操作（`ApplicationFooBundle:Default:index`）映射的模式`/hello/{name}`。在这个例子中，该操作显示作为 URL 参数传递的名称，而不是`{name}`。

## 刚刚发生了什么？

每个 bundle 都托管在一个命名空间下（例如`Acme/Bundle/BlogBundle`或`Acme/BlogBundle`）。命名空间应以“供应商”名称开头，例如您的公司名称、项目名称或客户名称，后面跟着一个或多个可选的类别子命名空间，最后以 bundle 名称本身结尾（必须以`Bundle`作为后缀）。

### 注意

请参阅`http://symfony.com/doc/current/cookbook/bundles/best_practices.html#index-1`，了解有关 bundle 命名约定的更多详细信息。

我们已经看到了交互式控制台，它要求参数并自动创建整个`bundle`目录结构。此外，它将 bundle 注册到 Symfony 的`/app/AppKernel.php`中，并将 bundle 路由配置文件链接到`default/app/config/routing.yml`中。

### 注意

Symfony 学习资源：

[`symfony.com/doc/current/book/index.html`](http://symfony.com/doc/current/book/index.html)
