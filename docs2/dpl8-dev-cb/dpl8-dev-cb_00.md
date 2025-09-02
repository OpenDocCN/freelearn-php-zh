# 前言

Drupal 是一个用于构建小型企业、电子商务、企业系统等网站的内容管理系统。由超过 4,500 名贡献者创建，Drupal 8 为 Drupal 带来了许多新功能。无论你是 Drupal 的新手，还是经验丰富的 Drupalista，*Drupal 8 开发食谱*包含了深入探索 Drupal 8 所提供内容的食谱。

# 本书涵盖内容

第一章，*与 Drupal 8 一起启动*，首先介绍了运行 Drupal 8 的要求，然后介绍了安装过程和扩展 Drupal。

第二章，*内容创作体验*，深入探讨了 Drupal 中的内容管理体验，包括使用新捆绑的 CKEditor。

第三章，*通过视图显示内容*，探讨了如何使用视图在 Drupal 中创建不同的方式来列出和显示你的内容。

第四章，*扩展 Drupal*，介绍了如何为 Drupal 编写模块，这是 Drupal 中功能构建块。

第五章，*前端为王*，涵盖了如何创建主题，使用新的模板系统 Twig，以及利用 Drupal 的响应式设计功能。

第六章，*使用表单 API 创建表单*，解释了如何使用 Drupal 的表单 API 创建用于收集数据的自定义表单。

第七章，*插件即插即用*，介绍了插件，这是 Drupal 中最新的组件之一。本章将指导你开发插件系统以与字段一起工作。

第八章，*多语言和国际化*，介绍了 Drupal 8 提供的功能，以创建一个国际化的网站，支持内容和管理的多语言。

第九章，*配置管理 - 在 Drupal 8 中部署*，解释了 Drupal 8 中引入的配置管理系统以及如何导入和导出网站配置。

第十章，*实体 API*，深入探讨了 Drupal 中的实体 API，允许你创建自定义配置和内容实体。

第十一章，*走出 Drupalicon 岛*，解释了 Drupal 如何允许采用“自豪地构建在其他地方”的口号，并在你的 Drupal 网站上包含第三方库。

第十二章，*网络服务*，展示了如何通过 RESTful 接口将你的 Drupal 8 网站转变为网络服务 API 提供者。

第十三章，*Drupal CLI*，探讨了通过 Drupal 社区创建的两个命令行工具（Drush 和 Drupal Console）与 Drupal 8 一起工作。

# 你需要为此书准备的内容

为了使用 Drupal 8 并运行本书中的代码示例，以下软件将是必需的：

Web 服务器软件堆栈：

+   Web 服务器：Apache（推荐）、Nginx 或 Microsoft IIS

+   数据库：MySQL 5.5 或 MariaDB 5.5.20 或更高版本

+   PHP：PHP 5.5.9 或更高版本

第一章节详细介绍了所有这些要求，并包含了一个突出显示即插即用开发服务器设置的食谱。

您还需要一个文本编辑器；以下是一些流行编辑器和 IDE 的建议：

+   Atom.io 编辑器，[`atom.io/`](https://atom.io/)

+   Visual Code Studio, [`code.visualstudio.com/`](https://code.visualstudio.com/)

+   PHPStorm（特定 Drupal 集成），[`www.jetbrains.com/phpstorm/`](https://www.jetbrains.com/phpstorm/)

+   配置 Drupal 的 Vim，[`www.drupal.org/project/vimrc`](https://www.drupal.org/project/vimrc)

+   您操作系统的默认文本编辑器或命令行文件编辑器

# 这本书面向的对象

这本书是为那些已经在使用 Drupal 的人编写的，例如网站构建者、后端和前端开发者，以及那些渴望了解当他们开始使用 Drupal 8 时将面临什么的人。

# 部分

在本书中，您将找到几个频繁出现的标题（准备工作、如何操作、它是如何工作的、还有更多、参见等）。

为了清楚地说明如何完成食谱，我们使用以下部分如下：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

# 如何操作…

本节包含遵循食谱所需的步骤。

# 它是如何工作的…

本节通常包含对上一节发生情况的详细解释。

# 还有更多…

本节包含有关食谱的附加信息，以便使读者对食谱有更深入的了解。

# 参见

本节提供了对其他有用信息的链接，以帮助读者了解食谱。

# 惯例

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示：“您将看到 `drupal-org.make` 和 `drupal-org-core.make`。”

代码块设置如下：

```php
 public function alterRoutes(RouteCollection $collection) { 
    // Change path of mymodule.mypage to use a hyphen 
    if ($route = $collection->get('mymodule.mypage')) 
```

任何命令行输入或输出都如下所示：

```php
 $ CREATE USER username@localhost IDENTIFIED BY 'password';
```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中显示如下：“勾选复选框并点击安装。”

警告或重要注意事项如下所示。

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要向我们发送一般反馈，请简单地将邮件发送到 `feedback@packtpub.com`，并在邮件主题中提及书籍的标题。

如果你在某个领域有专业知识，并且对撰写或参与书籍的编写感兴趣，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助你从购买中获得最大收益。

# 下载示例代码

您可以从您的账户中下载此书的示例代码文件，网址为 [`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  点击“代码下载与错误清单”。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买此书籍的来源。

1.  点击“代码下载”。

您也可以通过点击 Packt Publishing 网站上书籍网页上的“代码文件”按钮来下载代码文件。您可以通过在搜索框中输入书籍名称来访问此页面。请注意，您需要登录您的 Packt 账户。

下载文件后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   Windows 系统下的 WinRAR / 7-Zip。

+   Mac 系统下的 Zipeg / iZip / UnRarX。

+   Linux 系统下的 7-Zip / PeaZip。

该书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Drupal-8-Development-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Drupal-8-Development-Cookbook-Second-Edition)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，网址为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。去看看吧！

# 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分现有的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分。

# 盗版

互联网上版权材料的盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`copyright@packtpub.com`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，您可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
