# 前言

Phalcon 的开发者将开发最快、最有效的 PHP 框架作为他们的使命，并且他们很好地完成了这个使命。Phalcon 结合了 C 编程的速度、PHP 的简洁性和框架的结构，简化了 Web 应用开发。

*Phalcon 入门*是使用 Phalcon 开发 Web 应用的介绍。您将通过构建一个博客应用来学习，使用 Phalcon 开发者工具和网络工具快速构建应用的 CRUD 骨架，然后修改它以添加功能。

随着博客功能的增加，您将学习如何使用其他 Phalcon 功能，如 Volt 模板引擎、视图助手、PHQL、验证、加密、cookie、会话和事件管理。

# 本书涵盖内容

第一章, *安装 Phalcon*，涵盖了在 Linux、Windows 或 Mac 上安装 Phalcon PHP 扩展，以及配置 PHP、Apache 或 Nginx。

第二章, *设置 Phalcon 项目*，涵盖了使用 Phalcon 开发者工具或 Phalcon 网络工具手动设置 Phalcon 项目。

第三章, *使用 Phalcon 模型、视图和控制器*，涵盖了 Phalcon 的 MVC 结构。

第四章, *在 Phalcon 中处理数据*，深入探讨了 Phalcon 模型、PHQL、会话数据和过滤及清理数据。

第五章, *使用 Phalcon 特性*，涵盖了 Phalcon 的更多特性，包括密码散列、控制用户访问、设置 cookie 以及使用视图部分、日志和视图助手。

# 本书面向对象

本书旨在为想要学习如何使用 Phalcon PHP 框架的 PHP 开发者编写。需要一些 PHP 知识，但不需要 MVC 框架的先验知识。

# 规范

在本书中，您将找到多种文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“因此，我们必须定位`php.ini`文件并对其进行编辑。”

代码块按如下方式设置：

```php
phalconBlog/
  app/
    config/
    controllers/
    library/
    logs/
    models/
    plugins/
    views/
      index/
      layouts/
  public/
    css/
    files/
    img/
    js/
    temp/
```

任何命令行输入或输出都按如下方式编写：

```php
sudo apt-get install git
sudo apt-get php5-devphp-mysqlgcc

```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中显示如下：“因此，在您的浏览器窗口中搜索单词**编译器**。”

### 注意

警告或重要提示以如下框中显示。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要发送一般反馈，只需发送一封电子邮件到<mailto:feedback@packtpub.com>，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册以将文件直接通过电子邮件发送给您。您还可以通过访问 GitHub 页面[`eristoddle.github.io/phalconBlog/`](http://eristoddle.github.io/phalconBlog/)找到书中使用的所有代码。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现了错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详情来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误清单中。

## 盗版

互联网上版权材料的盗版是所有媒体中持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接<mailto:copyright@packtpub.com>与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果您在书的任何方面遇到问题，可以通过<mailto:questions@packtpub.com>与我们联系，我们将尽力解决。
