# 前言

自其诞生以来，Web 开发已经走过了很长的路。今天，我们希望的是快速、强大和引人入胜的 Web 应用程序，而渐进式 Web 应用程序（PWA）是前进的道路。在这本书中，我们将利用 Angular 和 Laravel 这两个最流行的框架来构建强大的 Web 应用程序。

Angular 是用于创建现代快速 PWA 的最流行的前端 JavaScript 框架之一。除了非常多才多艺和完整之外，Angular 还包括用于生成模块、组件、服务和许多其他实用工具的 Angular CLI 工具。另一方面，我们有 Laravel 框架，这是用于开发 Web 应用程序的强大工具，探讨了约定优于配置的范式的使用。

这本书将为您提供从头开始使用 Angular 和 Laravel RESTful 后端构建现代全栈 Web 应用程序的实际知识。它将带您了解使用这两个框架开发的最重要的技术方面，并演示如何将这些技能付诸实践。

# 这本书是为谁准备的

这本书适用于初学 Angular 和 Laravel 的开发人员。需要了解 HTML、CSS 和 JavaScript 和 PHP 等脚本语言的知识。

本书的内容涵盖了软件工程生命周期的所有阶段，涵盖了现代工具和技术，包括但不限于 RESTful API、基于令牌的身份验证、数据库配置以及 Docker 容器和镜像。

# 本书涵盖了什么

第一章，*理解 Laravel 5 的核心概念*，介绍了 Laravel 框架作为开发 Web 应用程序的强大工具，并探讨了约定优于配置的范式的使用。我们将看到，Laravel 默认情况下具有构建现代 Web 应用程序所需的所有功能，包括基于令牌的身份验证、路由、资源等。此外，我们将了解为什么 Laravel 框架是当今最流行的 PHP 框架之一。我们将学习如何设置环境，了解 Laravel 应用程序的生命周期，并学习如何使用 Artisan CLI。

第二章，*TypeScript 的好处*，探讨了 TypeScript 如何使您能够编写一致的 JavaScript 代码。我们将研究它包括的功能，例如静态类型和其他在面向对象语言中非常常见的功能。此外，我们将研究如何使用最新版本的 ECMAScript 的新功能，并了解 TypeScript 如何帮助我们编写干净和组织良好的代码。在本章中，我们将看到 TypeScript 相对于传统 JavaScript 的好处，了解如何使用静态类型，并理解如何使用接口、类和泛型，以及导入和导出类。

第三章，*理解 Angular 6 的核心概念*，深入探讨了 Angular，这是用于开发前端 Web 应用程序的最流行的框架之一。除了非常多才多艺和完整之外，Angular 还包括用于生成模块、组件、服务和许多其他实用工具的 Angular CLI 工具。在本章中，我们将学习如何使用新版本的 Angular CLI，理解 Angular 的核心概念，并掌握组件的生命周期。

第四章，“构建基线后端应用程序”，是我们开始构建示例应用程序的地方。在本章中，我们将使用 RESTful 架构创建一个 Laravel 应用程序。我们将更仔细地研究一些在第一章中简要提到的要点，例如使用 Docker 容器来配置我们的环境，以及如何保持我们的数据库填充。我们甚至将查看如何使用 MySQL Docker 容器，如何使用迁移和数据库种子，以及如何使用 Swagger UI 创建一致的文档。

第五章，“使用 Laravel 创建 RESTful API - 第 1 部分”，将介绍 RESTful API。您将学习如何使用 Laravel 框架的核心元素构建 RESTful API - 控制器，路由和 eloquent 对象关系映射（ORM）。我们还展示了我们正在构建的应用程序的一些基本线框。此外，我们将更仔细地研究一些您需要熟悉的关系，例如一对一，一对多和多对多。

第六章，“使用 Laravel 创建 RESTful API - 第 2 部分”，继续我们构建示例 API 的项目，尽管在那时，我们在 Laravel 中仍有很长的路要走。我们将学习如何使用一些在 Web 应用程序中非常常见的功能，例如基于令牌的身份验证，请求验证和自定义错误消息；我们还将看到如何使用 Laravel 资源。此外，我们将看到如何使用 Swagger 文档来测试我们的 API。

第七章，“使用 Angular CLI 创建渐进式 Web 应用程序”，涵盖了自上一个 Angular 版本以来影响 angular-cli.json 的变化。angular-cli.json 文件现在改进了对多个应用程序的支持。我们将看到如何使用*ng add*命令创建 PWA，以及如何组织我们的项目结构，以留下一个可扩展项目的单一基础。此外，我们将看到如何使用 Angular CLI 创建 service-work 和清单文件。

第八章，“处理 Angular 路由器和组件”，是单页应用程序（SPA）中最重要的部分之一，即路由的使用。幸运的是，Angular 框架提供了一个强大的工具来处理应用程序路由：@angular/router 依赖项。在本章中，我们将学习如何使用其中一些功能，例如路由器出口和子视图，以及如何创建主细节页面。此外，我们将开始创建前端视图。

第九章，“创建服务和用户身份验证”，我们将创建许多新东西，并进行一些重构以记忆重要细节。这是以常规和渐进的方式学习新知识的好方法。此外，我们将深入研究 Angular 框架的 HTTP 模块的操作和使用，现在称为 httpClient。此外，我们将研究拦截器，处理错误，使用授权标头以及如何使用 r*oute guards*来保护应用程序路由。

第十章，“使用 Bootstrap 4 和 NgBootstrap 创建前端视图”，解释了如何使用 Angular CLI 的新*ng add*命令在运行中的 Angular 应用程序中包含 Bootstrap CSS 框架和 NgBootstrap 组件。此外，我们将看到如何将我们的 Angular 服务与组件连接起来，以及如何使用后端 API 将它们整合在一起。我们将学习在后端 API 上配置 CORS，以及如何在我们的 Angular 客户端应用程序中使用它。我们还将学习处理 Angular 管道，模板驱动表单，模型驱动表单和表单验证。

第十一章，*构建和部署 Angular 测试*，介绍了如何安装、自定义和扩展 Bootstrap CSS 框架，以及如何使用 NgBootstrap 组件以及如何将 Angular 服务与组件和 UI 界面连接。我们将学习编写 Angular 单元测试，配置应用程序的 linter（用于 SCSS 和 Tslint）以保持代码一致性，创建 NPM 脚本，以及创建 Docker 镜像并部署应用程序。

# 充分利用本书

一些命令行、Docker 和 MySQL 的知识将非常有帮助；但是，这并非完全必需，因为所有命令和示例都附有简要说明。

您需要在您的计算机上安装以下工具：

+   Node.js 和 NPM

+   Docker

+   代码编辑器——我们建议您使用 Visual Studio Code

+   推荐使用 Git 源代码控制，但不是必需的

# 下载示例代码文件

您可以从您在[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误表”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的以下工具解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Full-Stack-Web-Development-with-Angular-6-and-Laravel-5`](https://github.com/PacktPublishing/Hands-On-Full-Stack-Web-Development-with-Angular-6-and-Laravel-5)。如果代码有更新，将在现有的 GitHub 存储库中进行更新。

我们还有来自我们丰富书籍和视频目录的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载它[`www.packtpub.com/sites/default/files/downloads/HandsOnFullStackWebDevelopmentwithAngular6andLaravel5_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/HandsOnFullStackWebDevelopmentwithAngular6andLaravel5_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这里

这是一个例子：“所有使用 Composer 的 PHP 项目在根项目中都有一个名为`composer.json`的文件。”

代码块设置如下：

```php
{
 "require": {
     "laravel/framework": "5.*.*",
 }
}
```

任何命令行输入或输出都是这样写的：

```php
composer create-project --prefer-dist laravel/laravel chapter-01
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这样的形式出现在文本中。这是一个例子：“

“搜索`chapter-01`文件夹，然后点击打开。”

警告或重要说明会出现在这样。提示和技巧会出现在这样。
