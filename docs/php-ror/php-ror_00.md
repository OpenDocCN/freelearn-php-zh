# 前言

嗨，大家好！*从 PHP 到 Ruby 再到 Rails* 是一本针对有 PHP 背景和理解的读者的指南，他们希望将自己的知识扩展到另一种面向对象的编程语言：Ruby。Ruby 是一种由 Yukihiro “Matz” Matsumoto 最初创建并在 1995 年公开发布的编程语言。自从其诞生以来，Ruby 一直是一种非常冗长的语言，开源社区也采纳了这一特性，使得 Ruby 程序在每行代码的意图上非常明确。本指南不仅将向您介绍 Ruby 的语言特性，还将帮助您进入 Ruby 应用程序开发的思维模式。在学习 Ruby 的这条路上，您还将学习 Ruby on Rails 的基础知识，这是一个由 David Heinemeier Hansson 开发并在 2004 年作为开源工具发布的用于创建 Web 应用的框架。随着 Ruby 和 Ruby on Rails 在技术公司中的日益流行和采用，全球对 Ruby 开发者的需求也在不断增加。无论您对 Ruby 和 Ruby on Rails 感兴趣，还是想了解 Ruby 是否适合您，这本指南都是为您准备的。

# 本书面向对象

本书面向那些已经有一定编程经验，并希望利用这些经验和知识来学习 Ruby 编程语言和思维模式的人。

从本书中受益最大的三种类型的人如下：

+   使用过其他编程语言的开发者。

+   有 PHP 工作经验的开发者，希望学习另一种编程语言。他们将通过示例和等效 PHP 代码的审查来获得 Ruby 语言的知识。

+   有任何 PHP 框架（Laravel、CodeIgniter、Symfony、CakePHP 等）工作经验的开发者。本书涵盖了 Ruby on Rails 框架。熟悉任何 Web 框架的人通过学习 Ruby on Rails 的**模型-视图-控制器**（**MVC**）实现将受益匪浅。

# 本书涵盖内容

*第一章*，*理解 Ruby 思维方式和文化*，介绍了 Ruby 开发者思考问题和编写代码的方式。

*第二章*，*设置我们的本地环境*，提供了一篇简短的指南，介绍如何安装 Ruby，以便您能够跟随本书中的练习。

*第三章*，*比较基本的 Ruby 语法与 PHP*，通过比较 PHP 和 Ruby 语法来帮助您轻松进入这门语言，了解它们的相似之处和不同之处。

*第四章*，*Ruby 脚本与 PHP 脚本的比较*，旨在利用 Ruby 的语法来编写更易读的代码。

*第五章*，*库和类语法*，介绍了 Ruby 的库（gem）及其安装和使用。本章还介绍了 Ruby 领域中的面向对象编程。

*第六章*，*Ruby 调试*，介绍了 Ruby 中可用于修复我们在运行时可能遇到的脚本中的错误和错误的工具。本章还提供了安装和使用这些工具的指南。

*第七章*，*理解配置优于约定*，介绍了 Ruby on Rails 网络框架，其安装以及使用它的最简单示例。

*第八章*，*模型、数据库和 Active Record*，介绍了通过模型使用 Ruby on Rails 处理数据库。它还涵盖了使用 `ActiveRecord` 的数据库操作的基本知识。

*第九章*，*整合一切*，提供了一篇指南，通过一个更实际的示例来生成一个简单的 Ruby on Rails 应用程序，使用我们在书中学到的一切。

*第十章*，*托管 Rails 应用程序与 PHP 应用程序考虑因素*，提供了一篇简短的指南，介绍了在现实场景，即生产环境中发布 Rails 应用程序时必须考虑的因素。

# 要充分利用这本书

您需要安装 Ruby 3.1.1（如有需要，请遵循书中的说明来设置 Ruby）。您还需要在您的计算机上安装 Git，因为 Ruby 库（gem）依赖于 git 来获取其源代码。如果您使用 macOS，您将需要安装 Xcode 命令行工具或 Xcode 本身，然后才能使用 Ruby。虽然不是强制性的，但您也应该安装 `rbenv` 以允许您安装不同的 Ruby 版本。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Ruby | Windows、macOS 或 Linux |
| Ruby on Rails | Windows、macOS 或 Linux |

对于 Linux 用户，您需要使用以下命令（或根据您使用的发行版适当的等效命令）安装一系列库：

```php
sudo yum install git-core zlib zlib-devel gcc-c++ patch readline readline-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison curl sqlite-devel
```

**如果您使用的是这本书的数字版，我们建议您亲自输入代码或从书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails`](https://github.com/PacktPublishing/From-PHP-to-Ruby-on-Rails)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“在 Ruby 中，我们实际上没有`var_dump()`函数，而是每个对象都有一个名为`inspect()`的现有方法。”

代码块如下设置：

```php
require 'oj'
json_text = '{"name":"Sarah Kerrigan", "age":23, "human":true}'
ruby_hash = Oj.load(json_text)
puts ruby_hash
puts ruby_hash["name"]
```

任何命令行输入或输出都应如下所示：

```php
gem uninstall oj
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“当您点击**注册**按钮时，您应该立即看到您在重定向之前试图浏览的页面。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件发送至 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将非常感谢。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将非常感谢。请通过 copyright@packtpub.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《从 PHP 到 Ruby on Rails》，我们很乐意听听您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1804610097)并分享您的反馈。

您的评论对我们和科技社区都非常重要，它将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在移动中阅读，但无法携带您的印刷书籍到处走吗？

您的电子书购买是否与您选择的设备不兼容？

不要担心，现在，每购买一本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不止于此，您还可以获得独家折扣、时事通讯和每日免费内容的专属访问权限

按照以下简单步骤获取好处：

1.  扫描下面的二维码或访问以下链接

![](img/B19230_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781804610091`](https://packt.link/free-ebook/9781804610091)

1.  提交您的购买证明

1.  就这样！我们将直接将您的免费 PDF 和其他优惠发送到您的电子邮件

# 第一部分：从 PHP 到 Ruby 基础

在本部分中，您将了解 Ruby 的全套做事方式，同时将其与您使用 PHP 做事的方式进行比较。此外，您还将学习如何使用 Ruby 在语法、库（称为 gem）和调试工具方面的优势，使故障排除尽可能无痛。

本部分包含以下章节：

+   *第一章*，*理解 Ruby 思维方式和文化*

+   *第二章*，*设置我们的本地环境*

+   *第三章*，*比较基本的 Ruby 语法与 PHP*

+   *第四章*，*Ruby 脚本与 PHP 脚本比较*

+   *第五章*，*库和类语法*

+   *第六章*，*Ruby 调试*
